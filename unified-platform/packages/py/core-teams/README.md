# core-teams

**Tier 1 — Identity.**

Named teams, memberships, team-scoped tags. A thin specialization on top of `core-access`.

## Source material

`Organizer/team-service.md` (Teams, memberships, team tags).

## Responsibilities

- Team entity (name, description, owner, optional avatar).
- Memberships with roles delegated to `core-access`.
- Team-scoped tags (used by Organizer to classify shared templates/exercises; reusable for any domain).
- Team invitation flow.

## Dependencies

- External: `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-users`, `core-access`, `core-data-access`

## Consumers

`domain-fitness` (coach↔athlete teams), `app-coach`, share workflows in `core-templates`, `web-features-shared` (teams admin UI).
