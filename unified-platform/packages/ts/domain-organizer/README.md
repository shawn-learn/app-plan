# domain-organizer (TypeScript)

**Domain plugin — Personal organizer (frontend).**

Twin of `packages/py/domain-organizer/`. Tasks & lists, calendar/scheduling glue, habits, personal goals.

## Responsibilities

- Task list & detail views, list management, drag-to-reorder.
- Habit tracker with streak visualizations.
- Personal-goal CRUD specializing the generic goals UI from `web-features-shared`.
- "Today" dashboard pulling from `web-calendar` + tasks + habits.
- Daily-checkin wizard (registers via `web-plugin-ui`).

## Dependencies

- External: `react`, `react-router-dom`
- Internal: `web-ui-kit`, `web-api-client`, `web-auth-session`, `web-wizard-runtime`, `web-plugin-ui`, `web-calendar`, `web-features-shared`, `core-schemas-ts`

## Consumers

`app-organizer` primarily. `app-coach` may bundle for athletes' personal scheduling.

## Boundaries

No notes/journal in v1 (locked decision). No fitness or industrial concepts.
