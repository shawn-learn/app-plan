# ProcessWizard Library Extraction Analysis

## Goal and constraints

This analysis identifies relatively granular library boundaries that can be extracted from the current monorepo while minimizing cross-library coupling. The proposed boundaries preserve the existing architecture patterns already visible in the code:

- API layer delegates to repositories/services instead of embedding data logic.
- Service layer already has clear orchestration modules (completion, resolver, analytics).
- Plugin and registry patterns already decouple domain-specific behavior.
- Frontend already separates API client, auth context, route shell, and feature components.

## Current architectural layers observed

## Backend

1. **Transport/API layer (FastAPI routers)**
   - `process_wizard.main` initializes app and wires routers.
   - Routers in `process_wizard.api.*` perform request validation/auth checks and call services/repositories.

2. **Application services/orchestration**
   - Workflow orchestration in `services/completion.py`.
   - Data requirement resolution in `services/resolver.py`.
   - Analytics projection and reference indexing in dedicated services.

3. **Domain models (Pydantic)**
   - Models under `process_wizard.models.*` define contracts shared by API/repository/service code.

4. **Persistence layer**
   - SQLAlchemy table mappings in `db/tables.py`.
   - Data access functions in `db/repositories.py`.
   - Session wiring/seeding/migrations under `db/` + Alembic.

5. **Rules/decision engine layer**
   - GoRules wrapper in `engine/decisions.py`.
   - Evaluator and enricher registries under `services/evaluators` and `services/enrichers`.

6. **Plugin runtime**
   - Plugin discovery, module loading, and frontend component metadata in `plugin.py` + `plugins/`.

## Frontend

1. **Application shell/routing**
   - `App.tsx` handles route graph and auth protection.

2. **State/session contexts**
   - Auth and toast contexts in `contexts/`.

3. **Gateway/API client**
   - Shared HTTP client and typed endpoint helpers in `api/client.ts`.

4. **Feature slices**
   - Distinct page/component groups (`wizard`, `provider`, `groups`, `library`, `records`, `stylesheets`, etc.).

5. **Plugin UI extension layer**
   - Dynamic plugin components through `plugins/registry.ts`.

## Proposed library decomposition

The list below is ordered for practical extraction (top = easiest/lowest-risk first).

### 1) `pw-domain-contracts` (shared backend domain contracts)

**Scope**
- `backend/src/process_wizard/models/*`
- `backend/src/process_wizard/models/enums.py`

**Responsibility**
- Canonical domain types: documents, runs, users, workflows, references, enums.

**Why this is a good extraction**
- Minimal framework coupling (mostly Pydantic + typing).
- Centralizes schemas used across API/service/repository code.

**Likely dependencies**
- External: `pydantic`
- Internal: none

---

### 2) `pw-plugin-contracts` (plugin spec + registries)

**Scope**
- `backend/src/process_wizard/plugin.py`
- `backend/src/process_wizard/services/evaluators/__init__.py`
- `backend/src/process_wizard/services/enrichers/__init__.py`

**Responsibility**
- Plugin metadata contract (`PluginSpec`), discovery protocol, evaluator/enricher registration APIs.

**Why this is a good extraction**
- This is already a formal extension boundary.
- Decouples core runtime from domain/plugin packages.

**Likely dependencies**
- External: stdlib only
- Internal: may reference decision-engine types in evaluator function signatures

---

### 3) `pw-rules-engine` (decision execution abstraction)

**Scope**
- `backend/src/process_wizard/engine/decisions.py`

**Responsibility**
- Load/compile/execute decision tables from data dirs.
- Provide a stable API (`evaluate`, `tables`) to application services/evaluators.

**Why this is a good extraction**
- Isolates vendor-specific runtime (`zen`) behind a narrow wrapper.
- Makes it easier to swap/upgrade decision backends.

**Likely dependencies**
- External: `zen`
- Internal: optional plugin data-dir lookup hook

---

### 4) `pw-data-access` (database infrastructure + repositories)

**Scope**
- `backend/src/process_wizard/db/tables.py`
- `backend/src/process_wizard/db/repositories.py`
- `backend/src/process_wizard/db/session.py`
- maybe `backend/src/process_wizard/db/seed.py`

**Responsibility**
- ORM mappings, DB session lifecycle, persistence queries.

**Why this is a good extraction**
- Consolidates SQLAlchemy/DB concerns into a single package.
- Gives services a focused dependency on repository interfaces.

**Likely dependencies**
- External: `sqlalchemy`
- Internal: `pw-domain-contracts`

---

### 5) `pw-run-pipeline` (application services for run lifecycle)

**Scope**
- `backend/src/process_wizard/services/completion.py`
- `backend/src/process_wizard/services/resolver.py`
- `backend/src/process_wizard/services/reference_writer.py`
- `backend/src/process_wizard/services/analytics.py`

**Responsibility**
- Orchestration logic for run completion, references, analytics projections, data resolution.

**Why this is a good extraction**
- High-value business logic, currently cohesive and service-oriented.
- Can depend on abstractions from data-access and rules-engine libraries.

**Likely dependencies**
- Internal: `pw-domain-contracts`, `pw-data-access`, `pw-rules-engine`, `pw-plugin-contracts`

---

### 6) `pw-api-runtime` (HTTP app adapter)

**Scope**
- `backend/src/process_wizard/main.py`
- `backend/src/process_wizard/api/*`

**Responsibility**
- FastAPI transport, auth dependency wiring, request/response translation.

**Why this is a good extraction**
- Keeps transport concerns separated from domain logic.
- Enables alternate transports in future (e.g., gRPC, GraphQL) with same service libraries.

**Likely dependencies**
- External: `fastapi`
- Internal: all core backend libraries above

---

### 7) `pw-web-api-client` (frontend typed gateway)

**Scope**
- `frontend/src/api/client.ts`
- likely shared request/response types from `frontend/src/types`

**Responsibility**
- Typed HTTP calls, auth token handling, retry/error policies.

**Why this is a good extraction**
- Reusable by web app, admin app, future mobile web wrappers.
- Already centralizes API interactions.

**Likely dependencies**
- External: browser fetch runtime
- Internal: shared frontend type package

---

### 8) `pw-web-auth-session` (frontend auth state package)

**Scope**
- `frontend/src/contexts/AuthContext.tsx`

**Responsibility**
- Auth state and login/signup/logout/session bootstrap behavior.

**Why this is a good extraction**
- Narrow and cohesive.
- Avoids duplicating auth flow logic in multiple apps.

**Likely dependencies**
- External: React
- Internal: `pw-web-api-client`

---

### 9) `pw-web-wizard-runtime` (frontend wizard execution UX)

**Scope**
- `frontend/src/components/wizard/*`
- `frontend/src/hooks/useStylesheet.ts`

**Responsibility**
- Wizard lifecycle UX: bundle load, run create/resume, autosave, completion/cancel, Form.io integration.

**Why this is a good extraction**
- High-value isolated workflow with clear inputs/outputs.
- Could be embedded in multiple host apps.

**Likely dependencies**
- External: React, Form.io
- Internal: `pw-web-api-client`, `pw-web-auth-session` (or host-provided auth)

---

### 10) `pw-web-plugin-ui` (frontend plugin component registry)

**Scope**
- `frontend/src/plugins/registry.ts`
- plugin component bundles under `frontend/src/plugins/*`

**Responsibility**
- Runtime mapping of plugin names/domains to lazy React components.

**Why this is a good extraction**
- Mirrors backend plugin extension model.
- Allows domain bundles to evolve independently.

**Likely dependencies**
- External: React lazy loading
- Internal: shared plugin naming contracts

## Recommended dependency direction

Use a one-way dependency rule to prevent coupling drift:

- `pw-domain-contracts` ← depended on by everything, depends on almost nothing.
- `pw-plugin-contracts` and `pw-rules-engine` depend only on contracts.
- `pw-data-access` depends on contracts.
- `pw-run-pipeline` depends on contracts + data-access + rules-engine + plugin-contracts.
- `pw-api-runtime` depends on all backend libraries, but no backend library depends on it.
- Frontend: `pw-web-api-client` at the base, then auth/wizard/plugin packages on top, app shell last.

## Granularity check (against your objective)

The proposed libraries are intentionally **granular** (10 units) but still map to **cohesive concern boundaries**:

- Each package owns a clear axis of change (API transport, DB, rules engine, run orchestration, auth UX, wizard UX, plugin UI).
- Cross-library knowledge is mostly contract-level (Pydantic/TS types + repository/service interfaces), not implementation-level.
- Plugin contracts on backend/frontend create explicit extension seams.

## Risks and mitigations

1. **Large `repositories.py` as a coupling hotspot**
   - Risk: one mega-module causes many transitive dependencies.
   - Mitigation: split into domain repositories (`documents_repo`, `runs_repo`, `users_repo`, etc.) before extraction.

2. **Shared type drift between backend and frontend**
   - Risk: duplicate model semantics diverge.
   - Mitigation: adopt generated OpenAPI client/types or a schema-sync pipeline.

3. **Plugin initialization side effects**
   - Risk: discovery order and import-time registration create hidden coupling.
   - Mitigation: formalize plugin bootstrap lifecycle (`discover -> validate -> register -> activate`).

4. **Current app router imports many pages directly**
   - Risk: hard to move feature bundles independently.
   - Mitigation: introduce route manifest per feature package and mount centrally.

## Suggested extraction sequence (practical)

1. Extract `pw-domain-contracts`.
2. Extract `pw-plugin-contracts` and `pw-rules-engine`.
3. Refactor/split repositories, then extract `pw-data-access`.
4. Extract `pw-run-pipeline`.
5. Extract `pw-api-runtime` as thin adapter.
6. On frontend: extract `pw-web-api-client`, then `pw-web-auth-session`, then `pw-web-wizard-runtime`, then `pw-web-plugin-ui`.

This order minimizes breakage because each step creates a cleaner seam for the next one.
