# Attribute-Based Syntax Implementation

## Overview

The system now uses **attributes** (`gen`, `retry`, `undo`) instead of the old `<chat>` wrapper + empty `<ai/>` pattern.

## New Syntax

### 1. Generate (gen)

```xml
<!-- Simple: content-based -->
<create-button gen>Make a blue button</create-button>

<!-- Attribute-based: structured specs -->
<my-button
  title="Get Lucky Number"
  color="blue"
  size="large"
  click="show message"
  gen />

<!-- Mixed: attributes + content -->
<login-form username="email" password="required" gen>
  Add validation and error messages
</login-form>
```

**After processing:**
- `gen` attribute is removed
- `<ai>` child element is added with response

```xml
<my-button title="Get Lucky Number" color="blue" size="large" click="show message">
  <ai>
    <changes>Created interactive button</changes>
    <details>...</details>
  </ai>
</my-button>
```

### 2. Retry (retry)

```xml
<my-button color="pink" retry>Change to pink button
  <ai>Previous response...</ai>
</my-button>
```

Regenerates the response with the same spec.

### 3. Undo (undo)

```xml
<my-button undo>Revert changes
  <ai>Previous response...</ai>
</my-button>
```

Reverts to the previous implementation.

### 4. Lock (lock)

```xml
<website gen="">
  <nav-menu lock="">
    <!-- This component will NOT be regenerated -->
    <ai>Previous implementation...</ai>
  </nav-menu>
  <main-content>
    <!-- This will be regenerated -->
  </main-content>
</website>
```

Prevents an element and all its children from being regenerated when a parent has `gen` or `retry`.

### 5. Unlock (unlock)

```xml
<website lock="" gen="">
  <header unlock="">
    <!-- This WILL be regenerated (overrides parent lock) -->
  </header>
  <sidebar>
    <!-- This will NOT be regenerated (locked by parent) -->
  </sidebar>
</website>
```

Overrides a parent's `lock` attribute, allowing a specific child to be regenerated.

**Lock/Unlock Pattern:**
- `lock` on parent = lock everything by default
- `unlock` on child = override parent lock for that specific child
- Useful for iterative development and selective regeneration

See [LOCK-UNLOCK-GUIDE.md](LOCK-UNLOCK-GUIDE.md) for detailed examples.

## Key Features

### Freeform Tag Names
- **Any tag name works** - use whatever makes sense for organization
- Tag names act as **semantic headers** that help:
  - You organize your thinking
  - AI understand the structure and intent

```xml
<create-responsive-navbar gen>...</create-responsive-navbar>
<feature-panel gen>...</feature-panel>
<user-dashboard gen>...</user-dashboard>
```

### Attributes = Specifications
- Attributes are **part of the specification**
- They're included in the prompt sent to AI
- Use them for structured data

```xml
<button
  title="Submit Form"
  color="green"
  size="large"
  icon="checkmark"
  validation="required fields"
  gen />
```

AI receives:
```
Element: button
Specifications:
  title: Submit Form
  color: green
  size: large
  icon: checkmark
  validation: required fields
```

### Content = Description
- Text content and nested elements provide additional context
- Mix freeform text with structured nested tags

```xml
<feature gen>
  Create a dashboard with real-time updates
  <charts>Line chart for activity over time</charts>
  <stats>User stats: posts, followers, engagement</stats>
  <refresh>Auto-refresh every 30 seconds</refresh>
</feature>
```

## No More `<chat>` Wrapper!

Old syntax:
```xml
<button>
  <chat>
    <make-it-blue/>
    <ai/>
  </chat>
</button>
```

New syntax:
```xml
<button gen>make it blue</button>
```

Much cleaner and more intuitive!

## Files Modified

### Core Implementation
1. **src/types.ts** - Updated ChatPattern interface
   - Added `action`, `element`, `elementName`, `specAttributes`
   - Removed old `aiElement` field

2. **src/parser.ts** - New detection logic
   - Detects elements with `gen`/`retry`/`undo` attributes
   - Extracts tag names as semantic headers
   - Extracts attributes as specifications
   - Extracts content as user message

3. **src/updater.ts** - Updated response handling
   - `updateElementWithResponse()` - removes action attribute, adds `<ai>` child
   - Preserves all other attributes

4. **src/index.ts** - Updated processing
   - `buildFullPrompt()` - combines element name, attributes, and content
   - Removed old `autoCreateElement` logic
   - Updated startup tips

5. **src/dev-server.ts** - Fixed TypeScript types
   - Added proper type annotations for Vite plugin

### Schema & Documentation
6. **spec.xsd** - Updated XML Schema
   - Added `actionAttributes` group
   - Documented `gen`, `retry`, `undo` attributes
   - Updated element definitions

7. **.development/new-syntax-examples.html** - Usage examples
   - Complete examples of all patterns
   - Shows before/after states

8. **ATTRIBUTE-SYNTAX.md** - This file (documentation)

## How It Works

1. **User adds `gen` attribute:**
   ```xml
   <create-button color="blue" gen>Make a button</create-button>
   ```

2. **System detects the pattern:**
   - Finds element with `gen` attribute
   - Extracts element name: "create-button"
   - Extracts spec attributes: `{color: "blue"}`
   - Extracts user message: "Make a button"

3. **Builds prompt:**
   ```
   Element: create-button
   Specifications:
     color: blue
   Description:
   Make a button
   ```

4. **Sends to Claude** with full hierarchy context

5. **Receives response** and updates element:
   - Removes `gen` attribute
   - Adds `<ai>` child with response
   ```xml
   <create-button color="blue">Make a button
     <ai>
       <changes>Created blue button</changes>
       <details>...</details>
     </ai>
   </create-button>
   ```

## Benefits

✅ **Simpler** - No `<chat>` wrapper needed
✅ **Cleaner** - Attributes are self-documenting
✅ **Flexible** - Any tag name, any attributes, any content
✅ **Structured** - Attributes provide structured specifications
✅ **Semantic** - Tag names organize thinking and help AI
✅ **Natural** - More XML-like, follows standard patterns

## Testing

Build the project:
```bash
npm run build
```

Run the development server:
```bash
npm run dev
# or
npm run dev:cli
# or
npm run dev:api
```

Test with a simple example in spec.html:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<app>
  <test-button color="purple" size="large" gen>
    Create a purple button with text "Test Button"
  </test-button>
</app>
```

The system will:
1. Detect the `gen` attribute
2. Remove it after processing
3. Add `<ai>` child with response
4. Generate spec.code.html with the implementation

## Next Steps

The attribute-based syntax is now fully implemented! You can:
- Test with various patterns
- Use attributes for structured specifications
- Mix attributes with content for rich descriptions
- Use any tag names for semantic organization
