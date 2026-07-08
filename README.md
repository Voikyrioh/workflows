# workflows

Workflows GitHub Actions **réutilisables** consommés par les repos d'apps
(INFRA-18 : extraits d'`infra-as-code` pour permettre son passage en privé —
un repo public ne peut pas appeler un workflow d'un repo privé).

⚠️ Ce repo est PUBLIC : jamais d'IP, de port SSH, de nom d'hôte interne ni de
secret dans les fichiers ou les commits. Les valeurs sensibles passent par les
GitHub Secrets des repos appelants (`secrets: inherit` / secrets déclarés).

## Contenu

| Workflow | Rôle | Inputs |
|---|---|---|
| `ci-release.yml` | Bump semver (tag git) + build/push image GHCR + release GitHub | `app_name`, `bump_type`, `build_args` (opt) |
| `deploy-app.yml` | Build+push image, `.env` depuis Vault (AppRole), scp + `docker compose up` sur le VPS | `app_name`, `image_name` + secrets SSH/Vault/GHCR |

## Usage (repo d'app)

```yaml
jobs:
  release:
    uses: voikyrioh/workflows/.github/workflows/ci-release.yml@main
    with:
      app_name: mon-app
      bump_type: ${{ inputs.bump_type }}
    secrets: inherit
```

Le déploiement d'une version passe par `deploy-version.yml` (repo infra,
workflow_dispatch app_name + vX.Y.Z) ou le dashboard.
