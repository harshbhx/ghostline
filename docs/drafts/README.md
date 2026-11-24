# Drafts (written outside Kiro)

Everything in this folder was drafted outside Kiro, with AI help disclosed in the README. These files are **inputs**, not specs.

How they are used:

1. Open Kiro spec mode for one feature, for example `ghostline-core`.
2. Point it at the matching draft: "Create the spec from `docs/drafts/specs/ghostline-core.md` and `docs/ARCHITECTURE.md`."
3. Kiro writes `.kiro/specs/<name>/requirements.md`, `design.md` and `tasks.md`. Review each step in Kiro before moving on.
4. From then on the files in `.kiro/specs/` are the source of truth for tasks. Drafts are not updated.

| Draft | Spec | Requirements | Priority | Build order |
|---|---|---|---|---|
| `specs/ghostline-core.md` | `ghostline-core` | FR-C1 – FR-C15 | P0 | 1 |
| `specs/ghostline-runtime.md` | `ghostline-runtime` | FR-R1 – FR-R11, FR-W1 – FR-W6 | P0 | 2 |
| `specs/ghostline-adapters.md` | `ghostline-adapters` | FR-A1 – FR-A8 | P0 / P1 | 3 |
| `specs/dead-air-fm.md` | `dead-air-fm` | FR-F1 – FR-F10 | P0 | 4 |
| `specs/missed-call-oracle.md` | `missed-call-oracle` | FR-O1 – FR-O13 | P0 / P1 / P2 | 5 |
| `specs/barge-in-bench.md` | `barge-in-bench` | FR-B1 – FR-B6, FR-H1 – FR-H3 | P0 / P1 | 6 |
| `specs/ghostline-power-mcp.md` | `ghostline-power-mcp` | FR-P1 – FR-P5 | P1 | 7 |

Tip to save credits: keep each Kiro request to one task, and never ask Kiro to regenerate a whole spec.
