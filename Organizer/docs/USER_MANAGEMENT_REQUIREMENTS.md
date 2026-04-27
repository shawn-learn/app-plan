# User Management Service Requirements

This document captures the reusable requirements for the shared user-management service that powers all applications in the ecosystem. It consolidates backend and frontend expectations, covering schema design, authentication flows, preference handling, personalization, and integration guidance.

## Data Schema & Persistence
- Core user records live in the `users` table with identity details (`name`, `email`, hashed password), demographic fields (`birthdate`, constrained `gender`, `ip_address`), status flags (`is_guest`, `is_admin`, `on_team`, `is_client`, `status`), profile assets (`user_image`), calendar token, and a JSON `preferences` blob for per-user settings.
- Versioned public keys for end-to-end messaging are stored in `user_public_keys`, keyed by `(user_id, version)` with timestamps to audit key rotation events.
- Additional user context includes `user_injury` (injury history), `user_relationships` (typed connections with permissions and lifecycle states), and `user_subscriptions` (plan membership joined to `subscription_plans` and `plan_features`).
- Preferences that need relational lookups are normalized through `user_unit_preferences`, which ties a user to a measurement/unit pair with optional exercise or tag scoping, enforcing uniqueness per scope to avoid conflicts.
- Customization metadata is seeded via `interest_tags`; users opt into them through `user_interest_links`, enabling many-to-many tagging for personalization workflows.

## Authentication & Account Lifecycle
- FastAPI routes manage guest session bootstrap, signup, login with optional “remember me”, logout, and email uniqueness checks. Guest creation seeds defaults (including `preferences`) and issues a cookie-backed JWT, while signup/login return bearer tokens for client storage.
- Token semantics rely on OAuth2 bearer tokens decoded by dependency helpers, enforcing authentication and allowing cookie-based retrieval when needed.
- Passwords are hashed via Passlib’s bcrypt context; JWTs are minted with configurable expiry using the shared secret and `utc_now`. Email addresses are normalized before persistence to enforce case-insensitive uniqueness.
- The documented flow clarifies that only guest accounts use the `user_token` cookie; registered users must supply the bearer token in the `Authorization` header.

## Profile & Security Management APIs
- `UserUpdatePayload` allows optional updates to name, email, birthdate (validated to not exceed today), gender, and avatar path; change-password and public-key payloads define the security operations for authenticated users.
- The `/users` router exposes endpoints to fetch/update self metadata, upload profile images, change passwords with verification, rotate or read public keys, list relationships, enumerate earned badges, and retrieve an aggregated profile snapshot.
- Profile aggregation merges stored preferences with unit overrides, computes derived data (age, estimated HR max) and surfaces the latest weight metric alongside badges for UI display.

## Preferences & Measurement Units
- Application defaults cover measurement units, intensity methods, and UI theme/language; they are distributed in both backend (`DEFAULT_PREFERENCES`) and frontend constants for synchronized behavior.
- Preference payloads are validated against a JSON Schema that constrains allowed unit symbols, intensity selectors, and UI choices, ensuring consistent client/server expectations.
- `/users/me/preferences` supports full replacement or partial patches, validating via JSON Schema and persisting values while calling the service layer to upsert normalized unit mappings into `user_unit_preferences` according to field-to-measure translations.

## Interests & Demographic Preferences
- Interest tags span training goals, modalities, and coaching focus; users can add/remove them via protected endpoints that verify ownership and reject duplicates.
- Frontend surfaces both a chip-based editor and an infinite-scroll selector. They load catalog and user selections with authenticated fetches, enforce a minimum of three interests when continuing, and propagate changes through the REST endpoints.
- The settings UI exposes quick access to edit interests alongside other profile controls, reinforcing their role in personalization workflows.

## Calendar & Relationship Features
- Calendar tokens are generated lazily or regenerated on demand, enabling an ICS feed served under `/calendar/{token}.ics`; unauthorized tokens raise 404s for safety.
- The user settings page mirrors backend capabilities by showing the ICS URL, offering copy/regenerate controls, and displaying relationship and badge summaries fetched from `/users/me/*` endpoints.
- Relationship listings return partner IDs, names, types, statuses, and permissions derived from `user_relationships`, supporting social or coaching workflows.

## Front-End Integration & Flows
- `UserSettingsPage` orchestrates profile data loading, saving, preference editing, calendar management, relationship/badge display, and navigation callbacks. It leverages `UserPreferencesForm` for structured selects with context help and default fallbacks.
- Authentication flows include signup (with email validation and client-side public key registration), login with “trust this computer”, and password change dialogs. All rely on shared utilities that append bearer tokens, detect expiry, and surface unauthorized states.
- Preference constants on the client mirror backend enumerations, ensuring forms render the same option lists validated server-side.

## Service Integration Guidance
- Expose the user-management service over an internal API gateway so multiple applications can authenticate and share user state consistently.
- Provide SDKs or client libraries (TypeScript for web, Python for services) that wrap authentication, profile fetch/update, preferences, and interest management endpoints with robust error handling and retries.
- Ensure downstream applications treat the user service as the source of truth for preferences, demographic data, relationships, and subscriptions; synchronize via webhook events or polling for changes when necessary.
- Document endpoint contracts and JSON schemas for consumers, and version APIs to support rolling upgrades across applications.

## Testing
⚠️ Not run (documentation-only change).
