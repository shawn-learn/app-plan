# core-calendar

**Tier 2 — Platform.**

Calendar views (day/week/month/year) and ICS feed support, built on `core-scheduling`.

## Source material

`Organizer/calendar-service.md` (views slice).

## Responsibilities

- Day, week, month, year browsers — query helpers returning materialized occurrences for a range.
- ICS feed generation per user / per team.
- Free/busy calculation (used by future scheduling assistants).

## Dependencies

- External: `icalendar`, `pydantic`
- Internal: `core-domain-contracts`, `core-scheduling`, `core-users`

## Consumers

`web-calendar` is the primary frontend consumer. App shells expose ICS endpoints.
