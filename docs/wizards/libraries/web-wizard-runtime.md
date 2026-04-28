# `@process-wizard/web-wizard-runtime`

## Purpose

Provides the React component that drives the Form.io wizard lifecycle: loading a wizard bundle from the API, creating or resuming a run, rendering the multi-step form with autosave, and submitting the completed run. Can be embedded in any host application that wants to present a ProcessWizard document to a user without re-implementing the run lifecycle.

**Not responsible for:** form schema authoring (that is `@process-wizard/form-editors`), completion summary display, user authentication, or workout sessions (those use `@process-wizard/workout-runtime`).

---

## Status

**Extract** from `frontend/src/components/wizard/`. **Deferred — not in scope for Phases A-C.**

---

## Source location

```
frontend/src/components/wizard/
├── DocumentWizard.tsx         ← main wizard component
├── AssignmentConfirmation.tsx ← pre-run assignment acknowledgment screen
└── ...
frontend/src/hooks/useStylesheet.ts  ← dynamic stylesheet loading for wizard theming
```

---

## Public API / Exports

```typescript
export { DocumentWizard } from "./DocumentWizard";
export type { DocumentWizardProps } from "./DocumentWizard";

export { useStylesheet } from "./useStylesheet";
```

### `DocumentWizardProps`

```typescript
interface DocumentWizardProps {
  documentId: string;
  runId?: string;              // if provided, resumes an existing run
  apiClient: ApiClient;        // injected
  onComplete: (runId: string) => void;
  onAbandon?: () => void;
}
```

---

## Dependencies

**External (npm):** `@formio/js`, `@formio/react` (peer), `react ^18 || ^19` (peer)

**Internal pw-* dependencies:**
- `@process-wizard/web-api-client` — fetches wizard bundle, creates/updates/completes run
- `@process-wizard/form-editors` — `registerCustomComponents()` called on mount so custom fields work in the runtime

---

## Out of scope

- Rendering post-completion summaries (host app decides what to show after `onComplete` fires)
- Stylesheet management beyond loading and injecting for the current wizard
- Workout plan execution — use `@process-wizard/workout-runtime` for that

---

## Acceptance criteria

1. `DocumentWizard` mounts with a `documentId` and `apiClient`; creates a new run, renders the Form.io wizard, and calls `onComplete` when submitted.
2. Autosave: collected data is persisted on step navigation so a page reload resumes from the current step.
3. Custom Form.io fields (unit number, ranking slider, etc.) render correctly without the host calling `registerCustomComponents` manually.
4. If `runId` is provided and the run is `in_progress`, the wizard resumes at the correct step with previously entered data.

---

## Phase

**Deferred** — the current `components/wizard/` is adequate for the single-app monorepo. Extract when a second frontend app needs to embed the wizard execution UX.
