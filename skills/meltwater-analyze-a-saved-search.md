---
name: Analyze a saved search
description: Pull volume, sentiment, top entities, sources, topics and shared content for a Meltwater saved search over a time window.
api: openapi/meltwater-listening-analytics-api-openapi.yml
operations:
  - list_searches
  - getV3Analytics-searchId-start-end-tz-source-country-language-company_id
  - getV3AnalyticsTop_tags-searchId-start-end-tz-source-country-language-size-company_id
  - getV3AnalyticsTop_entities-searchId-start-end-tz-source-country-language-size-sentiment-company_id
  - getV3AnalyticsTop_sources-searchId-start-end-tz-source-country-language-size-min_authority-company_id
  - getV3AnalyticsTop_keyphrases-searchId-start-end-tz-source-country-language-size-sentiment-company_id
  - getV3AnalyticsTop_locations-searchId-start-end-tz-source-country-language-size-level-company_id
  - getV3AnalyticsTop_mentions-searchId-start-end-tz-source
  - getV3AnalyticsTop_topics-searchId-start-end-tz-source
  - getV3AnalyticsTop_shared-searchId-start-end-tz-source-country-language-size-sort_by-company_id
  - getV3AnalyticsTop_sharedLinks-searchId-start-end-tz-source-country-language-size-sentiment-company_id
generated: '2026-08-13'
method: generated
source: openapi/_original/meltwater-api-openapi.yaml
---

# Analyze a saved search

Pull volume, sentiment, top entities, sources, topics and shared content for a Meltwater saved search over a time window.

## Before you call anything

- Base URL is `https://api.meltwater.com`. HTTPS only — plain HTTP fails.
- Authenticate with the customer's Meltwater API token in an `apikey` request header. Never put it in a query string, a client bundle, or source control.
- Accounts can span several companies and workspaces. Most operations take a `company_id` (and sometimes `workspace_id`) query parameter — resolve it first with `list_companies` / `list_workspaces` rather than assuming a default.
- Every response carries an `X-Request-Id`. Log it; Meltwater asks for it on support tickets.
- There is no idempotency key. A retried POST creates a second resource. Read back with the list operation before retrying a create.

## Steps

1. **Resolve the search.** `GET /v3/searches` (`list_searches`).
2. **Get the summary first.** `GET /v3/analytics/{searchId}` returns volume, sentiment and country breakdowns for the window. Use it to decide whether the deeper cuts are worth spending calls on.
3. **Then take the cut you need**, one call each:
   - `GET /v3/analytics/{searchId}/top_entities` — named entities, filterable by `sentiment`
   - `GET /v3/analytics/{searchId}/top_sources` — sources, with `min_authority`
   - `GET /v3/analytics/{searchId}/top_keyphrases`, `/top_topics`, `/top_tags`
   - `GET /v3/analytics/{searchId}/top_locations` — takes a `level`
   - `GET /v3/analytics/{searchId}/top_mentions`, `/top_shared`, `/top_shared_links`
4. **Window every call.** `start`, `end` and `tz` are how you bound the query; `size` caps the number of buckets returned. There is no pagination on these endpoints — `size` is the whole control.

## Cost model

Each analytics call spends a daily API credit. Inclusive access is 50 calls/day; an API package sets its own number. `GET /v3/usage/me/requests` shows what you have spent. Earned media **search** calls are counted against the same daily analytics budget.

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
