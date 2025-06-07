# Utility Functions

The DatoCMS Plugin SDK provides a collection of utility functions for common operations, type guards, and data manipulation. These utilities help ensure type safety and simplify plugin development.

## Import

```typescript
import { 
  pick, 
  omit, 
  isNullish,
  isBoolean,
  isString,
  isNumber,
  isRecord,
  isArray,
  isPlacement,
  MaybePromise,
  AwaitedReturnType,
  fromOneFieldIntoMultipleAndResultsById,
  containedRenderModeBootstrapper,
  fullScreenRenderModeBootstrapper
} from 'datocms-plugin-sdk';
```

## Object Utilities

### pick

Creates a new object with only the specified properties from the source object.

```typescript
export function pick<T extends object, K extends keyof T>(
  obj: T,
  keys: readonly K[],
): Pick<T, K> {
  const result = {} as Pick<T, K>;
  for (const key of keys) {
    if (key in obj) {
      result[key] = obj[key];
    }
  }
  return result;
}
```

**Example:**
```typescript
const user = { id: 1, name: 'John', email: 'john@example.com', age: 30 };
const publicInfo = pick(user, ['id', 'name']);
// Result: { id: 1, name: 'John' }
```

### omit

Creates a new object excluding the specified properties from the source object.

```typescript
export function omit<T extends object, K extends keyof T>(
  obj: T,
  keys: readonly K[],
): Omit<T, K> {
  const result = { ...obj } as T;
  for (const key of keys) {
    delete result[key];
  }
  return result as Omit<T, K>;
}
```

**Example:**
```typescript
const user = { id: 1, name: 'John', password: 'secret', email: 'john@example.com' };
const safeUser = omit(user, ['password']);
// Result: { id: 1, name: 'John', email: 'john@example.com' }
```

## Type Guards

Type guards are functions that perform runtime type checking and narrow TypeScript types.

### isNullish

Checks if a value is `null` or `undefined`.

```typescript
export function isNullish(value: unknown): value is null | undefined {
  return value === null || value === undefined;
}
```

**Example:**
```typescript
const value: string | null | undefined = getOptionalValue();
if (isNullish(value)) {
  console.log('Value is null or undefined');
} else {
  console.log(value.length); // TypeScript knows value is string
}
```

### isBoolean

Checks if a value is a boolean.

```typescript
export function isBoolean(value: unknown): value is boolean {
  return typeof value === 'boolean';
}
```

**Example:**
```typescript
if (isBoolean(value)) {
  console.log(value ? 'True' : 'False');
}
```

### isString

Checks if a value is a string.

```typescript
export function isString(value: unknown): value is string {
  return typeof value === 'string';
}
```

**Example:**
```typescript
if (isString(value)) {
  console.log(value.toUpperCase());
}
```

### isNumber

Checks if a value is a number.

```typescript
export function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}
```

**Example:**
```typescript
if (isNumber(value)) {
  console.log(value.toFixed(2));
}
```

### isRecord

Checks if a value is a plain object (not null, not an array).

```typescript
export function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}
```

**Example:**
```typescript
if (isRecord(value)) {
  // Safe to access object properties
  Object.keys(value).forEach(key => {
    console.log(`${key}: ${value[key]}`);
  });
}
```

### isArray

Checks if a value is an array and optionally validates all items.

```typescript
export function isArray<T>(
  value: unknown,
  checkItem: (item: unknown) => item is T,
): value is T[] {
  return Array.isArray(value) && value.every(checkItem);
}
```

**Example:**
```typescript
// Simple array check
if (isArray(value, isString)) {
  // TypeScript knows value is string[]
  console.log(value.join(', '));
}

// Custom item validation
function isUser(item: unknown): item is User {
  return isRecord(item) && isString(item.id) && isString(item.name);
}

if (isArray(value, isUser)) {
  // TypeScript knows value is User[]
  value.forEach(user => console.log(user.name));
}
```

### isPlacement

Checks if a value is a valid placement tuple for UI positioning.

```typescript
export function isPlacement(value: unknown) {
  return (
    isArray(value, isString) &&
    value.length === 2 &&
    ['before', 'after'].includes(value[0])
  );
}
```

**Example:**
```typescript
const placement = ['before', 'mainNavigationTab'];
if (isPlacement(placement)) {
  // Valid placement for inserting UI elements
  console.log(`Insert ${placement[0]} ${placement[1]}`);
}
```

## Type Utilities

### MaybePromise

A type that represents either a value or a Promise of that value.

```typescript
export type MaybePromise<T> = T | Promise<T>;
```

**Example:**
```typescript
// Function can return value synchronously or asynchronously
function getData(): MaybePromise<string> {
  if (cached) {
    return 'cached data'; // Synchronous
  }
  return fetchData(); // Returns Promise<string>
}
```

### AwaitedReturnType

Extracts the resolved return type of an async function.

```typescript
export type AwaitedReturnType<T extends (...args: any) => any> = Awaited<
  ReturnType<T>
>;
```

**Example:**
```typescript
async function fetchUser() {
  return { id: '1', name: 'John' };
}

type User = AwaitedReturnType<typeof fetchUser>;
// User is { id: string; name: string; }
```

## Internal Utilities

### fromOneFieldIntoMultipleAndResultsById

Internal utility used by the SDK to transform field-based hook functions into a format that processes multiple fields and returns results indexed by field ID.

```typescript
export function fromOneFieldIntoMultipleAndResultsById<Result>(
  fn:
    | ((
        field: Field,
        ctx: Ctx<any, any> & {
          itemType: ItemType;
        },
      ) => Result)
    | undefined,
) {
  return (
    fields: Field[],
    ctx: Ctx<any, any>,
  ): Record<string, Result> | undefined => {
    if (!fn) {
      return undefined;
    }

    const result: Record<string, Result> = {};

    for (const field of fields) {
      const itemType = ctx.itemTypes[
        field.relationships.item_type.data.id
      ] as ItemType;
      result[field.id] = fn(field, { ...ctx, itemType });
    }

    return result;
  };
}
```

This function is used internally by hooks like `overrideFieldExtensions` and `customMarksForStructuredTextField`.

## Bootstrapping Utilities

### containedRenderModeBootstrapper

Creates a bootstrapper for render modes that support self-resizing (field extensions, sidebars).

```typescript
export function containedRenderModeBootstrapper<
  Ctx extends SelfResizingPluginFrameCtx<any, {}, {}>
>(
  mode: ModeForPluginFrameCtx<Ctx>,
  callConfigurationMethod: (
    connectConfiguration: Partial<ExtractRenderHooks<FullConnectParameters>>,
    ctx: Ctx,
  ) => void,
): Bootstrapper<Ctx['mode']> {
  const bootstrapper: Bootstrapper<Ctx['mode']> = (
    connectConfiguration,
    methods,
    initialProperties,
  ) => {
    if (initialProperties.mode !== mode) {
      return undefined;
    }

    const sizingUtilities = buildSizingUtilities(methods);

    const render = (properties: Record<string, unknown>) => {
      callConfigurationMethod(connectConfiguration, {
        ...methods,
        ...properties,
        ...sizingUtilities,
      } as unknown as Ctx);
    };

    render(initialProperties);

    return render;
  };

  bootstrapper.mode = mode;

  return bootstrapper;
}
```

### fullScreenRenderModeBootstrapper

Creates a bootstrapper for render modes with imposed size (config screens, pages).

```typescript
export function fullScreenRenderModeBootstrapper<
  Ctx extends ImposedSizePluginFrameCtx<any, {}, {}>
>(
  mode: ModeForPluginFrameCtx<Ctx>,
  callConfigurationMethod: (
    connectConfiguration: Partial<ExtractRenderHooks<FullConnectParameters>>,
    ctx: Ctx,
  ) => void,
): Bootstrapper<Ctx['mode']> {
  const bootstrapper: Bootstrapper<Ctx['mode']> = (
    connectConfiguration,
    methods,
    initialProperties,
  ) => {
    if (initialProperties.mode !== mode) {
      return undefined;
    }

    const render = (properties: Record<string, unknown>) => {
      callConfigurationMethod(connectConfiguration, {
        ...methods,
        ...properties,
      } as unknown as Ctx);
    };

    render(initialProperties);

    return render;
  };

  bootstrapper.mode = mode;

  return bootstrapper;
}
```

## Usage Examples

### Validating Plugin Parameters

```typescript
import { isRecord, isString, isNumber, isArray } from 'datocms-plugin-sdk';

interface PluginParams {
  apiKey: string;
  maxItems: number;
  allowedTypes: string[];
}

function validateParams(params: unknown): params is PluginParams {
  if (!isRecord(params)) return false;
  
  return isString(params.apiKey) &&
         isNumber(params.maxItems) &&
         isArray(params.allowedTypes, isString);
}

// In your plugin
connect({
  renderConfigScreen(ctx) {
    const params = ctx.plugin.attributes.parameters;
    
    if (!validateParams(params)) {
      ctx.alert('Invalid plugin configuration');
      return;
    }
    
    // TypeScript knows params is PluginParams
    console.log(`API Key: ${params.apiKey}`);
  }
});
```

### Filtering Object Properties

```typescript
import { pick, omit } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(item, ctx) {
    // Only send specific fields to external API
    const publicData = pick(item, ['title', 'slug', 'publishedAt']);
    
    // Remove sensitive fields before logging
    const loggableData = omit(item, ['internalNotes', 'apiKeys']);
    console.log('Item data:', loggableData);
    
    return item;
  }
});
```

### Handling Optional Values

```typescript
import { isNullish } from 'datocms-plugin-sdk';

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    const currentValue = ctx.formValues[ctx.fieldPath];
    
    if (isNullish(currentValue)) {
      // Provide default value
      ctx.setFieldValue(ctx.fieldPath, getDefaultValue());
    } else {
      // Work with existing value
      processValue(currentValue);
    }
  }
});
```

### Type-Safe Array Processing

```typescript
import { isArray, isRecord, isString } from 'datocms-plugin-sdk';

interface Tag {
  id: string;
  label: string;
}

function isTag(value: unknown): value is Tag {
  return isRecord(value) && 
         isString(value.id) && 
         isString(value.label);
}

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    const tags = ctx.formValues.tags;
    
    if (isArray(tags, isTag)) {
      // TypeScript knows tags is Tag[]
      tags.forEach(tag => {
        console.log(`Tag: ${tag.label} (${tag.id})`);
      });
    }
  }
});
```

### Building Complex Validators

```typescript
interface Config {
  endpoint: string;
  timeout?: number;
  headers?: Record<string, string>;
  retryOptions?: {
    attempts: number;
    delay: number;
  };
}

function isConfig(value: unknown): value is Config {
  if (!isRecord(value)) return false;
  if (!isString(value.endpoint)) return false;
  
  // Optional fields
  if (value.timeout !== undefined && !isNumber(value.timeout)) return false;
  
  if (value.headers !== undefined) {
    if (!isRecord(value.headers)) return false;
    // Check all header values are strings
    if (!Object.values(value.headers).every(isString)) return false;
  }
  
  if (value.retryOptions !== undefined) {
    if (!isRecord(value.retryOptions)) return false;
    if (!isNumber(value.retryOptions.attempts)) return false;
    if (!isNumber(value.retryOptions.delay)) return false;
  }
  
  return true;
}
```

## Best Practices

1. **Use Type Guards**: Always validate unknown data with type guards before use
2. **Compose Guards**: Build complex type checks by combining simple guards
3. **Type Safety**: Leverage TypeScript's type narrowing with guards
4. **Validate External Data**: Always validate data from APIs, user input, or storage
5. **Consistent Validation**: Create reusable validation functions for common types
6. **Early Returns**: Validate early and return on invalid data
7. **Provide Defaults**: When validation fails, provide sensible defaults

## Type Safety Benefits

The utility functions provide several benefits:

1. **Runtime Safety**: Validate data at runtime to prevent errors
2. **Type Narrowing**: TypeScript understands the type after guard checks
3. **Reusability**: Common operations are standardized
4. **Clarity**: Code intent is clear with descriptive function names
5. **Maintenance**: Centralized utilities are easier to maintain
6. **IntelliSense**: Full autocomplete support for validated types

## Related Documentation

- [Plugin Lifecycle](./plugin-lifecycle.md) - Understanding bootstrapper usage
- [Context Object](./context-object.md) - Working with context data
- [Shared Types](./shared-types.md) - Common type definitions