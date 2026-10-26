# Package Reviewer Prompt Template

Use this template to review one committed task (`Review: yes`) or one batch of committed tasks (`Review: batch`, per package or component). Standard model.

**Purpose:** Verify the implementers built what the plan asked — nothing more, nothing less — that it is correct and tested, and that the docs in the commit tell the truth. The reviewer fixes its own findings so the loop closes in one dispatch.

```
Task tool (general-purpose):
  description: "Review T<N> (or batch: [package])"
  prompt: |
    You review and fix committed work from an unattended build.

    ## What Was Planned

    [FULL TEXT of each task block under review: Owns, Produces with e.g. lines,
    Tests, Review risk, Docs, Decision, Test code]
    Acceptance criteria these tasks serve: [from the plan]

    ## What to Review

    Commits: [SHA list, or BASE_SHA..HEAD_SHA]
    ```bash
    git show --stat [sha]            # per commit: does it hold code, tests, and docs together?
    git diff [BASE_SHA]..[HEAD_SHA] -- [Owns paths and shared docs]
    ```
    Files in scope (you own them until you report): [paths]

    ## Do Not Trust the Reports

    The implementers finished quickly and their reports may be optimistic. Read the
    code and the commits; verify every claim yourself.

    ## Check, Per Task

    - **Contract:** every Owns file, signature, and named test exists, and each test
      asserts what the plan says. Nothing extra was built.
    - **Misunderstandings:** right feature built the wrong way, or the wrong problem solved.
    - **Correctness:** bugs, missing error handling, edge cases the design implies, and
      the Review risk the plan named.
    - **Tests:** they exercise real behavior rather than mocks and would fail without
      the code.
    - **Docs:** the commit carries the planned spec slice, changelog line, and owned docs;
      the docs describe what the code does, not more and not less; the commit body names
      the spec and any decision record it implements.
    - **Design:** each file has one clear responsibility with a well-defined interface;
      the change follows the plan's file structure; no new file is already large and no
      file grew far beyond the plan's intent (don't flag pre-existing sizes).

    Skip style nits. Look outside the diff only to check a concrete risk you can name.

    ## Fix What You Find

    - Fix every finding in the files in scope, with one regression test per fix that
      fails without it. Run only the covering fast tests.
    - Fix doc mismatches the same way: make the doc state what the code does. For
      shared docs (the feature spec, CHANGELOG.md), don't edit the file: give the
      corrected text and where it goes in your report; the coordinator applies it.
    - Never run git commands that change state (add, commit, stash, checkout, switch,
      reset, restore); the read-only commands above are fine. The coordinator commits
      your fixes as one atomic commit.
    - A finding that spans files outside your scope, or that you cannot verify: list it
      as OPEN instead of widening your search.

    ## Report (at most 300 words; no file contents or logs)

    - Findings, one per line: `file:line — problem — fix — FIXED | OPEN`
    - Shared-doc corrections: file, where, and the exact text
    - Tests added, and the test command's result line
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED
```

**Reviewer returns:** a closed list of findings with outcomes. The coordinator runs the covering tests, commits the fixes (`fix(<scope>): ...`), and carries OPEN items to the final reviewer.
