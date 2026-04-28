# 05 — Legacy Document Crosswalk

This document tracks where legacy content moved in the reorganized documentation set.

## Migration Policy

- A legacy document is marked `DELETE` only after key requirements are captured in the reorganized docs.
- Legacy documents without `DELETE` are intentionally still active until their content is migrated.

## Fully Migrated (Marked `DELETE`)

- `plan.md` → architecture, roadmap, packaging, and migration guidance now consolidated in:
  - `01-platform-architecture.md`
  - `03-delivery-roadmap.md`
  - `04-repo-and-packaging-standards.md`
  - `07-operational-and-delivery-details.md`
- `architecture_review_feedback.md` → architecture guardrails, quality gates, and operational constraints absorbed into:
  - `01-platform-architecture.md`
  - `03-delivery-roadmap.md`
  - `07-operational-and-delivery-details.md`
- `Organizer/OVERVIEW.md` → package catalog and decomposition captured in:
  - `01-platform-architecture.md`
  - `04-repo-and-packaging-standards.md`
  - `06-functional-requirements.md`
- `Process Wizard Docs/library-overview.md` → platform/package decomposition and fitness runtime details captured in:
  - `01-platform-architecture.md`
  - `03-delivery-roadmap.md`
  - `06-functional-requirements.md`

## Delete-Document Coverage Checklist

The following checklist records the still-valid items from each `DELETE` document and where they now live.

### `plan.md`

- Three-layer model (app shells, domain plugins, platform tiers) → `01-platform-architecture.md`.
- Downward-only dependency rule and domain isolation → `01-platform-architecture.md`.
- Canonical package catalog (core + web + domain packages) → `01-platform-architecture.md`.
- Core reuse decisions (wizard runtime as primitive, generic scheduling, teams/access split, signatures in core, fitness as plugin, cross-language contracts) → `01-platform-architecture.md`.
- Phased implementation sequence and phase exit intent → `03-delivery-roadmap.md`.
- Polyrepo-ready standards and workspace tooling choices (`uv`, `pnpm`) → `04-repo-and-packaging-standards.md`.

### `architecture_review_feedback.md`

- ADR-first requirement for hard-to-reverse decisions (tenancy, eventing, contract evolution, trust model) → `07-operational-and-delivery-details.md`.
- Domain leakage guards beyond import direction (API surface checks + package dependency matrix) → `01-platform-architecture.md`, `03-delivery-roadmap.md`.
- Event-first integration backbone and idempotent outbox expectations → `07-operational-and-delivery-details.md`.
- Plugin contract compatibility and runtime version negotiation requirements → `07-operational-and-delivery-details.md`.
- Early quality gates and observability baselines in delivery sequencing → `03-delivery-roadmap.md`, `07-operational-and-delivery-details.md`.
- Security/compliance readiness expectations (signature verification, tenant boundaries, retention/deletion, key rotation) → `07-operational-and-delivery-details.md`.

### `Organizer/OVERVIEW.md`

- Organizer feature inventory that remains in scope (templates, planning/logging, metrics, teams/sharing, goals, calendar, content, auth, offline sync, messaging seam) → `06-functional-requirements.md`.
- Existing stack choices and backend/frontend technology assumptions retained in new architecture docs → `04-repo-and-packaging-standards.md`, `07-operational-and-delivery-details.md`.
- Decomposition intent (backend service boundaries + shared UI contracts) retained as generalized platform package model → `01-platform-architecture.md`.

### `Process Wizard Docs/library-overview.md`

- Reference pipeline and reusable wizard execution model → `01-platform-architecture.md`, `06-functional-requirements.md`.
- Fitness domain scope that remains valid (intake/assessment flows, workout authoring/runtime split, FIT ingestion, training metrics) → `06-functional-requirements.md`, `03-delivery-roadmap.md`.
- Plugin runtime and form-builder role in platform composition → `01-platform-architecture.md`, `03-delivery-roadmap.md`.
- Multi-tenant/auth/audit/security fundamentals retained as cross-domain requirements → `06-functional-requirements.md`, `07-operational-and-delivery-details.md`.

## Partially Migrated (Remain Active)

### Organizer feature/requirements docs
- `Organizer/docs/ACTIVITY_SCHEMAS.md`
- `Organizer/docs/TEMPLATE_FORMAT.md`
- `Organizer/docs/OFFLINE_SYNC.md`
- `Organizer/docs/AUTH_FLOW.md`
- `Organizer/docs/UI_STANDARDS.md`
- `Organizer/docs/USER_MANAGEMENT_REQUIREMENTS.md`
- `Organizer/docs/FEATURES_OVERVIEW.md`
- and related service-specific docs under `Organizer/*.md`

These are now summarized in `06-functional-requirements.md` and `07-operational-and-delivery-details.md`, but remain active until section-level migration is completed.

### Process Wizard requirements docs
- `Process Wizard Docs/system-requirements.md`
- `Process Wizard Docs/user-management-service-requirements.md`
- `Process Wizard Docs/ui-standards.md`
- `Process Wizard Docs/user_stories.md`
- `Process Wizard Docs/implementation_guide_v2.md`
- and `Process Wizard Docs/libraries/*.md`

These are now summarized in `01`, `06`, and `07`, but remain active until section-level migration is completed.

## Next Migration Steps

1. Convert legacy library docs into canonical per-library docs under `unified-platform/docs/per-library/`.
2. Migrate detailed endpoint/spec tables from user-management and auth docs.
3. Migrate full step catalogs and acceptance criteria from Process Wizard requirements/user stories.
4. Mark remaining legacy docs `DELETE` only after per-section parity review.
