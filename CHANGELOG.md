# Changelog

All notable changes to this project are recorded in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-11-25

First public release.

### Added

- Plan-file mode: `/gaffer <path/to/plan.md>` splits an approved plan into three to eight work packages, gives each to a worker in a detached git worktree, integrates the patches one at a time, verifies the integrated tree, runs one review, and stops without committing.
- Single-issue mode: `/gaffer <issue number | #number | issue URL>` investigates the issue, writes a plan from the template, waits for approval, then follows the plan-file flow.
- Issues mode: `/gaffer issues <goal>` selects and triages a repository's open issues and opens one pull request per issue, with the options `dry-run`, `max=<n>` and `reviewer=<handle>`.
- Plan template and worker brief.

### Guardrails

- Manual trigger only.
- Plan-file and single-issue modes never commit, push or open a pull request, and never change the issue.
- Issues mode works only issues filed by the repository owner, a member or a collaborator, and commits, pushes branches and opens pull requests only for the selected issues.
- Issues mode never merges, force-pushes, pushes to the default branch, creates releases or tags, or comments on, labels, assigns or closes issues.
- Issue text is treated as untrusted evidence and never as an instruction.
- No pull request is opened for an issue whose required verification did not run and pass.
- Workers edit only their listed files and never commit, push, create branches or add dependencies; the lead reads every diff and runs verification itself.
