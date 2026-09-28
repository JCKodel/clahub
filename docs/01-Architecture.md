# Architecture

## In one sentence

One Next.js App Router application that serves the pages, the server
actions, the REST API and the GitHub App webhook, over a single SQLite file
through Prisma.

## Stack

| Piece | Choice | Why, where there was an alternative |
|---|---|---|
| Framework | Next.js 16, React 19, App Router | pages, server actions and API in one deployable (ADR-0001) |
| Language | TypeScript, strict | |
| Database | SQLite via Prisma 7, better-sqlite3 adapter | zero-ops single file, self-hostable; Prisma keeps Postgres possible (ADR-0002) |
| Auth | Auth.js v5 (next-auth beta), JWT sessions, two GitHub OAuth providers | owners and contributors get different scopes (ADR-0003) |
| GitHub | GitHub App via `@octokit/app`, Checks API | installation-level webhooks, private repos, richer checks than Statuses (ADR-0004) |
| UI | Tailwind CSS 4, shadcn/ui on Radix, lucide icons, sonner toasts | accessible primitives |
| Forms and validation | React Hook Form + Zod 4 | same schema on client and server |
| Markdown | react-markdown + remark-gfm | |
| Email | Resend, optional | |
| Errors and logs | Sentry (optional, `SENTRY_DSN`), JSON logger in `src/lib/logger.ts` | |
| Export | papaparse (CSV), @react-pdf/renderer (PDF) | |
| Tests | Vitest + Testing Library (jsdom), Playwright + axe-core | |

`hono` is a direct dependency and an `overrides` entry, but no source file
imports it (open question: transitive pin only?).

## How the code is organized

The project's own conventions, organized by technical layer (ADR-0009):

* `src/app/`: routes. `(marketing)/` public pages (landing, about, docs,
  privacy, terms, why-cla); `agreements/` dashboard, create, edit and the
  public signing pages `[owner]` and `[owner]/[repo]`; `settings/api-keys`;
  `auth/signin`; `api/` route handlers (`v1/` REST, `badge/`, `health/`,
  `webhooks/github`, `github/` helper lookups, `auth/`).
* `src/components/`: `ui/` shadcn primitives, `agreements/` and
  `settings/` feature components.
* `src/lib/actions/`: server actions, one file per area (agreement,
  signing, signature, exclusion, api-key, audit-log, recheck, contributing),
  plus `result.ts`.
* `src/lib/schemas/`: Zod schemas, shared by forms, actions and the API.
* `src/lib/`: everything else. `cla-check.ts` holds the core rule (author
  classification, check runs, re-check with retry); `github.ts` the App and
  webhook handlers; `access.ts` and `org-membership.ts` access levels;
  `audit.ts`, `api-*.ts`, `rate-limit.ts`, `email.ts`, `export-*.ts`,
  `badge.ts`, `branding.ts`, `templates.ts`, `prisma.ts`.
* `src/middleware.ts`: redirects unauthenticated or non-owner users away
  from owner pages; `/api/v1` authenticates itself.
* `src/generated/prisma/`: generated client, never edited.

Business rules live mostly in `cla-check.ts`, `access.ts` and the server
actions; the actions also do data access directly through `prisma`. There
is no repository layer.

## Data access

Direct Prisma calls from server actions, route handlers, server components
and `cla-check.ts`, through the singleton in `src/lib/prisma.ts`. Every
mutation runs in `prisma.$transaction` and writes an `AuditLog` row through
`logAudit(tx, …)` (ADR-0005). Agreements are soft-deleted (`deletedAt`),
signatures revoked (`revokedAt`). Schema changes go through
`prisma/migrations/`.

## How errors travel

* **Server actions** return `ActionResult`: `{ success: true }` or
  `{ success: false, error, code?, fieldErrors? }`, with Zod issues turned
  into `fieldErrors` by `validationError()`. Known conflicts (unique
  constraint) become `CONFLICT`. Exceptions: `requireOwner()` throws
  `Unauthorized`, and unexpected Prisma errors are rethrown to the error
  boundary.
* **REST routes** return `apiError(code, message, status, fields?)` with a
  code from `ErrorCode` (ADR-0006).
* **Webhooks** catch per handler and log; the route answers 401 on a bad
  signature and 500 otherwise.
* **Fire-and-forget work** (re-check, email) is `.catch(() => {})`; re-check
  retries three times with exponential backoff and logs.
* **UI** shows field errors inline and the rest as toasts; `error.tsx` and
  `global-error.tsx` report to Sentry.

## Environments

* **Local:** `npm run dev` on port 3000; webhooks need a tunnel (ngrok).
  Configuration comes from two files read by different tools:
  * Next.js (`dev`, `build`, `start`) reads `.env.local` (and `.env`).
  * The Prisma CLI (`db:push`, `migrate`, `db:seed`, `studio`) reads only
    `.env`: `prisma.config.ts` and `prisma/seed.ts` load it through
    `import "dotenv/config"`, which ignores `.env.local`.
  * `DATABASE_URL` is a path relative to the repository root
    (`file:./clahub.db` in `.env.local.example`); the app and the Prisma CLI
    both resolve it from there.
  * Playwright loads `.env.local`, then `.env.test`, without overriding, so
    locally the E2E run sees the `DATABASE_URL` of `.env.local`. When no dev
    server is on port 3000, its global setup force-resets and seeds that
    database; otherwise it reuses the running server and its data.

  Where README, `docs/getting-started.md` and the code disagree, see the
  open questions below.
* **CI (GitHub Actions):** on push and PR to `main`: lint, `tsc --noEmit`,
  unit tests, build; then Playwright E2E against a fresh `test.db`.
* **Production:** https://www.cla-hub.io. `docs/deployment.md` describes a
  GCP Compute Engine VM with Node 20, systemd, nginx and Let's Encrypt;
  deployment is manual. Which target cla-hub.io actually runs on is not
  recorded in the code.
* **Self-hosted:** Docker (`Dockerfile`, `docker-compose.yml`, migration on
  start via `docker-entrypoint.sh`), Vercel, Railway, Fly.io, PM2/systemd.

The rate limiter is in memory, so limits are per process.

## Open questions

* **Which env file the Prisma CLI reads.** README and
  `docs/getting-started.md` say to copy `.env.local.example` to `.env.local`
  and then run `npm run db:push` / `npx prisma db push`. The Prisma CLI reads
  only `.env`, so with `.env.local` alone `DATABASE_URL` is undefined and the
  command fails unless the variable is exported in the shell or also written
  to `.env`. Fix the guides, or make `prisma.config.ts` and `prisma/seed.ts`
  load `.env.local` too?
* **`APP_URL` is documented but never read.** README, `.env.local.example`,
  `.env.docker.example` and `docs/configuration.md` list `APP_URL` as
  required. The code reads `NEXT_PUBLIC_APP_URL` for check-run and
  CONTRIBUTING.md links (default `http://localhost:3000`) and `NEXTAUTH_URL`
  for links in emails (default `https://cla-hub.io`); CI and `.env.test` set
  `NEXT_PUBLIC_APP_URL`. A self-hoster following the guides gets check runs
  linking to localhost. Which variable is the one?
* **Local E2E can reset the development database.** Because `.env.local`
  wins over `.env.test`, running `npm run test:e2e` locally without a dev
  server force-resets the database named in `.env.local`, not `test.db`.
  Intended?

## Tried and removed on purpose

* The original Ruby on Rails app on Heroku, with OAuth App webhooks and the
  Statuses API: replaced by this rewrite (ADR-0001, ADR-0004).
* Repo identity by `owner/name`: replaced by the numeric GitHub repo id,
  with names updated on rename and transfer events (ADR-0007).
* Copying `.prisma` into the Docker image: Prisma 7 generates into
  `src/generated`.
