# domain-organizer (Python)

**Domain plugin — Personal organizer.**

Backend half of the personal-organizer domain. Has a TS twin at `packages/ts/domain-organizer/`.

## Scope

Per the locked decision: **tasks & lists, calendar/scheduling glue, habits, and personal goals**. Notes/journal are **out of scope for v1**.

## Responsibilities

- **Tasks & lists:** task entity (title, status, priority, due date, list membership), list entity, ordering.
- **Calendar/scheduling glue:** mounts `core-scheduling` for personal events; recurring tasks via RRULE.
- **Habits:** habit definition, streak tracking, daily-completion log.
- **Personal goals:** specializations of `core-goals.Milestone` for individual targets.
- Plugin registration with `core-plugin-contracts` so habit/goal evaluations participate in the run pipeline (e.g., a daily-checkin wizard updates streaks).

## Dependencies

- External: `pydantic`, `sqlalchemy`
- Internal: `core-domain-contracts`, `core-plugin-contracts`, `core-templates`, `core-scheduling`, `core-calendar`, `core-goals`, `core-users`, `core-data-access`

## Consumers

`app-organizer` primarily. `app-coach` may optionally bundle this for athletes' personal scheduling.

## Boundaries

No multi-tenant onboarding logic, no industrial signatures, no fitness/exercise concepts.
