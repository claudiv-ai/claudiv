# Contributing to Claudiv

Thank you for your interest in contributing to Claudiv. This guide covers the development setup and contribution process.

## Repository Structure

Claudiv is **not a monorepo**. Each package lives in its own independent Git repository and is pulled into this parent repo as a **git submodule**:

| Package | Submodule Path | Repository |
|---------|---------------|------------|
| `@claudiv/core` | `packages/core` | [claudiv-ai/core](https://github.com/claudiv-ai/core) |
| `@claudiv/cli` | `packages/cli` | [claudiv-ai/cli](https://github.com/claudiv-ai/cli) |
| `@claudiv/vite-sdk` | `packages/vite-sdk` | [claudiv-ai/vite-sdk](https://github.com/claudiv-ai/vite-sdk) |
| `@claudiv/designer` | `packages/designer` | [claudiv-ai/designer](https://github.com/claudiv-ai/designer) |

This parent repo (`claudiv-ai/claudiv`) provides:
- Unified CI that builds all packages together
- Documentation (`docs/`)
- GitHub governance (issue/PR templates, security policy)
- Turbo + pnpm workspace orchestration for local development

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 20+
- [pnpm](https://pnpm.io/) 8+
- [Git](https://git-scm.com/)

### Clone & Setup

```bash
git clone --recurse-submodules https://github.com/claudiv-ai/claudiv.git
cd claudiv
pnpm install
pnpm build
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

### Build Order

`core` must build first — `cli`, `vite-sdk`, and `designer` depend on it. Turbo handles this automatically via `pnpm build`.

## Development Workflow

### Building

```bash
# Build all packages (respects dependency order via Turbo)
pnpm build

# Build a specific package
cd packages/core && pnpm build
```

### Running Locally

```bash
# Run the CLI in development mode
cd packages/cli && pnpm dev

# Run the designer
cd packages/designer && pnpm dev
```

## Contributing Changes

### Where to Make Changes

Since each package is a separate git repo, your changes need to be committed and pushed to the **individual package repo**, not the parent:

```bash
# Example: make changes to core
cd packages/core
# ... edit files ...
git add src/executor.ts
git commit -m "feat: add timeout configuration"
git push origin main
```

### Updating the Parent Repo

After pushing to a package repo, update the parent repo's submodule pointer:

```bash
cd /path/to/claudiv   # parent repo root
git add packages/core  # stages the new submodule commit
git commit -m "chore: update core submodule"
git push origin main
```

### Pull Request Process

1. Fork the **package repository** you want to change (e.g., `claudiv-ai/core`).
2. Create your branch from `main`.
3. Make your changes and ensure `pnpm build` passes.
4. Submit the PR against `main` on the package repo.
5. Once merged, the parent repo submodule will be updated separately.

For documentation or CI changes, submit PRs directly to this parent repo (`claudiv-ai/claudiv`).

## Version Alignment

All 4 packages must always share the same version number. When releasing:

1. Bump the version in all 4 `package.json` files to the same value.
2. Update cross-dependency references (cli, vite-sdk, designer depend on `@claudiv/core`).
3. Commit and push to each package repo.
4. Tag each package repo: `git tag v<version> && git push origin v<version>`
5. GitHub Actions publishes to npm automatically.
6. Update the parent repo submodules to the new tags.

**Core must publish first** — the other packages depend on it and will fail CI if core isn't available on npm yet.

## Branch Naming

- `feat/description` — New features
- `fix/description` — Bug fixes
- `docs/description` — Documentation changes

## Commit Conventions

This project follows [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add plan question validation
fix: resolve FQN parsing for nested scopes
docs: update architecture overview
refactor: simplify context engine scope resolution
```

## PR Checklist

- [ ] Changes build successfully (`pnpm build`)
- [ ] Commit messages follow conventional commit format
- [ ] Documentation updated if applicable
- [ ] PR is submitted to the correct repo (package repo for code, parent repo for docs/CI)

## Reporting Issues

- **Bugs and feature requests**: [GitHub Issues](https://github.com/claudiv-ai/claudiv/issues)
- **Security vulnerabilities**: See [SECURITY.md](.github/SECURITY.md) — do **not** use public issues

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).
