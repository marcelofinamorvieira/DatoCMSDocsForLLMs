# SidebarPanel Component

A collapsible panel component designed for use in sidebars, providing a consistent way to organize sidebar content in DatoCMS plugins.

## Import

```jsx
import { SidebarPanel } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `title` | `string` | - | **Required.** Panel title |
| `children` | `ReactNode` | - | Panel content |
| `startOpen` | `boolean` | `true` | Initial expanded state |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { SidebarPanel, Canvas } from 'datocms-react-ui';

function MySidebarPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <SidebarPanel title="Quick Actions">
        <Button onClick={handleAction}>
          Perform Action
        </Button>
      </SidebarPanel>
      
      <SidebarPanel title="Information" startOpen={false}>
        <p>Additional information here</p>
      </SidebarPanel>
    </Canvas>
  );
}
```

## Common Patterns

### Item Form Sidebar

```jsx
function ItemSidebar({ ctx }) {
  const item = ctx.item;
  
  return (
    <>
      <SidebarPanel title="Publishing">
        <div style={{ display: 'grid', gap: 'var(--spacing-s)' }}>
          <div>
            <strong>Status:</strong> {item.meta.status}
          </div>
          <div>
            <strong>Updated:</strong> {new Date(item.meta.updatedAt).toLocaleDateString()}
          </div>
          <Button 
            buttonType="primary" 
            fullWidth
            onClick={() => ctx.publishItem()}
          >
            Publish
          </Button>
        </div>
      </SidebarPanel>
      
      <SidebarPanel title="SEO Preview" startOpen={false}>
        <div style={{ 
          padding: 'var(--spacing-s)',
          backgroundColor: 'var(--light-color)',
          borderRadius: 'var(--border-radius-s)'
        }}>
          <div style={{ color: '#1a0dab', fontSize: '18px' }}>
            {item.title || 'Page Title'}
          </div>
          <div style={{ color: '#006621', fontSize: '14px' }}>
            example.com/page-url
          </div>
          <div style={{ color: '#545454', fontSize: '13px' }}>
            {item.description || 'Page description...'}
          </div>
        </div>
      </SidebarPanel>
    </>
  );
}
```

### Media Sidebar Panels

```jsx
function MediaSidebar({ ctx }) {
  const upload = ctx.upload;
  
  return (
    <>
      <SidebarPanel title="File Information">
        <dl style={{ 
          display: 'grid', 
          gridTemplateColumns: 'auto 1fr',
          gap: 'var(--spacing-xs) var(--spacing-s)',
          margin: 0
        }}>
          <dt>Type:</dt>
          <dd>{upload.mimeType}</dd>
          
          <dt>Size:</dt>
          <dd>{formatFileSize(upload.size)}</dd>
          
          <dt>Dimensions:</dt>
          <dd>{upload.width} × {upload.height}px</dd>
          
          <dt>Created:</dt>
          <dd>{new Date(upload.createdAt).toLocaleDateString()}</dd>
        </dl>
      </SidebarPanel>
      
      <SidebarPanel title="Alt Text & Caption">
        <TextField
          label="Alt Text"
          value={upload.alt || ''}
          onChange={(value) => ctx.updateUpload({ alt: value })}
          hint="Describe the image for accessibility"
        />
        
        <TextareaField
          label="Caption"
          value={upload.title || ''}
          onChange={(value) => ctx.updateUpload({ title: value })}
          rows={3}
        />
      </SidebarPanel>
      
      <SidebarPanel title="Tags" startOpen={false}>
        <TagInput
          tags={upload.tags || []}
          onChange={(tags) => ctx.updateUpload({ tags })}
        />
      </SidebarPanel>
    </>
  );
}
```

## Advanced Examples

### Dynamic Content Loading

```jsx
function DynamicPanel({ title, loadContent }) {
  const [content, setContent] = useState(null);
  const [loading, setLoading] = useState(false);
  const [isOpen, setIsOpen] = useState(false);

  useEffect(() => {
    if (isOpen && !content) {
      setLoading(true);
      loadContent()
        .then(setContent)
        .catch(err => setContent({ error: err.message }))
        .finally(() => setLoading(false));
    }
  }, [isOpen]);

  return (
    <SidebarPanel 
      title={title}
      startOpen={false}
      onToggle={setIsOpen}
    >
      {loading && (
        <div style={{ textAlign: 'center', padding: 'var(--spacing-m)' }}>
          <Spinner />
        </div>
      )}
      
      {content?.error && (
        <div style={{ color: 'var(--alert-color)' }}>
          Error: {content.error}
        </div>
      )}
      
      {content && !content.error && (
        <div>{content}</div>
      )}
    </SidebarPanel>
  );
}
```

### Settings Panel

```jsx
function SettingsPanel({ settings, onChange }) {
  return (
    <SidebarPanel title="Plugin Settings">
      <Form>
        <SwitchField
          label="Enable Notifications"
          value={settings.notifications}
          onChange={(value) => onChange({ ...settings, notifications: value })}
        />
        
        <SelectField
          label="Theme"
          value={settings.theme}
          onChange={(value) => onChange({ ...settings, theme: value })}
        >
          <option value="light">Light</option>
          <option value="dark">Dark</option>
          <option value="auto">Auto</option>
        </SelectField>
        
        <TextField
          label="API Endpoint"
          value={settings.apiEndpoint}
          onChange={(value) => onChange({ ...settings, apiEndpoint: value })}
          placeholder="https://api.example.com"
        />
      </Form>
    </SidebarPanel>
  );
}
```

### Analytics Panel

```jsx
function AnalyticsPanel({ itemId }) {
  const [stats, setStats] = useState(null);

  useEffect(() => {
    fetchAnalytics(itemId).then(setStats);
  }, [itemId]);

  return (
    <SidebarPanel title="Analytics" startOpen={false}>
      {stats ? (
        <div style={{ display: 'grid', gap: 'var(--spacing-m)' }}>
          <div style={{ 
            display: 'grid', 
            gridTemplateColumns: '1fr 1fr',
            gap: 'var(--spacing-s)'
          }}>
            <div style={{ textAlign: 'center' }}>
              <div style={{ fontSize: 'var(--font-size-xl)', fontWeight: 'bold' }}>
                {stats.views.toLocaleString()}
              </div>
              <div style={{ color: 'var(--muted-color)' }}>Views</div>
            </div>
            <div style={{ textAlign: 'center' }}>
              <div style={{ fontSize: 'var(--font-size-xl)', fontWeight: 'bold' }}>
                {stats.clicks.toLocaleString()}
              </div>
              <div style={{ color: 'var(--muted-color)' }}>Clicks</div>
            </div>
          </div>
          
          <div>
            <div style={{ marginBottom: 'var(--spacing-xs)' }}>
              <strong>Top Referrers</strong>
            </div>
            <ul style={{ margin: 0, paddingLeft: 'var(--spacing-l)' }}>
              {stats.referrers.map(ref => (
                <li key={ref.domain}>
                  {ref.domain} ({ref.count})
                </li>
              ))}
            </ul>
          </div>
        </div>
      ) : (
        <Spinner />
      )}
    </SidebarPanel>
  );
}
```

### Action Panel with Confirmation

```jsx
function ActionsPanel({ ctx, item }) {
  const handleDelete = async () => {
    const confirmed = await ctx.openConfirm({
      title: 'Delete Item',
      content: 'Are you sure you want to delete this item? This action cannot be undone.',
      cancel: { label: 'Cancel', value: false },
      choices: [{ label: 'Delete', value: true, intent: 'negative' }]
    });

    if (confirmed) {
      await ctx.deleteItem(item.id);
      ctx.notice('Item deleted successfully');
    }
  };

  return (
    <SidebarPanel title="Actions">
      <div style={{ display: 'grid', gap: 'var(--spacing-s)' }}>
        <Button 
          onClick={() => ctx.copyItem(item.id)}
          fullWidth
        >
          Duplicate Item
        </Button>
        
        <Button 
          onClick={() => ctx.openItemInNewTab(item.id)}
          fullWidth
        >
          Open in New Tab
        </Button>
        
        <hr style={{ 
          margin: 'var(--spacing-s) 0',
          border: 'none',
          borderTop: '1px solid var(--border-color)'
        }} />
        
        <Button 
          buttonType="negative"
          onClick={handleDelete}
          fullWidth
        >
          Delete Item
        </Button>
      </div>
    </SidebarPanel>
  );
}
```

### Workflow Status Panel

```jsx
function WorkflowPanel({ workflow, onStageChange }) {
  const stages = ['draft', 'review', 'approved', 'published'];
  const currentIndex = stages.indexOf(workflow.stage);

  return (
    <SidebarPanel title="Workflow Status">
      <div style={{ marginBottom: 'var(--spacing-m)' }}>
        {stages.map((stage, index) => (
          <div 
            key={stage}
            style={{ 
              display: 'flex',
              alignItems: 'center',
              marginBottom: 'var(--spacing-s)',
              opacity: index > currentIndex ? 0.5 : 1
            }}
          >
            <div style={{
              width: '24px',
              height: '24px',
              borderRadius: '50%',
              backgroundColor: index <= currentIndex 
                ? 'var(--primary-color)' 
                : 'var(--border-color)',
              color: 'white',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              marginRight: 'var(--spacing-s)',
              fontSize: 'var(--font-size-s)'
            }}>
              {index < currentIndex ? '✓' : index + 1}
            </div>
            <span style={{ 
              textTransform: 'capitalize',
              fontWeight: index === currentIndex ? 'bold' : 'normal'
            }}>
              {stage}
            </span>
          </div>
        ))}
      </div>
      
      {currentIndex < stages.length - 1 && (
        <Button 
          buttonType="primary"
          fullWidth
          onClick={() => onStageChange(stages[currentIndex + 1])}
        >
          Move to {stages[currentIndex + 1]}
        </Button>
      )}
    </SidebarPanel>
  );
}
```

### Related Items Panel

```jsx
function RelatedItemsPanel({ ctx, currentItemId }) {
  const [relatedItems, setRelatedItems] = useState([]);
  const [loading, setLoading] = useState(false);

  const loadRelatedItems = async () => {
    setLoading(true);
    try {
      const client = ctx.createClient();
      const items = await client.items.list({
        filter: {
          fields: {
            relatedTo: { eq: currentItemId }
          }
        },
        page: { limit: 5 }
      });
      setRelatedItems(items);
    } finally {
      setLoading(false);
    }
  };

  return (
    <SidebarPanel title="Related Items" startOpen={false}>
      {loading ? (
        <Spinner />
      ) : relatedItems.length > 0 ? (
        <ul style={{ margin: 0, padding: 0, listStyle: 'none' }}>
          {relatedItems.map(item => (
            <li 
              key={item.id}
              style={{ 
                padding: 'var(--spacing-s)',
                borderBottom: '1px solid var(--border-color)',
                cursor: 'pointer'
              }}
              onClick={() => ctx.navigateToItem(item.id)}
            >
              <div>{item.title}</div>
              <div style={{ 
                fontSize: 'var(--font-size-s)',
                color: 'var(--muted-color)' 
              }}>
                {item.itemType.name}
              </div>
            </li>
          ))}
        </ul>
      ) : (
        <p style={{ color: 'var(--muted-color)' }}>
          No related items found
        </p>
      )}
      
      <Button 
        onClick={loadRelatedItems}
        fullWidth
        buttonType="muted"
        style={{ marginTop: 'var(--spacing-s)' }}
      >
        Refresh
      </Button>
    </SidebarPanel>
  );
}
```

## Styling

### Custom Panel Styling

```jsx
<SidebarPanel
  title="Custom Styled Panel"
  style={{
    backgroundColor: 'var(--light-color)',
    border: '2px solid var(--primary-color)',
    borderRadius: 'var(--border-radius-m)'
  }}
  className="custom-sidebar-panel"
>
  <p>Custom styled content</p>
</SidebarPanel>
```

## Accessibility

- Keyboard navigation for expand/collapse
- ARIA attributes for state
- Screen reader announcements
- Focus management
- Semantic HTML structure

## TypeScript

```typescript
import { SidebarPanel } from 'datocms-react-ui';
import { ReactNode } from 'react';

interface PanelConfig {
  title: string;
  content: ReactNode;
  defaultOpen?: boolean;
}

interface SidebarPanelsProps {
  panels: PanelConfig[];
}

function SidebarPanels({ panels }: SidebarPanelsProps) {
  return (
    <>
      {panels.map((panel, index) => (
        <SidebarPanel
          key={index}
          title={panel.title}
          startOpen={panel.defaultOpen ?? true}
        >
          {panel.content}
        </SidebarPanel>
      ))}
    </>
  );
}
```

## Best Practices

1. **Clear titles**: Use descriptive panel titles
2. **Logical grouping**: Group related functionality
3. **Default states**: Consider which panels should start open
4. **Performance**: Load heavy content on demand
5. **Visual hierarchy**: Order panels by importance
6. **Consistent spacing**: Let the component handle spacing

## Related Components

- [Section](./Section.md) - Similar collapsible behavior
- [Canvas](./Canvas.md) - Root container
- [Button](./Button.md) - Common panel actions
- [Form](./Form.md) - Often used within panels