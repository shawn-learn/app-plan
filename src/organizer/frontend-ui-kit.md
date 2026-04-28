# frontend-ui-kit

## 1. Purpose

`frontend-ui-kit` is the shared React component library, design system primitives, hook utilities, API client, and React Query setup that every feature package consumes. It is the only frontend library that knows about MUI directly; feature packages depend on the kit's wrapped components rather than reaching for raw MUI.

## 2. Scope

**In scope**
- Shared components currently under `frontend/src/components/` (e.g., `AgendaDialog.tsx`, `BracketRail.tsx`, `CircularCountdownTimer.jsx`, `DurationInput.jsx`, `DurationRangePicker.jsx`, `ErrorBoundary.jsx`, `ExerciseDialog.tsx`, `GroupDialog.tsx`, `HelpDrawer.jsx`, `ItemRow.tsx`, `JSONToolbar.jsx`, `MarkdownRenderer.jsx`, `MiniDurationSpinner.tsx`, `RecurrenceRuleEditor.jsx`, `Ticker.jsx`).
- The full set of unit-aware range and number inputs (`*RangeInput.jsx`, `NumberOrRangeInput.jsx`, `FloatRangeWithUnit.jsx`, `PaceRangeInput.jsx`, etc.) — these are unit-aware widgets backed by `shared-units`.
- The main layout chrome (`MainLayout.jsx`, `ServerDown.jsx`).
- Theming (`editorTheme.ts`, MUI theme provider).
- API client wrapping `fetch` / `axios` with auth headers; React Query setup; auth helper (`auth.js`).
- Drag-and-drop primitives (`frontend/src/dnd/`).
- Generic types (`frontend/src/types/`).
- Storybook setup (`MiniDurationSpinner.stories.tsx` is a precedent).

**Out of scope**
- Domain-specific pages (workout planner, calendar views, settings) — those are `frontend-features`.
- Backend code.

## 3. Current Capabilities

The current `frontend/src/` is mostly flat. `frontend/src/components/` is the closest thing to a kit today; shared inputs and dialogs live there in mixed `.jsx` and `.tsx` files. `auth.js`, `config.js`, `preferences.js`, `rangeParsers.js`, and `useHelp.js` are de-facto utility modules. UI standards are documented in `docs/UI_STANDARDS.md`.

## 4. Public API Surface

- React components (named exports), grouped by category: layout, forms, dialogs, range inputs, calendar primitives, markdown, dnd, recurrence editor.
- Hooks: `useApi`, `useAuth`, `useHelp`, `useUnit`, `useFormatValue`.
- A typed `apiClient` instance.
- A theme provider and a small set of design tokens.

## 5. Data Model

None. The kit is stateless except for an auth token in memory and React Query's cache.

## 6. Dependencies

- Depends on: `shared-schemas` (for types in props), `shared-units` (for unit-aware inputs). Indirectly depends on backend routes via the API client, but is decoupled from any specific service library.
- Must NOT depend on: `frontend-features` (the kit is a leaf for the frontend).

## 7. Cross-Cutting Concerns

- **Accessibility**: every interactive component must be keyboard-navigable and screen-reader-labeled.
- **Theming**: dark mode (User_Stories future), high-contrast support.
- **Performance**: heavy components (calendar, range editors) must be memoized; the kit owns the perf budget.
- **TypeScript**: kit is the first lib to be fully TS — `.jsx` files migrate to `.tsx` here first.

## 8. Open Questions / Known Gaps

- Mixed `.jsx` / `.tsx` today; full TS migration is a stated goal in the README.
- `*RangeInput.jsx` proliferation (~15 files) suggests a generic `RangeInput<measure>` would replace many of them.
- No Storybook deployment yet; `.stories.tsx` precedent exists for one component only.
- `frontend/src/types/` is partly hand-maintained; should be generated from `shared-schemas` instead.
