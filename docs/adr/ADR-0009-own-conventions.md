# ADR-0009: Keep the project's own conventions, not FOCUS

**Date:** 2026-09-28

**Context.** The code is organized by technical layer (`src/lib/actions`,
`src/lib/schemas`, `src/components/<area>`) and already returns errors as
values in actions and the API. Restructuring into FOCUS pieces or vertical
slices would touch every file for no user-visible gain.

**Decision.** Neither FOCUS whole nor its two principles as a rule: the
conventions in docs/01 stay as they are. New code follows the existing
layout and the error conventions of ADR-0006.

**Consequences.** Business rules and data access stay mixed in server
actions; a delivery that wants to change that proposes it as its own line
in the queue.
