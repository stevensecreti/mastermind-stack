---
name: fix-ci
description: Diagnose and fix failing CI checks on the current PR. Fetches GitHub Actions results, identifies failures, implements fixes, pushes, and loops until all checks pass.
disable-model-invocation: true
argument-hint: "[max-attempts (default 5)]"
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, MultiTool
---

# Fix Failing CI Checks

You are fixing CI failures on the current PR. Your goal: **all checks green.** Work autonomously in a diagnose→fix→push→verify loop until done or max attempts exhausted.

## Current PR context

- PR info: !`gh pr view --json number,title,headRefName,url 2>/dev/null || echo "ERROR: No PR found for current branch. Abort and tell user."`
- Current check status: !`gh pr checks 2>/dev/null || echo "ERROR: Could not fetch checks."`

## Process

Set MAX_ATTEMPTS from $ARGUMENTS (default 5). Then loop:

### 1. Identify failures

Run `gh pr checks` and parse for failed/pending checks. If all checks pass, report success and stop.

### 2. Fetch failure details

For each failing check, get logs:

```
gh run view <run-id> --log-failed 2>&1 | tail -200
```

If `--log-failed` returns nothing useful, try:

```
gh run view <run-id> --log 2>&1 | tail -300
```

Extract the run ID from the check URL or use `gh run list --branch <branch> --limit 5`.

### 3. Diagnose root cause

Analyze the logs. Common categories:

- **Lint/format**: Run the project's lint/format commands locally to reproduce, then fix
- **Type errors**: Read the referenced files, fix type issues
- **Test failures**: Run the failing test(s) locally first to confirm, then fix
- **Build errors**: Check imports, missing deps, syntax errors
- **CI config issues**: Check workflow YAML if the failure is infra-related

Always reproduce locally before pushing a fix when possible.

### 4. Implement fix

Make targeted changes. Do NOT refactor unrelated code. Run the relevant local validation (lint, typecheck, test) to confirm the fix before pushing.

### 5. Push

```
git add -A && git commit -m "fix: resolve CI failures - <brief description>" && git push
```

### 6. Wait and verify

After pushing, checks take time. Wait with a polling loop:

```
sleep 30
gh pr checks
```

Poll every 30s. After 3 consecutive polls with no status change, extend to 60s intervals. Cap total wait at 10 minutes per attempt. If checks are still pending after that, report status and let the user decide.

### 7. Evaluate

- **All passed** → Report success, summarize what was fixed, stop.
- **Still failing** → Decrement remaining attempts, go to step 1.
- **Max attempts exhausted** → Report what was tried, what's still failing, and suggest next steps.

## Rules

- Never force-push unless explicitly told to.
- Each commit should address a specific failure; don't bundle unrelated fixes.
- If a failure looks like a flaky test (passes locally, fails in CI with no code cause), flag it to the user rather than looping endlessly.
- If you're stuck on the same failure for 2+ attempts, stop and ask the user for guidance.
