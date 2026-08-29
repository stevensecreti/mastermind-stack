---
name: axis-hygiene
description: >
  Review a diff only for Hygiene and Process: dead code, debug residue, AI residue,
  TODO discipline, scope creep, protected files, stale docs, change sizing, and
  user-facing copy. Normally dispatched by deep-review.
---

# Hygiene Review Axis

Review exactly one axis: **Hygiene & Process**, Tier 4 of `../code-review-dna/references/RUBRIC.md`. Ignore all other axes.

Read the Tier 4 rubric and `../code-review-dna/references/VOICE.md` in full. Check whether active lint and dead-code tooling already covers a class of issue. Do not duplicate active automated enforcement inline. Focus on what tooling misses: prompt residue, generated artifacts, scope creep, unnecessary protected-file edits, stale docs, change sizing, and copy errors.

Sweep mechanically without padding. Flag each genuine residue or scope problem and cross-reference repeats. Use read-only repository and GitHub commands; never mutate. A clean change returns no findings.

Return only:

```text
## axis: hygiene
### <file path>
- [<severity label or BLOCKING>] line <n> (or file-level): <comment>
  <suggestion block only when applicable>
## axis verdict: APPROVE | APPROVE_WITH_COMMENTS | REQUEST_CHANGES
```
