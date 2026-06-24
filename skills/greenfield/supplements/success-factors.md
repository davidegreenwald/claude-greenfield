# The 13 success factors, in depth

The factors that decide whether an agentic coding project stays correct and resumable.
Each holds across stacks; what differs is always *form* (a stack-specific gate command, a
platform-specific reviewer), never *presence*. That is what "full kit, adaptive form" means.
The examples below draw on three archetypes — a Python data pipeline, a SwiftUI mobile app,
and a TypeScript editor plugin — to show how one factor takes different shapes.

Each factor below gives: the principle, the mechanism (why it works), the failure mode
(the specific consequence of omitting it), and the form (how the realization varies by
stack). Realizations are cited by relative path so they can be adapted.

---

## Planning factors — resolved before code is written

### 1. Decision-completeness
**Principle.** The ticket resolves every decision — files touched, schema, flags, blast
radius, test scenarios — before implementation begins. No "TBD" survives into execution.
**Mechanism.** A decision-complete ticket makes execution mechanical: the agent transcribes
a resolved plan instead of designing at the keyboard, where it has the least context and
the most room to drift. A dedicated checker agent gates the ticket before any code.
**Failure without it.** The agent invents architecture mid-implementation, picks the first
workable option, and the design only surfaces in review — after the cost is sunk.
**Form.** Ticket sections drop domain-specific rows by stack (a data pipeline carries a
data-quality/provenance row a UI app omits); the decision-checker's checklist tracks the
template. Realization: `tickets/TEMPLATE.md`, `.claude/agents/ticket-planner.md`.

### 2. Evidence over recall
**Principle.** Every claim cites a primary source or a `file:line` read from source; every
count is produced by a tool; every change is tagged `proven — matches <file:line>` or
`novel — validated via <url>`.
**Mechanism.** LLMs emit fluent, specific, and wrong details. Forcing each decision to point
at precedent (proven) or an external source (novel) converts recall into verification, and a
fact-checker agent re-runs the citations before the plan is trusted.
**Failure without it.** A hallucinated API signature, version number, or "best practice"
ships as fact and is discovered only when it breaks.
**Form.** Citation targets vary (framework docs, repo files, saved research notes). The
evidence is produced by a shape-gated research fan-out in `/work` Phase 1: a Component ticket
fans out 2-3 generic research agents (`Explore`, or `general-purpose`) against the specific
algorithm/API it needs; a Small change does at most one targeted lookup. This per-ticket
research is narrow and distinct from the one-time project research run at init.
Realization: `.claude/agents/fact-checker.md`; the template's Technical Plan "Proven / Novel" field;
`/work` Phase 1 research fan-out.

### 3. Specify-first
**Principle.** Write the test scenarios as `scenario → expected` stubs before the
implementation, then make each pass.
**Mechanism.** Expectations fixed in advance are an external oracle the implementation must
satisfy. Written after, tests tend to assert whatever the code already does, so they confirm
the bug instead of catching it.
**Failure without it.** The agent rationalizes its output as correct; tests codify the
current behavior; regressions pass.
**Form.** Test framework and assertion style vary (pytest/Hypothesis, Vitest, Swift Testing);
the testing rule encodes determinism (fixed clock, seeded randomness).
Realization: `.claude/rules/testing.md`; `/work` execute phase.

### 4. Externalized memory
**Principle.** Project state lives in durable files — tickets, a work-items registry, ADRs —
not in the conversation.
**Mechanism.** The context window is volatile and lossy across sessions and compaction.
State written to disk lets any fresh context resume exactly where the last left off; the
ticket file is the record, so the registry can shed closed rows.
**Failure without it.** Work-in-progress lives only in chat; a new session re-derives intent,
duplicates effort, or contradicts an earlier decision. (A common anti-pattern: planning kept
in untracked scratch files — memory with no registry.)
**Form.** Portable. The registry columns (`ID | Title | Status | Phase | Ticket`) and the
"remove the row on close" rule hold across stacks.
Realization: `WORK-ITEMS.md`, `tickets/`.

### 5. Decision provenance
**Principle.** Architecture-level decisions are recorded as ADRs: context, the decision, its
consequences, and the alternatives rejected.
**Mechanism.** A recorded rationale with alternatives stops the next agent from relitigating a
settled call or violating an invariant it cannot see. The ADR is the why behind a rule the
gate enforces.
**Failure without it.** Invariants erode silently — a later change breaks an assumption no
one wrote down, and the gate that should have caught it was never derived from the decision.
**Form.** Portable structure; ADR count scales with the project (a multi-decision pipeline may
carry eight; a single-stack app, one). ADRs are created at three moments: seeded at init
(`adr-001` architecture, `adr-002` gates); written at Component-ticket prep when the `/work`
Phase 1 approach pass yields a durable cross-ticket decision (the ongoing source, recorded
before execution); and backfilled at the retrospective for any design-shape decision made
mid-implementation. The approach pass always happens for a Component ticket; it produces an ADR
only when there is a lasting decision to record.
Realization: `docs/adr/` (Status / Context / Decision / Consequences / Alternatives); `/work`
Phase 1 approach pass.

---

## Guardrail factors — secure work while it is built

### 6. Fast deterministic gate = "done"
**Principle.** One quick command — lint + types + tests + architecture — is the Definition of
Done, enforced by a hook at commit time, not invoked at the agent's discretion.
**Mechanism.** A deterministic, sub-second-to-seconds gate runs on every change at near-zero
cost, so it actually runs every time. Binding it to the commit hook removes the option to
skip it; the faster it is, the tighter the loop and the earlier the catch.
**Failure without it.** A slow or optional gate gets skipped under time pressure; broken code
reaches main; "done" means "the agent believes it works." A mobile app whose UI suite runs in
minutes can bind only a fast subset to commit; a pipeline with fast Python tooling can
commit-enforce the whole gate.
**Form.** The command maps to the stack: `make verify` (Python: ruff + mypy --strict +
import-linter + pytest), `npm run check` (Node/TS: tsc + ESLint + Vitest), `swift test +
swiftlint` (Swift). Keep slow suites (UI/e2e) *out* of the fast gate; run them as a separate
pre-merge step.
Realization: a `Makefile` `verify` target, or an existing `package.json` `check` script reused as-is.

### 7. Executable architecture
**Principle.** Layering and dependency intent is encoded as a fitness function that fails the
build, not as prose in a doc.
**Mechanism.** A machine-checked contract cannot drift; the architecture is true because the
build proves it on every change. Where the ecosystem has no such tool, a path-scoped rule plus
a reviewer agent guarding the boundary is the fallback — never silently drop the factor.
**Failure without it.** Documented boundaries decay into spaghetti as each expedient import
goes unchecked; by the time anyone notices, unwinding it is a project.
**Form.** import-linter contracts (Python), `dependency-cruiser` / `eslint-plugin-boundaries`
/ ESLint `no-restricted-imports` (Node/TS), SPM module boundaries + a lint rule (Swift). An
editor plugin can enforce "no host-API import in pure core" via an ESLint
`no-restricted-imports` rule — a real fitness function.
Realization: `pyproject.toml` `[tool.importlinter]`; an `eslint.config.mjs` boundary rule.

### 8. Independent adversarial review
**Principle.** Read-only reviewer subagents with distinct lenses return structured verdicts
before merge — decision-completeness and citations before code, blast-radius/architecture,
correctness, and pattern-conformance after.
**Mechanism.** A separate, differently-prompted context catches what the author's context is
blind to. Reviewers run in parallel and return only a concise verdict, so they add coverage
without flooding the orchestrator's context. Distinct lenses beat N identical reviewers — each
failure mode gets a dedicated eye.
**Failure without it.** The author reviews their own work in the same context that produced the
mistake and confirms it; subtle blast-radius and provenance errors slip through.
**Form.** Which lenses, and which domain reviewers are added (security, perf, a11y,
API-compat). Pre/post split and parallelism are constant.
Realization: `.claude/agents/` (architecture-reviewer, correctness-reviewer, patterns-auditor,
…); see `./exemplars/agent-roster.md` for the roster rationale.

### 9. Isolated work, always-green main
**Principle.** Each unit of work happens on its own branch or worktree and lands only when the
gate is green, via a clean fast-forward (local) or PR (remote).
**Mechanism.** Isolation keeps half-done work off main; the green-only merge keeps main
releasable and every change reversible. A fast-forward advances main without authoring a merge
commit, so a no-direct-commit hook still permits it.
**Failure without it.** Partial work pollutes main; a red main blocks everyone; bisecting a
regression is hopeless when commits are not individually green.
**Form.** Local-only → worktree-per-ticket + `git merge --ff-only`; remote → branch + PR + CI.
The interview picks from git posture.
Realization: `.claude/skills/work/SKILL.md` Phase 0 and Phase 8.

### 10. Progressive disclosure of conventions
**Principle.** CLAUDE.md carries project-wide context; path-scoped `.claude/rules/*.md` deliver
the narrow constraint only when a matching file is touched.
**Mechanism.** Attention is finite and instructions decay with distance and volume. A rule that
loads exactly when its `paths:` glob matches stays salient at the point of use; a 70KB wall of
every rule at once does not. Per-module CLAUDE.md localizes "what goes here."
**Failure without it.** One monolithic instruction file (a 70KB CLAUDE.md is common) buries the
load-bearing rule among hundreds; the agent misses the constraint that applied to the line it
just wrote.
**Form.** Which path globs, and whether per-module CLAUDE.md is warranted (scales with module
count). Always-on rules (shell hygiene, evidence) carry no `paths:`.
Realization: `.claude/rules/` (e.g. db.md, domain-purity.md, llm.md, testing.md scoped;
shell-hygiene.md always-on).

### 11. Calibrated human-in-the-loop
**Principle.** The agent proceeds autonomously on resolved decisions and halts to ask only on a
defined surprise list (gate would refuse the merge, an unresolved review block, a design-shape
change made mid-ticket, a preexisting bug surfaced).
**Mechanism.** A written trigger list converts "when should I ask?" from a judgment call into a
rule. It spends the human's attention only where a decision is genuinely theirs, and never
silently exceeds the approved scope.
**Failure without it.** Either the agent asks at every step (the human becomes the bottleneck)
or it never asks (unapproved architectural drift lands on main). A higher-stakes project may
gate every merge on the user; a trusted local project auto-merges on green and asks only on a
surprise.
**Form.** Merge approval (remote/PR, or higher-stakes projects) vs auto-merge-on-green (local,
trusted gate). The trigger list itself is near-constant. Beyond the surprise-driven halts, one
planned touchpoint is always-on: a Component ticket's `/work` Phase 1 approach pass presents
2-4 approaches and asks the human to pick before decomposing — calibrated by ticket shape so it
never fires on a Small change.
Realization: `.claude/skills/work/SKILL.md` Phase 8 triggers; `/work` Phase 1 approach pass.

### 12. Effort-proportional model tiering
**Principle.** Assign the strongest model to judgment-heavy review and cheaper, faster models to
mechanical and citation checks.
**Mechanism.** Review quality and token cost both scale with model tier, but the work does not
need uniform tier. Architecture and correctness judgment justify the strongest model; a
decision-completeness checklist or a citation re-run does not.
**Failure without it.** Uniform top-tier wastes budget on grep-shaped work; uniform low-tier
misses the architectural problems that only a strong model catches.
**Form.** Which tiers exist for the provider. A typical balance: judgment agents
(architecture-reviewer, correctness-reviewer) on the strongest tier; mechanical agents
(ticket-planner, fact-checker, patterns-auditor, pm-explainer) on a mid tier. Set via each
agent's `model:` frontmatter.
Realization: the roster table in `./exemplars/agent-roster.md`.

---

## Meta factor

### 13. Self-improving harness
**Principle.** Every work cycle ends with a retrospective that inspects the process — the
ticket, CLAUDE.md, rules, the gate, the reviewers, any friction hit — and lands fixes in the
same session.
**Mechanism.** The harness is a living system; small frictions left unfixed compound. A standing
retrospective phase routes obvious fixes to immediate landing and larger ones to a ticket, so
the workflow gets sharper each cycle instead of accreting cruft.
**Failure without it.** Stale rules, slow gates, and contradictory instructions persist for
months; the same friction is paid every session.
**Form.** Portable. The retrospective inspects the ticket, CLAUDE.md, the rules, the gate, and
the reviewers, and lands obvious in-scope fixes in the same session.
Realization: `.claude/skills/work/SKILL.md` retrospective phase.

---

## Using these at runtime

- **Teaching/justifying** a guardrail to the user: quote the principle + failure mode.
- **Unusual stack:** keep the principle and mechanism fixed; derive a new form from the
  ecosystem (see `./ecosystem-profiles.md`). If you cannot find a machine-checked realization
  for factor 7, fall back to rule + reviewer — do not drop it.
- **Scaling down a small project:** every factor is still present; reduce *sizing* (fewer ADRs,
  no per-module CLAUDE.md, a 2-3 agent roster), not *coverage*.
