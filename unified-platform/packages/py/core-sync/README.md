# core-sync

**Tier 2 — Platform.**

Offline-first sync helpers and conflict-resolution primitives.

## Source material

Merges:
- `Organizer/sync-service.md` (offline-DB sync helpers, planned for desktop/CLI)
- Process Wizard `system-requirements.md` §5.3 offline-run requirements (autosave, hidden-until-synced unsynced runs, server-side verification on upload)

## Responsibilities

- Client-side change journaling format.
- Server upload endpoint signatures.
- Server-side verification hooks (used by `core-run-pipeline` on upload).
- Conflict resolution strategies: last-write-wins, three-way-merge, domain-defined merger.
- "Run not visible to others until upload completes" enforcement.

## Dependencies

- External: `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-data-access`, `core-run-pipeline`

## Consumers

App shells (mount sync endpoints), CLI tooling, future desktop/mobile clients.
