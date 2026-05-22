# Backend Development Guidelines

This backend is the Go side of new-api: an AI API gateway/proxy that aggregates
OpenAI-compatible, Claude, Gemini, Azure, AWS Bedrock, Midjourney/task-style,
and other upstream providers behind one API with accounts, quota, billing,
rate limiting, channel management, and an embedded admin UI.

The local architecture is:

```text
router -> middleware -> controller -> service/model -> relay/channel adapters
```

Important project rules from `CLAUDE.md` and source inspection:

- Keep support for SQLite, MySQL, and PostgreSQL unless the change is explicitly
  scoped away from persistence.
- Use `common.Marshal`, `common.Unmarshal`, `common.DecodeJson`, and
  `common.UnmarshalBodyReusable`; do not introduce direct `encoding/json`
  marshal/unmarshal calls in business code.
- Preserve explicit zero values in request DTOs that are parsed from client JSON
  and then re-marshaled upstream. Optional scalar request fields should be
  pointer types with `omitempty`.
- Do not remove or rename protected new-api / QuantumNous project identity,
  attribution, module paths, image names, or metadata.
- For tiered/dynamic billing changes, read `pkg/billingexpr/expr.md` first.

## Guidelines Index

| Guide | Use it for |
| --- | --- |
| [Directory Structure](./directory-structure.md) | Where Go routes, controllers, services, models, relay adapters, settings, and utilities belong |
| [Database Guidelines](./database-guidelines.md) | GORM models, migrations, cross-database SQL, transactions, query safety |
| [Error Handling](./error-handling.md) | Dashboard API errors, relay errors, upstream masking, retry behavior |
| [Logging Guidelines](./logging-guidelines.md) | `logger.*`, `common.SysLog`, request IDs, quota formatting, secret handling |
| [Quality Guidelines](./quality-guidelines.md) | Backend review checks, tests, forbidden patterns, channel/billing invariants |

## Reference Files

- `main.go` initializes resources, embeds both frontends, starts background jobs,
  installs global middleware, and attaches routers.
- `router/api-router.go`, `router/relay-router.go`, and `router/web-router.go`
  define the main HTTP surfaces.
- `controller/channel.go`, `service/billing_session.go`, `model/channel.go`,
  and `relay/channel/adapter.go` show the core dashboard, billing, persistence,
  and provider-adapter patterns.
