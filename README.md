# HikerAPI → herascrape

Move your HikerAPI integration to [herascrape](https://herascrape.com/?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration), the
HikerAPI alternative, in one paste. Same Instagram data, returned as
**Instagram's own JSON**, so your parsing code does not change, behind a
smaller, cheaper, faster API.

- **37 endpoints, not 154.** HikerAPI exposes the same capability under `/v1`,
  `/v2`, `/v3`, `/gql`, `/g1`, `/g2` and `/a2`. herascrape collapses each to one
  RESTful path and picks the source for you.
- **Flat $0.0004 per call.** One price for every endpoint, billed only on
  success: no credit currency, and no per-endpoint premium (comments and likers
  included).
- **Faster.** In our head-to-head measurements, p50 / p90 of
  **1024 / 1644 ms** against HikerAPI's **1356 / 2358 ms**.
- **Instagram's JSON, verbatim.** Object fields land where HikerAPI's `v1` / `v2`
  payloads already put them; list responses gain an
  `{items, next_cursor, has_more}` wrapper.
- **Fresh, not cached.** Every call fetches live from Instagram: real-time
  data, not a stale snapshot.
- **Free trial balance**, no card: run the whole migration before you pay
  anything.

See the full [herascrape vs HikerAPI comparison](https://herascrape.com/compare/herascrape-vs-hikerapi?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration).

## Migrate in one paste

Paste [`PROMPT.md`](PROMPT.md) into your coding agent (Claude Code, Cursor, or
any agent that reads your repository and fetches URLs) and point it at your
project. It reads the herascrape docs itself and rewrites every HikerAPI call
(base URL, auth header, endpoint paths, pagination, and the HikerAPI SDK),
flagging anything it cannot map instead of guessing.

## What changes

Three mechanical edits, then the paths.

| | HikerAPI | herascrape |
| --- | --- | --- |
| Base URL | `api.hikerapi.com` | `api.herascrape.com` |
| Auth header | `x-access-key` | `x-api-key` |
| Paging | a different param per endpoint | one opaque `cursor` |

```diff
- GET https://api.hikerapi.com/v2/user/followers?user_id=25025320&page_id=ABC
- x-access-key: <key>
+ GET https://api.herascrape.com/users/25025320/followers?cursor=ABC
+ x-api-key: <key>
```

```diff
- GET https://api.hikerapi.com/v2/media/info/by/code?code=B4XpqViUFUI&safe_int=true
+ GET https://api.herascrape.com/posts/B4XpqViUFUI
```

```diff
- GET https://api.hikerapi.com/v2/user/by/username?username=instagram
- GET https://api.hikerapi.com/v2/user/stories/by/username?username=instagram
+ GET https://api.herascrape.com/users/username/instagram   # returns pk, cache it
+ GET https://api.herascrape.com/users/25025320/stories
```

Using the HikerAPI Python SDK? Each call maps cleanly onto one herascrape
method (several HikerAPI methods often collapse onto the same one):

```diff
- cl.user_by_username_v2("instagram")
+ client.user_by_username(username="instagram")
```

The full translation tables live in the
[migration guide](https://herascrape.com/docs/guides/hikerapi-migration?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration); the
prompt reads them so you do not have to.

## What we do not have

An honest list, so you find out here rather than in production:

- **Not implemented:** related-profile suggestions, guides, media share
  template, owner-only insights, offensive-text check, reels search, and the
  audio canonical-id / stream lookups. If one of these blocks you,
  [tell us](https://herascrape.com/?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration); several are small.
- **Standalone geo search** was removed by Instagram; the venue picker behind
  `/search/places` covers the common case.

The guide sorts every HikerAPI endpoint into translates-directly,
translates-elsewhere, no-equivalent, or removed-by-Instagram.

## Links

- [Migration guide](https://herascrape.com/docs/guides/hikerapi-migration?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration)
- [herascrape vs HikerAPI](https://herascrape.com/compare/herascrape-vs-hikerapi?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration)
  · [HikerAPI alternative](https://herascrape.com/alternatives/hikerapi-alternative?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration)
- [herascrape client: Python & TypeScript](https://github.com/herascrape/herascrape)
- [Docs](https://herascrape.com/docs?utm_source=github&utm_medium=readme&utm_campaign=hikerapi-migration)

---

herascrape is not affiliated with, authorized, maintained, or endorsed by
Instagram or Meta. Instagram is a trademark of Meta Platforms, Inc.
