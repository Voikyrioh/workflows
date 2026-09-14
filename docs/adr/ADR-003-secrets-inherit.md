# ADR-003: Secrets via `secrets: inherit`

**Status**: Acceptée  
**Type**: Security  
**Decision Date**: 2024-07-29  

## Context

Workflows need access to deployment secrets (SSH keys, GHCR tokens, Vault credentials) without hardcoding them.

## Decision

All secrets are passed via `secrets: inherit` from the calling repository. Calling repos declare their own secrets in GitHub Actions settings.

## Rationale

- Secrets are repository-specific (e.g., SSH_HOST, VAULT_ROLE_ID depend on the app and environment).
- `secrets: inherit` prevents workflows repo from needing to store or manage any secrets.
- Each app repo explicitly declares which secrets it provides.
- Aligns with principle of least privilege: workflows never have more secrets than needed.

## Implementation

- Workflows declare required secrets in `on.workflow_call.secrets`.
- Calling repo sets these secrets in GitHub Actions settings.
- Calling workflow passes secrets via: `secrets: inherit` or explicit mappings:
  ```yaml
  secrets:
    SSH_HOST: ${{ secrets.SSH_HOST }}
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
  ```

## Calling Repo Setup (via provision-app.yml)

- `provision-app.yml` (infra repo) automatically sets secrets when onboarding a new app:
  - SSH_HOST, SSH_PRIVATE_KEY, SSH_PORT
  - VAULT_ADDR, VAULT_ROLE_ID, VAULT_SECRET_ID
  - GHCR_TOKEN
  - (Optional) NPM_TOKEN, DISCORD_WEBHOOK_PLAYER, DISCORD_WEBHOOK_STAFF

## Related

- ADR-001: Public workflows (no secrets stored).

