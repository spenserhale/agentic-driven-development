# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

A flat collection of agent skills (installable with `npx skills`) that ships the coupled
**spec → plan → implement → verify** workflow tuned to run **fully unattended** and finish by
opening a pull request. There is no application code — the "product" is the skill content in
`skills/`.

It is a de-hooked clone of `AgenticDevelopmentSkills`: no plugin manifest, no SessionStart hook,
no auto-injection. A run starts only when the user invokes
`/agentic-driven-development <task>`.

## Structure

```
skills/
  agentic-driven-development/SKILL.md   # boot/router — holds the Unattended Operation Contract
  add-<name>/SKILL.md                   # workflow sub-skills, loaded on demand
docs/add/
  specs/   # brainstorming output
  plans/   # writing-plans output
```

## Conventions

- Every skill is prefixed `add-` (the boot skill is `agentic-driven-development`) so it never
  collides with the `hdd-` skills from Human-Driven Development when both are installed flat.
- Inside skill bodies, reference other skills by their prefixed name: `add-writing-plans`.
- The skills are **coupled** — they hand off in a fixed order. When editing one, check that its
  hand-off targets and `## Integration` lists still point at skills that exist here.
- Workflow artifacts default to `docs/add/specs/` and `docs/add/plans/`.

## The load-bearing invariant: unattended operation

This variant's whole identity is that it **never pauses for a human**. When editing any skill here,
preserve the Unattended Operation Contract behavior:

- No approval gates, no clarifying questions, no option menus, no confirmations.
- Every "ask your human partner" / "get consent" / "present options and wait" from the upstream
  skills must stay converted to **decide + record**.
- Non-obvious decisions go in the **Assumptions & Decisions** log (seeded in the spec, carried to
  the plan, reproduced in the PR body).
- Only unrecoverable external blockers stop a *task* (→ deferred TODO in the PR), never the whole run.
- The terminal state is always a pushed branch + open PR — never a question back to the user.

If you find yourself adding a human touchpoint, you are turning this back into the `hdd` variant.
The human-in-the-loop counterpart is the separate `HumanDrivenDevelopment` (`hdd-`) collection.

## Editing skill content

Skill text shapes agent behavior — treat it like code, not prose. Preserve the tuned elements
(Red Flags tables, rationalization lists, "Iron Law" framing) unless deliberately changing behavior.
