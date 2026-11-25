# Security policy

## Supported versions

The latest release on the `main` branch is supported.

## Reporting a vulnerability

Please report vulnerabilities privately through GitHub's private vulnerability reporting on this repository: open the **Security** tab and choose **Report a vulnerability**, or go to <https://github.com/danushkastanley/gaffer/security/advisories/new>.

Do not open a public issue, discussion or pull request for a suspected vulnerability.

A useful report includes:

- the file and the instruction concerned;
- the invocation and mode used;
- what the lead or a worker did, and what the skill says it should have done;
- the Claude Code version and model, if the behaviour was observed in a real run.

## What counts

gaffer is a set of instructions that an agent follows with your permissions, so a vulnerability here is an instruction, or a gap between instructions, that could lead an agent to do harm. For example:

- an instruction that could lead the lead or a worker to act on untrusted issue text, such as running a command, opening a link or changing scope because an issue says to;
- a path by which the skill could exceed its stated limits: committing or pushing in plan-file or single-issue mode, merging, force-pushing, pushing to the default branch, editing issues, or opening a pull request without passing verification;
- a way for an issue from someone who is not the owner, a member or a collaborator to be worked in issues mode;
- a way for a worker to edit outside its worktree or its listed files without the lead's checks catching it.

## What does not count

- Behaviour that follows from the permission mode you chose for your own Claude Code session.
- Vulnerabilities in Claude Code, `git` or the GitHub CLI themselves. Please report those to their maintainers.

The limits in this skill are instructions, not a sandbox. The README's safety model and background sections describe what is and is not enforced.
