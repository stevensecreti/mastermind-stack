# Engineering Conventions

The standards reviewers enforce, published so they can be learned before they're violated rather than discovered through review comments. This is the same rule set the `code-review-dna` reviewer applies, written as forward-facing standards instead of review questions. Language- and framework-agnostic; apply each through the conventions of your actual stack.

The unifying principle behind all of it: **one source of truth, in the right place, with nothing extra.**

## Architecture & API

- Every piece of shared state or derived value has exactly one source of truth. Consume it from its owner; never recompute, mirror, or re-store it elsewhere. Shared clients/connections are instantiated once and reused, never re-created.
- One responsibility per module, class, function, or component. Split distinct concerns into named units. Every abstraction earns its existence: no value-less wrappers, no indirection with a single trivial caller, no single-use helper that belongs inline.
- Reusable/shared code contains zero project-specific content (it could drop into another project unchanged). Project-specific code doesn't over-generalize for reuse that will never come.
- A public interface extends the correct base type, exposes only what it should, never redeclares inherited members, makes required-vs-optional deliberate, and groups related parameters rather than flattening them into parallel top-level ones.
- No speculative code (YAGNI). It ships when a ticket asks for it.
- Reuse or extend existing code instead of recreating it. When a change replaces something, the replaced code is deleted in the same change.
- API schema, server implementation, client types, and docs stay in sync. Services own only their own config, are invoked over their defined interface, and don't take on another service's concerns.
- Metadata lives on the data structure it describes, not a parallel mapping object.

## Implementation

- State that mirrors a source of truth and re-syncs it is a defect where the value could be derived or consumed directly. No lifecycle/effect hook where initialization or inline derivation suffices. One variable per fact. No side effects buried in state-update logic. Prefer the platform's built-in state primitives over hand-rolled equivalents.
- Failures surface as values the caller can handle, not unhandled throws; catch blocks don't swallow-then-crash; user-facing failures surface through the app's feedback mechanism, not stdout/console; required env/config fails fast naming every missing key; confirm an error is the expected type before treating it as one.
- Server-owned values (totals, taxes, derived prices, IDs) are computed on the trusted side only. Validation/normalization happens at the trust boundary via a schema-validation library. LLM integrations use structured/schema-enforced outputs, never generate IDs, and use the cheapest sufficient model.
- No committed secrets, no error/stack detail exposed to the client, no PII in analytics, no policy-violating external resources, no remote-URL imports of executable code.
- Watch performance: no values recomputed each invocation that could be hoisted/cached; cache per actual need; no timers driving needless repeated work; bound queries at the source; fire side effects only on success paths.
- No arbitrary timeouts/sleeps; a magic delay masks a real synchronization problem.

## Types & Naming

- No type escape hatch (a cast, an `any`/dynamic-typing hole, or a loose/partial type) used to bypass the type system instead of modeling the real shape. A cast usually means something upstream is mistyped; fix that root cause. Use library-exported types before rolling your own.
- Types live in dedicated, discoverable locations near usage, exported where consumers need them, defined once. Named constants/enums over repeated magic literals. Use the language's null-aware operators where falsy-but-valid values (`0`, `false`, `""`) must be preserved.
- Follow the language's casing conventions consistently. Booleans read as predicates (`is`/`has`/`should`), adjective-first. Files named for what they export; index/barrel files are re-export only. Names semantic, not implementation-derived; self-describing over terse; functions verb-first. No misleading names (a `DEFAULT_*` fed by env, a `render*` that doesn't render).

## Tests, Docs & Observability

This area is historically under-enforced in review; it is a standard, not an afterthought.

- New behavior carries tests: unit where possible, integration where necessary, plus the corner cases. Don't weaken a test's expectations to make it pass.
- New/changed public API is documented in the same change (README, API/reference docs, changelog with a descriptive entry; component stories if the project uses them). Public interfaces carry doc comments in the language's format. Docs link to source rather than copy-pasting types; no unverifiable claims.
- Non-trivial logic is observable: appropriate logging levels, and metrics/tracing where the operation crosses a service boundary or matters to ops.

## Hygiene & Process (automate this, see `ci/`)

- No dead code, unused imports/files/assets/exports, debug artifacts, empty files, or stray debug/print statements. No AI residue (prompt comments, formatter/codegen artifacts, unreviewed generated scaffolding). No TODO notes without tickets.
- Lint/format/CI green; never suppress a linter or dead-code tool with ignore patterns; fix the root cause.
- No superfluous changes outside the change's stated intent; protected files (lockfiles, build/CI config, registry config) untouched unless necessary.
- Right-sized changes (see `LARGE_PR_WORKFLOW.md`); description matches the diff. No typos in user-facing copy.

Most of this last tier should be enforced by tooling, not humans. See `ci/` for the lint/CI templates that offload it.
