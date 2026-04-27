# `@process-wizard/exercise-catalog`

## Purpose

A TypeScript library that provides a canonical catalog of exercises and movements — lifts, running variants, cycling workouts, swims, and bodyweight movements — with typed accessors for lookup, filtering, and browsing. The catalog ships as package-bundled JSON; hosts can extend it with domain-specific or custom exercises via an injection hook.

Used by `@process-wizard/workout-editor` to populate the exercise picker and by `@process-wizard/workout-runtime` to render exercise names, load units, and instructions. Also used server-side (Python evaluators import the catalog data as JSON fixtures) to validate workout plan contents.

**Not responsible for:** workout plan structure (that is the editor), execution/logging UI, user preferences, or storing exercises in a database.

---

## Status

**New** — no existing source. Shape is informed by the structured data patterns already visible in `backend/src/process_wizard/plugins/fitness/data/` JSON files.

---

## Source location

New package at:

```
packages/exercise-catalog/
├── package.json
└── src/
    ├── index.ts             ← public barrel
    ├── types.ts             ← Exercise, MuscleGroup, Equipment, Sport, UnitSchema
    ├── catalog.ts           ← CatalogAccessor class + createCatalog()
    ├── data/
    │   ├── strength.json    ← ≥50 strength exercises (Phase B seed)
    │   ├── running.json     ← running variants (intervals, tempo, easy)
    │   └── cycling.json     ← cycling variants
    └── hooks/
        └── useCatalog.ts    ← React hook for catalog access in components
```

---

## Public API / Exports

```typescript
// Types
export interface Exercise {
  id: string;                      // stable slug, e.g. "barbell-back-squat"
  name: string;
  sport: Sport;                    // "strength" | "running" | "cycling" | "swimming" | "other"
  primaryMuscleGroups: MuscleGroup[];
  secondaryMuscleGroups: MuscleGroup[];
  equipment: Equipment[];
  loadSchema: LoadSchema;          // defines valid units for this exercise
  instructions?: string;
  aliases?: string[];              // alternate search terms
}

export type Sport = "strength" | "running" | "cycling" | "swimming" | "other";

export interface LoadSchema {
  type: "reps_load" | "duration_hr" | "duration_pace" | "duration_power" | "bodyweight";
  primaryUnit: string;             // e.g. "kg", "min:sec/km", "watts"
  alternateUnits?: string[];
}

// Catalog accessor
export class CatalogAccessor {
  getById(id: string): Exercise | undefined;
  listBySport(sport: Sport): Exercise[];
  listByMuscleGroup(group: MuscleGroup): Exercise[];
  search(query: string): Exercise[];           // fuzzy name + alias match
  all(): Exercise[];
}

// Factory: creates a catalog from the built-in seed + optional host overrides
export function createCatalog(overrides?: Exercise[]): CatalogAccessor;

// React hook
export function useCatalog(): CatalogAccessor;
export function CatalogProvider(props: { catalog: CatalogAccessor; children: ReactNode }): JSX.Element;
```

---

## Dependencies

**External (npm):**
- none (plain TypeScript + React peer dep only)

**Peer dependencies:**
- `react ^18 || ^19` (for `useCatalog` hook — optional, tree-shakable)

**Internal pw-* dependencies:** none.

---

## Out of scope

- Storing exercises in a user database — the catalog is static; custom exercises are injected in-memory
- Exercise video/image hosting
- Progressive overload algorithms or load progression rules — those belong in `pw-training-metrics` or workout-editor logic
- Any backend/Python catalog access — Python evaluators read the raw JSON fixtures directly

---

## Acceptance criteria

1. `createCatalog().listBySport("strength")` returns ≥50 exercises in the Phase B seed.
2. `catalog.getById("barbell-back-squat")` returns an exercise with `loadSchema.type === "reps_load"`.
3. `catalog.search("squat")` returns results that include both "barbell-back-squat" and "goblet-squat".
4. `createCatalog([{ id: "custom-exercise", ... }])` includes the custom exercise in `catalog.all()`.
5. TypeScript strict mode passes with no `any` usage in the public API.
6. The package builds with no external runtime dependencies (only React as optional peer).

---

## Phase

**Phase B** — first Phase B deliverable; required by `@process-wizard/workout-editor`.
