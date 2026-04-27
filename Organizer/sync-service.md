# sync-service

## 1. Purpose

`sync-service` provides offline-first synchronization between a local (SQLite) database and the server. It lets a user log workouts on a device with no connectivity, then reconcile with the server later without losing data or producing duplicates. It is also the basis for the CLI / desktop / future mobile workflows.

## 2. Scope

**In scope**
- Sync helpers (push local → server, pull server → local, resolve conflicts).
- Stable client-generated IDs for offline-created records.
- Per-table sync metadata (last-synced cursor, pending-write queue).
- The CLI surface that exposes sync (`scripts/manage.py` already supports CSV round-trip; sync is the natural extension).
- Integration points in `activity-service`, `exercise-service`, `template-service` so they expose sync-friendly read/write APIs.

**Out of scope**
- Real-time collaborative editing — this is convergent sync, not CRDT.
- Messaging delivery (→ `messaging-service`).

## 3. Current Capabilities

Service: `app/services/sync.py`. Doc: `docs/OFFLINE_SYNC.md`. README mentions "offline sync helpers" as a feature highlight and `python scripts/manage.py export-sqlite` / `build-db` as the current portability story. The user-stories doc lists offline mobile logging as "Future Considerations."

## 4. Public API Surface

- HTTP: `POST /sync/push`, `POST /sync/pull`, `GET /sync/cursor`.
- Python: `push(local_db, server_client)`, `pull(server_client, local_db, since_cursor)`, `resolve_conflict(local, remote, policy)`.
- CLI: `python scripts/manage.py sync` (target).

## 5. Data Model

Owns: per-table sync-metadata tables (e.g., `sync_cursors`, `pending_writes`). Reads/writes the canonical tables via the owning service's API — does not bypass the service layer.

## 6. Dependencies

- Depends on: `activity-service`, `exercise-service`, `template-service`, `auth-service`, `shared-schemas`.
- Must NOT depend on: any frontend lib.

## 7. Cross-Cutting Concerns

- **Conflict resolution**: the policy must be documented per entity (last-write-wins for logs is usually fine; templates need user prompt).
- **Idempotency**: every operation must be safe to replay.
- **Schema drift**: a client at schema version N sync'ing to a server at version N+1 needs a migration path.

## 8. Open Questions / Known Gaps

- No mobile or desktop client exists today; sync is mostly CLI-shaped.
- The conflict-resolution policy per entity is undocumented.
- The relationship between `sync-service`'s push API and `activity-service`'s normal create endpoints is not yet drawn — overlap is likely.
