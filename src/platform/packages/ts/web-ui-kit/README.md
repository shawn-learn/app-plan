# web-ui-kit

**Tier 3 — Frontend platform.**

Themable React component library. Reusable primitives, form inputs, layout, charts wrappers, React Query setup, i18n scaffolding.

## Source material

`Organizer/frontend-ui-kit.md` (today's `frontend/src/components/`) + Process Wizard `ui-standards.md` and `brand-guidelines.md`.

## Responsibilities

- MUI-based primitive components with consistent theming hooks.
- Form inputs aligned with the step-field catalog (text, numeric with units, date/time, enumeration, dropdown, checkbox, rating, signature pad).
- Layout primitives (page shell, sidebar, drawer, modal, toast).
- Recharts wrappers with house style.
- React Query provider + default options.
- i18n provider + translation hooks (English-only content in v1, infrastructure ready for more).
- Theme-token system so each app can rebrand without forking components.

## Dependencies

- External: `@mui/material`, `react`, `react-hook-form`, `recharts`, `@tanstack/react-query`, `react-i18next`
- Internal: `core-schemas-ts`, `core-units`

## Consumers

Every TS feature/domain package and app shell.
