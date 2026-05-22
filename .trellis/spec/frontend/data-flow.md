# Frontend Data Flow

The default frontend uses a shared axios instance plus TanStack Query for server
data. Keep API calls and response handling consistent with the existing feature
modules.

## API Client

Use `api` from `web/default/src/lib/api.ts` for HTTP calls.

The shared instance provides:

- same-origin `baseURL`
- `withCredentials: true`
- `Cache-Control: no-store`
- `New-Api-User` header from localStorage when available
- concurrent GET de-duplication unless `disableDuplicate` is set
- global business error toasts for `{ success: false, message }`
- global HTTP error handling unless `skipErrorHandler` is set

Feature API functions should return `res.data`, not the full axios response,
unless the caller explicitly needs headers/status.

Reference files: `web/default/src/lib/api.ts`,
`web/default/src/features/channels/api.ts`,
`web/default/src/features/users/api.ts`,
`web/default/src/features/setup/api.ts`.

## Backend Response Shape

Most dashboard API responses are:

```ts
{
  success: boolean
  message?: string
  data?: T
}
```

Define or reuse typed response aliases in each feature's `types.ts`. Keep field
names snake_case when they mirror backend JSON. Convert to presentation-friendly
labels at the component boundary, not in the shared API client.

## React Query

`src/main.tsx` configures QueryClient defaults:

- retry is disabled in development and capped in production
- 401 and 403 are not retried
- query stale time defaults to 10 seconds
- global query errors handle 401 and 500 navigation/toasts
- global mutation errors call `handleServerError`

For feature work:

- Put query-key builders beside the feature when a feature has multiple queries.
  `web/default/src/features/models/lib` and `deploymentsQueryKeys` are an
  example.
- Use `useQueryClient().invalidateQueries` or prefetch APIs instead of manually
  re-fetching scattered requests.
- Use `skipBusinessError`, `skipErrorHandler`, or `disableDuplicate` only for
  specific flows that need custom UX. Do not disable global handling by default.

## Zustand and Local Persistence

Use Zustand stores in `src/stores/` for client state that outlives one component
tree. Patterns:

- `auth-store.ts` restores and writes the cached user in localStorage.
- `notification-store.ts` uses `persist` for notice/announcement read state.
- system/UI preference stores keep display configuration close to UI behavior.

Prefer React Query for server state. Use Zustand for authenticated user cache,
UI settings, local-only flags, and persisted preferences.

## Internationalization

Use `useTranslation()` in React components so language changes re-render. Use
`i18next.t` only in non-React utility code where reactive updates are not
needed.

User-visible strings should be passed through `t(...)`. When constants store
labels or toast messages, store translation keys and translate at render/use
time. Keep locale files under `web/default/src/i18n/locales/`.

Run `bun run i18n:sync` from `web/default` after adding or moving significant
user-visible strings.
