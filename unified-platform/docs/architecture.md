# Architecture

This is a condensed companion to the approved plan at `~/.claude/plans/please-review-the-two-hashed-plum.md`. Refer to that file for full context, rationale, and phased delivery.

## Three layers

```
APP SHELLS         app-industrial    app-organizer    app-coach
                              ▲
DOMAIN PLUGINS     domain-industrial  domain-organizer  domain-fitness
                              ▲
PLATFORM CORE      Tier 0  contracts, schemas, units
                   Tier 1  auth, tenancy, users, access, teams
                   Tier 2  wizard runtime, rules engine, plugin contracts,
                           run pipeline, signatures+audit, templates,
                           scheduling, calendar, goals, content, messaging,
                           sync, data access
                   Tier 3  api-runtime, web-api-client, web-ui-kit,
                           web-auth-session, web-wizard-runtime,
                           web-plugin-ui, web-form-editors, web-calendar,
                           web-features-shared
```

## Locked decisions

- **Industrial** = PW `system-requirements.md` as-written (multi-tenant compliance, signed canonical records, audit log, branching steps, deviations).
- **Organizer** = tasks & lists + calendar/scheduling + habits & personal goals. No notes/journal in v1.
- **Coach** = single DB with org scoping. `core-tenancy` available for future flip.
- **Workspace** = pnpm + uv, polyrepo-ready (§6.1 in plan).

## Key reuse seams

1. **Wizard runtime is the unifying primitive.** Org templates and PW procedures both collapse to "versioned document executed as a run, producing a canonical record."
2. **Scheduling is generalized.** RRULE-based planned occurrences applied to procedures, tasks, and workouts alike.
3. **Teams = access groups + memberships.**
4. **Signatures & audit live in core**, opt-in per app.
5. **Fitness is demoted to a plugin.** No `Exercise` model in the platform.
6. **Reference store is generic.** Any computed run output can be indexed and consumed by downstream documents.
7. **One source of truth for cross-language types.** JSON Schema → Pydantic + generated TS, contract-tested via fixture round-trip.

## Tech stack

| Concern   | Choice |
|-----------|--------|
| Backend   | Python 3.12, FastAPI, Pydantic v2, SQLAlchemy 2.0 async, Alembic |
| Frontend  | React + TypeScript, Vite, MUI, React Hook Form, React Query, Recharts |
| DB        | PostgreSQL (TimescaleDB extension where time-series matters); SQLite for local dev |
| Wizard    | Form.io schemas + custom fields |
| Rules     | GoRules Zen Engine |
| Auth      | OAuth2 + JWT (Ed25519), TOTP MFA, refresh-token rotation |
| Storage   | S3-compatible |

## Polyrepo-ready discipline

- Self-contained package directories (own manifest, README, tests, CHANGELOG, version).
- No relative cross-package imports — consume by published name.
- Pinned, declared dependencies — every internal dep ranged in the manifest.
- Independent SemVer per package.
- Per-package CI matrix.
- Internal registry seam from day one (Verdaccio/GitHub Packages for TS, simple index for Py).

A package leaving the monorepo becomes a `git filter-repo` extraction + lockfile flip, not a refactor.
