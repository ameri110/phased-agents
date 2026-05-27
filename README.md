# phased-agents

**A methodology for AI-driven long-running software projects, plus a copy-and-go template for Claude Code.**

If you've ever started a multi-month project with an AI coding agent and watched it drift into chaos around month two — losing track of what's been decided, why, what's actually working vs what just compiles — this is for you.

## The shape of it

Standard Claude Code is excellent at single tasks and bad at long projects. The reasons: no shared memory across sessions, no audit trail of decisions, no enforced verification, no protection against subtle bugs that pass tests but don't actually work.

`phased-agents` adds five things on top:

1. **A project bible** (`CLAUDE.md`) loaded into every session — goal, current phase, conventions, how to run things.
2. **Five specialized subagents** with focused roles: `researcher`, `implementer`, `reviewer`, `eval-runner`, `doc-keeper`.
3. **Phases with hard acceptance gates** — measurable contracts for each chunk of work. No "directionally good enough."
4. **A pipeline** the agents execute together: research → implement → review → eval → commit → doc-keep.
5. **Commit discipline** — one sub-phase, one commit, with messages that explain what landed and why.

The result: you can come back after a week and know exactly where the project is, why every decision was made, and what's the next concrete thing to do. Sub-phases hand off cleanly across separate Claude sessions.

## When to use this

Right-sized for projects with at least one of:

- Multi-month timeline
- Risk of silent failures (algorithms, security, performance, design correctness)
- Multi-domain expertise needed (you'll need to consult papers, RFCs, design systems, etc.)
- Decisions that have lasting consequences

Overkill for one-week projects, throw-away prototypes, or "build me a marketing site." Use a single Claude session for those.

## Quick start

You don't run an installer or pick a "scenario." Open Claude Code in your project directory (or an empty directory for a brand-new project) and paste one prompt:

```
Set up the phased-agents methodology in this directory. Clone
https://github.com/ameri110/phased-agents into a temp dir, read its SETUP.md,
and follow it: work out whether this is a new or existing project, install
accordingly, run the day-1 interview, and then take over as project-lead.
```

That's it. Claude clones the repo, reads [`SETUP.md`](./SETUP.md), detects which situation you're in, installs the methodology files in place, interviews you (domain, stack, eval semantics, timeline) to customize the generic template, and ends by taking over as your **project-lead**. No code is written before that interview.

**Returning to a project later?** Paste the same prompt. `SETUP.md` notices the methodology is already set up and customized, so instead of reinstalling it resumes as the project-lead and briefs you on where things stand. (If you prefer, you can still open Claude and paste `RESUME-LEAD.md` directly — same result.)

**Updating an existing project to newer methodology files?** Paste the same prompt and say you want to *update*. `SETUP.md` refreshes only the safe surface (agent definitions, doc scaffolding, role docs) and never clobbers your customized `CLAUDE.md` or your `docs/` content without showing you a diff first.

### Which file does what

You rarely touch these directly — `SETUP.md` routes to the right one — but it helps to know the map:

| File | What it is | When it's used |
|---|---|---|
| `SETUP.md` | The installer brain (instructions to Claude) | Every time, via the paste prompt above |
| `BOOTSTRAP.md` | The day-1 interview prompt | First-time setup, run by SETUP |
| `RESUME-LEAD.md` | The "wake up a project-lead" prompt | Every session after day-1 |
| `PROJECT-LEAD.md` | The project-lead's role *contract* (reference) | Read by the lead — never pasted to start a session |

### Manual / scriptable path (advanced)

If you'd rather drive it yourself, or wire it into CI, the underlying script is still there:

```bash
git clone https://github.com/ameri110/phased-agents.git
cd phased-agents

./bin/phased-agents install                          # install the 5 agents globally to ~/.claude/agents/
./bin/phased-agents new ~/Desktop/projects/my-thing  # scaffold a new project
cd ~/my-existing-project && /path/to/phased-agents/bin/phased-agents adopt  # retrofit an existing repo
```

Then open Claude in the project and paste `BOOTSTRAP.md` (new/adopt) or `RESUME-LEAD.md` (returning). Re-run `install` with `--force` to overwrite existing agent files.

## How the pieces fit

| Piece | Lives at | Job |
|---|---|---|
| Project bible | `CLAUDE.md` | Loaded every session; current state of the world |
| Phase plans | `docs/phases/` | Contracts with acceptance gates |
| Decisions | `docs/decisions/` | ADRs for non-obvious choices |
| Research notes | `docs/research/` | Source-material summaries the implementer reads |
| Eval reports | `docs/evals/` | PASS/FAIL evidence anchored to git SHAs |
| Reviews | `docs/reviews/` | Independent audits before declaring done |
| Subagents | `.claude/agents/` | Five specialized roles |

## The five agents in one line each

- **researcher** — reads sources (papers, RFCs, design docs), writes implementation-ready notes
- **implementer** — writes code from a finished spec; will refuse to start without one
- **reviewer** — independently audits implementations against the source spec, adversarially
- **eval-runner** — runs eval scripts, declares PASS/FAIL against the phase gate, never edits source
- **doc-keeper** — keeps `CLAUDE.md` and phase docs in sync with reality, periodic sweeps

The body of each (what they audit for, what tools they use) is generic in this template. Day-1 customization tunes them to your domain — the `reviewer` for an algorithms project hunts for sign errors, the `reviewer` for an API project hunts for authn/authz leaks.

## The sixth role: project-lead

Above the five subagents sits a sixth role: the **project-lead**. It's the Claude session you (the human) drive directly in the project root. It reads sub-session reports, surfaces decisions, drafts kickoff prompts for the next pipeline, and catches process drift. It does *not* write code, run evals, or audit — those are the subagents' jobs.

Because it lives in your conversation rather than in a `.claude/agents/` file, it needs a way to respawn when your session ends. The template ships two files for that:

- `PROJECT-LEAD.md` — describes the role
- `RESUME-LEAD.md` — the prompt you paste into a fresh Claude Code session to bring up a new project-lead from the durable state (`CLAUDE.md` + git log + recent reports)

Open Claude in your project root, paste `RESUME-LEAD.md`, get a briefing, continue where you left off. No context is lost because the durable state lives in files.

## Read more

- **[SETUP.md](./SETUP.md)** — the AI-mode installer Claude follows to set up, resume, or update a project
- **[METHODOLOGY.md](./METHODOLOGY.md)** — the philosophy, why each rule exists, anti-patterns, customization guide
- **[template/](./template/)** — the actual files copied into your project
- **[bin/phased-agents](./bin/phased-agents)** — the entry-point script (manual path)

## Contributing

This is a methodology, not a framework — there's almost no code. If you adapt it for a domain (UI, infra, mobile, etc.) and find generic improvements worth back-porting, PRs welcome. Domain-specific case studies under `examples/` are also welcome.

## License

MIT. See [LICENSE](./LICENSE).
