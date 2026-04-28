# domain-industrial (Python)

**Domain plugin — Industrial procedure execution.**

Backend half of the industrial domain. Has a TS twin at `packages/ts/domain-industrial/`.

## Source material

Process Wizard `system-requirements.md` (esp. §5.3 step catalog and §5.1 access/audit), `user-management-service-requirements.md`, `implementation_guide_v2.md`.

## Responsibilities

- Industrial step-field types per `system-requirements.md` §5.3.1: decision branch, text, numeric, date/time, enumeration, dropdown select, checkbox boolean, rating scale, signature capture.
- Failure-handling responses per step (abort, flag, inline deviation note).
- Deviation records.
- Equipment / standards / specifications reference catalogs (consumed by dropdown-select fields).
- Evaluators registered with `core-plugin-contracts` for SOP-specific computations.
- JDM decision tables (e.g., compliance gating, deviation severity classification) loaded by `core-rules-engine`.
- Audit-tag integration with `core-signatures-audit`.

## Dependencies

- External: `pydantic`, `sqlalchemy`
- Internal: `core-domain-contracts`, `core-plugin-contracts`, `core-rules-engine`, `core-templates`, `core-run-pipeline`, `core-signatures-audit`, `core-access`, `core-tenancy`

## Consumers

`app-industrial` only.

## Boundaries

This package owns nothing about workouts, fitness, tasks, or personal organization. If a feature isn't industrial-procedure-execution, it doesn't live here.
