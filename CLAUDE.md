# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a pnpm monorepo template for building and publishing TypeScript/Node packages. It uses:

- **pnpm** as the package manager (required: `pnpm >= 10.0.0`, currently using `pnpm@10.31.0`)
- **Node.js** (required: `node >= 22`, CI uses Node 24)
- **Turborepo** for task orchestration across workspaces
- **Biome** for linting and formatting (extends `@pelatform/biome-config/base`)
- **Changesets** for versioning and publishing
- **Husky** for Git hooks
- **Commitlint** with `@commitlint/config-conventional` for commit message linting

**Important**: This project uses pnpm as the package manager. Always use `pnpm` commands instead of `npm` or `bun`.

## Common Commands

```bash
# Install dependencies
pnpm install

# Development (persistent tasks across all workspaces)
pnpm run dev

# Build all workspaces
pnpm run build

# Type-check all workspaces
pnpm run types:check

# Lint (check only - runs biome check + turbo run lint)
pnpm run lint

# Lint (auto-fix - runs biome check --write --unsafe + turbo run lint -- --fix)
pnpm run lint:fix

# Format code with Biome (writes to files)
pnpm run format

# Check formatting without writing (currently checks, not formats)
pnpm run format:check

# Clean build artifacts
pnpm run clean

# Deep clean (removes node_modules, .husky, .turbo, pnpm-lock.yaml)
pnpm run clean:all

# Create a changeset for versioning
npx changeset

# Version packages (updates versions and runs pnpm install)
pnpm run version

# Publish packages (via scripts/publish.sh)
pnpm run release
```

## Running Tasks in Specific Workspaces

Use Turbo's `--filter` flag to run tasks in specific packages:

```bash
# Build a specific package
pnpm run build --filter=@pelatform/core

# Run dev in a specific workspace
pnpm run dev --filter=@pelatform/web

# Run tests for a specific package
pnpm run test --filter=@pelatform/main
```

## Workspace Structure

```
packages/           # Published or internal packages
├── core/          # Core package
└── main/          # Main package

apps/              # Optional applications consuming packages
├── web/           # Web application
└── docs/          # Documentation

examples/          # Example implementations
├── next/          # Next.js example
└── vite/          # Vite example
```

**Note**: This is a template repository. Workspace directories contain placeholder directories with `.gitkeep` files:

- `packages/` → `core/`, `main/` (example package locations)
- `apps/` → `web/`, `docs/` (example application locations)
- `examples/` → `next/`, `vite/` (example implementations)

You need to implement actual packages and applications in these directories.

Workspaces are defined in both `package.json` (`packages/**`, `apps/**`) and `pnpm-workspace.yaml` (`packages/*`, `apps/*`, `examples/*`). The `**` pattern matches nested directories, while `*` matches only immediate children.

**Internal Dependencies**: When a workspace depends on another workspace in the same monorepo, use the workspace protocol (e.g., `"@pelatform/core": "workspace:*"`) in `package.json`. This ensures:

- pnpm uses the local version during development via symlink
- After `pnpm run version`, Changesets replaces `workspace:*` with the actual published version
- Published packages reference the correct versions from the npm registry

The Changesets configuration enables `bumpVersionsWithWorkspaceProtocolOnly` to enforce this pattern and `updateInternalDependencies: "patch"` to bump internal dependency versions as patch changes.

## Turborepo Pipeline

The `turbo.json` defines task dependencies:

- `build`: Depends on `^build` (build dependencies first). Outputs: `.next/**`, `.source/**`, `build/**`, `dist/**`, `out/**`
- `dev`: Persistent, uncached
- `start`: Depends on `^build`, persistent
- `lint`: No dependencies
- `types:check`: Depends on `^build`
- `clean`/`clean:all`: Uncached, non-persistent

## Code Quality

**Biome** (`biome.jsonc`) extends `@pelatform/biome-config/base` for consistent linting and formatting across the project. Lint-staged with Husky pre-commit hooks ensures code quality before commits.

**Lint-staged configuration**:

- TypeScript/JavaScript: Biome check with auto-fix
- Markdown/YAML: Prettier formatting
- JSON/HTML: Biome format

**Code Standards**:

- **Indentation**: 2 spaces
- **Line Width**: 100 characters
- **Quotes**: Single quotes for JavaScript/TypeScript
- **Semicolons**: Always required
- **Trailing Commas**: All
- **Arrow Parentheses**: Always

## CI/CD

GitHub Actions workflows:

- **Lint** (`.github/workflows/lint.yml`):
  - Runs on PRs to main
  - Uses Node 24 with pnpm
  - Runs: `pnpm install --frozen-lockfile`, `pnpm run build`, `pnpm run lint:fix && pnpm run lint`, `pnpm run types:check`

- **Release** (`.github/workflows/release.yml`):
  - Triggered on pushes to main with changes in `.changeset/**` or `packages/**`
  - Configured git user: `Lukman Aviccena <lukmanaviccena@gmail.com>`
  - Uses Node 24 with pnpm (note: CI uses Node 24 while `package.json` specifies `node >= 22`)
  - Sets up `.npmrc` with `NPM_TOKEN` for publishing
  - Runs build and lint with `pnpm run lint:fix && pnpm run lint`
  - Uses `changesets/action@v1` to create release PRs or publish to npm
  - Requires secrets: `NPM_TOKEN` (required), `GITHUB_TOKEN` (default), or `GH_PAT` (fallback)
  - Sets `NODE_OPTIONS: --max-old-space-size=4096` for memory-intensive operations

## Commit Convention

Commitlint enforces conventional commits with types: `feat`, `feature`, `fix`, `refactor`, `docs`, `build`, `test`, `ci`, `chore`. Configured in `.commitlintrc.cjs`.

**Format**: `type(scope): description`

**Examples**:

- `feat(template): add package scaffolding script`
- `fix(ci): correct pnpm run command in workflow`
- `docs(readme): clarify release process`
- `refactor(workspace): simplify outputs in turbo.json`

## Changesets Configuration

Changesets is configured in `.changeset/config.json`:

- Changelog generation uses `@changesets/changelog-github` for GitHub releases
- Repository: `pelatformlabs/template`
- Access level: `public` (packages are published to public npm)
- Internal dependencies are bumped as `patch` versions
- Base branch: `main`
- Workspace protocol-only version bumping enabled (`bumpVersionsWithWorkspaceProtocolOnly`)

## Publishing

The `scripts/publish.sh` script:

1. Finds all `package.json` files under `packages/` (excludes `node_modules`)
2. For each package:
   - Skips if `"private": true` in package.json
   - Publishes with `pnpm publish --no-git-checks` (continues even if publish fails)
3. Creates Git tags via `changeset tag`

Requires `NPM_TOKEN` environment variable. The script is designed to handle multiple packages and continues on individual publish failures.

## Git Hooks

Husky is configured for pre-commit hooks via `pnpm prepare`. The pre-commit hook runs lint-staged which:

- Runs Biome check with auto-fix on TypeScript/JavaScript files (`*.{js,jsx,ts,tsx,cjs,mjs,cts,mts}`)
- Runs Prettier on Markdown/YAML files (`*.{md,yml,yaml}`)
- Runs Biome format on JSON/HTML files (`*.{json,jsonc,html}`)

This ensures code quality before commits are created. All lint-staged commands use `--no-errors-on-unmatched` to avoid failing on empty match sets.

## Architecture Notes

### Package Manager Specification

The root `package.json` specifies `"packageManager": "pnpm@10.31.0"`, which enforces this specific version of pnpm via Corepack. To enable Corepack:

```bash
corepack enable
corepack prepare pnpm@10.31.0 --activate
```

If you encounter version mismatches, ensure Corepack is enabled and the correct version is prepared.

### Workspace Protocol

Internal workspace dependencies must use the `workspace:*` protocol in package.json:

```json
{
  "dependencies": {
    "@pelatform/core": "workspace:*"
  }
}
```

This protocol:

- Links to local workspace versions during development
- Automatically resolves to published versions after `pnpm run version`
- Is enforced by Changesets' `bumpVersionsWithWorkspaceProtocolOnly` setting

### Workspace Pattern Behavior

- **package.json workspaces**: Uses `packages/**` and `apps/**` (matches nested directories)
- **pnpm-workspace.yaml**: Uses `packages/*`, `apps/*`, `examples/*` (matches immediate children only)

When adding new workspaces, ensure they match at least one of these patterns. The `examples/` directory is only defined in `pnpm-workspace.yaml`, not in `package.json` workspaces.

### Turbo Task Execution

Turbo executes tasks in parallel by default, respecting dependency graphs:

- `build` tasks run topologically (dependencies before dependents)
- `^build` means "all build tasks in workspace dependencies"
- Use `--filter` to target specific workspaces
- Turbo caches outputs based on inputs (files, env vars)

Example:

```bash
# Build core and its dependencies
pnpm run build --filter=@pelatform/core

# Build all packages that depend on core
pnpm run build --filter=...@pelatform/core
```
