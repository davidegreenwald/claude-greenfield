# Ecosystem profiles — realizing factors 6 and 7 per stack

Factors 6 (fast deterministic gate) and 7 (executable architecture) are where "adaptive form"
bites hardest: the principle is constant, the tooling is not. This maps each to concrete tools
per stack. Use these tables for the gate and the boundary check; defer deeper stack idioms to
the language's own toolchain.

The gate composes the four checks into one command, ordered cheap → expensive (fail fast):
**format/lint → types → architecture → tests**. Keep slow suites (UI, e2e) out of it.

---

## Python

| Check | Tool | Command |
|-------|------|---------|
| Lint + format | ruff | `ruff check src tests && ruff format --check src tests` |
| Types | mypy (strict) | `mypy` (config in `pyproject.toml`) |
| Architecture | import-linter | `lint-imports` (layers + forbidden-module contracts) |
| Tests + coverage | pytest | `pytest` (coverage floor as a fail-under) |

- **Gate:** a `make verify` target chaining the four.
- **Architecture (factor 7):** `[tool.importlinter]` contracts — `layers` for stage/layer
  ordering, `forbidden` to keep a module (e.g. `sqlite3`, an LLM client) out of layers that
  must not import it. Realization: `[tool.importlinter]` contracts in `pyproject.toml`.
- **Env:** stdlib `venv`; asdf-pinned Python via `.tool-versions`.

## Node / TypeScript

| Check | Tool | Command |
|-------|------|---------|
| Build | esbuild / tsc | `tsc -noEmit` (or the build script) |
| Lint + format | ESLint (+ Prettier/Biome) | `eslint . --max-warnings=0` |
| Types | tsc | `tsc -p tsconfig.test.json -noEmit` (strict; `noUncheckedIndexedAccess`) |
| Architecture | ESLint boundaries / dependency-cruiser | `depcruise src` or ESLint `no-restricted-imports` |
| Tests | Vitest / Jest | `vitest run` |

- **Gate:** a `check` package script chaining build + typecheck + lint + test. If the repo
  already has one, reuse it. Realization: a `check` script in `package.json`.
- **Architecture (factor 7):** start with ESLint `no-restricted-imports` /
  `eslint-plugin-boundaries` to forbid framework or host-API imports in pure core (e.g. an
  editor plugin enforcing "no host-API import in core" via `eslint.config.mjs`); add
  `dependency-cruiser` for acyclicity and layer rules when the module graph grows.
- **Hooks:** husky + lint-staged, or raw `.git/hooks/pre-commit` running the `check` script.

## Swift / iOS / macOS

| Check | Tool | Command |
|-------|------|---------|
| Lint | SwiftLint | `swiftlint --strict` |
| Format | swift-format / SwiftFormat | `swift-format lint -r Sources` |
| Tests (unit) | Swift Testing / XCTest | `swift test` (per SPM package) |
| Cross-file audit | project script | `scripts/audit-*.sh` |
| UI / a11y (slow — NOT in the fast gate) | XCUITest + AXe | a separate `test-ui` target |

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

- **Gate:** a `just verify` / `make verify` chaining these. Architecture is largely enforced by
  the crate/module visibility system; `cargo-deny` guards the dependency graph.

## Go

| Check | Tool | Command |
|-------|------|---------|
| Lint | golangci-lint | `golangci-lint run` |
| Format | gofmt/goimports | `gofmt -l .` |
| Architecture | go-arch-lint / internal packages | `go-arch-lint check` |
| Tests | go test | `go test ./... -race` |

- **Architecture:** `internal/` packages enforce real boundaries by the compiler;
  `go-arch-lint` adds layered contracts.

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

Hooks are universal: a `pre-commit` running the gate (authoritative), a turn-end backstop, and
a format-on-write step. See `./exemplars/hooks.md`.
