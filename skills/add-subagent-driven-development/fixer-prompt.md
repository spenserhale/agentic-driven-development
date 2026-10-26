# Fixer Prompt Template

Use this template to fix named findings from the final reviewer, a red test suite, or an OPEN package-review item. Pick the model by tier (see Fixing in SKILL.md): tiers 1-2 standard, tier 3 most capable with add-systematic-debugging.

```
Task tool (general-purpose):
  description: "Fix [finding ids] (tier N)"
  prompt: |
    You fix the named findings and nothing else.

    ## Findings

    [closed list, one per line: `F<n> file:line - problem - fix`, or a failing test id]

    ## Previous Rounds

    [For a second round or an escalation: the last fixer's report — what it changed,
    what failed, the evidence. Otherwise "none".]

    ## Rules

    - Fix only the listed findings. Touch only their files, plus the test file for each
      regression test. No refactors.
    - For each finding: read the code, make the fix, add one regression test that fails
      without it. Run only the touched tests (fast), never the full suite.
    - If a doc states the behavior you changed (a spec line, a guide), correct it in the
      same change so the coordinator's commit stays atomic. For shared docs (the feature
      spec, CHANGELOG.md), give the corrected text and where it goes in your report
      instead of editing the file.
    - Tier 3: find the root cause with add-systematic-debugging before changing code.
    - If a finding's stated cause or fix turns out wrong, or the code does not match,
      don't guess: report it NOT_FIXED with the reason.
    - Never run git commands that change state (add, commit, stash, checkout, switch,
      reset, restore); read-only status, diff, log, and show are fine. The coordinator
      commits your work.
    - Kill only processes you started, by PID.

    ## Report (at most 150 words; no file contents or logs)

    One line per finding — never leave one off:
    `FIXED - F<n>` or `NOT_FIXED - F<n> - <reason>`
    Then the test command's result line and the files you changed.
```

**Fixer returns:** a closed list. The coordinator runs the touched tests and commits each fix atomically (`fix(<scope>): ...`). NOT_FIXED items go one tier up after two failed rounds; still open after tier 3, they become rulings with `cost if wrong: high` in the PR.
