# domain-fitness (TypeScript)

**Domain plugin — Fitness coaching (frontend).**

Twin of `packages/py/domain-fitness/`. The largest TS domain package.

## Source material

- `Organizer/frontend-features.md` (exercise/activity/template/progress/calendar/teams slices)
- Process Wizard `libraries/exercise-catalog.md`, `workout-editor.md`, `workout-runtime.md`
- Existing fitness plugin under `frontend/src/plugins/fitness/`

## Responsibilities

- **Exercise catalog UI:** browse, filter, alias management.
- **Workout editor:** block-based authoring (sets/reps/load/RPE, supersets, rest timers; HR/pace zone × duration).
- **Workout runtime:** logger mode (post-hoc entry) and live mode (interval/rest timers, set-by-set guidance).
- **FIT upload & review:** drag-to-upload, parsed metrics summary.
- **Progress & analytics:** charts of volume / weight / estimated 1RM, ACWR & zone distribution, recovery status.
- **Intake & assessment wizards:** Form.io schemas registered via `web-plugin-ui` (PAR-Q intake, threshold test, strength assessment, body comp, running plan).
- **Coach console:** roster, client profiles, assign documents, run-status tracking.
- **Athlete portal:** plan view, today's workout, log session, history.
- **Team views:** athlete↔coach team management on top of `web-features-shared` teams admin.
- **Periodization & milestones:** badges UI, milestone scheduling.

## Dependencies

- External: `react`, `react-router-dom`, `recharts`
- Internal: `web-ui-kit`, `web-api-client`, `web-auth-session`, `web-wizard-runtime`, `web-plugin-ui`, `web-form-editors`, `web-calendar`, `web-features-shared`, `core-schemas-ts`, `core-units`

## Consumers

`app-coach` primarily.
