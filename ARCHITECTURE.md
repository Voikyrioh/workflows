# ARCHITECTURE

## Overview

Public repository of reusable GitHub Actions workflows. Each workflow is consumed via `workflow_call` by application repositories to standardize CI/CD pipelines across the organization. This repo is PUBLIC; production secrets are passed via `secrets: inherit` from calling repos.

## Design Principle

**Public workflows + Private infra-as-code**: GitHub Actions cannot call private workflows from public repos. This repo contains the workflow definitions; deployment infrastructure and secrets management live in a private `infra` repo.

## Workflows

- **`ci-release.yml`** — Bump semver, tag git, build/push Docker image to GHCR, create GitHub release.
- **`deploy-app.yml`** — Build/push Docker image, fetch `.env` from Vault via AppRole, SCP config, `docker compose up` on VPS.
- **`discord-notify.yml`** — Post formatted Discord embeds to release or deploy webhook (non-blocking; never fails pipeline).

[ARCHITECTURE.md]: workflows

## Calling Conventions

All workflows are called via `workflow_call` with explicit inputs and secrets (never assume ENV). Secrets are inherited from calling repo:

```yaml
jobs:
  release:
    uses: voikyrioh/workflows/.github/workflows/ci-release.yml@main
    with:
      app_name: chess-api
      bump_type: minor
    secrets: inherit
```

## Security

- No IP addresses, SSH keys, or internal hostnames in YAML.
- Secrets passed only via GitHub Secrets (declared in calling repo via `provision-app.yml`).
- Discord webhooks are non-mandatory; workflow continues even if webhook unavailable.
- GHCR token and SSH key controlled by calling repo; not stored here.

