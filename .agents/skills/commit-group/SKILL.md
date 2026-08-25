---
name: commit-group
description: Use when the user wants to analyze current git changes and suggest logical commit groupings with commit messages.
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git rev-parse:*), Bash(git submodule:*), Bash(git -C:*)
---

# Commit Group

Analyze the current git working tree and suggest how to group changes into focused, atomic commits.

## Step 1: Gather context

Inspect the current git state:
- `git status --short`
- `git diff --stat HEAD`
- `git diff HEAD`

If the repository may contain nested isolated git repositories or submodules:
- Identify the current repository boundary with `git rev-parse --show-toplevel`
- Check known submodules with `git submodule status`
- When a dirty path is itself a separate git repository, inspect it with `git -C <path> status --short`, `git -C <path> diff --stat HEAD`, and `git -C <path> diff HEAD`
- Treat each nested git repository as its own commit target. Do not infer inner changes from only the parent repository's submodule pointer or dirty marker.

## Step 2: Analyze and suggest groups

Based on the diff above, identify logical groupings by:
- **Purpose**: features, bug fixes, refactors, config changes, tests, docs
- **Module/domain**: changes that belong to the same feature area or layer
- **Dependency order**: if commits must land in a specific sequence, say so
- **Repository boundary**: changes in different isolated git repositories must be committed from their own repository unless the user explicitly asks for parent submodule pointer commits too

## Step 3: Output format

For each suggested commit group, provide:

### Group N: `<conventional-commit message>`

**Repository:** `<path to git repository root, or . for the current repository>`

**Files to stage:**
```bash
git add <file1> <file2> ...
```
Use `git -C <repo-path> add <inner-file>` when the files belong to a nested repository.

**Why together:** One sentence explaining the cohesion.

## Rules

- Use [Conventional Commits](https://www.conventionalcommits.org/) format: `type(scope): description`
- Choose `scope` from the changed domain, feature, package, or layer inside the git repository being committed.
- Do not use the repository folder name, app wrapper name, or submodule mount path as the scope just because it appears in the path. For example, for isolated repos mounted at `apps/api` or `apps/web`, avoid scopes like `api`, `web`, `apps-api`, or `apps-web` unless the diff is genuinely about that top-level app boundary.
- In nested repositories, write file paths and commands relative to that nested repository when practical, using `git -C <repo-path> add <inner-file>` and `git -C <repo-path> commit -m "..."`
- Each commit should be independently deployable and reviewable
- Unrelated changes must never share a commit
- If a file spans multiple concerns, call it out and suggest splitting it
- End with the exact `git add`/`git commit -m` commands ready to copy, using `git -C <repo-path>` for nested repositories
