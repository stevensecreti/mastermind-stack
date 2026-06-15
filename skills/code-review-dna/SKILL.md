---
name: code-review-dna
description: >
  This skill should be used when the user asks to "review this PR", "review my
  changes", "do a DNA review", "review this branch", or wants a fast code review
  in Steven's style. It orchestrates a structured single-pass review against a
  rubric of principles (references/RUBRIC.md) written in a specific voice
  (references/VOICE.md), dispatching analysis to the dna-reviewer agent. For a
  thorough multi-axis review, use deep-review instead.
version: 0.1.0
---

# Code Review DNA

Perform a code review that applies a fixed rubric of principles in a fixed voice. Two reference documents define the review completely; read both before reviewing and do not deviate from them:

- `${CLAUDE_PLUGIN_ROOT}/skills/code-review-dna/references/RUBRIC.md`: WHAT to look for (four tiers, ordered by cost-to-change-later, plus verdict rules)
- `${CLAUDE_PLUGIN_ROOT}/skills/code-review-dna/references/VOICE.md`: HOW to say it (question/directive calibration, severity labels, mechanics, hard prohibitions)

The rubric and voice are language- and framework-agnostic; apply them through the conventions of whatever stack the project uses.

## Step 1: Determine the review surface

- A PR number/URL was given, or the user references "this PR" and `gh` is authenticated (`gh auth status`) → **PR mode**.
- Otherwise → **local mode**: review the working diff against the merge base (`git merge-base HEAD origin/<default-branch>`), or staged/unstaged changes if no branch divergence exists.

## Step 2: Gather context

PR mode:
```bash
gh pr view <ref> --json number,title,body,author,baseRefName,headRefName,files
gh pr diff <ref>
```
Local mode: `git diff <merge-base>...HEAD` plus `git status` for untracked files.

In both modes, also collect: the linked issue/ticket text if referenced, the repo's `CLAUDE.md`/contributing conventions if present, and the full contents of any changed file whose diff hunks are insufficient to judge architecture (Tier 1 questions usually require whole-file reads, not hunks).

## Step 3: Dispatch to the reviewer agent

Launch the `dna-reviewer` agent with: the diff, the PR/ticket context, paths of changed files (the agent reads full files itself), and explicit pointers to RUBRIC.md and VOICE.md. For very large changes (>40 changed files), fan out one agent per cohesive area (by layer or by feature directory) and merge their findings, deduplicating cross-file repeats into cross-references ("Same comment as above but for `X`").

The agent returns structured findings:

```
- file, line (or file-level), severity label, comment body, optional suggestion block
- verdict recommendation: APPROVE | APPROVE_WITH_COMMENTS | REQUEST_CHANGES
- review body text
```

## Step 4: Compose the verdict

Apply the verdict rules at the bottom of RUBRIC.md. Do not soften REQUEST_CHANGES into comments to be polite, and do not invent findings on clean code; a bare "LGTM" approval is a legitimate outcome.

## Step 5: Deliver

**Local mode**: write a markdown report (`review-<branch>.md` in the repo root, or print inline if short) listing findings grouped by file, each with severity label, line reference, comment, and suggestion where applicable, ending with the verdict and review body.

**PR mode**: posting is governed by `reviewAutoPost` in `mastermind.config.json` (default `true`) plus the verdict and PR ownership:

- **Auto-post, no interruption**: when `reviewAutoPost` is not `false`, post the review directly. This is the default for approvals, comment-only reviews, and anything on your own branch/PR.
- **Confirm first**: when the verdict is `REQUEST_CHANGES` **and** the PR is not yours (compare `gh pr view <ref> --json author` against `gh api user --jq .login`), OR when `reviewAutoPost` is `false`. Show the complete proposed review (inline comments + verdict + body) and wait for explicit approval before posting.

Post as a single review:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/reviews -X POST \
  --input review.json
```

where `review.json` contains `body`, `event` (`APPROVE` / `COMMENT` / `REQUEST_CHANGES`), and `comments[]` with `path`, `line`, `side`, `body`. Use the platform's suggestion syntax inside comment bodies for concrete fixes. When you auto-post, print the posted review and the PR URL afterward so it stays visible and reversible. Never post emoji or content violating VOICE.md prohibitions.

## Invariants

- Hygiene sweep (Tier 4) runs first because it is mechanical, but Tier 1-2 findings dominate the review's substance.
- Read changed files holistically before commenting; map the target architecture first, then comment (prescribe concrete names for extractions: "New unit: `SearchBar`").
- Every comment must trace to a rubric question. If it doesn't, delete it.
- The voice prohibitions in VOICE.md (no emojis, no slang, no padding, no personal criticism, no ungrounded generic advice) override everything, including patterns observed in the historical corpus.
