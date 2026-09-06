# Exemplar — ADR template

**Adapt, do not copy verbatim.** Realizes factor 5 (decision provenance). Portable.

An ADR records one architecture-level decision so the next agent does not relitigate it or
break the invariant it created. The generated template file is `docs/adr/TEMPLATE.md`; ADRs are
`docs/adr/adr-NNN-slug.md`, monotonic, append-only (supersede with a new ADR; never rewrite history).

---

```markdown
# ADR-NNN: <decision title>

**Status:** Accepted | Proposed | Superseded by ADR-NNN   ·   **Date:** <YYYY-MM-DD>

## Context
The problem, the constraints, and the prior art. Why a decision is needed now.

## Decision
The choice made and its technical justification. Be specific enough that the gate, a rule,
or a reviewer can be derived from it. Cite sources for non-obvious claims.

## Consequences
What this makes easy, what it costs, and the invariants it now imposes (what later changes
must not break).

## Alternatives considered
Each option evaluated and why it was rejected — one or two lines each.
```

---

## When ADRs are created

Three moments:
1. **Init** — `adr-001` (architecture + boundary), `adr-002` (gates), and `adr-003` (definition
   of done) are seeded from the interview decisions.
2. **Ticket prep — the ongoing source.** A Component ticket's approach pass (`/work` Phase 1)
   that yields a durable, cross-ticket decision writes an ADR before execution; the ticket sets
   `ADR: creates adr-NNN`. A Component ticket implementing within existing ADRs cites them
   (`ADR: cites adr-NNN`) and writes none.
3. **Retrospective (`/work` Phase 9)** — backfill an ADR for any design-shape decision made
   mid-implementation, so the record matches reality.

Not every Component ticket writes an ADR — only those that make a lasting decision. The
approach pass always *happens* for a Component ticket; it produces an ADR only when there is a
durable decision to record. Small changes never touch ADRs.

## Seeds every project gets

- **`adr-001-architecture.md`** — the architecture shape and the one boundary the build
  enforces (factor 7), and the performance-sensitive surface or its justified N/A (factor 17). The
  arch-enforcement tool is derived from this ADR.
- **`adr-002-quality-gates.md`** — what `verify` runs, why those checks, and that it is
  hook-enforced (factor 6) — one condition of done, not the whole definition (`adr-003` holds that);
  the test-strength check (factor 14) and the performance benchmark (factor 17, where applicable) that
  run *separately* from the fast gate; and the runtime harness (factor 16) — built now, or deferred to
  the first ticket whose claim is about the running system, with the scripted-runbook fallback named.
- **`adr-003-definition-of-done.md`** — the definition of a bug (factor 15) and a per-dimension
  "done means" table (Bugs / Security / Performance / Architecture → who holds it afterwards), so
  "fixed" and "done" are decidable, not vibes.

## Audit: migrating inline decisions

Existing projects often carry decisions inline in CLAUDE.md or a market/design doc. Migrate
each into an ADR: lift the context and rationale, name the alternatives that were weighed, and
leave a one-line pointer where the decision used to live. Preserve the original date if known.
