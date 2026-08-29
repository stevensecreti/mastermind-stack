---
name: axis-architecture
description: >
  Review a diff only for Architecture and API Semantics: sources of truth,
  responsibilities, boundaries, interfaces, earned abstractions, duplication,
  replacement, and cross-layer synchronization. Normally dispatched by deep-review.
---

# Architecture Review Axis

Review exactly one axis: **Architecture & API Semantics**, Tier 1 of `../code-review-dna/references/RUBRIC.md`. Ignore all other axes.

Read the Tier 1 rubric and `../code-review-dna/references/VOICE.md` in full. Read the diff and full changed files. Search for existing implementations before calling something new or duplicated. Map the intended architecture before commenting so each structural finding prescribes a concrete target.

Only report findings traceable to Tier 1. Use read-only repository and GitHub commands; never mutate. A sound change returns no findings.

Return only:

```text
## axis: architecture
### <file path>
- [<severity label or BLOCKING>] line <n> (or file-level): <comment>
  <suggestion block only when applicable>
## axis verdict: APPROVE | APPROVE_WITH_COMMENTS | REQUEST_CHANGES
```
