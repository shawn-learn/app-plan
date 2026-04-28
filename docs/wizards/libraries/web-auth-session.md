# `@process-wizard/web-auth-session`

## Purpose

A React context package that manages frontend authentication state: user profile, login/logout/signup actions, token refresh on 401, and session bootstrap on page load. Any React application that connects to a ProcessWizard backend wraps itself in the `AuthProvider` from this package and uses the `useAuth` hook to access the current user.

**Not responsible for:** HTTP token storage mechanics (that is `@process-wizard/web-api-client`'s `TokenStore`), protected route gating (that is the host app's router), or any UI beyond a loading state.

---

## Status

**Extract** from `frontend/src/contexts/AuthContext.tsx`. **Deferred — not in scope for Phases A-C.**

---

## Source location

```
frontend/src/contexts/AuthContext.tsx   ← auth state, login/logout/signup, 401 interceptor
```

---

## Public API / Exports

```typescript
export { AuthProvider } from "./AuthContext";
export type { AuthProviderProps } from "./AuthContext";

export { useAuth } from "./AuthContext";
export type {
  AuthState,          // { user: UserProfile | null; isLoading: boolean; isAuthenticated: boolean }
  AuthActions,        // { login, logout, signup }
} from "./AuthContext";
```

### `AuthProvider`

```typescript
interface AuthProviderProps {
  apiClient: ReturnType<typeof createApiClient>;  // injected — no hard dep on web-api-client's internals
  children: ReactNode;
}
```

---

## Dependencies

**External (npm):** `react ^18 || ^19` (peer)

**Internal pw-* dependencies:**
- `@process-wizard/web-api-client` — calls `auth.login()`, `auth.logout()`, `auth.me()`

---

## Out of scope

- MFA/TOTP enrollment UI (that is a host-app feature or a separate `@process-wizard/web-mfa` package)
- Route protection — host app does that using the `isAuthenticated` flag
- Any non-auth state

---

## Acceptance criteria

1. `AuthProvider` wraps the app with an injected `apiClient`; no hardcoded API URLs in the package.
2. `useAuth().login()` sets `isAuthenticated: true` and populates `user` on success.
3. A 401 API response triggers a token refresh attempt via the injected client; if refresh fails, `isAuthenticated` becomes `false`.
4. `useAuth()` throws a helpful error when called outside `AuthProvider`.

---

## Phase

**Deferred** — the current `AuthContext.tsx` is adequate for the single-app monorepo. Extract when a second React application needs the same auth state.
