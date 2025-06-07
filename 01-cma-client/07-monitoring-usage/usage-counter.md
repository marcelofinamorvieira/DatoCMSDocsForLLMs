# UsageCounter

The UsageCounter resource provides real-time, granular analytics for tracking API usage and resource consumption in your DatoCMS project. Unlike daily aggregated statistics, usage counters offer per-occurrence metrics updated every minute, making them ideal for debugging, optimization, and real-time monitoring.

## Available Operations

### Find Usage Counter

```javascript
const counter = await client.usageCounters.find(counterId, queryParams);
```

**Parameters:**
- `counterId` (string, required): The type of counter to retrieve (see counter types below)
- `queryParams` (object, optional):
  - `period`: Time period filter - `'today'`, `'current_month'`, or `'last_month'`

**Returns:** A UsageCounter object with occurrence counts

## The UsageCounter Object

```typescript
{
  id: string;                // The counter type
  type: 'usage_counter';     // Always 'usage_counter'
  result: Array<{
    value: string;           // The specific occurrence (IP, path, user ID, etc.)
    counter: number;         // Count for that occurrence
  }>
}
```

## Counter Types

### Asset Usage Counters

| Counter ID | Description | Value Type |
|------------|-------------|------------|
| `assets_path_bytes` | Bytes transferred per asset path | Asset path |
| `assets_referrer_bytes` | Bytes transferred per referrer | Referrer URL |
| `assets_ip_bytes` | Bytes transferred per IP | IP address |
| `assets_full_path_bytes` | Bytes per full asset path | Full asset URL |
| `assets_path_requests` | Requests per asset path | Asset path |
| `assets_full_path_requests` | Requests per full path | Full asset URL |

### Content Delivery API (CDA) Counters

| Counter ID | Description | Value Type |
|------------|-------------|------------|
| `cda_access_token_id_bytes` | Bytes per access token | Access token ID |
| `cda_access_token_id_requests` | Requests per access token | Access token ID |
| `cda_referrer_bytes` | Bytes per referrer | Referrer URL |
| `cda_referrer_requests` | Requests per referrer | Referrer URL |
| `cda_ip_bytes` | Bytes per IP address | IP address |
| `cda_ip_requests` | Requests per IP address | IP address |

### Content Management API (CMA) Counters

| Counter ID | Description | Value Type |
|------------|-------------|------------|
| `cma_endpoint_bytes` | Bytes per API endpoint | Endpoint path |
| `cma_endpoint_requests` | Requests per API endpoint | Endpoint path |
| `cma_user_bytes` | Bytes per user | User ID |
| `cma_user_requests` | Requests per user | User ID |
| `cma_ip_bytes` | Bytes per IP address | IP address |
| `cma_ip_requests` | Requests per IP address | IP address |

### Video Usage Counter

| Counter ID | Description | Value Type |
|------------|-------------|------------|
| `video_path_seconds` | Streaming seconds per video | Video path |

## Common Use Cases

### 1. Debug API Quota Issues

```javascript
// Find which IPs are making the most CMA requests today
const ipRequests = await client.usageCounters.find('cma_ip_requests', {
  period: 'today'
});

// Sort by highest usage
const topOffenders = ipRequests.result
  .sort((a, b) => b.counter - a.counter)
  .slice(0, 10);

console.log('Top 10 IPs by request count:');
topOffenders.forEach(({ value, counter }) => {
  console.log(`${value}: ${counter.toLocaleString()} requests`);
});
```

### 2. Analyze Endpoint Usage

```javascript
// Get CMA endpoint usage for current month
const endpointUsage = await client.usageCounters.find('cma_endpoint_requests', {
  period: 'current_month'
});

// Create endpoint report
const report = endpointUsage.result
  .sort((a, b) => b.counter - a.counter)
  .map(({ value, counter }) => ({
    endpoint: value,
    requests: counter,
    percentage: ((counter / endpointUsage.result.reduce((sum, e) => sum + e.counter, 0)) * 100).toFixed(2)
  }));

console.table(report);
```

### 3. Monitor Asset Bandwidth

```javascript
// Track which assets consume the most bandwidth
const assetBandwidth = await client.usageCounters.find('assets_path_bytes', {
  period: 'current_month'
});

// Convert bytes to GB and sort
const heavyAssets = assetBandwidth.result
  .map(({ value, counter }) => ({
    path: value,
    gigabytes: (counter / (1024 ** 3)).toFixed(2),
    bytes: counter
  }))
  .sort((a, b) => b.bytes - a.bytes)
  .slice(0, 20);

console.log('Top 20 assets by bandwidth:');
heavyAssets.forEach(asset => {
  console.log(`${asset.path}: ${asset.gigabytes} GB`);
});
```

### 4. Access Token Analysis

```javascript
// Monitor CDA access token usage
const tokenUsage = await client.usageCounters.find('cda_access_token_id_requests', {
  period: 'current_month'
});

// Group by access token
for (const { value: tokenId, counter } of tokenUsage.result) {
  // Fetch token details
  const token = await client.accessTokens.find(tokenId);
  console.log(`Token "${token.name}": ${counter.toLocaleString()} requests`);
}
```

### 5. Real-time Abuse Detection

```javascript
// Check for potential abuse patterns
async function detectAbusePatterns() {
  // Check IP request patterns
  const ipRequests = await client.usageCounters.find('cma_ip_requests', {
    period: 'today'
  });
  
  const THRESHOLD = 10000; // Daily request threshold per IP
  
  const suspiciousIPs = ipRequests.result
    .filter(({ counter }) => counter > THRESHOLD)
    .sort((a, b) => b.counter - a.counter);
  
  if (suspiciousIPs.length > 0) {
    console.warn('⚠️ Suspicious activity detected:');
    suspiciousIPs.forEach(({ value, counter }) => {
      console.warn(`IP ${value}: ${counter.toLocaleString()} requests today`);
    });
  }
  
  return suspiciousIPs;
}
```

### 6. Video Usage Tracking

```javascript
// Monitor video streaming usage
const videoUsage = await client.usageCounters.find('video_path_seconds', {
  period: 'current_month'
});

// Calculate viewing hours per video
const videoReport = videoUsage.result
  .map(({ value, counter }) => ({
    video: value,
    hours: (counter / 3600).toFixed(2),
    views: Math.ceil(counter / 180) // Estimate views (3 min average)
  }))
  .sort((a, b) => parseFloat(b.hours) - parseFloat(a.hours));

console.log('Video streaming report:');
videoReport.forEach(({ video, hours, views }) => {
  console.log(`${video}: ${hours} hours (~${views} views)`);
});
```

### 7. Referrer Analysis

```javascript
// Analyze CDA traffic sources
const referrers = await client.usageCounters.find('cda_referrer_requests', {
  period: 'current_month'
});

// Group by domain
const domainStats = {};
referrers.result.forEach(({ value, counter }) => {
  try {
    const domain = new URL(value).hostname;
    domainStats[domain] = (domainStats[domain] || 0) + counter;
  } catch (e) {
    domainStats['direct'] = (domainStats['direct'] || 0) + counter;
  }
});

console.log('Traffic sources:');
Object.entries(domainStats)
  .sort(([,a], [,b]) => b - a)
  .forEach(([domain, count]) => {
    console.log(`${domain}: ${count.toLocaleString()} requests`);
  });
```

## Time Periods

Usage counters support three time period filters:

- **`today`**: Current day's data (updates every minute)
- **`current_month`**: Current calendar month
- **`last_month`**: Previous calendar month

```javascript
// Compare this month vs last month
const thisMonth = await client.usageCounters.find('cma_endpoint_requests', {
  period: 'current_month'
});

const lastMonth = await client.usageCounters.find('cma_endpoint_requests', {
  period: 'last_month'
});

// Calculate growth
const thisMonthTotal = thisMonth.result.reduce((sum, e) => sum + e.counter, 0);
const lastMonthTotal = lastMonth.result.reduce((sum, e) => sum + e.counter, 0);
const growth = ((thisMonthTotal - lastMonthTotal) / lastMonthTotal * 100).toFixed(1);

console.log(`API usage growth: ${growth}%`);
```

## Important Notes

1. **Update Frequency**: Counters are updated every minute
2. **Data Retention**: Historical data availability depends on your plan
3. **Value Limits**: Each counter returns top occurrences by count
4. **Performance**: These are read-only operations with minimal impact
5. **Time Zone**: All time periods are in UTC

## Raw Response Access

For accessing the raw API response:

```javascript
const response = await client.usageCounters.rawFind('cma_endpoint_requests', {
  period: 'today'
});

console.log(response.data); // Raw counter data
console.log(response.meta); // Response metadata
```

## Related Resources

- [Daily Usage](/01-cma-client/07-monitoring-usage/daily-usage.md) - Aggregated daily statistics
- [Subscription Limits](/01-cma-client/07-monitoring-usage/subscription-limit.md) - View plan limits
- [Access Tokens](/01-cma-client/05-access-control/access-token.md) - Manage API access tokens

---

# Usage Counter Methods

The Usage Counter resource provides methods for retrieving real-time, granular analytics about API usage and resource consumption in your DatoCMS project. This document covers all available methods for working with usage counters.

## Table of Contents
- [find() - Get Current Usage Counters](#find---get-current-usage-counters)
- [Available Counter Types](#available-counter-types)
- [Usage Tracking Patterns](#usage-tracking-patterns)
- [Limit Checking Workflows](#limit-checking-workflows)
- [Usage Alerts Implementation](#usage-alerts-implementation)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)
- [Related Resources](#related-resources)

## find() - Get Current Usage Counters

Retrieves usage counter data for a specific counter type and time period.

### Basic Syntax

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get usage counter data
const counter = await client.usageCounters.find(counterId, queryParams);
```

### Parameters

```typescript
// Method signature
find(
  counterId: string,                    // Required: The counter type to retrieve
  queryParams?: {                       // Optional: Query parameters
    period?: 'today' | 'current_month' | 'last_month';  // Time period filter
  }
): Promise<UsageCounter>
```

### Response Structure

```typescript
interface UsageCounter {
  id: string;                          // The counter type identifier
  type: 'usage_counter';               // Resource type (always 'usage_counter')
  result: Array<{
    value: string;                     // The specific occurrence (IP, path, user ID, etc.)
    counter: number;                   // Count for that occurrence
  }>;
}
```

### Complete Example with All Options

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function getUsageCounterData() {
  try {
    // Get CMA endpoint requests for today
    const endpointUsageToday = await client.usageCounters.find(
      'cma_endpoint_requests',
      { period: 'today' }
    );
    
    console.log('CMA Endpoint Usage Today:');
    console.log(`Counter Type: ${endpointUsageToday.id}`);
    console.log(`Total Endpoints: ${endpointUsageToday.result.length}`);
    
    // Display top 10 endpoints by request count
    const topEndpoints = endpointUsageToday.result
      .sort((a, b) => b.counter - a.counter)
      .slice(0, 10);
    
    topEndpoints.forEach(({ value, counter }, index) => {
      console.log(`${index + 1}. ${value}: ${counter.toLocaleString()} requests`);
    });
    
    // Get current month's data for comparison
    const endpointUsageMonth = await client.usageCounters.find(
      'cma_endpoint_requests',
      { period: 'current_month' }
    );
    
    const monthTotal = endpointUsageMonth.result.reduce(
      (sum, item) => sum + item.counter, 
      0
    );
    console.log(`\nTotal requests this month: ${monthTotal.toLocaleString()}`);
    
  } catch (error) {
    console.error('Error fetching usage counter:', error);
  }
}

getUsageCounterData();
```

## Available Counter Types

### Asset Usage Counters

```typescript
// Track asset bandwidth by path
const assetBandwidth = await client.usageCounters.find('assets_path_bytes', {
  period: 'current_month'
});

// Example response structure
{
  id: 'assets_path_bytes',
  type: 'usage_counter',
  result: [
    { value: '/uploads/123/hero-image.jpg', counter: 524288000 },  // 500 MB
    { value: '/uploads/456/product-video.mp4', counter: 1073741824 }, // 1 GB
    // ... more asset paths
  ]
}

// Available asset counters:
const assetCounters = [
  'assets_path_bytes',          // Bytes transferred per asset path
  'assets_referrer_bytes',      // Bytes transferred per referrer
  'assets_ip_bytes',            // Bytes transferred per IP
  'assets_full_path_bytes',     // Bytes per full asset path
  'assets_path_requests',       // Requests per asset path
  'assets_full_path_requests'   // Requests per full path
];
```

### Content Delivery API (CDA) Counters

```typescript
// Track CDA usage by access token
const tokenUsage = await client.usageCounters.find('cda_access_token_id_requests', {
  period: 'today'
});

// Example response structure
{
  id: 'cda_access_token_id_requests',
  type: 'usage_counter',
  result: [
    { value: 'abc123', counter: 15420 },  // Token ID with request count
    { value: 'def456', counter: 8932 },
    // ... more access tokens
  ]
}

// Available CDA counters:
const cdaCounters = [
  'cda_access_token_id_bytes',     // Bytes per access token
  'cda_access_token_id_requests',  // Requests per access token
  'cda_referrer_bytes',            // Bytes per referrer
  'cda_referrer_requests',         // Requests per referrer
  'cda_ip_bytes',                  // Bytes per IP address
  'cda_ip_requests'                // Requests per IP address
];
```

### Content Management API (CMA) Counters

```typescript
// Track CMA usage by user
const userActivity = await client.usageCounters.find('cma_user_requests', {
  period: 'current_month'
});

// Example response structure
{
  id: 'cma_user_requests',
  type: 'usage_counter',
  result: [
    { value: '12345', counter: 4521 },  // User ID with request count
    { value: '67890', counter: 2156 },
    // ... more users
  ]
}

// Available CMA counters:
const cmaCounters = [
  'cma_endpoint_bytes',      // Bytes per API endpoint
  'cma_endpoint_requests',   // Requests per API endpoint
  'cma_user_bytes',         // Bytes per user
  'cma_user_requests',      // Requests per user
  'cma_ip_bytes',           // Bytes per IP address
  'cma_ip_requests'         // Requests per IP address
];
```

### Video Usage Counter

```typescript
// Track video streaming time
const videoUsage = await client.usageCounters.find('video_path_seconds', {
  period: 'current_month'
});

// Example response structure
{
  id: 'video_path_seconds',
  type: 'usage_counter',
  result: [
    { value: '/stream/123/product-demo.mp4', counter: 86400 },  // 24 hours
    { value: '/stream/456/tutorial.mp4', counter: 43200 },      // 12 hours
    // ... more videos
  ]
}
```

## Usage Tracking Patterns

### Pattern 1: Comprehensive Usage Dashboard

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

interface UsageDashboard {
  api: {
    cma: { requests: number; bytes: number; topEndpoints: Array<{endpoint: string; requests: number}> };
    cda: { requests: number; bytes: number; topTokens: Array<{tokenId: string; requests: number}> };
  };
  assets: {
    totalBytes: number;
    totalRequests: number;
    topAssets: Array<{path: string; bytes: number}>;
  };
  videos: {
    totalSeconds: number;
    topVideos: Array<{path: string; hours: number}>;
  };
}

async function generateUsageDashboard(period: 'today' | 'current_month' | 'last_month'): Promise<UsageDashboard> {
  // Fetch all counters in parallel
  const [
    cmaRequests,
    cmaBytes,
    cdaRequests,
    cdaBytes,
    assetBytes,
    assetRequests,
    videoSeconds
  ] = await Promise.all([
    client.usageCounters.find('cma_endpoint_requests', { period }),
    client.usageCounters.find('cma_endpoint_bytes', { period }),
    client.usageCounters.find('cda_access_token_id_requests', { period }),
    client.usageCounters.find('cda_access_token_id_bytes', { period }),
    client.usageCounters.find('assets_path_bytes', { period }),
    client.usageCounters.find('assets_path_requests', { period }),
    client.usageCounters.find('video_path_seconds', { period })
  ]);
  
  // Calculate totals and prepare dashboard
  const dashboard: UsageDashboard = {
    api: {
      cma: {
        requests: cmaRequests.result.reduce((sum, item) => sum + item.counter, 0),
        bytes: cmaBytes.result.reduce((sum, item) => sum + item.counter, 0),
        topEndpoints: cmaRequests.result
          .sort((a, b) => b.counter - a.counter)
          .slice(0, 5)
          .map(({ value, counter }) => ({ endpoint: value, requests: counter }))
      },
      cda: {
        requests: cdaRequests.result.reduce((sum, item) => sum + item.counter, 0),
        bytes: cdaBytes.result.reduce((sum, item) => sum + item.counter, 0),
        topTokens: cdaRequests.result
          .sort((a, b) => b.counter - a.counter)
          .slice(0, 5)
          .map(({ value, counter }) => ({ tokenId: value, requests: counter }))
      }
    },
    assets: {
      totalBytes: assetBytes.result.reduce((sum, item) => sum + item.counter, 0),
      totalRequests: assetRequests.result.reduce((sum, item) => sum + item.counter, 0),
      topAssets: assetBytes.result
        .sort((a, b) => b.counter - a.counter)
        .slice(0, 5)
        .map(({ value, counter }) => ({ path: value, bytes: counter }))
    },
    videos: {
      totalSeconds: videoSeconds.result.reduce((sum, item) => sum + item.counter, 0),
      topVideos: videoSeconds.result
        .sort((a, b) => b.counter - a.counter)
        .slice(0, 5)
        .map(({ value, counter }) => ({ 
          path: value, 
          hours: parseFloat((counter / 3600).toFixed(2)) 
        }))
    }
  };
  
  return dashboard;
}

// Usage
const dashboard = await generateUsageDashboard('current_month');
console.log('Usage Dashboard:', JSON.stringify(dashboard, null, 2));
```

### Pattern 2: Historical Usage Comparison

```typescript
async function compareUsagePeriods(counterId: string) {
  const [today, currentMonth, lastMonth] = await Promise.all([
    client.usageCounters.find(counterId, { period: 'today' }),
    client.usageCounters.find(counterId, { period: 'current_month' }),
    client.usageCounters.find(counterId, { period: 'last_month' })
  ]);
  
  const todayTotal = today.result.reduce((sum, item) => sum + item.counter, 0);
  const currentMonthTotal = currentMonth.result.reduce((sum, item) => sum + item.counter, 0);
  const lastMonthTotal = lastMonth.result.reduce((sum, item) => sum + item.counter, 0);
  
  // Calculate daily average for current month
  const daysInCurrentMonth = new Date().getDate();
  const dailyAverage = currentMonthTotal / daysInCurrentMonth;
  
  // Project month-end total
  const daysInMonth = new Date(
    new Date().getFullYear(), 
    new Date().getMonth() + 1, 
    0
  ).getDate();
  const projectedTotal = dailyAverage * daysInMonth;
  
  return {
    today: todayTotal,
    currentMonth: currentMonthTotal,
    lastMonth: lastMonthTotal,
    dailyAverage: Math.round(dailyAverage),
    projectedMonthEnd: Math.round(projectedTotal),
    growthRate: ((currentMonthTotal - lastMonthTotal) / lastMonthTotal * 100).toFixed(2) + '%',
    onTrackForGrowth: projectedTotal > lastMonthTotal
  };
}

// Example usage
const apiUsageComparison = await compareUsagePeriods('cma_endpoint_requests');
console.log('API Usage Trends:', apiUsageComparison);
```

### Pattern 3: Resource-Specific Tracking

```typescript
// Track specific resource usage patterns
async function trackResourceUsage(resourceType: 'items' | 'uploads' | 'users') {
  const endpointPatterns = {
    items: ['/items', '/item-types/*/items'],
    uploads: ['/uploads', '/upload-requests'],
    users: ['/users', '/user/*']
  };
  
  const endpoints = await client.usageCounters.find('cma_endpoint_requests', {
    period: 'current_month'
  });
  
  // Filter endpoints by resource type
  const resourceEndpoints = endpoints.result.filter(({ value }) => 
    endpointPatterns[resourceType].some(pattern => {
      const regex = new RegExp(pattern.replace('*', '.*'));
      return regex.test(value);
    })
  );
  
  // Calculate usage by operation type
  const operations = {
    create: 0,
    read: 0,
    update: 0,
    delete: 0
  };
  
  resourceEndpoints.forEach(({ value, counter }) => {
    if (value.includes('POST')) operations.create += counter;
    else if (value.includes('GET')) operations.read += counter;
    else if (value.includes('PUT') || value.includes('PATCH')) operations.update += counter;
    else if (value.includes('DELETE')) operations.delete += counter;
  });
  
  return {
    resourceType,
    totalRequests: resourceEndpoints.reduce((sum, item) => sum + item.counter, 0),
    operations,
    topEndpoints: resourceEndpoints
      .sort((a, b) => b.counter - a.counter)
      .slice(0, 10)
  };
}

// Track different resources
const itemUsage = await trackResourceUsage('items');
const uploadUsage = await trackResourceUsage('uploads');
const userUsage = await trackResourceUsage('users');

console.log('Resource Usage Analysis:', { itemUsage, uploadUsage, userUsage });
```

## Limit Checking Workflows

### Workflow 1: Real-time Limit Monitoring

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

interface UsageStatus {
  resource: string;
  current: number;
  limit: number;
  percentage: number;
  remaining: number;
  willExceed: boolean;
}

async function checkUsageAgainstLimits(): Promise<UsageStatus[]> {
  // Get subscription limits
  const limits = await client.subscriptionLimits.list();
  
  // Get current usage for various resources
  const [
    apiRequests,
    uploadBytes,
    videoSeconds
  ] = await Promise.all([
    client.usageCounters.find('cma_endpoint_requests', { period: 'current_month' }),
    client.usageCounters.find('assets_path_bytes', { period: 'current_month' }),
    client.usageCounters.find('video_path_seconds', { period: 'current_month' })
  ]);
  
  const usageStatuses: UsageStatus[] = [];
  
  // Check API request limit
  const apiLimit = limits.find(l => l.id === 'api_requests');
  if (apiLimit) {
    const currentApiRequests = apiRequests.result.reduce((sum, item) => sum + item.counter, 0);
    const percentage = (currentApiRequests / apiLimit.hardLimit) * 100;
    
    usageStatuses.push({
      resource: 'API Requests',
      current: currentApiRequests,
      limit: apiLimit.hardLimit,
      percentage: parseFloat(percentage.toFixed(2)),
      remaining: apiLimit.hardLimit - currentApiRequests,
      willExceed: percentage > 80 // Alert at 80%
    });
  }
  
  // Check bandwidth limit
  const bandwidthLimit = limits.find(l => l.id === 'bandwidth');
  if (bandwidthLimit) {
    const currentBandwidth = uploadBytes.result.reduce((sum, item) => sum + item.counter, 0);
    const percentage = (currentBandwidth / bandwidthLimit.hardLimit) * 100;
    
    usageStatuses.push({
      resource: 'Bandwidth',
      current: currentBandwidth,
      limit: bandwidthLimit.hardLimit,
      percentage: parseFloat(percentage.toFixed(2)),
      remaining: bandwidthLimit.hardLimit - currentBandwidth,
      willExceed: percentage > 80
    });
  }
  
  // Check video streaming limit
  const videoLimit = limits.find(l => l.id === 'video_streaming_seconds');
  if (videoLimit) {
    const currentVideoSeconds = videoSeconds.result.reduce((sum, item) => sum + item.counter, 0);
    const percentage = (currentVideoSeconds / videoLimit.hardLimit) * 100;
    
    usageStatuses.push({
      resource: 'Video Streaming',
      current: currentVideoSeconds,
      limit: videoLimit.hardLimit,
      percentage: parseFloat(percentage.toFixed(2)),
      remaining: videoLimit.hardLimit - currentVideoSeconds,
      willExceed: percentage > 80
    });
  }
  
  return usageStatuses;
}

// Check usage and alert if needed
const usageStatuses = await checkUsageAgainstLimits();
const warnings = usageStatuses.filter(status => status.willExceed);

if (warnings.length > 0) {
  console.warn('⚠️ Usage Warnings:');
  warnings.forEach(warning => {
    console.warn(`${warning.resource}: ${warning.percentage}% used (${warning.current.toLocaleString()} / ${warning.limit.toLocaleString()})`);
  });
}
```

### Workflow 2: Predictive Limit Analysis

```typescript
async function predictLimitExceeding() {
  const limits = await client.subscriptionLimits.list();
  const predictions = [];
  
  for (const limit of limits) {
    // Map limit types to counter IDs
    const counterMapping: Record<string, string> = {
      'api_requests': 'cma_endpoint_requests',
      'bandwidth': 'assets_path_bytes',
      'video_streaming_seconds': 'video_path_seconds'
    };
    
    const counterId = counterMapping[limit.id];
    if (!counterId) continue;
    
    // Get current month and last month data
    const [currentMonth, lastMonth] = await Promise.all([
      client.usageCounters.find(counterId, { period: 'current_month' }),
      client.usageCounters.find(counterId, { period: 'last_month' })
    ]);
    
    const currentTotal = currentMonth.result.reduce((sum, item) => sum + item.counter, 0);
    const lastMonthTotal = lastMonth.result.reduce((sum, item) => sum + item.counter, 0);
    
    // Calculate daily rate and project
    const daysElapsed = new Date().getDate();
    const dailyRate = currentTotal / daysElapsed;
    const daysInMonth = new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0).getDate();
    const projectedTotal = dailyRate * daysInMonth;
    
    predictions.push({
      resource: limit.id,
      currentUsage: currentTotal,
      projectedUsage: Math.round(projectedTotal),
      limit: limit.hardLimit,
      willExceed: projectedTotal > limit.hardLimit,
      projectedOverage: Math.max(0, projectedTotal - limit.hardLimit),
      daysUntilLimit: limit.hardLimit > currentTotal 
        ? Math.floor((limit.hardLimit - currentTotal) / dailyRate)
        : 0,
      recommendedDailyRate: limit.hardLimit / daysInMonth,
      currentDailyRate: dailyRate
    });
  }
  
  return predictions;
}

// Get predictions and show warnings
const predictions = await predictLimitExceeding();
const risks = predictions.filter(p => p.willExceed);

if (risks.length > 0) {
  console.error('🚨 Projected Limit Exceeding:');
  risks.forEach(risk => {
    console.error(`${risk.resource}: Projected to use ${risk.projectedUsage.toLocaleString()} (limit: ${risk.limit.toLocaleString()})`);
    console.error(`  Overage: ${risk.projectedOverage.toLocaleString()}`);
    console.error(`  Reduce daily rate from ${Math.round(risk.currentDailyRate).toLocaleString()} to ${Math.round(risk.recommendedDailyRate).toLocaleString()}`);
  });
}
```

## Usage Alerts Implementation

### Alert System with Thresholds

```typescript
import { buildClient } from '@datocms/cma-client-node';

interface AlertThreshold {
  resource: string;
  counterId: string;
  warningPercent: number;
  criticalPercent: number;
  limitType: string;
}

interface Alert {
  level: 'info' | 'warning' | 'critical';
  resource: string;
  message: string;
  currentUsage: number;
  limit: number;
  percentage: number;
  recommendation?: string;
}

class UsageAlertSystem {
  private client: any;
  private thresholds: AlertThreshold[] = [
    {
      resource: 'API Requests',
      counterId: 'cma_endpoint_requests',
      warningPercent: 70,
      criticalPercent: 90,
      limitType: 'api_requests'
    },
    {
      resource: 'Bandwidth',
      counterId: 'assets_path_bytes',
      warningPercent: 75,
      criticalPercent: 90,
      limitType: 'bandwidth'
    },
    {
      resource: 'Video Streaming',
      counterId: 'video_path_seconds',
      warningPercent: 80,
      criticalPercent: 95,
      limitType: 'video_streaming_seconds'
    }
  ];
  
  constructor(apiToken: string) {
    this.client = buildClient({ apiToken });
  }
  
  async checkAllResources(): Promise<Alert[]> {
    const alerts: Alert[] = [];
    const limits = await this.client.subscriptionLimits.list();
    
    for (const threshold of this.thresholds) {
      const limit = limits.find(l => l.id === threshold.limitType);
      if (!limit) continue;
      
      const usage = await this.client.usageCounters.find(threshold.counterId, {
        period: 'current_month'
      });
      
      const currentUsage = usage.result.reduce((sum, item) => sum + item.counter, 0);
      const percentage = (currentUsage / limit.hardLimit) * 100;
      
      if (percentage >= threshold.criticalPercent) {
        alerts.push({
          level: 'critical',
          resource: threshold.resource,
          message: `${threshold.resource} usage is critical!`,
          currentUsage,
          limit: limit.hardLimit,
          percentage: parseFloat(percentage.toFixed(2)),
          recommendation: `Immediate action required. Consider upgrading plan or reducing usage.`
        });
      } else if (percentage >= threshold.warningPercent) {
        alerts.push({
          level: 'warning',
          resource: threshold.resource,
          message: `${threshold.resource} usage is approaching limit`,
          currentUsage,
          limit: limit.hardLimit,
          percentage: parseFloat(percentage.toFixed(2)),
          recommendation: `Monitor usage closely. ${100 - percentage}% remaining.`
        });
      }
    }
    
    return alerts;
  }
  
  async getDetailedAnalysis(counterId: string): Promise<any> {
    const [today, currentMonth, lastMonth] = await Promise.all([
      this.client.usageCounters.find(counterId, { period: 'today' }),
      this.client.usageCounters.find(counterId, { period: 'current_month' }),
      this.client.usageCounters.find(counterId, { period: 'last_month' })
    ]);
    
    // Identify top consumers
    const topConsumers = currentMonth.result
      .sort((a, b) => b.counter - a.counter)
      .slice(0, 10);
    
    // Calculate growth trends
    const currentTotal = currentMonth.result.reduce((sum, item) => sum + item.counter, 0);
    const lastMonthTotal = lastMonth.result.reduce((sum, item) => sum + item.counter, 0);
    const todayTotal = today.result.reduce((sum, item) => sum + item.counter, 0);
    
    return {
      topConsumers,
      usage: {
        today: todayTotal,
        currentMonth: currentTotal,
        lastMonth: lastMonthTotal
      },
      trends: {
        monthOverMonth: ((currentTotal - lastMonthTotal) / lastMonthTotal * 100).toFixed(2) + '%',
        dailyAverage: Math.round(currentTotal / new Date().getDate()),
        projectedMonthEnd: Math.round((currentTotal / new Date().getDate()) * 30)
      }
    };
  }
  
  async sendAlertNotification(alert: Alert): Promise<void> {
    // Implement your notification logic here
    // Could be email, Slack, webhook, etc.
    console.log(`[${alert.level.toUpperCase()}] ${alert.message}`);
    console.log(`Current: ${alert.currentUsage.toLocaleString()} / Limit: ${alert.limit.toLocaleString()} (${alert.percentage}%)`);
    if (alert.recommendation) {
      console.log(`Recommendation: ${alert.recommendation}`);
    }
  }
}

// Usage
const alertSystem = new UsageAlertSystem('YOUR_API_TOKEN');

// Run periodic checks
async function runUsageMonitoring() {
  const alerts = await alertSystem.checkAllResources();
  
  for (const alert of alerts) {
    await alertSystem.sendAlertNotification(alert);
    
    // Get detailed analysis for critical alerts
    if (alert.level === 'critical') {
      const analysis = await alertSystem.getDetailedAnalysis(
        alertSystem.thresholds.find(t => t.resource === alert.resource)!.counterId
      );
      console.log(`Detailed analysis for ${alert.resource}:`, analysis);
    }
  }
}

// Run every hour
setInterval(runUsageMonitoring, 3600000);
```

### Automated Response System

```typescript
class AutomatedUsageResponse {
  private client: any;
  
  constructor(apiToken: string) {
    this.client = buildClient({ apiToken });
  }
  
  async handleHighUsage(resource: string, percentage: number): Promise<void> {
    if (percentage > 95) {
      await this.emergencyMeasures(resource);
    } else if (percentage > 85) {
      await this.throttleMeasures(resource);
    } else if (percentage > 70) {
      await this.preventiveMeasures(resource);
    }
  }
  
  private async emergencyMeasures(resource: string): Promise<void> {
    console.log(`🚨 Emergency measures for ${resource}`);
    
    switch (resource) {
      case 'API Requests':
        // Identify and temporarily block abusive IPs
        const ipRequests = await this.client.usageCounters.find('cma_ip_requests', {
          period: 'today'
        });
        
        const abusiveIPs = ipRequests.result
          .filter(({ counter }) => counter > 10000) // High threshold
          .map(({ value }) => value);
        
        console.log(`Identified ${abusiveIPs.length} potentially abusive IPs`);
        // Implement IP blocking logic here
        break;
        
      case 'Bandwidth':
        // Identify large assets and consider CDN caching
        const assetBytes = await this.client.usageCounters.find('assets_path_bytes', {
          period: 'today'
        });
        
        const largeAssets = assetBytes.result
          .filter(({ counter }) => counter > 1073741824) // > 1GB
          .map(({ value, counter }) => ({
            path: value,
            size: (counter / 1073741824).toFixed(2) + ' GB'
          }));
        
        console.log('Large assets consuming bandwidth:', largeAssets);
        // Implement CDN strategy here
        break;
    }
  }
  
  private async throttleMeasures(resource: string): Promise<void> {
    console.log(`⚠️ Throttling measures for ${resource}`);
    
    // Implement rate limiting
    // Notify heavy users
    // Enable caching where possible
  }
  
  private async preventiveMeasures(resource: string): Promise<void> {
    console.log(`ℹ️ Preventive measures for ${resource}`);
    
    // Send usage reports
    // Optimize queries
    // Review access patterns
  }
}
```

## Error Handling

### Comprehensive Error Handling

```typescript
import { buildClient, ApiError } from '@datocms/cma-client-node';

async function safeGetUsageCounter(
  counterId: string,
  period?: 'today' | 'current_month' | 'last_month'
): Promise<UsageCounter | null> {
  const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });
  
  try {
    const counter = await client.usageCounters.find(counterId, { period });
    return counter;
    
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.statusCode) {
        case 404:
          console.error(`Counter type '${counterId}' not found`);
          console.log('Available counter types:', [
            'assets_path_bytes',
            'assets_referrer_bytes',
            'assets_ip_bytes',
            'assets_full_path_bytes',
            'assets_path_requests',
            'assets_full_path_requests',
            'cda_access_token_id_bytes',
            'cda_access_token_id_requests',
            'cda_referrer_bytes',
            'cda_referrer_requests',
            'cda_ip_bytes',
            'cda_ip_requests',
            'cma_endpoint_bytes',
            'cma_endpoint_requests',
            'cma_user_bytes',
            'cma_user_requests',
            'cma_ip_bytes',
            'cma_ip_requests',
            'video_path_seconds'
          ]);
          break;
          
        case 422:
          console.error('Invalid parameters:', error.errors);
          if (period && !['today', 'current_month', 'last_month'].includes(period)) {
            console.error(`Invalid period '${period}'. Must be: today, current_month, or last_month`);
          }
          break;
          
        case 401:
          console.error('Authentication failed. Check your API token.');
          break;
          
        case 429:
          console.error('Rate limit exceeded. Please retry after:', error.headers['x-ratelimit-reset']);
          break;
          
        default:
          console.error(`API error ${error.statusCode}:`, error.message);
      }
    } else {
      console.error('Unexpected error:', error);
    }
    
    return null;
  }
}

// Usage with error handling
const counter = await safeGetUsageCounter('cma_endpoint_requests', 'today');
if (counter) {
  console.log(`Found ${counter.result.length} endpoints with activity today`);
}
```

### Retry Logic for Rate Limits

```typescript
async function getUsageCounterWithRetry(
  counterId: string,
  options?: { period?: 'today' | 'current_month' | 'last_month' },
  maxRetries: number = 3
): Promise<UsageCounter | null> {
  const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });
  let lastError: Error | null = null;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await client.usageCounters.find(counterId, options);
      
    } catch (error) {
      lastError = error as Error;
      
      if (error instanceof ApiError && error.statusCode === 429) {
        // Rate limited - wait and retry
        const resetTime = error.headers['x-ratelimit-reset'];
        const waitTime = resetTime 
          ? Math.max(0, new Date(resetTime).getTime() - Date.now())
          : Math.min(1000 * Math.pow(2, attempt), 30000); // Exponential backoff
        
        console.log(`Rate limited. Waiting ${waitTime}ms before retry ${attempt}/${maxRetries}`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }
      
      // Non-retryable error
      throw error;
    }
  }
  
  console.error('Max retries reached:', lastError);
  return null;
}
```

## Best Practices

### 1. Efficient Data Fetching

```typescript
// ❌ Inefficient: Sequential requests
const requests1 = await client.usageCounters.find('cma_endpoint_requests', { period: 'today' });
const requests2 = await client.usageCounters.find('cma_endpoint_requests', { period: 'current_month' });
const requests3 = await client.usageCounters.find('cma_endpoint_requests', { period: 'last_month' });

// ✅ Efficient: Parallel requests
const [todayData, monthData, lastMonthData] = await Promise.all([
  client.usageCounters.find('cma_endpoint_requests', { period: 'today' }),
  client.usageCounters.find('cma_endpoint_requests', { period: 'current_month' }),
  client.usageCounters.find('cma_endpoint_requests', { period: 'last_month' })
]);
```

### 2. Data Caching Strategy

```typescript
class UsageCounterCache {
  private cache: Map<string, { data: any; timestamp: number }> = new Map();
  private ttl: number = 60000; // 1 minute cache
  private client: any;
  
  constructor(apiToken: string) {
    this.client = buildClient({ apiToken });
  }
  
  async get(
    counterId: string, 
    period?: 'today' | 'current_month' | 'last_month'
  ): Promise<UsageCounter> {
    const cacheKey = `${counterId}-${period || 'default'}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.data;
    }
    
    const data = await this.client.usageCounters.find(counterId, { period });
    this.cache.set(cacheKey, { data, timestamp: Date.now() });
    
    return data;
  }
  
  clearCache(): void {
    this.cache.clear();
  }
  
  setTTL(milliseconds: number): void {
    this.ttl = milliseconds;
  }
}

// Usage
const cachedClient = new UsageCounterCache('YOUR_API_TOKEN');
const data1 = await cachedClient.get('cma_endpoint_requests', 'today'); // Fetches from API
const data2 = await cachedClient.get('cma_endpoint_requests', 'today'); // Returns from cache
```

### 3. Aggregation Helpers

```typescript
class UsageAggregator {
  static sumCounter(counter: UsageCounter): number {
    return counter.result.reduce((sum, item) => sum + item.counter, 0);
  }
  
  static topN(counter: UsageCounter, n: number): Array<{ value: string; counter: number }> {
    return counter.result
      .sort((a, b) => b.counter - a.counter)
      .slice(0, n);
  }
  
  static groupByDomain(counter: UsageCounter): Record<string, number> {
    const domains: Record<string, number> = {};
    
    counter.result.forEach(({ value, counter }) => {
      try {
        const domain = new URL(value).hostname;
        domains[domain] = (domains[domain] || 0) + counter;
      } catch {
        domains['unknown'] = (domains['unknown'] || 0) + counter;
      }
    });
    
    return domains;
  }
  
  static formatBytes(bytes: number): string {
    const units = ['B', 'KB', 'MB', 'GB', 'TB'];
    let size = bytes;
    let unitIndex = 0;
    
    while (size >= 1024 && unitIndex < units.length - 1) {
      size /= 1024;
      unitIndex++;
    }
    
    return `${size.toFixed(2)} ${units[unitIndex]}`;
  }
  
  static formatDuration(seconds: number): string {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = seconds % 60;
    
    return `${hours}h ${minutes}m ${secs}s`;
  }
}

// Usage
const assetUsage = await client.usageCounters.find('assets_path_bytes', { 
  period: 'current_month' 
});

const totalBytes = UsageAggregator.sumCounter(assetUsage);
console.log(`Total bandwidth: ${UsageAggregator.formatBytes(totalBytes)}`);

const top10Assets = UsageAggregator.topN(assetUsage, 10);
console.log('Top 10 assets by bandwidth:', top10Assets);
```

### 4. Performance Monitoring Pattern

```typescript
class PerformanceMonitor {
  private client: any;
  
  constructor(apiToken: string) {
    this.client = buildClient({ apiToken });
  }
  
  async analyzeEndpointPerformance(): Promise<any> {
    const endpoints = await this.client.usageCounters.find('cma_endpoint_requests', {
      period: 'today'
    });
    
    // Group by operation type
    const analysis = {
      byMethod: {} as Record<string, number>,
      byResource: {} as Record<string, number>,
      slowEndpoints: [] as Array<{ endpoint: string; requests: number }>,
      recommendations: [] as string[]
    };
    
    endpoints.result.forEach(({ value, counter }) => {
      // Extract method (GET, POST, etc.)
      const methodMatch = value.match(/^(GET|POST|PUT|PATCH|DELETE)/);
      if (methodMatch) {
        const method = methodMatch[1];
        analysis.byMethod[method] = (analysis.byMethod[method] || 0) + counter;
      }
      
      // Extract resource type
      const resourceMatch = value.match(/\/(items|uploads|users|roles|webhooks)/);
      if (resourceMatch) {
        const resource = resourceMatch[1];
        analysis.byResource[resource] = (analysis.byResource[resource] || 0) + counter;
      }
      
      // Identify potentially slow endpoints (high request count)
      if (counter > 1000) {
        analysis.slowEndpoints.push({ endpoint: value, requests: counter });
      }
    });
    
    // Generate recommendations
    if (analysis.byMethod.GET > (analysis.byMethod.POST || 0) * 10) {
      analysis.recommendations.push('Consider implementing caching for frequent GET requests');
    }
    
    if (analysis.slowEndpoints.length > 0) {
      analysis.recommendations.push('Review and optimize high-traffic endpoints');
    }
    
    return analysis;
  }
}
```

## Related Resources

- [Usage Counter Resource](/01-cma-client/07-monitoring-usage/usage-counter.md) - Detailed resource documentation
- [Daily Usage](/01-cma-client/07-monitoring-usage/daily-usage.md) - Aggregated daily statistics
- [Subscription Limits](/01-cma-client/07-monitoring-usage/subscription-limit.md) - View and understand plan limits
- [Subscription Features](/01-cma-client/07-monitoring-usage/subscription-feature.md) - Available features by plan
- [Access Token Methods](/01-cma-client/05-access-control/access-token.md) - Manage API access tokens
- [Webhook Methods](/01-cma-client/04-site-configuration/webhook.md) - Set up usage alerts via webhooks
- [Performance Optimization](/04-advanced-topics/performance-optimization.md) - Best practices for API usage