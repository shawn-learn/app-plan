# core-plugin-contracts

**Tier 2 — Platform.**

The plugin extension boundary. Defines `PluginSpec`, evaluator/enricher registries, and discovery protocol.

## Source material

Process Wizard `libraries/pw-plugin-contracts.md` (currently `plugin.py` + evaluator/enricher registries).

## Responsibilities

- `PluginSpec` Pydantic model (name, version, data-dir, evaluators, enrichers, frontend-component metadata).
- Decorator-based registration: `@evaluator("name")`, `@enricher("name")`.
- Entry-point–based plugin discovery (Python `importlib.metadata`).
- Bootstrap lifecycle: `discover → validate → register → activate`.

## Dependencies

- External: `pydantic`
- Internal: `core-domain-contracts`

## Consumers

`core-run-pipeline`, `core-rules-engine`, every `domain-*` plugin.

## Notes

Discovery side-effects are explicit and ordered — no import-time registration magic. Plugins that fail validation are reported, not silently dropped.
