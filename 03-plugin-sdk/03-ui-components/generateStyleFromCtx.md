# generateStyleFromCtx

A utility function that converts a DatoCMS plugin context into CSS properties, applying theme colors and appropriate padding to match the DatoCMS interface.

## Function Signature

```typescript
function generateStyleFromCtx(
  ctx: ImposedSizePluginFrameCtx<any, {}, {}>,
  noBodyPadding?: boolean
): React.CSSProperties
```

## Parameters

- **`ctx`** - The plugin context object containing theme and padding information
  - Must include `theme` object with color properties
  - Must include `bodyPadding` array when padding is applied
  
- **`noBodyPadding`** (optional) - Boolean flag to disable body padding
  - Default: `false`
  - Set to `true` for full-width layouts

## Return Value

Returns a `React.CSSProperties` object containing:
- Padding values (unless disabled)
- CSS custom properties for all theme colors
- RGB component variants for transparency support

## Theme Properties

The function generates CSS variables from these theme properties:

```typescript
interface Theme {
  primaryColor: string;
  accentColor: string;
  semiTransparentAccentColor: string;
  lightColor: string;
  darkColor: string;
  // Additional colors from DatoCMS theme
}
```

Each color generates two CSS variables:
- `--{kebab-case-name}`: The full color value
- `--{kebab-case-name}-rgb-components`: RGB values for use with `rgba()`

## Usage Examples

### Basic Usage

```typescript
import { generateStyleFromCtx } from 'datocms-react-ui';

export default function MyPlugin(ctx: PluginContext) {
  const styles = generateStyleFromCtx(ctx);
  
  return (
    <div style={styles}>
      {/* Content automatically styled with theme colors and padding */}
      <h1>My Plugin</h1>
    </div>
  );
}
```

### Without Padding

```typescript
const styles = generateStyleFromCtx(ctx, true);
// Returns theme variables without padding
```

### Using Generated CSS Variables

```typescript
export default function MyPlugin(ctx: PluginContext) {
  const rootStyles = generateStyleFromCtx(ctx);
  
  return (
    <div style={rootStyles}>
      <button 
        style={{
          backgroundColor: 'var(--primary-color)',
          color: 'var(--light-color)',
          border: '1px solid var(--accent-color)'
        }}
      >
        Themed Button
      </button>
      
      <div 
        style={{
          // Using RGB components for transparency
          backgroundColor: 'rgba(var(--primary-color-rgb-components), 0.1)'
        }}
      >
        Semi-transparent background
      </div>
    </div>
  );
}
```

### Full Example with TypeScript

```typescript
import React from 'react';
import { generateStyleFromCtx } from 'datocms-react-ui';
import type { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

export default function FieldExtension(ctx: RenderFieldExtensionCtx) {
  const rootStyles = generateStyleFromCtx(ctx);
  
  // Custom styles using theme variables
  const customStyles: React.CSSProperties = {
    ...rootStyles,
    minHeight: '200px',
    display: 'flex',
    flexDirection: 'column',
    gap: '1rem'
  };
  
  return (
    <div style={customStyles}>
      <header 
        style={{
          borderBottom: '1px solid var(--light-color)',
          paddingBottom: '0.5rem'
        }}
      >
        <h2 style={{ color: 'var(--dark-color)' }}>
          Custom Field Editor
        </h2>
      </header>
      
      <main style={{ flex: 1 }}>
        {/* Field content */}
      </main>
      
      <footer 
        style={{
          borderTop: '1px solid var(--light-color)',
          paddingTop: '0.5rem',
          color: 'var(--semi-transparent-accent-color)'
        }}
      >
        <small>Powered by My Plugin</small>
      </footer>
    </div>
  );
}
```

## Generated CSS Variables

Example output for a typical context:

```css
{
  padding: '10px 20px 10px 20px',
  '--primary-color': '#1e88e5',
  '--primary-color-rgb-components': '30, 136, 229',
  '--accent-color': '#ff5722',
  '--accent-color-rgb-components': '255, 87, 34',
  '--semi-transparent-accent-color': 'rgba(255, 87, 34, 0.1)',
  '--semi-transparent-accent-color-rgb-components': '255, 87, 34',
  '--light-color': '#f5f5f5',
  '--light-color-rgb-components': '245, 245, 245',
  '--dark-color': '#212121',
  '--dark-color-rgb-components': '33, 33, 33'
}
```

## Best Practices

1. **Always apply to root element**: Apply the generated styles to your plugin's root element to ensure all child elements can access the CSS variables.

2. **Combine with custom styles**: Spread the generated styles into your custom style objects:
   ```typescript
   const styles = {
     ...generateStyleFromCtx(ctx),
     // Your custom styles
   };
   ```

3. **Use CSS variables for consistency**: Reference the generated CSS variables instead of hardcoding colors:
   ```typescript
   // Good
   style={{ color: 'var(--primary-color)' }}
   
   // Avoid
   style={{ color: '#1e88e5' }}
   ```

4. **Handle full-width layouts**: Use `noBodyPadding` for components that need edge-to-edge layouts:
   ```typescript
   // Full-width image gallery
   const styles = generateStyleFromCtx(ctx, true);
   ```

## Related

- [Context Object](../../01-core-concepts/context-object.md) - Understanding the context object
- [Theme Type](../../01-core-concepts/shared-types.md#theme-type) - Theme property definitions
- [Canvas Component](./Canvas.md) - Pre-styled container using this utility