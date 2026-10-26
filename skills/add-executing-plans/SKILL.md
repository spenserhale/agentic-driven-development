---
name: add-executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load the plan, review it critically, execute every task yourself in dependency order, and commit each task atomically. This is the fallback when subagents are unavailable; with subagents, use add-subagent-driven-development.

**Announce at start:** "I'm using the add-executing-plans skill to implement this plan."

**Each task uses TDD.** When a task implements behavior, follow add-test-driven-development for that task — write the failing test first, watch it fail, then write minimal code to pass.

## The Process

### Step 1: Load and Review Plan
1. Read the plan doc (design sections and tasks)
2. Review critically - identify any questions or concerns about the plan
3. If concerns: resolve them with your own best judgment, amend the plan if needed, and record a ruling in the plan's `## Rulings` — do NOT wait for a human (this is unattended)
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

Work in dependency order (`Depends on`, or the task graph). For each task:
1. Mark as in_progress
2. Follow its steps exactly (tests first, per the plan's Detail level)
3. Run the verifications it specifies
4. Apply its Docs: owned docs and the shared-doc text (spec slice, changelog line) exactly as planned; if the planned text no longer matches what you built, correct it to match the code and record a ruling
5. Commit it as one atomic commit — stage by path (`git add -- <Owns> <docs it touched> <plan doc if rulings changed>`) and use the task's Commit message (add-documenting-changes)
6. Mark as completed

A strong decision forced mid-task gets a decision record in the same commit as the code that embodies it.

### Step 3: Complete Development

After all tasks complete:
- **Final review of the whole change** against the plan, including whether the docs tell the truth. The range is start SHA..HEAD, where the start SHA is the parent of the plan commit. This skill runs because subagents are unavailable, so walk the checklist in `../add-requesting-code-review/code-reviewer.md` yourself, with fresh eyes on the diff, and say in the PR that the review was self-run. Fix Critical and Important findings, each as its own atomic commit.
- **REQUIRED SUB-SKILL:** Use add-verification-before-completion before claiming the work is done — run the full test/build/lint commands and confirm the output yourself.
- Set the plan's `status: done` and commit it with the final rulings.
- Then: **REQUIRED SUB-SKILL:** Use add-finishing-a-development-branch to verify tests, push the branch, and automatically open a PR (no menu — this is unattended).

## When Blocked (unattended — decide, don't ask)

You are running unattended, so "ask for help" is not available. Handle blockers yourself:

- **Unclear instruction / plan gap:** interpret it with best judgment, amend the plan, and record a ruling.
- **Test / verification fails:** route through add-systematic-debugging and fix it.
- **True external blocker** (missing dependency you can't install, missing credential/service/network): record it under `## Deferred`, skip or stub that task, and continue the rest of the plan. Surface it in the PR.

Never halt the whole run waiting for a human. A recorded ruling or deferred item always beats a stall.

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- You amend the plan based on something you discovered
- Fundamental approach needs rethinking (rethink it yourself, record why — as a decision record if the call is strong)

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- One task, one commit: code, tests, and docs together
- Reference skills when plan says to
- When blocked, decide and record — don't stall
- Never start implementation on main/master branch — auto-create a feature branch instead (unattended: branch, don't ask)

## Integration

**Required workflow skills:**
- **add-writing-plans** - Creates the plan this skill executes
- **add-test-driven-development** - Each behavior-implementing task follows the red/green/refactor cycle
- **add-documenting-changes** - Shared docs, rulings, decision records, and the atomic commit
- **add-requesting-code-review** - Final review of the whole change before finishing
- **add-verification-before-completion** - Gate before claiming the work is complete
- **add-finishing-a-development-branch** - Push the branch and open the PR after verification

**Alternative execution path:**
- **add-subagent-driven-development** - Parallel subagents with selective and final reviews (use it when subagents are available)
