# core-tenancy

**Tier 1 — Identity.**

Multi-tenant foundation: per-tenant DB routing, tenant onboarding, tenant registry. Used by the industrial app; available but unused by organizer/coach apps.

## Source material

Process Wizard `system-requirements.md` REQ-TENANT-001..003 and the "Tenant Provisioning Decisions" section.

## Responsibilities

- Tenant registry (UUID-named DBs, encrypted DSNs at rest, optional credential rotation).
- Tenant resolver: read `tid` from subdomain or header → fetch DSN → warm connection pool.
- Operator-only onboarding API (`POST /ops/tenants`):
  - Synchronous create within 10s (201) or async with status URL (202).
  - Idempotent via `X-Idempotency-Key`.
  - Creates DB from template or runs Alembic migrations, seeds defaults, creates tenant admin with activation link, registers routing metadata, emits audit event.
- Operator auth realm (`X-Operator-Auth`) separate from tenant auth.
- Status polling: `GET /ops/tenants/{tenant_id}`.

## Dependencies

- External: `sqlalchemy`, `alembic`
- Internal: `core-domain-contracts`, `core-data-access`, `core-signatures-audit` (for provisioning audit events)

## Consumers

`app-industrial` mounts onboarding routes. `core-data-access` consults the resolver for per-request session routing.
