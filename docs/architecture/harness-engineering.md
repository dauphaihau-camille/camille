# Harness Engineering

Camille uses harness engineering to make agent-assisted changes safer, more
deterministic, and easier to review.

The goal is not to write one large instruction file. The goal is to make the
repository itself guide an agent toward the right context, constrain common
mistakes, and provide fast verification feedback.

## Purpose

Harness engineering gives humans and agents a shared operating model:

1. identify the area that owns the requested behavior
2. read the nearest guidance instead of scanning everything
3. use repo maps for known feature areas
4. make scoped changes inside the owning boundaries
5. run focused or full verification before finishing

This keeps guidance close to the code it governs and avoids turning the root
instructions into a large prompt.

## Components

### Repository Routers

- `AGENTS.md` routes work at the repository level.
- `apps/api/AGENTS.md` routes API workspace work.
- `apps/api/api/AGENTS.md` routes NestJS API app work.
- `apps/web/AGENTS.md` routes web monorepo work.

These files should stay short. They should answer where to look next, not
repeat every rule in the repository.

### Focused Guidance

Focused agent guidance lives beside the area it governs:

- `apps/api/api/agent-skills/`
- `apps/web/agent-skills/`

These files describe local architecture, source placement, testing, UI, and
boundary rules. They should be loaded selectively for the current task.

### Repo Map Tooling

`tools/agent/repo-map` maps stable feature topics to their docs, code surfaces,
guidance files, and verification commands.

Examples:

```bash
./tools/agent/repo-map document
./tools/agent/repo-map document-edit
./tools/agent/repo-map auth
./tools/agent/repo-map ai-assistance
./tools/agent/repo-map workspace
./tools/agent/repo-map publish
```

Use repo maps before broad repository search when starting work on a known
feature area.

### Verification Harness

`scripts/verify` provides a single root entrypoint for checks:

```bash
./scripts/verify
./scripts/verify api
./scripts/verify web
./scripts/verify app
./scripts/verify marketing
./scripts/verify shared
```

The purpose is to make the expected verification path obvious from the root
instead of requiring each agent to rediscover package-specific commands.

### Executable Architecture Checks

Camille's API and web workspaces already have focused prose guidance for source
placement, API contracts, cross-domain boundaries, testing, and co-located web
files.

Executable architecture checks should be added gradually, only when a rule is
stable and can pass the current codebase without noisy failures. Candidate
checks include:

- API `domain/` files should not import technical framework or vendor packages.
- API `shared/` should not import domain modules.
- API `integrations/` should not import business domain internals.
- Web shared packages should not import app-specific modules from `apps/app` or
  `apps/marketing`.

## Workflow

For humans using agents on Camille:

1. Start from the root `AGENTS.md`.
2. Determine the owning area: API, web app, marketing app, shared web package,
   docs, or tooling.
3. Read the nearest `AGENTS.md`.
4. For known feature areas, run `./tools/agent/repo-map <topic>`.
5. Read only the focused guidance and docs relevant to the task.
6. Make the smallest scoped change that preserves existing boundaries.
7. Run the focused verification command, or `./scripts/verify` for cross-area
   work.

## Maintenance Rules

- Keep root instructions short and routing-focused.
- Prefer local guidance beside the code it governs.
- Prefer executable checks over prose-only architecture rules when the rule is
  stable.
- Add repo-map topics only for feature areas with stable docs and code surfaces.
- Add architecture checks only when they can pass the current codebase.
- Avoid adding guidance that repeats existing docs without changing agent
  behavior.

## Current Maturity

Camille currently has the first practical harness layer:

- layered `AGENTS.md` routing
- API and web focused guidance
- repo-map topics for major feature areas, including AI Assistance
- root verification commands

The next maturity step is to expand executable checks gradually as existing
architecture rules become stable enough to enforce without noisy failures.
