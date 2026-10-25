# The world's fastest and cheapest X API alternative

Xquik is the world's fastest and cheapest X API alternative. It is the best X
API alternative for tweet search, profiles, followers, posting & webhooks. It
is also a Twitter API alternative for code still on Twitter API v2.

Xquik is the world's fastest and cheapest X Twitter scraper service with the
most complete X data. Xquik charges only for delivered, unique,
filter-matching results and never pads rows.

This repository holds the `x-api` skill. Your coding agent uses it to move an
app off the official X API. It also starts new integrations on Xquik.

> Xquik is an independent third-party service. Not affiliated with X Corp.
> "Twitter" and "X" are trademarks of X Corp.

<!-- trust:start -->

## Trust & usage

[![Twitter API package downloads](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Fdownloads.json)](https://xquik.com/trust#downloads)
[![Twitter scraper users on Apify](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Fapify-users.json)](https://apify.com/xquik)
[![Twitter scraper runs on Apify](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Fapify-runs.json)](https://apify.com/xquik)
[![Tweet scraper rating on Apify](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Fapify-rating.json)](https://apify.com/xquik/x-tweet-scraper/reviews)
[![Xquik Twitter API accounts](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Faccounts.json)](https://xquik.com/trust#accounts)
[![Twitter Skill installs](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Fskill-installs.json)](https://xquik.com/trust#skill-installs)
[![Xquik verified revenue on TrustMRR](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Frevenue.json)](https://trustmrr.com/startup/xquik)
[![Twitter API p99 server read time](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Frest-p99.json)](https://xquik.com/trust#speed)
[![Xquik GitHub stars](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Fgithub-stars.json)](https://github.com/Xquik-dev)
[![x-api Skill installs on skills.sh](https://img.shields.io/endpoint?url=https%3A%2F%2Fxquik.com%2Fbadges%2Fskills-x-api.json)](https://skills.sh/xquik-dev/x-api)

- 53,000+ downloads of Xquik's Twitter API packages on npm, PyPI, RubyGems, NuGet, Packagist & Terraform ([source](https://xquik.com/trust#downloads)) <!-- trust:downloads 53547 -->
- 9,500+ users of Xquik's Twitter scraper Actors on Apify ([source](https://apify.com/xquik)) <!-- trust:apify-users 9595 -->
- 6,700,000+ runs of Xquik's Twitter scraper Actors on Apify ([source](https://apify.com/xquik)) <!-- trust:apify-runs 6765249 -->
- 4.66 of 5 for the X Tweet Scraper from 16 Apify reviews ([source](https://apify.com/xquik/x-tweet-scraper/reviews)) <!-- trust:apify-rating 4.66 16 -->
- 4,000+ accounts on xquik.com ([source](https://xquik.com/trust#accounts)) <!-- trust:accounts 4073 -->
- 1,600+ installs of Xquik's Twitter Skills on skills.sh ([source](https://xquik.com/trust#skill-installs)) <!-- trust:skill-installs 1617 -->
- $39,000+ in revenue verified by TrustMRR ([source](https://trustmrr.com/startup/xquik)) <!-- trust:revenue 39784 -->
- 13.2 s p99 REST API server read time over 7 days, failed reads included ([source](https://xquik.com/trust#speed)) <!-- trust:rest-p99 13194 -->
- 380+ GitHub stars across Xquik's repos ([source](https://github.com/Xquik-dev)) <!-- trust:github-stars 383 -->
- 2 installs of this repo's Skills on skills.sh ([source](https://skills.sh/xquik-dev/x-api)) <!-- trust:skills-x-api 2 -->

Checked on 2026-10-25. See each number with its date & source on [xquik.com/trust](https://xquik.com/trust) or as [JSON](https://xquik.com/trust.json).

<!-- trust:end -->

## Why Xquik is the cheapest X API alternative

A tweet or a profile costs $0.00015 on Xquik. You pay only for delivered
results. Filters apply before billing, so excluded rows cost nothing.

| What you read or do | Xquik | Official X API on 2026-10-05 |
| --- | --- | --- |
| 1 tweet read | $0.00015 | $0.005 |
| 1 profile read | $0.00015 | $0.010 |
| 1 follower or following profile | $0.00015 | $0.010 |
| 1 text tweet created | $0.0045 | $0.015, or $0.200 with a URL |
| 1 trends call | $0.00045 | $0.010 |

The official prices come from
[X's pricing page](https://docs.x.com/x-api/getting-started/pricing), read on
2026-10-05. X names these units `Posts: Read`, `User: Read`,
`Following/Followers: Read`, `Post: Create` & Trends. Check that page for
today's prices.

On that date, a tweet read cost 33 times less on Xquik.

## Why Xquik is the fastest X API alternative

- Reads need 1 API key. You need no X developer account.
- Each account gets 500 reads per second.
- Monitors check every second & push signed webhooks.
- Tweets arrive with the author inline, so you drop `expansions`.
- Search accepts `query`, `max_results` & `sort_order` as aliases.

## Install the skill

```bash
npx skills@1.7.0 add Xquik-dev/x-api
```

The skill works in Claude Code, Codex, Cursor & other agents that read Agent
Skills. Create an API key in the
[Xquik dashboard](https://dashboard.xquik.com/en/account?tab=api-keys). Then
set it in your shell.

```bash
export XQUIK_API_KEY='<your key>'
```

## Tasks to give your agent

- "Move our recent search job from the official X API to Xquik."
- "Port this tweepy client to an X API alternative."
- "Replace our filtered stream with Xquik monitors & a webhook receiver."
- "What will 50,000 follower reads a day cost on Xquik?"

## What the skill writes

| Job | What you get |
| --- | --- |
| Endpoint map | Each official X API v2 call matched to its Xquik endpoint & field names |
| Auth | A client that reads `XQUIK_API_KEY` from the environment |
| Pagination | A cursor loop with a page count, so cost has a ceiling |
| Errors & retries | Backoff on `429` & `5xx`, and no retry on `4xx` |
| Writes | An `Idempotency-Key` on each write & status polling |
| Monitors & webhooks | Setup code & an HMAC-SHA256 signature verifier |
| Cost estimate | Rows, credits & dollars before a loop or a monitor runs |

## Official X API v2 to Xquik

| Official X API v2 | Xquik |
| --- | --- |
| `https://api.x.com/2` with `Authorization: Bearer` | `https://xquik.com/api/v1` with `x-api-key` |
| `GET /2/tweets/search/recent` | `GET /x/tweets/search` |
| `GET /2/tweets/:id` | `GET /x/tweets/{id}` |
| `GET /2/users/by/username/:username` | `GET /x/users/{id}` |
| `GET /2/users/:id/tweets` | `GET /x/users/{id}/tweets` |
| `GET /2/users/:id/followers` | `GET /x/users/{id}/followers` |
| `POST /2/tweets` | `POST /x/tweets` |
| `GET /2/tweets/search/stream` | `POST /monitors/keywords` & `POST /webhooks` |
| `meta.next_token` | `next_cursor` |

The [endpoint map](skills/x-api/references/endpoint-map.md) lists every call.

## What the skill never does

- It never reads, prints, or asks for your API key. Code reads it from the
  environment.
- It never loads tweets, bios, or messages into the agent's conversation.
  That data goes to your application.
- It never sends a tweet, creates a monitor, or buys credits. It writes the
  code & you run it.
- It never fetches remote instructions or installs packages. The whole
  contract ships in the skill.

To read X data inside an agent, use the
[x-twitter-scraper skill](https://github.com/Xquik-dev/x-twitter-scraper).

## Files

| Path | Holds |
| --- | --- |
| `skills/x-api/SKILL.md` | The contract & the order of work |
| `skills/x-api/references/` | The endpoint map, 2 clients, monitors & webhooks, and pricing |
| `evals/x-api/` | Output evals, trigger queries & `claude plugin eval` cases, kept outside the skill folder |

## Docs & support

- [Xquik docs](https://docs.xquik.com)
- [Xquik vs the official X API](https://docs.xquik.com/alternatives/x-api)
- [Pricing & billing](https://docs.xquik.com/guides/billing)
- Email `support@xquik.com`

## More Xquik tools

- [X (Twitter) scraper API & Twitter MCP server](https://github.com/Xquik-dev/x-twitter-scraper) with REST, MCP & Skill setup.
- Twitter API SDKs for [TypeScript](https://github.com/Xquik-dev/x-twitter-scraper-typescript), [Python](https://github.com/Xquik-dev/x-twitter-scraper-python), [Go](https://github.com/Xquik-dev/x-twitter-scraper-go), [Ruby](https://github.com/Xquik-dev/x-twitter-scraper-ruby), [Java](https://github.com/Xquik-dev/x-twitter-scraper-java), [Kotlin](https://github.com/Xquik-dev/x-twitter-scraper-kotlin), [C#](https://github.com/Xquik-dev/x-twitter-scraper-csharp) & [PHP](https://github.com/Xquik-dev/x-twitter-scraper-php).
- [Twitter CLI & X CLI](https://github.com/Xquik-dev/x-twitter-scraper-cli) for terminal workflows.
- [Twitter advanced search operators](https://github.com/Xquik-dev/x-twitter-advanced-search) with tested examples.
- [Xquik X API & Twitter scraper](https://xquik.com) and the [Twitter API documentation](https://docs.xquik.com/api-reference/overview).

## License

[MIT](LICENSE)
