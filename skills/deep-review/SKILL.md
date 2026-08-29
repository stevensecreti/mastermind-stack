---
name: deep-review
description: >
  This skill should be used when the user asks for a "deep review", "thorough
  multi-pass review", "full review", or wants a PR/diff reviewed across every
  axis at once. It fans out five specialized review workers in parallel
  (architecture, implementation, types & naming, hygiene, and
  tests/docs/observability coverage), then merges their findings into one review.
  For a fast single-pass review in the historical style, use code-review-dna.
---

# Deep Review (multi-axis)

Run a complete code review by dispatching one specialized worker per review axis in parallel, then merging. This is the thorough counterpart to the single-pass `code-review-dna` skill: it trades speed and token cost for breadth and depth, and it adds a coverage axis that the historical solo posture under-enforced.

The shared references define the review content. Read `../code-review-dna/references/RUBRIC.md`, `../code-review-dna/references/COVERAGE.md`, and `../code-review-dna/references/VOICE.md` before dispatching work.

## Step 1: Determine surface and gather context (once)

Same as the single-pass skill. PR mode (a PR ref given and `gh` authenticated) uses `gh pr view --json ...` plus `gh pr diff`. Local mode uses `git diff <merge-base>...HEAD` plus `git status`. Gather the diff, changed-file paths, PR/ticket text, governing instructions (`AGENTS.md`, `CLAUDE.md`, or equivalent), conventions, and lint/CI configuration once. Pass the same context to every worker.

## Step 2: Fan out the five axes

Read the five axis skills named below. Use the host's native parallel-subagent or collaborator mechanism to run all five concurrently. Give each the same payload: diff, changed-file paths, and PR/ticket context. If parallel workers are unavailable, run the five procedures sequentially in the current session and preserve the same separation of concerns.

| Agent | Axis | Rubric source |
|---|---|---|
| `axis-architecture` | API & Architecture Semantics | RUBRIC.md Tier 1 |
| `axis-implementation` | Implementation Semantics (correctness, state, error handling, security, perf) | RUBRIC.md Tier 2 |
| `axis-types-naming` | Types & Naming | RUBRIC.md Tier 3 |
| `axis-hygiene` | Hygiene & Process (CI-aware) | RUBRIC.md Tier 4 |
| `axis-coverage` | Tests, Docs & Observability | COVERAGE.md |

For very large changes (>40 changed files), each axis may need to focus on the highest-signal files. The same threshold trips the large-change circuit breaker below.

## Step 3: Merge

Collect the five structured outputs and merge:

- **Deduplicate cross-axis overlap.** The same line can draw findings from multiple axes (a misnamed parameter that's also an architectural smell). Keep one comment per issue, attributed to the most specific axis, carrying the **highest** severity any axis assigned it.
- **Order** findings by tier priority (architecture → implementation → types/naming → coverage → hygiene), then by file.
- **Voice consistency pass.** Even though every worker read VOICE.md, do a final pass to ensure uniform register and obey all prohibitions. Drop any finding that does not trace to a rubric or coverage question.
- **Verdict.** REQUEST_CHANGES if any axis returned it for a real blocking issue; otherwise APPROVE_WITH_COMMENTS if there are findings; otherwise APPROVE with a bare "LGTM". Do not inflate nits into a block, and do not soften a real block to be polite.

## Step 4: Deliver

- **Local mode:** write a merged markdown report (`deep-review-<branch>.md`) grouped by file, each finding tagged with its axis and severity, ending with the verdict and review body. Optionally include a short per-axis summary line so the user sees which axes fired.
- **PR mode:** posting follows the same policy as `code-review-dna`, governed by `reviewAutoPost` (default `true`). Auto-post the merged review directly **unless** the verdict is `REQUEST_CHANGES` on a PR you don't own (compare `gh pr view --json author` against `gh api user --jq .login`), or `reviewAutoPost` is `false`, in which case present the full review and wait for explicit confirmation. Post as a single `gh api .../reviews` call with `body`, `event`, and `comments[]`. When auto-posting, print the posted review and PR URL afterward. Never post emoji or prohibited content.

## Oversized changes (circuit breaker)

If the change trips the large-change threshold (>~40 files or >~800 changed lines), do not produce 100+ inline comments. Instead, run the axes, then deliver a **thematic** review: the 3-6 recurring problems with one representative example each, and recommend a split or a pairing session. This mirrors the policy in the project's `LARGE_PR_WORKFLOW.md` and keeps the review useful instead of overwhelming.

## Relationship to the single-pass skill

`code-review-dna`: one review worker, fast, faithful to the historical solo posture. Use for quick passes.
`deep-review`: five axes, parallel when supported, broader and deeper, including the coverage backstop. Use for substantial changes, pre-merge gating, or when thoroughness matters more than latency.
