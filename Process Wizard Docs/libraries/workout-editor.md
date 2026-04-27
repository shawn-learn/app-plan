# `@process-wizard/workout-editor`

## Purpose

A bespoke React component library for authoring workout plans. Plans are composed of blocks (warm-up, working blocks, cool-down), where each block contains exercises with modality-specific parameters: sets/reps/load/RPE for strength, HR/pace zone × duration/distance for endurance, or interval ladders. The editor outputs a `WorkoutPlan` JSON document. It does not use Form.io.

Mirrors the architecture of `@process-wizard/form-editors`: all storage and exercise-catalog concerns are injected via adapters; the component has zero knowledge of the host application.

**Not responsible for:** executing or logging sessions (that is `@process-wizard/workout-runtime`), user authentication, database access, or rendering completed records.

---

## Status

**New** — no existing source. Mirrors the adapter pattern established by `packages/form-editors/`.

---

## Source location

New package at:

```
packages/workout-editor/
├── package.json
└── src/
    ├── index.ts                      ← public barrel
    ├── adapters/
    │   ├── WorkoutStorageAdapter.ts  ← load/save WorkoutPlan (host implements)
    │   └── CatalogAdapter.ts        ← exercise lookup injection
    ├── editor/
    │   ├── WorkoutEditorPage.tsx     ← top-level component
    │   ├── PlanCanvas.tsx            ← block list + add/reorder/remove
    │   ├── BlockEditor.tsx           ← per-block editing (type-switched)
    │   ├── blocks/
    │   │   ├── StrengthBlock.tsx     ← sets × reps × load × RPE, supersets
    │   │   └── EnduranceBlock.tsx    ← zone × duration/distance; intervals
    │   ├── hooks/useEditorState.ts
    │   └── utils/
    │       ├── planValidator.ts
    │       └── formioExporter.ts     ← optional: compile plan → Form.io schema
    └── types.ts                      ← WorkoutPlan, Block, Exercise*, etc.
```

---

## Public API / Exports

```typescript
// Top-level component
export { WorkoutEditorPage } from "./editor/WorkoutEditorPage";
export type { WorkoutEditorPageProps } from "./editor/WorkoutEditorPage";

// Adapters (interfaces for host to implement)
export type { WorkoutStorageAdapter, WorkoutRecord } from "./adapters/WorkoutStorageAdapter";
export type { CatalogAdapter } from "./adapters/CatalogAdapter";

// WorkoutPlan document schema (the output format)
export type {
  WorkoutPlan,
  PlanBlock,
  StrengthBlock,
  EnduranceBlock,
  StrengthSet,
  IntervalStep,
  ZoneTarget,
  LoadTarget,
  BlockType,
} from "./types";

// Optional Form.io schema export
export { exportToFormio } from "./editor/utils/formioExporter";
export type { FormioExportOptions } from "./editor/utils/formioExporter";

// Validation
export { validateWorkoutPlan } from "./editor/utils/planValidator";
export type { PlanValidationResult } from "./editor/utils/planValidator";
```

### `WorkoutPlan` shape

```typescript
interface WorkoutPlan {
  id: string;
  title: string;
  sport: Sport;           // from @process-wizard/exercise-catalog
  blocks: PlanBlock[];
  metadata: {
    created_at: string;   // ISO 8601
    updated_at: string;
    schema_version: "1.0";
  };
}

type PlanBlock =
  | { type: "strength";  order: number; label?: string; sets: StrengthSet[] }
  | { type: "endurance"; order: number; label?: string; target: ZoneTarget | IntervalStep[] }
  | { type: "rest";      order: number; label?: string; duration_seconds: number };
```

### `WorkoutStorageAdapter`

```typescript
interface WorkoutStorageAdapter {
  load(id: string): Promise<WorkoutRecord>;
  create(plan: WorkoutPlan): Promise<WorkoutRecord>;
  update(id: string, plan: WorkoutPlan): Promise<WorkoutRecord>;
  list(): Promise<WorkoutRecord[]>;
}
```

---

## Dependencies

**External (npm):**
- `@dnd-kit/core`, `@dnd-kit/sortable` — block reordering (same as form-editors)
- `nanoid` — block/set ID generation

**Peer dependencies:**
- `react ^18 || ^19`, `react-dom`

**Internal pw-* dependencies:**
- `@process-wizard/exercise-catalog` — exercise picker and load-schema awareness

**Does NOT depend on:**
- `@formio/js` or `@formio/react` — bespoke UI only; Form.io export is an optional output transform, not a rendering dependency

---

## Out of scope

- Session logging or live execution — `@process-wizard/workout-runtime`
- Any Form.io rendering of the resulting plan (the optional `exportToFormio` only converts the JSON; rendering is still the host's responsibility)
- User account management, permissions, or document lifecycle

---

## Acceptance criteria

1. `WorkoutEditorPage` mounts with only a `WorkoutStorageAdapter` and `CatalogAdapter`; no other app context required.
2. A practitioner can build a strength plan with ≥2 blocks, each with ≥1 set (including a superset group), save it, reload it from storage, and see an identical plan rendered.
3. A practitioner can build an endurance block with zone × duration targets and an interval ladder with repeats.
4. `exportToFormio(plan)` produces a Form.io schema that renders in a Form.io wizard with inputs for each set/rep field.
5. `validateWorkoutPlan(plan)` returns errors for plans with empty blocks or missing required fields.
6. TypeScript strict mode passes; no `any` in public API.
7. No `@formio/js` or `@formio/react` imports in the component tree (only in `formioExporter.ts`).

---

## Phase

**Phase B** — second Phase B deliverable (after `@process-wizard/exercise-catalog`).
