# Storybook Patterns for Design Variations

> **Optional.** This reference applies only when the project uses [Storybook](https://storybook.js.org) as its
> component workshop (gated by the `storybook` key in `mastermind.config.json`). The `ui-design` skill works
> without Storybook too: any running dev-server / preview URL that Playwright can navigate is sufficient.
>
> Storybook supports many frameworks (React, Vue, Svelte, Angular, Web Components, …). Everything below is
> framework-neutral; write story files using **your project's** Storybook renderer (`@storybook/react`,
> `@storybook/vue3`, `@storybook/svelte`, etc.) and **your design system's** components; never assume a
> specific UI library.

## Storybook MCP Server

When Storybook dev is running, the official `@storybook/addon-mcp` exposes an MCP endpoint (default `http://localhost:6006/mcp`). Three toolsets ship with it: `dev`, `docs`, and `test`. Reach for these tools before grepping the codebase or fabricating a new component.

**Dev toolset:**
- `get-storybook-story-instructions`: guidance on how to write useful stories (which props to capture, how to write interaction tests). Call this BEFORE authoring a new story.
- `preview-stories`: returns story previews in chat (when supported), otherwise links. Use to share visual context without screenshotting.

**Docs toolset:**
- `list-all-documentation`: index of every component and docs entry. The first stop for "does this already exist?" before writing anything new.
- `get-documentation`: detailed docs for a component (props, first stories). Use to study a candidate for reuse.
- `get-documentation-for-story`: full story + docs for one story. Use when you need the rendered example.

**Test toolset:**
- `run-story-tests`: runs the test runner against specific stories, returning results including accessibility issues. Use for a focused a11y/component check scoped to the stories you changed, instead of the full suite.

### When to use the MCP server vs direct file editing

| Goal | Reach for |
| --- | --- |
| Discover whether a component already exists | `list-all-documentation` |
| Study a candidate component before reusing | `get-documentation`, `get-documentation-for-story` |
| Write a new story | `get-storybook-story-instructions` first, then author the file directly |
| Share rendered context with a teammate | `preview-stories` |
| Verify a11y on stories you just changed | `run-story-tests` scoped to those stories |
| Edit a component or story file | Direct file editing (`Read`/`Edit`/`Write`); MCP is for discovery and validation, not writing |

If Storybook dev is not running, fall back to grep/glob over the project's story files (e.g. `**/*.stories.*`).

---

## Story organization

Group design-exploration stories under a dedicated top-level hierarchy so they don't pollute the production story tree:

```
Design Variations/
├── Components/   (Button, Card, Navigation, …)
├── Sections/     (Hero, Features, Pricing, …)
└── Pages/        (Landing, Dashboard, Settings, …)
```

Story title format: `Design Variations/[Category]/[ComponentName]`.

## Per-variation stories

For each variation, author one story whose `meta.title` places it in the hierarchy above. In the story:

- Render the variation component with representative `args`.
- In the story's docs description, capture the **design philosophy**, **core principles**, **visual characteristics**, and **best-for** context; this is what the user reads when choosing a direction.
- Name variations by philosophy (`Minimalist`, `BoldTypography`, `Interactive`), never `VariantA`.

## The CompareAll story (REQUIRED)

Always include one `CompareAll` story per component that renders **every variation side by side** in a comparison grid, each labeled with its name and a few descriptive tags, plus a short "which direction best represents the brand?" prompt. This is the primary surface the user reviews during Phase 3.5 selection. Scale variations down (e.g. a CSS `transform: scale(...)`) so several fit on screen at once.

## Useful addons

These Storybook addons are framework-agnostic and pull their weight for design work:

- `@storybook/addon-a11y`: accessibility checks per variation
- `@storybook/addon-themes`: light/dark (and other theme) switching
- `@storybook/addon-docs`: autodocs / MDX descriptions
- `@storybook/addon-links`: inter-story navigation
- `@storybook/addon-mcp`: the MCP server described above (`{ toolsets: { dev: true, docs: true, test: true } }`)

Configure responsive viewports (mobile / tablet / desktop) in `.storybook/preview.*` so the evaluator can check breakpoints.

## Monorepos

If the project is a monorepo, point the Storybook `stories` glob at each package's story files (e.g. `../../<package>/src/**/*.stories.@(ts|tsx|js|jsx|vue|svelte)`) and keep one shared `.storybook/` config. Co-locate variation stories with their components (`<Component>.variations.stories.*`) to keep them out of the production story set.
