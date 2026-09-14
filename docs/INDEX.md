# Documentation Index

## Overview

Infrastructure documentation for **workflows**, a public GitHub Actions repository providing reusable CI/CD pipelines.

## Structure

| Section | Purpose |
|---|---|
| [ARCHITECTURE.md](../ARCHITECTURE.md) | Workflow design, calling conventions, security model. |
| [adr/](adr/INDEX.md) | Architectural decisions (public/private split, workflow composition). |
| [business-rules/](business-rules/INDEX.md) | Operational rules (versioning, deployment approval). |
| [bugs/](bugs/INDEX.md) | Known issues (reference via FIX:ULID tickets). |
| [runbooks/](runbooks/INDEX.md) | Operational procedures (how to use each workflow, inputs/outputs). |

## Quick Links

- **Calling a workflow**: See `runbooks/` for each workflow's integration guide.
- **GitHub repo**: https://github.com/Voikyrioh/workflows (public)
- **Related infra**: Private repo `infra` (contains deploy-version.yml, provision-app.yml, secrets management).

## Public vs. Private

This repo is **PUBLIC**. The infrastructure that consumes these workflows lives in a private **infra** repo.

| Repo | Contents | Access | Purpose |
|---|---|---|---|
| workflows | CI/CD workflow definitions | Public | Shared across app repos |
| infra (private) | Deployment logic, Vault, Ansible | Private | Orchestrates releases to production |

## Skills

Before modifying: `/code-search`, `/dev-task`, `/bugfix`, `/doc-update` (orga-global skills).

