---
name: greenfield
description: "Build and customize a production-ready agent workflow from scratch, or audit and upgrade an existing repo — decision-complete tickets with success criteria bound to red oracles, a hook-enforced gate, a definition of done and of a bug, adversarial review that executes, and runtime proof against the real system."
argument-hint: "[optional: project path]"
disable-model-invocation: true
last_updated: 2026-09-05
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
- **Full kit, adaptive form.** Every one of the 17 success factors below is secured by
  *some* artifact in every project — except factor 17 (performance), whose artifact is present
  only where the project has a perf-sensitive surface, else a justified N/A. Each artifact's
  *form* (the exact ticket fields, phase list, reviewer roster, gate command) is decided at
  runtime from the interview — never stamped from a fixed template.
- **Adapt, don't stamp; stay self-contained.** The bundled exemplars in
  `./supplements/exemplars/` are reference realizations to adapt to the project's stack,
  not templates to copy verbatim. This skill carries every format it needs and depends on
  no other skill.

## The 17 success factors (the spine)

These hold across stacks — a data pipeline, a mobile app, an editor plugin — and only their
form varies. Each: the principle → the guardrail that secures it → the failure it prevents.
Depth and per-stack variation live in `./supplements/success-factors.md`.

### Planning — resolved before code is written
1. **Decision-completeness** — the ticket resolves every decision (files, schema, flags,
   blast radius); no "TBD" reaches the keyboard. → ticket template + a decision-checker
   agent. Prevents the agent improvising architecture mid-stream. **Continuation form:** the
   ticket carries numbered acceptance criteria, each bound to a named oracle — a test, or a
   runtime-harness step — that was seen red before implementation; a criterion with no oracle,
   or one already green, is a gap the planner returns.
2. **Evidence over recall** — claims cite a primary source or `file:line`; counts come
   from tools; each change is `proven — matches <file:line>`, `novel — validated via <url>`,
   or `novel — driven via <probe + output>`; recall is not a fourth tag. → research step +
   citation field + a fact-checker agent. Prevents hallucinated specifics shipping as fact.
   **Continuation form:** the proof of done is a rerunnable command + its actual output (an
   `## Evidence` block), not a citation alone — a command a human re-runs cannot be fabricated
   after the fact.
3. **Specify-first** — `scenario → expected` written as test stubs before implementation.
   → a Verification ticket section + a testing rule. Prevents the agent rationalizing
   whatever it produced as correct. **Continuation form:** confirm each oracle *fails first*
   (red) before implementing, and write it from the acceptance criterion, not the produced
   code (the oracle-overfitting guard).
4. **Externalized memory** — tickets, a work-items registry, and ADRs hold project
   state, not the conversation. → `tickets/` + `WORK-ITEMS.md` + `docs/adr/`. Survives
   context loss, compaction, and session boundaries.
5. **Decision provenance** — ADRs record the *why* plus alternatives. → `docs/adr/` +
   ADR template. Stops future agents relitigating or silently breaking settled
   invariants.

### Guardrails — secure work while it is built
6. **Fast deterministic gate** — one quick command (lint + types + tests + architecture)
   runs at commit time, hook-enforced, not left to agent goodwill. → a `verify` gate + a
   commit-time hook. The faster the gate, the more often it runs. The gate is one condition
   of done, not the whole definition (see factors 1, 2, 8, 16 and `adr-003`).
7. **Executable architecture** — layering/dependency intent encoded as fitness functions
   that fail the build. → an arch-enforcement tool (or, where none exists, a rule + a
   reviewer). Docs drift; checked contracts do not.
8. **Independent adversarial review** — read-only reviewer subagents with distinct lenses
   return structured verdicts before merge, run in parallel to protect the orchestrator's
   context. → a reviewer roster wired to `/work` phases. A differently-prompted second
   set of eyes catches what the author's context cannot. **Continuation form:** one seat tries
   to *break* the ticket before it is built — re-runs its commands, re-opens its citations,
   cross-checks Plan against Verification, drives the plan in a scratch worktree — and returns
   `BREAKS (evidence) | BREAKS (plan) | SURVIVES`; at least one post-code reviewer *executes* —
   runs the tests, reproduces the defect, refutes by running — and pastes the output. Consensus
   is not verification; execution is. Reviewers are isolated by the tool grant (`Read, Grep,
   Glob` + `Bash` for the one that executes; never `Write`/`Edit`), and the diff is frozen (a
   commit) before any reviewer spawns.
9. **Isolated work, always-green main** — branch/worktree per unit; lands only when
   green, via clean fast-forward or PR. → `/work` Phase 0 + merge gate. Keeps main
   releasable and changes reversible.
10. **Progressive disclosure & context scoping** — each instruction reaches exactly the context
    that needs it. Durable standards go in CLAUDE.md (inherited wholesale by every subagent);
    narrow constraints in path-scoped `.claude/rules/*.md`; orchestrator-only coordination in the
    `/work` skill (not inherited); task-specific constraints in the subagent that runs the task.
    → layered CLAUDE.md + rules + correct skill/agent placement, audited by the context-scoping
    sub-audit. A flat wall inherited by every agent decays and taxes the whole fleet; scoped
    instructions stay salient.
11. **Calibrated human-in-the-loop** — proceed autonomously on resolved decisions; take the
    one planned halt (the Component-ticket approach pass) and halt otherwise only on a defined
    surprise list. → `/work` Phase 1 approach pass (planned) + Phase 8 stop-and-ask
    (surprises). Too many asks waste the human; too few risk unapproved drift. **Anti-looping
    form:** a broken design (`BREAKS (plan)`) gets a step-back to 2–4 approaches and **one**
    human-authorized retry; a second break parks the ticket with its evidence — two broken
    designs mean the problem is not understood, and a third pass will not understand it either.
12. **Effort-proportional tiering** — strongest model for judgment review, cheaper for
    mechanical checks. → per-agent `model:` frontmatter. Match spend to the judgment
    required.

### Meta
13. **Self-improving harness** — each cycle runs a retrospective that lands fixes to the
    rules, skills, and gates. → `/work` retrospective phase. The harness is a living
    system; surfaced friction compounds when fixed. **Escaped-defect form:** a bug that passed
    a green gate is a harness defect — the retrospective lands the fixture, rule, gate, or
    harness step that catches the *class*, or writes the test and deletes the story.

### Continuation — securing the debugging cycle
Load-bearing once the project accumulates bugs and leans on AI-written tests. Universal in
*presence*; the form scales from a lightweight convention on a throwaway to full tooling on a
mature repo. Depth in `./supplements/continuation.md`.

14. **Test-strength** — a passing suite proves nothing unless it can fail. Every test dies on
    revert; the suite is measured by mutation on changed files. → a mutation/revert check as a
    standing gate, kept *separate* from the fast commit gate. Prevents a green suite that cannot
    fail from hiding regressions — coverage proves a line ran, not that its effect was checked.
15. **Defect intake (the definition of a bug)** — a bug enters as a typed `Bug` ticket carrying
    three fields before a fix is written: the named consequence, a reproduction, and a red
    regression test that fails first. → a `Bug` ticket type + a defect rule. Prevents ad-hoc
    fixes that leave the class undetected and recurring.
16. **Runtime proof (drive the real system)** — a behavioral claim is proven by driving the
    code, never by citing it; where the claim is about the running system — the app, the
    service, the device, the plugin host, the external API — only a run against the real thing
    settles it. If you cannot run the real thing, you are theorizing. → an always-on evidence
    rule + a runtime harness (a distinct artifact from the test suite, carrying its own can-fail
    `--negative-control` and determinism `--twice` proofs; built at `/work` discretion by the
    first ticket whose claim needs it, else a recorded deferral in `adr-002`) + a `/work` Phase 8
    step that runs it before a behavior change lands. Prevents the green-suite escape: fixtures
    written from the same model of the system as the code share its blind spot, so a suite stays
    green while the real system loses data. → `./supplements/exemplars/rules/evidence.md`,
    `./supplements/exemplars/runtime-harness.md`.

### Non-functional guardrails — project-specific (present when applicable)
The artifact is required only where the project has the surface it defends; the principle (name
the surface, decide) is universal. Scored present-or-justified-N/A, not forced.

17. **Performance measure** — where a project has a performance-sensitive surface (page render, DB
    query, hot compute path, memory), it carries a repeatable benchmark that fails the build on
    regression past a threshold/baseline, with profiling as the escalation. → a benchmark +
    threshold as a *separate* (non-fast-gate) step + a documented profiler invocation; a justified
    N/A where there is no perf surface. Kept out of the fast gate like factor 14. Prevents
    regressions landing silently and being found by users, not CI.

## The non-negotiable kit

Every factor maps to an artifact; the form varies. The interview fills the right column.

| # | Factor | Securing artifact | Varies by stack as… |
|---|--------|-------------------|---------------------|
| 1 | Decision-completeness | `tickets/TEMPLATE.md` (acceptance criteria + oracles) + `ticket-planner` agent | ticket sections drop domain-specific rows |
| 2 | Evidence over recall | Research + proven/novel/driven field + `## Evidence` block + `fact-checker` | citation targets (docs/repos) |
| 3 | Specify-first | Verification section (red oracles) + testing rule | test framework + assertion style |
| 4 | Externalized memory | `tickets/` + `WORK-ITEMS.md` + `docs/adr/` | none (portable) |
| 5 | Decision provenance | `docs/adr/` + ADR template (`adr-001` architecture, `adr-002` quality gates, `adr-003` definition of done) | none (portable) |
| 6 | Fast deterministic gate | a single `verify` command + commit hook | `make verify` / `npm run check` / `swift test`… |
| 7 | Executable architecture | fitness-function tool or rule+reviewer | import-linter / dependency-cruiser / SPM+lint |
| 8 | Independent review | `.claude/agents/*` roster incl. `ticket-adversary` (pre-code) and an executing `correctness-reviewer`; read-only by tool grant | which lenses; domain reviewers added |
| 9 | Isolated / green main | `/work` Phase 0 + merge gate | worktree+ff-only (local) / branch+PR (remote) |
| 10 | Progressive disclosure & context scoping | layered CLAUDE.md + `.claude/rules/*` + skill/agent placement | which path globs; the CLAUDE.md/skill/agent split |
| 11 | Human-in-the-loop | `/work` Phase 1 approach pass + Phase 8 triggers + the plan-break pivot protocol | merge approval vs auto-merge |
| 12 | Effort tiering | per-agent `model:` | available model tiers |
| 13 | Self-improving harness | `/work` retrospective phase + escaped-defect clause | none (portable) |
| 14 | Test-strength | mutation/revert check on changed files (separate gate) | Stryker / mutmut / cargo-mutants / muter / gremlins |
| 15 | Defect intake | `Bug` ticket type + defect rule (consequence + repro + red test) | none (portable) |
| 16 | Runtime proof | `.claude/rules/evidence.md` (always-on) + a runtime harness (`qa/`/`e2e/`) or a recorded deferral in `adr-002` + the `/work` Phase 8 runtime step | which real system is driven and how (`./supplements/ecosystem-profiles.md` § runtime drivers) |
| 17 | Performance measure *(present-when-applicable)* | benchmark + threshold (separate step) + profiler on escalation | Lighthouse/k6 · pgbench · pytest-benchmark · Vitest bench · XCTest measure · Criterion · testing.B+benchstat; N/A if no perf surface |

Install the **refined** realizations as the default: parallel review, hook-enforced gate,
auto-merge on green, retrospective + next-ticket phases, path-scoped rules, per-module
CLAUDE.md. These are the strongest forms of each factor; scale them down for a small project
by reducing sizing (fewer ADRs, a trimmed roster), never by dropping a factor — except factor 17,
which is present only where the project has a perf-sensitive surface (else a justified N/A).

On a project with a phased roadmap, add one cadence the per-ticket loop does not cover: a
**phase-gate audit**. At the end of each roadmap phase — not each ticket — seed a ticket that runs a deep, multi-agent audit (an
"ultracode" pass: many reviewer subagents fanned out in parallel) across architecture, code
correctness, security, and performance, with every finding adversarially verified. The per-ticket
gate and reviewers each see one diff at a time, so cross-cutting drift slips through; the phase
boundary is the cheapest place to catch it before it compounds. Fold the audit's findings back in
as the next phase's tickets. Template: `./supplements/exemplars/TEMPLATE-phase-audit.md`.

## Process

Run these steps. Load supplements as each step needs them.

### 1. Open and choose mode
Your first output on invocation — before the mode question and before anything else — is this
summary, so the user knows what they are getting. Emit it verbatim and identical every time,
as plain text (not a quote, blockquote, or code block), with the bullet list intact:

greenfield installs a planning-and-guardrail harness so agent-written code stays correct and
resumable. It secures all 17 success factors with concrete artifacts:

- **Decision-complete tickets** — every decision resolved before code is written, no "TBD".
- **Success criteria bound to red oracles** — numbered acceptance criteria per ticket, each bound
  to a named test or runtime step seen failing before the work began.
- **A fast hook-enforced quality gate** — one `verify` command (lint + types + tests +
  architecture) runs at commit time.
- **A definition of done** — a ticket closes on evidence (a rerunnable command and its output
  per criterion, a green gate, recorded reviewer verdicts), never on the agent's say-so; a broken
  design gets one retry, then the ticket parks instead of looping.
- **A definition of a bug** — a `Bug` ticket carries its consequence, a reproduction, and a red
  regression test before any fix is written.
- **Runtime proof** — a behavioral claim is proven by driving the real system, never by citing
  code: an always-on evidence rule, a runtime-harness exemplar with its own can-fail and
  determinism proofs, and a `/work` step that runs it before a behavior change lands.
- **Independent adversarial review that executes** — read-only reviewers with distinct lenses:
  one tries to break the ticket before it is built, one runs the tests on the diff.
- **Test-strength** — a mutation or revert check, kept out of the fast gate, proves the suite
  can go red.
- **ADRs** — decisions recorded with their alternatives and the *why*, including what "done"
  means per dimension.
- **Path-scoped rules and context scoping** — layered CLAUDE.md plus `.claude/rules/*` deliver
  each constraint only to the agents that need it.
- **A performance measure where it applies** — a repeatable benchmark with a threshold that
  fails the build on regression, or a justified N/A.
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
free-text prompt) so you know the project shape to research. That shape — "a Rust CLI", "a
SwiftUI to-do app", "a FastAPI service" — is usually one you have not set up before, and recall is
not enough to constrain it well. Then, before the architecture questions, fan out a round of
research and let primary sources drive the recommendations. Run several searches in parallel (and
fetch the sources behind the hits):

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
constrain the stack from primary knowledge and the user confirms. If research is unavailable
(offline, no web tools), take the tooling from `./supplements/ecosystem-profiles.md`, cite the
profile as the source, and mark `adr-001`'s sources for re-verification on the first Component
ticket — never present recall as a researched recommendation.

### 3. Interview
Resolve the runtime details with `./supplements/interview.md`. Infer what you can (in audit,
from the profile); ask only the rest, and ask every question through AskUserQuestion — never a
free-form prose prompt — except purpose, which is a single free-text question. Open-ended
answers like path and stack still go through AskUserQuestion: offer inferred candidate options
and let the user type a custom value in the "Other" field. It resolves: identity, stack,
architecture + the boundary to enforce, gate command, git posture, reviewer roster + model tiers,
rules, the continuation discipline (defect intake, test-strength, the real system and its driver),
the performance surface or its N/A, and scale knobs. Present each technical choice as researched
options: mark the staff-engineer pick `(recommended)` with a one-line why and its source, and
**name the architecture style and cite the sources that will constrain the project** so the user
can accept it or offer their own. Reach decision-completeness before generating — no "TBD".

### 4. Generate (init) / Audit-and-install (audit)
Author each artifact at runtime, adapted to the project. Use `./supplements/exemplars/*` as
references to adapt, not stamp — each carries the format it needs, so no other skill is required.
First **emit the would-generate manifest** (the files and their one-line purpose) and get the
user's go-ahead; write only what is approved. Declining the manifest writes nothing — that is
the no-write preview, in either mode.
- **Init:** walk `./supplements/checklist.md` top to bottom; for each factor, create its
  securing artifact. Seed `adr-001` with the architecture decision — its named style, the
  sources behind it, and the boundary the arch-enforcement guards; `adr-002` with the gates
  (the fast gate, the separate test-strength and performance steps, and the runtime harness or
  its recorded deferral with the runbook fallback); `adr-003` with the definition of done — the
  definition of a bug and a per-dimension "done means" table.
- **Audit:** score each factor present / partial / absent against the checklist and present the
  gap report. Then ask the user which gaps to fill before writing anything. Add only the
  approved partial/absent artifacts; **never clobber** what is present; **reuse** existing gates
  and config (an existing `check`/`test` script becomes the gate). Migrate inline decisions into
  ADRs and backlog/scratch notes into tickets. Run the **context-scoping sub-audit** (checklist
  10a–10c): where CLAUDE.md carries orchestrator-only or task-specific content, or the workflow
  leans on a global skill/agent, **move** it to the correct layer
  (`./supplements/exemplars/instruction-placement.md`) rather than clobbering — a relocation, not a
  rewrite.

For factors 2/3/8/14/15/16, install the continuation-grade forms (rerunnable proof, red oracles
per criterion, an executing reviewer and a pre-code `ticket-adversary`, a mutation/revert gate kept
separate from the fast gate, a `Bug` ticket type, the always-on evidence rule, and the runtime
harness or its recorded deferral) per `./supplements/continuation.md`.

For factors 6, 7, 16, and 17, map the gate, the boundary check, the runtime driver, and the
performance benchmark to the stack with `./supplements/ecosystem-profiles.md` (Python, Node/TS,
Swift, Rust, Go, and a generic fallback). Factor 17 installs only where the project has a
perf-sensitive surface — else record a justified N/A in `adr-001`; wire the benchmark+threshold as
a separate pre-merge/scheduled step (never the fast gate) and document the profiler for triage.

### 5. Verify
- Run the new `verify` gate once — it must pass green on the seeded project.
- Confirm the commit hook fires and the `/work` skill's agent references all resolve — and that
  each resolves to a repo-scoped agent/skill (or a declared dependency), not a silent global.
- Walk `./supplements/checklist.md` — every factor present (17 present-or-justified-N/A), and the
  context-scoping sub-audit (10a–10c) passes: CLAUDE.md carries only durable cross-agent standards.
- Report what was created vs reused (audit), and the next action (write the first real
  ticket, or `/work` it).

## Modes

- **Init** — create the full tree from the checklist; seed `WORK-ITEMS.md` empty and write
  `tickets/TEMPLATE.md`, but do **not** author a first ticket — recommend it as the next action
  (step 5) for the user to `/work`; write `adr-001` (architecture), `adr-002` (quality gates), and
  `adr-003` (definition of done); then, in this order: initialize git, install hooks
  (`install-hooks.sh`, wired to the install step), then prove the gate green on the empty scaffold. Do not build the runtime harness at init — record the deferral in `adr-002`; the
  first ticket whose claim is about the running system builds it as part of its scope.
- **Audit** — review-first. Score the repo against the checklist, present the present /
  partial / absent report, and ask which gaps to fill. Then generate only the approved
  absent/partial artifacts. Migrate existing inline decisions into ADRs and existing
  backlog/notes into tickets. Reuse the existing gate; wrap it with hooks. Slim an oversized
  CLAUDE.md into path-scoped rules. Confirm the working tree is otherwise untouched.

## Supplements

| If the step is… | Read |
|-----------------|------|
| Understanding/teaching the factors | `./supplements/success-factors.md` |
| Applying the continuation / debugging-cycle discipline (factors 2/3/8/14/15/16) | `./supplements/continuation.md` |
| Auditing a repo or building the to-do | `./supplements/checklist.md` |
| Running the interview | `./supplements/interview.md` |
| Choosing gate / arch-enforcement / test / runtime-driver / benchmark tooling | `./supplements/ecosystem-profiles.md` |
| Deciding where an instruction lives (CLAUDE.md / skill / agent / rule) | `./supplements/exemplars/instruction-placement.md` |
| Building or auditing the runtime harness (factor 16) | `./supplements/exemplars/runtime-harness.md` |
| Authoring a specific artifact | `./supplements/exemplars/<artifact>.md` |

### Factor depth → `./supplements/success-factors.md`
**What:** the 17 factors elaborated — mechanism, failure mode, per-stack form.
**When to read:** explaining a factor, or deciding how to realize one for an unusual stack.

### Continuation discipline → `./supplements/continuation.md`
**What:** the debugging-cycle depth — provable outcome, red-before-green, review that executes, the
pre-code adversary, mutation test-strength, the definition of a bug, runtime proof — each with its
source and its exemplar.
**When to read:** installing or auditing factors 2/3/8/14/15/16; any project past initial scaffold.

### Audit rubric → `./supplements/checklist.md`
**What:** factor × securing artifact × present/partial/absent, plus the context-scoping sub-audit
(10a–10c). The single spine for both modes.
**When to read:** every run — it is the build to-do (init) and the gap audit (audit).

### Interview → `./supplements/interview.md`
**What:** the question script; infer-vs-ask guidance; AskUserQuestion groupings; how to present
researched options with a `(recommended)` pick and cited architecture.
**When to read:** step 3, every run.

### Ecosystem profiles → `./supplements/ecosystem-profiles.md`
**What:** concrete gate / arch-enforcement / test / mutation / runtime-driver /
performance-benchmark mappings for Python, Node/TS, Swift/iOS, Rust, Go, and a generic fallback.
**When to read:** resolving factors 6, 7, 14, 16, and 17 for the project's stack.

### Exemplars → `./supplements/exemplars/`
**What:** "adapt, do not copy" reference realizations — `ticket-template.md` (acceptance criteria,
the `Bug` variant, the `Irreversible sink` row), `work-items.md`, `work-skill.md` (the `/work`
pipeline incl. the plan-break pivot protocol), `adr-template.md` (incl. the `adr-003` seed),
`agent-roster.md` (incl. § reviewer isolation, § the adversary seat, § domain-knowledge preload),
`rules.md` (+ `rules/evidence.md`, `rules/coverage-derives-its-domain.md`, `rules/symbol-deletion.md`),
`hooks.md` (incl. `verify-hooks-live.sh`; test-strength is not a hook), `runtime-harness.md` (the
factor-16 harness — layout, the two proofs, anti-vacuity assertions, destructive interlocks),
`findings-bar.md` (the review-finding bar — TAG/LENS/CONSEQUENCE), `TEMPLATE-phase-audit.md` (the
phase-exit audit ticket), `instruction-placement.md` (CLAUDE.md / skill / agent / rule placement).
**When to read:** authoring that artifact in step 4.

## Self-test

A run is correct when: the target's `verify` gate passes green; the commit hook is
installed and fires; every `.claude/agents/*` referenced by the generated `/work` resolves
repo-locally (not from a silent global); `tickets/TEMPLATE.md` carries `## Acceptance criteria`
and `.claude/rules/evidence.md` exists; `docs/adr/` holds `adr-001`–`adr-003`;
`./supplements/checklist.md` shows factors 1–16 present, factor 17 present-or-justified-N/A, and
the context-scoping sub-audit (10a–10c) passing — CLAUDE.md holding only durable cross-agent
standards; and (audit) `git status` shows only intended additions.
