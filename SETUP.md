# SETUP — the AI-mode installer

This file is instructions **for Claude**, not for the user. A fresh Claude
Code session is pointed at this repo (see the README's one-paste prompt),
clones it, reads this file, and follows it. The user shouldn't have to pick
between "new vs existing" or run shell commands by hand — you figure out
what's needed and do it, asking only the questions that actually matter.

You are about to take over as this project's **project-lead** (see
`template/PROJECT-LEAD.md`). Everything below gets you there.

## Vocabulary

- **PROJECT** — the directory the user opened Claude in (your CWD). This is
  the repo the methodology is being applied *to*.
- **REPO** — your local clone of phased-agents. If you haven't cloned it
  yet, do it now into a temp dir:
  `git clone --depth 1 https://github.com/ameri110/phased-agents.git /tmp/phased-agents-src`
  Then `REPO=/tmp/phased-agents-src`. Never run the methodology *from*
  REPO — REPO only provides the script and template.

## Step 1 — figure out which of four situations you're in

Inspect PROJECT (do not modify anything yet):

1. **Resume** — `PROJECT/CLAUDE.md` exists and contains **no** `{{...}}`
   placeholders. The methodology is already installed *and* customized.
   → go to **Resume**.
2. **Finish bootstrap** — `PROJECT/CLAUDE.md` exists but still has `{{...}}`
   placeholders. Files installed, day-1 interview never finished.
   → go to **Bootstrap**, using whichever `BOOTSTRAP.md` is already present.
3. **Fresh install, empty project** — no `CLAUDE.md`, and PROJECT has little
   or no existing source (empty dir, or just a README/license).
   → go to **Install (empty)**, then **Bootstrap**.
4. **Fresh install, existing codebase** — no `CLAUDE.md`, but PROJECT has
   real source code already.
   → go to **Install (existing)**, then **Bootstrap**.

In the **Resume** case you also check for methodology updates automatically
(see Resume below) and offer them — the user does not need to ask. If the
user *explicitly* said "update", skip straight to **Update** and apply the
full refresh.

When you're unsure between empty and existing, ask the user one plain
question ("Is this a brand-new project, or does it already have code I'm
adding the methodology to?") and don't guess.

## Install (empty)

The project is essentially empty. Set up in place:

1. If PROJECT is not a git repo yet: `git -C PROJECT init`.
2. Run the script's adopt (it installs into the current dir):
   `cd PROJECT && "$REPO/bin/phased-agents" adopt`
3. Adopt installs the *existing-repo* bootstrap. Since this is a fresh
   project, replace it with the new-project variant:
   `cp "$REPO/template/BOOTSTRAP.md" PROJECT/BOOTSTRAP.md`
   (overwrites the adopt variant adopt just placed — that's intended here).
4. Continue to **Bootstrap**.

## Install (existing)

The project already has code. Set up in place without disturbing it:

1. Ensure PROJECT is a git repo (it should be; if not, ask before
   `git init` — don't initialize someone's source tree silently).
2. `cd PROJECT && "$REPO/bin/phased-agents" adopt`
   (adopt backs up any existing `CLAUDE.md` to `CLAUDE.md.bak`, installs
   only files that don't already conflict, and ships the audit-style
   `BOOTSTRAP.md`).
3. Continue to **Bootstrap**.

## Bootstrap

Run the day-1 interview that customizes the generic template into this
project. Read `PROJECT/BOOTSTRAP.md` and follow it exactly — it tells you
to interview the user (domain, stack, eval semantics, timeline), rewrite
`CLAUDE.md`'s placeholders, tune the reviewer's bug-list, draft the phase
plan and Phase 0, and write the first ADR. BOOTSTRAP ends by handing the
user the floor as project-lead. Don't write production code during this.

## Resume

The methodology is already set up. You are a *returning* project-lead, not
an installer.

**First, check for methodology updates** (you already cloned REPO, so the
latest files are in hand). Diff the **role docs** between PROJECT and
`$REPO/template`:

- `PROJECT-LEAD.md`, `RESUME-LEAD.md`

These two are pure role contracts — never project-specific — so they are the
only reliable staleness signal: a difference means the methodology changed
since this project was set up. **Do NOT diff `CLAUDE.md`, the agents, or any
`docs/**` file for this purpose** — `CLAUDE.md` and the agents are
customized, and `docs/*/README.md` holds the project's *own* phase index and
ADR index. All of those always differ, so using them would either spam a
false "update available" on every resume or risk proposing to overwrite real
project content. Decide:

- **All match** → say nothing about updates; just resume.
- **Some differ** → fold a one-line "📦 methodology updates available (N
  files)" into your briefing and **offer** to apply them via **Update**
  (below). Don't apply without the user's go-ahead — show the diffs and let
  them confirm. The offer is safe to make because Update diff-asks before
  touching anything customized.

Then read durable state (`CLAUDE.md`, `PROJECT-LEAD.md`, phase docs, git log,
recent eval/review reports), give the under-200-word briefing (with the
updates line if any), flag anomalies, and hand the user the floor. Do not
start forward work without their input.

## Update

Reached two ways: the user explicitly said "update", **or** a Resume
detected stale role docs and the user accepted the offer. Either way, only
refresh the **safe surface** — never blow away their customizations or
project content. Sort every candidate into one of three tiers:

**1. Overwrite directly** from `$REPO/template/` — pure methodology, not
meant to be hand-edited:
- `PROJECT-LEAD.md`, `RESUME-LEAD.md` — role contracts. (If the project's
  copy has local edits beyond the template's own changes, drop to tier 2.)

**2. Diff and ask** — may carry local customization; show the diff and let
the user decide per file:
- `.claude/agents/*.md` — agents are routinely domain-customized (the
  reviewer's "Bugs to hunt for", the eval-runner's report template).
- `docs/*/_*-template.md` — the per-phase / per-ADR scaffold templates; a
  project may have tweaked them.

**3. NEVER overwrite — project content, not methodology:**
- `CLAUDE.md` — customized. Show a diff of any *structural* template changes
  (new sections, changed rules) and let the user merge by hand.
- `docs/*/README.md` — these are the project's **phase index** and **ADR
  index** (`docs/phases/README.md`, `docs/decisions/README.md`) and the like.
  They start as template scaffolds but become living project state. Blindly
  copying the template over them destroys the index. Leave them; at most,
  mention a relevant scaffold change for the user to fold in manually.
- Anything under `docs/phases|decisions|research|evals|reviews/` that holds
  real project content.

After updating, summarize what changed and let the project-lead commit it
(only the lead/user commits to `main` — see `PROJECT-LEAD.md`).

## Notes

- Prefer running `bin/phased-agents` for file installs; it's deterministic.
  Only hand-copy the few files this guide names explicitly.
- After you finish setup or resume, you ARE the project-lead. Operate per
  `PROJECT-LEAD.md` from then on — supervised by default, autonomous only
  if the user opts in.
