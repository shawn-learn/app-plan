# shared-units

## 1. Purpose

`shared-units` provides a uniform model of physical measurements (distance, duration, weight, height, speed, pace, power, energy, temperature, pressure, percentages, etc.), the units they can be expressed in, conversions between those units, and per-user unit preferences. It is the only library that should know how many meters are in a mile or how to convert a pace from min/km to min/mi.

## 2. Scope

**In scope**
- Catalog of measures (the `measures` table) and units (the `units` table) with conversion factors.
- Conversion functions (e.g., `convert(value, from_unit, to_unit, measure)`).
- Range parsing (`frontend/src/rangeParsers.js` equivalent on backend).
- Strength-related calculations that depend on units: estimated 1RM, volume, work, intensity ranges (`app/services/strength.py`).
- Per-user default-unit lookup (delegated to `user-service` for the preference, but the resolution logic lives here).
- Frontend equivalents: parsers, formatters, and React hooks for unit-aware inputs.

**Out of scope**
- The user record itself (owned by `user-service`).
- The activity payload that *uses* a value (owned by `activity-service`).
- UI input components — those live in `frontend-ui-kit` but consume helpers from this lib.

## 3. Current Capabilities

The `measures` and `units` tables already define the catalog with conversion factors. `app/services/preferences.py` handles per-user unit selection. `app/services/strength.py` performs the strength-domain math that depends on units. The frontend has many `*RangeInput.jsx`, `*RangeWithUnit.jsx`, and parser files (`rangeParsers.js`) that all touch units.

## 4. Public API Surface

- Backend: `convert()`, `format(value, unit, locale)`, `parse_range(str, measure)`, `default_unit_for(user, measure)`, `estimate_1rm(weight, reps)`, `volume(sets)`.
- Frontend: equivalent hooks (`useUnit(measure)`, `useFormatValue`), parser utilities, range/unit-aware MUI input adapters.
- Read-only access to the measures/units catalog via a small endpoint (used to populate dropdowns).

## 5. Data Model

Owns: `measures`, `units`. Reads from: `users` (for unit preferences) and `user_unit_preferences` (if present); does not write to user-owned tables — that goes through `user-service`.

## 6. Dependencies

- Depends on: `shared-schemas` (for the measurement schema).
- Must NOT depend on: any Tier 2 service library. `user-service` depends on this lib, not the other way around — unit preferences are stored in user-owned tables but the *interpretation* of a unit is a `shared-units` concern.

## 7. Cross-Cutting Concerns

- **Locale**: formatting must respect locale-specific decimal separators and unit conventions.
- **Precision**: conversions must round-trip without lossy drift; conversion factors are stored as exact rationals where possible.
- **Validation**: invalid unit/measure combinations must raise a typed error caught at the API boundary.

## 8. Open Questions / Known Gaps

- Unit-preference storage is currently spread across `users` and possibly `user_unit_preferences`; a single source of truth needs documenting.
- No central catalog of all `*RangeInput.jsx` components — there are ~15 of them. Unifying them via a generic `RangeInput<measure>` is a likely cleanup.
- Pace ↔ speed conversions are done in multiple places (`PaceRangeInput`, `SpeedPaceRangeInput`, backend); duplication needs collapsing.
