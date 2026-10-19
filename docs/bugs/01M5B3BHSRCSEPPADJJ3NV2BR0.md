# 01M5B3BHSRCSEPPADJJ3NV2BR0 — Release : npm 401 quand le Dockerfile attend un secret buildx

## Symptômes
`Release` de Voikyrioh/Voikyrioh.fr en échec au build Docker (run 39332672457, 2026-10-19) :
`npm error 401 Unauthorized - GET https://npm.pkg.github.com/download/@Voikyrioh/vue-translate/... authentication token not provided`.

## Repro
`gh workflow run "Release" -R Voikyrioh/Voikyrioh.fr -f bump_type=minor`.

## Cause
`ci-release.yml` et `deploy-app.yml` ne passaient le token npm qu'en **build-arg** `GITHUB_TOKEN`. Le Dockerfile de voikyrioh.fr (antérieur à INFRA-18) le lit en **secret buildx** (`RUN --mount=type=secret,id=github_token`) : fichier vide, npm sans token. Dernière release verte v1.3.0 (2026-05-17), avant la bascule vers ces workflows.

## Fix
Les deux workflows passent aussi `secrets: github_token=...` à `docker/build-push-action`. Le secret buildx n'est ni dans l'historique de l'image ni dans le cache gha, contrairement au build-arg (conservé pour les Dockerfiles existants qui font `ARG GITHUB_TOKEN`). Aucun changement côté repos appelants.

## Suivi
Migrer les Dockerfiles en `ARG GITHUB_TOKEN` vers le secret buildx, puis retirer le build-arg.
