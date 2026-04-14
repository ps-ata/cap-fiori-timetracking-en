# ADR 0004: Consistent TypeScript Tooling and Developer Workflow

## Status

Accepted - Migration to full TypeScript

## Context and Problem Statement

The project started with a mixed setup of CAP JavaScript examples and new TypeScript code. Without a binding toolchain, there were missing type informations, manual edits in `gen/`, and broken imports. We needed a workflow that automates type safety, generation, and formatting while also supporting UI5 workspaces.

## Decision Factors

- Automatic type generation from CDS without a direct dependency on `gen/`.
- Hot reload for service development with TypeScript transpile.
- Consistent formatting before commits.
- Compatibility with UI5 subprojects (workspaces in `app/*`).

## Considered Options

### Option A - Minimal setup with `cds watch` and manual typing

- Direct `cds watch` without typer, imports from `gen/`.
- No centralized format or build scripts.

### Option B - Script-based workflow with typer and tooling

- `npm run watch` starts `cds-tsx w` for TypeScript hot reload (`package.json`).
- `@cap-js/cds-typer` generates models under `@cds-models` automatically on `.cds` changes; path alias in `tsconfig.json` and `package.json` (`imports` section).
- `npm run format` runs Prettier over relevant directories.
- `npm run generate-entry-point` (dev-cap-tools) is an optional helper script to regenerate entry points/CLI wrappers.

## Decision

We choose Option B. The workspace uses `@cap-js/cds-typer` as a dev dependency and imports types via `import { TimeEntry } from '#cds-models/TrackService';`. `npm run watch` is the standard command for local development because it combines transpile and service restart. UI5 apps remain integrated as Yarn/NPM workspaces and can be started in parallel.

## Consequences

- Positive: Type-safe handlers, commands, and services without direct use of `gen/`.
- Positive: Developers follow clear scripts (`watch`, `build`, `format`, optional `generate-entry-point` for CLI wrappers).
- Positive: The path alias `#cds-models/*` avoids fragile relative paths.
- Negative: Dev CAP tools require an up-to-date CLI version when using `generate-entry-point`; otherwise no manual steps for types are needed.
- Negative: New developers must install tooling (`cds-tsx`, `dev-cap-tools`) before they can start.

## References

- `package.json`
- `tsconfig.json`
- `@cds-models/TrackService/index.ts`
- `.github/copilot-instructions.md`
