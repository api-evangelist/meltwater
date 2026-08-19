---
name: Export earned media for a saved search
description: Run a one-time or recurring export of Meltwater earned media (news, blogs, social) for an existing saved search, and collect the result.
api: openapi/meltwater-listening-exports-api-openapi.yml
operations:
  - list_searches
  - search_count
  - create_onetime_export
  - list_onetime_exports
  - show_onetime_export
  - delete_onetime_export
  - create_recurring_export
  - list_recurring_exports
  - show_recurring_export
  - delete_recurring_export
generated: '2026-08-13'
method: generated
source: openapi/_original/meltwater-api-openapi.yaml
---

# Export earned media for a saved search

Run a one-time or recurring export of Meltwater earned media (news, blogs, social) for an existing saved search, and collect the result.

## Before you call anything

- Base URL is `https://api.meltwater.com`. HTTPS only — plain HTTP fails.
- Authenticate with the customer's Meltwater API token in an `apikey` request header. Never put it in a query string, a client bundle, or source control.
- Accounts can span several companies and workspaces. Most operations take a `company_id` (and sometimes `workspace_id`) query parameter — resolve it first with `list_companies` / `list_workspaces` rather than assuming a default.
- Every response carries an `X-Request-Id`. Log it; Meltwater asks for it on support tickets.
- There is no idempotency key. A retried POST creates a second resource. Read back with the list operation before retrying a create.

## Steps

1. **Find the search.** `GET /v3/searches` (`list_searches`) returns the account's saved searches with `id` and `name`. Exports are always bound to a saved search — you cannot export an ad-hoc query.
2. **Size it before you spend.** `GET /v3/searches/{id}/count` (`search_count`) gives an approximate result count for a period. A single export is capped at 2,000,000 documents and is randomly sampled down if it would exceed that, so check first.
3. **Choose an output template.** Pass `template: {"name": "api.json"}` for the general-purpose shape. Omitting `template` silently selects the legacy format — do not omit it.
4. **Create the export.** `POST /v3/exports/one-time` (`create_onetime_export`) for a fixed window, or `POST /v3/exports/recurring` (`create_recurring_export`) for a schedule. One export may cover up to 5 saved searches or tags and still counts as one export against the monthly allowance.
5. **Poll for completion.** `GET /v3/exports/one-time/{id}` (`show_onetime_export`) or list with `list_onetime_exports` / `list_recurring_exports`.
6. **Clean up.** `DELETE /v3/exports/one-time/{id}` (`delete_onetime_export`) or `delete_recurring_export`. Recurring exports keep consuming the document allowance until deleted.

## Budget rules that will bite you

- Documents are counted **per export, not deduplicated** — exporting the same document twice costs two documents.
- Inclusive API access is 30,000 documents and 50 exports per month; an API package sets its own contract numbers. Both reset on the 1st.
- Analytics-style time windows are capped at 12 months unless the contract says otherwise.

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
