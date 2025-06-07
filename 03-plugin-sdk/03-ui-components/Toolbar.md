# Toolbar Component

A toolbar component system that provides a consistent way to create toolbars with titles, buttons, and actions in DatoCMS plugins.

## Import

```jsx
import { Toolbar, ToolbarButton, ToolbarTitle, ToolbarStack } from 'datocms-react-ui';
```

## Components

The Toolbar system consists of multiple components that work together to create flexible toolbars:

### Toolbar

The main toolbar container component that provides the structural layout for toolbar content.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Toolbar content (typically ToolbarStack components) |
| `className` | `string` | - | Additional CSS classes to apply to the toolbar |
| `style` | `CSSProperties` | - | Inline styles to apply to the toolbar |

### ToolbarTitle

A component for displaying the toolbar's main title or heading.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | The title text or content to display |
| `className` | `string` | - | Additional CSS classes to apply to the title |
| `style` | `CSSProperties` | - | Inline styles to apply to the title |

### ToolbarButton

A specialized button component designed for use within toolbars.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Button content (text, icons, or both) |
| `onClick` | `MouseEventHandler` | - | Click event handler function |
| `className` | `string` | - | Additional CSS classes to apply to the button |
| `style` | `CSSProperties` | - | Inline styles to apply to the button |

### ToolbarStack

A layout component that groups toolbar items together with configurable spacing.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | - | Items to be grouped together in the stack |
| `stackSize` | `'s' \| 'm' \| 'l'` | `'m'` | Controls the spacing between items in the stack |
| `className` | `string` | - | Additional CSS classes to apply to the stack |
| `style` | `CSSProperties` | - | Inline styles to apply to the stack |

## Basic Usage

```jsx
import { Toolbar, ToolbarButton, ToolbarTitle, ToolbarStack, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Toolbar>
        <ToolbarStack stackSize="l">
          <ToolbarTitle>My Toolbar</ToolbarTitle>
        </ToolbarStack>
        
        <ToolbarStack>
          <ToolbarButton onClick={() => console.log('Save')}>
            Save
          </ToolbarButton>
          <ToolbarButton onClick={() => console.log('Cancel')}>
            Cancel
          </ToolbarButton>
        </ToolbarStack>
      </Toolbar>
    </Canvas>
  );
}
```

## How Components Work Together

The Toolbar system is designed with composition in mind:

1. **Toolbar** - The container that provides the flex layout and borders
2. **ToolbarStack** - Groups items horizontally with configurable spacing
3. **ToolbarTitle** - Displays the main heading within a stack
4. **ToolbarButton** - Specialized buttons styled for toolbar use

### Component Hierarchy

```jsx
<Toolbar>
  <ToolbarStack>           // Left-aligned group
    <ToolbarButton />      // Navigation button
    <ToolbarTitle />       // Main title
  </ToolbarStack>
  
  <ToolbarStack>           // Right-aligned group
    <Button />             // Primary action (regular button)
    <ToolbarButton />      // Secondary action
  </ToolbarStack>
</Toolbar>
```

## Common Patterns

### Basic Toolbar with Title

```jsx
import { Toolbar, ToolbarStack, ToolbarTitle, Canvas } from 'datocms-react-ui';

function BasicToolbar({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Toolbar>
        <ToolbarStack stackSize="l">
          <ToolbarTitle>Media Area</ToolbarTitle>
        </ToolbarStack>
      </Toolbar>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
          alignItems: 'center',
          background: 'var(--light-bg-color)',
          height: '150px',
        }}
      >
        Main content
      </div>
    </Canvas>
  );
}
```

### Toolbar with Buttons and Actions

```jsx
import { Toolbar, ToolbarButton, ToolbarStack, ToolbarTitle, Button, Canvas } from 'datocms-react-ui';
import { BackIcon, SidebarLeftArrowIcon } from 'datocms-react-ui';

function ActionToolbar({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Toolbar>
        <ToolbarButton>
          <BackIcon />
        </ToolbarButton>
        <ToolbarStack stackSize="l">
          <ToolbarTitle>Media Area</ToolbarTitle>
          <div style={{ flex: '1' }} />
          <Button buttonType="primary">Action</Button>
        </ToolbarStack>
        <ToolbarButton>
          <SidebarLeftArrowIcon />
        </ToolbarButton>
      </Toolbar>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
          alignItems: 'center',
          background: 'var(--light-bg-color)',
          height: '150px',
        }}
      >
        Main content
      </div>
    </Canvas>
  );
}
```

### Toolbar with Button Group

```jsx
import { Toolbar, ToolbarStack, ToolbarTitle, ButtonGroup, ButtonGroupButton, Canvas } from 'datocms-react-ui';

function ButtonGroupToolbar({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Toolbar>
        <ToolbarStack stackSize="l">
          <ToolbarTitle>Media Area</ToolbarTitle>
          <div style={{ flex: '1' }} />
          <ButtonGroup>
            <ButtonGroupButton>First</ButtonGroupButton>
            <ButtonGroupButton selected>Second</ButtonGroupButton>
            <ButtonGroupButton>Third</ButtonGroupButton>
          </ButtonGroup>
        </ToolbarStack>
      </Toolbar>
      <div
        style={{
          display: 'flex',
          justifyContent: 'center',
          alignItems: 'center',
          background: 'var(--light-bg-color)',
          height: '150px',
        }}
      >
        Main content
      </div>
    </Canvas>
  );
}
```

## Advanced Examples

### Toolbar with Different Stack Sizes

```jsx
import { Toolbar, ToolbarButton, ToolbarStack, ToolbarTitle } from 'datocms-react-ui';

function StackSizeExample({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Toolbar>
        <ToolbarStack stackSize="s">
          <ToolbarButton onClick={() => console.log('Small')}>
            Small
          </ToolbarButton>
          <ToolbarButton onClick={() => console.log('Stack')}>
            Stack
          </ToolbarButton>
        </ToolbarStack>
        
        <ToolbarStack stackSize="m">
          <ToolbarTitle>Medium Stack</ToolbarTitle>
          <ToolbarButton onClick={() => console.log('Medium')}>
            Medium
          </ToolbarButton>
        </ToolbarStack>
        
        <ToolbarStack stackSize="l">
          <ToolbarButton onClick={() => console.log('Large')}>
            Large
          </ToolbarButton>
          <ToolbarButton onClick={() => console.log('Stack')}>
            Stack
          </ToolbarButton>
        </ToolbarStack>
      </Toolbar>
    </Canvas>
  );
}
```

### Toolbar with Multiple Actions

```jsx
import { Toolbar, ToolbarButton, ToolbarStack, ToolbarTitle, Button } from 'datocms-react-ui';
import { EditIcon, DeleteIcon, AddIcon } from 'datocms-react-ui';

function MultiActionToolbar({ selectedCount, onAction }) {
  return (
    <Toolbar>
      <ToolbarStack stackSize="l">
        <ToolbarTitle>
          {selectedCount > 0 
            ? `${selectedCount} items selected`
            : 'All Items'
          }
        </ToolbarTitle>
      </ToolbarStack>
      
      <ToolbarStack>
        {selectedCount > 0 && (
          <>
            <ToolbarButton onClick={() => onAction('edit')}>
              <EditIcon /> Edit
            </ToolbarButton>
            
            <ToolbarButton onClick={() => onAction('delete')}>
              <DeleteIcon /> Delete
            </ToolbarButton>
          </>
        )}
        
        <Button buttonType="primary" onClick={() => onAction('add')}>
          <AddIcon /> Add New
        </Button>
      </ToolbarStack>
    </Toolbar>
  );
}
```

### Toolbar with Custom Styling

```jsx
import { Toolbar, ToolbarButton, ToolbarStack, ToolbarTitle } from 'datocms-react-ui';

function StyledToolbar({ ctx }) {
  return (
    <Toolbar
      style={{
        backgroundColor: 'var(--light-bg-color)',
        padding: 'var(--spacing-xs) 0'
      }}
      className="custom-toolbar"
    >
      <ToolbarStack stackSize="l">
        <ToolbarTitle style={{ fontWeight: 'bold' }}>
          Custom Styled Toolbar
        </ToolbarTitle>
      </ToolbarStack>
      
      <ToolbarStack>
        <ToolbarButton 
          onClick={() => console.log('Action 1')}
          style={{ color: 'var(--primary-color)' }}
        >
          Primary Action
        </ToolbarButton>
        
        <ToolbarButton 
          onClick={() => console.log('Action 2')}
          className="secondary-button"
        >
          Secondary Action
        </ToolbarButton>
      </ToolbarStack>
    </Toolbar>
  );
}
```

### Toolbar with Mixed Components

```jsx
import { Toolbar, ToolbarButton, ToolbarStack, ToolbarTitle, Button, TextInput } from 'datocms-react-ui';
import { SearchIcon } from 'datocms-react-ui';

function MixedComponentsToolbar({ onSearch }) {
  const [searchValue, setSearchValue] = useState('');

  return (
    <Toolbar>
      <ToolbarStack stackSize="l">
        <ToolbarTitle>Search Toolbar</ToolbarTitle>
        
        <TextInput
          value={searchValue}
          onChange={(e) => setSearchValue(e.target.value)}
          placeholder="Search..."
          prefix={<SearchIcon />}
          style={{ width: '200px' }}
        />
      </ToolbarStack>
      
      <ToolbarStack stackSize="s">
        <Button 
          buttonType="primary" 
          onClick={() => onSearch(searchValue)}
        >
          Search
        </Button>
        
        <ToolbarButton onClick={() => setSearchValue('')}>
          Clear
        </ToolbarButton>
      </ToolbarStack>
    </Toolbar>
  );
}
```

### Responsive Toolbar

```jsx
import { Toolbar, ToolbarButton, ToolbarStack, ToolbarTitle, useMediaQuery } from 'datocms-react-ui';
import { MenuIcon } from 'datocms-react-ui';

function ResponsiveToolbar({ title, actions }) {
  const isMobile = useMediaQuery('(max-width: 768px)');

  return (
    <Toolbar>
      <ToolbarStack stackSize={isMobile ? 's' : 'l'}>
        <ToolbarTitle>{title}</ToolbarTitle>
      </ToolbarStack>
      
      {isMobile ? (
        <ToolbarButton onClick={() => console.log('Open menu')}>
          <MenuIcon />
        </ToolbarButton>
      ) : (
        <ToolbarStack>
          {actions.map(action => (
            <ToolbarButton 
              key={action.id} 
              onClick={action.handler}
            >
              {action.icon} {action.label}
            </ToolbarButton>
          ))}
        </ToolbarStack>
      )}
    </Toolbar>
  );
}
```

### Toolbar with Flex Layout

```jsx
import { Toolbar, ToolbarButton, ToolbarStack, ToolbarTitle, Button } from 'datocms-react-ui';

function FlexLayoutToolbar({ title, leftActions, rightActions }) {
  return (
    <Toolbar>
      <ToolbarStack stackSize="m">
        {leftActions.map(action => (
          <ToolbarButton 
            key={action.id} 
            onClick={action.handler}
          >
            {action.icon}
          </ToolbarButton>
        ))}
      </ToolbarStack>
      
      <ToolbarStack stackSize="l" style={{ flex: 1 }}>
        <ToolbarTitle>{title}</ToolbarTitle>
        <div style={{ flex: 1 }} />
        {rightActions.map(action => (
          <Button 
            key={action.id}
            buttonType={action.primary ? 'primary' : 'default'}
            onClick={action.handler}
          >
            {action.label}
          </Button>
        ))}
      </ToolbarStack>
    </Toolbar>
  );
}
```

## Styling

The Toolbar components support custom styling through both `style` and `className` props:

### Inline Styles

```jsx
<Toolbar style={{ backgroundColor: 'var(--light-bg-color)' }}>
  <ToolbarStack stackSize="l">
    <ToolbarTitle style={{ fontWeight: 'bold', color: 'var(--primary-color)' }}>
      Styled Title
    </ToolbarTitle>
  </ToolbarStack>
  
  <ToolbarButton style={{ marginLeft: 'auto' }}>
    Action
  </ToolbarButton>
</Toolbar>
```

### CSS Classes

```jsx
<Toolbar className="my-custom-toolbar">
  <ToolbarStack className="toolbar-left">
    <ToolbarTitle className="toolbar-title">Title</ToolbarTitle>
  </ToolbarStack>
  
  <ToolbarButton className="toolbar-action">
    Action
  </ToolbarButton>
</Toolbar>
```

## Layout Considerations

### Toolbar Structure

The Toolbar component uses flexbox layout:
- Items are arranged horizontally
- ToolbarStack components help group related items
- Use `flex: 1` on elements to create flexible spacing
- The toolbar has borders on top and bottom by default

### Stack Sizes

The `stackSize` prop on ToolbarStack controls spacing between items:
- `'s'` - Small spacing for tightly grouped items
- `'m'` - Medium spacing (default)
- `'l'` - Large spacing for more separation

## TypeScript

```typescript
import { 
  Toolbar, 
  ToolbarButton, 
  ToolbarTitle, 
  ToolbarStack,
  ToolbarProps,
  ToolbarButtonProps,
  ToolbarTitleProps,
  ToolbarStackProps
} from 'datocms-react-ui';
import { ReactNode, CSSProperties, MouseEventHandler } from 'react';

// The components have the following type definitions:

// ToolbarProps
interface ToolbarProps {
  children?: ReactNode;
  className?: string;
  style?: CSSProperties;
}

// ToolbarTitleProps  
interface ToolbarTitleProps {
  children?: ReactNode;
  className?: string;
  style?: CSSProperties;
}

// ToolbarButtonProps
interface ToolbarButtonProps {
  children?: ReactNode;
  onClick?: MouseEventHandler;
  className?: string;
  style?: CSSProperties;
}

// ToolbarStackProps
interface ToolbarStackProps {
  children?: ReactNode;
  stackSize?: 's' | 'm' | 'l';
  className?: string;
  style?: CSSProperties;
}

// Example usage with custom types
interface ToolbarAction {
  id: string;
  label: string;
  icon?: ReactNode;
  handler: () => void;
}

function TypedToolbar({ title, actions }: { title: string; actions: ToolbarAction[] }) {
  return (
    <Toolbar>
      <ToolbarStack stackSize="l">
        <ToolbarTitle>{title}</ToolbarTitle>
      </ToolbarStack>
      
      <ToolbarStack>
        {actions.map(action => (
          <ToolbarButton
            key={action.id}
            onClick={action.handler}
          >
            {action.icon} {action.label}
          </ToolbarButton>
        ))}
      </ToolbarStack>
    </Toolbar>
  );
}
```

## Best Practices

1. **Component Organization**:
   - Use `ToolbarStack` to group related items
   - Place primary actions on the right, secondary on the left
   - Use `stackSize` prop to control visual grouping

2. **Title Usage**:
   - Use `ToolbarTitle` for the main toolbar label
   - Keep titles concise and descriptive
   - Consider dynamic titles that reflect current state

3. **Button Patterns**:
   - Use `ToolbarButton` for toolbar-specific actions
   - Use regular `Button` component for primary actions
   - Include both icon and text for clarity when possible

4. **Layout Tips**:
   - Use `flex: 1` on a div within ToolbarStack to create flexible spacing
   - Combine multiple ToolbarStack components for complex layouts
   - Consider responsive behavior for mobile views

5. **Styling Guidelines**:
   - Maintain consistent styling with DatoCMS design system
   - Use CSS variables for colors and spacing
   - Apply custom styles sparingly and purposefully

## Related Components

- [Button](./Button.md) - Standard button component for primary actions
- [ButtonGroup](./ButtonGroup.md) - Group related buttons together
- [Canvas](./Canvas.md) - Container component for plugin UI
- [TextInput](./TextInput.md) - Text input for search functionality
- [Dropdown](./Dropdown.md) - Dropdown menus for additional actions
- [Icons](./icons.md) - Available icons for toolbar buttons