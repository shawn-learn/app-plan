# core-signatures-audit

**Tier 2 — Platform.**

Canonical-record signing and append-only audit log. Used heavily by industrial; opt-in for fitness (rehab signoffs); typically unused by organizer.

## Source material

Process Wizard `system-requirements.md` digital-signature and audit-log requirements.

## Responsibilities

- Build a **canonical record** from a completed run: immutable JSON of metadata, step answers, and signature binding.
- Sign the canonical record (server-signed, EdDSA/Ed25519, `kid`-rotated).
- Bind signer identity, timestamp, and device metadata.
- Prevent duplicate signatures per run unless an admin revokes a prior one.
- Append-only audit log capturing security-sensitive events (logins, role changes, group membership updates, signature applies/revokes, tenant provisioning).
- Verification API: `verify(canonical_record) → ok | tamper_detected`.

## Dependencies

- External: `cryptography`, `sqlalchemy`, `pydantic`
- Internal: `core-domain-contracts`, `core-users`, `core-data-access`

## Consumers

`core-run-pipeline` invokes signing during completion. `core-api-runtime` exposes verification endpoints. `app-industrial` is the primary consumer.
