---
name: greenfield
description: "Build and customize a production-ready agent workflow from scratch, or audit and upgrade your existing projects."
argument-hint: "[optional: project path]"
disable-model-invocation: true
last_updated: 2026-06-23
---

# greenfield — stand up the agentic-project success harness

Install the planning and guardrail harness that keeps an agentic coding project correct
and resumable, into a target repo — empty (**init**) or existing (**audit**). Lead with
the principles; derive the concrete artifacts at runtime to fit the project's stack.

**On invocation, go straight to Process step 1:** print the fixed summary below verbatim as
plain text, then ask init vs audit and begin the interview. Do not inspect the working
directory to guess the mode — ask the user. Running `/greenfield` is the instruction — do
not wait for further direction.

Premise under everything: **agent-written code is first-class**. It meets the same bar
as hand-written code, so the project needs guardrails that keep agent work correct,
verifiable, resumable, and consistent — not a lighter standard because "an agent wrote
it."

Two rules govern this skill:
- **Full kit, adaptive form.** Every one of the 13 success factors below is secured by
  *some* artifact in every project. Each artifact's *form* (the exact ticket fields,
  phase list, reviewer roster, gate command) is decided at runtime from the interview —
  never stamped from a fixed template.
- **Adapt, don't stamp; stay self-contained.** The bundled exemplars in
  `./supplements/exemplars/` are reference realizations to adapt to the project's stack,
  not templates to copy verbatim. This skill carries every format it needs and depends on
  no other skill.

## The 13 success factors (the spine)

These hold across stacks — a data pipeline, a mobile app, an editor plugin — and only their
form varies. Each: the principle → the guardrail that secures it → the failure it prevents.
Depth and per-stack variation live in `./supplements/success-factors.md`.

### Planning — resolved before code is written
1. **Decision-completeness** — the ticket resolves every decision (files, schema, flags,
   blast radius); no "TBD" reaches the keyboard. → ticket template + a decision-checker
   agent. Prevents the agent improvising architecture mid-stream.
2. **Evidence over recall** — claims cite a primary source or `file:line`; counts come
   from tools; each change is `proven — matches <file:line>` or `novel — validated via
   <url>`. → research step + citation field + a fact-checker agent. Prevents
   hallucinated specifics shipping as fact.
3. **Specify-first** — `scenario → expected` written as test stubs before implementation.
   → a Verification ticket section + a testing rule. Prevents the agent rationalizing
   whatever it produced as correct.
4. **Externalized memory** — tickets, a work-items registry, and ADRs hold project
   state, not the conversation. → `tickets/` + `WORK-ITEMS.md` + `docs/adr/`. Survives
   context loss, compaction, and session boundaries.
5. **Decision provenance** — ADRs record the *why* plus alternatives. → `docs/adr/` +
   ADR template. Stops future agents relitigating or silently breaking settled
   invariants.

### Guardrails — secure work while it is built
6. **Fast deterministic gate = "done"** — one quick command (lint + types + tests +
   architecture) is the Definition of Done, hook-enforced, not left to agent goodwill.
   → a `verify` gate + a commit-time hook. The faster the gate, the more often it runs.
7. **Executable architecture** — layering/dependency intent encoded as fitness functions
   that fail the build. → an arch-enforcement tool (or, where none exists, a rule + a
   reviewer). Docs drift; checked contracts do not.
8. **Independent adversarial review** — read-only reviewer subagents with distinct lenses
   return structured verdicts before merge, run in parallel to protect the orchestrator's
   context. → a reviewer roster wired to `/work` phases. A differently-prompted second
   set of eyes catches what the author's context cannot.
9. **Isolated work, always-green main** — branch/worktree per unit; lands only when
   green, via clean fast-forward or PR. → `/work` Phase 0 + merge gate. Keeps main
   releasable and changes reversible.
10. **Progressive disclosure of conventions** — CLAUDE.md plus path-scoped
    `.claude/rules/*.md` deliver the right constraint at the moment it applies. → layered
    CLAUDE.md + rules with `paths:` globs. A flat instruction wall decays; scoped rules
    stay salient.
11. **Calibrated human-in-the-loop** — proceed autonomously on resolved decisions; take the
    one planned halt (the Component-ticket approach pass) and halt otherwise only on a defined
    surprise list. → `/work` Phase 1 approach pass (planned) + Phase 8 stop-and-ask
    (surprises). Too many asks waste the human; too few risk unapproved drift.
12. **Effort-proportional tiering** — strongest model for judgment review, cheaper for
    mechanical checks. → per-agent `model:` frontmatter. Match spend to the judgment
    required.

### Meta
13. **Self-improving harness** — each cycle runs a retrospective that lands fixes to the
    rules, skills, and gates. → `/work` retrospective phase. The harness is a living
    system; surfaced friction compounds when fixed.

## The non-negotiable kit

Every factor maps to an artifact; the form varies. The interview fills the right column.

| # | Factor | Securing artifact | Varies by stack as… |
|---|--------|-------------------|---------------------|
| 1 | Decision-completeness | `tickets/TEMPLATE.md` + `ticket-planner` agent | ticket sections drop domain-specific rows |
| 2 | Evidence over recall | Research + proven/novel field + `fact-checker` | citation targets (docs/repos) |
| 3 | Specify-first | Verification section + testing rule | test framework + assertion style |
| 4 | Externalized memory | `tickets/` + `WORK-ITEMS.md` + `docs/adr/` | none (portable) |
| 5 | Decision provenance | `docs/adr/` + ADR template | none (portable) |
| 6 | Fast deterministic gate | a single `verify` command + commit hook | `make verify` / `npm run check` / `swift test`… |
| 7 | Executable architecture | fitness-function tool or rule+reviewer | import-linter / dependency-cruiser / SPM+lint |
| 8 | Independent review | `.claude/agents/*` roster | which lenses; domain reviewers added |
| 9 | Isolated / green main | `/work` Phase 0 + merge gate | worktree+ff-only (local) / branch+PR (remote) |
| 10 | Progressive disclosure | layered CLAUDE.md + `.claude/rules/*` | which path globs |
| 11 | Human-in-the-loop | `/work` Phase 1 approach pass + Phase 8 triggers | merge approval vs auto-merge |
| 12 | Effort tiering | per-agent `model:` | available model tiers |
| 13 | Self-improving harness | `/work` retrospective phase | none (portable) |

Install the **refined** realizations as the default: parallel review, hook-enforced gate,
auto-merge on green, retrospective + next-ticket phases, path-scoped rules, per-module
CLAUDE.md. These are the strongest forms of each factor; scale them down for a small project
by reducing sizing (fewer ADRs, a trimmed roster), never by dropping a factor.

Add one cadence the per-ticket loop does not cover: a **phase-gate audit**. At the end of each
roadmap phase — not each ticket — seed a ticket that runs a deep, multi-agent audit (an
"ultracode" pass: many reviewer subagents fanned out in parallel) across architecture, code
correctness, security, and performance. The per-ticket gate and reviewers each see one diff at a
time, so cross-cutting drift slips through; the phase boundary is the cheapest place to catch it
before it compounds. Fold the audit's findings back in as the next phase's tickets.

## Process

Run these steps. Load supplements as each step needs them.

### 1. Open and choose mode
Your first output on invocation — before the mode question and before anything else — is this
summary, so the user knows what they are getting. Emit it verbatim and identical every time,
as plain text (not a quote, blockquote, or code block), with the bullet list intact:

greenfield installs a planning-and-guardrail harness so agent-written code stays correct and
resumable. It secures all 13 success factors with concrete artifacts:

- **Decision-complete tickets** — every decision resolved before code is written, no "TBD".
- **A fast hook-enforced quality gate** — one `verify` command (lint + types + tests +
  architecture) is the Definition of Done, run at commit time.
- **Independent reviewer subagents** — read-only reviewers with distinct lenses return
  structured verdicts before merge.
- **ADRs** — decisions recorded with their alternatives and the *why*.
- **Path-scoped rules** — layered CLAUDE.md plus `.claude/rules/*` deliver the right
  constraint where it applies.
- **A `/work` command** — ties tickets, the gate, reviewers, and a retrospective together.

I'll ask a few questions, then either scaffold a new project (**init**) or audit this repo
and install what's missing (**audit**).

Then ask the mode outright with one AskUserQuestion — **init** (scaffold a new project) vs
**audit** (review this repo and install what's missing). Do not inspect the working
directory or guess the mode from a path argument; ask the user. The target path is not
resolved here — it is the interview's first question (Batch A), so a path passed as
`/greenfield <path>` only seeds that question's default, it never preselects the mode.

**Audit needs the path before profiling.** When the user picks audit, take the target path
as the interview's first question, then profile the stack there (manifests, build/test/lint
config, git remote, source layout) so the remaining questions confirm inferences instead of
asking them.

### 2. Research the stack and domain
Ask the interview's Batch A first (target path and stack via AskUserQuestion, purpose as a
free-text prompt) so you know the project shape to research. That shape — "a Rust CLI", "a SwiftUI to-do app", "a
FastAPI service" — is usually one you have not set up before, and recall is not enough to
constrain it well. Then, before the architecture questions, fan out a round of research and let
primary sources drive the recommendations. Run several searches in parallel (and fetch the
sources behind the hits):

- **Idiomatic project structure** — the language/framework's own recommended layout and the
  prevailing community convention.
- **Reference architectures for the domain** — the named styles that fit (e.g. hexagonal /
  ports-and-adapters, MVVM, a lib-core + thin-bin Cargo workspace) and when each is chosen.
- **Gate + arch-enforcement tooling** (factors 6, 7) — the current formatter, linter,
  type/compile check, test runner, and any dependency/layer linter; start from
  `./supplements/ecosystem-profiles.md` and extend or correct it from what you find.

Prefer primary sources; corroborate a secondary claim with a second source. Synthesize into
2-4 candidate architectures, each a named style with its tradeoff and its source — this is the
input the interview's architecture question presents. Shorten this only when you can already
constrain the stack from primary knowledge and the user confirms.

### 3. Interview
Resolve the runtime details with `./supplements/interview.md`. Infer what you can (in audit,
from the profile); ask only the rest, and ask every question through AskUserQuestion — never a
free-form prose prompt — except purpose, which is a single free-text question. Open-ended
answers like path and stack still go through AskUserQuestion: offer inferred candidate options
and let the user type a custom value in the "Other" field. It resolves: identity, stack, architecture + the boundary to enforce, gate
command, git posture, reviewer roster + model tiers, rules, and scale knobs. Present each
technical choice as researched options: mark
the staff-engineer pick `(recommended)` with a one-line why and its source, and **name the
architecture style and cite the sources that will constrain the project** so the user can
accept it or offer their own. Reach decision-completeness before generating — no "TBD".

### 4. Generate (init) / Audit-and-install (audit)
Author each artifact at runtime, adapted to the project. Use `./supplements/exemplars/*` as
references to adapt, not stamp — each carries the format it needs, so no other skill is required.
First **emit the would-generate manifest** (the files and their one-line purpose) and get the
user's go-ahead; write only what is approved. Declining the manifest writes nothing — that is
the no-write preview, in either mode.
- **Init:** walk `./supplements/checklist.md` top to bottom; for each factor, create its
  securing artifact. Seed `adr-001` with the architecture decision — its named style, the
  sources behind it, and the boundary the arch-enforcement guards.
- **Audit:** score each factor present / partial / absent against the checklist and present the
  gap report. Then ask the user which gaps to fill before writing anything. Add only the
  approved partial/absent artifacts; **never clobber** what is present; **reuse** existing gates
  and config (an existing `check`/`test` script becomes the gate). Migrate inline decisions into
  ADRs and backlog/scratch notes into tickets.

For factors 6 and 7, map the gate and the boundary check to the stack with
`./supplements/ecosystem-profiles.md` (Python, Node/TS, Swift, Rust, Go, and a generic fallback).

### 5. Verify
- Run the new `verify` gate once — it must pass green on the seeded project.
- Confirm the commit hook fires and the `/work` skill's agent references all resolve.
- Walk `./supplements/checklist.md` — every factor present.
- Report what was created vs reused (audit), and the next action (write the first real
  ticket, or `/work` it).

## Modes

- **Init** — create the full tree from the checklist; initialize git; seed `WORK-ITEMS.md`
  empty and write `tickets/TEMPLATE.md`, but do **not** author a first ticket — recommend it as
  the next action (step 5) for the user to `/work`; write `adr-001` (architecture) and
  `adr-002` (quality gates); install hooks; prove the gate green on the empty scaffold.
- **Audit** — review-first. Score the repo against the checklist, present the present /
  partial / absent report, and ask which gaps to fill. Then generate only the approved
  absent/partial artifacts. Migrate existing inline decisions into ADRs and existing
  backlog/notes into tickets. Reuse the existing gate; wrap it with hooks. Slim an oversized
  CLAUDE.md into path-scoped rules. Confirm the working tree is otherwise untouched.

## Supplements

| If the step is… | Read |
|-----------------|------|
| Understanding/teaching the factors | `./supplements/success-factors.md` |
| Auditing a repo or building the to-do | `./supplements/checklist.md` |
| Running the interview | `./supplements/interview.md` |
| Choosing gate / arch-enforcement / test tooling | `./supplements/ecosystem-profiles.md` |
| Authoring a specific artifact | `./supplements/exemplars/<artifact>.md` |

### Factor depth → `./supplements/success-factors.md`
**What:** the 13 factors elaborated — mechanism, failure mode, per-stack form.
**When to read:** explaining a factor, or deciding how to realize one for an unusual stack.

### Audit rubric → `./supplements/checklist.md`
**What:** factor × securing artifact × present/partial/absent. The single spine for both modes.
**When to read:** every run — it is the build to-do (init) and the gap audit (audit).

### Interview → `./supplements/interview.md`
**What:** the question script; infer-vs-ask guidance; AskUserQuestion groupings; how to present
researched options with a `(recommended)` pick and cited architecture.
**When to read:** step 3, every run.

### Ecosystem profiles → `./supplements/ecosystem-profiles.md`
**What:** concrete gate / arch-enforcement / test / hook mappings for Python, Node/TS,
Swift/iOS, Rust, Go, and a generic fallback.
**When to read:** resolving factors 6 and 7 for the project's stack.

### Exemplars → `./supplements/exemplars/`
**What:** "adapt, do not copy" reference realizations — `ticket-template.md`,
`work-items.md`, `work-skill.md`, `adr-template.md`, `agent-roster.md`, `rules.md`,
`hooks.md`.
**When to read:** authoring that artifact in step 4.

## Self-test

A run is correct when: the target's `verify` gate passes green; the commit hook is
installed and fires; every `.claude/agents/*` referenced by the generated `/work` resolves;
`./supplements/checklist.md` shows all 13 factors present; and (audit) `git status`
shows only intended additions.
