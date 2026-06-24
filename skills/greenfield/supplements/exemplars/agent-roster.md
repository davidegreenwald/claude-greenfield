# Exemplar — the reviewer roster

**Adapt, do not copy verbatim.** Realizes factors 8 and 12 (independent review, effort tiering).
Author each agent as a project subagent in `.claude/agents/` using the format below.

Every reviewer is **read-only**, returns a **concise structured verdict** (to protect the
orchestrator's context), and runs **single-pass** (the orchestrator verifies fixes itself, not
by re-spawning). Each agent's prompt instructs it to report only findings with a named
production consequence — not style or behavior-neutral refactors.

## Subagent definition format

A project agent lives at `<project>/.claude/agents/<name>.md`:

```markdown
---
name: <agent-name>
description: <role>. Use in /work Phase <N> to <purpose>. Read-only; returns <verdict shape>.
model: <strong | mid tier>     # factor 12
tools: Read, Grep, Glob, Bash  # + WebFetch, WebSearch for a fact-checker
# effort: high                 # optional, for the hardest judgment agents
---

<Contract: exactly what to check, against what (the ticket, the diff, the rules).>

Output: <the verdict format — a one-word verdict or a bounded list of findings,
each `[severity] <type> — <consequence> — <file:line>`. Cap the length.>
```

## The archetype library

Tier: **strong** = the most capable model (judgment); **mid** = a faster/cheaper model
(mechanical/citation). Map to whatever tiers the provider offers.

| Archetype | Tier | Phase | Checks | Returns |
|-----------|------|-------|--------|---------|
| **ticket-planner** | mid | 1 | decision-completeness vs the template; no TBD; Research + Proven/Novel cited; `ADR:` field correct for shape (Component that moves a boundary ≠ `none`); registered | `COMPLETE` or numbered gaps |
| **fact-checker** | mid | 2 | every citation resolves; counts re-run; claims supported | per-claim verdicts |
| **architecture-reviewer** | strong | 2 | blast radius, layering, interface/schema impact (pre-code) | `APPROVE`/`BLOCK` + gaps |
| **correctness-reviewer** | strong | 6 | line-level correctness, edge cases, end-to-end trace (the diff) | `APPROVE` or findings by severity |
| **patterns-auditor** | mid | 6 | conformance to patterns + path-scoped rules | `CONFORMS` or deviations |
| **pm-explainer** | mid | 8 | translate the change to user/product impact | 3-5 plain sentences |

Minimum viable roster (small project): `ticket-planner`, `correctness-reviewer`,
`patterns-auditor`. Add `fact-checker` when the project makes external claims, `pm-explainer`
for user-facing work, and `architecture-reviewer` once there is real architecture to protect.

**Research agents are not roster members.** The per-ticket research fan-out in `/work` Phase 1
uses the install's generic research agent (`Explore`, or `general-purpose`), capped at 2-3
parallel for a Component ticket — do not author bespoke researcher subagents. The roster is
reviewers only; research is a built-in capability the orchestrator drives, and its findings
are what `ticket-planner` then checks for citations.

## Domain reviewers (add as the domain demands)

Same format; a distinct lens beats a duplicate generalist:
- **security-reviewer** (strong) — authz, injection, secrets, unsafe deserialization.
- **perf-reviewer** (mid/strong) — hot paths, N+1, allocation, query plans.
- **a11y-reviewer** (mid) — contrast, focus order, labels, target size (UI projects).
- **api-compat-reviewer** (mid) — public API / plugin-host min-version / wire-format stability
  (libraries, plugins; e.g. an editor-plugin reviewer checking the host's `minAppVersion`).

## Wiring

The agent names here must match the `{{placeholders}}` in the project's `/work` skill
(`./work-skill.md`). Phase 2 and Phase 6 spawn their pair in a single parallel message.
