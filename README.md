# Claudiv

### *Claude in a Div*

> **Universal Declarative Generation Platform** — From Anything to Anything, Through Natural Language.

**Claudiv** transforms natural language specifications (`.cdml` files) into working code, documents, configurations, and more — in any language, any framework, any format. Describe what you want. Claudiv builds it.

## How It Works

Write a `.cdml` file with freeform XML-ish tags. Add `gen` to trigger AI generation.

```xml
<!-- app.cdml -->
<app target="html">
  <landing-page gen>
    Modern SaaS landing page with hero section, features grid,
    pricing table, and contact form
  </landing-page>
</app>
```

```bash
claudiv app.cdml   # → generates app.html
```

That's it. No boilerplate. No frameworks to learn. Just describe and generate.

---

## Core Syntax

### The `gen` Attribute

Add `gen` to any element to trigger generation. It supports three forms:

```xml
<!-- Boolean: generate based on element content and attributes -->
<button gen>Make a blue rounded button with hover effect</button>

<!-- Empty string: generate based on attributes only -->
<form payment style="same as create user form" gen="" />

<!-- With instructions: pass generation directives -->
<dashboard gen="use opus 4.6">
  Admin analytics dashboard with charts
</dashboard>
```

### Attribute-Based Specifications

Element names and attributes ARE the specification. Use any names you want:

```xml
<navigation-bar
  position="sticky top"
  theme="dark"
  logo="company-logo.svg"
  gen>
  <menu-items>
    Home, Products, Pricing, About, Contact
  </menu-items>
  <auth-buttons>Login and Sign Up</auth-buttons>
</navigation-bar>
```

### Action Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `gen` | Generate new code | `<button gen>Blue button</button>` |
| `gen="..."` | Generate with instructions | `<api gen="use opus 4.6">REST API</api>` |
| `retry` | Regenerate (not satisfied) | `<button retry>Make it bigger</button>` |
| `undo` | Revert to previous version | `<button undo />` |
| `lock` | Protect from regeneration | `<header lock>...</header>` |
| `unlock` | Allow regeneration of locked | `<footer unlock>...</footer>` |

### Target Languages

Specify output language with the `target` attribute:

```xml
<!-- Web -->
<app target="html" gen>Landing page</app>
<app target="react" gen>Dashboard component</app>
<app target="vue" gen>Todo app</app>
<app target="svelte" gen>Blog layout</app>

<!-- Backend -->
<api target="python" framework="fastapi" gen>
  User authentication REST API with JWT
</api>
<server target="typescript" framework="express" gen>
  WebSocket chat server
</server>

<!-- Scripts -->
<backup target="bash" gen>
  Backup PostgreSQL to S3, compress with gzip,
  delete backups older than 30 days
</backup>

<!-- Systems -->
<infra target="kubernetes" gen>
  3-replica Node.js deployment with Redis sidecar
</infra>
<pipeline target="cicd" gen>
  GitHub Actions: test, build, deploy to AWS
</pipeline>
```

### Nested Components

Nested elements provide hierarchical context. Claude sees the full tree:

```xml
<e-commerce target="react" gen>
  <product-catalog>
    Grid layout with filtering and search
    <product-card>
      Image, title, price, "Add to Cart" button
    </product-card>
  </product-catalog>

  <shopping-cart lock>
    <!-- Already generated and locked — won't regenerate -->
    <cart-item>Item with quantity selector</cart-item>
    <cart-total>Subtotal, tax, shipping, total</cart-total>
  </shopping-cart>

  <checkout-flow>
    Multi-step: Shipping → Payment → Confirmation
  </checkout-flow>
</e-commerce>
```

### Reference Existing Work

Use `style` or any attribute to reference other components:

```xml
<signup-form gen>
  User registration with email, password, name
</signup-form>

<!-- Reference existing component's style -->
<payment-form style="same as signup-form" gen="" />
```

---

## File Naming Convention

```
<name>.cdml  →  <name>.<ext>
```

| Input | Target | Output |
|-------|--------|--------|
| `app.cdml` | html | `app.html` |
| `api.cdml` | python | `api.py` |
| `backup.cdml` | bash | `backup.sh` |
| `deploy.cdml` | kubernetes | `deploy.yaml` |
| `Button.cdml` | react | Updates `Button.tsx` directly |

---

## Installation

**From GitHub Packages:**

```bash
# Configure npm to use GitHub Packages for @amirguterman scope
echo "@amirguterman:registry=https://npm.pkg.github.com" >> ~/.npmrc

# Install globally
npm install -g @amirguterman/claudiv

# Or run directly with npx (no install needed)
npx @amirguterman/claudiv
```

> **Note:** GitHub Packages requires authentication. Create a [personal access token](https://github.com/settings/tokens) with `read:packages` scope, then:
> ```bash
> npm login --registry=https://npm.pkg.github.com --scope=@amirguterman
> ```

**Requirements:**
- Node.js 20+
- Claude Code CLI (for CLI mode) or Anthropic API key (for API mode)

## CLI

### Quick Start (Empty Folder)

```bash
npx @amirguterman/claudiv new myapp
npx @amirguterman/claudiv gen myapp
```

### Commands

```
claudiv new <name>       Create a new .cdml file
claudiv gen <name>       Generate code from .cdml
claudiv reverse <file>   Reverse-engineer file → .cdml
claudiv watch <name>     Watch and regenerate on changes
claudiv help             Show full help
```

### Options

```
-s, --spec <xml>      Inline .cdml content (for 'new')
-g, --gen             Generate immediately after 'new'
-t, --target <name>   Target component or language
-f, --framework <fw>  Framework (fastapi, express, nextjs, etc.)
-o, --output <file>   Output file path
-w, --watch           Watch mode
--dry-run             Preview without writing files
```

### Examples

**Create a new project:**

```bash
claudiv new myapp                              # → myapp.cdml
claudiv new myapp -t python                    # → Python project template
claudiv new myapp -t python -f fastapi         # → FastAPI project template
claudiv new myapp -g                           # → Create + generate immediately
```

**Create with inline spec:**

```bash
claudiv new txt2img -s '<txt2img lang="python" type="cli" gen="">
  <ai provider="openai" />
  <config apikey organizationid />
  <args input="text|file, size" output="filename, format" />
</txt2img>'
```

**Create with inline spec + generate immediately:**

```bash
claudiv new txt2img -s '<txt2img lang="python" type="cli" gen="">
  <ai provider="openai" />
  <config apikey organizationid />
  <args input="text|file, size" output="filename, format" />
</txt2img>' -g
```

This generates a complete Python CLI project for text-to-image with OpenAI integration, config management, and argument parsing.

**Generate from .cdml:**

```bash
claudiv gen myapp                              # Generate all from myapp.cdml
claudiv gen myapp.cdml                         # Explicit path
claudiv gen myapp -w                           # Watch mode
claudiv gen myapp -o output.py                 # Custom output file
```

**Generate specific component (-t):**

```bash
claudiv gen txt2img -t config                  # Generates txt2img.config
claudiv gen myapp -t api                       # Generates myapp.api
claudiv gen myapp -t database                  # Generates myapp.database
```

**Reverse engineering:**

```bash
claudiv reverse api.py                         # → api.cdml
claudiv reverse Button.tsx                     # → Button.cdml
claudiv reverse backup.sh                      # → backup.cdml
claudiv reverse styles.css                     # → styles.cdml
claudiv reverse api.py -o spec.cdml            # Custom output name
```

**Watch mode:**

```bash
claudiv watch myapp                            # Watch myapp.cdml
claudiv gen myapp -w                           # Same as above
```

### Full Example: Text-to-Image CLI

```bash
# 1. Create project in empty folder
mkdir txt2img && cd txt2img
npx @amirguterman/claudiv new txt2img -s '<txt2img lang="python" type="cli" gen="">
  <ai provider="openai" />
  <config apikey organizationid />
  <args input="text|file, size" output="filename, format" />
</txt2img>' -g

# 2. Claudiv generates a Python project:
#    txt2img.py        - Main CLI entry point
#    config.py         - API key and org configuration
#    requirements.txt  - Dependencies

# 3. Generate just the config:
claudiv gen txt2img -t config

# 4. Watch for changes:
claudiv watch txt2img
```

---

## Reverse Generation

Generate `.cdml` specs FROM existing code:

```bash
# Code → .cdml
claudiv reverse api.py          → api.cdml
claudiv reverse Button.tsx      → Button.cdml
claudiv reverse backup.sh       → backup.cdml
claudiv reverse styles.css      → styles.cdml
```

This enables adopting Claudiv in existing projects — reverse-engineer your codebase, then continue development purely through `.cdml`.

---

## Scoped Boundaries

Use `<claudiv>` tags to mark Claudiv-managed regions in existing files:

```html
<!-- existing-page.html -->
<body>
  <header>Existing header (not managed by Claudiv)</header>

  <claudiv>
    <!-- Only this region is managed by Claudiv -->
    <dashboard gen>Analytics dashboard</dashboard>
  </claudiv>

  <footer>Existing footer</footer>
</body>
```

---

## Integration Modes

### 1. Standalone (CLI)
```bash
claudiv app.cdml  # Generates app.html with Vite dev server
```

### 2. Existing Project
```bash
# Install in existing React app
npm install @claudiv/core

# Generate .cdml from existing component
claudiv reverse src/Dashboard.tsx → Dashboard.cdml

# Edit Dashboard.cdml, then apply changes
claudiv Dashboard.cdml  # Updates Dashboard.tsx directly
```

### 3. Script Tag (Future)
```html
<script src="https://cdn.claudiv.dev/runtime.js"></script>
<claudiv src="app.cdml"></claudiv>
```

---

## Future Development

The following features are planned but not yet fully implemented:

### Backend & API Generation

```xml
<!-- Generate complete REST API -->
<api target="python" framework="fastapi" gen>
  <endpoint path="/users" method="GET">
    Paginated list of users with filtering
  </endpoint>
  <endpoint path="/users" method="POST">
    Create new user with validation
  </endpoint>
  <database target="database-schema" dialect="postgresql">
    Users table with auth fields
  </database>
  <websocket path="/notifications">
    Real-time notification stream
  </websocket>
</api>
```

### System State Capture

```xml
<!-- Capture system state as .cdml -->
<system-snapshot target="system-state" gen>
  Startup programs, services, environment variables,
  system configurations, file mappings
</system-snapshot>
```

```bash
claudiv capture system.cdml  # → Full system state snapshot
```

**Generated:**

```xml
<system-state captured="2026-02-14T01:00:00Z">
  <services>
    <service name="nginx" status="running" enabled="true" />
    <service name="postgresql" status="running" enabled="true" />
  </services>
  <environment>
    <var name="NODE_ENV">production</var>
    <var name="PATH">/usr/local/bin:/usr/bin</var>
  </environment>
  <file-mappings>
    <mapping type="symlink">
      <source>/var/www/app</source>
      <target>/home/deploy/app</target>
    </mapping>
  </file-mappings>
</system-state>
```

### Live Monitoring with Hooks

```xml
<monitor target="live-monitoring" gen>
  <paths>/var/log, /etc, /home/user/projects</paths>
  <report-format>markdown</report-format>
  Track all file operations and generate structured report
</monitor>
```

```bash
claudiv monitor app.cdml         # Start monitoring
# ... activity happens ...
# Ctrl+C → generates monitoring-report.md
```

### Log Aggregation

```xml
<log-report target="log-report" gen>
  <sources>
    /var/log/nginx/*.log
    /var/log/syslog
    ~/app/logs/*.log
  </sources>
  <filters>
    <level>error, warning</level>
    <time-range>last 24 hours</time-range>
  </filters>
  Generate structured report grouped by severity
</log-report>
```

### Document & Media Formats

```xml
<!-- Generate PDF report -->
<report target="pdf" gen>
  Q1 Performance Report with charts, tables, executive summary
</report>

<!-- Generate Excel analytics -->
<analytics target="excel" gen>
  User metrics dashboard with pivot tables and charts
</analytics>

<!-- Generate PowerPoint deck -->
<deck target="powerpoint" theme="modern" gen>
  AI-Powered Development — 10-slide presentation
</deck>

<!-- Reverse: extract from existing documents -->
claudiv reverse presentation.pptx  → presentation.cdml
claudiv reverse report.pdf         → report.cdml
claudiv reverse data.xlsx          → data.cdml
```

### Image OCR & Audio Transcription

```xml
<!-- Extract structure from architecture diagram -->
<diagram target="image-ocr" gen>
  <source>architecture.png</source>
  Extract system components and connections
</diagram>

<!-- Transcribe meeting recording -->
<meeting target="audio-transcription" gen>
  <source>standup.mp3</source>
  Identify speakers, extract action items and decisions
</meeting>
```

### Conversation Management & Summarization

```xml
<!-- Capture Slack conversations -->
<conversation-capture mcp-server="slack" gen>
  <channel>#engineering</channel>
  <date-range>last 2 weeks</date-range>
  Structure conversations with decisions and action items
</conversation-capture>

<!-- Generate daily digest -->
<daily-digest target="digest" gen>
  <sources>
    <slack channel="#engineering" />
    <github repo="user/myapp" />
    <jira project="MYAPP" />
  </sources>
  Aggregate activity from all sources
</daily-digest>

<!-- Summarize meeting notes -->
<summary target="meeting-notes" gen>
  <source>meeting-transcript.cdml</source>
  Executive summary with decisions and action items
</summary>
```

### External Integrations via MCP

```xml
<claudiv-config>
  <mcp-servers>
    <server name="desktop" url="http://localhost:3100/mcp" />
    <server name="atlassian" url="http://localhost:3101/mcp" />
    <server name="slack" url="http://localhost:3102/mcp" />
    <server name="github" url="http://localhost:3104/mcp" />
  </mcp-servers>
</claudiv-config>
```

#### Desktop GUI Automation

```xml
<desktop-automation mcp-server="desktop" gen>
  1. Launch Chrome
  2. Navigate to https://app.example.com
  3. Take screenshot
  4. Click "Sign Up"
  5. Fill form fields
  6. Take screenshot
  Generate user guide with annotated screenshots
</desktop-automation>
```

#### Jira Integration

```xml
<jira-issues mcp-server="atlassian" gen>
  <project>MYAPP</project>
  <issue type="bug" priority="high">
    <summary>Cart update fails on production</summary>
    <description>Steps to reproduce, expected vs actual behavior</description>
  </issue>
</jira-issues>
```

#### Slack Notifications

```xml
<slack-notification mcp-server="slack" gen>
  <channel>#deployments</channel>
  <message>
    Deployment complete — v2.3.0 to production
    All tests passing (142/142)
  </message>
</slack-notification>
```

#### Remote Git Operations

```xml
<remote-git mcp-server="github" gen>
  <repository>https://github.com/user/myapp</repository>
  Clone and generate .cdml representation of entire codebase
</remote-git>
```

### QA Automation

```xml
<qa-automation target="test-suite" framework="playwright" gen>
  <test-scenario name="User Login">
    1. Navigate to /login
    2. Enter credentials
    3. Click Login
    4. Verify dashboard loads
  </test-scenario>
</qa-automation>
```

### End-to-End Workflow Automation

```xml
<workflow target="automation" gen>
  Bug report (Slack) → Jira issue → Reproduce (desktop MCP) →
  Debug trace → AI fix → Tests → PR → Slack notification
</workflow>
```

### Multi-File Support

```xml
<!-- Reference another .cdml file -->
<component ref="shared/button.cdml" />

<!-- Embed content from another file -->
<styles embed="design-system.cdml" />
```

### Rules Integration

```xml
<!-- Persistent rules synced with .claude/rules -->
<rules>
  Always use Tailwind CSS for styling.
  Use TypeScript strict mode.
  Follow REST API naming conventions.
</rules>
```

### Bash Execution

```xml
<!-- Execute bash commands during generation -->
<bash>npm install tailwindcss</bash>
```

### Chat Mode

```xml
<!-- Discussion without code generation -->
<chat>
  What's the best approach for real-time notifications?
  Should we use WebSockets or Server-Sent Events?
</chat>
```

### Meta-Configuration

`.cdml` files can configure how Claudiv itself operates:

```xml
<claudiv-config>
  <model>opus</model>
  <use-agents>true</use-agents>
  <permissions>
    <bash>git, npm, docker</bash>
    <filesystem>read-write</filesystem>
    <network>true</network>
  </permissions>
  <tools>Bash, WebFetch, Grep, Read</tools>
</claudiv-config>
```

---

## Architecture

```
┌───────────────────────────────────────────┐
│              .cdml Files                  │
│   (Natural Language Specifications)       │
└───────────────────┬───────────────────────┘
                    ↓
┌───────────────────────────────────────────┐
│            Claudiv Parser                 │
│  (Lenient XML, Hierarchy, Attributes)     │
└───────────────────┬───────────────────────┘
                    ↓
┌───────────────────────────────────────────┐
│           Claude AI Engine                │
│   (Opus / Sonnet / Haiku + MCP)           │
└───────────────────┬───────────────────────┘
                    ↓
┌───────────────────────────────────────────┐
│         Generator Registry                │
│  (100+ target types, pluggable)           │
└───────────────────┬───────────────────────┘
                    ↓
┌───────────────────────────────────────────┐
│              Outputs                      │
│  Code · Docs · Configs · Media · Actions  │
└───────────────────────────────────────────┘
```

## Monorepo Structure

```
claudiv/
├── packages/
│   └── core/          # @claudiv/core npm package
│       ├── src/
│       │   ├── index.ts           # Main orchestrator
│       │   ├── parser.ts          # .cdml parser
│       │   ├── config.ts          # Configuration
│       │   ├── watcher.ts         # File watcher
│       │   ├── code-generator.ts  # Universal generator
│       │   ├── generators/        # Language-specific generators
│       │   │   ├── registry.ts
│       │   │   ├── html-generator.ts
│       │   │   ├── bash-generator.ts
│       │   │   ├── python-generator.ts
│       │   │   └── system-monitor-generator.ts
│       │   └── types.ts           # Type definitions
│       ├── bin/claudiv.js         # CLI entry point
│       └── claudiv.xsd            # XML Schema for IDE support
├── apps/
│   └── examples/      # Example .cdml files
├── docs/              # Documentation
├── turbo.json         # Turborepo config
└── pnpm-workspace.yaml
```

## IDE Support

Claudiv provides XML Schema (`.xsd`) for autocompletion in any XML-aware editor.

### VS Code

Install the [XML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-xml). Claudiv's `.vscode/settings.json` auto-associates `.cdml` files.

---

## Philosophy

> **"Describe what you want, not how to build it."**

Traditional development: write code → get application.
Claudiv development: write intent → get everything.

Not just code — get working applications, tests, documentation, deployment configs, monitoring, and integrations. All from natural language specifications.

## License

MIT

## Author

Amir Guterman — [GitHub](https://github.com/amirguterman)
