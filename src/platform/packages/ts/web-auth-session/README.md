# web-auth-session

**Tier 3 — Frontend platform.**

React auth context, token lifecycle, login/signup/logout state.

## Source material

Process Wizard `libraries/web-auth-session.md` (`frontend/src/contexts/AuthContext.tsx`).

## Responsibilities

- `<AuthProvider>` React context.
- Login, signup, logout, MFA challenge, password reset flows.
- Session bootstrap on app load.
- Auto-refresh hook integration with `web-api-client`'s 401 interceptor.
- Hooks: `useAuth()`, `useUser()`, `useRequireRole(role)`.

## Dependencies

- External: `react`
- Internal: `web-api-client`, `core-schemas-ts`

## Consumers

App shells wrap their tree in `<AuthProvider>`. Every protected feature uses `useAuth()`.
