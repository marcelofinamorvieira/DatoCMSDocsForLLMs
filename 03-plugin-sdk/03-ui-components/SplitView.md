# SplitView Component

A resizable split view component that allows dividing the interface into multiple panes with draggable dividers, perfect for creating flexible layouts in DatoCMS plugins.

## Import

```jsx
import { SplitView, SplitViewPane, SplitViewSash } from 'datocms-react-ui';
```

## Components

### SplitView

The main container component that manages split panes.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `JSX.Element[]` | Required | Array of child elements to split |
| `sizes` | `(string \| number)[]` | Required | Sizes for each pane (pixels, percentages, or 'auto') |
| `onChange` | `(sizes: number[]) => void` | Required | Callback when pane sizes change |
| `split` | `'vertical' \| 'horizontal'` | `'vertical'` | Direction of the split |
| `allowResize` | `boolean` | `true` | Whether panes can be resized |
| `resizerSize` | `number` | `10` | Size of the resize handle in pixels |
| `performanceMode` | `boolean` | `false` | Optimize rendering during resize |
| `onDragStart` | `() => void` | - | Callback when resize starts |
| `onDragEnd` | `() => void` | - | Callback when resize ends |
| `sashAction` | `SashAction` | - | Custom action for sash (resize handle) |

### SplitViewPane

Optional wrapper component for pane content with size constraints.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `children` | `ReactNode` | Required | Pane content |
| `minSize` | `number \| string` | - | Minimum size (pixels or percentage) |
| `maxSize` | `number \| string` | - | Maximum size (pixels or percentage) |
| `style` | `CSSProperties` | - | Custom styles for the pane |

### SplitViewSash

The resize handle component that appears between panes. This component is used internally by SplitView but is also exported for advanced use cases.

#### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `className` | `string` | - | Additional CSS class name |
| `style` | `CSSProperties` | Required | Position and size styles |
| `split` | `'vertical' \| 'horizontal'` | Required | Direction of the split |
| `allowResize` | `boolean` | - | Whether resizing is allowed |
| `action` | `SashAction` | - | Custom action button in the sash |
| `onMouseDown` | `(x: number, y: number) => void` | Required | Handle drag start |
| `onMouseMove` | `(x: number, y: number) => void` | Required | Handle drag move |
| `onMouseUp` | `() => void` | Required | Handle drag end |

## Basic Usage

```jsx
import { SplitView, Canvas } from 'datocms-react-ui';
import { useState } from 'react';

function MyPlugin({ ctx }) {
  const [sizes, setSizes] = useState([200, 'auto']);
  
  return (
    <Canvas ctx={ctx}>
      <SplitView sizes={sizes} onChange={setSizes}>
        <div style={{ padding: '20px' }}>Left Panel</div>
        <div style={{ padding: '20px' }}>Right Panel</div>
      </SplitView>
    </Canvas>
  );
}
```

## Examples

### Vertical Split (Default)

```jsx
function VerticalSplitExample({ ctx }) {
  const [sizes, setSizes] = useState([200, 300]);
  
  return (
    <SplitView
      sizes={sizes}
      onChange={setSizes}
    >
      <div style={{ padding: '20px' }}>First Panel</div>
      <div style={{ padding: '20px' }}>Second Panel</div>
    </SplitView>
  );
}
```

### Horizontal Split

```jsx
function HorizontalSplitExample({ ctx }) {
  const [sizes, setSizes] = useState([150, 250]);
  
  return (
    <SplitView
      split="horizontal"
      sizes={sizes}
      onChange={setSizes}
    >
      <div>Top Panel</div>
      <div>Bottom Panel</div>
    </SplitView>
  );
}
```

### Three-Way Split

```jsx
function ThreeWaySplitExample({ ctx }) {
  const [sizes, setSizes] = useState([200, 'auto', 200]);
  
  return (
    <SplitView
      sizes={sizes}
      onChange={setSizes}
    >
      <div>Left</div>
      <div>Center (auto)</div>
      <div>Right</div>
    </SplitView>
  );
}
```

### With Size Constraints

```jsx
function ConstrainedSplitExample({ ctx }) {
  const [sizes, setSizes] = useState([250, 'auto']);
  
  return (
    <SplitView
      sizes={sizes}
      onChange={setSizes}
    >
      <SplitViewPane minSize={100} maxSize={500}>
        <div>Constrained Panel (100-500px)</div>
      </SplitViewPane>
      <div>Flexible Panel</div>
    </SplitView>
  );
}
```

### Non-Resizable

```jsx
function FixedSplitExample({ ctx }) {
  const [sizes] = useState([300, 300]);
  
  return (
    <SplitView
      sizes={sizes}
      onChange={() => {}} // Required but won't be called
      allowResize={false}
    >
      <div>Fixed Panel 1</div>
      <div>Fixed Panel 2</div>
    </SplitView>
  );
}
```

### Percentage-Based Sizes

```jsx
function PercentageSplitExample({ ctx }) {
  const [sizes, setSizes] = useState(['30%', '70%']);
  
  return (
    <SplitView
      sizes={sizes}
      onChange={setSizes}
    >
      <div>30% Width</div>
      <div>70% Width</div>
    </SplitView>
  );
}
```

### Performance Mode

```jsx
function PerformanceSplitExample({ ctx }) {
  const [sizes, setSizes] = useState([300, 'auto']);
  
  return (
    <SplitView
      sizes={sizes}
      onChange={setSizes}
      performanceMode={true} // Optimizes rendering during drag
    >
      <div>Complex Content 1</div>
      <div>Complex Content 2</div>
    </SplitView>
  );
}
```

## Advanced Usage

### With Sash Action

```jsx
import { SplitView, Canvas } from 'datocms-react-ui';
import { FaSync } from 'react-icons/fa';

function SplitViewWithSashAction({ ctx }) {
  const [sizes, setSizes] = useState([300, 'auto']);

  const handleReset = () => {
    setSizes([300, 'auto']);
    ctx.notice('Layout reset!');
  };

  return (
    <Canvas ctx={ctx}>
      <SplitView
        sizes={sizes}
        onChange={setSizes}
        sashAction={{
          icon: <FaSync />,
          onClick: handleReset,
        }}
      >
        <div style={{ padding: '20px' }}>
          <h3>Left Panel</h3>
          <p>Drag the divider or click the icon to reset</p>
        </div>
        <div style={{ padding: '20px' }}>
          <h3>Right Panel</h3>
          <p>The sash action provides quick layout control</p>
        </div>
      </SplitView>
    </Canvas>
  );
}
```

### Nested Split Views

```jsx
function NestedSplitView({ ctx }) {
  const [outerSizes, setOuterSizes] = useState([300, 'auto']);
  const [innerSizes, setInnerSizes] = useState([200, 'auto']);
  
  return (
    <SplitView sizes={outerSizes} onChange={setOuterSizes}>
      <div>Sidebar</div>
      <SplitView 
        split="horizontal" 
        sizes={innerSizes} 
        onChange={setInnerSizes}
      >
        <div>Top Content</div>
        <div>Bottom Content</div>
      </SplitView>
    </SplitView>
  );
}
```

### Controlled Sizes

```jsx
function ControlledSplitView({ ctx }) {
  const [sizes, setSizes] = useState([250, 350]);
  
  const resetSizes = () => {
    setSizes([300, 300]);
  };
  
  return (
    <>
      <Button onClick={resetSizes}>Reset Sizes</Button>
      <SplitView
        sizes={sizes}
        onChange={setSizes}
      >
        <div>Panel 1: {Math.round(sizes[0])}px</div>
        <div>Panel 2: {Math.round(sizes[1])}px</div>
      </SplitView>
    </>
  );
}
```

### With Drag Callbacks

```jsx
function DragCallbackExample({ ctx }) {
  const [sizes, setSizes] = useState([300, 'auto']);
  const [isDragging, setIsDragging] = useState(false);
  
  return (
    <SplitView
      sizes={sizes}
      onChange={setSizes}
      onDragStart={() => {
        setIsDragging(true);
        console.log('Started dragging');
      }}
      onDragEnd={() => {
        setIsDragging(false);
        console.log('Finished dragging');
      }}
    >
      <div>Panel 1 {isDragging && '(resizing...)'}</div>
      <div>Panel 2</div>
    </SplitView>
  );
}
```

### Mixed Size Units

```jsx
function MixedSizeExample({ ctx }) {
  // Mix pixels, percentages, and auto
  const [sizes, setSizes] = useState([200, '30%', 'auto']);
  
  return (
    <SplitView sizes={sizes} onChange={setSizes}>
      <div>Fixed 200px</div>
      <div>30% of container</div>
      <div>Remaining space</div>
    </SplitView>
  );
}
```

### IDE-Style Layout

```jsx
function IDELayout({ ctx }) {
  const [mainSizes, setMainSizes] = useState([250, 'auto']);
  const [editorSizes, setEditorSizes] = useState(['auto', 200]);
  
  return (
    <div style={{ height: '100vh' }}>
      <SplitView sizes={mainSizes} onChange={setMainSizes}>
        {/* File Explorer */}
        <SplitViewPane minSize={200} maxSize={400}>
          <div style={{ height: '100%', overflow: 'auto' }}>
            <h3>Explorer</h3>
            {/* File tree */}
          </div>
        </SplitViewPane>
        
        {/* Editor Area */}
        <SplitView 
          split="horizontal" 
          sizes={editorSizes} 
          onChange={setEditorSizes}
        >
          {/* Code Editor */}
          <div style={{ height: '100%', overflow: 'auto' }}>
            <h3>Editor</h3>
            {/* Editor content */}
          </div>
          
          {/* Terminal */}
          <SplitViewPane minSize={100}>
            <div style={{ 
              height: '100%',
              backgroundColor: '#1e1e1e',
              color: '#fff',
              fontFamily: 'monospace',
              padding: '10px'
            }}>
              <h3>Terminal</h3>
            </div>
          </SplitViewPane>
        </SplitView>
      </SplitView>
    </div>
  );
}
```

## Size Units

The SplitView component supports flexible size specifications:

- **Pixels**: `300` or `'300px'` - Fixed pixel size
- **Percentages**: `'30%'` - Percentage of container
- **Auto**: `'auto'` - Divide remaining space equally among auto panes

## How It Works

1. **Size Calculation**: The component calculates absolute pixel sizes for each pane based on the provided sizes and container dimensions
2. **Constraints**: When using SplitViewPane, minSize and maxSize constraints are enforced during resize
3. **Resize Handle**: A draggable sash appears between panes when `allowResize` is true
4. **Performance Mode**: When enabled, panes don't update during drag for better performance

## Styling

### CSS Classes

The component applies these classes:
- `.SplitView` - Main container
- `.SplitView--vertical` - Applied for vertical splits
- `.SplitView--horizontal` - Applied for horizontal splits  
- `.SplitView--dragging` - Applied during resize
- `.SplitViewPane` - Individual pane wrapper

### Custom Styling

```jsx
<SplitView
  sizes={[300, 'auto']}
  onChange={setSizes}
  style={{ height: '400px' }}
>
  <SplitViewPane style={{ background: '#f0f0f0' }}>
    <div>Styled Pane 1</div>
  </SplitViewPane>
  <div style={{ padding: '20px' }}>Pane 2</div>
</SplitView>
```

## Type Definitions

```typescript
type SplitViewProps = {
  children: JSX.Element[];
  allowResize?: boolean;
  split?: 'vertical' | 'horizontal';
  sizes: (string | number)[];
  onChange: (sizes: number[]) => void;
  onDragStart?: () => void;
  onDragEnd?: () => void;
  performanceMode?: boolean;
  resizerSize?: number;
  sashAction?: SashAction;
};

export type SplitViewPaneProps = {
  maxSize?: number | string;
  minSize?: number | string;
  style?: React.CSSProperties;
  children?: ReactNode;
};

export type SashAction = {
  icon: ReactNode;
  onClick: () => void;
};

type SplitViewSashProps = {
  className?: string;
  style: React.CSSProperties;
  split: 'vertical' | 'horizontal';
  allowResize?: boolean;
  action?: SashAction;
  onMouseDown: (x: number, y: number) => void;
  onMouseMove: (x: number, y: number) => void;
  onMouseUp: () => void;
};
```

## Best Practices

1. **Always provide onChange**: The component requires an onChange handler even if sizes are fixed
2. **Use SplitViewPane for constraints**: Wrap panes in SplitViewPane when you need size limits
3. **Performance mode for complex content**: Enable performanceMode when panes contain heavy renders
4. **Auto for flexible layouts**: Use 'auto' for panes that should share remaining space
5. **State management**: Store sizes in state to maintain layout across re-renders
6. **Mobile considerations**: Consider different layouts for small screens

## Common Patterns

### Sidebar Layout
```jsx
const [sizes, setSizes] = useState([250, 'auto']);

<SplitView sizes={sizes} onChange={setSizes}>
  <SplitViewPane minSize={200} maxSize={400}>
    <Sidebar />
  </SplitViewPane>
  <MainContent />
</SplitView>
```

### Equal Split
```jsx
const [sizes, setSizes] = useState(['auto', 'auto', 'auto']);

<SplitView sizes={sizes} onChange={setSizes}>
  <Panel1 />
  <Panel2 />
  <Panel3 />
</SplitView>
```

## Related Components

- [VerticalSplit](./VerticalSplit.md) - Simpler two-pane split component
- [Canvas](./Canvas.md) - Root container for plugins