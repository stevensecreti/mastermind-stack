---
name: interview-user
description: Structured interview to fill knowledge gaps before starting a task or when a significant implementation decision arises. Use at task scoping, requirement gathering, and major decision points, not for every micro-ambiguity during execution.
---

# Interview User

Systematic gap-filling before proceeding. Do NOT begin work with unresolved ambiguities.

## Process

1. **Identify gaps**: analyze request → list what's known → list what's unknown
2. **Batch questions**: use the host's structured user-input mechanism when available: 1-4 questions per call, preferring fewer broader questions. Give each question 2-4 options with one context-dependent recommendation and brief trade-off rationale. If structured input is unavailable, ask the same concise questions directly.
3. **Process answers**: if any answer is ambiguous or contradicts a prior answer, follow up immediately
4. **Repeat until all gaps are filled**: no contradictions remain and there is enough information to proceed confidently. The active agent decides when the interview is complete; no explicit user signal is required.
5. **Summarize and proceed**: briefly restate key decisions made, then begin work.

## Rules

- Never proceed with assumptions when you could ask instead
- Never ask questions you can answer yourself from codebase context or project knowledge
- Prefer multi-select when choices aren't mutually exclusive
- Front-load the most consequential decisions: ask architecture/scope questions before implementation details
