# ADR 0017: Release Automation with `release-please`

## Status

Accepted – implementation completed in the current iteration

## Context and Problem Statement

- Versioning and changelog maintenance were previously done manually, even though commits already follow the Conventional Commits scheme.
- The repository contains multiple `package.json` files (root + UI5 apps under `app/*`) that must be versioned in sync.
- For future SAP BTP deployments (Cloud Foundry), a traceable release process should be established that can be easily integrated into CI/CD.
- GitHub is used as the source code and automation platform; branch strategy follows `develop` (integration) and `main` (release) with short-lived feature branches.

## Decision Factors

- **Traceability & reviewability** – release steps should remain reviewable via PRs.
- **Multi-package capability** – consistent versioning across root and app packages.
- **Stability & maintainability** – established, actively maintained solution without proprietary extensions.
- **CI/CD integration** – easy connection to GitHub Actions and future extensibility (deployments, artifacts).
- **Onboarding & documentation** – the process must be easy to understand and document.

## Considered Options

### Option A – `semantic-release`

- **Core elements**: CLI tool, plugins for changelog, git tags, optional npm publishing.
- **Pros**: Very common, large plugin ecosystem, fully automated releases.
- **Cons**: Less focus on PR-based approval, more custom configuration for monorepos, writes directly to `main`.

### Option B – `release-please` (Google)

- **Core elements**: release PRs, manifest/monorepo support, GitHub Action.
- **Pros**: PR-centered workflow, `node-workspace` plugin syncs multiple packages, native GitHub integration.
- **Cons**: Focused on the GitHub ecosystem, no direct npm publishing (not required here).

### Option C – Manual process

- **Core elements**: manual version bumps + changelog entries.
- **Pros**: No additional tooling dependency.
- **Cons**: Error-prone, unscalable, lacks automation for planned CI/CD extensions.

## Decision

- We choose **Option B – `release-please` with manifest workflow**.
- Configuration via `release-please-config.json` + `.release-please-manifest.json`, including the `node-workspace` plugin for `app/timetable` and `app/timetracking`.
- GitHub Action (`.github/workflows/release-please.yaml`) responds after successful `main` builds (`workflow_run`) and creates/updates release PRs with changelog, version bumps, and tags – the actual tag is created only after the release PR is merged.
- A local dry run via `npx release-please release-pr --dry-run` remains mandatory before the first real run.

## Consequences

- **Positive**: automated, reviewable releases; consistent versions across all packages; changelog is continuously maintained.
- **Negative/risks**: dependency on the release-please project; Conventional Commits discipline required; tokens/permissions must be maintained.
- **Follow-ups**: integrate the release step into future BTP deployments; regularly review the action version; add release notes templates once deployments go live.

## References

- Configuration: `release-please-config.json`, `.release-please-manifest.json`
- GitHub Action: `.github/workflows/release-please.yaml`
- Documentation: `README.md` (release process), `docs/ARCHITECTURE.md` section “Transport & Lifecycle Governance”
