---
name: add-finishing-a-development-branch
description: Use when unattended implementation is complete - verifies tests, then automatically pushes the branch and opens a pull request (carrying the Assumptions & Decisions log and any deferred TODOs). No menu, no merge, no human choice.
---

# Finishing a Development Branch

## Overview

Complete unattended development work by **automatically opening a pull request** — no menu, no human choice. This variant always finishes the same way: verify tests, push the branch, open a PR whose body carries the Assumptions & Decisions log and any deferred TODOs.

**Core principle:** Verify tests → Detect environment → Push + open PR (always) → Leave worktree intact.

**Announce at start:** "I'm using the add-finishing-a-development-branch skill to push and open a PR."

**No merge/keep/discard menu.** The whole point of Agentic-Driven Development is to hand the human a reviewable PR. Never merge to the base branch yourself, never discard work, never wait for a choice. The only decision is already made: open a PR.

## The Process

### Step 1: Verify Tests

**Before opening the PR, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**If tests fail:** Do not stop and wait — this is unattended. Route the failure through add-systematic-debugging and fix it. If a failure is genuinely unresolvable (external blocker: missing service/credential), record it as a deferred TODO, mark the PR as a draft, and note the failing tests prominently in the PR body. Otherwise, tests must pass before opening a non-draft PR.

Then continue to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Detect Environment

**Determine workspace state before pushing:**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

Environment detection only affects how the branch is pushed — the outcome is always a PR:

| State | Action |
|-------|--------|
| `GIT_DIR == GIT_COMMON` (normal repo) | Push current branch, open PR |
| `GIT_DIR != GIT_COMMON`, named branch | Push current branch, open PR |
| `GIT_DIR != GIT_COMMON`, detached HEAD | Create a named branch, push it, open PR |

### Step 3: Determine Base Branch

```bash
# Try common base branches — do NOT ask; infer it
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Infer the base branch from `git merge-base` and the repo's default branch. Do not ask for confirmation — if genuinely ambiguous, default to the repo's default branch (`main`, else `master`) and note the choice in the PR body.

### Step 4: Push and Open the PR (always)

There is no menu. Detached HEAD? Create a named branch first (`git switch -c <feature-branch>`). Then:

```bash
# Push branch
git push -u origin <feature-branch>

# Open PR — body carries the decisions log and any deferred TODOs
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Assumptions & Decisions
<the log carried from spec → plan: each non-obvious call, alternatives, why>

## Deferred / TODO
<external blockers you could not resolve, or "none">

## Test Plan
- [ ] <verification steps>
EOF
)"
```

If unresolved external blockers or failing tests remain, open the PR as a **draft** (`gh pr create --draft`) so the human knows it needs attention.

**Never clean up the worktree** — the human needs it alive to iterate on PR feedback. There is no merge, keep, or discard path in this variant.

### Step 5: Report

Report the PR URL that `gh pr create` printed, plus a one-line summary of any deferred TODOs. Then stop — the run is complete.

**Do not clean up the worktree.** The human needs it alive to iterate on PR feedback, and a parallel-leaf worktree may be reused. The host environment owns workspace teardown, not this skill.

## Quick Reference

| Step | Action |
|------|--------|
| 1. Verify tests | Pass → continue. Fail → debug/fix, or draft PR + deferred TODO if external blocker. |
| 2. Detect env | Named branch → push it. Detached HEAD → create a named branch first. |
| 3. Base branch | Infer from `git merge-base` / default branch. Never ask. |
| 4. Push + PR | `git push -u` then `gh pr create` (draft if unresolved issues). Body carries decisions log + TODOs. |
| 5. Report | Print PR URL + deferred TODOs. Leave worktree intact. |

## Common Mistakes

**Skipping test verification**
- **Problem:** Open a failing PR without flagging it
- **Fix:** Verify tests first; if unresolvable, open as draft and list the failures in the body

**Stopping to ask a question**
- **Problem:** The run stalls forever — no human is watching
- **Fix:** There is no menu and no confirmation. Infer, decide, record, open the PR.

**Merging or discarding**
- **Problem:** This variant hands the human a reviewable PR; merging or discarding removes their review
- **Fix:** Only ever push + open a PR. Never merge to base, never delete work.

**Cleaning up the worktree**
- **Problem:** Removes the workspace the human/PR-iteration needs
- **Fix:** Never remove the worktree in this variant

## Red Flags

**Never:**
- Stop and wait for a human choice (menu, confirmation, approval)
- Merge to the base branch yourself
- Discard or force-delete work
- Clean up the worktree
- Open a non-draft PR when tests fail or blockers remain

**Always:**
- Verify tests before opening a non-draft PR
- Infer the base branch instead of asking
- Carry the Assumptions & Decisions log and deferred TODOs into the PR body
- Open a draft PR (not silence) when something needs human attention
- End by reporting the PR URL
