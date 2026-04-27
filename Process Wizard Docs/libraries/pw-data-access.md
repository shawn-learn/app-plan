# `pw-data-access`

## Purpose

Consolidates all SQLAlchemy ORM table definitions, database session lifecycle, and repository (CRUD) functions into a single package. Gives the application services and the API layer a focused, stable dependency for persistence — without importing anything from the transport layer or the rules engine.

**Not responsible for:** business logic, HTTP routing, domain validation (models live in `pw-domain-contracts`), or decision evaluation.

---

## Status

**Extract** from `backend/src/process_wizard/db/`. **Deferred — not in scope for Phases A-C.**

This document exists so that when the extraction is prioritized, the boundary and contract are already specified.

---

## Source location

```
backend/src/process_wizard/db/
├── tables.py        ← SQLAlchemy ORM models (RunTable, DocumentTable, UserTable,
│                       RunAnalyticsTable, ReferenceIndexTable, etc.)
├── repositories.py  ← CRUD functions grouped by domain
├── session.py       ← async session factory, engine configuration
└── seed.py          ← database seeding (may stay in app or move here)
```

Pre-requisite: split the large `repositories.py` into domain-specific modules (`documents_repo.py`, `runs_repo.py`, `users_repo.py`, etc.) before extraction to prevent a coupling hotspot.

---

## Public API / Exports

```python
# Session management
from pw_data_access import get_async_session, create_engine_from_url

# ORM table classes (for direct queries in services)
from pw_data_access.tables import (
    RunTable, DocumentTable, UserTable, GroupTable,
    RunAnalyticsTable, ReferenceIndexTable, AssignmentTable,
    StylesheetTable, WorkflowTable,
)

# Repository functions (domain-grouped)
from pw_data_access.repositories import documents, runs, users, groups, references
# e.g.: await documents.get_by_id(session, document_id)
#        await runs.create(session, run_data)
```

---

## Dependencies

**External (pip):**
- `sqlalchemy[asyncio]>=2.0`
- `aiosqlite` (dev)
- `asyncpg` (production)
- `alembic` (migrations, may stay in app)

**Internal pw-* dependencies:**
- `pw-domain-contracts` — repository functions accept/return domain model types

---

## Out of scope

- Business logic (run completion orchestration, risk computation) — `pw-run-pipeline`
- Migration scripts — Alembic config stays in the app until explicitly moved
- Seeding domain data from plugin manifests — app-level concern

---

## Acceptance criteria

1. All existing `pytest` integration tests pass with the DB layer imported from `pw_data_access` instead of `process_wizard.db`.
2. `get_async_session()` works for both SQLite (`aiosqlite`) and PostgreSQL (`asyncpg`) connection strings.
3. Repository functions are split by domain (documents, runs, users, groups, references) with no mega-module.
4. No imports from `process_wizard.api` or any HTTP framework in the package.

---

## Phase

**Deferred** — does not block Phase A-C work. Extract when a second application (admin console, mobile backend) needs to share DB access.
