# core-content

**Tier 2 — Platform.**

Articles, in-app help content, system log.

## Source material

`Organizer/content-service.md`.

## Responsibilities

- Markdown articles served via API.
- Help-context index: `(route, section) → help_article_id`.
- In-app help drawer payloads.
- System log surface for ops/admin views.

## Dependencies

- External: `markdown-it`, `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-auth`, `core-data-access`

## Consumers

`web-features-shared` mounts the help drawer. App shells seed content per their domain.
