# ADR-002: workflow_call for Reusability

**Status**: Acceptée  
**Type**: Architecture  
**Decision Date**: 2024-07-29  

## Context

Workflows need to be invoked from multiple app repositories with consistent behavior. Manual copying or forking would cause divergence and maintenance burden.

## Decision

Use GitHub Actions `workflow_call` feature to define reusable workflows. All workflows are called via `uses:` with explicit inputs and secrets.

## Rationale

- `workflow_call` is GitHub's standard pattern for reusable workflows.
- Callers explicitly declare required inputs; no hidden dependencies.
- Easier to update workflows; changes propagate to all callers referencing `@main`.
- Each caller can specify `@v1.0`, `@main`, or a commit SHA for version pinning.

## Implementation

- Each workflow includes `on: workflow_call:` with declared inputs and secrets.
- Calling repo example:
  ```yaml
  jobs:
    release:
      uses: voikyrioh/workflows/.github/workflows/ci-release.yml@main
      with:
        app_name: chess-api
        bump_type: minor
      secrets: inherit
  ```

## Constraints

- All inputs and secrets must be explicitly typed and documented.
- Workflows cannot rely on repository-specific Environment Variables (except those passed via inputs).

## Related

- ADR-003: Secrets handling.

