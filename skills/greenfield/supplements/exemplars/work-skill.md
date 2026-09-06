# Exemplar — the project's `/work` skill

**Adapt, do not copy verbatim.** Realizes factors 8, 9, 11, 13, plus the continuation forms of
factors 14, 15, 16. Generalized from a 10-phase refined `/work` pipeline. Author the real file by
filling the template below — replace every `{{placeholder}}` from the interview decision sheet.

The project's `/work` skill is **project-level** (`<project>/.claude/skills/work/`), and is the
execution pipeline that ties the harness together: it spawns the reviewer roster, respects the
gate, and integrates work safely. It does not run the gate itself (the hook does) and it does
not auto-fire (`disable-model-invocation: true`).

Placeholders to fill:
- `{{GATE}}` — the verify command (e.g. `make verify`, `npm run check`).
- `{{POSTURE}}` — `local` (worktree + `git merge --ff-only`) or `remote` (branch + PR + CI).
- `{{SHAPES}}` — the ticket shapes (`Small change | Component | Bug`).
- `{{PLANNER}}`, `{{FACTCHECK}}`, `{{ARCH_REVIEWER}}`, `{{ADVERSARY}}`, `{{CORRECTNESS}}`,
  `{{PATTERNS}}`, `{{PM}}` — the chosen roster agent names (omit any the project doesn't use).

There is deliberately no research-agent placeholder: the Phase 1 research fan-out uses the
install's generic agent (`Explore`, or `general-purpose`), not a roster member — see
`./agent-roster.md`.

---

```markdown
---
name: work
description: Execute a ticket end-to-end with the review pipeline (plan + research + approach →
  pre-exec review → implement → mechanical gate → post-impl review → integrate → retrospective →
  next ticket). User-invocable only.
argument-hint: "[ticket path or task description]"
disable-model-invocation: true
---

The gate ({{GATE}}) runs at commit time via the pre-commit hook (authoritative) and at
turn-end via the Stop hook (backstop). Do not run it manually. Spawn reviewers in a single
message (parallel). Run each review pass once; verify fixes yourself.

## Phase 0 — Branch / worktree
{{POSTURE=local}}: `git worktree add ../<repo>-<slug> -b <slug>` (or the project's `make worktree
BRANCH=<slug>`) into an isolated checkout.
{{POSTURE=remote}}: `git checkout -b <slug>`. (Bootstrap exception: branch in place if there is
no commit history yet.)

## Phase 1 — Ticket shape & prep
Classify {{SHAPES}}. A single-purpose obvious change is a Small change; a defect is a Bug; anything
else is a Component and needs a ticket written from `tickets/TEMPLATE.md` first (decision-complete,
registered in `WORK-ITEMS.md`).

**Bug intake (factor 15).** A Bug ticket records three fields before any fix: the named consequence,
a reproduction, and a red regression test named by path with the red output it produced first. Do
not start the fix until that test exists and fails.

**Acceptance criteria and their oracles (factors 1, 3, 16).** Number the ticket's acceptance criteria
`C1…Cn`. Each names the test — or, for a claim about the running system, the runtime-harness step —
that proves it. Before Phase 3 every oracle exists and is **RED on the current tree**; commit the red
output. A criterion with no oracle, or one already green, is a gap the planner returns, not a detail
to fix later.

**Research (factor 2), gated by shape.** Small change: at most one targeted lookup. Component:
fan out 2-3 research agents in a single parallel message — use the install's generic research
agent (`Explore`, or `general-purpose`); do not assume bespoke researchers exist — to
validate the specific algorithm, API, or library this ticket needs against primary sources.
This is narrow, per-ticket research, distinct from the one-time project research run at init.
Each finding feeds the ticket's Research and Proven/Novel fields with a cited source or
`file:line` — no claim from recall.

**Approach pass (factors 5, 11) — always-on for Component tickets.** Before decomposing, present
2-4 concrete approaches (mechanism + tradeoff each) and ask the human to pick via one
AskUserQuestion. Small changes skip this. When the chosen approach is a durable, cross-ticket
decision — introduces a pattern, moves a boundary, or picks among options that will constrain
later work — write an ADR from `docs/adr/TEMPLATE.md` now, before execution, and set the
ticket's `ADR:` field to `creates adr-NNN`. A Component ticket that only implements within
existing ADRs cites them (`cites adr-NNN`) and writes none.

**Completeness gate.** Spawn {{PLANNER}}; resolve every gap; proceed only on COMPLETE. The
planner hard-fails a Component ticket that changes a boundary but carries `ADR: none`, or whose
Research/Proven-Novel fields cite nothing.

## Phase 2 — Pre-execution review
Small change: {{FACTCHECK}} where the roster has one; otherwise {{ADVERSARY}} duties 1–2 only
(re-run commands, re-open citations). Component: {{FACTCHECK}} ∥ {{ARCH_REVIEWER}} ∥ {{ADVERSARY}}
(parallel); a Bug also runs {{ADVERSARY}}. Resolve any BLOCK or unverified citation before Phase 3.

{{ADVERSARY}} tries to **BREAK** the ticket, not approve it (`./agent-roster.md` § the adversary seat):
re-runs its commands, re-opens its citations, cross-checks Plan vs Verification, then drives the plan in
a scratch worktree starting at the red oracles and attacks it. `BREAKS (evidence)` → repair the record,
re-derive, continue (no retry consumed). `BREAKS (plan)` → the plan-break pivot protocol below: one retry
on a genuinely different design. A second `BREAKS (plan)` on the same ticket → park/demote to a stub; keep the
problem, the evidence, and both counterexamples. Every probe the adversary built is in its report
verbatim — run it, do not rebuild it.

## Phase 3 — Execute
Work the decision-complete ticket; execution is mechanical. Start at the red oracles: write the
ticket's `scenario → expected` cases as tests first and **confirm they fail (red)** before
implementing — a test never seen to fail is a tautology — then make each pass (green). Adjacent bug
fixes are fine; design-shape changes are not (halt, resolve in the ticket, resume).

**Drive before you build on it (factor 16).** A plan is made of behavioral claims. Drive each one
against the real path — not a reconstruction of it — before building on it; a claim about the running
system is driven through the runtime harness (`./runtime-harness.md`), and building the harness is in
scope when none exists. Reasoning over the diff is theory; the run settles it.

## Phase 4 — Self-check
Re-read your own diff against the ticket's Blast radius. What could be silently wrong? What
drifted from the plan? Any claim without a `file:line`/url? Fix; cite; skip nits.

## Phase 5 — Mechanical gate
{{GATE}} runs at commit time (pre-commit hook) and turn-end (Stop hook). Do not invoke it
here. For a mid-implementation spot check, run a single sub-gate.

## Phase 5b — Test-strength (factor 14, changed files)
Run the stack's mutation or revert check over the changed files (see `../ecosystem-profiles.md`) —
not part of the fast gate; run it here, or for a trivial Small change defer to the PR check (remote
posture) — in local posture this phase is the only home it has; always run it for a Bug fix. Kill each surviving mutant by adding the missing assertion: a survivor is
a test that does not actually check the behavior it covers.

## Phase 6 — Post-implementation review
**Freeze the diff first — commit the change set before spawning any reviewer.** Reviewers read the
committed snapshot, not a live tree; if you keep editing while they run they review a tree that no
longer exists. The freeze is what makes the parallel spawn sound. Reviewers are read-only by tool
grant (`./agent-roster.md` § reviewer isolation); none runs in a worktree of its own.

Small change: {{CORRECTNESS}} only. Component: {{CORRECTNESS}} ∥ {{PATTERNS}} (parallel).
{{CORRECTNESS}} *executes* — runs the tests and, for a Bug, the reproduction — and pastes the
output; a correctness claim about running code is settled by running it, not by reading the diff
(consensus is not verification). Address by severity; re-verify fixes yourself by re-running.

## Phase 7 — Close-out
Fill the ticket Outcome (one line / Before·After·shipped) and the Evidence block per criterion —
each oracle's command and its actual green output. Remove the row from `WORK-ITEMS.md` on the
ticket branch; a filled Outcome and a live registry row never coexist. Commit at a clean point —
the pre-commit hook proves it green.

## Phase 8 — Integrate
{{PM}} (if used): relay a 3-5 sentence product-impact summary.

**Runtime run (factor 16).** If this change touches behavior, run the runtime harness (or the
recorded runbook) and paste the report line into the ticket's Evidence before landing; a red run is
a stop-and-ask trigger, never a skip.

Then:
{{POSTURE=local}}: if the gate is green and main is clean for the touched files, merge with
`git merge --ff-only <slug>`; confirm in one line; tear down the worktree.
{{POSTURE=remote}}: push and open a PR with the Outcome as the body; let CI + review gate it.
**Stop and ask only on a surprise:** the merge would refuse; an unresolved review block; a
design-shape decision made mid-ticket; a preexisting bug surfaced; anything else needing
approval.

## Phase 9 — Retrospective (always run)
Inspect the session: the ticket, CLAUDE.md, rules, the gate, the reviewers, any friction.
Surface only items with a named consequence. Obvious in-scope fix → land it via a branch +
ff-only (never a direct main commit). Larger → write a ticket. Nothing serious → say so.

**Escaped defect (factors 13, 15, 16).** A bug that passed a green gate is a harness defect, not only a
code defect. Before this retrospective closes, land the fixture, rule, gate, or harness step that would
have caught the **class** — or write the test and delete the story (`./rules/evidence.md` § incidents).
Name the class (fixture homogeneity, an untested default path, a repair more permissive than its input),
not the instance; a second bug of the same class after this is a retrospective that did not run.

## Phase 10 — Next ticket (always run, last)
Read `WORK-ITEMS.md`; recommend the single next actionable ticket (lowest open phase, skipping
blocked/deferred). Print the exact command to run it.
```

---

## Plan-break pivot protocol (factors 8, 11)

When the design is proven **wrong** — the adversary returns `BREAKS (plan)`, or a spike/review shows
the approach cannot produce the ticket's Verification — the response is fixed:

1. **Do not patch to the counterexample.** A fix shaped to the last failing case is what makes the next
   pass find the next one (an unbounded loop once ran to eleven passes). Step back to staff level: 2–4
   approaches, mechanism + tradeoff each, class-first.
2. **Get a second opinion from a different model than the judgment seats**, where the install offers
   one, so it widens the option space instead of confirming the blind spot that produced the break.
   Its output is claims to ground, not verdicts; it never approves, and it is read-only with no shell.
3. **One retry.** The human authorizes the pivot — AskUserQuestion surfaces the design change before
   the rewrite. Pick a genuinely different approach, drive it, and re-run the adversary/spike. It
   SURVIVES → continue. It `BREAKS (plan)` a **second** time → **park/demote**: keep the problem, the
   evidence, and both counterexamples as the record; cut the dead plans. Two broken designs means the
   problem is not understood, and a third pass will not understand it either.

An **evidence** break (a count that won't reproduce, a moved citation) is not a broken design — repair
the record and continue; it does not consume the one retry.

## Notes

- **Drop phases the project doesn't need**, but know what each secures: 0/8 → factor 9, 1 →
  factors 1, 2, 5, 11, 15 (ticket completeness, per-ticket research, ADR on durable decisions, the
  approach-pass human touchpoint, Bug intake), 2/6 → factor 8, 3 → factor 3 (red-first),
  2-adversary → factor 8 pre-code, 3-drive → factor 16, 5 → factor 6, 5b → factor 14,
  6-execute → factor 8 continuation, 8-runtime-run → factor 16, 8-triggers → factor 11,
  9 → factor 13 (+ the escaped-defect clause → 15, 16). Dropping a phase drops a factor — do it
  consciously, not by omission.
- **Parallel reviews** are a single message with multiple agent spawns — this protects the
  orchestrator's context and wall-clock.
- **Hooks are the gate**, not this skill; `./hooks.md` installs them.
- **Optional — a phase-exit audit at roadmap boundaries.** On a project with a phased roadmap, add a
  checkpoint audit as the last ticket of each phase: fan out one reviewer per dimension over the WHOLE
  tree (not a diff), adversarially verify every finding (default-refute), rank by impact (never by
  LOC/time), route dispositions. A roadmap phase advances only after its exit audit closes. Template:
  `./TEMPLATE-phase-audit.md`. Orthogonal to the per-ticket phases above — it audits the accumulated
  tree, not one change.
