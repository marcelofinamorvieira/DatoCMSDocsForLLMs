# CSS System and Theming

The DatoCMS Plugin SDK provides a comprehensive CSS system that ensures visual consistency between plugins and the main DatoCMS interface. This system includes CSS variables, theme utilities, typography, and spacing conventions.

## Core Concepts

### CSS Variables

The plugin SDK uses CSS custom properties (variables) for all styling values. These variables are automatically injected when using the Canvas component or generateStyleFromCtx utility.

### Theme Integration

Plugins receive theme information from the DatoCMS context and can adapt their appearance to match the current theme (light/dark) and brand colors.

### CSS Modules

React UI components use CSS Modules for style encapsulation, preventing style conflicts between plugins and the host application.

## CSS Variable Reference

### Color Variables

#### System Colors

```css
/* Text Colors */
--base-body-color: rgb(52, 54, 58);
--base-body-color-rgb-components: 52, 54, 58;
--light-body-color: rgb(132, 132, 132);
--light-body-color-rgb-components: 132, 132, 132;
--placeholder-body-color: rgb(198, 198, 198);
--placeholder-body-color-rgb-components: 198, 198, 198;

/* Background Colors */
--light-bg-color: rgb(245, 245, 245);
--light-bg-color-rgb-components: 245, 245, 245;
--lighter-bg-color: rgb(248, 248, 248);
--lighter-bg-color-rgb-components: 248, 248, 248;
--disabled-bg-color: rgb(237, 237, 237);
--disabled-bg-color-rgb-components: 237, 237, 237;

/* Border Colors */
--border-color: rgb(240, 240, 240);
--border-color-rgb-components: 240, 240, 240;
--darker-border-color: rgb(215, 215, 215);
--darker-border-color-rgb-components: 215, 215, 215;

/* Status Colors */
--alert-color: rgb(255, 94, 73);
--alert-color-rgb-components: 255, 94, 73;
--warning-color: rgb(255, 215, 0);
--warning-color-rgb-components: 255, 215, 0;
--notice-color: rgb(70, 215, 0);
--notice-color-rgb-components: 70, 215, 0;
--warning-bg-color: rgb(255, 255, 229);
--warning-bg-color-rgb-components: 255, 255, 229;

/* Action Colors */
--add-color: rgb(76, 176, 109);
--add-color-rgb-components: 76, 176, 109;
--remove-color: rgb(235, 87, 106);
--remove-color-rgb-components: 235, 87, 106;
```

#### Theme Colors

These colors are injected based on the current DatoCMS theme:

```css
/* Dynamic theme colors from context */
--primary-color: /* From ctx.theme.primaryColor */
--primary-color-rgb-components: /* RGB values */
--accent-color: /* From ctx.theme.accentColor */
--accent-color-rgb-components: /* RGB values */
--semi-transparent-accent-color: /* From ctx.theme.semiTransparentAccentColor */
--light-color: /* From ctx.theme.lightColor */
--dark-color: /* From ctx.theme.darkColor */
```

### Typography Variables

```css
/* Font Families */
--base-font-family: 'colfax-web', 'Roboto', 'Helvetica Neue', Helvetica, Roboto, Arial, sans-serif;
--monospaced-font-family: 'Roboto Mono', 'Menlo', 'Bitstream Vera Sans Mono', Consolas, Courier, monospace;

/* Font Weights */
--font-weight-bold: 500;

/* Font Sizes */
--font-size-xxs: 0.6875rem;  /* 11px */
--font-size-xs: 0.75rem;     /* 12px */
--font-size-s: 0.875rem;     /* 14px */
--font-size-m: 0.9375rem;    /* 15px */
--font-size-l: 1.0625rem;    /* 17px */
--font-size-xl: 1.1875rem;   /* 19px */
--font-size-xxl: 1.5625rem;  /* 25px */
--font-size-xxxl: 1.875rem;  /* 30px */
```

### Spacing Variables

```css
/* Positive Spacing */
--spacing-s: 0.375rem;    /* 6px */
--spacing-m: 0.75rem;     /* 12px */
--spacing-l: 1.5rem;      /* 24px */
--spacing-xl: 2.25rem;    /* 36px */
--spacing-xxl: 3.75rem;   /* 60px */
--spacing-xxxl: 6rem;     /* 96px */

/* Negative Spacing */
--negative-spacing-s: -0.375rem;
--negative-spacing-m: -0.75rem;
--negative-spacing-l: -1.5rem;
--negative-spacing-xl: -2.25rem;
--negative-spacing-xxl: -3.75rem;
--negative-spacing-xxxl: -6rem;
```

### Animation Variables

```css
/* Easing Functions */
--material-ease: cubic-bezier(0.55, 0, 0.1, 1);
--inertial-ease: cubic-bezier(0.19, 1, 0.22, 1);
```

## Using the CSS System

### Method 1: Canvas Component (Recommended)

The Canvas component automatically applies all CSS variables and theme styles:

```jsx
import { Canvas } from 'datocms-react-ui';

export default function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <div style={{ 
        color: 'var(--base-body-color)',
        backgroundColor: 'var(--light-bg-color)',
        padding: 'var(--spacing-m)',
        borderRadius: 'var(--border-radius-m)'
      }}>
        Plugin content with theme styling
      </div>
    </Canvas>
  );
}
```

### Method 2: generateStyleFromCtx Utility

For custom containers or when Canvas isn't suitable:

```jsx
import { generateStyleFromCtx } from 'datocms-react-ui';

export default function MyPlugin(ctx) {
  const rootStyles = generateStyleFromCtx(ctx);
  
  return (
    <div style={rootStyles}>
      {/* Content with theme variables available */}
    </div>
  );
}
```

### Method 3: Direct CSS Variable Usage

When using either Canvas or generateStyleFromCtx at the root level, you can reference CSS variables anywhere:

```css
/* In your component styles */
.myComponent {
  color: var(--base-body-color);
  background: var(--lighter-bg-color);
  border: 1px solid var(--border-color);
  padding: var(--spacing-m);
  font-family: var(--base-font-family);
  font-size: var(--font-size-m);
}

/* Status indicators */
.error {
  color: var(--alert-color);
  background: rgba(var(--alert-color-rgb-components), 0.1);
}

.warning {
  color: var(--warning-color);
  background: var(--warning-bg-color);
}

.success {
  color: var(--notice-color);
}
```

## Theme-Aware Components

### Dark Mode Support

```jsx
function ThemedComponent({ ctx }) {
  const isDarkMode = ctx.theme.name === 'dark';
  
  return (
    <div style={{
      backgroundColor: isDarkMode 
        ? 'var(--dark-color)' 
        : 'var(--light-bg-color)',
      color: isDarkMode 
        ? 'var(--light-color)' 
        : 'var(--base-body-color)'
    }}>
      Content adapts to theme
    </div>
  );
}
```

### Using Theme Colors

```jsx
function BrandedButton({ ctx }) {
  return (
    <button style={{
      backgroundColor: 'var(--primary-color)',
      color: 'white',
      border: '2px solid var(--accent-color)',
      padding: 'var(--spacing-s) var(--spacing-m)',
      borderRadius: '4px',
      cursor: 'pointer',
      transition: 'opacity 0.2s var(--material-ease)'
    }}>
      Branded Action
    </button>
  );
}
```

### Semi-Transparent Overlays

```jsx
function Modal({ ctx, children }) {
  return (
    <>
      {/* Backdrop */}
      <div style={{
        position: 'fixed',
        inset: 0,
        backgroundColor: 'rgba(var(--base-body-color-rgb-components), 0.5)'
      }} />
      
      {/* Modal */}
      <div style={{
        position: 'fixed',
        top: '50%',
        left: '50%',
        transform: 'translate(-50%, -50%)',
        backgroundColor: 'var(--lighter-bg-color)',
        border: '1px solid var(--border-color)',
        boxShadow: '0 10px 40px rgba(var(--base-body-color-rgb-components), 0.1)',
        padding: 'var(--spacing-l)'
      }}>
        {children}
      </div>
    </>
  );
}
```

## Typography Guidelines

### Font Hierarchy

```jsx
// Headings
<h1 style={{ 
  fontSize: 'var(--font-size-xxl)',
  fontWeight: 'var(--font-weight-bold)',
  marginBottom: 'var(--spacing-m)'
}}>
  Main Heading
</h1>

// Body Text
<p style={{ 
  fontSize: 'var(--font-size-m)',
  lineHeight: 1.5,
  color: 'var(--base-body-color)'
}}>
  Regular paragraph text
</p>

// Small Text
<small style={{ 
  fontSize: 'var(--font-size-s)',
  color: 'var(--light-body-color)'
}}>
  Secondary information
</small>

// Code
<code style={{ 
  fontFamily: 'var(--monospaced-font-family)',
  fontSize: 'var(--font-size-s)',
  backgroundColor: 'var(--lighter-bg-color)',
  padding: '2px 4px',
  borderRadius: '3px'
}}>
  monospace content
</code>
```

## Spacing System

### Consistent Spacing

```jsx
// Card component with consistent spacing
function Card({ children }) {
  return (
    <div style={{
      padding: 'var(--spacing-l)',
      marginBottom: 'var(--spacing-m)',
      backgroundColor: 'var(--lighter-bg-color)',
      border: '1px solid var(--border-color)',
      borderRadius: '4px'
    }}>
      {children}
    </div>
  );
}

// Form layout
function FormLayout({ children }) {
  return (
    <div style={{
      display: 'flex',
      flexDirection: 'column',
      gap: 'var(--spacing-m)'
    }}>
      {children}
    </div>
  );
}
```

## Animation Guidelines

### Smooth Transitions

```jsx
// Hover effects
<button style={{
  transition: 'all 0.2s var(--material-ease)',
  ':hover': {
    backgroundColor: 'var(--primary-color)',
    transform: 'translateY(-1px)'
  }
}}>
  Hover Me
</button>

// Loading states
<div style={{
  animation: 'spin 1s var(--inertial-ease) infinite'
}}>
  <Spinner />
</div>
```

## Best Practices

### 1. Always Use CSS Variables

```jsx
// ✅ Good
<div style={{ color: 'var(--base-body-color)' }}>

// ❌ Avoid
<div style={{ color: '#34363a' }}>
```

### 2. Respect Theme Context

```jsx
// ✅ Good - Adapts to theme
<div style={{ 
  backgroundColor: 'var(--lighter-bg-color)',
  color: 'var(--base-body-color)'
}}>

// ❌ Avoid - Hardcoded colors
<div style={{ 
  backgroundColor: 'white',
  color: 'black'
}}>
```

### 3. Use Semantic Colors

```jsx
// ✅ Good - Semantic meaning
<div className="error" style={{ color: 'var(--alert-color)' }}>
<div className="success" style={{ color: 'var(--notice-color)' }}>

// ❌ Avoid - Arbitrary colors
<div style={{ color: 'red' }}>
<div style={{ color: 'green' }}>
```

### 4. Consistent Spacing

```jsx
// ✅ Good - Using spacing scale
<div style={{ 
  padding: 'var(--spacing-m)',
  marginBottom: 'var(--spacing-l)'
}}>

// ❌ Avoid - Arbitrary values
<div style={{ 
  padding: '13px',
  marginBottom: '27px'
}}>
```

### 5. Typography Hierarchy

```jsx
// ✅ Good - Using font size scale
<h2 style={{ fontSize: 'var(--font-size-xl)' }}>
<p style={{ fontSize: 'var(--font-size-m)' }}>

// ❌ Avoid - Arbitrary sizes
<h2 style={{ fontSize: '20px' }}>
<p style={{ fontSize: '14px' }}>
```

## Advanced Theming

### Custom CSS Properties

You can define additional CSS properties for your plugin:

```jsx
function MyPlugin({ ctx }) {
  const rootStyles = {
    ...generateStyleFromCtx(ctx),
    '--my-plugin-radius': '8px',
    '--my-plugin-shadow': '0 2px 8px rgba(var(--base-body-color-rgb-components), 0.1)'
  };
  
  return (
    <div style={rootStyles}>
      <div style={{
        borderRadius: 'var(--my-plugin-radius)',
        boxShadow: 'var(--my-plugin-shadow)'
      }}>
        Custom styled content
      </div>
    </div>
  );
}
```

### Responsive Design

While plugins typically have constrained widths, you can still use CSS variables for responsive behavior:

```jsx
function ResponsiveGrid() {
  return (
    <div style={{
      display: 'grid',
      gridTemplateColumns: 'repeat(auto-fit, minmax(200px, 1fr))',
      gap: 'var(--spacing-m)',
      padding: 'var(--spacing-l)'
    }}>
      {/* Grid items */}
    </div>
  );
}
```

## CSS Reset

The SDK includes a minimal CSS reset that:
- Sets consistent font rendering
- Removes default margins and padding from html/body
- Applies the Colfax font family
- Sets base font size to 16px

## Font Loading

The SDK automatically loads the Colfax web font from Adobe Fonts. This ensures visual consistency with the DatoCMS interface. The font includes:
- Regular (400)
- Medium (500) 
- Bold (700)
- Italic variants

## Debugging CSS Variables

To see all available CSS variables in your plugin:

```jsx
function DebugCSSVars() {
  const styles = getComputedStyle(document.documentElement);
  const cssVars = Array.from(styles)
    .filter(prop => prop.startsWith('--'))
    .map(prop => ({
      name: prop,
      value: styles.getPropertyValue(prop)
    }));
    
  return (
    <pre style={{ fontSize: 'var(--font-size-xs)' }}>
      {JSON.stringify(cssVars, null, 2)}
    </pre>
  );
}
```

## Related Documentation

- [Canvas Component](../03-ui-components/Canvas.md) - Pre-styled container
- [generateStyleFromCtx](../03-ui-components/generateStyleFromCtx.md) - Theme utility function
- [UI Components](../03-ui-components/README.md) - Pre-styled React components
- [Context Object](./context-object.md) - Theme information in context