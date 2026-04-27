# ProcessWizard Requirements Specification

## 1. Introduction
- **Purpose**: Define comprehensive functional and non-functional requirements for the ProcessWizard compliance platform, consolidating user stories and existing documentation into an actionable specification.
- **Scope**: Covers multi-tenant procedure execution, authoring lifecycle, completion records, user management, auditability, search/discovery, offline support, and UI standards for web applications.
- **Audience**: Product managers, engineering teams (backend, frontend, DevOps), QA, and compliance stakeholders responsible for delivering and validating the platform.

## 2. System Overview
ProcessWizard is a SaaS compliance system enabling organizations to author procedural workflows, execute them with controlled permissions, capture completion records with digital signatures, and maintain auditable links to regulatory references. It supports multi-tenant isolation, offline execution with deferred sync, and role-based governance across authoring and execution lifecycles.

## 3. Terminology
- **Tenant**: An isolated organization instance; all data is scoped to a tenant_id.
- **Procedure**: A structured workflow composed of versioned steps, metadata, references, and attached assets.
- **Run**: A single execution instance of a procedure by a user.
- **Canonical Record**: The immutable JSON payload representing the outcome of a run, including metadata, step answers, and signature binding.
- **Digital Signature**: Server-signed artifact binding a user and timestamp to a canonical record.
- **Branch**: Logical namespace for procedure versions (e.g., `main`, `development`).
- **Audit Log**: Append-only timeline capturing security-sensitive events.

## 4. Actors and Roles
- **System**: Automated behaviors and services.
- **Tenant Admin**: Manages organizations, users, groups, permissions, and retention policies.
- **Author**: Drafts and edits procedures, submits for review.
- **Reviewer**: Reviews drafts, recommends actions.
- **Approver**: Issues final publish decisions and manages version history.
- **Procedure Executor**: Runs procedures, records step data, applies signatures.
- **Localization Planner**: Plans translation readiness.
- **Security Officer / Auditor**: Reviews audit logs, verifies records.
- **Integration Developer**: Consumes APIs for search, diff, verification.

## 5. Functional Requirements

### 5.1 Tenancy & Access Control
1. The system SHALL allow tenant admins to create organizations with isolated data contexts; cross-tenant data access MUST be prevented via tenant_id scoping.
2. Tenant admins SHALL define reusable access groups (view, execute, edit) and assign users to multiple groups.
3. Permission checks MUST ensure users only see or execute procedures according to group-level grants.
4. Admins SHALL grant edit rights to specific groups per procedure to constrain authoring.
5. The system SHALL log logins, role changes, and group membership updates for auditability.
6. **REQ-AUTH-001.** The system SHALL authenticate users via web login issuing a short-lived access JWT and a rotating refresh JWT stored in HttpOnly Secure cookies.
7. **REQ-AUTH-002.** The system SHALL support MFA via TOTP; email one-time codes are permitted as fallback. Tenants MAY require MFA by policy.
8. **REQ-AUTH-003.** The system SHALL provide password reset via single-use, time-limited links and SHALL revoke existing refresh tokens on reset.
9. **REQ-TENANT-001.** SaaS operators SHALL provision tenants via an authenticated onboarding API.
10. **REQ-TENANT-002.** Onboarding SHALL create a dedicated database per tenant, apply migrations, seed defaults, create a tenant admin, register routing metadata, and return either 201 on completion or 202 with a status URL.
11. **REQ-TENANT-003.** The onboarding API SHALL be idempotent using a caller-supplied idempotency key and SHALL be restricted to operator roles.

#### Authentication Architecture Decisions
- **Token lifecycle**: Issue 15-minute access JWTs and 7-day rotating refresh JWTs stored as HttpOnly, Secure, SameSite=Lax cookies. Refresh tokens rotate on every use and are invalidated via rotation nonces stored in Redis. Sessions bind to IP subnet (/24 IPv4, /56 IPv6) and user-agent fingerprint during refresh to reduce token theft abuse while permitting normal roaming.
- **Signing**: Sign JWTs with EdDSA (Ed25519) and rotate keys using the `kid` header to advertise active keys.
- **MFA**: Provide TOTP (RFC 6238) with authenticator app enrollment, email one-time codes as fallback, and roadmap WebAuthn/Passkeys for v2. Enrollment occurs post-login when tenant policy requires it, includes QR provisioning, backup code issuance (10 codes), and one-time verification before activation. MFA verification upgrades the session with an `mfa=1` claim.
- **Account recovery**: Password resets issue 15-minute, single-use links delivered by email via `itsdangerous`. Successful resets revoke all outstanding refresh tokens. Passwords use Argon2id hashing with calibrated parameters and enforce a minimum length of 12 characters, supporting passphrases.
- **Client behavior**: The React client refreshes tokens once upon 401 responses via an interceptor and logs the user out if refresh fails. Access tokens remain accessible only through server-set cookies; no long-lived tokens persist in JavaScript memory.

#### Tenant Provisioning Decisions
- **Isolation**: Allocate a dedicated database per tenant, named by UUID, with DSNs stored encrypted at rest. Database credentials rotate per tenant when supported.
- **Routing**: Resolve tenant databases through a router that reads `tid` from subdomain or header, fetches the DSN from the tenant registry, and warms the connection pool after provisioning.
- **Provisioning workflow**: Synchronously create tenants within 10 seconds when possible; otherwise return HTTP 202 with a status URL for polling. Provisioning creates the database from a template or applies Alembic migrations, seeds organization metadata, default roles, and settings, creates the first tenant admin user with an activation link, registers routing metadata, and emits an audit event. Tenant user management beyond the initial admin occurs via tenant-scoped APIs.
- **Access control & observability**: Onboarding endpoints require a separate SaaS operator realm authenticated via `X-Operator-Auth`, enforce idempotency through caller-supplied `X-Idempotency-Key` headers, surface `409` conflicts for duplicate subdomains or keys, and correlate requests using `X-Request-Id`. Full audit logs capture provisioning actions.
- **API surface**: Provide `POST /ops/tenants` (operator) for provisioning, returning 201 with tenant details or 202 with a status URL; `GET /ops/tenants/{tenant_id}` for status polling; and `POST /tenants/users` (tenant admin) for inviting additional users post-onboarding.

### 5.2 Procedure Discovery & Context
1. Users SHALL search procedures by title, tag, or reference identifier with pagination and role-aware filtering.
2. Users SHALL filter results by branch, version, and access group membership.
3. Procedure overview pages SHALL display linked standards/specifications and contextual metadata (purpose, safety, tools).
4. The system SHALL allow authors to create new tags without requiring an approval workflow, while admins retain permissions to edit or delete tags.
5. Bulk tag edits MAY be supported in a future release but SHALL NOT be included in the MVP scope.
6. Integration developers SHALL be able to query a paginated search API that respects user permissions.

### 5.3 Procedure Execution Experience
1. Procedure executors SHALL see only procedures for which their groups have view permission.
2. Executors SHALL only be able to initiate runs when their groups possess execute permission.
3. Procedure detail views SHALL expose context (purpose, safety notes, required tools) prior to run start.
4. Prior to starting a run, executors MUST acknowledge required steps/expectations via explicit confirmation.
5. Steps SHALL support rich content (text, images, links, videos, files) embedded by authors.
6. Step forms SHALL support the field catalog defined in Section 5.3.1, including branching logic and validation semantics per field type.
7. Authors SHALL define allowed failure responses per step: abort, flag, or create inline deviation note.
8. During execution, failure handling MUST offer only the configured responses.
9. Authors MAY enable a “review all answers” page summarizing captured inputs before completion.
10. Executors SHALL be able to review and edit responses prior to final submission.
11. Autosave MUST preserve step progress locally, including during offline use.
12. Offline runs MUST remain local until upload completes; unsynced runs SHALL be hidden from other users.
13. Upon connectivity restoration, the client SHALL sync pending runs, invoking server-side verification and signature flows.
14. Executors SHALL apply a digital signature once per run; system MUST prevent duplicate signatures unless revoked by an admin.
15. Executors SHALL access a history of their own runs and runs shared with them.
16. Localization framework MUST at minimum load English content and be extensible for additional locales.

#### 5.3.1 Step Field Catalog

To guarantee consistency between authoring and execution experiences, the platform SHALL implement the following step field types. Each field type inherits base metadata (`step_id`, `label`, `help_text`, `required`, `visibility_condition`, `audit_tags`) and MAY specify additional configuration as noted.

1. **Decision Branch**
   - Provides a mutually exclusive choice (e.g., Yes/No, Option A/B/C) whose selected value determines the subsequent branch path.
   - Authors SHALL declare target step IDs or branch namespaces per option. When the branch path completes, the run SHALL return to the parent branch or the root sequence.
   - Executors SHALL see only options available within their permission scope; default options MAY be configured.
2. **Text Input**
   - Supports single-line or multi-line capture with configurable max length.
   - Authors MAY supply input masks (e.g., phone number, postal code) and validation regex patterns; masked inputs SHALL display placeholders to executors.
3. **Numeric Input**
   - Captures integers or decimals with configurable precision, min/max bounds, and unit annotations.
   - Authors SHALL specify whether negative values are permitted; validation errors MUST block progression until resolved.
4. **Date/Time Input**
   - Supports `date`, `time`, or combined `datetime` modes with timezone awareness.
   - Authors MAY define relative constraints (e.g., not before run start, not after current time plus tolerance) and default to current timestamp when permitted.
5. **Enumeration**
   - Presents mutually exclusive radio buttons or multi-select checkboxes derived from a controlled value list.
   - Authors SHALL flag whether multiple selections are allowed and MAY set display order, grouping, and minimum/maximum selection counts.
6. **Dropdown Select**
   - Provides a single-selection list suitable for large option sets with optional search-as-you-type filtering.
   - Authors MAY source options from static lists or dynamic references (e.g., equipment catalog) and SHALL define fallback behavior when referenced data is unavailable offline.
7. **Checkbox Boolean**
   - Represents a single boolean toggle with optional confirmation text.
   - Authors MAY require explicit acknowledgement (e.g., “I have reviewed the safety checklist”) before allowing completion.
8. **Rating Scale**
   - Collects ordinal feedback across a configurable range (e.g., 1–10) with optional labels for endpoints and midpoints.
   - Authors MAY enforce whole-number or half-step increments and SHALL define interpretation guidance for compliance review.
9. **Signature Capture**
   - Enables typed name, drawn signature, or certificate-backed signature capture, binding the executor identity, timestamp, and device metadata to the canonical record.
   - Signature fields SHALL trigger server-side validation to prevent duplicate signatures per run unless prior signatures are revoked by an administrator.

All field types MUST support localization of labels/help text, audit logging of value changes (including offline edits queued for sync), and client-side validation feedback consistent with UI standards.

### 5.4 Completion Record & Signature (MVP)
1. Completing a run SHALL invoke `POST /runs/{id}/complete`, generating a canonical JSON payload containing fields defined in the MVP data model and conforming to schema `run.v1`.
2. The system SHALL produce a deterministic SHA-256 hash of the canonical payload with stable key ordering and encoding.
3. `POST /runs/{id}/sign` SHALL accept one signature per run; duplicates MUST be rejected unless an admin revokes the prior signature.
4. Online signing flow SHALL validate the authenticated session, sign the canonical payload using a server-managed key, and persist signature metadata (`signature`, `alg`, `signer_user_id`, `signed_at_utc`).
5. Offline runs SHALL transition to `"awaiting_signature"` after sync; users must complete signing before the run is considered finalized.
6. `GET /verify/{record_hash}` SHALL return canonical payload, signature data, signer reference, and verification status (`valid`, `invalid`, `unsigned`); unknown hashes return HTTP 404.
7. Attachment integrity MUST be enforced via stored SHA-256 hashes; verification SHALL recompute and report mismatches.
8. Users SHALL download proof artifacts in JSON and PDF formats. PDF MUST display summary data and a QR code linking to the verification endpoint using the header, footer, and typography defined in the Brand Guidelines document template (docs/brand-guidelines.md §8).
9. Audit log entries SHALL be emitted for `run_created`, `run_completed`, `record_hashed`, `record_signed`, `verify_request` with actor, IP, timestamp, and entity IDs.
10. Sign endpoint MUST ensure only the executor or an admin override (with logged reason) can sign the run.
11. All timestamps SHALL be stored in UTC; client-side times are for display only.
12. Admins SHALL be able to revoke signatures, transitioning run state to `revoked` and preserving the immutable payload; verification MUST show revocation details.
13. API MUST block updates to signed fields post-signing except for admin revocation metadata.
14. Hashing plus signing operations MUST complete within 300 ms p95 for 50-step runs.
15. Verification endpoint availability MUST achieve 99.9% uptime.
16. Verification responses MUST minimize PII by replacing user identifiers with deterministic tenant-scoped pseudonyms computed as `u_` plus the first 10 hex characters of `SHA-256(APP_PSEUDO_SECRET | tenant_id | "|" | user_id)`. The `APP_PSEUDO_SECRET` environment variable SHALL be the shared secret for pseudonym generation and SHALL NOT require rotation in the MVP. No database table is required for pseudonym storage; identical inputs MUST yield identical pseudonyms.

### 5.5 Procedure Authoring Lifecycle (MVP)
1. Admins SHALL assign Author, Reviewer, Approver roles per procedure or organization with overlapping roles permitted.
2. Authors SHALL submit drafts with change messages and proposed semantic version bumps, moving state to `in_review`.
3. Reviewers SHALL record recommendations (approve or request changes) with comments; system logs audits.
4. Approvers SHALL approve or reject reviewer-approved drafts, transitioning states accordingly.
5. Approvers SHALL select the final semantic version bump during publish, override allowed.
6. Publishing SHALL persist immutable content, content hash, version history, and branch targeting.
7. Merge governance SHALL comply with the following requirements:
   - **REQ-GIT-001.** The `main` branch SHALL be protected to require fast-forward merges only; merge commits and force-pushes are prohibited.
   - **REQ-GIT-002.** Divergent branches that cannot fast-forward MUST be rebased on the latest `main` before merging.
   - **REQ-GIT-003.** The remediation workflow SHALL include: (1) fetching the latest `main`, (2) rebasing the feature branch, (3) resolving conflicts, and (4) force-pushing with `--force-with-lease`.
   - **REQ-GIT-004.** A developer guide SHALL document these steps and include an example of rebase conflict resolution.

#### Merge Governance Verification

| ID | Requirement | Verification Method | Evidence | Status |
| --- | --- | --- | --- | --- |
| REQ-GIT-001 | Branch protection enforces fast-forward merges | Repo settings review | Screenshot of GitHub “Require linear history” and “Disallow force pushes” | Planned |
| REQ-GIT-002 | Divergent branches blocked | Functional test | Attempt to merge an outdated PR shows fast-forward error | Planned |
| REQ-GIT-003 | Remediation workflow documented and followed | Procedure test | Developer guide snippet or wiki page plus successful rebase log | Planned |
| REQ-GIT-004 | Developer guide includes rebase conflict example | Documentation review | Developer guide excerpt demonstrating conflict resolution steps | Planned |

_Optional automation (post-MVP): introduce a pre-merge bot that detects non-fast-forward branches via `git merge-base --is-ancestor main HEAD` and comments with remediation instructions. The MVP relies solely on static branch protection and the documented workflow._
8. Diff views SHALL display text, metadata, and reference changes and indicate media differences without binary rendering.
9. Archiving SHALL hide versions from start lists while allowing active runs to continue; audit entry required.
10. Approvers SHALL republish prior versions as new releases with incremented semantic versions while reusing original content hashes.
11. Post-publish notifications (Notification of Change) SHALL support selecting recipients (users/groups) with delivery status tracking and audit logging.
12. Subscription management SHALL allow users/admins to subscribe recipients by procedure or tag; opt-outs MUST be respected.
13. Reference attachments SHALL follow a validated schema at publish-time and display within execution sidebar.
14. Notification delivery MUST exhibit <30s p95 latency with exponential backoff retries.
15. All review, approval, publish, and notification events SHALL generate audit entries.

### 5.6 Version History & Diffs
1. Users SHALL view chronological version history including author, reviewer, approver, and change messages.
2. System SHALL provide human-readable diffs covering text, metadata, and references.
3. Admins SHALL export complete version lineage for audits.
4. Diff API SHALL be accessible to integrations for synchronization.
5. Version and merge events MUST be logged immutably.

### 5.7 Audit, Integrity, and Retention
1. System SHALL log authentication, edits, signatures, publications, verifications in an append-only store.
2. Monthly integrity hashes or Merkle roots SHALL be computed to detect tampering.
3. Tenant admins SHALL configure retention policies (e.g., 10-year run storage) with enforcement schedules.
4. Admins SHALL be able to export audit logs for regulators.
5. Purge schedules MUST honor governance rules, excluding immutable records.
6. System SHALL alert admins on tampering or verification mismatches via configured notification channels. **MVP implementation:** write high-priority alert entries to a dedicated log file that operations staff monitor; future releases may extend this to additional notification transports.

### 5.8 Offline, Sync, and Resilience
1. Executors SHALL initiate procedures offline using locally cached procedures.
2. Client SHALL autosave progress to browser storage for offline resilience.
3. Upon reconnect, client SHALL sync runs, recording actual completion timestamps server-side.
4. Local unsynced data SHALL remain private and invisible to other users.
5. UI SHALL display sync status indicators (pending, uploaded, error).
6. Server SHALL resolve sync conflicts with last-writer-wins for MVP while surfacing an overwrite warning to the user; when the conflicting artifact is a JSON procedure payload, the system SHALL present a source comparison generated via an approved third-party diff utility (e.g., WinDiff) to aid review.
7. Roadmap SHALL replace the third-party diff integration with an in-house, WYSIWYG comparison experience and continue to target encrypted local storage and delta sync improvements for future iterations. [review] Define backlog prioritization.

### 5.9 User Management Service Integration
1. Backend SHALL implement data schema: `users`, `user_public_keys`, `user_unit_preferences` with constraints outlined in documentation.
2. Passwords MUST use Passlib bcrypt hashing; JWTs minted with configurable expiry and stored secrets.
3. Email normalization (case-insensitive) MUST be enforced before persistence.
4. Guest accounts MAY rely on `user_token` cookie, while registered users MUST authenticate via bearer token headers.
5. API SHALL provide endpoints for self profile retrieval/update, profile image upload, password changes, public key rotation, relationship listing, badge enumeration, and aggregated profile snapshot retrieval. HOLD (Endpoint URLs unspecified beyond `/users` router).
6. `UserUpdatePayload` SHALL validate optional fields including birthdate (not in future) and avatar path.
7. Preferences payloads MUST conform to shared JSON Schema and align with frontend constants.
8. Frontend SHALL offer flows for signup (with email validation, public key registration), login with “trust this computer”, password change, and preference editing using shared utilities for auth tokens.
9. SDKs/clients (TypeScript, Python) SHALL wrap authentication and user management endpoints with retries and error handling. HOLD (SDK distribution strategy unspecified).
10. Service SHALL be exposed via internal API gateway for consumption by multiple apps.

### 5.10 Frontend UI Standards
1. Web UI SHALL adopt MUI components with a clean, minimalist aesthetic (light theme, white panels, subtle shadows).
2. Layout SHALL include left sidebar navigation with profile info and section grouping (Dashboard, Calendar, Exercise Editor, History, Settings). [review] Confirm applicability to compliance-focused UI.
3. Large-screen layout SHALL present a two-column structure: scrollable calendar/time grid on left, dynamic form editor on right; small screens use drawer interaction. HOLD (Exact responsive breakpoints unspecified).
4. Forms SHALL prioritize structured inputs (checkboxes, selects) with real-time enhancements planned for future activity wizard. [review] Validate future roadmap alignment.
5. Responsiveness SHALL utilize `useMediaQuery` and conditional rendering to ensure mobile usability; additional polishing needed for scrollbars and spacing.
6. Branding SHALL remain professional and utilitarian, adhering to the Process Wizard Brand Guidelines for logo usage, color, and typography.

## 6. Non-Functional Requirements
1. Availability targets: Verification endpoint 99.9%; overall platform SLA TBD. HOLD (SLA specifics pending).
2. Performance: Hash + sign ≤300 ms p95 for 50-step runs; notification delivery <30s p95.
3. Security: Enforce EdDSA-signed JWT auth with rotating refresh tokens, Argon2id password storage, optional MFA per tenant policy; maintain append-only audit logs, attachment hashes, and monthly integrity root for tamper detection.
4. Privacy: Verification endpoint returns minimal PII (masked identifiers). Data residency/compliance requirements TBD. HOLD.
5. Localization: English baseline with extensibility for additional languages using localization framework. HOLD (Implementation strategy needed).
6. Offline resilience: Autosave, conflict resolution, and sync reliability as defined in Section 5.8.

## 7. Data Model Summary
- `runs`: `id`, `tenant_id`, `procedure_id`, `version_id`, `actor_user_id`, `started_at`, `completed_at`, `status`.
- `run_payloads`: `run_id`, `canonical_json`, `record_hash`, `created_at`.
- `signatures`: `id`, `run_id`, `signer_user_id`, `alg`, `signature`, `signed_at`, `status (valid|revoked)`, `key_ref`.
- `attachments`: `id`, `run_id`, `step_idx`, `filename`, `size`, `sha256`, `storage_uri`.
- `procedure_roles`: `user_id`, `procedure_id`, `role`.
- `notifications`: `id`, `procedure_id`, `version_id`, `recipients_json`, `status`, `created_at`, `sent_at`, `error`.
- `subscriptions`: `id`, `tenant_id`, `user_or_group_id`, `scope`, `ref`, `created_at`.
- `users`: `id`, `name`, `email`, `hashed_password`, `is_admin`, `is_client`, `status`, `user_image`, `organization_id`.
- `user_public_keys`: `(user_id, version)`, `public_key`, `created_at`.
- `user_unit_preferences`: `user_id`, `measurement`, `unit`, optional `exercise_id` or `tag`, uniqueness per scope.

## 8. API Surface Summary
- Completion & signature: `POST /runs/{id}/complete`, `POST /runs/{id}/sign`, `GET /runs/{id}`, `GET /verify/{record_hash}`, `POST /runs/{id}/revoke`, `GET /runs/{id}/proof.json`, `GET /runs/{id}/proof.pdf`.
- Authoring lifecycle: `POST /drafts/{id}/recommend`, `POST /drafts/{id}/approve`, `POST /drafts/{id}/publish`, `POST /versions/{id}/notify`, `POST /subscriptions`, `DELETE /subscriptions/{id}`.
- Diffing & history: APIs to list versions, fetch diffs, and export lineage. HOLD (Endpoint definitions pending).
- Search/discovery: Paginated, role-aware search API. HOLD (Endpoint specifications pending).
- User management: `/users` router endpoints for profile management, preferences, security operations. HOLD (Full route mapping pending).
- Authentication & session management: `POST /auth/login`, `POST /auth/mfa/verify`, `POST /auth/mfa/enroll`, `POST /auth/mfa/activate`, `POST /auth/refresh`, `POST /auth/logout`, `POST /auth/password/reset-request`, `POST /auth/password/reset-confirm`.
- Tenant onboarding API for SaaS operators: `POST /ops/tenants` (idempotent provisioning with 201/202 semantics), `GET /ops/tenants/{tenant_id}` (status), and `POST /tenants/users` (tenant admin user invitations post-onboarding).

## 9. Integration & SDK Requirements
1. Internal API gateway SHALL front user-management and procedure services for shared authentication.
2. TypeScript and Python SDKs SHALL include authentication helpers, profile management, preference APIs, and notification subscriptions with retry logic and error normalization. HOLD (SDK release process unspecified).
3. External integrators SHALL be provided documentation and JSON Schemas for all exposed endpoints, with versioning to support rolling upgrades.

## 10. Compliance & Audit Considerations
1. Audit logs MUST be exportable in regulator-friendly formats (e.g., CSV, JSON). HOLD (Exact format requirements unspecified).
2. Digital signatures SHALL be managed via server-held signing keys (e.g., Ed25519) with secure rotation policies. HOLD (Key rotation cadence unspecified).
3. Monthly integrity proofs SHALL be stored securely and verifiable by auditors.
4. Retention policies SHALL support tenant-specific durations with safeguards to prevent deletion of immutable run payloads.
5. Verification mismatch alerts SHALL reach designated admins promptly. HOLD (Notification channels unspecified).

## 11. Open Questions & HOLD Items
- Define purpose and scope statements precisely (Section 1). HOLD
- Detail tag taxonomy management workflow and permissions. HOLD
- Provide localization implementation strategy and roadmap. HOLD
- Establish alerting channels/mechanisms for tampering notifications. HOLD
- Document SDK packaging, distribution, and maintenance responsibilities. HOLD
- Clarify responsive breakpoints and layout applicability to compliance UI. HOLD
- Provide comprehensive API endpoint specifications for search, diff, history, and user management. HOLD
- Define audit export formats and cadence. HOLD
- Outline signing key rotation and storage strategy. HOLD

## 12. Assumptions & [review] Recommendations
1. [review] Adopt FastAPI + SQLAlchemy backend with TimescaleDB for time-series auditing, aligning with technical stack preferences.
2. [review] Utilize React + TypeScript + MUI for web client, employing React Hook Form for procedure execution and authoring forms.
3. [review] Implement Recharts or similar for run analytics dashboards in future iterations.
4. [review] Introduce encrypted local storage for offline data when roadmap prioritizes resilience.
5. [review] Provide admin tooling for retention policy configuration and monitoring.

## 13. Verification Plan

| ID | Requirement | Verification Method | Evidence | Status |
| --- | --- | --- | --- | --- |
| REQ-AUTH-001 | Web login issues access+refresh JWT in Secure HttpOnly cookies with rotation | Functional test + code review | Postman/Newman run: login, refresh, rotation; code snippet showing cookie flags and rotation nonce in Redis | Planned |
| REQ-AUTH-002 | MFA via TOTP; email OTP fallback; tenant-policy enforcement | Functional test + security test | MFA enrollment + verify flows, backup codes, policy toggle test; screenshots of tenant policy UI | Planned |
| REQ-AUTH-003 | Password reset via single-use link revokes refresh tokens | Functional test | Reset link use, attempt to use old refresh token fails; log evidence | Planned |
| REQ-TENANT-001 | Authenticated onboarding API for operators only | Security test + access control review | RBAC config showing operator realm; 403 when non-operator calls /ops/tenants | Planned |
| REQ-TENANT-002 | Onboarding creates per-tenant DB, migrates, seeds, admin, routing | Integration test | Provisioning log, new DB present, Alembic version at head, admin activation email, registry row | Planned |
| REQ-TENANT-003 | Idempotent onboarding with key; 201/202 semantics | Functional test | Two identical POSTs with same X-Idempotency-Key return same result; 202→200 via status URL | Planned |

**Minimal Test Plan (current environment)**
- Auth:
  - Try 11 bad passwords → account locked.
  - Login → Set-Cookie has HttpOnly; Secure; SameSite=Lax.
  - Refresh twice → `jti` changes, old refresh invalid.
  - Enable MFA, verify TOTP, check `mfa:1` claim.
  - Use backup code once → marked consumed.
- Tenant:
  - `POST /ops/tenants` with idempotency key → 201.
  - Repeat same call → same 201 payload, no new DB.
  - Connect with new subdomain → routes to the new DB; default roles seeded.

## 14. Traceability
- User stories in `docs/user_stories.md` map to functional requirements in Sections 5.1–5.8.
- User management service requirements align with Section 5.9 and data schema in Section 7.
- UI standards align with Section 5.10 and pending design refinements noted in HOLD items.

## 15. Approval & Revision History
- **Version**: 0.1-draft
- **Prepared By**: Engineering (Generated requirements synthesis)
- **Reviewed By**: HOLD (Assign reviewers)
- **Approval Date**: HOLD
- **Next Review**: HOLD
