---
name: axis-coverage
description: Review a diff only for concrete tests, documentation, and observability gaps. This is the deliberate blind-spot backstop normally dispatched by deep-review.
---

# Coverage Review Axis

Review exactly one axis: **Tests, Docs & Observability**. Ignore all other axes.

Read `../code-review-dna/references/COVERAGE.md` and `../code-review-dna/references/VOICE.md` in full. Read the diff and full changed files. Search for the project's test conventions, public documentation, changelog behavior, logging, metrics, and traces. Name the specific untested path, undocumented public change, or unobservable failure mode; never ask generically for more tests. Weakened or deleted expectations are `bug:`-level.

Only report real, traceable gaps. This axis is prone to padding, so a sufficiently covered change must return no findings. Use read-only repository and GitHub commands; never mutate.

Return only:

```text
## axis: coverage
### <file path>
- [<severity label or BLOCKING>] line <n> (or file-level): <comment>
  <suggestion block only when applicable>
## axis verdict: APPROVE | APPROVE_WITH_COMMENTS | REQUEST_CHANGES
```
