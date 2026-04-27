# auth-service

## 1. Purpose

`auth-service` owns identity verification for the platform: signup, login, logout, password reset, JWT issuance/verification, and guest-cookie sessions. It does **not** own the user profile (that is `user-service`); it owns the credentials, tokens, and the dependency that other services use to find out *who is making this request*.

## 2. Scope

**In scope**
- `/auth` endpoints: signup, login, logout, refresh.
- Password hashing and verification.
- JWT issuance, signing, validation, expiry, rotation.
- Guest cookie sessions (anonymous users with limited functionality, per `docs/AUTH_FLOW.md`).
- The `get_current_user` FastAPI dependency consumed by every other router.
- Rate limiting on auth endpoints (intent — not yet implemented).

**Out of scope**
- User profile, preferences, interests (→ `user-service`).
- Authorization / role-based access (each service decides what its endpoints require, but this lib provides the principal).
- Team membership permissions (→ `team-service`).

## 3. Current Capabilities

`app/routers/auth.py` exposes the auth endpoints. `app/security.py` contains password hashing and JWT helpers. `app/dependencies/` supplies `get_current_user`. `docs/AUTH_FLOW.md` documents the JWT-bearer + guest-cookie fallback model. Pydantic schemas live in `app/schemas/auth.py`.

## 4. Public API Surface

- HTTP: `POST /auth/signup`, `POST /auth/login`, `POST /auth/logout`, `POST /auth/refresh`, `POST /auth/change-password`.
- Python: `get_current_user(token) -> User`, `issue_token(user_id, scopes)`, `verify_token(token)`, `hash_password()`, `verify_password()`.
- Cookie contract for guest sessions (name, signing, lifetime).

## 5. Data Model

Owns the credential-bearing fields on `users` (password hash, status). The user record itself is shared with `user-service` — both libraries read from `users` but only `auth-service` writes the credential columns. Owns any `password_resets` / `refresh_tokens` table that gets added.

## 6. Dependencies

- Depends on: nothing in the system (Tier 2 leaf).
- Must NOT depend on: `user-service` (reverse dependency — `user-service` is built atop this lib). May read minimal user fields via a thin internal adapter.

## 7. Cross-Cutting Concerns

- **Logging**: never log secrets, tokens, or passwords. Failed-login events should hit `system_log` via `content-service`.
- **CORS / cookies**: the `ALLOWED_ORIGINS` environment variable and credentialed cookies are auth concerns.
- **Secret management**: JWT signing key from environment, with rotation strategy documented.

## 8. Open Questions / Known Gaps

- No refresh-token rotation today.
- No multi-factor auth.
- `DEV_BYPASS_MESSAGING_AUTH` env flag exists; ensure prod cannot accidentally enable it.
- See `docs/USER_MANAGEMENT_REQUIREMENTS.md` for the broader auth model the team is converging on (kept independent from this doc per the planning brief).
