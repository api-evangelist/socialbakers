---
name: Pull social profile metrics
description: Authenticate to the Emplifi (Socialbakers) Public API and retrieve profile metrics for one or more managed social profiles over a date range.
api: openapi/socialbakers-emplifi-public-api-openapi.yml
operations: [listProfiles, getFacebookMetrics, getInstagramMetrics, getAggregatedMetrics]
---

# Pull social profile metrics (Emplifi / Socialbakers Public API v3)

Use this skill to fetch analytics for managed social profiles.

## Authenticate
- Base URL: `https://api.emplifi.io`.
- Use HTTP Basic auth: `Authorization: Basic base64(token:secret)` with your Emplifi API token and secret. (OAuth 2.0 authorization code is also supported for Custom integrations.)
- Send `Content-Type: application/json; charset=utf-8`.

## Steps
1. **Discover profiles** — `GET /3/{network}/profiles` (`listProfiles`) for each network you manage (e.g. `facebook`, `instagram`) to get the profile ids and labels.
2. **Request metrics** — `POST /3/facebook/metrics` (`getFacebookMetrics`) or `POST /3/instagram/metrics` (`getInstagramMetrics`) with a JSON body:
   ```json
   { "profiles": ["<id>"], "metrics": ["fans","fans_change"], "date_start": "2026-06-01", "date_end": "2026-06-30" }
   ```
3. **Aggregate across profiles** — for a single roll-up across many profiles, use `POST /3/aggregated-metrics` (`getAggregatedMetrics`).

## Rules
- Max **100 profiles**, **25 metrics**, and a **12-month** date range per request (else HTTP 400, error `code: 3`).
- Rate limits: **1000 req/hour** per account, **500 req/hour** per user; back off on HTTP 429.
- Responses are a `{ "success": true, "header": [...], "data": ... }` envelope; on failure `{ "success": false, "errors": [{ "code", "errors" }] }`.
- No idempotency key — metrics reads are safe to retry.
