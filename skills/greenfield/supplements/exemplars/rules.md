# Exemplar — `.claude/rules/*` path-scoped rules

**Adapt, do not copy verbatim.** Realizes factors 3, 10, 14, 15, 16 (specify-first, progressive
disclosure, test-strength, defect intake, runtime proof). Keep each rule short and enforced; derive its wording
from the ADR it secures.

A rule is a short, enforced checklist that loads when its `paths:` glob matches the file being
touched — the constraint arrives exactly when it applies, instead of sitting in a monolithic
CLAUDE.md the agent skims once. Omit `paths:` for an always-on rule.

**Subagent caveat.** Path-scoped rules govern the *main session*; they are not listed among what
a subagent inherits (Claude Code docs, `sub-agents.md`), so treat them as main-session-only until
confirmed. A constraint a reviewer or worker subagent must honor has to live in that agent's
definition or a skill in its `skills:` list — mirror it there, don't rely on the rule reaching it.
For the full CLAUDE.md / skill / agent / rule placement decision, see `./instruction-placement.md`.

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

## The rules every project gets (always-on)

- **shell-hygiene** — the block above. Portable as-is.
- **evidence** — a behavioral claim is proven by DRIVING the code, never by citing it (and a plan is made
  of behavioral claims); a negative from an unvalidated instrument — including a reviewer's — is worth
  nothing; a guard on an irreversible operation is proven in both directions; the three evidence tags
  (`proven — matches` / `novel — validated via` / `novel — driven via`), recall is not a fourth; counts
  come from a tool with the command inline. `./rules/evidence.md`. (Factors 2 and 16 at the always-on
  layer — and mirrored into every reviewer/worker seat, since rules do not reach subagents.)
- **defect** — a `Bug` ticket is not started until it carries a named consequence, a reproduction,
  and a red regression test that fails first; the fix turns it green. (This encodes factor 15 — the
  definition of a bug — at the always-on layer.)
- **coverage-derives-its-domain** — any "is X exhaustive?" check derives its set from the producer's
  own output, never a hand-typed literal; the authoring test is "where does my notion of *everything*
  come from?" `./rules/coverage-derives-its-domain.md`.
- **symbol-deletion** — on an export rename/delete, grep the whole harness surface (code + rules +
  agents + configs + docs) in BOTH directions, enumerated at plan time (four checks). `./rules/symbol-deletion.md`.

## Path-scoped rules, derived from the architecture

Generate one rule per enforced boundary or hot spot. The architecture decision (`adr-001`)
tells you which. Examples by stack:

| Rule | `paths:` | Encodes |
|------|----------|---------|
| core-purity | the pure-core glob | no framework/IO imports in core; deterministic; injected clock |
| testing | `tests/**` | specify-first, **confirmed red before green**; a test must fail on revert (factor 14); determinism (fixed clock, seeded randomness); contract tests; **test the invariant, not the case list** (a list passes while the property fails); a cross-cutting change gets a **default-settings fixture** (every fixture that flips a setting off leaves the default path untested); a guard on an irreversible op is asserted in both directions with a witness |
| runtime-harness | `<harness dir>/**` | every destructive interlock and the driven bug behind it (`./runtime-harness.md`): the wipe takes no path argument; liveness by the at-risk resource, never a process name; fail-closed allowlists; the run lock never uses elapsed time as liveness; which verbs a reviewer may run |
| db / persistence | the data-access glob | migrations, strict schemas, indexes, upsert idioms |
| external-api | the client glob | retries, rate limits, error normalization, min-version handling |

Keep each rule short. If a rule grows past a screen, it is probably an ADR with a rule pointer,
not a rule.
