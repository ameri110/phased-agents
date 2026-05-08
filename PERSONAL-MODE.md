# Mode C — fully personal

Use Mode C when you want to use the phased-agents methodology **without
ever committing a phased-agents file to your project repo**. Useful when:

- You're on a team that doesn't use Claude Code (or doesn't want
  AI-tooling files in the repo)
- You want your AI workflow to stay private even on solo projects
- You work across many projects and want a single backup-able store of
  AI workflow state

## Mental model

Phased-agents files live in a **personal vault** — a separate git repo,
typically at `~/.phased-agents/`, that you push to a private GitHub for
backup. Each project you link gets a subdirectory in the vault. The
project itself contains symlinks pointing into the vault, with
`.git/info/exclude` entries that hide them from git.

```
~/.phased-agents/                        ← vault (private git repo)
├── .git/
├── README.md
├── projects.json                         ← project-name → real path map
└── projects/
    ├── poker/                            ← one dir per linked project
    │   ├── CLAUDE.md
    │   ├── PROJECT-LEAD.md
    │   ├── RESUME-LEAD.md
    │   ├── .claude/
    │   └── docs/
    │       ├── phases/  decisions/  research/  evals/  reviews/
    └── another-project/
        └── ...

~/Desktop/projects/poker/                 ← team's project repo
├── src/                                  ← committed (team)
├── .git/info/exclude                     ← local-only ignore entries
├── CLAUDE.md         → ~/.phased-agents/projects/poker/CLAUDE.md
├── .claude           → ~/.phased-agents/projects/poker/.claude
└── docs/
    └── phases        → ~/.phased-agents/projects/poker/docs/phases
        ...
```

Claude opens the project normally and reads through the symlinks. Your
teammates never see `CLAUDE.md` or any other phased-agents file in
`git status`.

## The five commands

```bash
phased-agents personal init                  # one-time, set up the vault
phased-agents personal link                  # per-project, in the project root
phased-agents personal relink <name>         # after a fresh vault clone on a new machine
phased-agents personal status                # health check across all linked projects
phased-agents personal commit -m "msg"       # commit pending vault changes
```

`personal help` documents flags and longer examples.

## Workflows

### First time: initialize the vault and back it up

```bash
phased-agents personal init
# vault created at ~/.phased-agents (or PHASED_AGENTS_VAULT)

cd ~/.phased-agents
git remote add origin git@github.com:<you>/phased-agents-vault.git
git push -u origin main
```

This is one-time setup. The vault is now version-controlled with a remote.

### Migrating an existing Mode-A project to Mode C

If your project already has phased-agents files committed to its repo
(`CLAUDE.md`, `.claude/agents/`, `docs/phases/`, etc.), `link` will
**migrate** them — move them into the vault, `git rm` them in the
project, replace with symlinks, and add the exclude block:

```bash
cd ~/Desktop/projects/poker
phased-agents personal link
# interactive — confirms before moving files
# moves CLAUDE.md, .claude/, docs/phases, ..., into the vault
# creates symlinks pointing at the vault
# stages git rm in the project repo

# Commit the deletions in the project (one-way migration):
git status   # review the rm's
git commit -m "chore: switch to phased-agents personal mode (Mode C)"

# Commit and push the vault (the file moves are auto-committed):
cd ~/.phased-agents
git push
```

After this, your team's repo no longer contains any phased-agents files.
You continue working as before; the durable state is just stored
elsewhere.

### New project: link it without migration

If the project doesn't have phased-agents files yet, `link` populates
the vault from templates:

```bash
cd ~/Desktop/projects/new-thing
phased-agents personal link
# no migration to confirm — vault populated from template
```

After this, the project's symlinks point to fresh template files in
the vault. Open Claude Code and start the BOOTSTRAP flow as you would
in any new project.

### New machine: clone the vault, relink each project

After your laptop dies / you switch machines / etc.:

```bash
git clone git@github.com:<you>/phased-agents-vault.git ~/.phased-agents
phased-agents personal relink poker
# uses the path from projects.json by default
# or pass --project /new/path if the project lives somewhere else now
```

Repeat `relink` for each project you've linked. Each one regenerates
its symlinks pointing at the freshly-cloned vault.

### Daily use

No different from any other mode. Open Claude in the project root,
operate normally. The methodology's pipeline (researcher → implementer
→ reviewer → eval-runner → commit → doc-keeper) is unchanged.

The one twist: **commits land in two repos**. Code commits go to the
project. CLAUDE.md / phase doc / ADR / eval / review commits go to the
vault. `phased-agents personal commit` is a small convenience for the
vault commit; the project commit is just `git commit` as usual.

### Periodic backups

```bash
phased-agents personal status        # see what's pending
phased-agents personal commit        # commit pending vault changes
cd ~/.phased-agents && git push     # push to your private remote
```

## What goes where

| File / dir | Lives in | Reason |
|---|---|---|
| Source code, tests, build configs, CI | Project repo | Team artifact |
| `README.md` (the team-facing one) | Project repo | Team artifact |
| `CLAUDE.md` | Vault, symlinked | Only Claude reads it |
| `PROJECT-LEAD.md`, `RESUME-LEAD.md` | Vault, symlinked | Personal AI workflow |
| `.claude/agents/`, `.claude/settings.json` | Vault, symlinked | Personal AI workflow |
| `docs/phases/`, `docs/decisions/`, `docs/research/` | Vault, symlinked | Methodology bookkeeping |
| `docs/evals/`, `docs/reviews/` | Vault, symlinked | Methodology bookkeeping |

## Caveats

- **`git status` won't show your phase-doc / ADR / eval changes.** They
  land in the vault, not the project. Get used to running `phased-agents
  personal status` (or `cd ~/.phased-agents && git status`) to check the
  vault.
- **Symlinks can be fragile across some tools.** If a tool refuses to
  follow them (rare), the workaround is to use copies + a sync script
  instead, but you lose the live-write property. So far symlinks work
  with Claude Code, pytest, ruff, mypy, and standard build tooling.
- **The migration is one-way.** Once you commit the deletions in the
  project repo, teammates pull and lose access to those files. If a
  teammate was actively reading your `docs/phases/`, coordinate before
  migrating. (For solo projects this isn't a concern.)
- **The vault is your responsibility.** It's a regular git repo; no
  magic. Push it. Back it up. If you lose it without a remote, you lose
  everything you've written under `docs/phases/`, `docs/decisions/`,
  etc. Set the remote during `init`'s "next steps" — don't defer.
- **If a project moves on disk**, run `relink --project <new-path>` to
  update the symlinks and the entry in `projects.json`.

## Recovering when things break

- **Symlinks are dangling** (e.g., vault moved): `phased-agents personal
  relink <name>`.
- **Vault project entry is stale** (e.g., project moved): `phased-agents
  personal relink <name> --project <new-path>`.
- **You linked the wrong path**: `phased-agents personal relink <name>
  --project <correct-path>` is the right fix; manually fixing
  `projects.json` is the wrong fix (the symlinks won't update).
- **You want to undo Mode C for a project**: not yet supported by a
  command. Manually: `cd <project>`, remove the symlinks, copy the
  files back from the vault, remove the `# >>> phased-agents personal
  mode` block from `.git/info/exclude`, `git add` the restored files,
  commit. Then optionally `rm -rf ~/.phased-agents/projects/<name>`.
