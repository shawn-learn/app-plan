# web-wizard-runtime

**Tier 3 — Frontend platform.**

Form.io wizard execution UX: bundle load, run create/resume, autosave, completion/cancel.

## Source material

Process Wizard `libraries/web-wizard-runtime.md` (`frontend/src/components/wizard/*`, `useStylesheet.ts`).

## Responsibilities

- Render a Form.io schema as a multi-step wizard.
- Run create/resume flows backed by `web-api-client`.
- Autosave with offline buffering (coordinates with `core-sync` server endpoints).
- Custom field renderers registered from `web-form-editors`.
- Completion / cancel UX, including signature capture handoff.
- Stylesheet hot-loading per tenant/brand.

## Dependencies

- External: `react`, `@formio/react`
- Internal: `web-api-client`, `web-auth-session`, `web-ui-kit`, `web-form-editors`, `core-schemas-ts`

## Consumers

Every app shell — the wizard is the unifying primitive across all three apps.
