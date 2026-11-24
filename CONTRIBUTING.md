# Contributing

Thank you for helping. This is a volunteer, non-profit project that handles sensitive data, so a few rules matter more than usual.

## Ground rules
1. **Never commit real personal data.** Use synthetic data only (names, photos, phone numbers, addresses).
2. Open an issue before large changes, so we can agree on direction (see the PRD and ADRs).
3. Every change touching personal data needs a short **privacy note** in the pull request.
4. Pastoral rules in `rules/packs` should be reviewed by a priest or experienced servant before merging.
5. Tenant isolation is non-negotiable: any new table needs `tenant_id`, an RLS policy, and a test.
6. Be kind. See the [Code of Conduct](CODE_OF_CONDUCT.md).

## Good places to start
- Documentation: improve the PRD, add Arabic translations.
- Rules: propose new rule packs in `rules/packs` (YAML, with Arabic and English reasons).
- Tooling: CI, linting, TypeScript configs, dev-environment polish.
- Review: read the ADRs and challenge them.

## Workflow
1. Fork and create a branch (`feat/...`, `fix/...`, `docs/...`).
2. Keep pull requests focused and small.
3. Run `pnpm lint`, `pnpm typecheck` and `pnpm test` when applicable.
4. Describe what changed and why, and link the issue.

## Using AI assistance
AI tools may be used to help write code and docs. The contributor remains fully responsible for the result: review it, test it, and never paste real personal data into any tool.
