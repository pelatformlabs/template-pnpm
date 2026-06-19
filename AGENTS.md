# AGENTS.md

This is a **pnpm monorepo template** for TypeScript/Node packages. Workspace directories (`packages/`, `apps/`, `examples/`) contain only `.gitkeep` placeholders — this is scaffolding for new projects.

## Toolchain

- **Package manager**: pnpm only. `packageManager` field enforces `pnpm@11.8.0` via corepack.
- **Build orchestration**: Turborepo (`turbo@2.9.18`)
- **Lint/format**: Biome, extends `@pelatform/biome-config/base` (`biome.jsonc`). No separate ESLint/Prettier config.
- **Versioning**: Changesets with `@changesets/changelog-github`, repo `pelatformlabs/template-pnpm`
- **Git hooks**: Husky v9 + lint-staged + commitlint (conventional commits)
- **TypeScript**: `5.9.3` via pnpm catalog

## Commands

```bash
pnpm install                     # frozen lockfile in CI: pnpm install --frozen-lockfile
pnpm run build                   # dependsOn ^build (deps first)
pnpm run types:check             # dependsOn ^build — run AFTER build
pnpm run lint                    # biome check + turbo run lint
pnpm run lint:fix                # biome check --write --unsafe + turbo run lint -- --fix
pnpm run format                  # biome format --write
pnpm run format:check            # biome check --write (writes, not read-only)
pnpm run dev                     # persistent, uncached, all workspaces
pnpm run clean                   # turbo run clean
pnpm run clean:all               # rm -rf .husky .turbo pnpm-lock.yaml node_modules
npx changeset                    # create version changeset
pnpm run version                 # changeset version && pnpm install
pnpm run release                 # bash ./scripts/publish.sh
```

**Turbo filtering**:

```bash
pnpm run build --filter=@pelatform/core
pnpm run build --filter=...@pelatform/core   # + dependents
```

## CI verification chain (also CI)

```
pnpm install --frozen-lockfile → pnpm run build → pnpm run lint:fix && pnpm run lint → pnpm run types:check
```

`types:check` depends on `^build` — must run after build succeeds.

## Workspace conventions

Both `package.json` and `pnpm-workspace.yaml` define workspace patterns as `packages/*`, `apps/*`, `examples/*` (immediate children, not nested). All three directories are workspace roots.

**Internal dependencies** MUST use the workspace protocol:

```json
{ "dependencies": { "@pelatform/core": "workspace:*" } }
```

Changesets' `bumpVersionsWithWorkspaceProtocolOnly: true` enforces this. `updateInternalDependencies: "patch"` auto-bumps internal deps.

## Commit convention

Conventional commits enforced by commitlint. Allowed types: `feat`, `feature`, `fix`, `refactor`, `docs`, `build`, `test`, `ci`, `chore`.
Format: `type(scope): description`

## Publishing

- CI release workflow triggers on push to `main` when `.changeset/**` or `packages/**` change.
- Uses `changesets/action@v1` with `setupGitUser: false` (git user configured inline).
- Publishes via `pnpm run release` → `scripts/publish.sh` which iterates all `packages/*/package.json`, skips `"private": true`, runs `pnpm publish --no-git-checks`, then `changeset tag`.
- Requires secrets: `NPM_TOKEN`, `GITHUB_TOKEN` (or `GH_PAT`).
- `NODE_OPTIONS: --max-old-space-size=4096` set for memory-intensive builds.

## pre-commit

Husky runs lint-staged on commit:

- `*.{js,jsx,ts,tsx,cjs,mjs,cts,mts}` → `biome check --write --no-errors-on-unmatched`
- `*.{md,yml,yaml}` → `prettier --write`
- `*.{json,jsonc,html}` → `biome format --write --no-errors-on-unmatched`

## Turbo gotchas

- `build` inputs include `.env*` files — env changes invalidate build cache.
- `dev` and `start` are persistent (long-running), `start` depends on `^build`.
- `types:check` has no outputs defined — always runs, never cached.

## Code style (via @pelatform/biome-config/base)

Indent: 2 spaces. Line width: 100. Single quotes. Semicolons: always. Trailing commas: all. Arrow parentheses: always.
