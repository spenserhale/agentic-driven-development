---
name: add-executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans

## Overview

Load plan, review critically, execute all tasks, report when complete.

**Announce at start:** "I'm using the add-executing-plans skill to implement this plan."

**Each task uses TDD.** When a task implements behavior, follow add-test-driven-development for that task — write the failing test first, watch it fail, then write minimal code to pass.

## The Process

### Step 1: Load and Review Plan
1. Read plan file
2. Review critically - identify any questions or concerns about the plan
3. If concerns: resolve them with your own best judgment, amend the plan if needed, and record the change in the Assumptions & Decisions log — do NOT wait for a human (this is unattended)
4. If no concerns: Create TodoWrite and proceed

### Step 2: Execute Tasks

For each task:
1. Mark as in_progress
2. Follow each step exactly (plan has bite-sized steps)
3. Run verifications as specified
4. Mark as completed

### Step 3: Complete Development

After all tasks complete:
- **REQUIRED SUB-SKILL:** Use add-verification-before-completion before claiming the work is done — run the full test/build/lint commands and confirm the output yourself.
- Then: **REQUIRED SUB-SKILL:** Use add-finishing-a-development-branch to verify tests, push the branch, and automatically open a PR (no menu — this is unattended).

## When Blocked (unattended — decide, don't ask)

You are running unattended, so "ask for help" is not available. Handle blockers yourself:

- **Unclear instruction / plan gap:** interpret it with best judgment, amend the plan, and record the decision in the Assumptions & Decisions log.
- **Test / verification fails:** route through add-systematic-debugging and fix it.
- **True external blocker** (missing dependency you can't install, missing credential/service/network): record a deferred TODO, skip or stub that task, and continue the rest of the plan. Surface it in the PR.

Never halt the whole run waiting for a human. A recorded decision or deferred TODO always beats a stall.

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- You amend the plan based on something you discovered
- Fundamental approach needs rethinking (rethink it yourself, record why)

## Remember
- Review plan critically first
- Follow plan steps exactly
- Don't skip verifications
- Reference skills when plan says to
- When blocked, decide and record — don't stall
- Never start implementation on main/master branch — auto-create a feature branch instead (unattended: branch, don't ask)

## Integration

**Required workflow skills:**
- **add-writing-plans** - Creates the plan this skill executes
- **add-test-driven-development** - Each behavior-implementing task follows the red/green/refactor cycle
- **add-verification-before-completion** - Gate before claiming the work is complete
- **add-finishing-a-development-branch** - Integrate the work (merge/PR/keep/discard) after verification

**Alternative execution path:**
- **add-subagent-driven-development** - Fresh subagent per task with two-stage review (recommended when subagents are available)
