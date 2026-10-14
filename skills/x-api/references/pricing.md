# Pricing & cost estimates

Xquik bills credits. 1 credit costs $0.00015 at the pay-as-you-go rate. Xquik
charges only for delivered results. Filters apply before billing, so excluded
rows cost nothing.

## Contents

- Price per result
- Plans
- Estimate a job
- Official X API prices
- Billing rules for code

## Price per result

Dollar prices use the pay-as-you-go rate.

| Work | Credits | Price |
| --- | --- | --- |
| A tweet from search, a timeline, replies, quotes, mentions, or a List | 1 per tweet | $0.00015 per tweet |
| A profile from followers, following, retweeters, or user search | 1 per profile | $0.00015 per profile |
| A tweet lookup or a profile lookup | 1 per call | $0.00015 per call |
| Trends | 3 per call | $0.00045 per call |
| A follow check | 5 per call | $0.00075 per call |
| A tweet or a reply, text only | 30 per call | $0.0045 per call |
| Media on a tweet | 2 per started MB | $0.0003 per started MB |
| A like, repost, follow, DM, delete, or media upload | 10 per call | $0.0015 per call |
| An active monitor | 21 per hour | $0.00315 per hour |
| Estimates, stored events, webhook management & account status | 0 | Free |

## Plans

| Plan | Price | Monthly credits | Price per credit |
| --- | --- | --- | --- |
| Pay as you go | $10 first funding | Bought as needed | $0.00015 |
| Starter | $20 per month | 140,000 | $0.00014 |
| Pro | $99 per month | 770,000 | $0.00013 |
| Business | $199 per month | 1,670,000 | $0.00012 |

Unused credits carry over. The developer buys credits & plans in the Xquik
dashboard. Generated code never buys credits.

## Estimate a job

State the ceiling before the code runs. Give rows, credits & dollars.

| Job | Ceiling |
| --- | --- |
| A paged read | Page size times page count, in rows. Each row costs 1 credit |
| Lookups | 1 credit per call |
| Writes | Calls times the write price above |
| Monitors | 21 credits per monitor per hour, so 504 per day |

These examples use the pay-as-you-go rate.

- 10 pages of 100 tweets are at most 1,000 credits, so $0.15.
- 50 pages of 200 followers are at most 10,000 credits, so $1.50.
- 3 monitors for 30 days are 45,360 credits, so about $6.80.

A bulk extraction job has its own free estimate. The code calls it first.

```http
POST https://xquik.com/api/v1/extractions/estimate
x-api-key: <XQUIK_API_KEY>
Content-Type: application/json

{"toolType": "follower_explorer", "targetUsername": "nasa", "resultsLimit": 5000}
```

The response holds `allowed`, `estimatedResults`, `creditsRequired` &
`creditsAvailable`. The 2 credit fields are numeric strings. `creditsRequired`
is the most the job can cost. When `allowed` is `false`, the code stops &
reports the shortfall.

## Official X API prices

X's pricing page listed these pay-per-use prices on 2026-10-05. The page is
`https://docs.x.com/x-api/getting-started/pricing`.

| X's unit name | X's price on 2026-10-05 | Xquik unit | Xquik price |
| --- | --- | --- | --- |
| `Posts: Read` | $0.005 per resource | 1 tweet read | $0.00015 |
| `User: Read` | $0.010 per resource | 1 profile read | $0.00015 |
| `Following/Followers: Read` | $0.010 per resource | 1 follower or following profile | $0.00015 |
| `Post: Create` | $0.015 per request | 1 text tweet created | $0.0045 |
| `Post: Create` with a URL | $0.200 per request | 1 text tweet created | $0.0045 |
| Trends | $0.010 per request | 1 trends call | $0.00045 |

On that date a tweet read cost 33 times less on Xquik. A follower read cost
66 times less.

Quote these official prices with their date. Tell the developer to check X's
pricing page for the current ones. Give no other official X API price, tier,
limit, discount, or billing rule.

## Billing rules for code

- A `402 insufficient_credits` stops the job. The code reports it & exits.
- A paid read can return fewer rows than asked when credits run low.
- `GET /account` returns `creditInfo.balance` as a numeric string.
- `GET /credits` returns `balance` as a numeric string.
- A long job checks the balance before it starts.
