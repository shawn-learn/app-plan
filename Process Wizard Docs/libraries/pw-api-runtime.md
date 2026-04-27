# `pw-api-runtime`

## Purpose

The FastAPI transport adapter for the ProcessWizard backend. Owns all HTTP concerns: route definitions, request/response serialization, auth dependency injection, error handling, CORS configuration, and plugin startup wiring. Acts as the thinnest possible layer between HTTP and the underlying service/repository libraries.

**Not responsible for:** any business logic (that lives in `pw-run-pipeline` and domain plugins), ORM definitions, domain models, or UI.

---

## Status

**Extract** from `backend/src/process_wizard/main.py` and `backend/src/process_wizard/api/`. **Deferred — not in scope for Phases A-C.**

---

## Source location

```
backend/src/process_wizard/
├── main.py          ← FastAPI app initialization, CORS, plugin startup
└── api/
    ├── auth.py      ← /api/auth/*
    ├── users.py     ← /api/users/*
    ├── documents.py ← /api/documents/*
    ├── runs.py      ← /api/runs/*
    ├── forms.py     ← /api/forms/*
    ├── records.py   ← /api/records/*
    ├── groups.py    ← /api/groups/*
    ├── stylesheets.py
    ├── workflows.py
    ├── assignments.py
    ├── roster.py
    ├── references.py
    ├── units.py
    └── deps.py      ← dependency injection (get_current_user, get_session)
```

---

## Public API / Exports

```python
from pw_api_runtime import create_app

app = create_app(
    db_url="sqlite+aiosqlite:///./process_wizard.db",
    plugins_env="fitness",   # maps to PW_PLUGINS
    cors_origins=["http://localhost:5173"],
)
# Returns a configured FastAPI application, ready for uvicorn.
```

All route modules remain internal; hosts interact only with the ASGI app instance.

---

## Dependencies

**External (pip):**
- `fastapi`, `uvicorn` (or any ASGI server)
- `python-multipart` (file upload)
- `python-jose` or equivalent for JWT verification

**Internal pw-* dependencies:**
- `pw-domain-contracts`
- `pw-data-access`
- `pw-run-pipeline`
- `pw-plugin-contracts`
- `pw-rules-engine`

---

## Out of scope

- Business logic — delegates entirely to `pw-run-pipeline` and domain plugins
- Alternative transports (gRPC, GraphQL) — those are separate adapter packages
- Frontend serving — the frontend app is deployed separately

---

## Acceptance criteria

1. `create_app()` returns a working FastAPI instance; all existing integration tests pass.
2. The app starts cleanly with `uvicorn pw_api_runtime:app` after `pip install pw-api-runtime`.
3. No domain or business logic in any route module — all routes delegate to service functions.

---

## Phase

**Deferred** — the app's current `main.py` + `api/` already does this job. Extract when a second deployment configuration (e.g., serverless, different DB engine) needs a different app factory.
