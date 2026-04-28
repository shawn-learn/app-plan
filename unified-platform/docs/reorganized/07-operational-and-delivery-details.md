# 07 — Operational, Quality, and Delivery Details

## 1) Development and Testing Baseline

- Python backend: FastAPI + SQLAlchemy (async) + Pydantic v2.
- Frontend: React + TypeScript + MUI + React Hook Form + Recharts.
- Required test layers:
  - unit tests (per package)
  - API/contract tests
  - cross-language schema round-trip fixtures
  - app smoke tests (login + run lifecycle)

## 2) Observability and Troubleshooting

- Preserve documented logging patterns for backend and frontend runtime debugging.
- Standardize structured logs for run lifecycle, auth events, sync conflicts, and signature operations.
- Keep operator troubleshooting runbooks in docs for local/dev/deploy environments.

## 3) Infrastructure and Security Expectations

- CI/CD with package-level build/test matrix.
- Database backup/disaster recovery expectations retained from infra TODO docs.
- Secret management and environment segregation are mandatory before production.
- Security review includes token lifecycle, signature verification, and least-privilege access controls.

## 4) Product Readiness and Next Steps

- UAT, bug-fix, performance, and compliance checklists remain required gates.
- Keep phased rollout discipline from roadmap, with measurable completion criteria per phase.

## 5) Deferred/Backlog Items

- Messaging full implementation beyond phase-0 seam.
- Additional UI polish and unresolved TODO items from legacy docs.
- Optional notes/journal scope (explicitly out of v1 for organizer).

## 6) Documentation Lifecycle

- `unified-platform/docs/reorganized/*` is canonical.
- Legacy docs not marked `DELETE` remain active references until migrated.
- A legacy file can only be marked `DELETE` after its key requirements are captured in this reorganized set.
