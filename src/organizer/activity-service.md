# activity-service

## 1. Purpose

`activity-service` owns the runtime side of training: the planned activities a user has scheduled, the activity logs that record what they actually did, the per-exercise log rows with their flexible JSON performance data, and the metrics derived from those logs. It is the busiest, highest-write library in the system.

## 2. Scope

**In scope**
- Planned activities (`planned_activities`): create, update, status transitions (planned → in-progress → completed/skipped).
- Activity logs (`activity_logs`) and their per-exercise rows (`workout_log_exercises`).
- Free-form (no-template) logging.
- Strength-domain calculations and metric aggregations (`/metrics`).
- Activity types lookup.
- The `planned_activity_json` and log-side JSON contracts (validated via `shared-schemas`).

**Out of scope**
- The template that produced the planned activity (→ `template-service`).
- Recurrence rules and calendar views (→ `calendar-service`).
- Milestones tied to activities (→ `goals-service`).

## 3. Current Capabilities

Routers: `app/routers/activities.py`, `workout_logs.py`, `metrics.py`. Services: `app/services/activities.py`, `strength.py`. Schemas: `app/schemas/activities.py`, `metrics.py`. The README's three-tiered architecture explicitly identifies this layer; the Mermaid workflow diagram shows the Template → Plan → Log pipeline that this library implements the second and third stages of.

## 4. Public API Surface

- HTTP: `GET/POST/PATCH/DELETE /planned-activities`, `/activity-logs`, `/workout-logs`, `/metrics/*`.
- Python: `create_planned(user, payload)`, `start_log(planned_id)`, `append_set(log, exercise, payload)`, `complete_log(log)`, `metrics_for(user, exercise, range)`.

## 5. Data Model

Owns: `planned_activities`, `activity_logs`, `workout_log_exercises`, `activity_types`. Reads from: `users`, `exercises` (via `exercise-service`), `activity_templates` (when generating from a template).

## 6. Dependencies

- Depends on: `shared-schemas`, `shared-units`, `exercise-service`, `user-service`, `auth-service`.
- Must NOT depend on: `template-service`, `calendar-service`, `goals-service` (those depend on this lib).

## 7. Cross-Cutting Concerns

- **JSONB validation**: all writes to `*_json` columns must be schema-validated.
- **Time zones**: `planned_start` is timezone-aware; logs record both local and UTC times.
- **Idempotency**: clients (especially offline ones) may retry creates; logs need stable client-side IDs to dedupe.
- **TimescaleDB**: log tables are time-series candidates; chunking strategy lives here.

## 8. Open Questions / Known Gaps

- `parse_planned_start` lives in `app/utils` and is shared between activity and calendar code; its home is ambiguous.
- The boundary between "metric" (this lib) and "milestone" (goals-service) needs sharpening.
- Recent commit history mentions a "workout template editor" loop bug — there is implicit state coupling between this lib and `template-service` that should be made explicit.
