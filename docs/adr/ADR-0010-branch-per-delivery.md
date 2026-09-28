# ADR-0010: A branch per delivery

**Date:** 2026-09-28

**Context.** The repository already works with `feat/`, `fix/` and similar
branches merged into `main` through pull requests, and CI runs on those
pull requests.

**Decision.** Each delivery is built on its own branch `<type>/<slug>` from
`main`. The agent stages and suggests the commit; the person commits,
pushes, opens the pull request and merges.

**Consequences.** One delivery at a time per working copy; CI checks each
delivery before it lands.
