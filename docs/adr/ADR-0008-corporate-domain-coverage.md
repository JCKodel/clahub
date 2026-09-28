# ADR-0008: Corporate CLA covers by email domain

**Date:** 2026-02-12 (migration `20260212194514_add_corporate_signature_fields`; observed in `src/lib/cla-check.ts`)

**Context.** Companies want to sign once for all their employees.

**Decision.** A `corporate` signature carries a `companyDomain`; any commit
author whose commit email or account email has that domain, and who has no
individual signature, is covered while the corporate signature is not
revoked.

**Consequences.** Coverage trusts the email on the commit. Whether that is
acceptable is an open decision in docs/00.
