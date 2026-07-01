# Agentic-Driven Development (`add`)

A de-hooked clone of the Agentic Development Skills workflow, tuned to run **fully unattended**.
You boot it explicitly, it researches, decides for itself, builds, and hands you a **pull request** —
no human in the loop. Built to run as one of several background leaves in parallel.

> **spec → plan → implement → verify → PR**, with the agent making and recording every decision
> instead of asking.

There is **no SessionStart hook** — nothing is auto-injected. Start a run on demand:

```
/agentic-driven-development add react to this project
```

The `agentic-driven-development` skill is a short router (progressive disclosure): it holds the
workflow map + the **Unattended Operation Contract**, and loads each sub-skill on demand.

## The workflow

```
agentic-driven-development   (router — invoked explicitly via /agentic-driven-development)
  │  Unattended Operation Contract: never pause; decide + record; end with a PR
  │
  ├─ add-brainstorming ─────────►  spec doc   docs/add/specs/…   (no approval gate; records Assumptions & Decisions)
  │     └─ add-writing-plans ───►  plan doc   docs/add/plans/…   (auto-continues to execution)
  │           └─ execute the plan, task-by-task:
  │                • add-subagent-driven-development   (REQUIRED — fresh subagent + 2-stage review, continuous)
  │                     ├─ add-test-driven-development        (red → green → refactor, per task)
  │                     ├─ add-requesting-code-review         (reviewers are subagents)
  │                     ├─ add-verification-before-completion (evidence before any "done")
  │                     └─ add-finishing-a-development-branch (auto: push + open PR)
  │                • add-executing-plans               (fallback if subagents unavailable)
  │
  ├─ add-using-git-worktrees      (DEFAULT ON — isolate each leaf so parallel runs don't collide)
  └─ add-systematic-debugging     (any bug / test failure — root cause before fixes)
```

## What "unattended" changes vs. the original

- **No approval gates.** brainstorming writes the spec and moves on; no design/spec sign-off.
- **No execution-path question.** writing-plans hands straight to the continuous subagent path.
- **No merge/keep/discard menu.** finishing always pushes and opens a PR.
- **Decisions are recorded, not asked.** Every non-obvious call goes in an **Assumptions & Decisions**
  log that travels spec → plan → PR body, so you can review the agent's judgment after the fact.
- **Blockers don't stall the run.** Only unrecoverable external blockers (missing creds/services)
  stop a task — they become deferred TODOs surfaced in the PR; the rest of the plan continues.
- **Isolation on by default.** Each leaf gets its own branch/worktree.

For the involved, human-in-the-loop counterpart, use `human-driven-development`
(`/human-driven-development`).

## Install (npx skills)

```
npx skills add ./AgenticDrivenDevelopment
```

Skills install flat, so every skill is prefixed `add-` to avoid colliding with the `hdd-`
skills from Human-Driven Development when both are installed.

## Conventions

- Skills live in `skills/<name>/SKILL.md` (YAML frontmatter + markdown body).
- Skills reference each other by their prefixed name, `add-<skill>`.
- Workflow artifacts go in `docs/add/specs/` and `docs/add/plans/` (user preferences override).

> Ported from [obra/superpowers](https://github.com/obra/superpowers) via Agentic Development Skills.
