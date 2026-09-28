# ADR-0007: Numeric GitHub ids identify repositories

**Date:** 2026-02-12 (PRD.md AD-2; observed in the schema and `repository.renamed|transferred` handlers)

**Context.** The original broke when repositories were renamed or
transferred.

**Decision.** `Agreement.githubRepoId` (and `githubOrgId`) is the identity;
`ownerName` and `repoName` are display fields updated by webhook events.

**Consequences.** URLs use names, so a lookup by name can miss between a
rename and its webhook.
