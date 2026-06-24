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

## Detect the mode and profile (audit)

```
ls -la <target>                        # empty-ish → init; populated → audit
git -C <target> rev-list --count HEAD  # history present?
git -C <target> remote -v              # remote → PR posture; none → local posture
```
Read the manifest(s) for stack + scripts: `package.json` / `pyproject.toml` / `Package.swift`
/ `Cargo.toml` / `go.mod`. Read existing build/test/lint config. Note the source layout.

---

## Factor-by-factor rubric

| # | Factor | Securing artifact(s) | Present when… | Action if partial/absent |
|---|--------|----------------------|----------------|--------------------------|
| 1 | Decision-completeness | `tickets/TEMPLATE.md` + `ticket-planner` agent | a decision-complete template exists and is gated by an agent | author template (drop irrelevant rows) + the agent |
| 2 | Evidence over recall | Research step + proven/novel field + `fact-checker` | tickets force citations; an agent verifies them | add the field to the template + the agent |
| 3 | Specify-first | Verification section + `tests/**` rule | tests are written before code by rule; determinism encoded | add the rule + the template's Verification section |
| 4 | Externalized memory | `tickets/` + `WORK-ITEMS.md` | a ticket dir and a live registry exist | create both; migrate any scratch/backlog notes into tickets |
| 5 | Decision provenance | `docs/adr/` + ADR template | decisions are ADRs, not inline prose | create `docs/adr/`; migrate inline decisions into ADRs |
| 6 | Fast deterministic gate | a `verify` command + commit hook | one command runs lint+types+tests+arch AND a hook enforces it | reuse existing script as the gate; add the hook |
| 7 | Executable architecture | fitness-function tool, or rule+reviewer | a build-failing check enforces the boundary | add the tool, or a path rule + a reviewer; seed an ADR |
| 8 | Independent review | `.claude/agents/*` roster | read-only reviewers wired to `/work` phases | author the roster (factor-appropriate lenses) |
| 9 | Isolated / green main | `/work` Phase 0 + merge gate | work is branch/worktree-isolated; merge requires green | encode in the `/work` skill per git posture |
| 10 | Progressive disclosure | layered CLAUDE.md + `.claude/rules/*` | a root CLAUDE.md + path-scoped rules; not one monolith | split oversized CLAUDE.md into rules; add per-module CLAUDE.md if warranted |
| 11 | Human-in-the-loop | `/work` Phase 1 approach pass + Phase 8 trigger list | the Component-ticket approach pass is the one planned halt; a defined surprise list governs the rest | encode the approach pass (Phase 1) and the trigger list (Phase 8) in the `/work` skill |
| 12 | Effort tiering | per-agent `model:` frontmatter | judgment agents use a stronger tier than mechanical | set `model:` per agent role |
| 13 | Self-improving harness | `/work` retrospective phase | a retrospective runs every cycle | add the phase to the `/work` skill |

---

## The artifact tree a complete project has

Init creates this whole tree; audit fills the gaps. Exact contents come from the exemplars,
adapted to the stack.

```
<project>/
├── CLAUDE.md                      # factor 10 — project-wide context; points to rules
├── WORK-ITEMS.md                  # factor 4 — live registry (ID | Title | Status | Phase | Ticket)
├── tickets/
│   └── TEMPLATE.md                # factors 1,2,3 — decision-complete template
├── docs/
│   └── adr/
│       ├── adr-001-architecture.md    # factor 5,7 — the architecture + boundary
│       └── adr-002-quality-gates.md   # factor 5,6 — the gate definition
├── .claude/
│   ├── skills/work/SKILL.md       # factors 8,9,11,13 — the execution pipeline
│   ├── agents/                    # factors 8,12 — the reviewer roster
│   │   └── *.md
│   ├── rules/                     # factors 3,10 — path-scoped + always-on
│   │   ├── shell-hygiene.md       # always-on (portable)
│   │   └── *.md                   # path-scoped (architecture-derived)
│   └── settings.json              # hook + permission config
├── <gate command>                 # factor 6 — Makefile target / package script / etc.
├── <arch-enforcement config>      # factor 7 — import-linter / dependency-cruiser / lint rule
└── <hooks>                        # factor 6 — pre-commit + Stop + format-on-write
```

---

## Closing a run

A run is done when the rubric scores **all 13 present**, the gate passes green on the seeded
project, the commit hook fires, and the `/work` skill's agent references resolve. For an
audit, also confirm `git status` shows only the intended additions. Report the present /
created / reused breakdown and recommend the first real ticket.
