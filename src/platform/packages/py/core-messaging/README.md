# core-messaging

**Tier 2 — Platform. Phase-0 stub.**

Direct messages, group chats, broadcasts, attachments, E2EE seam. Kept as a stub so the seam exists and feature flags can flip on; no real implementation in v1.

## Source material

`Organizer/messaging-service.md` (Phase-0 stub today, gated by `MESSAGING_ENABLED` and `VIDEO_MESSAGING_ENABLED` flags).

## Responsibilities (planned, not implemented in v1)

- 1:1 DMs and group chats.
- Team broadcasts.
- Attachments via S3.
- Practitioner↔patient E2EE channel (clinical scenarios).
- Read receipts, typing indicators, push hooks.

## Dependencies

- External: TBD (likely `aioboto3` for attachments, a websocket library)
- Internal: `core-domain-contracts`, `core-users`, `core-teams`, `core-data-access`

## Consumers

None in v1. The seam is preserved so `app-coach` (athlete↔coach DMs) and `app-industrial` (deviation discussions) can flip it on later without API churn.

## v1 deliverable

Models, table stubs, feature-flag-gated routers that return 501 unless flags are enabled. No real transport.
