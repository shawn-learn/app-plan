# core-templates

**Tier 2 — Platform.**

Versioned document/template storage and import/export. Generalizes Organizer's activity templates and Process Wizard's Form.io documents into one primitive.

## Source material

Merges:
- `Organizer/template-service.md` (`activity_templates`, `template_id` + `version`, JSON-defined structures, periodization, recurrence)
- Process Wizard Form.io document storage (procedures, intake schemas, assessment schemas)

## Responsibilities

- Store any structured document keyed by `(template_id, version)`.
- Branch namespaces (`main`, `development`) per Process Wizard's branch concept.
- Version history with diff generation.
- Import/export to canonical JSON (CLI-driven via Organizer's `import_template.py` / `export_template.py` patterns).
- Share-grants delegated to `core-access`.
- Tag and metadata indexing for discovery.

## Dependencies

- External: `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-access`, `core-data-access`

## Consumers

`core-run-pipeline` (instantiates a run from a versioned template), `core-scheduling` (planned occurrences reference a template version), every domain plugin (each ships seed templates).
