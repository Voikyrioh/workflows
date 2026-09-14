# Business Rules

## Operational Rules

| Rule | Domain | Status |
|---|---|---|
| Release workflow must tag git; no automatic tags on every merge | CI/Release | Active |
| Versioning follows semver (major.minor.patch); manual bump via workflow input | Versioning | Active |
| Deploy workflow is non-blocking; workflow continues even if Discord webhook fails | Discord Notifications | Active |
| Docker images pushed to GHCR with two tags: commit SHA + "latest" | Container Registry | Active |

## Future Rules (Not Yet Implemented)

- Automatic rollback on deploy failure.
- Approval gates for production releases.
- Automatic changelog generation from commit messages.

