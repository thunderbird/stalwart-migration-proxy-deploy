# Agents

This repository uses the following AI agents:

- **Qwythos** – The primary development agent responsible for building Docker images and managing the `stalwart-migration-proxy-deploy` repository.

---

## Core Principles

- **Never push directly to `main`** – All modifications should be made on a new or existing non-main branch. Use the branch as a working area, then open a pull request when you're ready.
- **Create new branches for each task** – This provides a clean audit trail of what each agent or person is working on.
- **Use descriptive branch names** – Include the task context (e.g., `agent-added-agents-guide`, `use-custom-alpine`).
- **Include a diff URL with your instructions** – A URL like https://github.com/thunderbird/stalwart-migration-proxy-deploy/compare/add-agent-instructions allows reviewers to see the exact changes without navigating the repo.
- **Push to the working branch first** – If the file already exists, update it; if not, create it. Avoid committing directly to main.

## PR Guidelines

When you open a pull request, include a sentence such as:

> *This code was written by a coding agent (Qwythos) as instructed.*

This makes it clear that automated assistance was used, and helps satisfy any organizational or compliance requirements.
