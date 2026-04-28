DELETE

# Unified Platform — Library Merge Plan

> Draft plan. Open questions at the bottom — please review the **Open Questions** section before approving.

## 1. Context

Two adjacent projects exist as documentation-only design sets in `C:\Users\shawn\New Projects\`:

- **Organizer** (`Organizer/`) — a workout/training/rehab platform (FastAPI + React/MUI + TimescaleDB) with rich activity templates, planned activities, exercise catalog, calendar/recurrence, goals/badges, teams, sharing, sync, articles, and a Phase-0 messaging stub. Already proposed as **15 libraries** across 3 tiers (`shared-schemas`, `shared-units`, `auth-service`, `user-service`, `exercise-service`, `activity-service`, `template-service`, `calendar-service`, `goals-service`, `team-service`, `messaging-service`, `sync-service`, `content-service`, `frontend-ui-kit`, `frontend-features`).
- **Process Wizard** (`Process Wizard Docs/`) — generically described in `system-requirements.md` as a **multi-tenant compliance/procedure-execution platform** (tenants, procedures, runs, canonical records, digital signatures, audit log, access groups, branching steps, offline sync, localization). It executes Form.io documents through a wizard runtime with a GoRules decision engine and a plugin model; the *current* plugin is fitness (intake → assessments → workout authoring/runtime → FIT parser → training metrics). Already proposed as **16 packages** in `library-overview.md`.

The user wants to **merge the two** into a single library set that powers **three target applications** with maximum code reuse:

1. **Industrial Process Wizard** — true industrial procedure execution (SOPs, compliance, audit signatures).
2. **Generalized Organizer** — personal organizer for individuals (calendar, tasks, goals, notes, habits).
3. **Personal Trainer / Coach** — fitness coaching with team functionality (intake, assessments, periodized plans, session logging, athlete↔coach teams).

The opportunity: Process Wizard's runtime is already the right abstraction for "executing structured documents and indexing their results." Organizer's domain (templates, planned activities, recurrence, calendar, goals, sync, teams, users, content) is what the Process Wizard *needs* to feel like a polished product and what the Organizer/Coach apps need wholesale. Recognizing the fitness vertical inside Process Wizard as **a domain plugin** (not the platform) is the unlock that lets all three apps share the same core.

## 2. Architecture — Three Layers

```
┌─────────────────────────────────────────────────────────────────┐
│ APP SHELLS  (thin; compose platform + one domain plugin)         │
│   app-industrial    app-organizer    app-coach                   │
└─────────────────────────────────────────────────────────────────┘
                              ▲
┌─────────────────────────────────────────────────────────────────┐
│ DOMAIN PLUGINS  (each: schemas, evaluators, decision tables,     │
│                       seed data, frontend feature components)    │
│   domain-industrial   domain-organizer   domain-fitness          │
└─────────────────────────────────────────────────────────────────┘
                              ▲
┌─────────────────────────────────────────────────────────────────┐
│ PLATFORM CORE  (domain-agnostic libraries)                       │
│   Tier 0 — contracts, schemas, units                             │
│   Tier 1 — auth, tenancy, users, teams, access-groups            │
│   Tier 2 — wizard runtime, rules engine, plugin contracts,       │
│            run pipeline / reference store, signatures & audit,   │
│            calendar/recurrence, templates, scheduler, sync,      │
│            content/articles, messaging                           │
│   Tier 3 — api-runtime (FastAPI), web-api-client, ui-kit,        │
│            web-auth-session, web-wizard-runtime, web-plugin-ui   │
└─────────────────────────────────────────────────────────────────┘
```

**Rule:** dependencies point *down only*. Platform never imports a domain. Domains never import each other. App shells import platform + exactly one (or more) domains.

## 3. Library Catalog (proposed)

### Tier 0 — Foundations (shared, no internal deps)

| # | Package | Lang | Source | Purpose |
|---|---------|------|--------|---------|
| 1 | `core-domain-contracts` | Py | PW `pw-domain-contracts` ⊕ Org `shared-schemas` | Pydantic types + JSON Schemas for runs, documents, references, users, tenants, plus reusable activity/measurement schemas |
| 2 | `core-units` | Py + TS | Org `shared-units` | Units, conversions, user-scoped preferences |
| 3 | `core-schemas-ts` | TS | generated from #1 | Cross-language types via JSON-Schema → TS codegen |

### Tier 1 — Identity & Tenancy

| # | Package | Lang | Source | Purpose |
|---|---------|------|--------|---------|
| 4 | `core-auth` | Py | PW auth + Org `auth-service` | OAuth2/JWT, refresh, MFA (TOTP), guest cookie sessions, password reset |
| 5 | `core-tenancy` | Py | PW tenancy reqs (REQ-TENANT-*) | Per-tenant DB routing, onboarding API, tenant registry |
| 6 | `core-users` | Py | Org `user-service` | Users, profiles, preferences, parameter-visibility rules |
| 7 | `core-access` | Py | PW access groups + Org `team-service` | Access groups, roles, permissions, share-grants. Teams modeled as a specialization of access-groups |
| 8 | `core-teams` | Py | Org `team-service` (membership/tags) | Team memberships and team-scoped tags (consumes `core-access`) |

### Tier 2 — Platform Services

| # | Package | Lang | Source | Purpose |
|---|---------|------|--------|---------|
| 9 | `core-rules-engine` | Py | PW `pw-rules-engine` | GoRules Zen wrapper, decision-table loading from plugin data dirs |
| 10 | `core-plugin-contracts` | Py | PW `pw-plugin-contracts` | `PluginSpec`, evaluator/enricher registries, discovery |
| 11 | `core-run-pipeline` | Py | PW `pw-run-pipeline` | Run lifecycle, resolver, reference-store writer, analytics projection |
| 12 | `core-signatures-audit` | Py | PW signature/audit reqs | Canonical-record signing (EdDSA/Ed25519), audit log append, signature verification |
| 13 | `core-templates` | Py | Org `template-service` | Versioned document/template storage, import/export (generalizes Org templates and PW Form.io documents) |
| 14 | `core-scheduling` | Py | Org `calendar-service` + `activity-service` (planning slice) | Planned-activities, RRULE recurrence, exception dates, ICS feed; *generic* "planned occurrence of a template" |
| 15 | `core-calendar` | Py | Org `calendar-service` (views slice) | Day/week/month/year views built on `core-scheduling` |
| 16 | `core-goals` | Py | Org `goals-service` | Generic milestones + badges (no fitness specifics) |
| 17 | `core-content` | Py | Org `content-service` | Articles, help drawer, system log |
| 18 | `core-messaging` | Py | Org `messaging-service` | DMs, group chats, broadcasts, attachments, E2EE seam (kept Phase-0 stub for now) |
| 19 | `core-sync` | Py | Org `sync-service` | Offline-first sync helpers, conflict resolution |
| 20 | `core-data-access` | Py | PW `pw-data-access` | SQLAlchemy 2.0 async ORM + repository base classes, per-tenant session routing |

### Tier 3 — Transport & Frontend Platform

| # | Package | Lang | Source | Purpose |
|---|---------|------|--------|---------|
| 21 | `core-api-runtime` | Py | PW `pw-api-runtime` | FastAPI app factory, router mounting, dependency wiring |
| 22 | `web-api-client` | TS | PW `web-api-client` | Typed fetch gateway, retry, refresh interceptor |
| 23 | `web-auth-session` | TS | PW `web-auth-session` | React auth context + token lifecycle |
| 24 | `web-ui-kit` | TS | Org `frontend-ui-kit` | Themable MUI-based primitives, form inputs, layout, charts wrappers, React Query setup |
| 25 | `web-wizard-runtime` | TS | PW `web-wizard-runtime` | Form.io wizard lifecycle UX, autosave, completion/cancel |
| 26 | `web-plugin-ui` | TS | PW `web-plugin-ui` | Lazy-loaded plugin component registry |
| 27 | `web-form-editors` | TS | PW `@process-wizard/form-editors` | Drag-and-drop Form.io builder + custom fields |
| 28 | `web-calendar` | TS | Org calendar feature | Calendar views consuming `core-scheduling` |
| 29 | `web-features-shared` | TS | Org `frontend-features` (cross-domain bits) | Generic feature components: settings, profile, content viewer, messaging shell, teams admin, goals/milestones |

### Domain Plugins (each is one Python package + one TS package)

| # | Package | Lang | Source | Purpose |
|---|---------|------|--------|---------|
| 30 | `domain-industrial` | Py + TS | new (driven by PW `system-requirements.md` §5.3 step catalog) | Industrial procedure step types (decision-branch, signature-capture, deviations, equipment-ref dropdowns), SOP authoring extensions, compliance audit views |
| 31 | `domain-organizer` | Py + TS | new (informed by Org but personal-scoped) | Personal **tasks & lists**, **calendar/scheduling glue**, **habits**, personal goals; reuses `core-scheduling`/`core-calendar`/`core-goals`. *Excludes* notes/journal (out of scope for v1). |
| 32 | `domain-fitness` | Py + TS | Org exercise/activity/goals fitness slice ⊕ PW fitness plugin (intake, assessments, FIT parser, training metrics, workout-editor, workout-runtime, exercise-catalog) | Exercise catalog, workout editor & runtime, FIT ingestion, training-load math, intake/assessment wizards, periodization, badges; uses `core-teams` for coach↔athlete |

### App Shells

| App | Composes | Notes |
|-----|----------|-------|
| `app-industrial` | platform + `domain-industrial` | Multi-tenant, signature/audit emphasis, offline procedure execution, no fitness/organizer code |
| `app-organizer` | platform + `domain-organizer` | Single-tenant or "personal tenant" mode, no signatures, light auth |
| `app-coach` | platform + `domain-fitness` (+ optional `domain-organizer` for clients' personal scheduling) | Team-centric, athlete portal + coach console, FIT upload, intake → plan → log loop |

## 4. Key Reuse Decisions

1. **Wizard runtime is the unifying primitive.** Both Org templates and PW procedures collapse to "versioned document executed as a run, producing a canonical record." `core-templates` + `core-run-pipeline` + `web-wizard-runtime` are shared by all three apps.
2. **Scheduling is generalized.** Org's `planned_activities` + RRULE machinery becomes `core-scheduling` — applies to procedure runs (industrial), tasks (organizer), and workout sessions (fitness).
3. **Teams = access groups + memberships.** PW's access-group model and Org's team model collapse to `core-access` (permissions) + `core-teams` (named groups & tags). Coach↔athlete and tenant-org-roles share the same primitive.
4. **Signatures & audit live in core**, not the industrial plugin — fitness can opt in for liability-sensitive rehab signoffs and the organizer can ignore them.
5. **Fitness is demoted to a plugin.** All exercise catalog, workout authoring/runtime, FIT parsing, and training-metrics code lives in `domain-fitness`. The platform has no `Exercise` model.
6. **Reference store is generic.** PW's reference pipeline (computed outputs indexed for downstream documents to consume) is broadly useful — Organizer's "1RM history feeds future templates" is the same pattern.
7. **One source of truth for cross-language types.** JSON Schema in `core-domain-contracts` → Pydantic on backend, generated TS in `core-schemas-ts`. Contract test round-trips a fixture (per PW `library-overview.md` Open Question #3 default).

## 5. Tech Stack (recommended unified)

| Concern | Choice |
|---------|--------|
| Backend | Python 3.12, FastAPI, Pydantic v2, SQLAlchemy 2.0 async, Alembic |
| Frontend | React + TypeScript, Vite, MUI, React Hook Form, React Query, Recharts |
| DB | PostgreSQL (TimescaleDB extension where time-series is needed); SQLite for local dev |
| Wizard | Form.io schemas + custom fields (from `web-form-editors`) |
| Rules | GoRules Zen Engine via `core-rules-engine` |
| Auth | OAuth2 + JWT (Ed25519), TOTP MFA, refresh-token rotation |
| Storage | S3-compatible (attachments, FIT files) |
| Workspace | **pnpm workspaces (TS) + `uv` workspaces (Python)**, designed polyrepo-ready (see §6.1) |

## 6. Repository Layout (proposed)

```
unified-platform/
├── packages/                       # all libraries
│   ├── py/
│   │   ├── core-domain-contracts/
│   │   ├── core-units/
│   │   ├── core-auth/
│   │   ├── core-tenancy/
│   │   ├── core-users/
│   │   ├── core-access/
│   │   ├── core-teams/
│   │   ├── core-rules-engine/
│   │   ├── core-plugin-contracts/
│   │   ├── core-run-pipeline/
│   │   ├── core-signatures-audit/
│   │   ├── core-templates/
│   │   ├── core-scheduling/
│   │   ├── core-calendar/
│   │   ├── core-goals/
│   │   ├── core-content/
│   │   ├── core-messaging/
│   │   ├── core-sync/
│   │   ├── core-data-access/
│   │   ├── core-api-runtime/
│   │   ├── domain-industrial/
│   │   ├── domain-organizer/
│   │   └── domain-fitness/
│   └── ts/
│       ├── core-schemas-ts/
│       ├── core-units/             (TS twin)
│       ├── web-api-client/
│       ├── web-auth-session/
│       ├── web-ui-kit/
│       ├── web-wizard-runtime/
│       ├── web-plugin-ui/
│       ├── web-form-editors/
│       ├── web-calendar/
│       ├── web-features-shared/
│       ├── domain-industrial/      (TS twin)
│       ├── domain-organizer/       (TS twin)
│       └── domain-fitness/         (TS twin)
├── apps/
│   ├── industrial/                 (FastAPI + Vite)
│   ├── organizer/
│   └── coach/
├── docs/
│   ├── architecture.md
│   ├── per-library/                (one file per package, like the existing per-library docs)
│   └── adr/                        (decision records)
├── scripts/                        (codegen, schema-sync, db build)
└── tooling/                        (CI templates, lint, format)
```

### 6.1 Polyrepo-ready discipline

The user wants the option to extract any library into its own git repository later. To make that cheap, every package follows these rules from day one:

1. **Self-contained package directory.** Each `packages/py/<name>/` and `packages/ts/<name>/` is a complete buildable unit: its own `pyproject.toml` / `package.json`, README, tests, CHANGELOG, and version. No reaching outside its directory for source.
2. **No relative cross-package imports.** Sibling packages are consumed by their published name (`core-domain-contracts`), never by relative path. Workspace tooling resolves this in dev; in a polyrepo world, the same import works against the published artifact.
3. **Pinned, declared dependencies.** Every internal dep lists a version range in the manifest — no implicit "everything in the workspace is current."
4. **Independent versioning.** SemVer per package. A changeset/release tool (e.g., `changesets` for TS, `towncrier` or hand-managed `CHANGELOG.md` for Py) ships per-package.
5. **Per-package CI matrix.** CI builds and tests each package in isolation against its declared deps, not just "the whole monorepo."
6. **Private registry seam.** Set up an internal registry config from the start (Verdaccio/GitHub Packages for TS, a simple index for Py) so the moment a package moves repos, consumers flip from workspace resolution to registry resolution with no source change.

A package leaving the monorepo becomes a `git filter-repo` extraction + flipping its consumers' lockfiles to the registry version — not a refactor.

## 7. Phased Delivery

- **Phase 0 — Skeleton & contracts.** Stand up monorepo, codegen pipeline, `core-domain-contracts`, `core-units`, `core-schemas-ts`. Cross-language fixture round-trip test green.
- **Phase 1 — Identity layer.** `core-auth`, `core-tenancy`, `core-users`, `core-access`, `core-teams`. App-shell stubs boot with login.
- **Phase 2 — Platform runtime.** `core-rules-engine`, `core-plugin-contracts`, `core-templates`, `core-run-pipeline`, `core-signatures-audit`, `core-data-access`, `core-api-runtime`. End-to-end: a stub plugin defines one document, runs end-to-end, produces a signed canonical record.
- **Phase 3 — Frontend platform.** `web-api-client`, `web-auth-session`, `web-ui-kit`, `web-wizard-runtime`, `web-plugin-ui`, `web-form-editors`. Empty app shell renders a wizard.
- **Phase 4 — Scheduling & supporting services.** `core-scheduling`, `core-calendar`, `core-goals`, `core-content`, `core-sync`, `web-calendar`, `web-features-shared`. (Messaging stays Phase-0 stub.)
- **Phase 5 — Industrial plugin & app.** `domain-industrial`, `app-industrial`. Step-field catalog from PW `system-requirements.md` §5.3.1 implemented.
- **Phase 6 — Organizer plugin & app.** `domain-organizer`, `app-organizer`.
- **Phase 7 — Fitness plugin & app.** `domain-fitness` (largest — absorbs exercise catalog, workout-editor/runtime, FIT parser, training-metrics, intake/assessments, periodization, badges), `app-coach`.

## 8. Verification

- Per-library: unit tests + a public-API contract test.
- Cross-language: JSON-Schema round-trip fixture for every shared contract type.
- Per-app smoke: boot app, log in, run one wizard end-to-end, see canonical record + reference-store entry.
- Plugin isolation test: build each app shell with **only** its declared domain plugin and assert no other domain symbols are linked.
- Dependency-direction lint: CI rule that platform packages cannot import `domain-*`.

## 9. Critical Files to Reference (existing documentation to mine)

- `Organizer/OVERVIEW.md` — full library catalog & dep graph (already aligned with this plan).
- `Organizer/docs/FEATURES_OVERVIEW.md`, `ACTIVITY_SCHEMAS.md`, `TEMPLATE_FORMAT.md`, `OFFLINE_SYNC.md`, `AUTH_FLOW.md`, `UI_STANDARDS.md`, `USER_MANAGEMENT_REQUIREMENTS.md`, `PLANNED_WORKOUT_EDITOR_SUPPORT.md`.
- `Process Wizard Docs/library-overview.md`, `library-extraction-analysis.md`, `system-requirements.md` (esp. §5.3.1 step catalog), `implementation_guide_v2.md`, `user-management-service-requirements.md`, `ui-standards.md`, `brand-guidelines.md`.
- Per-library specs already drafted in `Organizer/*.md` (15 files) and `Process Wizard Docs/libraries/*.md` (16 files) — each Tier 0–3 entry above will adopt and reconcile its two source documents.

## 10. Decisions Locked In

- **Industrial scope** = PW `system-requirements.md` as-written (multi-tenant compliance/SOP execution).
- **Organizer scope** = tasks & lists + calendar/scheduling + habits & personal goals. **No notes/journal** in v1.
- **Workspace** = pnpm + uv, polyrepo-ready (§6.1) — any library can be lifted into its own git repo later.
- **Coach tenancy** = single DB with org scoping. `core-tenancy` remains available for the industrial app and as a future flip for coach.

## 11. Remaining Defaults (call out if you want different)

- **Replacement vs. coexistence.** New `unified-platform/` lives alongside existing `Organizer/` and `Process Wizard Docs/`; originals stay as reference, archived after first app boots.
- **Messaging.** Keep as Phase-0 stub seam in `core-messaging`, no implementation in v1.
- **Localization.** Bake i18n into `web-ui-kit` from day one, ship English-only content.
- **Cross-language types.** Hand-keep Pydantic + generated TS, with a contract test that round-trips a fixture per shared type (per PW Open Question #3 default).