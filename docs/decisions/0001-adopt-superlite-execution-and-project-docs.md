---
title: "0001: Adopt superlite's execution model and the shared project docs structure"
description: ADD sizes requests, plans a task graph for parallel tiered agents, records rulings and decision records, and keeps docs true in every commit.
status: proposed
date: 2026-10-25
decision-makers: Spenser Hale (direction); drafted by Claude
---

## Context

ADD was a decide-and-record clone of superpowers: one implementer at a time, a spec-compliance review and a code-quality review loop after every task, and an "Assumptions & Decisions" log inside a dated design spec under `docs/add/specs/`. Two problems showed up.

- **Speed.** superlite ([iha2/superlite](https://github.com/iha2/superlite)) profiled superpowers and found the time going to human gates, strictly sequential dispatch, per-task review loops, repeated full-suite runs, and the most capable model writing code. On the same 177-test benchmark it finished in about half the wall-clock time with the same pass rate, using a task graph, parallel tiered agents, selective reviews, and a findings-only final review.
- **Lost reasons.** An unattended run makes calls no human saw. A log buried in a dated design doc does not tell a reviewer which calls are risky, strong decisions never became durable records, and docs were updated apart from the code they describe.

## Options considered

1. **Keep the superpowers flow and change only the paths**: least effort, but keeps the sequential bottleneck and leaves strong decisions in a log.
2. **Vendor superlite wholesale** (its Python scripts, hooks, and lint): measured speed and code-enforced gates, but it contradicts ADD's hook-free, script-free design and its auto-PR ending, and adds a Python 3.10 dependency to every target project.
3. **Adopt superlite's model in the skill text, plus the shared docs structure**: sizing, a task graph with Owns, Produces, and `e.g.` contracts, a coordinator that never edits code and commits each task, parallel tiered implementers, selective package reviews, a fresh final reviewer, tiered fixers, rulings with their cost if wrong, decision records for strong calls, and atomic commits.

## Decision

We adopt option 3. The dated design spec becomes the design half of one plan doc in the project's plans folder, and add-writing-plans appends the task graph. The Assumptions & Decisions log becomes rulings (`[scope] decision — why — cost if wrong: low|medium|high, what breaks`), listed costliest first in the PR. Strong decisions get decision records with `decision-makers: agent (unattended run)`, ratified by merging the PR. Each task's commit carries its code, tests, and docs; the coordinator applies shared doc text (the feature spec, the changelog) when it commits a task. `spec-reviewer-prompt.md` and `code-quality-reviewer-prompt.md` give way to `package-reviewer-prompt.md`, `final-reviewer-prompt.md`, and `fixer-prompt.md`. Artifacts follow the target project's `## Documentation` section in `AGENTS.md` (written by agents-init), defaulting to `docs/plans/`, `docs/specs/`, and `docs/decisions/`.

## Consequences

- Independent tasks run at once, and the PR tells the reviewer which calls to second-guess first.
- Specs, decision records, and the changelog stay true at every commit, so later runs can trust them.
- Nothing enforces the rules mechanically: the coordinator not editing code, disjoint Owns, and staging by path are prose-only, where superlite uses hooks and a plan linter.
- Plans need more structure up front (contracts, `e.g.` values, Docs and Commit fields).
- Parallel implementers share one working tree, so an Owns mistake can mix two tasks' changes.
- Projects with old `docs/add/` folders keep them as history; installed copies of the skills need a reinstall to pick this up.

## Revisit when

Runs show the coordinator editing code, overlapping Owns, or docs drifting from code despite the rules (then port superlite's plan lint and checkpoint scripts, or hooks), or measured run times do not improve on the sequential flow.

## Links

- superlite post: [Superlite: hand an agent a spec, come back to tested work](https://contextfirst.blog/superlite-hand-an-agent-a-spec/)
- Shared structure: agents-init `references/project-structure.md` in [spenserhale/skills](https://github.com/spenserhale/skills)
