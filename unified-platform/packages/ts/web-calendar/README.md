# web-calendar

**Tier 3 — Frontend platform.**

Calendar views (day/week/month/year) consuming `core-scheduling` data via `web-api-client`.

## Source material

Organizer calendar feature components.

## Responsibilities

- Day, week, month, year view components.
- Drag-to-schedule and drag-to-reschedule of planned occurrences.
- Recurrence editor UX.
- ICS subscription URL surface.
- Subject-agnostic rendering — accepts a render-prop or slot for each occurrence.

## Dependencies

- External: `react`, `dayjs`, `react-big-calendar` (or similar)
- Internal: `web-ui-kit`, `web-api-client`, `core-schemas-ts`

## Consumers

`web-features-shared`, `domain-organizer` (TS), `domain-fitness` (TS), `app-industrial`.
