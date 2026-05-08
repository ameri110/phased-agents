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

### One-time, optional: install agents globally

If you'll use the methodology across multiple projects, install the five subagents to `~/.claude/agents/` once. They're then available in every project without per-project copies. Project-scoped agents (in `<project>/.claude/agents/`) override these globals when present.

```bash
git clone https://github.com/<you>/phased-agents.git
cd phased-agents
./bin/phased-agents install
```

Re-run with `--force` to overwrite existing agent files.

### Pick your scenario

|  | **Public** — files committed to the project repo | **Personal** — files invisible to the project repo |
|---|---|---|
| **New project** | [Public + new](#public--new) | [Personal + new](#personal--new) |
| **Existing project** | [Public + existing](#public--existing) | [Personal + existing](#personal--existing) |

**Public** is the default — methodology files live in the project repo and your team sees them. Best when teammates use Claude Code, or are happy with `docs/decisions/` (ADRs), `docs/phases/` (phase plans), and `docs/evals/` (PASS/FAIL reports) being committed regardless of who or what authored them.

**Personal** keeps everything out of the project repo. Files live in `~/.phased-agents/` (a private git repo you push to a private remote for backup), and the project gets symlinks pointing into the vault. Entries in `.git/info/exclude` hide the symlinks from git, so teammates never see a phased-agents file. Best when you don't want to impose AI tooling on teammates, or when you just want your workflow private.

In all four cases, day-1 setup is the same: open Claude in the project root and paste `BOOTSTRAP.md`. Claude interviews you (domain, stack, eval semantics, timeline) and customizes the template before any code is written.

---

### Public + new

```bash
./bin/phased-agents new ~/Desktop/projects/my-thing
cd ~/Desktop/projects/my-thing
claude
# paste the contents of BOOTSTRAP.md when Claude opens
```

The first commit lands after Claude finishes the day-1 BOOTSTRAP and customizes the template.

### Public + existing

```bash
cd ~/Desktop/projects/my-existing-project
/path/to/phased-agents/bin/phased-agents adopt
claude
# paste the contents of BOOTSTRAP.md when Claude opens
```

`adopt` backs up any existing `CLAUDE.md` to `CLAUDE.md.bak` and only installs files that don't already conflict. Its BOOTSTRAP variant asks Claude to *audit* your repo first and propose a phase plan that picks up where you are, rather than starting from zero.

### Personal + new

```bash
# One-time vault setup (skip if you've already done this)
./bin/phased-agents personal init
cd ~/.phased-agents
git remote add origin git@github.com:<you>/phased-agents-vault.git
git push -u origin main

# Per-project
./bin/phased-agents new ~/Desktop/projects/my-thing
cd ~/Desktop/projects/my-thing
/path/to/phased-agents/bin/phased-agents personal link
claude
# paste the contents of BOOTSTRAP.md when Claude opens
```

`personal link` moves the just-scaffolded methodology files into the vault and replaces them with symlinks; it appends a managed block to `.git/info/exclude` so git doesn't see them. The project repo's first commit will only contain whatever non-phased-agents content `phased-agents new` happens to ship — typically nothing project-specific, so commit that or don't.

### Personal + existing

Adopt the methodology in an existing repo without ever committing a phased-agents file to it.

```bash
# One-time vault setup (skip if you've already done this)
./bin/phased-agents personal init
cd ~/.phased-agents
git remote add origin git@github.com:<you>/phased-agents-vault.git
git push -u origin main

# Per-project: install methodology files, then immediately move them into the vault
cd ~/Desktop/projects/my-existing-project
/path/to/phased-agents/bin/phased-agents adopt
/path/to/phased-agents/bin/phased-agents personal link
# (interactive — confirms before migrating any files)

claude
# paste the contents of BOOTSTRAP.md when Claude opens
```

Because the files were just installed by `adopt` and not yet committed, `personal link` cleanly moves them into the vault without any project-repo deletions to commit afterward. If you're migrating a project that *already* had phased-agents files committed, `personal link` will also stage `git rm` for those — review with `git status` and commit in the project repo: `git commit -m "chore: switch to phased-agents personal mode"`.

### After day-1 (any mode)

The pipeline runs the same regardless of mode: researcher → implementer → reviewer → eval-runner → commit → doc-keeper. The one difference for **personal mode** is that methodology bookkeeping commits go to the vault, not the project repo:

```bash
# Periodically, or after each sub-phase:
phased-agents personal commit -m "phase-Nx: doc-keeper sync"
cd ~/.phased-agents && git push
```

See **[PERSONAL-MODE.md](./PERSONAL-MODE.md)** for personal-mode details (multi-machine workflow, recovery, caveats, two-commit boundary).

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

- **[METHODOLOGY.md](./METHODOLOGY.md)** — the philosophy, why each rule exists, anti-patterns, customization guide
- **[PERSONAL-MODE.md](./PERSONAL-MODE.md)** — Mode C: keep phased-agents files out of the project repo entirely
- **[template/](./template/)** — the actual files copied into your project
- **[bin/phased-agents](./bin/phased-agents)** — the entry-point script

## Contributing

This is a methodology, not a framework — there's almost no code. If you adapt it for a domain (UI, infra, mobile, etc.) and find generic improvements worth back-porting, PRs welcome. Domain-specific case studies under `examples/` are also welcome.

## License

MIT. See [LICENSE](./LICENSE).
