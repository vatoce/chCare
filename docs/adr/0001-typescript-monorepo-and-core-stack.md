# ADR-0001: TypeScript monorepo and core stack

- **Status:** Accepted
- **Date:** 2026-11-24

## Context
Non-profit project, small team of volunteers with mixed availability. Need one language across backend, web, mobile and edge services to keep onboarding and maintenance cheap, and to share validation schemas, API types and the rules-engine types.

## Decision
- TypeScript everywhere; pnpm workspaces + Turborepo.
- Backend: NestJS modular monolith (REST + OpenAPI).
- Database: PostgreSQL; Drizzle ORM; Redis + BullMQ for jobs.
- Web: Next.js, Tailwind, shadcn/ui (RTL-first).
- Auth: Better Auth (OIDC-ready).
- Infra: Docker Compose on a VPS (Coolify) with a path to managed Kubernetes.

## Consequences
- (+) Shared code between all apps; one skill set.
- (+) Low operating cost.
- (-) Some device SDKs are not JS-native; handled by adapters in the Edge Connector (see ADR-0004 when written).
- (-) Modular monolith requires discipline in module boundaries (enforced by lint rules).

## Alternatives considered
Django + DRF (strong admin, easier ML): revisit if the contributing team is Python-centric. Microservices: rejected for operational overhead.
