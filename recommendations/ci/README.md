# CI Offload

The highest-volume category of review comments is mechanical hygiene (commented-out code, stray debug statements, unused imports, formatting). That tier requires no human judgment, so it belongs in CI, not in review. Moving it there frees human review time for architecture and correctness. The `code-review-dna` hygiene axis is explicitly written to **defer** to this tooling where it runs and focus only on what linters can't catch (AI residue, scope creep, protected-file edits).

## The principle (language-agnostic)

Automate your highest-volume comment categories in **your** stack's linter / formatter / dead-code tool:

- stray debug/print statements
- unused imports and variables
- TODO/FIXME without a tracking ticket
- type escape hatches (casts/`any`/loose types that bypass the type system)
- any other mechanical rule your reviewer repeats by hand

Never suppress these tools with ignore patterns, which defeats the purpose; fix the root cause. Keep your dead-code/dead-export gate enabled and unsuppressed.

## What's here

- `pr-size-guard.yml`: a GitHub Action that comments (non-blocking) on changes exceeding a soft size threshold, nudging toward splitting before review starts. Implements the circuit breaker from `LARGE_PR_WORKFLOW.md` at the CI layer. Platform-specific to GitHub Actions; the threshold logic ports to any CI.
- `examples/`: concrete, language-specific instances of the offload principle above. Currently a JS/TS + ESLint example; adapt the idea to your own linter.
