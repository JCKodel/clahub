# ADR-0005: Transactional mutations with an audit log

**Date:** 2026-02-12 (PRD.md AD-4; observed in `src/lib/audit.ts` and the server actions)

**Context.** The original lost data during ownership transfers.

**Decision.** Every write runs in `prisma.$transaction` and records an
`AuditLog` row with the user, action, before and after, and IP, through
`logAudit(tx, …)`. Agreements are soft-deleted; signatures are revoked,
not deleted.

**Consequences.** A new mutation without an audit entry is a defect. The
audit log grows without bound.
