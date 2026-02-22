# Architecture

## Overview

Claudiv is a declarative AI interaction platform. You describe software systems as structured `.cdml` component files; Claudiv detects changes, assembles context, and invokes Claude to generate or update code.

```
.cdml files ──► Differ ──► Context Engine ──► Executor ──► File System
                  │              │                │
                  ▼              ▼                ▼
              cached state   context.cdml    Claude Agent SDK
```

## Core Processing Pipeline

### 1. Diff Detection

The **differ** compares the current `.cdml` file against its cached state to produce a structured diff:

- **Added** elements — new components, sections, or attributes
- **Removed** elements — deleted content
- **Changed** elements — modified attributes or text content
- **Moved** elements — reordered within parent

No manual triggers or special attributes. Just save the file.

### 2. Context Assembly

The **context engine** reads `.claudiv/context.cdml` — a manifest that maps scopes to:

- **Code references** — existing source files relevant to the scope
- **Interface contracts** — projected interfaces from dependencies
- **Architectural facts** — decisions, constraints, patterns

The engine assembles a structured prompt containing only the information relevant to the changed scope. Dependencies see only the public interface (view-filtered projection), never implementation details.

### 3. Headless Execution

The **executor** invokes Claude with the assembled prompt using the Claude Agent SDK (SDK mode) or the direct Anthropic API. In SDK mode, Claude can explore the codebase with read-only tools and use multi-turn thinking with sub-agents. If user input is needed, it outputs `<plan:questions>` blocks rather than free-text questions.

### 4. Transactional Commit

Changes are written atomically:
- If all generated files are valid, they are committed to disk and the cache is updated.
- If anything fails, the entire transaction rolls back — no partial state.

## Component Model

Every `.cdml` component has four sections:

| Section | Purpose | Visibility |
|---------|---------|------------|
| `<interface>` | What other components see | Public — projected to dependents |
| `<constraints>` | Environment requirements | Public — used for validation |
| `<requires>` | Dependencies by interface | Public — drives context assembly |
| `<implementation>` | Internal structure | Private — own scope only |

## FQN Addressing

Components are addressed by Fully Qualified Names:

```
project:scope:path:component#fragment@version
```

Fragments target specific sections: `#api` (interface), `#impl` (implementation), `#infra`, `#data`, etc.

See [FQN Reference](fqn-reference.md) for the full grammar.

## Context Manifest

`.claudiv/context.cdml` is the structural bridge between your component specs and your codebase:

```xml
<claudiv-context for="user-service.cdml" auto-generated="true">
  <global>
    <refs><code file="package.json" role="project-config" /></refs>
    <facts><fact>Uses TypeScript strict mode</fact></facts>
  </global>
  <scope path="user-service > implementation > user-controller">
    <interfaces>
      <fulfills fqn="user-service#api:create-user" />
    </interfaces>
    <refs><code file="src/controllers/user.ts" role="implementation" /></refs>
  </scope>
</claudiv-context>
```

## Plan Directives

Plans enable one-level-deep expansion proposals. Add a `plan` attribute or element to request Claude to propose structure rather than generate code:

```xml
<cloud plan="private cloud infrastructure">
  <existing-child>Immutable during planning</existing-child>
</cloud>
```

Claude proposes new elements. If clarification is needed, it generates `<plan:questions>` for interactive resolution.

## Package Architecture

```
@claudiv/core          @claudiv/cli
  types, parser,         new, dev, gen, init
  differ, context        commands
  engine, executor
       │                      │
       └──────────┬───────────┘
                  │
           @claudiv/vite-sdk        @claudiv/designer
             framework detection      visual CDML editor
             dev/gen runners          React + Express + WS
```

`@claudiv/core` provides the engine; `@claudiv/cli` orchestrates commands; `@claudiv/vite-sdk` integrates with Vite projects; `@claudiv/designer` provides visual editing.
