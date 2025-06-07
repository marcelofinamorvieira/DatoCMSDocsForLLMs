# FieldError Component

A component for displaying field validation errors with consistent styling that matches DatoCMS's design system.

## Import

```jsx
import { FieldError } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Error message content |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { TextField, FieldError, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');

  const validateEmail = (value) => {
    if (!value) {
      setError('Email is required');
    } else if (!/\S+@\S+\.\S+/.test(value)) {
      setError('Please enter a valid email address');
    } else {
      setError('');
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div>
        <TextField
          label="Email"
          value={email}
          onChange={(value) => {
            setEmail(value);
            validateEmail(value);
          }}
          error={!!error}
        />
        {error && <FieldError>{error}</FieldError>}
      </div>
    </Canvas>
  );
}
```

## Integration with Form Fields

### With TextField

```jsx
function ValidatedTextField({ label, value, onChange, validate }) {
  const [error, setError] = useState('');
  const [touched, setTouched] = useState(false);

  const handleBlur = () => {
    setTouched(true);
    if (validate) {
      setError(validate(value) || '');
    }
  };

  const handleChange = (newValue) => {
    onChange(newValue);
    if (touched && validate) {
      setError(validate(newValue) || '');
    }
  };

  return (
    <div>
      <TextField
        label={label}
        value={value}
        onChange={handleChange}
        onBlur={handleBlur}
        error={!!error && touched}
      />
      {error && touched && <FieldError>{error}</FieldError>}
    </div>
  );
}
```

### With Custom Field Wrapper

```jsx
import { FieldWrapper, FieldError } from 'datocms-react-ui';

function CustomField({ label, error, children }) {
  return (
    <FieldWrapper>
      <FormLabel>{label}</FormLabel>
      {children}
      {error && <FieldError>{error}</FieldError>}
    </FieldWrapper>
  );
}
```

## Validation Patterns

### Multiple Errors

```jsx
function MultipleErrorsField({ value, onChange }) {
  const [errors, setErrors] = useState([]);

  const validate = (val) => {
    const newErrors = [];
    
    if (!val) {
      newErrors.push('This field is required');
    }
    if (val && val.length < 3) {
      newErrors.push('Must be at least 3 characters');
    }
    if (val && val.length > 50) {
      newErrors.push('Must be less than 50 characters');
    }
    if (val && !/^[a-zA-Z0-9]+$/.test(val)) {
      newErrors.push('Only letters and numbers allowed');
    }
    
    setErrors(newErrors);
  };

  return (
    <div>
      <TextField
        label="Username"
        value={value}
        onChange={(val) => {
          onChange(val);
          validate(val);
        }}
        error={errors.length > 0}
      />
      {errors.map((error, index) => (
        <FieldError key={index}>{error}</FieldError>
      ))}
    </div>
  );
}
```

### Async Validation

```jsx
function AsyncValidatedField({ ctx }) {
  const [username, setUsername] = useState('');
  const [error, setError] = useState('');
  const [checking, setChecking] = useState(false);

  const checkAvailability = async (value) => {
    if (!value) {
      setError('Username is required');
      return;
    }

    setChecking(true);
    setError('');

    try {
      const client = ctx.createClient();
      const items = await client.items.list({
        filter: {
          fields: {
            username: { eq: value }
          }
        }
      });

      if (items.length > 0) {
        setError('This username is already taken');
      }
    } catch (err) {
      setError('Error checking username availability');
    } finally {
      setChecking(false);
    }
  };

  return (
    <div>
      <TextField
        label="Username"
        value={username}
        onChange={setUsername}
        onBlur={() => checkAvailability(username)}
        suffix={checking ? <Spinner size="xs" /> : null}
        error={!!error}
      />
      {error && <FieldError>{error}</FieldError>}
    </div>
  );
}
```

## Form-Level Validation

### With Form Context

```jsx
function FormWithErrors({ ctx }) {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});

  const validateForm = () => {
    const newErrors = {};

    if (!formData.name) {
      newErrors.name = 'Name is required';
    }

    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (validateForm()) {
      ctx.notice('Form submitted successfully!');
    }
  };

  const updateField = (field, value) => {
    setFormData({ ...formData, [field]: value });
    // Clear error when user starts typing
    if (errors[field]) {
      setErrors({ ...errors, [field]: undefined });
    }
  };

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit}>
        <div>
          <TextField
            label="Name"
            value={formData.name}
            onChange={(value) => updateField('name', value)}
            error={!!errors.name}
          />
          {errors.name && <FieldError>{errors.name}</FieldError>}
        </div>

        <div>
          <TextField
            label="Email"
            type="email"
            value={formData.email}
            onChange={(value) => updateField('email', value)}
            error={!!errors.email}
          />
          {errors.email && <FieldError>{errors.email}</FieldError>}
        </div>

        <div>
          <TextField
            label="Password"
            type="password"
            value={formData.password}
            onChange={(value) => updateField('password', value)}
            error={!!errors.password}
          />
          {errors.password && <FieldError>{errors.password}</FieldError>}
        </div>

        <Button type="submit" buttonType="primary">
          Submit
        </Button>
      </Form>
    </Canvas>
  );
}
```

## Advanced Patterns

### Error Summary

```jsx
function ErrorSummary({ errors }) {
  const errorMessages = Object.entries(errors)
    .filter(([, error]) => error)
    .map(([field, error]) => ({ field, error }));

  if (errorMessages.length === 0) return null;

  return (
    <div style={{
      padding: 'var(--spacing-m)',
      backgroundColor: 'var(--alert-color-bg)',
      borderRadius: 'var(--border-radius-m)',
      marginBottom: 'var(--spacing-l)'
    }}>
      <strong>Please fix the following errors:</strong>
      <ul style={{ marginTop: 'var(--spacing-s)', paddingLeft: 'var(--spacing-l)' }}>
        {errorMessages.map(({ field, error }) => (
          <li key={field}>
            <FieldError>{error}</FieldError>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Inline vs Below Field Errors

```jsx
function FlexibleErrorDisplay({ error, inline = false }) {
  if (!error) return null;

  if (inline) {
    return (
      <span style={{ 
        color: 'var(--alert-color)',
        fontSize: 'var(--font-size-s)',
        marginLeft: 'var(--spacing-s)'
      }}>
        {error}
      </span>
    );
  }

  return <FieldError>{error}</FieldError>;
}

// Usage
<div>
  <label>
    Email
    <FlexibleErrorDisplay error={errors.email} inline />
  </label>
  <TextField 
    value={email} 
    onChange={setEmail}
    error={!!errors.email}
  />
  <FlexibleErrorDisplay error={errors.email} />
</div>
```

### Animated Error Messages

```jsx
function AnimatedFieldError({ children }) {
  const [show, setShow] = useState(false);

  useEffect(() => {
    if (children) {
      setShow(true);
    } else {
      const timer = setTimeout(() => setShow(false), 300);
      return () => clearTimeout(timer);
    }
  }, [children]);

  if (!show && !children) return null;

  return (
    <div
      style={{
        overflow: 'hidden',
        transition: 'all 0.3s ease',
        maxHeight: children ? '100px' : '0',
        opacity: children ? 1 : 0
      }}
    >
      <FieldError>{children}</FieldError>
    </div>
  );
}
```

## Styling

### Custom Error Styling

```jsx
<FieldError
  style={{
    fontWeight: 'bold',
    fontSize: 'var(--font-size-m)',
    marginTop: 'var(--spacing-m)'
  }}
  className="critical-error"
>
  This is a critical error that needs attention
</FieldError>
```

### Icon with Error

```jsx
import { FaExclamationCircle } from 'react-icons/fa';

function IconFieldError({ children }) {
  return (
    <FieldError>
      <span style={{ display: 'flex', alignItems: 'center', gap: 'var(--spacing-xs)' }}>
        <FaExclamationCircle />
        {children}
      </span>
    </FieldError>
  );
}
```

## Accessibility

- Uses proper ARIA attributes
- Announced to screen readers
- Associated with form fields via aria-describedby
- Uses semantic color that meets contrast requirements

## TypeScript

```typescript
import { FieldError } from 'datocms-react-ui';
import { ReactNode } from 'react';

interface ValidationError {
  field: string;
  message: string;
}

interface FieldWithErrorProps {
  label: string;
  value: string;
  onChange: (value: string) => void;
  error?: string;
  validate?: (value: string) => string | undefined;
}

function FieldWithError({ 
  label, 
  value, 
  onChange, 
  error,
  validate 
}: FieldWithErrorProps) {
  const [localError, setLocalError] = useState<string>('');
  
  const handleBlur = () => {
    if (validate) {
      setLocalError(validate(value) || '');
    }
  };

  const displayError = error || localError;

  return (
    <div>
      <TextField
        label={label}
        value={value}
        onChange={onChange}
        onBlur={handleBlur}
        error={!!displayError}
      />
      {displayError && <FieldError>{displayError}</FieldError>}
    </div>
  );
}
```

## Best Practices

1. **Show errors after interaction**: Don't show errors before user has interacted
2. **Clear error messages**: Use simple, actionable language
3. **Contextual errors**: Place errors near the relevant field
4. **Consistent styling**: Use FieldError for all error messages
5. **Accessibility**: Ensure errors are announced to screen readers
6. **Error prevention**: Validate on blur and provide helpful hints

## Related Components

- [TextField](./TextField.md) - Text input with error prop
- [FieldHint](./FieldHint.md) - Help text for fields
- [FieldWrapper](./FieldWrapper.md) - Field container
- [Form](./Form.md) - Form container