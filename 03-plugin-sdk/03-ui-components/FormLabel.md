# FormLabel Component

A label component for form fields that provides consistent styling and accessibility features matching DatoCMS's design system.

## Import

```jsx
import { FormLabel } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Label text content |
| `htmlFor` | `string` | - | ID of the associated form element |
| `required` | `boolean` | `false` | Show required indicator |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { FormLabel, TextInput, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <div>
        <FormLabel htmlFor="email">Email Address</FormLabel>
        <TextInput id="email" type="email" />
      </div>
    </Canvas>
  );
}
```

## Common Patterns

### Required Fields

```jsx
function RequiredField() {
  return (
    <div>
      <FormLabel htmlFor="username" required>
        Username
      </FormLabel>
      <TextInput 
        id="username" 
        required
        aria-required="true"
      />
    </div>
  );
}
```

### With Field Components

```jsx
function LabeledTextField({ label, id, required, ...props }) {
  return (
    <div>
      <FormLabel htmlFor={id} required={required}>
        {label}
      </FormLabel>
      <TextField 
        id={id}
        required={required}
        {...props}
      />
    </div>
  );
}
```

### Custom Form Controls

```jsx
function CustomSlider({ label, value, onChange }) {
  const id = `slider-${label.toLowerCase().replace(/\s+/g, '-')}`;
  
  return (
    <div>
      <FormLabel htmlFor={id}>
        {label}: {value}%
      </FormLabel>
      <input
        id={id}
        type="range"
        min="0"
        max="100"
        value={value}
        onChange={(e) => onChange(Number(e.target.value))}
        style={{
          width: '100%',
          marginTop: 'var(--spacing-xs)'
        }}
      />
    </div>
  );
}
```

## Advanced Examples

### Label with Helper Icon

```jsx
import { FaInfoCircle } from 'react-icons/fa';

function LabelWithHelp({ children, helpText, htmlFor }) {
  const [showHelp, setShowHelp] = useState(false);

  return (
    <div>
      <FormLabel htmlFor={htmlFor}>
        {children}
        <button
          type="button"
          onClick={() => setShowHelp(!showHelp)}
          style={{
            marginLeft: 'var(--spacing-xs)',
            background: 'none',
            border: 'none',
            cursor: 'pointer',
            color: 'var(--muted-color)'
          }}
          aria-label="Show help"
        >
          <FaInfoCircle />
        </button>
      </FormLabel>
      {showHelp && (
        <FieldHint>{helpText}</FieldHint>
      )}
    </div>
  );
}
```

### Label with Character Count

```jsx
function LabelWithCount({ label, currentLength, maxLength, htmlFor }) {
  return (
    <div style={{ 
      display: 'flex', 
      justifyContent: 'space-between',
      alignItems: 'baseline'
    }}>
      <FormLabel htmlFor={htmlFor}>{label}</FormLabel>
      <small style={{ color: 'var(--muted-color)' }}>
        {currentLength}/{maxLength}
      </small>
    </div>
  );
}
```

### Accessible Radio Group

```jsx
function RadioGroupWithLabels({ legend, name, options, value, onChange }) {
  return (
    <fieldset>
      <legend>
        <FormLabel>{legend}</FormLabel>
      </legend>
      {options.map((option) => {
        const id = `${name}-${option.value}`;
        return (
          <div key={option.value} style={{ marginBottom: 'var(--spacing-xs)' }}>
            <input
              type="radio"
              id={id}
              name={name}
              value={option.value}
              checked={value === option.value}
              onChange={() => onChange(option.value)}
            />
            <FormLabel 
              htmlFor={id}
              style={{ 
                marginLeft: 'var(--spacing-xs)',
                fontWeight: 'normal',
                cursor: 'pointer'
              }}
            >
              {option.label}
            </FormLabel>
          </div>
        );
      })}
    </fieldset>
  );
}
```

### Complex Label Content

```jsx
function RichLabel({ htmlFor }) {
  return (
    <FormLabel htmlFor={htmlFor}>
      <span>
        API Endpoint
        <code style={{
          marginLeft: 'var(--spacing-xs)',
          padding: '2px 6px',
          backgroundColor: 'var(--light-color)',
          borderRadius: 'var(--border-radius-s)',
          fontSize: 'var(--font-size-s)'
        }}>
          POST /api/v1/items
        </code>
      </span>
    </FormLabel>
  );
}
```

### Checkbox with Label

```jsx
function CheckboxField({ label, checked, onChange }) {
  const id = `checkbox-${label.toLowerCase().replace(/\s+/g, '-')}`;
  
  return (
    <div style={{ display: 'flex', alignItems: 'center' }}>
      <input
        type="checkbox"
        id={id}
        checked={checked}
        onChange={(e) => onChange(e.target.checked)}
      />
      <FormLabel 
        htmlFor={id}
        style={{ 
          marginLeft: 'var(--spacing-xs)',
          marginBottom: 0,
          cursor: 'pointer'
        }}
      >
        {label}
      </FormLabel>
    </div>
  );
}
```

### Label Groups

```jsx
function LabeledFieldGroup({ groupLabel, fields }) {
  return (
    <div>
      <FormLabel style={{ 
        fontSize: 'var(--font-size-l)',
        marginBottom: 'var(--spacing-m)'
      }}>
        {groupLabel}
      </FormLabel>
      
      <div style={{ 
        paddingLeft: 'var(--spacing-m)',
        borderLeft: '3px solid var(--border-color)'
      }}>
        {fields.map((field) => (
          <div key={field.id} style={{ marginBottom: 'var(--spacing-m)' }}>
            <FormLabel htmlFor={field.id} required={field.required}>
              {field.label}
            </FormLabel>
            <TextInput id={field.id} {...field.props} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Styling Variations

### Inline Labels

```jsx
function InlineLabel({ label, children }) {
  return (
    <div style={{ 
      display: 'flex', 
      alignItems: 'center',
      gap: 'var(--spacing-m)'
    }}>
      <FormLabel style={{ marginBottom: 0, minWidth: '120px' }}>
        {label}
      </FormLabel>
      {children}
    </div>
  );
}
```

### Small Labels

```jsx
function SmallLabel({ children, htmlFor }) {
  return (
    <FormLabel 
      htmlFor={htmlFor}
      style={{ 
        fontSize: 'var(--font-size-s)',
        textTransform: 'uppercase',
        letterSpacing: '0.05em',
        color: 'var(--muted-color)'
      }}
    >
      {children}
    </FormLabel>
  );
}
```

### Label with Badge

```jsx
function LabelWithBadge({ label, badge, htmlFor }) {
  return (
    <FormLabel htmlFor={htmlFor}>
      {label}
      {badge && (
        <span style={{
          marginLeft: 'var(--spacing-xs)',
          padding: '2px 8px',
          backgroundColor: 'var(--primary-color)',
          color: 'white',
          borderRadius: 'var(--border-radius-s)',
          fontSize: 'var(--font-size-xs)',
          fontWeight: 'normal'
        }}>
          {badge}
        </span>
      )}
    </FormLabel>
  );
}

// Usage
<LabelWithBadge 
  label="Feature Flag" 
  badge="Beta" 
  htmlFor="feature-flag" 
/>
```

## Accessibility Features

### Screen Reader Support

```jsx
function AccessibleField({ label, description, error }) {
  const fieldId = 'field-1';
  const descId = `${fieldId}-desc`;
  const errorId = `${fieldId}-error`;

  return (
    <div>
      <FormLabel htmlFor={fieldId}>{label}</FormLabel>
      {description && (
        <FieldHint id={descId}>{description}</FieldHint>
      )}
      <TextInput
        id={fieldId}
        aria-describedby={[description && descId, error && errorId].filter(Boolean).join(' ')}
        aria-invalid={!!error}
      />
      {error && (
        <FieldError id={errorId}>{error}</FieldError>
      )}
    </div>
  );
}
```

## TypeScript

```typescript
import { FormLabel } from 'datocms-react-ui';
import { ReactNode } from 'react';

interface LabeledFieldProps {
  label: ReactNode;
  fieldId: string;
  required?: boolean;
  help?: string;
  children: ReactNode;
}

function LabeledField({ 
  label, 
  fieldId, 
  required = false,
  help,
  children 
}: LabeledFieldProps) {
  return (
    <div>
      <FormLabel htmlFor={fieldId} required={required}>
        {label}
      </FormLabel>
      {help && <FieldHint>{help}</FieldHint>}
      {children}
    </div>
  );
}
```

## Best Practices

1. **Always use htmlFor**: Connect labels to their inputs
2. **Indicate required fields**: Use the `required` prop
3. **Keep labels concise**: Short, descriptive labels
4. **Consistent capitalization**: Use sentence case
5. **Avoid placeholder as label**: Always use proper labels
6. **Group related fields**: Use fieldsets with legends

## Related Components

- [TextField](./TextField.md) - Text input fields
- [FieldWrapper](./FieldWrapper.md) - Field container
- [FieldHint](./FieldHint.md) - Help text
- [FieldError](./FieldError.md) - Error messages
- [Form](./Form.md) - Form container