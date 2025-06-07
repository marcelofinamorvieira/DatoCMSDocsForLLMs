# useMediaQuery and useElementLayout

React hooks for responsive design and element dimension tracking in DatoCMS plugins.

## Import

```jsx
import { useMediaQuery, MediaQuery, useElementLayout } from 'datocms-react-ui';
```

## Hooks

### useMediaQuery

A React hook that tracks the state of a CSS media query and triggers re-renders when the query result changes.

#### Signature

```typescript
function useMediaQuery(media: string): MediaQueryList
```

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `media` | `string` | A valid CSS media query string |

#### Returns

Returns a `MediaQueryList` object with these properties:

| Property | Type | Description |
|----------|------|-------------|
| `matches` | `boolean` | Whether the media query currently matches |
| `media` | `string` | The media query string |
| `addEventListener` | `function` | Add a change listener |
| `removeEventListener` | `function` | Remove a change listener |

#### Example

```jsx
import { useMediaQuery } from 'datocms-react-ui';

function ResponsiveComponent() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1025px)');
  const prefersDark = useMediaQuery('(prefers-color-scheme: dark)');
  
  if (isMobile.matches) {
    return <MobileLayout />;
  }
  
  if (isTablet.matches) {
    return <TabletLayout />;
  }
  
  return <DesktopLayout darkMode={prefersDark.matches} />;
}
```

### useElementLayout

A React hook that tracks the dimensions of a DOM element using ResizeObserver.

#### Signature

```typescript
function useElementLayout(
  ref: React.MutableRefObject<Element> | React.RefObject<Element>
): DOMRect
```

#### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `ref` | `React.MutableRefObject<Element> \| React.RefObject<Element>` | A ref to the element to observe |

#### Returns

Returns a `DOMRect` object with these properties:

| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | X coordinate (always 0 for this hook) |
| `y` | `number` | Y coordinate (always 0 for this hook) |
| `width` | `number` | Element width in pixels |
| `height` | `number` | Element height in pixels |
| `top` | `number` | Top position (always 0 for this hook) |
| `right` | `number` | Right position (width) |
| `bottom` | `number` | Bottom position (height) |
| `left` | `number` | Left position (always 0 for this hook) |

#### Example

```jsx
import { useElementLayout } from 'datocms-react-ui';
import { useRef } from 'react';

function DimensionAwareComponent() {
  const elementRef = useRef(null);
  const rect = useElementLayout(elementRef);
  
  return (
    <div ref={elementRef}>
      <p>Width: {rect.width}px</p>
      <p>Height: {rect.height}px</p>
      <p>Aspect Ratio: {(rect.width / rect.height).toFixed(2)}</p>
    </div>
  );
}
```

## Components

### MediaQuery

A render prop component that provides media query state to its children.

#### Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `media` | `string` | Yes | A valid CSS media query string |
| `children` | `(mql: MediaQueryList) => JSX.Element` | Yes | Render function receiving the MediaQueryList |

#### Type Definition

```typescript
export type MediaQueryProps = {
  media: string;
  children: (mql: MediaQueryList) => JSX.Element;
};
```

#### Example

```jsx
import { MediaQuery } from 'datocms-react-ui';

function ResponsiveApp() {
  return (
    <MediaQuery media="(max-width: 768px)">
      {(mql) => (
        <div>
          {mql.matches ? (
            <MobileMenu />
          ) : (
            <DesktopMenu />
          )}
        </div>
      )}
    </MediaQuery>
  );
}
```

## Implementation Details

### useMediaQuery Implementation

The hook efficiently manages media query listeners by:
1. **Sharing MediaQueryList instances**: Multiple components using the same query share the same MediaQueryList object
2. **Pooling listeners**: All components listening to the same query are notified through a shared listener
3. **Automatic cleanup**: Listeners are removed when no components are using the query
4. **SSR support**: Returns a fallback on the server with `matches: true`

### useElementLayout Implementation

The hook uses advanced ResizeObserver features:
1. **Border-box detection**: Automatically uses border-box measurements when supported by the browser
2. **Shared ResizeObserver**: A single ResizeObserver instance is shared across all components
3. **Efficient updates**: Only updates state when dimensions actually change
4. **DOM property storage**: Uses a custom DOM property to associate update handlers with elements

## Common Use Cases

### Responsive Layouts

```jsx
function ResponsiveLayout({ ctx }) {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isLandscape = useMediaQuery('(orientation: landscape)');
  
  return (
    <Canvas ctx={ctx}>
      <div style={{
        display: isMobile.matches ? 'block' : 'flex',
        flexDirection: isLandscape.matches ? 'row' : 'column'
      }}>
        <Sidebar collapsed={isMobile.matches} />
        <MainContent fullWidth={isMobile.matches} />
      </div>
    </Canvas>
  );
}
```

### Adaptive Components

```jsx
function AdaptiveTable({ data }) {
  const isSmallScreen = useMediaQuery('(max-width: 640px)');
  
  if (isSmallScreen.matches) {
    // Render as cards on small screens
    return (
      <div className="card-list">
        {data.map(item => (
          <Card key={item.id} {...item} />
        ))}
      </div>
    );
  }
  
  // Render as table on larger screens
  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Value</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        {data.map(item => (
          <tr key={item.id}>
            <td>{item.name}</td>
            <td>{item.value}</td>
            <td><Button>Edit</Button></td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Dynamic Sizing

```jsx
function DynamicChart() {
  const containerRef = useRef(null);
  const { width, height } = useElementLayout(containerRef);
  
  return (
    <div ref={containerRef} style={{ width: '100%', height: '400px' }}>
      {width > 0 && height > 0 && (
        <Chart 
          width={width} 
          height={height}
          margin={{ top: 20, right: 20, bottom: 20, left: 20 }}
        />
      )}
    </div>
  );
}
```

### Conditional Features

```jsx
function FeatureToggle({ ctx }) {
  const hasSpace = useMediaQuery('(min-width: 1200px)');
  const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)');
  
  return (
    <div>
      {hasSpace.matches && (
        <AdvancedToolbar />
      )}
      
      <AnimatedComponent 
        animate={!prefersReducedMotion.matches}
      />
    </div>
  );
}
```

### Container Queries Polyfill

```jsx
function ContainerQuery({ children }) {
  const containerRef = useRef(null);
  const { width } = useElementLayout(containerRef);
  
  return (
    <div ref={containerRef} data-container-width={width}>
      <div style={{
        fontSize: width < 400 ? '14px' : '16px',
        padding: width < 600 ? '10px' : '20px'
      }}>
        {children}
      </div>
    </div>
  );
}
```

## Advanced Examples

### Breakpoint System

```jsx
const breakpoints = {
  mobile: '(max-width: 767px)',
  tablet: '(min-width: 768px) and (max-width: 1023px)',
  desktop: '(min-width: 1024px)',
  wide: '(min-width: 1440px)'
};

function useBreakpoint() {
  const mobile = useMediaQuery(breakpoints.mobile);
  const tablet = useMediaQuery(breakpoints.tablet);
  const desktop = useMediaQuery(breakpoints.desktop);
  const wide = useMediaQuery(breakpoints.wide);
  
  if (mobile.matches) return 'mobile';
  if (tablet.matches) return 'tablet';
  if (desktop.matches) return wide.matches ? 'wide' : 'desktop';
  return 'unknown';
}

function ResponsiveGrid() {
  const breakpoint = useBreakpoint();
  
  const columns = {
    mobile: 1,
    tablet: 2,
    desktop: 3,
    wide: 4
  }[breakpoint];
  
  return (
    <div style={{
      display: 'grid',
      gridTemplateColumns: `repeat(${columns}, 1fr)`,
      gap: '20px'
    }}>
      {/* Grid items */}
    </div>
  );
}
```

### Aspect Ratio Maintenance

```jsx
function AspectRatioBox({ ratio = 16/9, children }) {
  const containerRef = useRef(null);
  const { width } = useElementLayout(containerRef);
  
  return (
    <div ref={containerRef} style={{ width: '100%' }}>
      <div style={{
        width: '100%',
        height: `${width / ratio}px`,
        position: 'relative'
      }}>
        {children}
      </div>
    </div>
  );
}
```

### Performance Optimization

```jsx
function OptimizedMediaQuery() {
  // Share media query results across components
  const queries = {
    mobile: useMediaQuery('(max-width: 768px)'),
    tablet: useMediaQuery('(min-width: 769px) and (max-width: 1024px)'),
    desktop: useMediaQuery('(min-width: 1025px)')
  };
  
  // Memoize expensive calculations based on media queries
  const layout = useMemo(() => {
    if (queries.mobile.matches) return 'single-column';
    if (queries.tablet.matches) return 'two-column';
    return 'three-column';
  }, [queries.mobile.matches, queries.tablet.matches]);
  
  return <Layout type={layout} />;
}
```

### Responsive Canvas with Retina Support

```jsx
function ResponsiveCanvas() {
  const containerRef = useRef(null);
  const canvasRef = useRef(null);
  const { width, height } = useElementLayout(containerRef);
  const isRetina = useMediaQuery('(min-resolution: 2dppx)');
  
  useEffect(() => {
    if (!canvasRef.current || width === 0 || height === 0) return;
    
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    const scale = isRetina.matches ? 2 : 1;
    
    // Set actual size in memory
    canvas.width = width * scale;
    canvas.height = height * scale;
    
    // Scale down using CSS
    canvas.style.width = width + 'px';
    canvas.style.height = height + 'px';
    
    // Scale context to match device pixel ratio
    ctx.scale(scale, scale);
    
    // Draw your canvas content here
    drawContent(ctx, width, height);
  }, [width, height, isRetina.matches]);
  
  return (
    <div ref={containerRef} style={{ width: '100%', height: '400px' }}>
      <canvas ref={canvasRef} />
    </div>
  );
}
```

## Performance Considerations

### useMediaQuery Performance

1. **Shared MediaQueryList**: The hook shares MediaQueryList instances across components using the same query string
2. **Efficient Updates**: Only components using a specific query re-render when that query changes
3. **Automatic Cleanup**: Removes event listeners when no components are using a query
4. **Minimal Re-renders**: Uses state updates to trigger re-renders only when matches change

### useElementLayout Performance

1. **ResizeObserver Pooling**: Uses a single ResizeObserver instance for all components
2. **Border-box Support**: Automatically uses more efficient border-box measurements when available
3. **Change Detection**: Only updates state when dimensions actually change
4. **DOM Property Method**: Uses a custom DOM property (`__reactLayoutHandler`) for efficient handler storage

## Browser Support

- **useMediaQuery**: Requires `window.matchMedia` support (all modern browsers)
- **useElementLayout**: Requires `ResizeObserver` support (modern browsers, may need polyfill)
- **Server-Side Rendering**: Both hooks handle SSR gracefully with fallback values

## TypeScript

```typescript
import { 
  useMediaQuery, 
  MediaQuery, 
  MediaQueryProps,
  useElementLayout 
} from 'datocms-react-ui';

// useMediaQuery
const mql: MediaQueryList = useMediaQuery('(min-width: 768px)');
const isWide: boolean = mql.matches;

// MediaQuery component
const props: MediaQueryProps = {
  media: '(max-width: 768px)',
  children: (mql) => <div>{mql.matches ? 'Mobile' : 'Desktop'}</div>
};

// useElementLayout
const ref = useRef<HTMLDivElement>(null);
const rect: DOMRect = useElementLayout(ref);
const width: number = rect.width;
```

## Best Practices

1. **Use Standard Queries**: Stick to common breakpoints for consistency
2. **Mobile-First**: Design for mobile by default, enhance for larger screens
3. **Combine Queries**: Use multiple queries for complex responsive logic
4. **Performance**: Avoid expensive operations in components that re-render on query changes
5. **Accessibility**: Consider `prefers-reduced-motion` and `prefers-color-scheme`
6. **Fallbacks**: Always provide sensible defaults for SSR and initial render
7. **Cleanup**: The hooks handle cleanup automatically, no manual cleanup needed

## Common Media Queries

```jsx
// Screen sizes
const isMobile = useMediaQuery('(max-width: 768px)');
const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
const isDesktop = useMediaQuery('(min-width: 1025px)');

// Device capabilities
const hasTouch = useMediaQuery('(pointer: coarse)');
const hasHover = useMediaQuery('(hover: hover)');
const isHighDPI = useMediaQuery('(min-resolution: 2dppx)');

// User preferences
const prefersDark = useMediaQuery('(prefers-color-scheme: dark)');
const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)');
const prefersHighContrast = useMediaQuery('(prefers-contrast: high)');

// Orientation
const isPortrait = useMediaQuery('(orientation: portrait)');
const isLandscape = useMediaQuery('(orientation: landscape)');

// Print
const isPrint = useMediaQuery('print');
const isScreen = useMediaQuery('screen');
```

## Related Components

- [Canvas](./Canvas.md) - Root container that may affect media queries
- [SplitView](./SplitView.md) - Component that can benefit from responsive behavior
- [Section](./Section.md) - Collapsible sections based on screen size
- [useClickOutside](./useClickOutside.md) - Another utility hook for UI interactions