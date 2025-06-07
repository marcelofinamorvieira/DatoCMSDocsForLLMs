# FieldWrapper Component

A container component that provides consistent spacing and structure for form fields, including labels, inputs, hints, and errors.

## Import

```jsx
import { FieldWrapper } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Field content (input, label, hints, errors) |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { FieldWrapper, FormLabel, TextInput, FieldHint, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <FieldWrapper>
        <FormLabel>Custom Field</FormLabel>
        <TextInput placeholder="Enter value" />
        <FieldHint>This is a helpful hint</FieldHint>
      </FieldWrapper>
    </Canvas>
  );
}
```

## Common Patterns

### Complete Field Structure

```jsx
function CompleteField({ label, value, onChange, hint, error, required }) {
  return (
    <FieldWrapper>
      <FormLabel required={required}>{label}</FormLabel>
      <TextInput 
        value={value}
        onChange={(e) => onChange(e.target.value)}
        error={!!error}
      />
      {hint && <FieldHint>{hint}</FieldHint>}
      {error && <FieldError>{error}</FieldError>}
    </FieldWrapper>
  );
}
```

### Custom Input Components

```jsx
function ColorPickerField({ label, value, onChange }) {
  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      <div style={{ display: 'flex', gap: 'var(--spacing-s)', alignItems: 'center' }}>
        <input
          type="color"
          value={value}
          onChange={(e) => onChange(e.target.value)}
          style={{
            width: '50px',
            height: '40px',
            border: '1px solid var(--border-color)',
            borderRadius: 'var(--border-radius-s)',
            cursor: 'pointer'
          }}
        />
        <TextInput
          value={value}
          onChange={(e) => onChange(e.target.value)}
          placeholder="#000000"
          style={{ flex: 1 }}
        />
      </div>
      <FieldHint>Choose a color or enter a hex value</FieldHint>
    </FieldWrapper>
  );
}
```

### Radio Button Group

```jsx
function RadioField({ label, options, value, onChange }) {
  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      <div role="radiogroup" aria-labelledby={label}>
        {options.map((option) => (
          <label 
            key={option.value}
            style={{ 
              display: 'flex', 
              alignItems: 'center',
              marginBottom: 'var(--spacing-xs)',
              cursor: 'pointer'
            }}
          >
            <input
              type="radio"
              name={label}
              value={option.value}
              checked={value === option.value}
              onChange={() => onChange(option.value)}
              style={{ marginRight: 'var(--spacing-xs)' }}
            />
            {option.label}
          </label>
        ))}
      </div>
    </FieldWrapper>
  );
}
```

### Checkbox Group

```jsx
function CheckboxGroupField({ label, options, values, onChange }) {
  const toggleOption = (optionValue) => {
    if (values.includes(optionValue)) {
      onChange(values.filter(v => v !== optionValue));
    } else {
      onChange([...values, optionValue]);
    }
  };

  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      <div role="group" aria-labelledby={label}>
        {options.map((option) => (
          <label 
            key={option.value}
            style={{ 
              display: 'flex', 
              alignItems: 'center',
              marginBottom: 'var(--spacing-xs)',
              cursor: 'pointer'
            }}
          >
            <input
              type="checkbox"
              value={option.value}
              checked={values.includes(option.value)}
              onChange={() => toggleOption(option.value)}
              style={{ marginRight: 'var(--spacing-xs)' }}
            />
            {option.label}
          </label>
        ))}
      </div>
      <FieldHint>Select all that apply</FieldHint>
    </FieldWrapper>
  );
}
```

## Advanced Examples

### Range Slider Field

```jsx
function RangeField({ label, value, onChange, min = 0, max = 100, step = 1 }) {
  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      <div style={{ display: 'flex', gap: 'var(--spacing-s)', alignItems: 'center' }}>
        <input
          type="range"
          min={min}
          max={max}
          step={step}
          value={value}
          onChange={(e) => onChange(Number(e.target.value))}
          style={{ flex: 1 }}
        />
        <span style={{ 
          minWidth: '50px', 
          textAlign: 'right',
          fontWeight: 'bold' 
        }}>
          {value}
        </span>
      </div>
      <FieldHint>
        Adjust between {min} and {max}
      </FieldHint>
    </FieldWrapper>
  );
}
```

### File Upload Field

```jsx
function FileUploadField({ label, onChange, accept, maxSize }) {
  const [file, setFile] = useState(null);
  const [error, setError] = useState('');

  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    
    if (selectedFile) {
      if (maxSize && selectedFile.size > maxSize) {
        setError(`File size must be less than ${maxSize / 1024 / 1024}MB`);
        return;
      }
      
      setFile(selectedFile);
      setError('');
      onChange(selectedFile);
    }
  };

  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      <div style={{
        border: '2px dashed var(--border-color)',
        borderRadius: 'var(--border-radius-m)',
        padding: 'var(--spacing-l)',
        textAlign: 'center',
        cursor: 'pointer'
      }}>
        <input
          type="file"
          accept={accept}
          onChange={handleFileChange}
          style={{ display: 'none' }}
          id={`file-${label}`}
        />
        <label htmlFor={`file-${label}`} style={{ cursor: 'pointer' }}>
          {file ? (
            <div>
              <strong>{file.name}</strong>
              <br />
              <small>{(file.size / 1024).toFixed(2)} KB</small>
            </div>
          ) : (
            <div>
              <FaUpload size={24} />
              <br />
              Click to upload or drag and drop
            </div>
          )}
        </label>
      </div>
      {error && <FieldError>{error}</FieldError>}
      <FieldHint>
        {accept ? `Accepted formats: ${accept}` : 'All file types accepted'}
      </FieldHint>
    </FieldWrapper>
  );
}
```

### Date/Time Picker

```jsx
function DateTimeField({ label, value, onChange, includeTime = false }) {
  const [date, setDate] = useState(value?.split('T')[0] || '');
  const [time, setTime] = useState(value?.split('T')[1]?.slice(0, 5) || '');

  const updateDateTime = (newDate, newTime) => {
    if (includeTime && newDate && newTime) {
      onChange(`${newDate}T${newTime}`);
    } else if (newDate) {
      onChange(newDate);
    }
  };

  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      <div style={{ display: 'flex', gap: 'var(--spacing-s)' }}>
        <input
          type="date"
          value={date}
          onChange={(e) => {
            setDate(e.target.value);
            updateDateTime(e.target.value, time);
          }}
          style={{
            flex: includeTime ? 1 : undefined,
            padding: 'var(--spacing-s)',
            border: '1px solid var(--border-color)',
            borderRadius: 'var(--border-radius-s)'
          }}
        />
        {includeTime && (
          <input
            type="time"
            value={time}
            onChange={(e) => {
              setTime(e.target.value);
              updateDateTime(date, e.target.value);
            }}
            style={{
              padding: 'var(--spacing-s)',
              border: '1px solid var(--border-color)',
              borderRadius: 'var(--border-radius-s)'
            }}
          />
        )}
      </div>
      <FieldHint>
        {includeTime ? 'Select date and time' : 'Select a date'}
      </FieldHint>
    </FieldWrapper>
  );
}
```

### Toggle with Description

```jsx
function ToggleField({ label, description, value, onChange }) {
  return (
    <FieldWrapper>
      <div style={{ 
        display: 'flex', 
        justifyContent: 'space-between',
        alignItems: 'flex-start'
      }}>
        <div style={{ flex: 1 }}>
          <FormLabel>{label}</FormLabel>
          <FieldHint>{description}</FieldHint>
        </div>
        <SwitchInput
          value={value}
          onChange={onChange}
        />
      </div>
    </FieldWrapper>
  );
}
```

### Custom Complex Field

```jsx
function AddressField({ value, onChange }) {
  const updateField = (field, fieldValue) => {
    onChange({ ...value, [field]: fieldValue });
  };

  return (
    <FieldWrapper>
      <FormLabel>Address</FormLabel>
      
      <div style={{ display: 'grid', gap: 'var(--spacing-s)' }}>
        <TextInput
          placeholder="Street address"
          value={value.street || ''}
          onChange={(e) => updateField('street', e.target.value)}
        />
        
        <div style={{ display: 'grid', gridTemplateColumns: '2fr 1fr', gap: 'var(--spacing-s)' }}>
          <TextInput
            placeholder="City"
            value={value.city || ''}
            onChange={(e) => updateField('city', e.target.value)}
          />
          <TextInput
            placeholder="State"
            value={value.state || ''}
            onChange={(e) => updateField('state', e.target.value)}
          />
        </div>
        
        <div style={{ display: 'grid', gridTemplateColumns: '1fr 2fr', gap: 'var(--spacing-s)' }}>
          <TextInput
            placeholder="ZIP"
            value={value.zip || ''}
            onChange={(e) => updateField('zip', e.target.value)}
          />
          <SelectInput
            value={value.country || ''}
            onChange={(e) => updateField('country', e.target.value)}
          >
            <option value="">Select country</option>
            <option value="us">United States</option>
            <option value="ca">Canada</option>
            <option value="uk">United Kingdom</option>
          </SelectInput>
        </div>
      </div>
      
      <FieldHint>
        Enter your complete mailing address
      </FieldHint>
    </FieldWrapper>
  );
}
```

## Styling

### Custom Wrapper Styling

```jsx
<FieldWrapper
  style={{
    backgroundColor: 'var(--light-color)',
    padding: 'var(--spacing-m)',
    borderRadius: 'var(--border-radius-m)',
    border: '1px solid var(--border-color)'
  }}
  className="highlighted-field"
>
  {/* Field content */}
</FieldWrapper>
```

## Accessibility

- Provides semantic structure for form fields
- Ensures proper spacing for touch targets
- Maintains consistent visual hierarchy
- Supports screen reader navigation

## TypeScript

```typescript
import { FieldWrapper, FormLabel, TextInput, FieldHint, FieldError } from 'datocms-react-ui';

interface CustomFieldProps {
  label: string;
  value: string;
  onChange: (value: string) => void;
  hint?: string;
  error?: string;
  required?: boolean;
}

function CustomField({
  label,
  value,
  onChange,
  hint,
  error,
  required = false
}: CustomFieldProps) {
  return (
    <FieldWrapper>
      <FormLabel required={required}>{label}</FormLabel>
      <TextInput
        value={value}
        onChange={(e) => onChange(e.target.value)}
        error={!!error}
      />
      {hint && <FieldHint>{hint}</FieldHint>}
      {error && <FieldError>{error}</FieldError>}
    </FieldWrapper>
  );
}
```

## Best Practices

1. **Consistent structure**: Always use the same order: label, input, hint, error
2. **Single responsibility**: One field per wrapper
3. **Clear labeling**: Always include a label for accessibility
4. **Helpful hints**: Provide guidance when needed
5. **Error handling**: Show errors clearly below the input
6. **Spacing**: Let FieldWrapper handle vertical spacing

## Related Components

- [FormLabel](./FormLabel.md) - Field labels
- [FieldHint](./FieldHint.md) - Help text
- [FieldError](./FieldError.md) - Error messages
- [TextInput](./TextInput.md) - Text input component
- [Form](./Form.md) - Form container