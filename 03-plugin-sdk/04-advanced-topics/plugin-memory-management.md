# Plugin Memory Management

## Overview

DatoCMS plugins run in iframes with potentially long lifespans, making memory management crucial for performance. This guide covers best practices for preventing memory leaks, managing resources efficiently, and ensuring optimal plugin performance.

## Common Memory Issues in Plugins

### 1. Event Listener Leaks

Event listeners that aren't properly cleaned up are the most common source of memory leaks in plugins.

```typescript
// ❌ BAD: Event listener not cleaned up
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    window.addEventListener('resize', handleResize);
    
    function handleResize() {
      console.log('Window resized');
    }
  }
});

// ✅ GOOD: Proper cleanup
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    function handleResize() {
      console.log('Window resized');
    }
    
    window.addEventListener('resize', handleResize);
    
    // Cleanup on unmount
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }
});
```

### 2. Timer and Interval Leaks

Timers that continue running after the component unmounts waste resources.

```typescript
// ❌ BAD: Timer not cleared
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    setInterval(() => {
      fetchData();
    }, 5000);
  }
});

// ✅ GOOD: Proper timer management
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    const intervalId = setInterval(() => {
      fetchData();
    }, 5000);
    
    return () => {
      clearInterval(intervalId);
    };
  }
});
```

### 3. Observer Leaks

ResizeObserver, MutationObserver, and IntersectionObserver instances must be disconnected.

```typescript
// ✅ GOOD: Proper observer cleanup
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    const resizeObserver = new ResizeObserver(entries => {
      // Handle resize
    });
    
    const element = document.querySelector('.my-element');
    if (element) {
      resizeObserver.observe(element);
    }
    
    return () => {
      resizeObserver.disconnect();
    };
  }
});
```

## React-Specific Memory Management

### 1. useEffect Cleanup

Always return cleanup functions from useEffect hooks.

```typescript
import React, { useEffect, useState } from 'react';
import { connect } from 'datocms-plugin-sdk';

function FieldExtension({ ctx }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchData() {
      const result = await fetch('/api/data');
      if (!cancelled) {
        setData(await result.json());
      }
    }
    
    fetchData();
    
    // Cleanup function
    return () => {
      cancelled = true;
    };
  }, []);
  
  return <div>{/* Component content */}</div>;
}

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    render(<FieldExtension ctx={ctx} />);
  }
});
```

### 2. Event Handler References

Avoid creating new function instances on every render.

```typescript
// ❌ BAD: New function on every render
function MyComponent() {
  return (
    <button onClick={() => console.log('clicked')}>
      Click me
    </button>
  );
}

// ✅ GOOD: Stable function reference
function MyComponent() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

### 3. Large Object References

Be careful with storing large objects in state or closures.

```typescript
// ❌ BAD: Storing large objects unnecessarily
function MyComponent({ ctx }) {
  const [allItems, setAllItems] = useState([]);
  
  useEffect(() => {
    // This could load thousands of items
    loadAllItems().then(setAllItems);
  }, []);
}

// ✅ GOOD: Paginate or virtualize large lists
function MyComponent({ ctx }) {
  const [page, setPage] = useState(1);
  const [items, setItems] = useState([]);
  
  useEffect(() => {
    // Load only what's needed
    loadItemsPage(page, 20).then(setItems);
  }, [page]);
}
```

## Managing Plugin State

### 1. State Cleanup

Clean up state when components unmount to prevent memory leaks.

```typescript
import { connect } from 'datocms-plugin-sdk';

class StateManager {
  private listeners = new Set<Function>();
  private state = {};
  
  subscribe(listener: Function) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }
  
  cleanup() {
    this.listeners.clear();
    this.state = {};
  }
}

const stateManager = new StateManager();

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Subscribe to state changes
    const unsubscribe = stateManager.subscribe(handleStateChange);
    
    return () => {
      unsubscribe();
      // Clean up if this is the last instance
      if (stateManager.listeners.size === 0) {
        stateManager.cleanup();
      }
    };
  }
});
```

### 2. Caching Strategies

Implement proper cache invalidation to prevent unbounded memory growth.

```typescript
class CacheManager {
  private cache = new Map();
  private maxSize = 100;
  private maxAge = 5 * 60 * 1000; // 5 minutes
  
  set(key: string, value: any) {
    // Remove oldest entries if cache is full
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, {
      value,
      timestamp: Date.now()
    });
  }
  
  get(key: string) {
    const entry = this.cache.get(key);
    if (!entry) return null;
    
    // Check if entry is expired
    if (Date.now() - entry.timestamp > this.maxAge) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.value;
  }
  
  clear() {
    this.cache.clear();
  }
}
```

## Resource Management Best Practices

### 1. Lazy Loading

Load resources only when needed.

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Lazy load heavy dependencies
    const loadEditor = async () => {
      const { Editor } = await import('./heavy-editor');
      return Editor;
    };
    
    render(
      <React.Suspense fallback={<div>Loading...</div>}>
        <LazyEditor loadEditor={loadEditor} />
      </React.Suspense>
    );
  }
});
```

### 2. Resource Pooling

Reuse expensive resources instead of creating new ones.

```typescript
class WebSocketPool {
  private connections = new Map<string, WebSocket>();
  
  getConnection(url: string): WebSocket {
    if (!this.connections.has(url)) {
      const ws = new WebSocket(url);
      this.connections.set(url, ws);
      
      ws.addEventListener('close', () => {
        this.connections.delete(url);
      });
    }
    
    return this.connections.get(url)!;
  }
  
  closeAll() {
    this.connections.forEach(ws => ws.close());
    this.connections.clear();
  }
}
```

### 3. Debouncing and Throttling

Reduce memory pressure from frequent operations.

```typescript
import { debounce, throttle } from 'lodash-es';

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Debounce expensive operations
    const debouncedSave = debounce(async (value) => {
      await ctx.setFieldValue(ctx.fieldPath, value);
    }, 500);
    
    // Throttle frequent updates
    const throttledUpdate = throttle((value) => {
      updatePreview(value);
    }, 100);
    
    return () => {
      debouncedSave.cancel();
      throttledUpdate.cancel();
    };
  }
});
```

## Monitoring Memory Usage

### 1. Performance Observer

Monitor your plugin's memory usage in development.

```typescript
function monitorMemory() {
  if ('memory' in performance) {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        console.log('Memory usage:', {
          usedJSHeapSize: (performance as any).memory.usedJSHeapSize,
          totalJSHeapSize: (performance as any).memory.totalJSHeapSize,
          jsHeapSizeLimit: (performance as any).memory.jsHeapSizeLimit
        });
      }
    });
    
    observer.observe({ entryTypes: ['measure'] });
  }
}
```

### 2. Memory Profiling

Use Chrome DevTools for detailed memory analysis:

1. Open Chrome DevTools
2. Go to Memory tab
3. Take heap snapshots before and after operations
4. Compare snapshots to find leaks

### 3. Automated Testing

Create tests to detect memory leaks.

```typescript
// memory.test.ts
describe('Memory Management', () => {
  it('should not leak memory on mount/unmount cycles', async () => {
    const initialHeap = (performance as any).memory.usedJSHeapSize;
    
    // Mount and unmount component multiple times
    for (let i = 0; i < 10; i++) {
      const container = document.createElement('div');
      document.body.appendChild(container);
      
      const { unmount } = render(<MyComponent />, container);
      
      await new Promise(resolve => setTimeout(resolve, 100));
      
      unmount();
      document.body.removeChild(container);
    }
    
    // Force garbage collection if available
    if (global.gc) {
      global.gc();
    }
    
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    const finalHeap = (performance as any).memory.usedJSHeapSize;
    const heapGrowth = finalHeap - initialHeap;
    
    // Allow for some growth but flag significant leaks
    expect(heapGrowth).toBeLessThan(1000000); // 1MB threshold
  });
});
```

## Common Patterns and Solutions

### 1. Auto-Resizer Memory Management

When using the SDK's auto-resizer, ensure proper cleanup:

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Start auto-resizer
    ctx.startAutoResizer();
    
    // Important: Stop on cleanup
    return () => {
      ctx.stopAutoResizer();
    };
  }
});
```

### 2. External API Connections

Properly manage external connections:

```typescript
class ApiClient {
  private abortControllers = new Set<AbortController>();
  
  async fetch(url: string, options?: RequestInit) {
    const controller = new AbortController();
    this.abortControllers.add(controller);
    
    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal
      });
      return response;
    } finally {
      this.abortControllers.delete(controller);
    }
  }
  
  cancelAll() {
    this.abortControllers.forEach(controller => controller.abort());
    this.abortControllers.clear();
  }
}
```

### 3. DOM References

Clean up DOM references to prevent detached nodes:

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    let elementRef: HTMLElement | null = null;
    
    const handleMount = (element: HTMLElement) => {
      elementRef = element;
      // Set up element
    };
    
    const handleUnmount = () => {
      if (elementRef) {
        // Clean up any data or event listeners
        elementRef = null;
      }
    };
    
    render(<Component onMount={handleMount} onUnmount={handleUnmount} />);
  }
});
```

## Best Practices Summary

1. **Always clean up**: Remove event listeners, clear timers, disconnect observers
2. **Use weak references**: Consider WeakMap/WeakSet for object associations
3. **Limit cache sizes**: Implement max size and TTL for caches
4. **Profile regularly**: Use DevTools to monitor memory usage
5. **Test for leaks**: Include memory leak tests in your test suite
6. **Lazy load**: Load heavy dependencies only when needed
7. **Batch operations**: Reduce memory churn with debouncing/throttling
8. **Monitor production**: Log memory metrics in production builds

## Debugging Memory Issues

### Chrome DevTools Workflow

1. **Baseline snapshot**: Take initial heap snapshot
2. **Perform actions**: Use your plugin normally
3. **Force GC**: Click garbage can icon
4. **Compare snapshot**: Take another snapshot
5. **Analyze growth**: Look for unexpected retained objects

### Common Leak Indicators

- Continuously growing memory usage
- Detached DOM nodes in heap snapshots
- Event listener count increasing
- Unclosed connections or streams
- Large arrays or objects that never shrink

## Related Documentation

- [Plugin Performance](./plugin-performance.md)
- [Plugin Testing](./plugin-testing.md)
- [Context Object](../01-core-concepts/context-object.md)