# Result Examples

Human-only examples showing the difference between answering normally and answering with the `commit-group` skill.

## Example 1

### Prompt

```text
Help me split my current changes into good commits.
```

### Without Skill

```text
You should probably separate your changes into a few smaller commits. For example, keep feature work separate from tests and documentation. That will make the history easier to review.
```

### With `commit-group`

```text
Group 1: `feat(auth): add password reset endpoint`

Files to stage:
git add app/Http/Controllers/PasswordResetController.php routes/api.php

Why together: These files implement the new API surface for password reset.

Group 2: `test(auth): cover password reset flow`

Files to stage:
git add tests/Feature/Auth/PasswordResetTest.php

Why together: The tests verify the new endpoint behavior and should land with feature coverage.

Group 3: `docs(auth): document password reset API`

Files to stage:
git add docs/api/auth.md

Why together: Documentation is related but independently reviewable from implementation.

Commands:
git add app/Http/Controllers/PasswordResetController.php routes/api.php
git commit -m "feat(auth): add password reset endpoint"

git add tests/Feature/Auth/PasswordResetTest.php
git commit -m "test(auth): cover password reset flow"

git add docs/api/auth.md
git commit -m "docs(auth): document password reset API"
```

## Example 2

### Prompt

```text
Can you group these git changes into logical commits?
```

### Without Skill

```text
Try to keep related files together and avoid mixing unrelated work. If some changes are refactors and others are bug fixes, they should usually be split into separate commits.
```

### With `commit-group`

```text
Group 1: `refactor(editor): extract autosave service`

Files to stage:
git add src/editor/services/AutoSaveService.ts src/editor/hooks/useAutoSave.ts

Why together: These files are part of the same internal refactor of autosave responsibilities.

Group 2: `fix(editor): prevent duplicate save trigger`

Files to stage:
git add src/editor/components/SaveButton.tsx src/editor/store/editor.store.ts

Why together: These files fix one user-visible save bug and should be reviewed together.

Group 3: `test(editor): add duplicate save regression coverage`

Files to stage:
git add src/editor/__tests__/save-button.test.tsx

Why together: The regression test belongs with the bug fix but remains a separate, focused commit.

Commands:
git add src/editor/services/AutoSaveService.ts src/editor/hooks/useAutoSave.ts
git commit -m "refactor(editor): extract autosave service"

git add src/editor/components/SaveButton.tsx src/editor/store/editor.store.ts
git commit -m "fix(editor): prevent duplicate save trigger"

git add src/editor/__tests__/save-button.test.tsx
git commit -m "test(editor): add duplicate save regression coverage"
```

## Example 3

### Prompt

```text
What commits should I make from this worktree?
```

### Without Skill

```text
You should make a couple of focused commits based on the kind of work you changed. Separate implementation, tests, and docs when possible, and make sure each commit is easy to review.
```

### With `commit-group`

```text
Group 1: `chore(ci): cache pnpm store in workflow`

Files to stage:
git add .github/workflows/ci.yml

Why together: This is isolated CI configuration work.

Group 2: `feat(search): add query highlighting`

Files to stage:
git add src/search/highlight.ts src/search/SearchResults.tsx

Why together: These files implement the same user-facing feature.

Group 3: `test(search): add query highlight coverage`

Files to stage:
git add src/search/__tests__/highlight.test.ts

Why together: Test coverage for the new highlighting behavior is independently reviewable.

Commands:
git add .github/workflows/ci.yml
git commit -m "chore(ci): cache pnpm store in workflow"

git add src/search/highlight.ts src/search/SearchResults.tsx
git commit -m "feat(search): add query highlighting"

git add src/search/__tests__/highlight.test.ts
git commit -m "test(search): add query highlight coverage"
```

## Example 4

### Prompt

```text
Group changes in this monorepo. apps/api and apps/web are separate git repositories, so do not use api or web as commit scopes.
```

### Without Skill

```text
Make one API commit and one web commit:

feat(api): update checkout behavior
fix(web): refresh checkout UI
```

### With `commit-group`

```text
Group 1: `feat(checkout): validate promotion eligibility`

Repository: apps/api

Files to stage:
git -C apps/api add src/checkout/promotion.service.ts src/checkout/promotion.controller.ts

Why together: These files implement the same checkout-domain eligibility rule inside the API repository.

Group 2: `fix(checkout): show expired promotion state`

Repository: apps/web

Files to stage:
git -C apps/web add src/features/checkout/PromotionBanner.tsx src/features/checkout/PromotionSummary.tsx

Why together: These files update one checkout UI state inside the web repository.

Commands:
git -C apps/api add src/checkout/promotion.service.ts src/checkout/promotion.controller.ts
git -C apps/api commit -m "feat(checkout): validate promotion eligibility"

git -C apps/web add src/features/checkout/PromotionBanner.tsx src/features/checkout/PromotionSummary.tsx
git -C apps/web commit -m "fix(checkout): show expired promotion state"
```
