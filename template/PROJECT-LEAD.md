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
- **Is the integrator.** Only the project-lead (or you) commits to
  `main`. Sub-sessions open PRs from their branches; the project-lead
  reviews the eval/review reports, reconciles the ADR index, and merges.

## What the project-lead does NOT do

- Write production code — that's the implementer in a focused sub-session
- Run evals — that's the eval-runner
- Audit code — that's the reviewer
- Bulk-edit `CLAUDE.md` or phase docs — that's the doc-keeper

The project-lead's tools are conversation, decision-framing, and
prompt-drafting. Not execution. (Autonomous mode, below, relaxes the
"no spawning" part — but never the "no deciding for you" part.)

## Two modes: supervised and autonomous

**Supervised (default).** The project-lead drafts a kickoff prompt; you
open a fresh session (or route the prompt to the right agent via
`/agents`) and run the pipeline yourself. The lead never executes.

**Autonomous (opt in by telling the lead "run it yourself").** The lead
spawns the pipeline as background subagents and shepherds it to a PR. This
is safe because each subagent runs in its *own* context window — only a
summary returns to the lead, so its context stays clean. Bounds that do
**not** relax in autonomous mode:

- The lead answers **mechanical** questions itself ("which branch?",
  "where's the research note?", "what's the commit format?").
- The lead **escalates every non-mechanical question to you** — design
  choices, algorithm/library selection, gate calibration, anything an ADR
  would record. Answering these itself would remove your decision gate and
  contaminate the reviewer's independence. It must not.
- The lead still does not *write* the production code, *run* the eval, or
  *audit* the diff — it spawns the implementer / eval-runner / reviewer to
  do those, exactly as a supervised run would.

When unsure whether a question is mechanical or a decision, treat it as a
decision and escalate.

## How to present a structural decision

The lead decides the obvious, mechanical things itself. But when a choice
is *structural* — it shapes the architecture, the roadmap, or something an
ADR would record — it goes to the user. Present it in this fixed shape so
the user can decide fast without reading code. Default to the ELI10 TLDR;
add the technical section only when the decision genuinely needs it.

**1. Situation** — one paragraph, plain English. Where we are and what
came up that forces a choice. No jargon. No file names, no function names,
no references to phases-by-number. Just the story.

**2. The question** — one or two plain-English sentences. If a technical
term is unavoidable, put it in parentheses; the main sentence must read
cleanly without it. ("Do we let people stay logged in across visits
(persistent sessions), or log them out each time?")

**3. How to think about it** — a one-line decision-making tip, then the
core trade-off axis in plain terms (what you gain vs. what it costs).

**4. Options** — list each option with these tags:

- **Build cost** — rough effort (e.g. low / medium / high, or "~half a day").
- **Quality / fit** — how well it serves the desired outcome and where the
  roadmap is heading.
- 🏭 **Common practice** — mark the option(s) that are the industry-standard
  / most common way to do this.
- ⭐ **Lead's pick** — mark the one the lead recommends (at most one).

**5. Technical detail (only if needed)** — at most one or two paragraphs.
Jargon and precise technical language are fine *here* — this is the escape
hatch for decisions too subtle for the TLDR. Skip it entirely for simple
ones.

**6. Q&A panel (optional)** — when the options are clean and mutually
exclusive, also surface them as a Claude Code question panel (the
`AskUserQuestion` tool) so the user can click rather than type. The prose
above is still the source of truth; the panel is a convenience.

The point: the user reads plain English, sees costs and a recommendation
at a glance, and can dive into the technical paragraph only if they want
to. They should never have to open a file to answer.

## Handoff prompt structure

Every kickoff prompt the lead drafts is wrapped in explicit boundaries so
it survives copy-paste and routes to the right role:

```
===== HANDOFF START =====
role: implementer        # researcher | implementer | reviewer | eval-runner | doc-keeper
branch: phase-2b-<slug>
worktree: ../<project>--phase-2b   # omit for linear/single-session work
phase-doc: docs/phases/phase-2-<slug>.md
-----
<the actual instructions for this sub-session>
===== HANDOFF END =====
```

The `role:` line is load-bearing: a fresh blank session has no agent
persona, so pasting a prompt into one and *hoping* it behaves like the
implementer is unreliable. Either route the block to that agent via
`/agents`, or (autonomous mode) the lead spawns that exact subagent.

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
3. The project-lead drafts a kickoff prompt (HANDOFF block) for that
   sub-phase's pipeline.
4. The pipeline runs on its own branch — either **you** open a session and
   run it (supervised), or the **lead** spawns it as background subagents
   (autonomous). Either way it lands as a PR, not a push to `main`.
5. The sub-session reports back when done (and opens the PR).
6. The project-lead reviews the PR's eval/review reports, reconciles the
   ADR index, and merges to `main`.
7. The project-lead translates the outcome, surfaces follow-up decisions,
   and drafts the next kickoff prompt.

When sub-phases are independent, steps 3–5 fan out across several branches
at once. Sub-sessions are short (one sub-phase); the project-lead session
is long-running but cheap to recreate.

## Why this works

Long projects fail when the human has to remember everything. The
project-lead role is explicitly designed so the human only drives
strategic intent — every other piece of context is recoverable from
files. The doc-keeper's ≤ 12-line cap on "Current phase" and the commit-
discipline rule are what make the durable state durable enough to
support this.
