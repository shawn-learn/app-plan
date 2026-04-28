# core-users

**Tier 1 — Identity.**

User profiles, preferences, parameter-visibility rules, interest tags.

## Source material

`Organizer/user-service.md`.

## Responsibilities

- User entity beyond auth credentials (display name, locale, avatar, timezone).
- Per-user preferences: unit system, notification settings, UI preferences.
- Parameter-visibility rules (mutually-exclusive visibility groups, log groups) — generic mechanism originally for exercise parameter UX, applicable to any domain wanting to hide/show input fields per user role/preference.
- Interest tags.

## Dependencies

- External: `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-auth`, `core-units`, `core-data-access`

## Consumers

Every Tier 2 service that personalizes UX. `web-features-shared` mounts the settings/profile feature.
