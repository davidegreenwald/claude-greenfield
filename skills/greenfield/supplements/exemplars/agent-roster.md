# Exemplar — the reviewer roster

**Adapt, do not copy verbatim.** Realizes factors 8 and 12 (independent review, effort tiering).
Author each agent as a project subagent in `.claude/agents/` using the format below.

Every reviewer is **read-only on the codebase** (it never edits the diff), returns a **concise
structured verdict** (to protect the orchestrator's context), and runs **single-pass** (the
orchestrator verifies fixes itself, not by re-spawning). Each agent's prompt instructs it to report
only findings with a named production consequence — not style or behavior-neutral refactors. The
shared definition of *important* — the named-consequence bar plus the `TAG` / `LENS` / `CONSEQUENCE`
finding format every seat emits — lives in `./findings-bar.md`, loaded on demand by each reviewer (its
prompt says "read `findings-bar.md`"), not on every main-agent turn.

Read-only is not diff-reading-only. The correctness reviewer *executes* (factor 8 continuation form)
— runs the tests and, for a Bug, the reproduction — and a finding that names an execution must carry
that execution's output; where it did not execute, it reports a question, not a defect. Keep it
sandboxed: it may run the suite, not deploy or write outside a scratch dir.

## Reviewer isolation — read-only by tool restriction, never a worktree (prescriptive)

The isolation mechanism is **the tool grant, not a sandbox worktree.** This is how the reference
projects actually run reviewers — codified here so a project does not reinvent it wrongly:

- **Every reviewer carries `tools: Read, Grep, Glob`** (plus `Bash` for the one that executes) — **no
  `Write`, no `Edit`.** That restriction *is* the isolation. **Never give a reviewer `isolation: worktree`.**
  A reviewer spun up in its own worktree nests a second checkout under the repo and pollutes the primary
  checkout's own gate (a repo-root `eslint .` / `grep` walks the foreign trees), and the abandoned trees
  pile up under `.claude/worktrees/`. That drift is self-inflicted; the tool grant already isolates.
- **A repro that must WRITE runs in a throwaway scratch location** — a `/tmp` file, or a `git worktree` the
  reviewer removes in the same breath — **never a path in the diff under review, and never a persistent
  tree.** The `Bash`-carrying reviewer measures and reproduces; it does not mutate the subject.
- **The review TARGET is isolated by FREEZING THE DIFF before any reviewer is spawned.** The orchestrator
  commits the change set first (`/work` Phase 6), so every reviewer reads one stable, committed snapshot.
  Reviewers pointed at a still-mutating working tree review a tree that no longer exists — the freeze is
  what makes the parallel spawn sound.
- **A scratch probe is HANDED OVER, not deleted.** Read-only governs the *repo*, not the *findings*. Every
  probe a reviewer builds to establish a break — the scratch test, the driver, the exact command — goes into
  its report verbatim, at the path it ran from, with its actual output, so the author re-runs it rather than
  rebuilds it. A reviewer that reports only a probe's conclusion and throws the instrument away forces the
  finding to be reconstructed — and it is reconstructed with the same blind spot, which is how a bug sits
  inside the blind spot the whole time (a driven sweep and a driven live-assertion test were each deleted,
  leaving only the number; both had to be rebuilt). The instrument is the asset; the number is a souvenir.

Worktree isolation (factor 9) is for the **author's** unit of work, not for reviewers: the author works a
ticket in its own worktree; reviewers read that worktree's frozen commit read-only.

## Subagent definition format

A project agent lives at `<project>/.claude/agents/<name>.md`:

```markdown
---
name: <agent-name>
description: <role>. Use in /work Phase <N> to <purpose>. Read-only; returns <verdict shape>.
model: <opus | sonnet | haiku>  # factor 12 — the alias the tier maps to (§ the archetype library), never the tier word
tools: Read, Grep, Glob, Bash  # + WebFetch, WebSearch for a fact-checker
# effort: high                 # optional, for the hardest judgment agents
---

<Contract: exactly what to check, against what (the ticket, the diff, the rules).>

Evidence discipline:
- Drive a behavioral claim; never cite it.
- A negative from an instrument you have not watched fire on a known-true case is worth nothing.
- Paste the command and its output for every "I ran X".

Output: <the verdict format — a one-word verdict or a bounded list of findings,
each `[severity] <type> — <consequence> — <file:line>`. Cap the length.>
```

## The archetype library

Tier: **strong** = the most capable model (judgment); **mid** = a faster/cheaper model
(mechanical/citation). Map to whatever tiers the provider offers. In Claude Code the `model:` field
accepts a model alias (`sonnet`, `opus`, `haiku`, `fable`), a full model ID, or `inherit` —
https://code.claude.com/docs/en/sub-agents ; map **strong** → `opus`, **mid** → `sonnet`,
**cheapest** → `haiku` unless the project's evals say otherwise; never write the tier word itself
into `model:`.

| Archetype | Tier | Phase | Checks | Returns |
|-----------|------|-------|--------|---------|
| **ticket-planner** | mid | 1 | decision-completeness vs the template; no TBD; Research + Proven/Novel cited; `ADR:` field correct for shape (Component that moves a boundary ≠ `none`); registered | `COMPLETE` or numbered gaps |
| **plan-reviewer** *(optional, not wired by default)* | cheapest (low effort) | 2 | a fast pre-build *sniff test* of the plan: what's missing, what looks likely-wrong, any factual error — before a line is built. NOT a behavioral-correctness judgment (that is what the spike/execution proves by running); this is the cheap screen that runs *because* it is cheap | `OK` or a short list of gaps/errors |
| **fact-checker** | mid | 2 | every citation resolves; counts re-run; claims supported | per-claim verdicts |
| **architecture-reviewer** | strong | 2 (pre-code) | the categorical + blast-radius lens: names the CLASS the change instances and its live siblings (driven, not recalled); right-sizes in BOTH directions — narrow one-off where a class-fix is available AND speculative generality no Verification scenario exercises; layering/interface/schema impact | `RIGHT-SIZED`/`NARROW`/`OVER-BUILT` + the shape to ship |
| **ticket-adversary** | strong | 2 (every Component and Bug, after the planner's COMPLETE) | BREAKS the ticket before it is built: re-runs every command, re-opens every `file:line`, cross-checks Plan vs Verification, then drives the plan in a scratch worktree starting at the red oracles and attacks it | `BREAKS (evidence)` / `BREAKS (plan)` / `SURVIVES` + what it tried |
| **correctness-reviewer** | strong | 6 | line-level correctness, edge cases; **runs the tests + any Bug reproduction** (pastes output), not diff-trace alone | `APPROVE` or findings by severity |
| **patterns-auditor** | mid | 6 | conformance to patterns + path-scoped rules | `CONFORMS` or deviations |
| **pm-explainer** | mid | 8 | translate the change to user/product impact | 3-5 plain sentences |

Minimum viable roster (small project): `ticket-planner`, `ticket-adversary`, `correctness-reviewer`,
`patterns-auditor`. Add `fact-checker` when the project makes external claims, `pm-explainer`
for user-facing work, and `architecture-reviewer` once there is real architecture to protect.
Without `fact-checker`, factor 2's citation check is the adversary's duties 1–2.

`plan-reviewer` is optional and not wired into the `/work` exemplar; add it as a standing cheap
pre-build pass (a `{{PLAN_SCREEN}}` seat spawned with the Phase 2 set) when its whole value holds —
being cheap enough (the cheapest tier, low effort) to run on *every* ticket before code. It is distinct from
`ticket-planner`: the planner asks "is the ticket *decided*?" (no TBD, fields filled), the
plan-reviewer asks "is it obviously *wrong or missing something*?" (a factual/gap screen). Neither
judges behavioral correctness — that is proven by running the code (the spike, then execution), never
asserted from reading. Keep the plan-reviewer's budget small; if it starts asserting behavioral breaks
it can't run, that is the un-driven-critique trap (the expensive, low-yield failure mode) — cut it back
to factual/gap findings.

**Research agents are not roster members.** The per-ticket research fan-out in `/work` Phase 1
uses the install's generic research agent (`Explore`, or `general-purpose`), capped at 2-3
parallel for a Component ticket — do not author bespoke researcher subagents. The roster is
reviewers only; research is a built-in capability the orchestrator drives, and its findings
are what `ticket-planner` then checks for citations.

## The adversary seat — `ticket-adversary` (factor 8, pre-code)

Every other reviewer is asked whether something is sound. This seat is asked to find the case where it is
**not**, and it has failed only when it cannot find one — a ticket that survives it is a ticket someone
tried to kill. It runs in Phase 2 on every Component and every Bug, after `ticket-planner` returns
COMPLETE, because a ticket can be completely decided and wrong: the reference incident was a plan whose
Technical Plan said *strip the ref* and whose Verification, forty lines below, said *the ref stays*. It
passed a completeness check; driven, it deleted records with their review history. Nothing before this
seat was asked to run the plan and look.

**Four duties, in order.** The first three produce a record; the fourth finds bugs.
1. **Re-run every command** the ticket claims; report its number and yours side by side. A derived value
   (an offset, a `count ± N`) is recomputed by running the command, never by re-checking the arithmetic.
2. **Re-open every `file:line`.** "Close" is not cited — a citation that points at a different function
   than the claim describes is a break.
3. **Cross-check the Technical Plan against the Verification.** For each `scenario → expected`: does the
   prescribed change actually produce it? Do this *especially* when the plan looks obviously right — that is
   the plan nobody re-derived.
4. **Drive the plan, and hunt for what it makes worse.** Start at the ticket's red oracles, three checks
   that each can end the pass: is there one per acceptance criterion (none → `BREAKS (evidence)`, every
   downstream verdict would be an opinion about prose); is each actually **RED on the current tree** (a test
   already green proves nothing about the change); does it assert the **Goal**, not the plan's mechanism (a
   test written from the implementation is a tautology dressed as an oracle)? Then apply the plan as a
   scratch diff in a throwaway worktree, confirm it turns the oracles green, and attack it: feed it the
   input the ticket did **not** stage (the blind spots its own destructive-operations row lists — an empty
   row is itself a finding); ask what the change makes *worse*, not only what it fails to fix (a repair
   that no-ops is a missed repair, a repair that writes the wrong thing is data loss); for anything that
   writes user data, check the **round trip through the real reader**, never the bytes — byte-level
   invariants can all hold while the artifact stops being readable by the thing that has to read it;
   assume any recorded metadata is stale.

**The verdict is classified — the class decides the author's next move.**
- `BREAKS (evidence)` — duties 1–2. The record is wrong; the design may be right. The author repairs the
  record and re-derives it; if the *reproduction* itself does not reproduce, there is no finding and no
  ticket. Does not consume the retry.
- `BREAKS (plan)` — duties 3–4. The design is wrong. The author steps back to 2–4 approaches
  (`./work-skill.md` § plan-break pivot protocol) — never a patch aimed at the counterexample, which is
  what makes the next pass find the next one. Found both kinds? It is `BREAKS (plan)`.
- `SURVIVES` — with the list of what it tried and could not break. A `SURVIVES` with an empty list is an
  agent that did not try. An uncertain-but-real defect is a `BREAKS (evidence)` with the confidence
  stated, never a silent `SURVIVES` — a false SURVIVES ships the ticket. No suggestion lists: a suggestion
  is what a reviewer writes when it could not find a defect and did not want to say so.

**Capped at two rounds per ticket.** The author gets one retry; a second `BREAKS (plan)` demotes the ticket to
a stub that keeps the problem, the evidence, and both counterexamples as its record — the dead plan is
cut. Unbounded, this loop once ran to eleven passes without reaching code.

**Read-only — and hand the instrument over.** `tools: Read, Grep, Glob, Bash`; it never mutates the repo,
drives only in `/tmp` or a throwaway worktree it removes, and may not run the runtime harness's
destructive verbs (its `Bash` grant reaches the wipe otherwise — the harness rule names what it may run).
Read-only governs the repo, not the findings: every probe it builds — the scratch test, the driver, the
exact command — goes in the report **verbatim, with its output**, so the author re-runs it instead of
rebuilding it with the same blind spot. The instrument is the asset; the number is a souvenir.

```yaml
---
name: ticket-adversary
description: Adversarial pass on a ticket BEFORE it is built. Re-runs every command, re-opens every file:line, cross-checks Plan vs Verification, and drives the plan in a scratch worktree starting at the red oracles. Read-only; returns BREAKS (evidence) | BREAKS (plan) | SURVIVES with what it tried.
model: <strong tier>
# effort: high
tools: Read, Grep, Glob, Bash
skills:
  - <repo-local-domain-skill>   # a claim about what the platform DOES is checked against the maintained record, not recall
---
```

Output shape, fixed: `VERDICT` · `## Commands re-run` (`<cmd> → ticket says X, actual Y [MATCH|MISMATCH]`)
· `## Citations re-opened` · `## Plan vs. Verification` · `## Driven` (pasted output) · `## What I tried to
break it with, and could not`. On `BREAKS`, lead with the failing case — input, prescribed change, actual
bad outcome — so the author's next move is obvious from the first paragraph.

## The categorical / blast-radius seat — `architecture-reviewer`

The staff-engineer lens that keeps the harness from accreting narrow point-fixes. It runs at plan time —
the cheapest place to change shape (Google's design-doc review earns its value there; AWS's Correction-of-
Error requires proposing "common solutions for problems that are used in other places in your systems") —
and answers one question: what CLASS does this change instance, and is the fix right-sized? Prior art
grounds the *direction*, but there is no external precedent for an LLM reviewer whose mandate is scope-
widening; this seat is a practice ("step back as a staff engineer, look at the full blast radius")
reified so it fires without a human remembering to ask.

Two-sided, or it just trades under-building for over-building:
- **Narrow** — a one-off patch where a class-fix is available. The fix: shape the current change to be
  class-correct — put the invariant in the producer, not in whoever remembers to post-process. This is
  FREE; prefer it always.
- **Over-built** — a layer/abstraction/gate/registry no Verification scenario exercises. Speculative
  generality is its own maintenance surface; name it and the simpler shape that drops it.

The admission bar keeps the two honest: SHAPING a fix class-correct needs no evidence; BUILDING new
standalone machinery needs **≥2 realized incidents** of the same shape. One incident is a fix; two is a
class. Recommending machinery on one incident is itself the over-built failure.

Make the seat structural, not advisory (factor 8 — structural over behavioral): its output is a fixed
shape — CLASS · SIBLINGS (each `file:line`, each backed by a pasted `grep`/`ast-grep` command + output,
never recalled) · RIGHT-SIZE verdict · RECOMMENDATION. The pasted-search requirement is load-bearing: a
plausible sibling list is worthless, and a low-effort or fast model will answer from memory unless the
deliverable *is* the search output.

It doubles as an on-call **consultant** — spawn it ad hoc for a second opinion when the orchestrator hits a
design problem, not only as a standard plan-time gate. A reasoning-capable model at an effort that actually
drives its search tools is the fit. But for a second opinion on a design that has already **broken** (an
adversary `BREAKS`), prefer a second opinion from a different model than the judgment seats, where
available — this seat shares the judgment family that produced the break, so it shares its blind spots.

## Domain reviewers (add as the domain demands)

Same format; a distinct lens beats a duplicate generalist:
- **security-reviewer** (strong) — authz, injection, secrets, unsafe deserialization.
- **perf-reviewer** (mid/strong) — hot paths, N+1, allocation, query plans; where factor 17 is
  present, reads the benchmark result and the threshold breach rather than eyeballing.
- **a11y-reviewer** (mid) — contrast, focus order, labels, target size (UI projects).
- **api-compat-reviewer** (mid) — public API / plugin-host min-version / wire-format stability
  (libraries, plugins; e.g. an editor-plugin reviewer checking the host's `minAppVersion`).

## Domain knowledge (§ 10b) — preload a repo-local skill onto a reviewer

A reviewer that judges domain claims (an external-API / host-compat seat, a platform-behavior seat)
must not judge them from recall. Preload a **repo-local domain skill** onto the seat via the agent
front-matter `skills:` key — one of the only two channels that reach a subagent's context, the other
being its own def (`./instruction-placement.md`):

```yaml
---
name: api-compat-reviewer
model: <strong | mid tier>
tools: Read, Grep, Glob
skills:
  - <repo-local-domain-skill>   # e.g. the plugin-host's API surface, vendored under .claude/skills/
---

<!-- The skill is preloaded: every domain claim you judge is checked against the repo's MAINTAINED
     RECORD per the skill's own rule — never against recall, and never against a stale public mirror
     (an archived snapshot can be years behind and invert the current behavior). The skill's
     references/ are NOT preloaded; Read them at the paths SKILL.md names only when a data-model claim
     is load-bearing. -->
```

The discipline the note encodes, portable to any domain seat:
- **Check against the maintained record, not recall** — a same-model reviewer shares the author's
  prior and will confirm a plausible-but-wrong claim (`./findings-bar.md`, behavior-claim drift).
- **Name the authoritative source, not a stale mirror** — an archived/snapshot copy is a known
  inversion risk; the seat checks the record the repo maintains.
- **`references/` are lazy-loaded** — the skill's `SKILL.md` is the always-on contract; its deep
  reference files load on demand, keeping the seat's context lean (factor 10).

## Wiring

The agent names here must match the `{{placeholders}}` in the project's `/work` skill
(`./work-skill.md`). Phase 2 (fact-checker [+ architecture-reviewer + ticket-adversary]) and Phase 6
(correctness-reviewer [+ patterns-auditor]) each spawn their set in a single parallel message.
