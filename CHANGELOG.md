# Changelog

All notable changes to this plugin are recorded here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project uses [semantic versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-06-23

### Added

- Initial release of the `greenfield` skill: a planning-and-guardrail harness that secures all 13 success factors for agentic coding projects with concrete artifacts (decision-complete tickets, a hook-enforced `verify` gate, independent reviewer subagents, ADRs, path-scoped rules, and a `/work` command). Two modes: **init** (scaffold a new project) and **audit** (review an existing repo and install only what's missing). Ships its own exemplars and supplements, so it depends on no other skill.
