# mergeRefs

A utility function that combines multiple React refs into a single ref callback. This is essential when you need to attach multiple refs to the same DOM element, such as combining a forwarded ref with an internal ref.

## Import

```jsx
import { mergeRefs } from 'datocms-react-ui';
```

## Function Signature

```typescript
export const mergeRefs = <T>(...refs: React.Ref<T>[]): React.RefCallback<T>
```

## Parameters

- **`...refs`** - Variable number of React refs to merge
  - Can be callback refs: `(instance: T) => void`
  - Can be ref objects: `React.MutableRefObject<T>` or `React.RefObject<T>`
  - Can be `null` or `undefined` (safely ignored)

## Return Value

Returns a `React.RefCallback<T>` that updates all provided refs when called.

## How It Works

The returned callback function:
1. Iterates through all provided refs
2. For ref objects, sets the `.current` property
3. For callback refs, calls the function with the instance
4. Safely handles null/undefined refs

## Usage Examples

### Combining ForwardRef with Internal Ref

```typescript
import React, { forwardRef, useRef, useImperativeHandle } from 'react';
import { mergeRefs } from 'datocms-react-ui';

interface CustomInputProps {
  label: string;
}

interface CustomInputHandle {
  focus: () => void;
  clear: () => void;
}

const CustomInput = forwardRef<CustomInputHandle, CustomInputProps>(
  ({ label }, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);
    
    // Expose methods via ref
    useImperativeHandle(ref, () => ({
      focus: () => inputRef.current?.focus(),
      clear: () => {
        if (inputRef.current) {
          inputRef.current.value = '';
        }
      }
    }));
    
    return (
      <div>
        <label>{label}</label>
        <input ref={inputRef} type="text" />
      </div>
    );
  }
);

// Using the component
function Form() {
  const customInputRef = useRef<CustomInputHandle>(null);
  
  const handleSubmit = () => {
    customInputRef.current?.clear();
  };
  
  return (
    <form>
      <CustomInput ref={customInputRef} label="Name" />
      <button type="button" onClick={handleSubmit}>Clear</button>
    </form>
  );
}
```

### Multiple Refs on Same Element

```typescript
import { useRef, useEffect } from 'react';
import { mergeRefs, useClickOutside } from 'datocms-react-ui';

function Dropdown() {
  const measureRef = useRef<HTMLDivElement>(null);
  const clickOutsideRef = useClickOutside(() => {
    console.log('Clicked outside');
  });
  
  useEffect(() => {
    if (measureRef.current) {
      console.log('Width:', measureRef.current.offsetWidth);
    }
  }, []);
  
  return (
    <div ref={mergeRefs(measureRef, clickOutsideRef)}>
      <h3>Dropdown Menu</h3>
      <ul>
        <li>Option 1</li>
        <li>Option 2</li>
      </ul>
    </div>
  );
}
```

### With Callback Refs

```typescript
import { useState } from 'react';
import { mergeRefs } from 'datocms-react-ui';

function MeasuredComponent() {
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });
  
  // Callback ref that measures element
  const measureCallback = (element: HTMLDivElement | null) => {
    if (element) {
      setDimensions({
        width: element.offsetWidth,
        height: element.offsetHeight
      });
    }
  };
  
  // Regular ref for other purposes
  const elementRef = useRef<HTMLDivElement>(null);
  
  return (
    <div ref={mergeRefs(measureCallback, elementRef)}>
      <p>Width: {dimensions.width}px</p>
      <p>Height: {dimensions.height}px</p>
    </div>
  );
}
```

### In a Custom Hook

```typescript
import { useRef, useEffect, RefObject } from 'react';
import { mergeRefs } from 'datocms-react-ui';

function useHover<T extends HTMLElement>(): [RefObject<T>, boolean] {
  const ref = useRef<T>(null);
  const [isHovered, setIsHovered] = useState(false);
  
  useEffect(() => {
    const element = ref.current;
    if (!element) return;
    
    const handleMouseEnter = () => setIsHovered(true);
    const handleMouseLeave = () => setIsHovered(false);
    
    element.addEventListener('mouseenter', handleMouseEnter);
    element.addEventListener('mouseleave', handleMouseLeave);
    
    return () => {
      element.removeEventListener('mouseenter', handleMouseEnter);
      element.removeEventListener('mouseleave', handleMouseLeave);
    };
  }, []);
  
  return [ref, isHovered];
}

// Using with other refs
function HoverCard() {
  const [hoverRef, isHovered] = useHover<HTMLDivElement>();
  const clickRef = useClickOutside(() => console.log('clicked outside'));
  
  return (
    <div 
      ref={mergeRefs(hoverRef, clickRef)}
      style={{ 
        backgroundColor: isHovered ? '#f0f0f0' : 'white' 
      }}
    >
      <h3>Hover over me</h3>
      <p>Hovered: {isHovered ? 'Yes' : 'No'}</p>
    </div>
  );
}
```

### Handling Conditional Refs

```typescript
import { forwardRef } from 'react';
import { mergeRefs } from 'datocms-react-ui';

interface ButtonProps {
  trackClicks?: boolean;
  children: React.ReactNode;
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ trackClicks, children }, ref) => {
    const trackingRef = trackClicks 
      ? (el: HTMLButtonElement) => {
          if (el) {
            el.addEventListener('click', () => {
              console.log('Button clicked:', el.textContent);
            });
          }
        }
      : null;
    
    return (
      <button ref={mergeRefs(ref, trackingRef)}>
        {children}
      </button>
    );
  }
);
```

## TypeScript Support

The function is fully typed and supports generic type parameters:

```typescript
// Specific element type
const divRefs = mergeRefs<HTMLDivElement>(ref1, ref2);

// Custom component handle
interface MyHandle {
  focus: () => void;
}
const componentRefs = mergeRefs<MyHandle>(ref1, ref2);
```

## Common Patterns

### Pattern 1: ForwardRef + Internal Logic

```typescript
const Component = forwardRef((props, ref) => {
  const internalRef = useRef();
  
  // Use internal ref for component logic
  useEffect(() => {
    // Access DOM via internalRef.current
  }, []);
  
  // Merge refs to support both
  return <div ref={mergeRefs(ref, internalRef)} />;
});
```

### Pattern 2: Multiple Hooks

```typescript
function Component() {
  const ref1 = useHook1();
  const ref2 = useHook2();
  const ref3 = useHook3();
  
  return <div ref={mergeRefs(ref1, ref2, ref3)} />;
}
```

### Pattern 3: Safe Ref Merging

```typescript
// All of these are safely handled
const merged = mergeRefs(
  validRef,
  null,
  undefined,
  callbackRef,
  refObject
);
```

## Best Practices

1. **Order doesn't matter**: Refs are all updated with the same value, so order is not significant

2. **Handle cleanup**: When using callback refs, ensure proper cleanup:
   ```typescript
   const callbackRef = (el: HTMLElement | null) => {
     if (el) {
       // Setup
     } else {
       // Cleanup
     }
   };
   ```

3. **Type your refs**: Always specify the element type for better TypeScript support:
   ```typescript
   const ref1 = useRef<HTMLDivElement>(null);
   const ref2 = useRef<HTMLDivElement>(null);
   const merged = mergeRefs(ref1, ref2);
   ```

4. **Avoid recreation**: Memoize merged refs if used in dependency arrays:
   ```typescript
   const mergedRef = useMemo(
     () => mergeRefs(ref1, ref2),
     [ref1, ref2]
   );
   ```

## Implementation

The actual implementation is concise and efficient:

```typescript
export const mergeRefs =
  <T>(...refs: React.Ref<T>[]): React.RefCallback<T> =>
  (element: T) => {
    for (const ref of refs) {
      if (typeof ref === 'function') ref(element);
      else if (ref && typeof ref === 'object')
        (ref as React.MutableRefObject<T>).current = element;
    }
  };
```

## Related

- [useClickOutside](./useClickOutside.md) - Hook that returns a ref
- [React forwardRef](https://react.dev/reference/react/forwardRef) - React's ref forwarding