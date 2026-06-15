# Evaluation Methodology

The evaluator follows a diagnostic sequence, not a scoring checklist. The goal is to SEE what's actually there, name it precisely, and produce feedback the generator can act on. Scores come last, as a summary.

## Diagnostic Sequence

Follow every step in order. Do not skip or merge steps.

### Step 0: Context

Before critiquing, establish:

- **What is this?** App type, screen purpose, target user.
- **What emotional context surrounds this task?** Stressful? Casual? High-stakes? Routine?

This matters. A project planning canvas demands different care than a settings page. Name the context so the critique respects it.

### Step 1: First Impressions

One paragraph on gut reaction. What stands out? What feels off? What's the overall impression? Be honest and direct.

This is the "noticing" step. The skill of seeing what's actually in front of you, not what you expect to see.

### Step 2: Visual Design

Audit each dimension. For every issue found, use this structure:

> **[Issue name]**: [Specific factual observation]. [Impact on user or experience]. [What it could be instead.]

Be precise. Count things. Quote text. Name colors (hex values). Measure relative sizes. "There are four distinct background colors competing for attention" beats "too many colors."

| Dimension                        | What to Look For                                                                                                                                                          |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Color intentionality**         | Is every color used with purpose? Or are colors decoration without meaning? Too many background colors, competing accents, colors that don't establish hierarchy.         |
| **Typographic hierarchy**        | Is there a clear scale from most to least important? Count distinct sizes/weights. Are headlines distinguished from body from labels?                                     |
| **Shadow & stroke quality**      | Are shadows crisp or muddy? Are strokes too prominent, competing with content? Do borders add structure or noise?                                                         |
| **Visual weight vs. importance** | Do the heaviest elements match what's semantically most important? Or do decorative elements steal attention from primary actions?                                        |
| **Spacing & alignment**          | Is spacing consistent? Elements aligned to a clear grid? Excess padding pushing content away from where it belongs? Does spacing follow a rhythmic system (4px/8px grid)? |
| **Icon consistency**             | Same family? Same weight/stroke width? Same optical size? Or a mix of styles?                                                                                             |

### Step 3: Interface Design

Frame issues as missed opportunities: "We're missing an opportunity to [reward progress / reduce cognitive load / set expectations / etc.]"

| Dimension                  | What to Look For                                                                                                         |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Focusing mechanism**     | Is it clear where the user should look first? Is there a visual entry point? Or does everything compete equally?         |
| **Progressive disclosure** | Is complexity revealed gradually, or dumped at once? Are there 40 things on screen when 5 would suffice?                 |
| **Information density**    | Is density appropriate for context? Data dashboards can be dense; onboarding should not be.                              |
| **Expectation setting**    | Does the user know what happens next? Is progress communicated? Is scope clear?                                          |
| **Feedback & reward**      | Does the interface acknowledge user actions? Are completed items celebrated or just checked off?                         |
| **Redundancy**             | Are labels or descriptions repeating information the user already knows? Can anything be removed without losing meaning? |

### Step 4: Consistency & Conventions

| Dimension                    | What to Look For                                                                                                               |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Pattern consistency**      | Are similar actions handled the same way throughout? Or do patterns shift without reason?                                      |
| **Component reuse**          | Elements that look like they should be the same component but aren't? Inconsistent card styles, button treatments, list items? |
| **Visual language cohesion** | Does the interface feel like one designer made it? Or assembled from different kits?                                           |

### Step 5: User Context

- **How does this design make the user feel?** Name the emotion: overwhelmed, confused, unsupported, confident, focused.
- **What is the user's likely state of mind?** What are they trying to accomplish?
- **Does the interface respect that state?** Or does it add unnecessary cognitive burden?
- **What would "uncommon care" look like here?** What would surprise the user with thoughtfulness? Empty states, error messages, transitions, loading states, edge cases.

---

## Canvas-Specific Diagnostics

Apply ONLY to components that render to `<canvas>` or implement spatial/diagrammatic UIs. These supplement the standard diagnostics above, not replace them.

### Spatial Clarity

| What to Look For                    | Signs of Problems                                                         |
| ----------------------------------- | ------------------------------------------------------------------------- |
| Node/element spacing and alignment  | Overlapping elements, inconsistent gaps between siblings                  |
| Visual grouping of related elements | Related nodes have no spatial proximity or shared background              |
| Connection/edge routing clarity     | Edges cross over unrelated nodes, bezier curves overlap                   |
| Label placement and readability     | Labels truncated, overlapping other elements, unreadable at overview zoom |
| Zoom-level legibility               | Elements unreadable at either overview OR detail zoom level               |
| Minimap accuracy                    | Minimap doesn't reflect actual canvas state or is missing                 |

### Interaction Affordances

> **[Issue]**: [What the user sees/doesn't see]. [What they'd expect based on platform conventions]. [Specific fix.]

| What to Look For                       | Anti-patterns                                                  |
| -------------------------------------- | -------------------------------------------------------------- |
| Click/drag target sizing               | Targets < 24px with no expanded hit area                       |
| Cursor changes on interactive elements | No cursor change on hover over draggable/clickable elements    |
| Drag operation feedback                | Element doesn't move with cursor, no ghost/preview during drag |
| Selection state visibility             | Selected element looks identical to unselected                 |
| Zoom/pan indicators                    | No indication of current zoom level or canvas bounds           |
| Undo affordances                       | No visible undo for destructive canvas operations              |
| Snap/alignment guides                  | No snap-to-grid or alignment guides during drag                |
| Context menus                          | Right-click context menu absent where expected                 |

### Canvas Feedback

| What to Look For        | Signs of Problems                                                         |
| ----------------------- | ------------------------------------------------------------------------- |
| Selection highlighting  | Can't tell which elements are selected (single or multi)                  |
| Connection previews     | Creating connections shows no preview of the proposed edge                |
| Operation confirmations | Destructive operations happen silently with no confirmation               |
| Error indicators        | Invalid operations (e.g., incompatible node connections) show no feedback |
| Loading states          | Async canvas operations show no progress indication                       |

---

## Animation Diagnostics

Apply when the component includes motion/animation. These supplement the standard diagnostics.

| Dimension              | What to Look For                                                                                                      |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Timing readability** | Can you mentally narrate the animation sequence? Or is it a blur of simultaneous motion?                              |
| **Stage clarity**      | Are there clear stages (entrance → settle → interactive)? Or does everything happen at once?                          |
| **Spring quality**     | Do springs feel natural or mechanical? Is there appropriate bounce vs. overdamping?                                   |
| **Stagger rhythm**     | Are staggered animations rhythmic and predictable? Or random-feeling?                                                 |
| **Purpose**            | Does every animation serve a purpose (guide attention, communicate state, provide feedback)? Or is motion decorative? |
| **Performance**        | Any jank, dropped frames, or layout thrashing during animation?                                                       |

---

## Voice Rules

### BE

- **Specific**: "There are six columns of data per row" not "there's a lot of data"
- **Decisive**: "This is overwhelming" not "this might feel overwhelming"
- **Factual first**: State what you see before judging it
- **Impact-aware**: Connect every observation to how it affects the user
- **Constructive**: Every problem gets paired with an opportunity or direction
- **Quantitative**: Count elements, name colors (hex), measure relative sizes

### DO NOT

- **Hedge**: No "maybe," "perhaps," "it could be argued that"
- **Be vague**: No "the design feels off" without saying exactly what and why
- **Add praise padding**: Don't sandwich criticism with empty compliments
- **Prescribe without reasoning**: Never say "change X to Y" without explaining why

---

## Anti-Patterns (Auto-Flag)

If the evaluator sees any of these, name them explicitly. These cap scores in their respective categories.

**Visual design anti-patterns (cap Design Quality at 5):**

- Purple/violet gradients over white cards
- Unmodified framework / component-library default styling
- Generic "SaaS blue" (#3182CE or similar) as primary color
- Symmetrical card grids with identical shadow/radius/padding
- Hero sections with centered text + gradient background + "Get Started" CTA
- Gratuitous glassmorphism or blur effects with no functional purpose

**Canvas anti-patterns (cap Interaction Affordances at 5):**

- Tiny click/drag targets (< 24px) with no expanded hit area
- No cursor change on hover over interactive elements
- Drag operations with no visual feedback
- No visible selection state
- Zoom/pan with no level indicator

---

## Score Summary (Secondary Output)

After the full diagnostic critique, produce a score summary table for trajectory tracking across iterations.

| Category                | Score | One-Line Justification                             |
| ----------------------- | ----- | -------------------------------------------------- |
| Design Quality          | N/10  | {key finding from Visual Design audit}             |
| Originality             | N/10  | {how distinctive vs. generic framework defaults}   |
| Craft                   | N/10  | {spacing, typography, shadow, alignment quality}   |
| Functionality           | N/10  | {usability, affordances, feedback quality}         |
| Spatial Clarity         | N/10  | {canvas only: layout readability and organization} |
| Interaction Affordances | N/10  | {canvas only: discoverability and feedback}        |
| Animation Quality       | N/10  | {if applicable: timing, staging, spring quality}   |

**Calibration:**

- 5/10: functional but generic, nothing memorable.
- 7/10: good, intentional decisions visible, minor issues.
- 9/10: exceptional, portfolio-worthy.
- 10/10: almost never. Museum-quality only.

Scores must be justified by specific findings from the diagnostic steps above, not vibes. If you can't point to a specific observation that justifies a score, re-evaluate.

---

## Top Opportunities (Primary Output)

After scores, rank the **3-5 highest-impact changes**, each as one sentence following the pattern:

> **{Priority N}: {Issue name}**: {specific change} because {specific impact on user experience}.

These are what the generator receives as its primary improvement signal. They must be concrete enough that a developer can implement them without further clarification.

**Severity ordering:**

1. **Structural**: Problems with information architecture, mental model, missing functionality. Changes what the interface IS.
2. **Behavioral**: Problems with response, flow, communication. Changes how the interface FEELS.
3. **Visual**: Problems with color, type, spacing, shadows. Changes how the interface LOOKS.
