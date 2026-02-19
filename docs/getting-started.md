# Getting Started

## Prerequisites

- [Node.js](https://nodejs.org/) 20+
- [Claude CLI](https://docs.anthropic.com/en/docs/claude-cli) or an Anthropic API key

## Create a New Project

```bash
npx @claudiv/cli new vite my-app
cd my-app
```

This scaffolds a Vite project with Claudiv integration, including:
- A starter `.cdml` component file
- `@claudiv/vite-sdk` configured in `vite.config.ts`
- A `.claudiv/` directory for context and cache

## Initialize in an Existing Project

```bash
cd your-project
npx @claudiv/cli init
```

The `init` command scans your project structure and generates an initial `.cdml` file and context manifest.

## Development Workflow

### 1. Define a Component

Create or edit a `.cdml` file:

```xml
<component name="user-service" fqn="my-app:user-service">
  <interface>
    <endpoints>
      <create-user description="Create a new user">
        <address route="/api/users" method="post" />
      </create-user>
      <get-user description="Get user by ID">
        <address route="/api/users/:id" method="get" />
      </get-user>
    </endpoints>
  </interface>

  <implementation target="typescript" framework="express">
    <modules>
      <user-controller>HTTP request handlers</user-controller>
      <user-service>Business logic and validation</user-service>
    </modules>
  </implementation>
</component>
```

### 2. Start Dev Mode

```bash
npx @claudiv/cli dev
```

Claudiv watches your `.cdml` files. When you save a change, it:
1. Detects the diff against the cached state
2. Assembles context from `.claudiv/context.cdml`
3. Invokes Claude to generate the corresponding code
4. Commits the changes atomically

### 3. One-Shot Generation

For a single generation pass without watching:

```bash
npx @claudiv/cli gen
```

## Visual Designer

Launch the web-based CDML editor:

```bash
npx @claudiv/cli designer --open
```

The designer provides a visual component tree, property editing, and real-time bidirectional sync with `.cdml` files on disk.

## Next Steps

- [Architecture](architecture.md) — How Claudiv works under the hood
- [CDML Reference](cdml-reference.md) — Full component syntax
- [FQN Reference](fqn-reference.md) — Fully Qualified Name grammar
