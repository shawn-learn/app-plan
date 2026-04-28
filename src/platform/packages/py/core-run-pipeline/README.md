# core-run-pipeline

**Tier 2 — Platform.**

Run lifecycle orchestration: completion, resolver, reference-store writer, analytics projection.

## Source material

Process Wizard `libraries/pw-run-pipeline.md` (currently `services/completion.py`, `resolver.py`, `reference_writer.py`, `analytics.py`).

## Responsibilities

- **Resolver:** before launching a run step, look up its data requirements in the reference store and inject resolved values.
- **Completion:** on run completion, invoke domain evaluators, write computed outputs to the reference store, project analytics.
- **Reference store writer:** index a run's computed outputs keyed by `(user_or_subject, reference_name, version)`.
- **Analytics projection:** push run outcomes to whatever analytics tables the host app maintains.
- **Workflow chaining:** advance multi-document workflows when prerequisites resolve.

## Dependencies

- External: `pydantic`, `sqlalchemy`
- Internal: `core-domain-contracts`, `core-data-access`, `core-rules-engine`, `core-plugin-contracts`, `core-signatures-audit`

## Consumers

`core-api-runtime` mounts run endpoints. Every domain plugin participates by registering evaluators that this pipeline invokes.
