# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains reusable GitHub Actions workflows for Teraslice asset repositories. These workflows are called from individual asset repos (e.g., elasticsearch-assets, kafka-assets) via `workflow_call` triggers.

## Workflow Architecture

All workflows are in `.github/workflows/` and designed to be called from consuming repositories:

### Core Workflows (called by asset repos)
- **asset-test.yml** - Tests assets on Linux (Node 22, 24) and macOS. Runs lint, tests, and builds dev artifacts.
- **asset-build-and-publish.yml** - Builds, publishes to NPM, creates GitHub releases with asset ZIPs, and announces in Slack. Only runs when package.json version is greater than latest release.
- **prerelease-asset-test.yml** / **prerelease-asset-build-and-publish.yml** - Same as above but for prerelease branches (uses `teraslice-cli@prerelease`).

### Support Workflows (called by core workflows)
- **cache-docker-images.yml** / **cache-asset-bundles.yml** - Cache Docker images and asset bundles if not already cached.
- **refresh-docker-cache.yml** / **refresh-asset-bundles.yml** - Force refresh caches (clears old, pulls fresh). Only runs on master.
- **check-docker-limit.yml** / **check-github-api-limit.yml** - Check rate limits before/after operations.

## Key Implementation Details

- Node versions are hardcoded in matrices as `[22, 24]` - must be kept in sync between test and build workflows.
- Asset builds use `pnpm dlx teraslice-cli assets build` (or `teraslice-cli@prerelease` for prereleases).
- Version comparison uses `scripts/semver-tool.sh` (expected in consuming repos).
- Docker images list is generated via `pnpm docker:listImages` and stored in `./images/image-list.txt`.
- Test workflows append `-dev.<short-sha>` to version for artifact naming.
- Releases are tagged as `v<version>` and marked as prerelease by default.

## Required Secrets/Variables in Consuming Repos

Secrets: `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `NPM_TOKEN`, `SLACK_BOT_TOKEN`, `RELEASES_PRIVATE_KEY`
Variables: `RELEASES_APP_ID`, `SLACK_RELEASES_CHANNEL_ID`

## Expected Scripts in Consuming Repos

- `pnpm run setup` - Initial setup after install
- `pnpm build` - Build the asset
- `pnpm lint` - Run linting
- `pnpm test:all` - Run all tests
- `pnpm docker:listImages` - Generate Docker image list
- `pnpm docker:saveImages` - Save Docker images to cache
- `pnpm docker:limit` - Check Docker Hub rate limit
- `scripts/semver-tool.sh` - Semver comparison utility
- `scripts/publish.sh` - NPM publish script
- `scripts/build-documentation.sh` - Build docs (if website exists)
