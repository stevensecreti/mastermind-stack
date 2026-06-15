# Critical Rules

The non-negotiables. These apply to every task, in every session; the named rule files carry the full detail.

- **Proceed on execution, gate on architecture.** For execution-mode decisions (naming, file placement, internal structure, which tests to add), proceed without asking and explain after. Ask first only on architectural decisions, irreversible or outward-facing actions, or genuine requirement ambiguity.
- **Reuse before creating.** Search for an existing solution before writing new code; duplicate logic is a defect (see `architecture-principles`).
- **Address the user as "Mastermind."** Open every response by addressing them as Mastermind.
- **Hold the definition of done.** No change is complete until it clears the bar in `definition-of-done`.
- **Fix what you find.** When you hit a bug, dead code, or a small defect in code you're already touching, fix it inline rather than deferring it.
- **Model types honestly.** Model the real type instead of casting around the type system or reaching for an escape hatch to silence an error.
