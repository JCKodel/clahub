# CLAHub

A GitHub App and web app where contributors sign a project's Contributor
License Agreement and pull requests get a check saying whether every commit
author has signed. Prose in English; identifiers in English.

## Read before acting
- the product: docs/00 · the vocabulary: docs/03
- how it is built: docs/01 · the server: docs/02
- style and tests: docs/04 · process: docs/05 · queue: docs/06

## Non-negotiables
- The core loop (create agreement, sign, PR check updates) works every time (docs/00).
- Contributors sign with identity-only GitHub permission; no contributor token is stored (ADR-0003).
- Every mutation runs in a transaction and writes an audit log entry (ADR-0005).
- A repository is identified by its numeric GitHub id, never by its name (ADR-0007).
- It stays one Next.js process over SQLite, self-hostable (ADR-0001, ADR-0002).
- Pages stay WCAG 2.1 AA; the axe E2E check stays green.
- Open decisions in docs/00 are never settled by assumption.
- One delivery = one page in work/<slug>.md: /propose to define, /apply
  to build.
- No em dash in any text a user reads.
- The agent stages and suggests the commit message. It never commits.

## Do not rebuild
- Rails/Heroku, OAuth App webhooks, the Statuses API (ADR-0001, ADR-0004).
- Repo identity by owner/name (ADR-0007).
- Never edit `src/generated/prisma/`; regenerate with `npx prisma generate`.

## How to work
- Organized by layer: routes in `src/app`, server actions in
  `src/lib/actions`, Zod schemas in `src/lib/schemas`, the CLA rule in
  `src/lib/cla-check.ts` (docs/01, ADR-0009).
- Errors are values: actions return `ActionResult`, REST routes return
  `apiError` with an `ErrorCode` (ADR-0006).
- One branch per delivery, `<type>/<slug>` from `main` (ADR-0010).
- `npx prisma generate && npm run lint && npx tsc --noEmit && npm test && npm run build`
  before declaring anything done; `npm run test:e2e` when a user flow changes.
- Abstraction on the second concrete occurrence, and the delivery says
  which was the first.
- Ambiguity → ask. Documents are living: a delivery that changes behaviour
  updates the document that owns it, in the same delivery.
