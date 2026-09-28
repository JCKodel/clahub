# ADR-0001: Rewrite as one Next.js application

**Date:** 2026-02-12 (recorded in PRD.md; observed in code)

**Context.** The original CLAHub was a Rails app on Heroku that was down,
timed out on webhooks and had accumulated 58 open issues (`Old-Issues.md`).

**Decision.** Rewrite it as a single Next.js App Router application in
TypeScript: pages, server actions, REST API and webhook receiver in one
deployable.

**Consequences.** One process to host and self-host; no separate worker, so
background work runs inside the web process (see the open decision on
background re-check in docs/00).
