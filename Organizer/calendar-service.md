# calendar-service

## 1. Purpose

`calendar-service` owns time-based composition of activities: training periods, recurrence rules (RRULE) with exceptions, and calendar views (day, week, month, year). It is the layer that answers "what should this user do on this date?" and provides the materialized event stream that the frontend renders.

## 2. Scope

**In scope**
- Periods (`/periods`): named training blocks with start/end and a set of recurrence rules.
- Recurrence rules and exceptions (`/recurrence`).
- The calendar query API (`/calendar`) that materializes events for a date range.
- Tasks/appointments routes (`/tasks`) that share the calendar surface.
- ICS feed generation.

**Out of scope**
- The activity content of any event (→ `activity-service`).
- Milestones placed within periods (→ `goals-service`); the milestone *date* lives there, the period it belongs to lives here.
- External calendar integration (Apple Health, Google Calendar) — future work; tokens live in `user-service`.

## 3. Current Capabilities

Routers: `app/routers/calendar.py`, `periods.py`, `recurrence.py`, `tasks.py`. Service: `app/services/calendar.py`. Schemas: `app/schemas/recurrence.py`. Frontend views: `FullCalendarDayView.jsx`, `FullCalendarWeekView.jsx`, `FullCalendarView.jsx` (month), `FullCalendarYearView.jsx`, `CalendarView.jsx`, `PeriodizationTimeline.jsx`. Frontend RRULE helper: `frontend/src/rruleUtils.js`.

## 4. Public API Surface

- HTTP: `/calendar?from=&to=`, `/periods`, `/recurrence`, `/tasks`, `/calendar/feed.ics`.
- Python: `events_in_range(user, from_dt, to_dt)`, `expand_recurrence(rule, range)`, `apply_exceptions(events, exceptions)`.

## 5. Data Model

Owns: `periods`, `recurrence_rules`, `recurrence_exceptions`, `period_recurrence_rules`, plus any `tasks`/`appointments` table. Reads from: `planned_activities`, `planned_milestones`.

## 6. Dependencies

- Depends on: `activity-service` (to read planned activities), `user-service`, `auth-service`, `shared-schemas`.
- Must NOT depend on: `goals-service`, `template-service`.

## 7. Cross-Cutting Concerns

- **Time zones**: a user spans time zones; expansion must use the user's preferred TZ for date boundaries.
- **DST transitions**: RRULE expansion must handle spring-forward / fall-back deterministically.
- **Performance**: large date-range queries should stream and cap event counts.
- **ICS**: feed must be authenticated via a per-user token (provisioning lives in `user-service`).

## 8. Open Questions / Known Gaps

- `parse_planned_start` and similar helpers are shared with `activity-service`; ownership unclear.
- ICS feed generation is referenced in `FEATURES_OVERVIEW.md` and the user-management requirements but the implementation status needs auditing.
- Apple Health / wearable sync (`User_Stories.md` FUT-08) is a future scope that will likely cross into this lib.
