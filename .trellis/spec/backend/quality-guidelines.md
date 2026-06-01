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

## Docker Build Module Proxy Contract

### 1. Scope / Trigger
- Trigger: Docker image builds run `go mod download` inside Alpine builder
  stages and may execute from CI networks that cannot reach `proxy.golang.org`.

### 2. Signatures
- Docker build args:
  - `GOPROXY`, default `https://goproxy.cn,direct`
  - `GOSUMDB`, default `sum.golang.org`

### 3. Contracts
- Production and dev Dockerfiles must expose the same `GOPROXY` and `GOSUMDB`
  build args before `RUN go mod download`.
- The build stage must export those args through `ENV GOPROXY=... GOSUMDB=...`
  so Go commands use them consistently.
- CI may override either value with `docker build --build-arg KEY=value`.

### 4. Validation & Error Matrix
- `proxy.golang.org` timeout in CI -> keep or override `GOPROXY` to a reachable
  proxy and retry the Docker build.
- Internal/private module checksum failure -> override `GOSUMDB` or use
  `GONOSUMDB` only for the affected private module path.
- Proxy unavailable -> include `direct` as the last `GOPROXY` entry.

### 5. Good/Base/Bad Cases
- Good: `GOPROXY=https://goproxy.cn,direct` for China-hosted Jenkins agents.
- Base: `GOPROXY=https://proxy.golang.org,direct` for unrestricted networks.
- Bad: relying on Go's implicit default in Dockerfiles used by CI.

### 6. Tests Required
- For Dockerfile-only changes, run a Docker build of the affected file when the
  daemon is available and verify `go mod download` reaches the configured
  proxy.
- If Docker is unavailable locally, report that limitation and provide the exact
  build command for CI verification.

### 7. Wrong vs Correct

#### Wrong
```dockerfile
RUN go mod download
```

#### Correct
```dockerfile
ARG GOPROXY=https://goproxy.cn,direct
ARG GOSUMDB=sum.golang.org
ENV GOPROXY=${GOPROXY} GOSUMDB=${GOSUMDB}
RUN go mod download
```

## CI Docker Compose Deployment Contract

### 1. Scope / Trigger
- Trigger: Jenkins deployment runs Docker Compose on the target host, where the
  host may have either Docker Compose V2 (`docker compose`) or V1
  (`docker-compose`).

### 2. Signatures
- Compose V2 command shape: `docker compose -f <file> <command>`
- Compose V1 command shape: `docker-compose -f <file> <command>`
- Production compose path copied by CI:
  `deploy/docker-compose.prod.yml` -> `/data/new-api-gigi/docker-compose.yml`

### 3. Contracts
- Jenkins deployment scripts should use the Compose command known to exist on
  the deployment host. For the `/data/new-api-gigi` host, use
  `docker-compose`.
- Deployment scripts should print the Compose version before `pull`/`up` so CI
  logs prove which command path was executed.
- Production compose files used by CI should keep a `version` field so older
  Compose V1 installations can parse the file.
- CI should fail with an explicit message if neither Compose command is
  installed.

### 4. Validation & Error Matrix
- `unknown shorthand flag: 'f' in -f` after `docker compose -f ...` -> the host
  does not support the Compose V2 command path; fall back to `docker-compose`.
- `docker-compose: not found` and `docker compose version` fails -> install
  either Docker Compose V2 plugin or Docker Compose V1 on the deployment host.
- Compose file parse errors on V1 -> ensure the file declares a supported
  `version` and uses compatible service keys.

### 5. Good/Base/Bad Cases
- Good: use `docker-compose` when the target host reports
  `Docker Compose version v2.27.0` from `docker-compose --version`.
- Base: deployment host has one verified Compose command installed.
- Bad: Jenkinsfile hard-codes `docker compose -f ...` without a fallback.

### 6. Tests Required
- For Jenkinsfile deployment changes, run shell syntax validation where possible
  and verify the target host reports either `docker compose version` or
  `docker-compose version`.
- For compose file changes, run `docker compose -f deploy/docker-compose.prod.yml
  config` or `docker-compose -f deploy/docker-compose.prod.yml config` on a host
  with the matching command.

### 7. Wrong vs Correct

#### Wrong
```sh
docker compose -f docker-compose.yml up -d
```

#### Correct
```sh
docker-compose --version
docker-compose -f docker-compose.yml up -d
```

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
