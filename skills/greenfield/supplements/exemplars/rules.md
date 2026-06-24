# Exemplar — `.claude/rules/*` path-scoped rules

**Adapt, do not copy verbatim.** Realizes factors 3 and 10 (specify-first, progressive
disclosure). Keep each rule short and enforced; derive its wording from the ADR it secures.

A rule is a short, enforced checklist that loads when its `paths:` glob matches the file being
touched — the constraint arrives exactly when it applies, instead of sitting in a monolithic
CLAUDE.md the agent skims once. Omit `paths:` for an always-on rule.

## File format

```markdown
---
paths:
  - "src/**/domain.*"      # this rule loads only for matching files
---
# <domain> rules

- Bare, enforced constraints. One per line. State the rule, not the justification.
- Cite the ADR or pattern the rule enforces, by file.
```

Always-on rule (no frontmatter `paths:`):

```markdown
# Shell & evidence hygiene

Always-on: applies to any bash command, not a file type.
- Verify state after a state-changing command; don't trust an exit code read through a pipe.
  `cmd | tail` returns tail's status — after a commit, confirm with `git log`, not `$?`.
- Don't `2>/dev/null` an exploratory query — a suppressed error reads as an empty result.
- Anchor grep patterns to the field you mean; a bare number matches timestamps and ids too.
```

## Two rules every project gets (always-on)

- **shell-hygiene** — the block above. Portable as-is.
- **evidence** — claims cite `file:line` read from source; counts come from a tool with the
  command shown inline; numeric claims without tool output stay in drafts. (This encodes factor
  2 at the always-on layer.)

## Path-scoped rules, derived from the architecture

Generate one rule per enforced boundary or hot spot. The architecture decision (`adr-001`)
tells you which. Examples by stack:

| Rule | `paths:` | Encodes |
|------|----------|---------|
| core-purity | the pure-core glob | no framework/IO imports in core; deterministic; injected clock |
| testing | `tests/**` | specify-first; determinism (fixed clock, seeded randomness); contract tests |
| db / persistence | the data-access glob | migrations, strict schemas, indexes, upsert idioms |
| external-api | the client glob | retries, rate limits, error normalization, min-version handling |

Keep each rule short. If a rule grows past a screen, it is probably an ADR with a rule pointer,
not a rule.
