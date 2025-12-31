# pnpm Package Template

**A minimal, production-ready pnpm monorepo template for building and publishing TypeScript/Node packages.**

This template provides a ready-to-use setup for multi-package repositories with pnpm as the package manager, Turborepo for task orchestration, Biome for linting/formatting, Changesets for versioning and releases, and GitHub Actions for CI.

## Features

- Monorepo workspaces: `packages/**`, `apps/**`, `examples/**`
- pnpm-first workflows: `pnpm install`, `pnpm run <script>`
- Turborepo pipelines for `dev`, `build`, `start`, `types:check`
- Biome-based lint and format with consistent project style
- Versioning and publishing via Changesets
- CI workflows for linting and releases

## Getting Started

```bash
# Install dependencies
pnpm install

# Development
pnpm run dev

# Build all workspaces
pnpm run build

# Type-check
pnpm run types:check

# Lint (check)
pnpm run lint

# Lint (auto-fix)
pnpm run lint:fix

# Format code
pnpm run format
```

## Workspace Layout

- `packages/` – Each published or internal package lives here
- `apps/` – Optional applications (e.g., docs, demos) consuming packages

Example package structure:

```
packages/your-package/
├── src/
│   ├── index.ts
│   └── ...
├── package.json
├── tsconfig.json
├── tsup.config.ts (optional)
└── README.md
```

## Releases

This template uses Changesets.

```bash
# Create a changeset describing your changes
npx changeset

# Version packages (CI will also do this)
pnpm run version

# Publish updated packages (CI release workflow)
pnpm run release
```

Ensure `NPM_TOKEN` is configured in CI for publishing.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md) and follow the [Code of Conduct](./CODE_OF_CONDUCT.md).

## Security

Please report vulnerabilities privately as described in [SECURITY.md](./SECURITY.md). Replace the contact email with your own when using this template.

## License

MIT License – see [LICENSE](./LICENSE).

By contributing to this template, you agree that your contributions will be licensed under the MIT License.
