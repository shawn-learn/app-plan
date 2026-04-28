# Unified Documentation (Reorganized)

This folder contains the consolidated documentation set for the merged platform.

## Document Structure

1. `01-platform-architecture.md` — canonical architecture, layers, package catalog, dependency rules.
2. `02-domains-and-apps.md` — domain plugin responsibilities and app-shell composition.
3. `03-delivery-roadmap.md` — phased implementation and validation strategy.
4. `04-repo-and-packaging-standards.md` — monorepo layout, polyrepo-readiness, release discipline.
5. `05-legacy-crosswalk.md` — mapping from legacy Organizer + Process Wizard documents to new canonical docs.
6. `06-functional-requirements.md` — consolidated feature and functional requirements.
7. `07-operational-and-delivery-details.md` — testing, operations, security, and migration policy details.

## Usage Rules

- Treat this directory as the source of truth for newly migrated content.
- Legacy docs marked `DELETE` are retained temporarily for traceability only.
- Legacy docs **without** `DELETE` are still active references pending migration.
- Package-level README files under `unified-platform/packages/**` and app README files under `unified-platform/apps/**` remain implementation-local references.
