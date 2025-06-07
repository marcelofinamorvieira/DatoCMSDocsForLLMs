# SwitchInput Component

A standalone toggle switch input component that provides just the switch element without label or error handling, matching DatoCMS's design system.

## Import

```jsx
import { SwitchInput } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `value` | `boolean` | `false` | Switch state |
| `onChange` | `(value: boolean) => void` | - | Change handler |
| `disabled` | `boolean` | `false` | Disable the switch |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { SwitchInput, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [isOn, setIsOn] = useState(false);

  return (
    <Canvas ctx={ctx}>
      <SwitchInput
        value={isOn}
        onChange={setIsOn}
      />
    </Canvas>
  );
}
```

## Common Patterns

### With Custom Label

```jsx
function CustomLabelSwitch() {
  const [enabled, setEnabled] = useState(false);

  return (
    <label style={{ 
      display: 'flex', 
      alignItems: 'center', 
      gap: 'var(--spacing-s)',
      cursor: 'pointer'
    }}>
      <SwitchInput
        value={enabled}
        onChange={setEnabled}
      />
      <span>Enable feature</span>
    </label>
  );
}
```

### Inline with Text

```jsx
function InlineSwitch() {
  const [autoSave, setAutoSave] = useState(true);

  return (
    <div style={{ 
      display: 'flex', 
      alignItems: 'center', 
      gap: 'var(--spacing-s)' 
    }}>
      <span>Auto-save is</span>
      <SwitchInput
        value={autoSave}
        onChange={setAutoSave}
      />
      <span>{autoSave ? 'ON' : 'OFF'}</span>
    </div>
  );
}
```

### Table Row Toggle

```jsx
function ToggleTable({ items, onToggle }) {
  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Status</th>
          <th>Enabled</th>
        </tr>
      </thead>
      <tbody>
        {items.map(item => (
          <tr key={item.id}>
            <td>{item.name}</td>
            <td>{item.status}</td>
            <td>
              <SwitchInput
                value={item.enabled}
                onChange={(value) => onToggle(item.id, value)}
              />
            </td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## Advanced Examples

### Loading State

```jsx
function AsyncSwitch({ loadValue, saveValue }) {
  const [value, setValue] = useState(false);
  const [loading, setLoading] = useState(true);
  const [saving, setSaving] = useState(false);

  useEffect(() => {
    loadValue().then(val => {
      setValue(val);
      setLoading(false);
    });
  }, []);

  const handleChange = async (newValue) => {
    setValue(newValue);
    setSaving(true);
    
    try {
      await saveValue(newValue);
    } catch (error) {
      // Revert on error
      setValue(!newValue);
      console.error('Failed to save:', error);
    } finally {
      setSaving(false);
    }
  };

  return (
    <div style={{ 
      display: 'inline-flex', 
      alignItems: 'center',
      gap: 'var(--spacing-s)'
    }}>
      <SwitchInput
        value={value}
        onChange={handleChange}
        disabled={loading || saving}
      />
      {(loading || saving) && <Spinner size="xs" />}
    </div>
  );
}
```

### Toolbar Switch

```jsx
import { Toolbar, SwitchInput } from 'datocms-react-ui';

function ToolbarWithSwitch() {
  const [editMode, setEditMode] = useState(false);
  const [preview, setPreview] = useState(true);

  return (
    <Toolbar.Toolbar>
      <Toolbar.Stack>
        <Toolbar.Title>Editor</Toolbar.Title>
      </Toolbar.Stack>
      
      <Toolbar.Stack>
        <label style={{ 
          display: 'flex', 
          alignItems: 'center',
          gap: 'var(--spacing-xs)'
        }}>
          <span>Edit Mode</span>
          <SwitchInput
            value={editMode}
            onChange={setEditMode}
          />
        </label>
        
        <label style={{ 
          display: 'flex', 
          alignItems: 'center',
          gap: 'var(--spacing-xs)'
        }}>
          <span>Preview</span>
          <SwitchInput
            value={preview}
            onChange={setPreview}
          />
        </label>
      </Toolbar.Stack>
    </Toolbar.Toolbar>
  );
}
```

### Settings List

```jsx
function SettingsList({ settings }) {
  return (
    <ul style={{ 
      listStyle: 'none', 
      padding: 0,
      margin: 0 
    }}>
      {settings.map(setting => (
        <li 
          key={setting.id}
          style={{
            display: 'flex',
            justifyContent: 'space-between',
            alignItems: 'center',
            padding: 'var(--spacing-m)',
            borderBottom: '1px solid var(--border-color)'
          }}
        >
          <div>
            <div style={{ fontWeight: 'bold' }}>{setting.title}</div>
            <div style={{ 
              fontSize: 'var(--font-size-s)', 
              color: 'var(--muted-color)' 
            }}>
              {setting.description}
            </div>
          </div>
          <SwitchInput
            value={setting.value}
            onChange={setting.onChange}
          />
        </li>
      ))}
    </ul>
  );
}
```

### Card with Toggle

```jsx
function FeatureCard({ feature, enabled, onToggle }) {
  return (
    <div style={{
      border: '1px solid var(--border-color)',
      borderRadius: 'var(--border-radius-m)',
      padding: 'var(--spacing-m)',
      display: 'flex',
      justifyContent: 'space-between',
      alignItems: 'flex-start'
    }}>
      <div style={{ flex: 1 }}>
        <h3>{feature.name}</h3>
        <p style={{ 
          color: 'var(--muted-color)',
          marginBottom: 0 
        }}>
          {feature.description}
        </p>
      </div>
      <SwitchInput
        value={enabled}
        onChange={onToggle}
      />
    </div>
  );
}
```

### Grouped Switches

```jsx
function SwitchGroup({ options, values, onChange }) {
  return (
    <div style={{ 
      display: 'grid', 
      gap: 'var(--spacing-s)' 
    }}>
      {options.map(option => (
        <label
          key={option.id}
          style={{
            display: 'flex',
            alignItems: 'center',
            padding: 'var(--spacing-s)',
            cursor: 'pointer',
            borderRadius: 'var(--border-radius-s)',
            ':hover': {
              backgroundColor: 'var(--light-color)'
            }
          }}
        >
          <SwitchInput
            value={values[option.id] || false}
            onChange={(value) => onChange(option.id, value)}
          />
          <span style={{ marginLeft: 'var(--spacing-s)' }}>
            {option.label}
          </span>
        </label>
      ))}
    </div>
  );
}
```

### Confirmation Toggle

```jsx
function ConfirmationSwitch({ label, value, onChange, confirmMessage }) {
  const handleChange = async (newValue) => {
    if (newValue && confirmMessage) {
      const confirmed = confirm(confirmMessage);
      if (!confirmed) return;
    }
    onChange(newValue);
  };

  return (
    <label style={{ 
      display: 'flex', 
      alignItems: 'center',
      gap: 'var(--spacing-s)' 
    }}>
      <SwitchInput
        value={value}
        onChange={handleChange}
      />
      <span>{label}</span>
    </label>
  );
}
```

### Styled States

```jsx
function StyledSwitch() {
  const [value, setValue] = useState(false);

  return (
    <div>
      <div style={{ marginBottom: 'var(--spacing-m)' }}>
        <h4>Normal</h4>
        <SwitchInput value={value} onChange={setValue} />
      </div>
      
      <div style={{ marginBottom: 'var(--spacing-m)' }}>
        <h4>Disabled (Off)</h4>
        <SwitchInput value={false} onChange={() => {}} disabled />
      </div>
      
      <div style={{ marginBottom: 'var(--spacing-m)' }}>
        <h4>Disabled (On)</h4>
        <SwitchInput value={true} onChange={() => {}} disabled />
      </div>
      
      <div style={{ 
        marginBottom: 'var(--spacing-m)',
        padding: 'var(--spacing-s)',
        backgroundColor: 'var(--primary-color)',
        borderRadius: 'var(--border-radius-s)'
      }}>
        <h4 style={{ color: 'white' }}>On Dark Background</h4>
        <SwitchInput value={value} onChange={setValue} />
      </div>
    </div>
  );
}
```

## Form Integration

### With FieldWrapper

```jsx
import { FieldWrapper, FormLabel, SwitchInput, FieldHint } from 'datocms-react-ui';

function CustomSwitchField({ label, hint, value, onChange }) {
  const id = `switch-${label.toLowerCase().replace(/\s+/g, '-')}`;
  
  return (
    <FieldWrapper>
      <div style={{ 
        display: 'flex', 
        justifyContent: 'space-between',
        alignItems: 'flex-start'
      }}>
        <div style={{ flex: 1 }}>
          <FormLabel htmlFor={id}>{label}</FormLabel>
          {hint && <FieldHint>{hint}</FieldHint>}
        </div>
        <SwitchInput
          id={id}
          value={value}
          onChange={onChange}
        />
      </div>
    </FieldWrapper>
  );
}
```

## Accessibility

- Native checkbox input under the hood
- Keyboard support (Space to toggle)
- Focus indicators
- ARIA attributes
- Works with screen readers

## TypeScript

```typescript
import { SwitchInput } from 'datocms-react-ui';

interface ToggleOption {
  id: string;
  label: string;
  value: boolean;
  onChange: (value: boolean) => void;
}

interface ToggleListProps {
  options: ToggleOption[];
}

function ToggleList({ options }: ToggleListProps) {
  return (
    <div>
      {options.map(option => (
        <div 
          key={option.id}
          style={{ 
            display: 'flex',
            alignItems: 'center',
            gap: 'var(--spacing-s)',
            marginBottom: 'var(--spacing-s)'
          }}
        >
          <SwitchInput
            value={option.value}
            onChange={option.onChange}
          />
          <label>{option.label}</label>
        </div>
      ))}
    </div>
  );
}
```

## Best Practices

1. **Always label**: Provide clear labels for accessibility
2. **Visual feedback**: Show state changes immediately
3. **Confirmation**: Confirm destructive toggles
4. **Loading states**: Show when async operations occur
5. **Disabled state**: Use when temporarily unavailable
6. **Keyboard support**: Ensure Space key toggles

## Related Components

- [SwitchField](./SwitchField.md) - Complete switch with label
- [FieldWrapper](./FieldWrapper.md) - For custom layouts
- [FormLabel](./FormLabel.md) - For labeling
- [TextInput](./TextInput.md) - Text input component