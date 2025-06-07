# ContextInspector Component

A developer tool component that displays the current plugin context in a formatted, collapsible tree view. Useful for debugging and understanding the available context properties.

## Import

```jsx
import { ContextInspector } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `ctx` | `any` | - | **Required.** The context object to inspect |
| `title` | `string` | `'Context Inspector'` | Title for the inspector panel |
| `expanded` | `boolean` | `false` | Initial expanded state |
| `maxDepth` | `number` | `3` | Maximum depth to expand objects |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { Canvas, ContextInspector } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <ContextInspector ctx={ctx} />
    </Canvas>
  );
}
```

## Development Tool

### Conditional Rendering

Only show in development:

```jsx
function MyPlugin({ ctx }) {
  const isDevelopment = process.env.NODE_ENV === 'development';

  return (
    <Canvas ctx={ctx}>
      {/* Your plugin UI */}
      
      {isDevelopment && (
        <ContextInspector 
          ctx={ctx} 
          title="Plugin Context (Dev Only)"
          expanded={true}
        />
      )}
    </Canvas>
  );
}
```

### Debug Mode Toggle

```jsx
function MyPlugin({ ctx }) {
  const [showDebug, setShowDebug] = useState(false);

  return (
    <Canvas ctx={ctx}>
      <Button 
        onClick={() => setShowDebug(!showDebug)}
        buttonType="muted"
        size="s"
      >
        {showDebug ? 'Hide' : 'Show'} Debug Info
      </Button>
      
      {showDebug && <ContextInspector ctx={ctx} />}
    </Canvas>
  );
}
```

## Advanced Usage

### Custom Context Objects

Inspect any object, not just the plugin context:

```jsx
function DataInspector({ data }) {
  return (
    <div>
      <h3>API Response</h3>
      <ContextInspector 
        ctx={data} 
        title="Response Data"
        expanded={true}
        maxDepth={5}
      />
    </div>
  );
}
```

### Multiple Inspectors

```jsx
function DebugPanel({ ctx, formValues, fieldErrors }) {
  return (
    <div style={{ display: 'grid', gap: 'var(--spacing-m)' }}>
      <ContextInspector 
        ctx={ctx} 
        title="Plugin Context"
      />
      
      <ContextInspector 
        ctx={formValues} 
        title="Form Values"
        expanded={true}
      />
      
      <ContextInspector 
        ctx={fieldErrors} 
        title="Field Errors"
        expanded={true}
      />
    </div>
  );
}
```

### Filtered Context

Show only specific parts of the context:

```jsx
function FilteredInspector({ ctx }) {
  // Extract only relevant parts
  const relevantContext = {
    currentUser: ctx.currentUser,
    environment: ctx.environment,
    fieldPath: ctx.fieldPath,
    formValues: ctx.formValues,
    parameters: ctx.parameters,
  };

  return (
    <ContextInspector 
      ctx={relevantContext} 
      title="Relevant Context"
      expanded={true}
    />
  );
}
```

## Use Cases

### Field Extension Development

```jsx
function FieldExtensionPlugin({ ctx }) {
  const [showContext, setShowContext] = useState(true);

  return (
    <Canvas ctx={ctx}>
      <SplitView.SplitView orientation="horizontal">
        <SplitView.SplitViewPane>
          {/* Your field extension UI */}
          <TextField
            label="Custom Field"
            value={ctx.formValues[ctx.fieldPath]}
            onChange={(value) => ctx.setFieldValue(ctx.fieldPath, value)}
          />
        </SplitView.SplitViewPane>
        
        <SplitView.SplitViewPane>
          <ContextInspector 
            ctx={{
              fieldPath: ctx.fieldPath,
              field: ctx.field,
              formValues: ctx.formValues,
              fieldErrors: ctx.fieldErrors,
              parameters: ctx.parameters,
            }}
            title="Field Context"
            expanded={true}
          />
        </SplitView.SplitViewPane>
      </SplitView.SplitView>
    </Canvas>
  );
}
```

### API Testing

```jsx
function ApiTester({ ctx }) {
  const [apiResponse, setApiResponse] = useState(null);
  const [loading, setLoading] = useState(false);

  const testApi = async () => {
    setLoading(true);
    try {
      const client = ctx.createClient();
      const response = await client.items.list({
        filter: { type: ctx.itemType.id },
        page: { limit: 5 }
      });
      setApiResponse(response);
    } catch (error) {
      setApiResponse({ error: error.message });
    } finally {
      setLoading(false);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <Button onClick={testApi} loading={loading}>
        Test API
      </Button>
      
      {apiResponse && (
        <ContextInspector 
          ctx={apiResponse}
          title="API Response"
          expanded={true}
          maxDepth={4}
        />
      )}
    </Canvas>
  );
}
```

### State Debugging

```jsx
function StatefulPlugin({ ctx }) {
  const [state, setState] = useState({
    count: 0,
    items: [],
    filters: {},
    selectedId: null,
  });

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: 'var(--spacing-l)' }}>
        <div>
          {/* Plugin UI that modifies state */}
          <Button onClick={() => setState(s => ({ ...s, count: s.count + 1 }))}>
            Increment Count: {state.count}
          </Button>
        </div>
        
        <div>
          <ContextInspector 
            ctx={state}
            title="Component State"
            expanded={true}
          />
        </div>
      </div>
    </Canvas>
  );
}
```

## Styling

### Custom Appearance

```jsx
<ContextInspector 
  ctx={ctx}
  style={{
    maxHeight: '400px',
    overflow: 'auto',
    border: '1px solid var(--border-color)',
    borderRadius: 'var(--border-radius-m)',
    padding: 'var(--spacing-s)'
  }}
  className="my-inspector"
/>
```

### In a Collapsible Panel

```jsx
import { Section, ContextInspector } from 'datocms-react-ui';

function CollapsibleDebug({ ctx }) {
  return (
    <Section title="Debug Information" startOpen={false}>
      <ContextInspector ctx={ctx} expanded={true} />
    </Section>
  );
}
```

## Features

### Tree View Navigation

- Click on objects/arrays to expand/collapse
- Automatic syntax highlighting
- Type indicators (string, number, boolean, null, undefined)
- Array length indicators
- Object key count

### Performance

- Lazy rendering of nested objects
- Maximum depth control to prevent infinite recursion
- Efficient re-rendering on context changes

### Copy Functionality

Users can select and copy values from the inspector for debugging.

## TypeScript

```typescript
import { ContextInspector } from 'datocms-react-ui';
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

interface DebugPanelProps {
  ctx: RenderFieldExtensionCtx;
  showAdvanced?: boolean;
}

function DebugPanel({ ctx, showAdvanced = false }: DebugPanelProps) {
  const contextToShow = showAdvanced ? ctx : {
    field: ctx.field,
    formValues: ctx.formValues,
    parameters: ctx.parameters,
  };

  return (
    <ContextInspector 
      ctx={contextToShow}
      title={showAdvanced ? "Full Context" : "Basic Context"}
      expanded={true}
    />
  );
}
```

## Best Practices

1. **Development only**: Remove or conditionally render in production
2. **Filter sensitive data**: Don't expose API keys or secrets
3. **Performance**: Limit depth for large objects
4. **User experience**: Place in non-intrusive location
5. **Clear labeling**: Use descriptive titles for multiple inspectors

## Common Patterns

### Tab-based Debug Panel

```jsx
function DebugTabs({ ctx }) {
  const [activeTab, setActiveTab] = useState('context');
  
  const tabs = {
    context: { label: 'Context', data: ctx },
    form: { label: 'Form', data: { values: ctx.formValues, errors: ctx.fieldErrors } },
    site: { label: 'Site', data: ctx.site },
    user: { label: 'User', data: ctx.currentUser },
  };

  return (
    <div>
      <ButtonGroup.Group>
        {Object.entries(tabs).map(([key, tab]) => (
          <ButtonGroup.Button
            key={key}
            selected={activeTab === key}
            onClick={() => setActiveTab(key)}
          >
            {tab.label}
          </ButtonGroup.Button>
        ))}
      </ButtonGroup.Group>
      
      <ContextInspector 
        ctx={tabs[activeTab].data}
        title={tabs[activeTab].label}
        expanded={true}
      />
    </div>
  );
}
```

## Related Components

- [Canvas](./Canvas.md) - Required wrapper component
- [Section](./Section.md) - Collapsible sections
- [SplitView](./SplitView.md) - Side-by-side layouts