# Changelog

All notable changes to this plugin are recorded here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project uses [semantic versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] — 2026-06-24

### Added

- SKILL.md: recommend a phase-gate audit — at the end of each roadmap phase (not each ticket), seed a ticket that runs a deep multi-agent ("ultracode") audit across architecture, code correctness, security, and performance, to catch cross-cutting drift the per-ticket loop can miss.

### Changed

- README: add an example `/work` pipeline diagram (Mermaid) that labels the reviewer subagent for each phase and shows the parallel research and review passes, and clarify that the generated workflow is customizable.

## [1.0.0] — 2026-06-23

### Added

- Initial release of the `greenfield` skill: a planning-and-guardrail harness that secures all 13 success factors for agentic coding projects with concrete artifacts (decision-complete tickets, a hook-enforced `verify` gate, independent reviewer subagents, ADRs, path-scoped rules, and a `/work` command). Two modes: **init** (scaffold a new project) and **audit** (review an existing repo and install only what's missing). Ships its own exemplars and supplements, so it depends on no other skill.
