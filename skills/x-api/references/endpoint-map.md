# Endpoint map: official X API v2 to Xquik

Every Xquik endpoint starts at `https://xquik.com/api/v1`.

## Contents

- Base URL & auth
- Tweets
- Users
- Lists, Spaces & trends
- Reads that need a connected X account
- Writes
- Streams
- Parameter names
- Field names
- Calls with no Xquik endpoint

## Base URL & auth

| Official X API v2 | Xquik |
| --- | --- |
| `https://api.x.com/2` or `https://api.twitter.com/2` | `https://xquik.com/api/v1` |
| App-only `Authorization: Bearer <token>` | `x-api-key: <key>` |
| OAuth 2.0 with PKCE, or OAuth 1.0a user context | `x-api-key`, plus `account` naming the connected X account |
| Developer app, client ID & client secret | None. The developer connects X accounts in the Xquik dashboard |

## Tweets

A tweets page holds `tweets`, `has_next_page` & `next_cursor`.

| Official X API v2 | Xquik | Inputs | Output |
| --- | --- | --- | --- |
| `GET /2/tweets/search/recent`, `GET /2/tweets/search/all` | `GET /x/tweets/search` | `q` (required), `queryType` (`Latest` default, or `Top`), `limit` (1 to 10,000, default 20), `cursor`, filters below | Tweets page |
| `GET /2/tweets/:id` | `GET /x/tweets/{id}` | Tweet ID or tweet URL | `tweet`, `author` |
| `GET /2/tweets?ids=` | `GET /x/tweets?ids=` | Up to 100 comma-separated tweet IDs | `tweets` |
| `GET /2/tweets/:id/quote_tweets` | `GET /x/tweets/{id}/quotes` | `pageSize` (1 to 300), `cursor` | Tweets page |
| Search with `conversation_id:` | `GET /x/tweets/{id}/replies` | `pageSize` (1 to 300), `sort` (`relevance`, `latest`, `oldest`, `likes`), `cursor` | Tweets page |
| Search with `conversation_id:` | `GET /x/tweets/{id}/thread` | `pageSize` (1 to 100), `cursor` | Tweets page |
| `GET /2/tweets/:id/retweeted_by` | `GET /x/tweets/{id}/retweeters` | `pageSize` (1 to 200), `cursor` | Users page |

Search filters are named query parameters. Send only the ones the application
needs.

- Authors: `fromUser`, `toUser`, `mentioning`, `verifiedOnly`
- Text: `exactPhrase`, `anyWords`, `excludeWords`, `hashtags`, `cashtags`, `url`
- Time: `sinceTime` (inclusive) & `untilTime` (exclusive), both ISO 8601
- Language: `language`, such as `en`
- Media: `mediaType` (`images`, `videos`, `gifs`, `media`, `links`, `none`)
- Engagement: `minLikes`, `minRetweets`, `minReplies`, `minQuotes`, `minViews`
- Tweet types: `replies`, `retweets`, `quotes`, each `include`, `exclude`, or `only`

Words, quoted phrases, `-word` & `from:` stay in `q`. Replace these official
X API v2 operators with named filters:

| Official X API v2 operator | Xquik filter |
| --- | --- |
| `-is:retweet`, `is:retweet` | `retweets=exclude`, `retweets=only` |
| `-is:reply`, `is:reply` | `replies=exclude`, `replies=only` |
| `-is:quote`, `is:quote` | `quotes=exclude`, `quotes=only` |
| `has:media`, `has:images`, `has:videos`, `has:links` | `mediaType=media`, `images`, `videos`, `links` |
| `lang:en` | `language=en` |
| `is:verified` | `verifiedOnly=true` |

## Users

A users page holds `users`, `has_next_page` & `next_cursor`.

| Official X API v2 | Xquik | Inputs | Output |
| --- | --- | --- | --- |
| `GET /2/users/:id`, `GET /2/users/by/username/:username` | `GET /x/users/{id}` | User ID or username | Profile object |
| `GET /2/users?ids=`, `GET /2/users/by?usernames=` | `GET /x/users/batch` | `ids` or `usernames`, up to 100, not both | `users` |
| `GET /2/users/search` | `GET /x/users/search` | `q` (required), `pageSize` (1 to 100), `cursor` | Users page |
| `GET /2/users/:id/tweets` | `GET /x/users/{id}/tweets` | `pageSize` (1 to 300, default 20), `sinceTime`, `untilTime`, `includeReplies` (default `false`), `cursor` | Tweets page |
| `GET /2/users/:id/mentions` | `GET /x/users/{id}/mentions` | `pageSize` (1 to 100), `sinceTime`, `untilTime`, `cursor` | Tweets page |
| `GET /2/users/:id/followers` | `GET /x/users/{id}/followers` | `pageSize` (1 to 300, default 200), `cursor`, profile filters | Users page |
| `GET /2/users/:id/following` | `GET /x/users/{id}/following` | Same as followers | Users page |

Profile filters apply before billing: `minFollowers`, `maxFollowers`,
`minFollowing`, `maxFollowing`, `minStatuses`, `minAccountAgeDays`,
`verifiedOnly`, `bioContains`, `locationContains`.

## Lists, Spaces & trends

| Official X API v2 | Xquik | Inputs | Output |
| --- | --- | --- | --- |
| `GET /2/lists/:id/tweets` | `GET /x/lists/{id}/tweets` | `pageSize` (1 to 300), `cursor` | Tweets page |
| `GET /2/lists/:id/members` | `GET /x/lists/{id}/members` | `pageSize` (1 to 300), `cursor` | Users page |
| `GET /2/lists/:id/followers` | `GET /x/lists/{id}/followers` | `pageSize` (1 to 300), `cursor` | Users page |
| `GET /2/spaces/:id` | `GET /x/spaces/{id}` | Space ID | `space` |
| `GET /2/spaces/search` | `GET /x/spaces/search` | `q` (required), `pageSize` (1 to 100), `cursor` | `spaces`, `has_next_page`, `next_cursor` |
| `GET /2/trends/by/woeid/:woeid` | `GET /x/trends` | `woeid` (default 1, worldwide), `count` (1 to 50, default 30) | `trends`, `count`, `woeid` |

## Reads that need a connected X account

| Official X API v2 | Xquik | Inputs | Output |
| --- | --- | --- | --- |
| `GET /2/users/:id/timelines/reverse_chronological` | `GET /x/timeline` | `cursor` | Tweets page |
| `GET /2/users/:id/bookmarks` | `GET /x/bookmarks` | `folderId`, `cursor` | Tweets page |
| `GET /2/users/:id/liked_tweets` | `GET /x/users/{id}/likes` | `pageSize` (1 to 100), `cursor`. The account's own likes only | Tweets page |
| `GET /2/tweets/:id/liking_users` | `GET /x/tweets/{id}/favoriters` | `pageSize` (1 to 200), `cursor` | Users page |
| `GET /2/dm_conversations/with/:participant_id/dm_events` | `GET /x/dm/{userId}/history` | `account` (required), `cursor` | `messages`, `has_next_page`, `next_cursor` |
| `GET /2/users/me` | `GET /x/accounts` | None | `accounts`, the connected X accounts |

The timeline, bookmarks, likes & likers endpoints return
`424 account_required` without a connected X account.

## Writes

Every write names the connected X account in `account` & sends an
`Idempotency-Key` header. Every write returns a write action with `status`,
`terminal`, `safeToRetry`, `statusUrl` & `pollAfterMs`. A new tweet's ID is in
`tweetId`.

| Official X API v2 | Xquik | Body |
| --- | --- | --- |
| `POST /2/tweets` | `POST /x/tweets` | `account`, `text`, optional `reply_to_tweet_id`, `community_id`, `media` (up to 4 image URLs or 1 MP4 URL) |
| `DELETE /2/tweets/:id` | `DELETE /x/tweets/{id}` | `account` |
| `POST /2/users/:id/likes` | `POST /x/tweets/{id}/like` | `account` |
| `DELETE /2/users/:id/likes/:tweet_id` | `DELETE /x/tweets/{id}/like` | `account` |
| `POST /2/users/:id/retweets` | `POST /x/tweets/{id}/retweet` | `account` |
| `DELETE /2/users/:id/retweets/:source_tweet_id` | `DELETE /x/tweets/{id}/retweet` | `account` |
| `POST /2/users/:id/following` | `POST /x/users/{id}/follow` | `account` |
| `DELETE /2/users/:source_user_id/following/:target_user_id` | `DELETE /x/users/{id}/follow` | `account` |
| `POST /2/dm_conversations/with/:participant_id/messages` | `POST /x/dm/{userId}` | `account`, `text` |
| `POST /2/media/upload` | `POST /x/media` | JSON `account`, `url`, or multipart `account`, `file` |

The official API puts the acting user's ID in the path. Xquik puts the target
in the path & the acting account in `account`.

## Streams

| Official X API v2 | Xquik |
| --- | --- |
| `POST /2/tweets/search/stream/rules` | `POST /monitors/keywords` with `query` & `eventTypes` |
| `GET /2/tweets/search/stream`, a held connection | `POST /webhooks` with `url` & `eventTypes`, or poll `GET /events` |
| Polling `GET /2/users/:id/tweets` for new tweets | `POST /monitors` with `username` & `eventTypes` |

See [monitors and webhooks](monitors-webhooks.md) for events & signatures.

## Parameter names

| Official X API v2 | Xquik |
| --- | --- |
| `query` | `q`, with the same search operators |
| `max_results` | `limit` on search, `pageSize` on other lists |
| `pagination_token` or `next_token` | `cursor` |
| `start_time`, `end_time` | `sinceTime`, `untilTime` |
| `sort_order=recency` or `relevancy` | `queryType=Latest` or `Top` |
| `tweet.fields`, `user.fields`, `expansions` | Remove them. Xquik returns the full tweet with its author inline |

Search also accepts `query`, `max_results` & `sort_order` as aliases. New code
uses the Xquik names.

## Field names

| Official X API v2 tweet | Xquik tweet |
| --- | --- |
| `data[]` | `tweets[]` |
| `id`, `text`, `lang` | `id`, `text`, `lang` |
| `created_at` | `createdAt` |
| `author_id` & `includes.users[]` | `author.id`, `author.username`, `author.name` |
| `public_metrics.like_count` | `likeCount` |
| `public_metrics.retweet_count` | `retweetCount` |
| `public_metrics.reply_count` | `replyCount` |
| `public_metrics.quote_count` | `quoteCount` |
| `public_metrics.impression_count` | `viewCount` |
| `public_metrics.bookmark_count` | `bookmarkCount` |
| `attachments.media_keys` & `includes.media[]` | `media[]` |
| `meta.next_token` | `next_cursor`, with `has_next_page` |

| Official X API v2 user | Xquik user |
| --- | --- |
| `id`, `username`, `name`, `description`, `location` | Same names |
| `public_metrics.followers_count` | `followers` |
| `public_metrics.following_count` | `following` |
| `public_metrics.tweet_count` | `statusesCount` |
| `verified` | `verified`, `isBlueVerified`, `verifiedType` |
| `created_at` | `createdAt` |
| `profile_image_url` | `profilePicture` |
| `protected` | `protected` |

Xquik omits an optional field when X does not return it. Read optional fields
with a default.

## Calls with no Xquik endpoint

These official calls have no Xquik endpoint:

- Tweet counts, such as `GET /2/tweets/counts/recent`
- Mutes & blocks
- List creation & editing
- Hiding replies
- Adding or removing bookmarks
- Compliance jobs

Tell the developer which calls stay unmapped. They keep those calls on the
official X API or drop them.
