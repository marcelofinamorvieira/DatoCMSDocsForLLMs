# REST Client Utils

This guide covers the `@datocms/rest-client-utils` package, which contains the fundamental building blocks for all DatoCMS API clients. These utilities provide request handling, error classes, serialization logic, and other low-level functions.

## Overview

The `@datocms/rest-client-utils` package is the foundation for:
- `@datocms/cma-client` - Content Management API client
- `@datocms/dashboard-client` - Dashboard API client
- Custom API integrations

It provides:
- HTTP request handling with retry logic
- JSON:API serialization/deserialization
- Error handling and custom error classes
- Pagination utilities
- Cancelable promises
- Job polling mechanisms

## Installation

```bash
npm install @datocms/rest-client-utils
```

## request

The core function for all API calls. Handles HTTP requests with automatic retry logic, timeouts, and comprehensive error handling.

### Function Signature

```typescript
import { request } from '@datocms/rest-client-utils';

async function request<T = any>(
  options: RequestOptions,
  preCallStack?: string
): Promise<T>
```

### RequestOptions

```typescript
interface RequestOptions {
  baseUrl: string;              // Base API URL (e.g., 'https://site-api.datocms.com')
  apiToken?: string;            // DatoCMS API token
  method?: 'GET' | 'PUT' | 'POST' | 'DELETE';  // Default: 'GET'
  url: string;                  // Endpoint path (e.g., '/items')
  body?: any;                   // Request body (will be serialized to JSON)
  queryParams?: Record<string, any>;  // Query parameters
  logLevel?: LogLevel;          // Logging verbosity
  autoRetry?: boolean;          // Enable automatic retry (default: true)
  fetchJobResult?: boolean;     // Auto-fetch async job results (default: true)
  userAgent?: string;           // Custom User-Agent header
  extraHeaders?: Record<string, string>;  // Additional headers
  fetch?: typeof fetch;         // Custom fetch implementation
  requestTimeout?: number;      // Timeout in ms (default: 30000)
  onProgress?: (info: ProgressInfo) => void;  // Progress callback
  signal?: AbortSignal;         // AbortController signal for cancellation
}
```

### Complete Example

```typescript
import { request, LogLevel, ApiError } from '@datocms/rest-client-utils';

async function makeApiCall() {
  try {
    const result = await request({
      baseUrl: 'https://site-api.datocms.com',
      apiToken: 'YOUR_API_TOKEN',
      method: 'POST',
      url: '/items',
      body: {
        type: 'item',
        attributes: {
          title: 'New Article'
        },
        relationships: {
          item_type: {
            data: { type: 'item_type', id: '123' }
          }
        }
      },
      queryParams: {
        include: 'item_type'
      },
      logLevel: LogLevel.BASIC,
      requestTimeout: 60000, // 1 minute
      onProgress: (info) => {
        if (info.type === 'upload') {
          console.log(`Upload progress: ${info.payload.progress}%`);
        }
      }
    });
    
    console.log('Created item:', result);
    return result;
    
  } catch (error) {
    if (error instanceof ApiError) {
      console.error('API Error:', {
        status: error.statusCode,
        message: error.message,
        errors: error.errors
      });
      
      // Check specific error codes
      const validationError = error.findError('title');
      if (validationError?.code === 'VALIDATION_REQUIRED') {
        console.error('Title is required');
      }
    }
    throw error;
  }
}

// Example with custom fetch and retry control
async function customRequest() {
  const customFetch: typeof fetch = (url, init) => {
    console.log('Custom fetch:', url);
    return fetch(url, init);
  };
  
  const result = await request({
    baseUrl: 'https://site-api.datocms.com',
    apiToken: 'YOUR_API_TOKEN',
    url: '/items',
    fetch: customFetch,
    autoRetry: false, // Disable automatic retry
    logLevel: LogLevel.BODY_AND_HEADERS // Maximum logging
  });
  
  return result;
}
```

### Retry Logic

The request function automatically retries on:
- **429 Rate Limit**: Waits based on `X-RateLimit-Reset` header
- **503 Service Unavailable**: Exponential backoff
- **Timeout errors**: Up to 2 retries
- **Network errors**: Up to 2 retries

## serializeRequestBody

Converts simple JavaScript objects into the JSON:API format required by DatoCMS.

### Function Signature

```typescript
import { serializeRequestBody } from '@datocms/rest-client-utils';

function serializeRequestBody<T>(
  body: any,
  options?: SerializationOptions
): T
```

### SerializationOptions

```typescript
interface SerializationOptions {
  type?: string;                    // Resource type (e.g., 'item')
  attributes?: string[] | '*';      // Attributes to include
  relationships?: string[] | '*';   // Relationships to include
  requiredAttributes?: string[];    // Required attributes
  requiredRelationships?: string[]; // Required relationships
}
```

### Complete Example

```typescript
import { serializeRequestBody } from '@datocms/rest-client-utils';

// Simple object
const input = {
  title: 'My Article',
  slug: 'my-article',
  author: { id: '456', type: 'author' },
  category: '789'
};

// Serialize with explicit configuration
const jsonApiBody = serializeRequestBody(input, {
  type: 'item',
  attributes: ['title', 'slug'],
  relationships: ['author', 'category']
});

console.log(jsonApiBody);
/*
{
  type: 'item',
  attributes: {
    title: 'My Article',
    slug: 'my-article'
  },
  relationships: {
    author: {
      data: { id: '456', type: 'author' }
    },
    category: {
      data: { id: '789', type: 'category' }
    }
  }
}
*/

// With wildcard attributes
const autoDetected = serializeRequestBody({
  id: '123',
  title: 'Updated Article',
  content: 'Lorem ipsum...',
  author: { id: '456', type: 'author' }
}, {
  type: 'item',
  attributes: '*',  // All properties except relationships
  relationships: ['author']
});

// Array serialization
const bulkData = serializeRequestBody([
  { id: '1', title: 'Item 1', item_type: '123' },
  { id: '2', title: 'Item 2', item_type: '123' }
], {
  type: 'item',
  attributes: ['title'],
  relationships: ['item_type']
});
```

## deserializeResponseBody

Converts JSON:API responses back to simple JavaScript objects, flattening the structure.

### Function Signature

```typescript
import { deserializeResponseBody } from '@datocms/rest-client-utils';

function deserializeResponseBody<T = any>(body: any): T
```

### Complete Example

```typescript
import { deserializeResponseBody } from '@datocms/rest-client-utils';

// JSON:API response
const jsonApiResponse = {
  data: {
    id: '123',
    type: 'item',
    attributes: {
      title: 'My Article',
      slug: 'my-article'
    },
    relationships: {
      author: {
        data: { id: '456', type: 'user' }
      },
      category: {
        data: { id: '789', type: 'category' }
      }
    }
  },
  included: [
    {
      id: '456',
      type: 'user',
      attributes: { name: 'John Doe' }
    }
  ]
};

// Deserialize to flat object
const result = deserializeResponseBody(jsonApiResponse);

console.log(result);
/*
{
  id: '123',
  type: 'item',
  title: 'My Article',
  slug: 'my-article',
  author: { id: '456', type: 'user' },
  category: { id: '789', type: 'category' }
}
*/

// Array response
const listResponse = {
  data: [
    { id: '1', type: 'item', attributes: { title: 'Item 1' } },
    { id: '2', type: 'item', attributes: { title: 'Item 2' } }
  ]
};

const items = deserializeResponseBody(listResponse);
// Returns array of flattened items
```

## pollJobResult

Polls for asynchronous job completion results with exponential backoff for 404 responses.

### Function Signature

```typescript
import { pollJobResult } from '@datocms/rest-client-utils';

async function pollJobResult(
  jobId: string,
  options: RequestOptions & { attempt?: number }
): Promise<JobResult>
```

### Complete Example

```typescript
import { pollJobResult } from '@datocms/rest-client-utils';

async function waitForJob(jobId: string) {
  try {
    const result = await pollJobResult(jobId, {
      baseUrl: 'https://site-api.datocms.com',
      apiToken: 'YOUR_API_TOKEN',
      url: `/job-results/${jobId}`,
      logLevel: LogLevel.BASIC
    });
    
    if (result.status === 200) {
      console.log('Job completed successfully:', result.payload);
    } else if (result.status >= 400) {
      console.error('Job failed:', result.payload);
    }
    
    return result;
    
  } catch (error) {
    console.error('Error polling job:', error);
    throw error;
  }
}

// With custom attempt tracking
async function pollWithRetry(jobId: string, maxAttempts = 10) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const result = await pollJobResult(jobId, {
        baseUrl: 'https://site-api.datocms.com',
        apiToken: 'YOUR_API_TOKEN',
        url: `/job-results/${jobId}`,
        attempt // Affects backoff timing for 404s
      });
      
      return result;
      
    } catch (error) {
      if (attempt === maxAttempts) throw error;
      await wait(1000 * attempt); // Linear backoff
    }
  }
}
```

## rawPageIterator

An async generator that automates pagination through API responses with configurable concurrency.

### Function Signature

```typescript
import { rawPageIterator } from '@datocms/rest-client-utils';

async function* rawPageIterator<T>(
  request: (url: string) => Promise<{ body: any; response: Response }>,
  baseUrl: string,
  url: string,
  options?: PageIteratorOptions
): AsyncGenerator<T>
```

### PageIteratorOptions

```typescript
interface PageIteratorOptions {
  perPage?: number;      // Items per page (default: 500)
  concurrency?: number;  // Concurrent page fetches (default: 5)
}
```

### Complete Example

```typescript
import { rawPageIterator, request } from '@datocms/rest-client-utils';

async function* getAllItems() {
  const requestFn = (url: string) => request({
    baseUrl: 'https://site-api.datocms.com',
    apiToken: 'YOUR_API_TOKEN',
    url,
    method: 'GET'
  }).then(body => ({
    body,
    response: { headers: new Headers() } as Response
  }));
  
  const iterator = rawPageIterator(
    requestFn,
    'https://site-api.datocms.com',
    '/items',
    {
      perPage: 100,
      concurrency: 3
    }
  );
  
  for await (const item of iterator) {
    yield item;
  }
}

// Usage with filtering and processing
async function processAllPublishedItems() {
  const items = getAllItems();
  let count = 0;
  
  for await (const item of items) {
    if (item.meta.status === 'published') {
      console.log(`Processing item ${item.id}`);
      count++;
    }
  }
  
  console.log(`Processed ${count} published items`);
}

// Collecting all results
async function fetchAllItemsToArray() {
  const items = [];
  
  for await (const item of getAllItems()) {
    items.push(item);
  }
  
  return items;
}
```

## makeCancelablePromise & CanceledPromiseError

Creates promises that can be canceled, useful for abort-able operations.

### Function Signatures

```typescript
import { makeCancelablePromise, CanceledPromiseError } from '@datocms/rest-client-utils';

function makeCancelablePromise<T>(
  promise: Promise<T>
): CancelablePromise<T>

interface CancelablePromise<T> extends Promise<T> {
  cancel(): void;
}
```

### Complete Example

```typescript
import { makeCancelablePromise, CanceledPromiseError } from '@datocms/rest-client-utils';

async function cancelableOperation() {
  // Create a long-running operation
  const longOperation = new Promise((resolve) => {
    setTimeout(() => resolve('Complete'), 5000);
  });
  
  // Make it cancelable
  const cancelable = makeCancelablePromise(longOperation);
  
  // Cancel after 1 second
  setTimeout(() => {
    cancelable.cancel();
    console.log('Operation canceled');
  }, 1000);
  
  try {
    const result = await cancelable;
    console.log('Result:', result);
  } catch (error) {
    if (error instanceof CanceledPromiseError) {
      console.log('Promise was canceled');
    } else {
      throw error;
    }
  }
}

// With fetch requests
async function cancelableFetch(url: string) {
  const controller = new AbortController();
  
  const fetchPromise = fetch(url, {
    signal: controller.signal
  });
  
  const cancelable = makeCancelablePromise(fetchPromise);
  
  // Override cancel to also abort the fetch
  const originalCancel = cancelable.cancel;
  cancelable.cancel = () => {
    controller.abort();
    originalCancel();
  };
  
  return cancelable;
}

// Usage in a component
function SearchComponent() {
  let currentSearch: CancelablePromise<any> | null = null;
  
  async function search(query: string) {
    // Cancel previous search
    if (currentSearch) {
      currentSearch.cancel();
    }
    
    currentSearch = makeCancelablePromise(
      searchAPI(query)
    );
    
    try {
      const results = await currentSearch;
      displayResults(results);
    } catch (error) {
      if (!(error instanceof CanceledPromiseError)) {
        showError(error);
      }
    }
  }
}
```

## ApiError

A comprehensive error class for API errors with detailed request/response information.

### Class Definition

```typescript
class ApiError extends Error {
  request: Request;
  response: Response;
  errors?: ErrorEntity[];
  
  findError(path: string): ErrorEntity | undefined;
  get statusCode(): number;
}

interface ErrorEntity {
  id: string;
  status: string;
  code: string;
  title: string;
  detail: string;
  source?: {
    pointer?: string;
    parameter?: string;
  };
}
```

### Complete Example

```typescript
import { ApiError } from '@datocms/rest-client-utils';

async function handleApiErrors() {
  try {
    const result = await request({
      baseUrl: 'https://site-api.datocms.com',
      apiToken: 'YOUR_API_TOKEN',
      method: 'POST',
      url: '/items',
      body: { /* ... */ }
    });
    
  } catch (error) {
    if (error instanceof ApiError) {
      console.error('API Error Details:', {
        status: error.statusCode,
        message: error.message,
        url: error.request.url,
        method: error.request.method
      });
      
      // Handle specific status codes
      switch (error.statusCode) {
        case 401:
          console.error('Authentication failed');
          break;
        case 403:
          console.error('Permission denied');
          break;
        case 422:
          console.error('Validation errors:');
          error.errors?.forEach(err => {
            console.error(`- ${err.source?.pointer}: ${err.detail}`);
          });
          break;
        case 429:
          const resetTime = error.response.headers.get('X-RateLimit-Reset');
          console.error(`Rate limited until ${new Date(Number(resetTime) * 1000)}`);
          break;
      }
      
      // Find specific field errors
      const titleError = error.findError('title');
      if (titleError) {
        console.error('Title error:', titleError.detail);
        
        // Check error codes
        switch (titleError.code) {
          case 'VALIDATION_REQUIRED':
            console.error('Title is required');
            break;
          case 'VALIDATION_UNIQUE':
            console.error('Title must be unique');
            break;
          case 'VALIDATION_FORMAT':
            console.error('Title format is invalid');
            break;
        }
      }
      
      // Access raw response if needed
      const responseText = await error.response.text();
      console.error('Raw response:', responseText);
    }
    
    throw error;
  }
}
```

## TimeoutError

Error thrown when a request exceeds the timeout limit.

### Class Definition

```typescript
class TimeoutError extends Error {
  constructor(message: string);
}
```

### Example

```typescript
import { TimeoutError, request } from '@datocms/rest-client-utils';

async function timeoutHandling() {
  try {
    const result = await request({
      baseUrl: 'https://site-api.datocms.com',
      apiToken: 'YOUR_API_TOKEN',
      url: '/large-operation',
      requestTimeout: 5000 // 5 seconds
    });
    
  } catch (error) {
    if (error instanceof TimeoutError) {
      console.error('Request timed out after 5 seconds');
      // Retry with longer timeout
      return request({
        baseUrl: 'https://site-api.datocms.com',
        apiToken: 'YOUR_API_TOKEN',
        url: '/large-operation',
        requestTimeout: 30000 // 30 seconds
      });
    }
    throw error;
  }
}
```

## Other Utilities

### toId

Extracts ID from a string or object.

```typescript
import { toId } from '@datocms/rest-client-utils';

toId('123');                          // '123'
toId({ id: '456', type: 'item' });   // '456'
toId(null);                           // null
```

### wait

Promise-based delay function.

```typescript
import { wait } from '@datocms/rest-client-utils';

async function delayedOperation() {
  console.log('Starting...');
  await wait(2000); // Wait 2 seconds
  console.log('Continuing after delay');
}
```

### LogLevel

Enum for controlling request logging verbosity.

```typescript
import { LogLevel } from '@datocms/rest-client-utils';

enum LogLevel {
  NONE = 0,              // No logging
  BASIC = 1,             // URL and method
  BODY = 2,              // + Request/response bodies
  BODY_AND_HEADERS = 3   // + All headers
}
```

### buildNormalizedParams

Converts nested objects to query string parameters.

```typescript
import { buildNormalizedParams } from '@datocms/rest-client-utils';

const params = buildNormalizedParams({
  filter: {
    type: 'article',
    fields: {
      title: { matches: { pattern: 'news' } }
    }
  },
  page: { limit: 20, offset: 0 }
});

// Returns: filter[type]=article&filter[fields][title][matches][pattern]=news&page[limit]=20&page[offset]=0
```

## Best Practices

### Error Handling

Always check for specific error types:

```typescript
try {
  const result = await request(options);
} catch (error) {
  if (error instanceof ApiError) {
    // Handle API errors
  } else if (error instanceof TimeoutError) {
    // Handle timeouts
  } else if (error instanceof CanceledPromiseError) {
    // Handle cancellation
  } else {
    // Handle other errors
  }
}
```

### Request Configuration

```typescript
const defaultOptions: Partial<RequestOptions> = {
  logLevel: process.env.NODE_ENV === 'development' 
    ? LogLevel.BODY 
    : LogLevel.NONE,
  requestTimeout: 60000, // 1 minute for production
  autoRetry: true
};

function apiRequest(options: RequestOptions) {
  return request({ ...defaultOptions, ...options });
}
```

### Pagination Best Practices

```typescript
// Process items in chunks
async function processInChunks(chunkSize = 100) {
  const items = [];
  
  for await (const item of getAllItems()) {
    items.push(item);
    
    if (items.length >= chunkSize) {
      await processBatch(items);
      items.length = 0;
    }
  }
  
  // Process remaining items
  if (items.length > 0) {
    await processBatch(items);
  }
}
```

## Integration Examples

### Building a Custom Client

```typescript
import {
  request,
  ApiError,
  LogLevel,
  serializeRequestBody,
  deserializeResponseBody
} from '@datocms/rest-client-utils';

class CustomDatoCMSClient {
  private baseUrl = 'https://site-api.datocms.com';
  private apiToken: string;
  private logLevel: LogLevel;
  
  constructor(apiToken: string, options: { logLevel?: LogLevel } = {}) {
    this.apiToken = apiToken;
    this.logLevel = options.logLevel || LogLevel.NONE;
  }
  
  private async request<T = any>(options: Partial<RequestOptions>): Promise<T> {
    const response = await request({
      baseUrl: this.baseUrl,
      apiToken: this.apiToken,
      logLevel: this.logLevel,
      ...options
    });
    
    return deserializeResponseBody(response);
  }
  
  async getItem(itemId: string) {
    return this.request({
      url: `/items/${itemId}`
    });
  }
  
  async createItem(data: any) {
    const body = serializeRequestBody(data, {
      type: 'item',
      attributes: '*',
      relationships: ['item_type']
    });
    
    return this.request({
      method: 'POST',
      url: '/items',
      body
    });
  }
  
  async *listAllItems() {
    const requestFn = (url: string) => request({
      baseUrl: this.baseUrl,
      apiToken: this.apiToken,
      url
    }).then(body => ({
      body,
      response: { headers: new Headers() } as Response
    }));
    
    const iterator = rawPageIterator(
      requestFn,
      this.baseUrl,
      '/items',
      { perPage: 500 }
    );
    
    for await (const item of iterator) {
      yield deserializeResponseBody({ data: item });
    }
  }
}
```

## Related Documentation

- [CMA Client](../01-cma-client/README.md) - High-level client built on these utilities
- [Dashboard Client](../02-dashboard-client/README.md) - Account API client using these utilities
- [Error Handling Patterns](./error-handling-patterns.md) - Best practices for error handling
- [REST API Events](./rest-api-events.md) - WebSocket integration for async operations