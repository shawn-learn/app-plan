# app-organizer

**App shell — Generalized personal Organizer.**

A personal-productivity app for individuals.

## Composition

- Backend: `core-api-runtime` + Tier 0–2 platform packages (no `core-tenancy`, no `core-signatures-audit`) + `domain-organizer` (Python)
- Frontend: `web-ui-kit`, `web-api-client`, `web-auth-session`, `web-wizard-runtime`, `web-plugin-ui`, `web-calendar`, `web-features-shared`, `domain-organizer` (TS)

## Scope (locked decision)

- **Tasks & lists**
- **Calendar / scheduling** (events, recurring tasks)
- **Habits** (tracker with streaks)
- **Personal goals** (milestones)

Notes/journal are **out of scope for v1**.

## Distinguishing features

- Single-tenant or "personal tenant" mode — no per-tenant DB routing.
- Light auth: standard JWT, MFA optional, no operator realm.
- Daily-checkin wizard (sample wizard demonstrating the runtime in a non-industrial context).

## Out of scope

- Multi-tenant onboarding.
- Compliance signatures and audit log (`core-signatures-audit` not mounted).
- Fitness/exercise concepts.
