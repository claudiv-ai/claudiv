# Claudiv

Declarative AI interaction platform for building software through structured component specifications.

[![npm version](https://img.shields.io/npm/v/@claudiv/core?label=%40claudiv%2Fcore)](https://www.npmjs.com/package/@claudiv/core)
[![npm version](https://img.shields.io/npm/v/@claudiv/cli?label=%40claudiv%2Fcli)](https://www.npmjs.com/package/@claudiv/cli)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![CI](https://github.com/claudiv-ai/claudiv/actions/workflows/ci.yml/badge.svg)](https://github.com/claudiv-ai/claudiv/actions/workflows/ci.yml)

## What is Claudiv?

Claudiv is a **declarative AI interaction platform** that lets you describe software systems as structured `.cdml` component files. When you change a component specification, Claudiv detects the diff, assembles the right context, and invokes Claude to generate or update the corresponding code — all through headless, transactional processing.

## Key Features

- **Interface-first components** — Four-section model: interface, constraints, requires, implementation. Dependencies see only the public interface.
- **Diff-based change detection** — No manual triggers. Edit a `.cdml` file and Claudiv detects exactly what changed.
- **Transactional processing** — Every change is atomic: diff → context → execute → commit/rollback. No partial state.
- **Fully Qualified Names (FQN)** — Cross-file component references with scope resolution and fragment addressing.
- **Context engine** — Structured context manifests map scopes to code artifacts, interface contracts, and architectural facts.
- **Plan directives** — One-level-deep expansion proposals with interactive question workflows.
- **Visual designer** — Web-based CDML editor with real-time bidirectional file sync.

## Packages

| Package | Version | Description |
|---------|---------|-------------|
| [`@claudiv/core`](https://www.npmjs.com/package/@claudiv/core) | [![npm](https://img.shields.io/npm/v/@claudiv/core)](https://www.npmjs.com/package/@claudiv/core) | Core engine: types, parser, differ, context engine, executor |
| [`@claudiv/cli`](https://www.npmjs.com/package/@claudiv/cli) | [![npm](https://img.shields.io/npm/v/@claudiv/cli)](https://www.npmjs.com/package/@claudiv/cli) | CLI tool: `new`, `dev`, `gen`, `init` commands |
| [`@claudiv/vite-sdk`](https://www.npmjs.com/package/@claudiv/vite-sdk) | [![npm](https://img.shields.io/npm/v/@claudiv/vite-sdk)](https://www.npmjs.com/package/@claudiv/vite-sdk) | Vite integration: framework detection, dev/gen runners |
| [`@claudiv/designer`](https://www.npmjs.com/package/@claudiv/designer) | [![npm](https://img.shields.io/npm/v/@claudiv/designer)](https://www.npmjs.com/package/@claudiv/designer) | Visual CDML editor: React 19 + Express 5 + WebSocket |

## Quick Start

```bash
# Create a new Vite project with Claudiv
npx @claudiv/cli new vite my-app

# Or initialize Claudiv in an existing project
cd existing-project
npx @claudiv/cli init
```

Edit a `.cdml` file to describe your component, then run:

```bash
npx @claudiv/cli dev
```

Claudiv watches for changes, detects diffs, and generates code automatically.

## How It Works

```
.cdml file edited
       │
       ▼
  ┌─────────┐     ┌──────────────┐     ┌───────────────┐     ┌────────┐
  │  Differ  │────►│ Context      │────►│ Headless      │────►│ Commit │
  │          │     │ Engine       │     │ Claude        │     │        │
  └─────────┘     └──────────────┘     └───────────────┘     └────────┘
  detect what      assemble prompt      generate/update       write files
  changed          with context         code                  atomically
```

1. **Diff** — The differ compares the current `.cdml` against the cached state to identify exactly what changed.
2. **Context** — The context engine reads `.claudiv/context.cdml` to gather relevant code files, interface contracts, and architectural facts for the changed scope.
3. **Execute** — Claude receives a structured prompt with the diff, context, and interface projections, then generates the appropriate code changes.
4. **Commit** — Changes are written atomically. If anything fails, the transaction rolls back.

## Documentation

- [Getting Started](docs/getting-started.md) — Quick start guide
- [Architecture](docs/architecture.md) — System architecture overview
- [CDML Reference](docs/cdml-reference.md) — Component syntax and examples
- [FQN Reference](docs/fqn-reference.md) — Fully Qualified Name grammar

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and guidelines.

## Community

- [GitHub Issues](https://github.com/claudiv-ai/claudiv/issues) — Bug reports and feature requests
- [GitHub Discussions](https://github.com/claudiv-ai/claudiv/discussions) — Questions and ideas

## License

[MIT](LICENSE)
