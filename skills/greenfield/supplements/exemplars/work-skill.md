# Exemplar — the project's `/work` skill

**Adapt, do not copy verbatim.** Realizes factors 8, 9, 11, 13. Generalized from a 10-phase
refined `/work` pipeline. Author the real file by filling the template below — replace every
`{{placeholder}}` from the interview decision sheet.

The project's `/work` skill is **project-level** (`<project>/.claude/skills/work/`), and is the
execution pipeline that ties the harness together: it spawns the reviewer roster, respects the
gate, and integrates work safely. It does not run the gate itself (the hook does) and it does
not auto-fire (`disable-model-invocation: true`).

Placeholders to fill:
- `{{GATE}}` — the verify command (e.g. `make verify`, `npm run check`).
- `{{POSTURE}}` — `local` (worktree + `git merge --ff-only`) or `remote` (branch + PR + CI).
- `{{SHAPES}}` — the two ticket shapes (`Small change | Component`).
- `{{PLANNER}}`, `{{FACTCHECK}}`, `{{ARCH_REVIEWER}}`, `{{CORRECTNESS}}`, `{{PATTERNS}}`,
  `{{PM}}` — the chosen roster agent names (omit any the project doesn't use).

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
{{POSTURE=local}}: `git worktree` (or `make worktree BRANCH=<slug>`) into an isolated checkout.
{{POSTURE=remote}}: `git checkout -b <slug>`. (Bootstrap exception: branch in place if there is
no commit history yet.)

## Phase 1 — Ticket shape & prep
Classify {{SHAPES}}. A single-purpose obvious change is a Small change; anything else is a
Component and needs a ticket written from `tickets/TEMPLATE.md` first (decision-complete,
registered in `WORK-ITEMS.md`).

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
Small change: {{FACTCHECK}} only. Component: {{FACTCHECK}} ∥ {{ARCH_REVIEWER}} (parallel).
Resolve any BLOCK or unverified citation before Phase 3.

## Phase 3 — Execute
Work the decision-complete ticket; execution is mechanical. Write the ticket's
`scenario → expected` cases as test stubs first, then make each pass. Adjacent bug fixes are
fine; design-shape changes are not (halt, resolve in the ticket, resume).

## Phase 4 — Self-check
Re-read your own diff against the ticket's Blast radius. What could be silently wrong? What
drifted from the plan? Any claim without a `file:line`/url? Fix; cite; skip nits.

## Phase 5 — Mechanical gate
{{GATE}} runs at commit time (pre-commit hook) and turn-end (Stop hook). Do not invoke it
here. For a mid-implementation spot check, run a single sub-gate.

## Phase 6 — Post-implementation review
Small change: {{CORRECTNESS}} only. Component: {{CORRECTNESS}} ∥ {{PATTERNS}} (parallel).
Address by severity; re-verify fixes yourself.

## Phase 7 — Close-out
Fill the ticket Outcome (one line / Before·After·shipped). Remove the row from `WORK-ITEMS.md`
on the ticket branch. Commit at a clean point — the pre-commit hook proves it green.

## Phase 8 — Integrate
{{PM}} (if used): relay a 3-5 sentence product-impact summary. Then:
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

## Phase 10 — Next ticket (always run, last)
Read `WORK-ITEMS.md`; recommend the single next actionable ticket (lowest open phase, skipping
blocked/deferred). Print the exact command to run it.
```

---

## Notes

- **Drop phases the project doesn't need**, but know what each secures: 0/8 → factor 9, 1 →
  factors 1, 2, 5, 11 (ticket completeness, per-ticket research, ADR on durable decisions, the
  approach-pass human touchpoint), 2/6 → factor 8, 5 → factor 6, 8-triggers → factor 11, 9 →
  factor 13. Dropping a phase drops a factor — do it consciously, not by omission.
- **Parallel reviews** are a single message with multiple agent spawns — this protects the
  orchestrator's context and wall-clock.
- **Hooks are the gate**, not this skill; `./hooks.md` installs them.
