# ProcessWizard Library Overview

## Product Description

ProcessWizard is a fitness coaching platform that executes structured documents — intake forms, clinical assessments, workout logs, and training plans — as interactive wizard sessions. It is built around a **reference pipeline**: when a session completes, its computed outputs are indexed in a reference store so that later documents can resolve those values automatically. A client's intake risk classification, volume modifier, and intensity ceiling determined at onboarding are available to every subsequent workout template without any manual copy-paste.

### What the platform does today

**Client intake and risk stratification.** The fitness intake wizard (26 steps, Form.io schema) collects PAR-Q responses, medical history, medications, pain severity, training age, sleep quality, and scheduling availability. On completion the GoRules decision engine evaluates risk stratification and medication implications; a Python evaluator computes volume modifier, intensity ceiling, pain-adjusted training load, and availability summary. All values are indexed in the reference store.

**Clinical assessments.** Four additional wizard documents cover metabolic threshold testing (4 phases, safety gating), musculature/strength assessment (5 phases, 1-RM progression), body composition (5selectable methods: BIA, calipers, DEXA, Navy, screening), and running program planning. Each has its own GoRules decision tables and evaluators; results feed the reference store.

**Running program and session logging.** The fitness plugin includes a running program evaluator and Form.io schemas for session logging (`running_session_log.json`), weekly summaries, and next-week plan generation. Training-load aggregates (ACWR, zone distribution, recovery status) are computed from the `run_analytics` table using rolling-window queries.

**Provider dashboard and roster management.** Practitioners can invite clients, view the full client roster, assign documents to clients, track run status, and read per-client reference data on a profile page.

**Form builder.** A drag-and-drop Form.io builder (`@process-wizard/form-editors`) lets practitioners author new questionnaires and assessments. The builder supports custom fields — unit-aware numeric inputs, ranking sliders, weekly availability grids, and range pickers — and exports Form.io JSON schemas.

**Workflow chaining.** Workflows link documents together in sequence with prerequisite gates; the run pipeline resolves data requirements from the reference store before launching each step.

**Multi-tenancy groundwork.** The system-requirements document specifies per-tenant isolated databases, JWT/refresh-token auth, TOTP MFA, role-based access, audit logging, and tenant onboarding via operator APIs. These are designed but not fully implemented.

---

## Current Architecture

### Backend layers

| Layer | Modules | Notes |
|-------|---------|-------|
| Transport | `main.py`, `api/*.py` (14 routers) | FastAPI, auth dependency injection |
| Application services | `services/completion.py`, `resolver.py`, `reference_writer.py`, `analytics.py` | Run lifecycle orchestration |
| Domain models | `models/*.py` (10 Pydantic v2 modules) | Shared contracts: documents, runs, users, workflows, etc. |
| Rules/decision engine | `engine/decisions.py` | GoRules Zen Engine wrapper, auto-discovers JDM JSON from plugin data dirs |
| Plugin runtime | `plugin.py`, `services/evaluators/__init__.py`, `services/enrichers/__init__.py` | Decorator-based registration, entry-point discovery |
| Persistence | `db/tables.py`, `db/repositories.py`, `db/session.py` | SQLAlchemy 2.0 async; SQLite dev / PostgreSQL prod |

### Frontend layers

| Layer | Modules | Notes |
|-------|---------|-------|
| App shell | `App.tsx` | React Router 7, auth-protected routes (16 routes) |
| Auth session | `contexts/AuthContext.tsx` | Token storage, refresh interceptor, login/logout state |
| API gateway | `api/client.ts` (~700 lines) | Typed fetch wrappers, retry logic, 50+ endpoints |
| Feature components | `components/wizard/`, `components/provider/`, `components/builder/`, etc. | Wizard lifecycle, completion, profile, roster, stylesheets, workflows |
| Plugin UI | `plugins/registry.ts`, `plugins/fitness/` | Lazy-loaded domain components (FitnessIntakeSummary, FitnessProfile, etc.) |

### Existing extracted package

`@process-wizard/form-editors` lives in `packages/form-editors/` and is consumed by the frontend as a workspace dependency. It is already fully decoupled — all host concerns are injected via adapters.

---

## Why Libraries

The monorepo works for a single team building one product. Libraries become necessary when:

- **Multiple applications** need the same logic (a mobile app, a separate admin console, or a third-party that needs the wizard runtime).
- **Independent versioning** is needed — the rules engine or domain contracts can be bumped without touching the UI.
- **Clean extension seams** — the fitness plugin today is embedded in the app; extracting the foundational contracts lets future sport-specific or domain-specific plugins be developed and distributed as separate packages.
- **New capabilities** (workout authoring, FIT file ingestion) are cleanly new surface area that should not accumulate inside the app.

---

## Library Inventory

| # | Package | Language | Category | Status | Phase | Description |
|---|---------|----------|----------|--------|-------|-------------|
| 1 | `@process-wizard/form-editors` | TypeScript | Frontend | Exists | — | Form.io drag-and-drop builder + custom fields |
| 2 | `pw-domain-contracts` | Python | Backend | Extract | A | Canonical Pydantic types for all domain entities |
| 3 | `pw-rules-engine` | Python | Backend | Extract | A | GoRules Zen Engine wrapper for decision tables |
| 4 | `pw-plugin-contracts` | Python | Backend | Extract | A | PluginSpec, evaluator/enricher registries |
| 5 | `pw-fit-parser` | Python | Backend | New | A | Parse Garmin .fit files into normalized activity records |
| 6 | `pw-training-metrics` | Python | Backend | New | A | Sport-agnostic training-load math (ACWR, TSS, zones) |
| 7 | `@process-wizard/exercise-catalog` | TypeScript | Frontend | New | B | Canonical exercise + movement catalog with typed accessors |
| 8 | `@process-wizard/workout-editor` | TypeScript | Frontend | New | B | Bespoke block-based workout plan authoring UI |
| 9 | `@process-wizard/workout-runtime` | TypeScript | Frontend | New | C | Renders workout plans as logger or live-guided session |
| 10 | `pw-data-access` | Python | Backend | Extract | Deferred | SQLAlchemy ORM + repositories abstraction |
| 11 | `pw-run-pipeline` | Python | Backend | Extract | Deferred | Run completion orchestration, resolver, reference writer |
| 12 | `pw-api-runtime` | Python | Backend | Extract | Deferred | FastAPI transport adapter |
| 13 | `@process-wizard/web-api-client` | TypeScript | Frontend | Extract | Deferred | Typed HTTP gateway for all API domains |
| 14 | `@process-wizard/web-auth-session` | TypeScript | Frontend | Extract | Deferred | Auth state, token lifecycle, login/logout context |
| 15 | `@process-wizard/web-wizard-runtime` | TypeScript | Frontend | Extract | Deferred | Form.io wizard lifecycle UX component |
| 16 | `@process-wizard/web-plugin-ui` | TypeScript | Frontend | Extract | Deferred | Plugin component registry + lazy-loading |

> **Not a library:** `plugins/fitness/` stays in the app. It is a domain plugin — a bundle of evaluators, enrichers, decision tables, and frontend components that is deployed with the platform, not distributed separately.

---

## Dependency Graph

One-way dependency rule: lower-numbered tiers may not import from higher-numbered tiers.

```
Tier 0 (no internal deps)
  pw-domain-contracts
  @process-wizard/exercise-catalog

Tier 1 (depend on Tier 0 only)
  pw-rules-engine          → (no internal deps, wraps zen only)
  pw-plugin-contracts      → pw-domain-contracts
  pw-fit-parser            → (no internal deps, pure conversion)
  pw-training-metrics      → (no internal deps, pure functions)
  @process-wizard/form-editors → (no internal deps, uses adapters)

Tier 2 (depend on Tier 0-1)
  pw-data-access           → pw-domain-contracts
  @process-wizard/workout-editor → @process-wizard/exercise-catalog

Tier 3 (depend on Tier 0-2)
  pw-run-pipeline          → pw-domain-contracts, pw-data-access,
                             pw-rules-engine, pw-plugin-contracts
  @process-wizard/workout-runtime → @process-wizard/workout-editor,
                             @process-wizard/exercise-catalog

Tier 4 (depend on Tier 0-3)
  pw-api-runtime           → all backend Tier 0-3 libraries
  @process-wizard/web-api-client → (no internal deps, thin HTTP wrapper)

Tier 5 (app-level, not extracted yet)
  @process-wizard/web-auth-session → @process-wizard/web-api-client
  @process-wizard/web-wizard-runtime → @process-wizard/web-api-client
  @process-wizard/web-plugin-ui → (plugin naming contracts only)

App shell
  frontend app             → all frontend libraries above
  backend app              → all backend libraries above (or direct modules until extracted)
```

---

## Phased Delivery

### Phase A — Foundations + FIT ingestion

Extract the three foundational backend libraries so new fitness work has clean contracts from day one. Add FIT parsing and generalize training-load math as standalone libraries.

1. Extract `pw-domain-contracts` from `models/*`
2. Extract `pw-rules-engine` from `engine/decisions.py`
3. Extract `pw-plugin-contracts` from `plugin.py` + evaluator/enricher registries
4. Create `pw-fit-parser` — parse Garmin `.fit` files into normalized activity records; wire `/api/fit/upload` endpoint
5. Create `pw-training-metrics` — lift `services/training_load.py`, generalize across HR/power/pace, remove DB dependency

**Exit criteria:** existing app still builds and all pytest tests pass; FIT upload endpoint produces a session record with computed metrics for at least one running activity.

### Phase B — Workout authoring

Deliver the exercise catalog and the bespoke workout editor. Strength-focused content first; endurance blocks added in Phase B.2.

6. Create `@process-wizard/exercise-catalog` with a strength seed dataset (≥50 lifts)
7. Create `@process-wizard/workout-editor` — block-based UI (sets/reps/load, RPE, supersets, rest timers for strength; HR/pace zone × duration for endurance)

**Exit criteria:** a practitioner can author a strength plan, save as JSON, reload it, and confirm the round-trip. Editor is independent of Form.io.

### Phase C — Session execution

8. Create `@process-wizard/workout-runtime` in *logger* mode — post-hoc entry view: set ticks, weights, RPE for strength; metrics review + optional FIT upload for endurance
9. Add *live* mode — interval/rest timers, set-by-set guidance (biometric streaming is out of scope in v1)

**Exit criteria:** a session can be logged end-to-end and produces a completed run that the reference pipeline indexes.

### Deferred (future phases)

`pw-data-access`, `pw-run-pipeline`, `pw-api-runtime`, `@process-wizard/web-api-client`, `@process-wizard/web-auth-session`, `@process-wizard/web-wizard-runtime`, `@process-wizard/web-plugin-ui`.

None of these block Phase A-C work. Each is a large extraction with meaningful risk; revisit once the new libraries ship and the seams prove stable.

---

## Open Questions

| # | Question | Default for v1 |
|---|----------|---------------|
| 1 | **Time-series storage:** keep activity samples in JSONB, push to a dedicated timeseries table, or keep file-backed (parse on demand)? | JSONB for v1; revisit at scale |
| 2 | **Live-mode biometrics:** HR/power broadcast over Bluetooth in v1, or guided UI only without real-time biometrics? | Guided UI only; biometric streaming is a later phase |
| 3 | **Cross-language type sharing:** `WorkoutPlan` shape lives in TS (editor/runtime) and Python (evaluators). Hand-keep parallel definitions, or generate from a single source (JSON Schema → both)? | Hand-keep with a contract test that round-trips a fixture |
| 4 | **Catalog ownership:** ship exercise-catalog seed data as JSON bundled in the package, or as a seed loaded into the host DB? | Package-bundled JSON; host can override/extend via injection hook |

---

## Per-library Documents

See `docs/libraries/` for detailed requirements for each library:

- [form-editors.md](libraries/form-editors.md)
- [pw-domain-contracts.md](libraries/pw-domain-contracts.md)
- [pw-rules-engine.md](libraries/pw-rules-engine.md)
- [pw-plugin-contracts.md](libraries/pw-plugin-contracts.md)
- [pw-fit-parser.md](libraries/pw-fit-parser.md)
- [pw-training-metrics.md](libraries/pw-training-metrics.md)
- [exercise-catalog.md](libraries/exercise-catalog.md)
- [workout-editor.md](libraries/workout-editor.md)
- [workout-runtime.md](libraries/workout-runtime.md)
- [pw-data-access.md](libraries/pw-data-access.md)
- [pw-run-pipeline.md](libraries/pw-run-pipeline.md)
- [pw-api-runtime.md](libraries/pw-api-runtime.md)
- [web-api-client.md](libraries/web-api-client.md)
- [web-auth-session.md](libraries/web-auth-session.md)
- [web-wizard-runtime.md](libraries/web-wizard-runtime.md)
- [web-plugin-ui.md](libraries/web-plugin-ui.md)
