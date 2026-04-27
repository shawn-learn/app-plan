# core-units (Python)

**Tier 0 — Foundation.**

Measurement units, conversions, and user-scoped unit preferences. Has a TS twin at `packages/ts/core-units/`.

## Source material

`Organizer/shared-units.md` — already a normalized DB concept in the Organizer project.

## Responsibilities

- Unit definitions (mass, distance, duration, pace, power, HR, etc.).
- Conversions between unit systems (metric ↔ imperial).
- User-preference resolution: "this user logs distance in km but reads pace in min/mi."
- Locale-aware formatting helpers.

## Dependencies

- External: `pydantic`
- Internal: `core-domain-contracts`

## Consumers

`core-users`, `domain-fitness`, `web-ui-kit` (for display formatters), `web-features-shared`.
