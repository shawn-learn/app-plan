# 01 — Platform Architecture

## Purpose

Define the unified architecture that merges Organizer and Process Wizard into one reusable platform with domain plugins and thin app shells.

## Layers

- **App Shells**: `app-industrial`, `app-organizer`, `app-coach`
- **Domain Plugins**: `domain-industrial`, `domain-organizer`, `domain-fitness` (Python + TypeScript twins)
- **Platform Core**:
  - Tier 0: contracts, schemas, shared units
  - Tier 1: auth, tenancy, users, access, teams
  - Tier 2: runtime services (rules, plugin contracts, run pipeline, signatures/audit, templates, scheduling, calendar, goals, content, messaging, sync, data access)
  - Tier 3: transport + frontend platform (API runtime, API client, auth session, UI kit, wizard runtime, plugin UI, form editors, calendar UI, shared features)

## Dependency Direction

1. Dependencies point downward only.
2. Platform libraries never import domain libraries.
3. Domain libraries never import each other.
4. App shells compose platform + selected domains.

## Canonical Library Catalog

### Tier 0
- `core-domain-contracts`
- `core-units` (Python + TypeScript)
- `core-schemas-ts`

### Tier 1
- `core-auth`
- `core-tenancy`
- `core-users`
- `core-access`
- `core-teams`

### Tier 2
- `core-rules-engine`
- `core-plugin-contracts`
- `core-run-pipeline`
- `core-signatures-audit`
- `core-templates`
- `core-scheduling`
- `core-calendar`
- `core-goals`
- `core-content`
- `core-messaging`
- `core-sync`
- `core-data-access`

### Tier 3
- `core-api-runtime`
- `web-api-client`
- `web-auth-session`
- `web-ui-kit`
- `web-wizard-runtime`
- `web-plugin-ui`
- `web-form-editors`
- `web-calendar`
- `web-features-shared`

## Architectural Decisions

- Wizard run execution is the platform primitive for all domains.
- Scheduling is generic and reused across industrial procedures, organizer planning, and fitness sessions.
- Teams are built from access groups plus memberships.
- Signatures and audit are core capabilities, opt-in by app/domain.
- Fitness-specific models are plugin-owned, not platform-owned.
- Cross-language contracts originate from JSON Schema and generate TS types.
