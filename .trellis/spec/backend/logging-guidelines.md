# Logging Guidelines

Logging is split between process/system logs and request-aware logs.

## Logger APIs

Use the existing helpers:

- `common.SysLog` for lifecycle events, startup messages, migrations, background
  jobs, and non-request diagnostics.
- `common.SysError` for system-level errors that should be visible outside a
  request context.
- `common.FatalLog` for unrecoverable startup or resource initialization errors.
- `logger.LogInfo(ctx, msg)`, `logger.LogWarn(ctx, msg)`, and
  `logger.LogError(ctx, msg)` for request-aware application logs.
- `logger.LogDebug(ctx, format, args...)` for debug-only request-aware detail.
  It emits only when `common.DebugEnabled` is true.
- `logger.LogJson` is for debug/test diagnostics only.

Reference files: `logger/logger.go`, `main.go`, `service/billing_session.go`,
`controller/task_video.go`.

## Format and Request IDs

`logger.logHelper` writes lines as:

```text
[LEVEL] YYYY/MM/DD - HH:MM:SS | request-id-or-SYSTEM | message
```

Request IDs are attached by `middleware.RequestId()` in `main.go`; pass
`*gin.Context` or `c.Request.Context()` to `logger.*` so request IDs survive.
Use `nil` only for truly background/system logs.

## Log Levels

- `INFO`: successful business milestones worth auditing, such as billing
  pre-consume, refunds, and background task state.
- `WARN`: degraded but recoverable behavior, skipped duplicate work, failed
  refunds that were already handled, or suspicious state.
- `ERR`: failures that require attention or explain why a request/background
  task failed.
- `DEBUG`: detailed intermediate state, provider payload summaries, pricing
  traces, and JSON diagnostics. Guarded by `common.DebugEnabled`.

## Quota and Billing Logs

Use quota formatters instead of ad hoc conversions:

- `logger.LogQuota(quota)` includes the localized unit label.
- `logger.FormatQuota(quota)` returns the display value without the trailing
  unit word.

These helpers respect `operation_setting.GetQuotaDisplayType()` and are used in
`service/billing_session.go` and `controller/task_video.go`.

## Sensitive Data

- Do not log API keys, full token keys, passwords, OAuth credentials, payment
  secrets, or raw channel `Key` values.
- Use masking helpers such as `common.MaskSensitiveInfo`,
  `types.NewAPIError.MaskSensitiveError`, `model.MaskTokenKey`, and channel key
  omission before emitting or returning sensitive data.
- Be especially careful with debug payload logs in relay/provider code; upstream
  errors and request bodies can contain user prompts and credentials.

## File Logging

`logger.SetupLogger` writes to `LOG_DIR` when configured and rotates after a
large line count. Do not replace `gin.DefaultWriter` or `gin.DefaultErrorWriter`
directly; use the existing setup path.
