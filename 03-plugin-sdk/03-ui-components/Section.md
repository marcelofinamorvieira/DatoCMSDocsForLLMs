# Section Component

A collapsible content section component that provides consistent styling and behavior for organizing content in DatoCMS plugins.

## Import

```jsx
import { Section } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `title` | `string` | - | **Required.** Section title |
| `children` | `ReactNode` | - | Section content |
| `startOpen` | `boolean` | `true` | Initial expanded state |
| `collapsible` | `boolean` | `true` | Whether section can be collapsed |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { Section, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Section title="General Settings">
        <TextField label="API Key" />
        <TextField label="Endpoint URL" />
      </Section>
      
      <Section title="Advanced Options" startOpen={false}>
        <SwitchField label="Enable debug mode" />
        <SelectField label="Log level">
          <option>Info</option>
          <option>Warning</option>
          <option>Error</option>
        </SelectField>
      </Section>
    </Canvas>
  );
}
```

## Common Patterns

### Form Sections

```jsx
function SettingsForm({ ctx }) {
  return (
    <Form onSubmit={handleSubmit}>
      <Section title="Basic Information">
        <TextField label="Name" required />
        <TextField label="Email" type="email" required />
        <TextareaField label="Description" />
      </Section>
      
      <Section title="Preferences" startOpen={false}>
        <SelectField label="Language">
          <option>English</option>
          <option>Spanish</option>
          <option>French</option>
        </SelectField>
        <SelectField label="Timezone">
          <option>UTC</option>
          <option>EST</option>
          <option>PST</option>
        </SelectField>
      </Section>
      
      <Section title="Notifications" startOpen={false}>
        <SwitchField label="Email notifications" />
        <SwitchField label="SMS notifications" />
        <SwitchField label="Push notifications" />
      </Section>
      
      <Button type="submit" buttonType="primary">
        Save Settings
      </Button>
    </Form>
  );
}
```

### Non-Collapsible Sections

```jsx
function StaticSections() {
  return (
    <>
      <Section title="Always Visible" collapsible={false}>
        <p>This section cannot be collapsed</p>
      </Section>
      
      <Section title="Can Be Hidden" collapsible={true}>
        <p>This section can be toggled</p>
      </Section>
    </>
  );
}
```

### Nested Sections

```jsx
function NestedSections() {
  return (
    <Section title="Parent Section">
      <p>Some content in the parent section</p>
      
      <Section title="Child Section 1" style={{ marginTop: 'var(--spacing-m)' }}>
        <p>Nested content 1</p>
      </Section>
      
      <Section title="Child Section 2" style={{ marginTop: 'var(--spacing-m)' }}>
        <p>Nested content 2</p>
      </Section>
    </Section>
  );
}
```

## Advanced Examples

### Dynamic Sections

```jsx
function DynamicSections({ sections }) {
  const [openSections, setOpenSections] = useState(
    sections.reduce((acc, section) => ({
      ...acc,
      [section.id]: section.defaultOpen ?? true
    }), {})
  );

  const toggleSection = (sectionId) => {
    setOpenSections(prev => ({
      ...prev,
      [sectionId]: !prev[sectionId]
    }));
  };

  return (
    <>
      {sections.map(section => (
        <Section
          key={section.id}
          title={section.title}
          startOpen={openSections[section.id]}
          onToggle={() => toggleSection(section.id)}
        >
          {section.content}
        </Section>
      ))}
    </>
  );
}
```

### Section with Actions

```jsx
function SectionWithActions({ title, children, onEdit, onDelete }) {
  return (
    <div style={{ position: 'relative' }}>
      <Section title={title}>
        {children}
      </Section>
      
      <div style={{
        position: 'absolute',
        top: 'var(--spacing-s)',
        right: 'var(--spacing-s)',
        display: 'flex',
        gap: 'var(--spacing-xs)'
      }}>
        <Button size="s" onClick={onEdit}>
          Edit
        </Button>
        <Button size="s" buttonType="negative" onClick={onDelete}>
          Delete
        </Button>
      </div>
    </div>
  );
}
```

### Conditional Content

```jsx
function ConditionalSection({ userRole }) {
  return (
    <>
      <Section title="General Settings">
        <TextField label="Site Name" />
        <TextField label="Site URL" />
      </Section>
      
      {userRole === 'admin' && (
        <Section title="Admin Settings" startOpen={false}>
          <SwitchField label="Maintenance Mode" />
          <TextField label="Admin Email" />
          <TextareaField label="System Message" />
        </Section>
      )}
      
      {userRole === 'developer' && (
        <Section title="Developer Settings" startOpen={false}>
          <SwitchField label="Debug Mode" />
          <SwitchField label="Show Performance Metrics" />
          <TextField label="API Rate Limit" type="number" />
        </Section>
      )}
    </>
  );
}
```

### Loading State in Sections

```jsx
function SectionWithLoading({ title, loadData }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [isOpen, setIsOpen] = useState(false);

  useEffect(() => {
    if (isOpen && !data) {
      setLoading(true);
      loadData()
        .then(setData)
        .finally(() => setLoading(false));
    }
  }, [isOpen]);

  return (
    <Section 
      title={title}
      startOpen={false}
      onToggle={setIsOpen}
    >
      {loading ? (
        <div style={{ 
          display: 'flex', 
          justifyContent: 'center',
          padding: 'var(--spacing-l)' 
        }}>
          <Spinner />
        </div>
      ) : data ? (
        <div>{data}</div>
      ) : (
        <p>No data available</p>
      )}
    </Section>
  );
}
```

### Section Grid

```jsx
function SectionGrid({ sections }) {
  return (
    <div style={{
      display: 'grid',
      gridTemplateColumns: 'repeat(auto-fit, minmax(300px, 1fr))',
      gap: 'var(--spacing-l)'
    }}>
      {sections.map(section => (
        <Section 
          key={section.id}
          title={section.title}
          collapsible={false}
        >
          {section.content}
        </Section>
      ))}
    </div>
  );
}
```

### Tabbed Sections

```jsx
function TabbedSections({ tabs }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div>
      <div style={{ 
        display: 'flex', 
        borderBottom: '1px solid var(--border-color)',
        marginBottom: 'var(--spacing-m)'
      }}>
        {tabs.map((tab, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            style={{
              padding: 'var(--spacing-s) var(--spacing-m)',
              border: 'none',
              background: activeTab === index ? 'var(--light-color)' : 'transparent',
              borderBottom: activeTab === index ? '2px solid var(--primary-color)' : 'none',
              cursor: 'pointer'
            }}
          >
            {tab.title}
          </button>
        ))}
      </div>
      
      <Section 
        title={tabs[activeTab].sectionTitle}
        collapsible={false}
      >
        {tabs[activeTab].content}
      </Section>
    </div>
  );
}
```

### Progress Sections

```jsx
function ProgressSections({ steps, currentStep, onStepComplete }) {
  return (
    <>
      {steps.map((step, index) => (
        <Section
          key={step.id}
          title={
            <span>
              {index < currentStep && '✓ '}
              {step.title}
              {index === currentStep && ' (Current)'}
            </span>
          }
          startOpen={index <= currentStep}
          collapsible={index < currentStep}
          style={{
            opacity: index > currentStep ? 0.5 : 1,
            pointerEvents: index > currentStep ? 'none' : 'auto'
          }}
        >
          {step.content}
          
          {index === currentStep && (
            <Button 
              onClick={() => onStepComplete(index)}
              buttonType="primary"
              style={{ marginTop: 'var(--spacing-m)' }}
            >
              Complete Step
            </Button>
          )}
        </Section>
      ))}
    </>
  );
}
```

## Styling

### Custom Section Styling

```jsx
<Section
  title="Highlighted Section"
  style={{
    backgroundColor: 'var(--light-color)',
    border: '2px solid var(--primary-color)',
    borderRadius: 'var(--border-radius-m)',
    padding: 'var(--spacing-m)'
  }}
  className="important-section"
>
  <p>This section has custom styling</p>
</Section>
```

### Section with Icons

```jsx
import { FaCog, FaUser, FaBell } from 'react-icons/fa';

function IconSections() {
  const sections = [
    { icon: <FaCog />, title: 'Settings', content: '...' },
    { icon: <FaUser />, title: 'Profile', content: '...' },
    { icon: <FaBell />, title: 'Notifications', content: '...' }
  ];

  return (
    <>
      {sections.map(section => (
        <Section
          key={section.title}
          title={
            <span style={{ display: 'flex', alignItems: 'center', gap: 'var(--spacing-s)' }}>
              {section.icon}
              {section.title}
            </span>
          }
        >
          {section.content}
        </Section>
      ))}
    </>
  );
}
```

## Accessibility

- Keyboard navigation support (Enter/Space to toggle)
- ARIA attributes for expanded/collapsed state
- Screen reader announcements
- Focus management
- Semantic HTML structure

## TypeScript

```typescript
import { Section } from 'datocms-react-ui';
import { ReactNode } from 'react';

interface ConfigSection {
  id: string;
  title: string;
  content: ReactNode;
  defaultOpen?: boolean;
  collapsible?: boolean;
}

interface ConfigSectionsProps {
  sections: ConfigSection[];
}

function ConfigSections({ sections }: ConfigSectionsProps) {
  return (
    <>
      {sections.map(section => (
        <Section
          key={section.id}
          title={section.title}
          startOpen={section.defaultOpen}
          collapsible={section.collapsible}
        >
          {section.content}
        </Section>
      ))}
    </>
  );
}
```

## Best Practices

1. **Logical grouping**: Group related content in sections
2. **Clear titles**: Use descriptive section titles
3. **Default state**: Consider which sections should start open
4. **Performance**: For heavy content, load on expand
5. **Accessibility**: Ensure keyboard navigation works
6. **Visual hierarchy**: Don't nest too deeply

## Related Components

- [Form](./Form.md) - Often contains sections
- [FieldGroup](./FieldGroup.md) - For smaller groupings
- [Canvas](./Canvas.md) - Root container
- [SidebarPanel](./SidebarPanel.md) - Similar collapsible behavior