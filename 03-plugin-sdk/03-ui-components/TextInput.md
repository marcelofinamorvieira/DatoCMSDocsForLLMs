# TextInput Component

A standalone single-line text input component that provides just the input element without label or error handling, matching DatoCMS's design system.

## Import

```jsx
import { TextInput } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `value` | `string` | - | Current value |
| `onChange` | `(e: ChangeEvent<HTMLInputElement>) => void` | - | Change handler |
| `type` | `string` | `'text'` | Input type |
| `placeholder` | `string` | - | Placeholder text |
| `disabled` | `boolean` | `false` | Disable the input |
| `readOnly` | `boolean` | `false` | Make read-only |
| `error` | `boolean` | `false` | Show error state |
| `prefix` | `ReactNode` | - | Content before input |
| `suffix` | `ReactNode` | - | Content after input |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |
| ...rest | `InputHTMLAttributes` | - | Standard input attributes |

## Basic Usage

```jsx
import { TextInput, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [value, setValue] = useState('');

  return (
    <Canvas ctx={ctx}>
      <TextInput
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Enter text..."
      />
    </Canvas>
  );
}
```

## Common Patterns

### With Custom Label

```jsx
function CustomLabelInput() {
  const [value, setValue] = useState('');

  return (
    <div>
      <label 
        htmlFor="custom-input"
        style={{ 
          display: 'block',
          marginBottom: 'var(--spacing-xs)',
          fontWeight: 'bold'
        }}
      >
        Custom Label
      </label>
      <TextInput
        id="custom-input"
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
    </div>
  );
}
```

### With Icons

```jsx
import { FaSearch, FaTimes, FaUser } from 'react-icons/fa';

function IconInputs() {
  const [search, setSearch] = useState('');
  const [username, setUsername] = useState('');

  return (
    <div style={{ display: 'grid', gap: 'var(--spacing-m)' }}>
      {/* Search with prefix icon */}
      <TextInput
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search..."
        prefix={<FaSearch />}
      />
      
      {/* Username with prefix icon */}
      <TextInput
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="Enter username"
        prefix={<FaUser />}
      />
      
      {/* Search with clear button */}
      <TextInput
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search..."
        prefix={<FaSearch />}
        suffix={
          search && (
            <button
              onClick={() => setSearch('')}
              style={{
                background: 'none',
                border: 'none',
                cursor: 'pointer',
                padding: 0,
                display: 'flex'
              }}
            >
              <FaTimes />
            </button>
          )
        }
      />
    </div>
  );
}
```

### Different Input Types

```jsx
function InputTypes() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [number, setNumber] = useState('');
  const [date, setDate] = useState('');

  return (
    <div style={{ display: 'grid', gap: 'var(--spacing-m)' }}>
      <TextInput
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="email@example.com"
      />
      
      <TextInput
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Enter password"
      />
      
      <TextInput
        type="number"
        value={number}
        onChange={(e) => setNumber(e.target.value)}
        placeholder="Enter number"
        min="0"
        max="100"
      />
      
      <TextInput
        type="date"
        value={date}
        onChange={(e) => setDate(e.target.value)}
      />
    </div>
  );
}
```

## Advanced Examples

### Currency Input

```jsx
function CurrencyInput({ value, onChange, currency = 'USD' }) {
  const [localValue, setLocalValue] = useState(value || '');

  const handleChange = (e) => {
    const val = e.target.value;
    // Remove non-numeric characters except decimal
    const cleaned = val.replace(/[^0-9.]/g, '');
    
    // Ensure only one decimal point
    const parts = cleaned.split('.');
    const formatted = parts[0] + (parts[1] !== undefined ? '.' + parts[1].slice(0, 2) : '');
    
    setLocalValue(formatted);
    onChange(parseFloat(formatted) || 0);
  };

  return (
    <TextInput
      value={localValue}
      onChange={handleChange}
      prefix={
        <span style={{ fontWeight: 'bold' }}>
          {currency === 'USD' ? '$' : currency === 'EUR' ? '€' : '£'}
        </span>
      }
      placeholder="0.00"
      style={{ textAlign: 'right' }}
    />
  );
}
```

### Masked Input

```jsx
function PhoneInput({ value, onChange }) {
  const formatPhone = (val) => {
    const cleaned = val.replace(/\D/g, '');
    const match = cleaned.match(/^(\d{3})(\d{3})(\d{4})$/);
    
    if (match) {
      return `(${match[1]}) ${match[2]}-${match[3]}`;
    }
    
    return val;
  };

  const handleChange = (e) => {
    const formatted = formatPhone(e.target.value);
    onChange(formatted);
  };

  return (
    <TextInput
      value={value}
      onChange={handleChange}
      placeholder="(555) 123-4567"
      maxLength={14}
    />
  );
}
```

### Debounced Search

```jsx
function DebouncedSearch({ onSearch }) {
  const [value, setValue] = useState('');
  const [searching, setSearching] = useState(false);

  useEffect(() => {
    const timer = setTimeout(() => {
      if (value) {
        setSearching(true);
        onSearch(value).finally(() => setSearching(false));
      }
    }, 300);

    return () => clearTimeout(timer);
  }, [value]);

  return (
    <TextInput
      value={value}
      onChange={(e) => setValue(e.target.value)}
      placeholder="Type to search..."
      prefix={<FaSearch />}
      suffix={searching && <Spinner size="xs" />}
    />
  );
}
```

### Auto-Complete Input

```jsx
function AutoCompleteInput({ suggestions, onSelect }) {
  const [value, setValue] = useState('');
  const [showSuggestions, setShowSuggestions] = useState(false);
  const [filteredSuggestions, setFilteredSuggestions] = useState([]);

  const handleChange = (e) => {
    const val = e.target.value;
    setValue(val);
    
    if (val) {
      const filtered = suggestions.filter(s => 
        s.toLowerCase().includes(val.toLowerCase())
      );
      setFilteredSuggestions(filtered);
      setShowSuggestions(filtered.length > 0);
    } else {
      setShowSuggestions(false);
    }
  };

  const selectSuggestion = (suggestion) => {
    setValue(suggestion);
    onSelect(suggestion);
    setShowSuggestions(false);
  };

  return (
    <div style={{ position: 'relative' }}>
      <TextInput
        value={value}
        onChange={handleChange}
        onFocus={() => setShowSuggestions(filteredSuggestions.length > 0)}
        onBlur={() => setTimeout(() => setShowSuggestions(false), 200)}
        placeholder="Start typing..."
      />
      
      {showSuggestions && (
        <div style={{
          position: 'absolute',
          top: '100%',
          left: 0,
          right: 0,
          marginTop: '4px',
          backgroundColor: 'white',
          border: '1px solid var(--border-color)',
          borderRadius: 'var(--border-radius-s)',
          boxShadow: 'var(--box-shadow-m)',
          maxHeight: '200px',
          overflow: 'auto',
          zIndex: 10
        }}>
          {filteredSuggestions.map((suggestion, index) => (
            <div
              key={index}
              onClick={() => selectSuggestion(suggestion)}
              style={{
                padding: 'var(--spacing-s)',
                cursor: 'pointer',
                ':hover': {
                  backgroundColor: 'var(--light-color)'
                }
              }}
            >
              {suggestion}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Color Picker Input

```jsx
function ColorInput({ value, onChange }) {
  const [showPicker, setShowPicker] = useState(false);

  return (
    <div style={{ position: 'relative' }}>
      <TextInput
        value={value}
        onChange={(e) => onChange(e.target.value)}
        placeholder="#000000"
        prefix={
          <div
            style={{
              width: '20px',
              height: '20px',
              backgroundColor: value || '#ccc',
              border: '1px solid var(--border-color)',
              borderRadius: 'var(--border-radius-s)',
              cursor: 'pointer'
            }}
            onClick={() => setShowPicker(!showPicker)}
          />
        }
      />
      
      {showPicker && (
        <input
          type="color"
          value={value}
          onChange={(e) => {
            onChange(e.target.value);
            setShowPicker(false);
          }}
          style={{
            position: 'absolute',
            top: '100%',
            left: 0,
            marginTop: '4px'
          }}
        />
      )}
    </div>
  );
}
```

### Tag Input

```jsx
function TagInput({ tags, onTagsChange }) {
  const [inputValue, setInputValue] = useState('');

  const addTag = () => {
    if (inputValue && !tags.includes(inputValue)) {
      onTagsChange([...tags, inputValue]);
      setInputValue('');
    }
  };

  const removeTag = (tagToRemove) => {
    onTagsChange(tags.filter(tag => tag !== tagToRemove));
  };

  return (
    <div>
      <div style={{ 
        display: 'flex', 
        flexWrap: 'wrap', 
        gap: 'var(--spacing-xs)',
        marginBottom: 'var(--spacing-s)'
      }}>
        {tags.map(tag => (
          <span
            key={tag}
            style={{
              padding: '4px 8px',
              backgroundColor: 'var(--primary-color)',
              color: 'white',
              borderRadius: 'var(--border-radius-s)',
              fontSize: 'var(--font-size-s)',
              display: 'flex',
              alignItems: 'center',
              gap: 'var(--spacing-xs)'
            }}
          >
            {tag}
            <button
              onClick={() => removeTag(tag)}
              style={{
                background: 'none',
                border: 'none',
                color: 'white',
                cursor: 'pointer',
                padding: 0
              }}
            >
              ×
            </button>
          </span>
        ))}
      </div>
      
      <TextInput
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            e.preventDefault();
            addTag();
          }
        }}
        placeholder="Add tag and press Enter"
      />
    </div>
  );
}
```

## Form Integration

```jsx
function InlineForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: ''
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
  };

  const updateField = (field, value) => {
    setFormData({ ...formData, [field]: value });
  };

  return (
    <form onSubmit={handleSubmit}>
      <div style={{ 
        display: 'grid', 
        gridTemplateColumns: '1fr 1fr 2fr auto',
        gap: 'var(--spacing-s)',
        alignItems: 'end'
      }}>
        <div>
          <FormLabel htmlFor="first">First Name</FormLabel>
          <TextInput
            id="first"
            value={formData.firstName}
            onChange={(e) => updateField('firstName', e.target.value)}
            required
          />
        </div>
        
        <div>
          <FormLabel htmlFor="last">Last Name</FormLabel>
          <TextInput
            id="last"
            value={formData.lastName}
            onChange={(e) => updateField('lastName', e.target.value)}
            required
          />
        </div>
        
        <div>
          <FormLabel htmlFor="email">Email</FormLabel>
          <TextInput
            id="email"
            type="email"
            value={formData.email}
            onChange={(e) => updateField('email', e.target.value)}
            required
          />
        </div>
        
        <Button type="submit" buttonType="primary">
          Submit
        </Button>
      </div>
    </form>
  );
}
```

## Accessibility

- Native input element with full keyboard support
- Works with screen readers
- Supports standard HTML attributes
- Error state communicated visually
- Can be labeled with external labels

## TypeScript

```typescript
import { TextInput } from 'datocms-react-ui';
import { ChangeEvent } from 'react';

interface CustomInputProps {
  value: string;
  onChange: (value: string) => void;
  type?: string;
  error?: boolean;
  icon?: React.ReactNode;
}

function CustomInput({ 
  value, 
  onChange,
  type = 'text',
  error,
  icon
}: CustomInputProps) {
  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.value);
  };

  return (
    <TextInput
      type={type}
      value={value}
      onChange={handleChange}
      error={error}
      prefix={icon}
    />
  );
}
```

## Best Practices

1. **Controlled component**: Always use controlled mode with value/onChange
2. **Input types**: Use appropriate type for better UX
3. **Error handling**: Use the error prop for validation states
4. **Accessibility**: Ensure proper labeling with htmlFor
5. **Placeholder text**: Use meaningful placeholders
6. **Icon usage**: Use prefix/suffix for visual context

## Related Components

- [TextField](./TextField.md) - Complete text field with label
- [TextareaInput](./TextareaInput.md) - Multi-line text input
- [SelectInput](./SelectInput.md) - Dropdown input
- [FieldWrapper](./FieldWrapper.md) - For custom field layouts