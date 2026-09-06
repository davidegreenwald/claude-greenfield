# Ecosystem profiles — realizing factors 6, 7, 14, 16, and 17 per stack

Factors 6 (fast deterministic gate), 7 (executable architecture), 14 (test-strength), 16 (runtime
proof), and 17 (performance measure) are where "adaptive form" bites hardest: the principle is constant, the
tooling is not. This maps each to concrete tools per stack. Use these tables for the gate, the
boundary check, the mutation runner, the runtime driver, and the performance benchmark; defer deeper
stack idioms to the language's own toolchain.

The gate composes the four checks into one command, ordered cheap → expensive (fail fast):
**format/lint → types → architecture → tests**. Keep slow suites (UI, e2e) out of it.

Factor 14 (test-strength) is a *separate*, slower check — a mutation runner over changed files,
with a break threshold — that never joins the fast gate. Each table's last row names it; tool
recency was verified 2026-08-21.

---

## Python

| Check | Tool | Command |
|-------|------|---------|
| Lint + format | ruff | `ruff check src tests && ruff format --check src tests` |
| Types | mypy (strict) | `mypy` (config in `pyproject.toml`) |
| Architecture | import-linter | `lint-imports` (layers + forbidden-module contracts) |
| Tests + coverage | pytest | `pytest` (coverage floor as a fail-under) |
| Test strength (factor 14, separate) | mutmut | `mutmut run` — https://github.com/boxed/mutmut |

- **Gate:** a `make verify` target chaining the four.
- **Architecture (factor 7):** `[tool.importlinter]` contracts — `layers` for stage/layer
  ordering, `forbidden` to keep a module (e.g. `sqlite3`, an LLM client) out of layers that
  must not import it. Realization: `[tool.importlinter]` contracts in `pyproject.toml`.
- **Env:** stdlib `venv`; asdf-pinned Python via `.tool-versions`.

## Node / TypeScript

| Check | Tool | Command |
|-------|------|---------|
| Build | the bundler or `tsc -p tsconfig.build.json` | only if the project ships a build artifact |
| Lint + format | ESLint (+ Prettier/Biome) | `eslint . --max-warnings=0` |
| Types | tsc | `tsc -noEmit` (strict; `noUncheckedIndexedAccess`) |
| Architecture | ESLint boundaries / dependency-cruiser | `depcruise src` or ESLint `no-restricted-imports` |
| Tests | Vitest / Jest | `vitest run` |
| Test strength (factor 14, separate) | Stryker | `npx stryker run` — https://stryker-mutator.io |

- **Gate:** a `check` package script chaining build + typecheck + lint + test. If the repo
  already has one, reuse it. Realization: a `check` script in `package.json`.
- **Architecture (factor 7):** start with ESLint `no-restricted-imports` /
  `eslint-plugin-boundaries` to forbid framework or host-API imports in pure core (e.g. an
  editor plugin enforcing "no host-API import in core" via `eslint.config.mjs`); add
  `dependency-cruiser` for acyclicity and layer rules when the module graph grows.
- **Hooks:** husky + lint-staged, or raw `.git/hooks/pre-commit` running the `check` script.
- **Pin `typescript` to the range `typescript-eslint` peers** (`npm view typescript-eslint
  peerDependencies`) — a fresh `npm install typescript@latest` can land a major the lint stack does
  not support yet and the gate fails on install (e.g. on 2026-09-05: `typescript@latest` is 7.0.2,
  `typescript-eslint` peers `typescript: ">=4.8.4 <6.1.0"`).

## Swift / iOS / macOS

| Check | Tool | Command |
|-------|------|---------|
| Lint | SwiftLint | `swiftlint --strict` |
| Format | swift-format / SwiftFormat | `swift-format lint -r Sources` |
| Tests (unit) | Swift Testing / XCTest | `swift test` (per SPM package) |
| Cross-file audit | project script | `scripts/audit-*.sh` |
| UI / a11y (slow — NOT in the fast gate) | XCUITest + AXe | a separate `test-ui` target |
| Test strength (factor 14, separate) | muter | `muter run` — https://github.com/muter-mutation-testing/muter (dev-active; last tagged v16, build from `master` for current fixes) |

- **Gate:** `make verify` = lint + format-check + `swift test` + audit. Critically, keep
  XCUITest/a11y suites in a separate `test-ui` step — they run in minutes and would defeat
  factor 6's "fast" property. Realization: a `make verify` target plus a separate `test-ui` target.
- **Architecture (factor 7):** Swift has no import-linter equivalent. Enforce boundaries with
  SPM package separation (a module simply cannot import what it doesn't depend on) plus a
  SwiftLint custom rule for finer constraints. Where a boundary can't be machine-checked, fall
  back to a path-scoped rule + a reviewer agent.
- **Defer** simulator/build/test specifics to the platform toolchain (`xcodebuild`, `swift test`).

## Rust

| Check | Tool | Command |
|-------|------|---------|
| Lint | clippy | `cargo clippy -- -D warnings` |
| Format | rustfmt | `cargo fmt --check` |
| Types | the compiler | `cargo check` |
| Architecture | the module/visibility system; `cargo-deny` for deps | `cargo deny check` |
| Tests | cargo test | `cargo test` |
| Test strength (factor 14, separate) | cargo-mutants | `cargo mutants` — https://github.com/sourcefrog/cargo-mutants |

- **Gate:** a `just verify` / `make verify` chaining these. Architecture is largely enforced by
  the crate/module visibility system; `cargo-deny` guards the dependency graph.

## Go

| Check | Tool | Command |
|-------|------|---------|
| Lint | golangci-lint | `golangci-lint run` |
| Format | gofmt/goimports | `gofmt -l .` |
| Architecture | go-arch-lint / internal packages | `go-arch-lint check` |
| Tests | go test | `go test ./... -race` |
| Test strength (factor 14, separate) | gremlins | `gremlins unleash` — https://github.com/go-gremlins/gremlins (pre-1.0) |

- **Architecture:** `internal/` packages enforce real boundaries by the compiler;
  `go-arch-lint` adds layered contracts.

---

## Runtime harness drivers (factor 16)

Factor 16 says a claim about the running system is proven by driving the real system. The harness shape
is constant (`./exemplars/runtime-harness.md`); what varies by stack is the **driver** — the thing that
launches, attaches to, and steers the real system. Pick the row whose "real system" matches the claim,
not the language: a Node CLI and a Rust CLI share the subprocess row.

| Real system | Driver | Notes |
|---|---|---|
| Web app / site | Playwright (or Cypress) against a **running** dev server or container | real browser + real server; assert on rendered DOM, network, and storage — never on component internals |
| HTTP service / API | real backing deps via docker compose or testcontainers + an HTTP client script | never mock the store or queue the claim is about |
| CLI (any language) | a subprocess drives the **built** binary in a temp dir (`subprocess.run`, `execa`, `assert_cmd`, `os/exec`) | the built artifact, not the module under test; assert on exit code + stdout + the files it wrote |
| iOS / macOS app | XCUITest + `xcrun simctl` (boot, install, launch, screenshot); AXe for accessibility | the simulator is the real system for most claims; a device for hardware ones |
| Electron app / desktop plugin host | CDP attach to the host's debug port (puppeteer-core) — attach to the existing page; Electron does not create targets | launch the host with its own config dir so the user's instance is never touched |
| Host with an add-on HTTP API | the host's API for state (a localhost add-on API), the UI driver for the human-visible claims | liveness by the API socket and the file holder, never a process-name match |
| Browser extension | Playwright with the unpacked extension loaded (`--load-extension`) | assert through the page the extension modifies |
| Generic fallback | a scripted runbook whose every step writes its output to a file and diffs it against a committed expected-output file — that diff is the red; the ticket's Evidence cites the run | the harness *form* for a project with no automatable driver yet; record it in `adr-002` |

Both harness proofs (`--negative-control`, `--twice`) apply to every row, including the runbook: plant a
fault and confirm the runbook's recorded output goes red at that step.

---

## Performance measure — benchmark → profiler (factor 17, where applicable)

Factor 17 is *present-when-applicable*: install it where the project has a perf-sensitive surface
(page render, DB query, hot compute path, memory). The shape is constant across stacks — a
repeatable, variance-aware **benchmark** produces a number, a **threshold or stored baseline**
fails the build on regression (via a non-zero exit code), and a **profiler** is the on-demand
escalation when the gate trips. Like factor 14, the benchmark is slow/noisy and runs as a
*separate* pre-merge or scheduled step, never in the fast commit gate.

| Stack / surface | Benchmark tool | CI regression gate | Profiler (escalation) |
|-----------------|----------------|--------------------|-----------------------|
| Web rendering | Lighthouse CI; k6 (load) | Lighthouse CI `assert` error-level / `budget.json`; k6 `thresholds` → non-zero exit | Chrome DevTools Performance; WebPageTest; 0x |
| Database / query | pgbench; `EXPLAIN ANALYZE` | script a TPS/latency threshold against pgbench output (no built-in fail flag) | `EXPLAIN (ANALYZE, BUFFERS)`; `pg_stat_statements` |
| Python | pytest-benchmark; pyperf | `pytest-benchmark --benchmark-compare-fail mean:5%` | cProfile; py-spy; scalene |
| Node / TS | Vitest `bench` / tinybench | diff against a stored baseline (e.g. CodSpeed) → fail | `node --prof`; clinic.js; 0x |
| Swift / iOS / macOS | XCTest `measure` / `XCTMetric` | perf test fails when a metric exceeds the recorded baseline | Instruments — Time Profiler / Allocations |
| Rust | Criterion (`harness = false`) | Criterion in CI + a threshold, or `iai` for instruction-count stability | `cargo flamegraph`; `perf` |
| Go | `testing.B` (`go test -bench`) | `benchstat` before/after comparison in CI | `runtime/pprof` (`-cpuprofile` / `-memprofile`) |

Verified 2026-08-24 against primary docs for the CI-gate mechanics (Lighthouse CI, k6 thresholds,
pytest-benchmark, pgbench). The DB, Node-baseline, and Rust/Go gates are *conventions* — you script
the threshold check against the tool's output; there is no vendor fail-flag. The constant is:
**a number vs a threshold/baseline → non-zero exit → CI fails.** Wire only the benchmark+threshold
into CI; keep the profiler as a documented on-demand invocation — it tells you *why* something
regressed once the benchmark tells you *that* it did.

---

## Generic fallback (no profile above)

Realize the principle from first parts:
1. **Gate (factor 6):** find or compose the four checks the language offers — a formatter, a
   linter, a type/compile check, a test runner — and chain them cheap → expensive into one
   command bound to a commit hook.
2. **Architecture (factor 7), in priority order:**
   1. a dedicated dependency/layer linter if one exists for the ecosystem;
   2. else the language's own module/visibility system, structured so the boundary is
      unrepresentable (separate packages/modules);
   3. else — and only else — a path-scoped rule documenting the boundary **plus** a reviewer
      agent assigned to guard it. Never silently drop factor 7; downgrade it visibly to
      rule+reviewer and note it in `adr-001`.
3. **Test strength (factor 14):** if the ecosystem has a mutation tool, run it over changed
   files as a separate step with a break threshold; else use the revert-check convention —
   deliberately break a line of logic, confirm a test goes red, then revert.
4. **Runtime harness (factor 16):** name the real system and pick its driver from the table above;
   if nothing automatable exists yet, the scripted runbook with recorded output is the form, recorded
   in `adr-002` until the first behavioral ticket builds the harness.
5. **Performance measure (factor 17), where a perf-sensitive surface exists:** find the
   language's benchmark harness (or time a representative operation), record a baseline number,
   and fail a separate (non-fast-gate) step when a later run regresses past a threshold; document
   a profiler invocation for triage. If the project has no perf-sensitive surface, record that as
   an explicit N/A in `adr-001` rather than inventing a hollow benchmark.

Hooks are universal: a `pre-commit` running the gate (authoritative), a turn-end backstop, and
a format-on-write step. See `./exemplars/hooks.md`.

---

## Makefile conveniences (any make-driven stack)

A small pattern that pays off in any repo whose gate is a Makefile:

- **Self-documenting `help`.** Tag each public target with a `## <description>` trailing comment and
  make `help` the default target; one `grep`/`awk` line prints the catalog, so `make` with no args is
  a live index that cannot drift from the targets it lists:
  ```make
  help: ## List targets
  	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | \
  		awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-14s\033[0m %s\n", $$1, $$2}'
  ```
