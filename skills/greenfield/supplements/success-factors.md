# The 17 success factors, in depth

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
count is produced by a tool; every change is tagged `proven — matches <file:line>`,
`novel — validated via <url>`, or `novel — driven via <probe + output>`; recall is not a fourth tag.
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
**Continuation form.** Beyond a citation, the proof of done is a *rerunnable command + its actual
output* recorded in the ticket (an Admission/Evidence field) — the command a human re-runs to
regenerate the proof, not a one-time capture. A recorded command-plus-output cannot be fabricated
after the fact. See `./continuation.md` §1.

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
**Continuation form.** Specify-first alone lets a test written against the produced code pass
vacuously. Confirm each test is *red* (fails) before implementation, then green; write it from the
acceptance criteria, not the code (the oracle-overfitting guard). A test never seen to fail is a
tautology. See `./continuation.md` §2.

### 4. Externalized memory
**Principle.** Project state lives in durable files — tickets, a work-items registry, ADRs —
not in the conversation.
**Mechanism.** The context window is volatile and lossy across sessions and compaction.
State written to disk lets any fresh context resume exactly where the last left off; the
ticket file is the record, so the registry can shed closed rows.
**Failure without it.** Work-in-progress lives only in chat; a new session re-derives intent,
duplicates effort, or contradicts an earlier decision. (A common anti-pattern: planning kept
in untracked scratch files — memory with no registry.)
**Form.** Portable. The registry columns (`ID | Title | Type | Status | Phase | Ticket`) and the
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
(`adr-001` architecture, `adr-002` gates, `adr-003` definition of done); written at
Component-ticket prep when the `/work` Phase 1 approach pass yields a durable cross-ticket
decision (the ongoing source, recorded before execution); and backfilled at the retrospective
for any design-shape decision made mid-implementation. The approach pass always happens for a
Component ticket; it produces an ADR only when there is a lasting decision to record.
Realization: `docs/adr/` (Status / Context / Decision / Consequences / Alternatives); `/work`
Phase 1 approach pass.

---

## Guardrail factors — secure work while it is built

### 6. Fast deterministic gate — one condition of done
**Principle.** One quick command — lint + types + tests + architecture — runs on every change,
enforced by a hook at commit time, not invoked at the agent's discretion. It is one condition of
done, never the whole definition — that lives in `adr-003` (criteria, evidence, gate, reviewers, bugs).
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
**Continuation form.** Diff-reading is theory; a dispute about correctness is settled by running the
code. At least one reviewer *executes* — runs the tests, reproduces the defect, refutes the result by
running it — and pastes the output; where it did not execute, it reports a question, not a defect.
Consensus among diff-readers is not verification. See `./continuation.md` §3, and §7 for the
pre-code adversary that tries to break the ticket before it is built.
**Isolation (prescriptive).** Reviewers are isolated **by tool grant, not a sandbox worktree** —
`Read, Grep, Glob` (+`Bash` to execute), never `Write`/`Edit`, never `isolation: worktree` (a
reviewer worktree nests a checkout under the repo and pollutes its own gate). The review target is
isolated by **freezing the diff (a commit) before spawning**. See `./exemplars/agent-roster.md`.

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

### 10. Progressive disclosure and context scoping
**Principle.** Each instruction reaches exactly the context that needs it, and no other.
Project-wide durable standards live in CLAUDE.md; a narrow constraint lives in a path-scoped
`.claude/rules/*.md` that loads only when a matching file is touched; orchestrator-only
coordination lives in the `/work` skill; a task-specific constraint lives in the subagent that
performs the task. The size of what any one agent loads is matched to the model and effort
running it.
**Mechanism.** Three facts about how Claude Code assembles context make *placement*
load-bearing, not cosmetic (source: Claude Code docs — `sub-agents.md`, `memory.md`,
`skills.md`):
- **CLAUDE.md is inherited wholesale by every subagent** — root, nested, and user-global, plus
  managed policy. (The built-in `Explore` and `Plan` agents are the only ones that skip it — so
  the `/work` research fan-out, which runs on `Explore`/`general-purpose`, does not even receive
  CLAUDE.md.) Anything in CLAUDE.md is paid for in *every* subagent's context whether or not that
  agent can act on it.
- **A subagent gets only its own system prompt** — its agent-definition body, plus the CLAUDE.md
  hierarchy, its git status, and the skills named in its `skills:` field. It does *not* inherit
  the orchestrator's system prompt, the orchestrator's conversation, or any skill the
  orchestrator invoked. So orchestrator-only logic in CLAUDE.md reaches subagents as noise;
  placed in the `/work` skill it reaches no subagent at all — correct, because only the
  orchestrator runs it.
- **Skills and rules are opt-in, not ambient** — a skill preloads into a subagent only when named
  in its `skills:` field, and a path-scoped rule is *not* listed among what a subagent inherits
  (undocumented as of this writing — treat rules as main-session-only until confirmed). A
  constraint a subagent must honor therefore belongs in that subagent's definition or its listed
  skills, not only in a path-scoped rule.
Attention is also finite and capability is tiered: a model at a given effort has a baseline
competence for a task (measurable by eval), and instructions help most when they close exactly
the gap to that baseline. Over-instructing a capable agent buries the load-bearing line; a 70KB
CLAUDE.md inherited by every subagent is the worst case — noise and token cost multiplied across
the fleet.
**Failure without it.** One per scoping axis:
- *Inheritance leak.* Orchestrator-only pipeline steps and task-specific constraints sit in
  CLAUDE.md; every reviewer and worker subagent inherits them, the load-bearing standard is
  buried, and the cost is paid on every spawn.
- *Mis-sized instruction.* A constraint a subagent must honor lives only in a rule or skill it
  never loads, so it silently violates it; or a capable agent is handed a wall it did not need.
- *Silent global dependency.* The workflow leans on a skill or agent that exists only in a
  personal `~/.claude/`; a second contributor or a CI run without it gets different behavior, and
  the project is not reproducible.
**Form.** Which path globs; whether per-module CLAUDE.md is warranted (scales with module count);
which skills each subagent lists; which model/effort each agent runs (factor 12) and the
instruction load matched to it. Always-on rules (shell hygiene, evidence) carry no `paths:`. The
CLAUDE.md / skill / agent / rule split is portable; only the specific contents vary.
Realization: root + per-module `CLAUDE.md` (durable cross-agent standards only); `.claude/rules/`
(path-scoped + always-on); the `/work` skill (orchestrator-only pipeline); each
`.claude/agents/*` definition and its `skills:` list (task-specific); the placement decision
table in `./exemplars/instruction-placement.md`; the context-scoping sub-audit in
`./checklist.md`.

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
agent's `model:` frontmatter. This pairs with factor 10: match the *instruction load* to the
chosen tier, not just the tier to the task — a stronger tier at higher effort needs less spelled
out, so its context stays lean.
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

## Continuation factors — the debugging cycle

Load-bearing once the project accumulates bugs and leans on AI-written tests. Universal in
*presence*; the form scales from a lightweight convention (throwaway) to full tooling (mature repo).
Depth and sources: `./continuation.md`.

### 14. Test-strength
**Principle.** A passing suite proves nothing unless it can fail. Every test dies on revert (a
negative-control check); the suite as a whole is measured by mutation testing on changed files.
**Mechanism.** Coverage proves a line executed, not that its effect was asserted — a file can report
100% coverage with tests that verify nothing. Mutation (and the per-test revert check) is the sensor
that a green suite can actually go red. It matters most exactly here, where an agent writes most of
the tests.
**Failure without it.** A suite of vacuous, always-green tests passes the gate while regressions slip
under it; "tests pass" stops meaning "behavior is checked."
**Form.** The mutation tool varies by stack (Stryker / mutmut / cargo-mutants / muter / gremlins; a
revert-check convention where none exists) — see `./ecosystem-profiles.md`. Run it as a standing
check on changed files, kept *out* of the fast commit gate (mutation is slow); wire it as a
pre-merge or scheduled step. Realization: `./exemplars/hooks.md`, `./exemplars/rules.md`.

### 15. Defect intake — the definition of a bug
**Principle.** A bug enters as a typed defect carrying three fields before a fix is written: (a) the
named behavioral consequence, (b) a reproduction, (c) a red regression test that fails first.
**Mechanism.** The intake forces every bug through the red/green loop: no fix is written until a test
reproduces the failure and fails first. That converts a one-off fix into a permanent detector for the
whole class, and stops "fixed it" from meaning "cannot see it anymore."
**Failure without it.** Bugs are patched ad hoc with no regression test; the class recurs, and the
same defect is rediscovered later with no sensor guarding it.
**Form.** Full form: a distinct `Bug` ticket type + a defect rule + a registry `Type` column.
Lightweight form (a project already enforcing red-test-first everywhere): the same three fields
folded into the generic ticket via a defect admission criterion + a per-dimension "Bugs"
definition-of-done. Realization: `./exemplars/ticket-template.md`, `./exemplars/rules.md`.

### 16. Runtime proof — drive the real system
**Principle.** A behavioral claim is proven by driving the code, never by citing it. Where the claim is
about the running system — the app, the service, the device, the plugin host, the external API — neither
a unit test nor a `file:line` citation can establish it; only a run against the real thing can. If you
cannot run the real thing, you are theorizing.
**Mechanism.** A citation shows the code exists and says what you quoted; the inference from it to a
behavior is the agent's, and it can be wrong while every citation is correct. A unit suite exercises what
its fixtures stage, and an agent writes fixtures from the same model of the system that wrote the code —
so both share the blind spot. A runtime harness — a distinct artifact from the test suite — drives the
real system end to end without human hands, carries its own proof that it can fail
(`--negative-control`) and that it is deterministic (`--twice`), and refuses vacuous assertions. An
always-on evidence rule makes "drive, don't cite" the standard for every claim an agent makes, including
the claims a plan is made of; `/work` Phase 8 runs the harness before a behavior change lands.
**Failure without it.** Four data-destroying defects passed fully green unit suites in the reference
project: fixture homogeneity (every fixture staged the same shape), a default-settings path no fixture
exercised, a repair that ended more permissive than its input, and a plan whose Technical Plan
contradicted its own Verification forty lines below. One shipped; the ones caught were caught only because
the plan or the repair was driven against the real system. Consensus is no substitute: ten reviewers
unanimously endorsed a non-existent vulnerability, and a single empirical test killed it —
https://arxiv.org/abs/2604.19049
**Form.** Three artifacts, sized to the project. (1) `.claude/rules/evidence.md`, always-on
(`./exemplars/rules/evidence.md`). (2) A runtime harness, built at `/work` discretion — the first ticket
whose claim is about the running system builds it as part of its scope; until then `adr-002` records the
deferral and the fallback is a scripted runbook whose every step writes its output to a file and diffs it
against a committed expected-output file — that diff is the red; the ticket's Evidence cites the run
(`./exemplars/runtime-harness.md`;
drivers per stack in `./ecosystem-profiles.md` § Runtime harness drivers). (3) The landing discipline:
`/work` Phase 8 runs the harness (or the recorded runbook) before a behavior change lands and pastes the
report line into the ticket's Evidence; a red run is a stop-and-ask trigger (`./exemplars/work-skill.md`
Phase 8). Realization: `./exemplars/rules/evidence.md`, `./exemplars/runtime-harness.md`,
`./exemplars/work-skill.md` Phases 1, 3, 8.

---

## Non-functional factors — project-specific (present when applicable)

Unlike factors 1–16, this factor's *artifact* is required only where the project has the surface
it defends; its *principle* (name that surface and decide) is universal. Score it
present-or-justified-N/A, never forced.

### 17. Performance measure — benchmark, then profiler
**Principle.** Where a project has a performance-sensitive surface — a rendered page, a database
query, a hot compute path, a memory footprint — it carries at least one *repeatable benchmark*
that produces a number, and a threshold or baseline that **fails the build when the number
regresses**. Profiling is the escalation, not the measure. A project with no perf-sensitive
surface records that as an explicit, justified N/A (in `adr-001`) rather than a hollow benchmark.
**Mechanism.** A repeatable, variance-aware benchmark converts "feels fast" into a measured number
with known noise; wiring its threshold to a non-zero exit code makes CI hard-fail on regression,
so performance is defended continuously instead of discovered in production. Raw timing is noisy,
so the benchmark reports a distribution, not one sample (Criterion's confidence intervals, Go's
benchstat, pytest-benchmark's saved runs). When the gate trips, a profiler attributes the
regression to a call site — the benchmark says *that* it regressed and by how much; the profiler
says *why*. Like the mutation gate (factor 14), it is slow and noisy, so it is kept **out** of the
fast commit gate and run as a separate pre-merge or scheduled step.
**Failure without it.** Regressions land silently and compound; "slow" is reported by users, not
CI; and by the time anyone notices there is no baseline to bisect against and no profiler wired to
localize it — every investigation starts from zero.
**Form.** The perf-sensitive surface decides the benchmark type (page render, query, hot loop,
allocation); the stack decides the tool (see `./ecosystem-profiles.md`): web → Lighthouse CI
assertions / k6 `thresholds`; DB → pgbench + `EXPLAIN ANALYZE` behind a threshold wrapper; Python →
pytest-benchmark `--benchmark-compare-fail`; Node/TS → Vitest `bench` / tinybench vs a stored
baseline; Swift/iOS/macOS → XCTest `measure` / `XCTMetric` baselines + Instruments; Rust → Criterion
+ cargo-flamegraph; Go → `testing.B` + benchstat + pprof. The gate mechanism is constant — a number
vs a threshold/baseline → non-zero exit → CI fails — and the profiler is a documented, on-demand
triage invocation, not a CI step.
Realization: a benchmark step + threshold/baseline per stack (`./ecosystem-profiles.md`), wired as a
separate pre-merge/scheduled step like factor 14; the profiler invocation recorded in an ADR or
rule; the existing `perf-reviewer` (roster) reads the benchmark result rather than eyeballing hot
paths. See `./exemplars/agent-roster.md`.

---

## Using these at runtime

- **Teaching/justifying** a guardrail to the user: quote the principle + failure mode.
- **Unusual stack:** keep the principle and mechanism fixed; derive a new form from the
  ecosystem (see `./ecosystem-profiles.md`). If you cannot find a machine-checked realization
  for factor 7, fall back to rule + reviewer — do not drop it.
- **Scaling down a small project:** factors 1–16 are still present; reduce *sizing* (fewer ADRs,
  no per-module CLAUDE.md, a 2-3 agent roster), not *coverage*. Factor 17 is the one
  present-when-applicable exception — a project with no perf-sensitive surface records a justified
  N/A instead of a hollow benchmark.
