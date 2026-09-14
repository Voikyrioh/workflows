# discord-notify.yml Runbook

**File**: `.github/workflows/discord-notify.yml`  
**Type**: Reusable workflow (`workflow_call`)  
**Design**: DSC-01 — Single source of truth for Discord embed format; non-blocking (never fails pipeline).

## Purpose

Posts formatted Discord embed notifications for releases (player channel) and deployments (staff channel). Runs in parallel with other jobs; never blocks release or deployment if webhook is unavailable.

## Inputs

| Input | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | `release` (player channel) or `deploy` (staff channel) |
| `version` | string | Yes | Version/tag (e.g., `v1.4.0`) |
| `changelog` | string | No (unless type=release) | Release notes / changelog (markdown) |
| `service` | string | No (unless type=deploy) | Service name being deployed |
| `status` | string | No (unless type=deploy) | `success` or `failure` |

## Secrets Required

| Secret | Required | Applies To | Description |
|---|---|---|---|
| `DISCORD_WEBHOOK_PLAYER` | No (type=release) | Release notifications | Webhook URL for player/public channel |
| `DISCORD_WEBHOOK_STAFF` | No (type=deploy) | Deploy notifications | Webhook URL for staff/internal channel |

## Workflow Steps

1. **Validate inputs** — Check type, version, and required fields.
2. **Format embed** — Build Discord embed JSON with color, title, description, thumbnail.
3. **Post webhook** — Send POST to Discord webhook.
4. **Handle failure** — Log if webhook missing or POST failed; never exit with error code.

## Usage

### Release Notification

```yaml
- uses: voikyrioh/workflows/.github/workflows/discord-notify.yml@main
  with:
    type: release
    version: v1.4.0
    changelog: |
      - Added new chess openings
      - Fixed pawn promotion bug
      - Performance improvements
  secrets:
    DISCORD_WEBHOOK_PLAYER: ${{ secrets.DISCORD_WEBHOOK_PLAYER }}
```

### Deploy Notification

```yaml
- uses: voikyrioh/workflows/.github/workflows/discord-notify.yml@main
  with:
    type: deploy
    version: v1.4.0
    service: chess-api
    status: success
  secrets:
    DISCORD_WEBHOOK_STAFF: ${{ secrets.DISCORD_WEBHOOK_STAFF }}
```

## Discord Embed Format

### Release (type=release)

```
┌─────────────────────────────────┐
│ 🎉 New Release: chess-api v1.4.0 │
│─────────────────────────────────│
│ - Added new chess openings      │
│ - Fixed pawn promotion bug      │
│ - Performance improvements      │
│─────────────────────────────────│
│ Released: 2024-09-14 15:30 UTC  │
└─────────────────────────────────┘
```

Color: Green (#2ecc71)

### Deploy (type=deploy)

Success:
```
┌──────────────────────────────────┐
│ ✅ Deploy Successful            │
│──────────────────────────────────│
│ Service: chess-api               │
│ Version: v1.4.0                  │
│ Deployed: 2024-09-14 15:35 UTC   │
└──────────────────────────────────┘
```

Color: Green (#2ecc71)

Failure:
```
┌──────────────────────────────────┐
│ ❌ Deploy Failed                 │
│──────────────────────────────────│
│ Service: chess-api               │
│ Version: v1.4.0                  │
│ Time: 2024-09-14 15:35 UTC       │
│ Check logs for details.          │
└──────────────────────────────────┘
```

Color: Red (#e74c3c)

## Setup Discord Webhooks

1. **Create Discord server channel** (if not already exists):
   - Player channel: `#releases` (or similar)
   - Staff channel: `#deployments` (or similar)

2. **Create webhook** for each channel:
   - Right-click channel → "Edit Channel" → "Integrations" → "Webhooks" → "New Webhook"
   - Copy webhook URL

3. **Add secret to repo**:
   - GitHub repo → Settings → Secrets → New repository secret
   - Name: `DISCORD_WEBHOOK_PLAYER` (for releases) or `DISCORD_WEBHOOK_STAFF` (for deploys)
   - Value: Paste webhook URL

## DSC-01: Non-Blocking Design

**Principle**: Discord notifications are "nice-to-have"; they never fail the pipeline.

If webhook is missing (secret not set):
- Log: `[WARN] DISCORD_WEBHOOK_PLAYER not configured; skipping notification`
- Workflow: Continues successfully

If webhook POST fails (network error, Discord down):
- Log: `[WARN] Failed to post to Discord: HTTP 503`
- Workflow: Continues successfully

This ensures releases and deployments succeed even if Discord is unavailable or webhook URLs are misconfigured.

## Customization (DSC-01: Central Format)

The embed format (title, colors, fields) is defined **only in this workflow**. All calling repos inherit the same visual appearance. Do not duplicate or modify formatting in app repos.

To update Discord appearance:
- Edit `.github/workflows/discord-notify.yml` in this repo
- Changes apply to all apps using the workflow

## Troubleshooting

| Issue | Cause | Fix |
|---|---|---|
| "No notification posted" | Webhook secret not set | Add `DISCORD_WEBHOOK_PLAYER` or `DISCORD_WEBHOOK_STAFF` to repo secrets |
| "Webhook URL invalid" | Copied wrong URL or expired | Re-create webhook in Discord, copy full URL |
| "Embed formatting looks wrong" | Discord API changed | Check Discord embed documentation; file issue in workflows repo |
| "Webhook works in test, not in CI" | Repo secret not inherited | Use `secrets: inherit` in calling workflow; verify secret is set in this repo too |

