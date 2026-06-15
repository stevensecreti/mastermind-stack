---
name: ui-breakdown
description: Decompose attached UI design images into a 5-tier component hierarchy (primitives, components, feature components, sections, views) and produce a comprehensive, framework-agnostic composition plan as a design document. Detects and follows whatever UI stack the project actually uses. Read-only analysis; never writes source code.
disable-model-invocation: true
argument-hint: [optional-output-path]
---

# UI Breakdown

Decompose attached UI design images into a 5-tier component hierarchy and write a comprehensive design document. The deliverable is exactly one markdown file.

This skill is **framework- and library-agnostic**. It makes no assumption about React/Vue/Svelte/Angular/etc. or any particular component library; it detects the project's actual stack (step 2) and expresses the breakdown in those terms. The taxonomy and methodology below apply to any component-based UI.

## Hard constraints

- **Read-only on source code.** Do not create, edit, or write any component or source file (any framework, any extension) during this skill. If you reach for Edit/Write/Bash on a non-markdown source file, stop; you've drifted from the skill.
- **The design document is the only artifact.** Implementation is a separate step. Producing code here is failure even if the code looks correct.
- **Every visible element in every image is accounted for** in the inventory with an explicit tier assignment. Missing elements cause missing components downstream.
- **Reuse precedes invention.** Nothing is listed as `new` until the existing-design-system audit has been performed and recorded.
- **Every view has a complete composition tree.** A tree is complete when every leaf is a primitive.

## Workflow

Advance only when the current step's gate is met. Do not collapse steps.

1. **Verify images.** Confirm images are attached and image-to-view mapping is unambiguous. If either fails, stop and ask the user to clarify or label.
2. **Detect project context** (see below). Gate: you have identified the project's UI framework, component library/design system, and conventions, and inventoried the existing UI directory structure.
3. **Resolve output path** (see below). Gate: path is confirmed with the user, or `$ARGUMENTS` was explicit.
4. **Per-image element enumeration.** For each image, walk every visible element top-to-bottom, left-to-right. Group repeats within and across images. Note state variants visible (empty, loading, error, populated, hover, focused, disabled, selected). Gate: every element on screen is named.
5. **Existing-design-system audit.** For every enumerated element, check the detected design system / component library for an existing match. Record matches by path. Gate: each element has either an existing-match path or an explicit "no existing match" note.
6. **Tier assignment.** Place every element into exactly one of the 5 tiers. For anything tiered as Feature Component or above, write the specific justification (domain coupling, business logic, contained state). Gate: zero elements untyped.
7. **Composition trees.** For every view, produce an indented tree from view down to primitives. Repeated components appear repeatedly. Gate: every tree leaf is a primitive.
8. **State & shared logic.** Identify every piece of state with type, owner, consumers, mutation path. Propose shared-logic primitives (hooks/composables/stores/services, whatever the framework uses) only where passing props/inputs is demonstrably insufficient.
9. **Write the design doc** to the resolved path using the structure below.
10. **Summarize in chat.** Report path written, per-tier component counts split as reused vs. new, and any open questions.

## Project context detection

Before analyzing images, make NO assumption about the stack; detect it:

1. Read the root `CLAUDE.md` (and any `mastermind.config.json`) and any nested `CLAUDE.md` in UI directories.
2. Read the project's manifest (`package.json`, `pubspec.yaml`, `*.csproj`, etc.): identify the UI framework, component library / design system, and any form/state/data-fetching/icon libraries.
3. Glob the existing design system: e.g. `**/components/**`, `**/design-system/**`, `**/ui/**`, `**/theme/**`, `**/tokens/**`. Read enough to know what already exists.
4. Skim any design-system or component docs (Storybook, a docs site) if present.

Record the detected stack at the top of the design doc. If no framework can be determined, ask the user rather than guessing.

## Output path resolution

1. If `$ARGUMENTS` non-empty, use it.
2. Else if a docs site exists (e.g. `docs/` with a site config), use `docs/<design-subdir>/<kebab-feature-name>.md` with that site's expected frontmatter.
3. Else if `.claude/plans/` exists, use `.claude/plans/<kebab-feature-name>-design.md`.
4. Else create `.claude/plans/` and use it.

## 5-tier component hierarchy

Every element belongs to exactly one tier. (These tiers are a component-model concept and apply to any framework.)

1. **Primitives**: unstyled or minimally-styled atoms with no domain knowledge: `Button`, `Input`, `Text`, `Box`, `Icon`, `Badge`, `Tooltip`. Usually these come straight from the project's component library or are thin styled wrappers over it.
2. **Components**: styled compositions of primitives, domain-agnostic, fully reusable: `Card`, `FormField`, `IconButton`, `TabBar`, `EmptyState`. No business logic, no app-specific data shapes.
3. **Feature Components**: domain-aware with business logic, app-specific data shapes, or shared-state coupling: `IssueCard`, `UserAvatar` (reads auth), `StatusBadge`. Reusable inside the app, not outside.
4. **Sections**: large compositions forming a region of a view (drawer panel, page header, sidebar, tab panel). Owns its internal layout and often local state.
5. **Views / Pages**: route-level or modal-level compositions wiring sections together, owning page-level state, connecting to global data.

Tier rules:

- Logic lives at the lowest tier that can hold it; never push business logic below Feature Components.
- Single-use with no domain logic stays inline in its section; do not promote to Feature Component for the sake of it.
- Follow the project's established naming conventions for components, shared-logic primitives, and modules (detected in step 2).

## Design document structure

Use this exact section order.

```
# <Feature Name>: UI Design Breakdown

## Overview
2-4 sentences: what this UI is, what views/states the images cover, user-facing goal.

## Stack & Conventions
Detected framework, component library, design-system patterns, and conventions. Cite paths to relevant existing pieces. Note anything unusual.

## Image-to-View Mapping
Table: image label/index → view name → state variant.

## Element Enumeration
Per image, the full list of visible elements walked top-to-bottom. This is the raw input to the inventory; missing elements here cause missing components later.

## Component Inventory

### Primitives
Per entry: name, source (`existing at <path>` | `<library component>` | `new`), key props/inputs, notes.

### Components
Same format. Mark existing vs. new.

### Feature Components
Same format, plus: specific domain coupling (which shared state/data deps), reuse scope.

### Sections
Same format, plus: internal layout strategy, local state owned.

### Views / Pages
Same format, plus: route or trigger, page-level state, data sources.

## Composition Trees
One indented tree per view, from view → sections → feature components → components → primitives. Every leaf is a primitive. Repeated components appear multiple times. Note key props at composition points where they materially affect rendering.

## Data Flow & State Ownership
Per piece of state:
- Name
- Type (signature in the project's language)
- Owner (component or shared store/context)
- Consumers
- Mutation path (actions/handlers that change it)

Distinguish server state, client UI state, derived state.

## Shared Logic Primitives
Per proposed hook/composable/store/service:
- Name and signature
- Responsibility (one sentence)
- Why a shared primitive vs. passing props/inputs or lifting state
- Consumers

If an existing one fits, reference it instead.

## Reusability Audit
Two lists:
- **Reused (exists)**: name → existing path
- **New (build)**: name → tier → why nothing existing satisfies it

Any "New" entry without a concrete reuse-blocker is a bug; either find the existing match or sharpen the rationale.

## Accessibility Considerations
Keyboard nav, focus management (drawers, modals, tabs), ARIA roles/labels, contrast, screen reader behavior for dynamic states.

## Integration Considerations
How this fits the existing app shell: routing, layout interactions (overlay, push, blur), conflicts with concurrent UI work, backend/data dependencies.

## Open Questions
Anything the user must clarify before implementation.
```

## Principles to enforce

- **Single responsibility.** Split layout, data fetching, and variant rendering into separate components.
- **Variant props, not component proliferation.** Empty, loading, error, and populated states are one `state` or `status` prop on one component.
- **Composition over configuration.** Prefer slot/children/compound-component APIs over giant prop bags for layout.
- **State at lowest common ancestor.** Reach for a shared store/context only for genuinely cross-cutting state, not to dodge 2-level prop drilling.
- **Prop minimalism.** List only props that materially affect behavior or appearance.

## Ambiguity handling

If images don't clearly show a state, variant, or edge case the breakdown implies, surface it under **Open Questions**. Do not fabricate UI not present in the images unless the user explicitly asks for it.
