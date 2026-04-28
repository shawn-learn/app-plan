# app-coach

**App shell — Personal Trainer / Coach with team functionality.**

Fitness coaching platform — the merged successor to today's Organizer fitness focus and Process Wizard's fitness plugin.

## Composition

- Backend: `core-api-runtime` + Tier 0–2 platform packages + `core-teams` (active) + `domain-fitness` (Python) + optionally `domain-organizer` (athlete personal scheduling)
- Frontend: `web-ui-kit`, `web-api-client`, `web-auth-session`, `web-wizard-runtime`, `web-plugin-ui`, `web-form-editors`, `web-calendar`, `web-features-shared`, `domain-fitness` (TS) + optionally `domain-organizer` (TS)

## Tenancy

Single DB with org scoping (locked decision). Each coaching practice is an org row. `core-tenancy` remains available for a future flip to per-tenant DBs.

## Distinguishing features

- Coach console + athlete portal.
- Intake (PAR-Q) → assessments → periodized plan → session logging → progress analytics loop, all via wizards.
- FIT file upload and parsing.
- Training-load metrics (ACWR, TSS, zones).
- Athlete↔coach teams via `core-teams`.
- Optional rehab signatures (opt-in `core-signatures-audit` for liability-sensitive flows).
- Optional personal scheduling for athletes (`domain-organizer` bundled).

## Out of scope

- Multi-tenant operator onboarding.
- Industrial procedure execution / deviations.
