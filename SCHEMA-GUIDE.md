# XML Schema Autocomplete Guide

## Setup

### 1. Install VS Code Extension

Install the **Red Hat XML** extension:
- Open VS Code
- Go to Extensions (Ctrl+Shift+X)
- Search for "Red Hat XML"
- Install "XML Language Support by Red Hat"

### 2. Schema Files

The following files enable autocomplete:

- **spec.xsd** - XML Schema Definition with all available elements and attributes
- **.vscode/settings.json** - VS Code configuration for XML support
- **spec.html** - Updated with schema reference at the top

## Usage

### Autocomplete Features

1. **Element Suggestions**
   - Type `<` and press Ctrl+Space to see available elements
   - Suggestions include: `<app>`, `<button>`, `<component>`, `<form>`, etc.

2. **Attribute Suggestions** ⭐ NEW!
   - Inside an element tag, press Ctrl+Space to see available attributes
   - **Action attributes**: `gen`, `retry`, `undo` appear for ALL elements
   - Example: `<button |` → suggests `gen`, `retry`, `undo`, plus any element-specific attrs

3. **Documentation on Hover**
   - Hover over any element or attribute to see its documentation
   - Example: Hover over `gen` attribute to see: "Triggers AI to generate new implementation"

4. **Validation**
   - Red squiggly lines show schema validation errors
   - Helps catch typos and structural issues

### New Attribute-Based Syntax

#### Generate Pattern (gen)
```xml
<!-- Simple content-based -->
<create-button gen>Make a blue button</create-button>

<!-- Attribute-based specs -->
<my-button color="blue" size="large" gen />

<!-- Mixed -->
<login-form username="email" password="required" gen>
  Add validation and error messages
</login-form>
```

#### Retry Pattern
```xml
<my-button color="pink" retry>
  Change to pink button
  <ai>Previous response...</ai>
</my-button>
```

#### Undo Pattern
```xml
<my-button undo>
  Revert changes
  <ai>Previous response...</ai>
</my-button>
```

#### AI Response Structure (auto-generated)
```xml
<ai>
  <changes>Brief description</changes>
  <details>
    <feature-name>Detail about feature</feature-name>
  </details>
  <summary>Overall summary</summary>
</ai>
```

#### Common Components
- `<button>` - Button component
- `<component>` - Generic component container
- `<page>` - Page container
- `<sidebar>` - Sidebar navigation
- `<menu>` - Menu component
- `<form>` - Form component
- `<input>` - Input field
- `<header>` - Header component
- `<footer>` - Footer component
- `<card>` - Card container
- `<modal>` - Modal dialog

#### Special Commands
- `<undo/>` - Revert previous change
- `<ref>` - Reference external resources

### Extending the Schema

To add new elements to autocomplete:

1. Open `spec.xsd`
2. Add a new `<xs:element>` definition:
   ```xml
   <xs:element name="your-element">
     <xs:annotation>
       <xs:documentation>Description shown on hover</xs:documentation>
     </xs:annotation>
     <xs:complexType>
       <xs:sequence>
         <xs:any minOccurs="0" maxOccurs="unbounded" processContents="lax"/>
       </xs:sequence>
       <xs:attribute name="your-attribute" type="xs:string"/>
     </xs:complexType>
   </xs:element>
   ```
3. Save the file - autocomplete will update immediately

## Troubleshooting

**Autocomplete not working?**
1. Check that Red Hat XML extension is installed
2. Verify spec.html has the XML declaration and schema reference
3. Restart VS Code
4. Check the XML language is active (bottom-right status bar should show "XML")

**Validation errors?**
- The schema is intentionally flexible (uses `xs:any` extensively)
- Most validation is for guidance, not strict enforcement
- You can still use freeform tag names - they'll just show as "not defined" (which is fine)

## Benefits

✅ **Faster Development** - Autocomplete reduces typing
✅ **Documentation** - Hover to see element descriptions
✅ **Fewer Typos** - Validation catches common mistakes
✅ **Discoverability** - Browse available elements easily
✅ **Consistent Structure** - Suggested elements follow conventions
