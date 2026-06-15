# Large-Change Workflow (the circuit breaker)

Past a certain size, an inline-comment review stops being a review and becomes compensation for a missing upstream control: the sheer volume of comments is itself the signal that something should have been caught earlier. Marathon reviews don't scale, frustrate both sides, and correlate with review tone decaying under load. This policy defines the control.

## The threshold

A change trending past **~40 changed files or ~800 changed lines** trips the circuit breaker. (`ci/pr-size-guard.yml` surfaces this automatically and non-blockingly.)

## What to do instead of 100+ inline comments

When a change trips the threshold, switch review mode:

1. **Thematic review, not line-by-line.** Write one structured review identifying the 3-6 recurring problems (e.g. "redundant state across these files", "this should use the shared handler", "types belong in a dedicated location") with one representative example each, rather than flagging every instance inline. The author applies the pattern fix across the change themselves.
2. **Pair instead of comment.** For architectural divergence (the usual cause of marathon changes), a short pairing session resolves what dozens of inline comments and several rounds cannot. Make escalation to synchronous discussion the default for big changes, not the fallback.
3. **Request a split.** If the work isn't tightly coupled, ask for it to be broken into changes grouped by what's actually coupled.

## Upstream prevention (cheaper than any review)

- **Draft-stage checkpoint.** For any feature expected to be large, a quick review of the *approach* at draft/skeleton stage costs a few comments and prevents the post-hoc marathon. Extend the design-review habit to implementation.
- **Tone safeguard.** Large changes are exactly where review tone tends to decay. Review comments are a permanent written record; the standing rule is that comment register stays constant regardless of the change's quality. The circuit breaker removes the conditions that produce tone decay in the first place.

## Net effect

Combined with the CI offload (`ci/`), this targets the two weakest points of a high-volume reviewing practice: manual hygiene (now handled by tooling) and tone-under-load (the conditions that erode it stop occurring). Breadth (tests/docs/observability) is addressed separately by the `axis-coverage` reviewer and the corresponding tier in `CONVENTIONS.md`.
