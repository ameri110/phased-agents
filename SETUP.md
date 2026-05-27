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

If the user explicitly said "update" rather than install/use, go to
**Update** instead.

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
an installer. Read `PROJECT/RESUME-LEAD.md` and follow it: read durable
state (`CLAUDE.md`, `PROJECT-LEAD.md`, phase docs, git log, recent
eval/review reports), give the under-200-word briefing, flag anomalies,
and hand the user the floor. Do not start forward work without their input.

## Update

The user wants newer methodology files pulled into an already-set-up
project. Only refresh the **safe surface** — never blow away their
customizations:

**Safe to overwrite** from `$REPO/template/`:
- `.claude/agents/*.md` — but if an agent file was domain-customized (its
  "Bugs to hunt for" / report template differs from the template),
  show a diff and ask before overwriting.
- `docs/*/README.md` and `docs/*/_*-template.md` — the doc scaffolding.
- `PROJECT-LEAD.md`, `RESUME-LEAD.md` — role docs.

**Never overwrite blindly:**
- `CLAUDE.md` — it's customized. Show the user a diff of any *structural*
  template changes (new sections, changed rules) and let them merge.
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
