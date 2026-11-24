# ADR-0007: License - GNU AGPL-3.0

- **Status:** Proposed (adopted in the repository; maintainer may change before first release)
- **Date:** 2026-11-24

## Context
The project is non-profit and must stay free for churches. It will be offered as a hosted multi-tenant service, so improvements made by anyone who runs a modified hosted version should flow back to the community. Some programs and funders also require an OSI-approved license.

## Decision
License the repository under **AGPL-3.0**. Content in `docs/` and rule packs follow the same license unless stated otherwise.

## Consequences
- (+) Hosted forks must share their source, protecting the non-profit mission.
- (+) OSI-approved open-source license.
- (-) Some organizations avoid AGPL. If adoption suffers, relicensing is possible while the project has few contributors, so consider a Contributor License Agreement or DCO early.

## Alternatives considered
MIT / Apache-2.0: maximum adoption, but allows closed commercial forks of a project intended to stay free for churches.
