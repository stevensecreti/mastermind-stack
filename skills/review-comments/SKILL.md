---
name: review-comments
description: Fetch and analyze PR review comments, implement agreed fixes, reply to disagreements, and post "addressed in <sha>" replies on fixed threads so the PR always reflects current state
disable-model-invocation: true
argument-hint: [optional extra instructions]
allowed-tools: Bash(gh *), Bash(git *)
---

## PR context

- **Metadata:** !`gh pr view --json number,title,url,headRefName --jq '{number, title, url, branch: .headRefName}'`
- **Changed files:** !`gh pr diff --name-only`
- **Review comments (inline):** !`PR=$(gh pr view --json number -q .number) && gh api "repos/{owner}/{repo}/pulls/$PR/comments" --jq '[.[] | {path: .path, line: .original_line, body: .body, author: .user.login, id: .id, in_reply_to_id: .in_reply_to_id}]'`
- **Reviews (top-level):** !`PR=$(gh pr view --json number -q .number) && gh api "repos/{owner}/{repo}/pulls/$PR/reviews" --jq '[.[] | select(.body != "") | {state: .state, body: .body, author: .user.login}]'`
- **Conversation comments:** !`gh pr view --json comments --jq '[.comments[] | {author: .author.login, body: .body}]'`
- **Review threads (with thread IDs for resolving):** !`OWNER=$(gh repo view --json owner -q .owner.login) && REPO=$(gh repo view --json name -q .name) && PR=$(gh pr view --json number -q .number) && gh api graphql -f query="query { repository(owner:\"$OWNER\", name:\"$REPO\") { pullRequest(number:$PR) { reviewThreads(first:100) { nodes { id isResolved comments(first:20) { nodes { databaseId author { login } path body } } } } } } }" --jq '.data.repository.pullRequest.reviewThreads.nodes'`

## Task

1. Parse all comments. Group inline comments by file and thread (use `in_reply_to_id`, or the `reviewThreads` data above which is authoritative).
2. For each comment, read the referenced source to understand full context.
3. Evaluate each: Agree / Partial / Disagree / Nitpick / Not applicable.
4. Output a **report table**: File, Line, Author, Comment (summary), Stance, Reasoning. Include the thread ID and the top-level comment's `databaseId` so the actions in step 5 can reference them.
5. Then take action automatically; do not wait to be asked:
   - **Agree / Partial / actionable Nitpick** → implement the fix in code.
   - **Disagree** → post a reply on that thread explaining the pushback (see commands below). Be respectful and specific; cite the code or constraint that informs the disagreement.
   - **Not applicable** (e.g., out of scope, already fixed elsewhere) → reply briefly explaining why no change is being made.
6. Once fixes are implemented, commit and push to the PR branch. Capture the resulting commit SHA (`git rev-parse HEAD` after push).
7. After the push lands, on every thread you fixed, post a reply: `Addressed in <sha>` (use the short SHA, optionally with a one-line note if the fix was non-obvious). Leave the thread open; the reviewer resolves it themselves. Do not call `resolveReviewThread`.
8. End with a short summary: how many fixed, how many pushed back on, the commit SHA, and anything still needing the user's attention.

## Commands

Reply to a review comment thread (use the top-level comment's `databaseId` as `$COMMENT_ID`):
```
gh api -X POST "repos/{owner}/{repo}/pulls/$PR/comments/$COMMENT_ID/replies" -f body="<reply text>"
```

Get the SHA to cite after pushing:
```
git rev-parse --short HEAD
```

## Rules

- Default behavior is full action (fix + push + "Addressed in <sha>" reply on fixed threads, pushback reply on disagreements). The user does not need to ask.
- Do **not** resolve threads; leave that to the reviewer. The "Addressed in <sha>" reply is the signal that the fix has landed.
- Never silently skip a comment; every comment ends up with either an "Addressed in <sha>" reply or a pushback/explanation reply.
- If `$ARGUMENTS` contains overrides (e.g. "report only", "don't push", "resolve threads"), honor them.
