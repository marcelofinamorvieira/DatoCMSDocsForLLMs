# SelectField Component

A dropdown select field that combines a label, select input, hint text, and error handling in one component, matching DatoCMS's design system. This module provides four variants: `SelectField`, `AsyncSelectField`, `CreatableSelectField`, and `AsyncCreatableSelectField`.

## Import

```jsx
import { 
  SelectField,
  AsyncSelectField,
  CreatableSelectField,
  AsyncCreatableSelectField 
} from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `label` | `string` | - | Field label |
| `value` | `string \| number` | - | Selected value |
| `onChange` | `(value: string) => void` | - | Change handler |
| `children` | `ReactNode` | - | Option elements |
| `hint` | `string` | - | Help text below field |
| `error` | `string` | - | Error message |
| `required` | `boolean` | `false` | Mark as required |
| `disabled` | `boolean` | `false` | Disable the field |
| `placeholder` | `string` | - | Placeholder option text |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { SelectField, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [value, setValue] = useState('');

  return (
    <Canvas ctx={ctx}>
      <SelectField
        label="Country"
        value={value}
        onChange={setValue}
        placeholder="Select a country"
        required
      >
        <option value="us">United States</option>
        <option value="ca">Canada</option>
        <option value="uk">United Kingdom</option>
        <option value="au">Australia</option>
      </SelectField>
    </Canvas>
  );
}
```

## Common Patterns

### With Placeholder

```jsx
<SelectField
  label="Status"
  value={status}
  onChange={setStatus}
  placeholder="Choose status..."
>
  <option value="draft">Draft</option>
  <option value="published">Published</option>
  <option value="archived">Archived</option>
</SelectField>
```

### With Help Text

```jsx
<SelectField
  label="Timezone"
  value={timezone}
  onChange={setTimezone}
  hint="All dates will be displayed in this timezone"
>
  <option value="UTC">UTC (±00:00)</option>
  <option value="EST">Eastern Time (−05:00)</option>
  <option value="PST">Pacific Time (−08:00)</option>
  <option value="CET">Central European Time (+01:00)</option>
</SelectField>
```

### With Validation

```jsx
function ValidatedSelect() {
  const [value, setValue] = useState('');
  const [error, setError] = useState('');

  const handleChange = (newValue) => {
    setValue(newValue);
    if (!newValue) {
      setError('Please select an option');
    } else {
      setError('');
    }
  };

  return (
    <SelectField
      label="Priority"
      value={value}
      onChange={handleChange}
      error={error}
      required
    >
      <option value="">Select priority</option>
      <option value="low">Low</option>
      <option value="medium">Medium</option>
      <option value="high">High</option>
      <option value="critical">Critical</option>
    </SelectField>
  );
}
```

### Option Groups

```jsx
<SelectField
  label="Assign To"
  value={assignee}
  onChange={setAssignee}
>
  <optgroup label="Developers">
    <option value="dev1">John Smith</option>
    <option value="dev2">Jane Doe</option>
    <option value="dev3">Bob Johnson</option>
  </optgroup>
  <optgroup label="Designers">
    <option value="des1">Alice Brown</option>
    <option value="des2">Charlie Davis</option>
  </optgroup>
  <optgroup label="Managers">
    <option value="mgr1">Eve Wilson</option>
    <option value="mgr2">Frank Miller</option>
  </optgroup>
</SelectField>
```

## Advanced Examples

### Dynamic Options

```jsx
function DynamicSelect({ ctx }) {
  const [categories, setCategories] = useState([]);
  const [selectedCategory, setSelectedCategory] = useState('');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function loadCategories() {
      try {
        const client = ctx.createClient();
        const items = await client.items.list({
          filter: { type: 'category' },
          orderBy: 'name_ASC'
        });
        setCategories(items);
      } finally {
        setLoading(false);
      }
    }
    loadCategories();
  }, [ctx]);

  return (
    <SelectField
      label="Category"
      value={selectedCategory}
      onChange={setSelectedCategory}
      disabled={loading}
      hint={loading ? 'Loading categories...' : undefined}
    >
      <option value="">Select a category</option>
      {categories.map(category => (
        <option key={category.id} value={category.id}>
          {category.name}
        </option>
      ))}
    </SelectField>
  );
}
```

### Dependent Selects

```jsx
function DependentSelects() {
  const [country, setCountry] = useState('');
  const [state, setState] = useState('');

  const states = {
    us: ['California', 'Texas', 'New York', 'Florida'],
    ca: ['Ontario', 'Quebec', 'British Columbia', 'Alberta'],
    uk: ['England', 'Scotland', 'Wales', 'Northern Ireland'],
  };

  const handleCountryChange = (value) => {
    setCountry(value);
    setState(''); // Reset state when country changes
  };

  return (
    <>
      <SelectField
        label="Country"
        value={country}
        onChange={handleCountryChange}
        required
      >
        <option value="">Select country</option>
        <option value="us">United States</option>
        <option value="ca">Canada</option>
        <option value="uk">United Kingdom</option>
      </SelectField>

      <SelectField
        label="State/Province"
        value={state}
        onChange={setState}
        disabled={!country}
        hint={!country ? 'Please select a country first' : undefined}
      >
        <option value="">Select state/province</option>
        {country && states[country]?.map(s => (
          <option key={s} value={s}>{s}</option>
        ))}
      </SelectField>
    </>
  );
}
```

### Multi-Select Alternative

```jsx
function MultiSelectField({ label, options, values, onChange }) {
  const [isOpen, setIsOpen] = useState(false);

  const toggleOption = (value) => {
    if (values.includes(value)) {
      onChange(values.filter(v => v !== value));
    } else {
      onChange([...values, value]);
    }
  };

  return (
    <div>
      <FormLabel>{label}</FormLabel>
      <div style={{ position: 'relative' }}>
        <button
          type="button"
          onClick={() => setIsOpen(!isOpen)}
          style={{
            width: '100%',
            padding: 'var(--spacing-s)',
            textAlign: 'left',
            border: '1px solid var(--border-color)',
            borderRadius: 'var(--border-radius-s)',
            background: 'white',
            cursor: 'pointer'
          }}
        >
          {values.length === 0 
            ? 'Select options...'
            : `${values.length} selected`
          }
        </button>
        
        {isOpen && (
          <div style={{
            position: 'absolute',
            top: '100%',
            left: 0,
            right: 0,
            marginTop: '4px',
            background: 'white',
            border: '1px solid var(--border-color)',
            borderRadius: 'var(--border-radius-s)',
            boxShadow: 'var(--box-shadow-m)',
            maxHeight: '200px',
            overflow: 'auto',
            zIndex: 10
          }}>
            {options.map(option => (
              <label
                key={option.value}
                style={{
                  display: 'block',
                  padding: 'var(--spacing-s)',
                  cursor: 'pointer'
                }}
              >
                <input
                  type="checkbox"
                  checked={values.includes(option.value)}
                  onChange={() => toggleOption(option.value)}
                  style={{ marginRight: 'var(--spacing-xs)' }}
                />
                {option.label}
              </label>
            ))}
          </div>
        )}
      </div>
    </div>
  );
}
```

### Searchable Select

```jsx
function SearchableSelect({ label, options, value, onChange }) {
  const [search, setSearch] = useState('');
  const [isOpen, setIsOpen] = useState(false);

  const filteredOptions = options.filter(opt =>
    opt.label.toLowerCase().includes(search.toLowerCase())
  );

  const selectedOption = options.find(opt => opt.value === value);

  return (
    <div>
      <FormLabel>{label}</FormLabel>
      <div style={{ position: 'relative' }}>
        <TextField
          value={isOpen ? search : (selectedOption?.label || '')}
          onChange={setSearch}
          onFocus={() => setIsOpen(true)}
          placeholder="Type to search..."
        />
        
        {isOpen && (
          <div style={{
            position: 'absolute',
            top: '100%',
            left: 0,
            right: 0,
            marginTop: '4px',
            background: 'white',
            border: '1px solid var(--border-color)',
            borderRadius: 'var(--border-radius-s)',
            boxShadow: 'var(--box-shadow-m)',
            maxHeight: '200px',
            overflow: 'auto',
            zIndex: 10
          }}>
            {filteredOptions.length === 0 ? (
              <div style={{ padding: 'var(--spacing-s)', color: 'var(--muted-color)' }}>
                No options found
              </div>
            ) : (
              filteredOptions.map(option => (
                <div
                  key={option.value}
                  onClick={() => {
                    onChange(option.value);
                    setSearch('');
                    setIsOpen(false);
                  }}
                  style={{
                    padding: 'var(--spacing-s)',
                    cursor: 'pointer',
                    backgroundColor: value === option.value ? 'var(--light-color)' : 'transparent'
                  }}
                >
                  {option.label}
                </div>
              ))
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

### Form Integration

```jsx
function UserForm({ ctx }) {
  const [formData, setFormData] = useState({
    name: '',
    role: '',
    department: '',
    location: ''
  });

  const updateField = (field, value) => {
    setFormData({ ...formData, [field]: value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    ctx.notice('User saved successfully!');
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
        
        <SelectField
          label="Role"
          value={formData.role}
          onChange={(value) => updateField('role', value)}
          required
        >
          <option value="">Select role</option>
          <option value="admin">Administrator</option>
          <option value="editor">Editor</option>
          <option value="viewer">Viewer</option>
        </SelectField>
        
        <SelectField
          label="Department"
          value={formData.department}
          onChange={(value) => updateField('department', value)}
          hint="Used for routing and permissions"
        >
          <option value="">Select department</option>
          <option value="engineering">Engineering</option>
          <option value="marketing">Marketing</option>
          <option value="sales">Sales</option>
          <option value="support">Support</option>
        </SelectField>
        
        <SelectField
          label="Office Location"
          value={formData.location}
          onChange={(value) => updateField('location', value)}
        >
          <option value="">Select location</option>
          <optgroup label="North America">
            <option value="nyc">New York</option>
            <option value="sf">San Francisco</option>
            <option value="tor">Toronto</option>
          </optgroup>
          <optgroup label="Europe">
            <option value="lon">London</option>
            <option value="ber">Berlin</option>
            <option value="par">Paris</option>
          </optgroup>
        </SelectField>
        
        <Button type="submit" buttonType="primary">
          Save User
        </Button>
      </Form>
    </Canvas>
  );
}
```

## Styling

### Custom Styling

```jsx
<SelectField
  label="Custom Styled Select"
  value={value}
  onChange={setValue}
  style={{
    marginBottom: 'var(--spacing-l)'
  }}
  className="custom-select-field"
>
  <option>Option 1</option>
  <option>Option 2</option>
</SelectField>
```

## Accessibility

- Proper label association
- Required field indication
- Error announcements
- Keyboard navigation
- Screen reader support

## TypeScript

```typescript
import { SelectField } from 'datocms-react-ui';

interface Option {
  value: string;
  label: string;
  group?: string;
}

interface CustomSelectProps {
  label: string;
  options: Option[];
  value: string;
  onChange: (value: string) => void;
  required?: boolean;
  error?: string;
}

function CustomSelect({ 
  label, 
  options, 
  value, 
  onChange,
  required,
  error
}: CustomSelectProps) {
  const groupedOptions = options.reduce((acc, option) => {
    const group = option.group || 'default';
    if (!acc[group]) acc[group] = [];
    acc[group].push(option);
    return acc;
  }, {} as Record<string, Option[]>);

  return (
    <SelectField
      label={label}
      value={value}
      onChange={onChange}
      required={required}
      error={error}
    >
      <option value="">Select an option</option>
      {Object.entries(groupedOptions).map(([group, opts]) => {
        if (group === 'default') {
          return opts.map(opt => (
            <option key={opt.value} value={opt.value}>
              {opt.label}
            </option>
          ));
        }
        return (
          <optgroup key={group} label={group}>
            {opts.map(opt => (
              <option key={opt.value} value={opt.value}>
                {opt.label}
              </option>
            ))}
          </optgroup>
        );
      })}
    </SelectField>
  );
}
```

## Best Practices

1. **Always include a placeholder**: Help users understand what to select
2. **Group related options**: Use optgroup for better organization
3. **Consider searchability**: For many options, consider a searchable variant
4. **Show loading states**: Indicate when options are being loaded
5. **Clear error messages**: Explain what went wrong
6. **Accessible labels**: Always provide clear labels

## Related Components

- [SelectInput](./SelectInput.md) - Just the select input
- [TextField](./TextField.md) - Text input field
- [Form](./Form.md) - Form container
- [FieldGroup](./FieldGroup.md) - Group fields together

---

## Component Variants

### AsyncSelectField

A select field that loads options asynchronously, useful for fetching data from APIs or large datasets.

#### Import

```jsx
import { AsyncSelectField } from 'datocms-react-ui';
```

#### Additional Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `loadOptions` | `(inputValue: string) => Promise<Option[]>` | - | **Required.** Function to load options |
| `defaultOptions` | `boolean \| Option[]` | `false` | Initial options or true to auto-load |
| `cacheOptions` | `boolean` | `false` | Cache loaded options |
| `isLoading` | `boolean` | - | External loading state |
| `onInputChange` | `(inputValue: string) => void` | - | Input change handler |
| `placeholder` | `string` | `'Select...'` | Placeholder text |

#### Basic Example

```jsx
import { AsyncSelectField } from 'datocms-react-ui';

function UserSelector({ ctx }) {
  const [selectedUser, setSelectedUser] = useState(null);

  const loadUsers = async (inputValue) => {
    const client = ctx.createClient();
    const users = await client.users.list({
      filter: {
        query: inputValue
      }
    });
    
    return users.map(user => ({
      value: user.id,
      label: `${user.firstName} ${user.lastName}`
    }));
  };

  return (
    <AsyncSelectField
      label="Assign to User"
      value={selectedUser}
      onChange={setSelectedUser}
      loadOptions={loadUsers}
      defaultOptions={true}
      cacheOptions
      placeholder="Search for a user..."
    />
  );
}
```

#### Advanced Example with Debouncing

```jsx
function DebouncedAsyncSelect({ ctx }) {
  const [value, setValue] = useState(null);
  
  // Debounce the search
  const loadOptions = useCallback(
    debounce(async (inputValue, callback) => {
      if (inputValue.length < 2) {
        callback([]);
        return;
      }
      
      try {
        const response = await fetch(`/api/search?q=${inputValue}`);
        const data = await response.json();
        const options = data.results.map(item => ({
          value: item.id,
          label: item.name,
          extra: item.metadata
        }));
        callback(options);
      } catch (error) {
        console.error('Failed to load options:', error);
        callback([]);
      }
    }, 300),
    []
  );

  return (
    <AsyncSelectField
      label="Search Items"
      value={value}
      onChange={setValue}
      loadOptions={loadOptions}
      placeholder="Type at least 2 characters..."
      hint="Search across all items in the database"
    />
  );
}
```

### CreatableSelectField

A select field that allows users to create new options on the fly by typing them.

#### Import

```jsx
import { CreatableSelectField } from 'datocms-react-ui';
```

#### Additional Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `isValidNewOption` | `(inputValue: string, value: any, options: Option[]) => boolean` | - | Validate new option |
| `getNewOptionData` | `(inputValue: string, label: string) => Option` | - | Transform new option |
| `onCreateOption` | `(inputValue: string) => void` | - | Handle option creation |
| `createOptionPosition` | `'first' \| 'last'` | `'last'` | Where to add new option |
| `formatCreateLabel` | `(inputValue: string) => string` | - | Format the create label |

#### Basic Example

```jsx
import { CreatableSelectField } from 'datocms-react-ui';

function TagSelector() {
  const [tags, setTags] = useState([
    { value: 'react', label: 'React' },
    { value: 'typescript', label: 'TypeScript' },
    { value: 'graphql', label: 'GraphQL' }
  ]);
  const [selectedTag, setSelectedTag] = useState(null);

  const handleCreate = (inputValue) => {
    const newTag = {
      value: inputValue.toLowerCase().replace(/\s+/g, '-'),
      label: inputValue
    };
    setTags([...tags, newTag]);
    setSelectedTag(newTag);
  };

  return (
    <CreatableSelectField
      label="Tags"
      value={selectedTag}
      onChange={setSelectedTag}
      options={tags}
      onCreateOption={handleCreate}
      formatCreateLabel={(input) => `Create tag "${input}"`}
      placeholder="Select or create a tag..."
    />
  );
}
```

#### Advanced Example with Validation

```jsx
function ValidatedCreatableSelect() {
  const [categories, setCategories] = useState([
    { value: 'general', label: 'General' },
    { value: 'technical', label: 'Technical' }
  ]);
  const [selected, setSelected] = useState(null);
  const [error, setError] = useState('');

  const isValidNewOption = (inputValue, selectValue, selectOptions) => {
    if (!inputValue || inputValue.trim().length < 3) {
      return false;
    }
    
    const exists = selectOptions.some(
      option => option.label.toLowerCase() === inputValue.toLowerCase()
    );
    
    return !exists;
  };

  const getNewOptionData = (inputValue, optionLabel) => ({
    value: inputValue.toLowerCase().replace(/[^a-z0-9]+/g, '-'),
    label: optionLabel,
    isNew: true
  });

  const handleCreate = async (inputValue) => {
    try {
      // Validate with backend
      const response = await fetch('/api/categories', {
        method: 'POST',
        body: JSON.stringify({ name: inputValue })
      });
      
      if (!response.ok) {
        throw new Error('Failed to create category');
      }
      
      const newCategory = await response.json();
      setCategories([...categories, newCategory]);
      setSelected(newCategory);
      setError('');
    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <CreatableSelectField
      label="Category"
      value={selected}
      onChange={setSelected}
      options={categories}
      onCreateOption={handleCreate}
      isValidNewOption={isValidNewOption}
      getNewOptionData={getNewOptionData}
      error={error}
      hint="Type to search or create a new category"
    />
  );
}
```

### AsyncCreatableSelectField

Combines async loading with the ability to create new options, perfect for dynamic datasets where users might need to add new values.

#### Import

```jsx
import { AsyncCreatableSelectField } from 'datocms-react-ui';
```

#### Props

Combines all props from both `AsyncSelectField` and `CreatableSelectField`.

#### Comprehensive Example

```jsx
import { AsyncCreatableSelectField } from 'datocms-react-ui';

function DynamicAuthorField({ ctx }) {
  const [selectedAuthor, setSelectedAuthor] = useState(null);
  const [isCreating, setIsCreating] = useState(false);

  // Load existing authors from API
  const loadAuthors = async (inputValue) => {
    const client = ctx.createClient();
    const items = await client.items.list({
      filter: {
        type: 'author',
        query: inputValue
      }
    });
    
    return items.map(author => ({
      value: author.id,
      label: author.name,
      email: author.email
    }));
  };

  // Validate new author name
  const isValidNewOption = (inputValue) => {
    // Basic validation - at least first and last name
    const parts = inputValue.trim().split(' ');
    return parts.length >= 2 && parts.every(part => part.length >= 2);
  };

  // Create new author
  const handleCreateAuthor = async (inputValue) => {
    setIsCreating(true);
    
    try {
      const [firstName, ...lastNameParts] = inputValue.trim().split(' ');
      const lastName = lastNameParts.join(' ');
      
      const client = ctx.createClient();
      const newAuthor = await client.items.create({
        itemType: 'author',
        name: inputValue,
        firstName,
        lastName,
        slug: inputValue.toLowerCase().replace(/\s+/g, '-')
      });
      
      const option = {
        value: newAuthor.id,
        label: newAuthor.name
      };
      
      setSelectedAuthor(option);
      ctx.notice(`Author "${inputValue}" created successfully!`);
    } catch (error) {
      ctx.alert(`Failed to create author: ${error.message}`);
    } finally {
      setIsCreating(false);
    }
  };

  return (
    <AsyncCreatableSelectField
      label="Author"
      value={selectedAuthor}
      onChange={setSelectedAuthor}
      loadOptions={loadAuthors}
      onCreateOption={handleCreateAuthor}
      isValidNewOption={isValidNewOption}
      formatCreateLabel={(input) => `Create author: ${input}`}
      placeholder="Search or create an author..."
      hint="Enter first and last name to create a new author"
      isLoading={isCreating}
      defaultOptions={true}
      cacheOptions
    />
  );
}
```

#### Real-World Example: Location Selector

```jsx
function LocationSelector({ ctx }) {
  const [location, setLocation] = useState(null);
  const [recentLocations, setRecentLocations] = useState([]);

  useEffect(() => {
    // Load recent locations on mount
    const stored = localStorage.getItem('recentLocations');
    if (stored) {
      setRecentLocations(JSON.parse(stored));
    }
  }, []);

  const searchLocations = async (inputValue) => {
    if (inputValue.length < 3) return recentLocations;
    
    // Use a geocoding API
    const response = await fetch(
      `https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(inputValue)}.json?access_token=${API_KEY}`
    );
    const data = await response.json();
    
    return data.features.map(feature => ({
      value: feature.id,
      label: feature.place_name,
      coordinates: feature.center,
      type: feature.place_type[0]
    }));
  };

  const createLocation = (inputValue) => {
    const newLocation = {
      value: `custom_${Date.now()}`,
      label: inputValue,
      type: 'custom',
      isCustom: true
    };
    
    // Save to recent locations
    const updated = [newLocation, ...recentLocations.slice(0, 4)];
    setRecentLocations(updated);
    localStorage.setItem('recentLocations', JSON.stringify(updated));
    
    setLocation(newLocation);
  };

  const formatOptionLabel = (option) => (
    <div style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
      <span>{option.label}</span>
      {option.type && (
        <span style={{ 
          fontSize: '0.75em', 
          color: 'var(--muted-color)',
          backgroundColor: 'var(--light-color)',
          padding: '2px 6px',
          borderRadius: '3px'
        }}>
          {option.type}
        </span>
      )}
    </div>
  );

  return (
    <AsyncCreatableSelectField
      label="Location"
      value={location}
      onChange={setLocation}
      loadOptions={searchLocations}
      onCreateOption={createLocation}
      defaultOptions={recentLocations}
      placeholder="Search for a location or enter custom..."
      formatOptionLabel={formatOptionLabel}
      hint="Search for cities, addresses, or enter a custom location"
      isClearable
    />
  );
}
```

## Common Props for All Variants

All select field variants share these base props from react-select:

| Prop | Type | Description |
|------|------|-------------|
| `isClearable` | `boolean` | Allow clearing selection |
| `isDisabled` | `boolean` | Disable the select |
| `isLoading` | `boolean` | Show loading indicator |
| `isMulti` | `boolean` | Allow multiple selections |
| `isSearchable` | `boolean` | Allow searching (default: true) |
| `menuIsOpen` | `boolean` | Control menu open state |
| `closeMenuOnSelect` | `boolean` | Close menu after selection |
| `hideSelectedOptions` | `boolean` | Hide selected options in multi-select |
| `formatOptionLabel` | `(option: Option) => ReactNode` | Custom option rendering |
| `noOptionsMessage` | `() => string` | Message when no options found |
| `loadingMessage` | `() => string` | Message while loading |

## Multi-Select Example

All variants support multi-select mode:

```jsx
import { AsyncCreatableSelectField } from 'datocms-react-ui';

function MultiTagSelector() {
  const [selectedTags, setSelectedTags] = useState([]);

  const loadTags = async (input) => {
    // Load from API
    const response = await fetch(`/api/tags?search=${input}`);
    const tags = await response.json();
    return tags;
  };

  const createTag = (input) => {
    const newTag = { value: input, label: input };
    setSelectedTags([...selectedTags, newTag]);
  };

  return (
    <AsyncCreatableSelectField
      label="Tags"
      value={selectedTags}
      onChange={setSelectedTags}
      loadOptions={loadTags}
      onCreateOption={createTag}
      isMulti
      placeholder="Select or create tags..."
      hint="Press enter to create a new tag"
    />
  );
}
```

## Performance Tips

1. **Use `cacheOptions`** for async selects to avoid repeated API calls
2. **Implement debouncing** for search-heavy async selects
3. **Set `defaultOptions`** to true or provide initial options for better UX
4. **Use `isLoading`** prop to show loading states during async operations
5. **Limit the number of options returned** from async load functions