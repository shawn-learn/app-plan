# 03 — Delivery Roadmap

## Phase 0 — Foundation Contracts
- Establish workspace and package skeletons
- Implement contract generation and round-trip tests
- Deliver `core-domain-contracts`, `core-units`, `core-schemas-ts`

## Phase 1 — Identity and Tenant Context
- Deliver auth, tenancy, users, access, teams
- Boot app shells with authentication stubs

## Phase 2 — Runtime Core
- Deliver rules engine, plugin contracts, template storage, run pipeline, signatures/audit, data access, API runtime
- Validate end-to-end stub wizard run and canonical record creation

## Phase 3 — Frontend Runtime
- Deliver web API client, auth session, UI kit, wizard runtime, plugin UI, form editors
- Validate browser-based wizard execution

## Phase 4 — Horizontal Capabilities
- Deliver scheduling, calendar, goals, content, sync, shared feature shells
- Keep messaging as phase-0 stub unless explicitly prioritized

## Phase 5 — Industrial App
- Build `domain-industrial` + `app-industrial`

## Phase 6 — Organizer App
- Build `domain-organizer` + `app-organizer`

## Phase 7 — Coach App
- Build `domain-fitness` + `app-coach`

## Verification Strategy
- Per-package unit + public API tests
- Cross-language contract fixture tests
- Per-app smoke tests for login + wizard run
- Isolation tests to prevent undeclared domain linking
- Dependency-direction linting
