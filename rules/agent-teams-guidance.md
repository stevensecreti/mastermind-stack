# Agent Teams & Parallel Work

## When to Use What

- **Single session**: sequential work, <10 tool calls, tight file coupling
- **Subagents**: self-contained tasks where only the result matters, no inter-worker communication needed, isolating verbose output
- **Agent Teams**: 3+ independent parallel streams, cross-cutting concerns, workers need to coordinate. Default choice for multi-subtask implementation work.

## Lead Role

The lead orchestrates. It does NOT implement. Use delegate mode (Shift+Tab).

- Decompose objective into self-contained, file-disjoint subtasks before spawning workers
- Assign each worker explicit file ownership boundaries in the spawn prompt
- Workers do NOT need the full plan. Give them just: what to build, where it goes, and any interface contracts. The lead retains all architectural context, rationale, and cross-cutting details. Over-briefing workers wastes tokens and invites off-script decisions.
- Monitor progress, synthesize outputs, resolve interface mismatches
- All implementation decisions stay with the lead. Workers escalate, lead decides
- Wait for all workers to finish before integration. Never code alongside workers.

## Task Decomposition

Each subtask must be:
- **Self-contained**: produces a clear deliverable (function, component, test file)
- **File-disjoint**: no two tasks touch the same file. Two workers editing one file = overwrites. If tasks share an interface, define the contract up front and assign one side per worker.
- **Context-minimal**: workers get ONLY what they need to execute: task spec, file paths, interface contracts. NOT the full plan, NOT the rationale, NOT the conversation history. Excess context invites workers to freelance.
- **Right-sized**: aim for 5-6 tasks per worker

Declare ownership explicitly in spawn prompts:
```
"You own ONLY: src/components/UserCard.tsx, src/components/UserCard.test.tsx. Do not modify files outside this set."
```

## Model Routing

| Complexity | Model | Examples |
|---|---|---|
| Trivial/mechanical | Haiku | File moves, renames, boilerplate, import updates |
| Standard implementation | Sonnet | New functions, components, tests, well-defined features |
| Complex/architectural | Opus | Cross-module refactors, security-sensitive, significant judgment required |

Default to Sonnet. Require plan approval before implementation for Opus-tier tasks.

## Worker Rules

1. Do exactly what the task says, nothing more, nothing less. You don't have the full picture and that's by design. Don't interpret, infer, or freelance beyond the explicit task spec.
2. Stay within file ownership boundaries
3. Escalate ambiguities/decisions to the lead. Never decide unilaterally
4. Peer-message only for direct interface dependencies with a specific other worker. All else goes through lead.
5. Finish and stop. Don't look for extra work or refactor adjacent code.

## Anti-Patterns

- Teaming tasks a single session handles in 5 minutes
- More than 5 workers (coordination cost > parallelism gain)
- Vague tasks ("handle the frontend") vs actionable specs with file paths + contracts
- Workers making architectural decisions
- Lead implementing alongside workers
- Two workers editing the same file
