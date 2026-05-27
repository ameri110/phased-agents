# {{PROJECT_NAME}} — Project Bible

This file is loaded into every Claude Code session. Keep it short, accurate,
and honest about what works.

> **Day 1:** placeholders below (`{{...}}`) get filled in via the
> `BOOTSTRAP.md` flow. Don't edit them by hand before running BOOTSTRAP — let
> Claude interview you and produce a calibrated version.

## Goal

{{ONE-PARAGRAPH PROJECT GOAL}}

## Current phase

**Phase 0 — {{PHASE_0_TITLE}}.** {{ONE-LINE CURRENT STATE}}

Acceptance gate: {{GATE}}. See `docs/phases/phase-0-{{PHASE_0_SLUG}}.md`.

> **Hard rule for the doc-keeper:** this section MUST fit in ≤ 12 lines.
> When it bloats, archive prior content into the relevant phase doc and
> rewrite this section to current state only.

## Why this structure exists

Long projects fail without (a) a written architecture, (b) hard acceptance
gates per phase, and (c) brutal evaluation discipline. {{DOMAIN-SPECIFIC
SILENCE-OF-BUGS NOTE — e.g., "numerical/algorithmic code is silent when
buggy", "auth/security bugs pass tests but leak in production", "UI a11y
bugs are invisible until a screen reader user hits them"}}. Every change
must pass its acceptance test before it counts as done.

## Repo layout

```
.claude/agents/    # specialized subagent definitions
docs/
  research/        # source-material summaries — written by Researcher
  decisions/       # ADRs (architecture decision records)
  phases/          # per-phase plans with acceptance gates
  evals/           # eval reports — written by Eval Runner
  reviews/         # review reports — written by Reviewer
{{SOURCE_DIR}}/    # {{PROJECT-SPECIFIC LAYOUT}}
tests/             # {{TEST FRAMEWORK}}
scripts/           # runnable entry points (training, evals, ...)
```

## Conventions

- {{LANGUAGE/RUNTIME}} — {{PACKAGE_MANAGER}}. New deps require an ADR.
- {{TEST_DISCIPLINE — e.g., "convergence tests required for any algorithm",
  "contract tests required for every API endpoint", "a11y audits required
  for every screen"}}.
- Match source notation in code; deviations need a one-line comment with a
  pointer to the research note.
- **No speculative abstractions.** Build what this phase needs, no more.
- **Phase gates are hard.** No "directionally good enough" — the eval either
  passes or it doesn't.

## How to run things

```bash
{{SETUP_COMMANDS}}

{{TEST_COMMANDS}}

{{EVAL_COMMANDS}}
```

## The project-lead role

The Claude session in this project's root is the **project-lead** — see
`PROJECT-LEAD.md`. It's not a subagent; it's the conversational layer
that holds the project together across sub-sessions.

If your project-lead session ends (closed, crashed, hit context limits),
spawn a new one by opening Claude Code in this directory and pasting the
contents of `RESUME-LEAD.md`.

The project-lead runs **supervised** by default (drafts kickoff prompts;
you open the sub-sessions). It can also run **autonomous** if you opt in —
spawning pipelines as background subagents and answering agents' mechanical
questions itself, while still escalating every design/spec/gate decision to
you. See `PROJECT-LEAD.md`.

## Working with subagents

Five specialized agents live in `.claude/agents/`:

| Agent         | Use when                                                   |
| ------------- | ---------------------------------------------------------- |
| `researcher`  | Reading new source material or surveying prior art         |
| `implementer` | Writing code from a finished spec / research note          |
| `reviewer`    | Independent review of a change                             |
| `eval-runner` | Running phase eval scripts and producing reports           |
| `doc-keeper`  | Periodic sync of CLAUDE.md / phase docs against the code   |

Typical phase loop:

1. `researcher` produces or updates a note in `docs/research/`
2. `implementer` writes code + tests against that note
3. `reviewer` audits the implementation against the note
4. `eval-runner` runs the phase eval and reports PASS/FAIL
5. **commit the sub-phase** (see "Commit & PR discipline" below)
6. `doc-keeper` updates phase status and CLAUDE.md "current phase"

## Commit & PR discipline

**Every sub-phase ends with a single git commit before doc-keeper sign-off.**

- **One sub-phase = one commit.** Don't bundle multiple sub-phases.
- **Commit after eval PASS, before doc-keeper.** Doc-keeper updates land
  in their own follow-up commit.
- **Subject:** `phase-Nx: <topic>`. Body explains what landed, the gate
  result, and any notable bug-found-during-implementation.
- **Do not amend or force-push.** New commits only.
- **`git add` specific files**, never `-A` or `.` — avoids accidentally
  committing caches, .env files, build artifacts.
- If a sub-phase ends and `git status` shows files outside that sub-phase's
  scope as modified, **stop and ask the project lead** — usually means a
  previous sub-phase wasn't committed.

**Only the project-lead (or the user) commits to `main`.** Sub-sessions
work on a branch and open a PR; they never push to `main` directly. The
project-lead is the integrator: it reviews the PR's eval/review reports,
resolves any ADR-index collisions, and merges. This keeps `main` an
honest, linear audit trail no matter how many pipelines ran in parallel.

## Parallel pipelines

When sub-phases are **independent** (no gate dependency between them),
run each in its own worktree + branch and land it as a PR. The unit of
isolation is the **sub-phase**, not the individual agent — all five RIRED
agents (researcher → implementer → reviewer → eval-runner → doc-keeper)
share one worktree so they can see each other's notes, diffs, and reports.

```bash
# from the project root, for sub-phase 2b:
git worktree add ../{{PROJECT_SLUG}}--phase-2b phase-2b-<slug>
# ... run the full pipeline inside that worktree ...
# commit per discipline above, push the branch, open a PR:
git push -u origin phase-2b-<slug>
gh pr create --fill
git worktree remove ../{{PROJECT_SLUG}}--phase-2b   # after merge
```

- Use this only for work that genuinely fans out. A linear phase chain
  (phase 2's gate depends on phase 1) does **not** parallelize — run it
  the normal single-session way.
- Each branch's ADRs use date+slug filenames so they never collide
  (see `docs/decisions/README.md`).
- The project-lead merges PRs into `main` and reconciles the ADR index.

## Critical reminders for any agent

- **Don't trust intuition.** Re-derive specs and verify against sources.
- **Run the eval before claiming a phase is done.** Always.
- **Commit the sub-phase before declaring it done.** See "Commit & PR discipline".
- **Update this file** when layout, conventions, or current phase changes.
  A stale CLAUDE.md is a project-killer.
