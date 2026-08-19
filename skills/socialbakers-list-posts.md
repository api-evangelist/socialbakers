---
name: Retrieve published posts with pagination
description: Fetch published content (posts, videos, tweets) for managed Emplifi (Socialbakers) profiles, walking the cursor pagination.
api: openapi/_original/socialbakers-emplifi-public-api-openapi.yml
operations: [listProfiles, getFacebookPosts, getInstagramPosts, getTwitterTweets]
---

# Retrieve published posts (Emplifi / Socialbakers Public API v3)

Use this skill to export published content for a profile over a date range.

## Authenticate
- Base URL `https://api.emplifi.io`; HTTP Basic `Authorization: Basic base64(token:secret)`.

## Steps
1. **Get profile ids** — `GET /3/{network}/profiles` (`listProfiles`).
2. **Request posts** — `POST /3/facebook/page/posts` (`getFacebookPosts`), `POST /3/instagram/profile/posts` (`getInstagramPosts`), or `POST /3/twitter/profile/tweets` (`getTwitterTweets`) with:
   ```json
   { "profiles": ["<id>"], "date_start": "2026-06-01", "date_end": "2026-06-30" }
   ```
3. **Paginate** — the response includes `next` (a cursor) and `remaining` (items left). While `next` is non-null, re-issue the request with `"cursor": "<next>"` until `remaining` reaches 0. Pages return up to **100** items.

## Rules
- Rate limits: 1000 req/hour account, 500 req/hour user — back off on HTTP 429.
- Envelope: `{ "success": true, "data": [...], "next": ..., "remaining": ... }`.
- Post label ids in results map to names via `GET /3/post/labels` (`getPostLabels`).
