# Church Care Platform

> A non-profit, open-source, multi-tenant platform for **pastoral service, visitation, care and follow-up** in churches.
> منصة غير ربحية مفتوحة المصدر لإدارة الخدمة الكنسية والافتقاد والرعاية والمتابعة.

**Status:** Planning / PRD v0.3 (tech stack approved; served-person growth plans added)

## Vision
Help servants know **who to visit first, and why**, by combining people & family data, automatic attendance (face-recognition devices), and an explainable rules engine, while protecting privacy and dignity.

## Planned Modules
- Multi-tenant hierarchy: Eparchy → Church
- People, families & automatic family-tree suggestions
- Automatic attendance via face-recognition devices (+ QR/manual fallback)
- Smart, explainable visitation recommendations (rule packs)
- Native **iOS & Android** app (offline-first) for servants and families
- Personal accounts for every served person with daily, monthly and yearly growth tasks and progress tracking
- Education classes, tutoring & activities
- Public API & widgets for each church's website
- Media library with points & quizzes
- Brothers of the Lord care management

## Tech Stack (summary)
TypeScript monorepo · NestJS · PostgreSQL (RLS) · Redis/BullMQ · Next.js · **React Native + Expo** (iOS/Android) · Node.js Edge Connector. See [docs/PRD.md](docs/PRD.md#7-technology-stack-approved) and [ADRs](docs/adr/README.md).

## Repository Layout
```
apps/api               NestJS backend
apps/web               Next.js web app & church websites
apps/mobile            React Native (Expo) iOS/Android app
apps/edge-connector    Local service for attendance devices
packages/shared        Shared schemas & types
packages/api-client    Typed API client
packages/rules-engine  Declarative visitation rules engine
rules/packs            Pre-built rule packs (YAML)
docs/                  PRD and ADRs
infra/                 Deployment configs (to come)
```

## Getting Started (dev environment)
```bash
nvm use            # Node 20
corepack enable
pnpm install
docker compose up -d   # Postgres + Redis
```
Apps are not scaffolded yet; see the roadmap in the PRD (Phase 0).

## Documentation
- [Product Requirements Document](docs/PRD.md)
- [Architecture Decision Records](docs/adr/README.md)

## Contributing
See [CONTRIBUTING.md](CONTRIBUTING.md). Never commit real personal data.

## Privacy & Ethics
This system handles sensitive data about children and vulnerable people. Read [SECURITY.md](SECURITY.md). The platform supports shepherds; it never replaces them.

## License
To be decided (PRD open question #7). Placeholder: AGPL-3.0.
