# Icons

Built-in SVG icons provided by the DatoCMS React UI library for common UI patterns.

## Import

```jsx
import { 
  BackIcon,
  SidebarLeftArrowIcon,
  SidebarRightArrowIcon,
  CaretDownIcon,
  CaretUpIcon,
  ChevronsLeftIcon,
  ChevronsRightIcon,
  SidebarFlipIcon
} from 'datocms-react-ui';
```

## Common Props

All icons accept the following props:

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `width` | `string \| number` | `'1em'` | Icon width |
| `height` | `string \| number` | `'1em'` | Icon height |
| `className` | `string` | - | CSS class name |
| `style` | `CSSProperties` | - | Inline styles |

## Available Icons

### BackIcon

Navigation back arrow, typically used for "go back" actions.

```jsx
<BackIcon width={24} height={24} />
```

### SidebarLeftArrowIcon

Arrow pointing left with a vertical bar, used for collapsing left sidebars.

```jsx
<SidebarLeftArrowIcon style={{ color: 'var(--primary-color)' }} />
```

### SidebarRightArrowIcon

Arrow pointing right with a vertical bar, used for collapsing right sidebars.

```jsx
<SidebarRightArrowIcon className="sidebar-toggle" />
```

### CaretDownIcon

Downward-pointing caret, commonly used in dropdowns and accordions.

```jsx
<CaretDownIcon width={16} height={16} />
```

### CaretUpIcon

Upward-pointing caret, used for expanded states.

```jsx
<CaretUpIcon style={{ transform: isOpen ? 'rotate(180deg)' : 'none' }} />
```

### ChevronsLeftIcon

Double chevron pointing left, used for "skip to start" or major navigation.

```jsx
<ChevronsLeftIcon width={20} height={20} />
```

### ChevronsRightIcon

Double chevron pointing right, used for "skip to end" or major navigation.

```jsx
<ChevronsRightIcon width={20} height={20} />
```

### SidebarFlipIcon

Icon for toggling sidebar position or flipping layout.

```jsx
<SidebarFlipIcon style={{ cursor: 'pointer' }} />
```

## Usage Examples

### In Buttons

```jsx
import { Button, BackIcon, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  return (
    <Canvas ctx={ctx}>
      <Button onClick={() => history.back()}>
        <BackIcon width={16} height={16} style={{ marginRight: '8px' }} />
        Go Back
      </Button>
    </Canvas>
  );
}
```

### In Dropdowns

```jsx
import { Dropdown, CaretDownIcon } from 'datocms-react-ui';

function DropdownExample() {
  return (
    <Dropdown
      renderTrigger={(open) => (
        <button onClick={open}>
          Options <CaretDownIcon width={12} height={12} />
        </button>
      )}
    >
      <Dropdown.Menu>
        <Dropdown.Option>Option 1</Dropdown.Option>
        <Dropdown.Option>Option 2</Dropdown.Option>
      </Dropdown.Menu>
    </Dropdown>
  );
}
```

### In Collapsible Sections

```jsx
import { Section, CaretDownIcon, CaretUpIcon } from 'datocms-react-ui';

function CollapsibleSection({ title, children }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <Section>
      <div 
        onClick={() => setIsOpen(!isOpen)}
        style={{ cursor: 'pointer', display: 'flex', alignItems: 'center' }}
      >
        {isOpen ? <CaretUpIcon /> : <CaretDownIcon />}
        <span style={{ marginLeft: '8px' }}>{title}</span>
      </div>
      {isOpen && children}
    </Section>
  );
}
```

### In Sidebar Controls

```jsx
import { SidebarLeftArrowIcon, SidebarRightArrowIcon } from 'datocms-react-ui';

function SidebarToggle({ isCollapsed, onToggle, position = 'left' }) {
  const Icon = position === 'left' 
    ? (isCollapsed ? SidebarRightArrowIcon : SidebarLeftArrowIcon)
    : (isCollapsed ? SidebarLeftArrowIcon : SidebarRightArrowIcon);

  return (
    <button
      onClick={onToggle}
      style={{
        background: 'none',
        border: 'none',
        cursor: 'pointer',
        padding: 'var(--spacing-s)',
        color: 'var(--text-color)'
      }}
      aria-label={isCollapsed ? 'Expand sidebar' : 'Collapse sidebar'}
    >
      <Icon width={20} height={20} />
    </button>
  );
}
```

### In Pagination

```jsx
import { ChevronsLeftIcon, ChevronsRightIcon, BackIcon } from 'datocms-react-ui';

function Pagination({ currentPage, totalPages, onPageChange }) {
  return (
    <div style={{ display: 'flex', gap: 'var(--spacing-xs)', alignItems: 'center' }}>
      <button 
        onClick={() => onPageChange(1)}
        disabled={currentPage === 1}
        aria-label="First page"
      >
        <ChevronsLeftIcon width={16} height={16} />
      </button>
      
      <button 
        onClick={() => onPageChange(currentPage - 1)}
        disabled={currentPage === 1}
        aria-label="Previous page"
      >
        <BackIcon width={16} height={16} />
      </button>
      
      <span>{currentPage} / {totalPages}</span>
      
      <button 
        onClick={() => onPageChange(currentPage + 1)}
        disabled={currentPage === totalPages}
        aria-label="Next page"
      >
        <BackIcon width={16} height={16} style={{ transform: 'rotate(180deg)' }} />
      </button>
      
      <button 
        onClick={() => onPageChange(totalPages)}
        disabled={currentPage === totalPages}
        aria-label="Last page"
      >
        <ChevronsRightIcon width={16} height={16} />
      </button>
    </div>
  );
}
```

## Styling Icons

### With CSS Variables

```jsx
<BackIcon 
  style={{ 
    color: 'var(--primary-color)',
    fill: 'currentColor'
  }} 
/>
```

### Animated Icons

```jsx
function AnimatedCaret({ isOpen }) {
  return (
    <CaretDownIcon 
      style={{ 
        transition: 'transform 0.2s ease',
        transform: isOpen ? 'rotate(180deg)' : 'none'
      }} 
    />
  );
}
```

### Icon Buttons

```jsx
function IconButton({ icon: Icon, onClick, ...props }) {
  return (
    <button
      onClick={onClick}
      style={{
        background: 'none',
        border: '1px solid var(--border-color)',
        borderRadius: 'var(--border-radius-s)',
        padding: 'var(--spacing-s)',
        cursor: 'pointer',
        display: 'inline-flex',
        alignItems: 'center',
        justifyContent: 'center',
        transition: 'all 0.2s ease',
        ':hover': {
          backgroundColor: 'var(--light-color)'
        }
      }}
      {...props}
    >
      <Icon width={20} height={20} />
    </button>
  );
}

// Usage
<IconButton icon={BackIcon} onClick={handleBack} aria-label="Go back" />
```

## TypeScript

```typescript
import { IconProps, BackIcon } from 'datocms-react-ui';

interface CustomIconButtonProps {
  icon: React.ComponentType<IconProps>;
  onClick: () => void;
  size?: number;
  color?: string;
}

function CustomIconButton({ 
  icon: Icon, 
  onClick, 
  size = 24, 
  color = 'currentColor' 
}: CustomIconButtonProps) {
  return (
    <button onClick={onClick}>
      <Icon width={size} height={size} style={{ color }} />
    </button>
  );
}
```

## Best Practices

1. **Consistent sizing**: Use consistent icon sizes throughout your plugin
2. **Accessibility**: Always provide aria-labels for icon-only buttons
3. **Color inheritance**: Icons use `currentColor` by default
4. **Responsive sizing**: Use relative units (em) for scalable icons
5. **Meaningful usage**: Use icons that match their intended purpose

## Related Components

- [Button](./Button.md) - For icon buttons
- [Toolbar](./Toolbar.md) - Icons in toolbar buttons
- [Dropdown](./Dropdown.md) - Caret icons for dropdowns
- [SidebarPanel](./SidebarPanel.md) - Collapse/expand icons