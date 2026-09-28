# ADR-0004: GitHub App and the Checks API

**Date:** 2026-02-12 (PRD.md AD-1 and AD-3; observed in `src/lib/github.ts`)

**Context.** The original used OAuth App webhooks and the Statuses API: no
private repos, no retries, poor feedback on PRs.

**Decision.** CLAHub is a GitHub App. It receives installation-level
webhooks and reports on PRs with check runs (`success` or
`action_required`, linking to the signing page).

**Consequences.** Each instance registers its own App; an agreement works
only while its `installationId` is set.
