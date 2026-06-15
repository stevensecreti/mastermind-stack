# Coverage Axis: Tests, Docs & Observability

This axis is NOT a replication of the historical reviewer's posture. It is the deliberate backstop for three areas the review-posture analysis found consistently under-enforced: tests, docs, and observability rarely appeared in the source comments. The single-reviewer `code-review-dna` skill stays faithful to the historical style and omits this; the multi-axis `deep-review` adds it to be *better* than the solo posture.

Apply the same voice (VOICE.md) and the same restraint: only flag a real, traceable gap. "Add more tests" with no specific missing case is exactly the kind of generic padding VOICE.md prohibits.

## Tests

Questions to ask:

- Does new behavior have **any** test? If a feature, endpoint, unit, or component was added or changed and no test touches it, name the specific untested path.
- Are the **corner cases** covered, not just the happy path? Empty states, error branches, boundary values, the conditions the code itself special-cases.
- Right level: **unit where possible, integration where necessary**. Flag integration-heavy tests for pure logic, and missing integration tests for cross-boundary behavior (API + data store, service-to-service).
- Were any existing test **expectations weakened or deleted** to make a suite pass? (This is a hard `bug:`; the historical reviewer caught it when visible; the coverage axis looks for it deliberately.)
- For non-functional requirements that matter to this change: is there a test for the performance/limit characteristic the code claims (pagination caps, concurrency limits)?

## Documentation

Questions to ask:

- Is new/changed **public API documented in the same change** (README, API/reference docs, component stories (if the project uses them), changelog entry) and is the changelog description substantive (what changed and what it enables), not a bare subject line?
- Do **public interfaces / exported types carry doc comments** in the language's format, for editor/IntelliSense support?
- Is the documentation **accurate and free of unverifiable claims** (benchmark numbers without a source, "supports X" with no example)? Does it **link to source** rather than copy-pasting types/code that will drift?
- Are user-facing strings and docs free of typos and grammar errors?

## Observability

Questions to ask:

- Does non-trivial logic that can fail have **appropriate logging** at the right level (through the proper logger, not stray prints, and not silent)? Are error paths observable to operators without leaking detail to end users?
- Where an operation **crosses a service boundary**, matters to ops, or has a failure mode that would be invisible in production, are there **metrics or tracing**, or at least a structured log that would let someone diagnose it?
- Is anything that silently swallows a failure (empty catch, fallback to a fabricated default) flagged for at least logging the swallowed condition?

## Severity guidance

- Missing tests on genuinely new logic, weakened test expectations, and undocumented breaking public-API changes → blocking (`bug:` for the weakened test, `principle:`/directive for the rest).
- Missing doc comments, thin changelog descriptions, missing observability on non-critical paths → `nit:`/`enhancement:`.
- Never invent a coverage gap to look thorough. If tests, docs, and observability are adequate for the change, this axis returns nothing.
