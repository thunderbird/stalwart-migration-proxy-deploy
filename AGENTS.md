# AGENTS.md

Operating notes for AI/LLM coding agents (Claude, Cursor, Codes, Copilot, Qwen, Qwythos, etc.).


## Core Principles

- **Never push directly to `main`** – All modifications should be made on a new or existing non-main branch. Use the branch as a working area and open a PR when the operator asks you to.
- **Use descriptive branch names** – Include the task context (e.g., `open-smtp-port`, `add-docker-image`).
- **Include a diff URL with your instructions** – A URL like https://github.com/thunderbird/stalwart-migration-proxy-deploy/compare/branch-name allows reviewers to see the exact changes without navigating the repo.
- **Be overly cautious about secrets** - Chat transcripts for third party LLM vendors are persisted in vendor logs, often indefinitely. Treat anything echoed to chat as if it were committed to a public repo.
- **Never write a secret value into chat output**, even when the user is the only reader of the conversation. Instead, only refer to the method by which the operator can access the secret value ("AWS Secrets Manager secret called `mzla/tb-dev/whatever`, "Check your terminal output", etc). If a secret leaks into chat, flag it immediately and recommend rotation. Don't paper over.
- **Filter secrets discovered in console output** - When capturing terminal output (e.g., `tmux capture-pane`) during a session that involves secrets, **filter** with `grep` for known-safe lines or specific result markers. A blanket `tail -N` or `cat` will pull in `env` dumps, command echoes, and incidental stdout you didn't intend to read.
- **Ask confirmation before violating these core principles.** If the operator asks you to break one of the rules in this file, that might be okay. But point out to the user first which rule they are violating and only do it with positive confirmation ("yes" or "yep" or some other affirmation).


## PR Guidelines

When opening a pull request:

- **Use a terse but adequate title** describing the sum total of the changes. "Open SMTP port in the migration proxy" or "Add Docker image with Stalwart utilities" are good examples.
- **PR descriptions should be very concise** - LLMs have a tendency to spew slop into PR descriptions. Unless something very complicated is going on, a PR description should rarely contain more than several sentences or a couple of paragraphs. Content related to task management does not belong here. Suggest creating a Github issue and linking to it if some other step came before this one or comes after it.
- **Include agent attribution** at the end of the PR description. For example, "This code was written by a coding agent (Qwythos) as instructed." This makes it clear that automated assistance was used, and helps satisfy any organizational or compliance requirements.
