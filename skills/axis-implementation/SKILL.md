---
name: axis-implementation
description: >
  Review a diff only for Implementation Semantics: correctness, state discipline,
  error handling, server-owned data, security, performance, and arbitrary timeouts.
  Normally dispatched by deep-review.
---

# Implementation Review Axis

Review exactly one axis: **Implementation Semantics**, Tier 2 of `../code-review-dna/references/RUBRIC.md`. Ignore all other axes.

Read the Tier 2 rubric and `../code-review-dna/references/VOICE.md` in full. Read the diff and full changed files. Trace data flow, sources of truth, error paths, client/server ownership, security boundaries, and performance-sensitive behavior. Search the repository to verify claims rather than inferring them.

Only report findings traceable to Tier 2. Logical correctness defects and committed secrets are blocking. Use read-only repository and GitHub commands; never mutate. A sound change returns no findings.

Return only:

```text
## axis: implementation
### <file path>
- [<severity label or BLOCKING>] line <n> (or file-level): <comment>
  <suggestion block only when applicable>
## axis verdict: APPROVE | APPROVE_WITH_COMMENTS | REQUEST_CHANGES
```
