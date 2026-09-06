# Exemplar — the findings bar (the review-finding standard)

**Adapt, do not copy verbatim.** Realizes factor 8 (independent review) at the *output* layer: one
definition of "important" that every reviewer applies, so passes calibrate against each other instead
of each inventing a bar. **Loaded on demand** by the review seats — each agent's prompt says "read
`findings-bar.md`" — not on every main-agent turn; it is paid for only when a review runs. Keep the
named-consequence filter, the three-field format, and the late-finding calibration rule; map the lens
names to your roster (`./agent-roster.md`). The generated file is `.claude/findings-bar.md` — not under
`agents/`, where it would register as an agent.

## The bar — a finding qualifies only with a named production consequence

State the specific production effect, or cut the finding. Qualifying consequences:

- incorrect behavior, data loss, security exposure;
- performance degradation at a realistic baseline (e.g. the 5-year power-user scale the architecture
  targets) — not a micro-optimization;
- compounding tech debt — a stale comment, dead code, or misleading name that leads a future reader
  to a wrong conclusion;
- a decision-completeness gap that blocks execution (a TBD, a missing API name, a partial sibling
  enumeration);
- citation drift — a `file:line` or quoted-mechanism claim that does not match the cited file;
- behavior-claim drift — a load-bearing claim about framework/OS/library runtime behavior stated
  without an executable probe (command + output) or a primary source. The fix is to run it or cite
  the source, never to re-read the prose: a same-model reviewer shares the author's prior and will
  confirm a plausible-but-wrong claim. Flag it `[UNVERIFIED]` — verify-or-cut before approval.

**Skip:** style preferences with no correctness consequence, edge cases the contract excludes,
behavior-neutral refactors, anything phrased "could be cleaner" / "for symmetry" without naming what
breaks. When unsure, state a question, not a recommendation.

## Required finding format — three fields on every finding

| Field | Values | Purpose |
|---|---|---|
| `TAG:` | `[load-bearing]` / `[induced]` / `[polish]` | severity against the bar |
| `LENS:` | the reviewer domain that owned the catch | cross-pass calibration |
| `CONSEQUENCE:` | one sentence naming the specific production effect | forces the named-consequence filter at output time |

**Tags:**
- **`[load-bearing]`** — fixes a named production consequence. Must close before approval.
- **`[induced]`** — surfaced only because an earlier-pass fix *this round* introduced it (the flagged
  code/text did not exist before review started). Still close it; the tag is a diagnostic that the
  fix loop is creating its own surface.
- **`[polish]`** — no named consequence. Skip unless polish was explicitly requested.

## Lens = whose domain should have caught it, not which pass surfaced it

The lens names a reviewer *domain* (map to your roster — planner, architecture, fact-check,
correctness, patterns, impl-drift), regardless of which pass actually found it. That attribution is
the calibration signal:

**Late-finding calibration.** If a late pass surfaces a `[load-bearing]` finding whose `LENS:` is an
*earlier* pass's domain, that earlier pass drifted — the fix is to tighten that agent's prompt, not
just to close the finding. A late pass returning mostly `[polish]` means the bar held (done). A late
pass returning mostly `[induced]` means the fix loop itself is the bug — narrow what each fix is
allowed to touch.

## Why one bar → convergence (a result, not a target)

The same bar across N lenses makes late passes surface fewer items, because earlier passes already
closed everything that crossed it. Convergence is the *evidence* the bar is consistent — do not chase
it by loosening late passes. A worked example calibrates faster than a definition: keep 2-3 real
findings in-repo, each with its consequence and fix, as the bar's reference points.
