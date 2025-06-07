# Spinner Component

A loading spinner component that indicates processing or loading states, matching DatoCMS's design system.

## Import

```jsx
import { Spinner } from 'datocms-react-ui';
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `size` | `'xs' \| 's' \| 'm' \| 'l' \| 'xl'` | `'m'` | Spinner size |
| `className` | `string` | - | Additional CSS classes |
| `style` | `CSSProperties` | - | Inline styles |

## Basic Usage

```jsx
import { Spinner, Canvas } from 'datocms-react-ui';

function MyPlugin({ ctx }) {
  const [loading, setLoading] = useState(true);

  return (
    <Canvas ctx={ctx}>
      {loading ? (
        <Spinner />
      ) : (
        <div>Content loaded!</div>
      )}
    </Canvas>
  );
}
```

## Sizes

### All Available Sizes

```jsx
function SpinnerSizes() {
  return (
    <div style={{ 
      display: 'flex', 
      alignItems: 'center', 
      gap: 'var(--spacing-l)' 
    }}>
      <Spinner size="xs" />
      <Spinner size="s" />
      <Spinner size="m" />
      <Spinner size="l" />
      <Spinner size="xl" />
    </div>
  );
}
```

## Common Patterns

### Loading Button

```jsx
function LoadingButton({ loading, onClick, children }) {
  return (
    <Button 
      onClick={onClick}
      disabled={loading}
      style={{ minWidth: '120px' }}
    >
      {loading ? (
        <Spinner size="xs" />
      ) : (
        children
      )}
    </Button>
  );
}
```

### Centered Loading State

```jsx
function CenteredSpinner() {
  return (
    <div style={{
      display: 'flex',
      justifyContent: 'center',
      alignItems: 'center',
      minHeight: '200px'
    }}>
      <Spinner size="l" />
    </div>
  );
}
```

### Loading Overlay

```jsx
function LoadingOverlay({ loading, children }) {
  return (
    <div style={{ position: 'relative' }}>
      {children}
      
      {loading && (
        <div style={{
          position: 'absolute',
          top: 0,
          left: 0,
          right: 0,
          bottom: 0,
          backgroundColor: 'rgba(255, 255, 255, 0.8)',
          display: 'flex',
          justifyContent: 'center',
          alignItems: 'center',
          zIndex: 10
        }}>
          <Spinner size="l" />
        </div>
      )}
    </div>
  );
}
```

### Inline Loading

```jsx
function InlineSpinner({ loading, text }) {
  return (
    <div style={{ 
      display: 'flex', 
      alignItems: 'center', 
      gap: 'var(--spacing-s)' 
    }}>
      {loading && <Spinner size="xs" />}
      <span>{text}</span>
    </div>
  );
}
```

## Advanced Examples

### Data Loading Component

```jsx
function DataLoader({ loadData, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    loadData()
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, []);

  if (loading) {
    return (
      <div style={{ padding: 'var(--spacing-xl)', textAlign: 'center' }}>
        <Spinner size="l" />
        <p style={{ marginTop: 'var(--spacing-m)', color: 'var(--muted-color)' }}>
          Loading data...
        </p>
      </div>
    );
  }

  if (error) {
    return (
      <div style={{ color: 'var(--alert-color)', padding: 'var(--spacing-m)' }}>
        Error: {error.message}
      </div>
    );
  }

  return render(data);
}
```

### Progressive Loading

```jsx
function ProgressiveLoader({ steps }) {
  const [currentStep, setCurrentStep] = useState(0);
  const [completed, setCompleted] = useState(false);

  useEffect(() => {
    const processSteps = async () => {
      for (let i = 0; i < steps.length; i++) {
        setCurrentStep(i);
        await steps[i].action();
      }
      setCompleted(true);
    };
    
    processSteps();
  }, []);

  if (completed) {
    return <div>All steps completed!</div>;
  }

  return (
    <div style={{ textAlign: 'center', padding: 'var(--spacing-l)' }}>
      <Spinner size="m" />
      <p style={{ marginTop: 'var(--spacing-m)' }}>
        {steps[currentStep]?.label || 'Processing...'}
      </p>
      <small style={{ color: 'var(--muted-color)' }}>
        Step {currentStep + 1} of {steps.length}
      </small>
    </div>
  );
}
```

### Form Field with Loading

```jsx
function AsyncValidatedField({ ctx }) {
  const [value, setValue] = useState('');
  const [checking, setChecking] = useState(false);
  const [isValid, setIsValid] = useState(null);

  const checkValue = async (val) => {
    if (!val) return;
    
    setChecking(true);
    try {
      const result = await validateAsync(val);
      setIsValid(result.valid);
    } finally {
      setChecking(false);
    }
  };

  return (
    <div>
      <TextField
        label="Username"
        value={value}
        onChange={setValue}
        onBlur={() => checkValue(value)}
        suffix={
          checking ? (
            <Spinner size="xs" />
          ) : isValid === true ? (
            <span style={{ color: 'var(--success-color)' }}>✓</span>
          ) : isValid === false ? (
            <span style={{ color: 'var(--alert-color)' }}>✗</span>
          ) : null
        }
      />
    </div>
  );
}
```

### Table with Loading States

```jsx
function DataTable({ columns, loadData }) {
  const [data, setData] = useState([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);

  const fetchData = async (showSpinner = true) => {
    if (showSpinner) setLoading(true);
    else setRefreshing(true);
    
    try {
      const result = await loadData();
      setData(result);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, []);

  return (
    <div>
      <div style={{ 
        display: 'flex', 
        justifyContent: 'space-between',
        marginBottom: 'var(--spacing-m)'
      }}>
        <h2>Data Table</h2>
        <Button 
          onClick={() => fetchData(false)}
          disabled={refreshing}
          size="s"
        >
          {refreshing ? <Spinner size="xs" /> : 'Refresh'}
        </Button>
      </div>
      
      {loading ? (
        <div style={{ padding: 'var(--spacing-xl)', textAlign: 'center' }}>
          <Spinner size="l" />
        </div>
      ) : (
        <table>
          {/* Table content */}
        </table>
      )}
    </div>
  );
}
```

### Skeleton Loading Alternative

```jsx
function ContentWithSpinner({ loading, skeleton = false }) {
  if (loading && skeleton) {
    return (
      <div>
        {[1, 2, 3].map(i => (
          <div 
            key={i}
            style={{
              height: '20px',
              backgroundColor: 'var(--light-color)',
              marginBottom: 'var(--spacing-s)',
              borderRadius: 'var(--border-radius-s)',
              animation: 'pulse 1.5s ease-in-out infinite'
            }}
          />
        ))}
      </div>
    );
  }

  if (loading) {
    return <Spinner />;
  }

  return <div>Content loaded</div>;
}
```

### Multiple Loading States

```jsx
function MultiStateComponent() {
  const [states, setStates] = useState({
    data: 'loading',
    permissions: 'loading',
    config: 'loading'
  });

  const allLoaded = Object.values(states).every(s => s === 'loaded');

  return (
    <div>
      {!allLoaded && (
        <div style={{ 
          display: 'grid', 
          gap: 'var(--spacing-m)',
          marginBottom: 'var(--spacing-l)'
        }}>
          {Object.entries(states).map(([key, status]) => (
            <div 
              key={key}
              style={{ 
                display: 'flex', 
                alignItems: 'center',
                gap: 'var(--spacing-s)'
              }}
            >
              {status === 'loading' ? (
                <Spinner size="xs" />
              ) : (
                <span style={{ color: 'var(--success-color)' }}>✓</span>
              )}
              <span style={{ textTransform: 'capitalize' }}>
                Loading {key}...
              </span>
            </div>
          ))}
        </div>
      )}
      
      {allLoaded && <div>All resources loaded!</div>}
    </div>
  );
}
```

## Styling

### Custom Colors

```jsx
<Spinner 
  style={{ 
    color: 'var(--primary-color)' 
  }}
  className="custom-spinner"
/>
```

### With Background

```jsx
function SpinnerWithBackground() {
  return (
    <div style={{
      display: 'inline-flex',
      alignItems: 'center',
      justifyContent: 'center',
      width: '60px',
      height: '60px',
      backgroundColor: 'var(--light-color)',
      borderRadius: 'var(--border-radius-m)'
    }}>
      <Spinner size="m" />
    </div>
  );
}
```

## Accessibility

- Uses appropriate ARIA attributes
- Announces loading state to screen readers
- Does not steal focus
- Provides context when used inline

## TypeScript

```typescript
import { Spinner } from 'datocms-react-ui';

type LoadingState = 'idle' | 'loading' | 'success' | 'error';

interface LoadingWrapperProps {
  state: LoadingState;
  error?: Error;
  children: React.ReactNode;
}

function LoadingWrapper({ state, error, children }: LoadingWrapperProps) {
  switch (state) {
    case 'idle':
      return null;
      
    case 'loading':
      return (
        <div style={{ textAlign: 'center', padding: '20px' }}>
          <Spinner size="l" />
        </div>
      );
      
    case 'error':
      return (
        <div style={{ color: 'var(--alert-color)' }}>
          Error: {error?.message || 'Unknown error'}
        </div>
      );
      
    case 'success':
      return <>{children}</>;
      
    default:
      return null;
  }
}
```

## Best Practices

1. **Appropriate sizing**: Use size that fits the context
2. **Loading feedback**: Always show loading state for async operations
3. **Centered placement**: Center spinners in their containers
4. **Text context**: Add loading text for clarity when needed
5. **Avoid layout shift**: Reserve space for content to prevent jumps
6. **Timeout handling**: Consider showing error after extended loading

## Related Components

- [Button](./Button.md) - Often contains spinners
- [TextField](./TextField.md) - Can show spinner in suffix
- [Canvas](./Canvas.md) - Root container