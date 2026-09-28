# Conventions

## Language

Documentation in English. Identifiers in English.

## Naming

* Files kebab-case (`cla-check.ts`, `signing-form.tsx`); components
  PascalCase; functions and variables camelCase.
* Server actions are verbs on the domain (`signAgreement`,
  `revokeSignature`). Audit actions are `entity.verb` (`signature.sign`,
  `webhook.repository.renamed`).
* Route segments follow GitHub names: `[owner]`, `[repo]`, `[username]`.
* Import through the `@/` alias for `src/`.

## Style

* ESLint 9 with `eslint-config-next` (core-web-vitals, typescript) and
  `eslint-config-prettier`: `npm run lint`.
* TypeScript strict: `npx tsc --noEmit`.
* Prettier: `npm run format`. Observed contradiction: `.prettierrc` asks for
  single quotes and width 100, while `src/` is written with double quotes;
  CI does not check formatting. CONTRIBUTING.md mentions Biome, which is
  not installed. Until settled, match the surrounding file and do not
  reformat files you do not otherwise change.
* UI from `src/components/ui/` (shadcn); accessibility is WCAG 2.1 AA.

## Tests

* **Unit (Vitest, jsdom):** `tests/unit/`. `schemas/` for Zod schemas,
  `lib/` for utilities and rules with mocked Prisma and Octokit,
  `actions/` for server actions with mocked auth and Prisma, `cla-check*`
  for the core rule. Run with `npm test`.
* **E2E (Playwright, Chromium):** `tests/e2e/`: signing flow, dashboard,
  webhook, accessibility (axe). Sessions injected with `authenticateAs()`
  from `tests/e2e/auth-helpers.ts`. Global setup resets and seeds `test.db`.
  Run with `npm run test:e2e`.
* `tests/components/` and `tests/api/` exist and are empty.
* A rule gets a unit test; a user flow gets an E2E test.

## Commits

Conventional Commits: `type(scope): subject`, types `feat`, `fix`,
`refactor`, `chore`, `docs`, `test`, `style`, `perf`. The shape of the body
is in docs/05 §6.

## Branches

`feat/`, `fix/`, `refactor/`, `chore/`, `docs/` prefixes, with the issue
number when there is one (`fix/270-…`). Pull requests target `main`.
