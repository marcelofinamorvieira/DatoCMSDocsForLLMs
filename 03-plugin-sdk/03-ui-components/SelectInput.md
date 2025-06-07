# SelectInput Component

A low-level select input component that provides more control than `SelectField`. This module includes four variants built on [react-select](https://react-select.com/): `SelectInput`, `AsyncSelectInput`, `CreatableSelectInput`, and `AsyncCreatableSelectInput`.

## Purpose

While `SelectField` provides a complete form field with label, hint, and error handling, `SelectInput` gives you just the select control itself. Use these components when you need:
- Custom form layouts
- Integration with custom form libraries
- Direct control over the select behavior
- Advanced features like async loading or option creation

## Import

```jsx
import { 
  SelectInput,
  AsyncSelectInput,
  CreatableSelectInput,
  AsyncCreatableSelectInput 
} from 'datocms-react-ui';
```

## Components

### SelectInput

The base select input component for static options.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `options` | `Option[]` | - | **Required.** Array of options |
| `value` | `Option \| Option[] \| null` | - | Selected value(s) |
| `onChange` | `(value: Option \| Option[] \| null) => void` | - | **Required.** Change handler |
| `isMulti` | `boolean` | `false` | Enable multi-select |
| `isClearable` | `boolean` | `false` | Allow clearing selection |
| `isDisabled` | `boolean` | `false` | Disable the input |
| `isLoading` | `boolean` | `false` | Show loading state |
| `isSearchable` | `boolean` | `true` | Enable search/filter |
| `placeholder` | `string` | `'Select...'` | Placeholder text |
| `error` | `boolean` | `false` | Show error state |
| `menuIsOpen` | `boolean` | - | Control menu state |
| `closeMenuOnSelect` | `boolean` | `true` | Close menu on selection |
| `hideSelectedOptions` | `boolean` | `true` | Hide selected options |
| `formatOptionLabel` | `(option: Option) => ReactNode` | - | Custom option renderer |
| `noOptionsMessage` | `() => string` | - | No options message |

#### Basic Usage

```jsx
import { SelectInput } from 'datocms-react-ui';
import { useState } from 'react';

function BasicExample() {
  const [selected, setSelected] = useState(null);

  const options = [
    { value: 'chocolate', label: 'Chocolate' },
    { value: 'strawberry', label: 'Strawberry' },
    { value: 'vanilla', label: 'Vanilla' }
  ];

  return (
    <SelectInput
      options={options}
      value={selected}
      onChange={setSelected}
      placeholder="Choose a flavor..."
    />
  );
}
```

#### Multi-Select Example

```jsx
function MultiSelectExample() {
  const [selected, setSelected] = useState([]);

  const options = [
    { value: 'react', label: 'React' },
    { value: 'vue', label: 'Vue' },
    { value: 'angular', label: 'Angular' },
    { value: 'svelte', label: 'Svelte' }
  ];

  return (
    <SelectInput
      options={options}
      value={selected}
      onChange={setSelected}
      isMulti
      isClearable
      placeholder="Select frameworks..."
    />
  );
}
```

#### Grouped Options

```jsx
function GroupedSelectExample() {
  const [selected, setSelected] = useState(null);

  const groupedOptions = [
    {
      label: 'Frontend',
      options: [
        { value: 'react', label: 'React' },
        { value: 'vue', label: 'Vue' }
      ]
    },
    {
      label: 'Backend', 
      options: [
        { value: 'node', label: 'Node.js' },
        { value: 'python', label: 'Python' }
      ]
    }
  ];

  return (
    <SelectInput
      options={groupedOptions}
      value={selected}
      onChange={setSelected}
    />
  );
}
```

#### Custom Option Rendering

```jsx
function CustomOptionExample() {
  const [selected, setSelected] = useState(null);

  const options = [
    { value: 'active', label: 'Active', color: '#10b981' },
    { value: 'pending', label: 'Pending', color: '#f59e0b' },
    { value: 'inactive', label: 'Inactive', color: '#ef4444' }
  ];

  const formatOptionLabel = (option) => (
    <div style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
      <span
        style={{
          width: '12px',
          height: '12px',
          borderRadius: '50%',
          backgroundColor: option.color
        }}
      />
      <span>{option.label}</span>
    </div>
  );

  return (
    <SelectInput
      options={options}
      value={selected}
      onChange={setSelected}
      formatOptionLabel={formatOptionLabel}
    />
  );
}
```

### AsyncSelectInput

A select input that loads options asynchronously, perfect for API data or large datasets.

#### Additional Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `loadOptions` | `(inputValue: string) => Promise<Option[]>` | - | **Required.** Async option loader |
| `defaultOptions` | `boolean \| Option[]` | `false` | Initial options or auto-load |
| `cacheOptions` | `boolean` | `false` | Cache loaded options |
| `onInputChange` | `(inputValue: string) => void` | - | Input change handler |

#### Basic Example

```jsx
import { AsyncSelectInput } from 'datocms-react-ui';

function UserSearchExample({ ctx }) {
  const [selectedUser, setSelectedUser] = useState(null);

  const loadUsers = async (inputValue) => {
    // Return empty array for short queries
    if (inputValue.length < 2) {
      return [];
    }

    // Fetch from API
    const client = ctx.createClient();
    const users = await client.users.list({
      filter: {
        query: inputValue
      },
      page: {
        limit: 10
      }
    });

    return users.map(user => ({
      value: user.id,
      label: `${user.firstName} ${user.lastName}`,
      email: user.email
    }));
  };

  return (
    <AsyncSelectInput
      value={selectedUser}
      onChange={setSelectedUser}
      loadOptions={loadUsers}
      placeholder="Search for users..."
      cacheOptions
    />
  );
}
```

#### With Default Options

```jsx
function AsyncWithDefaultsExample() {
  const [value, setValue] = useState(null);

  const defaultOptions = [
    { value: 'recent1', label: 'Recently Used Item 1' },
    { value: 'recent2', label: 'Recently Used Item 2' }
  ];

  const loadOptions = async (inputValue) => {
    const response = await fetch(`/api/search?q=${inputValue}`);
    const data = await response.json();
    return data.results;
  };

  return (
    <AsyncSelectInput
      value={value}
      onChange={setValue}
      loadOptions={loadOptions}
      defaultOptions={defaultOptions}
      placeholder="Search or select recent..."
    />
  );
}
```

#### With Debouncing

```jsx
import { useMemo } from 'react';
import { debounce } from 'lodash';

function DebouncedAsyncExample() {
  const [value, setValue] = useState(null);

  const loadOptions = useMemo(
    () => debounce(async (inputValue, callback) => {
      try {
        const response = await fetch(`/api/search?q=${inputValue}`);
        const data = await response.json();
        callback(data.results);
      } catch (error) {
        console.error('Search failed:', error);
        callback([]);
      }
    }, 300),
    []
  );

  return (
    <AsyncSelectInput
      value={value}
      onChange={setValue}
      loadOptions={loadOptions}
      placeholder="Type to search..."
    />
  );
}
```

### CreatableSelectInput

A select input that allows users to create new options by typing.

#### Additional Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `onCreateOption` | `(inputValue: string) => void` | - | **Required.** Create option handler |
| `isValidNewOption` | `(inputValue: string, value: any, options: Option[]) => boolean` | - | Validate new option |
| `getNewOptionData` | `(inputValue: string, label: string) => Option` | - | Transform new option |
| `createOptionPosition` | `'first' \| 'last'` | `'last'` | New option position |
| `formatCreateLabel` | `(inputValue: string) => string` | - | Format create label |

#### Basic Example

```jsx
import { CreatableSelectInput } from 'datocms-react-ui';

function TagInputExample() {
  const [tags, setTags] = useState([
    { value: 'javascript', label: 'JavaScript' },
    { value: 'typescript', label: 'TypeScript' }
  ]);
  const [selected, setSelected] = useState([]);

  const handleCreate = (inputValue) => {
    const newOption = {
      value: inputValue.toLowerCase().replace(/\W/g, ''),
      label: inputValue
    };
    setTags([...tags, newOption]);
    setSelected([...selected, newOption]);
  };

  return (
    <CreatableSelectInput
      options={tags}
      value={selected}
      onChange={setSelected}
      onCreateOption={handleCreate}
      isMulti
      isClearable
      placeholder="Select or create tags..."
      formatCreateLabel={(input) => `Create "${input}"`}
    />
  );
}
```

#### With Validation

```jsx
function ValidatedCreatableExample() {
  const [options, setOptions] = useState([
    { value: 'existing', label: 'Existing Option' }
  ]);
  const [value, setValue] = useState(null);

  const isValidNewOption = (inputValue, selectValue, selectOptions) => {
    // Must be at least 3 characters
    if (!inputValue || inputValue.trim().length < 3) {
      return false;
    }

    // Must not already exist
    const exists = selectOptions.some(
      option => option.label.toLowerCase() === inputValue.toLowerCase()
    );

    return !exists;
  };

  const getNewOptionData = (inputValue, optionLabel) => ({
    value: `custom_${Date.now()}`,
    label: optionLabel,
    isCustom: true
  });

  const handleCreate = (inputValue) => {
    const newOption = getNewOptionData(inputValue, inputValue);
    setOptions([...options, newOption]);
    setValue(newOption);
  };

  return (
    <CreatableSelectInput
      options={options}
      value={value}
      onChange={setValue}
      onCreateOption={handleCreate}
      isValidNewOption={isValidNewOption}
      getNewOptionData={getNewOptionData}
      placeholder="Type at least 3 characters..."
    />
  );
}
```

### AsyncCreatableSelectInput

Combines async loading with option creation - the most flexible variant.

#### Props

This component combines all props from both `AsyncSelectInput` and `CreatableSelectInput`.

#### Comprehensive Example

```jsx
import { AsyncCreatableSelectInput } from 'datocms-react-ui';

function DynamicEmailExample({ ctx }) {
  const [selectedEmails, setSelectedEmails] = useState([]);
  const [customEmails, setCustomEmails] = useState([]);

  // Load existing contacts
  const loadContacts = async (inputValue) => {
    if (inputValue.length < 2) return customEmails;

    const client = ctx.createClient();
    const users = await client.users.list({
      filter: {
        query: inputValue
      }
    });

    const existingOptions = users.map(user => ({
      value: user.email,
      label: `${user.firstName} ${user.lastName}`,
      email: user.email,
      isUser: true
    }));

    // Include custom emails in results
    return [...existingOptions, ...customEmails];
  };

  // Validate email format
  const isValidNewOption = (inputValue) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(inputValue);
  };

  // Create custom email option
  const handleCreateEmail = (inputValue) => {
    const newEmail = {
      value: inputValue,
      label: inputValue,
      email: inputValue,
      isCustom: true
    };
    
    setCustomEmails([...customEmails, newEmail]);
    setSelectedEmails([...selectedEmails, newEmail]);
  };

  // Custom option display
  const formatOptionLabel = (option) => (
    <div>
      <div>{option.label}</div>
      {option.email && option.label !== option.email && (
        <div style={{ fontSize: '0.875em', color: 'var(--muted-color)' }}>
          {option.email}
        </div>
      )}
    </div>
  );

  return (
    <AsyncCreatableSelectInput
      value={selectedEmails}
      onChange={setSelectedEmails}
      loadOptions={loadContacts}
      onCreateOption={handleCreateEmail}
      isValidNewOption={isValidNewOption}
      formatCreateLabel={(input) => `Add email: ${input}`}
      formatOptionLabel={formatOptionLabel}
      placeholder="Search contacts or enter email..."
      isMulti
      isClearable
      cacheOptions
    />
  );
}
```

#### Real-World Example: Product Selector

```jsx
function ProductSelector({ ctx }) {
  const [selectedProducts, setSelectedProducts] = useState([]);
  const [isCreating, setIsCreating] = useState(false);

  // Search existing products
  const searchProducts = async (inputValue) => {
    const client = ctx.createClient();
    
    const products = await client.items.list({
      filter: {
        type: 'product',
        query: inputValue
      },
      page: { limit: 20 }
    });

    return products.map(product => ({
      value: product.id,
      label: product.name,
      price: product.price,
      sku: product.sku,
      thumbnail: product.thumbnail?.url
    }));
  };

  // Create new product
  const createProduct = async (inputValue) => {
    setIsCreating(true);
    
    try {
      const client = ctx.createClient();
      
      // Create a draft product
      const newProduct = await client.items.create({
        itemType: 'product',
        name: inputValue,
        status: 'draft',
        sku: `DRAFT-${Date.now()}`
      });

      const option = {
        value: newProduct.id,
        label: newProduct.name,
        isDraft: true
      };

      setSelectedProducts([...selectedProducts, option]);
      
      ctx.notice(`Draft product "${inputValue}" created`);
    } catch (error) {
      ctx.alert(`Failed to create product: ${error.message}`);
    } finally {
      setIsCreating(false);
    }
  };

  // Custom option rendering with thumbnails
  const formatOptionLabel = (option) => (
    <div style={{ display: 'flex', alignItems: 'center', gap: '12px' }}>
      {option.thumbnail ? (
        <img 
          src={option.thumbnail} 
          alt=""
          style={{ 
            width: '40px', 
            height: '40px', 
            objectFit: 'cover',
            borderRadius: '4px'
          }}
        />
      ) : (
        <div style={{
          width: '40px',
          height: '40px',
          backgroundColor: 'var(--light-color)',
          borderRadius: '4px',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center'
        }}>
          <span style={{ fontSize: '0.75em', color: 'var(--muted-color)' }}>
            No image
          </span>
        </div>
      )}
      <div>
        <div>{option.label}</div>
        {option.sku && (
          <div style={{ fontSize: '0.75em', color: 'var(--muted-color)' }}>
            SKU: {option.sku} {option.price && `• $${option.price}`}
          </div>
        )}
      </div>
    </div>
  );

  return (
    <AsyncCreatableSelectInput
      value={selectedProducts}
      onChange={setSelectedProducts}
      loadOptions={searchProducts}
      onCreateOption={createProduct}
      formatOptionLabel={formatOptionLabel}
      formatCreateLabel={(input) => `Create draft product: "${input}"`}
      placeholder="Search or create products..."
      isMulti
      isClearable
      isLoading={isCreating}
      defaultOptions
      cacheOptions
    />
  );
}
```

## Advanced Patterns

### Custom Styling with Error States

```jsx
function StyledSelectExample() {
  const [value, setValue] = useState(null);
  const [error, setError] = useState(false);

  const options = [
    { value: 'opt1', label: 'Option 1' },
    { value: 'opt2', label: 'Option 2' }
  ];

  const handleChange = (newValue) => {
    setValue(newValue);
    setError(!newValue); // Show error if no selection
  };

  return (
    <div>
      <SelectInput
        options={options}
        value={value}
        onChange={handleChange}
        error={error}
        placeholder="This field is required"
      />
      {error && (
        <div style={{ 
          color: 'var(--alert-color)', 
          fontSize: '0.875em',
          marginTop: '4px'
        }}>
          Please select an option
        </div>
      )}
    </div>
  );
}
```

### Integration with Form Libraries

```jsx
import { Controller } from 'react-hook-form';
import { SelectInput } from 'datocms-react-ui';

function FormIntegrationExample() {
  const { control, handleSubmit } = useForm();

  const options = [
    { value: 'option1', label: 'Option 1' },
    { value: 'option2', label: 'Option 2' }
  ];

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="selectField"
        control={control}
        rules={{ required: 'This field is required' }}
        render={({ field, fieldState: { error } }) => (
          <SelectInput
            {...field}
            options={options}
            error={!!error}
            placeholder="Select an option"
          />
        )}
      />
    </form>
  );
}
```

### Dynamic Loading States

```jsx
function LoadingStatesExample() {
  const [options, setOptions] = useState([]);
  const [value, setValue] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  const loadOptions = async () => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/options');
      const data = await response.json();
      setOptions(data);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    loadOptions();
  }, []);

  return (
    <SelectInput
      options={options}
      value={value}
      onChange={setValue}
      isLoading={isLoading}
      isDisabled={isLoading}
      placeholder={isLoading ? 'Loading options...' : 'Select an option'}
    />
  );
}
```

## TypeScript Usage

```typescript
import { 
  SelectInput, 
  AsyncSelectInput,
  CreatableSelectInput,
  AsyncCreatableSelectInput 
} from 'datocms-react-ui';

interface CustomOption {
  value: string;
  label: string;
  metadata?: {
    description?: string;
    icon?: string;
  };
}

// Type-safe select component
function TypedSelectExample() {
  const [value, setValue] = useState<CustomOption | null>(null);

  const options: CustomOption[] = [
    {
      value: 'opt1',
      label: 'Option 1',
      metadata: { description: 'First option' }
    },
    {
      value: 'opt2', 
      label: 'Option 2',
      metadata: { description: 'Second option' }
    }
  ];

  return (
    <SelectInput<CustomOption>
      options={options}
      value={value}
      onChange={setValue}
      formatOptionLabel={(option) => (
        <div>
          <div>{option.label}</div>
          {option.metadata?.description && (
            <div style={{ fontSize: '0.875em', color: 'var(--muted-color)' }}>
              {option.metadata.description}
            </div>
          )}
        </div>
      )}
    />
  );
}

// Type-safe async creatable
function TypedAsyncCreatableExample() {
  const [values, setValues] = useState<CustomOption[]>([]);

  const loadOptions = async (input: string): Promise<CustomOption[]> => {
    // Type-safe API call
    const response = await fetch(`/api/search?q=${input}`);
    const data: CustomOption[] = await response.json();
    return data;
  };

  const createOption = (input: string): void => {
    const newOption: CustomOption = {
      value: input,
      label: input,
      metadata: { description: 'User created' }
    };
    setValues([...values, newOption]);
  };

  return (
    <AsyncCreatableSelectInput<CustomOption, true>
      value={values}
      onChange={setValues}
      loadOptions={loadOptions}
      onCreateOption={createOption}
      isMulti
    />
  );
}
```

## Performance Considerations

1. **Use `cacheOptions`** for `AsyncSelectInput` to avoid repeated API calls
2. **Implement debouncing** for search-heavy async selects
3. **Limit option count** - Return no more than 50-100 options at once
4. **Use `defaultOptions`** wisely - Either `true` for auto-load or provide initial options
5. **Memoize `loadOptions`** to prevent unnecessary re-renders
6. **Virtual scrolling** - For very large lists, consider implementing virtualization

## Best Practices

1. **Error Handling**: Always handle errors in async operations
2. **Loading States**: Show appropriate loading indicators
3. **Empty States**: Provide helpful messages when no options are available
4. **Validation**: Validate user input before creating new options
5. **Accessibility**: Ensure proper ARIA labels and keyboard navigation
6. **Performance**: Debounce search inputs and limit result sets

## When to Use

### Use SelectInput when:
- You need custom form layouts
- You're building a custom field component
- You need direct control over the select
- You're integrating with form libraries

### Use SelectField when:
- You want a complete form field with label and error handling
- You're building standard forms
- You want consistent styling with other form fields

## Related Components

- [SelectField](./SelectField.md) - Complete form field with label and error handling
- [TextField](./TextField.md) - Text input component
- [TextInput](./TextInput.md) - Low-level text input
- [Form](./Form.md) - Form container component