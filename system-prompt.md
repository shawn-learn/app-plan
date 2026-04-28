---
Status: Approved
Last Updated: 2026-04-28
Relates to: [01-metabolic-core]
---
# System Prompt: Metabolic Testing Source of Truth

## Purpose
This file is the canonical context map for AI coding agents working in this repository.

## Domain Truth Hierarchy
1. Safety constraints and protocol assumptions in this file.
2. Milestone specifications in `milestones/`.
3. Technical schemas in `specs/`.
4. Implementation details in `src/`.

## Metabolic Testing Formula Baselines
- **VO2max estimation**: Must be tied to the selected protocol (e.g., Cooper, Bruce, or lab-graded treadmill) and stored with `protocol_id` and `estimation_method`.
- **Lactate threshold**: Determined from stage-based lactate sampling and pace/power breakpoints; retain both raw measurements and computed threshold markers.
- **Derived metrics**: Every computed metric must include provenance fields (`formula_version`, `input_sample_ids`, `computed_at`).

## Data Integrity Rules
- Never overwrite raw test measurements.
- Derived metrics must be reproducible from stored inputs.
- Unit conversions must be explicit and auditable.

## Architecture Mapping
- Domain logic: `src/platform/packages/py/domain-fitness/` and `src/platform/packages/ts/domain-fitness/`
- Shared contracts: `src/platform/packages/py/core-domain-contracts/`
- Wizard and protocol flows: `docs/wizards/`

## AI Agent Working Rules
- Prefer schema-first changes.
- Keep domain-specific logic out of platform core packages.
- Add or update tests with every formula or contract change.
