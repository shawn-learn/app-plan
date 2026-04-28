# frontend-features

## 1. Purpose

`frontend-features` is the umbrella for per-domain React feature packages — the actual user-facing pages, flows, and feature-scoped state. It mirrors the backend Tier 2 domains so each domain has a dedicated feature module that owns its routes, page components, domain-specific hooks, and view models. The umbrella library is documented as one entry to keep the per-library doc set manageable; in implementation it would be split into ~8 sub-packages.

## 2. Scope

**In scope** — the following sub-features (one per backend domain):

- **auth** — `LoginForm.jsx`, `SignupForm.jsx`, `ChangePasswordPage.jsx`.
- **users / settings** — `UserSettingsPage.jsx`, `UserPreferencesForm.jsx`, `ExerciseAttributeVisibilityPage.jsx`, `UserInterestsPage.jsx`, `UserInterestEditor.jsx`, `UserInterestEditorPage.jsx`, `InterestPreferenceSelector.jsx`.
- **exercises** — `frontend/src/features/exercises/` (existing folder; the seed of the feature-package convention).
- **templates** — `WorkoutTemplatesPage.jsx`, `ModernTemplateEditor.jsx`, `CompactWorkoutStructureEditor.jsx`, `WorkoutItemDialog.jsx`, `WorkoutItemEditor.jsx`, `WorkoutOutline.jsx`, `GroupEditor.jsx`, `ListEditor.jsx`, `FormEditor.jsx`, `AgendaEditor.jsx`, `AgendaItemEditor.jsx`, `LogJsonEditor.jsx`.
- **activities (planner & guide)** — `WorkoutPlannerPage.jsx`, `WorkoutGuide.jsx`, `PlannedActivityForm.jsx`, `ActivityLogList.jsx`, the per-exercise editors (`ExerciseRepsEditor.jsx`, `ExerciseDistanceEditor.jsx`, `ExerciseDurationEditor.jsx`, `ExerciseElevationEditor.jsx`).
- **calendar** — the `FullCalendar*View.jsx` family, `CalendarView.jsx`, `PeriodizationTimeline.jsx`.
- **progress / metrics** — `Dashboard.jsx`, `MetricsPage.jsx`, charts.
- **teams** — `TeamManagerPage.jsx`.
- **content / help** — `ArticlePage.jsx` (page-level wrapper around the kit's renderer).
- **messaging** — *future*; depends on backend `messaging-service` becoming real.
- **app shell** — `App.jsx`, `App.css`, `index.css`, top-level routing, `main.jsx`.

**Out of scope**
- Reusable widgets (→ `frontend-ui-kit`).
- Cross-cutting concerns like auth-token handling (→ `frontend-ui-kit`).

## 3. Current Capabilities

The `frontend/src/` folder is largely flat today, with most page-level `.jsx` files at the top level of `src/`. The `features/exercises/` folder is the only existing feature-package. The pages enumerated above implement the bulk of the user-facing product as described in `docs/FEATURES_OVERVIEW.md`.

## 4. Public API Surface

Each feature package exports:
- A top-level route component / route-config fragment.
- Domain hooks (e.g., `useTemplates`, `useActiveWorkout`, `useCalendarRange`).
- Feature-scoped contexts where needed.

The umbrella exposes a route table that the app shell mounts.

## 5. Data Model

None. State lives in React Query cache (server state) and feature-local React state / context (UI state). No persistent client storage beyond what `sync-service`'s frontend integration provides.

## 6. Dependencies

- Depends on: `frontend-ui-kit`, `shared-schemas`, `shared-units`, and (at the API boundary) every relevant Tier 2 backend library's HTTP surface.
- Must NOT depend on: another feature package directly. Cross-feature composition happens at the app-shell level via routing or via shared hooks promoted to the kit.

## 7. Cross-Cutting Concerns

- **Routing**: each sub-feature owns its own routes and lazy-loads itself.
- **Authorization gates**: routes consult `useAuth` from the kit; per-feature role checks (e.g., team admin pages) call into `team-service`.
- **Error boundaries**: every feature is wrapped in the kit's `ErrorBoundary`.
- **Telemetry**: feature-level usage events go through the kit's logger which posts to `content-service`'s `system_log`.

## 8. Open Questions / Known Gaps

- Most pages live at the root of `frontend/src/` and need to be moved into per-feature folders before the split is real.
- Several pages (`Dashboard.jsx`, `MetricsPage.jsx`) cut across multiple domains and may need to be decomposed.
- TypeScript adoption is uneven; many feature files are still `.jsx`.
- The recent fix "Fix workout guide editor loops" indicates feature-internal state coupling that needs cleanup before extraction.
- Messaging UI is greenfield once `messaging-service` lands.
