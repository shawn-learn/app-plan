# 04 — Repository and Packaging Standards

## Repository Shape

- `packages/py/*` for backend libraries
- `packages/ts/*` for frontend/shared TS libraries
- `apps/*` for assembled products
- `docs/*` for architecture, ADRs, and migration docs

## Polyrepo-Ready Rules

1. Each package is independently buildable/testable.
2. Use package-name imports only; avoid relative cross-package imports.
3. Declare and pin internal dependencies in manifests.
4. Maintain independent semantic versions and changelogs.
5. Run package-isolated CI jobs.
6. Preserve an internal registry path for future extraction.

## Tooling Baseline

- Python: `uv` workspace + `pyproject.toml`
- TypeScript: `pnpm` workspace + `package.json`
- Backend stack: FastAPI + SQLAlchemy + Pydantic
- Frontend stack: React + TypeScript + MUI + React Hook Form + Recharts
- Database: PostgreSQL/TimescaleDB (SQLite for local dev)
