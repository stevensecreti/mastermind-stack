---
name: refactor
description: Improve code structure and maintainability without changing observable behavior. Use subtractive mode to remove complexity, duplication, dead paths, and premature abstraction; use restructure mode to split responsibilities and introduce only the seams or patterns the code has earned.
---

# Refactor

Improve structure and maintainability without changing observable behavior. Choose the mode before touching code and resist mixing modes without a demonstrated need.

## Modes

**Subtract.** Minimize footprint. Remove superfluous code, redundancy, dead paths, verbose constructs, and premature abstraction. Search for existing helpers before authoring another one.

**Restructure.** Introduce missing structure: extract a method or class, separate responsibilities, restore an abstraction boundary, replace sprawling conditional dispatch, introduce a parameter object, or apply an earned design pattern.

Choose subtract when tightening a completed change or answering “is this too much?” Choose restructure for demonstrable structural friction such as repeated cross-file logic, long mixed-abstraction methods, or god objects. When both are necessary, establish the right shape first and then remove the excess.

Over-abstraction and under-abstraction are both defects. Do not add a pattern without demonstrated complexity, and do not collapse a real repeated domain concept into ad hoc copies.

## Scope

- **Diff-scoped:** inspect changed lines and the surrounding code. Report lines removed and functions consolidated.
- **Area-scoped:** inspect the named module or files, search for duplication and coupling, and sequence behavior-preserving improvements into independently verifiable steps.

If the available repository context cannot support a reuse claim, say so and narrow the claim.

## Method

1. Establish the goal, scope, and dominant mode.
2. Read the repository's governing instructions (`AGENTS.md`, `CLAUDE.md`, or equivalents), `mastermind.config.json`, and representative surrounding code.
3. Find the highest-impact structural problems: duplication, over- or under-abstraction, mixed abstraction levels, deep nesting, weak boundaries, data clumps, or misplaced responsibilities.
4. Make concrete, behavior-preserving improvements. Prefer a handful of high-impact moves over a long list of nits.
5. Validate the original public contract, relevant tests, configured checks, dependency set, and local conventions.

## Principles

- Behavior is sacred unless the user explicitly authorizes a behavior change.
- Reuse before authoring; subtract before adding.
- Patterns must earn their place by reducing demonstrated complexity.
- Match the project's idioms rather than imposing personal preferences.
- Leave justified complexity alone and state why.

## Output

For a diff-scoped subtractive review, lead with the change's purpose, concrete opportunities, before/after examples, rationale, and measurable footprint reduction. For an area-scoped restructure, give each issue, its impact, the proposed refactoring, benefits, tradeoffs, and a safe landing sequence.

Before finishing, verify that every change preserves behavior, the selected mode fits the goal, reuse claims are evidenced, and the result improves maintainability rather than merely reducing line count.
