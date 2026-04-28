# `@process-wizard/web-plugin-ui`

## Purpose

Provides the frontend plugin component registry: a runtime mapping from plugin domain names to lazily-loaded React components (summary views, profile sections, session viewers). Mirrors the backend plugin system on the frontend. A host application registers plugin component bundles at startup; individual features look up components by plugin name at render time without hard-coding imports.

**Not responsible for:** plugin component implementations (those live in `frontend/src/plugins/fitness/` and future domain packages), plugin discovery on the backend, or any data fetching.

---

## Status

**Extract** from `frontend/src/plugins/registry.ts` and the plugin component files. **Deferred — not in scope for Phases A-C.**

---

## Source location

```
frontend/src/plugins/
├── registry.ts                       ← plugin component registry, lazy-load map
├── fitness/FitnessIntakeSummary.tsx
├── fitness/BodyCompositionScreeningSummary.tsx
└── fitness/FitnessProfile.tsx
```

---

## Public API / Exports

```typescript
export { PluginRegistry, PluginRegistryProvider } from "./registry";
export type { PluginComponentMap, PluginSlot } from "./registry";

export { usePluginComponent } from "./registry";
// Usage: const SummaryComponent = usePluginComponent("fitness_intake", "summary");
```

### `PluginComponentMap`

```typescript
type PluginComponentMap = Record<
  string,          // plugin key, e.g. "fitness_intake"
  {
    summary?: React.LazyExoticComponent<any>;
    profile?: React.LazyExoticComponent<any>;
    session_viewer?: React.LazyExoticComponent<any>;
  }
>;
```

---

## Dependencies

**External (npm):** `react ^18 || ^19` (peer, for `React.lazy` + `Suspense`)

**Internal pw-* dependencies:** none — the registry is a pure lookup mechanism.

---

## Out of scope

- The plugin component implementations themselves (fitness summary, profile, etc.) — those ship with their respective domain plugins
- Backend plugin discovery — that is `pw-plugin-contracts`
- Any data fetching or API calls

---

## Acceptance criteria

1. `usePluginComponent("fitness_intake", "summary")` returns the `FitnessIntakeSummary` component when the fitness plugin is registered.
2. `usePluginComponent("unknown_plugin", "summary")` returns `null` without throwing.
3. Plugin components are loaded lazily (no upfront bundle cost for unregistered plugins).
4. `PluginRegistryProvider` can be initialized with an empty map without errors.

---

## Phase

**Deferred** — the current `plugins/registry.ts` is adequate for the single-app monorepo. Extract when a second frontend application needs to mount fitness or other domain plugin components.
