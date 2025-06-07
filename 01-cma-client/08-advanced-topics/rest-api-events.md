# REST API Events

Real-time job result notifications via WebSocket for DatoCMS async operations.

## Overview

The `@datocms/rest-api-events` package provides WebSocket-based real-time notifications for async job results in DatoCMS. When you perform operations that return a Job (like site duplication, bulk operations, etc.), this package allows you to receive the results instantly via WebSocket instead of polling.

## Installation

```bash
npm install @datocms/rest-api-events
```

## Basic Usage

### With Event Subscription Wrapper

The easiest way to use this package is with the `withEventsSubscription` wrapper that augments your CMA client:

```typescript
import { buildClient } from '@datocms/cma-client';
import { withEventsSubscription } from '@datocms/rest-api-events';

// Create base client
const baseClient = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

// Augment with event subscription
const [client, unsubscribe] = await withEventsSubscription(baseClient);

// Now use the client normally - job results will come via WebSocket
const job = await client.site.duplicate('site-id', {
  name: 'Copy of My Site'
});

// This will receive the result via WebSocket when ready
const result = await client.jobResultsFetcher(job.id);
console.log('Duplication completed:', result);

// Clean up when done
unsubscribe();
```

### Direct Subscription Usage

For more control, you can use the subscription directly:

```typescript
import { buildClient } from '@datocms/cma-client';
import { subscribeToEvents, JobResultsFetcher } from '@datocms/rest-api-events';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

// Create subscription
const subscription = await subscribeToEvents({
  client,
  onJobResult: (jobId, result) => {
    console.log(`Job ${jobId} completed:`, result);
  }
});

// Create fetcher
const fetcher = new JobResultsFetcher({
  client,
  subscription
});

// Use for job results
const job = await client.site.duplicate('site-id', {
  name: 'Copy of My Site'  
});

const result = await fetcher.get(job.id);

// Clean up
subscription.unsubscribe();
```

## How It Works

1. **WebSocket Connection**: Uses Pusher.js to establish a WebSocket connection to DatoCMS
2. **Private Channel**: Subscribes to a private channel specific to your API token
3. **Job Notifications**: Receives `job-result` events when async jobs complete
4. **Automatic Fallback**: If the job result payload is too large (>10KB), it automatically falls back to fetching via REST API
5. **No Polling**: Eliminates the need for polling the job result endpoint

## API Reference

### `withEventsSubscription(client, options?)`

Augments a CMA client with WebSocket-based job result fetching.

**Parameters:**
- `client`: CMA client instance
- `options` (optional):
  - `fetchJobResult`: Custom function to fetch job results via REST
  - `onJobResult`: Callback when job results are received

**Returns:**
- Tuple: `[augmentedClient, unsubscribe]`

**Example:**
```typescript
const [client, unsubscribe] = await withEventsSubscription(baseClient, {
  onJobResult: (jobId, result) => {
    console.log(`Job ${jobId} completed`);
  }
});
```

### `subscribeToEvents(options)`

Creates a WebSocket subscription for job results.

**Parameters:**
- `client`: CMA client instance (required)
- `onJobResult`: Callback function `(jobId: string, result: any) => void`

**Returns:**
- Subscription object with `unsubscribe()` method

### `JobResultsFetcher`

Class that manages fetching job results with WebSocket support.

**Constructor:**
```typescript
new JobResultsFetcher({
  client: CmaClient,
  subscription: Subscription,
  fetchJobResult?: (jobId: string) => Promise<JobResult>
})
```

**Methods:**
- `get(jobId: string, attempt?: number): Promise<JobResult>` - Fetches a job result

## Common Use Cases

### Site Operations

```typescript
const [client, unsubscribe] = await withEventsSubscription(baseClient);

// Duplicate site - result via WebSocket
const duplicateJob = await client.site.duplicate('site-id', {
  name: 'Copy of Site'
});
const duplicateResult = await client.jobResultsFetcher(duplicateJob.id);

// API token regeneration - result via WebSocket  
const tokenJob = await client.account.apiTokenRegenerate('account-id', {
  current_password: 'password'
});
const tokenResult = await client.jobResultsFetcher(tokenJob.id);

unsubscribe();
```

### Bulk Operations

```typescript
const [client, unsubscribe] = await withEventsSubscription(baseClient);

// Bulk update - result via WebSocket
const bulkJob = await client.items.bulkUpdate({
  items: items.map(item => ({
    id: item.id,
    attributes: { title: 'Updated' }
  }))
});

const result = await client.jobResultsFetcher(bulkJob.id);
console.log(`Updated ${result.data.length} items`);

unsubscribe();
```

### With Error Handling

```typescript
const [client, unsubscribe] = await withEventsSubscription(baseClient);

try {
  const job = await client.site.duplicate('site-id', {
    name: 'Copy'
  });
  
  const result = await client.jobResultsFetcher(job.id);
  
  if (result.status === 'failed') {
    console.error('Job failed:', result.error);
  } else {
    console.log('Success:', result.payload);
  }
} catch (error) {
  console.error('Error:', error);
} finally {
  unsubscribe();
}
```

## Integration with Async/Await

The package integrates seamlessly with async/await patterns:

```typescript
async function duplicateSite(siteId: string, newName: string) {
  const baseClient = buildClient({ apiToken: API_TOKEN });
  const [client, unsubscribe] = await withEventsSubscription(baseClient);
  
  try {
    const job = await client.site.duplicate(siteId, { name: newName });
    const result = await client.jobResultsFetcher(job.id);
    
    if (result.status === 'completed') {
      return result.payload; // New site data
    } else {
      throw new Error(result.error);
    }
  } finally {
    unsubscribe();
  }
}
```

## Performance Benefits

- **No Polling**: Eliminates repeated HTTP requests to check job status
- **Instant Notifications**: Receive results as soon as jobs complete
- **Reduced Latency**: WebSocket delivery is faster than polling intervals
- **Lower API Usage**: Fewer API calls means more headroom for other operations

## Important Notes

1. **Job Results Only**: This package only handles job results, not general content events
2. **Pusher-based**: Uses Pusher.js under the hood for WebSocket connectivity
3. **Automatic Cleanup**: Always call `unsubscribe()` to close the WebSocket connection
4. **Payload Size**: Large job results (>10KB) automatically fall back to REST API fetching
5. **Connection Limit**: Each API token can maintain multiple concurrent WebSocket connections

## Error Handling

The WebSocket connection handles errors gracefully:

```typescript
const [client, unsubscribe] = await withEventsSubscription(baseClient, {
  onJobResult: (jobId, result) => {
    if (result.status === 'failed') {
      console.error(`Job ${jobId} failed:`, result.error);
      // Handle failure
    }
  }
});
```

## TypeScript Support

Full TypeScript support with proper typing:

```typescript
import { 
  withEventsSubscription, 
  JobResultsFetcher,
  Subscription 
} from '@datocms/rest-api-events';
import type { JobResult } from '@datocms/cma-client';

const handleResult = (jobId: string, result: JobResult) => {
  // Fully typed
};
```

## subscribeToEvents Function

The core function for creating a WebSocket subscription. This is the foundation that other utilities build upon.

### Function Signature

```typescript
import { subscribeToEvents } from '@datocms/rest-api-events';

async function subscribeToEvents(config: SubscriptionConfig): Promise<EventsSubscription>
```

### Parameters

- `authEndpoint`: The authentication endpoint URL (e.g., 'https://account-api.datocms.com/pusher/authenticate')
- `apiToken`: Your DatoCMS API token
- `channelName`: The private channel name (format: `private-v3-channel-{channelId}`)
- `cluster` (optional): Pusher cluster (default: 'eu')
- `appKey` (optional): Pusher app key (default: 'd53b5fbc76b7e96e6c5a')

### Returns

An `EventsSubscription` object containing:
- `channel`: The Pusher channel instance
- `waitJobResult(jobId: string)`: Method to wait for a specific job result
- `unsubscribe()`: Method to close the connection

### Complete Example

```typescript
import { subscribeToEvents } from '@datocms/rest-api-events';
import { buildClient } from '@datocms/cma-client';

async function customEventHandling() {
  const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });
  
  // Create subscription with custom configuration
  const subscription = await subscribeToEvents({
    authEndpoint: 'https://account-api.datocms.com/pusher/authenticate',
    apiToken: 'YOUR_API_TOKEN',
    channelName: `private-v3-channel-${client.config.environment || 'main'}`,
    cluster: 'eu',
    appKey: 'd53b5fbc76b7e96e6c5a'
  });
  
  // Listen to the raw channel events
  subscription.channel.bind('job-result', (data: any) => {
    console.log('Received job result event:', data);
  });
  
  // Start an async operation
  const job = await client.site.duplicate('site-id', {
    name: 'Copy of Site'
  });
  
  // Wait for the specific job result
  const result = await subscription.waitJobResult(job.id);
  console.log('Job completed:', result);
  
  // Clean up
  subscription.unsubscribe();
}
```

## JobResultsFetcher Class

A stateful fetcher that uses an event subscription to receive job results. This class provides a higher-level interface for working with job results.

### Constructor

```typescript
import { JobResultsFetcher } from '@datocms/rest-api-events';

new JobResultsFetcher({
  client: GenericClient,
  subscription?: EventsSubscription,
  fetchJobResult?: (jobId: string) => Promise<JobResult>
})
```

### Parameters

- `client`: A DatoCMS API client instance
- `subscription` (optional): An existing EventsSubscription
- `fetchJobResult` (optional): Custom function to fetch job results via REST

### Methods

#### subscribeToEvents()

Creates or returns the event subscription.

```typescript
async subscribeToEvents(): Promise<EventsSubscription>
```

#### fetch(jobId, attempt?)

Fetches a job result, either from WebSocket events or REST API.

```typescript
async fetch(jobId: string, attempt?: number): Promise<JobResult>
```

- `jobId`: The job ID to fetch
- `attempt` (optional): Current attempt number for retries

#### unsubscribeToEvents()

Closes the WebSocket subscription if one exists.

```typescript
unsubscribeToEvents(): void
```

### Complete Example

```typescript
import { JobResultsFetcher, subscribeToEvents } from '@datocms/rest-api-events';
import { buildClient } from '@datocms/cma-client';

async function manualFetcherUsage() {
  const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });
  
  // Create subscription manually
  const subscription = await subscribeToEvents({
    authEndpoint: 'https://account-api.datocms.com/pusher/authenticate',
    apiToken: client.config.apiToken,
    channelName: `private-v3-channel-${client.config.environment || 'main'}`
  });
  
  // Create fetcher with custom job result handler
  const fetcher = new JobResultsFetcher({
    client,
    subscription,
    fetchJobResult: async (jobId) => {
      // Custom REST fallback logic
      console.log(`Fetching job ${jobId} via REST API`);
      const response = await fetch(
        `https://site-api.datocms.com/job-results/${jobId}`,
        {
          headers: {
            'Authorization': `Bearer ${client.config.apiToken}`,
            'X-Api-Version': '3'
          }
        }
      );
      return response.json();
    }
  });
  
  // Use the fetcher
  const job = await client.site.duplicate('site-id', {
    name: 'New Site Copy'
  });
  
  const result = await fetcher.fetch(job.id);
  console.log('Result:', result);
  
  // Clean up
  fetcher.unsubscribeToEvents();
}
```

## Advanced Usage

### Custom Event Handling

```typescript
import { subscribeToEvents } from '@datocms/rest-api-events';

async function advancedEventHandling() {
  const jobQueue = new Map<string, (result: any) => void>();
  
  const subscription = await subscribeToEvents({
    authEndpoint: 'https://account-api.datocms.com/pusher/authenticate',
    apiToken: 'YOUR_API_TOKEN',
    channelName: 'private-v3-channel-main'
  });
  
  // Custom event handler
  subscription.channel.bind('job-result', (event: any) => {
    const { id: jobId, status, payload } = event;
    
    // Custom logging
    console.log(`Job ${jobId} status: ${status}`);
    
    // Resolve waiting promises
    const resolver = jobQueue.get(jobId);
    if (resolver) {
      resolver({ id: jobId, status, payload });
      jobQueue.delete(jobId);
    }
    
    // Custom metrics
    trackJobCompletion(jobId, status);
  });
  
  // Helper to wait for jobs
  function waitForJob(jobId: string): Promise<any> {
    return new Promise((resolve) => {
      jobQueue.set(jobId, resolve);
    });
  }
  
  // Usage
  const job = await client.site.duplicate('site-id', { name: 'Copy' });
  const result = await waitForJob(job.id);
  
  subscription.unsubscribe();
}
```

### Multiple Subscriptions

```typescript
async function multipleSubscriptions() {
  const clients = [
    buildClient({ apiToken: 'TOKEN_1', environment: 'main' }),
    buildClient({ apiToken: 'TOKEN_2', environment: 'staging' })
  ];
  
  // Create separate subscriptions for each environment
  const subscriptions = await Promise.all(
    clients.map(async (client) => {
      const subscription = await subscribeToEvents({
        authEndpoint: 'https://account-api.datocms.com/pusher/authenticate',
        apiToken: client.config.apiToken,
        channelName: `private-v3-channel-${client.config.environment}`
      });
      
      return { client, subscription };
    })
  );
  
  // Use each subscription independently
  const jobs = await Promise.all(
    subscriptions.map(({ client, subscription }) => 
      client.site.duplicate('site-id', { name: 'Copy' })
        .then(job => subscription.waitJobResult(job.id))
    )
  );
  
  // Clean up all subscriptions
  subscriptions.forEach(({ subscription }) => subscription.unsubscribe());
}
```

### Connection Monitoring

```typescript
async function monitorConnection() {
  const subscription = await subscribeToEvents({
    authEndpoint: 'https://account-api.datocms.com/pusher/authenticate',
    apiToken: 'YOUR_API_TOKEN',
    channelName: 'private-v3-channel-main'
  });
  
  // Monitor connection state
  const pusher = subscription.channel.pusher;
  
  pusher.connection.bind('state_change', (states: any) => {
    console.log(`Connection state: ${states.previous} -> ${states.current}`);
  });
  
  pusher.connection.bind('connected', () => {
    console.log('WebSocket connected');
  });
  
  pusher.connection.bind('disconnected', () => {
    console.log('WebSocket disconnected');
  });
  
  pusher.connection.bind('error', (error: any) => {
    console.error('WebSocket error:', error);
  });
  
  // Your operations here
  
  subscription.unsubscribe();
}
```

### Retry Logic with Exponential Backoff

```typescript
async function fetchWithRetry(fetcher: JobResultsFetcher, jobId: string) {
  const maxAttempts = 5;
  let lastError;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      const result = await fetcher.fetch(jobId, attempt);
      
      if (result.status === 'failed' && attempt < maxAttempts) {
        // Job failed, but we might retry the operation
        console.log(`Job failed, attempt ${attempt}/${maxAttempts}`);
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
        continue;
      }
      
      return result;
    } catch (error) {
      lastError = error;
      console.error(`Attempt ${attempt} failed:`, error);
      
      if (attempt < maxAttempts) {
        // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
      }
    }
  }
  
  throw lastError;
}
```

## Implementation Details

### Connection Caching

The `subscribeToEvents` function implements connection caching to avoid creating duplicate WebSocket connections:

```typescript
// Connections are cached by: endpoint + channel + token
const cacheKey = `${authEndpoint}:${channelName}:${apiToken}`;

// Reuses existing connection if available
if (existingConnection) {
  return existingConnection;
}
```

### Large Payload Handling

When a job result payload exceeds 10KB, the system automatically falls back to REST API fetching:

```typescript
// In the WebSocket event handler
if (event.status === 413) { // Payload too large
  // Fetch via REST API instead
  const result = await fetchJobResult(jobId);
  return result;
}
```

### Event Queue System

The subscription maintains an internal queue for job results:

```typescript
// Results are queued as they arrive
jobResultsQueue.set(jobId, result);

// waitJobResult checks the queue first
if (jobResultsQueue.has(jobId)) {
  return jobResultsQueue.get(jobId);
}

// Otherwise waits for the event
return new Promise((resolve) => {
  waitingPromises.set(jobId, resolve);
});
```