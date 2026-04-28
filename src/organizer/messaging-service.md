# messaging-service

## 1. Purpose

`messaging-service` is the planned real-time communication layer: direct messages, group chats, team broadcasts, practitioner announcements, attachments (including video on a paid entitlement), and end-to-end encryption for physiotherapist↔patient conversations. **Today it exists only as Phase 0 stubs gated behind feature flags**; this document describes the target shape so the library boundary is correct from day one.

## 2. Scope

**In scope**
- Conversations (DM, group, team, broadcast) and their membership.
- Messages with text, attachments, and reactions.
- Attachments stored in S3 (image, audio, video — video gated by `VIDEO_MESSAGING_ENABLED`).
- E2EE for practitioner↔patient conversations: key exchange, sealed messages, server cannot read content.
- Conversation export.
- Real-time delivery via WebSocket / push.
- Feature-flag enforcement (`MESSAGING_ENABLED`, `VIDEO_MESSAGING_ENABLED`, `DEV_BYPASS_MESSAGING_AUTH`).

**Out of scope**
- Audience resolution (team broadcast → which users) → delegated to `team-service`.
- User identity / blocklists → delegated to `user-service`.
- Attachment-level virus scanning (assumed to be platform infra, not library code).

## 3. Current Capabilities

The README's "Phase 0 Messaging" section is the authoritative current-state description: stubbed endpoints behind feature flags, optional Alembic tables created when flags are enabled, S3 configuration via `.env.example`. There are no `conversations`/`messages`/`attachments` routers in `app/routers/` yet. `docs/MESSAGING_REQUIREMENTS.md` is referenced by the README but is not present in the current `docs/` listing — it should be reconstructed alongside this requirements doc when implementation begins.

## 4. Public API Surface (target)

- HTTP: `/conversations`, `/conversations/{id}/messages`, `/attachments`, `/conversations/{id}/export`.
- WebSocket: `/ws/conversations/{id}`.
- Python: `send_message(user, conversation, payload)`, `list_unread(user)`, `seal_message_for(recipients, ciphertext)`.

## 5. Data Model (target)

`conversations`, `conversation_members`, `messages`, `message_attachments`, `message_reactions`, `message_read_receipts`, `e2ee_keys`. All gated by feature flags at migration time.

## 6. Dependencies

- Depends on: `user-service`, `team-service`, `auth-service`, `shared-schemas`.
- Must NOT depend on: `activity-service`, `template-service`, `goals-service`. Messaging is pure communication; activity context is referenced by ID only.

## 7. Cross-Cutting Concerns

- **E2EE**: server must never log plaintext for practitioner↔patient conversations.
- **Feature flags**: every endpoint must respect `MESSAGING_ENABLED`; video paths must respect `VIDEO_MESSAGING_ENABLED`.
- **Storage**: S3 lifecycle policies for attachments (retention, deletion).
- **Real-time**: WebSocket scaling considerations (single-process today; future broker like Redis pub/sub).

## 8. Open Questions / Known Gaps

- `docs/MESSAGING_REQUIREMENTS.md` is referenced but missing — needs to be authored.
- Celery is in `requirements.txt` but not yet wired; intended for async delivery / push.
- Practitioner role definition lives in `team-service` and gates E2EE here; the contract needs explicit documentation.
- The "paid entitlement" model for video messaging has no current billing integration.
