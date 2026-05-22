# Routing and State

The default frontend uses TanStack Router file routes and a small set of global
providers installed at the root.

## Root Setup

`web/default/src/main.tsx`:

- creates the QueryClient
- creates the TanStack Router from `routeTree.gen`
- registers router types through module augmentation
- wraps the app with `QueryClientProvider`, `ThemeProvider`, `FontProvider`,
  `DirectionProvider`, and `RouterProvider`
- applies cached system branding before the first network refresh

`web/default/src/routes/__root.tsx`:

- uses `createRootRouteWithContext<{ queryClient: QueryClient }>()`
- loads system configuration
- preserves affiliate codes from URL query params
- installs `NavigationProgress`, `Outlet`, `Toaster`, and devtools in
  development
- checks setup status before navigation and redirects to `/setup` when needed

## Route Files

- Use TanStack Router route files under `src/routes`.
- Authenticated pages live under `_authenticated`.
- Route params use `$param` filenames, for example `models/$section.tsx`.
- Prefer `getRouteApi`, `useNavigate`, typed params, and typed search over
  stringly navigation.
- Keep route files thin: load route params/search and render the feature entry
  component from `src/features/<feature>`.
- Do not edit generated `routeTree.gen`.

Reference files: `web/default/src/routes/__root.tsx`,
`web/default/src/routes/_authenticated/models/$section.tsx`,
`web/default/src/features/models/index.tsx`.

## Auth State

Auth state is cached in `useAuthStore` and restored from localStorage. Root
guards avoid calling `getSelf()` on every navigation; authenticated routes are
responsible for redirect behavior when the cached user is absent or expired.

When a 401 is detected globally:

- axios/query error handlers show the session-expired toast
- `useAuthStore.getState().auth.reset()` clears cached auth state
- navigation redirects to sign-in where appropriate

Reference files: `web/default/src/stores/auth-store.ts`,
`web/default/src/lib/api.ts`, `web/default/src/main.tsx`.

## Section State and URL State

For tab-like app sections, prefer route params over independent local state so
deep links work. `features/models/index.tsx` reads `$section`, syncs it to the
feature provider, and navigates with typed params when the tab changes.

Use provider/context state only for state shared by a feature's child
components. Do not duplicate durable URL state in local component state unless a
component library requires a controlled value; if so, sync from the route.

## Browser Storage

Browser storage access should be guarded with `typeof window !== 'undefined'`
and wrapped in try/catch. Existing stores and root setup do this to avoid
runtime failures from malformed localStorage or unavailable storage.
