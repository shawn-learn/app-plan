# `@process-wizard/web-api-client`

## Purpose

A typed HTTP client for all ProcessWizard API domains. Provides a single place for auth token handling, request retry logic with exponential backoff, and typed endpoint helper functions. Any frontend application (the main web app, a future admin console, or a mobile web shell) that needs to talk to a ProcessWizard backend imports this package instead of writing raw `fetch` calls.

**Not responsible for:** React state management (that is `@process-wizard/web-auth-session`), UI components, or wizard rendering.

---

## Status

**Extract** from `frontend/src/api/client.ts` and `frontend/src/types/index.ts`. **Deferred — not in scope for Phases A-C.**

---

## Source location

```
frontend/src/api/client.ts     ← ~700 lines: auth setup, retry, 50+ typed endpoint functions
frontend/src/types/index.ts    ← TypeScript interfaces for all API response shapes
```

Extracted package groups endpoint functions by domain module:

```
packages/web-api-client/src/
├── index.ts             ← barrel
├── core/
│   ├── client.ts        ← base fetch setup, retry, token storage interface
│   └── errors.ts        ← typed API error classes
├── endpoints/
│   ├── auth.ts          ← login, signup, logout, me
│   ├── documents.ts
│   ├── runs.ts
│   ├── forms.ts
│   ├── groups.ts
│   ├── stylesheets.ts
│   ├── workflows.ts
│   ├── roster.ts
│   ├── assignments.ts
│   ├── references.ts
│   └── units.ts
└── types/
    ├── index.ts         ← all shared TypeScript interfaces (WizardBundle, Run, etc.)
    └── api.ts           ← request/response shapes per endpoint
```

---

## Public API / Exports

```typescript
// Client factory
export { createApiClient } from "./core/client";
export type { ApiClientOptions, TokenStore } from "./core/client";

// Domain endpoints (each returns typed response shapes)
export * from "./endpoints/auth";
export * from "./endpoints/documents";
export * from "./endpoints/runs";
export * from "./endpoints/forms";
export * from "./endpoints/groups";
export * from "./endpoints/stylesheets";
export * from "./endpoints/workflows";
export * from "./endpoints/roster";
export * from "./endpoints/assignments";
export * from "./endpoints/references";
export * from "./endpoints/units";

// All TypeScript interfaces
export * from "./types";
```

---

## Dependencies

**External (npm):** none (browser `fetch` only)

**Peer dependencies:** none (framework-agnostic)

**Internal pw-* dependencies:** none.

---

## Out of scope

- React context or hooks — `@process-wizard/web-auth-session`
- UI components
- Wizard rendering

---

## Acceptance criteria

1. All existing frontend API calls work after the app switches to importing from `@process-wizard/web-api-client`.
2. TypeScript strict mode passes with no `any` in the public API surface.
3. The client is framework-agnostic: it can be used in a plain HTML page (no React required).
4. Retry logic: a 401 response triggers a single token refresh attempt before failing.

---

## Phase

**Deferred** — the current `frontend/src/api/client.ts` is adequate for the single-app monorepo. Extract when a second frontend application needs shared API access.
