# Church Care Platform

> A non-profit, open-source, multi-tenant platform for **pastoral service, visitation, care and follow-up** in churches.
> منصة غير ربحية مفتوحة المصدر لإدارة الخدمة الكنسية والافتقاد والرعاية والمتابعة.

**Status:** 📝 Planning / PRD stage

## Vision
Help servants know **who to visit first, and why**, by combining people & family data, automatic attendance (face recognition devices), and an explainable rules engine, while protecting privacy and dignity.

## Planned Modules
- Multi-tenant hierarchy: Eparchy → Church
- People, families & automatic family-tree suggestions
- Automatic attendance via face-recognition devices (+ QR/manual fallback)
- Smart, explainable visitation recommendations (rule packs)
- Education classes, tutoring & activities
- Parent portal for remote follow-up
- Public API & widgets for each church's website
- Media library with points & quizzes
- Brothers of the Lord care management

## Documentation
- 📄 [Product Requirements Document](docs/PRD.md)

## Repository Layout
```
apps/api               Backend API
apps/web               Web app / parent portal (PWA)
apps/edge-connector    Local service for attendance devices
packages/rules-engine  Declarative visitation rules engine
rules/packs            Pre-built rule packs (YAML)
docs/                  PRD, ADRs, guides
```

## Contributing
Contributions from developers, servants and pastors are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Privacy & Ethics
This system handles sensitive data about children and vulnerable people. Read [SECURITY.md](SECURITY.md) before contributing. The platform supports shepherds; it never replaces them.

## License
To be decided (see PRD open question #7). Placeholder: AGPL-3.0.
