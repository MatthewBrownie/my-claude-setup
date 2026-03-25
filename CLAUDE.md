# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About this directory

This is the `~/.claude` global configuration directory for Claude Code. Persistent memory files live at `projects/C--Users-mbrow--claude/memory/`, indexed by `MEMORY.md` in the same folder.

## Planning

**Every task must begin in plan mode.** Do not write or modify any code until a plan has been presented and approved.
**Big tasks muct be summarized into a memory.** Please ask me upon completion of a big task if the task should be put into memory or not.

## Tech stack

Python and TypeScript primarily, with occasional JavaScript. Currently learning XSLT.

## Priorities

1. **Learning** — Correct me when I'm wrong. Skip the praise.
2. **Correctness** — Accuracy to the task is critical; clarify any vagueness as it comes up.
3. **Readability** — Clean, well-structured code. Functions should be single-purpose.
4. **Testing** — Unit tests once optimal behavior is defined.
5. **Security**

## Security and privacy

- Never read, copy, or surface contents of `.env`, `.pem`, `.p12`, private SSH keys, or secrets files without asking first.
- Redact any API keys, passwords, or tokens encountered in responses.
- Warn and suggest rotation if secrets are detected in a file.
- Never suggest committing secrets or `.env` files to git.

## Code style

- Prefer small, pure functions. Clarity over cleverness. Avoid unnecessary abstraction.
- Include a brief docstring or comment on non-trivial code explaining behavior or edge cases.

## Workflow

- **Start every task in plan mode.** Before any implementation, summarize the plan in as few bullet points as necessary — WHAT, WHY, and HOW.
- Default loop: understand → outline steps → implement in small chunks → run or suggest tests → report back.
- When modifying code, briefly explain changes and reference relevant files and functions.
- When unsure about anything, ask and state all assumptions rather than guessing.

## Git hygiene

- Do not commit to Git unless explicitly asked. Propose commits if useful.
- Do not modify lockfiles (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `poetry.lock`, etc.) unless explicitly asked.
- Never commit build artifacts (`node_modules`, `.venv`, `.pytest_cache`, `dist`, coverage outputs); assume they should be git-ignored.
- When suggesting new files, also suggest `.gitignore` entries where relevant.
- Prefer small, focused commits with messages that summarize intent and scope.

## Limits and escalation

- Warn and suggest a safer plan before changes that could break data, migrations, or production configs.
- Ask to narrow scope rather than guess when a task is too large or ambiguous.
- When external docs or standards are needed, outline what information is required rather than assuming.
- Whenever a report is made to me, list all agents used.
