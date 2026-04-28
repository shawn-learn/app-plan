---
Status: Draft
Last Updated: 2026-04-28
Relates to: [03-client-management]
---
# Milestone 03: Client Management

## Objective
Deliver coach-facing client management capabilities while extracting reusable approval/access controls into platform core.

## Scope
- Coach app: client roster, goals, and scheduling surfaces.
- Coach app: team/trainer relationship flows.
- Coach app: template-driven workout planning and execution tracking.
- Platform/base architecture: shared change-approval workflow contracts and policy checks.

## Deliverables
- `app-coach` domain boundaries and service contracts for client lifecycle.
- Coach UI modules for client lifecycle and workout assignment.
- Platform-level data model and service contracts for coach-client permissions and approval gating.
- Explicit non-goals list for `app-organizer` to avoid cross-app scope creep in this milestone.
