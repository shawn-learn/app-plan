# core-access

**Tier 1 — Identity.**

Access groups, roles, permissions, share-grants. The single permission primitive across all apps.

## Source material

Merges:
- Process Wizard `system-requirements.md` §5.1 (access groups: view, execute, edit; per-procedure group grants)
- Organizer team-share tables (template/exercise/exercise-template share grants)

## Responsibilities

- Access groups (named, reusable, multi-membership).
- Role definitions per group (view, execute, edit, admin — extensible).
- Permission checks: `can(user, action, resource) → bool`.
- Per-resource share grants (an entity can be shared with a user or a group with a specific role).
- Audit hooks for membership changes.

## Dependencies

- External: `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-users`, `core-data-access`

## Consumers

`core-teams` (teams are access groups + tags), every service that gates resource visibility, the industrial app's procedure-edit-rights flows.
