# Plugin State Management

## Overview

State management is crucial for building complex DatoCMS plugins. Since plugins run in iframes and may have multiple instances across different hooks, managing state effectively requires careful planning. This guide covers patterns, best practices, and solutions for plugin state management.

## State Management Challenges

### 1. Iframe Isolation

Each plugin instance runs in its own iframe, creating natural isolation:

```typescript
// This won't work across different hook instances
let sharedState = { count: 0 }; // ❌ Not shared between iframes

connect({
  renderFieldExtension(id, ctx) {
    sharedState.count++; // Only affects this instance
  },
  
  renderConfigScreen(ctx) {
    console.log(sharedState.count); // Always 0, different iframe
  }
});
```

### 2. Multiple Instances

The same hook can be rendered multiple times:

```typescript
// Multiple field extensions on the same page
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Each field gets its own instance
    // Need to manage state per field
  }
});
```

## State Management Solutions

### 1. Local Component State

For simple, isolated state needs, use React's built-in state management.

```typescript
import React, { useState, useReducer } from 'react';
import { connect } from 'datocms-plugin-sdk';

// Simple state with useState
function SimpleFieldExtension({ ctx }) {
  const [value, setValue] = useState(ctx.formValues[ctx.fieldPath] || '');
  const [loading, setLoading] = useState(false);
  
  const handleChange = async (newValue) => {
    setLoading(true);
    try {
      await ctx.setFieldValue(ctx.fieldPath, newValue);
      setValue(newValue);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <input
      value={value}
      onChange={(e) => handleChange(e.target.value)}
      disabled={loading}
    />
  );
}

// Complex state with useReducer
interface State {
  value: string;
  suggestions: string[];
  loading: boolean;
  error: string | null;
}

type Action =
  | { type: 'SET_VALUE'; payload: string }
  | { type: 'SET_SUGGESTIONS'; payload: string[] }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_VALUE':
      return { ...state, value: action.payload, error: null };
    case 'SET_SUGGESTIONS':
      return { ...state, suggestions: action.payload };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_ERROR':
      return { ...state, error: action.payload, loading: false };
    default:
      return state;
  }
}

function ComplexFieldExtension({ ctx }) {
  const [state, dispatch] = useReducer(reducer, {
    value: ctx.formValues[ctx.fieldPath] || '',
    suggestions: [],
    loading: false,
    error: null
  });
  
  // Component logic using state and dispatch
}
```

### 2. Context API for Component Trees

Share state within a single plugin instance.

```typescript
import React, { createContext, useContext, useState } from 'react';

// Create contexts
interface PluginState {
  config: any;
  cache: Map<string, any>;
  preferences: any;
}

const PluginStateContext = createContext<PluginState | null>(null);
const PluginDispatchContext = createContext<any>(null);

// Provider component
function PluginStateProvider({ children, ctx }) {
  const [state, setState] = useState<PluginState>({
    config: ctx.plugin.attributes.parameters,
    cache: new Map(),
    preferences: {}
  });
  
  const actions = {
    updateConfig: (config: any) => {
      setState(prev => ({ ...prev, config }));
    },
    setCache: (key: string, value: any) => {
      setState(prev => ({
        ...prev,
        cache: new Map(prev.cache).set(key, value)
      }));
    }
  };
  
  return (
    <PluginStateContext.Provider value={state}>
      <PluginDispatchContext.Provider value={actions}>
        {children}
      </PluginDispatchContext.Provider>
    </PluginStateContext.Provider>
  );
}

// Custom hooks
function usePluginState() {
  const context = useContext(PluginStateContext);
  if (!context) {
    throw new Error('usePluginState must be used within PluginStateProvider');
  }
  return context;
}

function usePluginDispatch() {
  const context = useContext(PluginDispatchContext);
  if (!context) {
    throw new Error('usePluginDispatch must be used within PluginStateProvider');
  }
  return context;
}

// Usage in components
function MyComponent() {
  const state = usePluginState();
  const { updateConfig } = usePluginDispatch();
  
  return (
    <div>
      <pre>{JSON.stringify(state.config, null, 2)}</pre>
      <button onClick={() => updateConfig({ theme: 'dark' })}>
        Update Config
      </button>
    </div>
  );
}

// Connect with provider
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    render(
      <PluginStateProvider ctx={ctx}>
        <MyComponent />
      </PluginStateProvider>
    );
  }
});
```

### 3. Plugin Parameters for Persistent State

Store configuration in plugin parameters for persistence.

```typescript
import { connect } from 'datocms-plugin-sdk';

interface PluginConfig {
  apiKey: string;
  endpoint: string;
  theme: 'light' | 'dark';
  features: {
    autoSave: boolean;
    spellCheck: boolean;
  };
}

// Configuration manager
class ConfigManager {
  private ctx: any;
  private config: PluginConfig;
  
  constructor(ctx: any) {
    this.ctx = ctx;
    this.config = this.loadConfig();
  }
  
  private loadConfig(): PluginConfig {
    const params = this.ctx.plugin.attributes.parameters as Partial<PluginConfig>;
    return {
      apiKey: params.apiKey || '',
      endpoint: params.endpoint || 'https://api.example.com',
      theme: params.theme || 'light',
      features: {
        autoSave: params.features?.autoSave ?? true,
        spellCheck: params.features?.spellCheck ?? true
      }
    };
  }
  
  async updateConfig(updates: Partial<PluginConfig>) {
    this.config = { ...this.config, ...updates };
    await this.ctx.updatePluginParameters(this.config);
  }
  
  get<K extends keyof PluginConfig>(key: K): PluginConfig[K] {
    return this.config[key];
  }
}

// Usage
connect({
  renderConfigScreen(ctx) {
    const configManager = new ConfigManager(ctx);
    
    function ConfigScreen() {
      const [config, setConfig] = useState(configManager.config);
      
      const handleSave = async () => {
        await configManager.updateConfig(config);
        ctx.notice('Configuration saved!');
      };
      
      return (
        <div>
          <input
            value={config.apiKey}
            onChange={(e) => setConfig({ ...config, apiKey: e.target.value })}
          />
          <button onClick={handleSave}>Save</button>
        </div>
      );
    }
    
    render(<ConfigScreen />);
  }
});
```

### 4. Browser Storage for User Preferences

Use localStorage/sessionStorage for user-specific preferences.

```typescript
// Storage utility
class StorageManager {
  private prefix: string;
  
  constructor(pluginId: string) {
    this.prefix = `datocms-plugin-${pluginId}`;
  }
  
  private getKey(key: string, userId?: string): string {
    const parts = [this.prefix, key];
    if (userId) parts.push(userId);
    return parts.join('-');
  }
  
  get<T>(key: string, defaultValue: T, userId?: string): T {
    try {
      const stored = localStorage.getItem(this.getKey(key, userId));
      return stored ? JSON.parse(stored) : defaultValue;
    } catch {
      return defaultValue;
    }
  }
  
  set(key: string, value: any, userId?: string): void {
    try {
      localStorage.setItem(
        this.getKey(key, userId),
        JSON.stringify(value)
      );
    } catch (e) {
      console.warn('Failed to save to localStorage:', e);
    }
  }
  
  remove(key: string, userId?: string): void {
    localStorage.removeItem(this.getKey(key, userId));
  }
  
  clear(userId?: string): void {
    const prefix = userId ? `${this.prefix}-*-${userId}` : `${this.prefix}-`;
    Object.keys(localStorage)
      .filter(key => key.startsWith(prefix))
      .forEach(key => localStorage.removeItem(key));
  }
}

// React hook for storage
function useLocalStorage<T>(
  key: string,
  defaultValue: T,
  ctx: any
): [T, (value: T) => void] {
  const storage = useMemo(
    () => new StorageManager(ctx.plugin.id),
    [ctx.plugin.id]
  );
  
  const userId = ctx.currentUser.id;
  
  const [value, setValue] = useState<T>(() =>
    storage.get(key, defaultValue, userId)
  );
  
  const setStoredValue = useCallback((newValue: T) => {
    setValue(newValue);
    storage.set(key, newValue, userId);
  }, [key, userId]);
  
  return [value, setStoredValue];
}

// Usage
function MyComponent({ ctx }) {
  const [preferences, setPreferences] = useLocalStorage('preferences', {
    viewMode: 'grid',
    sortBy: 'name',
    showAdvanced: false
  }, ctx);
  
  return (
    <div>
      <button onClick={() => setPreferences({
        ...preferences,
        viewMode: preferences.viewMode === 'grid' ? 'list' : 'grid'
      })}>
        Toggle View Mode
      </button>
    </div>
  );
}
```

### 5. State Synchronization Across Instances

For sharing state between different plugin instances, use external coordination.

```typescript
// Message-based state sync
class StateSyncManager {
  private channel: BroadcastChannel;
  private state: any = {};
  private listeners = new Set<Function>();
  
  constructor(pluginId: string) {
    this.channel = new BroadcastChannel(`plugin-${pluginId}`);
    this.channel.onmessage = (event) => {
      if (event.data.type === 'STATE_UPDATE') {
        this.state = { ...this.state, ...event.data.payload };
        this.notifyListeners();
      }
    };
  }
  
  updateState(updates: any) {
    this.state = { ...this.state, ...updates };
    this.channel.postMessage({
      type: 'STATE_UPDATE',
      payload: updates
    });
    this.notifyListeners();
  }
  
  subscribe(listener: Function) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  }
  
  private notifyListeners() {
    this.listeners.forEach(listener => listener(this.state));
  }
  
  getState() {
    return this.state;
  }
  
  dispose() {
    this.channel.close();
    this.listeners.clear();
  }
}

// React hook for synced state
function useSyncedState(key: string, initialValue: any, ctx: any) {
  const [manager] = useState(() => new StateSyncManager(ctx.plugin.id));
  const [value, setValue] = useState(initialValue);
  
  useEffect(() => {
    const unsubscribe = manager.subscribe((state) => {
      if (key in state) {
        setValue(state[key]);
      }
    });
    
    return () => {
      unsubscribe();
    };
  }, [key]);
  
  const updateValue = useCallback((newValue: any) => {
    manager.updateState({ [key]: newValue });
  }, [key]);
  
  useEffect(() => {
    return () => {
      manager.dispose();
    };
  }, []);
  
  return [value, updateValue];
}
```

### 6. External State Management with IndexedDB

For complex data that needs persistence.

```typescript
// IndexedDB wrapper
class PluginDatabase {
  private db: IDBDatabase | null = null;
  private dbName: string;
  
  constructor(pluginId: string) {
    this.dbName = `datocms-plugin-${pluginId}`;
  }
  
  async init() {
    return new Promise<void>((resolve, reject) => {
      const request = indexedDB.open(this.dbName, 1);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };
      
      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        
        // Create stores
        if (!db.objectStoreNames.contains('state')) {
          db.createObjectStore('state', { keyPath: 'id' });
        }
        
        if (!db.objectStoreNames.contains('cache')) {
          const cacheStore = db.createObjectStore('cache', { keyPath: 'key' });
          cacheStore.createIndex('expiry', 'expiry');
        }
      };
    });
  }
  
  async get(store: string, key: string) {
    if (!this.db) await this.init();
    
    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([store], 'readonly');
      const objectStore = transaction.objectStore(store);
      const request = objectStore.get(key);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async set(store: string, key: string, value: any) {
    if (!this.db) await this.init();
    
    return new Promise<void>((resolve, reject) => {
      const transaction = this.db!.transaction([store], 'readwrite');
      const objectStore = transaction.objectStore(store);
      const request = objectStore.put({ ...value, id: key });
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  async delete(store: string, key: string) {
    if (!this.db) await this.init();
    
    return new Promise<void>((resolve, reject) => {
      const transaction = this.db!.transaction([store], 'readwrite');
      const objectStore = transaction.objectStore(store);
      const request = objectStore.delete(key);
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  async clearExpiredCache() {
    if (!this.db) await this.init();
    
    const transaction = this.db!.transaction(['cache'], 'readwrite');
    const store = transaction.objectStore('cache');
    const index = store.index('expiry');
    const range = IDBKeyRange.upperBound(Date.now());
    
    const request = index.openCursor(range);
    
    request.onsuccess = (event) => {
      const cursor = (event.target as IDBRequest).result;
      if (cursor) {
        cursor.delete();
        cursor.continue();
      }
    };
  }
}

// React hook for IndexedDB
function usePluginDatabase(ctx: any) {
  const [db] = useState(() => new PluginDatabase(ctx.plugin.id));
  const [ready, setReady] = useState(false);
  
  useEffect(() => {
    db.init().then(() => setReady(true));
  }, []);
  
  const methods = useMemo(() => ({
    async saveState(key: string, value: any) {
      await db.set('state', key, value);
    },
    
    async loadState(key: string, defaultValue: any = null) {
      const result = await db.get('state', key);
      return result || defaultValue;
    },
    
    async cacheData(key: string, data: any, ttl = 3600000) {
      await db.set('cache', key, {
        key,
        data,
        expiry: Date.now() + ttl
      });
    },
    
    async getCachedData(key: string) {
      const result = await db.get('cache', key);
      if (result && result.expiry > Date.now()) {
        return result.data;
      }
      return null;
    }
  }), [db]);
  
  return { ready, ...methods };
}
```

## State Management Patterns

### 1. Form State Pattern

Managing form state with validation and dirty tracking.

```typescript
interface FormState<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  touched: Partial<Record<keyof T, boolean>>;
  isDirty: boolean;
  isSubmitting: boolean;
}

class FormManager<T extends Record<string, any>> {
  private state: FormState<T>;
  private initialValues: T;
  private validators: Partial<Record<keyof T, (value: any) => string | null>>;
  private subscribers = new Set<(state: FormState<T>) => void>();
  
  constructor(
    initialValues: T,
    validators: Partial<Record<keyof T, (value: any) => string | null>> = {}
  ) {
    this.initialValues = initialValues;
    this.validators = validators;
    this.state = {
      values: initialValues,
      errors: {},
      touched: {},
      isDirty: false,
      isSubmitting: false
    };
  }
  
  setValue<K extends keyof T>(field: K, value: T[K]) {
    this.state.values[field] = value;
    this.state.touched[field] = true;
    this.state.isDirty = true;
    
    // Validate field
    const validator = this.validators[field];
    if (validator) {
      const error = validator(value);
      if (error) {
        this.state.errors[field] = error;
      } else {
        delete this.state.errors[field];
      }
    }
    
    this.notify();
  }
  
  async submit(onSubmit: (values: T) => Promise<void>) {
    // Validate all fields
    for (const [field, validator] of Object.entries(this.validators)) {
      if (validator) {
        const error = (validator as any)(this.state.values[field]);
        if (error) {
          this.state.errors[field as keyof T] = error;
        }
      }
    }
    
    if (Object.keys(this.state.errors).length > 0) {
      this.notify();
      return;
    }
    
    this.state.isSubmitting = true;
    this.notify();
    
    try {
      await onSubmit(this.state.values);
      this.state.isDirty = false;
    } finally {
      this.state.isSubmitting = false;
      this.notify();
    }
  }
  
  reset() {
    this.state = {
      values: this.initialValues,
      errors: {},
      touched: {},
      isDirty: false,
      isSubmitting: false
    };
    this.notify();
  }
  
  subscribe(callback: (state: FormState<T>) => void) {
    this.subscribers.add(callback);
    return () => this.subscribers.delete(callback);
  }
  
  private notify() {
    this.subscribers.forEach(cb => cb(this.state));
  }
  
  getState() {
    return this.state;
  }
}

// React hook
function useForm<T extends Record<string, any>>(
  initialValues: T,
  validators?: Partial<Record<keyof T, (value: any) => string | null>>
) {
  const [manager] = useState(() => new FormManager(initialValues, validators));
  const [state, setState] = useState(manager.getState());
  
  useEffect(() => {
    return manager.subscribe(setState);
  }, []);
  
  return {
    values: state.values,
    errors: state.errors,
    touched: state.touched,
    isDirty: state.isDirty,
    isSubmitting: state.isSubmitting,
    setValue: (field: keyof T, value: any) => manager.setValue(field, value),
    submit: (onSubmit: (values: T) => Promise<void>) => manager.submit(onSubmit),
    reset: () => manager.reset()
  };
}
```

### 2. Async State Pattern

Managing async operations with loading and error states.

```typescript
interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useAsyncState<T>(
  asyncFunction: () => Promise<T>,
  dependencies: any[] = []
) {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: true,
    error: null
  });
  
  const execute = useCallback(async () => {
    setState({ data: null, loading: true, error: null });
    
    try {
      const data = await asyncFunction();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
    }
  }, dependencies);
  
  useEffect(() => {
    execute();
  }, [execute]);
  
  return { ...state, refetch: execute };
}

// Usage
function MyComponent({ ctx }) {
  const { data, loading, error, refetch } = useAsyncState(
    async () => {
      const response = await fetch(`/api/items/${ctx.itemId}`);
      return response.json();
    },
    [ctx.itemId]
  );
  
  if (loading) return <Spinner />;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <pre>{JSON.stringify(data, null, 2)}</pre>
      <button onClick={refetch}>Refresh</button>
    </div>
  );
}
```

## Best Practices

### 1. State Initialization

```typescript
// Initialize state based on context
function useInitialState(ctx: any) {
  return useMemo(() => {
    return {
      // From form values
      currentValue: ctx.formValues[ctx.fieldPath] || '',
      
      // From plugin parameters
      config: ctx.plugin.attributes.parameters || {},
      
      // From localStorage
      preferences: StorageManager.get('preferences', {
        theme: 'light',
        autoSave: true
      }),
      
      // Computed initial state
      isNewItem: !ctx.item || ctx.itemStatus === 'new',
      canEdit: !ctx.disabled && ctx.currentRole.can_edit_schema
    };
  }, []);
}
```

### 2. State Cleanup

```typescript
// Cleanup on unmount
useEffect(() => {
  return () => {
    // Cancel async operations
    abortController.abort();
    
    // Clear timers
    clearTimeout(saveTimeout);
    
    // Disconnect observers
    resizeObserver.disconnect();
    
    // Save state if needed
    saveStateToStorage(state);
    
    // Clean up event listeners
    window.removeEventListener('beforeunload', handleBeforeUnload);
  };
}, []);
```

### 3. State Persistence Strategy

```typescript
// Decide what to persist where
const statePersistence = {
  // Plugin parameters: Global config
  pluginParams: ['apiKey', 'endpoint', 'defaultSettings'],
  
  // localStorage: User preferences
  localStorage: ['viewMode', 'sortOrder', 'collapsedSections'],
  
  // sessionStorage: Temporary state
  sessionStorage: ['searchQuery', 'filters', 'currentPage'],
  
  // IndexedDB: Large data
  indexedDB: ['cachedResults', 'draftData', 'history'],
  
  // Don't persist: Runtime state
  memory: ['loading', 'errors', 'connectionStatus']
};
```

## Performance Considerations

### 1. Memoization

```typescript
// Memoize expensive computations
const processedData = useMemo(() => {
  return heavyDataProcessing(rawData);
}, [rawData]);

// Memoize callbacks
const handleClick = useCallback((id: string) => {
  doSomething(id);
}, [dependency]);
```

### 2. State Updates Batching

```typescript
// Batch multiple state updates
import { unstable_batchedUpdates } from 'react-dom';

function handleMultipleUpdates() {
  unstable_batchedUpdates(() => {
    setValue1(newValue1);
    setValue2(newValue2);
    setValue3(newValue3);
  });
}
```

### 3. Lazy State Initialization

```typescript
// Initialize state lazily
const [expensiveState, setExpensiveState] = useState(() => {
  return computeExpensiveInitialState();
});
```

## Debugging State

### 1. State Logger

```typescript
function useStateLogger(name: string, state: any) {
  useEffect(() => {
    console.group(`State Update: ${name}`);
    console.log('New State:', state);
    console.log('Timestamp:', new Date().toISOString());
    console.trace('Update Stack');
    console.groupEnd();
  }, [state]);
}
```

### 2. Redux DevTools Integration

```typescript
// Integrate with Redux DevTools for debugging
function useDevTools(name: string, state: any, actions: any) {
  const devTools = useMemo(() => {
    if (typeof window !== 'undefined' && (window as any).__REDUX_DEVTOOLS_EXTENSION__) {
      return (window as any).__REDUX_DEVTOOLS_EXTENSION__.connect({
        name: `DatoCMS Plugin: ${name}`
      });
    }
    return null;
  }, [name]);
  
  useEffect(() => {
    if (devTools) {
      devTools.send('STATE_UPDATE', state);
    }
  }, [state]);
  
  return devTools;
}
```

## Common Pitfalls

1. **Stale closures**: Always include dependencies in hooks
2. **Memory leaks**: Clean up subscriptions and timers
3. **Race conditions**: Handle async operations carefully
4. **Over-rendering**: Use memoization appropriately
5. **State synchronization**: Keep local and remote state in sync

## Related Documentation

- [Multi-Hook Patterns](./multi-hook-patterns.md)
- [Plugin Performance](./plugin-performance.md)
- [Plugin Memory Management](./plugin-memory-management.md)