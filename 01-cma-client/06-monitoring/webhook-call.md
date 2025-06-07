# Webhook Call

Webhook Calls represent individual webhook delivery attempts. They provide detailed information about webhook executions, including request/response data, timing, and retry attempts.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// List webhook calls for a specific webhook
const calls = await client.webhookCalls.list({
  filter: { webhook: { eq: 'webhook-id' } }
});

// Resend a failed webhook call
await client.webhookCalls.resendWebhook('webhook-call-id');
```

## API Reference

### List Webhook Calls

Retrieve webhook call history with filtering options.

```typescript
// List all webhook calls
const allCalls = await client.webhookCalls.list();

// Filter by webhook
const webhookCalls = await client.webhookCalls.list({
  filter: {
    webhook: { eq: 'webhook-id' }
  }
});

// Filter by status
const failedCalls = await client.webhookCalls.list({
  filter: {
    response_status: { gte: 400 }
  },
  order_by: ['created_at_DESC']
});

// Filter by date range
const recentCalls = await client.webhookCalls.list({
  filter: {
    created_at: {
      gte: '2024-01-01T00:00:00Z',
      lte: '2024-01-31T23:59:59Z'
    }
  }
});

// Paginated retrieval
const paginatedCalls = await client.webhookCalls.list({
  page: { offset: 0, limit: 100 }
});
```

**Parameters:**
- `filter` (object, optional): Filter conditions
  - `webhook`: Filter by webhook ID
  - `response_status`: Filter by HTTP response status
  - `created_at`: Filter by creation date
- `order_by` (array, optional): Sort order
- `page` (object, optional): Pagination options

**Returns:** Array of webhook call objects

### Find Webhook Call

Retrieve details of a specific webhook call.

```typescript
const call = await client.webhookCalls.find('webhook-call-id');

console.log(`Webhook: ${call.webhook.id}`);
console.log(`Status: ${call.response_status}`);
console.log(`Duration: ${call.response_time_ms}ms`);
console.log(`Attempt: ${call.attempt_number}`);
```

**Parameters:**
- `webhookCallId` (string, required): The webhook call ID

**Returns:** Webhook call object with full details

### Resend Webhook

Manually retry a webhook call.

```typescript
// Resend a failed webhook
const newCall = await client.webhookCalls.resendWebhook('webhook-call-id');

console.log(`New webhook call created: ${newCall.id}`);
console.log(`Status: ${newCall.response_status}`);
```

**Parameters:**
- `webhookCallId` (string, required): The webhook call ID to resend

**Returns:** New webhook call object from the resend attempt

## Webhook Monitoring

### Health Monitoring

Monitor webhook delivery health:

```typescript
async function monitorWebhookHealth(webhookId: string, hours = 24) {
  const since = new Date();
  since.setHours(since.getHours() - hours);
  
  const calls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      created_at: { gte: since.toISOString() }
    }
  });
  
  const stats = {
    total: calls.length,
    successful: 0,
    failed: 0,
    avgResponseTime: 0,
    statusCodes: {},
    errors: []
  };
  
  let totalResponseTime = 0;
  
  calls.forEach(call => {
    if (call.response_status >= 200 && call.response_status < 300) {
      stats.successful++;
    } else {
      stats.failed++;
      stats.errors.push({
        id: call.id,
        status: call.response_status,
        time: call.created_at
      });
    }
    
    // Track status codes
    stats.statusCodes[call.response_status] = 
      (stats.statusCodes[call.response_status] || 0) + 1;
    
    // Calculate average response time
    if (call.response_time_ms) {
      totalResponseTime += call.response_time_ms;
    }
  });
  
  stats.avgResponseTime = calls.length > 0 ? 
    Math.round(totalResponseTime / calls.length) : 0;
  
  return {
    webhookId,
    period: `${hours} hours`,
    health: stats.failed === 0 ? 'healthy' : 
            stats.failed / stats.total > 0.1 ? 'unhealthy' : 'degraded',
    stats
  };
}

// Check webhook health
const health = await monitorWebhookHealth('webhook-id');
console.log(`Webhook health: ${health.health}`);
console.log(`Success rate: ${(health.stats.successful / health.stats.total * 100).toFixed(1)}%`);
```

### Failure Analysis

Analyze webhook failures:

```typescript
async function analyzeWebhookFailures(webhookId: string, days = 7) {
  const since = new Date();
  since.setDate(since.getDate() - days);
  
  const failedCalls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      response_status: { gte: 400 },
      created_at: { gte: since.toISOString() }
    },
    order_by: ['created_at_DESC']
  });
  
  // Group failures by type
  const analysis = {
    totalFailures: failedCalls.length,
    byStatus: {},
    byHour: new Array(24).fill(0),
    commonErrors: {},
    retryPatterns: {
      retriedSuccessfully: 0,
      maxRetriesReached: 0,
      pendingRetry: 0
    }
  };
  
  failedCalls.forEach(call => {
    // Group by status code
    const statusGroup = `${Math.floor(call.response_status / 100)}xx`;
    analysis.byStatus[statusGroup] = (analysis.byStatus[statusGroup] || 0) + 1;
    
    // Group by hour of day
    const hour = new Date(call.created_at).getHours();
    analysis.byHour[hour]++;
    
    // Track error messages
    if (call.response_body) {
      const errorKey = call.response_body.substring(0, 100);
      analysis.commonErrors[errorKey] = (analysis.commonErrors[errorKey] || 0) + 1;
    }
    
    // Analyze retry patterns
    if (call.attempt_number > 1) {
      if (call.response_status >= 200 && call.response_status < 300) {
        analysis.retryPatterns.retriedSuccessfully++;
      } else if (call.attempt_number >= 5) {
        analysis.retryPatterns.maxRetriesReached++;
      } else {
        analysis.retryPatterns.pendingRetry++;
      }
    }
  });
  
  return analysis;
}
```

### Performance Tracking

Monitor webhook performance metrics:

```typescript
async function trackWebhookPerformance(webhookId: string) {
  const calls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      response_status: { gte: 200, lt: 300 }
    },
    order_by: ['created_at_DESC'],
    page: { limit: 1000 }
  });
  
  if (calls.length === 0) {
    return { error: 'No successful calls found' };
  }
  
  // Calculate percentiles
  const responseTimes = calls
    .map(c => c.response_time_ms)
    .filter(t => t !== null)
    .sort((a, b) => a - b);
  
  const percentile = (p: number) => {
    const index = Math.ceil(responseTimes.length * p / 100) - 1;
    return responseTimes[index] || 0;
  };
  
  return {
    metrics: {
      count: calls.length,
      avgResponseTime: Math.round(
        responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length
      ),
      minResponseTime: responseTimes[0],
      maxResponseTime: responseTimes[responseTimes.length - 1],
      p50: percentile(50),
      p95: percentile(95),
      p99: percentile(99)
    },
    performance: {
      fast: responseTimes.filter(t => t < 200).length,
      medium: responseTimes.filter(t => t >= 200 && t < 1000).length,
      slow: responseTimes.filter(t => t >= 1000).length
    }
  };
}
```

## Retry Management

### Manual Retry Strategy

Implement custom retry logic for failed webhooks:

```typescript
class WebhookRetryManager {
  static async retryFailedWebhooks(
    webhookId: string,
    options: {
      maxAge?: number; // hours
      statusCodes?: number[];
      maxRetries?: number;
    } = {}
  ) {
    const maxAge = options.maxAge || 24;
    const since = new Date();
    since.setHours(since.getHours() - maxAge);
    
    // Find failed calls
    const failedCalls = await client.webhookCalls.list({
      filter: {
        webhook: { eq: webhookId },
        response_status: { gte: 400 },
        created_at: { gte: since.toISOString() }
      }
    });
    
    // Filter by status codes if specified
    const callsToRetry = options.statusCodes ?
      failedCalls.filter(c => options.statusCodes.includes(c.response_status)) :
      failedCalls;
    
    const results = {
      total: callsToRetry.length,
      retried: [],
      failed: [],
      skipped: []
    };
    
    for (const call of callsToRetry) {
      // Skip if already retried too many times
      if (options.maxRetries && call.attempt_number >= options.maxRetries) {
        results.skipped.push({
          id: call.id,
          reason: 'Max retries reached',
          attempts: call.attempt_number
        });
        continue;
      }
      
      try {
        const newCall = await client.webhookCalls.resendWebhook(call.id);
        results.retried.push({
          originalId: call.id,
          newId: newCall.id,
          status: newCall.response_status
        });
      } catch (error) {
        results.failed.push({
          id: call.id,
          error: error.message
        });
      }
      
      // Add delay to avoid overwhelming the endpoint
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
    
    return results;
  }
}

// Retry failed webhooks from last 6 hours
const retryResults = await WebhookRetryManager.retryFailedWebhooks('webhook-id', {
  maxAge: 6,
  statusCodes: [500, 502, 503, 504], // Only retry server errors
  maxRetries: 3
});

console.log(`Retried: ${retryResults.retried.length}/${retryResults.total}`);
```

### Bulk Operations

Handle webhook calls in bulk:

```typescript
async function bulkResendWebhooks(callIds: string[]) {
  const results = {
    successful: [],
    failed: []
  };
  
  // Process in batches to avoid rate limits
  const batchSize = 10;
  for (let i = 0; i < callIds.length; i += batchSize) {
    const batch = callIds.slice(i, i + batchSize);
    
    const batchResults = await Promise.allSettled(
      batch.map(id => client.webhookCalls.resendWebhook(id))
    );
    
    batchResults.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        results.successful.push({
          originalId: batch[index],
          newCall: result.value
        });
      } else {
        results.failed.push({
          id: batch[index],
          error: result.reason
        });
      }
    });
    
    // Rate limiting
    if (i + batchSize < callIds.length) {
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }
  
  return results;
}
```

## Debugging and Troubleshooting

### Request/Response Inspector

Inspect webhook call details for debugging:

```typescript
async function inspectWebhookCall(callId: string) {
  const call = await client.webhookCalls.find(callId);
  
  const inspection = {
    summary: {
      id: call.id,
      webhook: call.webhook.id,
      status: call.response_status,
      duration: `${call.response_time_ms}ms`,
      attempt: call.attempt_number,
      timestamp: call.created_at
    },
    request: {
      url: call.request_url,
      method: call.request_method,
      headers: JSON.parse(call.request_headers || '{}'),
      body: call.request_body ? JSON.parse(call.request_body) : null
    },
    response: {
      status: call.response_status,
      headers: JSON.parse(call.response_headers || '{}'),
      body: call.response_body,
      time: call.response_time_ms
    },
    event: {
      type: call.event_type,
      entity: call.entity_type,
      entityId: call.entity_id
    }
  };
  
  // Format for display
  console.log('=== Webhook Call Inspection ===');
  console.log(`ID: ${inspection.summary.id}`);
  console.log(`Status: ${inspection.summary.status}`);
  console.log(`Duration: ${inspection.summary.duration}`);
  console.log('\n--- Request ---');
  console.log(`${inspection.request.method} ${inspection.request.url}`);
  console.log('Headers:', inspection.request.headers);
  if (inspection.request.body) {
    console.log('Body:', inspection.request.body);
  }
  console.log('\n--- Response ---');
  console.log(`Status: ${inspection.response.status}`);
  console.log('Headers:', inspection.response.headers);
  if (inspection.response.body) {
    console.log('Body:', inspection.response.body);
  }
  
  return inspection;
}
```

### Event Correlation

Correlate webhook calls with their triggering events:

```typescript
async function correlateWebhookEvents(webhookId: string, hours = 1) {
  const since = new Date();
  since.setHours(since.getHours() - hours);
  
  // Get webhook calls
  const calls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      created_at: { gte: since.toISOString() }
    }
  });
  
  // Group by event and entity
  const events = {};
  
  calls.forEach(call => {
    const key = `${call.event_type}:${call.entity_type}:${call.entity_id}`;
    
    if (!events[key]) {
      events[key] = {
        eventType: call.event_type,
        entityType: call.entity_type,
        entityId: call.entity_id,
        calls: []
      };
    }
    
    events[key].calls.push({
      id: call.id,
      status: call.response_status,
      attempt: call.attempt_number,
      time: call.created_at
    });
  });
  
  // Find duplicate events
  const duplicates = Object.values(events)
    .filter(e => e.calls.length > 1)
    .map(e => ({
      ...e,
      duplicateCount: e.calls.length
    }));
  
  return {
    totalCalls: calls.length,
    uniqueEvents: Object.keys(events).length,
    duplicateEvents: duplicates
  };
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.webhookCalls.resendWebhook('invalid-id');
} catch (error) {
  if (error instanceof ApiError) {
    if (error.response.status === 404) {
      console.log('Webhook call not found');
    } else if (error.response.status === 422) {
      console.log('Cannot resend webhook call - check if webhook still exists');
    }
  }
}

// Handle rate limiting
try {
  // Rapid resends
  for (let i = 0; i < 100; i++) {
    await client.webhookCalls.resendWebhook('call-id');
  }
} catch (error) {
  if (error instanceof ApiError && error.response.status === 429) {
    console.log('Rate limit exceeded - wait before retrying');
    const retryAfter = error.response.headers['retry-after'];
    console.log(`Retry after ${retryAfter} seconds`);
  }
}
```

## Webhook Call Object Structure

```typescript
interface WebhookCall {
  id: string;
  type: 'webhook_call';
  request_url: string;
  request_method: string;
  request_headers: string; // JSON string
  request_body: string; // JSON string
  response_status: number;
  response_headers: string; // JSON string
  response_body: string;
  response_time_ms: number;
  created_at: string;
  attempt_number: number;
  event_type: string; // e.g., 'create', 'update', 'publish'
  entity_type: string; // e.g., 'item', 'upload'
  entity_id: string;
  webhook: {
    type: 'webhook';
    id: string;
  };
}

// Example webhook call object
{
  id: '98765',
  type: 'webhook_call',
  request_url: 'https://api.example.com/webhook',
  request_method: 'POST',
  request_headers: '{"Content-Type":"application/json","X-DatoCMS-Topic":"item:create"}',
  request_body: '{"entity":{"id":"12345","type":"item"},"event_type":"create"}',
  response_status: 200,
  response_headers: '{"Content-Type":"application/json"}',
  response_body: '{"status":"ok"}',
  response_time_ms: 145,
  created_at: '2024-01-15T10:30:00Z',
  attempt_number: 1,
  event_type: 'create',
  entity_type: 'item',
  entity_id: '12345',
  webhook: {
    type: 'webhook',
    id: 'webhook-123'
  }
}
```

## Related Resources

- [Webhooks](../04-site-configuration/webhook.md) - Configure webhooks
- [Audit Log Events](./audit-log-event.md) - Track all system events
- [Build Events](./build-event.md) - Build trigger events
- [Real-time Events](../../06-advanced-operations/real-time-features/event-filtering.md) - WebSocket events

---

# Webhook Call Methods

This guide covers all methods available for monitoring and managing webhook calls in DatoCMS. Webhook calls represent the execution history of webhooks, including request/response details, status codes, and timing information.

## Available Methods

### list() - List Webhook Call History

Retrieves a paginated list of webhook call records, showing the execution history of webhooks in your project.

```typescript
import { buildClient, WebhookCall } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function listWebhookCalls() {
  try {
    // List all webhook calls
    const allCalls = await client.webhookCalls.list();
    
    // Response structure
    interface WebhookCallListResponse {
      data: WebhookCall[];
      meta: {
        total_count: number;
      };
    }
    
    // Each WebhookCall includes:
    interface WebhookCall {
      id: string;
      type: 'webhook_call';
      attributes: {
        // Request details
        request_url: string;              // URL the webhook was sent to
        request_headers: Record<string, string>; // Headers sent with request
        request_payload: any;             // JSON payload sent
        
        // Response details
        response_status: number;          // HTTP status code received
        response_headers: Record<string, string>; // Response headers
        response_payload: string | null;  // Response body (if any)
        
        // Timing information
        created_at: string;               // When webhook was triggered
        completed_at: string | null;      // When response was received
        response_time_ms: number | null;  // Response time in milliseconds
        
        // Status
        success: boolean;                 // Whether call was successful
        error_message: string | null;     // Error details if failed
      };
      relationships: {
        webhook: {
          data: {
            id: string;
            type: 'webhook';
          };
        };
        entity?: {                        // Related entity (if applicable)
          data: {
            id: string;
            type: string;                 // 'item', 'upload', etc.
          };
        };
      };
    }
    
    console.log(`Found ${allCalls.length} webhook calls`);
    
    // Process calls
    allCalls.forEach(call => {
      const { attributes } = call;
      console.log(`Call ${call.id}:`);
      console.log(`  URL: ${attributes.request_url}`);
      console.log(`  Status: ${attributes.response_status}`);
      console.log(`  Success: ${attributes.success}`);
      console.log(`  Response Time: ${attributes.response_time_ms}ms`);
    });
    
  } catch (error) {
    console.error('Error listing webhook calls:', error);
    throw error;
  }
}

// List with pagination
async function listWebhookCallsPaginated() {
  try {
    // First page
    const firstPage = await client.webhookCalls.list({
      page: {
        offset: 0,
        limit: 30
      }
    });
    
    console.log(`Total calls: ${firstPage.meta.total_count}`);
    console.log(`First page: ${firstPage.data.length} calls`);
    
    // Iterate through all pages
    const allCalls: WebhookCall[] = [];
    let offset = 0;
    const limit = 100;
    
    while (true) {
      const page = await client.webhookCalls.list({
        page: { offset, limit }
      });
      
      allCalls.push(...page.data);
      
      if (page.data.length < limit) {
        break;
      }
      
      offset += limit;
    }
    
    console.log(`Retrieved all ${allCalls.length} calls`);
    return allCalls;
    
  } catch (error) {
    console.error('Error in pagination:', error);
    throw error;
  }
}

// Filter by webhook
async function listCallsForWebhook(webhookId: string) {
  try {
    const calls = await client.webhookCalls.list({
      filter: {
        entity: {
          id: webhookId,
          type: 'webhook'
        }
      }
    });
    
    console.log(`Found ${calls.length} calls for webhook ${webhookId}`);
    return calls;
    
  } catch (error) {
    console.error('Error filtering calls:', error);
    throw error;
  }
}
```

### find() - Get Single Webhook Call Details

Retrieves detailed information about a specific webhook call, including full request and response data.

```typescript
import { buildClient, WebhookCall } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function getWebhookCallDetails(callId: string) {
  try {
    const call = await client.webhookCalls.find(callId);
    
    // Detailed response structure
    interface DetailedWebhookCall extends WebhookCall {
      attributes: {
        // All standard attributes plus:
        request_url: string;
        request_method: 'POST';          // Always POST for webhooks
        request_headers: {
          'Content-Type': string;         // Usually 'application/json'
          'X-DatoCMS-Topic': string;      // Event topic
          'X-DatoCMS-Timestamp': string;  // Event timestamp
          'X-DatoCMS-Signature': string;  // HMAC signature
          [key: string]: string;          // Other headers
        };
        request_payload: {
          event_type: string;             // e.g., 'item.create'
          entity: any;                    // Entity data
          related_entities?: any[];       // Related data
          environment?: string;           // Environment name
        };
        
        // Response details
        response_status: number;
        response_headers: Record<string, string>;
        response_payload: string | null;
        
        // Execution details
        created_at: string;
        completed_at: string | null;
        response_time_ms: number | null;
        
        // Status
        success: boolean;
        error_message: string | null;
        timed_out: boolean;               // If request timed out
      };
    }
    
    // Log detailed information
    console.log('Webhook Call Details:');
    console.log('====================');
    console.log(`ID: ${call.id}`);
    console.log(`URL: ${call.attributes.request_url}`);
    console.log(`Event: ${call.attributes.request_payload.event_type}`);
    console.log(`Status: ${call.attributes.response_status}`);
    console.log(`Success: ${call.attributes.success}`);
    console.log(`Response Time: ${call.attributes.response_time_ms}ms`);
    
    if (!call.attributes.success) {
      console.log(`Error: ${call.attributes.error_message}`);
    }
    
    // Analyze request headers
    console.log('\nRequest Headers:');
    Object.entries(call.attributes.request_headers).forEach(([key, value]) => {
      console.log(`  ${key}: ${value}`);
    });
    
    // Analyze response
    if (call.attributes.response_payload) {
      console.log('\nResponse Body:');
      console.log(call.attributes.response_payload);
    }
    
    return call;
    
  } catch (error) {
    console.error('Error getting webhook call:', error);
    throw error;
  }
}

// Get call with error analysis
async function analyzeFailedCall(callId: string) {
  try {
    const call = await client.webhookCalls.find(callId);
    
    if (call.attributes.success) {
      console.log('Call was successful');
      return;
    }
    
    console.log('Failed Webhook Analysis:');
    console.log('=======================');
    console.log(`Error: ${call.attributes.error_message}`);
    console.log(`Status Code: ${call.attributes.response_status}`);
    console.log(`Timed Out: ${call.attributes.timed_out}`);
    
    // Common error patterns
    if (call.attributes.response_status === 0) {
      console.log('➜ Connection failed (DNS, network, or SSL issue)');
    } else if (call.attributes.response_status >= 500) {
      console.log('➜ Server error on recipient side');
    } else if (call.attributes.response_status >= 400) {
      console.log('➜ Client error (bad request, auth, not found)');
    } else if (call.attributes.timed_out) {
      console.log('➜ Request timed out (30 second limit)');
    }
    
    // Response details for debugging
    if (call.attributes.response_payload) {
      console.log('\nError Response:');
      console.log(call.attributes.response_payload);
    }
    
  } catch (error) {
    console.error('Error analyzing call:', error);
    throw error;
  }
}
```

### resend() - Resend a Webhook Call

Resends a webhook call, useful for retrying failed deliveries or testing webhook endpoints.

```typescript
import { buildClient, WebhookCall } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function resendWebhookCall(callId: string) {
  try {
    // Resend returns a new webhook call
    const newCall = await client.webhookCalls.resend(callId);
    
    // Response is a new WebhookCall with fresh execution
    console.log('Webhook Resent:');
    console.log(`Original Call ID: ${callId}`);
    console.log(`New Call ID: ${newCall.id}`);
    console.log(`Status: ${newCall.attributes.response_status}`);
    console.log(`Success: ${newCall.attributes.success}`);
    
    return newCall;
    
  } catch (error) {
    console.error('Error resending webhook:', error);
    throw error;
  }
}

// Resend with retry logic
async function resendWithRetry(callId: string, maxRetries = 3) {
  let attempt = 0;
  let lastError: Error | null = null;
  
  while (attempt < maxRetries) {
    try {
      attempt++;
      console.log(`Resend attempt ${attempt}/${maxRetries}...`);
      
      const newCall = await client.webhookCalls.resend(callId);
      
      if (newCall.attributes.success) {
        console.log('✓ Webhook delivered successfully');
        return newCall;
      }
      
      console.log(`✗ Attempt ${attempt} failed: ${newCall.attributes.error_message}`);
      lastError = new Error(newCall.attributes.error_message || 'Unknown error');
      
      // Wait before retrying (exponential backoff)
      if (attempt < maxRetries) {
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Waiting ${delay}ms before retry...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
      
    } catch (error) {
      console.error(`Error on attempt ${attempt}:`, error);
      lastError = error as Error;
    }
  }
  
  throw lastError || new Error('All retry attempts failed');
}

// Batch resend failed webhooks
async function resendFailedWebhooks(webhookId: string) {
  try {
    // Get all failed calls for a webhook
    const calls = await client.webhookCalls.list({
      filter: {
        entity: {
          id: webhookId,
          type: 'webhook'
        }
      }
    });
    
    const failedCalls = calls.filter(call => !call.attributes.success);
    console.log(`Found ${failedCalls.length} failed calls to resend`);
    
    const results = {
      successful: 0,
      failed: 0,
      errors: [] as string[]
    };
    
    // Resend each failed call
    for (const call of failedCalls) {
      try {
        const newCall = await client.webhookCalls.resend(call.id);
        
        if (newCall.attributes.success) {
          results.successful++;
        } else {
          results.failed++;
          results.errors.push(newCall.attributes.error_message || 'Unknown error');
        }
        
        // Rate limiting - wait between resends
        await new Promise(resolve => setTimeout(resolve, 100));
        
      } catch (error) {
        results.failed++;
        results.errors.push(error instanceof Error ? error.message : 'Unknown error');
      }
    }
    
    console.log('\nResend Results:');
    console.log(`✓ Successful: ${results.successful}`);
    console.log(`✗ Failed: ${results.failed}`);
    
    if (results.errors.length > 0) {
      console.log('\nErrors:');
      results.errors.forEach((error, i) => {
        console.log(`  ${i + 1}. ${error}`);
      });
    }
    
    return results;
    
  } catch (error) {
    console.error('Error in batch resend:', error);
    throw error;
  }
}
```

## Webhook Debugging Patterns

### Filter by Status

```typescript
// Get only failed webhook calls
async function getFailedWebhooks() {
  try {
    const allCalls = await client.webhookCalls.list();
    
    const failedCalls = allCalls.filter(call => !call.attributes.success);
    
    console.log(`Found ${failedCalls.length} failed webhook calls`);
    
    // Group by error type
    const errorGroups = failedCalls.reduce((groups, call) => {
      const status = call.attributes.response_status;
      const key = status === 0 ? 'connection' : 
                  status >= 500 ? 'server_error' :
                  status >= 400 ? 'client_error' :
                  call.attributes.timed_out ? 'timeout' : 'other';
      
      if (!groups[key]) groups[key] = [];
      groups[key].push(call);
      
      return groups;
    }, {} as Record<string, WebhookCall[]>);
    
    // Report error distribution
    console.log('\nError Distribution:');
    Object.entries(errorGroups).forEach(([type, calls]) => {
      console.log(`  ${type}: ${calls.length} calls`);
    });
    
    return errorGroups;
    
  } catch (error) {
    console.error('Error filtering webhooks:', error);
    throw error;
  }
}

// Get successful webhooks with slow response times
async function getSlowWebhooks(thresholdMs = 5000) {
  try {
    const allCalls = await client.webhookCalls.list();
    
    const slowCalls = allCalls.filter(call => 
      call.attributes.success && 
      call.attributes.response_time_ms !== null &&
      call.attributes.response_time_ms > thresholdMs
    );
    
    console.log(`Found ${slowCalls.length} slow webhook calls (>${thresholdMs}ms)`);
    
    // Sort by response time
    slowCalls.sort((a, b) => 
      (b.attributes.response_time_ms || 0) - (a.attributes.response_time_ms || 0)
    );
    
    // Show top 10 slowest
    console.log('\nTop 10 Slowest Calls:');
    slowCalls.slice(0, 10).forEach((call, i) => {
      console.log(`  ${i + 1}. ${call.attributes.response_time_ms}ms - ${call.attributes.request_url}`);
    });
    
    return slowCalls;
    
  } catch (error) {
    console.error('Error finding slow webhooks:', error);
    throw error;
  }
}
```

### Response Time Analysis

```typescript
// Analyze webhook performance
async function analyzeWebhookPerformance(webhookId: string) {
  try {
    const calls = await client.webhookCalls.list({
      filter: {
        entity: {
          id: webhookId,
          type: 'webhook'
        }
      }
    });
    
    // Filter successful calls with response times
    const successfulCalls = calls.filter(call => 
      call.attributes.success && 
      call.attributes.response_time_ms !== null
    );
    
    if (successfulCalls.length === 0) {
      console.log('No successful calls with timing data');
      return;
    }
    
    // Calculate statistics
    const responseTimes = successfulCalls.map(c => c.attributes.response_time_ms!);
    const stats = {
      count: responseTimes.length,
      min: Math.min(...responseTimes),
      max: Math.max(...responseTimes),
      avg: responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length,
      median: 0,
      p95: 0,
      p99: 0
    };
    
    // Calculate percentiles
    const sorted = [...responseTimes].sort((a, b) => a - b);
    stats.median = sorted[Math.floor(sorted.length / 2)];
    stats.p95 = sorted[Math.floor(sorted.length * 0.95)];
    stats.p99 = sorted[Math.floor(sorted.length * 0.99)];
    
    console.log('Webhook Performance Analysis:');
    console.log('============================');
    console.log(`Total Calls: ${stats.count}`);
    console.log(`Min Response: ${stats.min}ms`);
    console.log(`Max Response: ${stats.max}ms`);
    console.log(`Avg Response: ${stats.avg.toFixed(2)}ms`);
    console.log(`Median: ${stats.median}ms`);
    console.log(`95th Percentile: ${stats.p95}ms`);
    console.log(`99th Percentile: ${stats.p99}ms`);
    
    // Time-based analysis
    const byHour = new Map<number, number[]>();
    
    successfulCalls.forEach(call => {
      const hour = new Date(call.attributes.created_at).getHours();
      if (!byHour.has(hour)) byHour.set(hour, []);
      byHour.get(hour)!.push(call.attributes.response_time_ms!);
    });
    
    console.log('\nAverage Response Time by Hour:');
    Array.from(byHour.entries())
      .sort(([a], [b]) => a - b)
      .forEach(([hour, times]) => {
        const avg = times.reduce((a, b) => a + b, 0) / times.length;
        console.log(`  ${hour.toString().padStart(2, '0')}:00 - ${avg.toFixed(2)}ms (${times.length} calls)`);
      });
    
    return stats;
    
  } catch (error) {
    console.error('Error analyzing performance:', error);
    throw error;
  }
}
```

### Retry Mechanism Patterns

```typescript
// Intelligent retry with backoff and circuit breaker
class WebhookRetryManager {
  private client: any;
  private failureThreshold = 5;
  private resetTimeout = 60000; // 1 minute
  private circuitState: Map<string, {
    failures: number;
    lastFailure: Date;
    isOpen: boolean;
  }> = new Map();
  
  constructor(client: any) {
    this.client = client;
  }
  
  async retryFailedWebhooks() {
    try {
      const calls = await this.client.webhookCalls.list();
      const failedCalls = calls.filter((c: WebhookCall) => !c.attributes.success);
      
      console.log(`Processing ${failedCalls.length} failed webhooks`);
      
      for (const call of failedCalls) {
        const url = call.attributes.request_url;
        
        // Check circuit breaker
        if (this.isCircuitOpen(url)) {
          console.log(`Circuit open for ${url}, skipping`);
          continue;
        }
        
        try {
          const newCall = await this.retryWithBackoff(call.id);
          
          if (newCall.attributes.success) {
            this.recordSuccess(url);
            console.log(`✓ Successfully delivered to ${url}`);
          } else {
            this.recordFailure(url);
            console.log(`✗ Failed to deliver to ${url}`);
          }
          
        } catch (error) {
          this.recordFailure(url);
          console.error(`Error retrying ${call.id}:`, error);
        }
      }
      
    } catch (error) {
      console.error('Error in retry manager:', error);
      throw error;
    }
  }
  
  private async retryWithBackoff(callId: string, attempt = 1): Promise<WebhookCall> {
    const maxAttempts = 3;
    const baseDelay = 1000;
    
    try {
      const newCall = await this.client.webhookCalls.resend(callId);
      return newCall;
      
    } catch (error) {
      if (attempt >= maxAttempts) {
        throw error;
      }
      
      // Exponential backoff with jitter
      const delay = baseDelay * Math.pow(2, attempt - 1) + Math.random() * 1000;
      console.log(`Waiting ${delay.toFixed(0)}ms before retry ${attempt + 1}/${maxAttempts}`);
      
      await new Promise(resolve => setTimeout(resolve, delay));
      return this.retryWithBackoff(callId, attempt + 1);
    }
  }
  
  private isCircuitOpen(url: string): boolean {
    const state = this.circuitState.get(url);
    if (!state) return false;
    
    // Check if circuit should be reset
    const timeSinceLastFailure = Date.now() - state.lastFailure.getTime();
    if (state.isOpen && timeSinceLastFailure > this.resetTimeout) {
      console.log(`Resetting circuit for ${url}`);
      state.isOpen = false;
      state.failures = 0;
    }
    
    return state.isOpen;
  }
  
  private recordFailure(url: string) {
    const state = this.circuitState.get(url) || {
      failures: 0,
      lastFailure: new Date(),
      isOpen: false
    };
    
    state.failures++;
    state.lastFailure = new Date();
    
    if (state.failures >= this.failureThreshold) {
      state.isOpen = true;
      console.log(`Circuit opened for ${url} after ${state.failures} failures`);
    }
    
    this.circuitState.set(url, state);
  }
  
  private recordSuccess(url: string) {
    const state = this.circuitState.get(url);
    if (state) {
      state.failures = 0;
      state.isOpen = false;
    }
  }
}

// Usage
async function intelligentRetry() {
  const retryManager = new WebhookRetryManager(client);
  await retryManager.retryFailedWebhooks();
}
```

## Error Handling Examples

```typescript
// Comprehensive error handling
async function handleWebhookErrors() {
  try {
    const calls = await client.webhookCalls.list();
    
    // Categorize errors
    const errors = {
      connection: [] as WebhookCall[],
      timeout: [] as WebhookCall[],
      authentication: [] as WebhookCall[],
      serverError: [] as WebhookCall[],
      clientError: [] as WebhookCall[],
      other: [] as WebhookCall[]
    };
    
    calls.filter(c => !c.attributes.success).forEach(call => {
      const status = call.attributes.response_status;
      const message = call.attributes.error_message || '';
      
      if (status === 0 || message.includes('connect')) {
        errors.connection.push(call);
      } else if (call.attributes.timed_out) {
        errors.timeout.push(call);
      } else if (status === 401 || status === 403) {
        errors.authentication.push(call);
      } else if (status >= 500) {
        errors.serverError.push(call);
      } else if (status >= 400) {
        errors.clientError.push(call);
      } else {
        errors.other.push(call);
      }
    });
    
    // Generate remediation suggestions
    console.log('Error Analysis and Remediation:');
    console.log('==============================');
    
    if (errors.connection.length > 0) {
      console.log(`\nConnection Errors (${errors.connection.length}):`);
      console.log('  • Check if webhook URL is accessible');
      console.log('  • Verify DNS resolution');
      console.log('  • Check SSL certificate validity');
      console.log('  • Ensure firewall allows DatoCMS IPs');
    }
    
    if (errors.timeout.length > 0) {
      console.log(`\nTimeout Errors (${errors.timeout.length}):`);
      console.log('  • Webhook must respond within 30 seconds');
      console.log('  • Consider async processing');
      console.log('  • Return 200 immediately, process in background');
    }
    
    if (errors.authentication.length > 0) {
      console.log(`\nAuthentication Errors (${errors.authentication.length}):`);
      console.log('  • Verify webhook secret/token');
      console.log('  • Check authentication headers');
      console.log('  • Ensure credentials haven\'t expired');
    }
    
    if (errors.serverError.length > 0) {
      console.log(`\nServer Errors (${errors.serverError.length}):`);
      console.log('  • Check server logs for errors');
      console.log('  • Monitor server resources');
      console.log('  • Implement proper error handling');
    }
    
    if (errors.clientError.length > 0) {
      console.log(`\nClient Errors (${errors.clientError.length}):`);
      console.log('  • Verify webhook expects correct payload format');
      console.log('  • Check for required headers');
      console.log('  • Ensure endpoint path is correct');
    }
    
    return errors;
    
  } catch (error) {
    console.error('Error analyzing webhook errors:', error);
    throw error;
  }
}

// Detailed error diagnosis
async function diagnoseWebhookCall(callId: string) {
  try {
    const call = await client.webhookCalls.find(callId);
    
    console.log('Webhook Call Diagnosis:');
    console.log('======================');
    
    // Basic info
    console.log(`\nCall ID: ${call.id}`);
    console.log(`URL: ${call.attributes.request_url}`);
    console.log(`Created: ${call.attributes.created_at}`);
    console.log(`Status: ${call.attributes.response_status}`);
    
    // Diagnose based on status
    if (call.attributes.success) {
      console.log('\n✓ Call was successful');
      console.log(`Response time: ${call.attributes.response_time_ms}ms`);
      
      if (call.attributes.response_time_ms! > 10000) {
        console.log('⚠️  Warning: Slow response time (>10s)');
      }
    } else {
      console.log('\n✗ Call failed');
      console.log(`Error: ${call.attributes.error_message}`);
      
      // Specific diagnosis
      const status = call.attributes.response_status;
      
      if (status === 0) {
        console.log('\nDiagnosis: Connection Failed');
        console.log('Possible causes:');
        console.log('  • Invalid URL or domain');
        console.log('  • DNS resolution failure');
        console.log('  • Network connectivity issue');
        console.log('  • SSL/TLS handshake failure');
        console.log('  • Firewall blocking connection');
      } else if (status === 401) {
        console.log('\nDiagnosis: Authentication Failed');
        console.log('Possible causes:');
        console.log('  • Missing or invalid API key');
        console.log('  • Incorrect webhook secret');
        console.log('  • Expired credentials');
      } else if (status === 404) {
        console.log('\nDiagnosis: Endpoint Not Found');
        console.log('Possible causes:');
        console.log('  • Incorrect URL path');
        console.log('  • Endpoint was removed');
        console.log('  • Wrong HTTP method expected');
      } else if (status === 500) {
        console.log('\nDiagnosis: Server Error');
        console.log('Possible causes:');
        console.log('  • Application error on server');
        console.log('  • Database connection issue');
        console.log('  • Out of memory or resources');
      } else if (call.attributes.timed_out) {
        console.log('\nDiagnosis: Request Timeout');
        console.log('Possible causes:');
        console.log('  • Slow server response');
        console.log('  • Long-running synchronous operation');
        console.log('  • Network latency');
      }
      
      // Remediation steps
      console.log('\nRecommended Actions:');
      console.log('1. Check server logs at the time of failure');
      console.log('2. Test webhook endpoint manually with curl');
      console.log('3. Verify webhook configuration in DatoCMS');
      console.log('4. Consider implementing retry logic');
    }
    
    // Show request details for debugging
    console.log('\nRequest Details:');
    console.log('Headers:', JSON.stringify(call.attributes.request_headers, null, 2));
    console.log('Payload:', JSON.stringify(call.attributes.request_payload, null, 2));
    
    if (call.attributes.response_payload) {
      console.log('\nResponse Body:');
      console.log(call.attributes.response_payload);
    }
    
  } catch (error) {
    console.error('Error diagnosing webhook call:', error);
    throw error;
  }
}
```

## Best Practices for Webhook Monitoring

### 1. Implement Health Monitoring

```typescript
// Monitor webhook health over time
async function monitorWebhookHealth(webhookId: string, hours = 24) {
  try {
    const since = new Date();
    since.setHours(since.getHours() - hours);
    
    const calls = await client.webhookCalls.list({
      filter: {
        entity: {
          id: webhookId,
          type: 'webhook'
        }
      }
    });
    
    // Filter calls within time window
    const recentCalls = calls.filter(call => 
      new Date(call.attributes.created_at) > since
    );
    
    if (recentCalls.length === 0) {
      console.log('No webhook calls in the specified time period');
      return;
    }
    
    // Calculate health metrics
    const metrics = {
      total: recentCalls.length,
      successful: recentCalls.filter(c => c.attributes.success).length,
      failed: recentCalls.filter(c => !c.attributes.success).length,
      successRate: 0,
      avgResponseTime: 0,
      maxResponseTime: 0,
      errors: new Map<string, number>()
    };
    
    metrics.successRate = (metrics.successful / metrics.total) * 100;
    
    // Calculate response times for successful calls
    const responseTimes = recentCalls
      .filter(c => c.attributes.success && c.attributes.response_time_ms)
      .map(c => c.attributes.response_time_ms!);
    
    if (responseTimes.length > 0) {
      metrics.avgResponseTime = responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length;
      metrics.maxResponseTime = Math.max(...responseTimes);
    }
    
    // Count error types
    recentCalls
      .filter(c => !c.attributes.success)
      .forEach(call => {
        const error = call.attributes.error_message || 'Unknown error';
        metrics.errors.set(error, (metrics.errors.get(error) || 0) + 1);
      });
    
    // Generate health report
    console.log(`Webhook Health Report (Last ${hours} hours):`);
    console.log('=========================================');
    console.log(`Total Calls: ${metrics.total}`);
    console.log(`Success Rate: ${metrics.successRate.toFixed(2)}%`);
    console.log(`Failed Calls: ${metrics.failed}`);
    
    if (responseTimes.length > 0) {
      console.log(`\nPerformance:`);
      console.log(`  Average Response: ${metrics.avgResponseTime.toFixed(2)}ms`);
      console.log(`  Max Response: ${metrics.maxResponseTime}ms`);
    }
    
    if (metrics.errors.size > 0) {
      console.log(`\nError Distribution:`);
      Array.from(metrics.errors.entries())
        .sort(([, a], [, b]) => b - a)
        .forEach(([error, count]) => {
          console.log(`  ${error}: ${count} occurrences`);
        });
    }
    
    // Health status
    console.log('\nHealth Status:');
    if (metrics.successRate >= 99) {
      console.log('✓ Excellent - 99%+ success rate');
    } else if (metrics.successRate >= 95) {
      console.log('⚠️  Good - 95%+ success rate');
    } else if (metrics.successRate >= 90) {
      console.log('⚠️  Fair - 90%+ success rate');
    } else {
      console.log('✗ Poor - Below 90% success rate');
    }
    
    return metrics;
    
  } catch (error) {
    console.error('Error monitoring webhook health:', error);
    throw error;
  }
}
```

### 2. Set Up Alerting

```typescript
// Alert on webhook failures
async function setupWebhookAlerting() {
  try {
    // Check recent failures
    const recentCalls = await client.webhookCalls.list({
      page: { limit: 100 }
    });
    
    const recentFailures = recentCalls.filter(c => 
      !c.attributes.success &&
      new Date(c.attributes.created_at) > new Date(Date.now() - 3600000) // Last hour
    );
    
    if (recentFailures.length > 5) {
      console.log('⚠️  ALERT: High webhook failure rate detected!');
      console.log(`${recentFailures.length} failures in the last hour`);
      
      // Group by URL for analysis
      const failuresByUrl = new Map<string, number>();
      recentFailures.forEach(call => {
        const url = call.attributes.request_url;
        failuresByUrl.set(url, (failuresByUrl.get(url) || 0) + 1);
      });
      
      console.log('\nAffected endpoints:');
      failuresByUrl.forEach((count, url) => {
        console.log(`  ${url}: ${count} failures`);
      });
      
      // Trigger your alerting mechanism here
      // e.g., send email, Slack notification, PagerDuty, etc.
    }
    
  } catch (error) {
    console.error('Error in webhook alerting:', error);
    throw error;
  }
}
```

### 3. Implement Recovery Strategies

```typescript
// Automated recovery for webhook failures
async function implementWebhookRecovery() {
  const strategies = {
    // Strategy 1: Immediate retry for transient failures
    async handleTransientFailures() {
      const recentCalls = await client.webhookCalls.list({
        page: { limit: 50 }
      });
      
      const transientFailures = recentCalls.filter(call => 
        !call.attributes.success &&
        (call.attributes.response_status >= 500 || call.attributes.response_status === 0)
      );
      
      for (const call of transientFailures) {
        try {
          const result = await client.webhookCalls.resend(call.id);
          if (result.attributes.success) {
            console.log(`✓ Recovered transient failure for ${call.id}`);
          }
        } catch (error) {
          console.error(`Failed to recover ${call.id}:`, error);
        }
      }
    },
    
    // Strategy 2: Disable failing webhooks
    async disableConsistentlyFailingWebhooks() {
      const webhookFailures = new Map<string, number>();
      
      const calls = await client.webhookCalls.list({
        page: { limit: 1000 }
      });
      
      // Count failures per webhook
      calls.filter(c => !c.attributes.success).forEach(call => {
        if (call.relationships.webhook) {
          const webhookId = call.relationships.webhook.data.id;
          webhookFailures.set(webhookId, (webhookFailures.get(webhookId) || 0) + 1);
        }
      });
      
      // Disable webhooks with too many failures
      for (const [webhookId, failures] of webhookFailures.entries()) {
        if (failures > 50) {
          console.log(`Disabling webhook ${webhookId} due to ${failures} failures`);
          // Implement webhook disabling logic here
        }
      }
    }
  };
  
  // Run recovery strategies
  await strategies.handleTransientFailures();
  await strategies.disableConsistentlyFailingWebhooks();
}
```

## Failed Webhook Recovery

```typescript
// Complete failed webhook recovery workflow
async function recoverFailedWebhooks() {
  try {
    // Step 1: Identify failed webhooks
    const allCalls = await client.webhookCalls.list();
    const failedCalls = allCalls.filter(c => !c.attributes.success);
    
    console.log(`Found ${failedCalls.length} failed webhook calls`);
    
    // Step 2: Group by recoverability
    const recoverable = {
      retry: [] as WebhookCall[],      // Can retry immediately
      fixRequired: [] as WebhookCall[], // Needs fix before retry
      permanent: [] as WebhookCall[]    // Cannot recover
    };
    
    failedCalls.forEach(call => {
      const status = call.attributes.response_status;
      
      if (status >= 500 || status === 0 || call.attributes.timed_out) {
        recoverable.retry.push(call);
      } else if (status === 401 || status === 403 || status === 404) {
        recoverable.fixRequired.push(call);
      } else {
        recoverable.permanent.push(call);
      }
    });
    
    console.log(`\nRecovery Analysis:`);
    console.log(`  Can retry: ${recoverable.retry.length}`);
    console.log(`  Needs fix: ${recoverable.fixRequired.length}`);
    console.log(`  Permanent failures: ${recoverable.permanent.length}`);
    
    // Step 3: Attempt recovery
    const results = {
      recovered: 0,
      failed: 0
    };
    
    for (const call of recoverable.retry) {
      try {
        console.log(`Retrying ${call.id}...`);
        const newCall = await client.webhookCalls.resend(call.id);
        
        if (newCall.attributes.success) {
          results.recovered++;
          console.log(`  ✓ Success`);
        } else {
          results.failed++;
          console.log(`  ✗ Failed: ${newCall.attributes.error_message}`);
        }
        
        // Rate limit
        await new Promise(resolve => setTimeout(resolve, 200));
        
      } catch (error) {
        results.failed++;
        console.error(`  ✗ Error: ${error}`);
      }
    }
    
    console.log(`\nRecovery Results:`);
    console.log(`  Recovered: ${results.recovered}`);
    console.log(`  Still failed: ${results.failed}`);
    
    // Step 4: Generate fix recommendations
    if (recoverable.fixRequired.length > 0) {
      console.log(`\nWebhooks requiring fixes:`);
      
      const fixGroups = recoverable.fixRequired.reduce((groups, call) => {
        const key = `${call.attributes.response_status}-${call.attributes.request_url}`;
        if (!groups[key]) groups[key] = [];
        groups[key].push(call);
        return groups;
      }, {} as Record<string, WebhookCall[]>);
      
      Object.entries(fixGroups).forEach(([key, calls]) => {
        const [status, url] = key.split('-');
        console.log(`\n  ${url} (Status ${status}):`);
        console.log(`    Failures: ${calls.length}`);
        
        if (status === '401' || status === '403') {
          console.log(`    Fix: Update authentication credentials`);
        } else if (status === '404') {
          console.log(`    Fix: Update webhook URL or restore endpoint`);
        }
      });
    }
    
    return results;
    
  } catch (error) {
    console.error('Error in webhook recovery:', error);
    throw error;
  }
}
```

## Related Resources

- [Webhook Documentation](./webhook.md) - Configure and manage webhooks
- [Audit Log Events](./audit-log-event.md) - Track webhook configuration changes
- [Build Events](./build-event.md) - Monitor build trigger webhooks
- [Site Configuration](../04-site-configuration/site.md) - Environment-specific webhook settings
- [Error Handling Patterns](../../04-advanced-topics/error-handling-patterns.md) - General error handling strategies
- [REST API Events](../../04-advanced-topics/rest-api-events.md) - Real-time webhook monitoring

## Common Use Cases

1. **Debugging Failed Deliveries** - Analyze why webhooks are failing and implement fixes
2. **Performance Monitoring** - Track response times and identify slow endpoints
3. **Reliability Tracking** - Monitor success rates and uptime
4. **Automated Recovery** - Resend failed webhooks automatically
5. **Audit Trail** - Maintain history of all webhook executions
6. **Integration Testing** - Verify webhook payloads and responses
7. **Capacity Planning** - Analyze webhook volume and patterns