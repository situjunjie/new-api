# Database Guidelines

The backend supports SQLite, MySQL, and PostgreSQL. Database changes must keep
all three working unless the task explicitly narrows scope.

## ORM and Connection Pattern

- Use GORM v2 through `model.DB` and `model.LOG_DB`.
- `model/main.go` opens SQLite, MySQL, or PostgreSQL from `SQL_DSN` and
  `LOG_SQL_DSN`, sets `common.UsingSQLite`, `common.UsingMySQL`, or
  `common.UsingPostgreSQL`, and runs migrations only on the master node.
- Prefer GORM APIs (`Model`, `Where`, `Find`, `Count`, `Updates`, `Create`,
  `Save`, `AutoMigrate`) over raw SQL.
- Use `gorm.Expr` for atomic increments, for example quota updates in
  `model/checkin.go` and `model/topup.go`.

## Models and Column Tags

- Models live in `model/` and expose JSON tags matching the API response shape.
- Time fields commonly use Unix timestamps in `int64` fields such as
  `CreatedTime`, `UpdatedTime`, `ExpiredTime`, and `CreatedAt`.
- For sensitive fields, provide clean/masked helpers. `model.Token.Clean`,
  `MaskTokenKey`, and channel key omission in `controller/channel.go` are local
  examples.
- For JSON-like columns, prefer `TEXT` storage plus `common.Marshal` /
  `common.Unmarshal` in `driver.Valuer` and `sql.Scanner` implementations.
  `model.ChannelInfo.Value` and `Scan` are the pattern.

## Cross-Database SQL

Raw SQL is allowed only when GORM cannot express the behavior cleanly. When it
is needed:

- Use `commonGroupCol` and `commonKeyCol` from `model/main.go` for reserved
  columns such as `group` and `key`.
- Use `commonTrueVal` and `commonFalseVal` for boolean literals in raw SQL.
- Branch with `common.UsingPostgreSQL`, `common.UsingMySQL`, and
  `common.UsingSQLite` for dialect-specific SQL.
- Keep LIKE escaping portable. `model/token.go` uses `!` as the escape
  character and sanitizes `%`, `_`, and `!` before building `LIKE ? ESCAPE '!'`
  filters.
- Avoid introducing database-specific JSON operators or column types without a
  fallback for the other supported databases.

Reference files: `model/main.go`, `model/channel.go`, `model/token.go`,
`model/model_meta.go`, `model/ability.go`.

## Migrations

- Add new models to `migrateDB` in `model/main.go`.
- Use `DB.AutoMigrate` for normal table/column additions.
- For one-off type migrations, make them idempotent: check table existence,
  column existence, and current type first. `migrateTokenModelLimitsToText` and
  `migrateSubscriptionPlanPriceAmount` are the examples.
- SQLite does not support every `ALTER COLUMN` operation. Use
  `ALTER TABLE ... ADD COLUMN` and table-specific helpers where needed.
  `ensureSubscriptionPlanTableSQLite` is the local pattern.
- Keep log DB migrations separate in `migrateLOGDB`.

## Transactions

- Use `DB.Transaction(func(tx *gorm.DB) error { ... })` for multi-row writes on
  MySQL/PostgreSQL or when the code path is known to be compatible.
- SQLite may need a sequential operation plus manual rollback if the code can be
  called inside another transaction. `model.UserCheckin` branches to
  `userCheckinWithoutTransaction` for this reason.
- Keep transaction-scoped reads/writes on `tx`, not `model.DB`.
- Make payment and quota operations idempotent. `model.ManualCompleteTopUp`
  checks current order status before applying quota.

## Query Safety

- Page list endpoints with `Limit` and `Offset`; use `common.GetPageQuery` in
  controllers.
- Bound expensive searches. `SearchUserTokens` and `SearchUserTopUps` enforce
  hard count/search limits.
- Do not concatenate user input into SQL. Column names may be selected from a
  whitelist, as `model.NewChannelSortOptions` does with `channelSortColumns`.
- For admin list APIs, omit or mask secrets before returning rows.
