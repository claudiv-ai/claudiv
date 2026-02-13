# Lock/Unlock Update - Correct Behavior

## What Changed

Fixed the lock/unlock implementation to work correctly:

### 1. Lock/Unlock Apply to CHILDREN (Not the Element Itself)

**Before (incorrect)**: Lock prevented the element itself from being regenerated
**After (correct)**: Lock prevents the element's CHILDREN from being regenerated

### 2. Lock/Unlock Can Coexist with Gen/Retry/Undo

**Before (incorrect)**: Couldn't use lock/unlock with action attributes
**After (correct)**: Can use them together on the same element

## Usage Patterns

### Pattern 1: Lock Specific Children

```xml
<website gen="">
  <nav-menu lock="">
    <!-- lock="" on child prevents this child from being regenerated -->
  </nav-menu>
  <content>
    <!-- No lock, will be regenerated -->
  </content>
</website>
```

**Result**: nav-menu preserved, content regenerated

### Pattern 2: Lock All Children by Default

```xml
<website lock="" gen="">
  <!-- lock="" WITH gen="" on same element locks all children -->
  <nav-menu>Locked by parent</nav-menu>
  <content>Locked by parent</content>
  <footer>Locked by parent</footer>
</website>
```

**Result**: All children are locked by default

### Pattern 3: Unlock Specific Child

```xml
<website lock="" gen="">
  <nav-menu unlock="">
    <!-- unlock="" overrides parent lock -->
    Regenerate only navigation
  </nav-menu>
  <content>Locked by parent</content>
</website>
```

**Result**: Only nav-menu regenerated, content preserved

### Pattern 4: Retry with Unlock

```xml
<website lock="">
  <nav-menu unlock="" retry="">
    <!-- unlock and retry coexist on same element -->
    Retry just the navigation
  </nav-menu>
</website>
```

**Result**: Only nav-menu regenerated

## Key Points

✅ **Attributes can be in any order** - not required to be last
✅ **Multiple control attributes work together** - lock/unlock + gen/retry/undo
✅ **Control applies to children** - lock/unlock control what happens to nested elements
✅ **Natural nesting** - Lock parent, unlock specific children as needed

## Implementation Details

### Code Changes

1. **[src/index.ts](src/index.ts:294-360)** - Updated `extractNestedSpecifications`
   - Checks if root element (with gen/retry/undo) has `lock` attribute
   - If yes, all children are locked by default
   - Children with `unlock` override parent lock

2. **[spec.xsd](spec.xsd:21-30)** - Updated documentation
   - Clarified that lock/unlock control children
   - Noted that they can coexist with gen/retry/undo

3. **[LOCK-UNLOCK-GUIDE.md](LOCK-UNLOCK-GUIDE.md)** - Updated examples
   - All examples now show correct usage
   - Emphasis on lock/unlock controlling children

4. **[.development/lock-unlock-examples.html](.development/lock-unlock-examples.html)** - Updated examples
   - Shows coexistence patterns
   - Clear comments explaining behavior

## Examples

See updated examples in:
- [LOCK-UNLOCK-GUIDE.md](LOCK-UNLOCK-GUIDE.md) - Complete guide
- [.development/lock-unlock-examples.html](.development/lock-unlock-examples.html) - Practical examples

## Testing

Build completed successfully. Try these patterns in your spec.html:

```xml
<!-- Test 1: Lock one child -->
<test-1 gen="">
  <header lock="">Keep this</header>
  <content>Regenerate this</content>
</test-1>

<!-- Test 2: Lock all children, unlock one -->
<test-2 lock="" gen="">
  <header unlock="">Only regenerate this</header>
  <content>Keep this</content>
  <footer>Keep this too</footer>
</test-2>

<!-- Test 3: Retry with unlock -->
<parent lock="">
  <child unlock="" retry="">
    Retry only this child
  </child>
</parent>
```
