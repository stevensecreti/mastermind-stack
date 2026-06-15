---
name: naturalize
description: Make AI/agent-written code read like a careful human contributor wrote it in THIS codebase. A diff-scoped, behavior-preserving pass that strips AI "tells" (narration comments, ceremonial defensive guards, `any` escape hatches, generic boilerplate/naming, needless nesting) and conforms new code to the surrounding code's idiom, density, and conventions. Use after an agent or AI assistant writes/edits code (before committing or opening a PR) when the code works but reads machine-generated. NOT for structural or architectural changes (use `refactor` for those).
---

# Naturalize

Make freshly written AI/agent code **indistinguishable from code a careful human expert would have written in this specific codebase.** Code can be perfectly correct and even well-structured and still *read* machine-generated: over-commented, over-defensive, generically named, stylistically off from its neighbors. This skill closes that gap.

Two non-negotiables:

- **Behavior is unchanged.** This is a stylistic/idiom pass. The only exception is fixing a clear, incidental bug you stumble on, and you call it out explicitly.
- **You conform to the codebase.** "Better" here means "reads like the rest of *this* code." Read before you cut.

## Boundary with `refactor`

| | `naturalize` (this skill) | `refactor` (agent) |
|---|---|---|
| Question | "Does this **read** like this codebase / like a human wrote it?" | "Is this **well-built**?" |
| Touches | comments, naming, idioms, density, local style, ceremonial code | structure: complexity, duplication, abstractions, SOLID, patterns |
| Changes structure? | **No** | Yes |

If a fix requires restructuring (split a god function, introduce/collapse an abstraction, change a design), **stop and hand it to `refactor`.** Naturalize changes how code *reads*, never how it's *organized*.

## Step 1: Learn the local idiom (before changing anything)

Read the changed files and 2-3 representative neighbors/siblings. Note the codebase's actual conventions, because these become your target:

- Comment density and voice (terse? none? doc-comments only on public APIs?)
- Naming conventions (length, casing, domain vocabulary)
- Error-handling style (throw? Result types? where guards live)
- Type strictness (how `any`/casts/`unknown` are treated)
- Control-flow habits (early returns vs nesting, guard clauses)
- Import ordering, formatting, async/promise style, test style

If the codebase is internally inconsistent, target the **dominant** local pattern; don't invent a new one.

## Step 2: Scan the diff for AI tells

Work only on the branch's changes (the diff vs. main, or uncommitted edits). Look for:

- **Narration comments**: comments restating what the code obviously does (`// increment counter`), echo-the-signature docstrings, leftover `TODO`/placeholder/“Note:” cruft. Cut to match local density.
- **Ceremonial defensiveness**: `try/catch`, null guards, optional chaining, or fallback defaults on paths that are trusted or can't fail here. **Keep guards that cover real edge cases; cut the ones that only exist because the model is cautious.** This is judgment.
- **Type escape hatches**: `any` / `as any` / `@ts-ignore` / loose casts used to silence the compiler instead of modeling the type. Replace with the real type. If a proper type needs structural change, flag it for `refactor` rather than forcing it here.
- **Generic boilerplate & naming**: placeholder names (`data`, `result`, `temp`, `handleClick2`), kitchen-sink helpers, verbose constructs where the codebase uses a terser idiom, or a re-implementation of an existing utility (small, obvious reuse is fair game; large extractions are `refactor`'s job).
- **Needless nesting**: arrow-of-doom that this codebase would express as early returns / guard clauses, *only if* that matches local style.
- **Idiom drift**: async/promise style, import ordering, formatting, or API choices that diverge from the surrounding code.

## Step 3: Conform, minimally

- Make the smallest edits that bring the code in line with its neighbors. Focused diffs, never broad rewrites.
- Match the file's established comment density, naming, error-handling, and patterns.
- Re-verify behavior is unchanged after each edit.

## Guardrails

- **Don't strip necessary defensiveness.** Distinguish ceremonial guards from ones covering genuine edge cases; when unsure, keep it and note the uncertainty.
- **Don't impose a foreign style.** No personal preferences: only the codebase's conventions.
- **Don't drift into structure.** The moment a fix is architectural, it belongs to `refactor`.
- **Don't manufacture work.** If the code already reads natural, say so and stop.

## Output

A concise report: the tells found and the conformance edits made (`file:line`, what / why), plus a 1-3 sentence summary. Note anything handed off to `refactor` and any incidental bug fixed.
