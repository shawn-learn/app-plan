# user-service

## 1. Purpose

`user-service` owns the user profile and everything attached to it that is not credentials: display data, preferences (units, locale, default views), interest tags, and the per-user parameter visibility / log-group rules that drive what each user sees and can record on exercises.

## 2. Scope

**In scope**
- User profile CRUD (display name, avatar, status, etc.).
- User preferences (`/users/me/preferences`).
- Interest tags and the link table to users.
- Parameter visibility rules (`/users/me/parameter-rules`) including visibility groups and log groups.
- Aggregating profile data for other services (e.g., a thin "current user view").

**Out of scope**
- Credentials / login (→ `auth-service`).
- Unit conversion math (→ `shared-units`); `user-service` only stores the *preference*.
- Team membership (→ `team-service`).

## 3. Current Capabilities

Routers: `app/routers/users.py`, `preferences.py`, `parameter_rules.py`, `interests.py`. Services: `app/services/users.py`, `preferences.py`, `parameter_rules.py`. Schemas: `app/schemas/users.py`, `parameter_rules.py`, `interests.py`. The README documents the parameter-visibility and log-group model in detail.

The product also already has a separate `docs/USER_MANAGEMENT_REQUIREMENTS.md` describing the full user-management vision (auth flows, OAuth providers, calendar tokens, relationships). That document is intentionally kept independent and is **not** linked from this requirements set.

## 4. Public API Surface

- HTTP: `GET/PUT /users/me`, `/users/me/preferences`, `/users/me/parameter-rules`, `/users/me/interests`, `/interests`, `/parameter-rules` (catalog).
- Python: `get_user(id)`, `get_preferences(user)`, `visible_parameters_for(user, exercise_category)`, `loggable_parameters_for(...)`.

## 5. Data Model

Owns: `users` (non-credential columns), `interest_tags`, `user_interest_links`, parameter-rule tables (`parameter_rules`, `visibility_groups`, `visibility_group_members`, `parameter_log_groups`, `log_group_members`), any `user_unit_preferences` or `user_preferences` table.

## 6. Dependencies

- Depends on: `auth-service` (to identify the current user), `shared-units` (to validate unit preferences).
- Must NOT depend on: any other Tier 2 lib.

## 7. Cross-Cutting Concerns

- **Privacy**: profile visibility rules; users can hide parameters and ranges from themselves and (eventually) from collaborators.
- **Default-on-create**: new users get sensible default parameter rules — defaults are owned here.
- **i18n**: locale preference is stored here; consumed by frontend.

## 8. Open Questions / Known Gaps

- The relationship between `user_unit_preferences` (if it exists) and `users` columns is ambiguous in the current code.
- Parameter-rule defaults are seeded but not centrally documented.
- `docs/USER_MANAGEMENT_REQUIREMENTS.md` describes scope (calendar tokens, OAuth providers, relationships) that is not yet implemented; tracking which of those land here vs. in `auth-service` vs. `calendar-service` is open.
