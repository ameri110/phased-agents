# Phase N — {{Title}}

**Status:** not-started | in-progress | done — closed by `docs/evals/<report>.md`

## Goal

One paragraph. What does success look like at the end of this phase?
What's the minimum viable thing that proves the goal is met?

## Why this phase exists

The motivation. What problem does this phase solve that the prior phase
left open? What does the next phase rely on this phase for?

## Scope

In:

- Item 1
- Item 2

Out (deferred):

- Item that will tempt you mid-phase but belongs in a later phase

## Acceptance gate

Concrete, measurable, runnable as a script. Examples:

- "Running `python scripts/foo.py` produces a report showing
  `metric < threshold`"
- "Running `pytest tests/contract/test_x.py` passes 100% with seeds
  {1, 2, 3}"
- "Visual regression on `pages/landing` returns zero diffs at 1440px,
  768px, 375px viewports"

## Sub-phase plan (if needed)

For phases bigger than ~one session of work. Each sub-phase has its
own gate.

### Na. {{Sub-phase title}} — not-started

- Deliverables
- Gate

### Nb. {{Sub-phase title}} — not-started

- Deliverables
- Gate

## Deliverables

- [ ] `<source-file>` — what it does
- [ ] `tests/<test>` — what it tests
- [ ] `scripts/<eval>` — what it produces
- [ ] `docs/research/<note>.md` — research note (if new concepts)
- [ ] `docs/decisions/<adr>.md` — ADR (if non-obvious choice made)
- [ ] `docs/evals/phase-N-<eval>-<date>.md` — produced by eval-runner

## How to advance

1. `eval-runner` runs the eval script and writes a report.
2. Report shows PASS.
3. Sub-phase commit lands per CLAUDE.md commit discipline.
4. `doc-keeper` updates this file's status to `done` and links the report.
5. Open Phase N+1.

## Pitfalls

> Read this section before debugging.

- {{Domain-specific gotcha #1}}
- {{Common mistake when implementing this phase}}
- {{Sign-error / corner-case / silent-failure to watch for}}
