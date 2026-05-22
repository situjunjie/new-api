# Directory Structure

The backend is organized by runtime responsibility, not by vertical feature
folders. Add new code beside the existing layer that owns the behavior.

## Top-Level Runtime Layout

| Path | Responsibility |
| --- | --- |
| `main.go` | Process startup, resource initialization, frontend embedding, global Gin middleware, background jobs |
| `router/` | HTTP route registration only. Keep auth, rate-limit, and route grouping here; call controllers for behavior |
| `middleware/` | Gin middleware for auth, rate limits, request IDs, request body cleanup, CORS, recovery, i18n, cache headers |
| `controller/` | Dashboard/admin/user/API handlers. Parse request inputs, call model/service/relay helpers, return JSON |
| `service/` | Cross-controller business logic, billing/session orchestration, task coordination, OAuth/passkey helpers |
| `model/` | GORM models, persistence methods, database migrations, DB/cache synchronization |
| `relay/` | LLM relay flow, request validation, billing, upstream response handling, WebSocket and task adapters |
| `relay/channel/<provider>/` | Provider-specific adapters that implement `relay/channel.Adaptor` or `TaskAdaptor` |
| `dto/` | Client/upstream request and response DTOs |
| `types/` | Shared error, relay format, price, file-source, and generic domain types |
| `constant/` | Cross-layer constants such as channel types, API types, context keys, task platforms |
| `setting/` | Runtime settings grouped by domain. Many settings packages register themselves through imports |
| `common/` | Shared low-level helpers: JSON wrappers, Gin body reuse, Redis/cache, env, crypto, pagination |
| `logger/` | Request-aware log helpers and quota formatting |
| `pkg/` | Focused internal packages with their own tests, such as `billingexpr`, `cachex`, `ionet`, `perf_metrics` |
| `i18n/` | Backend translation bundles and helpers |
| `oauth/` | OAuth provider registry and provider implementations |

Reference files: `main.go`, `router/api-router.go`, `router/relay-router.go`,
`controller/channel.go`, `model/channel.go`, `relay/relay_adaptor.go`.

## Router and Controller Boundaries

Routers should stay declarative:

- Group endpoints by surface (`/api`, `/v1`, `/mj`, `/suno`, web fallback).
- Attach middleware at the narrowest practical group.
- Keep wildcard routes after specific routes. `router/api-router.go` places
  `/oauth/state`, `/oauth/email/bind`, and other concrete OAuth routes before
  `/oauth/:provider`.

Controllers usually own:

- Query/path/body parsing with `strconv`, `c.Query`, `c.Param`, or
  `common.UnmarshalBodyReusable`.
- Small request-specific helper functions, for example
  `parseStatusFilter`, `buildChannelListQuery`, and
  `buildFetchModelsHeaders` in `controller/channel.go`.
- Calling `model` or `service` functions, then returning `common.ApiSuccess`,
  `common.ApiError`, or explicit relay-compatible JSON.

Avoid putting new route registration in controllers, and avoid making routers
call `model` directly.

## Model and Service Boundaries

Put database row shapes and GORM query helpers in `model/`. Examples:

- `model.Channel`, `ChannelInfo.Value`, and `ChannelInfo.Scan` in
  `model/channel.go`.
- `model.Token` and token search/pagination helpers in `model/token.go`.
- database initialization and migrations in `model/main.go`.

Put orchestration that spans persistence, billing, relay state, or background
behavior in `service/`. Examples:

- `service.BillingSession` coordinates pre-consume, settlement, refunds, and
  compatibility fields for `RelayInfo`.
- task polling is wired from `main.go` through `service.GetTaskAdaptorFunc` to
  avoid a service -> relay import cycle.

## Relay and Provider Adapters

Provider-specific relay behavior belongs in `relay/channel/<provider>/`.
Adapters implement `relay/channel.Adaptor` from `relay/channel/adapter.go`.
Register new adapters in `relay/relay_adaptor.go`.

For a new channel, check all of these:

- Channel/API constants in `constant/`.
- Model list and channel name constants under the provider package.
- `GetAdaptor` registration in `relay/relay_adaptor.go`.
- Request URL, auth headers, request conversion, response conversion, and model
  list in the provider adaptor.
- Stream option support in `relay/common/relay_info.go`
  `streamSupportedChannels` when the provider supports it.
- Tests near the changed helper or provider package when behavior is
  non-trivial.

Reference files: `relay/channel/adapter.go`,
`relay/channel/openai/adaptor.go`, `relay/channel/deepseek/adaptor.go`,
`relay/common/relay_info.go`.

## Naming and File Placement

- Go package names are short and lower-case, matching existing directories.
- Use snake_case JSON field names in DTO/model tags to match existing API
  responses.
- Keep tests beside the package under test with `*_test.go`.
- Add shared helpers to an existing local helper package before creating a new
  package. Search first in `common/`, `relay/common/`, `relay/helper/`,
  `service/`, and `pkg/`.
- Keep generated or embedded frontend assets under `web/default/dist` and
  `web/classic/dist`; Go code embeds those directories from `main.go`.
