# The interview — resolve the runtime details

The interview turns the principles into a project-specific plan. It resolves the variable
parts of the harness so generation is mechanical. Reach decision-completeness here — the same
bar the harness will later demand of every ticket.

## Rules for the interview

- **Research first.** The architecture, gate, and tooling options you present come from the
  research round (Process step 2), not recall. Each technical option carries the source that
  backs it.
- **Infer before asking.** In audit, profiling the target path answers most of this
  (language, build/test/lint, git posture, source layout). Confirm inferences in one line;
  ask only what the repo cannot tell you. In init, ask the identity and stack questions
  outright.
- **Group questions; don't interrogate.** Batch related decisions into a single
  AskUserQuestion call (2-4 questions). Never ask what you can read or sensibly default.
- **Every question through AskUserQuestion — except purpose.** Deliver every question as
  AskUserQuestion options, never a free-form prose prompt. The lone exception is purpose, asked
  as a standalone free-text prompt (it has no natural options). For the other open-ended answers
  (target path, stack when undecided), still use AskUserQuestion: offer the best 2-4 inferred
  candidates as options and let the user type a custom value in the always-present "Other" field.
- **Recommend, with a reason and a source.** This is a staff-engineer skill — guide the
  decision. For any technical choice (architecture, persistence, gate tool, boundary), put the
  staff-engineer pick first, append `(recommended)` to its label, and give a one-line why plus
  the primary source. The user can take it ("sure thing") or offer their own.

## Decision domains

Resolve each. The right-hand column is the default to propose unless the project says
otherwise.

| Domain | What to resolve | Sensible default |
|--------|-----------------|------------------|
| Location | target directory / repo path | ask first (init: where to scaffold, create if absent; audit: which repo); a `/greenfield <path>` arg is the default |
| Identity | name, one-line purpose, domain | derive name from the path; ask purpose (init); infer from the repo (audit) |
| Stack | language, package manager, build, test, lint/format | infer from manifest (audit); ask (init) |
| Architecture | the named style + the one boundary to enforce, each with sources | the researched recommendation; confirm or override |
| Gate | the single `verify` command | reuse existing script; else compose from stack |
| Git posture | local-only vs remote/PR | infer from `git remote`; local → worktree+ff-only |
| Reviewer roster | which lenses + model tiers | the 6-archetype set in `./exemplars/agent-roster.md`, trimmed to the project |
| Rules | which path-scoped rules to seed | always-on (shell, evidence) + one per boundary |
| Scale knobs | phases/roadmap? design track? per-module CLAUDE.md? | size to module count; all factors still present |

## Question groupings (adapt wording)

Run these as AskUserQuestion batches (2-4 questions each), skipping anything already inferred —
every batch is an AskUserQuestion call, never a prose prompt, except purpose, which is a
free-text prompt. Phrase options concretely; put the recommended option first and label it. Ask
the project-shape batch (path, purpose, stack)
before the research round, since research keys off the stack; ask the architecture and tooling
batches after research has produced the candidates.

**Batch A — project shape** (ask first, before research; skip inferred items)
- Target path: "Where should this live?" — init: the directory to scaffold (create it if it
  doesn't exist); audit: the repo to review. Ask this first; profiling, research, and
  generation all key off it. Options: the `/greenfield <path>` arg `(recommended)` plus 2-3
  sensible parent locations; the user types a custom path in "Other".
- Purpose: "One line — what does this project do, and who is it for?" Ask as a standalone
  free-text prompt, not an AskUserQuestion — it has no natural options. Path and stack stay as
  AskUserQuestion in this batch.
- Stack (init; inferred from the manifest in audit): "What's the stack?" — language + runtime +
  framework. Options: the most likely candidates for the project shape (recommended first),
  plus "Other" for anything undecided or custom. Resolve before research — research keys off it.

**Batch B — architecture + boundary** (after research)
- Architecture style: present the 2-4 researched candidates by name (layered · pipeline/filters ·
  MVVM · hexagonal / ports-and-adapters · modular monolith · lib-core + thin-bin · …). **Name the
  recommended style, give the one-line why, and cite the source(s)** that recommend it for this
  stack and domain, so the user understands what will constrain the project and can accept or counter.
- The boundary to enforce (factor 7): "Which dependency rule should the build refuse to break?"
  e.g. "pure core imports no framework", "UI imports no persistence", "stages don't import each
  other". The chosen style + its sources + this boundary seed `adr-001` and the arch-enforcement tool.

**Batch C — gate + git** (mostly inferable in audit)
- Gate command: confirm the reused/composed `verify` (show the exact command).
- Git posture: local-only (worktree per ticket, `git merge --ff-only`) vs remote (branch + PR
  + CI). Drives `/work` Phase 0 and Phase 8, and whether merge is auto-on-green or PR-gated.
- Hook enforcement: confirm commit-time gate + turn-end backstop + format-on-write are wanted
  (default yes; the gate is only as real as its enforcement).

**Batch D — reviewer roster + tiers**
- Which reviewers, from the archetype library (`./exemplars/agent-roster.md`): always include
  a decision-completeness checker, a correctness reviewer, and a pattern/consistency auditor;
  add a citations/fact-checker if the project makes external claims, a product-lens explainer
  for user-facing work, and domain reviewers (security, perf, a11y, API-compat) as the domain
  demands.
- Model tiers: judgment reviewers (architecture, correctness) on the strongest available tier;
  mechanical reviewers (completeness, citations, conformance, product-lens) on a mid tier.

**Batch E — scale knobs** (only if non-obvious)
- Phase roadmap / `PLAN.md`? (large multi-phase build → yes; single-purpose tool → no)
- Non-shipping design/prototype track? (UI-bearing product → maybe)
- Per-module CLAUDE.md? (many modules → yes; flat small repo → no)

## Output of the interview

A filled decision sheet — every domain resolved, no "TBD" — ready to drive generation. Restate
it back in a compact block (stack, gate, posture, architecture style + sources, boundary,
roster+tiers, rules), then emit the would-generate manifest (the files and their one-line
purpose) and get a go-ahead before writing. Declining the manifest writes nothing — that is the
no-write preview.

## Audit specifics

- Lead with the audit (`./checklist.md`) so the user sees present / partial / absent before any
  question — most questions collapse to "confirm this inference."
- Surface conflicts explicitly: an existing convention that contradicts a default (e.g. their
  lint config already enforces a different boundary) wins; adapt the harness to it.
- Migrate, don't invent: existing inline decisions become ADRs; existing backlog/scratch notes
  become tickets; an existing gate script becomes the gate.
