# Day-1 Bootstrap — new project

Paste the section below into a fresh Claude Code session opened in this
directory. It walks Claude through interviewing you and customizing the
template before any production code is written.

---

I'm starting a new long-running project using the **phased-agents**
methodology (see `METHODOLOGY.md` from the source repo, or the template
files already present in this directory). I want you to act as the project
lead. Walk me through the day-1 setup.

## Step 1: read what's here

- Read `CLAUDE.md` (it has `{{PLACEHOLDERS}}` you'll fill in).
- Read every file under `.claude/agents/`.
- Skim `docs/phases/README.md`, `docs/decisions/README.md`,
  `docs/research/README.md`.

## Step 2: interview me

Ask me these questions, one at a time, waiting for my answer before the next:

1. **Project name and one-paragraph goal.** What are we building, and what
   does success look like at the end of the project?
2. **Domain category.** Pick one or describe a hybrid:
   algorithms / ML, API / backend, UI / web, mobile, infrastructure / DevOps,
   data engineering, embedded, other. (This calibrates the reviewer's
   failure-mode list.)
3. **Tech stack.** Primary language, framework, package manager, runtime.
4. **Definition of "eval".** What does a passing acceptance gate look like
   for this project? Examples by domain:
   - Algorithms: convergence test, oracle comparison
   - API: contract test, load test, fuzz test
   - UI: visual regression, Lighthouse score, manual user-flow walkthrough
   - Mobile: device matrix tests, instrument profiling
   - Infrastructure: chaos test, dry-run, staging deploy
5. **Expected timeline and rough phase count.** Months? Years? 5 phases?
   12? It's fine to be vague — this just shapes the initial phase plan.
6. **Anything that already exists.** Empty repo, prior research notes,
   designs, partial code? (For an empty repo, say "nothing".)

## Step 3: customize the template

Based on my answers:

1. **Rewrite `CLAUDE.md`'s placeholders** in place. Goal, current phase
   (Phase 0), Why this structure (with the domain-specific silent-bug
   note), Repo layout, Conventions, How to run, Critical reminders.
2. **Update `.claude/agents/reviewer.md`**'s "Bugs to hunt for" section
   to be domain-specific — replace the generic list with the failure
   modes that actually matter for this project.
3. **Update `.claude/agents/eval-runner.md`**'s report template if my
   eval semantics warrant domain-specific report sections (e.g., visual
   diffs for UI, latency histograms for APIs).
4. **Draft `docs/phases/README.md`** with my actual phase plan — list
   the phases, their gates, and the sequence.
5. **Draft `docs/phases/phase-0-foundations.md`** with a small,
   low-risk Phase 0. Phase 0 should be a proof-of-concept that
   exercises the pipeline end-to-end on something simple, not the
   real work. (Examples: "implement the simplest version of the core
   thing", "stand up the engine with a hello-world test", "deploy a
   single endpoint with auth and verify it works.")
6. **Draft an initial `docs/decisions/0001-stack-and-conventions.md`**
   ADR documenting the tech-stack decisions from question 3.

## Step 4: present and confirm

Show me everything you wrote. Ask me to confirm or adjust each piece.
Iterate until I'm happy.

## Step 5: kick off Phase 0

Once I confirm:

1. Suggest the first commit:
   `chore: scaffold from phased-agents template; customize for {{project}}`
2. Tell me the prompt to use for the next session — the one that will
   start Phase 0's pipeline (researcher → implementer → reviewer →
   eval-runner). Make it self-contained so I can paste it into a fresh
   Claude session.
3. Stop. Don't write production code yet.

## Important rules during this bootstrap

- **No code under `src/` or wherever the project's source will live yet.**
  This is setup only.
- **Don't fabricate technical details.** If I haven't told you something,
  ask me. Don't invent.
- **Don't propose a 50-phase plan.** Aim for a manageable number (5–10
  phases at the top level; sub-phases come later as needed).
- **Phase 0 should be small.** A few hours of work, not weeks.
- **Be honest about what's domain-specific vs generic.** Where you're
  customizing the reviewer's bug list, explain why each item is on the
  list for my domain.
