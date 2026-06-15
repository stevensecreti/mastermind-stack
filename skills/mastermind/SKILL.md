---
name: mastermind
description: Router and table of contents for the mastermind-stack plugin. Maps developer intent to the right skill, agent, or house-style rule. Consult when unsure which mastermind capability fits a request, when the user asks "what can mastermind do", or when routing a vague ask ("clean this up", "review my PR", "build a UI", "set this up") to a concrete entry point.
---

# Mastermind Router

`mastermind-stack` is a portable Claude Code stack: skills + agents + house-style rules, de-coupled from any one project. This skill is the map. The harness already lists every skill's description each session; this adds the curated **intent → entry point** mapping on top so a vague request lands on the right tool.

When a request is ambiguous, match it to a row below and invoke that entry point directly. When nothing fits, fall back to normal behavior. Don't force a match.

## First run in a repo

If `mastermind.config.json` doesn't exist in the project yet, or the house-style rules aren't installed, run **`mastermind-setup`** first. It writes the config and installs the rules (which a plugin cannot auto-load).

## UI design & analysis

| You want to… | Entry point | Kind |
|---|---|---|
| Turn a UI screenshot/mockup into a framework-agnostic component hierarchy + composition plan | `ui-breakdown` | skill |
| Explore the design space when you're unsure of direction: generate several directionally different coded variations and pick one (Agent Teams) | `ui-explore` | skill |
| Refine an existing UI to excellence: an autonomous GAN generator/evaluator loop (Agent Teams + Playwright) on a component, page, section, or chosen variation | `ui-refine` | skill |

## Code quality & refactoring

| You want to… | Entry point | Kind |
|---|---|---|
| Make AI/agent-written code read like a careful human wrote it in this codebase: strip AI tells, conform to local idiom; no structural changes | `naturalize` | skill |
| Improve code without changing behavior: subtract excess (complexity, duplication, premature abstraction) and/or restructure for quality (SOLID, design patterns), on a diff or a codebase area | `refactor` | agent |

## Code review

| You want to… | Entry point | Kind |
|---|---|---|
| A fast single-pass review of a PR or local diff, in the author's rubric + voice | `code-review-dna` | skill |
| A thorough multi-axis review: five parallel axis agents (architecture, implementation, types/naming, hygiene, coverage), merged into one review | `deep-review` | skill |

Both auto-detect the surface (GitHub PR via `gh` if a ref is given and authenticated, otherwise the local diff against the merge base) and, in PR mode, **auto-post the review by default**, confirming only before a `REQUEST_CHANGES` on a PR you don't own (set `reviewAutoPost: false` to confirm every post; local-mode reports never post). The `dna-reviewer` and `axis-*` agents are dispatched by these skills, not invoked directly. Complementary process templates (conventions, large-change policy, CI guard) live in `recommendations/`; offer them via `mastermind-setup`.

## Pull requests, review & git flow

| You want to… | Entry point | Kind |
|---|---|---|
| Finish a batch of PRs (address comments, fix CI, push together) | `polish-prs` | skill |
| Fetch, address, and reply to PR review comments | `review-comments` | skill |
| Rebase onto latest main, resolve conflicts, force-push | `rebase-resolve-push` | skill |
| Diagnose and fix failing CI checks on the current PR | `fix-ci` | skill |

## Requirements & meta

| You want to… | Entry point | Kind |
|---|---|---|
| Run a structured interview to fill knowledge gaps before/while building | `interview-user` | skill |
| Turn the task we just did into a reusable, project-agnostic skill (abstract the recipe, strip incidentals) | `crystallize` | skill |
| Configure this plugin in a new repo | `mastermind-setup` | skill |

## House-style rules (always-on once installed)

These are always-on guidance, loaded into every session once `mastermind-setup` installs them:

- **`critical-rules`**: the non-negotiables every session follows (proceed on execution / gate on architecture, reuse-first, the definition of done, fix-what-you-find).
- **`architecture-principles`**: contract-first, modularity, converge-on-final-state, idempotency, reuse-first, exhaust-the-design-space, build-the-lever, fix-adjacent-problems.
- **`definition-of-done`**: testing, validation (`checkCommand`/`testCommand`), docs, stories, architectural quality, the bar every change must clear.
- **`agent-teams-guidance`**: when to use a single session vs subagents vs agent teams, lead/worker roles, task decomposition, model routing.
