---
name: add-documenting-changes
description: Use when an unattended change needs its docs - finding where the project keeps specs, decision records, and plans, deciding whether a call needs a decision record or only a ruling, writing either, applying a spec delta, or composing the commit that carries code, tests, and docs together
---

# Documenting Changes

## Overview

In an unattended run nobody is watching to remember why something was done, so the docs have to carry it. Every commit leaves the repo telling the truth: the code, the tests that prove it, and the docs that describe it change together. Strong decisions get a decision record; smaller calls get a ruling.

**Core principle:** Docs are part of the change, not a follow-up. A commit without its docs is incomplete the same way a commit without its tests is.

## Find the Project's Docs Map

1. Read the `## Documentation` section of `AGENTS.md` (or `CLAUDE.md`). Its table names the specs, decisions, and plans folders. Use them, and follow any rule it states.
2. No such section, or one without rows for specs, decisions, or plans (doc-it writes a prose one): use the defaults below for the missing kinds and record a ruling, `[docs] used the default docs/ layout — AGENTS.md has no docs map — cost if wrong: low, files move`. Do not add a docs map to `AGENTS.md` yourself; the PR can suggest running agents-init.
3. Existing folders win even without a map: a `docs/adr/` is the decisions folder. Folders left by older runs (`docs/add/`) stay where they are; new work goes to the resolved paths.

| Kind | Default path | Holds |
|------|--------------|-------|
| Spec | `docs/specs/<feature>.md` | What the feature does now |
| Decision record | `docs/decisions/NNNN-<title>.md` | Why: one decision, never rewritten |
| Plan | `docs/plans/YYYY-MM-DD-<topic>.md` | This change: design, spec delta, tasks, rulings |
| Changelog | `CHANGELOG.md` | One `## [Unreleased]` line per user-visible change |

Anything under `docs/` uses Starlight-compatible Markdown: frontmatter with `title` and `description`, no H1 in the body, relative links to `.md` files, plain GitHub-flavored Markdown. Copy the folder's `_template.md` when the project has one; otherwise copy `templates/spec.md`, `templates/decision.md`, or `templates/plan.md` beside this skill.

## Ruling or Decision Record?

Write a **decision record** when the call is any of:

- Hard or costly to reverse: data model, storage, file or wire formats, migrations
- A new dependency, framework, service, or language, or replacing one
- A pattern other code will copy, or one that competes with an existing pattern
- A change to a public interface others depend on: API, CLI, URLs, events, config keys
- Security, privacy, auth, or compliance posture
- Pricing, plans, entitlements, or limits
- A departure: contradicting an accepted decision record, or changing anything a spec states (behavior, a non-goal, a constraint) that the request did not ask to change. Behavior the request asks to change is not a departure; it goes through the plan's spec delta.

Everything else is a **ruling**, one line in the plan's `## Rulings`:

```
- [scope] <decision> — <why> — cost if wrong: <low|medium|high>, <what breaks>
```

The router, the plan, and the PR all use exactly this shape; the PR sorts by the cost level.

Unsure which? Ask "would the reviewer want to second-guess this first?" Yes means a record. A record costs minutes; a lost reason costs re-arguing the decision later. Cost if wrong is what lets the human review the riskiest calls first, so state it honestly.

## Writing a Decision Record (Unattended)

- **Number:** the highest in the decisions folder across the base branch and other open PRs (`gh pr list --state open --json files`), plus one. Picking it right the first time matters: commit bodies cite the number and cannot be changed after the fact. add-finishing-a-development-branch re-checks before pushing.
- **Frontmatter:** `status: accepted`, today's `date`, `decision-makers: agent (unattended run)`. Merging the PR ratifies it; the PR lists every record so the human can overturn one there.
- **Until the PR merges, the record is part of the proposal:** revise it on the branch when review or later work changes the call. "Never rewrite" starts once it is merged.
- **Body:** Context, Options considered (at least the runner-up and why it lost), Decision, Consequences (including what gets harder), Revisit when, Links to the spec and plan.
- **Superseding:** a new record with `supersedes: NNNN`. In the old record change only `status: superseded` and add `superseded-by: NNNN`; never rewrite its text.
- **Where it lands:** a record made while designing goes in the plan commit; one forced mid-implementation goes in the same commit as the code that embodies it.
- **Link it** from the plan's `## Decisions` and from the commit bodies of the code that implements it.

## Specs Are Current Truth

- A spec states what the feature does now. Planned behavior lives in the plan's `## Spec delta` until a task ships it.
- Each task applies its slice of the delta in the same commit as its code and tests. A new feature's spec is created by the first task that implements part of it.
- Each acceptance criterion in a spec names the test that proves it.
- Never edit a spec ahead of the code, and never let code change behavior a spec states without changing the spec in that commit.
- A spec or accepted record contradicts the task? Behavior the request asks to change goes through the spec delta. Anything else binds: follow it, or write the superseding record (for a spec line no record backs, a new record that states the change and links the spec) and change the spec in the commit that changes the behavior. Never deviate silently.

## The Atomic Commit

One commit is one complete, reviewable change that leaves the repo consistent:

- the code
- its tests
- every doc it makes true or stale: the spec slice, a guide or README section, the changelog line, the decision record for a strong call it embodies

```
feat(billing): cap free plan at 3 projects

Support cost per free account rose past the plan's margin.

Spec: docs/specs/billing.md
Decision: docs/decisions/0007-cap-free-tier-at-3-projects.md
Plan: docs/plans/2026-10-25-free-tier-cap.md
```

Conventional Commits subject; the body says why and names the docs it touched, so `git log --grep 0007` finds every commit tied to a decision. Stage by path (`git add -- <paths>`), never `git add -A`, so the commit holds exactly this change. A change starts with one docs-only commit, the plan commit (the plan plus any design-time decision records); after that, docs never trail the code. The one exception is add-finishing-a-development-branch's named fix-up for a gap found before pushing. A pure refactor or a pure docs fix is its own atomic commit.

## Red Flags

| Thought | Reality |
|---------|---------|
| "I'll update the docs at the end" | End-of-run docs describe what you remember, not what you built. Same commit. |
| "Nobody is watching, so a ruling is enough" | Unattended is exactly when the reason gets lost. A strong call gets a record. |
| "The spec should describe what we plan" | Specs describe shipped behavior. The plan holds the delta. |
| "This contradicts decision 0003, but mine is better" | Write the superseding record and flag it in the PR. Never silently. |
| "Too small for a changelog line" | User-visible means a line. Internal-only means none. |
| "I'll tidy the old record's wording" | Accepted records are history. Supersede, don't edit. |

## Integration

**Called by:**
- **add-brainstorming** - Rulings and decision records made while designing
- **add-writing-plans** - Per-task Docs fields and commit messages
- **add-subagent-driven-development** - Applying spec deltas and composing each task's commit
- **add-executing-plans** - Same, when executing inline
- **add-finishing-a-development-branch** - Decision record numbering and the PR's Rulings and Decisions sections
