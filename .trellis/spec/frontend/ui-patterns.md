# UI Patterns

The default frontend is an operational dashboard. Keep interfaces dense,
scan-friendly, and consistent with the existing component system.

## Component Primitives

Use wrappers from `src/components/ui` before importing UI primitives directly.
They combine Base UI primitives, Tailwind classes, variants, accessibility
attributes, and local conventions.

Examples:

- `components/ui/button.tsx` wraps `@base-ui/react/button`, uses `cva`, and
  exports `Button` plus `buttonVariants`.
- `components/ui/dialog.tsx`, `drawer.tsx`, `dropdown-menu.tsx`, `tabs.tsx`,
  `table.tsx`, and related files provide reusable behavior and styling.

Do not fork a one-off button/input/dialog style inside a feature when an
existing `components/ui` primitive can be composed.

## Layout

Use `src/components/layout` exports for page shells and section pages:

- `AuthenticatedLayout` and `PublicLayout` for app shells.
- `SectionPageLayout` for standard dashboard pages with title, description,
  actions, and content.
- workspace/sidebar/top-nav helpers for navigation structure.

Reference files: `web/default/src/components/layout/index.ts`,
`web/default/src/components/layout/types.ts`,
`web/default/src/features/models/index.tsx`.

## Tables and Lists

Use shared data-table components in `src/components/data-table` for repeated
admin tables, pagination, column headers, filters, empty states, and mobile card
fallbacks. Use TanStack Table and React Virtual for complex or large tables.

Keep list API params typed in the feature `types.ts`, and keep table-specific
column/rendering logic inside the feature.

## Styling

- Use Tailwind utilities and `cn()` for class composition.
- Use variant helpers (`class-variance-authority`) for reusable component
  variants.
- Use CSS variables and `dark:` variants for theme-aware styles.
- Avoid inline styles except for dynamic values that cannot be expressed with
  stable classes.
- Keep UI controls stable in size so loading labels, icons, and dynamic values
  do not shift layout.

## Icons and Actions

Use existing icon libraries already present in the app:

- `lucide-react` for common action icons.
- `@hugeicons/react` or local assets where existing feature code uses them.
- brand icons from `src/assets/brand-icons`.

Icon-only buttons should have accessible labels or tooltips. Text buttons are
fine for clear commands, especially in forms and tables.

## Dialogs, Drawers, and Toasts

- Feature-specific create/edit/delete dialogs belong under the feature's
  `components/dialogs` or equivalent folder.
- Use shared confirm/risk acknowledgement components for destructive actions
  when available.
- Use `sonner` toasts through existing error handling and feature actions; do
  not create another notification system.

## Accessibility

- Prefer semantic elements through shared components.
- Preserve keyboard behavior from Base UI primitives.
- Connect labels and fields in forms.
- Use `aria-label` for icon-only controls and `aria-hidden` for decorative
  icons.
