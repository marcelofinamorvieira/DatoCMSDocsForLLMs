# VerticalSplit Component

A simple two-pane vertical split layout component that divides content into left and right sections, perfect for creating side-by-side layouts in DatoCMS plugins.

## Import

```jsx
import { VerticalSplit } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode[]` | - | Two child elements (left and right panes) |
| `split` | `number` | `50` | Split percentage (0-100) |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { VerticalSplit, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <VerticalSplit split={30}>
        <div style={{ padding: 'var(--spacing-m)' }}>
          Left pane content (30%)
        </div>
        
        <div style={{ padding: 'var(--spacing-m)' }}>
          Right pane content (70%)
        </div>
      </VerticalSplit>
    </Canvas>
  );
}
```

## Common Patterns

### Navigation and Content

```jsx
function NavigationLayout() {
  return (
    <VerticalSplit split={25}>
      {/* Navigation Pane */}
      <nav style={{ 
        height: '100vh',
        backgroundColor: 'var(--light-color)',
        padding: 'var(--spacing-m)',
        overflow: 'auto'
      }}>
        <h3>Navigation</h3>
        <ul>
          <li><a href="#section1">Section 1</a></li>
          <li><a href="#section2">Section 2</a></li>
          <li><a href="#section3">Section 3</a></li>
        </ul>
      </nav>
      
      {/* Content Pane */}
      <main style={{ 
        height: '100vh',
        padding: 'var(--spacing-l)',
        overflow: 'auto'
      }}>
        <h1>Main Content</h1>
        <p>Your main content goes here...</p>
      </main>
    </VerticalSplit>
  );
}
```

### Form and Preview

```jsx
function FormWithPreview() {
  const [formData, setFormData] = useState({
    title: '',
    content: '',
    published: false
  });

  return (
    <VerticalSplit split={50}>
      {/* Form Pane */}
      <div style={{ padding: 'var(--spacing-m)', overflow: 'auto' }}>
        <h3>Edit Content</h3>
        <Form>
          <TextField
            label="Title"
            value={formData.title}
            onChange={(value) => setFormData({ ...formData, title: value })}
          />
          
          <TextareaField
            label="Content"
            value={formData.content}
            onChange={(value) => setFormData({ ...formData, content: value })}
            rows={10}
          />
          
          <SwitchField
            label="Published"
            value={formData.published}
            onChange={(value) => setFormData({ ...formData, published: value })}
          />
        </Form>
      </div>
      
      {/* Preview Pane */}
      <div style={{ 
        padding: 'var(--spacing-m)',
        backgroundColor: 'var(--light-color)',
        overflow: 'auto'
      }}>
        <h3>Preview</h3>
        <div style={{
          padding: 'var(--spacing-m)',
          backgroundColor: 'white',
          borderRadius: 'var(--border-radius-m)',
          minHeight: '200px'
        }}>
          <h1>{formData.title || 'Untitled'}</h1>
          <p>{formData.content || 'No content yet...'}</p>
          {formData.published && (
            <span style={{ 
              color: 'var(--success-color)',
              fontSize: 'var(--font-size-s)'
            }}>
              ✓ Published
            </span>
          )}
        </div>
      </div>
    </VerticalSplit>
  );
}
```

### List and Details

```jsx
function ListDetailView({ items, selectedId, onSelect }) {
  const selectedItem = items.find(item => item.id === selectedId);

  return (
    <VerticalSplit split={35}>
      {/* List Pane */}
      <div style={{ height: '600px', overflow: 'auto' }}>
        <ul style={{ listStyle: 'none', padding: 0, margin: 0 }}>
          {items.map(item => (
            <li
              key={item.id}
              onClick={() => onSelect(item.id)}
              style={{
                padding: 'var(--spacing-m)',
                borderBottom: '1px solid var(--border-color)',
                cursor: 'pointer',
                backgroundColor: item.id === selectedId ? 'var(--light-color)' : 'transparent',
                ':hover': {
                  backgroundColor: 'var(--light-color)'
                }
              }}
            >
              <div style={{ fontWeight: 'bold' }}>{item.title}</div>
              <div style={{ 
                fontSize: 'var(--font-size-s)',
                color: 'var(--muted-color)'
              }}>
                {item.subtitle}
              </div>
            </li>
          ))}
        </ul>
      </div>
      
      {/* Details Pane */}
      <div style={{ padding: 'var(--spacing-l)' }}>
        {selectedItem ? (
          <div>
            <h2>{selectedItem.title}</h2>
            <p>{selectedItem.description}</p>
            {/* More details... */}
          </div>
        ) : (
          <div style={{ 
            textAlign: 'center',
            color: 'var(--muted-color)',
            paddingTop: 'var(--spacing-xl)'
          }}>
            Select an item to view details
          </div>
        )}
      </div>
    </VerticalSplit>
  );
}
```

## Advanced Examples

### Code Editor Layout

```jsx
function CodeEditorLayout() {
  const [code, setCode] = useState('');
  const [output, setOutput] = useState('');

  const runCode = () => {
    try {
      // Simulate code execution
      const result = eval(code);
      setOutput(String(result));
    } catch (error) {
      setOutput(`Error: ${error.message}`);
    }
  };

  return (
    <div style={{ height: '600px' }}>
      <Toolbar.Toolbar>
        <Toolbar.Stack>
          <Toolbar.Title>Code Editor</Toolbar.Title>
        </Toolbar.Stack>
        <Toolbar.Stack>
          <Toolbar.Button onClick={runCode}>
            <FaPlay /> Run
          </Toolbar.Button>
        </Toolbar.Stack>
      </Toolbar.Toolbar>
      
      <VerticalSplit split={60}>
        {/* Editor Pane */}
        <div style={{ height: '100%', display: 'flex', flexDirection: 'column' }}>
          <div style={{ 
            padding: 'var(--spacing-s)',
            backgroundColor: 'var(--light-color)',
            fontWeight: 'bold'
          }}>
            editor.js
          </div>
          <TextareaInput
            value={code}
            onChange={(e) => setCode(e.target.value)}
            style={{
              flex: 1,
              fontFamily: 'monospace',
              fontSize: '14px',
              border: 'none',
              resize: 'none'
            }}
            placeholder="// Write your code here"
          />
        </div>
        
        {/* Output Pane */}
        <div style={{ height: '100%', display: 'flex', flexDirection: 'column' }}>
          <div style={{ 
            padding: 'var(--spacing-s)',
            backgroundColor: 'var(--light-color)',
            fontWeight: 'bold'
          }}>
            Output
          </div>
          <pre style={{
            flex: 1,
            margin: 0,
            padding: 'var(--spacing-m)',
            backgroundColor: '#1e1e1e',
            color: '#fff',
            overflow: 'auto'
          }}>
            {output || 'Run code to see output...'}
          </pre>
        </div>
      </VerticalSplit>
    </div>
  );
}
```

### Comparison View

```jsx
function ComparisonView({ original, modified }) {
  const [showDifferences, setShowDifferences] = useState(true);

  return (
    <div>
      <Toolbar.Toolbar>
        <Toolbar.Stack>
          <Toolbar.Title>Compare Versions</Toolbar.Title>
        </Toolbar.Stack>
        <Toolbar.Stack>
          <SwitchInput
            value={showDifferences}
            onChange={setShowDifferences}
          />
          <span>Highlight differences</span>
        </Toolbar.Stack>
      </Toolbar.Toolbar>
      
      <VerticalSplit split={50}>
        {/* Original Version */}
        <div style={{ padding: 'var(--spacing-m)' }}>
          <h3>Original Version</h3>
          <div style={{
            padding: 'var(--spacing-m)',
            backgroundColor: 'var(--light-color)',
            borderRadius: 'var(--border-radius-m)',
            whiteSpace: 'pre-wrap'
          }}>
            {original}
          </div>
        </div>
        
        {/* Modified Version */}
        <div style={{ padding: 'var(--spacing-m)' }}>
          <h3>Modified Version</h3>
          <div style={{
            padding: 'var(--spacing-m)',
            backgroundColor: 'var(--light-color)',
            borderRadius: 'var(--border-radius-m)',
            whiteSpace: 'pre-wrap'
          }}>
            {modified}
          </div>
        </div>
      </VerticalSplit>
    </div>
  );
}
```

### Settings and Preview

```jsx
function SettingsWithPreview({ ctx }) {
  const [settings, setSettings] = useState({
    theme: 'light',
    fontSize: 'medium',
    showGrid: true,
    columns: 3
  });

  return (
    <VerticalSplit split={40}>
      {/* Settings Pane */}
      <div style={{ padding: 'var(--spacing-m)', overflow: 'auto' }}>
        <h3>Widget Settings</h3>
        
        <Form>
          <SelectField
            label="Theme"
            value={settings.theme}
            onChange={(value) => setSettings({ ...settings, theme: value })}
          >
            <option value="light">Light</option>
            <option value="dark">Dark</option>
            <option value="auto">Auto</option>
          </SelectField>
          
          <SelectField
            label="Font Size"
            value={settings.fontSize}
            onChange={(value) => setSettings({ ...settings, fontSize: value })}
          >
            <option value="small">Small</option>
            <option value="medium">Medium</option>
            <option value="large">Large</option>
          </SelectField>
          
          <SwitchField
            label="Show Grid"
            value={settings.showGrid}
            onChange={(value) => setSettings({ ...settings, showGrid: value })}
          />
          
          <TextField
            label="Columns"
            type="number"
            value={settings.columns}
            onChange={(value) => setSettings({ ...settings, columns: parseInt(value) || 1 })}
            min="1"
            max="6"
          />
        </Form>
      </div>
      
      {/* Preview Pane */}
      <div style={{ 
        padding: 'var(--spacing-m)',
        backgroundColor: settings.theme === 'dark' ? '#1e1e1e' : 'white',
        color: settings.theme === 'dark' ? 'white' : 'black',
        height: '100%',
        overflow: 'auto'
      }}>
        <h3>Preview</h3>
        
        <div style={{
          display: settings.showGrid ? 'grid' : 'block',
          gridTemplateColumns: `repeat(${settings.columns}, 1fr)`,
          gap: 'var(--spacing-m)',
          fontSize: settings.fontSize === 'small' ? '12px' : 
                   settings.fontSize === 'large' ? '18px' : '14px'
        }}>
          {Array.from({ length: 6 }, (_, i) => (
            <div
              key={i}
              style={{
                padding: 'var(--spacing-m)',
                backgroundColor: settings.theme === 'dark' ? '#333' : 'var(--light-color)',
                borderRadius: 'var(--border-radius-m)',
                textAlign: 'center'
              }}
            >
              Item {i + 1}
            </div>
          ))}
        </div>
      </div>
    </VerticalSplit>
  );
}
```

### Responsive Behavior

```jsx
function ResponsiveVerticalSplit({ children }) {
  const [windowWidth, setWindowWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWindowWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  // Stack vertically on small screens
  if (windowWidth < 768) {
    return (
      <div style={{ display: 'flex', flexDirection: 'column' }}>
        {children}
      </div>
    );
  }

  // Side by side on larger screens
  return <VerticalSplit split={40}>{children}</VerticalSplit>;
}
```

## Styling

### Custom Split Styling

```jsx
<VerticalSplit 
  split={60}
  style={{
    height: '500px',
    border: '1px solid var(--border-color)',
    borderRadius: 'var(--border-radius-m)',
    overflow: 'hidden'
  }}
  className="custom-split"
>
  <div>Left content</div>
  <div>Right content</div>
</VerticalSplit>
```

### With Different Backgrounds

```jsx
function StyledSplit() {
  return (
    <VerticalSplit split={50}>
      <div style={{ 
        height: '400px',
        backgroundColor: 'var(--primary-color)',
        color: 'white',
        padding: 'var(--spacing-l)',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center'
      }}>
        <h2>Left Pane</h2>
      </div>
      
      <div style={{ 
        height: '400px',
        backgroundColor: 'var(--light-color)',
        padding: 'var(--spacing-l)',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center'
      }}>
        <h2>Right Pane</h2>
      </div>
    </VerticalSplit>
  );
}
```

## Accessibility

- Uses semantic HTML structure
- Keyboard accessible content
- Screen reader friendly
- Maintains proper tab order

## TypeScript

```typescript
import { VerticalSplit } from 'datocms-react-ui';
import { ReactNode } from 'react';

interface SplitLayoutProps {
  leftContent: ReactNode;
  rightContent: ReactNode;
  leftWidth?: number;
}

function SplitLayout({ 
  leftContent, 
  rightContent, 
  leftWidth = 30 
}: SplitLayoutProps) {
  return (
    <VerticalSplit split={leftWidth}>
      <div style={{ height: '100%', overflow: 'auto' }}>
        {leftContent}
      </div>
      <div style={{ height: '100%', overflow: 'auto' }}>
        {rightContent}
      </div>
    </VerticalSplit>
  );
}
```

## Best Practices

1. **Set container height**: VerticalSplit needs a defined height
2. **Handle overflow**: Add scroll to pane content when needed
3. **Responsive design**: Consider stacking on mobile
4. **Appropriate splits**: Choose split ratios based on content
5. **Consistent padding**: Add padding inside panes
6. **Accessibility**: Ensure both panes are keyboard accessible

## Related Components

- [SplitView](./SplitView.md) - More advanced split layouts
- [Section](./Section.md) - Collapsible sections
- [Canvas](./Canvas.md) - Root container