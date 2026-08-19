---
name: Track brand visibility in AI assistants (v4)
description: Use the Meltwater v4 API to fetch GenAI Lens prompts and run analytics on how a brand appears in ChatGPT, Gemini and Perplexity answers.
api: openapi/meltwater-api-v4-openapi.yml
operations:
  - listLLMPrompts
  - listLLMFolders
  - analyze
  - createExport
  - listExports
  - getExport
  - deleteExport
generated: '2026-08-13'
method: generated
source: openapi/_original/meltwater-api-openapi.yaml
---

# Track brand visibility in AI assistants (v4)

Use the Meltwater v4 API to fetch GenAI Lens prompts and run analytics on how a brand appears in ChatGPT, Gemini and Perplexity answers.

## Before you call anything

- Base URL is `https://api.meltwater.com`. HTTPS only — plain HTTP fails.
- Authenticate with the customer's Meltwater API token in an `apikey` request header. Never put it in a query string, a client bundle, or source control.
- Accounts can span several companies and workspaces. Most operations take a `company_id` (and sometimes `workspace_id`) query parameter — resolve it first with `list_companies` / `list_workspaces` rather than assuming a default.
- Every response carries an `X-Request-Id`. Log it; Meltwater asks for it on support tickets.
- There is no idempotency key. A retried POST creates a second resource. Read back with the list operation before retrying a create.

## Steps

1. **This is v4 only.** Base URL is `https://api.meltwater.com/v4`. AI Visibility (GenAI Lens) does not exist on v3, and v4 is `4.0.0-beta` — treat shapes as unstable and pin nothing.
2. **List the prompt folders.** `GET /account/llm/prompts/folders` (`listLLMFolders`).
3. **List the prompts.** `GET /account/llm/prompts` (`listLLMPrompts`) — these are the questions Meltwater puts to the assistants on your behalf.
4. **Analyze.** `POST /analytics/analyze` (`analyze`) is the single unified analytics entry point in v4: aggregate metrics, date histograms and top terms over mentions, visibility, sentiment and share of voice. Nested analysis is supported in one request rather than a call per cut, which is the main reason to prefer v4 over the v3 `top_*` family.
5. **Export if you need the rows.** `POST /content/export` (`createExport`), then `GET /content/export/{exportId}` (`getExport`), `GET /content/export` (`listExports`), `DELETE /content/export/{exportId}` (`deleteExport`). Explore+ exports on v4 are flagged Beta.

## Migration note

v3 remains the complete surface (80 operations). v4 currently carries only exports, unified analyze, and LLM prompts. Meltwater states v4 "in time will support all features" but publishes no dated migration plan and no `Sunset` headers — check `lifecycle/meltwater-lifecycle.yml` before you build anything long-lived on v3 paths that v4 already duplicates.

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
