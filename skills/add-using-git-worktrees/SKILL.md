---
name: add-using-git-worktrees
description: Isolate this unattended leaf on its own branch/worktree so parallel Agentic-Driven runs don't collide. Default ON — create isolation unless the harness already provides an isolated workspace (or a native worktree tool), in which case defer to that.
---

# Using Git Worktrees

## Overview

This variant runs as one of several **parallel background leaves**, so isolation is **the default**, not opt-in. Get this leaf onto its own dedicated branch (and ideally its own worktree) before writing code, so concurrent runs never step on each other. When isolation IS being set up, prefer your platform's native worktree tools and fall back to manual git worktrees only when no native tool is available.

**Core principle:** Default to isolating. First detect whether the harness already isolates you (Step 0) — if so, use that and don't double-isolate. Otherwise create a dedicated branch/worktree yourself. Never fight the harness, but never share a workspace with a sibling leaf either.

**When:** first, before anything is written. The router runs this skill before brainstorming, so the plan commit and every later commit land on this leaf's branch, never on the base branch.

**Announce at start:** "I'm using the add-using-git-worktrees skill to isolate this leaf."

## Step 0: Detect Existing Isolation

**Before creating anything, check if you are already in an isolated workspace.**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**Submodule guard:** `GIT_DIR != GIT_COMMON` is also true inside git submodules. Before concluding "already in a worktree," verify you are not in a submodule:

```bash
# If this returns a path, you're in a submodule, not a worktree — treat as normal repo
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**If `GIT_DIR != GIT_COMMON` (and not a submodule):** You are already in a linked worktree. Skip to Step 2 (Project Setup). Do NOT create another worktree.

Report with branch state:
- On a branch: "Already in isolated workspace at `<path>` on branch `<name>`."
- Detached HEAD: "Already in isolated workspace at `<path>` (detached HEAD, externally managed)." Create a named branch for this leaf now (`git switch -c <branch>`), before anything is committed.

**If `GIT_DIR == GIT_COMMON` (or in a submodule):** You are in a normal, shared repo checkout. Apply the default-isolate rule below.

### Default-Isolate Rule — default is to CREATE

Isolation is the default for this variant. Create a dedicated branch/worktree before writing code. Choose the mechanism by signal:

1. **Harness native tool available** (e.g. `EnterWorktree`, a `/worktree` command, a `--worktree` flag): use it — it's the cleanest isolation on this platform.
2. **Project `CLAUDE.md` / `AGENTS.md` recommends worktrees:** honor that convention.
3. **No signal, but a shared checkout:** create a git worktree yourself (Step 1). If worktrees aren't workable in this environment, at minimum create and switch to a dedicated feature branch so this leaf never shares a branch with a sibling run.

**Only skip isolation** if it is genuinely impossible here (e.g. a non-git directory) — in which case note it in one line and proceed carefully in place. Do NOT prompt the user; this is unattended.

## Step 1: Create Isolated Workspace

**You have two mechanisms. Try them in this order.**

### 1a. Native Worktree Tools (preferred)

Isolation is the default for this variant (Step 0). Do you already have a way to create a worktree? It might be a tool with a name like `EnterWorktree`, `WorktreeCreate`, a `/worktree` command, or a `--worktree` flag. If you do, use it and skip to Step 2.

Native tools handle directory placement, branch creation, and cleanup automatically. Using `git worktree add` when you have a native tool creates phantom state your harness can't see or manage.

Only proceed to Step 1b if you have no native worktree tool available.

### 1b. Git Worktree Fallback

**Only use this if Step 1a does not apply** — you have no native worktree tool available. Create a worktree manually using git.

#### Directory Selection

Follow this priority order. Explicit user preference always beats observed filesystem state.

1. **Check your instructions for a declared worktree directory preference.** If the user has already specified one, use it without asking.

2. **Check for an existing project-local worktree directory:**
   ```bash
   ls -d .worktrees 2>/dev/null     # Preferred (hidden)
   ls -d worktrees 2>/dev/null      # Alternative
   ```
   If found, use it. If both exist, `.worktrees` wins.

3. **Check for an existing global directory:**
   ```bash
   project=$(basename "$(git rev-parse --show-toplevel)")
   ls -d ~/.config/add/worktrees/$project 2>/dev/null
   ```
   If found, use it (backward compatibility with legacy global path).

4. **If there is no other guidance available**, default to `.worktrees/` at the project root.

#### Safety Verification (project-local directories only)

**MUST verify directory is ignored before creating worktree:**

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:** use the global directory (`~/.config/add/worktrees/<project>/`) instead. Do not commit a `.gitignore` change to the shared checkout: that would put a commit on a branch other leaves share.

**Why critical:** Prevents accidentally committing worktree contents to repository.

Global directories (`~/.config/add/worktrees/`) need no verification.

#### Create the Worktree

```bash
project=$(basename "$(git rev-parse --show-toplevel)")

# Determine path based on chosen location
# For project-local: path="$LOCATION/$BRANCH_NAME"
# For global: path="~/.config/add/worktrees/$project/$BRANCH_NAME"

git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

**Sandbox fallback:** If `git worktree add` fails with a permission error (sandbox denial), create and switch to a dedicated feature branch in the current directory instead (`git switch -c <branch>`), record a ruling that the sandbox blocked the worktree, and run setup and baseline tests in place.

## Step 2: Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## Step 3: Verify Clean Baseline

Run tests to ensure workspace starts clean:

```bash
# Use project-appropriate command
npm test / cargo test / pytest / go test ./...
```

**If the test runner is missing** (not installed, wrong invocation): find the form that works here (the project's virtualenv, `uv run`, `npx`, a `PYTHONPATH`), record a ruling, and use that form from now on. If none works, it is an external blocker: record it and continue without a baseline.

**If tests fail:** Do not ask, and do not fix them yet. Write down which tests already fail, so new failures stay distinguishable, and carry that list into the plan: add-brainstorming records it as a ruling, and a failure the task actually needs fixed becomes a plan task (committed after the plan, like any other code). Failures the task doesn't touch go under Deferred.

**If tests pass:** Report ready.

### Report

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| Already in linked worktree | Skip creation (Step 0) |
| In a submodule | Treat as normal repo (Step 0 guard) |
| Native worktree tool available | Use it (Step 1a) |
| No native tool | Git worktree fallback (Step 1b) |
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check instruction file, then default `.worktrees/` |
| Global path exists | Use it (backward compat) |
| Directory not ignored | Use the global directory instead; never commit to the shared checkout |
| Permission error on create | Sandbox fallback, work in place |
| Test runner missing | Find the working invocation, record a ruling |
| Tests fail during baseline | Record which fail; fix them only as plan tasks — never ask |
| No package.json/Cargo.toml | Skip dependency install |

## Common Mistakes

### Fighting the harness

- **Problem:** Using `git worktree add` when the platform already provides isolation
- **Fix:** Step 0 detects existing isolation. Step 1a defers to native tools.

### Skipping detection

- **Problem:** Creating a nested worktree inside an existing one
- **Fix:** Always run Step 0 before creating anything

### Skipping ignore verification

- **Problem:** Worktree contents get tracked, pollute git status
- **Fix:** Always use `git check-ignore` before creating project-local worktree

### Assuming directory location

- **Problem:** Creates inconsistency, violates project conventions
- **Fix:** Follow priority: existing > global legacy > instruction file > default

### Proceeding with failing tests

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **Fix:** Record the pre-existing failures before changing anything; fix the ones the task needs as plan tasks, defer the rest

## Red Flags

**Never:**
- Create a worktree when Step 0 detects existing isolation
- Use `git worktree add` when you have a native worktree tool (e.g., `EnterWorktree`). This is the #1 mistake — if you have it, use it.
- Skip Step 1a by jumping straight to Step 1b's git commands
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed without recording which tests already failed
- Commit anything (even the plan) before this leaf has its own branch

**Always:**
- Run Step 0 detection first
- Prefer native tools over git fallback
- Follow directory priority: existing > global legacy > instruction file > default
- Verify directory is ignored for project-local
- Auto-detect and run project setup
- Verify clean test baseline
