---
name: mastermind-setup
description: One-time setup for mastermind-stack in a consuming repository. Detects check/test commands, docs, and component-workshop setup; writes mastermind.config.json; and installs house-style rules into the current host's repository instructions. Use after installing the plugin or when the user asks to set up or configure Mastermind.
---

# Mastermind Setup

Bind the generic Mastermind tooling to this project. Plugin installation makes the skills available, while repository-specific configuration and standing rules require one explicit setup pass:

1. **A config file** (`mastermind.config.json`) so de-coupled skills/rules know the project's check/test commands, docs location, etc.
2. **The house-style rules** (`architecture-principles`, `definition-of-done`, `agent-teams-guidance`, `critical-rules`) installed through the current host's repository-instruction mechanism.
3. **(Optional) process artifacts** (`recommendations/`): engineering conventions, the large-change workflow, and CI templates that complement the code-review skills.

Run this skill once when the plugin lands in a new repo. It is idempotent: re-running re-detects, shows a diff, and only changes what's stale.

## Step 1: Detect project specifics

Inspect the repo and derive values for each config key. Use unambiguous repository
evidence directly. Ask only when a value is materially ambiguous or no safe default
exists; do not turn routine detection into a questionnaire.

| Key | What it controls | How to detect |
|---|---|---|
| `checkCommand` | Lint/typecheck gate in `definition-of-done` | `Makefile` target `check`; else `package.json` scripts (`check`/`lint`/`typecheck`); else `pyproject.toml`/`ruff`/`mypy`; else ask |
| `testCommand` | Test gate in `definition-of-done` | `Makefile` target `test`; else `package.json` `test`; else `pytest`/`go test`/etc.; else ask |
| `storybook` | Gates the "UI components need stories" DoD requirement and Storybook-MCP reuse hints | `true`/path if `.storybook/` exists or `storybook` is a dependency; else `false` |
| `docsDir` | Where `definition-of-done` expects docs to live | `docs/` if present; else the obvious docs root; else `null` |
| `coverageMatrix` | Path to a coverage matrix to keep in sync (rare) | a coverage-matrix file/dir if one exists; else `null` |
| `reviewAutoPost` | Whether the code-review skills auto-post to GitHub in PR mode | default `true` (cook: auto-post, confirm only before a `REQUEST_CHANGES` on a PR you don't own); set `false` to confirm every post |

The config surface is intentionally small and extensible. Unset keys fall back to the repository's governing instructions, then to sane defaults.

## Step 2: Write `mastermind.config.json`

Write the confirmed values to `mastermind.config.json` at the repository root. Model it on `../../mastermind.config.example.json`. Omit keys that are null or inapplicable. If a config already exists, show a diff and merge rather than clobbering.

## Step 3: Install the house-style rules

Read all four files under `../../rules/` in full. Detect the active host from available repository instructions and plugin context. If both Codex and Claude Code are used in the repository, offer to install both forms.

### Codex or host-neutral setup

Install the rule bodies into the root `AGENTS.md` inside one idempotent managed block:

```text
<!-- mastermind-stack:start -->
<Mastermind heading and the four rule bodies>
<!-- mastermind-stack:end -->
```

Preserve all content outside the markers. On re-run, replace only the managed block and show its diff before writing. Create `AGENTS.md` if it does not exist. Do not use imports that the host does not support.

### Claude Code setup

Default to copying the four files into `.claude/rules/`. If the user prefers imports, append `@` imports to `CLAUDE.md` using paths valid in the consuming repository. On re-run, diff and update only stale Mastermind-owned content.

The four files are `architecture-principles.md`, `definition-of-done.md`, `agent-teams-guidance.md`, and `critical-rules.md`.

### Throughline option

Offer to install the body of `../../output-styles/throughline.md` as standing prose guidance. For Codex, add it inside the same managed `AGENTS.md` block. For Claude Code, recommend the bundled `Throughline` output style first and only copy the prose rules when the user explicitly wants repository-pinned behavior.

## Step 4: (Optional) Offer the process artifacts

The `../../recommendations/` folder holds process templates that complement the review skills. Offer to adopt them, skipping anything the user declines:

- **`CONVENTIONS.md`** → publish into the repository's docs or instruction directory so the standards are visible before review.
- **`LARGE_PR_WORKFLOW.md`** → the large-change circuit-breaker policy; drop into docs/contributing.
- **`ci/pr-size-guard.yml`** → copy into `.github/workflows/` to surface oversized PRs automatically (GitHub Actions).
- **`ci/examples/`** → adapt the lint-offload example to the project's own linter (the *principle*, not the specific config, is what transfers: agnosticism first).

## Step 5: Confirm

Print a short summary: the config written, host instruction files changed, whether Throughline was installed, and which recommendations were adopted. Note that `mastermind.config.json` and repository instruction changes are typically committed so the team shares them.
