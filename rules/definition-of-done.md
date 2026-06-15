# Definition of Done

These checks apply to EVERY code change, regardless of how it's implemented (solo, agent teams, or manual).

> **Configuration:** This rule reads project specifics from `mastermind.config.json` (run the `mastermind-setup` skill to create it). Every key is optional. Where a key is unset, fall back to what the consuming repo's `CLAUDE.md` documents, then to sane defaults. The relevant keys are called out inline below.

## Universal Requirements

**Testing**: All code changes must have adequate test coverage. Don't just test the happy path; cover error cases, edge cases, and boundary conditions.

**Coverage tracking** *(only if your project maintains a coverage matrix: `coverageMatrix`)*: When you add or change a feature, update its row in the project's coverage matrix. Every testable feature should have a row. Prefer one partial per domain so parallel branches don't conflict on a single file; never edit a generated index's tables directly. Omit this requirement entirely if the project has no such artifact.

**UI component stories** *(only if your project uses a component workshop: `storybook`)*: Every UI component that renders anything must have stories. This includes:

- New components: create the stories file alongside the component
- Existing components with new states/variants: add stories for the new states
- The component owner also owns its stories file (never split across workers)
- When a Storybook (or equivalent) MCP server is connected, use its focused story/a11y tests to validate your changed stories without booting the full test runner.

**Docs** *(`docsDir`)*: Keep project docs current.

- Update existing docs if the change affects documented behavior
- Create new docs if the change introduces new concepts, features, APIs, or architectural patterns
- Check the docs root for any references to changed functionality

**Validation**: The project's **check** and **test** commands must pass before any work is considered complete. These are `checkCommand` and `testCommand` in `mastermind.config.json` (e.g. `make check` / `make test`, `npm run lint` / `npm test`, `pnpm check` / `pnpm test`). If unset, use whatever the consuming repo's `CLAUDE.md` documents as its lint/typecheck and test entry points.

**Architectural Quality**: The implementation must be the architecturally correct solution, not merely a working one. If a simpler or more maintainable approach exists using established codebase patterns, that approach must be used. Adding special-case logic, workarounds, or conditional branches to avoid proper restructuring is a DoD failure.

## How to Apply

- For issue-driven work: bake these into the issue brief's DoD checklist automatically.
- For solo work / bug fixes: verify these before committing, proportional to the change size.
- For reviews: check these as a review dimension.
