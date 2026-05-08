# The project-lead role

The **project-lead** is the Claude session you (the human) drive directly
in this project's root directory. It is not a subagent — it is the
conversational layer above them, and it is the role that holds your
project together across many sub-sessions.

## What the project-lead does

- Reads sub-session reports and translates them into plain-English
  summaries you can act on
- Surfaces decisions that need your input, with options + tradeoffs +
  a clear recommendation
- Drafts copy-pasteable kickoff prompts for the next sub-session's
  pipeline (researcher → implementer → reviewer → eval-runner)
- Catches process drift: uncommitted work, doc-bloat, missed ADRs,
  dropped balls
- Holds long-term context: the *why* behind decisions, the state of the
  roadmap, what's caveated and what isn't

## What the project-lead does NOT do

- Write production code — that's the implementer in a focused sub-session
- Run evals — that's the eval-runner
- Audit code — that's the reviewer
- Bulk-edit `CLAUDE.md` or phase docs — that's the doc-keeper

The project-lead's tools are conversation, decision-framing, and
prompt-drafting. Not execution.

## Where it lives

There is no `.claude/agents/project-lead.md`. The role is not invoked via
the subagent system; it is the default conversation in the project root
when you run `claude` there.

Practical version: open Claude Code in the project root, paste the
contents of `RESUME-LEAD.md` to prime the role from durable state, then
talk normally.

## When the current project-lead session ends

Closed, crashed, hit context limits — doesn't matter. To spawn a fresh
project-lead:

1. Open Claude Code in this project's root directory.
2. Paste the contents of `RESUME-LEAD.md`.
3. Wait for the briefing.
4. Continue where you left off.

Because the durable state lives in files (`CLAUDE.md`, git log, phase
docs, eval reports, review reports), the new project-lead recovers the
same information the previous one had. You don't lose ground.

You can also intentionally start a fresh project-lead at any time — for
example, if the current one's context has drifted or you want to "rubber
duck" a decision against a clean read of the project state.

## How the project-lead and sub-sessions interact

A typical week looks like:

1. The project-lead briefs you on state and proposes next steps.
2. You agree on a sub-phase.
3. The project-lead drafts a kickoff prompt for that sub-phase's
   pipeline.
4. You open a **new** Claude Code session (or a sub-agent invocation)
   and run the pipeline.
5. The sub-session reports back to you when done.
6. You return to the project-lead session and paste the report.
7. The project-lead translates the report, surfaces follow-up
   decisions, and drafts the next kickoff prompt.

Sub-sessions are short (one sub-phase). The project-lead session is
long-running but cheap to recreate.

## Why this works

Long projects fail when the human has to remember everything. The
project-lead role is explicitly designed so the human only drives
strategic intent — every other piece of context is recoverable from
files. The doc-keeper's ≤ 12-line cap on "Current phase" and the commit-
discipline rule are what make the durable state durable enough to
support this.
