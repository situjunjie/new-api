# Error Handling

This project has two distinct error surfaces:

- Dashboard/admin APIs usually return HTTP 200 with
  `{ "success": false, "message": "..." }`.
- Relay APIs must return provider-compatible error shapes and retry/auto-ban
  metadata through `types.NewAPIError`.

Choose the surface based on the route you are changing.

## Dashboard API Errors

Use helpers from `common/gin.go` for ordinary `/api` handlers:

- `common.ApiSuccess(c, data)` for `{ success: true, message: "", data }`.
- `common.ApiError(c, err)` for `{ success: false, message: err.Error() }`.
- `common.ApiErrorMsg(c, msg)` when there is no error value.
- `common.ApiErrorI18n` and `common.ApiSuccessI18n` when the response message
  should be translated server-side.

Existing controllers also use explicit `c.JSON(http.StatusOK, gin.H{...})`
when the response shape needs extra fields or compatibility with older callers.
Keep that style local to the handler and match nearby responses.

Reference files: `common/gin.go`, `controller/channel.go`,
`controller/setup.go`, `controller/user.go`.

## Relay Errors

For relay and provider-adapter code, use `types.NewAPIError` and the constructors
in `types/error.go`.

The error object carries:

- underlying `Err`
- provider-specific `RelayError`
- `ErrorType` and `ErrorCode`
- HTTP status code
- retry flags such as `ErrOptionWithSkipRetry`
- logging flags such as `ErrOptionWithNoRecordErrorLog`
- optional metadata

Convert relay errors with `ToOpenAIError`, `ToClaudeError`, and related methods
instead of hand-building provider error JSON. These methods also mask sensitive
information for most error codes.

Reference files: `types/error.go`, `types/channel_error.go`,
`relay/compatible_handler.go`, `service/billing_session.go`.

## Request Parsing and Body Reuse

- Use `common.UnmarshalBodyReusable(c, &value)` when middleware or relay code
  may need to read the request body more than once.
- Use `common.GetBodyStorage`, `common.GetRequestBody`, and
  `middleware.BodyStorageCleanup` rather than manually reading and replacing
  `c.Request.Body`.
- For JSON marshal/unmarshal, use `common.Marshal`, `common.Unmarshal`,
  `common.UnmarshalJsonStr`, or `common.DecodeJson`.

Reference files: `common/gin.go`, `middleware/body_cleanup.go`,
`relay/helper/valid_request.go`, `middleware/jimeng_adapter.go`.

## Logging Errors

- Log internal diagnostic detail with `common.SysError`, `common.SysLog`, or
  `logger.LogError` before returning a generic user-facing message when needed.
- Do not return raw upstream secrets, API keys, credentials, or unmasked channel
  data to clients.
- Use `NewAPIError.MaskSensitiveError` or related conversion helpers for relay
  errors that can contain upstream request/response detail.

## Common Mistakes

- Do not use dashboard `{ success: false }` responses in relay endpoints that
  must look like OpenAI, Claude, Gemini, or task-provider errors.
- Do not drop explicit client zero values in relay DTOs. Optional scalar request
  fields that are re-marshaled upstream should be pointers with `omitempty`.
- Do not parse request bodies with direct `io.ReadAll` unless the code restores
  reusable storage exactly like `common/gin.go`; prefer the existing helpers.
- Do not introduce direct `encoding/json` marshal/unmarshal calls in business
  code. `encoding/json.RawMessage` may be used as a type, but serialization
  should go through `common`.
