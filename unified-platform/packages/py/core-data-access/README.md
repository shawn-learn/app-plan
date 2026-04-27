# core-data-access

**Tier 2 — Platform.**

SQLAlchemy 2.0 async ORM base classes, repository helpers, per-tenant session routing.

## Source material

Process Wizard `libraries/pw-data-access.md` (`db/tables.py`, `db/repositories.py`, `db/session.py`).

## Responsibilities

- Async session factory and lifecycle.
- Per-tenant session router that consults `core-tenancy` to pick the correct DSN.
- Repository base class with common CRUD + filtered query patterns.
- Migration helpers (Alembic templates each package can extend).
- Seed-loading utilities used by tenant onboarding.

## Dependencies

- External: `sqlalchemy[asyncio]`, `alembic`, `asyncpg`/`aiosqlite`
- Internal: `core-domain-contracts`, `core-tenancy` (optional — single-tenant apps skip it)

## Consumers

Every Tier 2 service that touches the DB.

## Notes

Repositories should be split per domain area (`documents_repo`, `runs_repo`, `users_repo`, etc.) to avoid a coupling-hotspot mega-module — a known risk called out in `Process Wizard Docs/library-extraction-analysis.md`.
