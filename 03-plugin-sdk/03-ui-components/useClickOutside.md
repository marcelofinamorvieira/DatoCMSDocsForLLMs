# useClickOutside Hook

A custom React hook that detects clicks outside of a specified element, useful for closing dropdowns, modals, or other overlays.

## Import

```jsx
import { useClickOutside } from 'datocms-react-ui';
```

## Signature

```typescript
useClickOutside(
  ref: RefObject<HTMLElement>,
  handler: (event: MouseEvent | TouchEvent) => void,
  enabled?: boolean
): void
```

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ref` | `RefObject<HTMLElement>` | - | **Required.** Reference to the element |
| `handler` | `(event) => void` | - | **Required.** Function called when click outside occurs |
| `enabled` | `boolean` | `true` | Whether the hook is active |

## Basic Usage

```jsx
import { useClickOutside, Canvas } from 'datocms-react-ui';
import { useRef, useState } from 'react';

function MyPlugin({ ctx }) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  useClickOutside(dropdownRef, () => {
    setIsOpen(false);
  });

  return (
    <Canvas ctx={ctx}>
      <button onClick={() => setIsOpen(!isOpen)}>
        Toggle Dropdown
      </button>
      
      {isOpen && (
        <div ref={dropdownRef} style={{
          position: 'absolute',
          top: '100%',
          left: 0,
          background: 'white',
          border: '1px solid var(--border-color)',
          borderRadius: 'var(--border-radius-s)',
          padding: 'var(--spacing-m)',
          boxShadow: 'var(--box-shadow-m)'
        }}>
          Dropdown content
        </div>
      )}
    </Canvas>
  );
}
```

## Common Patterns

### Custom Dropdown

```jsx
function CustomDropdown({ options, onSelect }) {
  const [isOpen, setIsOpen] = useState(false);
  const [selected, setSelected] = useState(null);
  const dropdownRef = useRef(null);

  useClickOutside(dropdownRef, () => {
    setIsOpen(false);
  });

  const handleSelect = (option) => {
    setSelected(option);
    onSelect(option);
    setIsOpen(false);
  };

  return (
    <div ref={dropdownRef} style={{ position: 'relative' }}>
      <button
        onClick={() => setIsOpen(!isOpen)}
        style={{
          padding: 'var(--spacing-s) var(--spacing-m)',
          border: '1px solid var(--border-color)',
          borderRadius: 'var(--border-radius-s)',
          background: 'white',
          cursor: 'pointer',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'space-between',
          minWidth: '200px'
        }}
      >
        <span>{selected?.label || 'Select an option'}</span>
        <FaChevronDown />
      </button>
      
      {isOpen && (
        <div style={{
          position: 'absolute',
          top: 'calc(100% + 4px)',
          left: 0,
          right: 0,
          background: 'white',
          border: '1px solid var(--border-color)',
          borderRadius: 'var(--border-radius-s)',
          boxShadow: 'var(--box-shadow-m)',
          maxHeight: '200px',
          overflow: 'auto',
          zIndex: 10
        }}>
          {options.map(option => (
            <div
              key={option.value}
              onClick={() => handleSelect(option)}
              style={{
                padding: 'var(--spacing-s) var(--spacing-m)',
                cursor: 'pointer',
                backgroundColor: selected?.value === option.value 
                  ? 'var(--light-color)' 
                  : 'transparent',
                ':hover': {
                  backgroundColor: 'var(--light-color)'
                }
              }}
            >
              {option.label}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Context Menu

```jsx
function ContextMenu({ children, menuItems }) {
  const [menuPosition, setMenuPosition] = useState(null);
  const menuRef = useRef(null);

  useClickOutside(menuRef, () => {
    setMenuPosition(null);
  });

  const handleContextMenu = (e) => {
    e.preventDefault();
    setMenuPosition({
      x: e.clientX,
      y: e.clientY
    });
  };

  return (
    <>
      <div onContextMenu={handleContextMenu}>
        {children}
      </div>
      
      {menuPosition && (
        <div
          ref={menuRef}
          style={{
            position: 'fixed',
            left: menuPosition.x,
            top: menuPosition.y,
            background: 'white',
            border: '1px solid var(--border-color)',
            borderRadius: 'var(--border-radius-s)',
            boxShadow: 'var(--box-shadow-m)',
            padding: 'var(--spacing-xs)',
            zIndex: 1000
          }}
        >
          {menuItems.map((item, index) => (
            <div
              key={index}
              onClick={() => {
                item.action();
                setMenuPosition(null);
              }}
              style={{
                padding: 'var(--spacing-s) var(--spacing-m)',
                cursor: 'pointer',
                ':hover': {
                  backgroundColor: 'var(--light-color)'
                }
              }}
            >
              {item.icon} {item.label}
            </div>
          ))}
        </div>
      )}
    </>
  );
}
```

### Popover Component

```jsx
function Popover({ trigger, content, placement = 'bottom' }) {
  const [isOpen, setIsOpen] = useState(false);
  const popoverRef = useRef(null);
  const triggerRef = useRef(null);

  useClickOutside(popoverRef, (event) => {
    // Don't close if clicking the trigger
    if (triggerRef.current?.contains(event.target)) {
      return;
    }
    setIsOpen(false);
  });

  const getPlacementStyles = () => {
    switch (placement) {
      case 'top':
        return {
          bottom: 'calc(100% + 8px)',
          left: '50%',
          transform: 'translateX(-50%)'
        };
      case 'right':
        return {
          left: 'calc(100% + 8px)',
          top: '50%',
          transform: 'translateY(-50%)'
        };
      case 'left':
        return {
          right: 'calc(100% + 8px)',
          top: '50%',
          transform: 'translateY(-50%)'
        };
      default: // bottom
        return {
          top: 'calc(100% + 8px)',
          left: '50%',
          transform: 'translateX(-50%)'
        };
    }
  };

  return (
    <div style={{ position: 'relative', display: 'inline-block' }}>
      <div ref={triggerRef} onClick={() => setIsOpen(!isOpen)}>
        {trigger}
      </div>
      
      {isOpen && (
        <div
          ref={popoverRef}
          style={{
            position: 'absolute',
            ...getPlacementStyles(),
            background: 'white',
            border: '1px solid var(--border-color)',
            borderRadius: 'var(--border-radius-m)',
            boxShadow: 'var(--box-shadow-l)',
            padding: 'var(--spacing-m)',
            zIndex: 1000,
            minWidth: '200px'
          }}
        >
          {content}
        </div>
      )}
    </div>
  );
}
```

## Advanced Examples

### Multi-Select with Outside Click

```jsx
function MultiSelect({ options, selected, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  const selectRef = useRef(null);

  useClickOutside(selectRef, () => {
    setIsOpen(false);
  });

  const toggleOption = (value) => {
    if (selected.includes(value)) {
      onChange(selected.filter(v => v !== value));
    } else {
      onChange([...selected, value]);
    }
  };

  return (
    <div ref={selectRef} style={{ position: 'relative' }}>
      <button
        onClick={() => setIsOpen(!isOpen)}
        style={{
          width: '100%',
          padding: 'var(--spacing-s)',
          border: '1px solid var(--border-color)',
          borderRadius: 'var(--border-radius-s)',
          background: 'white',
          textAlign: 'left',
          cursor: 'pointer'
        }}
      >
        {selected.length === 0 
          ? 'Select options...'
          : `${selected.length} selected`}
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
                cursor: 'pointer',
                ':hover': {
                  backgroundColor: 'var(--light-color)'
                }
              }}
            >
              <input
                type="checkbox"
                checked={selected.includes(option.value)}
                onChange={() => toggleOption(option.value)}
                style={{ marginRight: 'var(--spacing-xs)' }}
              />
              {option.label}
            </label>
          ))}
        </div>
      )}
    </div>
  );
}
```

### Color Picker with Outside Click

```jsx
function ColorPicker({ value, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  const pickerRef = useRef(null);

  useClickOutside(pickerRef, () => {
    setIsOpen(false);
  });

  const presetColors = [
    '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4',
    '#FECA57', '#DDA0DD', '#B19CD9', '#FF6B9D',
    '#A8E6CF', '#FFD93D', '#6C5CE7', '#A29BFE'
  ];

  return (
    <div ref={pickerRef} style={{ position: 'relative' }}>
      <div
        onClick={() => setIsOpen(!isOpen)}
        style={{
          width: '40px',
          height: '40px',
          backgroundColor: value || '#ccc',
          border: '2px solid var(--border-color)',
          borderRadius: 'var(--border-radius-s)',
          cursor: 'pointer'
        }}
      />
      
      {isOpen && (
        <div style={{
          position: 'absolute',
          top: '100%',
          left: 0,
          marginTop: '8px',
          background: 'white',
          border: '1px solid var(--border-color)',
          borderRadius: 'var(--border-radius-m)',
          boxShadow: 'var(--box-shadow-l)',
          padding: 'var(--spacing-m)',
          zIndex: 1000
        }}>
          <div style={{
            display: 'grid',
            gridTemplateColumns: 'repeat(4, 1fr)',
            gap: 'var(--spacing-xs)',
            marginBottom: 'var(--spacing-m)'
          }}>
            {presetColors.map(color => (
              <div
                key={color}
                onClick={() => {
                  onChange(color);
                  setIsOpen(false);
                }}
                style={{
                  width: '30px',
                  height: '30px',
                  backgroundColor: color,
                  borderRadius: 'var(--border-radius-s)',
                  cursor: 'pointer',
                  border: value === color ? '2px solid var(--primary-color)' : 'none'
                }}
              />
            ))}
          </div>
          
          <input
            type="color"
            value={value}
            onChange={(e) => onChange(e.target.value)}
            style={{ width: '100%' }}
          />
        </div>
      )}
    </div>
  );
}
```

### Tooltip with Outside Click

```jsx
function Tooltip({ children, content, delay = 300 }) {
  const [isVisible, setIsVisible] = useState(false);
  const tooltipRef = useRef(null);
  const timeoutRef = useRef(null);

  useClickOutside(tooltipRef, () => {
    setIsVisible(false);
  });

  const showTooltip = () => {
    timeoutRef.current = setTimeout(() => {
      setIsVisible(true);
    }, delay);
  };

  const hideTooltip = () => {
    clearTimeout(timeoutRef.current);
    setIsVisible(false);
  };

  useEffect(() => {
    return () => clearTimeout(timeoutRef.current);
  }, []);

  return (
    <div 
      ref={tooltipRef}
      style={{ position: 'relative', display: 'inline-block' }}
      onMouseEnter={showTooltip}
      onMouseLeave={hideTooltip}
    >
      {children}
      
      {isVisible && (
        <div style={{
          position: 'absolute',
          bottom: 'calc(100% + 8px)',
          left: '50%',
          transform: 'translateX(-50%)',
          background: 'rgba(0, 0, 0, 0.8)',
          color: 'white',
          padding: 'var(--spacing-xs) var(--spacing-s)',
          borderRadius: 'var(--border-radius-s)',
          fontSize: 'var(--font-size-s)',
          whiteSpace: 'nowrap',
          zIndex: 1000
        }}>
          {content}
          <div style={{
            position: 'absolute',
            top: '100%',
            left: '50%',
            transform: 'translateX(-50%)',
            width: 0,
            height: 0,
            borderLeft: '4px solid transparent',
            borderRight: '4px solid transparent',
            borderTop: '4px solid rgba(0, 0, 0, 0.8)'
          }} />
        </div>
      )}
    </div>
  );
}
```

### Conditional Usage

```jsx
function ConditionalDropdown({ items, disabled }) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  // Only enable click outside when dropdown is open and not disabled
  useClickOutside(
    dropdownRef, 
    () => setIsOpen(false),
    isOpen && !disabled
  );

  return (
    <div ref={dropdownRef}>
      <button 
        onClick={() => !disabled && setIsOpen(!isOpen)}
        disabled={disabled}
      >
        Menu
      </button>
      
      {isOpen && !disabled && (
        <div>
          {/* Menu items */}
        </div>
      )}
    </div>
  );
}
```

## TypeScript

```typescript
import { useClickOutside } from 'datocms-react-ui';
import { useRef, useState, RefObject } from 'react';

interface DropdownProps<T> {
  options: Array<{ value: T; label: string }>;
  value?: T;
  onChange: (value: T) => void;
}

function TypedDropdown<T>({ options, value, onChange }: DropdownProps<T>) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);

  useClickOutside(dropdownRef, () => {
    setIsOpen(false);
  });

  const selectedOption = options.find(opt => opt.value === value);

  return (
    <div ref={dropdownRef} style={{ position: 'relative' }}>
      <button onClick={() => setIsOpen(!isOpen)}>
        {selectedOption?.label || 'Select...'}
      </button>
      
      {isOpen && (
        <div style={{ position: 'absolute', top: '100%' }}>
          {options.map(option => (
            <div
              key={String(option.value)}
              onClick={() => {
                onChange(option.value);
                setIsOpen(false);
              }}
            >
              {option.label}
            </div>
          ))}
        </div>
      )}
    </div>
  );
}
```

## Best Practices

1. **Ref placement**: Place ref on the outermost container
2. **Event handling**: Don't forget to handle the trigger element
3. **Cleanup**: The hook handles cleanup automatically
4. **Performance**: Use the enabled parameter for conditional behavior
5. **Accessibility**: Ensure keyboard users can close elements too
6. **Z-index**: Ensure dropdowns appear above other content

## Common Issues

### Click on trigger closes immediately

```jsx
// ❌ Wrong - trigger is outside the ref
<button onClick={() => setOpen(!open)}>Open</button>
<div ref={ref}>{open && <Menu />}</div>

// ✅ Correct - both trigger and content are inside ref
<div ref={ref}>
  <button onClick={() => setOpen(!open)}>Open</button>
  {open && <Menu />}
</div>
```

### Multiple overlapping elements

```jsx
// Use separate refs for each element
function MultipleDropdowns() {
  const dropdown1Ref = useRef(null);
  const dropdown2Ref = useRef(null);
  
  useClickOutside(dropdown1Ref, () => setDropdown1Open(false));
  useClickOutside(dropdown2Ref, () => setDropdown2Open(false));
  
  return (
    <>
      <div ref={dropdown1Ref}>{/* Dropdown 1 */}</div>
      <div ref={dropdown2Ref}>{/* Dropdown 2 */}</div>
    </>
  );
}
```

## Related Components

- [Dropdown](./Dropdown.md) - Built-in dropdown component
- [useMediaQuery](./useMediaQuery.md) - Another utility hook