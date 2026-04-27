# template-service

## 1. Purpose

`template-service` owns reusable workout templates: the JSON-defined, version-controlled training structures that users build once and schedule many times. It also owns the CLI tooling for importing and exporting templates and the helpers that resolve a template into a planned activity payload.

## 2. Scope

**In scope**
- Activity templates CRUD (`/templates`).
- Template versioning (`template_id` + `version` lineage).
- Template-to-planned-activity resolution (turning a template + scheduling parameters into a `planned_activity_json`).
- CLI import/export (`scripts/import_template.py`, `scripts/export_template.py`).
- The template format documented in `docs/TEMPLATE_FORMAT.md` and `docs/PLANNED_WORKOUT_EDITOR_SUPPORT.md`.

**Out of scope**
- Exercise definitions referenced by a template (→ `exercise-service`).
- The planned/log records that result from scheduling a template (→ `activity-service`).
- Sharing logic with teams (delegated to `team-service`).

## 3. Current Capabilities

Router: `app/routers/templates.py`. Helpers: `app/templates/`. Sample template: `frontend/src/sampleTemplate.json` and `docs/planned_workout.example.json`. Frontend pages: `WorkoutTemplatesPage.jsx`, `ModernTemplateEditor.jsx`, `CompactWorkoutStructureEditor.jsx`. The README describes the import/export CLI commands.

## 4. Public API Surface

- HTTP: `GET/POST/PUT/DELETE /templates`, `/templates/{id}/versions`, `/templates/{id}/resolve`.
- CLI: `import_template.py`, `export_template.py`.
- Python: `resolve_template(template, scheduling) -> PlannedActivityJSON`, `next_version(template_id)`.

## 5. Data Model

Owns: `activity_templates` (including its JSON content). Reads from: `exercises` (for name resolution at edit time and at resolve time).

## 6. Dependencies

- Depends on: `shared-schemas`, `exercise-service`, `auth-service`, `user-service` (creator).
- Must NOT depend on: `activity-service` (the resolve helper produces a payload but does not persist it).

## 7. Cross-Cutting Concerns

- **Versioning**: templates are immutable per version; edits create a new version.
- **Validation**: every save runs through `shared-schemas`.
- **Sharing**: respects share rules defined in `exercise-service`'s share-evaluation policy and team membership from `team-service`.

## 8. Open Questions / Known Gaps

- Template ↔ exercise references are currently by name (alias-resolved); a future mode using stable exercise IDs is unclear.
- The recent fix "Fix workout guide editor loops and onboarding guard" hints at editor-state issues in the frontend that map to this library — needs a dedicated editor-state contract.
- Bulk import/migration for existing users' templates is not covered.
