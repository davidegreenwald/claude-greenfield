# Exemplar — ticket template

**Adapt, do not copy verbatim.** This illustrates one good realization of factors 1-3
(decision-completeness, evidence over recall, specify-first), generalized from a
decision-complete ticket template. Drop rows a stack doesn't need; keep
the shape. The whole point is that a filled ticket has no "TBD" left for implementation.

The generated file is `tickets/TEMPLATE.md`. Tickets are `tickets/T-NNN-slug.md`.

---

```markdown
# T-NNN: <title>

**Type:** Small change | Component   ·   **Phase:** <N | infra | design>   ·   **Depends on:** <T-NNN, …>   ·   **ADR:** creates adr-NNN | cites adr-NNN | none

| Started | Completed |
| <date> | <date> |

## Goal
What this delivers and its user/product/data impact. For infra-only work: "No direct
user effect — <downstream impact>".

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
- **Proven / Novel:** `proven — matches <file:line>` OR `novel — validated via <doc/repo/research url>`.

Schema/interface changes name the exact fields and the migration. CLI/UI changes show the
command/flags or the screen/state.

## Blast radius
Fill the rows that apply; delete the rest.
- **Signature change** → every call site (production + test).
- **Schema / data change** → all readers + a schema snapshot test + the migration number.
- **Boundary / interface change** → all implementers + the shared contract test.
- **Architecture** → the layering/dependency contract; `<arch-enforcement command>`.
- **<domain-specific>** → e.g. provenance/audit, accessibility, API min-version — keep only if real.

## Verification (specify-first)
- **Test scenarios:** `scenario → expected`, written before implementation.
- **Commands:** `<verify gate command>` (must pass green).
- **<Data/quality checks>:** counts (with the query), audits, EXPLAIN on hot paths — keep only if real.

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

## Adapting by stack

- **UI / product app:** add a UX row to Blast radius (affected screens, the user's choices,
  the before/after state); keep Verification's scenario list as user-journey steps.
- **Library / CLI:** Blast radius centers on the public API surface and call sites; Verification
  centers on the contract tests.
- **Data/backend:** keep the data-quality/provenance rows (a data pipeline's domain block); they
  are the domain-specific block other stacks delete.
- **Two shapes only:** `Small change` (single-purpose, obvious decisions, one-line Outcome) vs
  `Component`/`Feature` (multi-faceted, needs the full template). The `/work` skill branches on
  this — keep the vocabulary the project uses.
