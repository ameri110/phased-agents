# Architecture Decision Records (ADRs)

Lightweight log of non-obvious decisions and their rationale. We use ADRs
so future-us (and future agents) can understand *why* the code is the
way it is.

## When to write an ADR

- Choosing between two reasonable algorithms / approaches / libraries
- Picking a new dependency
- Setting a hyperparameter or config value with wide implications
- Deferring something nontrivial
- Recalibrating a phase gate (transparently — never silently)

## When NOT

- Routine implementation choices
- Things obvious from the code

## Format

See `_adr-template.md`. Name files `YYYY-MM-DD-<slug>.md` —
e.g. `2026-05-27-postgres-vs-sqlite.md`.

**Why date+slug, not sequential numbers.** When sub-phases run in
parallel branches (see CLAUDE.md "Parallel pipelines"), two branches both
grabbing `0007-...md` collide at merge, and renumbering breaks every
inbound link. Date+slug filenames never collide and never need renaming.
The index table below is the canonical ordering; the project-lead keeps it
in sync when branches merge.

## Index

| File                              | Topic                | Status   |
| --------------------------------- | -------------------- | -------- |
| _none yet_                        |                      |          |
