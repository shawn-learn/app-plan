# `pw-rules-engine`

## Purpose

Wraps the GoRules Zen Engine (`zen`) behind a stable, narrow Python API for loading, compiling, and evaluating JDM decision tables. Isolates the `zen` vendor dependency so that the rest of the application never imports `zen` directly. Also handles auto-discovery of JDM JSON files from plugin data directories.

**Not responsible for:** which decision tables exist, what their inputs/outputs mean, plugin discovery, or database access.

---

## Status

**Extract** from `backend/src/process_wizard/engine/decisions.py`.

---

## Source location

```
backend/src/process_wizard/engine/decisions.py   ← single 128-line module
```

The extracted library is one module (`pw_rules_engine/decisions.py`) re-exported from the package root.

---

## Public API / Exports

```python
from pw_rules_engine import DecisionEngine

engine = DecisionEngine(data_dirs=[Path("data/"), Path("plugins/fitness/data/")])

# Evaluate a single-result table (first hit policy)
result: dict = engine.evaluate("risk_stratification", {"risk_score": 3})

# Evaluate a collect-policy table (all matching rows)
results: list[dict] = engine.evaluate("medication_implications",
                                      {"medications": ["metformin"]})

# Introspect loaded tables
names: list[str] = engine.tables()
```

**`DecisionEngine`**
- `__init__(data_dirs: list[Path])` — scans all dirs for `*.json` files that contain a valid JDM graph; compiles and caches each one.
- `evaluate(table_name: str, inputs: dict) -> dict | list[dict]` — dispatches to the named decision table; raises `KeyError` if the table is not loaded. Returns a single dict for `first`/`unique` hit policies, a list for `collect`.
- `tables() -> list[str]` — names of all loaded tables.

---

## Dependencies

**External (pip):**
- `zen-engine` (the GoRules Python SDK)

**Internal pw-* dependencies:** none (does not import domain contracts).

---

## Out of scope

- Plugin discovery — the caller (app startup or `pw-plugin-contracts`) supplies the list of data directories
- Decision table authoring tools
- Any ORM, HTTP, or evaluator logic

---

## Acceptance criteria

1. `pip install pw-rules-engine` installs successfully with only `zen-engine` as a transitive dependency.
2. `from pw_rules_engine import DecisionEngine` works without importing any process_wizard app code.
3. `engine.evaluate("risk_stratification", {...})` returns the same result as the current `decisions.py` call against the same fixture.
4. `engine.evaluate("nonexistent_table", {})` raises `KeyError` with a helpful message listing available table names.
5. Passing two overlapping data directories does not register the same table twice (last-writer-wins or first-wins with a warning).
6. All existing pytest tests for decision-table evaluation pass unchanged.

---

## Phase

**Phase A** — extract alongside `pw-domain-contracts` and `pw-plugin-contracts`.
