---
name: Stream mentions to a webhook
description: Create a Meltwater data stream (hook) that POSTs batches of matching documents to your own HTTPS endpoint as they arrive.
api: openapi/meltwater-listening-streaming-api-openapi.yml
operations:
  - list_searches
  - createHook
  - getAllHooks
  - getHook
  - deleteHook
generated: '2026-08-13'
method: generated
source: openapi/_original/meltwater-api-openapi.yaml
---

# Stream mentions to a webhook

Create a Meltwater data stream (hook) that POSTs batches of matching documents to your own HTTPS endpoint as they arrive.

## Before you call anything

- Base URL is `https://api.meltwater.com`. HTTPS only — plain HTTP fails.
- Authenticate with the customer's Meltwater API token in an `apikey` request header. Never put it in a query string, a client bundle, or source control.
- Accounts can span several companies and workspaces. Most operations take a `company_id` (and sometimes `workspace_id`) query parameter — resolve it first with `list_companies` / `list_workspaces` rather than assuming a default.
- Every response carries an `X-Request-Id`. Log it; Meltwater asks for it on support tickets.
- There is no idempotency key. A retried POST creates a second resource. Read back with the list operation before retrying a create.

## Steps

1. **Confirm entitlement.** Data Streams is a separately licensed feature. If the account lacks it, `createHook` fails — do not retry.
2. **Pick the saved search.** `GET /v3/searches` (`list_searches`); keep the `id`.
3. **Stand up a destination.** It must accept `POST` with `Content-Type: application/json` and return `200`. Meltwater documents no signature header, so authenticate the endpoint yourself (a secret path segment or an allowlist) — treat the payload as unauthenticated input.
4. **Create the stream.** `POST /v3/hooks` (`createHook`) with `search_id`, `target_url` and `template`. The response returns `hook_id`, `version`, `status` and `status_reason`; a new hook starts in `PENDING` with "Hook is in a starting state and documents will start being sent shortly".
5. **Inspect.** `GET /v3/hooks` (`getAllHooks`) and `GET /v3/hooks/{hook_id}` (`getHook`).
6. **Tear down.** `DELETE /v3/hooks/{hook_id}` (`deleteHook`). Streamed documents count against the monthly document allowance for as long as the hook lives.

## Payload shape

```json
{
  "request": { "company_id": "...", "hook_id": "...", "inputs": ["..."] },
  "documents": [ { "...": "shaped by the chosen template" } ]
}
```

Note the API calls these "hooks" for historical reasons; the product name is Data Streams. Full catalogue: `asyncapi/meltwater-webhooks.yml`.

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
