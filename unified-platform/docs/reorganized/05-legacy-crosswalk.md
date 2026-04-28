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
- `architecture_review_feedback.md` → resolved/absorbed into architecture and delivery guidance.
- `Organizer/OVERVIEW.md` → package catalog and decomposition captured in `01-platform-architecture.md`.
- `Process Wizard Docs/library-overview.md` → platform/package decomposition captured in `01-platform-architecture.md`.

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
