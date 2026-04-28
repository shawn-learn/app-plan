DELETE

# Architecture Review Feedback for `plan.md`

This review focuses on strategic architecture risk reduction and practical implementation guidance for a unified workout/coaching/organizer/industrial platform.

## Executive Assessment

The proposed direction is strong and coherent:

- The **platform + domain plugin** split is the right long-term architecture.
- Treating fitness as a plugin is the key decision that preserves reusability.
- A shared contract-first model (Pydantic + generated TS) is excellent.
- Polyrepo-ready package boundaries are a major force multiplier.

The biggest remaining risks are not conceptual—they are **operational and boundary risks**:

1. Overly broad “core” packages can become hidden domain leakage points.
2. Multi-tenant + time-series + offline sync creates hard data consistency requirements.
3. Security/compliance (audit/signatures/access) needs explicit threat model + verification strategy early.
4. UI/runtime plugin boundaries need stricter version/compatibility contracts to avoid integration drift.

---

## What Is Especially Strong

1. **Dependency direction is explicit and correct** (downward only).
2. **Templates + run pipeline as a universal primitive** is a scalable abstraction.
3. **Generalized scheduling** avoids fitness-specific lock-in.
4. **Per-package CI and independent versioning** align with long-term maintainability.
5. **Phased rollout** is sensible and enables incremental risk retirement.

---

## Strategic Recommendations (High Impact)

## 1) Introduce Formal Architecture Decision Records (ADRs) for non-reversible choices

Add ADRs immediately for:

- Tenancy model by app (shared DB + tenant key vs DB-per-tenant).
- Eventing model (outbox pattern, at-least-once vs exactly-once assumptions).
- Contract evolution policy (backward compatibility + deprecation windows).
- Signature and audit trust model (key ownership, rotation, verification chain).

**Why:** These decisions are expensive to reverse and will affect package APIs and data model choices across all tiers.

## 2) Add “Domain Leakage Guards” beyond import-direction linting

Current linting blocks platform importing domain packages, which is necessary but not sufficient.
Add:

- Allowed dependency matrix (`allowed_imports.yaml`) at package granularity.
- CI guard that checks no fitness/industrial nouns appear in platform public APIs (schema/name scanning).
- Public API diff checks per package release.

**Why:** Leakage often happens through naming and API fields, not just import statements.

## 3) Move to an event-first integration backbone for cross-package workflows

Define a platform event contract now (even with minimal implementation):

- `RunCompleted`, `TemplatePublished`, `ScheduleOccurrenceCreated`, `SignatureRecorded`, `SyncConflictDetected`.
- Include tenant ID, causation ID, correlation ID, actor, and monotonic sequence.
- Persist via outbox table + idempotent consumers.

**Why:** This reduces hard coupling between runtime/scheduling/goals/content/messaging later.

## 4) Establish a compatibility policy between plugin contracts and plugin implementations

For `core-plugin-contracts` + `web-plugin-ui`, add:

- SemVer compatibility matrix (platform contract vX supports plugin vA-B).
- Plugin manifest with declared contract version range.
- Runtime negotiation + hard fail with actionable diagnostics.

**Why:** Plugin ecosystems fail mostly due to compatibility drift, not feature gaps.

## 5) Split “core-data-access” into lower-level primitives and enforce repository boundaries

`core-data-access` risks becoming a god package. Keep it narrow:

- DB engine/session factories.
- Base repository interfaces.
- Migration helpers.

Do **not** allow package-specific query logic here.

**Why:** Centralized data access packages often become accidental domain layers and block extraction.

---

## Practical Recommendations (Implementation-Level)

## 6) Add canonical IDs and audit metadata conventions globally

Define once in `core-domain-contracts`:

- ID format (UUIDv7 recommended for sortability).
- `created_at`, `updated_at`, `actor_id`, `source`, `trace_id` conventions.
- Soft-delete strategy and archival marker semantics.

**Why:** Prevents drift and simplifies analytics/reference-store joins.

## 7) Clarify TimescaleDB strategy early

Document exactly where hypertables are used and where plain Postgres remains authoritative.

Suggested:

- Timeseries-heavy metrics (`training_load_samples`, run telemetry) => hypertables.
- Operational entities (templates, runs, users, ACLs, teams) => standard Postgres tables.

Add retention/compression policies and backfill strategy in docs.

## 8) Define offline sync conflict policy by entity category

`core-sync` needs deterministic rules by model type:

- Append-only entities (logs/messages): merge by timestamp/sequence.
- Mutable entities (templates/profile): server-authoritative + version vector/etag.
- Sensitive entities (signatures): immutable, conflict => explicit user workflow.

Also add conflict observability metrics from day one.

## 9) Make RBAC + ABAC explicit in `core-access`

Keep baseline RBAC, but support ABAC predicates where needed:

- Tenant scope, team membership, resource ownership, relationship-based checks (coach-athlete).
- Decision points centralized in policy service functions, not scattered decorators.

## 10) Treat signatures and audit as a compliance subsystem, not just utility APIs

Add requirements for:

- Key rotation and revocation.
- Signed payload canonicalization version.
- Audit tamper evidence (hash chaining or anchored checkpoints).
- Verification tooling for external auditors.

## 11) Define a query/read-model strategy for analytics and dashboarding

`core-run-pipeline` and references imply projections; make it explicit:

- Projection workers with replayable event streams.
- Versioned projection schemas.
- Rebuild tooling per projection.

**Why:** Avoid coupling UI directly to transactional tables.

## 12) Add strict API error envelope and problem taxonomy

Standardize backend errors:

- Stable error code, message, context object, retryable flag.
- Validation and policy denial shapes.
- Correlation ID propagation to frontend.

This dramatically improves operational debugging and UX consistency.

## 13) Add package templates to enforce consistency

Given 30+ packages, create scaffolds for both Python and TS packages with:

- Standard README sections.
- Lint/test config.
- Public API export pattern.
- Changelog and release metadata.

This reduces drift and accelerates onboarding.

---

## Gaps in Phase Plan and Suggested Adjustments

## 14) Add an explicit “Phase 0.5 — Platform quality gates”

Before identity/runtime features, set up:

- CI pipelines per package.
- Coverage thresholds.
- Contract tests and API diffing.
- Security scanning + dependency policies.

## 15) Pull “minimal observability” earlier (Phase 1)

Require baseline telemetry early:

- Structured logs with request/run IDs.
- OpenTelemetry traces across API + workers.
- Metrics for latency, sync conflicts, signature failures, rule-eval failures.

## 16) Keep messaging out of core critical path

Given “stub in v1,” avoid integrating it into shared runtime dependencies now.
Define seam interfaces only; postpone storage/protocol decisions.

## 17) Add formal non-functional SLOs per app shell

At least:

- API p95 latency targets.
- Offline sync reconciliation time targets.
- Wizard completion failure budget.
- Audit verification performance target.

These constraints should guide design choices in persistence and caching.

---

## Package-Level Boundary Adjustments

1. Consider merging `core-calendar` into `core-scheduling` unless you foresee multiple calendar rendering backends. If not merged, define a strict “view model only” contract.
2. Keep `web-features-shared` aggressively minimal (settings/profile/chrome). Anything domain-colored belongs in domain TS packages.
3. `core-goals` should remain generic milestones/objectives; ensure fitness metrics types stay in `domain-fitness`.
4. `core-content` should avoid becoming CMS-heavy in v1; preserve a simple article/help model.

---

## Security and Compliance Checklist (Should Be Added to Plan)

- Threat model document per app shell and shared platform.
- Token/session revocation semantics + device session management.
- Tenant boundary tests (negative tests mandatory).
- Data retention and deletion policy matrix by entity.
- PII classification and field-level encryption policy where applicable.
- Backup/restore and key-rotation runbooks.

---

## Recommended “Definition of Done” Enhancements

For each package PR:

- Public API documented and diff-checked.
- Contract tests added/updated.
- Security/privacy impact noted.
- Metrics/logging added for new runtime paths.
- Migration/backward-compatibility note included.

For each app-phase milestone:

- Demo script + reproducible seed data.
- SLO checks and baseline dashboards.
- Failure-injection test for at least one core workflow.

---

## Specific Open Questions You Should Resolve Now

1. What is the authoritative source for actor identity in offline flows before sync?
2. Can a run span template version upgrades, and how are partial runs migrated?
3. Is tenant portability required (export/import/move) for industrial customers?
4. How will plugin migrations be ordered relative to core migrations?
5. What is the deprecation period for shared contracts consumed by app shells?

---

## Suggested Next 2 Weeks (Concrete)

1. Create 6 ADRs (tenancy, events/outbox, contracts, signatures, sync conflict model, plugin compatibility).
2. Build package scaffolding generators for Py/TS with quality gates baked in.
3. Implement `core-domain-contracts` + TS generation + fixture round-trip tests.
4. Implement minimal event envelope and outbox writer in `core-run-pipeline`.
5. Add observability baseline (logs/traces/metrics) and correlation IDs in `core-api-runtime`.

This sequence reduces integration risk early while preserving your current phase plan.

---

## Final Verdict

Your architecture is already strong and directionally correct. If you add:

- stricter boundary enforcement,
- explicit compatibility/version policies,
- early eventing + observability foundations,
- and compliance-grade audit/signature operational design,

you will have a robust, scalable platform that can genuinely support all three products without domain entanglement.

---

## Evaluation of the Proposed Feature/Layer Draft (Calendar, Workflow, Forms, Groups)

This section evaluates the newly proposed structure against `plan.md`.

## Alignment With the Existing Plan

The draft aligns well with several core decisions already captured in `plan.md`:

1. **Calendar/Schedule as a first-class platform capability** matches `core-scheduling`, `core-calendar`, and `web-calendar`.
2. **Workflow + Forms + Document approvals** maps directly to `core-templates`, `core-run-pipeline`, `web-wizard-runtime`, and `core-signatures-audit`.
3. **Groups with role/permission controls** matches `core-access` and `core-teams`.
4. **JSON-centric user/application preferences** aligns with `core-users` and contract-first schemas in `core-domain-contracts`.
5. **SQLite for development and Timescale/Postgres for production time-series workloads** is directionally consistent with the unified stack assumptions.

## Gaps / Ambiguities to Resolve Before Implementation

The draft contains strong intent, but several elements need clarification so implementation is safe and consistent:

1. **Layer numbering collides (multiple “Layer 1” concerns).**
   - Current draft labels Data, Frontend orchestration, Users, Workflow, Approvals, and Schedule all as Layer 1.
   - Recommendation: map to the existing three-layer architecture in `plan.md`:
     - Platform core (backend services/contracts),
     - Domain plugins (fitness/organizer/industrial),
     - App shells (UI composition + integrations).

2. **Data-cross-reference rule is currently too restrictive/unclear.**
   - Statement: only user/group structures are expected to cross-reference, while other tables may not.
   - Risk: schedule entries, forms, approvals, and logs necessarily require relational links (`appointment -> form template -> questions -> approval record -> actor`).
   - Recommendation: allow controlled foreign keys and reference IDs through contract-governed schemas, rather than forbidding cross references globally.

3. **Ownership and admin semantics need canonical policy definitions.**
   - “Must contain one and only one owner” is good, but must define transfer and fail-safe behavior.
   - Add explicit rules for:
     - owner transfer,
     - preventing orphaned groups,
     - last-admin constraints,
     - system-admin override policy,
     - immutable audit trail for role changes.

4. **Approvals/signatures need compliance-grade detail.**
   - “Digital signatures” is insufficient for implementation.
   - Must define:
     - signature canonicalization format/version,
     - key management and rotation,
     - signer identity binding,
     - verification API behavior,
     - tamper-evident audit chaining.

5. **Schedule data model should separate planned vs logged entities cleanly.**
   - Draft already hints at this distinction (excellent).
   - Recommendation:
     - Planned activities: recurrence + template references + assignment metadata.
     - Logged activities: immutable event/log records + typed measurement rows + schema version markers.

6. **“Source Control” inside forms/documents needs a concrete interpretation.**
   - Clarify whether this means:
     - template version history, approvals, and diffing, or
     - true Git-style branching/merging.
   - For v1, prefer controlled template versioning via `core-templates` + promotion workflow.

7. **Calendar interface integrations should be explicit plugin adapters.**
   - Google/Office calendar connectivity should be implemented as integration adapters at app/domain edge, not platform-core hard dependencies.

8. **Messaging encryption requirement is high-stakes and domain-specific.**
   - “Encrypted for medical industry” implies compliance obligations.
   - Keep `core-messaging` minimal in early phases; define interface seams first, then add regulated messaging controls behind explicit compliance requirements.

## Recommended Mapping of Your Draft to the Unified Package Plan

- **Groups / roles / permissions** → `core-access`, `core-teams`
- **Users / preferences / auth** → `core-users`, `core-auth`, `web-auth-session`
- **Workflow / forms / docs** → `core-templates`, `core-run-pipeline`, `web-wizard-runtime`, `web-form-editors`
- **Approvals / signatures / audit** → `core-signatures-audit`
- **Schedule / calendar** → `core-scheduling`, `core-calendar`, `web-calendar`
- **Fitness utilities + workout manager** → `domain-fitness` (not platform core)
- **LLM + MCP integration** → app/domain integration seam; avoid placing model-provider coupling in core packages initially

## Suggested Priority Sequence for This Draft

1. Finalize canonical entities and ID conventions (users, groups, templates, runs, schedule items, logs).
2. Lock permission and ownership rules (including last-admin and owner transfer invariants).
3. Define planned vs logged schedule schemas and recurrence behavior.
4. Define approval/signature trust model and verification flow.
5. Add external integration contracts (Google/Office calendars, maps, LLM providers) as adapters.

## Bottom Line

Your draft is strongly aligned with the strategic architecture and can fit cleanly into the existing plan with moderate refinement. The most important next step is turning the current conceptual bullets into explicit contracts/invariants so package boundaries remain clean and implementation risk stays low.
