# web-api-client

**Tier 3 — Frontend platform.**

Typed HTTP gateway. The single networking layer for every web app.

## Source material

Process Wizard `libraries/web-api-client.md` (`frontend/src/api/client.ts`, ~700 lines, 50+ endpoints).

## Responsibilities

- Typed `fetch` wrappers per domain (auth, runs, documents, users, teams, …).
- Retry policies and timeout handling.
- Refresh-token interceptor: on 401, refresh once, replay request, otherwise log out.
- Request ID propagation header.
- Error envelope normalization aligned with `core-api-runtime`.

## Dependencies

- External: browser `fetch`
- Internal: `core-schemas-ts`

## Consumers

`web-auth-session`, `web-wizard-runtime`, `web-features-shared`, every domain TS plugin, every app shell.
