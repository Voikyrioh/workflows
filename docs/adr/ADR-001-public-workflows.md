# ADR-001: Public Workflows Repository

**Status**: Acceptée  
**Type**: Architecture  
**Decision Date**: 2024-07-29 (INFRA-18)  

## Context

GitHub Actions cannot call private workflows from public repositories. However, application repositories are often public (e.g., open-source games). Organization needs a way to provide reusable CI/CD workflows to all app repos without requiring them to be private.

## Decision

Extract reusable workflows into a separate **public** repository (`voikyrioh/workflows`). Deployment orchestration and secrets remain in a **private** `infra` repository.

## Rationale

- Allows app repos to remain public while consuming shared CI/CD logic.
- Workflows repo contains no secrets, IPs, or sensitive infrastructure details.
- Private `infra` repo can call workflows via `uses:` and add deployment layer.
- Clear separation of concerns: CI (public) vs. orchestration (private).

## Implementation

- Repository: https://github.com/Voikyrioh/workflows (PUBLIC)
- Workflows: `ci-release.yml`, `deploy-app.yml`, `discord-notify.yml` (see runbooks).
- Calling repos inherit secrets via `secrets: inherit` (each repo declares its own secrets in GitHub Actions settings).
- Infra repo orchestrates versions and handles deployment logic.

## Security Constraints

- No IP addresses, hostnames, or internal network details in workflows.
- No hardcoded secrets or API keys.
- Sensitive values passed only via GitHub Secrets (declared in calling repo).

## Related

- ADR-002: Workflow composition via `workflow_call`.
- ADR-003: Secrets inheritance mechanism.

