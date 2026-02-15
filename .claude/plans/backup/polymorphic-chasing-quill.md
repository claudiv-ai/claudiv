# Claudiv Monorepo Restructuring & GitHub Publishing - Implementation Plan

## Context

**Claudiv** (*Claude in a Div*) is a revolutionary development framework that transforms XML-like specifications into working web applications through natural conversations with Claude AI. The core npm package is working and ready for publication.

**Current State:**
- Single package structure in `/home/source/gui-driven-spec/`
- No git repository initialized yet
- Contains generated files (spec.code.html, dist/) that shouldn't be committed
- Working attribute-based syntax (gen, retry, undo, lock, unlock)
- Dual-mode support (Claude Code CLI + Anthropic API)
- Hot reload development server functional

**Goal:**
Transform this into a **professional monorepo** containing multiple projects (npm package, VSCode extension, Chrome extension, examples) with automated GitHub publishing, proper versioning, and clean separation of source vs generated files.

**Expanded Vision:**
Claudiv is not just a UI framework - it's a **universal declarative development platform** that can:
- Generate frontend UIs, backend APIs, databases, and full-stack apps
- Integrate into existing HTML pages via script tag
- Generate presentations, system designs, CI/CD pipelines, and automation scripts
- Provide AI capabilities to end-user applications
- Support chat-based planning without code generation
- Connect to MCP servers, WebSockets, webhooks, and external packages

## Research Summary

### Monorepo Best Practices (2026)
Based on [recent industry standards](https://medium.com/@jamesmiller22871/stop-fighting-node-modules-a-modern-guide-to-managing-monorepos-in-2026-16cbc79e190d):
- **pnpm workspaces** + **Turborepo** is the standard stack
- Directory structure: `apps/` for applications, `packages/` for libraries
- Use namespace prefix (`@claudiv/`) to avoid npm registry conflicts
- Turborepo provides remote caching and parallel builds
- Syncpack enforces consistent dependency versions

### npm Publishing from GitHub
Based on [npm trusted publishing documentation](https://docs.npmjs.com/trusted-publishers/):
- **Trusted Publishing** via OpenID Connect (OIDC) is the secure standard
- GitHub Actions with `id-token: write` permission
- Automatic provenance statements establish build authenticity
- No long-lived access tokens required

### VSCode Extension Development
Based on [VS Code extension guide](https://code.visualstudio.com/api/extension-guides/webview):
- TypeScript-based development
- Webview API for visual designer and live preview
- Language Server Protocol for custom file type support
- Can share code with core npm package via monorepo

### Chrome Extension Development
Based on [Chrome Manifest V3 documentation](https://developer.chrome.com/docs/extensions/reference/api/sidePanel):
- Manifest V3 is required for new extensions
- Side Panel API for in-browser editing interface
- Content scripts for live page manipulation
- Can share code with core package

### File Extension Recommendation
Based on [domain-specific language conventions](https://docs.oasis-open.org/dita/dita/v1.3/errata02/os/complete/part2-tech-content/non-normative/file-naming-conventions.html):
- `.html` currently used but may confuse users (they expect strict HTML)
- Custom extension `.claudiv` would be clearer and brandable
- Recommendation: **Primary: `.claudiv`, Fallback: `.html` for backward compatibility**
- Benefits of `.claudiv`:
  - Clear association with the framework
  - VSCode can register custom language support
  - No confusion with regular HTML files
  - Better for marketing/branding

## Proposed Monorepo Structure

```
claudiv/                              # Root repository
├── .github/
│   └── workflows/
│       ├── publish-npm.yml           # Auto-publish to npm
│       ├── publish-vscode.yml        # Publish VSCode extension
│       ├── publish-chrome.yml        # Publish Chrome extension
│       └── ci.yml                    # Tests, linting, type-check
├── packages/
│   ├── claudiv/                      # Core npm package (@claudiv/core)
│   │   ├── src/                      # TypeScript source (current src/)
│   │   ├── bin/                      # CLI entry point
│   │   ├── package.json              # Scoped: @claudiv/core
│   │   ├── tsconfig.json
│   │   ├── README.md
│   │   └── .gitignore                # Ignore dist/, *.claudiv.code.html
│   ├── vscode/                       # VSCode extension (@claudiv/vscode)
│   │   ├── src/
│   │   │   ├── extension.ts          # Extension entry point
│   │   │   ├── language-server.ts    # .claudiv language support
│   │   │   ├── preview-provider.ts   # Webview for live preview
│   │   │   └── designer.ts           # Visual designer webview
│   │   ├── package.json              # VSCode extension manifest
│   │   ├── syntaxes/
│   │   │   └── claudiv.tmLanguage.json # Syntax highlighting
│   │   └── README.md
│   └── chrome/                       # Chrome extension (@claudiv/chrome)
│       ├── src/
│       │   ├── background.ts         # Service worker
│       │   ├── sidepanel.ts          # Side panel UI
│       │   └── content.ts            # Content script for live editing
│       ├── manifest.json             # Manifest V3
│       ├── package.json
│       └── README.md
├── apps/
│   └── examples/                     # Example projects (not published to npm)
│       ├── basic-website/
│       │   ├── app.claudiv
│       │   └── README.md
│       ├── dashboard/
│       │   ├── app.claudiv
│       │   └── README.md
│       └── ecommerce/
│           ├── app.claudiv
│           └── README.md
├── docs/                             # Shared documentation
│   ├── ATTRIBUTE-SYNTAX.md
│   ├── FEATURES-SUMMARY.md
│   ├── LOCK-UNLOCK-GUIDE.md
│   └── SCHEMA-GUIDE.md
├── pnpm-workspace.yaml               # pnpm workspace config
├── turbo.json                        # Turborepo config
├── package.json                      # Root package.json (workspace)
├── .gitignore                        # Root gitignore
├── README.md                         # Main project README
└── LICENSE                           # MIT License

Generated files (NOT committed):
├── packages/claudiv/dist/            # Built JS (gitignored)
├── **/*.claudiv.code.html            # Generated output (gitignored)
├── **/*.claudiv.spec.html            # Generated artifacts (gitignored)
└── node_modules/                     # Dependencies (gitignored)
```

## File Format Strategy

**Decision: NO forced file extension - work with ANY file**

**Supported Files:**
- `.html` - Standard HTML files ✅
- `.xml` - XML files ✅
- `.php`, `.jsp`, `.erb` - Server templates ✅
- `.md` - Markdown with embedded claudiv ✅
- No extension - Any text file ✅

**Why No Forced Extension:**
1. **Flexibility**: Works with existing workflows
2. **Integration**: Add claudiv to ANY project without restructuring
3. **No Migration**: Use existing HTML files directly
4. **Universal**: `.claudiv` optional, not required
5. **Choice**: Users decide their file structure

**Generated File Naming:**
- Input: `index.html`
- Output: `index.html` (in-place updates) OR `index.generated.html` (separate file)

- Input: `app.xml`
- Output: `app.generated.html`

**User Choice:**
```bash
# In-place mode (updates source file)
claudiv gen index.html --in-place

# Separate output (preserves source)
claudiv gen index.html --output index.generated.html

# Auto-detect from config
claudiv gen index.html
```

**Configuration** (.claudiv.config.json):
```json
{
  "output": {
    "mode": "separate",  // "in-place" or "separate"
    "pattern": "${name}.generated.${ext}"
  }
}
```

## Git Commit Strategy

**Goal**: Create clean, logical git history split into semantic commits

**Commit Sequence:**
1. `chore: initialize repository with gitignore and basic config`
   - .gitignore, .editorconfig, LICENSE

2. `feat: core claudiv engine`
   - src/ directory (parser, watcher, claude clients, etc.)
   - tsconfig.json

3. `feat: CLI and development server`
   - bin/claudiv.js
   - dev-server.ts

4. `feat: attribute-based syntax (gen, retry, undo, lock, unlock)`
   - Implementation of all action attributes

5. `feat: dual-mode support (CLI + API)`
   - Claude Code CLI and Anthropic API integration

6. `docs: add comprehensive documentation`
   - README.md, FEATURES-SUMMARY.md, ATTRIBUTE-SYNTAX.md, etc.

7. `feat: XML Schema for IDE autocomplete`
   - spec.xsd (to be renamed claudiv.xsd)

8. `build: add build configuration`
   - package.json, package-lock.json

9. `chore: restructure as monorepo`
   - Move to packages/claudiv/
   - Add pnpm-workspace.yaml
   - Add turbo.json

## Critical Files That Should NOT Be Committed

**Generated Files (must be in .gitignore):**
- `dist/` - TypeScript build output
- `*.claudiv.code.html` - Generated HTML/CSS
- `*.claudiv.spec.html` - Generated specifications
- `*.claudiv.rules.html` - Generated rules
- `*.claudiv.models.html` - Generated models
- `*.claudiv.*.html` - Any other generated artifacts
- `node_modules/` - Dependencies
- `.env` - Environment variables (secrets)

**Development Files (should be ignored):**
- `.vscode/` - Personal IDE settings (except shared configs)
- `.idea/` - JetBrains IDE
- `.DS_Store` - macOS
- `*.log` - Log files

**What SHOULD Be Committed:**
- `src/` - All TypeScript source code
- `bin/` - CLI executable
- `docs/` - Documentation
- `README.md`, `LICENSE`, etc.
- `package.json`, `tsconfig.json` - Configuration
- `.gitignore`, `.editorconfig` - Repo standards
- `examples/` - Example .claudiv files (but NOT their generated outputs)
- `spec.html` in root - keep as working example (rename to `example.claudiv`)

## Implementation Steps

### Step 1: Initialize Git and Create Clean History
```bash
# Initialize git
git init

# Create proper .gitignore (update with generated files)
# Make initial commits following commit strategy above

# Create first commit
git add .gitignore LICENSE .editorconfig
git commit -m "chore: initialize repository with gitignore and basic config"

# Add core source
git add src/ tsconfig.json
git commit -m "feat: core claudiv engine"

# ... continue with commit sequence
```

### Step 2: Restructure as Monorepo

**2.1 Install pnpm (if not installed)**
```bash
npm install -g pnpm
```

**2.2 Create Root Workspace**
Create `pnpm-workspace.yaml`:
```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

Create root `package.json`:
```json
{
  "name": "claudiv-monorepo",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "test": "turbo run test",
    "lint": "turbo run lint"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0"
  }
}
```

**2.3 Create Turborepo Config**
Create `turbo.json`:
```json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "test": {
      "dependsOn": ["build"]
    }
  }
}
```

**2.4 Move Core Package**
```bash
# Create packages directory
mkdir -p packages/claudiv

# Move current files
mv src packages/claudiv/
mv bin packages/claudiv/
mv tsconfig.json packages/claudiv/
mv package.json packages/claudiv/package.json.old

# Update package.json for core package
```

Update `packages/claudiv/package.json`:
```json
{
  "name": "@claudiv/core",
  "version": "0.1.0",
  "description": "Conversational UI development with Claude - XML-based specifications that generate working code",
  "main": "dist/index.js",
  "type": "module",
  "bin": {
    "claudiv": "./bin/claudiv.js"
  },
  "files": [
    "dist",
    "bin",
    "README.md"
  ],
  "publishConfig": {
    "access": "public",
    "provenance": true
  }
}
```

**2.5 Move Documentation**
```bash
mkdir -p docs
mv *.md docs/ (except README.md)
```

**2.6 Create Examples**
```bash
mkdir -p apps/examples/basic-website
mv spec.html apps/examples/basic-website/app.claudiv
```

### Step 3: Set Up GitHub Repository

**3.1 Create Repository on GitHub**
- Repository name: `claudiv`
- Public repository
- No template, no README (we have our own)

**3.2 Connect and Push**
```bash
git remote add origin https://github.com/amirguterman/claudiv.git
git branch -M main
git push -u origin main
```

**3.3 Set Up npm Trusted Publishing**
1. Go to https://www.npmjs.com/settings/{username}/packages/granular-access-tokens
2. Create new granular access token with:
   - Permissions: Publish
   - Packages: @claudiv/core
3. Add to GitHub secrets as `NPM_TOKEN`

Or use Trusted Publishing (preferred):
1. Link npm account to GitHub via OIDC
2. No secrets needed - automatic via GitHub Actions

### Step 4: GitHub Actions Workflows

**4.1 NPM Publish Workflow**
Create `.github/workflows/publish-npm.yml`:
```yaml
name: Publish to npm

on:
  push:
    tags:
      - 'v*'  # Trigger on version tags (v0.1.0, v1.0.0, etc.)

permissions:
  contents: read
  id-token: write  # Required for trusted publishing

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: pnpm install

      - name: Build
        run: pnpm run build

      - name: Publish to npm
        run: cd packages/claudiv && pnpm publish --provenance --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**4.2 CI Workflow**
Create `.github/workflows/ci.yml`:
```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: pnpm install

      - name: Type check
        run: pnpm run build

      - name: Lint
        run: pnpm run lint

      - name: Test
        run: pnpm run test
```

### Step 5: VSCode Extension (Future Phase)

**Directory: `packages/vscode/`**

Key files:
- `package.json` - VSCode extension manifest
- `src/extension.ts` - Extension activation
- `src/language-server.ts` - LSP for .claudiv files
- `src/preview-provider.ts` - Live preview webview
- `syntaxes/claudiv.tmLanguage.json` - Syntax highlighting

Features:
- Syntax highlighting for .claudiv files
- Autocomplete from XSD schema
- Live preview panel showing generated HTML
- Visual designer (drag-and-drop interface)
- Integrated dev server

### Step 6: Chrome Extension (Future Phase)

**Directory: `packages/chrome/`**

Key files:
- `manifest.json` - Manifest V3
- `src/background.ts` - Background service worker
- `src/sidepanel.ts` - Side panel UI for editing
- `src/content.ts` - Content script for live page editing

Features:
- Side panel with Claudiv editor
- Live edit any webpage with Claude
- Extract page structure to .claudiv format
- Save and sync .claudiv files

## Updated .gitignore

Root `.gitignore`:
```gitignore
# Dependencies
node_modules/

# Build outputs
dist/
*.tsbuildinfo

# Generated Claudiv files
*.claudiv.code.html
*.claudiv.spec.html
*.claudiv.rules.html
*.claudiv.models.html
*.claudiv.tests.html
*.claudiv.api.html
*.claudiv.docs.html
*.claudiv.data.html
*.claudiv.workflows.html

# Legacy format (keep for backward compat)
spec.code.html
spec.spec.html
spec.rules.html
spec.models.html

# Environment
.env
.env.local
.env.*.local

# Logs
*.log
npm-debug.log*
pnpm-debug.log*

# Editor directories
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json
.idea/

# OS files
.DS_Store
Thumbs.db

# Temporary files
*.tmp
*.swp
*~

# Turbo
.turbo
```

## Package.json Updates

**Root package.json:**
```json
{
  "name": "claudiv-monorepo",
  "version": "1.0.0",
  "private": true,
  "description": "Claudiv - Claude in a Div. Conversational UI development framework.",
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "turbo": "^2.0.0",
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0",
    "syncpack": "^12.0.0"
  },
  "engines": {
    "node": ">=20.0.0",
    "pnpm": ">=8.0.0"
  },
  "packageManager": "pnpm@8.0.0"
}
```

**packages/claudiv/package.json:**
```json
{
  "name": "@claudiv/core",
  "version": "0.1.0",
  "description": "Claude in a Div - Conversational UI development with Claude",
  "main": "dist/index.js",
  "type": "module",
  "bin": {
    "claudiv": "./bin/claudiv.js"
  },
  "files": [
    "dist/**/*",
    "bin/**/*",
    "README.md",
    "LICENSE"
  ],
  "scripts": {
    "dev": "tsx src/index.ts",
    "build": "tsc",
    "clean": "rm -rf dist",
    "prepublishOnly": "pnpm run build"
  },
  "publishConfig": {
    "access": "public",
    "provenance": true
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/amirguterman/claudiv.git",
    "directory": "packages/claudiv"
  },
  "keywords": [
    "claudiv",
    "claude",
    "ai",
    "conversational-ui",
    "xml",
    "code-generation",
    "spec-driven"
  ],
  "author": "Amir Guterman",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/amirguterman/claudiv/issues"
  },
  "homepage": "https://github.com/amirguterman/claudiv#readme"
}
```

## Verification Steps

### After Monorepo Setup:
1. `pnpm install` at root - should install all workspace dependencies
2. `pnpm run build` - should build all packages via Turborepo
3. `cd packages/claudiv && pnpm link --global` - test CLI globally
4. `claudiv` in example directory - verify it works
5. Create a test .claudiv file and verify generation works

### After GitHub Setup:
1. Push to GitHub - verify all files are committed correctly
2. Check GitHub Actions run successfully
3. Create a test tag: `git tag v0.1.0 && git push origin v0.1.0`
4. Verify npm publish workflow runs
5. Check package appears on npmjs.com

### After npm Publish:
1. `npx @claudiv/core@latest` - verify package is installable
2. Create new project and test full workflow
3. Verify provenance badge shows on npm package page

## Critical Decisions

### Decision 1: File Extension `.claudiv` vs `.html`
**Recommendation: `.claudiv`**
- **Pro**: Clear branding, no confusion with HTML
- **Pro**: VSCode can provide specialized support
- **Pro**: Clean generated file naming (app.claudiv → app.claudiv.code.html)
- **Con**: Requires migration from existing examples
- **Solution**: Support both, promote .claudiv as primary

### Decision 2: Monorepo Tool - Turborepo vs Nx
**Recommendation: pnpm + Turborepo**
- **Pro**: Industry standard in 2026, faster than Nx for simple cases
- **Pro**: Simpler configuration, less overhead
- **Pro**: Remote caching built-in
- **Con**: Nx has more features (we don't need them yet)

### Decision 3: npm Package Naming - `claudiv` vs `@claudiv/core`
**Recommendation: `@claudiv/core`**
- **Pro**: Namespace allows multiple related packages
- **Pro**: Future packages: @claudiv/vscode, @claudiv/chrome
- **Pro**: Professional appearance
- **Con**: Slightly longer to type
- **Solution**: Use both (claudiv can be an alias package pointing to @claudiv/core)

### Decision 4: VSCode Extension - Now vs Later
**Recommendation: Later (Phase 2)**
- Focus on core package first
- Get npm publishing working
- Gather user feedback
- Then build extension with proven core

## Implementation Timeline

**Phase 1: Clean Git History & Monorepo (Week 1)**
- Initialize git with clean commits
- Restructure as monorepo
- Set up pnpm workspaces + Turborepo
- Update all configs

**Phase 2: GitHub & npm Publishing (Week 1)**
- Push to GitHub
- Set up GitHub Actions
- Configure npm trusted publishing
- Publish v0.1.0

**Phase 3: File Extension Migration (Week 2)**
- Update CLI to support .claudiv
- Update schema to claudiv.xsd
- Migrate examples
- Update documentation

**Phase 4: VSCode Extension (Week 3-4)**
- Create package structure
- Implement language server
- Add syntax highlighting
- Build preview webview
- Publish to VSCode Marketplace

**Phase 5: Chrome Extension (Week 5-6)**
- Create manifest v3 structure
- Implement side panel
- Build content script for live editing
- Publish to Chrome Web Store

## Files That Need Changes

### Immediate Changes (Phase 1):
1. **/.gitignore** - Update with all generated files
2. **/pnpm-workspace.yaml** - Create workspace config
3. **/turbo.json** - Create Turborepo config
4. **/package.json** - Create root workspace package.json
5. **packages/core/package.json** - Move and update (rename from claudiv)
6. **packages/core/README.md** - Move current README
7. **docs/** - Move all *.md documentation files
8. **/.github/workflows/** - Create CI and publish workflows

### Core Parser Changes (Phase 1):
9. **src/parser.ts** - Add `<claudiv>` boundary detection, multi-file support
10. **src/watcher.ts** - Watch any file type, detect changes in referenced files
11. **src/index.ts** - Add CLI commands (gen, chat, reverse)
12. **bin/claudiv.js** - Add new command routing

### New Files (Phase 1):
13. **src/multi-file-resolver.ts** - Handle `ref` and `<embed>` tags
14. **src/rules-generator.ts** - Extract `<rules>` and generate .claude files
15. **src/reverse-generator.ts** - HTML → Claudiv conversion
16. **src/chat-mode.ts** - Chat-only mode (no code generation)

### Backend Features (Phase 3):
17. **src/generators/backend-generator.ts** - API, WebSocket, webhook generation
18. **src/generators/database-generator.ts** - Schema, migrations
19. **src/bash-executor.ts** - Bash command integration

### Runtime Package (Phase 4):
20. **packages/runtime/** - Browser script tag mode
21. **packages/runtime/claudiv.ts** - Main runtime
22. **packages/runtime/api-client.ts** - Anthropic API client (browser-safe)

### File Organization:
- **NO file renames** - Keep spec.html as is (supports any extension)
- Move docs to `docs/`
- Create examples in `apps/examples/`

### Configuration Files:
23. **.claudiv.config.json** (new) - Project configuration
```json
{
  "output": {
    "mode": "separate",
    "pattern": "${name}.generated.${ext}"
  },
  "watch": {
    "include": ["**/*.html", "**/*.xml"],
    "exclude": ["node_modules", "*.generated.*"]
  },
  "rules": {
    "generateClaudeFiles": true,
    "updateIndex": true
  }
}
```

## Success Criteria

✅ Git repository initialized with clean history
✅ Monorepo structure working with pnpm + Turborepo
✅ All source code in packages/claudiv/
✅ Generated files properly gitignored
✅ GitHub repository created and pushed
✅ GitHub Actions CI running successfully
✅ npm package published with provenance
✅ `npx @claudiv/core` works globally
✅ Documentation moved to docs/
✅ Examples in apps/examples/
✅ File extension .claudiv supported
✅ Backward compatibility with .html maintained

---

## New Features & Capabilities

### 1. Flexible File Format (No Forced Extensions)

**Important**: Do NOT force users into `.claudiv` file extension. Support ANY file:
- `.html` - Standard HTML files
- `.xml` - XML files
- `.php`, `.jsp`, `.erb` - Server-side templates
- No extension - Works anywhere

**Claudiv scopes work in ANY file format**

### 2. Scoped Boundaries with `<claudiv>` Tags

**Problem**: Currently processes entire file. Need to limit scope for integration scenarios.

**Solution**: Introduce `<claudiv></claudiv>` boundary tags.

**Example**:
```html
<!DOCTYPE html>
<html>
<head>
  <title>My Existing App</title>
</head>
<body>
  <header>Existing header - untouched by Claudiv</header>

  <claudiv>
    <ai-chat-widget provider="openai" model="gpt-4.1-mini" role="booking assistant" gen="">
      <toggle-button layout="modal w=80% h=80% of page"></toggle-button>
      <flow>
        <greeting></greeting>
        <operations>
          <search></search>
          <book></book>
        </operations>
      </flow>
    </ai-chat-widget>
  </claudiv>

  <footer>Existing footer - untouched by Claudiv</footer>
</body>
</html>
```

**Implementation**:
- Parser scans for `<claudiv>` tags
- Only processes content within `<claudiv>` boundaries
- Multiple `<claudiv>` sections allowed per file
- Backward compatible: if no `<claudiv>` tags, treat entire file as claudiv scope

### 2. Three Integration Modes

**Mode 1: Vite Project (Full Stack from Scratch)**
- Start new project: `npm create claudiv@latest`
- Generates Vite project with Claudiv as build system
- Uses Claude Code CLI or Anthropic API
- Watch mode with hot reload
- Full control over entire application

**Mode 2: Script Tag Integration (Existing HTML Pages)**
- Add to existing HTML pages
- Include script in `<head>`:
  ```html
  <script src="https://cdn.claudiv.dev/claudiv.min.js"></script>
  <script>
    Claudiv.init({
      apiKey: 'your-anthropic-api-key', // API mode only (not CLI)
      scope: 'claudiv', // Target <claudiv> tags
      autoRender: true
    });
  </script>
  ```
- **API mode only** (can't use Claude Code CLI in browser)
- Processes `<claudiv>` sections on page load
- Manual trigger: `Claudiv.generate()` or CLI command

**Mode 3: CLI/npm Command (Manual Generation)**
- Separate command to trigger generation: `claudiv gen`
- Works with both API and CLI modes
- Useful for:
  - CI/CD pipelines
  - Manual control over generation timing
  - Integration scripts
  - Pre-build steps

**Commands**:
```bash
# Watch mode (current behavior)
claudiv

# One-time generation
claudiv gen

# Specify file
claudiv gen app.claudiv

# Specify scope
claudiv gen --scope="#my-widget"

# Use API mode
claudiv gen --api

# Use CLI mode
claudiv gen --cli
```

### 3. Backend & Full-Stack Generation

**Database Support**:
```xml
<claudiv>
  <db sqlite="">
    <!-- Claudiv infers schema from app structure -->
  </db>

  <users-table>
    <user>
      <name type="string"></name>
      <email type="string" unique=""></email>
      <created_at type="datetime"></created_at>
    </user>
  </users-table>

  <bookings-table>
    <booking>
      <user_id type="foreign-key" ref="users"></user_id>
      <date type="datetime"></date>
      <status type="enum" values="pending,confirmed,cancelled"></status>
    </booking>
  </bookings-table>
</claudiv>
```

**Generates**:
- SQL schema files
- Migration scripts
- ORM models (Prisma, Drizzle, etc.)

**Backend API**:
```xml
<claudiv>
  <api framework="express" port="3000">
    <endpoint path="/users" method="GET">
      Fetch all users from database
    </endpoint>

    <endpoint path="/users/:id" method="GET">
      Fetch single user by ID
    </endpoint>

    <endpoint path="/bookings" method="POST" auth="required">
      Create new booking, validate dates, send confirmation email
    </endpoint>
  </api>
</claudiv>
```

**Generates**:
- Express/Fastify/Hono server code
- Route handlers
- Middleware
- Database queries
- Validation schemas

**WebSockets**:
```xml
<claudiv>
  <websocket path="/chat">
    <on event="message">
      Broadcast to all connected clients
    </on>

    <on event="typing">
      Notify room participants
    </on>
  </websocket>
</claudiv>
```

**Webhooks**:
```xml
<claudiv>
  <webhook url="/stripe-webhook" provider="stripe">
    <on event="payment.succeeded">
      Update booking status, send confirmation email
    </on>
  </webhook>
</claudiv>
```

### 4. AI Integration for End-User Apps

**Allow developers to add AI capabilities to their apps**:

```xml
<claudiv>
  <ai-function name="generateSummary" provider="openai" model="gpt-4" gen="">
    <input>User's text content</input>
    <prompt>Summarize the following text in 2-3 sentences</prompt>
    <output>Summary text</output>
    <handler>Display in #summary-div</handler>
  </ai-function>

  <ai-function name="searchProducts" provider="anthropic" model="claude-sonnet-4" gen="">
    <input>User search query</input>
    <prompt>Convert this natural language query into structured product search filters</prompt>
    <output type="json">
      {
        "category": "string",
        "priceRange": {"min": number, "max": number},
        "keywords": ["string"]
      }
    </output>
    <handler>Call /api/products/search with filters</handler>
  </ai-function>

  <button onclick="Claudiv.ai.generateSummary(document.querySelector('#content').innerText)">
    Summarize
  </button>
</claudiv>
```

**Configuration**:
```xml
<claudiv>
  <ai-config>
    <provider name="openai" api-key="env:OPENAI_API_KEY"></provider>
    <provider name="anthropic" api-key="env:ANTHROPIC_API_KEY"></provider>
  </ai-config>
</claudiv>
```

**Generates**:
- Client-side or server-side AI function wrappers
- API route handlers for server-side AI calls
- Type-safe interfaces
- Error handling
- Response caching (optional)

### 5. External Packages & Scripts

**Import external libraries**:
```xml
<claudiv>
  <imports>
    <package name="lodash" from="npm"></package>
    <package name="chart.js" from="cdn"></package>
    <script src="https://unpkg.com/alpinejs"></script>
  </imports>

  <dashboard gen="">
    Use Chart.js to create a line chart showing revenue over time.
    Use lodash to group data by month.
  </dashboard>
</claudiv>
```

**Generates**:
- Import statements (ES modules or script tags)
- Package.json dependencies (for npm packages)
- CDN links in HTML head

### 6. MCP Server Integration

**Connect to Model Context Protocol servers**:
```xml
<claudiv>
  <mcp-server name="database" url="http://localhost:3001/mcp">
    Provides access to PostgreSQL database
  </mcp-server>

  <mcp-server name="filesystem" url="http://localhost:3002/mcp">
    Provides file system access
  </mcp-server>

  <admin-panel gen="">
    Create a database admin panel that uses the 'database' MCP server
    to browse tables, run queries, and export data.
  </admin-panel>
</claudiv>
```

**Claude receives context from MCP servers when generating code**

### 7. Non-Development Use Cases

**Presentations**:
```xml
<claudiv output="presentation" format="reveal.js">
  <presentation title="Q4 Product Roadmap" theme="dark">
    <slide>
      Title slide with company logo
    </slide>

    <slide title="Our Vision">
      Vision statement with animated graphics
    </slide>

    <slide title="Features">
      Three-column layout showing upcoming features
    </slide>
  </presentation>
</claudiv>
```

**System Design / Architecture**:
```xml
<claudiv output="diagram" format="mermaid">
  <architecture title="Microservices Architecture">
    <service name="API Gateway" connects="auth-service,user-service,booking-service"></service>
    <service name="auth-service" database="postgres"></service>
    <service name="user-service" database="postgres"></service>
    <service name="booking-service" database="postgres" queue="redis"></service>
    <service name="notification-service" triggered-by="booking-service"></service>
  </architecture>
</claudiv>
```

**Generates**: Mermaid diagram code, PlantUML, or visual diagram

**Project Planning**:
```xml
<claudiv output="plan">
  <project title="Mobile App Redesign">
    <phase name="Discovery" duration="2 weeks">
      User research, competitor analysis, requirements gathering
    </phase>

    <phase name="Design" duration="3 weeks" depends-on="Discovery">
      Wireframes, mockups, design system
    </phase>

    <phase name="Development" duration="8 weeks" depends-on="Design">
      Frontend, backend, testing
    </phase>
  </project>
</claudiv>
```

**Generates**: Gantt chart, task breakdown, timeline

**CI/CD Pipelines**:
```xml
<claudiv output="cicd" platform="github-actions">
  <pipeline name="Deploy to Production">
    <trigger on="push" branch="main"></trigger>

    <stage name="Test">
      Run unit tests, integration tests, e2e tests
    </stage>

    <stage name="Build">
      Build Docker image, tag with commit SHA
    </stage>

    <stage name="Deploy" requires="Test,Build">
      Deploy to AWS ECS, run database migrations, warm cache
    </stage>
  </pipeline>
</claudiv>
```

**Generates**: `.github/workflows/*.yml` files

**Automation Scripts**:
```xml
<claudiv output="script" language="bash">
  <automation name="backup-databases">
    <schedule cron="0 2 * * *"></schedule>
    <steps>
      Dump PostgreSQL databases
      Compress with gzip
      Upload to S3 bucket
      Delete backups older than 30 days
      Send notification to Slack
    </steps>
  </automation>
</claudiv>
```

**Generates**: Bash script, cron configuration

**Reminders / Task Management**:
```xml
<claudiv output="tasks">
  <tasks project="Website Launch">
    <task priority="high" due="2026-03-01">
      Set up production environment
    </task>

    <task priority="medium" depends-on="Set up production environment">
      Configure CDN and DNS
    </task>

    <reminder when="2026-02-25">
      Check SSL certificates are ready
    </reminder>
  </tasks>
</claudiv>
```

**Generates**: Task list (Markdown, JSON, or calendar events)

### 8. Chat Mode (No Code Generation)

**Conversation-only mode for planning and discussion**:

```xml
<claudiv>
  <dashboard lock="" gen="">
    <!-- Existing dashboard implementation -->
  </dashboard>

  <chat>
    <user>
      How should I structure the database for multi-tenancy?
      Should I use separate databases or schema-based isolation?
    </user>
    <ai></ai>

    <user>
      What are the performance implications of each approach?
    </user>
    <ai></ai>
  </chat>
</claudiv>
```

**Behavior**:
- `<chat>` blocks do NOT generate code
- Faster responses (no code generation overhead)
- AI receives context from surrounding `<claudiv>` scope
- Useful for:
  - Architectural decisions
  - Planning features
  - Debugging strategies
  - Code review discussions

**Implementation**:
- Parser detects `<chat>` tags (distinct from action attributes)
- Sends context-aware prompt to Claude
- Responds with text only (no code blocks)
- Conversation history maintained within `<chat>` block

### 9. Reverse Generation (HTML/App → Claudiv)

**Convert existing applications back to Claudiv specifications**

**Use Cases**:
- Migrate existing HTML sites to Claudiv
- Extract structure from legacy applications
- Generate claudiv specs from screenshots/designs
- Document existing UIs

**Command**:
```bash
claudiv reverse index.html > app.claudiv.xml
claudiv reverse --url https://example.com > example.claudiv.xml
claudiv reverse --screenshot design.png > design.claudiv.xml
```

**Example**:

**Input** (existing HTML):
```html
<!DOCTYPE html>
<html>
<body>
  <header>
    <h1>My Website</h1>
    <nav>
      <a href="/">Home</a>
      <a href="/about">About</a>
    </nav>
  </header>
  <main>
    <section id="hero">
      <h2>Welcome!</h2>
      <p>This is my website</p>
      <button onclick="alert('Hello!')">Click me</button>
    </section>
  </main>
</body>
</html>
```

**Output** (generated Claudiv spec):
```xml
<claudiv>
  <website>
    <header>
      <heading>My Website</heading>
      <navigation>
        <link href="/">Home</link>
        <link href="/about">About</link>
      </navigation>
    </header>

    <main>
      <hero-section>
        <title>Welcome!</title>
        <description>This is my website</description>
        <cta-button action="alert('Hello!')">Click me</cta-button>
      </hero-section>
    </main>
  </website>
</claudiv>
```

**AI-Powered Analysis**:
- Claude analyzes HTML structure
- Identifies semantic components
- Extracts patterns and intent
- Generates clean, hierarchical claudiv spec
- Preserves functionality (JS, event handlers)

**Advanced**:
```bash
# Reverse engineer with AI analysis
claudiv reverse app.js --analyze

# Extract from React app
claudiv reverse src/ --framework=react

# From live site
claudiv reverse --url https://app.example.com --screenshot
```

### 10. Multi-File Support & References

**Organize claudiv specifications across multiple files**

**Reference Other Files**:
```xml
<!-- main.claudiv.xml -->
<claudiv>
  <website>
    <header ref="components/header.claudiv.xml"></header>
    <sidebar ref="components/sidebar.claudiv.xml"></sidebar>
    <main ref="pages/home.claudiv.xml"></main>
  </website>
</claudiv>
```

```xml
<!-- components/header.claudiv.xml -->
<claudiv>
  <header>
    <logo></logo>
    <navigation>
      <link href="/">Home</link>
      <link href="/about">About</link>
    </navigation>
  </header>
</claudiv>
```

**Embed Files Inline**:
```xml
<claudiv>
  <dashboard>
    <!-- Inline embed: replaces <embed> with file contents -->
    <embed src="widgets/stats.claudiv.xml"></embed>
    <embed src="widgets/chart.claudiv.xml"></embed>
  </dashboard>
</claudiv>
```

**Benefits**:
- Better organization (components in separate files)
- Reusable components across projects
- Team collaboration (multiple devs, different files)
- Version control friendly

**Relative Paths**:
- Relative to current file: `ref="./header.claudiv.xml"`
- Relative to project root: `ref="/components/header.claudiv.xml"`
- From packages: `ref="@claudiv/ui/button.claudiv.xml"`

**Watch Mode**:
```bash
# Watch main file and all references
claudiv --watch main.claudiv.xml

# Auto-detects changes in referenced files
```

### 11. Scope-Specific `<rules>` Tag

**Integrate with Claude Code's native memory management**

**Local Rules** (apply only to current scope):
```xml
<claudiv>
  <rules>
    - Always use Tailwind CSS for styling
    - Follow Material Design guidelines
    - Use TypeScript for all scripts
    - API calls must have error handling
    - All forms must have validation
  </rules>

  <dashboard gen="">
    <!-- These rules apply when generating dashboard -->
    Create an admin dashboard with user stats and charts
  </dashboard>
</claudiv>
```

**Nested Rules** (hierarchical override):
```xml
<claudiv>
  <rules>
    Global rules for entire app
  </rules>

  <admin-panel>
    <rules>
      Admin-specific rules (override global)
      - Require authentication
      - Log all actions
    </rules>

    <users-table gen="">
      <!-- Uses both global + admin rules -->
    </users-table>
  </admin-panel>
</claudiv>
```

**Integration with .claude/rules**:

When Claudiv processes `<rules>` tags:
1. Extracts rules from all scopes (current + parents)
2. Generates `.claude/rules/claudiv-generated.md`
3. Updates `.claude/index` to reference generated rules
4. Claude Code automatically loads these rules

**Example Generated File**:
```markdown
<!-- .claude/rules/claudiv-dashboard.md -->
# Rules for dashboard component

Path: /dashboard

## Styling
- Always use Tailwind CSS for styling
- Follow Material Design guidelines

## TypeScript
- Use TypeScript for all scripts

## API & Validation
- API calls must have error handling
- All forms must have validation

Generated from: app.claudiv.xml > dashboard > rules
```

**Automatic Indexing**:
```markdown
<!-- .claude/index -->
# Claudiv Generated Rules

- [Dashboard Rules](rules/claudiv-dashboard.md) - Applies to: /dashboard, /admin/dashboard
- [API Rules](rules/claudiv-api.md) - Applies to: /api/**
```

**Benefits**:
- Rules persist across sessions
- Claude Code automatically applies them
- Hierarchical rule inheritance
- Auto-generated, version-controlled
- No manual .claude/rules maintenance

### 12. Bash Command Execution

**Execute bash commands from claudiv, integrated with Claude Code's Bash tool**

**Define Commands**:
```xml
<claudiv>
  <bash id="git-pull" permissions="git" cd="scripts/">
    git pull origin main
  </bash>

  <bash id="deploy-prod" permissions="git,npm,filesystem" confirm="true">
    npm run build
    git add dist/
    git commit -m "Build for production"
    git push origin main
  </bash>

  <bash id="backup-db" permissions="filesystem,network">
    pg_dump mydb > backups/$(date +%Y%m%d).sql
    aws s3 cp backups/ s3://my-backups/ --recursive
  </bash>
</claudiv>
```

**Trigger from UI**:
```xml
<claudiv>
  <admin-panel gen="">
    <button click="bash:git-pull">Pull Latest</button>
    <button click="bash:deploy-prod">Deploy to Production</button>
    <button click="bash:backup-db">Backup Database</button>
  </admin-panel>
</claudiv>
```

**Generated Code**:
```javascript
// Generated button handler
document.querySelector('#deploy-prod-btn').addEventListener('click', async () => {
  if (confirm('Run: npm run build && git add dist/ && ...?')) {
    const response = await fetch('/api/bash/deploy-prod', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${getToken()}` }
    });

    const result = await response.json();
    if (result.success) {
      alert('Deployment successful!');
    } else {
      alert(`Error: ${result.error}`);
    }
  }
});
```

**Backend API Route** (auto-generated):
```typescript
// Generated Express route
app.post('/api/bash/:commandId', authenticate, async (req, res) => {
  const { commandId } = req.params;

  // Security: Check permissions
  if (!hasPermission(req.user, commandId)) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  // Execute via Claude Code's Bash tool
  try {
    const result = await executeClaudeBash(commandId);
    res.json({ success: true, output: result.stdout });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

**Integration with Claude Code Bash Tool**:
```typescript
import { spawn } from 'child_process';

async function executeClaudeBash(commandId: string) {
  const bashDef = getBashDefinition(commandId); // From claudiv spec

  // Spawn Claude Code CLI to execute bash via its tool
  const claude = spawn('claude-code', [
    '--tool', 'Bash',
    '--command', bashDef.command,
    '--cwd', bashDef.cd,
    '--permissions', bashDef.permissions
  ]);

  return new Promise((resolve, reject) => {
    let stdout = '';
    let stderr = '';

    claude.stdout.on('data', (data) => { stdout += data });
    claude.stderr.on('data', (data) => { stderr += data });

    claude.on('close', (code) => {
      if (code === 0) resolve({ stdout, stderr });
      else reject(new Error(stderr));
    });
  });
}
```

**Permissions System**:
```xml
<bash id="safe-command" permissions="git">
  <!-- Only git commands allowed -->
</bash>

<bash id="dangerous-command" permissions="filesystem,network,git" confirm="true">
  <!-- Multiple permissions, requires confirmation -->
</bash>

<bash id="admin-only" permissions="*" roles="admin">
  <!-- Full permissions, admin role required -->
</bash>
```

**Security Features**:
- Permission validation before execution
- Role-based access control
- Confirmation prompts for destructive commands
- Command logging and audit trail
- Sandboxed execution (via Claude Code's Bash tool)
- Rate limiting

**Attributes**:
- `id` - Unique identifier
- `permissions` - Comma-separated (git, npm, filesystem, network, docker, etc.)
- `cd` - Working directory
- `confirm` - Require user confirmation (boolean)
- `roles` - Required user roles (admin, user, etc.)
- `timeout` - Max execution time (ms)

### 13. Demo Recording for GitHub

**Generate animated GIFs and videos showing Claudiv in action**

**Tools**:
- **asciinema** - Record terminal sessions
- **vhs** (by Charm) - Generate terminal GIFs from scripts
- **OBS Studio** - Record screen for VSCode extension demos
- **gifski** - Convert video to high-quality GIFs
- **ttygif** - Terminal to GIF

**Demo Scenarios**:
1. **Quick Start** - Create app from scratch (30 seconds)
2. **Attribute-Based Syntax** - Add gen, lock, unlock (20 seconds)
3. **Real-time Generation** - Watch as Claude builds UI (1 minute)
4. **Full-Stack App** - Database + API + Frontend (2 minutes)
5. **VSCode Extension** - Visual designer in action (1 minute)

**VHS Script Example** (creates animated GIF):
```tape
# quickstart.tape
Output demos/quickstart.gif

Set Shell bash
Set FontSize 14
Set Width 1200
Set Height 600
Set Theme "Dracula"

Type "echo '<app><button gen=\"\">Make a blue button</button></app>' > app.claudiv"
Enter
Sleep 1s

Type "claudiv"
Enter
Sleep 3s

Type "# Browser opens with working blue button!"
Sleep 2s
```

**Generation Command**:
```bash
vhs demos/quickstart.tape
```

**GitHub README Integration**:
```markdown
## Quick Start

![Quick Start Demo](./demos/quickstart.gif)

Create a working app in 30 seconds:
...
```

**Implementation Plan**:
1. Create `demos/` directory in monorepo
2. Write VHS tape scripts for each demo
3. Add npm script: `npm run demos` to generate all GIFs
4. Include in CI/CD to auto-update demos
5. Reference in README.md

## Updated Architecture

### Parser Changes

**Before**: Process entire file
**After**:
1. Scan for `<claudiv>` boundaries
2. Process only content within boundaries
3. Support multiple `<claudiv>` sections per file
4. Fallback: If no `<claudiv>` tags, treat whole file as claudiv scope

**New Detection Logic**:
```typescript
function parseClaudivScopes(content: string) {
  const scopes = [];
  const claudivRegex = /<claudiv>([\s\S]*?)<\/claudiv>/g;
  let match;

  while ((match = claudivRegex.exec(content)) !== null) {
    scopes.push({
      start: match.index,
      end: match.index + match[0].length,
      content: match[1],
      fullMatch: match[0]
    });
  }

  // Fallback: no <claudiv> tags = whole file is scope
  if (scopes.length === 0) {
    scopes.push({
      start: 0,
      end: content.length,
      content: content,
      fullMatch: content
    });
  }

  return scopes;
}
```

### Generator Changes

**Before**: Generate only HTML/CSS
**After**: Multi-format generation based on `output` attribute

**Output Types**:
- `output="html"` (default) - HTML/CSS/JS
- `output="backend"` - Express/Fastify server code
- `output="database"` - SQL schema, migrations
- `output="presentation"` - Reveal.js, PowerPoint
- `output="diagram"` - Mermaid, PlantUML
- `output="plan"` - Markdown, JSON
- `output="cicd"` - GitHub Actions YAML
- `output="script"` - Bash, Python, Node.js

**Format Detection**:
```typescript
function determineOutputFormat($: CheerioAPI, element: Element): OutputFormat {
  const outputAttr = $(element).attr('output');

  if (outputAttr) return outputAttr as OutputFormat;

  // Auto-detect from content
  if ($(element).find('api, endpoint').length) return 'backend';
  if ($(element).find('db, table, schema').length) return 'database';
  if ($(element).find('presentation, slide').length) return 'presentation';
  if ($(element).find('architecture, diagram').length) return 'diagram';
  if ($(element).find('pipeline, stage').length) return 'cicd';

  return 'html'; // default
}
```

### Script Tag Mode

**New Package**: `@claudiv/runtime`

**Usage**:
```html
<script src="https://cdn.claudiv.dev/claudiv.min.js"></script>
<script>
  Claudiv.init({
    apiKey: 'sk-ant-...',
    scope: 'claudiv',
    autoRender: true,
    model: 'claude-sonnet-4'
  });
</script>

<claudiv>
  <button gen="">Make it blue</button>
</claudiv>
```

**Runtime Behavior**:
1. Scans DOM for `<claudiv>` tags
2. Extracts specifications
3. Sends to Anthropic API (browser can't use Claude Code CLI)
4. Generates HTML/CSS/JS
5. Injects into `<claudiv>` container
6. Hot-swaps on attribute changes

**Security Considerations**:
- API key should be proxied through user's backend
- Never expose API keys in client-side code
- Provide proxy endpoint: `/api/claudiv/generate`

### CLI Commands

**New Commands**:
```bash
claudiv                    # Watch mode (current)
claudiv gen                # One-time generation
claudiv gen --file app.claudiv
claudiv gen --scope="#widget"
claudiv chat               # Chat-only mode (no generation)
claudiv init               # Initialize new project
claudiv init --vite        # Create Vite project
claudiv add <package>      # Add external package
claudiv db migrate         # Run database migrations
claudiv deploy             # Deploy to production
```

## Updated Monorepo Structure

```
claudiv/
├── packages/
│   ├── core/              # @claudiv/core (current package)
│   ├── runtime/           # @claudiv/runtime (browser script)
│   ├── vscode/            # VSCode extension
│   ├── chrome/            # Chrome extension
│   ├── generators/        # Code generators
│   │   ├── html/
│   │   ├── backend/
│   │   ├── database/
│   │   ├── presentation/
│   │   ├── diagram/
│   │   └── cicd/
│   └── templates/         # Project templates
│       ├── vite-react/
│       ├── vite-vue/
│       ├── fullstack/
│       └── api-only/
├── apps/
│   └── examples/
│       ├── basic-ui/
│       ├── fullstack-app/
│       ├── ai-integration/
│       ├── presentation/
│       ├── system-design/
│       └── cicd-pipeline/
├── demos/                 # VHS tape scripts for GIF demos
│   ├── quickstart.tape
│   ├── fullstack.tape
│   ├── ai-chat.tape
│   └── README.md
└── docs/
```

## Implementation Priority

### Phase 1: Core Restructuring & Foundation (Week 1)
**Status: Ready to start**
- ✅ Monorepo setup (pnpm + Turborepo)
- ✅ Git history (clean commits)
- ✅ GitHub publishing (Actions + npm)
- 🆕 Scoped boundaries (`<claudiv>` tags)
- 🆕 Flexible file format (any extension)
- 🆕 CLI commands (`claudiv gen`, `claudiv chat`, etc.)
- 🆕 Multi-file support (`ref`, `<embed>`)

### Phase 2: Advanced Features (Week 2)
- 🆕 Reverse generation (HTML → Claudiv)
- 🆕 `<rules>` tag + .claude integration
- 🆕 Chat mode (`<chat>` tags)
- 🆕 Bash execution (`<bash>` tags)
- External package imports (`<imports>`)

### Phase 3: Backend & Full-Stack (Week 3)
- Backend generation (API, Express/Fastify)
- Database schema generation (`<db>`)
- WebSocket support (`<websocket>`)
- Webhook handlers (`<webhook>`)
- MCP server integration

### Phase 4: Integration Modes (Week 4)
- Script tag mode (`@claudiv/runtime`)
- Vite project templates
- Manual generation command
- Configuration system

### Phase 5: AI Integration (Week 5)
- AI function generation (`<ai-function>`)
- Provider configuration (OpenAI, Anthropic)
- Client/server-side AI calls
- Response handling and caching

### Phase 6: Non-Dev Use Cases (Week 6)
- Presentation generation (Reveal.js, PowerPoint)
- System diagram generation (Mermaid, PlantUML)
- CI/CD pipeline generation (GitHub Actions, GitLab CI)
- Planning and task tools
- Automation scripts

### Phase 7: Extensions & Demos (Week 7)
- VSCode extension (language server, preview)
- Chrome extension (side panel editor)
- Demo recordings (VHS/asciinema GIFs)
- Example projects and tutorials

## Summary

This plan transforms Claudiv from a working prototype into a professional, scalable monorepo ready for public release and community contribution. The structure supports future expansion (VSCode extension, Chrome extension) while maintaining a clean, focused core package. By following 2026 best practices (pnpm, Turborepo, trusted publishing), we ensure the project is built on solid foundations that scale.

**Paradigm Shift:**
- **Traditional**: Write JSX/HTML → Get UI
- **This system**: Write XML-ish + Chat with AI → Get React/HTML automatically

**spec.html IS the development interface.** Developers primarily work in spec.html, not in the generated code files. The hierarchy defines structure, chat elements enable conversational feature development, and the system handles the translation to working code.

**Key Innovation - Living Development Interface:**
1. **spec.html** = Your workspace (what you edit daily)
2. **AI conversations** = Embedded directly in the hierarchy (`<chat>` elements)
3. **Generated code** = Output (React components or HTML) that updates automatically
4. **Ongoing development** = Continue using spec.html throughout the entire project lifecycle

**Why this approach:**
- **Lower cognitive load**: Describe what you want, not how to implement it
- **Conversational development**: Chat with AI contextually about any part of your app
- **Living specification**: spec.html is always up-to-date with your app's structure
- **Rapid iteration**: Change hierarchy or chat for modifications, see results instantly
- **Version control friendly**: spec.html is readable, reviewable, and tracks intent
- **No rigid conventions**: Freeform tags, AI understands from context

## Requirements Summary

### Core Features
1. **XML Code Editor** (focus for now, visual designer later)
   - Watch spec.html for changes
   - Parse XML with any tag names (no conventions)
   - Detect chat patterns (user message + empty `<ai/>` element)
   - Send requests to Claude Code in **subscription mode** (CLI, not API)
   - Stream responses back into the `<ai>` element

2. **Multi-File Artifact Architecture**

   **Primary Input:**
   - **spec.html** = Freeform XML-ish (user's intent with AI conversations)

   **Core Generated Artifacts:**
   - **{name}.spec.html** = Structured hierarchical specification (for Claude Code tracking/updates)
   - **{name}.rules.html** = Rules and guidance for Claude Code (constraints, conventions, project-specific rules)
   - **{name}.models.html** = Business objects, entities, data models extracted from conversations
   - **{name}.tests.html** = Test specifications and test cases derived from requirements and conversations
   - **spec.code.html** = Generated HTML/CSS (browser-ready implementation)

   **Simple Workflow:**
   ```
   User adds chat in spec.html:
   <chat>
     <any-tag-name>make button bigger</any-tag-name>  <!-- ANY tag name -->
     <ai/>  <!-- MUST be <ai> tag -->
   </chat>

   → AI responds in <ai> tag (with full scope context)
   → Component automatically regenerated
   → Browser updates via HMR
   ```

   **Extended Artifacts (Future/Optional):**
   - **{name}.api.html** = API endpoints, routes, request/response schemas
   - **{name}.docs.html** = Generated documentation (user guides, API docs, component docs)
   - **{name}.data.html** = Sample data, fixtures, seed data for testing
   - **{name}.workflows.html** = User flows, state machines, business processes
   - **{name}.schemas.html** = Database schemas, validation schemas
   - **{name}.permissions.html** = Access control, roles, permissions matrix
   - **{name}.integrations.html** = Third-party integrations, webhooks, external services
   - **{name}.config.html** = Configuration options, environment variables, feature flags
   - **{name}.i18n.html** = Internationalization strings and translations

   **Purpose of generated files:**
   - **`.spec.html`**: Well-formed, structured version for Claude tracking. Strips chat elements, normalizes tags, maintains clean hierarchy
   - **`.rules.html`**: Project-specific rules, conventions, and constraints that Claude should follow. Updated as the project evolves
   - **`.models.html`**: Data structures, entities, business objects that emerge from user requirements. Defines the "business logic layer"
   - **`.tests.html`**: Test specifications and test cases derived from requirements and conversations
   - **`.api.html`**: RESTful or GraphQL API specifications extracted from feature discussions
   - **`.docs.html`**: Auto-generated documentation from specs and conversations
   - **`.data.html`**: Sample/seed data for development and testing
   - **`.workflows.html`**: User journeys and business process flows
   - **`.code.html`**: Final browser-ready HTML/CSS implementation

   **Extensibility:** The system is designed to be modular - new artifact generators can be added as plugins

3. **Build Target Options**
   - **Standalone HTML** (default): Generate spec.code.html for direct browser use
   - **Vite/React Project**: Integrate into existing Vite/React projects, generate React components
   - **Ongoing Development Workflow**: spec.html remains the living source of truth throughout the project lifecycle
   - Continuously regenerate components as spec.html evolves
   - Not just scaffolding - this is the primary development method

4. **NPM Package**
   - Simple starter script to initialize and run the editor
   - Works in the same folder as spec.html
   - Supports both Claude Code CLI (subscription mode) and Anthropic API
   - Can initialize as standalone or within a Vite/React project

### User's Important Requirements
1. **Dual Mode Support:** Support both Claude Code subscription (CLI) and Anthropic API modes, configurable via `.env` or CLI flag
2. **Very Lenient Parsing:** The XML-ish structure doesn't follow strict XML rules - tags can be unclosed, have spaces/special chars, or be malformed. Focus on extracting hierarchy for context
3. **Hierarchy = Context:** The position of `<chat>` elements in the hierarchy automatically defines Claude's context through **recursive parent scope reference**. Context resolution combines:
   - **Full hierarchy chain** (app > sidebar > menu, with all attributes at each level)
   - **Recursive parent scopes** (each parent's content, attributes, and purpose)
   - **Rules from .rules.html** (project conventions apply at each hierarchy level)
   - **Models from .models.html** (available entities and their fields)
   - **Cross-references** (automatically resolve references to other elements in spec.html)
   - **Surrounding elements** (sibling components at current and parent levels)

   Example: Chat in `<app theme="dark"><sidebar><menu name="settings">` receives context about dark theme (from app), sidebar layout (from parent), and settings menu (current scope).

4. **Cross-Referencing:** Elements can reference other elements defined elsewhere
   - Define APIs, models, workflows once - reference them naturally in requests
   - System automatically detects references and includes full definitions in context
   - Example: `<fetch users on click/>` automatically includes the "fetch users" API definition
   - No special syntax - just use element names naturally
5. **Multiple Build Targets:**
   - **Standalone HTML**: Generate spec.code.html (browser-ready HTML/CSS)
   - **Vite/React Integration**: Generate React components and integrate into Vite projects
6. **Unique React Development Workflow:** When used with Vite/React, this provides a declarative, conversation-driven way to build React apps

## Technology Stack

### Core Dependencies
- **Node.js 20+** - Runtime environment
- **TypeScript 5.x** - Type safety and modern JavaScript
- **chokidar 5.x** - Cross-platform file watching
- **htmlparser2 9.x** - Very lenient HTML/XML parsing (handles malformed structures)
- **cheerio 1.0.x** - jQuery-like API for manipulating parsed hierarchy
- **lodash.debounce 4.0.8** - Debouncing file change events
- **dotenv** - Environment configuration for API keys and mode selection

### Build Target Dependencies
- **For React/Vite mode:**
  - **vite** - Build tool and dev server
  - **react** - React library (peer dependency)
  - **react-dom** - React DOM renderer (peer dependency)
  - **@vitejs/plugin-react** - Vite React plugin

### Claude Integration (Dual Mode)
- **Mode 1 - Subscription (CLI):**
  - **Claude Code CLI** - Use installed `claude-code` command as subprocess
  - **child_process** (Node.js built-in) - Spawn Claude processes

- **Mode 2 - API:**
  - **@anthropic-ai/sdk** - Direct Anthropic API for pay-per-use
  - **API streaming** - Real-time response streaming

- **Configuration:** `.env` file or CLI flag to choose mode

### Dev Dependencies
- **tsx** - TypeScript execution for development
- **@types/node** - Node.js type definitions
- **@types/lodash.debounce** - Type definitions

## Project Structure

```
/home/source/gui-driven-spec/
├── package.json                     # NPM package configuration
├── tsconfig.json                    # TypeScript configuration
├── .gitignore                       # Ignore node_modules, dist, etc.
├── README.md                        # Documentation
├── bin/
│   └── gui-spec-editor.js          # CLI entry point (executable)
├── src/
│   ├── index.ts                     # Main application entry
│   ├── config.ts                    # Configuration loader (.env, CLI args, build target)
│   ├── watcher.ts                   # File watching logic
│   ├── parser.ts                    # Lenient XML-ish parsing & hierarchy extraction
│   ├── claude-client.ts             # Unified Claude interface (CLI or API)
│   ├── claude-cli.ts                # Claude Code CLI integration
│   ├── claude-api.ts                # Anthropic API integration
│   ├── updater.ts                   # XML-ish response insertion
│   ├── build-targets/               # Build target implementations
│   │   ├── html-builder.ts          # Standalone HTML output
│   │   ├── react-builder.ts         # React/Vite output
│   │   └── builder-registry.ts      # Build target selection
│   ├── code-generator.ts            # Generate proper HTML for spec.code.html
│   ├── types.ts                     # TypeScript interfaces
│   └── utils/
│       ├── logger.ts                # Console logging utilities
│       └── hierarchy-helpers.ts     # Hierarchy traversal and context building
├── spec.html                        # User's freeform XML-ish (already exists)
├── spec.spec.html                   # Structured hierarchical specification
├── spec.rules.html                  # Project rules and conventions
├── spec.models.html                 # Business entities and data models
├── spec.tests.html                  # Test specifications and test cases
├── spec.api.html                    # API specifications (optional)
├── spec.docs.html                   # Generated documentation (optional)
├── spec.data.html                   # Sample/seed data (optional)
├── spec.tests.html                  # Test specifications (optional)
├── spec.workflows.html              # User flows and processes (optional)
└── spec.code.html                   # Final HTML/CSS implementation
```

## Core Components

### 1. Main Entry Point (src/index.ts)
**Responsibility:** Application orchestration and lifecycle management

- Initialize logger
- Verify Claude Code CLI is installed (`claude-code --version`)
- Set up file watcher for spec.html
- Coordinate workflow: watch → parse → send to Claude → update XML → generate code
- Handle graceful shutdown (Ctrl+C)
- Provide status output to user

### 2. File Watcher (src/watcher.ts)
**Responsibility:** Monitor spec.html for changes

- Use chokidar to watch spec.html
- Debounce changes (300ms) to avoid processing rapid edits
- Emit 'change' events when file stabilizes
- Ignore spec.code.html to prevent circular triggers
- Configuration:
  ```typescript
  chokidar.watch('spec.html', {
    ignoreInitial: false,      // Process on startup
    awaitWriteFinish: {
      stabilityThreshold: 300,  // Wait for file to stabilize
      pollInterval: 100
    }
  })
  ```

### 3. Lenient Parser (src/parser.ts)
**Responsibility:** Parse XML-ish structure and detect chat patterns with extreme leniency

**Key Functions:**
- `parseHierarchy(content: string)` - Parse using htmlparser2 (very lenient)
- `detectChatPatterns(parsed)` - Find user message + empty `<ai>` sequences
- `extractUserMessage(element)` - Extract ALL content from user element (text, nested tags, mixed content)
- `extractHierarchyContext(element)` - Build automatic context from recursive parent scope chain
- `buildElementRegistry(parsed)` - Track named elements for cross-referencing
- `resolveReferences(element, registry)` - Resolve references to other elements
- `locateElement(pattern)` - Track position for updates

**Lenient Parsing Strategy:**
Use `htmlparser2` which handles:
- Unclosed tags: `<menu>` without `</menu>`
- Invalid tag names: `<my button>`, `<item-123>`, `<user's request>`
- Malformed nesting: overlapping tags, missing closures
- Mixed content: text and tags intermixed

```typescript
import { parseDocument } from 'htmlparser2';
import { Element, Text } from 'domhandler';

function parseHierarchy(content: string) {
  // htmlparser2 is extremely lenient
  return parseDocument(content, {
    lowerCaseAttributeNames: false,
    lowerCaseTags: false,  // Preserve original case
    recognizeSelfClosing: true
  });
}
```

**Chat Pattern Detection (Lenient):**
1. Parse content with htmlparser2 (handles any malformed structure)
2. Use cheerio to traverse DOM-like tree
3. Find `<chat>` elements at any depth (or similar patterns)
4. **Look for ANY non-`<ai>` element as user message** (no required tag name!)
   - Extract ALL content from user element:
     - Plain text (can be multiline)
     - Nested tags and their content
     - Mixed content (text + tags)
   - Flatten nested structure into coherent prompt
5. **Look for empty `<ai>` or `<ai/>` element immediately after** (REQUIRED tag name)
6. Build automatic context from parent chain and surrounding scope

**Key Pattern Rules:**
- **User message**: ANY tag name allowed (`<request/>`, `<please/>`, `<make-it-blue/>`, `<foo/>`, etc.)
  - Can be self-closing: `<request/>`
  - Can have simple text: `<request>make it blue</request>`
  - **Can have mixed content**: Text + nested tags + multiline descriptions
  - **Can have nested elements**: `<specific feature>details</specific feature>` within user message
  - All content (text and nested tags) is extracted and sent to Claude
- **AI response**: MUST be `<ai>` tag (distinguishes AI responses from user messages)
- **Scope**: Context includes:
  - Current component's scope (where chat is added)
  - Parent component's scope (and its attributes)
  - Grandparent scope, great-grandparent scope, etc. (full recursive chain)
  - All attributes and sibling elements at each level

**Automatic Context Building:**
For a chat element at this position:
```xml
<app theme="dark">
  <sidebar width="250px">
    <menu name="settings" position="top">
      <chat>
        <request>make it blue</request>  <!-- ANY tag name OK for user -->
        <ai/>  <!-- MUST be <ai> tag for AI -->
      </chat>
    </menu>
  </sidebar>
</app>
```

Context automatically resolved with **recursive parent scope reference**:
```
Hierarchy Chain: app[theme="dark"] > sidebar[width="250px"] > menu[name="settings", position="top"]

Scope Stack (from outermost to innermost):
1. App Scope:
   - Element: <app theme="dark">
   - Attributes: theme="dark"
   - Contains: Full app structure

2. Sidebar Scope:
   - Element: <sidebar width="250px">
   - Parent: <app theme="dark">
   - Attributes: width="250px"
   - Contains: Navigation menus

3. Menu Scope (Current):
   - Element: <menu name="settings" position="top">
   - Parent: <sidebar width="250px"> inside <app theme="dark">
   - Attributes: name="settings", position="top"
   - Contains: Menu items and settings

Contextual Prompt to Claude:
"You are working on a menu with name 'settings' at position 'top',
 inside a sidebar with width '250px',
 within the main app with theme 'dark'.

 Consider:
 - App-level context: dark theme is active
 - Sidebar context: 250px width constrains your layout
 - Menu context: this is the settings menu at the top position

 Rules from .rules.html for: app components, sidebar components, menu components
 Models from .models.html: all available entities

 User request: make it blue"
```

**Recursive Parent Scope Algorithm:**
```typescript
function buildContextFromHierarchy(element) {
  const scopes = [];
  let current = element;

  // Walk up the parent chain
  while (current.parent) {
    scopes.unshift({
      element: current.parent.name,
      attributes: current.parent.attributes,
      description: describeElement(current.parent)
    });
    current = current.parent;
  }

  // Build nested context description
  let context = "You are working on";
  for (let i = scopes.length - 1; i >= 0; i--) {
    context += ` ${scopes[i].description}`;
    if (i > 0) context += ", inside";
  }

  return context;
}
```

**Example Detection (Flexible User Tags):**

**Simple patterns:**
```xml
<menu>
  <chat>
    <please>make it blue background</please>  <!-- User message - any tag -->
    <ai></ai>  <!-- AI response - MUST be <ai> -->
  </chat>
</menu>

<menu>
  <chat>
    <request/>  <!-- Self-closing user message - any tag -->
    <ai/>       <!-- AI response - MUST be <ai> -->
  </chat>
</menu>

<menu>
  <chat>
    <make-background-blue-with-rounded-corners/>  <!-- Long descriptive tag -->
    <ai/>  <!-- AI response -->
  </chat>
</menu>
```

**Complex patterns with mixed content:**
```xml
<chat>
  <create a blue button>
    multiline extended description not inside a tag
    additional context or requirements

    <specific feature>the button should have responsive style</specific feature>
    <another detail>add hover effect</another detail>

    more freeform text explaining the requirements
  </create a blue button>
  <ai/>
</chat>
```

**Key capabilities:**
- **Multiline text**: User messages can span multiple lines
- **Mixed content**: Text + nested tags within user message element
- **Nested features**: Use nested tags for specific requirements
- **Freeform text**: Not everything needs to be in tags
- **Any structure**: System extracts all content from user message element

**What gets sent to Claude:**
For the complex example above, Claude receives:
```
User request:
"create a blue button

multiline extended description not inside a tag
additional context or requirements

Specific feature: the button should have responsive style
Another detail: add hover effect

more freeform text explaining the requirements"
```

All patterns above are valid! The system detects:
- ANY element (except `<ai>`) followed by
- Empty `<ai>` or `<ai/>` element
- Extracts ALL content from user element (text, nested tags, mixed content)
- Context from **recursive parent scopes** (current menu scope, parent sidebar scope, grandparent app scope, with all attributes)
- **Cross-references** to other elements in the spec (resolved automatically)
- Rules and models applied at all hierarchy levels

### Cross-Referencing Elements

**Key Feature:** Elements can reference other elements defined elsewhere in spec.html

**Example:**
```xml
<api>
  <fetch users>
    fetches the users with orders
    <endpoint example>
      curl https://api.example.com/users?with=orders -H "Authorization: Bearer token"
    </endpoint example>
  </fetch users>
</api>

<button>
  <fetch users on click/>  <!-- References the "fetch users" API above -->
</button>
```

**How it works:**
1. **Element Registry**: System maintains a registry of named elements as it parses
   - `fetch users` is registered as an API endpoint with its definition
   - Registry includes: element name, location, content, attributes

2. **Reference Resolution**: When processing user messages, system detects references
   - `<fetch users on click/>` contains "fetch users" - matches registry entry
   - System includes the full "fetch users" definition in context

3. **Context Enhancement**: Claude receives both the request AND the referenced element
   ```
   User request: "fetch users on click"

   Referenced element "fetch users":
   - Type: API endpoint
   - Description: fetches the users with orders
   - Endpoint: curl https://api.example.com/users?with=orders -H "Authorization: Bearer token"

   Context: You are adding a button that should call the "fetch users" API when clicked.
   The API endpoint and details are provided above.
   ```

4. **Automatic Discovery**: System scans entire spec.html to build registry
   - Named elements in `<api>`, `<models>`, `<workflows>`, etc.
   - Any element with descriptive content can be referenced
   - No special syntax required - natural language references work

**Benefits:**
- **DRY**: Define APIs, models, workflows once, reference everywhere
- **Consistency**: Changes to API definition automatically reflected in all references
- **Rich Context**: Claude gets complete information about referenced elements
- **Natural**: Use element names naturally in requests

**Registry Structure:**
```typescript
interface ElementRegistry {
  [key: string]: {
    name: string;
    type: string; // 'api', 'model', 'workflow', 'component'
    location: string; // path in hierarchy
    content: string; // full content
    attributes: Record<string, string>;
  }
}

// Example registry entries:
{
  'fetch users': {
    name: 'fetch users',
    type: 'api',
    location: 'api > fetch users',
    content: 'fetches the users with orders\ncurl https://...',
    attributes: {}
  }
}
```

**Reference Detection Algorithm:**
```typescript
function detectReferences(userMessage: string, registry: ElementRegistry) {
  const references = [];

  // Check if user message contains any registered element names
  for (const [name, definition] of Object.entries(registry)) {
    if (userMessage.toLowerCase().includes(name.toLowerCase())) {
      references.push(definition);
    }
  }

  return references;
}
```

**Enhanced Context with References:**
When Claude processes the button example, it receives:
```
Hierarchy: app > button
User request: "fetch users on click"

REFERENCED ELEMENTS:
1. "fetch users" (API endpoint defined in app > api):
   Description: fetches the users with orders
   Endpoint: curl https://api.example.com/users?with=orders -H "Authorization: Bearer token"

Your task: Implement a button that calls the "fetch users" API when clicked.
Use the endpoint details provided above.
```

### 4. Unified Claude Client (src/claude-client.ts)
**Responsibility:** Abstract interface for both CLI and API modes

**Key Functions:**
- `createClient(config)` - Factory method that returns CLI or API client based on config
- `sendPrompt(message, context)` - Unified interface for both modes
- `streamResponse(message, context)` - Stream responses regardless of mode

```typescript
interface ClaudeClient {
  sendPrompt(message: string, context: HierarchyContext): AsyncGenerator<string>;
  checkAvailable(): Promise<boolean>;
}

function createClient(config: Config): ClaudeClient {
  if (config.mode === 'cli') {
    return new ClaudeCLIClient();
  } else {
    return new ClaudeAPIClient(config.apiKey);
  }
}
```

### 5. Claude CLI Integration (src/claude-cli.ts)
**Responsibility:** Interface with Claude Code CLI via subprocess

**Key Functions:**
- `checkClaudeInstalled()` - Verify Claude Code CLI is available
- `sendPrompt(userMessage, context)` - Send prompt and stream response
- `buildPrompt()` - Create prompt with hierarchy context

**Implementation Approach:**

Use `child_process.spawn()` to run Claude Code as subprocess:

```typescript
import { spawn } from 'child_process';

async function* streamClaudeResponse(
  userMessage: string,
  context: HierarchyContext
): AsyncGenerator<string> {
  // Build prompt with automatic context
  const prompt = buildPromptWithContext(userMessage, context);

  // Spawn Claude Code in headless mode
  const claude = spawn('claude-code', [
    '--headless',
    '--prompt', prompt
  ]);

  // Stream stdout chunks
  for await (const chunk of claude.stdout) {
    yield chunk.toString('utf-8');
  }

  // Handle errors
  claude.stderr.on('data', (data) => {
    console.error('Claude CLI Error:', data.toString());
  });

  await new Promise((resolve) => claude.on('close', resolve));
}
```

### 6. Claude API Integration (src/claude-api.ts)
**Responsibility:** Interface with Anthropic API for pay-per-use mode

**Key Functions:**
- `checkAPIKey()` - Verify API key is configured
- `sendPrompt(userMessage, context)` - Send to API with streaming
- `handleRateLimits()` - Manage API rate limiting

**Implementation Approach:**

Use `@anthropic-ai/sdk` for direct API access:

```typescript
import Anthropic from '@anthropic-ai/sdk';

async function* streamClaudeResponse(
  userMessage: string,
  context: HierarchyContext
): AsyncGenerator<string> {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  // Build system prompt with hierarchy context
  const systemPrompt = buildSystemPrompt(context);

  // Stream from API
  const stream = await client.messages.stream({
    model: 'claude-opus-4-6',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userMessage
    }]
  });

  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' &&
        chunk.delta.type === 'text_delta') {
      yield chunk.delta.text;
    }
  }
}
```

**System Prompt Template (with Hierarchy + References):**
```
You are an AI assistant helping generate HTML/CSS code from natural language requests.

**Hierarchy Context:**
{hierarchy path, e.g., "app > sidebar > menu[name=settings]"}

**Parent Elements:**
{description of parent elements and their attributes}

**Referenced Elements:**
{if user message references other elements, include their full definitions}
Example:
- "fetch users" (API endpoint):
  Description: fetches the users with orders
  Endpoint: curl https://api.example.com/users?with=orders -H "Authorization: Bearer token"

**Previous Messages in this chat:**
{previous user-ai exchanges in same chat element}

**User Request:** {user message from XML}

Instructions:
1. Provide a brief acknowledgment (1 sentence)
2. Generate the necessary HTML/CSS code appropriate for this context
3. Consider the hierarchy - you're working on {specific context}
4. If referenced elements are provided, use their definitions to inform your implementation

Format your response concisely.
```

### 7. XML-ish Updater (src/updater.ts)
**Responsibility:** Insert AI responses into spec.html

**Key Functions:**
- `updateAIElement(parsedXML, path, response)` - Insert response text into `<ai>` element
- `rebuildXML(parsedXML)` - Convert JS object back to XML string
- `writeFile(content)` - Write updated XML to spec.html safely

**Challenges & Solutions:**

**Challenge 1:** Preserve XML-ish structure with malformed tags
- **Solution:** Use htmlparser2 for parsing (handles anything), cheerio for manipulation
- Use cheerio's `.html()` method to serialize back (preserves malformed tags reasonably)
- Accept that severe malformations may be auto-corrected

**Challenge 2:** Update specific element without corrupting structure
- **Solution:**
  1. Parse with htmlparser2 (creates DOM-like tree)
  2. Use cheerio selectors to find target `<ai>` element
  3. Update text content: `$(aiElement).text(response)`
  4. Serialize back to string with `$.html()`

**Challenge 3:** Prevent circular file watching triggers
- **Solution:** Use a flag:
  ```typescript
  let isUpdating = false;

  async function updateSpecFile(content: string) {
    isUpdating = true;
    try {
      await fs.writeFile('spec.html', content);
      await sleep(500); // Grace period
    } finally {
      isUpdating = false;
    }
  }
  ```

### 8. Code Generator (src/code-generator.ts)
**Responsibility:** Generate spec.code.html from AI responses

**Key Functions:**
- `extractCodeBlocks(response)` - Parse code from AI response
- `accumulateCode(existingCode, newCode, context)` - Build up implementation
- `generateHTML(codeMap)` - Render final spec.code.html
- `linkToSpec(element)` - Add comments linking code to spec elements

**Strategy:**
1. **Initial Generation:** Start with proper HTML5 template
2. **Code Extraction:** When AI responds, extract HTML/CSS code blocks (even from XML-ish responses)
3. **Code Mapping:** Map code to source element hierarchy in spec.html
   ```javascript
   codeMap = {
     'app.sidebar.menu[settings]': '<div class="menu settings" style="background: blue;">...</div>',
     'app.page': '<div class="page">...</div>'
   }
   ```
4. **Composition:** Build complete, valid HTML document:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <title>Generated from spec.html</title>
     <style>/* Accumulated CSS */</style>
   </head>
   <body>
     <!-- Generated from <menu> in spec.html -->
     <div class="menu">...</div>

     <!-- Generated from <page> in spec.html -->
     <div class="page">...</div>
   </body>
   </html>
   ```
5. **Hot Reload:** Optionally inject live-reload script for preview

**Future Enhancement:** Maintain bidirectional mapping (spec ↔ code) for debugging

## Implementation Workflow

### Startup Sequence
```
1. User runs: npx gui-spec-editor (or npm start)
2. Load configuration from .env or CLI args
   - MODE=cli or MODE=api
   - ANTHROPIC_API_KEY (if API mode)
3. Check Claude availability:
   - CLI mode: verify `claude-code` is installed
   - API mode: verify API key is set
4. Initialize file watcher on spec.html
5. Process existing spec.html (initial scan for empty <ai> elements)
6. Display: "Watching spec.html... (Mode: {cli|api})"
```

### Change Detection Flow
```
1. User edits spec.html (adds user message + empty <ai/>)
2. Chokidar detects change after 300ms debounce
3. Parser reads and parses XML-ish content (lenient)
4. Build element registry (scan entire spec for named elements)
5. Detect new chat patterns (empty <ai> elements)
6. For each pattern:
   a. Extract user message
   b. Build automatic context from hierarchy (parent chain, attributes)
   c. Detect references in user message (match against registry)
   d. Include referenced element definitions in context
   e. Create unified client (CLI or API based on config)
   f. Send prompt with full context (hierarchy + references)
   g. Stream response chunks
   h. Accumulate full response
   i. Update <ai> element using cheerio
   j. Write back to spec.html (preserve structure)
   k. Extract code blocks and update spec.code.html (proper HTML)
7. Log completion to console
8. Continue watching
```

### Chat Pattern and Deduplication Strategy

**Pattern Detection:**
- **User message**: ANY tag name (except `<ai>`)
  - Can be self-closing: `<request/>`
  - Can have simple text: `<request>text</request>`
  - **Can have mixed content**: Text + nested tags + multiline
  - Example:
    ```xml
    <create button>
      description text
      <feature>responsive</feature>
      more text
    </create button>
    ```
  - System extracts entire content tree
- **AI response**: MUST be `<ai>` tag, and MUST be empty to trigger processing
- **Scope context**: Everything in the parent component where chat is added

**Deduplication:**
To avoid re-processing already-answered messages:
- **Rule:** Only process `<ai>` elements that are completely empty
- **Detection:**
  - `<ai/>` (self-closing) → empty ✓ (will process)
  - `<ai></ai>` (no text) → empty ✓ (will process)
  - `<ai>content</ai>` → filled ✗ (skip, already answered)
- **In-memory tracking:** Track processing patterns to prevent duplicates within same session

**Why this design:**
- Maximum flexibility for user messages (any semantic tag name)
- Clear distinction for AI responses (`<ai>` is reserved)
- Easy to see what's been processed (filled `<ai>` tags)
- Natural conversation flow in the XML

## NPM Package Setup

### package.json
```json
{
  "name": "gui-driven-spec",
  "version": "0.1.0",
  "description": "XML-ish code editor with AI integration (CLI and API modes)",
  "main": "dist/index.js",
  "type": "module",
  "bin": {
    "gui-spec-editor": "./bin/gui-spec-editor.js"
  },
  "scripts": {
    "dev": "tsx src/index.ts",
    "dev:cli": "MODE=cli tsx src/index.ts",
    "dev:api": "MODE=api tsx src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "watch": "tsx watch src/index.ts"
  },
  "dependencies": {
    "chokidar": "^5.0.0",
    "htmlparser2": "^9.1.0",
    "cheerio": "^1.0.0",
    "lodash.debounce": "^4.0.8",
    "dotenv": "^16.4.7",
    "@anthropic-ai/sdk": "^0.32.1"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "tsx": "^4.0.0",
    "@types/node": "^20.0.0",
    "@types/lodash.debounce": "^4.0.9"
  },
  "engines": {
    "node": ">=20.0.0"
  },
  "keywords": ["xml", "ai", "code-editor", "claude", "spec-driven", "hierarchy"],
  "license": "MIT"
}
```

### Starter Script (bin/gui-spec-editor.js)
```javascript
#!/usr/bin/env node

import { spawn } from 'child_process';
import { existsSync } from 'fs';
import { join } from 'path';
import { fileURLToPath } from 'url';
import dotenv from 'dotenv';

const __dirname = fileURLToPath(new URL('.', import.meta.url));

// Load .env if present
dotenv.config();

// Check for spec.html in current directory
const specPath = join(process.cwd(), 'spec.html');
if (!existsSync(specPath)) {
  console.error('❌ Error: spec.html not found in current directory');
  console.log('📝 Create a spec.html file to get started');
  console.log('Example:');
  console.log('  <app>');
  console.log('    <chat>');
  console.log('      <create a button/>');
  console.log('      <ai/>');
  console.log('    </chat>');
  console.log('  </app>');
  process.exit(1);
}

// Determine mode
const mode = process.env.MODE || 'cli';

if (mode === 'cli') {
  // Check if Claude Code is installed
  const checkClaude = spawn('claude-code', ['--version']);
  checkClaude.on('error', () => {
    console.error('❌ Error: Claude Code CLI not found');
    console.log('Please install Claude Code: https://code.claude.com');
    console.log('Or switch to API mode: MODE=api in .env');
    process.exit(1);
  });

  checkClaude.on('close', (code) => {
    if (code !== 0) {
      console.error('❌ Error: Claude Code CLI not working properly');
      process.exit(1);
    }
    startEditor('cli');
  });
} else if (mode === 'api') {
  // Check for API key
  if (!process.env.ANTHROPIC_API_KEY) {
    console.error('❌ Error: ANTHROPIC_API_KEY not found');
    console.log('Set ANTHROPIC_API_KEY in .env file');
    console.log('Or switch to CLI mode: MODE=cli in .env');
    process.exit(1);
  }
  startEditor('api');
} else {
  console.error('❌ Error: Invalid MODE in .env (use "cli" or "api")');
  process.exit(1);
}

function startEditor(mode) {
  console.log('🚀 Starting GUI-driven spec editor...');
  console.log(`📡 Mode: ${mode.toUpperCase()}`);
  console.log('👀 Watching spec.html for changes...');
  console.log('💡 Tip: Add a user message and empty <ai/> element to trigger AI');
  console.log('💡 Note: Tags can be freeform, hierarchy defines context\n');

  const editor = spawn('node', [join(__dirname, '../dist/index.js')], {
    stdio: 'inherit',
    cwd: process.cwd(),
    env: { ...process.env, MODE: mode }
  });

  editor.on('exit', (exitCode) => {
    process.exit(exitCode || 0);
  });
}
```

### TypeScript Configuration (tsconfig.json)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "node",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Key Design Decisions

### Decision 1: Dual Mode Support (CLI + API)
**Choice:** Support both Claude Code CLI and Anthropic API, configurable via .env
**Rationale:**
- User requested both modes for flexibility
- CLI mode: Uses Claude Pro subscription (no extra cost)
- API mode: Pay-per-use for users without subscriptions
- Unified interface abstracts the difference

**Trade-off:**
- More complex implementation (two integration paths)
- Need to maintain feature parity between modes
- But: Maximum flexibility for different user scenarios

### Decision 2: Batch Updates vs Streaming
**Choice:** Batch updates for MVP (accumulate full response)
**Rationale:**
- Simpler implementation
- Avoid complex file locking
- Prevent multiple file watcher triggers
- More reliable for initial version

**Trade-off:** User doesn't see real-time streaming, but gets cleaner experience

**Future:** Add incremental streaming by:
- Writing partial responses as they arrive
- Using file locking to prevent race conditions
- Showing progress indicator in terminal

### Decision 3: Lenient Parser
**Choice:** htmlparser2 + cheerio (jQuery-like API)
**Rationale:**
- User requested very lenient parsing (handles malformed, unclosed tags, spaces in names)
- htmlparser2 is designed for HTML (more forgiving than XML parsers)
- cheerio provides intuitive manipulation API
- Handles virtually any tag structure
- Round-trip capable with `.html()` serialization

**Trade-off:** May auto-correct severe malformations, but handles the 99% case of "XML-ish" hierarchies perfectly

### Decision 4: File Watching Strategy
**Choice:** chokidar with 300ms debounce
**Rationale:**
- Cross-platform reliability
- Handles rapid edits gracefully
- Battle-tested library
- Recommended best practice

**Trade-off:** Small delay before processing, but prevents multiple triggers on each keystroke

### Decision 5: Hierarchy = Automatic Context
**Choice:** Extract hierarchy position and automatically build Claude's context
**Rationale:**
- User requested automatic context from hierarchy
- Makes conversations more natural (don't repeat "this is for menu settings")
- Claude understands scope (e.g., styling a button inside a menu vs. page-level button)
- Reduces verbosity in user messages

**Trade-off:** Context building logic is more complex, but UX is dramatically better

### Decision 6: No Naming Conventions
**Choice:** Support any tag names (spaces, special chars, unclosed tags)
**Rationale:**
- User explicitly requested very lenient parsing
- Tags are just for hierarchy, not semantic validation
- More flexible and natural for users
- Reduces learning curve to zero

**Trade-off:** Can't rely on tag names for logic; must use structural patterns (empty `<ai>` elements). Enables maximum creative freedom

## Potential Challenges & Mitigations

### Challenge 1: Dual Mode Integration Complexity
**Issue:** Maintaining feature parity between CLI and API modes
**Mitigation:**
- Use unified interface (ClaudeClient abstraction)
- Share prompt building logic
- Test both modes equally
- Document differences if any
- Provide clear mode selection in docs

### Challenge 2: Claude Code CLI Subprocess Management
**Issue:** Managing spawned processes, handling crashes, timeouts
**Mitigation:**
- Set timeout for Claude responses (e.g., 60 seconds)
- Gracefully handle process errors
- Retry failed requests (max 3 attempts)
- Log errors clearly for debugging
- Clean up zombie processes on shutdown

### Challenge 3: Circular File Watching
**Issue:** Writing to spec.html triggers watcher, potentially infinite loop
**Mitigation:**
- Use `isUpdating` flag to skip self-triggered changes
- Add 500ms grace period after writes
- Track file modification timestamps
- Use chokidar's `awaitWriteFinish` to ensure writes complete

### Challenge 4: Lenient Parsing Edge Cases
**Issue:** Extremely malformed structures might break parser
**Mitigation:**
- htmlparser2 handles 99% of cases
- Wrap parsing in try-catch with clear error messages
- Provide fallback: show malformed section to user
- Document known limitations
- Consider allowing users to escape problematic sections

### Challenge 5: Concurrent Empty <ai> Elements
**Issue:** Multiple empty `<ai>` elements in one file change
**Mitigation:**
- Process patterns sequentially (queue-based)
- Track which patterns are currently processing
- Update all patterns in a single file write operation
- Prevent parallel Claude processes for same file

### Challenge 6: Preserving XML-ish Formatting
**Issue:** Cheerio may normalize whitespace/indentation when serializing
**Mitigation:**
- cheerio generally preserves structure well
- Use `.html()` with options to minimize changes
- Document that some formatting normalization is expected
- Maintain consistent indentation (2-space)
- Consider storing original formatting metadata

### Challenge 7: Large AI Responses
**Issue:** Very long responses could take time to process
**Mitigation:**
- Stream chunks as they arrive (log progress)
- Set reasonable timeout (60s)
- Allow user to cancel with Ctrl+C
- Cache responses for undo functionality

### Challenge 8: Building Good Hierarchy Context
**Issue:** Generic tag names might not provide enough context
**Mitigation:**
- Include full recursive parent chain (e.g., "app[theme=dark] > sidebar[width=250px] > settings menu[position=top]")
- Extract and include ALL attributes at EVERY level of hierarchy
- **Recursive parent scope reference**: describe each parent's purpose and constraints
- Include sibling elements at current and parent levels for additional context
- Include previous chat messages in same scope
- Apply rules from .rules.html that match each hierarchy level
- Reference models from .models.html that relate to parent scopes
- Provide clear examples in system prompt
- Allow HTML comments for explicit hints

**Example Context Build:**
```
<app theme="dark">
  <dashboard layout="grid">
    <user-panel role="admin">
      <chat><add button/><ai/></chat>
    </user-panel>
  </dashboard>
</app>

Context sent to Claude:
"You are adding a button to a user-panel with role 'admin',
 which is inside a dashboard with layout 'grid',
 which is in the main app with theme 'dark'.

 Consider:
 - App scope: dark theme colors apply
 - Dashboard scope: grid layout affects positioning
 - User-panel scope: admin role may need special styling

 Apply rules for: app.dark-theme, dashboard.grid, panel.admin"
```

## Implementation Sequence

### Phase 1: Project Setup (Day 1)
- Create package.json with dependencies
- Set up TypeScript configuration
- Create project structure (src/, bin/)
- Initialize git repository
- Create .gitignore

### Phase 2: File Watching (Day 1)
- Implement watcher.ts with chokidar
- Test file change detection
- Add debouncing
- Add circular trigger prevention

### Phase 3: Lenient Parsing (Day 2)
- Implement parser.ts with htmlparser2 + cheerio
- Test with malformed structures (unclosed tags, spaces in names)
- Implement chat pattern detection algorithm
- Implement element registry builder (scan for named elements)
- Implement reference detection (match user messages against registry)
- Test with various XML-ish structures from spec.html
- Build automatic hierarchy context extraction
- Test context building with nested structures
- Test cross-referencing (define API, reference it from button)

### Phase 4: Configuration System (Day 3)
- Implement config.ts for .env loading
- Support MODE=cli or MODE=api
- Test configuration loading
- Add validation and error messages

### Phase 5: Claude Integration (Days 4-5)
- Implement claude-client.ts (unified interface)
- Implement claude-cli.ts (CLI mode)
- Implement claude-api.ts (API mode)
- Test subprocess spawning (CLI)
- Test API streaming
- Handle errors and timeouts for both modes
- Build system prompts with hierarchy context
- Test context injection

### Phase 6: XML-ish Updates (Day 6)
- Implement updater.ts
- Test inserting responses into parsed XML
- Test XML rebuilding
- Verify structure preservation
- Add file writing with safety checks

### Phase 6: Code Generation (Day 5)
- Implement code-generator.ts
- Extract code blocks from responses
- Build spec.code.html structure
- Test end-to-end generation
- Add HTML template with styling

### Phase 7: Main Orchestration (Day 6)
- Implement index.ts
- Connect all components
- Add error handling
- Add logging and status output
- Test full workflow

### Phase 8: CLI & Scripts (Day 7)
- Create bin/gui-spec-editor.js
- Add startup checks (Claude CLI, spec.html)
- Test installation with npm link
- Add helpful error messages
- Test on clean system

### Phase 9: Testing & Polish (Day 8)
- End-to-end testing with spec.html
- Test edge cases (empty file, malformed XML, no chat elements)
- Improve error messages
- Add debug logging
- Performance testing

### Phase 10: Documentation (Day 9)
- Write comprehensive README.md
- Add code comments
- Create example spec.html files
- Document Claude CLI requirements
- Add troubleshooting guide

## Critical Files to Modify/Create

1. **[package.json](package.json)** - NPM package configuration with dependencies
2. **[src/parser.ts](src/parser.ts)** - Core chat pattern detection and context building
3. **[src/claude-cli.ts](src/claude-cli.ts)** - Claude Code CLI subprocess management
4. **[src/watcher.ts](src/watcher.ts)** - File watching with circular trigger prevention
5. **[src/updater.ts](src/updater.ts)** - XML modification and safe file writing
6. **[src/index.ts](src/index.ts)** - Main orchestrator connecting all components
7. **[bin/gui-spec-editor.js](bin/gui-spec-editor.js)** - CLI entry point with checks

## Verification Plan

### End-to-End Test
1. Create fresh spec.html:
   ```xml
   <app>
     <chat>
       <create a blue button/>
       <ai/>
     </chat>
   </app>
   ```
2. Run `npx gui-spec-editor` or `npm start`
3. Verify watcher starts and processes existing file
4. Check that `<ai/>` element gets filled with response
5. Verify spec.code.html is generated with HTML/CSS
6. Edit spec.html again (add another chat)
7. Verify real-time processing
8. Check console logs for clarity

### Edge Cases to Test
- Empty spec.html
- Malformed XML (unclosed tags)
- No `<ai/>` elements
- Multiple `<ai/>` elements in one change
- Very long AI responses
- Claude CLI not installed
- Claude CLI timeout/error
- Rapid file edits (debouncing)
- Large XML files (performance)

### Success Criteria
- ✅ File watcher detects changes reliably
- ✅ Chat patterns detected correctly
- ✅ Claude Code CLI integration works
- ✅ Responses inserted into correct `<ai>` elements
- ✅ XML structure preserved
- ✅ spec.code.html generated accurately
- ✅ No circular file watching loops
- ✅ Graceful error handling
- ✅ Clear console feedback
- ✅ Works as NPM package (`npm link`, `npx`)

## Future Enhancements (Post-MVP)

1. **Visual Designer** - Drag-and-drop UI for building XML structure
2. **VS Code Extension** - IDE integration with syntax highlighting and live preview
3. **Multi-file Support** - Work with multiple .spec.html files
4. **Undo/Redo** - Track changes and enable rollback
5. **Live Preview** - Auto-refresh browser showing spec.code.html
6. **Incremental Streaming** - Show AI responses in real-time
7. **Code Diffing** - Show what changed in spec.code.html
8. **Templates** - Pre-built XML templates for common patterns
9. **Plugin System** - Custom parsers and generators
10. **Collaborative Mode** - Multiple users editing same spec

---

## Summary

This implementation creates a simple but powerful XML-based code editor that leverages Claude Code's subscription mode to enable natural language UI development. Users describe what they want in `<chat>` elements using freeform XML tags, and Claude generates the actual HTML/CSS implementation in a separate file.

The architecture is intentionally minimal for MVP:
- **3 core files:** watcher, parser, claude-cli integration
- **1 orchestrator:** main index.ts
- **2 output files:** spec.html (intent), spec.code.html (implementation)
- **Simple CLI:** One command to start watching

This foundation can grow into a comprehensive visual designer and VS Code extension in future iterations.
