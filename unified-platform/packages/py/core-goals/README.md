# core-goals

**Tier 2 — Platform.**

Generic milestones and badges. No fitness specifics live here.

## Source material

`Organizer/goals-service.md` (planned milestones, milestone log, badges) — generalized.

## Responsibilities

- `Milestone`: a named target with a deadline and a completion predicate (predicate is data; evaluation happens at the consumer).
- `MilestoneLog`: append-only completion records.
- `Badge`: an award unlocked by milestones (or by domain-specific evaluator output).
- Milestone scheduling delegates to `core-scheduling`.

## Dependencies

- External: `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-users`, `core-scheduling`, `core-data-access`

## Consumers

`domain-organizer` (personal goals & habits), `domain-fitness` (training milestones, badges), `domain-industrial` (compliance milestones — e.g., "annual recertification").
