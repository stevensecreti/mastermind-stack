---
name: ui-explore
description: Explore the design space for a UI when you're not sure which direction to take. Generates 3+ meaningfully different coded variations, each a distinct design philosophy rather than a cosmetic tweak, so you can see the directions side by side and commit to one (or a hybrid). Framework- and design-system-agnostic; all exploration is done in code. Hand the chosen direction to ui-refine to polish it. Triggers: "explore design directions", "show me a few variations", "I'm not sure what direction to take this", "/ui-explore".
disable-model-invocation: true
user-invocable: true
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, AskUserQuestion, WebSearch, WebFetch
model: opus
argument-hint: <component-path OR "concept description">
---

# UI Explore

Reach for this when you don't yet know where a design should go. It generates several genuinely different directions as working code, so you can explore the space and commit to one. Once you've picked, hand that direction to `ui-refine` to polish it to excellence.

You are the **Design Lead**. You orchestrate the exploration with **Agent Teams teammates** and operate in **delegate mode**: you spawn the teammates who build the variations, you don't build them yourself. All exploration is done in code, never as mockups or documents.

This skill is framework- and library-agnostic. Detect the project's actual stack (framework, component library, design system, animation approach) and instruct teammates in those terms; never assume React, Chakra, Storybook, or any specific tool. Project specifics come from `mastermind.config.json` with sane fallbacks.

## Hard rules

1. **Never write or edit component code yourself.** You spawn teammates who do that. Your own tool use is limited to reading files, grep/glob discovery, bash for the dev server, web research, and spawning/messaging teammates.
2. **Never use `Agent()` subagents.** Use Agent Teams teammates exclusively.
3. **Use delegate mode** (Shift+Tab) after spawning. If it's unavailable, self-enforce rule 1 strictly.
4. **All teammates use Opus.**

## Prerequisites

1. Verify `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set. If it's unavailable, fall back to solo mode (see Error Recovery).
2. Verify a dev server (or component workshop such as Storybook, if the project uses one) is running or can be started, so each variation is viewable. Note the URL.

## Step 1: Detect mode

Determine whether `$ARGUMENTS` is a **file path** (an existing component to take in new directions) or a **concept description** (something new).

```bash
test -f "$ARGUMENTS" && echo "PATH_MODE" || echo "CONCEPT_MODE"
```

- **Path mode**: the component exists; use it as context for the variations.
- **Concept mode**: something new; proceed.

If the concept is vague (fewer than ~10 words, no clear component type), ask clarifying questions first: where it lives in the app, who uses it, the key interactions. Determine whether the component involves animation (any motion); if so, give teammates animation-specific guidance.

## Step 2: Research

Before generating anything, gather context and inspiration.

1. **Detect the project stack** (assume nothing): read root `CLAUDE.md` + `mastermind.config.json` and the project manifest to identify the UI framework, component library / design system, styling approach, and animation library. Glob existing UI directories. Record the stack.
2. **Web research**: search for 3-5 distinct design directions for this component type. Look for current trends, award-winning work (Awwwards, Dribbble, Behance), and non-obvious approaches.
3. **Codebase scan**: existing UI patterns (`Grep`), conventions, file locations.
4. **Load variation strategies**: read `${CLAUDE_PLUGIN_ROOT}/skills/ui-explore/references/VARIATION_STRATEGIES.md` for the differentiation matrix and archetypes.
5. **Load the project's design system / design guidance** if it documents one. If not, lean on the grading criteria below.
6. **Read the quality bar**: load `${CLAUDE_PLUGIN_ROOT}/skills/ui-explore/references/GRADING_CRITERIA.md` so variations are built to a real standard, and check accumulated taste at `.claude/mastermind/design-preferences.md` if it exists.

Synthesize a **brief** with 3 distinct design philosophies. Each gets a descriptive name (not "A/B/C") and a 1-2 sentence description. The whole point is that they differ **directionally**, across multiple dimensions at once (density, visual weight, hierarchy strategy, personality, structure), not just surface color swaps. Use the differentiation matrix in VARIATION_STRATEGIES.md to keep them genuinely far apart.

## Step 3: Generate the variations

Spawn an Agent Teams team with 3 Opus teammates, one per philosophy, named `variation-{philosophy-slug}`. Each teammate's prompt includes:

- The concept (or existing-component context), the brief, and their assigned philosophy
- **The detected stack**: "Build in {framework}; use {component library / design system} and its tokens exclusively; do not introduce a different UI library." Point them at the design-system doc if one exists.
- The grading criteria they're building toward
- If animated: framework-neutral animation guidance ("write an ASCII storyboard comment of the full sequence, keep all timings in named constants, group per-element motion config, drive it by explicit stages, use whatever animation primitive the project already uses").
- Reuse instruction: "Reuse existing design-system primitives wherever they fit; duplicating an existing component is a defect. {If the project uses Storybook (config `storybook`): see `${CLAUDE_PLUGIN_ROOT}/skills/ui-explore/references/STORYBOOK_PATTERNS.md` and query the Storybook MCP documentation tools before writing new components.}"
- Deliverables:
    1. The component implementation
    2. A stable, viewable URL for it (a route, a workshop/Storybook story, or a scratch preview page). Give it a descriptive, philosophy-based name (`MinimalistHero`, not `VariantA`).
    3. A summary at `/tmp/variation-{slug}.md`: the philosophy, the key decisions, and what makes it distinctive.

Wait for all 3, then shut the team down.

## Step 4: Selection

Read each `/tmp/variation-*.md` summary. Present via AskUserQuestion:

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
| Agent Teams unavailable | **Solo mode.** Build the 3 variations yourself, sequentially, each at its own preview URL. |
| Dev server won't start | Debug the build. Fall back to a component workshop (if any) or a static preview page. |
| Teammate crashes | Spawn a replacement with the same prompt. |
| Concept is vague | Clarify before Step 2: where in the app, who uses it, the key interactions. |
| Only 1 viable philosophy seems to fit | Still generate 3. Stretch the space; the point is to see real alternatives. |
