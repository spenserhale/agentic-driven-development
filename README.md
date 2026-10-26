# Agentic-Driven Development (`add`)

A de-hooked clone of the Agentic Development Skills workflow, tuned to run **fully unattended** and
reshaped after [superlite](https://github.com/iha2/superlite): one plan written for parallel work,
agents matched to the work, rulings instead of questions, and a fresh final review. You boot it
explicitly; it researches, decides for itself, builds, and hands you a **pull request** — no human
in the loop. Built to run as one of several background leaves in parallel.

> **size → plan → parallel build → review → PR**, with the agent making and recording every decision
> instead of asking, and the project's docs kept true in every commit.

There is **no SessionStart hook** — nothing is auto-injected. Start a run on demand:

```
/agentic-driven-development add react to this project
```

The `agentic-driven-development` skill is a short router (progressive disclosure): it holds the
workflow map + the **Unattended Operation Contract**, and loads each sub-skill on demand.

## The workflow

```
agentic-driven-development   (router — invoked explicitly via /agentic-driven-development)
  │  Unattended Operation Contract: never pause; rule + record; docs ship with code; end with a PR
  │  Size the request: trivial → TDD inline, one atomic commit, PR
  │
  ├─ add-brainstorming ─────────►  plan doc  docs/plans/YYYY-MM-DD-<topic>.md
  │     │                          (intent, acceptance criteria, design, spec delta, docs impact,
  │     │                           rulings; decision records for strong calls)
  │     └─ add-writing-plans ───►  same plan doc + task graph (Owns, Depends on, Produces, Docs,
  │           │                    Commit per task), committed with its decision records
  │           └─ add-subagent-driven-development   (coordinator: dispatch ready tasks in parallel,
  │                │                                one atomic commit per task, selective reviews,
  │                │                                fresh final reviewer, tiered fixers)
  │                ├─ add-test-driven-development        (every implementer: red → green → refactor)
  │                ├─ add-documenting-changes            (ruling vs. decision record, spec deltas, atomic commits)
  │                ├─ add-verification-before-completion (evidence before any "done")
  │                └─ add-finishing-a-development-branch (auto: push + open PR)
  │           • add-executing-plans               (fallback if subagents are unavailable)
  │
  ├─ add-using-git-worktrees      (DEFAULT ON — isolate each leaf so parallel runs don't collide)
  ├─ add-systematic-debugging     (any bug / test failure — root cause before fixes)
  └─ add-requesting-code-review   (ad-hoc reviews outside the plan's own)
```

## What "unattended" changes vs. the original

- **No approval gates.** Brainstorming writes the design into the plan and moves on.
- **No execution-path question.** Writing-plans hands straight to the parallel subagent path.
- **No merge/keep/discard menu.** Finishing always pushes and opens a PR.
- **Rulings, not questions.** Every non-obvious call is a ruling in the plan:
  `[scope] decision — why — cost if wrong: low|medium|high, what breaks`. The PR lists them costliest first, so you know which
  calls to second-guess first.
- **Strong decisions get decision records.** A call that is hard to reverse, adds a dependency or a
  pattern, changes a public interface, or touches security or pricing gets an ADR in the project's
  decisions folder, even though no one was there to ask.
- **Blockers don't stall the run.** Only unrecoverable external blockers (missing creds/services)
  stop a task — they go under the plan's Deferred section and the PR; the rest of the plan continues.
- **Isolation on by default.** Each leaf gets its own branch/worktree.

## What superlite changes vs. superpowers

- **Size first.** Trivial requests skip the design; bounded ones get a short plan; architectural ones
  get the full task graph.
- **A plan written for the split.** Each task owns its files, states the interfaces it consumes and
  produces (with concrete `e.g.` values), and declares only real execution dependencies, so
  independent tasks run at once against fakes.
- **Agents matched to the work.** The coordinator never edits code; cheap models build mechanical
  tasks, standard models build the rest, and the most capable model does the final review.
- **Selective reviews.** Risky tasks are reviewed as they land, the rest per batch, then a fresh
  findings-only final reviewer reads the whole change and tiered fixers close the findings.

## Docs ship with the code

The workflow reads the project's docs map from the `## Documentation` section of `AGENTS.md`
(written by [agents-init](https://github.com/spenserhale/skills/tree/main/skills/agents-init)) and
falls back to the same defaults:

| Kind | Default path | Rule |
|------|--------------|------|
| Spec | `docs/specs/<feature>.md` | What the feature does now; changes in the same commit as the code |
| Decision record | `docs/decisions/NNNN-<title>.md` | Why; superseded, never rewritten |
| Plan | `docs/plans/YYYY-MM-DD-<topic>.md` | This change: design, spec delta, tasks, rulings; committed before code |

Every commit is atomic: the code, its tests, and every doc it changes (spec slice, guide, changelog
line, decision record) land together.

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
- Workflow artifacts follow the project's docs map; defaults are `docs/plans/`, `docs/specs/`, and
  `docs/decisions/` (user preferences override).

> Ported from [obra/superpowers](https://github.com/obra/superpowers) via Agentic Development Skills;
> execution model adapted from [iha2/superlite](https://github.com/iha2/superlite).
