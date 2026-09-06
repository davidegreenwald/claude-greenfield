# The audit rubric — one spine for both modes

This is the single checklist that drives a `greenfield` run. It maps each success factor to
the artifact(s) that secure it, how to detect its state, and the action to take.

- **Init:** every factor starts **absent**; the rubric is the build to-do.
- **Audit:** profile the repo first, then score each factor **present / partial / absent**;
  present the gap report, ask which gaps to fill, then generate only the approved partial/absent
  artifacts; never clobber what is present.

Score honestly:
- **Present** — an artifact secures the factor and is wired in (e.g. the gate runs *and* a hook
  enforces it). Reuse it; do not regenerate.
- **Partial** — the capability exists but is not wired, is incomplete, or lives in the wrong
  form (e.g. a gate script with no hook; decisions in prose, not ADRs). Complete or wrap it.
- **Absent** — nothing secures the factor. Generate it.

Detection commands below are starting points; adapt to the stack.

---

## Profile the target (audit)

```
git -C <target> rev-list --count HEAD  # history present?
git -C <target> remote -v              # remote → PR posture; none → local posture
```
Read the manifest(s) for stack + scripts: `package.json` / `pyproject.toml` / `Package.swift`
/ `Cargo.toml` / `go.mod`. Read existing build/test/lint config. Note the source layout.

---

## Factor-by-factor rubric

| # | Factor | Securing artifact(s) | Present when… | Action if partial/absent |
|---|--------|----------------------|----------------|--------------------------|
| 1 | Decision-completeness | `tickets/TEMPLATE.md` + `ticket-planner` agent | a decision-complete template exists, carries `## Acceptance criteria` (numbered, each naming a red oracle), and is gated by an agent that returns a gap for a criterion with no oracle | author template (drop irrelevant rows) + the agent |
| 2 | Evidence over recall | Research step + proven/novel field + `fact-checker` | tickets force citations; an agent verifies them (`fact-checker`, or the adversary's duties 1–2); done is a rerunnable command + its output in the ticket's Evidence | add the field to the template + the agent; add the Evidence block |
| 3 | Specify-first | Verification section + `tests/**` rule | tests are written before code AND confirmed red-first by rule; determinism encoded | add the rule + the template's Verification section (red-before-green) |
| 4 | Externalized memory | `tickets/` + `WORK-ITEMS.md` | a ticket dir and a live registry exist | create both; migrate any scratch/backlog notes into tickets |
| 5 | Decision provenance | `docs/adr/` + ADR template | decisions are ADRs, not inline prose | create `docs/adr/`; migrate inline decisions into ADRs |
| 6 | Fast deterministic gate | a `verify` command + commit hook | one command runs lint+types+tests+arch AND a hook enforces it | reuse existing script as the gate; add the hook |
| 7 | Executable architecture | fitness-function tool, or rule+reviewer | a build-failing check enforces the boundary | add the tool, or a path rule + a reviewer; seed an ADR |
| 8 | Independent review | `.claude/agents/*` roster | reviewers wired to `/work` phases, ≥1 that *executes* (runs tests/repro), not diff-reading alone; a `ticket-adversary` seat runs pre-code; every reviewer is **read-only by tool grant** (`Read, Grep, Glob` [+`Bash`], no `Write`/`Edit`, no `isolation: worktree`); the diff is frozen (committed) before reviewers spawn | author the roster (factor-appropriate lenses) incl. an executing reviewer and the adversary; strip any reviewer `isolation: worktree` / `Write` grant |
| 9 | Isolated / green main | `/work` Phase 0 + merge gate | work is branch/worktree-isolated; merge requires green | encode in the `/work` skill per git posture |
| 10 | Progressive disclosure & context scoping | layered CLAUDE.md + `.claude/rules/*` + correct skill/agent placement | CLAUDE.md holds only durable cross-agent standards; constraints are path-scoped; orchestrator-only logic is in the `/work` skill; subagent-critical constraints are in agent defs / listed skills; depended-on skills+agents are repo-scoped | split oversized CLAUDE.md; run the **context-scoping sub-audit** below and move each mis-scoped block to its correct layer |
| 11 | Human-in-the-loop | `/work` Phase 1 approach pass + Phase 8 trigger list | the Component-ticket approach pass is the one planned halt; a defined surprise list governs the rest | encode the approach pass (Phase 1) and the trigger list (Phase 8) in the `/work` skill |
| 12 | Effort tiering | per-agent `model:` frontmatter | judgment agents use a stronger tier than mechanical | set `model:` per agent role |
| 13 | Self-improving harness | `/work` retrospective phase | a retrospective runs every cycle | add the phase to the `/work` skill |
| 14 | Test-strength | mutation/revert check (separate from fast gate) | the mutation/revert check is configured on changed files and wired into `/work` Phase 5b (a run is not required at init — there are no changed files); kept outside the commit gate | add the tool per stack + a pre-merge/scheduled step |
| 15 | Defect intake | `Bug` ticket type + defect rule | bugs enter with a named consequence + reproduction + a red-first regression test | add the `Bug` type (or a defect section) + the rule |
| 16 | Runtime proof | `.claude/rules/evidence.md` (always-on) + a runtime harness (`<harness dir>/`, `./exemplars/runtime-harness.md`) or a recorded deferral in `adr-002` + the `/work` Phase 8 landing run | the evidence rule is installed and mirrored into every seat via the format's Evidence discipline block; a harness exists that drives the real system and passes its own `--negative-control` and `--twice` proofs, OR `adr-002` records that no behavioral ticket has run yet and names the scripted-runbook fallback; `/work` Phase 8 runs the harness before a behavior change lands | install the rule; seed the harness exemplar + the stack's driver row; record the deferral in `adr-002`; add the Phase 8 runtime-run instruction to the `/work` skill |
| 17 | Performance measure *(present-when-applicable)* | benchmark + threshold/baseline (separate step) + a documented profiler | a perf-sensitive surface has a repeatable benchmark that fails the build on regression, kept out of the fast gate — OR a justified N/A in `adr-001` | identify the perf surface; add the stack's benchmark + threshold (`./ecosystem-profiles.md`) as a pre-merge/scheduled step + document the profiler; if none, record the N/A |

Factor 17 scores **N/A** only when the project genuinely has no perf-sensitive surface and that is
recorded in `adr-001`; otherwise it is present/partial/absent like the rest.

---

## Context-scoping sub-audit (factor 10)

Claude Code inherits the **whole CLAUDE.md hierarchy** (root + nested + user-global + managed
policy) into every subagent, but does **not** inherit `.claude/rules/`, the orchestrator's
conversation, or any skill unless the subagent lists it in `skills:` (source: Claude Code docs —
`sub-agents.md`, `memory.md`, `skills.md`). So *where* an instruction lives decides which agents
pay for it and which agents actually get it. Score these three checks; each maps to a placement
fix in `./exemplars/instruction-placement.md`. Do not clobber content — **move** it to the
correct layer.

**10a — CLAUDE.md inheritance hygiene.** Every subagent inherits CLAUDE.md, so it must hold only
durable standards *every* agent benefits from.
```
wc -l CLAUDE.md **/CLAUDE.md 2>/dev/null   # size; a wall (>~200 lines) is a smell
```
Read CLAUDE.md and classify each block: durable cross-agent standard (evidence hygiene, commit
convention, architecture invariant → **keep**) vs orchestrator-only coordination (phase order,
when-to-halt, roster wiring → **move to the `/work` skill**) vs task-specific constraint (a
per-layer or per-file rule → **move to a path-scoped rule or the owning agent**).
- **Present:** CLAUDE.md is only durable standards; nothing orchestrator-only or task-specific.
- **Partial/absent:** orchestrator-only or task-specific content sits in CLAUDE.md, inherited by
  every subagent as noise → move it to the layer above.

**10b — subagent-critical constraints actually reach the subagent.** For each constraint a
reviewer or worker subagent must honor, confirm it lives in that agent's definition or in a skill
named in its `skills:` list — **not** only in a path-scoped rule (subagents are not known to load
rules) and **not** only in the orchestrator's conversation.
- **Present:** every must-honor constraint is in the agent's own loaded context.
- **Partial/absent:** a constraint lives only where the subagent never loads it → inline it into
  the agent def, or add the carrying skill to the agent's `skills:`.

**10c — instruction sizing & explicit skill/agent scoping.**
- Each subagent's preloaded context (its system prompt + listed skills) is scoped to its one task
  and matched to its model/effort (factor 12) — not a general dump.
- Every skill and agent the workflow depends on is repo-scoped or a documented dependency, not a
  silent reliance on a personal `~/.claude/`.
```
ls .claude/skills .claude/agents 2>/dev/null   # what the repo actually ships
```
Cross-check every agent named in the `/work` skill and every entry in each agent's `skills:`
list against what the repo ships. Any name that resolves only from `~/.claude/` (global) is a
reproducibility gap — **vendor it into the repo** (`.claude/skills` / `.claude/agents`) or record
it as an explicit external dependency the project requires.
- **Git workflow is the standing must-fix instance.** The `/work` skill must carry the repo's own git
  instruction (worktree/commit/merge/PR) and must **not** invoke a global `git-workflow` (or any global
  git) skill. Flag any such reliance as **partial** and inline the git steps into the repo's `/work`.
  This is per-repo by design.
- **Present:** subagent contexts are task-sized, and every dependency is repo-scoped or declared.
- **Partial/absent:** a bloated general context, or a workflow that silently needs a global skill
  → trim/scope, and vendor or document the dependency.

---

## The artifact tree a complete project has

Init creates this whole tree; audit fills the gaps. Exact contents come from the exemplars,
adapted to the stack.

```
<project>/
├── CLAUDE.md                      # factor 10 — durable cross-agent standards ONLY (inherited by every subagent); points to rules
├── WORK-ITEMS.md                  # factor 4 — live registry (ID | Title | Type | Status | Phase | Ticket)
├── tickets/
│   └── TEMPLATE.md                # factors 1,2,3,15,16 — decision-complete template (acceptance criteria bound to red oracles, rerunnable Evidence, Bug variant)
├── docs/
│   └── adr/
│       ├── TEMPLATE.md                    # factor 5 — the ADR template (`./exemplars/adr-template.md`)
│       ├── adr-001-architecture.md        # factor 5,7 — the architecture + boundary
│       ├── adr-002-quality-gates.md       # factor 5,6 — the gate definition (+ the runtime-harness deferral, factor 16)
│       └── adr-003-definition-of-done.md  # factor 5 — what "done" means per dimension (criteria, evidence, gate, reviewers, bugs)
├── .claude/
│   ├── skills/work/SKILL.md       # factors 8,9,11,13,14,15,16 — the execution pipeline (orchestrator-only; NOT inherited)
│   ├── agents/                    # factors 8,12 — the reviewer roster
│   │   ├── ticket-adversary.md    # factor 8 — breaks the ticket before it is built (Phase 2)
│   │   └── *.md
│   ├── findings-bar.md            # factor 8 — the shared finding format every reviewer reads
│   ├── rules/                     # factors 2,3,10,14,15,16 — the five always-on rules (`./exemplars/rules.md`) + path-scoped
│   │   ├── shell-hygiene.md       # always-on — verify state after a state-changing command; no 2>/dev/null on a query
│   │   ├── evidence.md            # factors 2,16 always-on — drive, don't cite; the three rules + the evidence tags
│   │   ├── defect.md              # factor 15 always-on — a Bug is not started without consequence + repro + red test
│   │   ├── coverage-derives-its-domain.md  # always-on — an exhaustiveness check derives its set from the producer, never a literal
│   │   ├── symbol-deletion.md     # always-on — an export rename/delete greps the whole harness surface both ways
│   │   └── *.md                   # path-scoped (architecture-derived)
│   └── settings.json              # hook + permission config
├── <gate command>                 # factor 6 — Makefile target / package script / etc.
├── <arch-enforcement config>      # factor 7 — import-linter / dependency-cruiser / lint rule
├── <mutation config>              # factor 14 — stryker.conf / mutmut / cargo-mutants (separate step)
├── <harness dir>/                 # factor 16 — qa/ or e2e/: drives the real system (built at /work discretion; else adr-002 records the deferral)
├── <benchmark + threshold>        # factor 17 — perf measure where applicable (separate step; profiler documented)
└── <hooks>                        # factor 6 — pre-commit + Stop + format-on-write
```

---

## Closing a run

A run is done when the rubric scores **factors 1–16 present** and **factor 17
present-or-justified-N/A**, the context-scoping sub-audit (10a–10c) passes, the gate passes green
on the seeded project, the commit hook fires, and the `/work` skill's agent references resolve.
For an audit, also confirm `git status` shows only the intended additions. Report the present /
created / reused breakdown and recommend the first real ticket.
