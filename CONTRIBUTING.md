# Contributing to Claudiv

Thank you for your interest in contributing to Claudiv! This document provides guidelines and information for contributors.

---

## 📋 Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Project Structure](#project-structure)
- [Making Changes](#making-changes)
- [Testing](#testing)
- [Documentation](#documentation)
- [Demo Generation](#demo-generation)
- [Submitting Changes](#submitting-changes)

---

## Code of Conduct

Please be respectful and constructive in all interactions. We aim to maintain a welcoming and inclusive community.

---

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/YOUR_USERNAME/claudiv.git
   cd claudiv
   ```
3. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/claudiv-ai/claudiv.git
   ```

---

## Development Setup

### Prerequisites

- **Node.js** 20.0.0 or higher
- **npm** or **pnpm**
- **Git**

### Installation

```bash
# Install dependencies
npm install

# Build all packages
npm run build

# Link CLI for local testing
cd packages/cli
npm link
```

### Verify Setup

```bash
# Test CLI
claudiv --version

# Run any existing tests
npm test
```

---

## Project Structure

```
claudiv/
├── packages/
│   ├── cli/          # Command-line interface (@claudiv/cli)
│   ├── core/         # Core generation engine (@claudiv/core)
│   └── vite-sdk/     # Vite plugin (future)
├── docs/             # Documentation
├── .demos/           # Demo generation (separate git repo, not committed)
├── examples/         # Example projects
└── README.md
```

### Key Packages

- **@claudiv/cli**: Main CLI tool with file watching, dev server, Claude integration
- **@claudiv/core**: Pure generation engine (framework-agnostic)
- **@claudiv/vite-sdk**: Vite plugin with HMR (planned)

---

## Making Changes

### Branch Naming

- **Feature**: `feat/description`
- **Bug fix**: `fix/description`
- **Documentation**: `docs/description`
- **Chore**: `chore/description`

Example: `feat/add-retry-command`

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

**Examples**:
```bash
feat(cli): add --chat flag for interactive development
fix(core): resolve parsing error with nested elements
docs(readme): update installation instructions
```

---

## Testing

### Manual Testing

```bash
# Test CLI commands
claudiv new test-app
claudiv new test-app --spec paint app with tools
claudiv gen test-app
claudiv dev test-app

# Test new features
claudiv modify test-app --spec make it prettier
claudiv dev test-app --chat
```

### Building

```bash
# Build all packages
npm run build

# Build specific package
cd packages/cli
npm run build
```

---

## Documentation

### Updating Documentation

- **README.md**: Project overview and quick start
- **packages/cli/README.md**: CLI-specific documentation
- **docs/**: Detailed guides and references

### Documentation Standards

- Keep examples concise and working
- Include command outputs where helpful
- Use clear, descriptive language
- Update CHANGELOG.md for version changes

---

## Demo Generation

We maintain a **separate private repository** at `.demos/` for creating promotional GIFs and demo videos.

### Demo Repository

**Location**: `/path/to/claudiv/.demos/` (separate git repo, gitignored)

**Purpose**: Create high-quality demonstration GIFs for documentation and marketing

### Quick Start

```bash
# Navigate to demo repo
cd .demos

# Install tools (first time only)
brew install vhs gifski gifsicle

# Generate terminal demos
npm run gen:terminal

# See .demos/README.md for full documentation
```

### Creating New Demos

1. **Write VHS tape script** in `.demos/scripts/`
2. **Generate GIF**: `vhs scripts/your-demo.tape`
3. **Review output** in `.demos/output/`
4. **Deploy to main repo**: `npm run deploy`

**See `.demos/README.md` for detailed instructions.**

---

## Submitting Changes

### Pull Request Process

1. **Update your fork**:
   ```bash
   git fetch upstream
   git checkout main
   git merge upstream/main
   ```

2. **Create feature branch**:
   ```bash
   git checkout -b feat/your-feature
   ```

3. **Make your changes** and commit:
   ```bash
   git add .
   git commit -m "feat(cli): add new feature"
   ```

4. **Push to your fork**:
   ```bash
   git push origin feat/your-feature
   ```

5. **Open Pull Request** on GitHub

### PR Guidelines

- **Title**: Use conventional commit format
- **Description**: Explain what and why
- **Tests**: Include manual testing steps
- **Documentation**: Update relevant docs
- **Changelog**: Update CHANGELOG.md if applicable

### PR Checklist

- [ ] Code builds successfully
- [ ] Manual testing completed
- [ ] Documentation updated
- [ ] CHANGELOG.md updated (if applicable)
- [ ] Commit messages follow convention
- [ ] No unrelated changes included

---

## Release Process

### Version Bumping

Follow [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward-compatible)
- **PATCH**: Bug fixes

### Publishing (Maintainers Only)

```bash
# Update version in package.json
cd packages/cli
npm version minor  # or patch/major

# Build
npm run build

# Publish to npm
npm publish

# Tag release
git tag -a v0.2.0 -m "Release v0.2.0"
git push origin v0.2.0
```

---

## Code Style

### TypeScript

- Use TypeScript for all new code
- Prefer `interface` over `type` for object shapes
- Use explicit return types for functions
- Avoid `any` where possible

### Formatting

- Use provided ESLint/Prettier config (if available)
- 2 spaces for indentation
- Single quotes for strings
- Trailing commas in multi-line

### Naming

- **Variables/Functions**: camelCase
- **Classes/Interfaces**: PascalCase
- **Constants**: UPPER_SNAKE_CASE
- **Files**: kebab-case.ts

---

## Getting Help

- **Documentation**: Check `docs/` and package READMEs
- **Issues**: Search existing issues on GitHub
- **Discussions**: Start a discussion for questions
- **Discord**: (if available)

---

## Recognition

Contributors will be recognized in:
- GitHub contributors list
- CHANGELOG.md (for significant contributions)
- Release notes

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to Claudiv!** 🎉

Every contribution, big or small, helps make Claudiv better for everyone.
