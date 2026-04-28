# 06 — Functional Requirements (Consolidated)

This document consolidates feature-level requirements that were previously spread across Organizer and Process Wizard feature docs.

## 1) Authentication, Identity, and User Lifecycle

- Support OAuth2/JWT auth with refresh rotation and optional TOTP MFA.
- Support guest/bootstrap sessions where product mode allows pre-auth trial flows.
- Provide password reset and account recovery lifecycle.
- Expose profile/security APIs for:
  - profile updates
  - email/password change
  - MFA enrollment/reset
  - session/device revocation
- Persist user preferences, including units and locale.
- Maintain parameter visibility/privacy preferences (especially relevant for coaching metrics).

## 2) Wizard/Template/Run Model

- Templates are versioned documents with immutable historical revisions.
- Runs execute a specific template version and write canonical run records.
- Runs support:
  - save draft/autosave
  - complete/cancel flows
  - branch/decision steps
  - optional signature capture when policy requires
- Reference outputs from prior runs are queryable for downstream templates.

## 3) Scheduling and Calendar

- Core scheduler supports RRULE recurrence, exception dates, planned occurrences, and ICS feed export.
- Calendar supports day/week/month/year surfaces and filters by team, user, type, and status.
- Domain overlays:
  - Industrial: procedure recurrence and planned compliance actions.
  - Organizer: tasks/habits/personal planning blocks.
  - Fitness: session planning and coach-prescribed workouts.

## 4) Fitness Domain Requirements

- Exercise catalog with metadata and progression-ready attributes.
- Workout editor and runtime separation:
  - authoring views for coaches/program designers
  - execution views for athletes/end users
- Intake and assessment wizard flows.
- FIT ingestion and training-metrics computation.
- Team-aware coach↔athlete access controls.

## 5) Industrial Domain Requirements

- Multi-tenant procedure execution with strict auditability.
- Step catalog supports branch, deviation, checklists, captured data, and signatures.
- Canonical records are signable and verifiable.
- Access groups enforce role-based procedure visibility and actions.

## 6) Organizer Domain Requirements

- Personal tasks/lists and habits.
- Personal goals/milestones with badges.
- Scheduling + calendar-first user experience.
- Optional messaging and content modules as product tiers evolve.

## 7) Messaging and Content

- Messaging remains phase-0 stub but must preserve API seams for DM/group/broadcast growth.
- Content service supports articles/help items/system notices.

## 8) Offline Sync

- Provide sync helper APIs for:
  - fetching changes since cursor/version
  - applying user-local mutations
  - conflict-resolution hooks by entity type
- Must support offline-first UX in constrained environments.

## 9) UI/UX Standards (Shared)

- Clean, functional, low-friction workflows.
- Consistent hierarchy, spacing, and navigation.
- Responsive layouts for desktop/tablet/mobile.
- Accessible interactions with predictable controls and feedback.
- Domain-specific branding can vary without breaking shared UI primitives.

## 10) Validation and Acceptance

- Each domain/app must pass smoke flow:
  1. Authenticate
  2. Open template/wizard
  3. Complete run
  4. Verify canonical record
- Fitness app additionally validates workout authoring + execution + metrics ingestion.
- Industrial app additionally validates signature verification + audit trace.
