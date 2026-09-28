# ADR-0006: Structured errors returned as values

**Date:** 2026-02-11 (PRD.md AD-5; observed in `src/lib/actions/result.ts` and `src/lib/api-error.ts`)

**Context.** The original answered failed signings with generic 500 pages.

**Decision.** Server actions return `ActionResult` with a message, a code
and per-field errors; REST routes return `{ error: { code, message,
fields? } }` through `apiError` with a code from `ErrorCode`. The UI shows
field errors inline and the rest as toasts.

**Consequences.** Expected failures never throw. Observed exceptions:
`requireOwner()` throws, and unexpected database errors are rethrown to the
error boundary.
