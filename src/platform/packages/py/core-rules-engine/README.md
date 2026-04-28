# core-rules-engine

**Tier 2 — Platform.**

GoRules Zen Engine wrapper. Loads, compiles, and executes JDM decision tables from plugin data directories.

## Source material

Process Wizard `libraries/pw-rules-engine.md` (currently `engine/decisions.py`).

## Responsibilities

- Auto-discovery of JDM JSON files in plugin data dirs.
- Compile and cache decision tables.
- Stable narrow API: `evaluate(table_name, input) → output`, `tables() → list`.
- Optional plugin data-dir lookup hook (so plugins register their own tables).

## Dependencies

- External: `zen` (GoRules)
- Internal: `core-plugin-contracts` (for the data-dir lookup hook)

## Consumers

`core-run-pipeline` (resolver invokes decision tables during run completion). Domain plugins ship their own JDM files which this engine discovers.

## Notes

The Zen vendor coupling is isolated to this package by design — swapping decision backends is a single-package change.
