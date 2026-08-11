# AGENTS.md

Operating notes for AI/LLM coding agents (Claude, Cursor, Codes, Copilot, Qwen, Qwythos, etc.).


## Repo Purpose

This code repository contains Kustomize bases and overlays for our installations of the Stalwart Migration Proxy (hereafter, "SMP"). This project helps us route traffic between multiple Stalwart Mail Server backends for zero downtime migrations.


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
- **Do not ever merge a PR** on behalf of a user or perform any other actions that would result in a change to the `main` branch. Actions which result in `main` branch changes should always be done by a human operator. In this repo, merging a PR based against `main` will result in deployments to live environments, so caution must be exercised in these cases.


## Documentation and other text generation

- **Use Markdown** when writing documentation. Most documentation can go in the `README.md` file for now.
- **Use 100-character line lengths** when writing code.
- **Never use em-dash characters** in code files, or other special characters beyond the standard ASCII set.
- **It is okay to use special characters and emoji** in Markdown documentation, provided it is useful to do so. Remain professional.


## Directory structure

This repo is laid out in the following way:

├── .github                           # Directory for GitHub specific configurations
│   └── workflows                     # Directory for GitHub Actions workflow configurations
│       └── validate.yaml             # Workflows for PR validation, run when a PR is opened against `main`
├── AGENTS.md                         # This file, instructions on how LLMs should interact with this repo
├── bases                             # Directory containing Kustomize bases for all installations
│   ├── aws                           # Directory containing Kustomize bases for AWS/ACK resources
│   │   ├── elasticache.yaml          # ElastiCache Redis resources
│   │   ├── kustomization.yaml        # Kustomize configuration for resources in `bases/aws/`
│   │   ├── security-groups.yaml      # AWS security groups
│   │   └── subnet-groups.yaml        # AWS subnet groups used by ElastiCache Redis
│   ├── configmap.yaml                # SMP configuration file
│   ├── deployment.yaml               # Kubernetes deployment for SMP
│   ├── hpas.yaml                     # HorizontalPodAutoscaler for SMP
│   ├── kustomization.yaml            # Kustomize configuration for resources in `bases/`
│   ├── secrets.yaml                  # ExternalSecret for SMP's secret bearer token
│   └── service.yaml                  # Kubernetes Service resources exposing parts of SMP to different audiences
├── Dockerfile.util                   # Builds a Docker image containing helpful Stalwart management utilities
├── overlays                          # Kustomize overlays for various installations
│   ├── tb-dev                        # Kustomize overlays for the tb-dev environment
│   │   ├── aws                       # Kustomize overlays for AWS resources in the tb-dev environment
│   │   │   ├── elasticache.yaml      # ElastiCache Redis resource overlays for the tb-dev environment
│   │   │   ├── security-groups.yaml  # AWS security group resource overlays for the tb-dev environment
│   │   │   └── subnet-groups.yaml    # AWS subnet group resource overlays for the tb-dev environment
│   │   ├── configmap.yaml            # SMP configuration file overlay for the tb-dev environment
│   │   ├── deployment.yaml           # Deployment resource overlay for the tb-dev environment
│   │   ├── hpas.yaml                 # HorizontalPodAutoscaler overlay for the tb-dev environment
│   │   ├── kustomization.yaml        # Kustomize configuration for overlays in `overlays/tb-dev`
│   │   ├── secrets.yaml              # ExternalSecret overlay for the tb-dev environment
│   │   └── service.yaml              # Service overlay for the tb-dev environment
│   └── tb-prod                       # Kustomize overlays for the tb-prod environment
│       ├── aws                       # Kustomize overlays for AWS resources in the tb-prod environment
│       │   ├── security-groups.yaml  # AWS security group resource overlays for the tb-prod environment
│       │   └── subnet-groups.yaml    # AWS subnet group resource overlays for the tb-prod environment
│       ├── configmap.yaml            # SMP configuration file overlay for the tb-prod environment
│       ├── hpas.yaml                 # HorizontalPodAutoscaler overlay for the tb-prod environment
│       ├── kustomization.yaml        # Kustomize configuration for overlays in `overlays/tb-prod`
│       ├── secrets.yaml              # ExternalSecret overlay for the tb-prod environment
│       └── service.yaml              # Service overlay for the tb-prod environment
├── README.md                         # Basic information about this repo
└── util                              # Directory for handy utility scripts not part of normal operation
    └── kustomize-build-all.sh        # Script to test Kustomize builds for all environments
