# Changelog

All notable changes to this plugin are recorded here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project uses [semantic versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] — 2026-09-05

A major version: the factor count, the ticket template, and the `/work` phase contract all changed.

### Added

- Four success factors: **14 Test-strength** (a mutation or revert check, kept out of the fast gate, proves the suite can go red), **15 Defect intake — the definition of a bug** (a typed defect carries its named consequence, a reproduction, and a red-first regression test before any fix), **16 Runtime proof — drive the real system** (a behavioral claim is proven by driving the real system, never by citing code), and **17 Performance measure** (present-when-applicable: a repeatable benchmark with a threshold that fails the build on regression, or a justified N/A).
- Acceptance criteria bound to red oracles: every ticket carries numbered criteria, each bound to a named test or runtime-harness step that was seen failing before the work began; a criterion with no oracle, or one already green, stops the ticket.
- A definition of done, recorded per dimension in `adr-003-definition-of-done`, seeded at init alongside the architecture and quality-gate ADRs.
- A `Bug` ticket type.
- An always-on evidence rule (`.claude/rules/evidence.md`): drive, don't cite; a negative from an unvalidated instrument is worth nothing; an irreversible guard is proven in both directions; an incident becomes a test, not a war story.
- A runtime-harness exemplar with its two self-proofs (`--negative-control`, `--twice`), anti-vacuity assertions, and interlocks on destructive paths; per-stack driver table in the ecosystem profiles.
- A `ticket-adversary` reviewer seat that tries to break the ticket before it is built and returns `BREAKS (evidence)` / `BREAKS (plan)` / `SURVIVES`.
- An executing correctness reviewer: at least one post-implementation reviewer runs the tests and pastes the output rather than reading the diff.
- The plan-break pivot protocol: on `BREAKS (plan)`, step back to 2–4 approaches; the human authorizes one retry; a second break parks the ticket instead of looping.
- The context-scoping sub-audit (checklist 10a–10c) and an instruction-placement exemplar: what belongs in CLAUDE.md, a path-scoped rule, the `/work` skill, or an agent definition, given how Claude Code assembles subagent context.
- A phase-exit audit ticket template and a review-findings bar.
- New rule exemplars: `coverage-derives-its-domain` and `symbol-deletion`.

### Changed

- 13 → 17 success factors; `success-factors.md`, the checklist, the interview, and the ecosystem profiles extended accordingly (continuation batch, performance batch, runtime-driver rows).
- The fast `verify` gate is one condition of done, not the whole definition: a ticket closes on evidence per criterion, a green gate, a runtime run when behavior was touched, and recorded reviewer verdicts.
- README Requirements now recommends an Opus-class model or higher for the `/greenfield` run itself.
- Reviewer isolation is by tool grant (`Read, Grep, Glob`, plus `Bash` for the executing reviewer; never `Write`/`Edit`, never a sandbox worktree), with the diff frozen as a commit before reviewers spawn.

## [1.1.0] — 2026-06-24

### Added

- SKILL.md: recommend a phase-gate audit — at the end of each roadmap phase (not each ticket), seed a ticket that runs a deep multi-agent ("ultracode") audit across architecture, code correctness, security, and performance, to catch cross-cutting drift the per-ticket loop can miss.

### Changed

- README: add an example `/work` pipeline diagram (Mermaid) that labels the reviewer subagent for each phase and shows the parallel research and review passes, and clarify that the generated workflow is customizable.

## [1.0.0] — 2026-06-23

### Added

- Initial release of the `greenfield` skill: a planning-and-guardrail harness that secures all 13 success factors for agentic coding projects with concrete artifacts (decision-complete tickets, a hook-enforced `verify` gate, independent reviewer subagents, ADRs, path-scoped rules, and a `/work` command). Two modes: **init** (scaffold a new project) and **audit** (review an existing repo and install only what's missing). Ships its own exemplars and supplements, so it depends on no other skill.
