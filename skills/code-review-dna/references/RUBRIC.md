# Review Rubric

The WHAT dimension: the principles and patterns to check, distilled from analysis of a large corpus of one engineer's real review comments. Tiers are ordered bottom-up by **cost-to-change-later**: Tier 1 issues are the most expensive to fix after merge and deserve the most review attention; Tier 4 is cheap, mechanical, and should be swept first since it requires no judgment.

This rubric is deliberately **language- and framework-agnostic**. It states principles, not the idioms of any one stack; apply each through the conventions of whatever language, framework, and libraries the project actually uses.

The single unifying principle behind every tier: **one source of truth, in the right place, with nothing extra.** When unsure whether something is worth flagging, test it against that sentence.

Severity defaults per tier are guidance, not law; a Tier 4 committed secret is still blocking.

When a violation recurs, flag **every instance**, not just the first (cross-reference after fully explaining it once). This applies with particular force to type casts that bypass the type system, magic literals that should be named constants/enums, stray debug statements, and hardcoded values: in the source corpus these were flagged per-occurrence, every time.

---

## Tier 1: API & Architecture Semantics

Focus here. Highest effort to change later; these justify `CHANGES_REQUESTED`.

Questions to ask:

- Is there exactly **one source of truth** for every piece of shared state or derived value? Is anything computed, stored, or mirrored in more than one place (a value cached in local state that shadows the data it came from, a flag recalculated across files instead of read from its owner, a client/connection re-instantiated instead of reused from its single instance)?
- Does each module, class, function, or component have a **single responsibility**? Would distinct concerns be better split into named units? Conversely, does every new abstraction *earn its existence* (no value-less wrappers, no indirection with a single trivial caller, no single-use helper that should be inline)?
- Is the **abstraction boundary** respected: reusable/shared code contains zero project-specific content (could it be dropped into another project unchanged?), and project-specific code doesn't over-generalize for reuse that will never come?
- Does the **public interface** extend the correct base type, expose only what it should, avoid redeclaring inherited members, and make required-vs-optional deliberate? Are related parameters grouped rather than flattened into parallel top-level ones?
- Is anything **speculative**? (YAGNI: unconsumed config, placeholder surfaces, "for future use" methods, stubs.) Delete it; it can be added when a ticket asks for it.
- Does this **duplicate existing code or logic** that should have been reused or extended instead of recreated? (Recreation is both wasted effort and a second interface to maintain.)
- If this change **replaces** something (a module, util, pattern, asset), is the replaced code deleted in the same change? No parallel old/new implementations left behind.
- Are **cross-layer shapes in sync**: API schema ↔ server implementation ↔ client types ↔ docs? Field renames propagated everywhere?
- Are **service boundaries** respected? Each service owns only its own config/env, is invoked over its defined interface (not reached into as a library), and doesn't take responsibility for another's concerns.
- Does metadata live **on the data structure it describes**, not in a parallel mapping object (the route on the step, the handler on the item)?
- Does the change actually **satisfy the ticket/design spec**? Link to the exact spec lines when shapes diverge. Does the change description accurately describe only the work done?

## Tier 2: Implementation Semantics

Default severity: blocking when wrong, `unclear:` when intent is ambiguous.

Questions to ask:

- Is it **logically correct**? Inverted conditionals, wrong state-machine transitions, comparisons that can never be true, tests whose expectations were deleted rather than fixed.
- **State discipline**: is any state mirroring a source of truth and re-syncing it, where the value could be derived or consumed directly? Any lifecycle/effect hook that correct initialization or inline derivation would eliminate? Two variables tracking one fact? Side effects buried inside state-update logic? Prefer the platform's built-in state primitives over hand-rolled equivalents.
- **Error handling**: catch blocks must not swallow-then-rethrow in a way that crashes; failures should surface as values the caller can handle, not unhandled throws; user-facing failures surface through the app's feedback mechanism, not stdout/console; required env/config fails fast with every missing key named; confirm an error is the expected type before treating it as one.
- Are **server-owned values** (totals, taxes, derived prices, IDs) computed only on the trusted/server side? Is input validation/normalization done at the trust boundary using a schema-validation library rather than hand-rolled checks? Are any LLM integrations using structured/schema-enforced outputs, never asked to generate IDs, and pinned to the cheapest sufficient model?
- **Security**: committed secrets/credentials/tokens (instant block, rotate the secret); error or call-stack detail exposed to the client; PII in analytics; policy-violating external resources; remote-URL imports of executable code.
- **Performance**: values recomputed on every invocation that could be hoisted or cached; needless caching/memoization (and missing it where the access pattern demands it); timers driving needless repeated work; queries fetching everything to show a page (bound it at the source with pagination/limits); analytics or side effects firing on failure paths.
- Any **arbitrary timeout/sleep**? Always question it; a magic delay is usually masking a real synchronization problem.

## Tier 3: Types & Naming

Default severity: `convention:` or unlabeled directive; `bug:` when a type masks a real defect.

Questions to ask:

- Any **type escape hatch**, a cast, an `any`/dynamic-typing hole, or a loose/partial type, used to **bypass the type system instead of modeling the real shape**? A cast usually means something upstream is mistyped; find and fix that root cause rather than papering over it.
- Could a **library- or framework-exported type** be used instead of a hand-rolled one?
- Are types in **dedicated, discoverable locations** near their usage, exported wherever consumers need them? Shared types defined once, never redefined per file?
- **Named constants/enums over repeated magic literals**; union/variant types defined once and reused.
- **Naming**: follow the language's casing conventions consistently (e.g. distinct conventions for values vs. types/classes); booleans read as predicates (`is`/`has`/`should`, adjective-first); files named for what they export; index/barrel files are re-export only; names are semantic rather than implementation-derived (`Features`, not `FeatureScroll`); self-describing over terse (`filterableColumns`, not `filterable`); functions are verb-first (`getCurrentData`, not `currentData`); no misleading names (a `DEFAULT_*` fed by env vars, a `render*` that doesn't render, a `validate*` that doesn't validate).
- Use the language's **null-aware operators** where falsy-but-valid values (`0`, `false`, `""`) must be preserved, rather than truthiness checks that discard them.

## Tier 4: Hygiene & Process

Sweep first; mechanical, no judgment needed. Default severity: unlabeled one-word directive. Never skip the sweep, but never let it crowd out Tiers 1-2.

Questions to ask:

- Any **dead code**: commented-out code, unused imports/variables/files/assets/exports, debug artifacts, empty files? Any stray debug/print/log statements (route them to the proper logger or feedback mechanism, or delete)?
- Any **AI residue**: leftover prompt/instruction comments ("// Center the description box"), formatter or codegen artifacts, boilerplate the tooling emits by default, wholesale generated scaffolding left unreviewed? Fix the root cause too (editor/formatter/generator config), not just the instances.
- **TODO notes without tickets**? Make a ticket or delete the note.
- **Lint/format/CI green**? Never suppress a linter or dead-code tool with ignore patterns; the warning means something is actually wrong. Fix the root cause.
- Any **superfluous changes** outside the change's stated intent (drive-by formatting, unrelated refactors, cosmetic dependency bumps)? Ask for a revert. Are protected files (lockfiles, build/CI config, registry config) modified without necessity?
- **Docs/changelog in sync** with the API change, in the same change, with descriptive entries (and component stories, if the project uses them)? Do docs link to source instead of copy-pasting types? No unverifiable claims ("benchmark" numbers with no source)?
- **Public interfaces documented**: exported public types/interfaces carry doc comments in the language's format for editor/IntelliSense support?
- Is the **change itself right-sized**? Unrelated features split out; description matches the actual diff.
- Typos in user-facing copy (titles, labels, error messages); grammar in visible strings.

---

## Verdict rules

- Any Tier 1 architectural misconception, real `bug:`, committed secret, failing CI, or unaddressed prior round → **request changes**, with a numbered next-steps list when there are 3+ items.
- Substantive but non-architectural findings → **approve with comments**; this is the default outcome (most reviews with comments still approve).
- Nothing worth saying → **approve with a bare "LGTM"**. Do not invent comments to look thorough; a meaningful fraction of clean changes deserve zero comments.
