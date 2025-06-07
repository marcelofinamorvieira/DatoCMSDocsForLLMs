# SubscriptionLimit

The SubscriptionLimit resource provides real-time information about your DatoCMS project's plan limits and current usage. This is essential for monitoring resource consumption, implementing usage warnings, and ensuring your application respects plan constraints.

## Available Operations

### List All Subscription Limits

```javascript
const limits = await client.subscriptionLimits.list();
```

**Returns:** An array of all subscription limits for your project

### Find Specific Subscription Limit

```javascript
const limit = await client.subscriptionLimits.find(limitId);
```

**Parameters:**
- `limitId` (string, required): The limit identifier (e.g., 'api_calls', 'storage_bytes')

**Returns:** A single SubscriptionLimit object

## The SubscriptionLimit Object

```typescript
{
  id: string;                    // The limit identifier
  type: 'subscription_limit';    // Always 'subscription_limit'
  code: string;                  // The limit code/name
  usage: number;                 // Current usage amount
  limit: number | null;          // Maximum allowed (null = unlimited)
}
```

## Common Limit Types

### API Usage Limits
| Limit Code | Description | Reset Period |
|------------|-------------|--------------|
| `api_calls` | Total API calls | Monthly |
| `cda_api_calls` | Content Delivery API calls | Monthly |
| `cma_api_calls` | Content Management API calls | Monthly |

### Storage & Bandwidth
| Limit Code | Description | Type |
|------------|-------------|------|
| `storage_bytes` | Total storage used | Cumulative |
| `bandwidth_bytes` | Monthly bandwidth | Monthly |
| `upload_size_bytes` | Max file upload size | Per file |

### Content Limits
| Limit Code | Description | Type |
|------------|-------------|------|
| `items_count` | Total content items | Cumulative |
| `item_types_count` | Number of models | Cumulative |
| `locales_count` | Number of locales | Cumulative |
| `environments_count` | Sandbox environments | Cumulative |

### User & Access
| Limit Code | Description | Type |
|------------|-------------|------|
| `users_count` | Project users | Cumulative |
| `roles_count` | Custom roles | Cumulative |
| `access_tokens_count` | API tokens | Cumulative |

### Features
| Limit Code | Description | Type |
|------------|-------------|------|
| `build_triggers_count` | Build triggers | Cumulative |
| `webhooks_count` | Webhooks | Cumulative |
| `plugins_count` | Installed plugins | Cumulative |

## Common Use Cases

### 1. Usage Dashboard

```javascript
// Create a comprehensive usage report
const limits = await client.subscriptionLimits.list();

const dashboard = limits.map(limit => {
  const percentage = limit.limit 
    ? Math.round((limit.usage / limit.limit) * 100)
    : null;
  
  return {
    resource: limit.code.replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase()),
    usage: limit.usage.toLocaleString(),
    limit: limit.limit ? limit.limit.toLocaleString() : 'Unlimited',
    utilization: percentage !== null ? `${percentage}%` : '-',
    status: percentage > 90 ? '🔴' : percentage > 75 ? '🟡' : '🟢'
  };
});

console.table(dashboard);
```

### 2. Pre-flight Validation

```javascript
// Check limits before bulk operations
async function canCreateItems(client, count) {
  const itemsLimit = await client.subscriptionLimits.find('items_count');
  
  if (!itemsLimit.limit) {
    return { canCreate: true, message: 'No item limit' };
  }
  
  const remaining = itemsLimit.limit - itemsLimit.usage;
  
  if (count > remaining) {
    return {
      canCreate: false,
      message: `Cannot create ${count} items. Only ${remaining} slots available.`
    };
  }
  
  return {
    canCreate: true,
    message: `Creating ${count} items (${remaining - count} will remain)`
  };
}

// Usage
const validation = await canCreateItems(client, 50);
if (!validation.canCreate) {
  throw new Error(validation.message);
}
```

### 3. Usage Alerts

```javascript
// Set up usage alerts for critical resources
async function checkCriticalLimits(client, thresholds = {}) {
  const defaultThresholds = {
    warning: 0.75,  // 75%
    critical: 0.90  // 90%
  };
  
  const config = { ...defaultThresholds, ...thresholds };
  const alerts = [];
  
  const criticalResources = [
    'api_calls',
    'storage_bytes',
    'bandwidth_bytes',
    'items_count'
  ];
  
  for (const resourceCode of criticalResources) {
    try {
      const limit = await client.subscriptionLimits.find(resourceCode);
      
      if (limit.limit) {
        const utilization = limit.usage / limit.limit;
        
        if (utilization >= config.critical) {
          alerts.push({
            level: 'critical',
            resource: resourceCode,
            message: `${resourceCode} is at ${(utilization * 100).toFixed(1)}% capacity!`,
            usage: limit.usage,
            limit: limit.limit
          });
        } else if (utilization >= config.warning) {
          alerts.push({
            level: 'warning',
            resource: resourceCode,
            message: `${resourceCode} is at ${(utilization * 100).toFixed(1)}% capacity`,
            usage: limit.usage,
            limit: limit.limit
          });
        }
      }
    } catch (error) {
      console.error(`Failed to check ${resourceCode}:`, error);
    }
  }
  
  return alerts;
}

// Usage
const alerts = await checkCriticalLimits(client);
alerts.forEach(alert => {
  console[alert.level === 'critical' ? 'error' : 'warn'](alert.message);
});
```

### 4. Storage Management

```javascript
// Monitor and manage storage usage
async function analyzeStorageUsage(client) {
  const storageLimit = await client.subscriptionLimits.find('storage_bytes');
  
  const usageGB = (storageLimit.usage / (1024 ** 3)).toFixed(2);
  const limitGB = storageLimit.limit 
    ? (storageLimit.limit / (1024 ** 3)).toFixed(2)
    : null;
  
  const analysis = {
    current: `${usageGB} GB`,
    limit: limitGB ? `${limitGB} GB` : 'Unlimited',
    percentage: storageLimit.limit 
      ? `${((storageLimit.usage / storageLimit.limit) * 100).toFixed(1)}%`
      : 'N/A',
    remaining: storageLimit.limit
      ? `${((storageLimit.limit - storageLimit.usage) / (1024 ** 3)).toFixed(2)} GB`
      : 'Unlimited'
  };
  
  // Suggest cleanup if usage is high
  if (storageLimit.limit && storageLimit.usage > storageLimit.limit * 0.8) {
    analysis.recommendation = 'Consider cleaning up unused assets or upgrading your plan';
    
    // Get large uploads for cleanup suggestions
    const uploads = await client.uploads.list({
      filter: { size: { gt: 10485760 } }, // Files > 10MB
      order_by: 'size_DESC',
      per_page: 10
    });
    
    analysis.largestFiles = uploads.map(u => ({
      filename: u.filename,
      size: `${(u.size / (1024 ** 2)).toFixed(2)} MB`
    }));
  }
  
  return analysis;
}
```

### 5. API Rate Limit Monitoring

```javascript
// Track API usage throughout the month
async function getApiUsageStatus(client) {
  const apiLimit = await client.subscriptionLimits.find('api_calls');
  
  // Calculate days in current month
  const now = new Date();
  const daysInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
  const currentDay = now.getDate();
  const daysRemaining = daysInMonth - currentDay;
  
  // Calculate burn rate
  const dailyAverage = apiLimit.usage / currentDay;
  const projectedMonthly = dailyAverage * daysInMonth;
  
  const status = {
    current: apiLimit.usage.toLocaleString(),
    limit: apiLimit.limit ? apiLimit.limit.toLocaleString() : 'Unlimited',
    dailyAverage: Math.round(dailyAverage).toLocaleString(),
    projectedMonthly: Math.round(projectedMonthly).toLocaleString(),
    daysRemaining,
    willExceedLimit: apiLimit.limit && projectedMonthly > apiLimit.limit
  };
  
  if (status.willExceedLimit) {
    status.recommendedDailyLimit = Math.floor(
      (apiLimit.limit - apiLimit.usage) / daysRemaining
    ).toLocaleString();
  }
  
  return status;
}
```

### 6. Plan Upgrade Recommendations

```javascript
// Analyze usage patterns and recommend upgrades
async function analyzePlanFit(client) {
  const limits = await client.subscriptionLimits.list();
  const recommendations = [];
  
  for (const limit of limits) {
    if (limit.limit === null) continue; // Skip unlimited resources
    
    const utilization = (limit.usage / limit.limit) * 100;
    
    if (utilization > 80) {
      recommendations.push({
        resource: limit.code,
        severity: utilization > 95 ? 'critical' : 'warning',
        current: `${limit.usage} / ${limit.limit}`,
        utilization: `${utilization.toFixed(1)}%`,
        recommendation: `Consider upgrading - ${limit.code} is near capacity`
      });
    }
  }
  
  return {
    needsUpgrade: recommendations.some(r => r.severity === 'critical'),
    recommendations: recommendations.sort((a, b) => 
      b.utilization.localeCompare(a.utilization)
    )
  };
}
```

## Understanding Limit Values

### Interpreting the Data

1. **Numeric Limits**: A specific number indicates the hard cap for that resource
2. **Null Limits**: `null` means unlimited access on your current plan
3. **Usage Tracking**: Real-time or near-real-time consumption data
4. **Reset Periods**: 
   - Monthly: Resets on the first day of each month (API calls, bandwidth)
   - Cumulative: Never resets, always increases (storage, items, models)

### Unit Reference

| Resource Type | Unit | Example |
|---------------|------|---------|
| API Calls | Count | 1,000,000 calls |
| Storage/Bandwidth | Bytes | 5,368,709,120 bytes (5 GB) |
| Items/Users/Models | Count | 10,000 items |

## Raw Response Access

For accessing the complete API response:

```javascript
// Get raw response with metadata
const response = await client.subscriptionLimits.rawList();
console.log(response.data); // Array of limits
console.log(response.meta); // Response metadata

// Raw find
const rawLimit = await client.subscriptionLimits.rawFind('api_calls');
console.log(rawLimit.data); // Single limit with full JSON:API structure
```

## Important Notes

1. **Real-time Data**: Usage values are updated in real-time or with minimal delay
2. **Plan Dependencies**: Available limits depend on your subscription plan
3. **No Modification**: This is a read-only resource
4. **Caching**: Consider caching limit checks to reduce API calls
5. **Grace Periods**: Some limits may have grace periods - check your plan details

## Related Resources

- [Usage Counters](/01-cma-client/07-monitoring-usage/usage-counter.md) - Detailed usage analytics
- [Daily Usage](/01-cma-client/07-monitoring-usage/daily-usage.md) - Historical usage data
- [Subscription Features](/01-cma-client/07-monitoring-usage/subscription-feature.md) - Available features per plan
- [Plans](/02-dashboard-client/plan.md) - Plan details and pricing

---

# Subscription Limit Methods

This document provides comprehensive documentation for methods available on the Subscription Limit resource. These methods help you monitor and manage various limits within your DatoCMS subscription.

## Available Methods

### list()

Lists all subscription limits for the current project, including current usage and limit values.

```typescript
// Complete example with imports
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

// Initialize the client
const client = buildClient({
  apiToken: 'YOUR_API_TOKEN',
});

// TypeScript type definitions for response
interface SubscriptionLimit {
  id: string;
  type: 'subscription_limit';
  attributes: {
    limit_type: 'items' | 'uploads' | 'api_calls' | 'build_triggers' | 'traffic_bytes' | 'mux_encoding_seconds' | 'mux_streaming_seconds';
    limit_value: number | null; // null means unlimited
    limit_is_hard: boolean; // true = hard limit, false = soft limit
    current_usage: number;
    usage_updated_at: string; // ISO 8601 datetime
    reset_at: string | null; // ISO 8601 datetime, null for non-resetting limits
  };
}

// List all subscription limits
async function listSubscriptionLimits() {
  try {
    const limits = await client.subscriptionLimits.list();
    
    // Response structure
    console.log('Subscription limits:', limits);
    // Returns array of SubscriptionLimit objects
    
    return limits;
  } catch (error) {
    console.error('Error fetching subscription limits:', error);
    throw error;
  }
}
```

## Response Structure

```typescript
// Example response with detailed comments
const exampleResponse: SubscriptionLimit[] = [
  {
    id: "items",
    type: "subscription_limit",
    attributes: {
      limit_type: "items",              // Type of limit
      limit_value: 10000,               // Maximum allowed (null = unlimited)
      limit_is_hard: true,              // true = blocks operations, false = allows with warning
      current_usage: 8543,              // Current count/usage
      usage_updated_at: "2024-01-15T10:30:00Z",  // Last usage update
      reset_at: null                    // When counter resets (null = never)
    }
  },
  {
    id: "api_calls",
    type: "subscription_limit",
    attributes: {
      limit_type: "api_calls",
      limit_value: 1000000,             // 1M API calls/month
      limit_is_hard: false,             // Soft limit - warns but doesn't block
      current_usage: 750000,
      usage_updated_at: "2024-01-15T10:30:00Z",
      reset_at: "2024-02-01T00:00:00Z" // Resets monthly
    }
  },
  {
    id: "uploads",
    type: "subscription_limit",
    attributes: {
      limit_type: "uploads",
      limit_value: 5000,
      limit_is_hard: true,
      current_usage: 2340,
      usage_updated_at: "2024-01-15T10:30:00Z",
      reset_at: null
    }
  }
];
```

## Limit Checking Patterns

### Check Specific Limit Type

```typescript
async function checkItemsLimit() {
  try {
    const limits = await client.subscriptionLimits.list();
    const itemsLimit = limits.find(limit => limit.attributes.limit_type === 'items');
    
    if (!itemsLimit) {
      console.log('No items limit found');
      return null;
    }
    
    const usage = itemsLimit.attributes.current_usage;
    const limit = itemsLimit.attributes.limit_value;
    
    if (limit === null) {
      console.log('Items: Unlimited');
      return { unlimited: true, usage };
    }
    
    const percentage = (usage / limit) * 100;
    console.log(`Items: ${usage}/${limit} (${percentage.toFixed(1)}%)`);
    
    return {
      usage,
      limit,
      percentage,
      remaining: limit - usage,
      isHard: itemsLimit.attributes.limit_is_hard
    };
  } catch (error) {
    console.error('Error checking items limit:', error);
    throw error;
  }
}
```

### Check All Limits Status

```typescript
async function checkAllLimitsStatus() {
  try {
    const limits = await client.subscriptionLimits.list();
    
    const limitsStatus = limits.map(limit => {
      const { limit_type, limit_value, current_usage, limit_is_hard } = limit.attributes;
      
      if (limit_value === null) {
        return {
          type: limit_type,
          status: 'unlimited',
          usage: current_usage,
          isHard: limit_is_hard
        };
      }
      
      const percentage = (current_usage / limit_value) * 100;
      let status: 'ok' | 'warning' | 'critical' | 'exceeded';
      
      if (percentage >= 100) {
        status = 'exceeded';
      } else if (percentage >= 90) {
        status = 'critical';
      } else if (percentage >= 75) {
        status = 'warning';
      } else {
        status = 'ok';
      }
      
      return {
        type: limit_type,
        status,
        usage: current_usage,
        limit: limit_value,
        percentage,
        remaining: limit_value - current_usage,
        isHard: limit_is_hard
      };
    });
    
    return limitsStatus;
  } catch (error) {
    console.error('Error checking limits status:', error);
    throw error;
  }
}
```

## Usage vs Limit Comparison

### Generate Usage Report

```typescript
interface UsageReport {
  limitType: string;
  usage: number;
  limit: number | null;
  percentage: number | null;
  status: 'ok' | 'warning' | 'critical' | 'exceeded' | 'unlimited';
  daysUntilReset: number | null;
  recommendation?: string;
}

async function generateUsageReport(): Promise<UsageReport[]> {
  try {
    const limits = await client.subscriptionLimits.list();
    const now = new Date();
    
    return limits.map(limit => {
      const {
        limit_type,
        limit_value,
        current_usage,
        limit_is_hard,
        reset_at
      } = limit.attributes;
      
      // Calculate days until reset
      let daysUntilReset: number | null = null;
      if (reset_at) {
        const resetDate = new Date(reset_at);
        daysUntilReset = Math.ceil((resetDate.getTime() - now.getTime()) / (1000 * 60 * 60 * 24));
      }
      
      // Determine status and recommendations
      if (limit_value === null) {
        return {
          limitType: limit_type,
          usage: current_usage,
          limit: null,
          percentage: null,
          status: 'unlimited',
          daysUntilReset
        };
      }
      
      const percentage = (current_usage / limit_value) * 100;
      let status: UsageReport['status'];
      let recommendation: string | undefined;
      
      if (percentage >= 100) {
        status = 'exceeded';
        recommendation = limit_is_hard 
          ? 'Operations blocked. Upgrade plan immediately.'
          : 'Soft limit exceeded. Consider upgrading to avoid future blocks.';
      } else if (percentage >= 90) {
        status = 'critical';
        recommendation = 'Usage critical. Plan upgrade or reduce usage.';
      } else if (percentage >= 75) {
        status = 'warning';
        recommendation = 'Monitor usage closely. Consider planning for expansion.';
      } else {
        status = 'ok';
      }
      
      return {
        limitType: limit_type,
        usage: current_usage,
        limit: limit_value,
        percentage,
        status,
        daysUntilReset,
        recommendation
      };
    });
  } catch (error) {
    console.error('Error generating usage report:', error);
    throw error;
  }
}
```

## Hard vs Soft Limits Handling

### Implement Pre-operation Checks

```typescript
// Check before creating items
async function canCreateItems(count: number = 1): Promise<{
  allowed: boolean;
  reason?: string;
  currentUsage?: number;
  limit?: number;
}> {
  try {
    const limits = await client.subscriptionLimits.list();
    const itemsLimit = limits.find(l => l.attributes.limit_type === 'items');
    
    if (!itemsLimit) {
      return { allowed: true };
    }
    
    const { current_usage, limit_value, limit_is_hard } = itemsLimit.attributes;
    
    // No limit
    if (limit_value === null) {
      return { allowed: true, currentUsage: current_usage };
    }
    
    const projectedUsage = current_usage + count;
    
    // Within limit
    if (projectedUsage <= limit_value) {
      return {
        allowed: true,
        currentUsage: current_usage,
        limit: limit_value
      };
    }
    
    // Over limit
    if (limit_is_hard) {
      return {
        allowed: false,
        reason: `Hard limit exceeded. Current: ${current_usage}, Limit: ${limit_value}`,
        currentUsage: current_usage,
        limit: limit_value
      };
    } else {
      // Soft limit - warn but allow
      console.warn(`Soft limit will be exceeded. Current: ${current_usage}, Limit: ${limit_value}`);
      return {
        allowed: true,
        reason: `Warning: Soft limit exceeded`,
        currentUsage: current_usage,
        limit: limit_value
      };
    }
  } catch (error) {
    console.error('Error checking item creation limit:', error);
    // On error, allow operation but log warning
    console.warn('Could not verify limits, proceeding with operation');
    return { allowed: true, reason: 'Limit check failed' };
  }
}

// Use in practice
async function createItemsWithLimitCheck(itemsData: any[]) {
  const check = await canCreateItems(itemsData.length);
  
  if (!check.allowed) {
    throw new Error(`Cannot create items: ${check.reason}`);
  }
  
  if (check.reason) {
    console.warn(check.reason);
  }
  
  // Proceed with creation
  for (const data of itemsData) {
    await client.items.create(data);
  }
}
```

## Limit Enforcement Workflows

### Automated Limit Monitoring

```typescript
// Set up periodic limit monitoring
class LimitMonitor {
  private intervalId?: NodeJS.Timeout;
  private warningThreshold = 75;
  private criticalThreshold = 90;
  
  constructor(
    private client: any,
    private onWarning?: (limit: SubscriptionLimit) => void,
    private onCritical?: (limit: SubscriptionLimit) => void,
    private onExceeded?: (limit: SubscriptionLimit) => void
  ) {}
  
  async checkLimits() {
    try {
      const limits = await this.client.subscriptionLimits.list();
      
      for (const limit of limits) {
        const { current_usage, limit_value, limit_type } = limit.attributes;
        
        if (limit_value === null) continue;
        
        const percentage = (current_usage / limit_value) * 100;
        
        if (percentage >= 100 && this.onExceeded) {
          this.onExceeded(limit);
        } else if (percentage >= this.criticalThreshold && this.onCritical) {
          this.onCritical(limit);
        } else if (percentage >= this.warningThreshold && this.onWarning) {
          this.onWarning(limit);
        }
      }
    } catch (error) {
      console.error('Limit monitoring error:', error);
    }
  }
  
  start(intervalMinutes: number = 60) {
    // Initial check
    this.checkLimits();
    
    // Set up periodic checks
    this.intervalId = setInterval(
      () => this.checkLimits(),
      intervalMinutes * 60 * 1000
    );
  }
  
  stop() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  }
}

// Usage
const monitor = new LimitMonitor(
  client,
  (limit) => console.warn(`Warning: ${limit.attributes.limit_type} at ${(limit.attributes.current_usage / limit.attributes.limit_value! * 100).toFixed(1)}%`),
  (limit) => console.error(`Critical: ${limit.attributes.limit_type} at ${(limit.attributes.current_usage / limit.attributes.limit_value! * 100).toFixed(1)}%`),
  (limit) => console.error(`EXCEEDED: ${limit.attributes.limit_type} limit exceeded!`)
);

monitor.start(30); // Check every 30 minutes
```

### Implement Usage Throttling

```typescript
// Throttle operations based on limit proximity
class UsageThrottler {
  private throttleDelay = 0;
  
  constructor(private client: any) {}
  
  async calculateThrottleDelay(limitType: string): Promise<number> {
    try {
      const limits = await this.client.subscriptionLimits.list();
      const limit = limits.find(l => l.attributes.limit_type === limitType);
      
      if (!limit || limit.attributes.limit_value === null) {
        return 0; // No throttling needed
      }
      
      const { current_usage, limit_value } = limit.attributes;
      const percentage = (current_usage / limit_value) * 100;
      
      // Progressive throttling based on usage
      if (percentage >= 95) {
        return 5000; // 5 second delay
      } else if (percentage >= 90) {
        return 2000; // 2 second delay
      } else if (percentage >= 85) {
        return 1000; // 1 second delay
      } else if (percentage >= 80) {
        return 500;  // 0.5 second delay
      }
      
      return 0; // No throttling
    } catch (error) {
      console.error('Error calculating throttle delay:', error);
      return 0;
    }
  }
  
  async executeWithThrottle<T>(
    operation: () => Promise<T>,
    limitType: string
  ): Promise<T> {
    const delay = await this.calculateThrottleDelay(limitType);
    
    if (delay > 0) {
      console.log(`Throttling operation: ${delay}ms delay due to ${limitType} usage`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
    
    return operation();
  }
}

// Usage
const throttler = new UsageThrottler(client);

// Throttled item creation
async function createItemThrottled(data: any) {
  return throttler.executeWithThrottle(
    () => client.items.create(data),
    'items'
  );
}
```

## Approaching Limit Warnings

### Proactive Warning System

```typescript
interface LimitWarning {
  limitType: string;
  severity: 'low' | 'medium' | 'high' | 'critical';
  message: string;
  currentUsage: number;
  limit: number;
  percentage: number;
  recommendation: string;
}

async function getLimitWarnings(): Promise<LimitWarning[]> {
  try {
    const limits = await client.subscriptionLimits.list();
    const warnings: LimitWarning[] = [];
    
    for (const limit of limits) {
      const { limit_type, current_usage, limit_value, limit_is_hard, reset_at } = limit.attributes;
      
      if (limit_value === null) continue;
      
      const percentage = (current_usage / limit_value) * 100;
      
      // Only generate warnings for usage above 70%
      if (percentage < 70) continue;
      
      let severity: LimitWarning['severity'];
      let message: string;
      let recommendation: string;
      
      if (percentage >= 95) {
        severity = 'critical';
        message = `${limit_type} usage critical: ${percentage.toFixed(1)}%`;
        recommendation = limit_is_hard
          ? 'Immediate action required. Operations will be blocked soon.'
          : 'Upgrade recommended to avoid potential issues.';
      } else if (percentage >= 90) {
        severity = 'high';
        message = `${limit_type} usage high: ${percentage.toFixed(1)}%`;
        recommendation = 'Consider upgrading your plan or reducing usage.';
      } else if (percentage >= 80) {
        severity = 'medium';
        message = `${limit_type} usage elevated: ${percentage.toFixed(1)}%`;
        recommendation = 'Monitor usage closely.';
      } else {
        severity = 'low';
        message = `${limit_type} usage moderate: ${percentage.toFixed(1)}%`;
        recommendation = 'Usage approaching limit.';
      }
      
      // Add reset information if applicable
      if (reset_at) {
        const resetDate = new Date(reset_at);
        const daysUntilReset = Math.ceil((resetDate.getTime() - Date.now()) / (1000 * 60 * 60 * 24));
        recommendation += ` Resets in ${daysUntilReset} days.`;
      }
      
      warnings.push({
        limitType: limit_type,
        severity,
        message,
        currentUsage: current_usage,
        limit: limit_value,
        percentage,
        recommendation
      });
    }
    
    return warnings.sort((a, b) => b.percentage - a.percentage);
  } catch (error) {
    console.error('Error generating limit warnings:', error);
    throw error;
  }
}
```

## Limit Exceeded Handling

### Graceful Degradation

```typescript
// Handle operations when limits are exceeded
class LimitExceededHandler {
  constructor(private client: any) {}
  
  async handleLimitExceeded(
    limitType: string,
    operation: string,
    fallbackStrategy?: () => Promise<void>
  ) {
    try {
      const limits = await this.client.subscriptionLimits.list();
      const limit = limits.find(l => l.attributes.limit_type === limitType);
      
      if (!limit) return;
      
      const { current_usage, limit_value, limit_is_hard } = limit.attributes;
      
      if (limit_value && current_usage >= limit_value) {
        console.error(`${limitType} limit exceeded: ${current_usage}/${limit_value}`);
        
        if (limit_is_hard) {
          // Hard limit - operation will fail
          throw new Error(
            `Hard limit exceeded for ${limitType}. ` +
            `Current usage: ${current_usage}, Limit: ${limit_value}. ` +
            `Operation '${operation}' cannot proceed.`
          );
        } else {
          // Soft limit - warn and try fallback
          console.warn(
            `Soft limit exceeded for ${limitType}. ` +
            `Current usage: ${current_usage}, Limit: ${limit_value}.`
          );
          
          if (fallbackStrategy) {
            console.log('Executing fallback strategy...');
            await fallbackStrategy();
          }
        }
      }
    } catch (error) {
      console.error('Error handling limit exceeded:', error);
      throw error;
    }
  }
}

// Usage example
const limitHandler = new LimitExceededHandler(client);

async function createItemWithLimitHandling(itemData: any) {
  try {
    // Check limits first
    await limitHandler.handleLimitExceeded(
      'items',
      'create_item',
      async () => {
        // Fallback: Clean up old items
        console.log('Attempting to free up space by removing old drafts...');
        const oldDrafts = await client.items.list({
          filter: {
            type: 'draft',
            fields: {
              updated_at: {
                lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString() // 30 days old
              }
            }
          },
          page: { limit: 10 }
        });
        
        for (const draft of oldDrafts) {
          await client.items.destroy(draft.id);
        }
      }
    );
    
    // Proceed with creation
    return await client.items.create(itemData);
  } catch (error) {
    console.error('Failed to create item:', error);
    throw error;
  }
}
```

## Error Handling Examples

### Comprehensive Error Handling

```typescript
// Enhanced error handling for limit operations
async function safeListLimits() {
  try {
    const limits = await client.subscriptionLimits.list();
    return { success: true, data: limits };
  } catch (error: any) {
    // Handle specific error types
    if (error.response?.status === 401) {
      return {
        success: false,
        error: 'Authentication failed. Check your API token.',
        code: 'AUTH_ERROR'
      };
    } else if (error.response?.status === 429) {
      return {
        success: false,
        error: 'Rate limit exceeded. Try again later.',
        code: 'RATE_LIMIT',
        retryAfter: error.response.headers['retry-after']
      };
    } else if (error.response?.status === 500) {
      return {
        success: false,
        error: 'Server error. Please try again.',
        code: 'SERVER_ERROR'
      };
    } else if (error.code === 'ECONNREFUSED') {
      return {
        success: false,
        error: 'Connection failed. Check your internet connection.',
        code: 'CONNECTION_ERROR'
      };
    }
    
    // Generic error
    return {
      success: false,
      error: error.message || 'Unknown error occurred',
      code: 'UNKNOWN_ERROR'
    };
  }
}

// Retry logic for transient failures
async function listLimitsWithRetry(maxRetries: number = 3) {
  let lastError: any;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const limits = await client.subscriptionLimits.list();
      return limits;
    } catch (error: any) {
      lastError = error;
      
      // Don't retry on authentication errors
      if (error.response?.status === 401) {
        throw error;
      }
      
      // Calculate delay with exponential backoff
      const delay = Math.min(1000 * Math.pow(2, attempt - 1), 10000);
      console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
      
      if (attempt < maxRetries) {
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries} attempts: ${lastError.message}`);
}
```

## Best Practices for Limit Management

### 1. Implement Predictive Monitoring

```typescript
// Predict when limits will be reached
interface LimitPrediction {
  limitType: string;
  currentUsage: number;
  limit: number;
  averageDailyUsage: number;
  daysUntilLimit: number;
  predictedExhaustion: Date;
}

async function predictLimitExhaustion(
  historicalData: { date: Date; usage: number }[]
): Promise<LimitPrediction[]> {
  try {
    const limits = await client.subscriptionLimits.list();
    const predictions: LimitPrediction[] = [];
    
    for (const limit of limits) {
      const { limit_type, current_usage, limit_value } = limit.attributes;
      
      if (limit_value === null || historicalData.length < 7) continue;
      
      // Calculate average daily usage from historical data
      const lastWeekData = historicalData.slice(-7);
      const weeklyIncrease = lastWeekData[lastWeekData.length - 1].usage - lastWeekData[0].usage;
      const averageDailyUsage = weeklyIncrease / 7;
      
      if (averageDailyUsage <= 0) continue;
      
      const remainingCapacity = limit_value - current_usage;
      const daysUntilLimit = Math.floor(remainingCapacity / averageDailyUsage);
      const predictedExhaustion = new Date(Date.now() + daysUntilLimit * 24 * 60 * 60 * 1000);
      
      predictions.push({
        limitType: limit_type,
        currentUsage: current_usage,
        limit: limit_value,
        averageDailyUsage,
        daysUntilLimit,
        predictedExhaustion
      });
    }
    
    return predictions.sort((a, b) => a.daysUntilLimit - b.daysUntilLimit);
  } catch (error) {
    console.error('Error predicting limit exhaustion:', error);
    throw error;
  }
}
```

### 2. Create Usage Budgets

```typescript
// Allocate usage budgets for different operations
class UsageBudgetManager {
  private budgets: Map<string, number> = new Map();
  
  constructor(private client: any) {}
  
  async initializeBudgets() {
    try {
      const limits = await this.client.subscriptionLimits.list();
      
      for (const limit of limits) {
        const { limit_type, limit_value, current_usage } = limit.attributes;
        
        if (limit_value === null) continue;
        
        const remaining = limit_value - current_usage;
        
        // Allocate 80% of remaining capacity
        const budget = Math.floor(remaining * 0.8);
        this.budgets.set(limit_type, budget);
      }
    } catch (error) {
      console.error('Error initializing budgets:', error);
      throw error;
    }
  }
  
  canUse(limitType: string, amount: number): boolean {
    const budget = this.budgets.get(limitType) || 0;
    return budget >= amount;
  }
  
  use(limitType: string, amount: number): boolean {
    const budget = this.budgets.get(limitType) || 0;
    
    if (budget >= amount) {
      this.budgets.set(limitType, budget - amount);
      return true;
    }
    
    return false;
  }
  
  getRemaining(limitType: string): number {
    return this.budgets.get(limitType) || 0;
  }
}
```

### 3. Implement Caching Strategy

```typescript
// Cache limit data to reduce API calls
class LimitCache {
  private cache: Map<string, { data: any; timestamp: number }> = new Map();
  private ttl: number = 5 * 60 * 1000; // 5 minutes
  
  constructor(private client: any) {}
  
  async getLimits(forceRefresh: boolean = false): Promise<any[]> {
    const cacheKey = 'subscription_limits';
    const cached = this.cache.get(cacheKey);
    
    if (!forceRefresh && cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.data;
    }
    
    try {
      const limits = await this.client.subscriptionLimits.list();
      this.cache.set(cacheKey, {
        data: limits,
        timestamp: Date.now()
      });
      return limits;
    } catch (error) {
      // Return cached data on error if available
      if (cached) {
        console.warn('Using cached limit data due to error:', error);
        return cached.data;
      }
      throw error;
    }
  }
  
  invalidate() {
    this.cache.clear();
  }
}
```

## Related Resources

- [Subscription Limit Resource](./subscription-limit.md) - Detailed information about limit attributes
- [Usage Counter Methods](./usage-counter-methods.md) - Track detailed usage metrics
- [Daily Usage Methods](./daily-usage-methods.md) - Historical usage data
- [Subscription Feature Methods](./subscription-feature-methods.md) - Available features per plan
- [Site Resource](../04-site-configuration/site.md) - Overall site configuration

## Common Limit Types

- **items**: Total content items in the project
- **uploads**: Total media uploads
- **api_calls**: API requests per month (resets monthly)
- **build_triggers**: Build webhook triggers per month
- **traffic_bytes**: CDN traffic per month
- **mux_encoding_seconds**: Video encoding time
- **mux_streaming_seconds**: Video streaming time

## Summary

The subscription limits API provides essential functionality for monitoring and managing resource usage within DatoCMS. By implementing proper limit checking, monitoring, and handling strategies, you can ensure smooth operation of your applications while staying within subscription boundaries. Always check limits proactively, implement appropriate warning systems, and handle both hard and soft limits gracefully to provide the best user experience.