# Frontend Quality Guidelines

Frontend work should preserve type safety, i18n coverage, route type safety, and
the shared UI system.

## Required Commands

Run commands from `web/default` for default frontend work:

```bash
bun run typecheck
bun run lint
bun run build:check
```

For small changes, run the narrowest reliable subset and report the scope. Use
`bun run i18n:sync` when adding or moving user-facing strings.

For `web/classic` changes, use scripts from `web/classic/package.json`.

## TypeScript and Imports

The default frontend enforces:

- TypeScript recommended rules.
- React hooks rules.
- TanStack Query eslint rules.
- `no-console`.
- `@typescript-eslint/consistent-type-imports`.
- no duplicate imports.

Use `import type` for types. Prefer `unknown` or concrete types over `any`.
Route files are allowed to export TanStack route objects, as configured in
`eslint.config.js`.

Reference files: `web/default/eslint.config.js`,
`web/default/tsconfig.json`, `web/default/tsconfig.app.json`.

## API and State Review Checklist

- API functions use `api` from `src/lib/api.ts`.
- Response types match backend `{ success, message, data }` shape and snake_case
  fields.
- React Query is used for server state; Zustand is reserved for auth/UI/local
  persistent state.
- 401/session behavior still clears `useAuthStore` and redirects as expected.
- Query keys are stable and invalidated after mutations.
- User-visible errors are translated or passed through existing global handlers.

## Routing Review Checklist

- New pages live under the correct `src/routes` group.
- Route files render feature entry components rather than owning large page
  logic.
- Route params/search are typed and validated where applicable.
- Navigation uses `Link`, `useNavigate`, or TanStack route helpers, not direct
  `window.location` except for external navigation or one-off browser concerns.
- Generated route tree is not edited manually.

## UI Review Checklist

- Reuse `components/ui` primitives and `components/layout` before introducing
  new patterns.
- Use `t(...)` for visible strings.
- Loading, empty, and error states are present for async views.
- Tables remain usable on narrow screens or use existing mobile table/card
  fallbacks.
- Controls have accessible labels and stable dimensions.
- Dark/light theme behavior is preserved.

## Formatting and Build

Use the package scripts rather than ad hoc commands:

- `bun run format:check` for formatting checks.
- `bun run format` for formatting changes.
- `bun run build` for production build.
- `bun run copyright:check` when touching files that require project copyright
  headers.

Do not manually edit build output under `web/default/dist` or
`web/classic/dist`; edit source and rebuild when needed.
