# Contributing to Claudiv

Thank you for your interest in contributing to Claudiv. This guide covers the development setup and contribution process.

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

## Package Structure

| Package | Path | Description |
|---------|------|-------------|
| `@claudiv/core` | `packages/core` | Types, parser, differ, context engine, executor, FQN resolver |
| `@claudiv/cli` | `packages/cli` | CLI commands: `new vite`, `new system`, `dev`, `gen`, `init` |
| `@claudiv/vite-sdk` | `packages/vite-sdk` | Vite integration: framework detection, dev/gen runners |
| `@claudiv/designer` | `packages/designer` | Visual CDML editor: React + Express + WebSocket |

**Build order**: `core` must build first — `cli`, `vite-sdk`, and `designer` depend on it.

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

## Pull Request Process

1. Fork the repository and create your branch from `main`.
2. Make your changes in the appropriate package submodule.
3. Ensure `pnpm build` passes across all packages.
4. Write a clear PR description explaining what changed and why.
5. Submit the PR against `main`.

### PR Checklist

- [ ] Changes build successfully (`pnpm build`)
- [ ] Commit messages follow conventional commit format
- [ ] Documentation updated if applicable

## Reporting Issues

- **Bugs and feature requests**: [GitHub Issues](https://github.com/claudiv-ai/claudiv/issues)
- **Security vulnerabilities**: See [SECURITY.md](.github/SECURITY.md) — do **not** use public issues

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).
