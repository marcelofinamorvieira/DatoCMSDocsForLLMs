# Form Component

A form container that provides consistent styling and behavior for form elements in DatoCMS plugins.

## Import

```jsx
import { Form } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Form content |
| `onSubmit` | `(e: FormEvent) => void` | - | Submit handler |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |
| `noValidate` | `boolean` | `false` | Disable browser validation |
| `autoComplete` | `string` | - | Form autocomplete setting |

## Basic Usage

```jsx
import { Form, TextField, Button, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const handleSubmit = (e) => {
    e.preventDefault();
    // Handle form submission
  };

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit}>
        <TextField
          label="Name"
          required
        />
        
        <Button type="submit" buttonType="primary">
          Submit
        </Button>
      </Form>
    </Canvas>
  );
}
```

## Form Layouts

### Vertical Layout (Default)

```jsx
<Form onSubmit={handleSubmit}>
  <TextField label="First Name" />
  <TextField label="Last Name" />
  <TextField label="Email" type="email" />
  <TextareaField label="Message" />
  
  <Button type="submit" buttonType="primary">
    Submit
  </Button>
</Form>
```

### With Field Groups

```jsx
import { Form, FieldGroup, TextField } from 'datocms-react-ui';

<Form onSubmit={handleSubmit}>
  <FieldGroup>
    <TextField label="First Name" />
    <TextField label="Last Name" />
  </FieldGroup>
  
  <TextField label="Email" type="email" />
  
  <FieldGroup>
    <TextField label="City" />
    <TextField label="Postal Code" />
  </FieldGroup>
  
  <Button type="submit" buttonType="primary">
    Submit
  </Button>
</Form>
```

### With Sections

```jsx
import { Form, Section, TextField } from 'datocms-react-ui';

<Form onSubmit={handleSubmit}>
  <Section title="Personal Information">
    <TextField label="Name" required />
    <TextField label="Email" type="email" required />
  </Section>
  
  <Section title="Preferences">
    <SelectField label="Language">
      <option>English</option>
      <option>Spanish</option>
      <option>French</option>
    </SelectField>
    
    <SwitchField label="Subscribe to newsletter" />
  </Section>
  
  <Button type="submit" buttonType="primary">
    Save Settings
  </Button>
</Form>
```

## Form Validation

### Basic Validation

```jsx
function ValidatedForm({ ctx }) {
  const [values, setValues] = useState({
    email: '',
    password: '',
  });
  const [errors, setErrors] = useState({});

  const validate = () => {
    const newErrors = {};
    
    if (!values.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!values.password) {
      newErrors.password = 'Password is required';
    } else if (values.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();
    
    if (Object.keys(newErrors).length === 0) {
      // Submit form
      ctx.notice('Form submitted successfully!');
    } else {
      setErrors(newErrors);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <Form onSubmit={handleSubmit} noValidate>
        <TextField
          label="Email"
          type="email"
          value={values.email}
          onChange={(email) => setValues({ ...values, email })}
          error={errors.email}
          required
        />
        
        <TextField
          label="Password"
          type="password"
          value={values.password}
          onChange={(password) => setValues({ ...values, password })}
          error={errors.password}
          required
        />
        
        <Button type="submit" buttonType="primary">
          Submit
        </Button>
      </Form>
    </Canvas>
  );
}
```

### Async Validation

```jsx
function AsyncValidationForm({ ctx }) {
  const [username, setUsername] = useState('');
  const [checking, setChecking] = useState(false);
  const [error, setError] = useState('');

  const checkUsername = async (value) => {
    if (!value) return;
    
    setChecking(true);
    setError('');
    
    try {
      const response = await fetch(`/api/check-username?username=${value}`);
      const data = await response.json();
      
      if (!data.available) {
        setError('Username is already taken');
      }
    } catch (err) {
      setError('Error checking username');
    } finally {
      setChecking(false);
    }
  };

  return (
    <Form onSubmit={(e) => e.preventDefault()}>
      <TextField
        label="Username"
        value={username}
        onChange={setUsername}
        onBlur={() => checkUsername(username)}
        error={error}
        suffix={checking ? <Spinner size="xs" /> : null}
      />
      
      <Button 
        type="submit" 
        buttonType="primary"
        disabled={!!error || checking}
      >
        Create Account
      </Button>
    </Form>
  );
}
```

## Dynamic Forms

### Conditional Fields

```jsx
function ConditionalForm({ ctx }) {
  const [accountType, setAccountType] = useState('personal');

  return (
    <Form onSubmit={handleSubmit}>
      <SelectField 
        label="Account Type"
        value={accountType}
        onChange={setAccountType}
      >
        <option value="personal">Personal</option>
        <option value="business">Business</option>
      </SelectField>
      
      <TextField label="Name" required />
      
      {accountType === 'business' && (
        <>
          <TextField label="Company Name" required />
          <TextField label="VAT Number" />
        </>
      )}
      
      <TextField label="Email" type="email" required />
      
      <Button type="submit" buttonType="primary">
        Create Account
      </Button>
    </Form>
  );
}
```

### Dynamic Field Lists

```jsx
function DynamicFieldList({ ctx }) {
  const [fields, setFields] = useState([{ id: 1, value: '' }]);

  const addField = () => {
    setFields([...fields, { id: Date.now(), value: '' }]);
  };

  const removeField = (id) => {
    setFields(fields.filter(f => f.id !== id));
  };

  const updateField = (id, value) => {
    setFields(fields.map(f => f.id === id ? { ...f, value } : f));
  };

  return (
    <Form onSubmit={handleSubmit}>
      <FormLabel>Tags</FormLabel>
      
      {fields.map((field, index) => (
        <div key={field.id} style={{ display: 'flex', gap: 'var(--spacing-s)' }}>
          <TextField
            value={field.value}
            onChange={(value) => updateField(field.id, value)}
            placeholder={`Tag ${index + 1}`}
          />
          
          {fields.length > 1 && (
            <Button
              onClick={() => removeField(field.id)}
              buttonType="negative"
              size="s"
            >
              Remove
            </Button>
          )}
        </div>
      ))}
      
      <Button onClick={addField} buttonType="muted" size="s">
        Add Tag
      </Button>
      
      <Button type="submit" buttonType="primary">
        Save Tags
      </Button>
    </Form>
  );
}
```

## Form State Management

### Using React Hook Form

```jsx
import { useForm } from 'react-hook-form';

function HookForm({ ctx }) {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => {
    console.log(data);
    ctx.notice('Form submitted!');
  };

  return (
    <Form onSubmit={handleSubmit(onSubmit)}>
      <TextField
        label="Name"
        {...register('name', { required: 'Name is required' })}
        error={errors.name?.message}
      />
      
      <TextField
        label="Email"
        type="email"
        {...register('email', { 
          required: 'Email is required',
          pattern: {
            value: /\S+@\S+\.\S+/,
            message: 'Invalid email address'
          }
        })}
        error={errors.email?.message}
      />
      
      <Button type="submit" buttonType="primary">
        Submit
      </Button>
    </Form>
  );
}
```

### Connected to Plugin Context

```jsx
function ContextConnectedForm({ ctx }) {
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      // Save to plugin parameters
      await ctx.updatePluginParameters({
        apiKey: e.target.apiKey.value,
        endpoint: e.target.endpoint.value,
      });
      
      ctx.notice('Settings saved successfully!');
    } catch (error) {
      ctx.alert('Failed to save settings');
    }
  };

  return (
    <Form onSubmit={handleSubmit}>
      <TextField
        name="apiKey"
        label="API Key"
        defaultValue={ctx.plugin.attributes.parameters.apiKey}
        required
      />
      
      <TextField
        name="endpoint"
        label="API Endpoint"
        type="url"
        defaultValue={ctx.plugin.attributes.parameters.endpoint}
        required
      />
      
      <Button type="submit" buttonType="primary">
        Save Settings
      </Button>
    </Form>
  );
}
```

## Advanced Patterns

### Multi-Step Form

```jsx
function MultiStepForm({ ctx }) {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({});
  
  const nextStep = (data) => {
    setFormData({ ...formData, ...data });
    setStep(step + 1);
  };
  
  const prevStep = () => setStep(step - 1);
  
  const submitForm = (data) => {
    const finalData = { ...formData, ...data };
    console.log('Submitting:', finalData);
    ctx.notice('Form submitted successfully!');
  };

  return (
    <Canvas ctx={ctx}>
      {step === 1 && (
        <Form onSubmit={(e) => {
          e.preventDefault();
          nextStep({ name: e.target.name.value });
        }}>
          <h2>Step 1: Personal Info</h2>
          <TextField name="name" label="Name" required />
          <Button type="submit" buttonType="primary">Next</Button>
        </Form>
      )}
      
      {step === 2 && (
        <Form onSubmit={(e) => {
          e.preventDefault();
          nextStep({ email: e.target.email.value });
        }}>
          <h2>Step 2: Contact</h2>
          <TextField name="email" label="Email" type="email" required />
          <div style={{ display: 'flex', gap: 'var(--spacing-s)' }}>
            <Button onClick={prevStep}>Back</Button>
            <Button type="submit" buttonType="primary">Next</Button>
          </div>
        </Form>
      )}
      
      {step === 3 && (
        <Form onSubmit={(e) => {
          e.preventDefault();
          submitForm({ message: e.target.message.value });
        }}>
          <h2>Step 3: Message</h2>
          <TextareaField name="message" label="Message" />
          <div style={{ display: 'flex', gap: 'var(--spacing-s)' }}>
            <Button onClick={prevStep}>Back</Button>
            <Button type="submit" buttonType="primary">Submit</Button>
          </div>
        </Form>
      )}
    </Canvas>
  );
}
```

## Styling

### Custom Form Styling

```jsx
<Form 
  onSubmit={handleSubmit}
  style={{
    maxWidth: '500px',
    margin: '0 auto',
    padding: 'var(--spacing-l)',
    backgroundColor: 'var(--light-color)',
    borderRadius: 'var(--border-radius-m)'
  }}
  className="custom-form"
>
  {/* Form content */}
</Form>
```

## Accessibility

- Proper form structure with labels
- Keyboard navigation support
- Error announcements for screen readers
- Required field indicators
- ARIA attributes for validation states

## TypeScript

```typescript
import { Form } from 'datocms-react-ui';
import { FormEvent } from 'react';

interface FormData {
  name: string;
  email: string;
}

function TypedForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    
    onSubmit({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
    });
  };

  return (
    <Form onSubmit={handleSubmit}>
      <TextField name="name" label="Name" required />
      <TextField name="email" label="Email" type="email" required />
      <Button type="submit" buttonType="primary">Submit</Button>
    </Form>
  );
}
```

## Best Practices

1. **Always prevent default**: Use `e.preventDefault()` in submit handlers
2. **Show validation errors**: Display errors near the relevant fields
3. **Disable submit during processing**: Prevent double submissions
4. **Provide feedback**: Show success/error messages after submission
5. **Use semantic HTML**: Proper form elements and labels
6. **Handle loading states**: Show spinners during async operations

## Related Components

- [TextField](./TextField.md) - Text input fields
- [TextareaField](./TextareaField.md) - Multi-line text
- [SelectField](./SelectField.md) - Dropdown selections
- [SwitchField](./SwitchField.md) - Toggle switches
- [Button](./Button.md) - Submit buttons
- [FieldGroup](./FieldGroup.md) - Group related fields
- [Section](./Section.md) - Form sections