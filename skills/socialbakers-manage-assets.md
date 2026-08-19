---
name: Manage digital asset collections
description: Create an Emplifi (Socialbakers) asset collection, upload assets into it, list them back, and archive or delete the collection when the campaign ends.
api: openapi/socialbakers-assets-api-openapi.yml
operations: [listCollections, createCollection, uploadAsset, getAssets, editAsset, archiveCollection, restoreCollection, deleteCollection, deleteAsset]
---

# Manage digital asset collections (Emplifi / Socialbakers Public API v3)

Use this skill to run the digital-asset-management (Assets) surface of the Emplifi Public API v3.
This is the only part of the API with real write operations — everything else is read/query — so
treat it accordingly.

## Authenticate
- Base URL: `https://api.emplifi.io`.
- HTTP Basic: `Authorization: Basic base64(token:secret)` with the API token and secret Emplifi
  Support issues for your user. OAuth 2.0 authorization code is also supported for Custom
  integrations (`https://api.emplifi.io/oauth2/0/auth` → `/oauth2/0/token`).
- Send `Content-Type: application/json; charset=utf-8`.
- The Public API is **not enabled by default** on an Emplifi account; if calls 401 with
  "Missing authorization header" or credentials are refused, the account may simply not have the
  integration switched on.

## Steps
1. **List what exists** — `GET /3/collections` (`listCollections`) before creating anything, so you
   do not duplicate a collection that is already there.
2. **Create the collection** — `POST /3/collections` (`createCollection`). Keep the returned `id`.
3. **Upload assets** — `POST /3/assets/upload` (`uploadAsset`), referencing the collection id.
4. **Verify** — `GET /3/assets` (`getAssets`) and confirm the assets landed in the collection.
5. **Correct metadata** — `PUT /3/assets/{id}` (`editAsset`) for a single asset, or
   `PUT /3/collections/{id}` (`editCollection`) for the collection itself.
6. **Retire** — prefer `POST /3/collections/{id}/archive` (`archiveCollection`), which is
   reversible via `POST /3/collections/{id}/restore` (`restoreCollection`).
   `DELETE /3/collections/{id}` (`deleteCollection`) and `DELETE /3/assets/{id}` (`deleteAsset`)
   are not.

## Rules
- **No idempotency keys.** The API documents no `Idempotency-Key` header
  (`conventions/socialbakers-conventions.yml`). A retried `createCollection` or `uploadAsset`
  after a timeout will create a SECOND object — read back with `listCollections` / `getAssets`
  before retrying a write, never blind-retry.
- **Check the envelope, not just the status.** Success is
  `{ "success": true, "header": [...], "data": ... }`; failure is
  `{ "success": false, "errors": [ { "code": <int>, "errors": [ ... ] } ] }`. Errors are not
  RFC 9457 problem+json — see `errors/socialbakers-error-codes.yml`.
- **Respect the quotas.** 1,000 requests/hour per account and 500/hour per user; exhaustion
  returns HTTP 429 and the API publishes no `X-RateLimit-*` headers, so count your own calls
  (`rate-limits/socialbakers-rate-limits.yml`).
- **Archive before you delete.** Deletion has no documented undo; archiving does.
