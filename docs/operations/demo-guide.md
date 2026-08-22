# Demo Guide

Use this guide to test Camille demo environments, including deployed demo sites when available and locally seeded demo data. Keep checks focused, reversible, and safe for shared data and real integrations.

## Sites

| App | Deployed | Local |
| --- | --- | --- |
| App | `https://app.camille.hautran.me` | `http://localhost:4000` |
| Marketing | `https://camille.hautran.me` | `http://localhost:4001` |

## Suggested Test Accounts

Use these seeded demo accounts when testing Camille. Committed demo accounts use password `password123`. Realistic scenario accounts also use `password123` unless `SEED_REALISTIC_DEFAULT_PASSWORD` is overridden before seeding.

### Base App Accounts

Use these accounts after the regular demo seed.

| Accounts | Role | Best For |
| --- | --- | --- |
| `member@example.com` | Member | Main workspace smoke checks: document tree, editor, favorites, trash, search, and sharing flows. |
| `admin@example.com` | Admin | Permission, admin-role, and elevated-access smoke checks where the current app exposes admin-only behavior. |

### Realistic Scenario Accounts

Use these accounts after running the realistic seed. They are grouped around the seeded Camille AI and Northstar GTM AI workspaces.

| Accounts | Scenario Access | Best For |
| --- | --- | --- |
| `maya.chen@example.com` | Camille AI owner; Model Evaluation and AI Product teamspace manager | Owner-level workspace checks, teamspace management, document access grants, and broad Camille AI document navigation. |
| `jordan.lee@example.com` | Camille AI admin; Model Evaluation and AI Product teamspace editor | Admin/editor checks, restricted teamspace editing, and product/evaluation document workflows. |
| `sofie.nguyen@example.com` | Camille AI member; Model Evaluation viewer; Agent Permissions view grant | Viewer access checks, restricted evaluation visibility, and document-specific view grants. |
| `alex.rivera@example.com` | Camille AI member; AI Product viewer; Weekly Brief edit grant | Mixed workspace-member and document-specific edit permission checks. |
| `nina.patel@example.com` | Northstar GTM AI owner; Customer Intelligence and Launch Room teamspace manager | Owner-level GTM checks, teamspace management, and customer rollout document navigation. |
| `omar.hassan@example.com` | Northstar GTM AI admin; Customer Intelligence and Launch Room teamspace editor | Admin/editor checks, Account Health edit grant, and GTM operations document workflows. |
| `emily.tran@example.com` | Northstar GTM AI member; Customer Intelligence viewer | Viewer checks for restricted customer intelligence content. |
| `lucy.garcia@example.com` | Northstar GTM AI member; Launch Room viewer; GTM Home manage grant | Launch-room visibility checks and document-specific manage permission checks. |
| `daniel.kim@example.com` | Realistic user fixture without workspace membership in the default templates | No-access and invite/permission boundary checks. |

## Screenshot Demo Data

For local screenshots, use the realistic seed and open the `Camille AI` workspace. The seed is designed to fill the sidebar with distinct sections:

- Favorites: includes high-signal AI pages such as `Customer Signals` and `Model Evaluation`.
- Private: each member gets private roots such as `My AI Notes`, `Draft Prompts`, and `Research Queue`.
- Teamspaces: includes AI-focused hubs such as `AI Product`, `Model Evaluation`, `Security Review`, and `Customer Signals`.
- Shared: includes direct-shared pages with owner-specific titles such as `Maya Launch Review` or `Jordan Eval Feedback`.

Recommended screenshot flow:

- Sign in as `nina.patel@example.com` or `maya.chen@example.com`.
- Open `Camille AI`.
- Expand `AI Product`.
- Open `AI Product` and confirm the sidebar child order matches the page's `Related Pages` order: `Q3 Roadmap`, `Search Quality Spec`, `Launch Brief`, `Prompt Library`, `Tool Use Rules`, `Beta Feedback`, `Assistant UX`.
- Capture the sidebar with `Favorites`, `Private`, `Teamspaces`, and `Shared` visible.

### Public And Guest Testing

No login account is needed for public marketing and shared-document checks.

Use guest mode to test:

- Marketing homepage load and navigation.
- Anonymous public share route access.
- Revoked, missing, or private shared-document states.
- Protected app routes redirecting unauthenticated visitors to sign-in.

Local-only accounts can be added with `apps/api/seed-data/auth-users.local.tsv` when a tester needs machine-specific data. Do not rely on local-only accounts for deployed demo checks.

## Test Rules

### Safety Rules

- Do not run database clear, seed, migration rollback, or storage reset commands against shared or production-like environments.
- Use clearly named test workspaces and documents when checking write flows.
- Avoid changing shared demo account passwords or long-lived settings.
- Clean up test-only workspaces or public shares after release checks if they are visible to others.
- Keep OAuth, email, storage, and worker checks in sandbox or local mode unless a release explicitly requires external integration verification.

## Test Checklists

### Smoke Checklist

- Marketing site loads without console-visible runtime errors.
- App redirects unauthenticated users to login.
- API readiness endpoint is healthy.
- API docs or OpenAPI endpoint is reachable only as intended for the environment.
- Login, logout, refresh, and protected-route redirects work.
- Workspace entry resumes the expected workspace.
- Document tree, favorites, trash, and settings routes load.
- Search returns matching document titles or body content.
- Background worker starts without queue connection errors.
- WebSocket or realtime connection paths do not show authentication or transport errors.

### App Flow

- Sign in as `member@example.com`.
- Open the workspace landing route and confirm the sidebar renders.
- Create a document and edit its title and body.
- Refresh the page and confirm the latest title and body remain.
- Create a nested document or subdocument reference if seeded data supports the path.
- Favorite and unfavorite a document.
- Move a document to trash, restore it, then permanently delete only test-created content.
- Search for a document by title and by body text.

### Workspace And Access Flow

- Confirm workspace selection and last-active workspace behavior.
- Open workspace settings when the user has permission.
- Confirm restricted workspace or document routes deny access when the user lacks permission.
- Confirm teamspace or inherited document access behaves consistently when seeded data includes those relationships.
- Confirm member-management actions are visible or hidden according to the signed-in user role.

### Sharing Flow

- Publish a test-created document when the environment allows it.
- Open the public share route from the marketing app.
- Confirm anonymous users can view the shared document.
- Revoke or unpublish the document.
- Confirm the public route no longer exposes the document.

### Realtime And Offline Checks

- Open the same test document in two browser windows signed in as allowed users.
- Edit the title and body from both windows and confirm latest edits converge.
- Temporarily stop the API or disconnect the browser, make a local draft edit, then reconnect.
- Confirm stale saves do not overwrite newer local content.

### Integration Checks

- Email delivery uses the logger or sandbox provider.
- OAuth callback routes are configured only for the target environment under test.
- Storage uses local or approved object storage credentials.
- Queue jobs are visible at `http://localhost:3000/ops/queues` when Bull Board is enabled.
- Logs, metrics, and traces are emitted when observability mode is running.
