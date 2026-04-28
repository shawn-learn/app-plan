# `pw-run-pipeline`

## Purpose

Encapsulates the application-layer orchestration for the run lifecycle: completing a run (evaluator dispatch, output computation), resolving data requirements from the reference store before a run starts, writing reference index entries after completion, and projecting run data into the analytics table. This is the high-value, domain-agnostic business logic that every host application deploying ProcessWizard needs.

**Not responsible for:** HTTP routing (that is `pw-api-runtime`), ORM definitions (that is `pw-data-access`), domain evaluators (those are in domain plugins), or decision-table logic (that is `pw-rules-engine`).

---

## Status

**Extract** from `backend/src/process_wizard/services/`. **Deferred — not in scope for Phases A-C.**

---

## Source location

```
backend/src/process_wizard/services/
├── completion.py          ← run completion orchestrator
├── resolver.py            ← data requirement resolution
├── reference_writer.py    ← batch-write reference_index entries from output_data
└── analytics.py           ← denormalize run data into run_analytics table
```

---

## Public API / Exports

```python
from pw_run_pipeline import complete_run, resolve_data_requirements, write_references, project_analytics

async def complete_run(
    session: AsyncSession,
    run_id: str,
    decision_engine: DecisionEngine,
) -> CompletionResult:
    """Fetch run → evaluate output → update status → write references → project analytics."""

async def resolve_data_requirements(
    session: AsyncSession,
    document: DocumentRecord,
    user_id: str,
) -> dict[str, Any]:
    """Resolve user profile fields, reference lookups, and past data for a document's dataRequirements."""

async def write_references(
    session: AsyncSession,
    run: Run,
    output_data: dict[str, Any],
) -> None:
    """Write reference_index entries from output_data using the document's referenceable spec."""

async def project_analytics(
    session: AsyncSession,
    run: Run,
    output_data: dict[str, Any],
) -> None:
    """Flatten collected_data into the run_analytics table for time-series queries."""
```

---

## Dependencies

**External (pip):** none (delegates to injected session and engine)

**Internal pw-* dependencies:**
- `pw-domain-contracts` — `Run`, `DocumentRecord`, `CompletionResult`
- `pw-data-access` — `AsyncSession`, repository functions
- `pw-rules-engine` — `DecisionEngine` passed as a parameter
- `pw-plugin-contracts` — `evaluate()` dispatcher for evaluator dispatch

---

## Out of scope

- HTTP request handling — `pw-api-runtime`
- Plugin discovery — that happens at app startup before the pipeline is invoked
- Training-load or fitness-specific computation — domain evaluators/enrichers do that

---

## Acceptance criteria

1. `complete_run()` produces the same `CompletionResult` as the current `services/completion.py` for a set of known run fixtures.
2. `resolve_data_requirements()` correctly resolves reference-index lookups for the fitness intake document.
3. All `pytest` integration tests covering run completion pass unchanged.
4. No imports from `fastapi`, `starlette`, or any HTTP framework.

---

## Phase

**Deferred** — does not block Phase A-C work. Extract when a second deployment or CLI tooling needs run-pipeline logic without the HTTP layer.
