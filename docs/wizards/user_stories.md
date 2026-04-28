# User Stories

## Epic A – Execute a Procedure
1. As a tenant admin, I create an organization with isolated data so that each tenant stays secure.
2. As a tenant admin, I define access groups within the organization so that permissions are reusable.
3. As an admin, I assign users to one or more groups so that access aligns with responsibilities.
4. As a user, I only see procedures my groups have view access to so that cross-tenant data stays hidden.
5. As a user, I can execute only the procedures my groups have execute permission for so that runs stay controlled.
6. As an admin, I grant edit rights to specific groups so that authoring is limited to the right people.
7. As a procedure executor, I search or filter procedures by tags so that I can quickly find the right workflow.
8. As a procedure executor, I view procedure context such as purpose, safety, and tools so that I prepare correctly.
9. As a procedure executor, I acknowledge required steps before starting so that compliance expectations are clear.
10. As an author, I embed text, images, links, videos, and files into steps so that users have complete guidance.
11. As a procedure executor, I complete step forms with supported field types and signatures so that data capture is comprehensive.
12. As an author, I define allowed failure responses of abort, flag, or create an inline deviation note so that exceptions are governed.
13. As a procedure executor, I choose between aborting, flagging for review, or creating an inline deviation note when a step fails so that disposition is consistent.
14. As an author, I enable a review-all-answers page so that executors can confirm their inputs.
15. As a procedure executor, I review and edit answers before finishing so that the record is accurate.
16. As a procedure executor, I benefit from autosave and offline completion so that my work persists without connectivity.
17. As a procedure executor, I keep runs local until upload completes so that unsynced data stays private.
18. As a procedure executor, I understand that unsynced runs are hidden from other users so that visibility starts only after upload.
19. As the system, I trigger automatic record signing after the server verifies the upload so that offline runs remain trustworthy.
20. As a procedure executor, I view my past runs and shared runs so that I can reference prior work.
21. As a procedure executor, I apply a digital signature once per run so that the record is authenticated.
22. As a localization planner, I rely on the localization framework with English content so that future languages can be added.

## Epic B – Completion Record + Signature — MVP

**Goal**

Create an immutable completion record for a run and bind it to the executor via a digital signature. Verification must be self-serve.

### User Stories with Acceptance Criteria

1. **Generate completion record** – As a user, I want a canonical summary of my run at finish.
   - **AC:** Server produces a JSON payload containing `run_id`, `tenant_id`, `procedure_id`, `branch`, `semver`, `content_hash`, `steps[]`, `actor_user_id`, `started_at`, `completed_at`, `device_info`, `attachments[]` (hashes), `failure_flags[]`. Payload passes JSON Schema `run.v1`.
2. **Hash and canonicalize** – As the system, I want a deterministic hash of the record.
   - **AC:** Canonical JSON (sorted keys, UTF-8, no whitespace variance) → SHA-256 `record_hash`. Recomputing on the same record yields identical hash.
3. **Single signature per run** – As a user, I want to sign once per run.
   - **AC:** Endpoint accepts one signature per run; duplicate attempts are rejected unless previous is revoked by Admin.
4. **Signature capture (online)** – As a user, I sign the record after review.
   - **AC:** `POST /runs/{id}/sign` after standard web login. Server validates the authenticated session, signs the canonical payload with the system signing key on behalf of the user, and stores `signature`, `alg`, `signer_user_id`, `signed_at_utc`.
5. **Signature after offline sync** – As a user, my offline run is signed after upload.
   - **AC:** On sync, server builds canonical payload, computes `record_hash`, prompts sign flow; until signed, run is `"awaiting_signature"`.
6. **Public verification** – As an auditor, I can verify without login.
   - **AC:** `GET /verify/{record_hash}` returns: canonical payload, server-issued signature, `alg`, signer user reference, verification status `valid|invalid|unsigned`. HTTP 200 for valid/invalid, 404 for unknown.
7. **Evidence integrity** – As a user, I want attachment integrity guaranteed.
   - **AC:** Each attachment stored with SHA-256; record includes attachment hashes; verification recomputes and reports mismatches.
8. **Downloadable proof** – As a user, I can download JSON and PDF proof.
   - **AC:** JSON contains canonical record plus signature block. PDF displays summary and QR linking to verify endpoint; embedded `record_hash`.
9. **Audit entries** – As a security officer, I need traceability.
   - **AC:** Audit log entries for `run_created`, `run_completed`, `record_hashed`, `record_signed`, `verify_request` with actor, IP, timestamp, and entity IDs.
10. **Permission check** – As the system, only executors sign their runs.
    - **AC:** Sign endpoint enforces `actor_user_id == current_user_id` OR admin override with reason logged.
11. **Clock skew tolerance** – As the system, I want stable timestamps.
    - **AC:** All stored times in UTC; client times shown only for UX. Signature time uses server time.
12. **Revocation (Admin)** – As an admin, I can revoke a signature with reason.
    - **AC:** Run state → `revoked`, reason stored, verification shows revoked with details; record remains immutable.
13. **Tamper-evidence** – As the system, I must detect any post-sign changes.
    - **AC:** Any mutation to signed fields invalidates signature; API blocks updates after signing except admin revocation (which does not change signed payload).

### Non-Functional (Epic-specific)

- **Crypto:** Server-managed signing key (e.g., Ed25519) generates signatures on behalf of authenticated users and binds signer ID and timestamp into the record.
- **Performance:** hash + sign ≤ 300 ms p95 for 50-step runs.
- **Availability:** verify endpoint 99.9%.
- **Privacy:** verify returns minimal PII (e.g., signer display name optional, user ID masked by tenant-scoped pseudonym).

### Finalized MVP Rules

- **Signature and verification:** Authentication uses standard web login. The signature endpoint invokes the system signing key to bind the signer ID and server timestamp to the canonical payload. The public verification endpoint validates the record hash and signer identity via stored mappings. A future iteration can swap in WebAuthn or user-owned key pairs.
- **PDF proof:** The MVP exports a printable PDF containing the record hash and verification URL or QR code with no embedded PKCS #7 signature.
- **Offline:** Offline runs remain unsigned until uploaded and signed on the server.
- **Audit:** Every signature and revocation event is logged, and admin-driven revocations remain supported without mutating the signed payload.

### Data Model (MVP fields)

- `runs`: `id`, `tenant_id`, `procedure_id`, `version_id`, `actor_user_id`, `started_at`, `completed_at`, `status`
- `run_payloads`: `run_id`, `canonical_json`, `record_hash`, `created_at`
- `signatures`: `id`, `run_id`, `signer_user_id`, `alg`, `signature`, `signed_at`, `status (valid|revoked)`, `key_ref`
- `attachments`: `id`, `run_id`, `step_idx`, `filename`, `size`, `sha256`, `storage_uri`

### API (MVP)

- `POST /runs/{id}/complete` → returns canonical JSON + `record_hash`
- `POST /runs/{id}/sign` → `{alg, signature}` with server-side signing
- `GET /runs/{id}` → full record + signature block (auth)
- `GET /verify/{record_hash}` → public verification
- `POST /runs/{id}/revoke` (admin) → `{reason}`
- `GET /runs/{id}/proof.json` and `/proof.pdf`

## Epic C – Procedure Authoring Lifecycle — MVP (updated)

### Roles
- **Author:** Create or edit drafts and propose semantic version bumps.
- **Reviewer:** Review submissions and recommend approval or rejection.
- **Approver:** Issue final approval decisions and publish versions.
- **Admin:** Manage role assignments, override actions, and archive content.

### Key Policies
- Branch defaults inherit tags and permissions, with branch-level overrides permitted.
- Imports support either a fixed semantic version or automatic increment on next publish.
- Archiving allows in-flight runs to continue while blocking new starts.
- Diffs include text, metadata, and references only; binary media assets are excluded.

### User Stories with Acceptance Criteria

**C0. Role assignment**
- *Story:* As an Admin, I assign Author, Reviewer, and Approver roles per procedure or organization so that responsibilities are clear.
- *AC:* Role-based access control enforces permitted actions; a user may hold multiple roles concurrently.

**C5a. Submit for review (Author)**
- *Story:* As an Author, I submit a draft with a change message and proposed semantic version bump so that reviewers understand intent.
- *AC:* Draft state transitions to `in_review`, and the proposed bump is persisted with the submission metadata.

**C6a. Reviewer recommendation**
- *Story:* As a Reviewer, I recommend approval or request changes with comments so that feedback is traceable.
- *AC:* Draft state becomes `approved_pending_approver` or `changes_requested`, and the system writes an audit entry capturing the decision and comments.

**C7a. Approver decision**
- *Story:* As an Approver, I approve or reject a reviewer-approved draft so that publication is controlled.
- *AC:* Approval marks the draft eligible for publish; rejection returns the draft to `in_review` with a required reason.

**C7b. Publish with final bump**
- *Story:* As an Approver, I select the final semantic version bump (with authority to override the proposal) and publish to a branch so that release semantics remain consistent.
- *AC:* Draft state becomes `published`, the version increments accordingly, the immutable content blob and hash are stored, and version history records the event.

**C9a. Merge (fast-forward)**
- *Story:* As an Approver or Admin, I merge a branch into `main` only when a fast-forward merge is possible so that history stays linear.
- *AC:* If divergence exists, the system blocks the merge and provides guidance to reconcile.

**C10a. Diff view constraints**
- *Story:* As any role, I compare versions so that changes are transparent.
- *AC:* Diff views display only text, metadata, and reference changes, and explicitly indicate when media changed without rendering binaries.

**C12a. Archive behavior**
- *Story:* As an Admin, I archive a version so that deprecated content is hidden without disrupting active work.
- *AC:* Archived versions disappear from the start list, existing runs remain unaffected, and an audit entry is created.

**C13a. Rollback/Republish**
- *Story:* As an Approver, I republish a prior version as the latest so that controlled rollback is possible.
- *AC:* Republishing creates a new published version with an incremented semantic version while preserving the original content hash.

**CN1. Notification of Change (NoC)**
- *Story:* As an Approver, after publishing I send a notification of change to selected groups or users so that stakeholders are informed.
- *AC:* The system delivers email or in-app notifications containing procedure ID/title, branch, new semantic version, publish timestamp, author, approver, change message, and a diff summary link; delivery status is recorded.

**CN2. Subscription management**
- *Story:* As a User or Admin, I subscribe groups or users to notifications of change for specific procedures or tags so that communication is proactive.
- *AC:* Publishing triggers notifications to subscribed recipients, and user opt-outs are respected for future deliveries.

**C17a. Reference integrity**
- *Story:* As an Author, I attach structured references (standard or specification identifiers plus clause) so that compliance traceability is maintained.
- *AC:* Publish-time validation enforces the reference schema, and approved references display within the execution sidebar.

### Non-Functional (delta)
- Notification-of-Change delivery p95 latency is under 30 seconds, with retry logic using exponential backoff.
- All review, approval, publish, and notify actions generate audit log entries.

### Data Model (delta)
- `procedure_roles(user_id, procedure_id, role)`
- `notifications(id, procedure_id, version_id, recipients_json, status, created_at, sent_at, error)`
- `subscriptions(id, tenant_id, user_or_group_id, scope {procedure|tag}, ref, created_at)`

### API (delta)
- `POST /drafts/{id}/recommend` `{approve|changes_requested, comment}`
- `POST /drafts/{id}/approve` `{decision, comment}`
- `POST /drafts/{id}/publish` `{branch, bump}`
- `POST /versions/{id}/notify` `{recipients|scope}`
- `POST /subscriptions` / `DELETE /subscriptions/{id}`

### Open Item Resolved
- Semantic version policy: Authors propose a bump; Approvers select the final increment at publish time.

## Epic D – Version History & Diffs
1. As a user, I view chronological version history with author, reviewer, approver, and messages so that lineage is visible.
2. As the system, I display human-readable diffs covering text, metadata, and references so that changes are understandable.
3. As an admin, I export complete lineage for audits so that external reviews are supported.
4. As an integration developer, I call the diff endpoint API so that external systems stay synchronized.
5. As the system, I log all version and merge events immutably so that history cannot be altered.

## Epic E – Search / Discovery / Context
1. As a user, I search procedures by title, tag, or reference ID so that I quickly locate content.
2. As a user, I filter by branch, version, or access group so that results match my context.
3. As a user, I view linked standards and specifications in the procedure overview so that I understand compliance requirements.
4. As an admin, I manage existing tags (edit or delete) so that discoverability stays consistent.
5. As an author, I create individual tags without needing an approval workflow so that taxonomy can grow alongside procedure content.
   - **AC:** Authors with procedure edit rights can create single tags on demand; bulk tag editing remains a future enhancement beyond the MVP.
6. As an integration developer, I use the paginated, role-aware search API so that SaaS clients can embed discovery.

## Epic F – Roles / Auth / Tenancy
1. As the system, I isolate each organization by tenant_id so that SaaS data boundaries are enforced.
2. As a tenant admin, I manage users and groups so that staffing changes are handled internally.
3. As a tenant admin, I place users in multiple groups so that permissions mirror real teams.
4. As a tenant admin, I assign group permissions for view, edit, and execute so that access is precise.
5. As a user, I authenticate via standard web login with JWT sessions so that the experience is secure.
6. As a user, I optionally enable password reset and MFA so that account recovery and security improve.
7. As a SaaS operator, I onboard tenants through an API so that provisioning scales.
8. As the system, I log logins, role changes, and group changes so that audits capture identity events.

## Epic G – Audit / Integrity / Retention
1. As the system, I log every authentication, edit, signature, publication, and verification append-only so that traceability is complete.
2. As the system, I compute a monthly integrity hash or Merkle root so that tampering can be detected.
3. As a tenant admin, I configure retention policies such as ten-year run storage so that compliance mandates are met.
4. As a tenant admin, I export audit logs so that regulators can review activity.
5. As the system, I apply purge schedules while exempting immutable records so that governance rules hold.
6. As the system, I alert admins when tampering or verification mismatches occur so that issues are remediated quickly.

## Epic H – Offline / Sync / Resilience
1. As a procedure executor, I start a procedure offline using the local cache so that work can begin without connectivity.
2. As a procedure executor, I rely on autosave to preserve unsent runs in browser storage so that progress is never lost.
3. As the system, I sync runs upon reconnect and mark timestamps so that records reflect actual completion.
4. As a procedure executor, I see unsynced data only locally until upload so that incomplete work stays private.
5. As a procedure executor, I watch sync status indicators for pending, uploaded, or error so that I know the state of my run.
6. As the system, I resolve sync conflicts with server-side last-writer-wins for the MVP so that consistency remains manageable.
7. As a future planner, I target encrypted local storage and delta sync so that resilience continues to improve.
