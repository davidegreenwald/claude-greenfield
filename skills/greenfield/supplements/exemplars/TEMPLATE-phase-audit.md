# Exemplar — phase-exit audit ticket (TEMPLATE)

**Adapt, do not copy verbatim.** An OPTIONAL end-of-phase quality gate for a project with a phased
roadmap. At a phase boundary — every other ticket in the phase closed, `main` green — this ticket
fans out one reviewer per dimension over the **whole tree** (not a diff), adversarially verifies every
finding (default-refute), ranks by impact, and routes dispositions. **A roadmap phase advances only
after its exit audit closes.** Drop it on a project without a phased roadmap. Genericize the gate
commands and dimensions.

The fan-out is one parallel spawn from `/work`: one reviewer agent per dimension, then a refute pass
per finding. Fail loud — spawn no dimension agent — when the phase is genuinely absent, rather than
emit a plausible-but-wrong "current-phase" artifact.

---

```markdown
# T-NNN: Phase N exit audit

**Type:** Phase audit  ·  **Phase:** N

| Started | Completed |
|---------|-----------|

## Goal
No direct data effect — close roadmap Phase N clean before Phase N+1 builds on it. A **checkpoint
audit**: the fan-out reviews the whole codebase as it stands; the phase boundary is the trigger, not
the body of work audited.

## Precondition
Every other Phase N ticket is closed (its registry row removed) and `main` is green. This ticket is
always the last one in the phase. If a Phase N ticket is still Blocked/Deferred, note why it is
carried forward rather than audited around.

## Scope
Full tree — the whole codebase on `main` HEAD. No diff range, no tags: the deterministic gates
already run over current state, and the dimension agents read the full tree (`git ls-files …`). Each
phase audit re-reviews everything; record accept/wontfix dispositions in Outcome so a later audit's
repeat findings cross-check against prior rulings.

## Run
Fan out one reviewer per dimension over the full tree in a single parallel spawn from `/work`. It
inventories the tree, runs the deterministic gates
(verify / security / provenance), fans out the dimension agents, **adversarially verifies each
finding — default to not-real unless re-reading the source confirms it** — and returns a ranked,
deduped summary.

Dimensions (adapt): **correctness** · **architecture** (layering/isolation, blast radius) ·
**patterns/rules conformance** · **performance** (at scale) · **security** ·
**provenance/data-integrity** (if the project has a derived-data layer) · **loose-ends** (deferred
TODOs, blockers now cleared, schema-doc drift, uncaptured retro lessons, dead code).

## Findings → routing
Summarize the ranked findings in chat (each `[severity] dimension — claim — file:line`), then get the
owner's per-finding direction. **Rank by impact — never by lines-of-code or time.**
- **Fix in this ticket** — the fix rides this ticket's commit.
- **New ticket** — register a retro ticket (`(retro from Phase N audit)` in the commit), deferred or
  scheduled per the owner.
- **Accept / wontfix** — record the rationale in Outcome.

Do not auto-file tickets or auto-apply fixes without direction.

## Close
Route every finding (fixed / ticketed / accepted); fill Outcome; update the project status line if the
boundary changes project status. The phase advances only now — after this ticket closes.

## Outcome
_Filled at close. Findings by disposition (fixed / ticketed / accepted) and the go-ahead for Phase N+1._
```
