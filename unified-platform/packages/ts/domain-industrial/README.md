# domain-industrial (TypeScript)

**Domain plugin — Industrial procedure execution (frontend).**

Twin of `packages/py/domain-industrial/`. Registers industrial-specific frontend components into `web-plugin-ui`.

## Responsibilities

- Industrial step-field renderers beyond the generic step catalog (decision-branch with target-step navigation, deviation note inline, equipment-ref dropdowns sourcing from the industrial reference catalog).
- SOP authoring screens (procedure browser, procedure editor wrapper around `web-form-editors`).
- Run history & canonical-record viewer (signature verification UX from `web-features-shared`, plus industrial metadata).
- Compliance dashboards.
- Tenant operator console (uses `core-tenancy` admin endpoints).

## Dependencies

- External: `react`, `react-router-dom`
- Internal: `web-ui-kit`, `web-api-client`, `web-auth-session`, `web-wizard-runtime`, `web-plugin-ui`, `web-form-editors`, `web-features-shared`, `core-schemas-ts`

## Consumers

`app-industrial` only.
