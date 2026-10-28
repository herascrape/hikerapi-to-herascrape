# The migration prompt

Paste the block below into your coding agent — Claude Code, Cursor, or any
agent that can read your repository and fetch URLs — and point it at your
project. It rewrites your HikerAPI integration to herascrape. You do not need
to attach anything; the prompt fetches the docs it needs.

---

```text
You are migrating this codebase from HikerAPI (api.hikerapi.com) to
herascrape (api.herascrape.com).

First, read these — they are the source of truth, and this prompt alone is not
enough to do the migration correctly:

- https://herascrape.com/docs/llms-full.txt
  The complete herascrape documentation as one plain-text file: every endpoint,
  the pagination and error contracts, and the full HikerAPI translation tables.
- https://herascrape.com/docs/guides/hikerapi-migration
  The migration guide on its own, if you want just the tables.
- https://api.herascrape.com/openapi.json
  The OpenAPI schema emitted by the running service. Authoritative for paths
  and parameters; use the "What comes back" examples in the docs for response
  field names.

Then, across the whole repo:

1. Replace the base URL https://api.hikerapi.com with
   https://api.herascrape.com.
2. Rename the auth header `x-access-key` to `x-api-key`. Keep the value in its
   existing environment variable.
3. Rewrite every request path using the "Endpoint translation" tables. Many
   HikerAPI paths collapse onto one of ours — pick the matching row. Never
   invent a path that is not in the tables.
4. If the code uses the HikerAPI SDK (`from hikerapi import Client` /
   `AsyncClient`, calls like `cl.user_by_username_v2(...)`): the SDK method
   names are derived from the REST paths (`user_by_username_v2` ->
   `/v2/user/by/username`; `user_medias_gql` -> `/gql/user/medias`).
   Map each method to the herascrape client method for the same capability via
   the translation tables, and change the constructor's `token=` argument to
   herascrape's `api_key=` (or the HERASCRAPE_API_KEY environment variable).
5. Users are addressed by numeric id (/users/{id}). The only username path is
   GET /users/username/{username}, which returns the profile including `pk`.
   Where the code called a by-username sub-resource (stories, highlights,
   followers), resolve the username once, cache the id, then call the id-based
   endpoint. Flag any place where caching is not straightforward.
6. Posts: replace by-id, by-code and by-url calls with GET /posts/{id}, which
   accepts either the media pk or the shortcode. Extract the shortcode from any
   instagram.com/p/<code>/ URL.
7. Comments, likers, and comment replies take only the post/media id — no owner
   id. GET /posts/{id}/comments and GET /posts/{id}/likers accept a pk or
   shortcode; GET /comments/{id}/replies still needs the parent `media_id`.
   herascrape resolves the owner server-side, so HikerAPI's bare-id comments and
   likers calls map straight across — do not add a user_id parameter.
8. Replace all pagination parameters (page_id, next_page_id, max_id,
   next_max_id, min_id, end_cursor, page_token, reels_max_id,
   profile_grid_items_cursor, repost_next_max_id) with a single `cursor`. Read
   the next token from `next_cursor` and loop while `has_more` is true. Two
   exceptions: GET /posts/{id}/comments pages on `min_id` (read `next_min_id`,
   loop while `has_more`), and GET /users/{id}/posts pages on `cursor` taken
   from `feed.profile_grid_items_cursor` while `feed.more_available` is true.
   Never page /users/{id}/posts on max_id.
9. Delete these parameters: safe_int, flat, force, amount, rank_token,
   expand_suggestion, can_support_threading.
10. Every success is wrapped as {"status": "success", "data": ...}: read `data`
    first (the herascrape SDKs do this for you). Inside `data`, lists come as
    {items, next_cursor, has_more}; where the old code read a bare array, read
    `items`. Object fields are otherwise unchanged — both APIs return
    Instagram's own JSON. Errors come as
    {"status": ..., "error_code": ..., "detail": ...}; branch on `error_code`.
11. Story / highlight by id: GET /stories/{id} returns a single story by its
    media pk (live stories only); GET /highlights/{highlight_id} takes a
    highlight id. By URL: resolve the ready-made URL first with
    GET /oembed?url=<url> (a free call) to the story's media pk or the
    highlight's reel_id, then call the endpoint above. A user's whole active
    story tray is GET /users/{id}/stories.
12. Respect the client rate limit. The herascrape client's limiter defaults to
    50 requests/second per key — keep that default; do not lower it to 10. If
    the code fans out with raw HTTP instead of the client, cap concurrency to
    the same budget.
13. Do NOT invent endpoints. If a call maps to something in the "What we do not
    have" section of the migration guide, leave it in place and mark it with a
    TODO naming the limitation. Do the same for any field you cannot confidently
    map. Never guess a path that is not in the translation tables.

Finally, produce a summary of every call site you changed and the list of
TODOs, then scan your own changes for anything left behind — any
api.hikerapi.com, any x-access-key, any versioned path prefix
(/v1/ /v2/ /v3/ /gql/ /g1/ /g2/ /a2/), or any of the deleted parameters — and
fix or flag each.
```

---

If your agent cannot fetch URLs, download
[`llms-full.txt`](https://herascrape.com/docs/llms-full.txt) and attach it — it
is the whole of these docs in one file.
