# Performance Optimization Guide for DatoCMS

This guide provides strategies and best practices for optimizing performance when working with DatoCMS APIs and plugins.

## API Request Optimization

### Batch Operations

Use batch operations to reduce API calls:

```typescript
// ❌ Bad - Multiple sequential requests
for (const itemData of items) {
  await client.items.create(itemData);
}

// ✅ Good - Single batch request
const operations = items.map(itemData => ({
  type: 'item',
  attributes: itemData
}));

await client.items.bulkCreate({
  data: operations
});
```

### Field Selection

Request only needed fields to reduce payload size:

```typescript
// ❌ Bad - Fetches all fields
const items = await client.items.list();

// ✅ Good - Selective field fetching
const items = await client.items.list({
  filter: {
    type: 'article'
  },
  fields: {
    article: 'title,slug,excerpt'
  }
});
```

### Pagination Strategies

Implement efficient pagination:

```typescript
async function* fetchAllItems(client: CMAClient, itemTypeId: string) {
  let page = 1;
  const perPage = 100; // Maximum allowed
  
  while (true) {
    const items = await client.items.list({
      filter: { type: itemTypeId },
      page: { offset: (page - 1) * perPage, limit: perPage }
    });
    
    yield* items;
    
    if (items.length < perPage) break;
    page++;
  }
}

// Usage with streaming
for await (const item of fetchAllItems(client, 'article')) {
  processItem(item);
}
```

### Parallel Requests

Execute independent requests in parallel:

```typescript
// ❌ Bad - Sequential requests
const itemTypes = await client.itemTypes.list();
const uploads = await client.uploads.list();
const users = await client.users.list();

// ✅ Good - Parallel requests
const [itemTypes, uploads, users] = await Promise.all([
  client.itemTypes.list(),
  client.uploads.list(),
  client.users.list()
]);
```

## Caching Strategies

### In-Memory Caching

Implement simple in-memory caching:

```typescript
class CachedClient {
  private cache = new Map<string, { data: any; expires: number }>();
  
  constructor(private client: CMAClient, private ttl = 300000) {} // 5 minutes
  
  async getItemType(id: string) {
    const cacheKey = `itemType:${id}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached && cached.expires > Date.now()) {
      return cached.data;
    }
    
    const data = await this.client.itemTypes.find(id);
    this.cache.set(cacheKey, {
      data,
      expires: Date.now() + this.ttl
    });
    
    return data;
  }
}
```

### Redis Caching

For distributed caching:

```typescript
import Redis from 'ioredis';

class RedisCache {
  private redis: Redis;
  
  constructor() {
    this.redis = new Redis(process.env.REDIS_URL);
  }
  
  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(key);
    return data ? JSON.parse(data) : null;
  }
  
  async set<T>(key: string, value: T, ttl = 300): Promise<void> {
    await this.redis.setex(key, ttl, JSON.stringify(value));
  }
  
  async invalidate(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length) {
      await this.redis.del(...keys);
    }
  }
}

// Usage
const cache = new RedisCache();

async function getCachedItems(itemType: string) {
  const cacheKey = `items:${itemType}`;
  let items = await cache.get<Item[]>(cacheKey);
  
  if (!items) {
    items = await client.items.list({ filter: { type: itemType } });
    await cache.set(cacheKey, items, 600); // 10 minutes
  }
  
  return items;
}
```

### Webhook-Based Cache Invalidation

Automatically invalidate cache on content changes:

```typescript
app.post('/webhook/invalidate', async (req, res) => {
  const { entity_type, event_type, entity } = req.body;
  
  // Invalidate relevant cache entries
  if (entity_type === 'item') {
    await cache.invalidate(`items:${entity.item_type}`);
    await cache.invalidate(`item:${entity.id}`);
  }
  
  if (entity_type === 'item_type') {
    await cache.invalidate(`itemType:${entity.id}`);
    await cache.invalidate('itemTypes:*');
  }
  
  res.status(200).send('OK');
});
```

## Plugin Performance

### Lazy Loading

Load plugin components on demand:

```typescript
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    return (
      <Canvas ctx={ctx}>
        <Suspense fallback={<Spinner />}>
          <HeavyComponent />
        </Suspense>
      </Canvas>
    );
  }
});
```

### Debouncing Field Updates

Prevent excessive API calls:

```typescript
import { debounce } from 'lodash';

function FieldEditor({ ctx }) {
  const [localValue, setLocalValue] = useState(ctx.formValues[ctx.fieldPath]);
  
  const debouncedUpdate = useMemo(
    () => debounce((value: string) => {
      ctx.setFieldValue(ctx.fieldPath, value);
    }, 500),
    [ctx.fieldPath]
  );
  
  const handleChange = (value: string) => {
    setLocalValue(value);
    debouncedUpdate(value);
  };
  
  return (
    <TextInput
      value={localValue}
      onChange={handleChange}
    />
  );
}
```

### Virtual Scrolling

Handle large lists efficiently:

```typescript
import { FixedSizeList } from 'react-window';

function LargeItemList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].title}
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

### Memoization

Optimize expensive computations:

```typescript
import { useMemo, memo } from 'react';

const ExpensiveComponent = memo(({ items, filter }) => {
  const filteredItems = useMemo(() => {
    return items.filter(item => 
      item.title.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  const stats = useMemo(() => {
    return {
      total: filteredItems.length,
      published: filteredItems.filter(i => i.published).length,
      draft: filteredItems.filter(i => !i.published).length
    };
  }, [filteredItems]);
  
  return (
    <div>
      <div>Total: {stats.total}</div>
      <div>Published: {stats.published}</div>
      <div>Draft: {stats.draft}</div>
    </div>
  );
});
```

## Image Optimization

### Responsive Images

Use DatoCMS image transformations:

```typescript
function ResponsiveImage({ upload }) {
  const srcSet = [
    `${upload.url}?w=320 320w`,
    `${upload.url}?w=640 640w`,
    `${upload.url}?w=960 960w`,
    `${upload.url}?w=1280 1280w`,
  ].join(', ');
  
  return (
    <img
      src={`${upload.url}?w=640`}
      srcSet={srcSet}
      sizes="(max-width: 320px) 280px, (max-width: 640px) 640px, 960px"
      loading="lazy"
      alt={upload.alt}
    />
  );
}
```

### Progressive Loading

Implement blur-up technique:

```typescript
function ProgressiveImage({ upload }) {
  const [imageLoaded, setImageLoaded] = useState(false);
  
  return (
    <div style={{ position: 'relative' }}>
      {/* Low quality placeholder */}
      <img
        src={`${upload.url}?w=20&blur=10`}
        style={{
          filter: imageLoaded ? 'none' : 'blur(10px)',
          transition: 'filter 0.3s'
        }}
        alt=""
      />
      
      {/* High quality image */}
      <img
        src={`${upload.url}?w=800`}
        onLoad={() => setImageLoaded(true)}
        style={{
          position: 'absolute',
          top: 0,
          left: 0,
          opacity: imageLoaded ? 1 : 0,
          transition: 'opacity 0.3s'
        }}
        alt={upload.alt}
      />
    </div>
  );
}
```

## Query Optimization

### Efficient Filtering

Use indexed fields for filtering:

```typescript
// ❌ Bad - Filtering on non-indexed field
const items = await client.items.list({
  filter: {
    fields: {
      article: {
        custom_field: { eq: 'value' }
      }
    }
  }
});

// ✅ Good - Filter on indexed fields
const items = await client.items.list({
  filter: {
    type: 'article',
    query: 'search term' // Full-text search
  }
});
```

### Relationship Preloading

Include related data in single request:

```typescript
// ❌ Bad - N+1 query problem
const articles = await client.items.list({ filter: { type: 'article' } });
for (const article of articles) {
  article.author = await client.items.find(article.author.id);
}

// ✅ Good - Include relationships
const articles = await client.items.list({
  filter: { type: 'article' },
  nested: true // Automatically resolves relationships
});
```

## Bundle Size Optimization

### Tree Shaking

Import only what you need:

```typescript
// ❌ Bad - Imports entire library
import * as datocmsReactUi from 'datocms-react-ui';

// ✅ Good - Selective imports
import { Canvas, Button, TextField } from 'datocms-react-ui';
```

### Code Splitting

Split plugin code by feature:

```typescript
const routes = {
  '/settings': () => import('./pages/Settings'),
  '/editor': () => import('./pages/Editor'),
  '/preview': () => import('./pages/Preview')
};

function Router({ path }) {
  const [Component, setComponent] = useState(null);
  
  useEffect(() => {
    routes[path]?.().then(module => {
      setComponent(() => module.default);
    });
  }, [path]);
  
  return Component ? <Component /> : <Spinner />;
}
```

## Monitoring and Profiling

### Performance Metrics

Track key metrics:

```typescript
class PerformanceMonitor {
  private metrics: Map<string, number[]> = new Map();
  
  async measure<T>(name: string, fn: () => Promise<T>): Promise<T> {
    const start = performance.now();
    
    try {
      const result = await fn();
      const duration = performance.now() - start;
      
      if (!this.metrics.has(name)) {
        this.metrics.set(name, []);
      }
      this.metrics.get(name)!.push(duration);
      
      return result;
    } catch (error) {
      const duration = performance.now() - start;
      console.error(`${name} failed after ${duration}ms`, error);
      throw error;
    }
  }
  
  getStats(name: string) {
    const times = this.metrics.get(name) || [];
    if (times.length === 0) return null;
    
    return {
      count: times.length,
      min: Math.min(...times),
      max: Math.max(...times),
      avg: times.reduce((a, b) => a + b, 0) / times.length,
      p95: times.sort((a, b) => a - b)[Math.floor(times.length * 0.95)]
    };
  }
}

// Usage
const monitor = new PerformanceMonitor();

const items = await monitor.measure('fetchItems', async () => {
  return await client.items.list();
});
```

### React DevTools Profiler

Use React DevTools for component profiling:

```typescript
import { Profiler } from 'react';

function onRenderCallback(id, phase, actualDuration) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
}

<Profiler id="ItemList" onRender={onRenderCallback}>
  <ItemList items={items} />
</Profiler>
```

## Performance Checklist

### API Optimization
- [ ] Use batch operations where possible
- [ ] Implement pagination for large datasets
- [ ] Request only necessary fields
- [ ] Execute parallel requests for independent data
- [ ] Cache frequently accessed data

### Plugin Optimization
- [ ] Implement lazy loading for heavy components
- [ ] Debounce field updates
- [ ] Use virtual scrolling for long lists
- [ ] Memoize expensive computations
- [ ] Optimize bundle size with code splitting

### Image Optimization
- [ ] Use responsive images with srcset
- [ ] Implement progressive loading
- [ ] Leverage DatoCMS image transformations
- [ ] Use lazy loading for off-screen images

### Monitoring
- [ ] Track API response times
- [ ] Monitor bundle size
- [ ] Profile React components
- [ ] Set up performance budgets
- [ ] Implement error tracking

## Summary

Performance optimization is an iterative process:
1. **Measure**: Profile and identify bottlenecks
2. **Optimize**: Apply targeted improvements
3. **Monitor**: Track performance over time
4. **Iterate**: Continuously improve based on data