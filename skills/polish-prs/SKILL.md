---
name: polish-prs
description: For each PR in a comma-separated list, checkout the branch, run /review-comments to address feedback (commits stay local), fix any CI failures caused by the PR's diff, then push all commits together in one update. Used for finishing PRs that Claude Code routines put up.
disable-model-invocation: true
argument-hint: <pr-num>,<pr-num>,... [extra instructions]
allowed-tools: Bash(gh *), Bash(git *), Skill, Read, Write, Edit, Grep, Glob
---

## Inputs

`$ARGUMENTS`: comma-separated PR numbers (e.g. `1405,1406,1407`), optionally followed by extra instructions to forward to `/review-comments`.

Parse the leading comma-separated integer list as the PR set. Trim whitespace around each number. Anything after the last PR number is forwarded as extra args to `/review-comments`.

## Task

For each PR number, sequentially (never in parallel: they share a working tree), execute the full polish pipeline below. Finish one PR completely before starting the next.

### 1. Checkout the PR branch

Confirm the working tree is clean first:

```
git status --porcelain
```

If dirty, stop and ask the user. Do not blow away uncommitted work. Otherwise:

```
gh pr checkout <pr-num>
```

### 2. Run `/review-comments` with deferred push

Invoke the `review-comments` skill via the Skill tool, passing args that tell it to commit locally only, skip the push, and skip the "Addressed in <sha>" replies. The polish skill will push and post those replies itself once CI fixes are also in.

Pass args like:

```
don't push yet. I'll push after CI fixes are bundled in. Skip the "Addressed in <sha>" replies and instead report back the list of thread top-level comment databaseIds you would have replied to (one per fixed thread), so I can post the replies after the combined push lands with the final SHA. Disagreement pushback replies should still be posted now. <forward any extra args from $ARGUMENTS>
```

Capture from the review-comments output:
- The list of thread `databaseId`s pending an "Addressed in <sha>" reply.
- The count of disagreement / not-applicable replies already posted.
- Whether any threads were left unactioned (escalations).

### 3. Check CI for failures caused by the PR's changes

```
gh pr checks <pr-num>
```

For each failing or erroring check, fetch logs:

```
gh run view <run-id> --log-failed 2>&1 | tail -200
```

(Fall back to `--log` if `--log-failed` is empty.) Get run IDs from `gh pr checks <pr-num> --json` or `gh run list --branch <branch> --limit 5`.

Classify each failure:

- **Caused by the PR's diff** (test in a file the PR touched, type error in a modified module, lint violation in changed code, build error from a changed import, etc.) → diagnose, fix, commit locally. Reproduce locally first when feasible.
- **Flakiness or unrelated infra** (network blip, unrelated test already failing on main, runner timeout, transient registry error) → leave it alone. Note it in the per-PR summary.

If a failure's cause is genuinely ambiguous after looking at the logs, treat it as caused-by-changes and fix it. Don't shrug off real failures as "probably flaky."

### 4. Push everything together

```
git push
```

Capture the resulting short SHA:

```
git rev-parse --short HEAD
```

### 5. Post "Addressed in <sha>" replies

For each `databaseId` captured in step 2, post the reply with the final SHA from step 4:

```
PR=<pr-num>
gh api -X POST "repos/{owner}/{repo}/pulls/$PR/comments/$COMMENT_ID/replies" -f body="Addressed in $SHA"
```

If a specific thread's fix was non-obvious, add a one-line note after the SHA. Do not resolve threads; that's the reviewer's call.

### 6. Move to the next PR

Return to step 1 with the next PR number.

## Final summary

After all PRs are processed, print one table row per PR with:

- PR number
- Threads fixed / pushback count / escalations
- CI fixes applied (and which checks they targeted)
- Flaky / unrelated CI failures flagged (left untouched)
- Final pushed SHA
- Anything still needing the user's attention

Then a one-line overall: "N PRs polished, M needing manual follow-up."

## Rules

- Sequential only. Each PR involves a branch checkout; never interleave.
- One push per PR, and only after both review-comment fixes and CI fixes are committed. No intermediate pushes.
- Do not resolve review threads; the "Addressed in <sha>" reply is the signal.
- Do not force-push unless the user explicitly asks for it.
- If `git status` is dirty before any checkout, stop and ask.
- If CI is still pending (not failed) when you reach step 3, push first anyway; checks will re-run on the new commits. Don't sit and wait. The user can re-invoke this skill on the same PR list later if a real failure surfaces.
- If a PR has no review comments at all, step 2 is a no-op; still proceed to CI check and push (push will be a no-op if there are no new commits, which is fine).
- If you hit something genuinely unexpected (rebase needed, merge conflict, branch protection block, auth failure), stop on that PR, note it in the summary, and continue to the next PR rather than blocking the whole batch.
