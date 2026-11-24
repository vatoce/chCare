# ADR-0002: Multi-tenancy via PostgreSQL Row-Level Security

- **Status:** Accepted
- **Date:** 2026-11-24

## Context
Hierarchy is Eparchy -> Church. Data includes minors and vulnerable people. A single bug must not leak data across churches.

## Decision
- Shared database, shared schema, `tenant_id` (church) on every tenant-owned table.
- RLS policies enforce isolation; each request runs in a transaction that sets `SET LOCAL app.tenant_id` and `app.user_id`.
- The application DB role is **not** a superuser and cannot bypass RLS.
- Eparchy-level access uses separate aggregate views / materialized reports; personal-data drill-down requires an explicit, audited grant by the church.
- Sensitive fields (Brothers of the Lord cases, health notes) use field-level encryption with per-tenant keys.
- Automated tenant-isolation tests run in CI and must fail the build on any cross-tenant read/write.

## Consequences
- (+) Defense in depth: isolation does not rely only on application code.
- (-) Every query path must run within the tenant-scoped transaction; connection pooling must be transaction-mode aware.
