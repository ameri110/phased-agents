# Day-1 Bootstrap — adopting in an existing project

Paste the section below into a fresh Claude Code session opened in this
directory. Unlike the new-project flow, this version starts by **auditing
the existing repo** and proposes a phase plan that picks up where you are.

---

I'm adopting the **phased-agents** methodology in an existing project. The
template files were just installed (see `CLAUDE.md`, `.claude/agents/`,
`docs/`). My previous `CLAUDE.md` (if any) is at `CLAUDE.md.bak`. I want
you to act as the project lead and integrate the methodology into the
work that's already here.

## Step 1: audit what's here

Read in this order, take notes:

1. `CLAUDE.md.bak` (if it exists) — what was the project's prior north star?
2. `README.md` — what's the project supposed to do?
3. `git log --oneline` — the past 30 commits. What's the trajectory?
4. The repo's source tree — module layout, main entry points, test
   structure.
5. Any existing `docs/` content from before adoption.
6. The package manifest — language, framework, deps, scripts.
7. The current `CLAUDE.md` template (placeholder version we just installed).

## Step 2: interview me

Ask me these, one at a time:

1. **Where is the project today?** What's working, what's broken, what's
   the current focus? (Two minutes' worth of context, not a status report.)
2. **What's the immediate goal?** Next milestone, next deliverable, next
   release.
3. **Domain category.** Pick one or describe a hybrid:
   algorithms / ML, API / backend, UI / web, mobile, infrastructure / DevOps,
   data engineering, embedded, other.
4. **Definition of "eval"** for this project. What does a passing
   acceptance gate look like for the work you're describing in #2?
5. **Pain points the methodology should solve.** Why are you adopting
   this — context loss across sessions? Silent bugs? Drifting code? No
   audit trail? All of the above?
6. **Anything in CLAUDE.md.bak I should preserve verbatim?** (Architecture
   notes, conventions, pointers, etc.)

## Step 3: integrate

Based on my answers:

1. **Merge `CLAUDE.md.bak` into the new `CLAUDE.md`.** Keep the new
   structure (the template's sections), but pull forward any project-
   specific architecture, conventions, and pointers from the backup.
   Adapt the "Current phase" to where the project actually is, not
   "Phase 0."
2. **Update the reviewer agent's failure-mode list** to match the
   domain.
3. **Reverse-engineer a phase plan from git history.** Look at the past
   commits and infer the phases the project has effectively gone through,
   even if they weren't formally tracked. Document them in
   `docs/phases/README.md` with retroactive `done` status, then propose
   the next 2–3 phases based on the current goal.
4. **Draft any missing ADRs** for non-obvious choices visible in the code
   (framework choice, weird workarounds, etc.). Don't fabricate rationale
   — for choices where the reasoning isn't recoverable, write a short
   ADR that says "rationale unknown; preserved as-is" so the decision is
   at least catalogued.
5. **Identify the next sub-phase to run** through the pipeline. Pick
   something that will exercise the methodology end-to-end on this codebase
   — not necessarily the most important thing, but the highest-clarity
   small thing.

## Step 4: present and confirm

Show me:

- The merged `CLAUDE.md`
- The customized agent files
- The reverse-engineered phase plan
- The ADRs you drafted
- Your recommendation for the next sub-phase, with a copy-pasteable
  kickoff prompt

Ask me to confirm or adjust. Iterate until I approve.

## Step 5: commit and hand off

Once I approve:

1. Commit the methodology adoption as one commit:
   `chore: adopt phased-agents methodology — initial customization`
   (Don't bundle code changes; this commit is docs + agents only.)
2. Tell me the prompt for the next session and stop.

## Important rules during this bootstrap

- **No production-code changes.** Only docs and agent definitions.
- **Don't break what's working.** Existing tests must still pass after
  this commit (they're docs-only changes, so they should).
- **Don't invent project history.** If you can't infer something from
  `git log` and the code, ask me.
- **Preserve `CLAUDE.md.bak`** in the commit so the merged history is
  recoverable.
