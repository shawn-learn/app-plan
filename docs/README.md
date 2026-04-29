# Documentation layout

This repository intentionally keeps documentation in two places:

- `docs/`: cross-project documentation (wizard requirements, implementation standards, and shared guidance).
- `src/*/docs/`: docs that belong to a specific source tree and evolve with that package/app area.

## Should all `src` subfolders move under `docs/`?

No. The `src` directory should continue to hold source-owned modules (and any docs tightly coupled to those modules). Moving all `src` subfolders into `docs/` would blur the boundary between implementation structure and project-wide documentation.

## Recommended structure

- Keep product and architecture docs in `docs/`.
- Keep code-adjacent docs under the relevant `src/<area>/docs/` folder.
- If a `src/<area>/docs/*` file becomes broadly shared, copy or promote it into `docs/` and link back.


## Why are shared-looking items still under `src/organizer`?

Some material in `src/organizer` is intentionally ahead-of-code planning content for extracting reusable libraries. It may look shared because it describes capabilities (for example, auth, templates, or units) that are expected to become platform libraries, but those specs were drafted from the Organizer codebase first.

In short: this repository is in a transition state where domain-first docs and platform-target docs coexist.

## Why does `platform` also mention Organizer?

`src/platform` defines the future shared core and explicitly lists multiple apps, including Organizer, as consumers. That does **not** make Organizer part of platform internals; Organizer is a domain/app that depends on platform contracts.

A practical rule of thumb:

- Put app/domain-specific behavior in `src/organizer/*`.
- Put cross-app contracts and architecture in `src/platform/*` and top-level `docs/*`.
- If content starts in Organizer but is needed by multiple apps, promote it to platform/shared docs and leave a backlink in Organizer docs.
