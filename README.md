<div align="center">

# chCare

### Church Care Platform

**Open-source pastoral care software for churches: know who to visit first, and why.**

منصة مفتوحة المصدر وغير ربحية للخدمة الكنسية والافتقاد والرعاية والمتابعة

[![CI](https://github.com/vatoce/chCare/actions/workflows/ci.yml/badge.svg)](https://github.com/vatoce/chCare/actions/workflows/ci.yml)
[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg)](LICENSE)
![Status: pre-alpha](https://img.shields.io/badge/status-pre--alpha-orange)
![Non-profit](https://img.shields.io/badge/non--profit-yes-green)

</div>

> **Project status: pre-alpha (planning and foundation).** There is no runnable product yet. The repository currently contains the product requirements (PRD v0.3), architecture decision records, and the monorepo scaffold. We are looking for contributors and a first pilot church. See the [roadmap](#roadmap) and [how to help](#how-you-can-help).

## Why this project exists

Churches serve hundreds or thousands of people, yet the most valuable part of that service, personal pastoral visitation and follow-up, is the hardest to scale. People quietly drift away, children stop attending, and families in need go unnoticed, because servants rely on memory, paper lists and scattered spreadsheets.

Most existing church-management tools are proprietary, priced per seat in foreign currency, English-first, and designed around Western congregational workflows. We are not aware of an open-source, **Arabic-first (RTL)**, **multi-tenant** platform built around pastoral visitation (*al-iftiqad*) for liturgical churches, with a diocese/eparchy level above individual parishes. This project aims to fill that gap and to be free for every church.

> The software supports shepherds. It never replaces them. Every recommendation is a suggestion that a human reviews.

## What it will do

| Area | Summary |
|---|---|
| **Multi-tenant** | Eparchy → Church hierarchy with strict data isolation (PostgreSQL Row-Level Security) |
| **People & families** | Profiles, households, family trees, and automatic, explainable family-link suggestions |
| **Attendance** | Automatic capture from face-recognition devices (plus QR and manual fallback), offline-capable Edge Connector |
| **Smart visitation** | A rules engine ranks who needs a visit or call, always showing *why*; rules are YAML files that priests and servants can review |
| **Served-person accounts** | Every served person has an account with daily, monthly and yearly growth tasks (spiritual, educational, service) and progress tracking |
| **Parents** | Parents follow their children's attendance, classes and progress remotely |
| **Mobile (iOS & Android)** | React Native + Expo app, offline-first for servants in the field |
| **Education & activities** | Spiritual-education classes, tutoring, trips and clubs |
| **Media library** | Videos and quizzes with points and badges, child-safe mode |
| **Brothers of the Lord** | Dignified, confidential case management for people in need |
| **Church websites** | Public API and embeddable widgets for each church's own site |

Full details: [Product Requirements Document](docs/PRD.md).

## Design principles

1. **People first, data second.**
2. **Human in the loop:** the system suggests, shepherds decide.
3. **Explainable by default:** no black-box scoring.
4. **Privacy and dignity over convenience:** the data covers children and vulnerable people. No sacramental or confession data is ever stored.
5. **Works offline, works on cheap phones.**
6. **Free and open for churches.**

## Reusable building blocks

Several pieces are useful beyond churches and are planned as standalone packages:

- **Declarative, explainable rules engine** (`packages/rules-engine`): YAML rule packs with JSON Schema validation, scores plus human-readable reasons.
- **Arabic name normalization and entity resolution** for duplicate detection and family-link suggestions.
- **Multi-tenant PostgreSQL RLS reference setup** with a tenant-isolation test suite.
- **Offline-first sync patterns** for React Native with an encrypted local database.
- **Attendance device connector abstraction** for face-recognition and other terminals.

## Architecture at a glance

TypeScript monorepo · NestJS (modular monolith) · PostgreSQL with RLS · Redis + BullMQ · Next.js · **React Native + Expo** · Node.js Edge Connector.

- [Technology stack](docs/PRD.md#7-technology-stack-approved)
- [Architecture Decision Records](docs/adr/README.md)

```
apps/api               NestJS backend
apps/web               Next.js web app, admin console, church sites
apps/mobile            React Native (Expo) iOS/Android app
apps/edge-connector    Local service for attendance devices
packages/shared        Shared schemas and types
packages/api-client    Typed API client
packages/rules-engine  Declarative visitation rules engine
rules/packs            Pre-built rule packs (YAML)
docs/                  PRD and ADRs
```

## Roadmap

| Phase | Focus | Status |
|---|---|---|
| 0. Foundation | Monorepo, CI, tenant model + RLS, auth, RBAC, i18n/RTL, mobile shell | Planned |
| 1. MVP | People & families, QR/manual attendance, first ~30 care rules, visitation list, servant app with offline sync | Planned |
| 2. Automation | Device connectors, Edge Connector, auto family suggestions, rule UI | Planned |
| 3. Engagement | Classes, parent app, served-person growth plans, public API and widgets | Planned |
| 4. Growth | Media library and points, Brothers of the Lord module | Planned |
| 5. Scale | Eparchy dashboards, self-service onboarding, ML-assisted ranking | Planned |

## How you can help

We are a volunteer, non-profit effort and welcome all kinds of contributions:

- **Developers:** NestJS, PostgreSQL/RLS, React Native/Expo, Next.js, TypeScript tooling.
- **Pastors and experienced servants:** review the pastoral rule packs and task libraries for age groups.
- **Designers and Arabic UX writers:** RTL-first, simple interfaces for non-technical volunteers.
- **Security and privacy reviewers:** the system handles data about minors and vulnerable people.
- **Translators:** Arabic and English now, other languages later.
- **A pilot church:** a parish willing to try early versions and give honest feedback.

Start with [CONTRIBUTING.md](CONTRIBUTING.md), the open issues, and the [Code of Conduct](CODE_OF_CONDUCT.md).

## Getting started (dev environment)

```bash
nvm use                 # Node 20
corepack enable
pnpm install
docker compose up -d    # Postgres + Redis
```

Application code is not scaffolded yet; follow Phase 0 in the roadmap.

## Privacy, security and ethics

Never commit real personal data; use synthetic data only. Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## License

[GNU AGPL-3.0](LICENSE). Rationale in [ADR-0007](docs/adr/0007-license-agpl-3.0.md).

---

## بالعربي

**منصة مفتوحة المصدر وغير ربحية** تساعد الكنائس على معرفة من يحتاج للافتقاد أولاً ولماذا، عبر دمج بيانات الأسر والحضور التلقائي (بصمة الوجه) مع محرك قواعد واضح الأسباب.

**أهم الإمكانيات المخططة:** نظام متعدد الكنائس تحت مظلة الإيبارشية، وشجرة العائلة والتعرف التلقائي على الروابط، وربط بأجهزة بصمة الوجه للحضور، وترشيح ذكي للافتقاد، وتطبيق أندرويد وآيفون يعمل بدون إنترنت للخدام، وحساب لكل مخدوم بمهام يومية وشهرية وسنوية، ومتابعة أولياء الأمور عن بُعد، ومكتبة ميديا بنقاط واختبارات، وإدارة إخوة الرب بخصوصية وكرامة.

**الحالة:** مرحلة التخطيط والتأسيس. المطلوب: مطوّرون، وآباء وخدام لمراجعة قواعد الافتقاد، ومصممون، ومراجعو أمان، وكنيسة تجريبية.

**مبدأ أساسي:** النظام يساعد الراعي ولا يحلّ محله، ولا يخزّن أي بيانات اعتراف أو أسرار كنسية.

التفاصيل الكاملة في [وثيقة المتطلبات](docs/PRD.md).
