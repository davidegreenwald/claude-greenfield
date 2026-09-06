# Continuation — the debugging-cycle discipline

Depth for factors 2, 3, 8, 14, 15, 16 — the practices that keep an agentic project correct *after* it
stands up, once it accumulates bugs and leans on AI-written tests. Install the continuation form of
each factor at init; audit for it on a mature repo. Realizations are the skill's own exemplars in
`./exemplars/`.

Unifying principle: a behavioral rule ("claim done only when verified") is reliable only when reified
as a gate the workflow cannot skip. Agreement — a reviewer with itself, or reviewers with each other
— is not verification; execution is. And reasoning over a diff is theory: a behavioral claim is proven by
driving the code, never by citing it. Ten reviewers unanimously endorsed a non-existent OpenSSL
padding oracle; a single empirical test killed it — https://arxiv.org/abs/2604.19049

## 1. Provable outcome — a rerunnable command, not a citation (factor 2)
Done is proven by a command a human re-runs plus its actual output, recorded in the ticket. A
`file:line` citation shows precedent, not that the change works; a recorded command-plus-output
cannot be fabricated after the fact. "Your job is to deliver code you have proven to work" —
https://simonwillison.net/2025/Dec/18/code-proven-to-work/
- **Artifact:** an `Admission` line (which criterion is met) + an `## Evidence` block holding the
  exact invocation and the output it actually produced — so the next reader re-runs instead of
  believing. See `./exemplars/ticket-template.md`.

## 2. Red before green — the test must fail first (factor 3)
Specify-first is not enough: a test written after the code, or never seen to fail, asserts whatever
the code already does. Confirm the test is RED before implementation, and write it from the
acceptance criteria, not the produced code (the oracle-overfitting guard) — an agent's own
after-the-fact suite encodes the implementation, not the intent.
- **Artifact:** the ticket names the red test by path, the command that runs it, and the red output
  it produced before any implementation existed. Where the claim is about the running app, the red
  artifact is a live QA step instead — same rule, different harness. See
  `./exemplars/ticket-template.md` and the testing rule in `./exemplars/rules.md`.

## 3. Adversarial review that executes (factor 8)
Reasoning over a diff is theory; only running it settles a dispute. At least one reviewer runs the
tests, reproduces the defect, and tries to refute the result by execution — pasting the output. A
reviewer that only reads the diff, and reviewers that merely agree, do not verify. The positive case:
a coding agent beat expert-tuned GPU kernels over a 7-day autonomous run *because* its scoring
function executed (correctness gate + measured throughput) — autonomous improvement reaches exactly
as far as the check is executable — https://arxiv.org/abs/2603.24517
- **Artifact:** an executing reviewer (`tools: Read, Grep, Glob, Bash`) that stays read-only on the
  codebase but runs code; every "I ran X" claim carries the output, and where it did not execute it
  reports a question, not a defect. Keep it sandboxed — may run the test suite; may not deploy or
  write outside a scratch dir. See `./exemplars/agent-roster.md`.

## 4. Drive tests both ways — mutation/revert (factor 14)
A green suite proves nothing unless it can go red. Coverage lies — it proves a line ran, not that its
effect was checked; a file reporting 100% statement coverage had no real tests and 13 mutation
survivors — https://martinfowler.com/articles/sensors-for-coding-agents.html Per-test: a
revert/negative-control check (break the logic, confirm a test fails). Whole-suite: mutation testing
on changed files.
- **Artifact:** a mutation or revert check as a standing gate on changed files, kept **separate from
  the fast commit gate** — mutation is slow and must not sit in the pre-commit hook. Per-stack tool
  in `./ecosystem-profiles.md`; wiring in `./exemplars/hooks.md`.

## 5. The definition of a bug (factor 15)
A bug enters as a typed defect carrying three fields before a fix is written: (a) the named
behavioral consequence, (b) a reproduction, (c) a red regression test that fails first. Without this
intake an escaped bug has no path that forces the red/green loop, and the class recurs.
- **Artifact (adaptive form):** the full form is a distinct `Bug` ticket type + a defect rule
  enforcing the three fields + a registry `Type` column. The lightweight form (a project that
  already enforces red-test-first on every ticket) folds the same three fields into the generic
  ticket via a defect admission criterion plus a per-dimension "Bugs" definition-of-done. See
  `./exemplars/ticket-template.md` and `./exemplars/rules.md`.

## 6. Runtime proof — drive the real system (factor 16)
Where a claim is about the running system — the app, the service, the device, the plugin host, the
external API — neither a unit test nor a `file:line` citation can establish it: the fixtures were staged
from the same model of the system that wrote the code, so both share the blind spot. Only a run against
the real thing settles it. Four data-destroying defects in the reference project (a production plugin)
passed fully green unit suites (fixture homogeneity; a default-settings path no fixture staged; a repair
that ended more permissive than its input; a plan that contradicted its own Verification) — the ones
caught were caught only because the plan or the repair was driven against the real system.
- **Artifact:** (1) an always-on evidence rule — drive, don't cite; a negative from an unvalidated
  instrument is worth nothing; an irreversible guard is proven both ways (`./exemplars/rules/evidence.md`);
  (2) a runtime harness, distinct from the test suite, carrying its own `--negative-control` (it can go
  red at exactly the planted step) and `--twice` (two cold runs diff clean) proofs, with assertions that
  cannot pass vacuously and interlocked destructive verbs — built at `/work` discretion by the first
  ticket whose claim needs it, else a recorded deferral in `adr-002` (`./exemplars/runtime-harness.md`);
  (3) the landing discipline: `/work` Phase 8 runs the harness before a behavior change lands and pastes
  the report line into the ticket's Evidence; a red run is a stop-and-ask trigger
  (`./exemplars/work-skill.md` Phase 8).

## 7. The adversary — break the ticket before it is built (factor 8, pre-code)
A ticket can be completely decided and wrong. The completeness planner asks "is this decided?"; the
adversary asks "if I do exactly what this ticket says, what breaks?" — and it has failed only when it
cannot find the case. It re-runs every command, re-opens every citation, cross-checks the Technical Plan
against the Verification, then drives the plan as a scratch diff and attacks it — starting at the
ticket's red oracles (is there one per criterion? is each actually red? does it assert the Goal, not the
mechanism?). The reference incident: a plan reviewed as decision-complete whose Technical Plan and
Verification prescribed opposite writes forty lines apart; driven, it deleted records with their history.
- **Artifact:** a `ticket-adversary` seat (`./exemplars/agent-roster.md` § the adversary seat) in `/work`
  Phase 2 for every Component and Bug, returning `BREAKS (evidence)` / `BREAKS (plan)` / `SURVIVES` with
  the list of what it tried; every probe it builds is handed over verbatim; rounds capped at two per
  ticket, a second `BREAKS (plan)` parks the ticket (`./exemplars/work-skill.md` § plan-break pivot protocol).

## Adaptive form
Scale by sizing, never by dropping a factor. A throwaway: factor 14 = a revert-check convention,
factor 15 = a Bug section on the generic ticket, factor 16 = the evidence rule + a scripted runbook whose
steps record their output. A mature product: factor 14 = mutation with a break threshold, factor 15 = a
distinct `Bug` type + registry column + defect rule, factor 16 = the automated harness with both proofs,
run at Phase 8 before every behavior change lands. Every form keeps the practice present.
