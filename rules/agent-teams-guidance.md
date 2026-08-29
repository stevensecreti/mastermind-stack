# Parallel Work & Subagents

## Choose the smallest useful topology

- **Single session:** sequential work, fewer than roughly ten tool calls, or tightly coupled files.
- **Isolated subagent:** a self-contained task where only the result matters and verbose context should stay out of the main thread.
- **Persistent collaborators:** three or more independent streams, cross-cutting concerns, or work that needs coordination across rounds.

Use the host's native collaboration mechanism. If it is unavailable, preserve the same decomposition and run the work sequentially.

## Lead role

The lead owns decomposition, interfaces, integration, and decisions. When delegation is active, the lead should:

- split the objective into self-contained, file-disjoint tasks before dispatch;
- give every worker explicit file ownership and interface boundaries;
- retain architectural rationale and cross-cutting context rather than over-briefing workers;
- monitor progress and resolve interface mismatches;
- keep substantive decisions with the lead; and
- wait for all required outputs before integration.

Do not have two workers edit the same file. The lead may continue with genuinely independent local work when the host safely supports concurrent edits and ownership boundaries remain disjoint.

## Task decomposition

Each delegated task must be:

- **Self-contained:** it produces a clear deliverable.
- **File-disjoint:** one owner per writable file.
- **Context-minimal:** the worker receives its task, paths, constraints, and interface contracts.
- **Right-sized:** large enough to justify delegation, small enough to verify independently.

Declare ownership explicitly:

```text
You own only src/components/UserCard.tsx and src/components/UserCard.test.tsx. Do not modify files outside this set.
```

## Capability routing

- Mechanical work: use a fast, cost-efficient worker when the host offers model choice.
- Standard implementation: use the host's balanced default.
- Architectural or security-sensitive work: use the strongest available reasoning tier and keep approval gates around substantive decisions.

Do not hardcode provider-specific model names into durable project rules.

## Worker rules

1. Do exactly the assigned task.
2. Stay within ownership boundaries.
3. Escalate ambiguous decisions to the lead.
4. Communicate directly with peers only for explicit interface dependencies.
5. Finish the deliverable and stop.

## Anti-patterns

- Delegating work a single session can finish in minutes.
- More workers than the available independent work supports.
- Vague tasks without paths, outputs, or contracts.
- Workers making unowned architectural decisions.
- Overlapping file ownership.
- Treating parallelism as a requirement rather than a latency optimization.
