# Exemplar — WORK-ITEMS.md registry

**Adapt, do not copy verbatim.** Realizes factor 4 (externalized memory). Portable across stacks.

The registry is the live index of open work. The ticket file is the permanent record, so a
closed item's row is **removed** — the registry stays short and shows only what is open.

---

```markdown
# Work items

Open work only. When a ticket closes, remove its row — the ticket file is the record.

| ID | Title | Status | Phase | Ticket |
|----|-------|--------|-------|--------|
| T-001 | <title> | Not started | 1 | [ticket](tickets/T-001-slug.md) |
```

The `T-001` row above is **illustrative of the format**, not seeded by init. Init writes this
file with the header and an empty table (plus `tickets/TEMPLATE.md`) and *recommends* the first
ticket; T-001 is authored later, in `/work` Phase 1, when the user starts that work.

Optional: group rows under phase headings (`## Phase 2 — <name>`) for a multi-phase project;
a small project keeps one flat table.

## Status vocabulary

Use exactly these (add `Post-launch` only if the project ships and defers):
- `Not started`
- `In progress`
- `Code complete — pending user review`
- `Needs review`
- `Blocked on <reason>`
- `Deferred <justification>`
- `Abandoned <date>`

## ID and phase scheme

- IDs are `T-NNN`, monotonic, never reused.
- `Phase` is an integer for sequenced work, or a tag (`infra`, `design`, `compliance`) for
  cross-cutting work.
- `Depends on` lives in the ticket metadata, not the registry; a blocked item carries
  `Blocked on <dependency>` in Status.

## Lifecycle

Create ticket from template → register row (`Not started`) → `/work` advances Status →
on close, fill the ticket Outcome and **remove the row** on the ticket branch so the merge
carries the closure. Never edit the registry as a post-merge commit on main.
