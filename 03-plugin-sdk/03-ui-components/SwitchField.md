# SwitchField Component

A toggle switch field component that combines a label, switch input, hint text, and error handling, matching DatoCMS's design system.

## Import

```jsx
import { SwitchField } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `label` | `string` | - | Field label |
| `value` | `boolean` | `false` | Switch state |
| `onChange` | `(value: boolean) => void` | - | Change handler |
| `hint` | `string` | - | Help text below field |
| `error` | `string` | - | Error message |
| `disabled` | `boolean` | `false` | Disable the switch |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { SwitchField, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [enabled, setEnabled] = useState(false);

  return (
    <Canvas ctx={ctx}>
      <SwitchField
        label="Enable notifications"
        value={enabled}
        onChange={setEnabled}
      />
    </Canvas>
  );
}
```

## Common Patterns

### With Help Text

```jsx
<SwitchField
  label="Auto-save"
  value={autoSave}
  onChange={setAutoSave}
  hint="Automatically save changes every 30 seconds"
/>
```

### Settings Form

```jsx
function SettingsForm({ settings, onChange }) {
  const updateSetting = (key, value) => {
    onChange({ ...settings, [key]: value });
  };

  return (
    <Form>
      <Section title="General Settings">
        <SwitchField
          label="Dark mode"
          value={settings.darkMode}
          onChange={(value) => updateSetting('darkMode', value)}
          hint="Use dark theme throughout the interface"
        />
        
        <SwitchField
          label="Show hints"
          value={settings.showHints}
          onChange={(value) => updateSetting('showHints', value)}
          hint="Display helpful tips and suggestions"
        />
        
        <SwitchField
          label="Advanced mode"
          value={settings.advancedMode}
          onChange={(value) => updateSetting('advancedMode', value)}
          hint="Show advanced options and features"
        />
      </Section>
      
      <Section title="Privacy">
        <SwitchField
          label="Analytics"
          value={settings.analytics}
          onChange={(value) => updateSetting('analytics', value)}
          hint="Help us improve by sharing anonymous usage data"
        />
        
        <SwitchField
          label="Crash reports"
          value={settings.crashReports}
          onChange={(value) => updateSetting('crashReports', value)}
          hint="Automatically send crash reports"
        />
      </Section>
    </Form>
  );
}
```

### Conditional Settings

```jsx
function ConditionalSettings() {
  const [mainToggle, setMainToggle] = useState(false);
  const [subOption1, setSubOption1] = useState(false);
  const [subOption2, setSubOption2] = useState(false);

  return (
    <div>
      <SwitchField
        label="Enable advanced features"
        value={mainToggle}
        onChange={setMainToggle}
      />
      
      {mainToggle && (
        <div style={{ 
          marginLeft: 'var(--spacing-l)',
          marginTop: 'var(--spacing-m)',
          paddingLeft: 'var(--spacing-m)',
          borderLeft: '3px solid var(--border-color)'
        }}>
          <SwitchField
            label="Feature optimization"
            value={subOption1}
            onChange={setSubOption1}
            hint="Optimize performance for advanced features"
          />
          
          <SwitchField
            label="Experimental features"
            value={subOption2}
            onChange={setSubOption2}
            hint="Enable features that are still in development"
          />
        </div>
      )}
    </div>
  );
}
```

## Advanced Examples

### Feature Flags

```jsx
function FeatureFlags({ ctx }) {
  const [flags, setFlags] = useState(() => 
    ctx.plugin.attributes.parameters.featureFlags || {}
  );

  const updateFlag = async (flagName, value) => {
    const newFlags = { ...flags, [flagName]: value };
    setFlags(newFlags);
    
    // Save to plugin parameters
    await ctx.updatePluginParameters({
      ...ctx.plugin.attributes.parameters,
      featureFlags: newFlags
    });
    
    ctx.notice(`Feature "${flagName}" ${value ? 'enabled' : 'disabled'}`);
  };

  const features = [
    { 
      id: 'aiAssist', 
      label: 'AI Assistant', 
      hint: 'Enable AI-powered content suggestions' 
    },
    { 
      id: 'bulkActions', 
      label: 'Bulk Actions', 
      hint: 'Allow bulk operations on multiple items' 
    },
    { 
      id: 'advancedSearch', 
      label: 'Advanced Search', 
      hint: 'Enable complex search queries and filters' 
    },
    { 
      id: 'betaFeatures', 
      label: 'Beta Features', 
      hint: 'Access features currently in beta testing' 
    }
  ];

  return (
    <div>
      <h3>Feature Flags</h3>
      {features.map(feature => (
        <SwitchField
          key={feature.id}
          label={feature.label}
          value={flags[feature.id] || false}
          onChange={(value) => updateFlag(feature.id, value)}
          hint={feature.hint}
        />
      ))}
    </div>
  );
}
```

### Permission Toggles

```jsx
function PermissionToggles({ role, permissions, onChange }) {
  const permissionGroups = {
    content: [
      { id: 'create', label: 'Create items' },
      { id: 'edit', label: 'Edit items' },
      { id: 'delete', label: 'Delete items' },
      { id: 'publish', label: 'Publish items' }
    ],
    media: [
      { id: 'upload', label: 'Upload files' },
      { id: 'editMedia', label: 'Edit media' },
      { id: 'deleteMedia', label: 'Delete media' }
    ],
    admin: [
      { id: 'manageUsers', label: 'Manage users' },
      { id: 'manageRoles', label: 'Manage roles' },
      { id: 'viewAnalytics', label: 'View analytics' }
    ]
  };

  return (
    <div>
      <h3>Permissions for {role}</h3>
      
      {Object.entries(permissionGroups).map(([group, perms]) => (
        <Section key={group} title={group.charAt(0).toUpperCase() + group.slice(1)}>
          {perms.map(perm => (
            <SwitchField
              key={perm.id}
              label={perm.label}
              value={permissions[perm.id] || false}
              onChange={(value) => onChange(perm.id, value)}
            />
          ))}
        </Section>
      ))}
    </div>
  );
}
```

### Notification Preferences

```jsx
function NotificationPreferences({ ctx }) {
  const [prefs, setPrefs] = useState({
    email: {
      itemPublished: true,
      itemUpdated: false,
      weeklyDigest: true
    },
    inApp: {
      itemPublished: true,
      itemUpdated: true,
      mentions: true
    }
  });

  const updatePref = (channel, type, value) => {
    setPrefs(prev => ({
      ...prev,
      [channel]: {
        ...prev[channel],
        [type]: value
      }
    }));
  };

  return (
    <div>
      <Section title="Email Notifications">
        <SwitchField
          label="Item published"
          value={prefs.email.itemPublished}
          onChange={(value) => updatePref('email', 'itemPublished', value)}
          hint="Receive email when content is published"
        />
        <SwitchField
          label="Item updated"
          value={prefs.email.itemUpdated}
          onChange={(value) => updatePref('email', 'itemUpdated', value)}
          hint="Receive email when content is updated"
        />
        <SwitchField
          label="Weekly digest"
          value={prefs.email.weeklyDigest}
          onChange={(value) => updatePref('email', 'weeklyDigest', value)}
          hint="Receive weekly summary of activity"
        />
      </Section>
      
      <Section title="In-App Notifications">
        <SwitchField
          label="Item published"
          value={prefs.inApp.itemPublished}
          onChange={(value) => updatePref('inApp', 'itemPublished', value)}
        />
        <SwitchField
          label="Item updated"
          value={prefs.inApp.itemUpdated}
          onChange={(value) => updatePref('inApp', 'itemUpdated', value)}
        />
        <SwitchField
          label="Mentions"
          value={prefs.inApp.mentions}
          onChange={(value) => updatePref('inApp', 'mentions', value)}
          hint="Notify when someone mentions you"
        />
      </Section>
    </div>
  );
}
```

### With Loading State

```jsx
function AsyncSwitchField({ label, hint, loadValue, saveValue }) {
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
      alert('Failed to save setting');
    } finally {
      setSaving(false);
    }
  };

  return (
    <div style={{ opacity: loading ? 0.5 : 1 }}>
      <SwitchField
        label={label}
        value={value}
        onChange={handleChange}
        hint={hint}
        disabled={loading || saving}
      />
      {saving && (
        <div style={{ marginTop: 'var(--spacing-xs)' }}>
          <Spinner size="xs" /> Saving...
        </div>
      )}
    </div>
  );
}
```

### Grouped Switches

```jsx
function GroupedSwitches({ groups }) {
  return (
    <div>
      {groups.map(group => (
        <div 
          key={group.id}
          style={{
            marginBottom: 'var(--spacing-l)',
            padding: 'var(--spacing-m)',
            backgroundColor: 'var(--light-color)',
            borderRadius: 'var(--border-radius-m)'
          }}
        >
          <h4 style={{ marginBottom: 'var(--spacing-m)' }}>
            {group.title}
          </h4>
          
          {group.switches.map(switchItem => (
            <SwitchField
              key={switchItem.id}
              label={switchItem.label}
              value={switchItem.value}
              onChange={switchItem.onChange}
              hint={switchItem.hint}
              disabled={switchItem.disabled}
            />
          ))}
        </div>
      ))}
    </div>
  );
}
```

## Form Integration

```jsx
function CompleteForm({ ctx }) {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    newsletter: false,
    terms: false,
    notifications: true
  });

  const updateField = (field, value) => {
    setFormData({ ...formData, [field]: value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (!formData.terms) {
      ctx.alert('Please accept the terms and conditions');
      return;
    }
    
    ctx.notice('Form submitted successfully!');
  };

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit}>
        <TextField
          label="Name"
          value={formData.name}
          onChange={(value) => updateField('name', value)}
          required
        />
        
        <TextField
          label="Email"
          type="email"
          value={formData.email}
          onChange={(value) => updateField('email', value)}
          required
        />
        
        <FieldGroup>
          <SwitchField
            label="Subscribe to newsletter"
            value={formData.newsletter}
            onChange={(value) => updateField('newsletter', value)}
            hint="Receive weekly updates about new features"
          />
          
          <SwitchField
            label="Enable notifications"
            value={formData.notifications}
            onChange={(value) => updateField('notifications', value)}
            hint="Get notified about important updates"
          />
          
          <SwitchField
            label="I accept the terms and conditions"
            value={formData.terms}
            onChange={(value) => updateField('terms', value)}
            error={!formData.terms ? 'You must accept the terms' : undefined}
          />
        </FieldGroup>
        
        <Button type="submit" buttonType="primary">
          Submit
        </Button>
      </Form>
    </Canvas>
  );
}
```

## Styling

### Custom Styling

```jsx
<SwitchField
  label="Custom styled switch"
  value={value}
  onChange={setValue}
  style={{
    padding: 'var(--spacing-m)',
    backgroundColor: 'var(--light-color)',
    borderRadius: 'var(--border-radius-m)'
  }}
  className="custom-switch-field"
/>
```

## Accessibility

- Proper label association
- Keyboard navigation (Space to toggle)
- ARIA attributes for state
- Focus indicators
- Screen reader support

## TypeScript

```typescript
import { SwitchField } from 'datocms-react-ui';

interface Setting {
  id: string;
  label: string;
  hint?: string;
  defaultValue: boolean;
}

interface SettingsManagerProps {
  settings: Setting[];
  values: Record<string, boolean>;
  onChange: (id: string, value: boolean) => void;
}

function SettingsManager({ settings, values, onChange }: SettingsManagerProps) {
  return (
    <div>
      {settings.map(setting => (
        <SwitchField
          key={setting.id}
          label={setting.label}
          value={values[setting.id] ?? setting.defaultValue}
          onChange={(value) => onChange(setting.id, value)}
          hint={setting.hint}
        />
      ))}
    </div>
  );
}
```

## Best Practices

1. **Clear labels**: Describe what the switch controls
2. **Default states**: Choose sensible defaults
3. **Immediate feedback**: Update UI immediately
4. **Save states**: Persist user preferences
5. **Group related**: Use sections for related switches
6. **Hint text**: Explain implications of toggling

## Related Components

- [SwitchInput](./SwitchInput.md) - Just the switch input
- [TextField](./TextField.md) - Text input fields
- [Form](./Form.md) - Form container
- [Section](./Section.md) - Group related switches