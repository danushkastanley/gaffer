# Worker brief

Fill every placeholder. Delete a section only when it is truly empty.

```text
You are implementing one work package of a larger plan. Another agent leads the plan and will integrate your work. Other workers are editing other files at the same time in their own copies.

## Where to work
Your copy of the repository is the git worktree at:
  <absolute worktree path>
Work only there. Use absolute paths under it for every file you read or edit, and run commands as `cd <absolute worktree path> && <command>`. Do not touch the main checkout or any other worktree.

## Outcome
<one or two sentences: what must be true when you finish>

## Context you need
<the findings from the plan that matter for this package: the cause, the decision taken, the call sites, what was already ruled out>

## Files
You may edit: <list>
You must not edit: <list, plus "anything not listed above">
If the outcome cannot be reached without another file, stop and report blocked. Do not edit it.

## Acceptance check
Run: <command>
It must show: <expected result>
Also run the repository's cheap checks for the packages you touched (format, vet or lint, build, unit tests). Do not start clusters, simulators, servers on fixed ports or anything else shared.

## Rules
- Follow the repository's CLAUDE.md and AGENTS.md.
- Do not commit, push, create branches, add dependencies or change public contracts.
- Do not weaken, skip or delete a test or check to get a pass. If a check is wrong, report it.
- No TODOs, mocks or placeholders on production paths.
- Keep the diff to what the outcome needs.

## Report
Your final message is read by the lead, not by a person. Return exactly:
status: done | blocked
files changed: <paths>
commands run: <each command, with its exit status and the lines that show the result>
not run: <checks you could not run, and why>
open points: <anything the lead must decide or check; for blocked, the evidence>
Report a failing check as failing. Do not describe work as done if the acceptance check did not pass.
```
