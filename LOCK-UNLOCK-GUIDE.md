# Lock/Unlock Feature Guide

## Overview

The **lock/unlock** feature gives you fine-grained control over which components get regenerated when you add `gen` or `retry` to a parent element.

## Basic Concepts

### Lock Attribute
- **Purpose**: Prevents an element AND all its children from being regenerated
- **Use Case**: Preserve already-implemented components when regenerating a parent
- **Syntax**: `<element lock="">`

### Unlock Attribute
- **Purpose**: Overrides a parent's `lock`, allowing a specific child to be regenerated
- **Use Case**: Lock most of a component but regenerate only specific parts
- **Syntax**: `<element unlock="">`

## How It Works

### Default Behavior (No Lock)
By default, all nested components are regenerated:

```xml
<website gen="">
  <nav-menu>...</nav-menu>        <!-- Will be regenerated -->
  <pages>
    <home>...</home>              <!-- Will be regenerated -->
    <gallery>...</gallery>        <!-- Will be regenerated -->
  </pages>
</website>
```

Result: ALL components (nav-menu, pages, home, gallery) are regenerated.

### Lock Specific Children
Prevent specific children from being regenerated:

```xml
<website gen="">
  <nav-menu lock="">...</nav-menu>  <!-- LOCKED: Will NOT be regenerated -->
  <pages>
    <home>...</home>                <!-- Will be regenerated -->
    <gallery>...</gallery>          <!-- Will be regenerated -->
  </pages>
</website>
```

**Key Point**: `lock=""` on a child element prevents that child from being regenerated.

Result:
- ✅ `nav-menu` is preserved (has lock)
- ✅ `pages`, `home`, and `gallery` are regenerated

### Lock All Children by Default
Use `lock=""` WITH `gen=""` on the parent to lock all children by default:

```xml
<website lock="" gen="">
  <nav-menu>...</nav-menu>          <!-- LOCKED by parent -->
  <pages>
    <home>...</home>                <!-- LOCKED by parent -->
    <gallery>...</gallery>          <!-- LOCKED by parent -->
  </pages>
</website>
```

**Key Point**: `lock=""` on the element with `gen=""` locks ALL its children.

Result:
- ✅ ALL children are preserved (locked by parent)

### Unlock Specific Children
Use `unlock=""` to override parent's lock:

```xml
<website lock="" gen="">
  <nav-menu unlock="">...</nav-menu>  <!-- UNLOCKED: Will be regenerated -->
  <pages>                             <!-- LOCKED by parent -->
    <home>...</home>                  <!-- LOCKED by parent -->
    <gallery>...</gallery>            <!-- LOCKED by parent -->
  </pages>
</website>
```

**Key Point**: `unlock=""` on a child overrides the parent's `lock=""`.

Result:
- ✅ `nav-menu` is regenerated (has unlock)
- ✅ `pages`, `home`, and `gallery` are preserved (locked by parent)

### Nested Lock/Unlock
Lock parent, unlock one child, but lock grandchildren:

```xml
<website lock="" gen="">
  <nav-menu unlock="">...</nav-menu>
  <pages unlock="">
    <home lock="">...</home>          <!-- LOCKED: Won't be regenerated -->
    <gallery>...</gallery>            <!-- Will be regenerated -->
  </pages>
</website>
```

Result:
- ✅ `nav-menu` is regenerated (unlocked)
- ✅ `pages` is regenerated (unlocked)
- ✅ `home` is preserved (locked)
- ✅ `gallery` is regenerated (not locked)

## Common Use Cases

### 1. Regenerate Layout, Preserve Navigation
```xml
<app gen="">
  <sidebar lock="">
    <!-- Already implemented navigation stays the same -->
  </sidebar>
  <main-content>
    <!-- Only main content is regenerated -->
  </main-content>
</app>
```

### 2. Tweak One Page, Keep Others
```xml
<website gen="">
  <pages lock="">
    <home>...</home>            <!-- Locked -->
    <about unlock="">...</about> <!-- Regenerate only About page -->
    <contact>...</contact>       <!-- Locked -->
  </pages>
</website>
```

### 3. Update Theme, Preserve Components
```xml
<app theme="dark" gen="">
  <theme-toggle lock="">...</theme-toggle>
  <nav-menu lock="">...</nav-menu>
  <content>
    <!-- Content regenerated with new theme -->
  </content>
</app>
```

### 4. Iterative Development
Start broad, then lock what works:

**Step 1**: Generate everything
```xml
<dashboard gen="">
  <header>...</header>
  <sidebar>...</sidebar>
  <main>...</main>
</dashboard>
```

**Step 2**: Lock header (it's perfect), regenerate the rest
```xml
<dashboard retry="">
  <header lock="">...</header>    <!-- Keep this -->
  <sidebar>...</sidebar>          <!-- Regenerate -->
  <main>...</main>                <!-- Regenerate -->
</dashboard>
```

**Step 3**: Lock header and sidebar, regenerate only main
```xml
<dashboard retry="">
  <header lock="">...</header>    <!-- Keep -->
  <sidebar lock="">...</sidebar>  <!-- Keep -->
  <main>...</main>                <!-- Regenerate -->
</dashboard>
```

## Visual Prompt Display

When you use lock/unlock, the AI sees clear markers:

```
NESTED COMPONENTS TO IMPLEMENT:

1. <nav-menu> [LOCKED - DO NOT REGENERATE]
   ⚠️ This component is LOCKED - keep existing implementation, do NOT regenerate

2. <pages>
   Contains nested components:
     1. <home>
        Attributes:
          - content: classic home

     2. <gallery> [LOCKED - DO NOT REGENERATE]
        ⚠️ This component is LOCKED - keep existing implementation, do NOT regenerate
        Attributes:
          - layout: grid
```

The AI will:
- ✅ Skip regenerating `nav-menu` (locked)
- ✅ Regenerate `pages` parent container
- ✅ Regenerate `home` page
- ✅ Skip regenerating `gallery` (locked)

## Important Notes

1. **Lock Inheritance**: Children of locked elements are also locked unless they have `unlock`
2. **Unlock Scope**: `unlock` only affects the specific element it's on, not its children
3. **Both Attributes**: You can use both `lock` and `unlock` on siblings within the same parent
4. **With Gen/Retry**: Lock/unlock works with both `gen` and `retry` attributes

## Schema Support

Both `lock` and `unlock` attributes are defined in `spec.xsd` and will show up in IDE autocomplete:

```xml
<element lock="">    <!-- Autocomplete available -->
<element unlock="">  <!-- Autocomplete available -->
```

Hover over the attributes in your IDE to see documentation.

## Examples

See the examples below for practical demonstrations.
