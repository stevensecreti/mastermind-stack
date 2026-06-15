---
name: deep-review
description: >
  This skill should be used when the user asks for a "deep review", "thorough
  multi-pass review", "full review", or wants a PR/diff reviewed across every
  axis at once. It fans out five specialized axis agents in parallel
  (architecture, implementation, types & naming, hygiene, and
  tests/docs/observability coverage), then merges their findings into one review.
  For a fast single-pass review in the historical style, use code-review-dna.
version: 0.1.0
---

# Deep Review (multi-axis)

Run a complete code review by dispatching one specialized agent per review axis in parallel, then merging. This is the thorough counterpart to the single-pass `code-review-dna` skill: it trades speed and token cost for breadth and depth, and it adds a coverage axis (tests/docs/observability) that the historical solo posture under-enforced.

The two shared reference docs define the review content; the axis agents read the relevant slice of each:
- `${CLAUDE_PLUGIN_ROOT}/skills/code-review-dna/references/RUBRIC.md`: Tiers 1-4
- `${CLAUDE_PLUGIN_ROOT}/skills/code-review-dna/references/COVERAGE.md`: the coverage axis
- `${CLAUDE_PLUGIN_ROOT}/skills/code-review-dna/references/VOICE.md`: voice + prohibitions (all agents)

## Step 1: Determine surface and gather context (once)

Same as the single-pass skill. PR mode (a PR ref given and `gh` authenticated) → `gh pr view --json ...` + `gh pr diff`. Local mode → `git diff <merge-base>...HEAD` + `git status`. Gather once: the diff, the changed-file paths, the PR/ticket text, and repo conventions (`CLAUDE.md`, `CONVENTIONS.md`, lint/CI config). Do this a single time; pass the gathered context to every axis agent so they don't each re-fetch it.

## Step 2: Fan out the five axis agents in parallel

Launch all five in a **single message with five Agent tool calls** so they run concurrently. Give each the same payload: the diff, changed-file paths (agents read full files themselves), and PR/ticket context.

| Agent | Axis | Rubric source |
|---|---|---|
| `axis-architecture` | API & Architecture Semantics | RUBRIC.md Tier 1 |
| `axis-implementation` | Implementation Semantics (correctness, state, error handling, security, perf) | RUBRIC.md Tier 2 |
| `axis-types-naming` | Types & Naming | RUBRIC.md Tier 3 |
| `axis-hygiene` | Hygiene & Process (CI-aware) | RUBRIC.md Tier 4 |
| `axis-coverage` | Tests, Docs & Observability | COVERAGE.md |

For very large changes (>40 changed files), each axis agent may itself need to focus on a subset; instruct them to prioritize the highest-signal files. Note that >40 files also trips the large-change circuit breaker. See "Oversized changes" below.

## Step 3: Merge

Collect the five structured outputs and merge:

- **Deduplicate cross-axis overlap.** The same line can draw findings from multiple axes (a misnamed parameter that's also an architectural smell). Keep one comment per issue, attributed to the most specific axis, carrying the **highest** severity any axis assigned it.
- **Order** findings by tier priority (architecture → implementation → types/naming → coverage → hygiene), then by file.
- **Voice consistency pass.** Even though every agent read VOICE.md, do a final pass to ensure uniform register and obey all prohibitions (no emojis, no slang, no padding, no personal criticism, no ungrounded generic advice). Drop any finding that doesn't trace to a rubric/coverage question.
- **Verdict.** REQUEST_CHANGES if any axis returned it for a real blocking issue; otherwise APPROVE_WITH_COMMENTS if there are findings; otherwise APPROVE with a bare "LGTM". Do not inflate nits into a block, and do not soften a real block to be polite.

## Step 4: Deliver

- **Local mode:** write a merged markdown report (`deep-review-<branch>.md`) grouped by file, each finding tagged with its axis and severity, ending with the verdict and review body. Optionally include a short per-axis summary line so the user sees which axes fired.
- **PR mode:** posting follows the same policy as `code-review-dna`, governed by `reviewAutoPost` (default `true`). Auto-post the merged review directly **unless** the verdict is `REQUEST_CHANGES` on a PR you don't own (compare `gh pr view --json author` against `gh api user --jq .login`), or `reviewAutoPost` is `false`, in which case present the full review and wait for explicit confirmation. Post as a single `gh api .../reviews` call with `body`, `event`, and `comments[]`. When auto-posting, print the posted review and PR URL afterward. Never post emoji or prohibited content.

## Oversized changes (circuit breaker)

If the change trips the large-change threshold (>~40 files or >~800 changed lines), do not produce 100+ inline comments. Instead, run the axes, then deliver a **thematic** review: the 3-6 recurring problems with one representative example each, and recommend a split or a pairing session. This mirrors the policy in the project's `LARGE_PR_WORKFLOW.md` and keeps the review useful instead of overwhelming.

## Relationship to the single-pass skill

`code-review-dna`: one agent, fast, faithful to the historical solo posture (no coverage axis). Use for quick passes.
`deep-review`: five agents, parallel, broader and deeper, includes the coverage backstop. Use for substantial changes, pre-merge gating, or when thoroughness matters more than latency.
