# Exemplar — instruction placement (where each rule lives)

**Adapt, do not copy verbatim.** Realizes factor 10 (progressive disclosure & context scoping)
and pairs with factor 12 (effort tiering). Use this when authoring or auditing CLAUDE.md, the
`/work` skill, the reviewer roster, and `.claude/rules/*` — to decide which layer each instruction
belongs in.

The decision is forced by how Claude Code assembles a subagent's context. Placement is not
stylistic; it changes which agents pay for an instruction and which agents actually receive it.

## What reaches a subagent's context

Source: Claude Code docs — `sub-agents.md`, `memory.md`, `skills.md`. Verify against your CLI
version; the rules-inheritance row is undocumented and inferred.

| Content | Reaches a subagent? | Notes |
|---------|---------------------|-------|
| **CLAUDE.md** (root, nested, user-global, managed policy) | **Yes — wholesale** | The whole hierarchy loads. Built-in `Explore` and `Plan` agents are the exception: they skip CLAUDE.md. |
| **Orchestrator system prompt** | No | The subagent uses only its own agent-definition prompt. |
| **Orchestrator conversation / history** | No | Only the task summary the orchestrator writes is passed. |
| **A skill the orchestrator invoked** | No | Not inherited. A skill loads into a subagent only if named in its `skills:` field. |
| **Skills in the subagent's `skills:` list** | **Yes — preloaded** | This is the opt-in channel for task-specific reference. |
| **`.claude/rules/*` (path-scoped or always-on)** | **Not known to** | Not listed among inherited content; treat as main-session-only until confirmed. |
| **Agent-definition body** (`.claude/agents/<name>.md`) | **Yes** | This *is* the subagent's system prompt. |
| **Git status** | Yes | Snapshot at spawn (except Explore/Plan). |
| **Global `~/.claude/` skills & agents** | Discoverable | Available to reference/invoke, but not preloaded unless listed; a silent dependency breaks reproducibility. |

Two consequences drive every placement call below:
1. **CLAUDE.md is a broadcast.** Everything in it is paid for in every subagent's context.
   Put only what *every* agent needs.
2. **The subagent's own def + its `skills:` are the only targeted channels.** A constraint a
   specific subagent must honor has to live there — a path-scoped rule or the orchestrator's
   conversation will not reach it.

## Placement decision table

| The instruction is… | Put it in | Because |
|---------------------|-----------|---------|
| A durable standard every agent should follow (evidence hygiene, commit convention, an architecture invariant, shell hygiene) | **root CLAUDE.md** (or an always-on `.claude/rules/*` with no `paths:`) | It legitimately applies to every agent; broadcast is correct. |
| A narrow constraint that applies only when a certain kind of file is touched (persistence idioms, core-purity, test determinism) | **path-scoped `.claude/rules/*`** (`paths:` glob) **and** — if a subagent must honor it — the owning agent's def or its `skills:` | Keeps the main session's context lean; but because rules aren't known to reach subagents, mirror any subagent-critical rule into that agent. |
| Coordination the orchestrator alone runs (phase order, when to halt, merge gate, roster wiring) | **the `/work` skill** | Skills aren't inherited, so this reaches no subagent — correct, since only the orchestrator executes it. In CLAUDE.md it would be pure noise in every subagent. |
| A constraint one reviewer/worker subagent must apply to its task | **that agent's definition** (or a skill in its `skills:`) | The only channel targeted at that agent. |
| Reusable task reference several agents may need on demand | **a skill**, listed in each consuming agent's `skills:` | Preloads only where listed; keeps other contexts clean. |
| Something the whole workflow depends on that currently exists only in a personal `~/.claude/` | **vendor into `.claude/skills` / `.claude/agents`**, or record as a declared external dependency | Silent global reliance is non-reproducible: another contributor or CI without it behaves differently. |

## Sizing to model & effort (factor 12 pairing)

Instruction load is matched to the tier running the task, not stamped uniformly:
- A judgment agent on the strongest tier at high effort needs the *contract* (what to check,
  against what, what to return) — not a tutorial. Keep its def tight.
- A mechanical agent on a mid tier benefits from an explicit checklist; spell out the steps.
- Never inflate CLAUDE.md to compensate for an under-specified agent — that taxes every other
  agent. Fix the instruction at the agent that needs it.

Rule of thumb: if you are adding words to CLAUDE.md to fix one agent's behavior, you are
probably editing the wrong file.

## Anti-patterns this catches

- **The CLAUDE.md wall.** A 70KB CLAUDE.md carrying pipeline steps and per-layer rules — inherited
  in full by every reviewer and research subagent, burying the load-bearing standard and paying
  the token cost on every spawn. Split it: standards stay, orchestration → `/work`, per-file
  constraints → rules/agents.
- **The unreachable rule.** A must-honor constraint parked in a path-scoped rule the subagent
  never loads. Mirror it into the agent's def or listed skills.
- **The invisible dependency.** A `/work` phase that assumes a global `ios-dev` (or similar) skill.
  Decide deliberately per repo: vendor it in for reproducibility, or document it as required —
  never assume the author's environment.

## Nested per-package CLAUDE.md (localizing architectural guidance)

A multi-package repo puts each layer's rules in that layer's own `<pkg>/CLAUDE.md`, not the root.
Each nested file states, tightly: the layer's one-line **responsibility**; its **import boundary**
(backed by the arch tool — import-linter / dependency-cruiser); **what belongs and what does NOT**;
and any per-file **change-warning** ("changing `normalize()` invalidates every stored `alias_norm` —
read the module docstring first"). This is factor-7 guidance colocated with the code it governs, so
an agent working in that subtree gets the boundary at hand and the root stays lean.

Caveat: nested CLAUDE.md is inherited **wholesale**, same as the root (see the table above), so keep
each one to responsibility + boundary + contracts — never a tutorial. (Promoted from a reference repo that
carries one per `src/<pkg>/`.)
