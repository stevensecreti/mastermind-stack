---
name: rebase-resolve-push
description: Rebase current branch onto latest remote main, resolve conflicts intelligently, force-push
disable-model-invocation: true
allowed-tools: Bash(git *), AskUserQuestion
---

## Steps

1. Run `git fetch origin main` to ensure latest remote state.
2. Run `git rebase origin/main`. If no conflicts, skip to step 5.
3. For each conflict:
   - Read the conflicted file(s) and understand both sides.
   - **Auto-resolve** when both changes can coexist (e.g. additions in different sections, non-overlapping edits). Keep both sides' intent intact.
   - **Ask the user** (via `AskUserQuestion`) only for truly mutually exclusive conflicts where you can't preserve both. Present the two versions clearly and let the user decide.
   - After resolving, `git add` the file(s) and `git rebase --continue`.
   - Repeat until rebase completes.
4. After all conflicts are resolved, verify the build still looks sane: check for obvious syntax errors in conflicted files (dangling conflict markers, broken imports).
5. Run `git push --force-with-lease` to update the remote branch.
6. Summarize: how many commits rebased, how many conflicts hit, how each was resolved.