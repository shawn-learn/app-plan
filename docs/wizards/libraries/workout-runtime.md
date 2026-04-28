# `@process-wizard/workout-runtime`

## Purpose

Renders a `WorkoutPlan` (authored by `@process-wizard/workout-editor`) as an interactive session in one of two modes:

- **Logger mode** (ships first): post-hoc entry of completed work — tick sets, enter weights/reps/RPE for strength; upload a FIT file or enter metrics manually for endurance. Produces a completed run record.
- **Live mode** (ships after logger): guided real-time walk-through with rest timers, set-by-set callouts, and interval countdowns. Shares plan-rendering code with logger mode; live mode is an execution-mode flag, not a separate library.

Used by the fitness plugin's session entry flows. Replaces `running_session_log.json`-based Form.io sessions for structured workout plans, while Form.io continues to serve free-form questionnaires.

**Not responsible for:** plan authoring (that is `@process-wizard/workout-editor`), FIT file parsing (that is `pw-fit-parser` via the `/api/fit/upload` backend endpoint), user authentication, or displaying historical analytics.

---

## Status

**New** — no existing source.

---

## Source location

New package at:

```
packages/workout-runtime/
├── package.json
└── src/
    ├── index.ts
    ├── adapters/
    │   ├── SessionStorageAdapter.ts   ← save/load session progress + submit completion
    │   └── FitUploadAdapter.ts        ← optional: upload a .fit file, returns activity summary
    ├── runtime/
    │   ├── WorkoutSessionPage.tsx     ← top-level component, mode-switched
    │   ├── PlanRenderer.tsx           ← renders plan structure (shared by both modes)
    │   ├── logger/
    │   │   ├── StrengthLogger.tsx     ← set ticks, weight/reps/RPE entry
    │   │   └── EnduranceLogger.tsx    ← FIT upload or manual metrics entry
    │   └── live/
    │       ├── LiveSession.tsx        ← guided UI (timers, current target display)
    │       ├── RestTimer.tsx
    │       └── IntervalDisplay.tsx
    ├── hooks/
    │   ├── useSessionState.ts
    │   └── useRestTimer.ts
    └── types.ts                       ← SessionState, LoggedSet, SessionResult
```

---

## Public API / Exports

```typescript
// Top-level component
export { WorkoutSessionPage } from "./runtime/WorkoutSessionPage";
export type { WorkoutSessionPageProps } from "./runtime/WorkoutSessionPage";

// Adapters (host implements)
export type { SessionStorageAdapter, SessionRecord } from "./adapters/SessionStorageAdapter";
export type { FitUploadAdapter, FitSummary } from "./adapters/FitUploadAdapter";

// Session state shapes
export type {
  SessionState,
  LoggedSet,         // { set_index: number; reps: number; load_kg: number; rpe?: number; completed: boolean }
  SessionResult,     // { plan_id: string; logged_blocks: LoggedBlock[]; duration_seconds: number; notes?: string }
  ExecutionMode,     // "logger" | "live"
} from "./types";
```

### `WorkoutSessionPageProps`

```typescript
interface WorkoutSessionPageProps {
  plan: WorkoutPlan;
  mode: ExecutionMode;
  adapter: SessionStorageAdapter;
  fitUploadAdapter?: FitUploadAdapter;   // optional; enables FIT upload in endurance logger
  onComplete: (result: SessionResult) => void;
  onAbandon?: () => void;
}
```

### `SessionStorageAdapter`

```typescript
interface SessionStorageAdapter {
  load(planId: string): Promise<SessionRecord | null>;   // resume in-progress session
  save(state: SessionState): Promise<void>;              // autosave
  complete(result: SessionResult): Promise<void>;        // write completed run record
}
```

---

## Dependencies

**External (npm):**
- none (UI only, no heavy runtime deps)

**Peer dependencies:**
- `react ^18 || ^19`, `react-dom`

**Internal pw-* dependencies:**
- `@process-wizard/workout-editor` — imports `WorkoutPlan`, `PlanBlock`, and related types
- `@process-wizard/exercise-catalog` — exercise name/instruction lookup during live session display

---

## Out of scope

- FIT file parsing (the `FitUploadAdapter` calls a backend endpoint that uses `pw-fit-parser`; no client-side binary parsing)
- Real-time HR/power device pairing or Bluetooth streaming — v1 is guided UI only
- Training-load computation — the backend evaluator handles that after session completion
- Displaying historical charts or analytics — belongs in the fitness plugin UI components

---

## Acceptance criteria

**Logger mode:**
1. Mount `WorkoutSessionPage` with a strength plan in logger mode; tick all sets, enter weights, submit — `onComplete` fires with a `SessionResult` containing logged data for every block.
2. Partially logged session is saved by `adapter.save(state)` on each set tick; reloading the page resumes from the saved state.
3. Endurance logger renders a FIT upload input when `fitUploadAdapter` is provided; uploading a valid `.fit` file calls `fitUploadAdapter.upload()` and displays a summary.

**Live mode:**
4. Mount with a strength plan in live mode; rest timer counts down the configured rest duration after each set tick and auto-advances to the next set.
5. Mount with an interval plan; interval display shows the current target (zone/duration) and advances on completion.

**General:**
6. TypeScript strict mode passes; `WorkoutPlan` type is imported from `@process-wizard/workout-editor` (no duplicate definition).
7. No `@formio/js` imports anywhere in the package.

---

## Phase

**Phase C** — logger mode ships first; live mode added in Phase C.2.
