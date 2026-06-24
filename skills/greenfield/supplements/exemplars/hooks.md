# Exemplar — the enforcement hooks

**Adapt, do not copy verbatim.** Realizes factor 6's enforcement half (the gate is only as real
as what makes it run). Scripts are committed, shellcheck-clean, and testable.

Three hooks, each with a distinct job:

| Hook | Fires | Runs | Role |
|------|-------|------|------|
| `pre-commit` (git) | at commit creation | the `{{GATE}}` | **authoritative** — a red gate aborts the commit, so every commit that lands is green |
| `Stop` (Claude Code) | at turn end | the `{{GATE}}` on the main checkout | **backstop** — catches uncommitted breakage |
| `PostToolUse` (Edit\|Write) | after each edit | the formatter on the edited file | keeps the tree formatted so the gate's format-check passes |

## Why commit-time is authoritative

A `git merge --ff-only` runs no hooks, so gating at *commit creation* is what guarantees every
commit reaching main is green. The Stop hook is a backstop, not the gate — under a
worktree-per-ticket flow it checks the main checkout, not the worktree where the ticket commits.

## git `pre-commit`

A committed script (e.g. `scripts/hooks/pre-commit`) installed into the repo's hooks path:

```sh
#!/bin/sh
# Authoritative gate: a red verify aborts the commit.
set -e
{{GATE}}
```

Optionally pair with a no-direct-commit-to-`main` guard so work only lands via the reviewed
branch + ff-only merge. Install by pointing `core.hooksPath` at the committed dir (shared,
versioned) rather than copying into `.git/hooks` (per-clone, unversioned):

```
git config core.hooksPath scripts/hooks
```

**Node/TS projects** may prefer husky + lint-staged instead of a raw script — same effect:
`pre-commit` runs the `{{GATE}}` (e.g. `npm run check`). Use whichever the project already
leans on; do not add husky to a repo that has none if a plain script suffices.

## Claude Code hooks (`settings.json`)

```json
{
  "hooks": {
    "Stop": [{ "hooks": [{ "type": "command", "command": "<repo>/.claude/hooks/verify-on-stop.sh" }] }],
    "PostToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "<repo>/.claude/hooks/format-edited.sh" }] }
    ]
  }
}
```

- `verify-on-stop.sh` — `cd "$CLAUDE_PROJECT_DIR"` then run `{{GATE}}`; exit non-zero to block
  the stop and return the failure to the agent. Guard against block loops (the `stop_hook_active`
  field; built-in block cap).
- `format-edited.sh` — if the edited file matches the language extension, run the formatter
  (`ruff format` / `eslint --fix` / `swift-format` / `gofmt`). A no-op otherwise; never block the
  edit. Note it does not see files written via Bash — the gate's format-check is the backstop.

## Install order

1. Write the gate command first (it must pass green on the seeded project).
2. Install `pre-commit` + point `core.hooksPath` at it.
3. Add the `Stop` + `PostToolUse` hooks to `settings.json`.
4. Verify: make a trivial commit and confirm the gate runs; break formatting and confirm the
   edit hook fixes it.
