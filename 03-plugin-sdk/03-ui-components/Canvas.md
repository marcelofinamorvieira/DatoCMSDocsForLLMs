# Canvas Component

The root component that provides theming and context for all DatoCMS React UI components. All other components must be wrapped in a Canvas.

## Import

```jsx
import { Canvas } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `ctx` | `RenderCtx` | - | **Required.** Plugin context object |
| `children` | `ReactNode` | - | Child components |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      {/* All other UI components go here */}
    </Canvas>
  );
}
```

## What Canvas Provides

### 1. Theme Context

Canvas applies the DatoCMS theme based on the current project:

```jsx
function ThemedPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      {/* Components automatically use theme colors */}
      <Button buttonType="primary">
        Styled with project colors
      </Button>
    </Canvas>
  );
}
```

### 2. CSS Variables

Canvas sets CSS custom properties for consistent styling:

```css
/* Colors */
--primary-color
--accent-color
--semi-transparent-accent-color
--light-color
--dark-color
--alert-color
--warning-color
--notice-color

/* Spacing */
--spacing-xs
--spacing-s
--spacing-m
--spacing-l
--spacing-xl

/* Typography */
--font-family-sans
--font-family-mono
--font-size-xs
--font-size-s
--font-size-m
--font-size-l
--font-size-xl

/* Borders and Shadows */
--border-radius-s
--border-radius-m
--border-radius-l
--box-shadow-s
--box-shadow-m
--box-shadow-l
```

### 3. Base Styles

Canvas provides base styles for consistency:

```jsx
<Canvas ctx={ctx}>
  {/* Typography, spacing, and form elements are normalized */}
  <div>
    <h1>Heading with consistent styling</h1>
    <p>Paragraph with proper spacing</p>
  </div>
</Canvas>
```

## Advanced Usage

### Full Page Layout

```jsx
function FullPagePlugin({ ctx }) {
  return (
    <Canvas ctx={ctx} style={{ height: '100vh' }}>
      <div style={{ 
        height: '100%', 
        display: 'flex', 
        flexDirection: 'column' 
      }}>
        <header style={{ padding: 'var(--spacing-m)' }}>
          <h1>Plugin Header</h1>
        </header>
        
        <main style={{ flex: 1, overflow: 'auto' }}>
          {/* Main content */}
        </main>
        
        <footer style={{ padding: 'var(--spacing-m)' }}>
          <Button buttonType="primary">Save</Button>
        </footer>
      </div>
    </Canvas>
  );
}
```

### Multiple Canvas Instances

```jsx
// Main plugin
function MainPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <PluginContent />
    </Canvas>
  );
}

// Modal content (separate Canvas)
function ModalContent({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Form>
        {/* Modal form content */}
      </Form>
    </Canvas>
  );
}
```

### With Custom Theme Overrides

```jsx
function CustomThemedPlugin({ ctx }) {
  return (
    <Canvas 
      ctx={ctx}
      style={{
        '--primary-color': '#custom-color',
        '--spacing-unit': '8px'
      }}
    >
      {/* Components with custom theme */}
    </Canvas>
  );
}
```

## Context Access

Canvas makes the plugin context available to all child components:

```jsx
import { useCtx } from 'datocms-react-ui';

function ChildComponent() {
  const ctx = useCtx();
  
  return (
    <Button onClick={() => ctx.notice('Hello!')}>
      Show Notice
    </Button>
  );
}

// Must be used within Canvas
function Plugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <ChildComponent />
    </Canvas>
  );
}
```

## Common Patterns

### Loading State

```jsx
function PluginWithLoading({ ctx }) {
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Load data
    loadData().then(() => setLoading(false));
  }, []);
  
  return (
    <Canvas ctx={ctx}>
      {loading ? (
        <div style={{ 
          display: 'flex', 
          justifyContent: 'center',
          padding: 'var(--spacing-xl)' 
        }}>
          <Spinner />
        </div>
      ) : (
        <PluginContent />
      )}
    </Canvas>
  );
}
```

### Error Boundary

```jsx
class PluginErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <Canvas ctx={this.props.ctx}>
          <div style={{ padding: 'var(--spacing-l)' }}>
            <h2>Something went wrong</h2>
            <Button onClick={() => window.location.reload()}>
              Reload
            </Button>
          </div>
        </Canvas>
      );
    }
    
    return this.props.children;
  }
}

// Usage
function SafePlugin({ ctx }) {
  return (
    <PluginErrorBoundary ctx={ctx}>
      <Canvas ctx={ctx}>
        <PluginContent />
      </Canvas>
    </PluginErrorBoundary>
  );
}
```

### Responsive Layout

```jsx
function ResponsivePlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <div style={{
        display: 'grid',
        gridTemplateColumns: 'repeat(auto-fit, minmax(300px, 1fr))',
        gap: 'var(--spacing-m)',
        padding: 'var(--spacing-l)'
      }}>
        <Section title="Section 1">Content</Section>
        <Section title="Section 2">Content</Section>
        <Section title="Section 3">Content</Section>
      </div>
    </Canvas>
  );
}
```

## TypeScript

```typescript
import { Canvas, CanvasProps } from 'datocms-react-ui';
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

type PluginProps = {
  ctx: RenderFieldExtensionCtx;
};

function MyPlugin({ ctx }: PluginProps) {
  return (
    <Canvas ctx={ctx}>
      {/* Type-safe plugin content */}
    </Canvas>
  );
}
```

## Accessibility

Canvas provides:
- Proper document structure
- Focus management utilities
- ARIA live regions for announcements
- Keyboard navigation support
- High contrast mode support

## Performance

Canvas is lightweight and only provides:
- CSS custom properties injection
- React context for theme/ctx
- Base CSS reset and normalize
- No heavy dependencies or computations

## Best Practices

1. **Always wrap components**: All UI components must be inside Canvas
2. **One Canvas per render root**: Each React root should have its own Canvas
3. **Pass ctx prop**: Always provide the plugin context
4. **Use CSS variables**: Leverage provided CSS custom properties
5. **Avoid inline styles**: Use CSS variables for consistency
6. **Handle loading states**: Show appropriate feedback while loading

## Common Issues

### Components not styled properly
```jsx
// ❌ Wrong - components outside Canvas
<div>
  <Canvas ctx={ctx} />
  <Button>This won't be styled</Button>
</div>

// ✅ Correct - components inside Canvas
<Canvas ctx={ctx}>
  <Button>This will be styled</Button>
</Canvas>
```

### Missing context
```jsx
// ❌ Wrong - missing ctx prop
<Canvas>
  <Content />
</Canvas>

// ✅ Correct - ctx prop provided
<Canvas ctx={ctx}>
  <Content />
</Canvas>
```

## Related Components

- All other datocms-react-ui components require Canvas
- [Button](./Button.md) - Action buttons
- [Form](./Form.md) - Form container
- [Section](./Section.md) - Content sections