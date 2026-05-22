# Frontend Development Guidelines

The primary frontend is `web/default`: React 19, TypeScript, Rsbuild,
TanStack Router, TanStack Query, Zustand, Base UI, Tailwind CSS, and i18next.

`web/classic` is a legacy React 18 + Vite + Semi Design frontend that is still
embedded by `main.go`. Prefer `web/default` for new UI work unless the task
explicitly targets classic.

## Guidelines Index

| Guide | Use it for |
| --- | --- |
| [Directory Structure](./directory-structure.md) | Where routes, features, UI components, stores, lib helpers, styles, and i18n files live |
| [Data Flow](./data-flow.md) | Axios API client, React Query, response shape, Zustand persistence, i18n data |
| [Routing and State](./routing-and-state.md) | TanStack Router file routes, auth/setup guards, route params, stores |
| [UI Patterns](./ui-patterns.md) | Base UI wrappers, Tailwind, layout components, tables, dialogs, icons |
| [Quality Guidelines](./quality-guidelines.md) | Typecheck, lint, formatting, i18n, imports, accessibility, build checks |

## Key Reference Files

- `web/default/AGENTS.md` contains the broader frontend convention document.
- `web/default/package.json` defines scripts and dependencies.
- `web/default/src/main.tsx` creates the QueryClient and RouterProvider.
- `web/default/src/lib/api.ts` owns the shared axios instance and interceptors.
- `web/default/src/routes/__root.tsx` owns root route guards and app shell
  providers.
- `web/default/src/features/channels/api.ts` and
  `web/default/src/features/users/api.ts` show feature API modules.
- `web/default/src/components/ui/button.tsx` shows the Base UI + cva + Tailwind
  wrapper pattern.
