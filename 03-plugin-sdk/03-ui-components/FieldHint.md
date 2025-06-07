# FieldHint Component

A component for displaying helpful hints and guidance text for form fields, styled consistently with DatoCMS's design system.

## Import

```jsx
import { FieldHint } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Hint content |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { TextField, FieldHint, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <div>
        <TextField
          label="API Key"
          placeholder="your-api-key"
        />
        <FieldHint>
          You can find your API key in Settings → API Tokens
        </FieldHint>
      </div>
    </Canvas>
  );
}
```

## Common Patterns

### With Form Fields

```jsx
function FormWithHints() {
  return (
    <Form>
      <div>
        <TextField
          label="Slug"
          placeholder="my-awesome-post"
        />
        <FieldHint>
          URL-friendly version of the title. Use lowercase letters, numbers, and hyphens.
        </FieldHint>
      </div>

      <div>
        <TextField
          label="Meta Description"
          maxLength={160}
        />
        <FieldHint>
          Recommended length: 150-160 characters for optimal SEO
        </FieldHint>
      </div>

      <div>
        <SelectField label="Time Zone">
          <option>UTC</option>
          <option>EST</option>
          <option>PST</option>
        </SelectField>
        <FieldHint>
          All dates and times will be displayed in this timezone
        </FieldHint>
      </div>
    </Form>
  );
}
```

### Character Counter

```jsx
function TextFieldWithCounter({ label, maxLength }) {
  const [value, setValue] = useState('');
  const remaining = maxLength - value.length;

  return (
    <div>
      <TextField
        label={label}
        value={value}
        onChange={setValue}
        maxLength={maxLength}
      />
      <FieldHint>
        {remaining > 0 
          ? `${remaining} characters remaining`
          : `Maximum length reached (${maxLength} characters)`
        }
      </FieldHint>
    </div>
  );
}
```

### Dynamic Hints

```jsx
function PasswordField() {
  const [password, setPassword] = useState('');
  
  const getHint = () => {
    if (!password) {
      return 'Password must be at least 8 characters';
    }
    
    const hints = [];
    if (password.length < 8) {
      hints.push(`${8 - password.length} more characters needed`);
    }
    if (!/[A-Z]/.test(password)) {
      hints.push('Add an uppercase letter');
    }
    if (!/[0-9]/.test(password)) {
      hints.push('Add a number');
    }
    if (!/[!@#$%^&*]/.test(password)) {
      hints.push('Add a special character');
    }
    
    return hints.length > 0 
      ? hints.join(', ')
      : '✓ Password meets all requirements';
  };

  return (
    <div>
      <TextField
        label="Password"
        type="password"
        value={password}
        onChange={setPassword}
      />
      <FieldHint>{getHint()}</FieldHint>
    </div>
  );
}
```

## Advanced Examples

### Contextual Help

```jsx
function FieldWithContextualHelp({ fieldType }) {
  const hints = {
    email: 'We\'ll never share your email with anyone else',
    phone: 'Include country code for international numbers',
    website: 'Must start with http:// or https://',
    username: 'Username can only contain letters, numbers, and underscores',
    date: 'Format: YYYY-MM-DD',
    color: 'Use hex format (#RRGGBB) or color names',
  };

  return (
    <div>
      <TextField
        label={fieldType.charAt(0).toUpperCase() + fieldType.slice(1)}
        type={fieldType === 'website' ? 'url' : fieldType}
      />
      {hints[fieldType] && (
        <FieldHint>{hints[fieldType]}</FieldHint>
      )}
    </div>
  );
}
```

### Format Examples

```jsx
function PhoneField() {
  return (
    <div>
      <TextField
        label="Phone Number"
        type="tel"
        placeholder="+1 (555) 123-4567"
      />
      <FieldHint>
        <strong>Format examples:</strong>
        <ul style={{ marginTop: 'var(--spacing-xs)', paddingLeft: 'var(--spacing-l)' }}>
          <li>+1 (555) 123-4567</li>
          <li>+44 20 7946 0958</li>
          <li>+81 3-1234-5678</li>
        </ul>
      </FieldHint>
    </div>
  );
}
```

### Validation Hints

```jsx
function ValidatedField({ label, pattern, example }) {
  const [value, setValue] = useState('');
  const [isValid, setIsValid] = useState(null);

  const validate = (val) => {
    if (!val) {
      setIsValid(null);
      return;
    }
    setIsValid(pattern.test(val));
  };

  return (
    <div>
      <TextField
        label={label}
        value={value}
        onChange={(val) => {
          setValue(val);
          validate(val);
        }}
        error={isValid === false}
      />
      <FieldHint>
        {isValid === true && '✓ Valid format'}
        {isValid === false && `Invalid format. Example: ${example}`}
        {isValid === null && `Example: ${example}`}
      </FieldHint>
    </div>
  );
}

// Usage
<ValidatedField
  label="Product SKU"
  pattern={/^[A-Z]{3}-\d{4}$/}
  example="ABC-1234"
/>
```

### Multi-Line Hints

```jsx
function ComplexFieldHint() {
  return (
    <div>
      <TextareaField
        label="CSV Data"
        rows={10}
      />
      <FieldHint>
        <div>Upload CSV data with the following format:</div>
        <pre style={{ 
          marginTop: 'var(--spacing-xs)',
          padding: 'var(--spacing-s)',
          backgroundColor: 'var(--light-color)',
          borderRadius: 'var(--border-radius-s)',
          fontSize: 'var(--font-size-s)'
        }}>
{`name,email,role
John Doe,john@example.com,admin
Jane Smith,jane@example.com,user`}
        </pre>
      </FieldHint>
    </div>
  );
}
```

### Interactive Hints

```jsx
function InteractiveHint({ ctx }) {
  const [showDetails, setShowDetails] = useState(false);

  return (
    <div>
      <TextField
        label="Integration Token"
        type="password"
      />
      <FieldHint>
        Need a token? 
        <button
          onClick={() => setShowDetails(!showDetails)}
          style={{
            marginLeft: 'var(--spacing-xs)',
            color: 'var(--primary-color)',
            textDecoration: 'underline',
            background: 'none',
            border: 'none',
            cursor: 'pointer'
          }}
        >
          {showDetails ? 'Hide' : 'Show'} instructions
        </button>
        
        {showDetails && (
          <ol style={{ marginTop: 'var(--spacing-s)', paddingLeft: 'var(--spacing-l)' }}>
            <li>Go to Settings → Integrations</li>
            <li>Click "Generate New Token"</li>
            <li>Copy the token and paste it here</li>
          </ol>
        )}
      </FieldHint>
    </div>
  );
}
```

### Conditional Hints

```jsx
function ConditionalHintField({ fieldType }) {
  const [value, setValue] = useState('');
  
  const renderHint = () => {
    switch (fieldType) {
      case 'development':
        return (
          <FieldHint>
            ⚠️ Development mode: Changes will not affect production
          </FieldHint>
        );
      
      case 'staging':
        return (
          <FieldHint>
            🔍 Staging mode: Test changes before going live
          </FieldHint>
        );
      
      case 'production':
        return (
          <FieldHint>
            🚨 Production mode: Changes will be immediately visible to users
          </FieldHint>
        );
      
      default:
        return null;
    }
  };

  return (
    <div>
      <SelectField
        label="Environment"
        value={fieldType}
        onChange={setValue}
      >
        <option value="development">Development</option>
        <option value="staging">Staging</option>
        <option value="production">Production</option>
      </SelectField>
      {renderHint()}
    </div>
  );
}
```

## Styling

### Custom Styling

```jsx
<FieldHint
  style={{
    fontStyle: 'italic',
    color: 'var(--primary-color)',
    marginTop: 'var(--spacing-s)'
  }}
  className="custom-hint"
>
  This is a specially styled hint
</FieldHint>
```

### With Icons

```jsx
import { FaInfoCircle, FaLightbulb } from 'react-icons/fa';

function IconHint({ icon, children }) {
  return (
    <FieldHint>
      <span style={{ display: 'flex', alignItems: 'flex-start', gap: 'var(--spacing-xs)' }}>
        {icon}
        <span>{children}</span>
      </span>
    </FieldHint>
  );
}

// Usage
<IconHint icon={<FaInfoCircle />}>
  This field is required for SEO optimization
</IconHint>

<IconHint icon={<FaLightbulb />}>
  Pro tip: Use descriptive names for better organization
</IconHint>
```

## Accessibility

- Proper color contrast for readability
- Associated with form fields via aria-describedby
- Screen reader friendly
- Supports keyboard navigation

## TypeScript

```typescript
import { FieldHint } from 'datocms-react-ui';
import { ReactNode } from 'react';

interface FieldWithHintProps {
  label: string;
  hint?: ReactNode;
  type?: string;
  value: string;
  onChange: (value: string) => void;
}

function FieldWithHint({ 
  label, 
  hint, 
  type = 'text',
  value, 
  onChange 
}: FieldWithHintProps) {
  return (
    <div>
      <TextField
        label={label}
        type={type}
        value={value}
        onChange={onChange}
      />
      {hint && <FieldHint>{hint}</FieldHint>}
    </div>
  );
}
```

## Best Practices

1. **Be concise**: Keep hints short and to the point
2. **Be helpful**: Provide actionable information
3. **Show examples**: Include format examples when relevant
4. **Update dynamically**: Change hints based on user input
5. **Use sparingly**: Don't overwhelm users with too many hints
6. **Consistent placement**: Always place hints below the field

## Related Components

- [TextField](./TextField.md) - Text input fields
- [FieldError](./FieldError.md) - Error messages
- [FieldWrapper](./FieldWrapper.md) - Field container
- [FormLabel](./FormLabel.md) - Field labels