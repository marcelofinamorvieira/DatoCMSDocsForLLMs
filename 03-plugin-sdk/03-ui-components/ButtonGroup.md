# ButtonGroup Component

A container for grouping related buttons together with proper spacing and visual cohesion.

## Import

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';
```

## Exports

The ButtonGroup module exports:
- `ButtonGroup` - The container component (exported as `Group`)
- `ButtonGroupButton` - Individual button component (exported as `Button`)
- `ButtonGroupProps` - TypeScript type for ButtonGroup props
- `ButtonGroupButtonProps` - TypeScript type for ButtonGroupButton props

## ButtonGroup Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | ButtonGroupButton components to render inside the group |
| `className` | `string` | - | Additional CSS classes to apply to the container |
| `style` | `CSSProperties` | - | Inline styles for the container |

## ButtonGroupButton Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Button content |
| `onClick` | `MouseEventHandler` | - | Click event handler |
| `selected` | `boolean` | - | Whether the button is currently selected/active |
| `disabled` | `boolean` | - | Whether the button is disabled |
| `className` | `string` | - | Additional CSS classes to apply to the button |
| `style` | `CSSProperties` | - | Inline styles for the button |

## Differences from Regular Button Component

The `ButtonGroupButton` component is a simplified version of the regular `Button` component, specifically designed for use within a ButtonGroup:

### ButtonGroupButton Features:
- **Simplified props**: Only includes essential props (selected, disabled, onClick)
- **No size variations**: Uses a fixed size optimized for group display
- **No button types**: No primary/muted/negative variants
- **No icons**: No leftIcon/rightIcon props
- **Selected state**: Has a `selected` prop for toggle-like behavior
- **Automatic styling**: Styled to connect seamlessly with adjacent buttons
- **Border management**: Automatic border radius and border collapsing between buttons

### Regular Button Features (not in ButtonGroupButton):
- `buttonType`: 'primary' | 'muted' | 'negative'
- `buttonSize`: 'xxs' | 'xs' | 's' | 'm' | 'l' | 'xl'
- `leftIcon` and `rightIcon` props
- `fullWidth` prop
- `type` prop for form submission
- Standalone styling with full borders and spacing

## Basic Usage

```jsx
import { ButtonGroup, ButtonGroupButton, Canvas } from 'datocms-react-ui';
import { useState } from 'react';

function MyPlugin({ ctx }) {
  const [view, setView] = useState('grid');

  return (
    <Canvas ctx={ctx}>
      <ButtonGroup>
        <ButtonGroupButton 
          selected={view === 'grid'}
          onClick={() => setView('grid')}
        >
          Grid View
        </ButtonGroupButton>
        <ButtonGroupButton 
          selected={view === 'list'}
          onClick={() => setView('list')}
        >
          List View
        </ButtonGroupButton>
      </ButtonGroup>
    </Canvas>
  );
}
```

## Use Cases

### View Switcher

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';
import { FaThLarge, FaList, FaCalendar } from 'react-icons/fa';

function ViewSwitcher({ currentView, onViewChange }) {
  const views = [
    { id: 'grid', label: 'Grid', icon: <FaThLarge /> },
    { id: 'list', label: 'List', icon: <FaList /> },
    { id: 'calendar', label: 'Calendar', icon: <FaCalendar /> }
  ];

  return (
    <ButtonGroup>
      {views.map(view => (
        <ButtonGroupButton
          key={view.id}
          selected={currentView === view.id}
          onClick={() => onViewChange(view.id)}
        >
          {view.icon}
          <span style={{ marginLeft: 'var(--spacing-xs)' }}>
            {view.label}
          </span>
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

### Size Selector

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

function SizeSelector({ selectedSize, onSizeChange }) {
  const sizes = ['S', 'M', 'L', 'XL'];

  return (
    <ButtonGroup>
      {sizes.map(size => (
        <ButtonGroupButton
          key={size}
          selected={selectedSize === size}
          onClick={() => onSizeChange(size)}
        >
          {size}
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

### Filter Options

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

function FilterButtons({ activeFilters, toggleFilter }) {
  const filters = [
    { id: 'published', label: 'Published' },
    { id: 'draft', label: 'Draft' },
    { id: 'scheduled', label: 'Scheduled' }
  ];

  return (
    <ButtonGroup>
      {filters.map(filter => (
        <ButtonGroupButton
          key={filter.id}
          selected={activeFilters.includes(filter.id)}
          onClick={() => toggleFilter(filter.id)}
        >
          {filter.label}
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

### Alignment Controls

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';
import { 
  FaAlignLeft, 
  FaAlignCenter, 
  FaAlignRight, 
  FaAlignJustify 
} from 'react-icons/fa';

function TextAlignment({ alignment, onChange }) {
  const options = [
    { value: 'left', icon: <FaAlignLeft /> },
    { value: 'center', icon: <FaAlignCenter /> },
    { value: 'right', icon: <FaAlignRight /> },
    { value: 'justify', icon: <FaAlignJustify /> }
  ];

  return (
    <ButtonGroup>
      {options.map(option => (
        <ButtonGroupButton
          key={option.value}
          selected={alignment === option.value}
          onClick={() => onChange(option.value)}
          title={`Align ${option.value}`}
        >
          {option.icon}
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

## Advanced Examples

### Multi-Select Button Group

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

function MultiSelectButtonGroup({ selectedValues, onChange }) {
  const options = ['Bold', 'Italic', 'Underline'];

  const toggleOption = (option) => {
    if (selectedValues.includes(option)) {
      onChange(selectedValues.filter(v => v !== option));
    } else {
      onChange([...selectedValues, option]);
    }
  };

  return (
    <ButtonGroup>
      {options.map(option => (
        <ButtonGroupButton
          key={option}
          selected={selectedValues.includes(option)}
          onClick={() => toggleOption(option)}
        >
          {option}
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

### Tab-like Navigation

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

function TabNavigation({ activeTab, tabs, onTabChange }) {
  return (
    <div>
      <ButtonGroup>
        {tabs.map(tab => (
          <ButtonGroupButton
            key={tab.id}
            selected={activeTab === tab.id}
            onClick={() => onTabChange(tab.id)}
            disabled={tab.disabled}
          >
            {tab.label}
            {tab.count !== undefined && (
              <span style={{ marginLeft: 'var(--spacing-xs)' }}>
                ({tab.count})
              </span>
            )}
          </ButtonGroupButton>
        ))}
      </ButtonGroup>
      
      <div style={{ marginTop: 'var(--spacing-m)' }}>
        {tabs.find(t => t.id === activeTab)?.content}
      </div>
    </div>
  );
}
```

### Date Range Selector

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

function DateRangeSelector({ range, onChange }) {
  const ranges = [
    { id: 'today', label: 'Today' },
    { id: 'week', label: 'This Week' },
    { id: 'month', label: 'This Month' },
    { id: 'year', label: 'This Year' },
    { id: 'custom', label: 'Custom...' }
  ];

  return (
    <ButtonGroup>
      {ranges.map(r => (
        <ButtonGroupButton
          key={r.id}
          selected={range === r.id}
          onClick={() => {
            if (r.id === 'custom') {
              // Open date picker modal
              openCustomDatePicker();
            } else {
              onChange(r.id);
            }
          }}
        >
          {r.label}
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

### With Keyboard Navigation

**Note**: The ButtonGroupButton component accepts basic button props but doesn't include advanced keyboard event handlers or ARIA attributes. The component focuses on visual grouping rather than complex accessibility features.

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

function SimpleButtonGroup({ options, value, onChange }) {
  return (
    <ButtonGroup>
      {options.map((option) => (
        <ButtonGroupButton
          key={option.value}
          selected={value === option.value}
          onClick={() => onChange(option.value)}
        >
          {option.label}
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

## Styling

### Custom Styling

```jsx
import { ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

<ButtonGroup 
  style={{ 
    borderRadius: 'var(--border-radius-l)',
    boxShadow: 'var(--box-shadow-m)'
  }}
  className="my-button-group"
>
  <ButtonGroupButton>Option 1</ButtonGroupButton>
  <ButtonGroupButton>Option 2</ButtonGroupButton>
</ButtonGroup>
```

### CSS Variables

ButtonGroup uses the following CSS variables from the DatoCMS theme:

```css
/* Border and colors */
--border-color              /* Border between buttons */
--accent-color              /* Background for selected buttons */
--accent-color-rgb-components /* For hover states on selected buttons */
--light-bg-color            /* Hover background for unselected buttons */
--base-body-color           /* Text color */
--base-body-color-rgb-components /* For semi-transparent text */
--light-body-color          /* Disabled state text and icon color */
```

## Patterns with Other Components

### In a Toolbar

```jsx
import { Toolbar, ButtonGroup, ButtonGroupButton } from 'datocms-react-ui';

function EditorToolbar({ formatting, onChange }) {
  return (
    <Toolbar.Toolbar>
      <Toolbar.Stack>
        <ButtonGroup>
          <ButtonGroupButton
            selected={formatting.bold}
            onClick={() => onChange({ ...formatting, bold: !formatting.bold })}
          >
            B
          </ButtonGroupButton>
          <ButtonGroupButton
            selected={formatting.italic}
            onClick={() => onChange({ ...formatting, italic: !formatting.italic })}
          >
            I
          </ButtonGroupButton>
        </ButtonGroup>
      </Toolbar.Stack>
    </Toolbar.Toolbar>
  );
}
```

### With Form Fields

```jsx
import { Form, FieldGroup, ButtonGroup, ButtonGroupButton, TextField, Canvas, FormLabel } from 'datocms-react-ui';
import { useState } from 'react';

function SearchForm({ ctx }) {
  const [searchType, setSearchType] = useState('content');
  const [query, setQuery] = useState('');

  return (
    <Canvas ctx={ctx}>
      <Form>
        <FieldGroup>
          <FormLabel>Search in:</FormLabel>
          <ButtonGroup>
            <ButtonGroupButton
              selected={searchType === 'content'}
              onClick={() => setSearchType('content')}
            >
              Content
            </ButtonGroupButton>
            <ButtonGroupButton
              selected={searchType === 'media'}
              onClick={() => setSearchType('media')}
            >
              Media
            </ButtonGroupButton>
            <ButtonGroupButton
              selected={searchType === 'all'}
              onClick={() => setSearchType('all')}
            >
              All
            </ButtonGroupButton>
          </ButtonGroup>
        </FieldGroup>
        
        <TextField
          label="Search query"
          value={query}
          onChange={setQuery}
          placeholder={`Search in ${searchType}...`}
        />
      </Form>
    </Canvas>
  );
}
```

## Accessibility

- Proper button roles
- Keyboard navigation support
- ARIA attributes for selected state
- Focus management
- Screen reader support

## TypeScript

```typescript
import { ButtonGroup, ButtonGroupButton, type ButtonGroupProps, type ButtonGroupButtonProps } from 'datocms-react-ui';

type ViewMode = 'grid' | 'list' | 'calendar';

interface ViewSwitcherProps {
  currentView: ViewMode;
  onViewChange: (view: ViewMode) => void;
}

function ViewSwitcher({ currentView, onViewChange }: ViewSwitcherProps) {
  const views: ViewMode[] = ['grid', 'list', 'calendar'];

  return (
    <ButtonGroup>
      {views.map(view => (
        <ButtonGroupButton
          key={view}
          selected={currentView === view}
          onClick={() => onViewChange(view)}
        >
          {view.charAt(0).toUpperCase() + view.slice(1)}
        </ButtonGroupButton>
      ))}
    </ButtonGroup>
  );
}
```

## Best Practices

1. **Group related actions**: Only group buttons that perform related functions
2. **Indicate selection clearly**: Use `selected` prop for active states
3. **Limit group size**: 2-5 buttons per group for best usability
4. **Use icons wisely**: Icons can save space but include text when possible
5. **Keyboard navigation**: Support arrow keys for accessibility
6. **Mobile consideration**: Ensure buttons are touch-friendly

## Related Components

- [Button](./Button.md) - Individual button component
- [Toolbar](./Toolbar.md) - Toolbar container
- [Form](./Form.md) - Form integration