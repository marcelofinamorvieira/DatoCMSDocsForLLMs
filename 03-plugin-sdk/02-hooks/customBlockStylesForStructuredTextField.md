# customBlockStylesForStructuredTextField Hook

## Purpose

The `customBlockStylesForStructuredTextField` hook allows plugins to define custom styling options for block elements within Structured Text fields. This enables content editors to apply predefined styles to paragraphs, headings, lists, and other block-level elements, enhancing the visual formatting capabilities of the Structured Text editor.

## Signature

```typescript
customBlockStylesForStructuredTextField(
  field: Field,
  ctx: CustomBlockStylesForStructuredTextFieldCtx
): StructuredTextCustomBlockStyle[] | undefined
```

## Context Properties

The hook receives a context object with base properties plus field-specific information:

- All standard context properties (`plugin`, `currentUser`, `site`, etc.)
- `itemType`: ItemType - The model containing the Structured Text field

## Parameters

- `field`: Field - The Structured Text field for which to provide custom styles

## Return Value

The hook should return an array of `StructuredTextCustomBlockStyle` objects or `undefined`:

```typescript
type StructuredTextCustomBlockStyle = {
  id: string;                          // Unique identifier for the style
  node: BlockNodeTypeWithCustomStyle;  // Block type ('paragraph', 'heading', 'list', etc.)
  label: string;                       // Display name shown to editors
  appliedStyle: React.CSSProperties;   // CSS styles applied in the editor
  rank?: number;                       // Priority when multiple styles exist
}
```

## Supported Block Node Types

The following block node types support custom styles:
- `'paragraph'` - Regular text paragraphs
- `'heading'` - All heading levels (h1-h6)
- `'list'` - Both ordered and unordered lists
- `'blockquote'` - Blockquote elements
- `'code'` - Code blocks

## Use Cases

### 1. Typography Styles for Articles

Provide various text styles for article content:

```typescript
customBlockStylesForStructuredTextField(
  field: Field,
  ctx: CustomBlockStylesForStructuredTextFieldCtx
): StructuredTextCustomBlockStyle[] | undefined {
  // Only apply to article content fields
  if (ctx.itemType.attributes.api_key !== 'article' || 
      field.attributes.api_key !== 'content') {
    return undefined;
  }

  return [
    {
      id: 'lead-paragraph',
      node: 'paragraph',
      label: 'Lead Paragraph',
      appliedStyle: {
        fontSize: '1.25em',
        lineHeight: '1.6',
        fontWeight: '300',
        color: '#333'
      },
      rank: 1
    },
    {
      id: 'drop-cap',
      node: 'paragraph',
      label: 'Drop Cap',
      appliedStyle: {
        '&::first-letter': {
          fontSize: '3em',
          float: 'left',
          lineHeight: '1',
          marginRight: '0.1em',
          fontWeight: 'bold'
        }
      },
      rank: 2
    },
    {
      id: 'highlighted',
      node: 'paragraph',
      label: 'Highlighted',
      appliedStyle: {
        backgroundColor: '#fff3cd',
        padding: '12px 16px',
        borderLeft: '4px solid #ffc107',
        marginLeft: '0'
      },
      rank: 3
    },
    {
      id: 'chapter-heading',
      node: 'heading',
      label: 'Chapter Heading',
      appliedStyle: {
        fontSize: '2.5em',
        fontWeight: '300',
        borderBottom: '2px solid #eee',
        paddingBottom: '0.3em',
        marginTop: '1.5em'
      },
      rank: 1
    }
  ];
}
```

### 2. Brand-Specific Styles

Apply company brand guidelines to content:

```typescript
customBlockStylesForStructuredTextField(
  field: Field,
  ctx: CustomBlockStylesForStructuredTextFieldCtx
): StructuredTextCustomBlockStyle[] | undefined {
  const brandColors = ctx.plugin.attributes.parameters.brandColors || {
    primary: '#0066cc',
    secondary: '#00a86b',
    accent: '#ff6b6b'
  };

  return [
    {
      id: 'brand-callout',
      node: 'paragraph',
      label: 'Brand Callout',
      appliedStyle: {
        backgroundColor: brandColors.primary + '10', // 10% opacity
        borderLeft: `4px solid ${brandColors.primary}`,
        padding: '16px',
        fontSize: '1.1em'
      },
      rank: 1
    },
    {
      id: 'success-message',
      node: 'paragraph',
      label: 'Success Message',
      appliedStyle: {
        backgroundColor: brandColors.secondary + '15',
        borderRadius: '4px',
        padding: '12px 16px',
        color: brandColors.secondary
      },
      rank: 2
    },
    {
      id: 'warning-message',
      node: 'paragraph',
      label: 'Warning Message',
      appliedStyle: {
        backgroundColor: brandColors.accent + '15',
        borderRadius: '4px',
        padding: '12px 16px',
        color: brandColors.accent,
        borderLeft: `3px solid ${brandColors.accent}`
      },
      rank: 3
    },
    {
      id: 'brand-heading',
      node: 'heading',
      label: 'Brand Heading',
      appliedStyle: {
        color: brandColors.primary,
        fontFamily: ctx.plugin.attributes.parameters.brandFont || 'inherit',
        letterSpacing: '-0.02em'
      },
      rank: 1
    }
  ];
}
```

### 3. Documentation Styles

Styles for technical documentation:

```typescript
customBlockStylesForStructuredTextField(
  field: Field,
  ctx: CustomBlockStylesForStructuredTextFieldCtx
): StructuredTextCustomBlockStyle[] | undefined {
  // Only for documentation models
  if (!['docs', 'api_reference', 'guide'].includes(ctx.itemType.attributes.api_key)) {
    return undefined;
  }

  return [
    {
      id: 'note',
      node: 'paragraph',
      label: 'Note',
      appliedStyle: {
        backgroundColor: '#e3f2fd',
        borderLeft: '4px solid #2196f3',
        padding: '12px 16px',
        fontStyle: 'italic',
        '&::before': {
          content: '"ℹ️ Note: "',
          fontWeight: 'bold'
        }
      },
      rank: 1
    },
    {
      id: 'warning',
      node: 'paragraph',
      label: 'Warning',
      appliedStyle: {
        backgroundColor: '#fff3e0',
        borderLeft: '4px solid #ff9800',
        padding: '12px 16px',
        '&::before': {
          content: '"⚠️ Warning: "',
          fontWeight: 'bold'
        }
      },
      rank: 2
    },
    {
      id: 'tip',
      node: 'paragraph',
      label: 'Pro Tip',
      appliedStyle: {
        backgroundColor: '#e8f5e9',
        borderLeft: '4px solid #4caf50',
        padding: '12px 16px',
        '&::before': {
          content: '"💡 Tip: "',
          fontWeight: 'bold'
        }
      },
      rank: 3
    },
    {
      id: 'code-filename',
      node: 'code',
      label: 'Code with Filename',
      appliedStyle: {
        position: 'relative',
        paddingTop: '2.5em',
        '&::before': {
          content: 'attr(data-filename)',
          position: 'absolute',
          top: '0',
          left: '0',
          right: '0',
          padding: '0.5em 1em',
          backgroundColor: '#333',
          color: '#fff',
          fontSize: '0.875em'
        }
      },
      rank: 1
    }
  ];
}
```

### 4. E-commerce Product Descriptions

Styles for product content:

```typescript
customBlockStylesForStructuredTextField(
  field: Field,
  ctx: CustomBlockStylesForStructuredTextFieldCtx
): StructuredTextCustomBlockStyle[] | undefined {
  if (ctx.itemType.attributes.api_key !== 'product') {
    return undefined;
  }

  return [
    {
      id: 'feature-list',
      node: 'list',
      label: 'Feature List',
      appliedStyle: {
        listStyle: 'none',
        paddingLeft: '0',
        '& li': {
          paddingLeft: '1.5em',
          position: 'relative',
          marginBottom: '0.5em',
          '&::before': {
            content: '"✓"',
            position: 'absolute',
            left: '0',
            color: '#4caf50',
            fontWeight: 'bold'
          }
        }
      },
      rank: 1
    },
    {
      id: 'specifications',
      node: 'list',
      label: 'Specifications',
      appliedStyle: {
        backgroundColor: '#f5f5f5',
        padding: '16px',
        borderRadius: '4px',
        listStyle: 'none',
        '& li': {
          padding: '8px 0',
          borderBottom: '1px solid #e0e0e0',
          display: 'flex',
          justifyContent: 'space-between'
        }
      },
      rank: 2
    },
    {
      id: 'price-callout',
      node: 'paragraph',
      label: 'Price Callout',
      appliedStyle: {
        fontSize: '1.5em',
        fontWeight: 'bold',
        color: '#d32f2f',
        textAlign: 'center',
        padding: '16px',
        backgroundColor: '#ffebee',
        borderRadius: '4px'
      },
      rank: 1
    }
  ];
}
```

### 5. Multi-Language Content Styles

Different styles based on locale:

```typescript
customBlockStylesForStructuredTextField(
  field: Field,
  ctx: CustomBlockStylesForStructuredTextFieldCtx
): StructuredTextCustomBlockStyle[] | undefined {
  const currentLocale = ctx.ui.locale;
  
  // Base styles available for all locales
  const baseStyles: StructuredTextCustomBlockStyle[] = [
    {
      id: 'emphasis',
      node: 'paragraph',
      label: 'Emphasis',
      appliedStyle: {
        fontWeight: 'bold',
        fontSize: '1.1em'
      },
      rank: 1
    }
  ];

  // Locale-specific styles
  const localeStyles: Record<string, StructuredTextCustomBlockStyle[]> = {
    'ja': [
      {
        id: 'vertical-text',
        node: 'paragraph',
        label: '縦書き (Vertical)',
        appliedStyle: {
          writingMode: 'vertical-rl',
          textOrientation: 'upright',
          height: '200px'
        },
        rank: 10
      }
    ],
    'ar': [
      {
        id: 'rtl-emphasis',
        node: 'paragraph',
        label: 'تأكيد (Emphasis)',
        appliedStyle: {
          direction: 'rtl',
          textAlign: 'right',
          backgroundColor: '#f0f0f0',
          padding: '12px'
        },
        rank: 10
      }
    ],
    'zh': [
      {
        id: 'traditional-quote',
        node: 'blockquote',
        label: '古文引用 (Classical Quote)',
        appliedStyle: {
          fontFamily: 'serif',
          fontSize: '1.2em',
          textIndent: '2em',
          lineHeight: '2'
        },
        rank: 10
      }
    ]
  };

  return [
    ...baseStyles,
    ...(localeStyles[currentLocale] || [])
  ];
}
```

### 6. Conditional Styles Based on Field Configuration

Apply styles based on field parameters:

```typescript
customBlockStylesForStructuredTextField(
  field: Field,
  ctx: CustomBlockStylesForStructuredTextFieldCtx
): StructuredTextCustomBlockStyle[] | undefined {
  // Get field configuration from appearance parameters
  const fieldConfig = field.attributes.appearance.parameters;
  const enabledStyles = fieldConfig?.enabledStyles || [];
  
  const allStyles: Record<string, StructuredTextCustomBlockStyle> = {
    'highlight': {
      id: 'highlight',
      node: 'paragraph',
      label: 'Highlighted Text',
      appliedStyle: {
        backgroundColor: '#ffeb3b',
        padding: '2px 4px'
      },
      rank: 1
    },
    'pullquote': {
      id: 'pullquote',
      node: 'blockquote',
      label: 'Pull Quote',
      appliedStyle: {
        fontSize: '1.5em',
        fontStyle: 'italic',
        textAlign: 'center',
        margin: '2em 0',
        padding: '1em',
        borderTop: '3px solid #ccc',
        borderBottom: '3px solid #ccc'
      },
      rank: 1
    },
    'sidebar': {
      id: 'sidebar',
      node: 'paragraph',
      label: 'Sidebar Note',
      appliedStyle: {
        float: 'right',
        width: '40%',
        marginLeft: '20px',
        padding: '16px',
        backgroundColor: '#f9f9f9',
        borderLeft: '3px solid #ddd',
        fontSize: '0.9em'
      },
      rank: 2
    }
  };

  // Only return styles that are enabled for this field
  return enabledStyles
    .map(styleId => allStyles[styleId])
    .filter(Boolean);
}
```

## Best Practices

1. **Unique IDs**: Ensure each style has a unique ID within your plugin
2. **Clear Labels**: Use descriptive labels that help editors understand the style's purpose
3. **Visual Feedback**: Apply styles that are visible in the editor to help editors see the effect
4. **Performance**: Keep styles simple to avoid editor performance issues
5. **Rank Configuration**: Make rank configurable to avoid conflicts with other plugins
6. **Field Validation**: Check field type and configuration before applying styles
7. **Responsive Styles**: Consider how styles will appear on different screen sizes

## Working with CSS Properties

The `appliedStyle` property accepts standard React CSS properties:

```typescript
appliedStyle: {
  // Standard CSS properties
  fontSize: '1.2em',
  color: '#333',
  backgroundColor: 'rgba(0, 0, 0, 0.1)',
  
  // Nested selectors (limited support)
  '& li': {
    marginBottom: '0.5em'
  },
  
  // Pseudo-elements (limited support)
  '&::before': {
    content: '"→ "',
    fontWeight: 'bold'
  }
}
```

## Related Hooks

- **customMarksForStructuredTextField**: Define custom inline styles (bold, italic, etc.)
- **renderFieldExtension**: Create completely custom field editors
- **manualFieldExtensions**: Register custom field types

## Important Notes

- Custom styles only affect the editor display; rendering in your frontend is separate
- Not all CSS properties are supported in the editor environment
- Styles should enhance readability without interfering with editing
- The rank property determines the order in the style dropdown
- Return `undefined` to skip adding styles for specific fields
- Style IDs must be unique within your plugin to avoid conflicts