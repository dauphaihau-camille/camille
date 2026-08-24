# Camille Agent Index

Start here, then open the nearest guide for the area you are changing.

- `apps/api/AGENTS.md` - API workspace routing
- `apps/api/api/AGENTS.md` - NestJS API guidance and focused agent-skills
- `apps/web/AGENTS.md` - web monorepo guidance and focused agent-skills
- `docs/` - product, feature, ADR, local setup, and architecture documentation

For known feature areas, prefer `./tools/agent/repo-map <topic>` before broad
repository search.

Use `./scripts/verify` from the repository root for the default verification
path, or pass a focused target such as `api`, `web`, `app`, `marketing`, or
`shared`.

## Agent skills

### Issue tracker

Issues and specs are tracked in GitHub Issues for `dauphaihau-camille/camille`. See `docs/agents/issue-tracker.md`.

### Domain docs

This repo uses a multi-context domain-doc layout: root `CONTEXT-MAP.md`,
context-specific `CONTEXT.md` files under `docs/domain/`, and root
`docs/adr/`. See `docs/agents/domain.md`.
