# core-api-runtime

**Tier 3 — Transport.**

FastAPI app factory, router mounting, dependency wiring. The thin transport adapter on top of all Tier 0–2 backend libraries.

## Source material

Process Wizard `libraries/pw-api-runtime.md` (`main.py`, `api/*`).

## Responsibilities

- `create_app(config)` factory: wires auth, tenancy resolver, DB session, plugin registry, exception handlers, CORS, logging, request ID propagation.
- Standard router mounting helpers (`mount_router(app, plugin)`).
- Health and version endpoints.
- OpenAPI customization (consistent error envelopes).
- Dependency-injection wiring for `core-auth`, `core-tenancy`, `core-data-access`.

## Dependencies

- External: `fastapi`, `uvicorn`
- Internal: All Tier 0–2 backend packages.

## Consumers

App shells (`app-industrial`, `app-organizer`, `app-coach`) — each calls `create_app` and registers domain plugins.

## Notes

Transport-only: alternate transports (gRPC, GraphQL) could one day swap this package without touching the service libraries.
