# core-scheduling

**Tier 2 — Platform.**

Generic planned-occurrence engine: RRULE recurrence, exception dates, period grouping. Applies to procedures (industrial), tasks (organizer), and workout sessions (fitness) alike.

## Source material

`Organizer/calendar-service.md` (planning slice) + `Organizer/activity-service.md` (`planned_activities`, RRULE, exception dates, periods).

## Responsibilities

- `PlannedOccurrence`: a scheduled instance of *some template* at a date/time, with optional RRULE and exception dates.
- Periods: named time-bounded grouping of planned occurrences.
- Expansion: given a date range, materialize concrete occurrences from RRULEs.
- Subject-agnostic: doesn't care whether the template is a procedure, task, or workout.

## Dependencies

- External: `dateutil`, `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-templates`, `core-users`, `core-data-access`

## Consumers

`core-calendar` (views), `domain-organizer` (tasks have due dates), `domain-fitness` (workout schedules), `domain-industrial` (planned procedure executions).
