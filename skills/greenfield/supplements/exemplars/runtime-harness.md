# Exemplar — the runtime harness (factor 16, runtime proof)

**Adapt, do not copy verbatim.** Realizes factor 16: a behavioral claim about the running system is
proven by driving the real system, never by citing code or by a unit test whose fixtures the same agent
staged. This is the shape of the harness that does the driving, the two proofs it must carry about
itself, and the interlocks that keep a harness with destructive verbs from destroying the wrong thing.
Generalized from a production plugin's live-QA harness, which drives two real desktop applications end to
end without human hands.

**A harness, not a test suite.** The unit suite proves what its fixtures stage. The harness launches the
real system — the app, the service with its real backing store, the device or simulator, the plugin host,
the external API — puts it in a known cold state, drives an ordered sequence of steps against it, and
asserts on what the real system did. Reach for it whenever a claim is about **behavior against the real
thing** — "the sync writes nothing on a re-run", "the modal opens", "the record reads back" — because
neither a unit test nor a `file:line` citation can establish those (`./rules/evidence.md`, rule 1).

## When it gets built — `/work` discretion, not init

Greenfield installs the principle, the evidence rule, this exemplar, the per-stack driver row, and the
landing discipline. It does **not** build the harness at init: for an arbitrary stack that would be
theorizing about a system that does not exist yet. The first ticket whose Verification names a claim about
the running system builds the harness **as part of its scope** — the ticket is not done until the claim
is driven — and the smallest end-to-end slice that drives the ticket's most dangerous claim is enough to
start. Until that ticket, `adr-002` records the deferral and names the fallback: a scripted runbook whose
every step writes its output to a file and diffs it against a committed expected-output file — that
diff is the red; the ticket's Evidence cites the run.

## Layout

```
<harness dir>/                  # qa/ or e2e/ — a sibling of tests/, never inside it
├── run.*                       # the matrix runner: reset → launch → steps → report; --from/--to, --twice, --negative-control
├── <system>.*                  # one driver module per real system: launch, attach, liveness, cold reset
├── fixture.*                   # seeds the RUN COPY of the fixture from the tracked source (read-only source)
├── assert.*                    # the anti-vacuity assertion library (below)
├── lock.*                      # the run lock — two harness runs can never coexist
├── config.*                    # the destructive-operation allowlists and the never-click list
├── steps/                      # ordered, STATEFUL steps; each names the ticket criterion it proves
├── probe-*.*                   # one-off adversarial probes against the real API, kept, not deleted
└── report/                     # report.md + report.json + per-step screenshots/logs (gitignored)
```

The steps are **stateful and ordered** — a matrix, not independent tests: step 7 assumes the state
steps 1–6 left. A range run (`--from 5 --to 9`) still resets from cold. The report lists every step with
its denominator ("10 of 17 steps ran") and recommends nothing it did not run. **The harness never edits
tickets or the registry** — close-out is the orchestrator's move, made from the report.

Drivers per stack: `../ecosystem-profiles.md` § Runtime harness drivers.

## The two proofs the harness carries about itself

**A green suite is worthless unless it can go red.** Before any run counts as evidence, both must pass:

- **`--negative-control <step>`** — plant a fault the harness must catch; the run must go red at
  **exactly** that step and nowhere else. A harness that stays green with a planted fault, or goes red
  somewhere else, is not measuring what it claims.
- **`--twice`** — run, reset, run again, machine-diff the two reports clean. A harness whose two cold
  runs differ is reporting its own noise as the system's behavior.

Record both in the harness README and re-run them whenever the runner or a driver changes.

## Assertions cannot pass vacuously

An assertion library the steps must go through, throwing on:

- an `undefined`/`null` actual — the value was never produced, so nothing was compared;
- a **zero-of-zero** count — `0 of 0 items matched` reads as pass and proves nothing;
- an **empty-population** "every" — `every(item, pred)` over no items is true by definition;
- a step that ran and **asserted nothing** — `assertStepProducedEvidence()` at each step's end.

And the negative-reading rule from `./rules/evidence.md`: a step that concludes *absent / clean / no
matches* needs a positive-control witness in the same run — the instrument must be seen to fire on a
known-true case before its silence means anything.

## Destructive interlocks — for a harness that wipes, resets, or clicks

A harness that puts the real system into a cold state owns the most destructive code in the repo — a
data-store wipe, an `rm -rf` of the run fixture, a click driver that could confirm a destructive dialog —
and the user's real data often sits one directory over from the test profile. Every one of these was
paid for by a driven incident; keep all of them:

- **The wipe takes no path argument.** It derives its target from a **test profile resolved at
  runtime** by name pattern, asserts that exactly one profile matches, and asserts it is not the user's.
  No caller can aim it.
- **Liveness is proven by the resource actually at risk, never by a process-name match.** "Is anything
  holding the file I am about to delete" (`lsof`), "does the API socket answer" — not `pgrep -f
  <AppName>`, which matches nothing when the app is a launcher plus a venv process, so the not-running
  gate passes vacuously and the wipe fires against a live collection. A cleverer process regex is the
  same bug with a longer string.
- **The wipe allowlist is a claim, and it fails closed.** Enumerate what the system puts in a profile
  directory; throw on anything neither allowlisted nor known-benign. Never wipe whatever happens to be
  there — the enumeration is the safety property. A half-wiped state is worse than an aborted reset,
  because cold-verification cannot be trusted against it.
- **The test-profile assertion runs inside the client, not at the call sites.** Any destructive API
  action triggers `assertTestProfileActive()` automatically, so no caller can bypass it and no new
  caller can forget it — a call-site version of this rule was already bypassed once by a passthrough
  verb.
- **The reset destroys ONLY the run copy.** The tracked fixture (`test-data/`, `fixtures/…`) is seeded
  into a gitignored run copy; nothing in the harness writes to the tracked source.
- **Never auto-confirm a destructive dialog.** A never-click list of (dialog heading, button text)
  pairs the click driver refuses, with a driven test that it refuses them.
- **Guard the user's own running instance.** The harness attaches only to the instance it launched
  (its own config dir / debug port), never to whatever happens to be running.
- **The run lock is atomic and never uses elapsed time as liveness.** A kernel-atomic acquire
  (`link`/`O_EXCL`); the holder is alive iff its process (or its socket) is — not "it has been more than
  N minutes". Three separate lock rebuilds shipped destructive bugs from the clock assumption: a grace
  window produced two holders under an 11-second stall, and a hold ceiling broke a live holder's lock
  and went on to `rm -rf` the collection it was mid-sync against.

Write these into a **path-scoped rule** for the harness directory (`.claude/rules/<harness>.md`,
`paths: ["<harness dir>/**"]`) — each interlock with the driven bug behind it — so they load exactly when
someone edits the harness (`./rules.md`). Back up the user's real data store before the first destructive
run on a new machine.

## Wiring into `/work`

- **Phase 1** — a criterion about the running system names its harness step as the oracle.
- **Phase 3** — that step is written and seen RED before implementation, like any oracle; building the
  harness is in scope when none exists.
- **Phase 6** — whether reviewers may run the harness is a per-project decision recorded in the
  harness rule (default: the orchestrator runs it; the executing reviewer reads `report.json` and may
  run non-destructive verbs only) — a reviewer's `Bash` grant reaches the wipe otherwise.
- **Phase 8** — run the harness (or the runbook) before landing a behavior change; paste the report line
  into Evidence; a red run is a stop-and-ask trigger.
- **Phase 9** — an escaped bug that the harness would have caught becomes a step, not a story.
