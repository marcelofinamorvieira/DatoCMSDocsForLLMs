# FieldGroup Component

A container component that groups related form fields together with consistent spacing and optional visual grouping.

## Import

```jsx
import { FieldGroup } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Form fields to group |
| `direction` | `'vertical' \| 'horizontal'` | `'vertical'` | Layout direction |
| `gap` | `'xs' \| 's' \| 'm' \| 'l' \| 'xl'` | `'m'` | Space between fields |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { Form, FieldGroup, TextField, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Form>
        <FieldGroup>
          <TextField label="First Name" />
          <TextField label="Last Name" />
        </FieldGroup>
        
        <TextField label="Email" type="email" />
      </Form>
    </Canvas>
  );
}
```

## Layout Options

### Vertical Layout (Default)

```jsx
<FieldGroup direction="vertical">
  <TextField label="Street Address" />
  <TextField label="City" />
  <TextField label="Postal Code" />
</FieldGroup>
```

### Horizontal Layout

```jsx
<FieldGroup direction="horizontal">
  <TextField label="First Name" />
  <TextField label="Last Name" />
</FieldGroup>
```

### Custom Gap Spacing

```jsx
// Extra small gap
<FieldGroup gap="xs">
  <SwitchField label="Option 1" />
  <SwitchField label="Option 2" />
  <SwitchField label="Option 3" />
</FieldGroup>

// Large gap
<FieldGroup gap="l">
  <TextField label="Title" />
  <TextareaField label="Description" />
</FieldGroup>
```

## Common Patterns

### Name Fields

```jsx
function NameFields({ firstName, lastName, onChange }) {
  return (
    <FieldGroup direction="horizontal">
      <TextField
        label="First Name"
        value={firstName}
        onChange={(value) => onChange('firstName', value)}
        required
      />
      <TextField
        label="Last Name" 
        value={lastName}
        onChange={(value) => onChange('lastName', value)}
        required
      />
    </FieldGroup>
  );
}
```

### Address Form

```jsx
function AddressForm({ address, onChange }) {
  return (
    <>
      <TextField
        label="Street Address"
        value={address.street}
        onChange={(street) => onChange({ ...address, street })}
      />
      
      <FieldGroup direction="horizontal">
        <TextField
          label="City"
          value={address.city}
          onChange={(city) => onChange({ ...address, city })}
        />
        <TextField
          label="State/Province"
          value={address.state}
          onChange={(state) => onChange({ ...address, state })}
        />
        <TextField
          label="Postal Code"
          value={address.postalCode}
          onChange={(postalCode) => onChange({ ...address, postalCode })}
        />
      </FieldGroup>
      
      <SelectField
        label="Country"
        value={address.country}
        onChange={(country) => onChange({ ...address, country })}
      >
        <option value="us">United States</option>
        <option value="ca">Canada</option>
        <option value="uk">United Kingdom</option>
      </SelectField>
    </>
  );
}
```

### Date Range Fields

```jsx
function DateRangeFields({ startDate, endDate, onDateChange }) {
  return (
    <FieldGroup direction="horizontal">
      <TextField
        label="Start Date"
        type="date"
        value={startDate}
        onChange={(value) => onDateChange('start', value)}
        required
      />
      <TextField
        label="End Date"
        type="date"
        value={endDate}
        onChange={(value) => onDateChange('end', value)}
        min={startDate}
        required
      />
    </FieldGroup>
  );
}
```

### Related Options

```jsx
function RelatedOptions({ settings, onChange }) {
  return (
    <FieldGroup gap="s">
      <SwitchField
        label="Enable Notifications"
        value={settings.notifications}
        onChange={(notifications) => onChange({ ...settings, notifications })}
      />
      
      {settings.notifications && (
        <FieldGroup direction="horizontal" gap="s" style={{ marginLeft: 'var(--spacing-l)' }}>
          <SwitchField
            label="Email"
            value={settings.emailNotifications}
            onChange={(emailNotifications) => onChange({ ...settings, emailNotifications })}
          />
          <SwitchField
            label="SMS"
            value={settings.smsNotifications}
            onChange={(smsNotifications) => onChange({ ...settings, smsNotifications })}
          />
          <SwitchField
            label="Push"
            value={settings.pushNotifications}
            onChange={(pushNotifications) => onChange({ ...settings, pushNotifications })}
          />
        </FieldGroup>
      )}
    </FieldGroup>
  );
}
```

## Advanced Examples

### Responsive Field Groups

```jsx
function ResponsiveFieldGroup({ children }) {
  const isMobile = useMediaQuery('(max-width: 768px)');
  
  return (
    <FieldGroup direction={isMobile ? 'vertical' : 'horizontal'}>
      {children}
    </FieldGroup>
  );
}
```

### Nested Field Groups

```jsx
function ComplexForm({ data, onChange }) {
  return (
    <Form>
      <Section title="Personal Information">
        <FieldGroup>
          <FieldGroup direction="horizontal">
            <TextField label="First Name" />
            <TextField label="Last Name" />
          </FieldGroup>
          
          <TextField label="Email" type="email" />
          
          <FieldGroup direction="horizontal">
            <TextField label="Phone" type="tel" />
            <TextField label="Extension" style={{ maxWidth: '100px' }} />
          </FieldGroup>
        </FieldGroup>
      </Section>
      
      <Section title="Preferences">
        <FieldGroup gap="s">
          <SwitchField label="Newsletter" />
          <SwitchField label="Product Updates" />
          <SwitchField label="Marketing Emails" />
        </FieldGroup>
      </Section>
    </Form>
  );
}
```

### Dynamic Field Groups

```jsx
function DynamicFieldGroups({ items, onItemsChange }) {
  const addItem = () => {
    onItemsChange([...items, { id: Date.now(), name: '', value: '' }]);
  };
  
  const removeItem = (id) => {
    onItemsChange(items.filter(item => item.id !== id));
  };
  
  const updateItem = (id, field, value) => {
    onItemsChange(items.map(item => 
      item.id === id ? { ...item, [field]: value } : item
    ));
  };
  
  return (
    <>
      {items.map((item) => (
        <FieldGroup key={item.id} direction="horizontal">
          <TextField
            label="Name"
            value={item.name}
            onChange={(value) => updateItem(item.id, 'name', value)}
          />
          <TextField
            label="Value"
            value={item.value}
            onChange={(value) => updateItem(item.id, 'value', value)}
          />
          <Button
            onClick={() => removeItem(item.id)}
            buttonType="negative"
            size="s"
          >
            Remove
          </Button>
        </FieldGroup>
      ))}
      
      <Button onClick={addItem} buttonType="muted">
        Add Item
      </Button>
    </>
  );
}
```

### Conditional Field Groups

```jsx
function ConditionalFieldGroups({ userType, formData, onChange }) {
  return (
    <Form>
      <SelectField
        label="User Type"
        value={userType}
        onChange={onChange}
      >
        <option value="individual">Individual</option>
        <option value="company">Company</option>
      </SelectField>
      
      {userType === 'individual' ? (
        <FieldGroup>
          <FieldGroup direction="horizontal">
            <TextField label="First Name" required />
            <TextField label="Last Name" required />
          </FieldGroup>
          <TextField label="Personal Email" type="email" required />
        </FieldGroup>
      ) : (
        <FieldGroup>
          <TextField label="Company Name" required />
          <FieldGroup direction="horizontal">
            <TextField label="VAT Number" />
            <TextField label="Registration Number" />
          </FieldGroup>
          <TextField label="Company Email" type="email" required />
        </FieldGroup>
      )}
    </Form>
  );
}
```

## Styling

### Custom Styling

```jsx
<FieldGroup
  style={{
    padding: 'var(--spacing-m)',
    backgroundColor: 'var(--light-color)',
    borderRadius: 'var(--border-radius-m)',
    border: '1px solid var(--border-color)'
  }}
  className="custom-field-group"
>
  <TextField label="Field 1" />
  <TextField label="Field 2" />
</FieldGroup>
```

### With Visual Separator

```jsx
function SeparatedFieldGroups() {
  return (
    <Form>
      <FieldGroup>
        <TextField label="Field 1" />
        <TextField label="Field 2" />
      </FieldGroup>
      
      <hr style={{ 
        margin: 'var(--spacing-l) 0',
        border: 'none',
        borderTop: '1px solid var(--border-color)'
      }} />
      
      <FieldGroup>
        <TextField label="Field 3" />
        <TextField label="Field 4" />
      </FieldGroup>
    </Form>
  );
}
```

## Accessibility

- Maintains proper tab order
- Groups are semantically related
- Screen readers understand field relationships
- Keyboard navigation works naturally

## TypeScript

```typescript
import { FieldGroup, TextField } from 'datocms-react-ui';

interface PersonFormData {
  firstName: string;
  lastName: string;
  email: string;
}

interface PersonFieldsProps {
  data: PersonFormData;
  onChange: (data: PersonFormData) => void;
}

function PersonFields({ data, onChange }: PersonFieldsProps) {
  return (
    <FieldGroup>
      <FieldGroup direction="horizontal">
        <TextField
          label="First Name"
          value={data.firstName}
          onChange={(firstName) => onChange({ ...data, firstName })}
        />
        <TextField
          label="Last Name"
          value={data.lastName}
          onChange={(lastName) => onChange({ ...data, lastName })}
        />
      </FieldGroup>
      
      <TextField
        label="Email"
        type="email"
        value={data.email}
        onChange={(email) => onChange({ ...data, email })}
      />
    </FieldGroup>
  );
}
```

## Best Practices

1. **Group related fields**: Only group fields that are logically related
2. **Consistent spacing**: Use the same gap size within a form
3. **Responsive design**: Consider mobile layouts for horizontal groups
4. **Limit nesting**: Avoid deeply nested field groups
5. **Clear hierarchy**: Use sections for major groupings, field groups for minor
6. **Accessibility**: Ensure grouped fields make sense when read sequentially

## Related Components

- [Form](./Form.md) - Form container
- [TextField](./TextField.md) - Text input fields
- [Section](./Section.md) - Larger content sections
- [FieldWrapper](./FieldWrapper.md) - Individual field wrapper