# domain-fitness (Python)

**Domain plugin — Fitness coaching.**

Backend half of the fitness domain. Has a TS twin at `packages/ts/domain-fitness/`. The largest of the three domain plugins.

## Source material

Merges the entire Organizer fitness vertical with Process Wizard's fitness plugin:

- `Organizer/exercise-service.md` (catalog, aliases, muscles, equipment, complexity, exertion, exercise templates)
- `Organizer/activity-service.md` (logging slice — `activity_logs`, `workout_log_exercises`, JSONB performance data, free-form logging)
- `Organizer/goals-service.md` (fitness-specific badges)
- Process Wizard `libraries/exercise-catalog.md`, `workout-editor.md`, `workout-runtime.md`, `pw-fit-parser.md`, `pw-training-metrics.md`
- Process Wizard fitness plugin: intake (26-step PAR-Q), assessments (metabolic threshold, musculature/strength, body comp, running plan), GoRules tables for risk stratification

## Responsibilities

- **Exercise catalog:** canonical exercises with aliases, muscle groups, equipment, complexity ratings, exertion definitions, shareable exercise templates.
- **Workout authoring:** block-based plan model (sets/reps/load, RPE, supersets, rest timers; HR/pace/zone × duration for endurance).
- **Session logging:** logger model + JSONB performance payloads.
- **FIT parser:** parse Garmin `.fit` files → normalized activity records.
- **Training metrics:** sport-agnostic training-load math (ACWR, TSS, zones, rolling-window queries).
- **Intake & assessment evaluators:** PAR-Q risk stratification, volume modifier, intensity ceiling, pain-adjusted load, availability summary — all registered with `core-plugin-contracts` and backed by JDM tables.
- **Periodization:** sections, supersets, circuits, AMRAP/EMOM, pyramids.
- **Coach↔athlete teams:** uses `core-teams`; ships team-template-share patterns.
- **Fitness milestones & badges:** specializations of `core-goals`.

## Dependencies

- External: `pydantic`, `sqlalchemy`, `fitparse`
- Internal: `core-domain-contracts`, `core-plugin-contracts`, `core-rules-engine`, `core-templates`, `core-run-pipeline`, `core-scheduling`, `core-goals`, `core-teams`, `core-users`, `core-units`, `core-data-access`

## Consumers

`app-coach` primarily.

## Boundaries

No industrial signatures, no procedure deviations, no personal-task list management. Cross-domain features (e.g., scheduling a workout) come *from* `core-scheduling`, not from a sibling domain plugin.
