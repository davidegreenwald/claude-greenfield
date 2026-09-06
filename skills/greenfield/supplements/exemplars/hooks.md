# Exemplar — the enforcement hooks

**Adapt, do not copy verbatim.** Realizes factor 6's enforcement half (the gate is only as real
as what makes it run). Scripts are committed, shellcheck-clean, and tested.

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

## Test-strength is not a hook (factor 14)

Mutation testing is far too slow for commit time — it would make the gate unusable and get
bypassed. Keep it *out* of `{{GATE}}` and the hooks above; run it as a separate step (`/work`
Phase 5b, a pre-merge check, or a scheduled/CI job) over changed files. The fast gate proves the
code runs; the mutation step proves the tests can fail. Per-stack tool in `../ecosystem-profiles.md`.

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
versioned) rather than copying into `.git/hooks` (per-clone, unversioned). The pointer is set by a
committed `scripts/hooks/install-hooks.sh` that the stack's install step runs (§ Install order):

```
git config core.hooksPath scripts/hooks
```

**Node/TS projects** may prefer husky + lint-staged instead of a raw script — same effect:
`pre-commit` runs the `{{GATE}}` (e.g. `npm run check`). Use whichever the project already
leans on; do not add husky to a repo that has none if a plain script suffices.

### `verify-hooks-live.sh` — prove the gate can actually fire (wire into `{{GATE}}`)

The gate cannot detect its own absence, so this check runs *outside* the hook — in `verify`. It
**asks git which file it will run** (`git rev-parse --git-path hooks/pre-commit` honours
`core.hooksPath` from any source) rather than modelling the config, and asserts the whole chain: the
file git will run is the committed script, it is not a dangling link, it is executable, and it runs the
gate. Exit non-zero with a one-line reason otherwise.

```sh
#!/bin/sh
# verify-hooks-live.sh — exit 0 only if git will run the committed pre-commit gate.
set -u
EXPECTED_DIR="$(cd "$(git rev-parse --show-toplevel)/scripts/hooks" 2>/dev/null && pwd -P)" \
  || { echo "verify-hooks-live: scripts/hooks/ is missing"; exit 3; }
EXPECTED="${EXPECTED_DIR}/pre-commit"
HOOK="$(git rev-parse --git-path hooks/pre-commit)"          # honours core.hooksPath
HOOK_DIR="$(cd "$(dirname "${HOOK}")" 2>/dev/null && pwd -P)" \
  || { echo "verify-hooks-live: hooks dir does not exist: $(dirname "${HOOK}")"; exit 3; }
HOOK="${HOOK_DIR}/$(basename "${HOOK}")"
[ "${HOOK}" = "${EXPECTED}" ] \
  || { echo "verify-hooks-live: git would run ${HOOK}, not ${EXPECTED} (check core.hooksPath)"; exit 3; }
if [ -L "${HOOK}" ] && [ ! -e "${HOOK}" ]; then
  echo "verify-hooks-live: dangling symlink: ${HOOK}"; exit 4    # [ -f ] follows the link and reads FALSE
fi
[ -f "${HOOK}" ] || { echo "verify-hooks-live: hook is absent: ${HOOK}"; exit 4; }
[ -x "${HOOK}" ] || { echo "verify-hooks-live: hook is not executable (git skips it silently): ${HOOK}"; exit 4; }
grep -qF '{{GATE}}' "${HOOK}" \
  || { echo "verify-hooks-live: hook does not run the gate: ${HOOK}"; exit 5; }
```

Replace `{{GATE}}` in the last check with the literal gate command the hook runs. The check stays
correct under a local `core.hooksPath` (the install above), a global one, or none: it compares what
git resolves against the committed path, whichever way the resolution happened. (Promoted from an
incident where hooks had been silently off for an unknown number of commits; a reviewer proved that
checking the hook's existence alone returned 0 while the gate was dead.)

## Claude Code hooks (`settings.json`)

```json
{
  "hooks": {
    "Stop": [{ "hooks": [{ "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/verify-on-stop.sh" }] }],
    "PostToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/format-edited.sh" }] }
    ]
  }
}
```

- `verify-on-stop.sh` — `cd "$CLAUDE_PROJECT_DIR"` then run `{{GATE}}`; on red, print the failure
  to stderr and `exit 2` — only exit 2 blocks the stop and feeds stderr back to the agent; any other
  non-zero is a non-blocking error the stop proceeds through
  (https://code.claude.com/docs/en/hooks § exit codes). Guard against block loops (the
  `stop_hook_active` field; built-in block cap).
- `format-edited.sh` — if the edited file matches the language extension, run the formatter
  (`ruff format` / `eslint --fix` / `swift-format` / `gofmt`). A no-op otherwise; never block the
  edit. Note it does not see files written via Bash — the gate's format-check is the backstop.

## Install order

1. `git init` — the liveness check asks git which hook it will run, so the repo comes first.
2. Install the hook: commit `scripts/hooks/pre-commit` and `scripts/hooks/install-hooks.sh`, and run
   the installer from the stack's install step (`npm` `prepare` script, `make install-hooks`, the
   `poetry`/`cargo` equivalent) so a fresh clone gets the hook on install, not by hand:

   ```sh
   #!/bin/sh
   # install-hooks.sh — point git at the committed hooks dir. Run by the install step.
   set -e
   cd "$(git rev-parse --show-toplevel)"
   git config core.hooksPath scripts/hooks
   echo "core.hooksPath=$(git config core.hooksPath)"
   ```
3. Wire `verify-hooks-live.sh` into `{{GATE}}` so the gate proves its own enforcement.
4. Finish the gate command, add the `Stop` + `PostToolUse` hooks to `settings.json`, and prove the
   gate green on the seeded project.
5. Forced failure: make a trivial commit and confirm the gate runs; break formatting and confirm the
   edit hook fixes it; run `verify-hooks-live.sh` and confirm it exits 0, then point
   `core.hooksPath` at an empty dir and confirm it exits non-zero.
