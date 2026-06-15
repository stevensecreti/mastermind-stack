---
name: interview-user
description: Structured interview to fill knowledge gaps before starting a task or when a significant implementation decision arises. Use at task scoping, requirement gathering, and major decision points, not for every micro-ambiguity during execution.
---

# Interview User

Systematic gap-filling before proceeding. Do NOT begin work with unresolved ambiguities.

## Process

1. **Identify gaps**: analyze request → list what's known → list what's unknown
2. **Batch questions**: use `AskUserQuestion`: 1-4 questions per call, prefer fewer broader questions to minimize rounds. Each question gets 2-4 options with one marked recommended (based on project context and best practices, not a fixed position). Include brief trade-off rationale in descriptions.
3. **Process answers**: if any answer is ambiguous or contradicts a prior answer, follow up immediately
4. **Repeat until all gaps are filled**: no contradictions remain, sufficient info to proceed confidently. Claude decides when complete, no user signal needed.
5. **Summarize and proceed**: briefly restate key decisions made, then begin work.

## Rules

- Never proceed with assumptions when you could ask instead
- Never ask questions you can answer yourself from codebase context or project knowledge
- Prefer multi-select when choices aren't mutually exclusive
- Front-load the most consequential decisions: ask architecture/scope questions before implementation details
