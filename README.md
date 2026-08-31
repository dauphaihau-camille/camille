# Camille
A note-taking web application where you can think, write, and plan.

![Camille app workspace](./assets/screenshots/app-workspace.png)
[See more app screenshots](./docs/screenshots/app.md)

## Demo

| App | Deployed | Local |
| --- | --- | --- |
| App | `https://app.camille.hautran.me` | `http://localhost:4000` |
| Marketing | `https://camille.hautran.me` | `http://localhost:4001` |
| API Docs | `https://api.camille.hautran.me` | `http://localhost:3000/docs` |

- [Demo guide](./docs/demo-guide.md) - demo accounts, seed modes, smoke checks, and safe test flows.

## Product Capabilities

- **Realtime collaboration** - multiple users can edit the same document with shared document state and live presence foundations
- **Block-based documents** - documents are stored as structured editor blocks instead of raw HTML
- **Nested documents** - documents can contain subdocuments, allowing doc-within-doc organization
- **Offline-first editing foundation** - local editor state can survive reloads and temporary disconnection
- **Sync queue and stale-write protection** - saves are ordered so older responses cannot overwrite newer edits
- **Workspace document tree** - documents are organized by workspace, teamspace, parent-child hierarchy, favorites, and trash
- **Public document sharing** - selected documents can be published through share routes
- **Workspace access control** - document visibility and editing are controlled by workspace, teamspace, inherited grants, and roles
- **Full-text document search** - users can search workspace documents by title and body content
- **Optimistic document actions** - common actions update immediately in the UI and roll back if the request fails
- **Resumable workspace navigation** - the app remembers the user’s last active workspace and expanded document tree state
- **Multiple sign-in flows** - users can sign up or log in with email codes, Google OAuth, or GitHub OAuth
- **Workspace AI assistant** - users can chat with Camille AI in a workspace, attach current documents as explicit context, stream structured responses, and append completed AI responses to editable documents
- **Subscription plans and entitlement gates** - workspace plans control block limits, seat-based billing, checkout/cancellation flows, and AI response access through trial and upgrade gates

For deeper implementation details, see the frontend and backend capability inventories:

- [Frontend implemented patterns and capabilities](apps/web/README.md#implemented-patterns-and-capabilities)
- [Backend implemented patterns and capabilities](apps/api/README.md#implemented-patterns-and-capabilities)

## Tech stack

#### Frontend:
- [Next.js](https://nextjs.org/) - React framework
- [React](https://react.dev/) - UI library
- [TanStack Query](https://tanstack.com/query/latest) - Server-state fetching and caching
- [React Hook Form](https://react-hook-form.com/) - Form state management
- [Zustand](https://zustand-demo.pmnd.rs/) - Client state management
- [Tailwind CSS](https://tailwindcss.com/) - Utility-first CSS framework
- [shadcn/ui](https://ui.shadcn.com/) - Reusable component patterns adapted for this project’s Tailwind CSS and Base UI setup
- [Zod](https://zod.dev/) - Schema validation
- [TypeScript](https://www.typescriptlang.org/) - Static type checking

#### Backend:
- [NestJS](https://nestjs.com/) - Node.js backend framework
- [PostgreSQL](https://www.postgresql.org/) - Primary database
- [MikroORM](https://mikro-orm.io/) - ORM and migrations
- [JWT](https://jwt.io/) - Authentication
- [BullMQ](https://bullmq.io/) - Background jobs and workers
- [Redis](https://redis.io/) - Cache, queues, and rate limiting
- [OpenTelemetry](https://opentelemetry.io/) - Tracing and instrumentation
- [TypeScript](https://www.typescriptlang.org/) - Static type checking
- [Zod](https://zod.dev/) - Schema validation

## Workspace Structure

This repository is the parent workspace for the Camille project. The application code lives in separate Git submodules so each app keeps its own history and remote repository.

- Parent workspace: [dauphaihau-camille/camille](https://github.com/dauphaihau-camille/camille)
- Frontend app: [apps/web](apps/web) - [dauphaihau-camille/web](https://github.com/dauphaihau-camille/web)
- Backend app: [apps/api](apps/api) - [dauphaihau-camille/api](https://github.com/dauphaihau-camille/api)

## Setup

```bash
git clone --branch production --recurse-submodules git@github.com:dauphaihau-camille/camille.git
cd camille
git submodule update --init --recursive
```

- [Local setup](./docs/local-setup.md) - install dependencies, seed demo data, and run the API, worker, app, and marketing site locally.

## Contact

For any inquiries or feedback, feel free to contact [me](mailto:dauphaihau@gmail.com).
