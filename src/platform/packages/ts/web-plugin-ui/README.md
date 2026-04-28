# web-plugin-ui

**Tier 3 — Frontend platform.**

Lazy-loaded plugin component registry. Mirrors the backend `core-plugin-contracts` extension model on the frontend.

## Source material

Process Wizard `libraries/web-plugin-ui.md` (`frontend/src/plugins/registry.ts`).

## Responsibilities

- Plugin component manifest: `(plugin_name, slot_name) → React.lazy(() => import(...))`.
- Slot rendering helpers: `<PluginSlot name="fitness:profile" {...props} />`.
- Lazy-load fallback and error boundaries.
- Naming-contract types shared with backend plugin specs.

## Dependencies

- External: `react`
- Internal: `core-schemas-ts`

## Consumers

App shells declare which plugins are active; this package resolves slots at render time. Domain TS packages register their components against this registry.
