# app-industrial

**App shell — Industrial Process Wizard.**

Multi-tenant compliance/SOP execution platform per Process Wizard `system-requirements.md`.

## Composition

- Backend: `core-api-runtime` + Tier 0–2 platform packages + `core-tenancy` (active) + `core-signatures-audit` (active) + `domain-industrial` (Python)
- Frontend: `web-ui-kit`, `web-api-client`, `web-auth-session`, `web-wizard-runtime`, `web-plugin-ui`, `web-form-editors`, `web-features-shared`, `web-calendar`, `domain-industrial` (TS)

## Distinguishing features

- Per-tenant isolated databases via `core-tenancy`.
- Operator onboarding API at `/ops/tenants`.
- Every run produces a signed canonical record (Ed25519).
- Append-only audit log for compliance review.
- Step-field catalog from `system-requirements.md` §5.3.1.
- Branch namespaces (`main`, `development`) for procedure versions.
- Offline procedure execution with deferred sync (`core-sync`).
- Localization-ready (English-only content in v1).

## Out of scope

- Workouts, exercise catalogs, FIT files, training metrics.
- Personal task/list/habit features.
