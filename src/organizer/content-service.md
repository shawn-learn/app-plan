# content-service

## 1. Purpose

`content-service` is the catch-all for read-mostly content owned by the platform rather than the user: in-app articles, contextual help content, the help-context registry, and the structured system log. These concerns are small individually but share a pattern (mostly editorial / observational, low write volume, no domain entanglement) that makes a single library a reasonable home.

## 2. Scope

**In scope**
- Articles (`/articles`): markdown content rendered in the app.
- Help content (`/help`, `/help-content`) and the help-context registry (`app/help_contexts.py`).
- System log (`/system-log`): structured app event log persisted in the DB (`docs/LOGS_AND_TROUBLESHOOTING.md`).
- Markdown rendering helpers and content seeding.

**Out of scope**
- User-generated messages (→ `messaging-service`).
- Auth events (originate in `auth-service`; this lib only stores them).

## 3. Current Capabilities

Routers: `app/routers/articles.py`, `help.py`, `help_content.py`, `system_log.py`. Schemas: `app/schemas/articles.py`, `help_content.py`, `system_log.py`. Helpers: `app/articles/`, `app/help_contexts.py`. Top-level `help/` directory holds source content. Frontend: `ArticlePage.jsx`, `HelpDrawer.jsx`, `ContextHelpIcon.jsx`, `MarkdownRenderer.jsx`, `frontend/src/contextHelp.js`, `frontend/src/useHelp.js`. Doc: `docs/HELP_ITEMS.md`.

## 4. Public API Surface

- HTTP: `GET /articles`, `GET /articles/{slug}`, `GET /help`, `GET /help-content`, `POST /system-log`, `GET /system-log` (admin-gated).
- Python: `log_event(level, message, context)`, `help_for(context_key)`, `get_article(slug)`.

## 5. Data Model

Owns: `articles`, `help_content`, `system_log`. Help contexts are code-defined (in `app/help_contexts.py`).

## 6. Dependencies

- Depends on: `auth-service` (admin gate on log/article writes), `shared-schemas`.
- Must NOT depend on: any other Tier 2 service. Other services *use* `log_event` but this lib does not call them back.

## 7. Cross-Cutting Concerns

- **Logging contract**: `log_event` is the canonical sink — every Tier 2 service uses it instead of stdout for auditable events.
- **Authorization**: article writes and log reads are admin-only; help reads are public.
- **i18n**: articles and help content are localized via per-locale slugs.

## 8. Open Questions / Known Gaps

- "Admin" today is loosely defined; needs a formal role check (likely a flag on `team_memberships` or `users`).
- `help_contexts.py` hardcodes context keys — at scale they may need to live in the DB.
- Log retention policy is not documented.
