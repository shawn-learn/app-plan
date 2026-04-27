# Unified Platform

A merged library set powering three applications from one shared core:

1. **app-industrial** — multi-tenant compliance/SOP procedure execution (the true Industrial Process Wizard).
2. **app-organizer** — personal organizer (tasks & lists, calendar/scheduling, habits, personal goals).
3. **app-coach** — fitness coaching with team functionality (intake, assessments, periodized plans, session logging, athlete↔coach teams).

The full design is in [`../please-review-the-two-hashed-plum.md`](../../.claude/plans/please-review-the-two-hashed-plum.md) (the approved plan) and is summarized in [`docs/architecture.md`](docs/architecture.md).

## Layout

```
unified-platform/
├── packages/
│   ├── py/      # Python libraries (FastAPI/Pydantic/SQLAlchemy)
│   └── ts/      # TypeScript libraries (React/Vite/MUI)
├── apps/        # Thin app shells composing platform + one domain plugin
├── docs/        # Architecture + per-library specs + ADRs
├── scripts/     # Codegen, schema-sync, db build
└── tooling/     # CI templates, lint, format
```

## Architecture in one sentence

A **platform core** (auth, tenancy, users, teams, wizard runtime, rules engine, run pipeline, signatures/audit, scheduling, calendar, goals, content, sync, data access, API runtime, web UI kit) is consumed by three **domain plugins** (`domain-industrial`, `domain-organizer`, `domain-fitness`), each composed by a thin **app shell**.

**Dependency rule:** dependencies point downward only. Platform never imports a domain. Domains never import each other.

## Workspace

- **pnpm workspaces** for TS packages
- **uv workspaces** for Python packages
- Designed **polyrepo-ready** — any package can be extracted into its own git repository without source changes (see `docs/architecture.md` §6.1).
