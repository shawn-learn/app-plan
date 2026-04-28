# `@process-wizard/form-editors`

## Purpose

Provides a drag-and-drop Form.io form builder and a set of custom Form.io field components that the host application uses to let practitioners author wizard schemas. The library knows nothing about how forms are stored, what units the host uses, or where the resulting JSON goes — all of those concerns are injected via adapters.

**Not responsible for:** wizard execution (rendering a form for a respondent), document versioning, user authentication, or database access.

---

## Status

**Exists.** Lives at `packages/form-editors/`. Consumed as a workspace dependency by the frontend app.

---

## Source location

```
packages/form-editors/
├── src/
│   ├── index.ts                      ← public barrel
│   ├── adapters/
│   │   ├── FormStorageAdapter.ts     ← persistence contract (host implements)
│   │   └── UnitRegistry.ts          ← unit category definitions (host injects)
│   ├── builder/
│   │   ├── FormBuilderPage.tsx       ← main UI component
│   │   ├── types.ts                  ← BuilderState, FieldConfig variants, etc.
│   │   ├── hooks/useFormBuilderState.ts
│   │   ├── utils/schemaGenerator.ts ← BuilderState → Form.io JSON
│   │   └── utils/schemaParser.ts    ← Form.io JSON → BuilderState
│   └── components/
│       ├── registerCustomComponents.ts
│       ├── UnitNumberInput.tsx
│       ├── RankingSliderInput.tsx
│       ├── WeeklyAvailabilityGrid.tsx
│       ├── RangeInput.tsx
│       └── parseRange.ts
└── package.json
```

---

## Public API / Exports

```typescript
// Component
FormBuilderPage           // props: FormBuilderPageProps
FormBuilderPageProps      // { adapter, unitRegistry?, initialState?, ... }

// Custom Form.io field registration
registerCustomComponents  // (options?: RegisterCustomComponentsOptions) => void
RegisterCustomComponentsOptions

// Unit registry
UnitRegistryProvider, useUnitRegistry, createUnitRegistry
setDefaultUnitRegistry, getDefaultUnitRegistry, EMPTY_UNIT_REGISTRY
UnitRegistry, UnitCategoryDef, UnitDef, UnitRegistryProviderProps

// Storage adapter (interface only — host implements)
FormStorageAdapter        // { load(id): Promise<FormRecord>; create(state): Promise<FormRecord>; update(id, state): Promise<FormRecord> }
FormRecord

// Builder state shapes
FieldType, BuilderField, BuilderStep, BuilderState
FieldConfig, TextAreaConfig, RadioConfig, SelectConfig, NumberConfig,
CheckboxConfig, SignatureConfig, DateTimeConfig, HtmlElementConfig,
RankingSliderConfig, WeekCalendarConfig, RangeConfig
FieldCondition, Condition

// Schema round-trip
generateFormioSchema      // (state: BuilderState) => FormioSchema
parseFormioSchema         // (schema: FormioSchema) => BuilderState

// Runtime value types
UnitNumberValue
RangeValue, ParseRangeOptions, ParseRangeResult
parseRange, formatRangeValue
```

---

## Dependencies

**External (npm):**
- `@dnd-kit/core`, `@dnd-kit/sortable`, `@dnd-kit/utilities` — drag-and-drop for field reordering
- `@tiptap/react`, `@tiptap/starter-kit`, `@tiptap/extension-placeholder` — rich-text editing for HTML element fields
- `nanoid` — generating stable field keys

**Peer dependencies (host must supply):**
- `@formio/js ^5.3.1`, `@formio/react ^6.2.1-api98.0`
- `react ^18 || ^19`, `react-dom`

**Internal pw-* dependencies:** none.

---

## Out of scope

- Wizard execution (rendering a form for a respondent to fill in) — that is `@process-wizard/web-wizard-runtime`
- Document lifecycle (publish, version, branch) — belongs in the host application
- Any knowledge of how completed form data is stored or evaluated

---

## Acceptance criteria

1. `npm run build` in `packages/form-editors/` produces a clean TypeScript output with no errors.
2. Host application can import `FormBuilderPage` and mount it with only a `FormStorageAdapter` implementation; no other app state is required.
3. Round-trip: calling `generateFormioSchema(parseFormioSchema(fixture))` returns a schema that Form.io renders identically to the original fixture.
4. All four custom fields (`UnitNumberInput`, `RankingSliderInput`, `WeeklyAvailabilityGrid`, `RangeInput`) are visible in the builder palette and produce the correct Form.io component key in the exported schema.
5. `registerCustomComponents()` can be called before Form.io initialization without errors.

---

## Phase

**Existing** — no extraction work required. Requirements documented here as the canonical reference for its contract.
