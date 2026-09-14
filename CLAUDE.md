# workflows — Guide pour Claude

## Rôle

Public GitHub Actions workflows repository (INFRA-18 split). Provides reusable CI/CD workflows called via `workflow_call` by application repositories: ci-release.yml (versioning + image build), deploy-app.yml (VPS deployment), discord-notify.yml (notifications).

## Commandes

This repo contains no code to run locally. Workflows are triggered by calling repos via GitHub Actions.

- **Test a workflow locally**: Use `act` (GitHub Actions local runner) to simulate.
- **Publish a workflow change**: Commit and push to `main`; callers referencing `@main` inherit changes immediately.
- **Pin a workflow version**: Callers can pin to a tag (e.g., `@v1.0.0`) instead of `@main`.

## Documentation

Read **ARCHITECTURE.md** first, then **docs/INDEX.md** for all subsections (ADRs, runbooks, business rules).

Runbooks in `docs/runbooks/` show how each workflow is called from app repos.

## Security

This repo is PUBLIC. No secrets, IPs, or internal hostnames in YAML or commits. Secrets passed via `secrets: inherit` from calling repo.

## Skills obligatoires

Before any workflow change: `/code-search` (locate patterns), `/dev-task` (implement feature), `/bugfix` (fix issue), `/doc-update` (maintain docs).

Lire ARCHITECTURE.md puis docs/INDEX.md ; skills : code-search, dev-task, bugfix, doc-update.

