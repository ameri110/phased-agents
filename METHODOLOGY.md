# The phased-agents methodology

Deep dive on the principles, the agents, the loop, and what to avoid. This document exists so that someone reading the template files can understand *why* each rule is there.

## Premise

Long AI-driven projects fail in five ways:

1. **Context loss across sessions.** Each Claude session starts fresh. Without a written project bible, decisions made in week 2 are forgotten by week 6.
2. **Silent bugs.** Some classes of code (algorithms, security, performance, accessibility) fail without crashing. Tests pass, the bot ships, the bug ships with it. Standard "code review" by another LLM session is mediocre at catching these.
3. **Drift toward complexity.** Without a forcing function, AI agents build abstractions speculatively, add features beyond the task, and gradually create a codebase no human can navigate.
4. **No verification discipline.** Without a hard "does this pass an eval?" gate, "looks good" gradually replaces "is correct."
5. **Untraceable history.** Without commit discipline, the git log doesn't tell you what happened. Months of work blur into "various changes."

The methodology addresses each.

## The five principles

### 1. Phases are contracts

A phase is a goal + a measurable acceptance gate + an exit criterion. The gate must be verifiable by a script that produces a PASS/FAIL report. "Reasonable" or "good enough" don't count.

**Why:** Hard gates make verification cheap and force the project to confront problems early. A buggy CFR implementation looks identical to a working one until you measure exploitability — without the measurement, you'd ship the bug.

Sub-phases are smaller phases. A phase with too much in it gets split into 1a, 1b, 1c, etc. Each has its own gate.

### 2. One sub-phase, one session

Don't try to do everything in one Claude session. Each sub-phase is its own session, kicked off with a focused prompt that references the phase doc.

**Why:** Sessions accumulate context drift. Past 2–3 hours of work, agents start forgetting their own conventions, hallucinating files, and producing lower-quality output. Fresh sessions start clean.

The repo + `CLAUDE.md` + commit history + phase docs are how state survives across sessions. They are the project's persistent memory.

### 3. Five agents with one job each

The pipeline:

1. **researcher** — reads sources, writes notes the implementer can use
2. **implementer** — writes code from notes, refuses to start without one
3. **reviewer** — independently audits, looking for what the implementer missed
4. **eval-runner** — runs the gate, declares PASS/FAIL, never edits source
5. **doc-keeper** — keeps docs in sync with the code

**Why split them:**

- *Specialization beats generalization.* An agent told to "implement and review" implicitly trusts its own work. Two separate agents, each with a different prompt, catch each other's mistakes.
- *The reviewer must read the spec before reading the code.* This is the single biggest correctness lever. Reviewers who go straight to the code rationalize whatever they see; reviewers who re-derive the spec first detect divergence.
- *The eval-runner must not edit source.* If a single agent both runs evals and fixes failures, the failure record gets papered over. Separating the roles forces honest gate reporting.

### 4. Commit discipline

One sub-phase = one commit. Commit after the eval PASSes, before the doc-keeper signs off. Subject line is `phase-Nx: <topic>`; body explains what landed and what the gate said. Don't amend, don't force-push, don't bundle.

**Why:** The git log becomes the project's audit trail. `git log --oneline` should read like a project journal: which sub-phases happened, in what order, with what results. Eval reports embed git SHAs for reproducibility — which only works if the work was actually committed when the eval ran.

The most common failure mode of this methodology: agents skip the commit step. The template's `CLAUDE.md` codifies it; `.claude/settings.json` allows the routine git commands without permission prompts. If you find yourself with 40+ uncommitted files, something is broken — see "Recovery" below.

### 5. The main session co-manages

There's a sixth role no subagent can fill: **the main session as project lead**. This is the Claude session you (the human) drive directly. It:

- Reads sub-session reports
- Surfaces decisions that need user input
- Drafts kickoff prompts for next sub-sessions
- Catches process drift (uncommitted work, doc-bloat, dropped balls)
- Holds the long-term context the user couldn't be expected to remember

You don't define this role with a markdown file — it's just how you use Claude Code in the project's root directory. The main session reads `CLAUDE.md` like everyone else, but it also reads the user's strategic intent and helps shape what comes next.

## The five subagents in detail

### researcher

**Inputs:** a topic, a paper, an RFC, a design system, prior-art landscape.

**Outputs:** a markdown note in `docs/research/` containing citation, problem statement, approach (with verbatim quotes from sources for critical specs), key parameters, pitfalls, empirical claims, connections to other notes, open questions.

**Discipline:** never paraphrase critical specifications — copy verbatim then explain. Cite section/page numbers so the reviewer can verify. No code in research notes; that's the implementer's job.

**Customization for your domain:**

- *Algorithms / ML:* papers, math notation, convergence guarantees
- *API design:* RFCs, prior-art APIs, contract conventions
- *UI:* design systems, accessibility standards (WCAG), browser quirks
- *Mobile:* platform conventions (HIG, Material), perf budgets
- *Infrastructure:* cloud-vendor docs, blast-radius analysis

### implementer

**Inputs:** a research note, a phase plan, or a clear written description.

**Outputs:** code + tests in the project's source tree.

**Discipline:** refuses to start without a spec. Writes tests first for anything with subtle failure modes. Convergence/contract/correctness tests are mandatory — "compiles and runs" is not done. Doesn't claim a phase passed; hands off to the eval-runner.

**Customization for your domain:**

- The body is mostly stack-agnostic. Domain-specific bits go in the bug-list the implementer briefs the reviewer with at handoff.

### reviewer

**Inputs:** the implementer's diff + the research note / spec it was supposed to implement.

**Outputs:** a markdown review at `docs/reviews/<date>-<topic>.md` with severity-ranked findings and an approve/approve-with-changes/reject recommendation.

**Discipline:** re-derives the spec from the source material *before* reading the code. Builds a suspicion list (what would have to be true for this code to be wrong) and checks each item. Runs existing tests and verifies the convergence/correctness test actually exercises the path that matters, not just a smoke test.

**Customization for your domain (this is the most important customization):**

- *Algorithms:* sign errors, off-by-ones, silent convergence bugs, missing edge cases in recursion
- *APIs:* idempotency, error-path leaks, security (authn/authz), input validation, versioning
- *UI:* accessibility, loading/error states, race conditions, mobile/responsive breakpoints
- *State machines:* unreachable states, invariant violations, transition ordering
- *Concurrency:* races, deadlocks, lock ordering, cleanup on cancellation
- *Data:* schema migrations, null handling, encoding issues, idempotent processing

The day-1 BOOTSTRAP fills the right list in based on your project's domain.

### eval-runner

**Inputs:** an eval script + the phase doc with the acceptance gate.

**Outputs:** a markdown report at `docs/evals/phase-N-<eval>-<date>.md` declaring PASS or FAIL.

**Discipline:** never modifies source code. Records git SHA, exact command, seed, wall time, dep versions for reproducibility. Compares to previous report; flags regressions loudly. Doesn't summarize away noise — if results are noisy, runs more seeds.

### doc-keeper

**Inputs:** the entire repo state.

**Outputs:** updates to `CLAUDE.md`, `docs/phases/`, indexes to match reality.

**Discipline:** truth is the code. When docs disagree with code, fix the doc. **Current phase section in `CLAUDE.md` must fit in ≤ 12 lines** — if it bloats, archive prior content and rewrite. (This rule is in the template because doc-bloat is the #1 recurring drift.) No invented docs; if rationale is unclear, asks instead of fabricating. Touches only docs.

## The phase loop

A typical sub-phase session:

1. User opens Claude in the project directory; `CLAUDE.md` loads automatically.
2. User pastes a kickoff prompt (drafted by the main session): "Sub-phase Nx per `docs/phases/phase-N-...md`. Pre-decisions: ... Pipeline: researcher → implementer → reviewer → eval-runner. Commit per CLAUDE.md."
3. **Researcher** writes or updates `docs/research/<topic>.md`.
4. **Implementer** writes code + tests against the note.
5. **Reviewer** independently audits; files findings; possibly REJECTs and round-trips with implementer.
6. **Eval-runner** runs the gate script and writes a PASS/FAIL report.
7. The session commits the sub-phase as a single git commit.
8. **Doc-keeper** updates `CLAUDE.md` "Current phase" and the phase doc's status.

If any step fails (gate FAIL, reviewer REJECT, etc.), the session does not advance — the failure is real and gets fixed before the next sub-phase starts.

## Anti-patterns

Things to actively avoid:

- **Bundling sub-phases into one commit** — destroys the audit trail.
- **`git add -A` or `git add .`** — accidentally commits caches, .env files, build artifacts. Always add specific files.
- **Skipping the eval-runner** — "I tested it" from the implementer is not a passing gate.
- **Single-agent execution** — one agent that "researches, implements, reviews, and runs evals" is a worse outcome than five separate ones, even though it's faster.
- **Speculative abstraction** — building a generic interface for the second game when you have one game. Wait for the second game.
- **CLAUDE.md bloat** — accumulating decades of changelog into the bible. The doc-keeper rule (≤ 12 lines for current phase) prevents this.
- **Continuing on a failed gate** — "we'll fix it next sub-phase" never happens. Fix it now or recalibrate the gate transparently with an ADR.

## Recovery: when the methodology has slipped

If you discover late in the project that:

- Multiple sub-phases never got committed
- `CLAUDE.md` has drifted from reality
- Decisions were made without ADRs
- Eval reports embed wrong git SHAs

Don't panic. Recovery is mechanical:

1. **Stop forward progress.** Don't start the next sub-phase until recovery is done.
2. **Diagnose with `git status`** and the agents' session reports.
3. **Make catch-up commits**, one per missed sub-phase, in chronological order. Use commit messages that honestly describe what happened ("phase-1d: alternating CFR + MCCFR — landed during 2026-05-06 session, committed during 2026-05-08 recovery sweep"). Don't rewrite history.
4. **Run a doc-keeper sweep** to bring `CLAUDE.md` back in line with reality.
5. **File ADRs retroactively** for any major decisions that lack them.
6. **Update the methodology rules** in `CLAUDE.md` to prevent recurrence (e.g., add the missing rule that caused the drift).

## Customizing the template

The template ships generic. Day-1 customization happens in the `BOOTSTRAP.md` flow — Claude interviews you about domain, stack, eval semantics, timeline, and rewrites the template's placeholders.

After day 1, you can keep customizing:

- **Reviewer's failure-mode list** — narrow it as you learn what bugs your domain actually hits.
- **Eval-runner's report template** — domain-specific report sections (e.g., visual diffs for UI projects).
- **Phase docs** — your phase shape will not match the template's example phases.
- **Conventions** — language, test runner, commit-message format, deployment targets.

The methodology's spine (five agents, phase loop, hard gates, commit discipline, project bible) does not change.
