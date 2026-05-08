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

## Quick start — three entry points

### 1. Install agents globally (optional, one-time)

Makes the five subagents available in any project on your machine. Project-scoped agents (in `<project>/.claude/agents/`) override these when present.

```bash
git clone https://github.com/<you>/phased-agents.git
cd phased-agents
./bin/phased-agents install
```

This copies the agent definitions to `~/.claude/agents/`. Existing files are skipped (re-run with `--force` to overwrite).

### 2. Start a new project with the method

```bash
./bin/phased-agents new ~/Desktop/projects/my-thing
cd ~/Desktop/projects/my-thing
claude
# then paste the contents of BOOTSTRAP.md when Claude opens
```

What this does:

- Creates the project directory
- Copies the template (`CLAUDE.md`, `.claude/`, `docs/` skeleton, `BOOTSTRAP.md`)
- Initializes git
- Tells you how to kick off day-1 setup with Claude

The `BOOTSTRAP.md` is a prompt that walks Claude through interviewing you about your project (domain, stack, eval semantics, timeline) and customizing the template before any code is written.

### 3. Adopt the method in an existing project

```bash
cd ~/Desktop/projects/my-existing-project
/path/to/phased-agents/bin/phased-agents adopt
claude
# then paste the contents of BOOTSTRAP.md
```

What this does:

- Backs up any existing `CLAUDE.md` to `CLAUDE.md.bak` and installs the template version
- Adds `.claude/agents/` and `docs/` skeletons (only files that don't already exist)
- Drops a `BOOTSTRAP.md` tuned for adopting the method on existing code

The `BOOTSTRAP.md` for adoption asks Claude to *audit your existing repo first* and then propose a phase plan that picks up where you are, rather than starting from zero.

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

## Want to keep phased-agents files out of your project repo entirely?

For multi-developer projects where you don't want to impose AI tooling
on teammates, or solo projects where you just want your AI workflow
private, there's a fourth entry point: **personal mode** (Mode C). Files
live in a personal vault (a separate git repo, ~/.phased-agents/, you
push to a private remote) and the project gets symlinks pointing into
the vault. Entries in `.git/info/exclude` hide the symlinks from git.
Teammates see zero footprint.

```bash
phased-agents personal init                    # one-time, set up the vault
cd ~/Desktop/projects/poker
phased-agents personal link                    # migrate this project to Mode C
# (interactive — confirms before moving any existing files into the vault)
```

See **[PERSONAL-MODE.md](./PERSONAL-MODE.md)** for the full guide.

## Read more

- **[METHODOLOGY.md](./METHODOLOGY.md)** — the philosophy, why each rule exists, anti-patterns, customization guide
- **[PERSONAL-MODE.md](./PERSONAL-MODE.md)** — Mode C: keep phased-agents files out of the project repo entirely
- **[template/](./template/)** — the actual files copied into your project
- **[bin/phased-agents](./bin/phased-agents)** — the entry-point script

## Contributing

This is a methodology, not a framework — there's almost no code. If you adapt it for a domain (UI, infra, mobile, etc.) and find generic improvements worth back-porting, PRs welcome. Domain-specific case studies under `examples/` are also welcome.

## License

MIT. See [LICENSE](./LICENSE).
