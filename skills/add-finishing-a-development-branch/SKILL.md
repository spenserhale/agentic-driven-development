---
name: add-finishing-a-development-branch
description: Use when unattended implementation is complete - verifies tests and docs, then automatically pushes the branch and opens a pull request carrying the rulings, decision records, and any deferred items. No menu, no merge, no human choice.
---

# Finishing a Development Branch

## Overview

Complete unattended development work by **automatically opening a pull request** — no menu, no human choice. This variant always finishes the same way: verify tests and docs, push the branch, open a PR whose body lets the human review your judgment fast: rulings sorted by cost if wrong, decision records linked, docs changes listed, deferred items flagged.

**Core principle:** Verify tests → Verify docs → Detect environment → Push + open PR (always) → Leave worktree intact.

**Announce at start:** "I'm using the add-finishing-a-development-branch skill to push and open a PR."

**No merge/keep/discard menu.** The whole point of Agentic-Driven Development is to hand the human a reviewable PR. Never merge to the base branch yourself, never discard work, never wait for a choice. The only decision is already made: open a PR.

## The Process

### Step 1: Verify Tests

**Before opening the PR, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**If tests fail:** Do not stop and wait — this is unattended. Route the failure through add-systematic-debugging and fix it (coming from add-subagent-driven-development, the coordinator sends it to a tier-3 fixer rather than editing code itself). If a failure is genuinely unresolvable (external blocker: missing service/credential), record it under the plan's `## Deferred`, mark the PR as a draft, and note the failing tests prominently in the PR body. Otherwise, tests must pass before opening a non-draft PR.

**If tests pass:** Continue to Step 2.

### Step 2: Verify Docs and Commits

Infer the base branch first (Step 4), then check the branch against the plan:

```bash
git log --stat --oneline <base>..HEAD
```

- **Atomic commits:** every commit that changes behavior also carries its tests and the docs it changes (spec slice, changelog line, guides). A code commit found missing its docs here gets the one kind of docs-after-code commit this workflow allows: `docs(<scope>): complete <short-sha>`, naming the commit it completes, listed in the PR, with a ruling. Never rewrite pushed history.
- **Specs match the code:** each spec in the plan's Spec delta now states the shipped behavior, and only that.
- **Decision records:** every record in the plan's `## Decisions` exists, with `status: accepted` and `decision-makers: agent (unattended run)`; superseded records carry `status: superseded` and `superseded-by`.
- **Record numbers are free:** parallel runs can pick the same number. Fetch, then compare yours with the base branch and with other open PRs:

  ```bash
  git fetch origin
  git ls-tree --name-only origin/<base> <decisions folder>/
  gh pr list --state open --json number,files --jq '.[] | .files[].path' | grep '<decisions folder>/'
  ```

  On a collision, renumber yours to the next free number in one commit (`docs(decisions): renumber 0008 to 0009`), updating its title, filename, and every link in files. Commit bodies already pushed or written still say 0008, so the renumber commit and the PR body both name the old number.
- **Plan closed:** the plan's `status: done`, with its final Rulings and Deferred sections. Anything this skill adds to them (a test failure deferred in Step 1, a renumbering ruling) is committed before Step 5 pushes.

A trivial run has no plan doc: check only that its one commit carries the docs it changes, and write its rulings and deferred items straight into the PR body.

### Step 3: Detect Environment

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

### Step 4: Determine Base Branch

```bash
# Try common base branches — do NOT ask; infer it
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Infer the base branch from `git merge-base` and the repo's default branch. Do not ask for confirmation — if genuinely ambiguous, default to the repo's default branch (`main`, else `master`) and note the choice in the PR body.

### Step 5: Push and Open the PR (always)

There is no menu. Detached HEAD? Create a named branch first (`git switch -c <feature-branch>`). Then:

```bash
# Push branch
git push -u origin <feature-branch>

# Open PR — the body is how the human reviews your judgment
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>
Plan: <path to the plan doc>

## Rulings (decisions made without you, costliest first)
<every ruling from the plan, sorted high → medium → low cost if wrong>

## Decision records
<each record added or superseded: number, title, link, one-line decision>

## Docs changed
<specs, guides, changelog, pricing: one line each>

## Deferred / TODO
<external blockers you could not resolve, follow-up sub-projects, or "none">

## Test Plan
- [ ] <verification steps, with the commands and their results>
EOF
)"
```

If unresolved external blockers or failing tests remain, open the PR as a **draft** (`gh pr create --draft`) so the human knows it needs attention. When the project has no docs map in `AGENTS.md`, add one line suggesting agents-init to the Summary.

**Never clean up the worktree** — the human needs it alive to iterate on PR feedback. There is no merge, keep, or discard path in this variant.

### No Remote or No `gh` Login

`git push` or `gh pr create` cannot run (no remote configured, no network, `gh` not logged in): that is an external blocker, not a reason to stop early or to merge. Leave the branch committed, put the PR title and the full PR body in your final report (and the blocker in the plan's `## Deferred`), and say exactly which command failed and why.

### Step 6: Report

Report the PR URL that `gh pr create` printed, the three costliest rulings, and a one-line summary of any deferred items. Then stop — the run is complete.

**Do not clean up the worktree.** The human needs it alive to iterate on PR feedback, and a parallel-leaf worktree may be reused. The host environment owns workspace teardown, not this skill.

## Quick Reference

| Step | Action |
|------|--------|
| 1. Verify tests | Pass → continue. Fail → debug/fix, or draft PR + deferred item if external blocker. |
| 2. Verify docs | Atomic commits, specs match code, records exist and are numbered free, plan closed. |
| 3. Detect env | Named branch → push it. Detached HEAD → create a named branch first. |
| 4. Base branch | Infer from `git merge-base` / default branch. Never ask. |
| 5. Push + PR | `git push -u` then `gh pr create` (draft if unresolved issues). Body: rulings costliest first, records, docs, deferred, tests. |
| 6. Report | PR URL, top rulings, deferred items. Leave worktree intact. |

## Common Mistakes

**Skipping test verification**
- **Problem:** Open a failing PR without flagging it
- **Fix:** Verify tests first; if unresolvable, open as draft and list the failures in the body

**Burying the rulings**
- **Problem:** The human has to read the whole diff to find the risky calls
- **Fix:** Rulings sorted by cost if wrong, decision records linked, at the top of the body

**Duplicate decision record numbers**
- **Problem:** Two parallel runs both add record 0008
- **Fix:** Check the base branch and open PRs before pushing; renumber yours

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
- Discard or force-delete work, or rewrite pushed history
- Clean up the worktree
- Open a non-draft PR when tests fail or blockers remain
- Open a PR whose docs disagree with its code

**Always:**
- Verify tests and docs before opening a non-draft PR
- Infer the base branch instead of asking
- List every ruling (costliest first) and every decision record in the PR body
- Open a draft PR (not silence) when something needs human attention
- End by reporting the PR URL
