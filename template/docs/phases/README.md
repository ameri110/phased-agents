# Phases

Each phase is a contract: a goal, a measurable acceptance gate, and an
exit criterion. **Do not advance to phase N+1 until phase N's gate is
verifiably passed by an eval report in `docs/evals/`.**

> **Day 1:** the table below gets filled in during BOOTSTRAP based on
> your project's domain and timeline.

## Phase index

| #   | Topic                | Status      | Gate (short)                     |
| --- | -------------------- | ----------- | -------------------------------- |
| 0   | {{Phase 0 title}}    | not-started | {{gate}}                         |
| 1   | {{Phase 1 title}}    | not-started | {{gate}}                         |
| ... |                      |             |                                  |

## Why hard gates

"Looks like it works" is not evidence of correctness. A phase counts as
done only when an eval report in `docs/evals/` shows the gate met.

## Phase deliverables

Every phase must produce:

1. Working code in the project's source tree
2. Tests covering the correctness criteria, not just smoke-running
3. An eval script (or test) that produces a PASS/FAIL report
4. An eval report in `docs/evals/` showing the gate met
5. A research note in `docs/research/` if the phase introduces new
   external concepts (algorithms, RFCs, design systems, etc.)
6. An ADR in `docs/decisions/` for any non-obvious choice

## Phase doc template

See `_phase-template.md` for the structure of an individual phase doc.
