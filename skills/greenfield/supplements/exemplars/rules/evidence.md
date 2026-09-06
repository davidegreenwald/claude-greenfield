# Evidence — a behavioral claim is proven by driving the code, never by citing it

**Adapt, do not copy verbatim.** An always-on `.claude/rules/*` rule (no `paths:`). Realizes factor 2
at the always-on layer and factor 16 (runtime proof) at the level of *how an agent generates evidence*.
It is a rule rather than a test because it governs the production of evidence, not what the code does —
no test can make you drive a probe; that is the one case where a rule is the last resort rather than the
lazy one. Keep the three rules and the tag scheme; swap the examples for the project's own nouns.
Generalized from a production plugin's evidence rule, written after four data-destroying defects passed
fully green unit suites.

The generated file is `.claude/rules/evidence.md`. It replaces the one-line `evidence` bullet earlier
harnesses carried (claims cite `file:line`; counts come from tools) — that bullet survives here as the
fourth convention.

---

```markdown
# Evidence

Always-on.

## The three rules that prevent data loss

- **A behavioral claim is proven by DRIVING the code, never by citing it.** "The parser reads this
  back", "this round-trips", "the writer emits X" — a `file:line` citation establishes only that the
  code exists and says what you quoted. The inference from it to a *behavior* is yours, and it can be
  wrong while every citation is correct. Run the real function, paste its actual output, and state the
  probe's inputs — choosing the ones that could **falsify** the claim over the ones that confirm it.
  **This binds your PLAN as hard as your defect.** "The fix works because the parser does X" is a
  behavioral claim, and a plan is made of them. Drive each one before you build on it — and drive the
  real path, not a reconstruction of it: a loop that calls the same functions the engine calls is a
  *model* of the engine, and it will not tell you what the engine's guards do. Where the claim is
  about the running system — the app, the service, the device, the host — the probe is a runtime
  harness step (`<harness dir>/`), because neither a unit test nor a citation can establish it.
- **A NEGATIVE from an unvalidated instrument is worth nothing — and evidence HANDED to you is
  unvalidated.** A `pgrep` pattern, a grep regex, a "check X to know Y" is itself a claim. Never
  conclude *absent / clean / not running / no matches* from a probe you have never watched produce a
  **positive** on a known-true case. A guard that silently under-matches reads exactly like protection
  — and so does evidence that is empty for the wrong reason (a blank screenshot, a zero count from a
  mistyped path, a gate that never ran). The same bar applies to a number, a citation, or a *finding*
  you did not derive: a reviewer's, a subagent's, a doc comment's. Re-derive it before you repeat it or
  act on it. **A reviewer's negative is a claim, not a verdict** — theirs is an instrument you have
  never watched fire either, and it can under-match exactly like yours.
- **A guard on an IRREVERSIBLE operation gets both directions, once, in the suite** — the case that
  must fire, and the neighbour that must not. A guard has two failure modes and a positive-only probe
  sees one: it can match **nothing** (a wipe gate passing vacuously) or **too much** (a `pkill -f`
  substring that kills your own shell). Give the probe a witness, too: prove the live one is alive and
  the dead one is dead, because "excluded" and "survived" are both true of a process that never
  existed.

These are three lines and not three pages because of the rule at the bottom: the moment one of them
would grow a war story, the story becomes a test instead.

## The conventions tickets cite by name

- **Stable names in durable prose; `file:line` only for point-in-time evidence.** Comments and the
  narrative parts of docs name the function, type, or step. A line number written into a comment or a
  durable doc rots on the next edit; a name stays true. Keep `file:line` for evidence that is
  re-verified when cited — a ticket's `proven — matches` anchors and review findings. A ticket's
  Technical Plan is durable prose: it cites names, not lines, and its Evidence is re-run at execution.
- **One measurement, one number.** When a metric appears in more than one artifact (a ticket and its
  ADR, say), cite one authoritative figure and keep them in sync. A number written into a ticket
  *before* execution is an estimate — re-confirm it at execution and reconcile the durable doc, so two
  committed files never disagree about the same run.
- **The ticket evidence tag.** Every change in a ticket carries one of three (see
  `tickets/TEMPLATE.md`):
  - `proven — matches <file:line>` — the mechanism already exists in this codebase.
  - `novel — validated via <doc/repo/url>` — a primary source or a peer tool established it.
  - `novel — driven via <probe + output>` — **ours, with no precedent, DRIVEN against the real
    thing.** Paste the command and its actual output.

  The third is not a lesser tag. A design nobody has published can be correct, and driving it against
  the real API is stronger evidence than a URL — a URL says someone else's code worked in someone
  else's context, a probe says *this* works *here*. What the tags rank is the GROUND a claim stands
  on, and there are only two kinds: verified and recalled. All three of these are verified. **Recall
  is not a fourth tag** — an approach whose only support is memory is untagged, and untagged means it
  does not ship. Judge a novel design by whether its probe could have falsified it (rule 1), never by
  whether it is novel.
- **Counts come from a tool, with the command inline.** A claim without its supporting `file:line`
  or command-plus-output stays in a draft, not the ticket or PR.

## Incidents belong in the suite, not in this file

When a rule here would be a war story, **write the test and delete the story.** A test costs nothing
per turn and cannot be forgotten; a standing rule is re-read every turn, forever, and slowly turns
confidence into hesitancy. Keep this table current — every row is executable:

| Lesson | Enforced by |
|---|---|
| <the invariant an escaped bug taught> | `tests/<file>` · `<harness dir>/<step>` |
```

---

## Why the three rules, and what each one cost

The rule is short because each line was paid for. In the reference project a completeness check was
green while a plan, driven, would have deleted user records (rule 1 — the plan's own claims were
never driven); a liveness predicate matched nothing while the real process ran, so a collection wipe
would have fired against a live file (rule 2 — a negative from an instrument nobody had watched
produce a positive); and a wipe gate that could only be checked for "fires when it should" was never
checked for "does not fire when it should not" (rule 3). Each became a test in the suite the day it
was understood — which is why the rule file did not grow.

The always-on rule governs the main session. Rules are not known to reach subagents
(`../rules.md` § subagent caveat), so every reviewer and worker seat carries the three rules in its own
definition — the `Evidence discipline` block in `../agent-roster.md` § subagent definition format.
