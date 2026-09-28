# ADR-0003: Two GitHub OAuth apps, owner and contributor

**Date:** 2026-02-11 (observed in `src/lib/auth.ts`)

**Context.** Owners need repository and organization access; contributors
should grant only their identity.

**Decision.** Auth.js with two GitHub OAuth providers. Signing in through
the owner provider makes the user an `owner` and stores its token; the
contributor provider stores no token.

**Consequences.** Two OAuth apps to configure per instance; a user's role
is promoted to `owner` the first time they sign in as one.
