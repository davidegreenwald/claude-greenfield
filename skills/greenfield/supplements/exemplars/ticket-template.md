# Exemplar — ticket template

**Adapt, do not copy verbatim.** This illustrates one good realization of factors 1-3 plus the
continuation forms of 2/3, the defect intake of factor 15, and the runtime proof of factor 16
(decision-completeness, evidence over recall, specify-first — with numbered acceptance criteria bound to
red oracles, a rerunnable Evidence block, red-before-green, a `Bug` variant, and an `Irreversible sink` row),
generalized from a decision-complete ticket template. Drop rows a stack doesn't need; keep
the shape. The whole point is that a filled ticket has no "TBD" left for implementation.

The generated file is `tickets/TEMPLATE.md`. Tickets are `tickets/T-NNN-slug.md`.

---

```markdown
# T-NNN: <title>

**Type:** Small change | Component | Bug   ·   **Phase:** <N | infra | design>   ·   **Depends on:** <T-NNN, …>   ·   **ADR:** creates adr-NNN | cites adr-NNN | none

| Started | Completed |
| <date> | <date> |

## Goal
What this delivers and its user/product/data impact. For infra-only work: "No direct
user effect — <downstream impact>".

## Acceptance criteria
Numbered, `C1…Cn`. Each names its **oracle**: the test, or for a claim about the running system the
runtime-harness step, that proves it. The oracle is written from the criterion, not from the plan, and
is seen RED before implementation (Verification records the red). A criterion with no oracle, or one
already green on the current tree, is a gap, not a detail.

- **C1** — <criterion> · oracle: `<test path or harness step>`
- **C2** — …

**Done when:** every criterion's oracle is green with its command + output in Evidence; the gate is
green; the runtime run is green if a behavior path changed; every reviewer seat's verdict is recorded.

## Research
Primary sources and best practice validated before execution. Cite a doc URL, a repo
path, or a saved research note. An unvalidated approach is a research task that finishes
before review — never a "TBD" left in the plan. For a Component ticket this is produced by
the `/work` Phase 1 research fan-out (narrow, per-ticket — the specific algorithm/API/library,
not the whole stack); a Small change needs at most one targeted lookup.

## Technical Plan
For each change:
- **What:** the files / functions / tables / config that change.
- **Why it works:** the mechanism, 1-2 sentences.
- **Proven / Novel:** `proven — matches <file:line>` OR `novel — validated via <doc/repo/research url>`
  OR `novel — driven via <probe + output>` (ours, no precedent, driven against the real thing —
  `.claude/rules/evidence.md`). Recall is not a fourth tag; an untagged change does not ship.

Schema/interface changes name the exact fields and the migration. CLI/UI changes show the
command/flags or the screen/state.

## Blast radius
Fill the rows that apply; delete the rest.
- **Signature change** → every call site (production + test).
- **Schema / data change** → all readers + a schema snapshot test + the migration number.
- **Boundary / interface change** → all implementers + the shared contract test.
- **Architecture** → the layering/dependency contract; `<arch-enforcement command>`.
- **Irreversible sink** → any operation the change can reach that cannot be undone (a delete, a
  migration, an external write, a wipe): name it, the guard on it, and the input the ticket did NOT
  stage. A change that reaches one gets a design spike, the strongest review tier, and the adversary
  starts here. An empty row on a change that writes user data is itself a finding.
- **<domain-specific>** → e.g. provenance/audit, accessibility, API min-version — keep only if real.

## Verification (specify-first)
- **The red oracles, one per criterion:** name each by path, the command that runs it, and the red
  output it produced *before* any implementation existed. Write it from the acceptance criterion, not
  the plan — a test never seen to fail is a tautology. (For a claim about the running system, the red
  artifact is a runtime-harness step instead — same rule, different harness; building the harness is in
  scope when none exists.)
- **Commands:** `<verify gate command>` (must pass green after).
- **<Data/quality checks>:** counts (with the query), audits, EXPLAIN on hot paths — keep only if real.

## Evidence
The commands that produced this ticket's claims and the output they ACTUALLY produced — not a
summary of it, the output, so the next reader re-runs instead of believing. Every numeric claim
(X files, N calls) lives in a block here, or it is a draft, not a ticket.

```
$ <the exact invocation>
<the actual output>
```

## Decisions
Open questions resolved during planning, with rationale. (Optional.) For a Component ticket,
record the approaches the Phase 1 approach pass weighed and why the chosen one won; if that
choice is a durable cross-ticket decision, it becomes an ADR instead and the `ADR:` field
points to it.

## Outcome
Filled at close. Small change: one line. Component: Before / After / what shipped, with
test output or counts as evidence.
```

---

## The Bug variant (factor 15 — the definition of a bug)

A `Type: Bug` ticket adds three required fields before any fix is written — this is the definition
of a bug:
- **Consequence:** the observable wrong behavior (data loss, incorrect output, crash), in one line.
- **Reproduction:** the exact steps or input that trigger it.
- **Red regression test:** named by path, with the red output it produced *before* the fix. Do not
  start the fix until this test exists and fails.

The Goal states the consequence; Verification's red test is the regression test; Evidence carries its
red-then-green output. A mature project may fold these into the generic ticket via a defect admission
criterion instead of a separate type — keep whichever form the project uses.

## Adapting by stack

- **UI / product app:** add a UX row to Blast radius (affected screens, the user's choices,
  the before/after state); keep Verification's scenario list as user-journey steps.
- **Library / CLI:** Blast radius centers on the public API surface and call sites; Verification
  centers on the contract tests.
- **Data/backend:** keep the data-quality/provenance rows (a data pipeline's domain block); they
  are the domain-specific block other stacks delete.
- **Ticket types:** `Small change` (single-purpose, obvious decisions, one-line Outcome),
  `Component`/`Feature` (multi-faceted, needs the full template), and `Bug` (a defect — adds the
  three intake fields above). The `/work` skill branches on this — keep the vocabulary the project uses.
