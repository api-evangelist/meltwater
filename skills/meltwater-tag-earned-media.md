---
name: Tag and search earned media documents
description: Search Meltwater earned media documents for a saved search and apply or remove tags on the matching documents.
api: openapi/meltwater-listening-search-management-api-openapi.yml
operations:
  - list_searches
  - create-earned-search
  - list_tags
  - add_tags
  - remove_tags
  - list_filter_sets
  - list_custom_categories
generated: '2026-08-13'
method: generated
source: openapi/_original/meltwater-api-openapi.yaml
---

# Tag and search earned media documents

Search Meltwater earned media documents for a saved search and apply or remove tags on the matching documents.

## Before you call anything

- Base URL is `https://api.meltwater.com`. HTTPS only — plain HTTP fails.
- Authenticate with the customer's Meltwater API token in an `apikey` request header. Never put it in a query string, a client bundle, or source control.
- Accounts can span several companies and workspaces. Most operations take a `company_id` (and sometimes `workspace_id`) query parameter — resolve it first with `list_companies` / `list_workspaces` rather than assuming a default.
- Every response carries an `X-Request-Id`. Log it; Meltwater asks for it on support tickets.
- There is no idempotency key. A retried POST creates a second resource. Read back with the list operation before retrying a create.

## Steps

1. **Resolve the search.** `GET /v3/searches` (`list_searches`).
2. **Search the documents.** `POST /v3/search/{id}` (`create-earned-search`) returns individual mentions matching the saved search plus any filters you pass. Filters can be composed from `GET /v3/filter_sets` (`list_filter_sets`) and `GET /v3/custom_categories` (`list_custom_categories`).
3. **List the tag vocabulary.** `GET /v3/tags` (`list_tags`) before tagging — tags are account-level objects, not free text you invent per call.
4. **Apply tags.** `POST /v3/documents/add_tags` (`add_tags`) with the document identifiers and the tag ids.
5. **Remove tags.** `POST /v3/documents/remove_tags` (`remove_tags`).

## Cautions

- Search calls are charged against the **daily analytics** budget, not a separate search budget.
- `add_tags` / `remove_tags` are not idempotent-keyed; they are set operations, so re-sending the same body is safe in effect, but a partial failure is reported per field in the `{message, errors}` envelope — read `errors` rather than only the status code.
- Twitter/X and YouTube fields are deliberately reduced for terms-of-service compliance (ids instead of URLs and handles; no YouTube reach or AVE). Do not build a workflow that expects them.

## Errors and limits

- `400` bad request, `401` bad/missing token, `403` not authorised **or daily credit exhausted**, `404` unknown id, `409` conflict, `422` validation failure, `429` rate limited, `500`/`503` Meltwater-side.
- Errors are vendor JSON, not RFC 9457. Expect either `{message, errors}` (with `errors` keyed by field) or the discriminated `{type, errors[]}` shape. See `errors/meltwater-problem-types.yml`.
- Analytics responses carry `RateLimit-Day-Limit`, `RateLimit-Day-Remaining` and `RateLimit-Day-Reset`. Read them and stop before you hit zero — the day window starts at the first request after a reset, not at midnight.
- A `429` with `Service overloaded, please try again later` on Earned Analytics is a busy signal, not a quota breach: wait for the timestamp in `Retry-After` and retry.
- Export endpoints are capped at 20 calls/minute. See `rate-limits/meltwater-rate-limits.yml`.

## Related artifacts

- `authentication/meltwater-authentication.yml` — token model and OAuth on MCP
- `conventions/meltwater-conventions.yml` — pagination, tenancy, templates, tracing
- `rate-limits/meltwater-rate-limits.yml` — the numbers and the headers
- `errors/meltwater-problem-types.yml` — every declared 4xx/5xx
