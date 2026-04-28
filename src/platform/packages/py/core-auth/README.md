# core-auth

**Tier 1 — Identity.**

Authentication primitives: OAuth2 + JWT, refresh-token rotation, TOTP MFA, password reset, guest cookie sessions.

## Source material

Merges:
- Process Wizard `system-requirements.md` REQ-AUTH-001..003 (Ed25519 JWTs, TOTP, refresh rotation, IP/UA binding)
- Organizer `auth-service.md` (OAuth2 + JWT bearer, guest cookie fallback)

## Responsibilities

- Issue 15-minute access JWTs (Ed25519, `kid`-rotated keys) and 7-day rotating refresh JWTs (HttpOnly cookies).
- TOTP MFA with backup codes and email OTP fallback; `mfa=1` claim on session upgrade.
- Argon2id password hashing, ≥12 char policy.
- Password reset via single-use, time-limited links (`itsdangerous`); reset revokes all refresh tokens.
- Guest cookie sessions for unauthenticated app surfaces (Organizer-style).
- Rotation nonces stored in Redis.

## Dependencies

- External: `pyjwt`/`authlib`, `argon2-cffi`, `pyotp`, `redis`, `itsdangerous`
- Internal: none

## Consumers

`core-api-runtime`, every service that gates by user identity. App shells configure key rotation and Redis backend.
