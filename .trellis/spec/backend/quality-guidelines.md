# Quality Guidelines

Backend changes should preserve API compatibility, provider compatibility, and
cross-database behavior. This repository values local patterns over generic Go
advice.

## Required Patterns

- Run `gofmt` on changed Go files.
- Use project JSON helpers from `common/json.go`.
- Preserve SQLite, MySQL, and PostgreSQL compatibility for model/migration/query
  changes.
- Keep relay DTO optional scalar fields as pointers when absence and explicit
  zero/false are different upstream semantics.
- Keep protected new-api and QuantumNous project identity intact in source,
  metadata, README files, embedded HTML, Docker/CI config, and module paths.
- For tiered billing or dynamic pricing, read `pkg/billingexpr/expr.md` and
  follow the existing `pkg/billingexpr/*` tests and helpers.
- Search before adding helpers, constants, or provider-specific behavior.
  Common locations are `common/`, `relay/common/`, `relay/helper/`, `service/`,
  `setting/`, and `pkg/`.

## Testing Expectations

Choose the narrowest tests that cover the changed behavior:

- Pure billing/pricing/helper logic: add package-level unit tests beside the
  helper. Examples: `pkg/billingexpr/billingexpr_test.go`,
  `relay/helper/price_test.go`, `relay/common/override_test.go`.
- Controller behavior: use Gin test contexts or package tests near the
  controller. Examples: `controller/channel_test_internal_test.go`,
  `controller/model_list_test.go`, `controller/token_test.go`.
- Middleware behavior: use focused request/response tests. Example:
  `middleware/header_nav_test.go`.
- Model or DB behavior: use test DB setup and explicit database flags when the
  code depends on dialect behavior. Examples: `controller/token_test.go`,
  `service/task_billing_test.go`.

Useful verification commands:

```bash
gofmt -w <changed-go-files>
go test ./...
```

If `go test ./...` is too broad for a small change, run the affected package
tests and state that scope in the final report.

## Relay and Channel Review Checklist

When changing relay/provider behavior, check:

- Request mode and format (`types.RelayFormat*`, `relay/constant`) are correct.
- Request DTO conversion preserves explicit zero values.
- `RelayInfo` fields are updated when model names, reasoning effort, stream
  options, or task metadata are transformed.
- Provider URL and headers are built in the adaptor, not scattered across
  controllers.
- Upstream errors are converted through `types.NewAPIError` and masked before
  returning/logging.
- Billing pre-consume, settlement, and refund paths still work for stream and
  non-stream responses.
- If provider supports stream usage options, update
  `relay/common/relay_info.go` `streamSupportedChannels`.

## Database Review Checklist

- No unbounded count/search/list queries on user-controlled filters.
- No user input concatenated into SQL.
- Raw SQL handles reserved columns and dialect differences.
- Migrations are idempotent and safe when the table or column already exists.
- SQLite has a fallback when MySQL/PostgreSQL use unsupported `ALTER COLUMN` or
  transaction behavior.

## Forbidden Patterns

- New direct marshal/unmarshal calls through `encoding/json` in business code.
- MySQL-only or PostgreSQL-only SQL without a fallback.
- Returning raw keys/secrets in dashboard responses or logs.
- Bypassing the existing `api`/relay error response contracts.
- Adding a provider adaptor without registering it in `relay/relay_adaptor.go`.
- Modifying generated frontend build output manually instead of source under
  `web/default/src` or `web/classic/src`.
