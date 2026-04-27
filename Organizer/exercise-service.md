# exercise-service

## 1. Purpose

`exercise-service` is the catalog of canonical exercises and everything that classifies or describes them: aliases, tags, categories, complexity levels, equipment, muscle groups and individual muscles, exertion definitions, and reusable per-exercise templates. It is the lookup layer that all activity-related code uses to resolve exercise references.

## 2. Scope

**In scope**
- CRUD and lookup for exercises and aliases (`/exercises`, `/exercises/lookup`).
- Exercise categories, tags, complexity, equipment, muscles, muscle groups.
- Exertion types and exertion definitions.
- Exercise templates (reusable building blocks) and their share permissions.
- Seeding the database with the canonical exercise list (under `database_seeds/exercises/`).

**Out of scope**
- Activity logs that *use* an exercise (→ `activity-service`).
- Workout templates that compose exercises (→ `template-service`).
- Team-level share rules (→ `team-service`); the share *tables* belong here but the team membership logic is delegated.

## 3. Current Capabilities

Routers: `app/routers/exercises.py`, `exercise_tags.py`, `exercise_categories.py`, `exercise_templates.py`, `complexity.py`, `equipment.py`, `muscles.py`. Service: `app/services/exercises.py`. Schemas: `app/schemas/exercises.py`. README documents the alias-based lookup contract: templates reference exercises by name (canonical or alias), resolved via `GET /exercises/lookup`.

## 4. Public API Surface

- HTTP: `/exercises`, `/exercises/lookup`, `/exercise-categories`, `/exercise-tags`, `/exercise-templates`, `/complexity`, `/equipment`, `/muscles`, `/muscle-groups`.
- Python: `resolve_exercise(name_or_alias)`, `list_exercises(filters)`, `parameters_for(category)`, `share_exercise(exercise, target_user_or_team)`.

## 5. Data Model

Owns: `exercises`, `exercise_aliases`, `exercise_complexity`, `muscle_groups`, `muscles`, `exercise_muscle_groups`, `equipment`, `exercise_equipment`, `exercise_templates`, `exercise_template_shares`, `exercise_shares`, `exertion_types`, `exertion_definitions`, exercise tag/category tables.

## 6. Dependencies

- Depends on: `shared-schemas`, `user-service` (creator and shares), `auth-service`. Optional: `team-service` for team-scoped shares.
- Must NOT depend on: `activity-service`, `template-service` (those depend on this lib).

## 7. Cross-Cutting Concerns

- **Naming**: alias resolution is case-insensitive; canonical names are immutable once published.
- **Seeding**: the shipped seed list under `database_seeds/exercises/` is the system-of-record for built-in exercises.
- **Sharing**: a single share-evaluation policy must be shared with `template-service` (likely lives here, exposed as a helper).

## 8. Open Questions / Known Gaps

- Per-exercise parameter visibility rules overlap with `user-service` parameter rules; the split is not always clean.
- Exertion definitions are JSON-driven; their schema may belong in `shared-schemas` once formalized.
- `exercise_parameter_table.md` and `exercise_tag_list.md` at the repo root suggest there is canonical reference material that should be packaged with this lib.
