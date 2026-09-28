# ADR-0002: SQLite through Prisma

**Date:** 2026-02-11 (observed in `prisma/schema.prisma`; rationale in PRD.md and Analysis.md)

**Context.** Self-hosting must be simple; the data set is small.

**Decision.** SQLite as the only database, accessed through Prisma 7 with
the better-sqlite3 adapter, client generated into `src/generated/prisma`.
Enum-like columns are strings.

**Consequences.** Zero-ops single file and a Docker volume for persistence;
one writer, so horizontal scaling is not supported. Moving to Postgres or
Turso stays possible through Prisma.
