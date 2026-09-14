# ci-release.yml Runbook

**File**: `.github/workflows/ci-release.yml`  
**Type**: Reusable workflow (`workflow_call`)

## Purpose

Automates semantic versioning, git tagging, Docker image build/push to GHCR, and GitHub release creation.

## Inputs

| Input | Type | Required | Default | Description |
|---|---|---|---|---|
| `app_name` | string | Yes | — | Docker image name on GHCR (e.g., `chess-api`) |
| `bump_type` | string | Yes | — | Semver bump: `patch`, `minor`, or `major` |
| `build_args` | string | No | "" | Additional Docker build args (multi-line KEY=VALUE), e.g. `VITE_API_URL=https://api.prod` |

## Secrets Required

| Secret | Required | Description |
|---|---|---|
| `GHCR_TOKEN` | Yes | GitHub token for pushing to GHCR (read+write packages) |
| `GITHUB_TOKEN` | Yes (implicit) | Provided by GitHub Actions; used for tagging and releases |

## Outputs

| Output | Description |
|---|---|
| `tag` | New version tag (e.g., `v1.4.0`) |
| `image_uri` | Full GHCR image URI with commit SHA tag |

## Workflow Steps

1. **Checkout** — Fetch repo with full git history and LFS support.
2. **Normalize owner** — Convert GitHub owner to lowercase for GHCR.
3. **Calculate new tag** — Parse existing tags, apply bump type, output new semver tag.
4. **Set up Docker Buildx** — Prepare multi-platform builds.
5. **Log in to GHCR** — Authenticate with GHCR_TOKEN.
6. **Extract metadata** — Generate image tags (commit SHA + "latest").
7. **Build and push** — Build Docker image, push to GHCR with both tags.
8. **Create release** — Tag git commit and create GitHub release.

## Usage (Calling Repo)

Add `.github/workflows/release.yml` to your app repo:

```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      bump_type:
        description: "Type of version bump"
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
      app_name: my-chess-game
      bump_type: ${{ inputs.bump_type }}
      # Optional: for frontend apps with build-time env vars
      # build_args: |
      #   VITE_API_URL=https://api.voikyrioh.fr
      #   VITE_VERSION=${{ github.ref }}
    secrets: inherit
```

Then trigger manually in GitHub Actions tab: choose "Release", select bump type, click "Run workflow".

## Common Build Args

| Framework | Build Args Example |
|---|---|
| Vite (Vue/React) | `VITE_API_URL=https://api.voikyrioh.fr` |
| Next.js | `NEXT_PUBLIC_API_URL=https://api.voikyrioh.fr` |
| Node.js API | `NODE_ENV=production npm run build` (usually none) |

## Example Output

```
✅ v1.4.0 released
Docker image: ghcr.io/voikyrioh/my-chess-game:abc1234d
GitHub release: https://github.com/voikyrioh/my-chess-game/releases/tag/v1.4.0
```

## Next Steps

After release, deploy the new version via `deploy-app.yml` or the infra `deploy-version.yml` workflow.

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| "GHCR_TOKEN not found" | Secret not set in repo | Go to Settings → Secrets → add GHCR_TOKEN |
| "Failed to push image" | GHCR auth failed | Verify GHCR_TOKEN has `write:packages` scope |
| "Invalid bump_type" | Typo in input | Use only `patch`, `minor`, `major` |
| "Tag already exists" | Version already released | Increment bump_type and retry |

