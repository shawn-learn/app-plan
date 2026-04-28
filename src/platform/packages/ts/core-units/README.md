# core-units (TypeScript)

**Tier 0 — Foundation.**

TS twin of `packages/py/core-units/`. Same unit definitions, conversions, formatters; runs in the browser.

## Responsibilities

- Conversion helpers (mass, distance, duration, pace, power, HR).
- Locale-aware display formatters.
- User-preference resolution helpers.

## Dependencies

- External: none
- Internal: `core-schemas-ts`

## Consumers

`web-ui-kit`, `web-features-shared`, every domain TS plugin.

## Notes

Conversion math is mirrored from the Python package via shared fixtures to prevent drift.
