# `pw-plugin-contracts`

## Purpose

Defines the formal extension boundary between the ProcessWizard platform and domain-specific plugins. It provides `PluginSpec` (what a plugin declares), the evaluator registry (`@evaluator` decorator and `evaluate()` dispatcher), the enricher registry (`@enricher` decorator and `get_all()` / `get_enrichers()` accessors), and the plugin discovery lifecycle (`discover_plugins`, `get_data_dirs`, `get_seed_manifests`, `get_frontend_components`).

Any package that wants to contribute evaluators or enrichers to the platform depends on this library. The platform runtime imports it to drive discovery and dispatch. Plugins never depend on the app.

**Not responsible for:** the actual evaluator logic for any domain (that lives in `plugins/fitness/` or future domain packages), database access, HTTP, or frontend rendering.

---

## Status

**Extract** from three modules:

```
backend/src/process_wizard/plugin.py
backend/src/process_wizard/services/evaluators/__init__.py
backend/src/process_wizard/services/enrichers/__init__.py
```

---

## Source location

After extraction the package layout is:

```
pw_plugin_contracts/
├── __init__.py          ← re-exports everything below
├── spec.py              ← PluginSpec dataclass + discover_plugins, get_data_dirs,
│                           get_seed_manifests, get_frontend_components
├── evaluators.py        ← EvaluatorContext, EvaluatorFn, @evaluator decorator,
│                           register(), evaluate(), available()
└── enrichers.py         ← EnricherFn, @enricher decorator,
                            register(), get_all(), get_enrichers(), available()
```

---

## Public API / Exports

### Plugin spec and discovery

```python
from pw_plugin_contracts import PluginSpec, discover_plugins
from pw_plugin_contracts import get_plugins, get_data_dirs, get_seed_manifests, get_frontend_components

@dataclass
class PluginSpec:
    name: str
    version: str
    data_dir: Path
    evaluator_modules: list[str] = ...
    enricher_modules: list[str] = ...
    seed_manifest: str | None = None
    frontend_components: dict[str, str] = ...
    labels: dict[str, str] = ...
    default_tags: dict[str, str] = ...

discover_plugins() -> dict[str, PluginSpec]
get_plugins()      -> dict[str, PluginSpec]
get_data_dirs()    -> list[Path]
get_seed_manifests() -> list[tuple[str, Path]]
get_frontend_components() -> dict[str, str]
```

Plugin discovery scans `plugins/` subdirectories for `pw_plugin.py` files and also checks the `process_wizard.plugins` entry-point group for pip-installed plugins. The `PW_PLUGINS` env var filters which plugins load.

### Evaluator registry

```python
from pw_plugin_contracts import evaluator, register_evaluator, evaluate, available_evaluators
from pw_plugin_contracts import EvaluatorContext

@evaluator("fitness_intake")
async def my_evaluator(collected_data, decision_engine, *, ctx=None):
    ...
    return output_data, summary

# Or imperatively:
register_evaluator("name", fn)

# Dispatch (called by run completion):
output_data, summary = await evaluate("fitness_intake", data, engine, ctx)

available_evaluators() -> list[str]
```

`EvaluatorContext` carries `session: AsyncSession | None`, `user_id: str | None`, `document_id: str | None`, `completed_at: datetime | None`.

### Enricher registry

```python
from pw_plugin_contracts import enricher, register_enricher, get_all_enrichers, get_enrichers, available_enrichers

@enricher("fitness")
def my_enricher(grouped_refs: dict[str, dict], flat_refs: dict) -> dict:
    return {"derived_field": ...}

register_enricher("name", fn)
get_all_enrichers()        -> dict[str, EnricherFn]
get_enrichers(["fitness"]) -> list[EnricherFn]
available_enrichers()      -> list[str]
```

---

## Dependencies

**External (pip):**
- stdlib only (dataclasses, importlib, pathlib, typing)

**Internal pw-* dependencies:**
- `pw-domain-contracts` — `EvaluatorContext` references `AsyncSession` (SQLAlchemy type annotation only; may use `TYPE_CHECKING` guard to avoid a hard runtime dependency)
- `pw-rules-engine` — `evaluate()` dispatcher signature includes `DecisionEngine` as a parameter type

---

## Out of scope

- Any concrete evaluator or enricher implementations — those live in domain plugins
- Frontend component rendering — frontend plugin registry is `@process-wizard/web-plugin-ui`
- Seed data loading and database seeding — the app handles that using the manifests returned by `get_seed_manifests()`

---

## Acceptance criteria

1. `pip install pw-plugin-contracts` installs successfully.
2. A minimal plugin (`PluginSpec` + `@evaluator` decorator) can be written with no imports from `process_wizard.*`.
3. `discover_plugins()` finds and loads the existing fitness plugin when called from the app; all evaluators/enrichers registered by the fitness plugin appear in `available_evaluators()` / `available_enrichers()`.
4. `evaluate("fitness_intake", data, engine)` returns the same result as the current inline dispatch.
5. `PW_PLUGINS=fitness` env var loads only the fitness plugin; other plugins in the `plugins/` directory are skipped.
6. All existing pytest tests that invoke evaluators or enrichers pass unchanged.

---

## Phase

**Phase A** — extract alongside `pw-domain-contracts` and `pw-rules-engine`.
