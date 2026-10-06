# TypeScript client

This client follows the rules in `SKILL.md`. It uses the built-in `fetch`, so
it needs no package. Adapt names & style to the project. Keep the rules.

## Contents

- Client
- Usage
- Smoke test

## Client

```ts
import { randomUUID } from "node:crypto";

const BASE_URL = "https://xquik.com/api/v1";
const REQUEST_TIMEOUT_MS = 30_000;
const RETRY_BUDGET_MS = 300_000;
const MAX_BACKOFF_MS = 30_000;
const DEFAULT_POLL_MS = 2_000;
const RESTART_CODES = new Set(["coverage_cursor_gone", "invalid_coverage_cursor"]);

type Json = Record<string, unknown>;
type Query = Record<string, string | number | boolean | undefined>;

export class XquikError extends Error {
  readonly status: number;
  readonly code: string;
  readonly retryAfterSeconds: number | undefined;

  constructor(status: number, code: string, message: string, retryAfterSeconds?: number) {
    super(message);
    this.name = "XquikError";
    this.status = status;
    this.code = code;
    this.retryAfterSeconds = retryAfterSeconds;
  }
}

function apiKey(): string {
  const key = process.env.XQUIK_API_KEY?.trim();
  if (!key) throw new Error("Set the XQUIK_API_KEY environment variable.");
  return key;
}

function isJson(value: unknown): value is Json {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

const sleep = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms));

/** Sends 1 request. `path` starts after `/api/v1`, or is a `statusUrl`. */
async function send(
  method: string,
  path: string,
  options: { query?: Query; body?: Json; idempotencyKey?: string } = {},
): Promise<Json> {
  const url = new URL(path.startsWith("/api/v1/") ? path : BASE_URL + path, BASE_URL);
  if (url.origin !== new URL(BASE_URL).origin) throw new Error("Refusing to send the key to another host.");
  for (const [name, value] of Object.entries(options.query ?? {})) {
    if (value !== undefined) url.searchParams.set(name, String(value));
  }
  const headers: Record<string, string> = { "x-api-key": apiKey() };
  if (options.body) headers["content-type"] = "application/json";
  if (options.idempotencyKey) headers["idempotency-key"] = options.idempotencyKey;
  const init: RequestInit = {
    method,
    headers,
    redirect: "manual",
    signal: AbortSignal.timeout(REQUEST_TIMEOUT_MS),
  };
  if (options.body) init.body = JSON.stringify(options.body);
  let response: Response;
  try {
    response = await fetch(url, init);
  } catch {
    throw new XquikError(0, "network_error", `${method} ${path} did not complete.`);
  }
  let body: unknown;
  try {
    body = JSON.parse(await response.text());
  } catch {
    body = undefined;
  }
  if (response.ok && isJson(body)) return body;
  const code = isJson(body) && typeof body.error === "string" ? body.error : "invalid_response";
  const retryAfter = Number(response.headers.get("retry-after") ?? "");
  throw new XquikError(
    response.status,
    code,
    `${method} ${path} failed with ${response.status} ${code}.`,
    Number.isFinite(retryAfter) && retryAfter > 0 ? retryAfter : undefined,
  );
}

function isRetryable(error: XquikError): boolean {
  return (
    error.status === 0 ||
    error.status === 429 ||
    error.status >= 500 ||
    error.code === "coverage_cursor_unavailable" ||
    (error.code === "invalid_response" && error.status < 300)
  );
}

/** GET with retries. `path` starts after `/api/v1`, such as `/x/tweets/search`. */
export async function xquikGet(path: string, query: Query = {}): Promise<Json> {
  const deadline = Date.now() + RETRY_BUDGET_MS;
  for (let attempt = 0; ; attempt += 1) {
    try {
      return await send("GET", path, { query });
    } catch (error) {
      if (!(error instanceof XquikError) || !isRetryable(error)) throw error;
      const backoff = Math.min(MAX_BACKOFF_MS, 1_000 * 2 ** attempt) * (0.5 + Math.random() / 2);
      const wait = error.retryAfterSeconds ? error.retryAfterSeconds * 1_000 : backoff;
      if (Date.now() + wait > deadline) throw error;
      await sleep(wait);
    }
  }
}

/**
 * Yields each row once. `rowsKey` is `tweets` or `users`. The query holds the
 * page size, so the ceiling is that page size times `maxPages`, in rows.
 */
export async function* paginate(
  path: string,
  query: Query,
  rowsKey: "tweets" | "users",
  maxPages: number,
): AsyncGenerator<Json> {
  const seenIds = new Set<string>();
  const seenCursors = new Set<string>();
  let cursor: string | undefined;
  let restarted = false;
  for (let page = 0; page < maxPages; page += 1) {
    let body: Json;
    try {
      body = await xquikGet(path, { ...query, cursor });
    } catch (error) {
      const canRestart =
        error instanceof XquikError && RESTART_CODES.has(error.code) && cursor !== undefined && !restarted;
      if (!canRestart) throw error;
      restarted = true;
      cursor = undefined;
      seenCursors.clear();
      continue;
    }
    const value = body[rowsKey];
    const rows: unknown[] = Array.isArray(value) ? value : [];
    for (const row of rows) {
      if (isJson(row) && typeof row.id === "string" && !seenIds.has(row.id)) {
        seenIds.add(row.id);
        yield row;
      }
    }
    const next = body.next_cursor;
    if (body.has_next_page !== true || typeof next !== "string" || next === "") return;
    if (seenCursors.has(next)) throw new XquikError(0, "repeated_cursor", `${path} repeated a cursor.`);
    seenCursors.add(next);
    cursor = next;
  }
}

/**
 * Sends 1 write & polls it to its final state. Pass the same `idempotencyKey`
 * to resend the identical request after a timeout or a `429`.
 */
export async function xquikWrite(
  method: "POST" | "PATCH" | "DELETE",
  path: string,
  body: Json,
  idempotencyKey: string = randomUUID(),
): Promise<Json> {
  let action = await send(method, path, { body, idempotencyKey });
  const deadline = Date.now() + RETRY_BUDGET_MS;
  while (action.terminal !== true && Date.now() < deadline) {
    const statusUrl = action.statusUrl;
    if (typeof statusUrl !== "string") break;
    await sleep(typeof action.pollAfterMs === "number" ? action.pollAfterMs : DEFAULT_POLL_MS);
    action = await xquikGet(statusUrl);
  }
  return action;
}
```

## Usage

```ts
const query = { q: "solar panels", language: "en", retweets: "exclude", limit: 100 };
for await (const tweet of paginate("/x/tweets/search", query, "tweets", 10)) {
  await saveTweet(tweet);
}

const action = await xquikWrite("POST", "/x/tweets", { account: "my_brand", text: "Hello" });
if (action.status !== "success") throw new Error(`The write ended as ${String(action.status)}.`);
```

That search reads at most 100 rows times 10 pages. So it reads at most 1,000
tweets, which cost $0.15.

## Smoke test

The developer runs this smoke test. It proves the endpoint & the field names.
It prints counts & names, never tweet text. This one reads at most 5 tweets,
which cost $0.00075.

```ts
const page = await xquikGet("/x/tweets/search", { q: "from:nasa", limit: 5 });
const rows = Array.isArray(page.tweets) ? page.tweets : [];
const first: unknown = rows[0];
console.log({
  rows: rows.length,
  fields: typeof first === "object" && first !== null ? Object.keys(first).sort() : [],
  hasNextPage: page.has_next_page,
});
```
