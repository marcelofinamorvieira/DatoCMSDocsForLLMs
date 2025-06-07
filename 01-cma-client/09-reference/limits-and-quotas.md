# Limits and Quotas

## API Rate Limits

### Content Management API
- **Default**: 30 requests per 3 seconds
- **Burst capacity**: Up to 60 requests with proper spacing
- **Headers**: 
  - `X-RateLimit-Limit`: Maximum requests allowed
  - `X-RateLimit-Remaining`: Requests remaining
  - `X-RateLimit-Reset`: Unix timestamp when limit resets

### Dashboard API  
- **Default**: 30 requests per 3 seconds
- **Different limits** for resource-intensive operations

### Handling Rate Limits

```typescript
// The SDK handles rate limits automatically
// But you can also handle them manually:
try {
  await client.items.list();
} catch (error) {
  if (error.response?.status === 429) {
    const retryAfter = error.response.headers.get('X-RateLimit-Reset');
    const waitTime = retryAfter * 1000 - Date.now();
    await new Promise(resolve => setTimeout(resolve, waitTime));
    // Retry the request
  }
}
```

## File Upload Limits

### By Plan Type

| Plan | Max File Size | Monthly Bandwidth | Storage |
|------|--------------|-------------------|---------|
| Free | 25 MB | 10 GB | 1 GB |
| Basic | 250 MB | 100 GB | 10 GB |
| Professional | 500 MB | 500 GB | 50 GB |
| Enterprise | 1 GB+ | Custom | Custom |

### File Type Restrictions
- **Images**: JPG, PNG, GIF, SVG, WEBP, HEIC
- **Videos**: MP4, MOV, AVI, MKV, WEBM
- **Documents**: PDF, DOC, DOCX, XLS, XLSX, PPT, PPTX
- **Other**: Custom types allowed with configuration

## Content Limits

### Items
- **Maximum items per project**: Unlimited (plan-based soft limits)
- **Maximum item size**: 2 MB (including all fields)
- **Maximum fields per model**: 200
- **Maximum blocks in structured text**: 1000

### Models (Item Types)
- **Maximum models per project**: 200
- **Maximum singleton models**: 50
- **Maximum tree depth**: 6 levels

### Fields
- **String field**: 255 characters
- **Text field**: 65,535 characters  
- **Structured text**: ~2 MB
- **JSON field**: 65,535 characters
- **Number fields**: 
  - Integer: -2,147,483,648 to 2,147,483,647
  - Float: IEEE 754 double precision

### Locales
- **Maximum locales per project**: 10 (higher on Enterprise)
- **Locale code format**: ISO 639-1 + optional ISO 3166-1

## API Response Limits

### Pagination
- **Default page size**: 20 items
- **Maximum page size**: 500 items
- **Maximum offset**: 10,000 (use cursor-based pagination for larger datasets)

```typescript
// Efficient pagination for large datasets
for await (const item of client.items.listPagedIterator({
  filter: { type: 'article' },
  page: { limit: 100 } // Optimal batch size
})) {
  // Process each item
}
```

### Response Size
- **Maximum response size**: 10 MB
- **Nested includes depth**: 3 levels
- **Maximum includes**: 100 related records

## Environment Limits

- **Sandbox environments**: 2 (more on Enterprise)
- **Environment name length**: 20 characters
- **Fork operation time**: Depends on data size (up to 30 minutes)

## Webhook Limits

- **Maximum webhooks per project**: 50
- **Webhook timeout**: 30 seconds
- **Maximum retries**: 10
- **Payload size**: 5 MB

## Plugin Limits

- **Maximum plugins per project**: 50
- **Plugin parameter size**: 64 KB
- **Plugin bundle size**: 10 MB
- **Maximum field extensions per field type**: 10

## Build Trigger Limits

- **Maximum triggers per project**: 20
- **Minimum interval between triggers**: 60 seconds
- **Build timeout**: Depends on hosting provider

## User and Permission Limits

- **Maximum users per project**: Unlimited (plan-based)
- **Maximum roles**: 20
- **Maximum access tokens**: 50
- **Token name length**: 50 characters

## Search Limits

- **Maximum search query length**: 100 characters
- **Maximum search results**: 1000
- **Search result snippet length**: 200 characters

## Performance Best Practices

### Optimize Queries
```typescript
// Bad: Multiple requests
const items = await client.items.list();
for (const item of items) {
  const author = await client.items.find(item.author);
}

// Good: Single request with includes
const items = await client.items.list({
  nested: true, // Include nested blocks
  filter: { type: 'article' }
});
```

### Batch Operations
```typescript
// Use bulk operations when available
await client.items.bulkPublish({
  items: items.map(i => ({ type: 'item', id: i.id }))
});
```

### Caching Strategies
```typescript
// Cache frequently accessed data
const cache = new Map();

async function getCachedItemType(id: string) {
  if (!cache.has(id)) {
    cache.set(id, await client.itemTypes.find(id));
  }
  return cache.get(id);
}
```

## Monitoring Usage

```typescript
// Check current usage (Dashboard API)
const usage = await dashboardClient.resourceUsages.find('site-id');
console.log({
  apiCalls: usage.api_calls,
  bandwidth: usage.traffic_bytes,
  storage: usage.storage_bytes
});
```

## Error Codes for Limit Violations

| Code | Description |
|------|-------------|
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `STORAGE_LIMIT_EXCEEDED` | Storage quota exceeded |
| `BANDWIDTH_LIMIT_EXCEEDED` | Bandwidth quota exceeded |
| `PLAN_LIMIT_EXCEEDED` | Feature not available on current plan |
| `SIZE_LIMIT_EXCEEDED` | File or content too large |

## Related Documentation

- [API Endpoints](./api-endpoints.md)
- [Error Handling](../04-advanced-topics/error-handling-patterns.md)
- [Performance Optimization](../04-advanced-topics/performance-optimization.md)