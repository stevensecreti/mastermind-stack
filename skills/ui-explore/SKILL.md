---
name: ui-explore
description: >
  Explore the design space for a UI when you're not sure which direction to take. Generates 3+ meaningfully different coded variations, each a distinct design philosophy rather than a cosmetic tweak, so you can see the directions side by side and commit to one (or a hybrid). Framework- and design-system-agnostic; all exploration is done in code. Hand the chosen direction to ui-refine to polish it. Triggers: "explore design directions", "show me a few variations", "I'm not sure what direction to take this", "/ui-explore".
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, AskUserQuestion, WebSearch, WebFetch
---

# UI Explore

Reach for this when you don't yet know where a design should go. It generates several genuinely different directions as working code, so you can explore the space and commit to one. Once you've picked, hand that direction to `ui-refine` to polish it to excellence.

You are the **Design Lead**. When the host supports collaborators or isolated subagents, delegate each variation to a separate worker and keep the philosophies file-disjoint. When it does not, build the variations sequentially. All exploration is done in code, never as mockups or documents.

This skill is framework- and library-agnostic. Detect the project's actual stack and brief workers in those terms; never assume React, Chakra, Storybook, or any specific tool. Project specifics come from `mastermind.config.json` with sane fallbacks.

## Hard rules

1. **Keep one owner per variation.** No two workers edit the same files.
2. **When collaborators are available, the lead does not implement variations.** The lead owns context, contracts, selection, and integration.
3. **Use the host's native collaboration mechanism.** Do not hardcode one provider's tool or model names.
4. **In sequential fallback, preserve isolation.** Finish and preview one direction before starting the next.

## Prerequisites

Verify a dev server or component workshop can expose every variation at a stable URL. Note the URL before implementation.

## Step 1: Detect mode

Determine whether the provided target is a **file path** or a **concept description**.

```bash
test -f "<provided-target>" && echo "PATH_MODE" || echo "CONCEPT_MODE"
```

- **Path mode**: the component exists; use it as context for the variations.
- **Concept mode**: something new; proceed.

If the concept is vague (fewer than roughly ten words with no clear component type), ask where it lives, who uses it, and which interactions matter. Determine whether it involves motion and include animation-specific guidance when it does.

## Step 2: Research

Before generating anything, gather context and inspiration.

1. **Detect the project stack** (assume nothing): read the repository's governing instructions (`AGENTS.md`, `CLAUDE.md`, or equivalent), `mastermind.config.json`, and the project manifest. Identify the framework, design system, styling approach, and animation library. Scan existing UI directories and record the stack.
2. **Web research**: search for 3-5 distinct design directions for this component type. Look for current trends, award-winning work (Awwwards, Dribbble, Behance), and non-obvious approaches.
3. **Codebase scan**: existing UI patterns (`Grep`), conventions, file locations.
4. **Load variation strategies**: read `references/VARIATION_STRATEGIES.md` for the differentiation matrix and archetypes.
5. **Load the project's design system / design guidance** if it documents one. If not, lean on the grading criteria below.
6. **Read the quality bar**: load `references/GRADING_CRITERIA.md` and check accumulated taste at `.mastermind/design-preferences.md` if it exists.

Synthesize a **brief** with 3 distinct design philosophies. Each gets a descriptive name (not "A/B/C") and a 1-2 sentence description. The whole point is that they differ **directionally**, across multiple dimensions at once (density, visual weight, hierarchy strategy, personality, structure), not just surface color swaps. Use the differentiation matrix in VARIATION_STRATEGIES.md to keep them genuinely far apart.

## Step 3: Generate the variations

Create three workers, one per philosophy, using the host's parallel collaboration mechanism when available. Name them `variation-{philosophy-slug}` where the host supports names. Otherwise run the three worker briefs sequentially. Each prompt includes:

- The concept (or existing-component context), the brief, and their assigned philosophy
- **The detected stack**: "Build in {framework}; use {component library / design system} and its tokens exclusively; do not introduce a different UI library." Point them at the design-system doc if one exists.
- The grading criteria they're building toward
- If animated: framework-neutral animation guidance ("write an ASCII storyboard comment of the full sequence, keep all timings in named constants, group per-element motion config, drive it by explicit stages, use whatever animation primitive the project already uses").
- Reuse instruction: "Reuse existing design-system primitives wherever they fit; duplicating an existing component is a defect. {If the project uses Storybook: read `references/STORYBOOK_PATTERNS.md` and use available Storybook documentation tools before writing new components.}"
- Deliverables:
    1. The component implementation
    2. A stable, viewable URL for it (a route, a workshop/Storybook story, or a scratch preview page). Give it a descriptive, philosophy-based name (`MinimalistHero`, not `VariantA`).
    3. A summary at `/tmp/variation-{slug}.md`: the philosophy, the key decisions, and what makes it distinctive.

Wait for all three and close persistent collaborators when the host requires explicit cleanup.

## Step 4: Selection

Read each `/tmp/variation-*.md` summary. Use the host's structured user-input mechanism when available, or ask the same concise question directly:

```
question: "Which design direction should we take? (View each at its preview URL for full context.)"
type: single_select
options:
  - "{Philosophy A}: {2-sentence summary}"
  - "{Philosophy B}: {2-sentence summary}"
  - "{Philosophy C}: {2-sentence summary}"
  - "Hybrid: combine elements from multiple (I'll describe)"
```

If "Hybrid," ask which elements from which variations; you may merge the files yourself, since this is assembly, not design implementation. Set the chosen variation's primary file as the result. Archive the unselected variations to an `archived/` location and remove the `/tmp/variation-*.md` summaries.

## Hand-off

Exploration ends at a chosen direction, not a finished design. Tell the user the direction is set, then point `ui-refine` at the chosen file to run the generator/evaluator loop that polishes it to excellence.

## Error Recovery

| Scenario | Action |
|---|---|
| Parallel workers unavailable | Build the three variations sequentially, each at its own preview URL. |
| Dev server won't start | Debug the build. Fall back to a component workshop (if any) or a static preview page. |
| Worker crashes | Restart that one variation with the same brief. |
| Concept is vague | Clarify before Step 2: where in the app, who uses it, the key interactions. |
| Only 1 viable philosophy seems to fit | Still generate 3. Stretch the space; the point is to see real alternatives. |
