---
name: mastermind-setup
description: One-time setup for the mastermind-stack plugin in a consuming repo. Detects the project's check/test commands, docs, and component-workshop setup, writes a mastermind.config.json, and installs the house-style rules (which plugins cannot auto-load) into .claude/rules or CLAUDE.md. Use when the user says "set up mastermind", "configure mastermind-stack", "wire in the house rules", or right after installing the plugin in a new repo.
---

# Mastermind Setup

Bind the generic mastermind-stack tooling to *this* project. The plugin ships skills and agents that auto-load on install, plus a few things that need explicit wiring per repo:

1. **A config file** (`mastermind.config.json`) so de-coupled skills/rules know the project's check/test commands, docs location, etc.
2. **The house-style rules** (`architecture-principles`, `definition-of-done`, `agent-teams-guidance`, `critical-rules`): Claude Code does **not** auto-load a plugin's `rules/` directory, so they must be copied into the project (or imported from `CLAUDE.md`) to take effect.
3. **(Optional) process artifacts** (`recommendations/`): engineering conventions, the large-change workflow, and CI templates that complement the code-review skills.

Run this skill once when the plugin lands in a new repo. It is idempotent: re-running re-detects, shows a diff, and only changes what's stale.

## Step 1: Detect project specifics

Inspect the repo and propose values for each config key. Confirm with the user before writing; never guess silently.

| Key | What it controls | How to detect |
|---|---|---|
| `checkCommand` | Lint/typecheck gate in `definition-of-done` | `Makefile` target `check`; else `package.json` scripts (`check`/`lint`/`typecheck`); else `pyproject.toml`/`ruff`/`mypy`; else ask |
| `testCommand` | Test gate in `definition-of-done` | `Makefile` target `test`; else `package.json` `test`; else `pytest`/`go test`/etc.; else ask |
| `storybook` | Gates the "UI components need stories" DoD requirement and Storybook-MCP reuse hints | `true`/path if `.storybook/` exists or `storybook` is a dependency; else `false` |
| `docsDir` | Where `definition-of-done` expects docs to live | `docs/` if present; else the obvious docs root; else `null` |
| `coverageMatrix` | Path to a coverage matrix to keep in sync (rare) | a coverage-matrix file/dir if one exists; else `null` |
| `reviewAutoPost` | Whether the code-review skills auto-post to GitHub in PR mode | default `true` (cook: auto-post, confirm only before a `REQUEST_CHANGES` on a PR you don't own); set `false` to confirm every post |

The config surface is intentionally small and **extensible**: add keys here as future migrated skills need them. Unset keys fall back to `CLAUDE.md` documentation, then to sane defaults.

## Step 2: Write `mastermind.config.json`

Write the confirmed values to `mastermind.config.json` at the **repo root**. Model it on the plugin's `mastermind.config.example.json` (at `${CLAUDE_PLUGIN_ROOT}/mastermind.config.example.json`). Omit keys that are `null`/not applicable rather than writing noise. If a config already exists, show a diff and merge rather than clobbering.

## Step 3: Install the house-style rules

Ask the user which mechanism they prefer (default: **copy into `.claude/rules/`**):

- **Copy**: copy `${CLAUDE_PLUGIN_ROOT}/rules/*.md` into the repo's `.claude/rules/`. Self-contained and version-controlled with the project. On re-run, diff against the plugin's current versions and offer to update.
- **Import**: append `@`-imports to the repo's `CLAUDE.md` pointing at the plugin rules (e.g. `@${CLAUDE_PLUGIN_ROOT}/rules/architecture-principles.md`). Stays in sync with plugin upgrades automatically, but couples the repo to the plugin being installed.

Either way, the four rules are: `architecture-principles.md`, `definition-of-done.md`, `agent-teams-guidance.md`, `critical-rules.md`.

## Step 4: (Optional) Offer the process artifacts

The `recommendations/` folder holds process templates that complement the `code-review-dna` / `deep-review` skills. Offer to adopt them (skip any the user declines); they need light per-repo adaptation:

- **`CONVENTIONS.md`** → publish into the repo's docs (or `.claude/`) so the standards the reviewer enforces can be learned before they're violated.
- **`LARGE_PR_WORKFLOW.md`** → the large-change circuit-breaker policy; drop into docs/contributing.
- **`ci/pr-size-guard.yml`** → copy into `.github/workflows/` to surface oversized PRs automatically (GitHub Actions).
- **`ci/examples/`** → adapt the lint-offload example to the project's own linter (the *principle*, not the specific config, is what transfers: agnosticism first).

## Step 5: Confirm

Print a short summary: the config that was written, where the rules landed, which recommendations were adopted, and a reminder that the skills/agents are already active via the plugin. Note that `mastermind.config.json` is typically committed (so the whole team shares it). Confirm with the user.
