# Dropdown Component

A flexible dropdown menu component for creating contextual menus, action lists, and selection interfaces.

## Import

```jsx
import { Dropdown } from 'datocms-react-ui';
```

## Components

The Dropdown system consists of several components:

- `Dropdown` - Main container component
- `Dropdown.Menu` - The dropdown menu container
- `Dropdown.Option` - Clickable menu items
- `Dropdown.OptionAction` - Action buttons within options
- `Dropdown.Separator` - Visual separator between items
- `Dropdown.Group` - Group related options
- `Dropdown.Text` - Non-interactive text

## Props

### Dropdown Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `renderTrigger` | `(ctx: RenderTriggerCtx) => JSX.Element` | - | **Required.** Render function for trigger element. Receives an object with `open` (boolean) and `onClick` (function) |
| `children` | `React.ReactNode` | - | The dropdown content, typically a Dropdown.Menu component |

### Dropdown.Menu Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `React.ReactNode` | - | Menu items (Options, Groups, Separators, Text) |
| `alignment` | `'left' \| 'right'` | `'left'` | Horizontal alignment of the menu relative to the trigger |

### Dropdown.Option Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `onClick` | `(e: React.MouseEvent<HTMLButtonElement, MouseEvent>) => void` | - | Click handler function |
| `to` | `string` | - | Navigation path (uses ctx.navigateTo) |
| `active` | `boolean` | - | Shows the option in active state |
| `red` | `boolean` | - | Styles the option as a dangerous/destructive action |
| `valid` | `boolean` | - | Shows the option in valid state |
| `invalid` | `boolean` | - | Shows the option in invalid state |
| `children` | `React.ReactNode` | - | Option content and OptionAction components |
| `disabled` | `boolean` | - | Disables the option |
| `closeMenuOnClick` | `boolean` | `true` | Whether to close the menu when clicked |

### Dropdown.OptionAction Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `icon` | `ReactNode` | - | **Required.** Icon to display |
| `red` | `boolean` | - | Styles the action as dangerous/destructive |
| `onClick` | `(e: React.MouseEvent<HTMLButtonElement, MouseEvent>) => void` | - | **Required.** Click handler function |
| `closeMenuOnClick` | `boolean` | `true` | Whether to close the menu when clicked |

### Dropdown.Group Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `React.ReactNode` | - | Group content (typically Options) |
| `name` | `ReactNode` | - | **Required.** Group title/label |

### Dropdown.Separator Props

The Separator component has no props. It renders a visual divider between menu sections.

### Dropdown.Text Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `React.ReactNode` | - | Text content to display |

## Context

### DropdownContext

The Dropdown component provides a context with the following:

| Property | Type | Description |
|----------|------|-------------|
| `closeMenu` | `() => void` | Function to programmatically close the dropdown menu |

### MenuContext

The Menu component provides a context for internal option management:

| Property | Type | Description |
|----------|------|-------------|
| `searchTerm` | `string` | Current search term (when search is active) |
| `addOption` | `(id: string) => void` | Registers an option |
| `setClickHandlerForOption` | `(id: string, handler: (e: SyntheticEvent) => void) => void` | Sets click handler for an option |

## Features

### Automatic Search

When a dropdown menu contains more than 5 options, a search input automatically appears at the top of the menu. This allows users to filter options by typing. The search is case-insensitive and matches against the option's text content.

### Keyboard Navigation

The dropdown supports full keyboard navigation:
- **Arrow Up/Down**: Navigate between options
- **Enter**: Select the currently highlighted option
- **Escape**: Close the dropdown menu
- The selected option is automatically scrolled into view

### Auto-positioning

The menu automatically positions itself to stay within the viewport:
- Prefers to open below the trigger
- Opens above if there's not enough space below
- Adjusts horizontal alignment to stay on screen
- Works with DatoCMS's auto-resizer feature

## Basic Usage

```jsx
import { Dropdown, Button, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Dropdown
        renderTrigger={({ open, onClick }) => (
          <Button onClick={onClick}>
            Actions {open ? '▲' : '▼'}
          </Button>
        )}
      >
        <Dropdown.Menu>
          <Dropdown.Option onClick={() => console.log('Edit')}>
            Edit
          </Dropdown.Option>
          <Dropdown.Option onClick={() => console.log('Duplicate')}>
            Duplicate
          </Dropdown.Option>
          <Dropdown.Separator />
          <Dropdown.Option 
            onClick={() => console.log('Delete')}
            red
          >
            Delete
          </Dropdown.Option>
        </Dropdown.Menu>
      </Dropdown>
    </Canvas>
  );
}
```

### Using OptionAction

```jsx
import { Dropdown, Button, Canvas } from 'datocms-react-ui';
import { FaPlus, FaTrash } from 'react-icons/fa';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Dropdown
        renderTrigger={({ open, onClick }) => (
          <Button onClick={onClick}>
            Manage Items
          </Button>
        )}
      >
        <Dropdown.Menu>
          <Dropdown.Option>
            First Item
            <Dropdown.OptionAction 
              icon={<FaPlus />} 
              onClick={() => console.log('Add action for First Item')}
            />
            <Dropdown.OptionAction 
              icon={<FaTrash />} 
              red
              onClick={() => console.log('Delete First Item')}
            />
          </Dropdown.Option>
          <Dropdown.Option>
            Second Item
            <Dropdown.OptionAction 
              icon={<FaPlus />} 
              onClick={() => console.log('Add action for Second Item')}
            />
            <Dropdown.OptionAction 
              icon={<FaTrash />} 
              red
              onClick={() => console.log('Delete Second Item')}
            />
          </Dropdown.Option>
        </Dropdown.Menu>
      </Dropdown>
    </Canvas>
  );
}
```

## Common Patterns

### Action Menu

```jsx
import { FaEdit, FaCopy, FaTrash, FaDownload } from 'react-icons/fa';

function ActionMenu({ item, onAction }) {
  return (
    <Dropdown
      renderTrigger={(open) => (
        <Button onClick={open} buttonType="muted" size="s">
          Actions
        </Button>
      )}
    >
      <Dropdown.Menu>
        <Dropdown.Option 
          onClick={() => onAction('edit', item)}
          icon={<FaEdit />}
        >
          Edit Item
        </Dropdown.Option>
        
        <Dropdown.Option 
          onClick={() => onAction('duplicate', item)}
          icon={<FaCopy />}
        >
          Duplicate
        </Dropdown.Option>
        
        <Dropdown.Option 
          onClick={() => onAction('export', item)}
          icon={<FaDownload />}
        >
          Export
        </Dropdown.Option>
        
        <Dropdown.Separator />
        
        <Dropdown.Option 
          onClick={() => onAction('delete', item)}
          red
        >
          <FaTrash /> Delete Item
        </Dropdown.Option>
      </Dropdown.Menu>
    </Dropdown>
  );
}
```

### User Menu

```jsx
function UserMenu({ user, ctx }) {
  return (
    <Dropdown
      renderTrigger={({ open, onClick }) => (
        <button onClick={onClick} style={{ 
          display: 'flex', 
          alignItems: 'center',
          gap: 'var(--spacing-s)'
        }}>
          <img 
            src={user.avatar} 
            alt={user.name}
            style={{ 
              width: 32, 
              height: 32, 
              borderRadius: '50%' 
            }}
          />
          <span>{user.name}</span>
        </button>
      )}
      placement="bottom"
    >
      <Dropdown.Menu align="end">
        <Dropdown.Text>
          <strong>{user.name}</strong>
          <br />
          <small>{user.email}</small>
        </Dropdown.Text>
        
        <Dropdown.Separator />
        
        <Dropdown.Option onClick={() => ctx.navigateTo('/settings/profile')}>
          Profile Settings
        </Dropdown.Option>
        
        <Dropdown.Option onClick={() => ctx.navigateTo('/settings/api-tokens')}>
          API Tokens
        </Dropdown.Option>
        
        <Dropdown.Separator />
        
        <Dropdown.Option onClick={() => ctx.logout()}>
          Sign Out
        </Dropdown.Option>
      </Dropdown.Menu>
    </Dropdown>
  );
}
```

### Selection Dropdown

```jsx
function SelectionDropdown({ options, value, onChange, label }) {
  const selectedOption = options.find(opt => opt.value === value);

  return (
    <Dropdown
      renderTrigger={({ onClick }) => (
        <Button onClick={onClick} buttonType="muted">
          {selectedOption?.label || label}
          <FaChevronDown style={{ marginLeft: 'var(--spacing-xs)' }} />
        </Button>
      )}
    >
      <Dropdown.Menu>
        {options.map(option => (
          <Dropdown.Option
            key={option.value}
            onClick={() => onChange(option.value)}
            active={value === option.value}
          >
            {option.icon} {option.label}
          </Dropdown.Option>
        ))}
      </Dropdown.Menu>
    </Dropdown>
  );
}
```

### Grouped Options

```jsx
function GroupedDropdown({ groups }) {
  return (
    <Dropdown
      renderTrigger={({ onClick }) => (
        <Button onClick={onClick}>
          Insert Element
        </Button>
      )}
    >
      <Dropdown.Menu>
        {groups.map((group, index) => (
          <React.Fragment key={group.label}>
            {index > 0 && <Dropdown.Separator />}
            
            <Dropdown.Group name={group.label}>
              {group.items.map(item => (
                <Dropdown.Option
                  key={item.id}
                  onClick={item.action}
                  icon={item.icon}
                  disabled={item.disabled}
                >
                  {item.label}
                </Dropdown.Option>
              ))}
            </Dropdown.Group>
          </React.Fragment>
        ))}
      </Dropdown.Menu>
    </Dropdown>
  );
}
```


## Styling

### Custom Styling

```jsx
<Dropdown
  renderTrigger={({ onClick }) => (
    <Button onClick={onClick}>Custom Styled</Button>
  )}
>
  <Dropdown.Menu>
    {/* The menu automatically handles scrolling when needed */}
    {/* Options */}
  </Dropdown.Menu>
</Dropdown>
```

## Accessibility

- Keyboard navigation (Arrow keys, Enter, Escape)
- ARIA attributes for menu and options
- Focus management
- Screen reader support
- Proper roles (menu, menuitem)

## TypeScript

```typescript
import { Dropdown } from 'datocms-react-ui';

interface Action {
  id: string;
  label: string;
  icon?: React.ReactNode;
  handler: () => void;
  red?: boolean;
}

interface ActionDropdownProps {
  actions: Action[];
  label: string;
}

function ActionDropdown({ actions, label }: ActionDropdownProps) {
  return (
    <Dropdown
      renderTrigger={({ onClick }) => (
        <Button onClick={onClick}>{label}</Button>
      )}
    >
      <Dropdown.Menu>
        {actions.map(action => (
          <Dropdown.Option
            key={action.id}
            onClick={action.handler}
            red={action.red}
          >
            {action.icon} {action.label}
          </Dropdown.Option>
        ))}
      </Dropdown.Menu>
    </Dropdown>
  );
}
```

## Best Practices

1. **Clear trigger**: Make it obvious the element opens a menu
2. **Logical grouping**: Use separators and groups for organization
3. **Keyboard support**: Ensure all actions are keyboard accessible
4. **Auto-search**: Leverage automatic search for menus with many options
5. **Dangerous actions**: Use `red` prop for destructive operations
6. **Option Actions**: Use `OptionAction` for inline actions within options
7. **Close behavior**: Use `closeMenuOnClick` prop to control menu closing

## Related Components

- [Button](./Button.md) - Common trigger component
- [SelectField](./SelectField.md) - Form select alternative
- [Toolbar](./Toolbar.md) - Often contains dropdowns