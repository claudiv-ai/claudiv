# GUI-Driven Spec - Features Summary

## Core Features

### 1. Attribute-Based Syntax
Use simple attributes on ANY element to trigger AI actions:

```xml
<element gen="">Generate new implementation</element>
<element retry="">Regenerate with same spec</element>
<element undo="">Revert previous change</element>
```

See: [ATTRIBUTE-SYNTAX.md](ATTRIBUTE-SYNTAX.md)

### 2. Freeform Tag Names
Use any tag name that makes sense for your project:

```xml
<create-dashboard gen="">...</create-dashboard>
<feature-panel gen="">...</feature-panel>
<user-profile-card gen="">...</user-profile-card>
```

Tag names act as semantic headers that help organize thinking and guide the AI.

### 3. Nested Component Specifications
The AI automatically implements ALL nested components:

```xml
<website gen="">
  <nav-menu dock="left">...</nav-menu>
  <pages>
    <home content="classic">...</home>
    <gallery layout="grid">...</gallery>
  </pages>
</website>
```

Result: Complete implementation of website, nav-menu, pages, home, and gallery.

### 4. Lock/Unlock System 🆕
Fine-grained control over what gets regenerated:

```xml
<!-- Lock specific components -->
<website gen="">
  <nav-menu lock="">Preserve this</nav-menu>
  <content>Regenerate this</content>
</website>

<!-- Lock parent, unlock specific child -->
<website lock="" gen="">
  <header unlock="">Only regenerate header</header>
  <sidebar>Keep this locked</sidebar>
  <content>Keep this locked too</content>
</website>
```

**Use Cases:**
- Iterative development: Lock what works, regenerate what needs improvement
- Selective updates: Update one page while preserving others
- Theme changes: Update theme colors while preserving component structure

See: [LOCK-UNLOCK-GUIDE.md](LOCK-UNLOCK-GUIDE.md)

### 5. Attributes as Specifications
Attributes provide structured specifications to the AI:

```xml
<button
  title="Submit Form"
  color="green"
  size="large"
  icon="checkmark"
  validation="required fields"
  gen="">
</button>
```

The AI receives these as structured specifications and uses them to guide implementation.

### 6. XML Schema Support
IDE autocomplete for all attributes and elements:

- Install "Red Hat XML" extension in VS Code
- Press Ctrl+Space to see available attributes
- Hover over attributes to see documentation
- Real-time validation

See: [SCHEMA-GUIDE.md](SCHEMA-GUIDE.md)

### 7. Hot Reload
Browser automatically refreshes when code is regenerated:

```bash
npm run dev
# Opens browser at http://localhost:30006
# Auto-reloads on changes
```

### 8. Structured AI Responses
AI responses use semantic XML tags:

```xml
<ai>
  <changes>Brief description</changes>
  <details>
    <feature-name>Detail about feature</feature-name>
  </details>
  <summary>Overall summary</summary>
</ai>
```

## Workflow

### Basic Usage
1. Edit `spec.html` with your specifications
2. Add `gen=""` attribute to trigger generation
3. AI generates implementation
4. Review `spec.code.html` in browser
5. Iterate with `retry=""` or lock parts with `lock=""`

### Iterative Development
```xml
<!-- Step 1: Generate everything -->
<dashboard gen="">
  <header>...</header>
  <sidebar>...</sidebar>
  <main>...</main>
</dashboard>

<!-- Step 2: Lock header, regenerate rest -->
<dashboard retry="">
  <header lock="">...</header>
  <sidebar>...</sidebar>
  <main>...</main>
</dashboard>

<!-- Step 3: Lock header and sidebar, regenerate main -->
<dashboard retry="">
  <header lock="">...</header>
  <sidebar lock="">...</sidebar>
  <main>...</main>
</dashboard>
```

## Files Generated

- **spec.html** - Your specifications (what you edit)
- **spec.code.html** - Generated HTML/CSS (browser-ready)
- **spec.xsd** - XML Schema for autocomplete
- **.vscode/settings.json** - IDE configuration

## Commands

```bash
# Development mode (with hot reload)
npm run dev

# Build TypeScript
npm run build

# CLI mode (Claude Code subscription)
npm run dev:cli

# API mode (pay-per-use)
npm run dev:api
```

## Configuration

Create `.env` file:
```env
MODE=cli                    # or "api"
ANTHROPIC_API_KEY=sk-...   # for API mode
```

## Documentation

- [ATTRIBUTE-SYNTAX.md](ATTRIBUTE-SYNTAX.md) - Complete syntax reference
- [LOCK-UNLOCK-GUIDE.md](LOCK-UNLOCK-GUIDE.md) - Lock/unlock patterns and examples
- [SCHEMA-GUIDE.md](SCHEMA-GUIDE.md) - XML Schema autocomplete setup
- [.development/lock-unlock-examples.html](.development/lock-unlock-examples.html) - Practical examples

## Examples

See `.development/` folder for comprehensive examples of all features.

## Benefits

✅ **Lower cognitive load** - Describe intent, not implementation
✅ **Conversational development** - Natural language specifications
✅ **Iterative workflow** - Lock what works, regenerate what needs improvement
✅ **Fast iteration** - Hot reload shows changes instantly
✅ **Fine-grained control** - Lock/unlock for selective regeneration
✅ **IDE support** - Autocomplete and validation
✅ **Version control friendly** - Readable XML specifications
