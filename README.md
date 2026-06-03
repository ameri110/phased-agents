# phased-agents

**A methodology for AI-driven long-running software projects, plus a copy-and-go template for Claude Code.**

If you've started a multi-month project with an AI coding agent and watched it drift into chaos around month two — losing track of what's decided, why, and what actually works vs. what merely compiles — this is for you.

## What it adds

Standard Claude Code is great at single tasks and weak on long projects: no memory across sessions, no decision audit trail, no enforced verification, no guard against bugs that pass tests but don't work. `phased-agents` adds:

1. **A project bible** (`CLAUDE.md`) loaded every session — goal, current phase, conventions, how to run things.
2. **Five specialized subagents** — researcher, implementer, reviewer, eval-runner, doc-keeper — each with one job, so they catch each other's mistakes.
3. **Phases with hard acceptance gates** — measurable contracts per chunk of work. No "directionally good enough."
4. **A pipeline** — research → implement → review → eval → doc-keep — that each sub-phase runs and lands as a PR.
5. **PR discipline** — only the project-lead merges to `main`, so it stays a clean audit trail. Independent sub-phases run in parallel, each on its own branch/worktree.

Come back after a week and you know exactly where things are, why each decision was made, and what's next.

## When to use it

Worth the overhead when a project has at least one of: a multi-month timeline, risk of silent failures (algorithms, security, performance, correctness), multi-domain expertise (papers, RFCs, design systems), or decisions with lasting consequences.

Overkill for one-week builds, throwaway prototypes, or marketing sites — use a plain Claude session for those.

## Quick start

Open Claude Code in your project directory (or an empty one for a new project) and paste:

```
Use the phased-agents methodology in this directory. Clone
https://github.com/ameri110/phased-agents into a temp dir, read its SETUP.md,
and follow it — it detects whether to install fresh, resume, or update an
existing setup, then takes over as project-lead.
```

**One prompt covers all three cases.** [`SETUP.md`](./SETUP.md) detects from the project's state which applies: a first-time **install** customizes the generic template via a day-1 interview; a **resume** reads durable state and briefs you. No code is written before the interview.

**Updates are automatic on resume.** Because the prompt clones the latest phased-agents every time, a resume compares your installed **role docs** against it and, if they changed, offers to apply the update. Accept and it refreshes the pure-methodology files (the role contracts) directly, **diffs and asks** before touching anything you may have customized (agents, scaffold templates, `CLAUDE.md`), and **never** touches your project content — your phase index and ADR index are safe. So every repo that uses this method gets the latest by pasting the same prompt; you just confirm the offer, and nothing is overwritten silently. (Saying *"update the methodology files"* still forces the refresh if you ever want to trigger it explicitly.)

### Which file does what

| File | What it is | When |
|---|---|---|
| `SETUP.md` | Installer brain (instructions to Claude) | Every time, via the prompt above |
| `BOOTSTRAP.md` | Day-1 interview prompt | First-time setup, run by SETUP |
| `RESUME-LEAD.md` | "Wake up a project-lead" prompt | Sessions after day-1 |
| `PROJECT-LEAD.md` | The lead's role *contract* (reference) | Read by the lead — never pasted |

## The agents and the lead

- **researcher** — reads sources, writes implementation-ready notes
- **implementer** — writes code from a finished spec; refuses to start without one
- **reviewer** — independently, adversarially audits the code against the spec
- **eval-runner** — runs the gate, declares PASS/FAIL, never edits source
- **doc-keeper** — keeps `CLAUDE.md` and phase docs in sync with reality

Each is generic in the template; day-1 customization tunes them to your domain (the reviewer for an algorithms project hunts sign errors; for an API project, authn/authz leaks).

Above them sits the **project-lead** — the session you drive directly. It briefs you, surfaces decisions, drafts kickoff prompts, reviews PRs and merges to `main`, and catches drift. It doesn't write code, run evals, or audit — those are the subagents'. It runs *supervised* by default (you open the sub-sessions) or *autonomous* if you opt in (it spawns them as background subagents, still escalating every real decision to you). Because it lives in your conversation, you respawn it any time with `RESUME-LEAD.md` — no context lost, since durable state lives in files.

## Repo layout

| Piece | Lives at | Job |
|---|---|---|
| Project bible | `CLAUDE.md` | Current state of the world, loaded every session |
| Phase plans | `docs/phases/` | Contracts with acceptance gates |
| Decisions | `docs/decisions/` | ADRs for non-obvious choices |
| Research notes | `docs/research/` | Source-material summaries |
| Eval reports | `docs/evals/` | PASS/FAIL evidence anchored to git SHAs |
| Reviews | `docs/reviews/` | Independent audits |
| Subagents | `.claude/agents/` | The five roles |

## Manual / scriptable path

Prefer to drive it yourself or wire it into CI? The script underneath:

```bash
git clone https://github.com/ameri110/phased-agents.git && cd phased-agents
./bin/phased-agents install        # install the 5 agents globally to ~/.claude/agents/
./bin/phased-agents new <path>     # scaffold a new project
cd <existing-repo> && /path/to/phased-agents/bin/phased-agents adopt   # retrofit
```

Then paste `BOOTSTRAP.md` (new/adopt) or `RESUME-LEAD.md` (returning) into Claude.

## Read more

- **[SETUP.md](./SETUP.md)** — the AI-mode installer
- **[METHODOLOGY.md](./METHODOLOGY.md)** — the philosophy, why each rule exists, anti-patterns
- **[template/](./template/)** — the files copied into your project

## Contributing

A methodology, not a framework — almost no code. Generic improvements and domain case studies (under `examples/`) welcome via PR.

## License

MIT. See [LICENSE](./LICENSE).
