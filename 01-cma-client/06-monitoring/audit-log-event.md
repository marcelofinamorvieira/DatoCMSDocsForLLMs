# Audit Log Event

Audit Log Events provide a comprehensive record of all activities in your DatoCMS project. This includes user actions, API calls, content changes, and system events, helping you maintain security, compliance, and operational awareness.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Query recent audit events
const events = await client.auditLogEvents.query({
  filter: {
    start_date: '2024-01-01T00:00:00Z',
    end_date: '2024-01-31T23:59:59Z'
  }
});
```

## API Reference

### Query Audit Log Events

Retrieve audit log events based on various filters. This is a POST request that allows complex filtering.

```typescript
// Query events for a specific date range
const dateRangeEvents = await client.auditLogEvents.query({
  filter: {
    start_date: '2024-01-01T00:00:00Z',
    end_date: '2024-01-07T23:59:59Z'
  }
});

// Query events by action type
const createEvents = await client.auditLogEvents.query({
  filter: {
    action_type: 'create',
    resource_type: 'item'
  }
});

// Query events by user
const userEvents = await client.auditLogEvents.query({
  filter: {
    actor_type: 'user',
    actor_id: 'user-id-123'
  }
});

// Get detailed logs
const detailedEvents = await client.auditLogEvents.query({
  filter: {
    start_date: '2024-01-01T00:00:00Z'
  },
  detailed_log: true
});

// Paginated query
const firstPage = await client.auditLogEvents.query({
  filter: {
    start_date: '2024-01-01T00:00:00Z'
  }
});

// Get next page using token
if (firstPage.meta.next_token) {
  const nextPage = await client.auditLogEvents.query({
    filter: {
      start_date: '2024-01-01T00:00:00Z'
    },
    next_token: firstPage.meta.next_token
  });
}
```

**Parameters:**
- `filter` (object, required): Query filters
  - `start_date` (string): ISO 8601 start date
  - `end_date` (string): ISO 8601 end date
  - `action_type` (string): Type of action performed
  - `resource_type` (string): Type of resource affected
  - `resource_id` (string): Specific resource ID
  - `actor_type` (string): Type of actor (user, api_token, etc.)
  - `actor_id` (string): Specific actor ID
  - `environment` (string): Environment ID
- `detailed_log` (boolean, optional): Include detailed change information
- `next_token` (string, optional): Pagination token

**Returns:** Array of audit log events with pagination metadata

## Event Types

### User Actions

Track what users are doing:

```typescript
async function trackUserActivity(userId: string, days = 7) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const events = await client.auditLogEvents.query({
    filter: {
      actor_type: 'user',
      actor_id: userId,
      start_date: startDate.toISOString()
    }
  });
  
  // Group by action type
  const actions = {};
  events.forEach(event => {
    const key = `${event.action_type}_${event.resource_type}`;
    actions[key] = (actions[key] || 0) + 1;
  });
  
  return {
    userId,
    period: `${days} days`,
    totalActions: events.length,
    actionBreakdown: actions
  };
}
```

### Content Changes

Monitor content modifications:

```typescript
async function getContentChanges(itemTypeId?: string) {
  const filter: any = {
    resource_type: 'item',
    action_type: { in: ['create', 'update', 'delete', 'publish', 'unpublish'] },
    start_date: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
  };
  
  if (itemTypeId) {
    // Filter by specific model
    filter.resource_attributes = { item_type: itemTypeId };
  }
  
  const events = await client.auditLogEvents.query({
    filter,
    detailed_log: true
  });
  
  return events.map(event => ({
    action: event.action_type,
    item: event.resource_id,
    user: event.actor,
    timestamp: event.created_at,
    changes: event.detailed_log
  }));
}
```

### Schema Changes

Track model and field modifications:

```typescript
async function getSchemaChanges() {
  const events = await client.auditLogEvents.query({
    filter: {
      resource_type: { in: ['item_type', 'field', 'fieldset'] },
      start_date: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString()
    }
  });
  
  const changes = events.map(event => ({
    date: new Date(event.created_at).toLocaleDateString(),
    user: event.actor?.email || event.actor?.name || 'System',
    action: event.action_type,
    resourceType: event.resource_type,
    resourceName: event.resource_attributes?.name || event.resource_attributes?.label,
    resourceId: event.resource_id
  }));
  
  return changes;
}
```

### API Token Usage

Monitor API token activity:

```typescript
async function auditApiTokenUsage() {
  const events = await client.auditLogEvents.query({
    filter: {
      actor_type: 'access_token',
      start_date: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString()
    }
  });
  
  // Group by token
  const tokenUsage = {};
  events.forEach(event => {
    const tokenId = event.actor_id;
    if (!tokenUsage[tokenId]) {
      tokenUsage[tokenId] = {
        name: event.actor?.name || 'Unknown Token',
        actions: [],
        resources: new Set()
      };
    }
    
    tokenUsage[tokenId].actions.push(event.action_type);
    tokenUsage[tokenId].resources.add(event.resource_type);
  });
  
  // Convert to summary
  return Object.entries(tokenUsage).map(([id, data]) => ({
    tokenId: id,
    tokenName: data.name,
    totalActions: data.actions.length,
    uniqueResourceTypes: Array.from(data.resources),
    mostCommonAction: getMostCommon(data.actions)
  }));
}

function getMostCommon(arr: string[]): string {
  const counts = arr.reduce((acc, val) => {
    acc[val] = (acc[val] || 0) + 1;
    return acc;
  }, {});
  
  return Object.entries(counts)
    .sort(([, a], [, b]) => b - a)[0][0];
}
```

## Security Monitoring

### Suspicious Activity Detection

Identify unusual patterns:

```typescript
class SecurityMonitor {
  static async detectSuspiciousActivity() {
    const alerts = [];
    
    // Check for mass deletions
    const deletions = await client.auditLogEvents.query({
      filter: {
        action_type: 'delete',
        start_date: new Date(Date.now() - 60 * 60 * 1000).toISOString() // Last hour
      }
    });
    
    if (deletions.length > 10) {
      alerts.push({
        type: 'mass_deletion',
        severity: 'high',
        count: deletions.length,
        actors: [...new Set(deletions.map(e => e.actor?.email))]
      });
    }
    
    // Check for permission changes
    const permissionChanges = await client.auditLogEvents.query({
      filter: {
        resource_type: { in: ['role', 'access_token'] },
        action_type: { in: ['create', 'update'] },
        start_date: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
      }
    });
    
    if (permissionChanges.length > 0) {
      alerts.push({
        type: 'permission_change',
        severity: 'medium',
        count: permissionChanges.length,
        changes: permissionChanges.map(e => ({
          resource: e.resource_type,
          action: e.action_type,
          by: e.actor?.email
        }))
      });
    }
    
    // Check for failed login attempts (if available in audit)
    const failedAuth = await client.auditLogEvents.query({
      filter: {
        action_type: 'login_failed',
        start_date: new Date(Date.now() - 60 * 60 * 1000).toISOString()
      }
    });
    
    if (failedAuth.length > 5) {
      alerts.push({
        type: 'multiple_failed_logins',
        severity: 'high',
        count: failedAuth.length
      });
    }
    
    return alerts;
  }
}
```

### Access Pattern Analysis

Understand how your project is being accessed:

```typescript
async function analyzeAccessPatterns(days = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const events = await client.auditLogEvents.query({
    filter: {
      start_date: startDate.toISOString()
    }
  });
  
  // Time-based analysis
  const hourlyActivity = new Array(24).fill(0);
  const dailyActivity = {};
  const weeklyActivity = new Array(7).fill(0);
  
  events.forEach(event => {
    const date = new Date(event.created_at);
    
    // Hour of day
    hourlyActivity[date.getHours()]++;
    
    // Day of week (0 = Sunday)
    weeklyActivity[date.getDay()]++;
    
    // Daily totals
    const day = date.toISOString().split('T')[0];
    dailyActivity[day] = (dailyActivity[day] || 0) + 1;
  });
  
  // Find peak times
  const peakHour = hourlyActivity.indexOf(Math.max(...hourlyActivity));
  const peakDay = weeklyActivity.indexOf(Math.max(...weeklyActivity));
  const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
  
  return {
    totalEvents: events.length,
    averagePerDay: events.length / days,
    peakHour: `${peakHour}:00`,
    peakDayOfWeek: days[peakDay],
    hourlyDistribution: hourlyActivity,
    dailyTrend: dailyActivity
  };
}
```

## Compliance and Reporting

### Generate Audit Reports

Create compliance reports:

```typescript
async function generateComplianceReport(startDate: Date, endDate: Date) {
  const events = await client.auditLogEvents.query({
    filter: {
      start_date: startDate.toISOString(),
      end_date: endDate.toISOString()
    },
    detailed_log: true
  });
  
  const report = {
    period: {
      start: startDate.toISOString(),
      end: endDate.toISOString()
    },
    summary: {
      totalEvents: events.length,
      uniqueUsers: new Set(events.map(e => e.actor_id)).size,
      resourceTypes: {},
      actionTypes: {}
    },
    criticalEvents: [],
    userActivity: {},
    dataChanges: []
  };
  
  // Process events
  events.forEach(event => {
    // Count by resource type
    report.summary.resourceTypes[event.resource_type] = 
      (report.summary.resourceTypes[event.resource_type] || 0) + 1;
    
    // Count by action type
    report.summary.actionTypes[event.action_type] = 
      (report.summary.actionTypes[event.action_type] || 0) + 1;
    
    // Track user activity
    const userId = event.actor_id;
    if (!report.userActivity[userId]) {
      report.userActivity[userId] = {
        name: event.actor?.email || event.actor?.name,
        actions: 0,
        lastActive: event.created_at
      };
    }
    report.userActivity[userId].actions++;
    
    // Identify critical events
    const criticalActions = ['delete', 'bulk_destroy', 'permission_grant'];
    if (criticalActions.includes(event.action_type)) {
      report.criticalEvents.push({
        timestamp: event.created_at,
        action: event.action_type,
        resource: event.resource_type,
        actor: event.actor?.email,
        details: event.detailed_log
      });
    }
  });
  
  return report;
}
```

### Data Retention Compliance

Track data deletion for GDPR compliance:

```typescript
async function trackDataDeletion(startDate: Date) {
  const deletionEvents = await client.auditLogEvents.query({
    filter: {
      action_type: { in: ['delete', 'destroy', 'bulk_destroy'] },
      resource_type: { in: ['item', 'upload', 'user'] },
      start_date: startDate.toISOString()
    },
    detailed_log: true
  });
  
  const deletionLog = deletionEvents.map(event => ({
    date: event.created_at,
    resourceType: event.resource_type,
    resourceId: event.resource_id,
    deletedBy: event.actor?.email || 'System',
    reason: event.detailed_log?.reason || 'Not specified',
    affectedData: event.detailed_log?.attributes || {}
  }));
  
  return {
    period: `Since ${startDate.toISOString()}`,
    totalDeletions: deletionLog.length,
    byType: deletionLog.reduce((acc, log) => {
      acc[log.resourceType] = (acc[log.resourceType] || 0) + 1;
      return acc;
    }, {}),
    deletions: deletionLog
  };
}
```

## Advanced Patterns

### Change Tracking

Track specific resource changes over time:

```typescript
async function trackResourceHistory(resourceId: string, resourceType: string) {
  const events = await client.auditLogEvents.query({
    filter: {
      resource_id: resourceId,
      resource_type: resourceType
    },
    detailed_log: true
  });
  
  // Build change history
  const history = events
    .sort((a, b) => new Date(a.created_at).getTime() - new Date(b.created_at).getTime())
    .map(event => ({
      timestamp: event.created_at,
      action: event.action_type,
      by: event.actor?.email || 'System',
      changes: extractChanges(event.detailed_log),
      version: event.resource_attributes?.version
    }));
  
  return {
    resourceId,
    resourceType,
    created: history.find(h => h.action === 'create')?.timestamp,
    lastModified: history[history.length - 1]?.timestamp,
    totalChanges: history.length,
    history
  };
}

function extractChanges(detailedLog: any): any {
  if (!detailedLog?.changes) return {};
  
  // Extract before/after values
  const changes = {};
  Object.entries(detailedLog.changes).forEach(([field, values]) => {
    changes[field] = {
      before: values[0],
      after: values[1]
    };
  });
  
  return changes;
}
```

### Audit Log Archiving

Export audit logs for long-term storage:

```typescript
async function exportAuditLogs(startDate: Date, endDate: Date, format = 'json') {
  const allEvents = [];
  let nextToken = null;
  
  // Paginate through all results
  do {
    const response = await client.auditLogEvents.query({
      filter: {
        start_date: startDate.toISOString(),
        end_date: endDate.toISOString()
      },
      detailed_log: true,
      next_token: nextToken
    });
    
    allEvents.push(...response);
    nextToken = response.meta?.next_token;
  } while (nextToken);
  
  if (format === 'csv') {
    return convertToCSV(allEvents);
  }
  
  return {
    exportDate: new Date().toISOString(),
    period: {
      start: startDate.toISOString(),
      end: endDate.toISOString()
    },
    totalEvents: allEvents.length,
    events: allEvents
  };
}

function convertToCSV(events: any[]): string {
  const headers = [
    'Timestamp', 'Action', 'Resource Type', 'Resource ID',
    'Actor Type', 'Actor', 'Environment', 'IP Address'
  ];
  
  const rows = events.map(event => [
    event.created_at,
    event.action_type,
    event.resource_type,
    event.resource_id,
    event.actor_type,
    event.actor?.email || event.actor?.name || '',
    event.environment || '',
    event.ip_address || ''
  ]);
  
  return [headers, ...rows]
    .map(row => row.map(cell => `"${cell}"`).join(','))
    .join('\n');
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.auditLogEvents.query({
    filter: {
      // Missing required start_date
    }
  });
} catch (error) {
  if (error instanceof ApiError) {
    const dateError = error.findError('start_date');
    if (dateError?.code === 'VALIDATION_REQUIRED') {
      console.log('Start date is required for audit log queries');
    }
  }
}

// Handle permission errors
try {
  await client.auditLogEvents.query({
    filter: {
      start_date: new Date().toISOString()
    }
  });
} catch (error) {
  if (error instanceof ApiError && error.response.status === 403) {
    console.log('Insufficient permissions to access audit logs');
    console.log('Required: can_access_audit_log permission');
  }
}
```

## Audit Log Event Object Structure

```typescript
interface AuditLogEvent {
  id: string;
  type: 'audit_log_event';
  created_at: string; // ISO 8601 timestamp
  action_type: string; // 'create', 'update', 'delete', etc.
  resource_type: string; // 'item', 'field', 'user', etc.
  resource_id?: string;
  resource_attributes?: Record<string, any>;
  actor_type: 'user' | 'access_token' | 'sso_user' | 'account';
  actor_id: string;
  actor?: {
    id: string;
    email?: string;
    name?: string;
  };
  environment?: string;
  ip_address?: string;
  user_agent?: string;
  detailed_log?: {
    changes?: Record<string, [any, any]>; // [before, after]
    attributes?: Record<string, any>;
    reason?: string;
  };
}

// Pagination metadata
interface AuditLogResponse {
  data: AuditLogEvent[];
  meta: {
    next_token?: string;
  };
}
```

## Related Resources

- [Roles](../05-access-control/role.md) - Configure audit log access permissions
- [Users](../05-access-control/user.md) - Track user activities
- [Webhooks](./webhook.md) - Real-time event notifications
- [Security Best Practices](../../08-security-best-practices/index.md) - Security guide

---

# Audit Log Event Methods

This document provides comprehensive documentation for working with Audit Log Events in the DatoCMS Content Management API. Audit logs track all actions performed within your DatoCMS project for security, compliance, and monitoring purposes.

## Overview

The Audit Log Event resource provides methods to query and analyze the audit trail of activities in your DatoCMS project. Each event captures who performed an action, what action was taken, when it occurred, and relevant metadata.

## Available Methods

### list() - List audit log events

Lists all audit log events for the project with powerful filtering capabilities.

#### Method Signature

```typescript
async list(
  queryParams?: {
    filter?: {
      // Filter by event type
      type?: string | { eq?: string; neq?: string; in?: string[]; notIn?: string[] };
      
      // Filter by actor (user or API token)
      actor?: string | { eq?: string; neq?: string; in?: string[]; notIn?: string[] };
      
      // Filter by date range
      created_at?: {
        gte?: string; // Greater than or equal (ISO 8601)
        lte?: string; // Less than or equal (ISO 8601)
        gt?: string;  // Greater than (ISO 8601)
        lt?: string;  // Less than (ISO 8601)
      };
      
      // Filter by target resource
      target?: string | { eq?: string; neq?: string; in?: string[]; notIn?: string[] };
    };
    
    // Pagination
    page?: {
      offset?: number;
      limit?: number; // Max 100, default 20
    };
    
    // Sorting
    order_by?: 'created_at' | '-created_at'; // Default: -created_at (newest first)
  }
): Promise<AuditLogEvent[]>
```

#### Complete Examples

##### Basic Usage - List Recent Events

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function listRecentAuditEvents() {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  try {
    // List the 20 most recent audit events
    const events: AuditLogEvent[] = await client.auditLogEvents.list();
    
    console.log(`Found ${events.length} recent audit events`);
    
    events.forEach(event => {
      console.log({
        id: event.id,
        type: event.type,
        actionType: event.attributes.action_type,
        actor: event.attributes.actor,
        createdAt: event.attributes.created_at,
        metadata: event.attributes.metadata
      });
    });
    
    return events;
  } catch (error) {
    console.error('Error fetching audit events:', error);
    throw error;
  }
}
```

##### Filter by User Actions

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function getUserAuditTrail(userId: string) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  try {
    // Filter events by specific user
    const events: AuditLogEvent[] = await client.auditLogEvents.list({
      filter: {
        actor: {
          eq: `user:${userId}`
        }
      },
      page: {
        limit: 100 // Get up to 100 events
      }
    });
    
    console.log(`User ${userId} performed ${events.length} actions`);
    
    // Group events by action type
    const actionGroups = events.reduce((acc, event) => {
      const actionType = event.attributes.action_type;
      if (!acc[actionType]) {
        acc[actionType] = [];
      }
      acc[actionType].push(event);
      return acc;
    }, {} as Record<string, AuditLogEvent[]>);
    
    // Display summary
    Object.entries(actionGroups).forEach(([action, items]) => {
      console.log(`${action}: ${items.length} events`);
    });
    
    return events;
  } catch (error) {
    console.error('Error fetching user audit trail:', error);
    throw error;
  }
}
```

##### Filter by Date Range

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function getAuditEventsForDateRange(startDate: Date, endDate: Date) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  try {
    const events: AuditLogEvent[] = await client.auditLogEvents.list({
      filter: {
        created_at: {
          gte: startDate.toISOString(),
          lte: endDate.toISOString()
        }
      },
      order_by: 'created_at', // Oldest first
      page: {
        limit: 100
      }
    });
    
    console.log(`Found ${events.length} events between ${startDate} and ${endDate}`);
    
    // Create timeline summary
    const timeline = events.map(event => ({
      time: new Date(event.attributes.created_at).toLocaleString(),
      actor: event.attributes.actor,
      action: event.attributes.action_type,
      target: event.attributes.target_type,
      targetId: event.attributes.target_id
    }));
    
    timeline.forEach(entry => {
      console.log(`[${entry.time}] ${entry.actor} performed ${entry.action} on ${entry.target} (${entry.targetId})`);
    });
    
    return events;
  } catch (error) {
    console.error('Error fetching events for date range:', error);
    throw error;
  }
}
```

##### Filter by Event Type

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function getContentChanges() {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  try {
    // Filter for content-related events
    const events: AuditLogEvent[] = await client.auditLogEvents.list({
      filter: {
        type: {
          in: ['item.create', 'item.update', 'item.destroy', 'item.publish', 'item.unpublish']
        }
      },
      page: {
        limit: 50
      }
    });
    
    console.log(`Found ${events.length} content change events`);
    
    // Analyze content modifications
    const contentStats = {
      created: 0,
      updated: 0,
      deleted: 0,
      published: 0,
      unpublished: 0
    };
    
    events.forEach(event => {
      switch (event.attributes.action_type) {
        case 'item.create':
          contentStats.created++;
          break;
        case 'item.update':
          contentStats.updated++;
          break;
        case 'item.destroy':
          contentStats.deleted++;
          break;
        case 'item.publish':
          contentStats.published++;
          break;
        case 'item.unpublish':
          contentStats.unpublished++;
          break;
      }
    });
    
    console.log('Content change summary:', contentStats);
    
    return events;
  } catch (error) {
    console.error('Error fetching content changes:', error);
    throw error;
  }
}
```

## Response Structure

Each audit log event follows this structure:

```typescript
interface AuditLogEvent {
  id: string;
  type: 'audit_log_event';
  attributes: {
    // Timestamp when the event occurred
    created_at: string; // ISO 8601 format
    
    // Type of action performed
    action_type: string; // e.g., 'item.create', 'user.update'
    
    // Who performed the action
    actor: string; // Format: 'user:USER_ID' or 'access_token:TOKEN_ID' or 'system'
    
    // Target resource type
    target_type: string; // e.g., 'item', 'item_type', 'upload'
    
    // Target resource ID
    target_id: string;
    
    // Additional event metadata
    metadata: Record<string, any>;
    
    // IP address of the actor (if available)
    ip_address?: string;
    
    // User agent string (if available)
    user_agent?: string;
  };
}
```

## Event Types Reference

### Content Management Events

```typescript
// Item (Record) Events
'item.create'         // Record created
'item.update'         // Record updated
'item.destroy'        // Record deleted
'item.publish'        // Record published
'item.unpublish'      // Record unpublished
'item.bulk_publish'   // Multiple records published
'item.bulk_unpublish' // Multiple records unpublished
'item.bulk_destroy'   // Multiple records deleted

// Model Events
'item_type.create'    // Model created
'item_type.update'    // Model updated
'item_type.destroy'   // Model deleted
'item_type.reorder'   // Model position changed

// Field Events
'field.create'        // Field added to model
'field.update'        // Field configuration updated
'field.destroy'       // Field removed from model
'field.reorder'       // Field position changed
```

### Media Management Events

```typescript
// Upload Events
'upload.create'       // File uploaded
'upload.update'       // File metadata updated
'upload.destroy'      // File deleted
'upload.bulk_destroy' // Multiple files deleted

// Upload Collection Events
'upload_collection.create'   // Collection created
'upload_collection.update'   // Collection updated
'upload_collection.destroy'  // Collection deleted
```

### Access Control Events

```typescript
// User Events
'user.create'         // User invited
'user.update'         // User details updated
'user.destroy'        // User removed
'user.accept_invite'  // User accepted invitation

// Role Events
'role.create'         // Role created
'role.update'         // Role permissions updated
'role.destroy'        // Role deleted

// Access Token Events
'access_token.create'   // API token created
'access_token.update'   // API token updated
'access_token.destroy'  // API token revoked
```

### Configuration Events

```typescript
// Site Settings Events
'site.update'             // Project settings updated
'site.maintenance_mode'   // Maintenance mode toggled

// Environment Events
'environment.create'      // Environment created
'environment.update'      // Environment updated
'environment.destroy'     // Environment deleted
'environment.fork'        // Environment forked

// Plugin Events
'plugin.create'          // Plugin installed
'plugin.update'          // Plugin updated
'plugin.destroy'         // Plugin uninstalled

// Webhook Events
'webhook.create'         // Webhook created
'webhook.update'         // Webhook updated
'webhook.destroy'        // Webhook deleted
```

## Pagination Strategies

### Basic Pagination

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function paginateAuditLogs(pageSize: number = 50) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const allEvents: AuditLogEvent[] = [];
  let offset = 0;
  let hasMore = true;

  try {
    while (hasMore) {
      const events: AuditLogEvent[] = await client.auditLogEvents.list({
        page: {
          offset,
          limit: pageSize
        }
      });
      
      allEvents.push(...events);
      
      // Check if we've retrieved all events
      hasMore = events.length === pageSize;
      offset += pageSize;
      
      console.log(`Retrieved ${allEvents.length} events so far...`);
    }
    
    console.log(`Total events retrieved: ${allEvents.length}`);
    return allEvents;
  } catch (error) {
    console.error('Error during pagination:', error);
    throw error;
  }
}
```

### Time-Based Pagination for Large Datasets

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function timeBasedPagination(days: number = 30) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const allEvents: AuditLogEvent[] = [];
  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(endDate.getDate() - days);

  try {
    // Process data in daily chunks
    for (let d = 0; d < days; d++) {
      const dayStart = new Date(startDate);
      dayStart.setDate(startDate.getDate() + d);
      
      const dayEnd = new Date(dayStart);
      dayEnd.setDate(dayStart.getDate() + 1);
      
      console.log(`Processing ${dayStart.toDateString()}...`);
      
      let offset = 0;
      let hasMore = true;
      
      while (hasMore) {
        const events: AuditLogEvent[] = await client.auditLogEvents.list({
          filter: {
            created_at: {
              gte: dayStart.toISOString(),
              lt: dayEnd.toISOString()
            }
          },
          page: {
            offset,
            limit: 100
          }
        });
        
        allEvents.push(...events);
        hasMore = events.length === 100;
        offset += 100;
      }
    }
    
    console.log(`Total events for ${days} days: ${allEvents.length}`);
    return allEvents;
  } catch (error) {
    console.error('Error in time-based pagination:', error);
    throw error;
  }
}
```

## Compliance Reporting Patterns

### Security Audit Report

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

interface SecurityAuditReport {
  period: { start: Date; end: Date };
  totalEvents: number;
  uniqueActors: Set<string>;
  criticalActions: AuditLogEvent[];
  externalAccess: AuditLogEvent[];
  permissionChanges: AuditLogEvent[];
}

async function generateSecurityAuditReport(
  startDate: Date,
  endDate: Date
): Promise<SecurityAuditReport> {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const report: SecurityAuditReport = {
    period: { start: startDate, end: endDate },
    totalEvents: 0,
    uniqueActors: new Set(),
    criticalActions: [],
    externalAccess: [],
    permissionChanges: []
  };

  try {
    // Define critical action types
    const criticalActionTypes = [
      'user.destroy',
      'role.update',
      'role.destroy',
      'access_token.create',
      'site.update',
      'webhook.create',
      'webhook.update'
    ];

    // Fetch all events for the period
    let offset = 0;
    let hasMore = true;

    while (hasMore) {
      const events: AuditLogEvent[] = await client.auditLogEvents.list({
        filter: {
          created_at: {
            gte: startDate.toISOString(),
            lte: endDate.toISOString()
          }
        },
        page: {
          offset,
          limit: 100
        }
      });

      events.forEach(event => {
        report.totalEvents++;
        report.uniqueActors.add(event.attributes.actor);

        // Identify critical actions
        if (criticalActionTypes.includes(event.attributes.action_type)) {
          report.criticalActions.push(event);
        }

        // Track API token usage
        if (event.attributes.actor.startsWith('access_token:')) {
          report.externalAccess.push(event);
        }

        // Track permission changes
        if (event.attributes.action_type.includes('role') || 
            event.attributes.action_type.includes('user')) {
          report.permissionChanges.push(event);
        }
      });

      hasMore = events.length === 100;
      offset += 100;
    }

    // Generate report summary
    console.log('=== Security Audit Report ===');
    console.log(`Period: ${startDate.toISOString()} to ${endDate.toISOString()}`);
    console.log(`Total Events: ${report.totalEvents}`);
    console.log(`Unique Actors: ${report.uniqueActors.size}`);
    console.log(`Critical Actions: ${report.criticalActions.length}`);
    console.log(`API Token Actions: ${report.externalAccess.length}`);
    console.log(`Permission Changes: ${report.permissionChanges.length}`);

    // List critical actions
    if (report.criticalActions.length > 0) {
      console.log('\n=== Critical Actions ===');
      report.criticalActions.forEach(event => {
        console.log(`[${event.attributes.created_at}] ${event.attributes.actor} performed ${event.attributes.action_type}`);
      });
    }

    return report;
  } catch (error) {
    console.error('Error generating security audit report:', error);
    throw error;
  }
}
```

### Content Change Report

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

interface ContentChangeReport {
  period: { start: Date; end: Date };
  contentCreated: number;
  contentUpdated: number;
  contentDeleted: number;
  contentPublished: number;
  topContributors: Map<string, number>;
  changesByModel: Map<string, number>;
}

async function generateContentChangeReport(
  startDate: Date,
  endDate: Date
): Promise<ContentChangeReport> {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const report: ContentChangeReport = {
    period: { start: startDate, end: endDate },
    contentCreated: 0,
    contentUpdated: 0,
    contentDeleted: 0,
    contentPublished: 0,
    topContributors: new Map(),
    changesByModel: new Map()
  };

  try {
    // Fetch content-related events
    const contentEvents = await client.auditLogEvents.list({
      filter: {
        created_at: {
          gte: startDate.toISOString(),
          lte: endDate.toISOString()
        },
        type: {
          in: ['item.create', 'item.update', 'item.destroy', 'item.publish']
        }
      },
      page: {
        limit: 100
      }
    });

    // Process events
    contentEvents.forEach(event => {
      // Count by action type
      switch (event.attributes.action_type) {
        case 'item.create':
          report.contentCreated++;
          break;
        case 'item.update':
          report.contentUpdated++;
          break;
        case 'item.destroy':
          report.contentDeleted++;
          break;
        case 'item.publish':
          report.contentPublished++;
          break;
      }

      // Track contributors
      const actor = event.attributes.actor;
      report.topContributors.set(
        actor,
        (report.topContributors.get(actor) || 0) + 1
      );

      // Track changes by model
      const modelId = event.attributes.metadata?.item_type_id;
      if (modelId) {
        report.changesByModel.set(
          modelId,
          (report.changesByModel.get(modelId) || 0) + 1
        );
      }
    });

    // Sort top contributors
    const sortedContributors = Array.from(report.topContributors.entries())
      .sort((a, b) => b[1] - a[1])
      .slice(0, 10);

    console.log('=== Content Change Report ===');
    console.log(`Period: ${startDate.toISOString()} to ${endDate.toISOString()}`);
    console.log(`Content Created: ${report.contentCreated}`);
    console.log(`Content Updated: ${report.contentUpdated}`);
    console.log(`Content Deleted: ${report.contentDeleted}`);
    console.log(`Content Published: ${report.contentPublished}`);
    
    console.log('\n=== Top Contributors ===');
    sortedContributors.forEach(([actor, count]) => {
      console.log(`${actor}: ${count} actions`);
    });

    return report;
  } catch (error) {
    console.error('Error generating content change report:', error);
    throw error;
  }
}
```

## Error Handling Examples

### Comprehensive Error Handling

```typescript
import { buildClient, ApiError, AuditLogEvent } from '@datocms/cma-client-node';

async function fetchAuditLogsWithErrorHandling() {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  try {
    const events: AuditLogEvent[] = await client.auditLogEvents.list({
      page: { limit: 50 }
    });
    
    return events;
  } catch (error) {
    if (error instanceof ApiError) {
      // Handle specific API errors
      switch (error.response.status) {
        case 401:
          console.error('Authentication failed. Check your API token.');
          break;
        case 403:
          console.error('Access denied. Ensure your token has audit log permissions.');
          break;
        case 422:
          console.error('Invalid parameters:', error.response.body);
          break;
        case 429:
          console.error('Rate limit exceeded. Please try again later.');
          const retryAfter = error.response.headers['x-ratelimit-reset'];
          console.log(`Retry after: ${new Date(parseInt(retryAfter) * 1000)}`);
          break;
        case 500:
        case 502:
        case 503:
          console.error('Server error. Please try again later.');
          break;
        default:
          console.error(`API Error ${error.response.status}:`, error.message);
      }
    } else {
      // Handle non-API errors
      console.error('Unexpected error:', error);
    }
    
    throw error;
  }
}
```

### Retry Logic for Rate Limits

```typescript
import { buildClient, ApiError, AuditLogEvent } from '@datocms/cma-client-node';

async function fetchWithRetry(
  queryParams: any,
  maxRetries: number = 3
): Promise<AuditLogEvent[]> {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  let retries = 0;

  while (retries < maxRetries) {
    try {
      const events = await client.auditLogEvents.list(queryParams);
      return events;
    } catch (error) {
      if (error instanceof ApiError && error.response.status === 429) {
        retries++;
        
        if (retries >= maxRetries) {
          throw new Error('Max retries exceeded for rate limit');
        }
        
        // Calculate wait time
        const retryAfter = error.response.headers['x-ratelimit-reset'];
        const waitTime = retryAfter 
          ? (parseInt(retryAfter) * 1000) - Date.now()
          : 60000; // Default to 1 minute
        
        console.log(`Rate limited. Waiting ${waitTime}ms before retry ${retries}/${maxRetries}`);
        
        await new Promise(resolve => setTimeout(resolve, waitTime));
      } else {
        // Don't retry on other errors
        throw error;
      }
    }
  }
  
  throw new Error('Failed to fetch audit logs after retries');
}
```

## Best Practices for Audit Log Analysis

### 1. Efficient Filtering

```typescript
// DO: Use specific filters to reduce data transfer
const efficientQuery = await client.auditLogEvents.list({
  filter: {
    type: { in: ['item.create', 'item.update'] },
    created_at: { gte: oneWeekAgo.toISOString() }
  },
  page: { limit: 100 }
});

// DON'T: Fetch all events and filter in memory
const inefficientQuery = await client.auditLogEvents.list({
  page: { limit: 1000 } // Too large
});
```

### 2. Batch Processing for Analytics

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function analyzeAuditLogsInBatches(batchSize: number = 100) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const analytics = {
    totalEvents: 0,
    eventTypes: new Map<string, number>(),
    hourlyDistribution: new Array(24).fill(0),
    processedBatches: 0
  };

  try {
    let offset = 0;
    let hasMore = true;

    while (hasMore) {
      const batch: AuditLogEvent[] = await client.auditLogEvents.list({
        page: { offset, limit: batchSize }
      });

      // Process batch
      batch.forEach(event => {
        analytics.totalEvents++;
        
        // Count event types
        const type = event.attributes.action_type;
        analytics.eventTypes.set(type, (analytics.eventTypes.get(type) || 0) + 1);
        
        // Hourly distribution
        const hour = new Date(event.attributes.created_at).getHours();
        analytics.hourlyDistribution[hour]++;
      });

      analytics.processedBatches++;
      hasMore = batch.length === batchSize;
      offset += batchSize;

      // Log progress
      console.log(`Processed batch ${analytics.processedBatches}, total events: ${analytics.totalEvents}`);
    }

    return analytics;
  } catch (error) {
    console.error('Error in batch processing:', error);
    throw error;
  }
}
```

### 3. Real-time Monitoring

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function monitorRecentActivity(intervalMinutes: number = 5) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  let lastCheckTime = new Date();
  
  setInterval(async () => {
    const currentTime = new Date();
    
    try {
      const recentEvents = await client.auditLogEvents.list({
        filter: {
          created_at: {
            gt: lastCheckTime.toISOString()
          }
        },
        order_by: 'created_at'
      });

      if (recentEvents.length > 0) {
        console.log(`\n[${currentTime.toISOString()}] ${recentEvents.length} new events detected:`);
        
        recentEvents.forEach(event => {
          console.log(`- ${event.attributes.actor} performed ${event.attributes.action_type} at ${event.attributes.created_at}`);
        });
      }

      lastCheckTime = currentTime;
    } catch (error) {
      console.error('Error checking recent activity:', error);
    }
  }, intervalMinutes * 60 * 1000);
}
```

### 4. Anomaly Detection

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

interface AnomalyReport {
  unusualTimes: AuditLogEvent[];
  highVolumeActors: Map<string, number>;
  suspiciousPatterns: AuditLogEvent[];
}

async function detectAnomalies(days: number = 7): Promise<AnomalyReport> {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(endDate.getDate() - days);

  const report: AnomalyReport = {
    unusualTimes: [],
    highVolumeActors: new Map(),
    suspiciousPatterns: []
  };

  try {
    const events = await client.auditLogEvents.list({
      filter: {
        created_at: {
          gte: startDate.toISOString(),
          lte: endDate.toISOString()
        }
      },
      page: { limit: 100 }
    });

    // Track activity by actor
    const actorActivity = new Map<string, AuditLogEvent[]>();

    events.forEach(event => {
      const actor = event.attributes.actor;
      if (!actorActivity.has(actor)) {
        actorActivity.set(actor, []);
      }
      actorActivity.get(actor)!.push(event);

      // Check for unusual times (e.g., outside business hours)
      const hour = new Date(event.attributes.created_at).getHours();
      if (hour < 6 || hour > 22) {
        report.unusualTimes.push(event);
      }
    });

    // Identify high-volume actors
    const avgEventsPerActor = events.length / actorActivity.size;
    const threshold = avgEventsPerActor * 3; // 3x average

    actorActivity.forEach((actorEvents, actor) => {
      if (actorEvents.length > threshold) {
        report.highVolumeActors.set(actor, actorEvents.length);
      }

      // Check for suspicious patterns (rapid deletions)
      const deletions = actorEvents.filter(e => 
        e.attributes.action_type.includes('destroy')
      );

      if (deletions.length > 5) {
        report.suspiciousPatterns.push(...deletions);
      }
    });

    console.log('=== Anomaly Detection Report ===');
    console.log(`Unusual time activities: ${report.unusualTimes.length}`);
    console.log(`High-volume actors: ${report.highVolumeActors.size}`);
    console.log(`Suspicious patterns: ${report.suspiciousPatterns.length}`);

    return report;
  } catch (error) {
    console.error('Error detecting anomalies:', error);
    throw error;
  }
}
```

## Related Resources

- [User Methods](./user-methods.md) - Manage users mentioned in audit logs
- [Role Methods](./role-methods.md) - Understand permission changes
- [Access Token Methods](./access-token-methods.md) - Track API token usage
- [Item Methods](../01-content-management/item-methods.md) - Content management operations
- [Webhook Call Methods](./webhook-call-methods.md) - Monitor webhook executions
- [Build Event Methods](./build-event-methods.md) - Track build triggers

## Common Patterns

### Export Audit Logs to CSV

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';
import * as fs from 'fs';

async function exportAuditLogsToCSV(filename: string, days: number = 30) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(endDate.getDate() - days);

  const csvHeaders = [
    'Timestamp',
    'Actor',
    'Action',
    'Target Type',
    'Target ID',
    'IP Address',
    'User Agent'
  ].join(',');

  const stream = fs.createWriteStream(filename);
  stream.write(csvHeaders + '\n');

  try {
    let offset = 0;
    let hasMore = true;
    let totalExported = 0;

    while (hasMore) {
      const events: AuditLogEvent[] = await client.auditLogEvents.list({
        filter: {
          created_at: {
            gte: startDate.toISOString(),
            lte: endDate.toISOString()
          }
        },
        page: { offset, limit: 100 }
      });

      events.forEach(event => {
        const row = [
          event.attributes.created_at,
          event.attributes.actor,
          event.attributes.action_type,
          event.attributes.target_type,
          event.attributes.target_id,
          event.attributes.ip_address || '',
          event.attributes.user_agent || ''
        ].map(field => `"${field}"`).join(',');

        stream.write(row + '\n');
        totalExported++;
      });

      hasMore = events.length === 100;
      offset += 100;
    }

    stream.end();
    console.log(`Exported ${totalExported} audit log entries to ${filename}`);
  } catch (error) {
    console.error('Error exporting audit logs:', error);
    stream.end();
    throw error;
  }
}
```

### Track Schema Changes

```typescript
import { buildClient, AuditLogEvent } from '@datocms/cma-client-node';

async function trackSchemaChanges(days: number = 30) {
  const client = buildClient({ 
    apiToken: 'YOUR_API_TOKEN' 
  });

  const endDate = new Date();
  const startDate = new Date();
  startDate.setDate(endDate.getDate() - days);

  try {
    const schemaEvents = await client.auditLogEvents.list({
      filter: {
        created_at: {
          gte: startDate.toISOString(),
          lte: endDate.toISOString()
        },
        type: {
          in: [
            'item_type.create',
            'item_type.update',
            'item_type.destroy',
            'field.create',
            'field.update',
            'field.destroy'
          ]
        }
      },
      order_by: 'created_at'
    });

    console.log(`Found ${schemaEvents.length} schema changes in the last ${days} days`);

    // Group by model
    const changesByModel = new Map<string, AuditLogEvent[]>();

    schemaEvents.forEach(event => {
      const modelId = event.attributes.target_type === 'item_type' 
        ? event.attributes.target_id
        : event.attributes.metadata?.item_type_id;

      if (modelId) {
        if (!changesByModel.has(modelId)) {
          changesByModel.set(modelId, []);
        }
        changesByModel.get(modelId)!.push(event);
      }
    });

    // Display timeline
    changesByModel.forEach((changes, modelId) => {
      console.log(`\nModel ${modelId}:`);
      changes.forEach(event => {
        const time = new Date(event.attributes.created_at).toLocaleString();
        console.log(`  [${time}] ${event.attributes.action_type} by ${event.attributes.actor}`);
      });
    });

    return schemaEvents;
  } catch (error) {
    console.error('Error tracking schema changes:', error);
    throw error;
  }
}
```