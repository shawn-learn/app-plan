# User Management Service Requirements

This document captures the requirements for the user-management service that powers the application. It consolidates backend and frontend expectations, covering schema design, authentication flows, preference handling, personalization, and integration guidance.

## Data Schema & Persistence
- Core user records live in the `users` table with identity details (`name`, `email`, hashed password), status flags ( `is_admin`, `is_client`, `status`), profile assets (`user_image`), and an organization_id’ linked to the user organization.
- Versioned public keys for end-to-end messaging are stored in `user_public_keys`, keyed by `(user_id, version)` with timestamps to audit key rotation events.
- Preferences that need relational lookups are normalized through `user_unit_preferences`, which ties a user to a measurement/unit pair with optional exercise or tag scoping, enforcing uniqueness per scope to avoid conflicts.

## Authentication & Account Lifecycle
- Token semantics rely on OAuth2 bearer tokens decoded by dependency helpers, enforcing authentication and allowing cookie-based retrieval when needed.
- Passwords are hashed via Passlib’s bcrypt context; JWTs are minted with configurable expiry using the shared secret and `utc_now`. Email addresses are normalized before persistence to enforce case-insensitive uniqueness.
- The documented flow clarifies that only guest accounts use the `user_token` cookie; registered users must supply the bearer token in the `Authorization` header.

## Profile & Security Management APIs
- `UserUpdatePayload` allows optional updates to name, email, birthdate (validated to not exceed today), gender, and avatar path; change-password and public-key payloads define the security operations for authenticated users.
- The `/users` router exposes endpoints to fetch/update self metadata, upload profile images, change passwords with verification, rotate or read public keys, list relationships, enumerate earned badges, and retrieve an aggregated profile snapshot.

### 5.9.5 Endpoint Contract

The table below enumerates the concrete routes hosted under the `/users` router.  Each endpoint requires an OAuth2 bearer token unless otherwise stated, and all responses wrap payloads in the shared `{ "data": ... }` envelope used throughout the platform.

| Operation | Method | Path | Request Schema | Success Response | Notes |
| --- | --- | --- | --- | --- | --- |
| Fetch current profile | `GET` | `/users/me` | _None_ | `UserProfileResponse` | Returns the authenticated user's full profile including base metadata and resolved organization context. |
| Update current profile | `PATCH` | `/users/me` | `UserUpdatePayload` | `UserProfileResponse` | Applies partial updates; rejects attempts to change immutable identifiers (id, created_at). |
| Upload profile image | `POST` | `/users/me/avatar` | `multipart/form-data` with `file` part | `202 Accepted` with `ImageUploadReceipt` | Image uploads are streamed to object storage; the receipt contains the finalized CDN URL. |
| Change password | `POST` | `/users/me/change-password` | `ChangePasswordPayload` (`current_password`, `new_password`) | `204 No Content` | Enforces bcrypt verification of the current password before persisting the hash for the new password. |
| List public keys | `GET` | `/users/me/public-keys` | _None_ | `UserPublicKeyCollection` | Returns all active key versions with rotation timestamps; filters out revoked entries by default. |
| Rotate public key | `POST` | `/users/me/public-keys` | `RotatePublicKeyPayload` (`public_key_pem`, optional `expires_at`) | `201 Created` with `UserPublicKey` | Creates a new key version and deactivates the previous default. |
| Fetch preferences | `GET` | `/users/me/preferences` | Optional query `scope` (`exercise`, `tag`, `global`) | `UserPreferenceCollection` | Returns effective preferences merged from defaults and scoped overrides. |
| Replace preferences | `PUT` | `/users/me/preferences` | `UserPreferenceUpsertPayload` | `204 No Content` | Payload validated against the published JSON Schema; replaces the full preference document for the provided scope. |
| Fetch relationships | `GET` | `/users/me/relationships` | Optional query `type` (`coach`, `athlete`, `peer`) | `UserRelationshipCollection` | Supports pagination through `page` and `page_size` query parameters. |
| Fetch earned badges | `GET` | `/users/me/badges` | Optional query `include_hidden` (bool) | `UserBadgeCollection` | Hidden badges only available to admins or when explicitly requested with proper scope. |
| Fetch profile snapshot | `GET` | `/users/me/dashboard` | Optional query `tz` (IANA name) | `UserDashboardSnapshot` | Aggregates activity streaks, latest workouts, and outstanding tasks in a single payload. |
| Admin search users | `GET` | `/users` | Query parameters `email`, `status`, pagination args | `UserSearchResults` | Requires `is_admin=true`; provides sanitized summary records for list views. |
| Admin fetch user | `GET` | `/users/{user_id}` | _None_ | `UserProfileResponse` | Returns the full profile for the specified user, respecting PII sanitization rules where applicable. |
| Admin deactivate user | `POST` | `/users/{user_id}/deactivate` | `UserDeactivatePayload` (`reason`) | `202 Accepted` | Soft-deactivates the user and triggers asynchronous revocation of active sessions. |

Error responses follow the shared problem-details contract (`application/problem+json`) with machine-readable `code` values such as `user.not_found`, `user.invalid_password`, and `user.preference_conflict`.

## Preferences
- Preference payloads are validated against a JSON Schema that constrains allowed including UI choices, ensuring consistent client/server expectations.

## PII Handling & Sanitization
- **REQ-PII-001**: Verification responses MUST omit raw personal identifiers such as email addresses, phone numbers, and IP addresses. Only sanitized or pseudonymized values may be returned to clients.
- **REQ-PII-002**: Any user identifier present in verification payloads MUST be replaced with a tenant-scoped pseudonym computed as `u_` followed by the first 10 hexadecimal characters of `SHA-256(APP_PSEUDO_SECRET || tenant_id || '|' || user_id)`.
- **REQ-PII-003**: The secret `APP_PSEUDO_SECRET` MUST be supplied exclusively through an environment variable and MUST NEVER be logged or written to persistent storage.
- **REQ-PII-004**: Free-text fields in verification responses MUST be sanitized to mask emails (`j*****@…`), mask phone numbers (`******NN` pattern), and replace all IP addresses with `[ip]` before the response is emitted.

## Front-End Integration & Flows
- `UserSettingsPage` orchestrates profile data loading, saving, preference editing.
- Authentication flows include signup (with email validation and client-side public key registration), login with “trust this computer”, and password change dialogs. All rely on shared utilities that append bearer tokens, detect expiry, and surface unauthorized states.
- Preference constants on the client mirror backend enumerations, ensuring forms render the same option lists validated server-side.

## Service Integration Guidance
- Expose the user-management service over an internal API gateway so multiple applications can authenticate and share user state consistently.
- Provide SDKs or client libraries (TypeScript for web, Python for services) that wrap authentication, profile fetch/update, preferences, and interest management endpoints with robust error handling and retries.
- Document endpoint contracts and JSON schemas for consumers, and version APIs to support rolling upgrades across applications.
