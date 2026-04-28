# web-features-shared

**Tier 3 — Frontend platform.**

Generic feature components reused across all three apps: settings, profile, content viewer, messaging shell, teams admin, goals/milestones.

## Source material

Cross-domain bits of `Organizer/frontend-features.md`.

## Responsibilities

- **Settings & profile:** user preferences UI (`core-users`).
- **Content viewer:** in-app help drawer, articles (`core-content`).
- **Teams admin:** create team, manage members, manage shares (`core-teams` + `core-access`).
- **Goals & milestones:** generic milestone CRUD (`core-goals`).
- **Messaging shell:** chat list / thread UI gated by feature flag (`core-messaging`).
- **Audit & signature views:** generic verification UIs for runs that have signatures (`core-signatures-audit`).

## Dependencies

- External: `react`, `react-router-dom`
- Internal: `web-ui-kit`, `web-api-client`, `web-auth-session`, `core-schemas-ts`, `core-units`

## Consumers

Every app shell.

## Notes

Domain-specific feature components do *not* live here — they live in `domain-*` TS packages.
