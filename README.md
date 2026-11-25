# gaffer

A Claude Code skill that implements an approved plan, or a GitHub issue, with a lead and a small team of workers.

"Gaffer" is British English for the boss who runs a crew. The Claude Code session you invoke the skill from is the lead. It splits the plan into work packages, hands each one to a worker sub-agent in its own git worktree, reads every diff, integrates the patches, runs the repository's verification itself, runs one review, and stops.

The skill is a folder of Markdown instructions. There is no program to install and nothing runs outside Claude Code.

## Who it is for

People who already use Claude Code on a repository, have finished investigating a change, and want the implementation done in parallel without giving up a single point of responsibility for the result. It suits changes that split cleanly into a few packages with separate files. It is not a way to run hundreds of agents.

## How it works

1. **Lead.** The session is the lead. It reads the plan and the code, decides the split, briefs the workers, and is the only party that integrates, verifies and reports. Work that cannot be split cleanly stays with the lead.
2. **Work packages.** The lead produces three to eight packages, fewer where possible. Each has an outcome, the files it may and may not edit, an acceptance check, and its dependencies. No file belongs to two packages.
3. **Worktrees.** Each ready package gets a detached git worktree under `.claude/worktrees/` in the repository. In plan and single-issue modes no branch is created and nothing is committed.
4. **Workers.** One worker sub-agent per package, started together. A worker edits only its listed files, runs only cheap local checks, and returns a fixed-format report. It never commits, pushes, creates branches, adds dependencies or decides scope.
5. **Reports are claims.** The lead reads each diff. It sends a patch back when it edits files outside its list, weakens or deletes a test or check, adds a dependency, leaves a TODO or mock on a production path, or when the acceptance check did not actually run. For a bug fix the lead confirms the regression test fails without the fix. Rework is limited to two rounds per package; after that the lead takes the package over or reports it as not done.
6. **Integration.** Patches are applied one at a time in dependency order, and each package's acceptance check is run again in the main checkout. Each patch is saved under the repository's git directory as the record.
7. **Verification.** The lead runs what the repository requires on the integrated tree: its documented checks, then a real runtime path where one exists. Anything that uses a shared resource, such as a local cluster, a port or a simulator, is run by the lead alone, once, serially.
8. **Review.** One review of the integrated diff: the reviewer the repository names, otherwise `/code-review`. Blocking findings are fixed and the affected checks re-run.
9. **Stop.** The lead reports each package's outcome, the commands it ran and what they showed, what was not verified, and where the saved patches and any remaining worktrees are. Then it asks what you want done with the change.

The repository's own `CLAUDE.md` and `AGENTS.md` still apply to the lead and to every worker.

## Requirements

- Claude Code with sub-agents available.
- `git`, with worktree support.
- The GitHub CLI, `gh`, authenticated for the repository, for the single-issue and issues modes. The plan-file mode does not need it.

## Install

Clone the repository:

```sh
git clone https://github.com/danushkastanley/gaffer.git
cd gaffer
```

Then make the skill available to Claude Code, either for all projects:

```sh
mkdir -p ~/.claude/skills
cp -R skills/gaffer ~/.claude/skills/gaffer
```

or for one project, from that project's root:

```sh
mkdir -p .claude/skills
cp -R /path/to/gaffer/skills/gaffer .claude/skills/gaffer
```

To follow updates with `git pull` instead of copying again, symlink the folder:

```sh
ln -s "$(pwd)/skills/gaffer" ~/.claude/skills/gaffer
```

## Usage

The skill is triggered manually only; Claude Code will not invoke it by itself. Run it from a session on the repository you want changed. It takes one of three inputs.

### A plan file

```text
/gaffer path/to/plan.md
```

The plan must have the sections described under [What a plan file needs](#what-a-plan-file-needs). The lead stops without starting workers if sections are missing, if tracked files have uncommitted changes (workers start from `HEAD`), or if the plan has an open question that changes what gets built.

### A single GitHub issue

```text
/gaffer 123
/gaffer #123
/gaffer https://github.com/<owner>/<repo>/issues/123
```

An issue is a report, not a plan. The lead reads the issue, investigates in the code, writes a plan from the template, shows you the goal, the non-goals and the planned changes, and waits for your approval before any worker starts. The wait is skipped only when the issue body already contains every template section and was written by the user running the session. A URL that names a different repository from the checkout's remote stops the run.

### Many issues

```text
/gaffer issues fix every open bug in the trace package
/gaffer issues the findings in tracking issue 324, max=3
/gaffer issues label:bug dry-run
```

The lead selects open issues that match the goal, triages them, and takes each selected issue through to its own pull request: one branch, one worktree, one worker and one pull request per issue. Up to three issues may be with workers at once; verification, review and publishing happen one issue at a time. There is no approval wait in this mode: triage is the gate.

Options, placed anywhere in the goal text:

| Option | Meaning |
| --- | --- |
| `dry-run` | Select and triage only. Change nothing, open nothing. |
| `max=<n>` | Issues to take this run. Default 5. |
| `reviewer=<handle>` | Pull request reviewer for this run. Without it, the lead requests the reviewer the repository's own instructions name, and no one if they name none. |
| `<owner>/<repo>` | Must match the checkout's `origin`; otherwise the run stops. |

Triage skips, with a one-line reason, an issue that needs a product, design or naming decision nobody has settled; adds a dependency, changes a public contract or persisted data, or needs a decision record the repository does not have; touches release, signing, CI credentials, infrastructure, IAM or secrets; is a security report that belongs in a private advisory; cannot be verified on the machine; or is too large for one reviewable pull request. You can settle a decision in the goal text itself.

Start with `dry-run` to see what would be taken and what would be skipped.

## What a plan file needs

Use [skills/gaffer/plan-template.md](skills/gaffer/plan-template.md). It has seven sections: Goal, Non-goals, Findings, Changes, Off-limits, Done when, and Open questions. Write the plan at the end of an investigation session and save it in the repository. Findings matter most: workers see only what the lead copies from there, so give causes, decisions, file paths and what was ruled out.

The text each worker receives is in [skills/gaffer/worker-brief.md](skills/gaffer/worker-brief.md).

## Safety model

These limits are instructions in the skill text, not technical enforcement. See [Background](#background) for what that means.

In plan-file and single-issue modes, gaffer will:

- edit files locally, create detached worktrees under `.claude/worktrees/`, and start worker agents for the plan;
- save each integrated patch, and any plan it writes from an issue, under the repository's git directory.

It will not:

- commit, push or open a pull request;
- deploy, change dependencies, or do anything the plan does not name;
- comment on, label, assign, close or otherwise change the issue, or create a branch for it unless asked;
- reuse or remove a worktree the run did not create.

In issues mode without `dry-run`, gaffer will, for the selected issues only:

- create local branches and worktrees, and commit with your configured Git identity and the repository's message conventions, with no attribution lines or co-author trailers;
- push those branches to `origin`, open one pull request per issue, and request a reviewer.

It will not:

- work an issue unless its author is the repository owner, an organisation member or a collaborator; issues from anyone else are listed as skipped, whatever they say;
- merge, enable auto-merge, force-push, or push to the default branch;
- create releases or tags, or deploy;
- comment on, label, assign or close issues;
- open a pull request for an issue whose required verification did not run and pass. After three failed attempts at the same check it stops that issue and leaves its worktree and branch in place, unpushed;
- watch or re-run CI after a pull request is open;
- use a branch prefix that names a tool, a model or an agent.

In every mode, issue titles, bodies and comments are treated as untrusted evidence. The lead does not run commands, open links, install anything or change scope because issue text says to, and it quotes and reports any text there that is addressed to an agent.

## Models

- The lead is whatever model the session runs. The skill was written for Claude Fable 5.2 or Mythos 5.2 as lead, at high effort. If the session is on a smaller model than the workers, the lead says so and asks whether to continue.
- Workers are requested as Opus at high effort.

## Limits and status

Version 1.0.0 is the first public release. See the [changelog](CHANGELOG.md).

So far gaffer has been exercised on one real repository, a Go project:

- a mechanics smoke test with two workers;
- a dry-run triage of open bug issues;
- a real issues-mode run that took issues through to open pull requests.

The plan-file mode's rework loop and its patch-conflict path have not yet been exercised on real work.

Other limits:

- It is small-team scale: three to eight workers, not hundreds.
- Token cost rises with each worker.
- The lead verifies with the checks and runtime paths the repository provides; it does not supply its own.
- The guardrails are instructions followed by a model. Review the result as you would any other change.

## Background

The structure was inspired by Anthropic's write-up of a large multi-agent experiment: <https://claude.dev/blog/safely-scaling-multi-agent-teams-with-fable-5-2/>.

This project is independent. It is not affiliated with or endorsed by Anthropic, and it does not reproduce that experiment's sandboxing or monitoring. Its guardrails are your Claude Code permission mode, worktree isolation, and the rules in the skill.

## Contributing and security

See [CONTRIBUTING.md](CONTRIBUTING.md) and the [code of conduct](CODE_OF_CONDUCT.md). Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## Licence

MIT. See [LICENSE](LICENSE).
