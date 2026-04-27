# web-form-editors

**Tier 3 — Frontend platform.**

Drag-and-drop Form.io builder + custom field components. The only library already extracted in the source projects.

## Source material

Existing Process Wizard `@process-wizard/form-editors` package (in `packages/form-editors/`).

## Responsibilities

- Form.io builder UI with custom-field palette.
- Custom fields:
  - Unit-aware numeric inputs (uses `core-units`).
  - Ranking sliders.
  - Weekly availability grids.
  - Range pickers.
  - Signature capture.
  - All step-field types from PW `system-requirements.md` §5.3.1.
- Export to Form.io JSON schema consumable by `web-wizard-runtime`.
- Host-injected adapters (no direct host coupling — already polyrepo-clean).

## Dependencies

- External: `@formio/js`, `@formio/react`, `react`
- Internal: `web-ui-kit`, `core-units`, `core-schemas-ts`

## Consumers

`web-wizard-runtime` (renderers), authoring screens in app shells (builder).
