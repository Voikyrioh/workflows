# deploy-app.yml Runbook

**File**: `.github/workflows/deploy-app.yml`  
**Type**: Reusable workflow (`workflow_call`)

## Purpose

Builds and pushes a Docker image to GHCR, retrieves deployment configuration from Vault, and deploys to production VPS via SSH + `docker compose up`.

## Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| `app_name` | string | Yes | App name (= folder name in `/opt/infra/apps/` on VPS) |
| `image_name` | string | Yes | Docker image name on GHCR (same as ci-release.yml) |

## Secrets Required

| Secret | Required | Description |
|---|---|---|
| `SSH_HOST` | Yes | VPS hostname or IP (no internal IPs in public repo) |
| `SSH_PRIVATE_KEY` | Yes | Private SSH key for deployment user on VPS |
| `SSH_PORT` | Yes | SSH port (usually 22 or custom) |
| `VAULT_ADDR` | Yes | Vault server URL |
| `VAULT_ROLE_ID` | Yes | AppRole role ID |
| `VAULT_SECRET_ID` | Yes | AppRole secret ID |
| `GHCR_TOKEN` | Yes | GitHub token for pulling image from GHCR |
| `NPM_TOKEN` | No | Optional; for private npm packages (backend only) |

## Workflow Steps

1. **Checkout** — Fetch repo.
2. **Log in to GHCR** — Authenticate for image pull.
3. **Extract metadata** — Generate image tags.
4. **Build and push** — Build image, push to GHCR (tags: commit SHA + "latest").
5. **Authenticate to Vault** — Use AppRole to get auth token.
6. **Fetch deployment config** — Retrieve `.env` from Vault path `secret/apps/{app_name}`.
7. **Set up SSH** — Configure SSH key, known_hosts, SSH config.
8. **SCP config** — Copy `.env` to VPS at `/opt/infra/apps/{app_name}/.env`.
9. **Deploy** — SSH into VPS and run:
   ```bash
   cd /opt/infra/apps/{app_name}
   docker compose pull
   docker compose up -d
   ```

## Usage (Calling Repo)

Typically triggered by infra's `deploy-version.yml` workflow after a release. However, can also be invoked manually:

```yaml
# .github/workflows/deploy.yml (in calling repo)
name: Deploy

on:
  workflow_dispatch:

jobs:
  deploy:
    uses: voikyrioh/workflows/.github/workflows/deploy-app.yml@main
    with:
      app_name: chess-api
      image_name: chess-api
    secrets: inherit
```

Or by infra (private repo):

```yaml
# infra/.github/workflows/deploy-version.yml
- uses: voikyrioh/workflows/.github/workflows/deploy-app.yml@main
  with:
    app_name: chess-api
    image_name: chess-api
  secrets:
    SSH_HOST: ${{ secrets.VPS_SSH_HOST }}
    SSH_PRIVATE_KEY: ${{ secrets.VPS_SSH_KEY }}
    # ... other secrets
```

## Setup on VPS

Before deploying for the first time:

1. **Create app directory**: `mkdir -p /opt/infra/apps/{app_name}`
2. **Create docker-compose.yml**: Template should pull image and mount volumes.
3. **Store secrets in Vault**: `vault kv put secret/apps/{app_name} DB_URL=… API_KEY=…`

Example `docker-compose.yml`:

```yaml
version: '3.8'
services:
  app:
    image: ghcr.io/voikyrioh/${IMAGE_NAME}:latest
    container_name: ${APP_NAME}
    restart: unless-stopped
    env_file: .env
    environment:
      - NODE_ENV=production
    networks:
      - traefik-public
    labels:
      - traefik.enable=true
      - traefik.http.routers.${APP_NAME}.rule=Host(`${DOMAIN}`)
      # ... other Traefik config

networks:
  traefik-public:
    external: true
```

## Vault Configuration

Secrets in Vault are stored at: `secret/apps/{app_name}`

Example path: `secret/apps/chess-api`

```bash
# On infra machine (with Vault CLI):
vault kv put secret/apps/chess-api \
  DB_URL=postgresql://user:pass@db.local/chess \
  REDIS_URL=redis://cache.local:6379 \
  API_KEY=secret-key-here
```

## Example Output

```
✅ Deployed chess-api v1.4.0
- Image built and pushed: ghcr.io/voikyrioh/chess-api:abc1234d
- Config fetched from Vault: secret/apps/chess-api
- Deployed to: vps.internal.net
- Service status: Running
```

## Monitoring

After deployment, check logs:

```bash
# SSH into VPS
ssh user@vps.internal.net -p 22

# Check service status
docker compose -f /opt/infra/apps/chess-api/docker-compose.yml logs -f

# Verify container is running
docker ps | grep chess-api
```

## Rollback

If deployment fails, manually revert:

```bash
ssh user@vps.internal.net
cd /opt/infra/apps/chess-api
git log --oneline  # see previous release tag
docker compose down
# Then re-run deploy workflow with previous version
```

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| "SSH_HOST not found" | Secret not set | Add SSH_HOST to repo secrets |
| "Permission denied (publickey)" | SSH key mismatched | Verify SSH_PRIVATE_KEY matches VPS authorized_keys |
| "Vault auth failed" | AppRole creds invalid | Rerun `provision-app.yml` or verify in infra |
| "docker compose: command not found" | Old Docker installation | VPS needs Docker Compose v2 (docker compose not docker-compose) |
| "Cannot pull image" | GHCR auth failed | Ensure GHCR_TOKEN has read:packages scope |

