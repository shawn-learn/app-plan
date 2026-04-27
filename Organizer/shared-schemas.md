# shared-schemas

## 1. Purpose

`shared-schemas` is the canonical source of truth for the structured data exchanged between the Organizer backend, frontend, CLI, and any future mobile/offline clients. It owns the JSON Schemas for activity payloads (workout, agenda, list, form, log, plan), templates, measurements, and user-scoped data, as well as the generated TypeScript and Python types derived from them. Every other library validates against, or imports types from, this one.

## 2. Scope

**In scope**
- JSON Schema definitions for activity payload variants and template formats (`schemas/activity/v1/`, `schemas/user/v1/`, top-level `schemas/`).
- Measurement schema (`schemas/measurement.schema.json`), duration (`duration.schema.json`), tempo (`tempo.schema.json`), planned-workout (`planned_workout.schema.json`), exercise category template (`exercise_category_template.schema.json`).
- Generated/co-located Pydantic schemas in `app/schemas/` that mirror the JSON Schemas (activities, articles, auth, category_templates, exercises, help_content, interests, metrics, parameter_rules, recurrence, system_log, team_tags, teams, users).
- Schema versioning conventions (the `v1` suffix convention).
- Shared enums and constants referenced from multiple domains (priority, activity status, etc.).

**Out of scope**
- Database table definitions (owned by individual service libraries).
- Validation logic beyond what JSON Schema can express (belongs in the consuming service).
- Data transformation / unit conversion (see `shared-units`).

## 3. Current Capabilities

The repository already centralizes JSON Schemas under `schemas/` with a `v1` convention. `app/schemas/` mirrors many of these as Pydantic models used by FastAPI for request/response validation. `docs/ACTIVITY_SCHEMAS.md` and `docs/TEMPLATE_FORMAT.md` document the contracts. `app/activity/` and `app/templates/` contain helper modules that load and operate on schema-validated JSON (e.g., `planned_workout.example.json`).

## 4. Public API Surface

- JSON Schema files (consumable by any language).
- Python: importable Pydantic models (`from organizer_schemas.activity import PlannedActivity` etc.).
- TypeScript: generated `.d.ts` types under `frontend/src/types/`.
- A `validate(payload, schema_name)` helper for runtime validation in tests and CLI tools.
- A schema-version constant and a `latest_for(name)` resolver.

## 5. Data Model

This library owns no persistent data. It owns the *shape* of data persisted by other libraries. All consumers of `planned_activities.planned_activity_json`, `activity_logs` JSON columns, `workout_log_exercises` performance JSON, and `activity_templates.json` must validate against schemas defined here.

## 6. Dependencies

- Depends on: nothing in the Organizer system. Allowed external deps: `jsonschema`, `pydantic`, codegen tools.
- Must NOT depend on: any other Organizer library (it is a Tier 1 leaf).

## 7. Cross-Cutting Concerns

- **Versioning**: every schema is versioned (`v1`, `v2`, …). Breaking changes require a new version, not an in-place edit. Consumers declare which versions they accept.
- **Validation errors**: must surface a path-aware error structure suitable for both API responses and CLI output.
- **i18n**: schemas should not embed user-facing strings; localization happens in the frontend.
- **Backwards compatibility**: removed fields stay parseable as `additionalProperties: true` with a deprecation note until the next major.

## 8. Open Questions / Known Gaps

- TypeScript type generation is not yet automated; `frontend/src/types/` is partly hand-written.
- Some Pydantic models in `app/schemas/` have drifted from the JSON Schemas; reconciliation is needed.
- No formal versioning policy is documented — `v1` is the only version in use.
- See `docs/TODO_features.md` and `docs/TODO_infra_testing.md` for backlog items related to schema validation tightening.
