# Runbooks

Operational procedures for consuming and maintaining each workflow.

## Workflows

| Workflow | File | Purpose | Runbook |
|---|---|---|---|
| CI Release | `ci-release.yml` | Bump version, tag, build image | [ci-release.md](ci-release.md) |
| Deploy App | `deploy-app.yml` | Build, push image, deploy to VPS | [deploy-app.md](deploy-app.md) |
| Discord Notify | `discord-notify.yml` | Post release/deploy notifications | [discord-notify.md](discord-notify.md) |

## Common Integration Pattern

Most app repos use this pattern (see calling repo's `.github/workflows/`):

```yaml
# .github/workflows/release.yml (in calling repo, e.g., chess-api)
name: Release

on:
  workflow_dispatch:
    inputs:
      bump_type:
        description: "patch | minor | major"
        required: true
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  release:
    uses: voikyrioh/workflows/.github/workflows/ci-release.yml@main
    with:
      app_name: chess-api
      bump_type: ${{ inputs.bump_type }}
    secrets: inherit
```

## Requirements for Calling Repos

To use these workflows, an app repository must:

1. **Have GitHub Actions enabled** (default for public repos).
2. **Declare required secrets** in GitHub repo settings → Settings → Secrets and Variables → Actions:
   - For `ci-release.yml`: `GHCR_TOKEN` (GitHub token for container registry)
   - For `deploy-app.yml`: `SSH_HOST`, `SSH_PRIVATE_KEY`, `SSH_PORT`, `VAULT_ADDR`, `VAULT_ROLE_ID`, `VAULT_SECRET_ID`, `GHCR_TOKEN`
   - For `discord-notify.yml`: `DISCORD_WEBHOOK_PLAYER`, `DISCORD_WEBHOOK_STAFF` (optional; non-blocking if missing)

3. **Run `provision-app.yml`** (from private `infra` repo) to auto-populate secrets on app onboarding.

## See Also

- [ci-release.md](ci-release.md) — How to trigger a release.
- [deploy-app.md](deploy-app.md) — How to deploy an app.
- [discord-notify.md](discord-notify.md) — How to notify Discord.

