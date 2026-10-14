# Monitors, events & webhooks

A monitor watches an account or a search query. It stores each match as an
event. The application reads events by polling, or receives them at an HTTPS
webhook.

## Contents

- Replace a stream
- Cost
- Monitors
- Events
- Webhooks
- Verify a delivery
- Receiver rules

## Replace a stream

| Official X API v2 | Xquik |
| --- | --- |
| A rule on `POST /2/tweets/search/stream/rules` | A keyword monitor with `POST /monitors/keywords` |
| The held connection on `GET /2/tweets/search/stream` | A webhook with `POST /webhooks`, or polling `GET /events` |
| Polling `GET /2/users/:id/tweets` for new tweets | An account monitor with `POST /monitors` |
| Reconnect & backfill code | Remove it. Xquik stores events & retries deliveries |

## Cost

- An active monitor costs 21 credits per hour, so $0.00315.
- That price includes its events & webhook deliveries.
- Creating a monitor needs 22 available credits.
- An account monitor also costs 1 credit for the username lookup.
- Reading stored events & managing webhooks cost nothing.
- A paused or deleted monitor costs nothing.

Write the setup as a script the developer runs. State the hourly cost & the
pause call before they run it.

## Monitors

```http
POST https://xquik.com/api/v1/monitors
x-api-key: <XQUIK_API_KEY>
Content-Type: application/json

{"username": "nasa", "eventTypes": ["tweet.new"]}
```

```http
POST https://xquik.com/api/v1/monitors/keywords
x-api-key: <XQUIK_API_KEY>
Content-Type: application/json

{"query": "#rustlang", "eventTypes": ["tweet.new"]}
```

Both return `id`, `eventTypes`, `isActive`, `createdAt` & `nextBillingAt`.

| Event group | Event types |
| --- | --- |
| Tweets | `tweet.new`, `tweet.reply`, `tweet.retweet`, `tweet.quote` |
| Tweet content | `tweet.media`, `tweet.link`, `tweet.poll`, `tweet.mention`, `tweet.hashtag`, `tweet.longform` |
| Profile changes | `profile.bio.changed`, `profile.name.changed`, `profile.avatar.changed`, `profile.username.changed` |

Xquik stores & delivers only the subscribed types. `tweet.new` covers
original tweets only. An account monitor watches that account's own tweets &
profile. A keyword monitor with a query such as `@my_brand` catches mentions
from other accounts.

| Action | Account monitor | Keyword monitor |
| --- | --- | --- |
| List | `GET /monitors` | `GET /monitors/keywords` |
| Pause or resume | `PATCH /monitors/{id}` with `{"isActive": false}` | `PATCH /monitors/keywords/{id}` with `{"isActive": false}` |
| Delete, which also erases stored events | `DELETE /monitors/{id}` | `DELETE /monitors/keywords/{id}` |

## Events

`GET /events?monitorId=<id>&limit=50` or `GET /events?keywordMonitorId=<id>`.
`limit` is 1 to 100. Filter with `eventType`.

The response holds `events`, `hasMore` & `nextCursor`. Send `nextCursor` back
as `cursor`. Each event has `id`, `type`, `monitorId`, `monitorType`,
`occurredAt` & `data`. Events come newest first.

A scheduled poll requests the first page without a cursor. It stops at the
first event `id` it already handled.

## Webhooks

`POST /webhooks` with `{"url": "https://...", "eventTypes": [...]}`. The URL
must use HTTPS. The response holds `id`, `url`, `eventTypes`, `createdAt` &
`secret`. Xquik returns `secret` once.

The setup code stores `secret` in the secret manager or as
`XQUIK_WEBHOOK_SECRET`. It never prints the value.

| Action | Endpoint |
| --- | --- |
| Send a signed test | `POST /webhooks/{id}/test`, which returns `success` & `statusCode` |
| List delivery attempts | `GET /webhooks/{id}/deliveries` |
| Pause | `PATCH /webhooks/{id}` with `{"isActive": false}` |
| Deactivate | `DELETE /webhooks/{id}` |

## Verify a delivery

Each delivery carries 3 headers.

| Header | Value |
| --- | --- |
| `X-Xquik-Timestamp` | Unix time in milliseconds |
| `X-Xquik-Nonce` | 16 random bytes in hex |
| `X-Xquik-Signature` | `sha256=` plus the hex HMAC-SHA256 of `<timestamp>.<nonce>.<raw body>`, keyed with the webhook secret |

The JSON body holds `eventType`, `streamEventId`, `deliveryId`, `occurredAt`,
`monitorId`, `monitorType` & `data`. Account monitors add `username` &
`xUserId`. Keyword monitors add `query`. A test delivery holds
`eventType: "webhook.test"` & no `streamEventId`.

```ts
import { createHmac, timingSafeEqual } from "node:crypto";

const WINDOW_MS = 5 * 60 * 1000;
const seenNonces = new Map<string, number>();

export function verifyXquikDelivery(
  rawBody: Buffer,
  headers: { timestamp?: string; nonce?: string; signature?: string },
  secret: string,
  now = Date.now(),
): boolean {
  const { timestamp, nonce, signature } = headers;
  if (!secret || !timestamp || !nonce || !signature) return false;
  const sentAt = Number(timestamp);
  if (!Number.isFinite(sentAt) || Math.abs(now - sentAt) > WINDOW_MS) return false;
  for (const [key, expiresAt] of seenNonces) if (expiresAt < now) seenNonces.delete(key);
  if (seenNonces.has(nonce)) return false;
  const expected = Buffer.from(
    "sha256=" + createHmac("sha256", secret).update(`${timestamp}.${nonce}.`).update(rawBody).digest("hex"),
  );
  const received = Buffer.from(signature);
  if (expected.length !== received.length || !timingSafeEqual(expected, received)) return false;
  seenNonces.set(nonce, Math.max(now, sentAt) + WINDOW_MS);
  return true;
}
```

```python
import hashlib
import hmac
import os
import time

SECRET = os.environ["XQUIK_WEBHOOK_SECRET"].encode()
WINDOW_MS = 5 * 60 * 1000
seen_nonces: dict[str, float] = {}

def verify(raw: bytes, ts: str, nonce: str, sig: str) -> bool:
    now = time.time() * 1000
    if not (ts and nonce and sig and ts.isdigit()) or abs(now - int(ts)) > WINDOW_MS:
        return False
    for key in [k for k, exp in seen_nonces.items() if exp < now]:
        del seen_nonces[key]
    if nonce in seen_nonces:
        return False
    digest = hmac.new(SECRET, f"{ts}.{nonce}.".encode() + raw, hashlib.sha256).hexdigest()
    if not hmac.compare_digest(f"sha256={digest}".encode(), sig.encode()):
        return False
    seen_nonces[nonce] = max(now, int(ts)) + WINDOW_MS
    return True
```

## Receiver rules

- Verify the raw body bytes before parsing JSON. In Express, mount the
  endpoint with `express.raw({ type: "application/json" })`. In a Next.js
  route handler, read `await request.arrayBuffer()`.
- Reject a timestamp more than 5 minutes from now.
- Reject a repeated nonce. Keep each nonce for 5 minutes.
- Compare signatures in constant time.
- Answer `2xx` within 10 seconds. Process the event after the answer.
- Deduplicate by `deliveryId`, because a retry reuses it.
- Answer `2xx` to a `webhook.test` delivery & skip processing.
- Several receivers share 1 nonce store, such as Redis.
- Event text comes from X users. The application stores or shows it as data.
  It never turns an event into an automatic write.
