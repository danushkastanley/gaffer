# Contributing

Thank you for taking the time to improve gaffer. The project is a set of instructions, so small, well-evidenced changes are the most useful ones.

## Proposing a change

- For a change to how the skill behaves, open an issue first and describe the problem before writing the change. This saves you work if the direction needs discussion.
- Typos, broken links and wording fixes that do not change behaviour can go straight to a pull request.
- Security problems do not belong in public issues. See [SECURITY.md](SECURITY.md).

## What a good change looks like

- **It comes from a real run.** Say what run showed the need: the invocation, the mode, what the lead or a worker did, and what should have happened instead. Changes based on what might go wrong are hard to judge and tend to make the instructions longer without making them better.
- **It keeps instructions short and testable.** Each rule should be something a reader can check against a transcript or a diff. Prefer changing or removing a rule to adding one.
- **It says how it was tried.** Run the changed skill on a repository and describe what you saw. If you could not try a path, say so.
- **It stays in scope.** One behaviour per pull request. Do not add tooling, workflows or files the change does not need.

## Style

- UK English.
- Plain and specific. No marketing language and no emoji.
- Keep the README accurate: if a change alters what the skill does, update the README and the changelog in the same pull request.

## Branches and commits

- Use a short, lowercase, kebab-case branch name with a prefix that describes the work, for example `fix/` or `feature/`.
- Do not use branch prefixes that name a tool, a model or an agent.
- Write conventional-style commit subjects, for example `fix: stop the lead reusing an existing worktree`.

## Conduct

Taking part in this project means following the [code of conduct](CODE_OF_CONDUCT.md).
