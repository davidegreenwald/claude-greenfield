# A completeness check derives its domain from the producer — never a hand-list

**Adapt, do not copy verbatim.** An always-on `.claude/rules/*` rule (no `paths:`). Install it in any
project that builds "is X exhaustive?" gates — routes, fields, channels, modules, endpoints covered.
Keep the authoring-time test and the fail-closed-on-rename mechanism; swap the dependency cruiser for
whatever your producer is.

Always-on. Governs any check that asserts *everything* in a set is handled — every file, field,
channel, module, endpoint. The set it iterates MUST come from the authoritative producer's own
output, not from a literal you typed. A hand-enumerated domain drifts from the producer, and the
check then silently inherits the exact gap it exists to close.

## The test, at authoring time

For any "is everything covered?" check, ask: *where does my notion of "everything" come from?* If the
answer is a literal — an extension list, a field list, a filename list you maintain — it will drift
from whatever actually produces the set. Point it at the tool that owns the set (the cruiser's crawl,
the type's declared fields, the router's registered routes) and derive from that. Only the part the
producer genuinely cannot express — a bounded, reason-annotated exclusion set ("which crawled modules
are deliberately unconstrained") — stays local.

## The friction this prevents (a driven incident)

A coverage check was built to end a hand-kept whitelist that under-covered the source tree. Its first
draft listed modules by a hardcoded extension glob — itself a hand-list. The crawler covered more
extensions than the glob, and the classifying rules were anchored to one extension, so a
boundary-crossing file was crawled by the cruiser, guarded by no rule, and missed by the glob — it
passed a green gate. The completeness check reproduced the very defect class it was written to kill,
one axis over. The fix: derive the module set from the cruiser's own JSON output (one source of truth
for "what is crawled") and read classification from the config's *actual* regexes, never a re-typed
copy — which also fails the check closed the moment a rule is renamed.

Two things must both hold, or the guard is a prose rule wearing a script: the enumeration is
**mechanical**, AND its **source of truth is the producer.** This generalizes the enumeration
discipline of any trust-boundary rule ("enumerate every sibling channel") to the verification
mechanism itself.
