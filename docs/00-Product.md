# Product

## Purpose

CLAHub gives an open source project on GitHub a Contributor License
Agreement with as little friction as possible. The project owner writes or
picks a CLA, contributors sign it by signing in with GitHub, and every pull
request gets a GitHub check saying whether all its commit authors have
signed. It is a rewrite of the original Rails CLAHub (see `PRD.md`,
`Analysis.md`, `Old-Issues.md`), running at https://www.cla-hub.io and
self-hostable as a single Next.js + SQLite unit.

## Audience

Two sides, plus machines:

* **Owner:** a maintainer or organization admin. Creates agreements for a
  repository or a whole organization, manages versions, fields,
  exclusions and signatures, exports, receives notifications, uses the
  REST API. Served by: agreement editing, dashboard, settings, API, export,
  badges, CONTRIBUTING.md snippet.
* **Contributor:** someone opening a pull request. Signs in with minimal
  GitHub permissions, reads the CLA, fills the fields, signs, and their
  open PRs are re-checked. Served by: the public signing page and the PR
  check.
* **Org admin:** a GitHub organization admin who is not the agreement
  owner; can view agreements and signatories of that organization.
* **Bots and CI:** excluded automatically; can query signature status
  through the public check endpoint.

## Mechanics

1. The owner installs the CLAHub GitHub App on a repository or organization.
2. The owner creates an agreement for that repository or organization,
   from a template (Apache ICLA, DCO) or custom Markdown, with optional
   custom fields.
3. When a pull request is opened, synchronized or reopened, GitHub sends a
   webhook. CLAHub collects the commit authors and classifies each one as
   excluded, signed, covered by a corporate signature, or unsigned.
4. CLAHub posts a check run: success when nobody is unsigned, otherwise
   "action required" with a link to the signing page.
5. When a contributor signs, every open PR of that agreement is re-checked.
6. A company representative can sign a corporate CLA once; commit authors
   whose email domain matches are covered.

## Non-goals

* GitLab or Bitbucket support.
* Internationalization of the interface.
* Blockchain or cryptographic signature verification.
* Native mobile apps.
* Legal advice: nothing in CLAHub is legal advice.

## Values

Reliable, low-friction, self-hostable, auditable, minimal-permission,
accessible.

## Product questions

Every decision answers yes to all of these:

1. Does the core loop (create, sign, PR check updates) still work every time?
2. Does a contributor still sign without asking for more GitHub permission
   than identity?
3. Can a self-hoster still run it as one Next.js process with SQLite?
4. Is every mutation still recorded in the audit log, inside a transaction?
5. Does the user see a specific error, never a generic one?
6. Does it stay WCAG 2.1 AA?

## Open decisions

Nobody closes these alone; an agent never settles them by assumption.

* **Commit email trust.** `PRD.md` NFR-3 says only verified GitHub emails
  are used for CLA matching. `cla-check.ts` matches users by the commit
  author email and grants corporate coverage by the commit email domain,
  which any committer can set. Which is the intended rule?
* **Background re-check.** `PRD.md` asks for webhook processing to return
  within 500 ms with a background job. The code processes the webhook
  inline and re-checks PRs as an un-awaited promise with retries, which a
  serverless host may cut short. Is inline processing the accepted design?
* **Webhook delivery log.** Promised by `PRD.md` (FR-4.3, NFR-4); not built.
  Still wanted?
* **Test coverage target.** `PRD.md` NFR-5 asks for component tests, API
  integration tests and 80% coverage; `tests/components/` and `tests/api/`
  are empty and no coverage is measured. Still the target?
* **Status of PRD.md, Analysis.md, Old-Issues.md.** Historical planning
  documents for the rewrite. Keep as history, or retire now that docs/00
  to 06 exist?
