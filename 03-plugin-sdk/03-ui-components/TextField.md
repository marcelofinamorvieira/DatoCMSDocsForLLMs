# TextField Component

A single-line text input field that matches DatoCMS's design system.

## Import

```jsx
import { TextField } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `value` | `string` | - | Current field value |
| `onChange` | `(value: string) => void` | - | Change handler |
| `label` | `string` | - | Field label |
| `placeholder` | `string` | - | Placeholder text |
| `hint` | `string` | - | Help text below field |
| `error` | `string` | - | Error message |
| `required` | `boolean` | `false` | Mark as required |
| `disabled` | `boolean` | `false` | Disable the field |
| `readOnly` | `boolean` | `false` | Make field read-only |
| `type` | `string` | `'text'` | HTML input type |
| `maxLength` | `number` | - | Maximum character length |
| `pattern` | `string` | - | Validation pattern |
| `autoFocus` | `boolean` | `false` | Focus on mount |
| `autoComplete` | `string` | - | Autocomplete hint |
| `prefix` | `ReactNode` | - | Content before input |
| `suffix` | `ReactNode` | - | Content after input |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { TextField, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [value, setValue] = useState('');

  return (
    <Canvas ctx={ctx}>
      <TextField
        label="Title"
        value={value}
        onChange={setValue}
        placeholder="Enter a title..."
      />
    </Canvas>
  );
}
```

## Field States

### Required Field

```jsx
<TextField
  label="Email"
  value={email}
  onChange={setEmail}
  required
  type="email"
  placeholder="user@example.com"
/>
```

### With Help Text

```jsx
<TextField
  label="API Key"
  value={apiKey}
  onChange={setApiKey}
  hint="You can find your API key in the settings"
/>
```

### With Error

```jsx
<TextField
  label="Username"
  value={username}
  onChange={setUsername}
  error={errors.username}
  required
/>
```

### Disabled State

```jsx
<TextField
  label="System ID"
  value={systemId}
  disabled
  hint="This field is automatically generated"
/>
```

### Read-Only State

```jsx
<TextField
  label="Created At"
  value={createdAt}
  readOnly
/>
```

## Input Types

### Email Input

```jsx
<TextField
  label="Email Address"
  type="email"
  value={email}
  onChange={setEmail}
  autoComplete="email"
/>
```

### URL Input

```jsx
<TextField
  label="Website"
  type="url"
  value={website}
  onChange={setWebsite}
  placeholder="https://example.com"
/>
```

### Number Input

```jsx
<TextField
  label="Quantity"
  type="number"
  value={quantity}
  onChange={setQuantity}
  min="1"
  max="100"
/>
```

### Password Input

```jsx
<TextField
  label="Password"
  type="password"
  value={password}
  onChange={setPassword}
  autoComplete="new-password"
/>
```

## With Prefix/Suffix

### URL with Protocol

```jsx
<TextField
  label="Domain"
  value={domain}
  onChange={setDomain}
  prefix="https://"
  suffix=".com"
  placeholder="example"
/>
```

### Price Field

```jsx
<TextField
  label="Price"
  type="number"
  value={price}
  onChange={setPrice}
  prefix="$"
  suffix="USD"
  step="0.01"
/>
```

### With Icons

```jsx
import { FaSearch, FaTimes } from 'react-icons/fa';

<TextField
  label="Search"
  value={search}
  onChange={setSearch}
  prefix={<FaSearch />}
  suffix={
    search && (
      <button onClick={() => setSearch('')}>
        <FaTimes />
      </button>
    )
  }
/>
```

## Validation

### Character Limit

```jsx
<TextField
  label="Short Description"
  value={description}
  onChange={setDescription}
  maxLength={160}
  hint={`${description.length}/160 characters`}
  error={description.length > 160 ? 'Too long' : undefined}
/>
```

### Pattern Validation

```jsx
<TextField
  label="SKU"
  value={sku}
  onChange={setSku}
  pattern="[A-Z]{3}-[0-9]{4}"
  hint="Format: ABC-1234"
  error={!isValidSku(sku) ? 'Invalid SKU format' : undefined}
/>
```

### Custom Validation

```jsx
function ValidatedField({ label, validate, ...props }) {
  const [value, setValue] = useState('');
  const [error, setError] = useState('');

  const handleChange = (newValue) => {
    setValue(newValue);
    const validationError = validate(newValue);
    setError(validationError || '');
  };

  return (
    <TextField
      label={label}
      value={value}
      onChange={handleChange}
      error={error}
      {...props}
    />
  );
}

// Usage
<ValidatedField
  label="Slug"
  validate={(value) => {
    if (!value) return 'Required';
    if (!/^[a-z0-9-]+$/.test(value)) {
      return 'Only lowercase letters, numbers, and hyphens';
    }
    return null;
  }}
/>
```

## Advanced Examples

### Debounced Input

```jsx
import { useMemo, useCallback } from 'react';
import debounce from 'lodash/debounce';

function DebouncedTextField({ onDebouncedChange, delay = 300, ...props }) {
  const [localValue, setLocalValue] = useState(props.value || '');

  const debouncedChange = useMemo(
    () => debounce(onDebouncedChange, delay),
    [onDebouncedChange, delay]
  );

  const handleChange = useCallback((value) => {
    setLocalValue(value);
    debouncedChange(value);
  }, [debouncedChange]);

  return (
    <TextField
      {...props}
      value={localValue}
      onChange={handleChange}
    />
  );
}
```

### Auto-Save Field

```jsx
function AutoSaveField({ ctx, fieldPath, ...props }) {
  const [saving, setSaving] = useState(false);
  const value = ctx.formValues[fieldPath] || '';

  const handleChange = async (newValue) => {
    ctx.setFieldValue(fieldPath, newValue);
    
    setSaving(true);
    try {
      await ctx.saveCurrentItem();
      ctx.notice('Saved');
    } catch (error) {
      ctx.alert('Failed to save');
    } finally {
      setSaving(false);
    }
  };

  return (
    <TextField
      {...props}
      value={value}
      onChange={handleChange}
      suffix={saving ? <Spinner size="xs" /> : null}
    />
  );
}
```

### Connected Field

```jsx
function ConnectedTextField({ ctx, fieldPath, ...props }) {
  const value = ctx.formValues[fieldPath] || '';
  const fieldErrors = ctx.fieldErrors[fieldPath];

  return (
    <TextField
      value={value}
      onChange={(newValue) => ctx.setFieldValue(fieldPath, newValue)}
      error={fieldErrors?.length > 0 ? fieldErrors[0] : undefined}
      disabled={ctx.disabled}
      {...props}
    />
  );
}
```

## Form Integration

```jsx
import { Form, FieldGroup, TextField, Button } from 'datocms-react-ui';

function ContactForm({ ctx }) {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    // Handle submission
  };

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit}>
        <FieldGroup>
          <TextField
            label="Name"
            value={formData.name}
            onChange={(name) => setFormData({ ...formData, name })}
            required
          />
          
          <TextField
            label="Email"
            type="email"
            value={formData.email}
            onChange={(email) => setFormData({ ...formData, email })}
            required
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
<TextField
  label="Custom Field"
  value={value}
  onChange={setValue}
  style={{ marginBottom: 'var(--spacing-l)' }}
  className="my-custom-field"
/>
```

### CSS Variables

```css
--input-padding
--input-font-size
--input-line-height
--input-border-color
--input-focus-border-color
--input-error-border-color
--input-background-color
--input-disabled-background-color
```

## Accessibility

- Proper label association
- ARIA attributes for errors and hints
- Keyboard navigation
- Screen reader announcements
- Required field indication

## TypeScript

```typescript
import { TextField, TextFieldProps } from 'datocms-react-ui';

const fieldProps: TextFieldProps = {
  label: 'Name',
  value: '',
  onChange: (value: string) => console.log(value),
  required: true,
  error: 'This field is required'
};

<TextField {...fieldProps} />
```

## Best Practices

1. **Always provide labels**: For accessibility and usability
2. **Use appropriate input types**: email, url, number, etc.
3. **Provide helpful hints**: Guide users on expected format
4. **Show validation errors clearly**: Near the field that has the error
5. **Indicate required fields**: Use the `required` prop
6. **Debounce expensive operations**: Like API calls or validations
7. **Use placeholders sparingly**: They disappear when typing

## Related Components

- [TextareaField](./TextareaField.md) - Multi-line text input
- [SelectField](./SelectField.md) - Dropdown selection
- [Form](./Form.md) - Form container
- [FieldGroup](./FieldGroup.md) - Group related fields
- [FieldError](./FieldError.md) - Error messages
- [FieldHint](./FieldHint.md) - Help text