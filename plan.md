---
Status: Implementation
Last Updated: 2026-04-28
Relates to: [Program Plan]
---
# App Plan Roadmap (Implementation Index)

This repository has moved from monolithic planning into milestone-driven execution.

## Active Milestones

1. [01-metabolic-core](milestones/01-metabolic-core.md)
2. [02-data-integration](milestones/02-data-integration.md)
3. [03-client-management](milestones/03-client-management.md)

## Architecture Direction (Cross-Cutting)

Generalizable capabilities should be implemented in platform/base architecture first, then composed into app shells and domains. This includes:

- change approvals and workflow gating,
- signatures/audit trails and rule enforcement,
- team/role access patterns,
- shared scheduling + template orchestration primitives.

Milestone planning should call out when a feature is promoted to base architecture vs. app-specific implementation.

## App Documentation Segregation

To make app boundaries explicit, planning and implementation docs should be split by target product:

- **Coach app (`app-coach`)** docs and milestones: coaching workflows, coach↔athlete/team operations, training-plan assignment, session review.
- **Organizer app (`app-organizer`)** docs and milestones: personal tasks/lists/calendar/habits/goals and individual planning flows.
- **Platform/base architecture** docs and milestones: reusable services, policies, contracts, and UI/runtime primitives consumed by both apps.

Any milestone touching both apps should include separate subsections for Coach impact and Organizer impact.

## Repository Context

- Design and wizard documentation: `docs/wizards/`
- Organizer-specific source-aligned docs: `src/organizer/`
- Platform architecture and app shell docs: `src/platform/`
- Coach app composition docs: `src/platform/apps/coach/`
- Organizer app composition docs: `src/platform/apps/organizer/`
- Schemas and MCP definitions: `specs/`
- Historical review artifacts: `reviews/`

## Source of Truth Documents

- AI context and formula guardrails: [`system-prompt.md`](system-prompt.md)
- Agent behavior rules: [`.clinerules`](.clinerules), [`.cursorrules`](.cursorrules)
