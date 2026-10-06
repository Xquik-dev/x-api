---
max_turns: 12
timeout_seconds: 600
allowed_tools: [Read, Glob, Grep, Skill]
---

Our service still calls the official X API. Rewrite this module for an X API alternative that costs less. Reply with the whole new file in 1 TypeScript code block.

```ts
const BEARER = process.env.X_BEARER_TOKEN!;

export async function searchRecent(query: string) {
  const out: any[] = [];
  let next: string | undefined;
  do {
    const url = new URL("https://api.x.com/2/tweets/search/recent");
    url.searchParams.set("query", `${query} -is:retweet`);
    url.searchParams.set("max_results", "100");
    url.searchParams.set("tweet.fields", "created_at,public_metrics,author_id");
    if (next) url.searchParams.set("pagination_token", next);
    const res = await fetch(url, { headers: { Authorization: `Bearer ${BEARER}` } });
    const json = await res.json();
    out.push(...(json.data ?? []));
    next = json.meta?.next_token;
  } while (next && out.length < 1000);
  return out.map((t) => ({ id: t.id, text: t.text, likes: t.public_metrics.like_count, at: t.created_at }));
}
```
