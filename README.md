# Camille
A note-taking web application where you can think, write, and plan.

This repository is the parent workspace for the Camille project. The application code lives in separate Git submodules so each app keeps its own history and remote repository.

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

## Features
- Collaborative workspace parent repository
- Separate frontend and backend repositories managed as Git submodules
- Frontend app in [apps/web](/Volumes/Local/dev/pj-personal/apps/camille/camille-v2/apps/web)
- Backend app in [apps/api](/Volumes/Local/dev/pj-personal/apps/camille/camille-v2/apps/api)

## Repositories
- Parent workspace: [dauphaihau-camille/camille](https://github.com/dauphaihau-camille/camille)
- Frontend: [dauphaihau-camille/web](https://github.com/dauphaihau-camille/web)
- Backend: [dauphaihau-camille/api](https://github.com/dauphaihau-camille/api)

## Setup
```bash
git clone --branch production --recurse-submodules git@github.com:dauphaihau-camille/camille.git
cd camille
git submodule update --init --recursive
```

## Contact

For any inquiries or feedback, feel free to contact [me](mailto:dauphaihau@gmail.com).
