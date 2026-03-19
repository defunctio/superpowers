---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# Finishing a Development Branch

## Overview

Guide completion of development work by publishing autonomous implementation results safely.

**Core principle:** Verify tests → publish branch updates safely → preserve recoverability.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before publishing changes, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Determine Base Branch

```bash
# Prefer the remote default branch, then fall back
git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null | sed 's@^origin/@@' || echo main
```

If that fails, try `main`, then `master`, then use the current branch's upstream merge target if available. Do not stop to ask the user.

### Step 3: Publish Autonomously

Default autonomous behavior:

1. Never merge locally by default
2. Never discard work automatically
3. Preserve the branch and publish changes instead
4. Prefer updating an existing PR; otherwise push and create one

Determine whether an open PR already exists for the current branch:

```bash
branch="$(git branch --show-current)"
gh pr view --head "$branch" --json url 2>/dev/null
```

#### If an open PR already exists for the current branch

```bash
git push
```

Report that the existing PR branch was updated.

#### If no PR exists yet

```bash
git push -u origin <feature-branch>
gh pr create --base <base-branch> --head <feature-branch> --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [x] <verification steps actually run>
EOF
)"
```

Report the PR URL.

### Step 4: Preserve Recoverability

- Do not delete the branch automatically
- Do not discard the worktree automatically
- Do not merge locally automatically
- Do not remove local recovery paths unless a higher-priority instruction explicitly requests cleanup

## Quick Reference

| Situation | Action |
|-----------|--------|
| Tests fail | Stop and report blocker |
| Open PR exists | Push updates to current branch |
| No PR exists | Push branch and create PR |
| No remote push possible | Preserve branch/worktree and report blocker |
| Cleanup desired | Only if explicitly instructed |

## Common Mistakes

**Skipping test verification**
- **Problem:** Merge broken code, create failing PR
- **Fix:** Always verify tests before offering options

**Interactive choice prompts**
- **Problem:** Autonomous agents stall waiting for a human response
- **Fix:** Push or create PR by default; never pause for routine integration choices

**Automatic worktree cleanup**
- **Problem:** Deletes recovery path after autonomous work
- **Fix:** Preserve branch and worktree unless explicitly instructed otherwise

## Red Flags

**Never:**
- Proceed with failing tests
- Merge locally by default
- Delete work automatically
- Force-push without explicit request

**Always:**
- Verify tests before publishing changes
- Prefer updating an existing PR branch
- Create a PR if no PR exists yet
- Preserve the branch and worktree by default

## Integration

**Called by:**
- **subagent-driven-development** (Step 7) - After all tasks complete
- **executing-plans** (Step 5) - After all batches complete

**Pairs with:**
- **using-git-worktrees** - Cleans up worktree created by that skill
