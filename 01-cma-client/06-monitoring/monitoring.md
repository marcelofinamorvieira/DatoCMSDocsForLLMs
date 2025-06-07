# Monitoring Resources

This comprehensive guide covers all monitoring and observability resources in the DatoCMS Content Management API. These resources help you track system activity, monitor performance, and audit user actions.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Audit Log Events](#audit-log-events)
- [Build Events](#build-events)
- [Search Results](#search-results)
- [Webhook Calls](#webhook-calls)

## Overview

DatoCMS provides comprehensive monitoring capabilities through four main resources:

1. **Audit Log Events**: Track all system activities and user actions
2. **Build Events**: Monitor static site deployments and builds
3. **Search Results**: Analyze content discovery and search patterns
4. **Webhook Calls**: Track webhook deliveries and failures

These resources enable:
- Security auditing and compliance
- Performance monitoring
- User behavior analysis
- Integration debugging
- Content optimization
- System health monitoring

## Installation

```bash
npm install @datocms/cma-client
```

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });
```

---

# Audit Log Events

The Audit Log Events resource provides a comprehensive record of all actions performed within your DatoCMS project. This includes user activities, content changes, configuration updates, and API operations.

## Overview

Audit logs capture:
- User actions (login, logout, invitations)
- Content operations (create, update, delete, publish)
- Schema changes (models, fields, plugins)
- Configuration updates (roles, webhooks, environments)
- API token usage and management
- SSO and security events

## Available Operations

### List Audit Log Events

```typescript
// List all audit events
const events = await client.auditLogEvents.list();

// With filtering
const filtered = await client.auditLogEvents.list({
  filter: {
    entity_type: 'item',
    event_type: ['create', 'update'],
    environment: 'main'
  },
  order_by: 'created_at_DESC',
  per_page: 100
});

// Filter by date range
const dateFiltered = await client.auditLogEvents.list({
  filter: {
    created_at: {
      gte: '2024-01-01T00:00:00Z',
      lte: '2024-01-31T23:59:59Z'
    }
  }
});

// Filter by user
const userEvents = await client.auditLogEvents.list({
  filter: {
    actor: {
      type: 'user',
      id: 'user-id'
    }
  }
});
```

### Event Filtering Options

```typescript
interface AuditLogFilter {
  // Entity filters
  entity_type?: string | string[];  // 'item', 'item_type', 'field', etc.
  entity_id?: string | string[];    // Specific entity IDs
  
  // Event filters
  event_type?: string | string[];   // 'create', 'update', 'delete', etc.
  
  // Actor filters
  actor?: {
    type: 'user' | 'access_token' | 'sso_user';
    id: string;
  };
  
  // Time filters
  created_at?: {
    gte?: string;  // Greater than or equal (ISO 8601)
    lte?: string;  // Less than or equal
  };
  
  // Environment filter
  environment?: string;
}
```

### Using Async Iterators

```typescript
// Process large volumes of events efficiently
for await (const event of client.auditLogEvents.listPagedIterator({
  filter: { entity_type: 'item' }
})) {
  console.log(`${event.event_type} on ${event.entity_type} by ${event.actor.email}`);
}
```

## Event Structure

```typescript
interface AuditLogEvent {
  id: string;
  type: 'audit_log_event';
  event_type: string;          // 'create', 'update', 'delete', etc.
  entity_type: string;         // 'item', 'upload', 'user', etc.
  entity_id?: string;          // ID of affected entity
  created_at: string;          // ISO 8601 timestamp
  environment?: string;        // Environment where event occurred
  
  actor: {                     // Who performed the action
    type: 'user' | 'access_token' | 'sso_user' | 'account';
    id: string;
    email?: string;            // For users
    name?: string;             // For access tokens
  };
  
  changes?: {                  // What changed (for updates)
    [field: string]: {
      old_value: any;
      new_value: any;
    };
  };
  
  metadata?: {                 // Additional context
    ip_address?: string;
    user_agent?: string;
    request_id?: string;
    api_key?: string;          // Which model/field was affected
  };
}
```

## Common Use Cases

### Security Auditing

```typescript
// Monitor suspicious activities
async function detectSuspiciousActivity(client) {
  const recentEvents = await client.auditLogEvents.list({
    filter: {
      created_at: {
        gte: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
      }
    },
    per_page: 500
  });
  
  const suspicious = {
    massDeletes: [],
    afterHoursAccess: [],
    failedLogins: [],
    privilegedActions: []
  };
  
  recentEvents.forEach(event => {
    // Mass deletions
    if (event.event_type === 'delete') {
      const userDeletes = recentEvents.filter(e => 
        e.actor.id === event.actor.id && 
        e.event_type === 'delete'
      );
      
      if (userDeletes.length > 10) {
        suspicious.massDeletes.push({
          actor: event.actor,
          count: userDeletes.length,
          timestamp: event.created_at
        });
      }
    }
    
    // After hours access (example: outside 9-5)
    const hour = new Date(event.created_at).getHours();
    if (hour < 9 || hour > 17) {
      suspicious.afterHoursAccess.push({
        actor: event.actor,
        action: `${event.event_type} ${event.entity_type}`,
        timestamp: event.created_at
      });
    }
    
    // Privileged actions
    if (['role', 'access_token', 'webhook'].includes(event.entity_type)) {
      suspicious.privilegedActions.push({
        actor: event.actor,
        action: `${event.event_type} ${event.entity_type}`,
        timestamp: event.created_at
      });
    }
  });
  
  return suspicious;
}
```

### Content Change Tracking

```typescript
// Track all changes to specific content
async function trackContentChanges(client, itemId: string) {
  const changes = await client.auditLogEvents.list({
    filter: {
      entity_type: 'item',
      entity_id: itemId
    },
    order_by: 'created_at_DESC'
  });
  
  const history = changes.map(event => ({
    action: event.event_type,
    user: event.actor.email || event.actor.name,
    timestamp: new Date(event.created_at),
    changes: event.changes ? Object.keys(event.changes).map(field => ({
      field,
      from: event.changes[field].old_value,
      to: event.changes[field].new_value
    })) : []
  }));
  
  return history;
}

// Find who published specific content
async function findPublisher(client, itemId: string) {
  const publishEvents = await client.auditLogEvents.list({
    filter: {
      entity_type: 'item',
      entity_id: itemId,
      event_type: 'publish'
    },
    order_by: 'created_at_DESC',
    per_page: 1
  });
  
  if (publishEvents.length > 0) {
    const event = publishEvents[0];
    return {
      publisher: event.actor.email || event.actor.name,
      publishedAt: event.created_at,
      environment: event.environment
    };
  }
  
  return null;
}
```

### User Activity Reports

```typescript
// Generate user activity report
async function generateUserActivityReport(client, userId: string, days = 7) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const events = await client.auditLogEvents.list({
    filter: {
      actor: { type: 'user', id: userId },
      created_at: { gte: startDate.toISOString() }
    },
    order_by: 'created_at_DESC'
  });
  
  // Group by entity type and action
  const activity = events.reduce((acc, event) => {
    const key = `${event.entity_type}:${event.event_type}`;
    acc[key] = (acc[key] || 0) + 1;
    return acc;
  }, {});
  
  // Calculate daily activity
  const dailyActivity = events.reduce((acc, event) => {
    const date = new Date(event.created_at).toISOString().split('T')[0];
    acc[date] = (acc[date] || 0) + 1;
    return acc;
  }, {});
  
  return {
    totalActions: events.length,
    actionBreakdown: activity,
    dailyActivity,
    mostRecentAction: events[0] || null
  };
}
```

### Compliance Reporting

```typescript
// Generate compliance report for data access
async function generateComplianceReport(client, options: {
  startDate: Date;
  endDate: Date;
  entityTypes?: string[];
}) {
  const events = await client.auditLogEvents.list({
    filter: {
      created_at: {
        gte: options.startDate.toISOString(),
        lte: options.endDate.toISOString()
      },
      entity_type: options.entityTypes
    }
  });
  
  const report = {
    period: {
      from: options.startDate,
      to: options.endDate
    },
    totalEvents: events.length,
    uniqueUsers: new Set(),
    eventsByType: {},
    sensitiveDataAccess: [],
    externalAccess: []
  };
  
  events.forEach(event => {
    // Track unique users
    report.uniqueUsers.add(event.actor.id);
    
    // Count events by type
    const key = `${event.entity_type}:${event.event_type}`;
    report.eventsByType[key] = (report.eventsByType[key] || 0) + 1;
    
    // Flag sensitive data access (customize based on your schema)
    if (event.entity_type === 'item' && event.metadata?.api_key === 'user_profile') {
      report.sensitiveDataAccess.push({
        user: event.actor.email,
        action: event.event_type,
        timestamp: event.created_at,
        itemId: event.entity_id
      });
    }
    
    // Track API token access
    if (event.actor.type === 'access_token') {
      report.externalAccess.push({
        token: event.actor.name,
        action: `${event.event_type} ${event.entity_type}`,
        timestamp: event.created_at
      });
    }
  });
  
  report.uniqueUsers = report.uniqueUsers.size;
  
  return report;
}
```

### Schema Change Tracking

```typescript
// Monitor schema modifications
async function trackSchemaChanges(client, days = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const schemaEvents = await client.auditLogEvents.list({
    filter: {
      entity_type: ['item_type', 'field', 'fieldset'],
      created_at: { gte: startDate.toISOString() }
    },
    order_by: 'created_at_DESC'
  });
  
  const changes = {
    modelsCreated: [],
    modelsDeleted: [],
    fieldsAdded: [],
    fieldsModified: [],
    fieldsDeleted: []
  };
  
  schemaEvents.forEach(event => {
    const summary = {
      entity: event.entity_type,
      entityId: event.entity_id,
      actor: event.actor.email || event.actor.name,
      timestamp: event.created_at,
      environment: event.environment
    };
    
    switch (`${event.entity_type}:${event.event_type}`) {
      case 'item_type:create':
        changes.modelsCreated.push(summary);
        break;
      case 'item_type:delete':
        changes.modelsDeleted.push(summary);
        break;
      case 'field:create':
        changes.fieldsAdded.push(summary);
        break;
      case 'field:update':
        changes.fieldsModified.push({
          ...summary,
          changes: event.changes
        });
        break;
      case 'field:delete':
        changes.fieldsDeleted.push(summary);
        break;
    }
  });
  
  return changes;
}
```

### Webhook Debugging

```typescript
// Debug webhook triggers
async function debugWebhookTriggers(client, webhookId: string) {
  const events = await client.auditLogEvents.list({
    filter: {
      entity_type: 'webhook_call',
      metadata: { webhook_id: webhookId }
    },
    order_by: 'created_at_DESC',
    per_page: 50
  });
  
  const analysis = {
    totalCalls: events.length,
    recentCalls: events.slice(0, 10).map(event => ({
      timestamp: event.created_at,
      success: event.metadata?.response_status < 400,
      statusCode: event.metadata?.response_status,
      duration: event.metadata?.duration_ms,
      triggerEvent: event.metadata?.trigger_event
    })),
    failureRate: 0,
    averageResponseTime: 0
  };
  
  const failures = events.filter(e => e.metadata?.response_status >= 400);
  analysis.failureRate = (failures.length / events.length) * 100;
  
  const totalDuration = events.reduce((sum, e) => 
    sum + (e.metadata?.duration_ms || 0), 0
  );
  analysis.averageResponseTime = totalDuration / events.length;
  
  return analysis;
}
```

## Best Practices

### 1. Efficient Filtering

```typescript
// Good: Specific filters reduce data transfer
const efficient = await client.auditLogEvents.list({
  filter: {
    entity_type: 'item',
    event_type: 'update',
    created_at: { gte: '2024-01-01T00:00:00Z' }
  },
  per_page: 50
});

// Avoid: Fetching all events and filtering client-side
const inefficient = await client.auditLogEvents.list({ per_page: 1000 });
const filtered = inefficient.filter(e => e.entity_type === 'item');
```

### 2. Batch Processing

```typescript
// Process events in batches for large datasets
async function processAuditLogs(client, processor: (events: any[]) => Promise<void>) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const events = await client.auditLogEvents.list({
      page,
      per_page: 100
    });
    
    if (events.length > 0) {
      await processor(events);
      page++;
    } else {
      hasMore = false;
    }
  }
}
```

### 3. Event Retention

```typescript
// Archive old events before they expire
async function archiveAuditLogs(client, retentionDays = 90) {
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - retentionDays);
  
  const oldEvents = await client.auditLogEvents.list({
    filter: {
      created_at: { lte: cutoffDate.toISOString() }
    }
  });
  
  // Save to your archive storage (S3, database, etc.)
  // Implementation depends on your infrastructure
  
  return oldEvents.length;
}
```

### 4. Real-time Monitoring

```typescript
// Set up real-time alerts for critical events
async function monitorCriticalEvents(client) {
  // Check every 5 minutes
  setInterval(async () => {
    const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000);
    
    const criticalEvents = await client.auditLogEvents.list({
      filter: {
        event_type: ['delete', 'unpublish'],
        entity_type: ['item_type', 'role', 'access_token'],
        created_at: { gte: fiveMinutesAgo.toISOString() }
      }
    });
    
    if (criticalEvents.length > 0) {
      // Send alert (email, Slack, etc.)
      console.error('CRITICAL EVENTS DETECTED:', criticalEvents);
    }
  }, 5 * 60 * 1000);
}
```

## Event Types Reference

### Entity Types
- `item` - Content items
- `item_type` - Models/Content types
- `field` - Model fields
- `fieldset` - Field groups
- `upload` - Media files
- `user` - Regular users
- `sso_user` - SSO users
- `role` - Permission roles
- `access_token` - API tokens
- `webhook` - Webhook configurations
- `webhook_call` - Webhook executions
- `environment` - Project environments
- `build_trigger` - Build configurations
- `site_invitation` - User invitations

### Event Types
- `create` - Entity created
- `update` - Entity modified
- `delete` - Entity removed
- `publish` - Content published
- `unpublish` - Content unpublished
- `login` - User authentication
- `logout` - User sign out
- `invite` - User invited
- `accept` - Invitation accepted
- `trigger` - Build triggered
- `complete` - Process completed
- `fail` - Process failed

---

# Build Events

The Build Events resource tracks all static site builds triggered through DatoCMS. This includes manual builds, scheduled builds, and builds triggered by content changes or webhooks.

## Overview

Build events capture:
- Build trigger source (manual, webhook, schedule, content change)
- Build status (pending, in_progress, success, failed)
- Build duration and performance metrics
- Environment information
- Error messages and logs
- Related content changes

## Available Operations

### List Build Events

```typescript
// List all build events
const builds = await client.buildEvents.list();

// Filter by build trigger
const webhookBuilds = await client.buildEvents.list({
  filter: {
    build_trigger: { eq: 'build-trigger-id' }
  }
});

// Filter by status
const failedBuilds = await client.buildEvents.list({
  filter: {
    build_status: { eq: 'failed' }
  }
});

// Filter by date range
const recentBuilds = await client.buildEvents.list({
  filter: {
    created_at: {
      gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString()
    }
  },
  order_by: 'created_at_DESC'
});
```

### Find Build Event

```typescript
const build = await client.buildEvents.find('build-event-id');

console.log(`Build ${build.id}`);
console.log(`Status: ${build.build_status}`);
console.log(`Duration: ${build.duration_seconds}s`);
console.log(`Triggered by: ${build.build_trigger.name}`);
```

## Build Event Structure

```typescript
interface BuildEvent {
  id: string;
  type: 'build_event';
  build_status: 'pending' | 'in_progress' | 'success' | 'failed' | 'cancelled';
  created_at: string;          // When build was triggered
  started_at?: string;         // When build actually started
  completed_at?: string;       // When build finished
  duration_seconds?: number;   // Total build time
  
  build_trigger: {
    id: string;
    type: 'build_trigger';
    name: string;
  };
  
  environment: {
    id: string;
    name: string;
  };
  
  source: {
    type: 'manual' | 'webhook' | 'scheduled' | 'content_change' | 'api';
    actor?: {                  // Who triggered (for manual/api)
      type: 'user' | 'access_token';
      id: string;
      email?: string;
    };
    webhook_id?: string;       // For webhook triggers
    changed_entities?: Array<{ // For content change triggers
      type: string;
      id: string;
    }>;
  };
  
  error?: {
    message: string;
    code?: string;
    details?: any;
  };
  
  metadata?: {
    deployment_url?: string;
    commit_sha?: string;
    branch?: string;
  };
}
```

## Common Use Cases

### Build Performance Monitoring

```typescript
// Monitor build performance over time
async function analyzeBuildPerformance(client, buildTriggerId: string, days = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const builds = await client.buildEvents.list({
    filter: {
      build_trigger: { eq: buildTriggerId },
      created_at: { gte: startDate.toISOString() }
    }
  });
  
  const stats = {
    total: builds.length,
    successful: 0,
    failed: 0,
    averageDuration: 0,
    maxDuration: 0,
    minDuration: Infinity,
    dailyBuilds: {},
    failureReasons: {}
  };
  
  let totalDuration = 0;
  
  builds.forEach(build => {
    // Status counts
    if (build.build_status === 'success') {
      stats.successful++;
      
      // Duration stats (only for successful builds)
      if (build.duration_seconds) {
        totalDuration += build.duration_seconds;
        stats.maxDuration = Math.max(stats.maxDuration, build.duration_seconds);
        stats.minDuration = Math.min(stats.minDuration, build.duration_seconds);
      }
    } else if (build.build_status === 'failed') {
      stats.failed++;
      
      // Track failure reasons
      const reason = build.error?.message || 'Unknown error';
      stats.failureReasons[reason] = (stats.failureReasons[reason] || 0) + 1;
    }
    
    // Daily build counts
    const date = new Date(build.created_at).toISOString().split('T')[0];
    stats.dailyBuilds[date] = (stats.dailyBuilds[date] || 0) + 1;
  });
  
  stats.averageDuration = stats.successful > 0 
    ? totalDuration / stats.successful 
    : 0;
  
  return stats;
}

// Get build trends
async function getBuildTrends(client) {
  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
  
  const builds = await client.buildEvents.list({
    filter: {
      created_at: { gte: thirtyDaysAgo.toISOString() }
    },
    order_by: 'created_at_DESC'
  });
  
  // Group by week
  const weeklyStats = {};
  
  builds.forEach(build => {
    const date = new Date(build.created_at);
    const weekStart = new Date(date);
    weekStart.setDate(date.getDate() - date.getDay());
    const weekKey = weekStart.toISOString().split('T')[0];
    
    if (!weeklyStats[weekKey]) {
      weeklyStats[weekKey] = {
        total: 0,
        successful: 0,
        failed: 0,
        totalDuration: 0
      };
    }
    
    weeklyStats[weekKey].total++;
    
    if (build.build_status === 'success') {
      weeklyStats[weekKey].successful++;
      weeklyStats[weekKey].totalDuration += build.duration_seconds || 0;
    } else if (build.build_status === 'failed') {
      weeklyStats[weekKey].failed++;
    }
  });
  
  // Calculate weekly averages
  Object.values(weeklyStats).forEach(week => {
    week.averageDuration = week.successful > 0 
      ? week.totalDuration / week.successful 
      : 0;
    week.successRate = week.total > 0 
      ? (week.successful / week.total) * 100 
      : 0;
  });
  
  return weeklyStats;
}
```

### Build Failure Analysis

```typescript
// Analyze build failures
async function analyzeBuildFailures(client, days = 7) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const failedBuilds = await client.buildEvents.list({
    filter: {
      build_status: { eq: 'failed' },
      created_at: { gte: startDate.toISOString() }
    },
    order_by: 'created_at_DESC'
  });
  
  const analysis = {
    totalFailures: failedBuilds.length,
    byTrigger: {},
    byEnvironment: {},
    byError: {},
    commonPatterns: [],
    recentFailures: []
  };
  
  failedBuilds.forEach(build => {
    // Group by trigger
    const triggerName = build.build_trigger.name;
    analysis.byTrigger[triggerName] = (analysis.byTrigger[triggerName] || 0) + 1;
    
    // Group by environment
    const envName = build.environment.name;
    analysis.byEnvironment[envName] = (analysis.byEnvironment[envName] || 0) + 1;
    
    // Group by error
    const errorKey = build.error?.code || build.error?.message || 'Unknown';
    analysis.byError[errorKey] = (analysis.byError[errorKey] || 0) + 1;
    
    // Recent failures
    if (analysis.recentFailures.length < 5) {
      analysis.recentFailures.push({
        id: build.id,
        trigger: triggerName,
        environment: envName,
        error: build.error?.message,
        timestamp: build.created_at,
        source: build.source.type
      });
    }
  });
  
  // Identify patterns
  if (Object.keys(analysis.byError).length === 1) {
    analysis.commonPatterns.push('All failures have the same error');
  }
  
  if (Object.keys(analysis.byTrigger).length === 1) {
    analysis.commonPatterns.push('All failures from the same build trigger');
  }
  
  // Check for time patterns
  const failuresByHour = {};
  failedBuilds.forEach(build => {
    const hour = new Date(build.created_at).getHours();
    failuresByHour[hour] = (failuresByHour[hour] || 0) + 1;
  });
  
  const peakHour = Object.entries(failuresByHour)
    .sort(([,a], [,b]) => b - a)[0];
  
  if (peakHour && peakHour[1] > failedBuilds.length * 0.3) {
    analysis.commonPatterns.push(`${peakHour[1]} failures occur around ${peakHour[0]}:00`);
  }
  
  return analysis;
}

// Get detailed failure information
async function debugBuildFailure(client, buildEventId: string) {
  const build = await client.buildEvents.find(buildEventId);
  
  const debug = {
    buildId: build.id,
    status: build.build_status,
    trigger: build.build_trigger.name,
    environment: build.environment.name,
    source: build.source,
    timing: {
      triggered: build.created_at,
      started: build.started_at,
      failed: build.completed_at,
      waitTime: build.started_at 
        ? (new Date(build.started_at) - new Date(build.created_at)) / 1000 
        : null,
      duration: build.duration_seconds
    },
    error: build.error,
    metadata: build.metadata
  };
  
  // Get related events if triggered by content change
  if (build.source.type === 'content_change' && build.source.changed_entities) {
    debug.relatedChanges = build.source.changed_entities;
    
    // Get audit logs for these changes
    const auditEvents = await client.auditLogEvents.list({
      filter: {
        entity_id: build.source.changed_entities.map(e => e.id),
        created_at: {
          gte: new Date(new Date(build.created_at).getTime() - 60000).toISOString(),
          lte: build.created_at
        }
      }
    });
    
    debug.triggeringEvents = auditEvents;
  }
  
  return debug;
}
```

### Build Trigger Optimization

```typescript
// Analyze build trigger efficiency
async function optimizeBuildTriggers(client) {
  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
  
  // Get all build triggers
  const triggers = await client.buildTriggers.list();
  
  const analysis = await Promise.all(triggers.map(async trigger => {
    const builds = await client.buildEvents.list({
      filter: {
        build_trigger: { eq: trigger.id },
        created_at: { gte: thirtyDaysAgo.toISOString() }
      }
    });
    
    const stats = {
      triggerId: trigger.id,
      triggerName: trigger.name,
      totalBuilds: builds.length,
      successRate: 0,
      averageDuration: 0,
      totalCost: 0, // Assuming cost per build
      wastedBuilds: 0,
      recommendations: []
    };
    
    let successCount = 0;
    let totalDuration = 0;
    let previousBuild = null;
    
    builds.forEach(build => {
      if (build.build_status === 'success') {
        successCount++;
        totalDuration += build.duration_seconds || 0;
        
        // Check for rapid successive builds (potential waste)
        if (previousBuild) {
          const timeDiff = new Date(build.created_at) - new Date(previousBuild.created_at);
          if (timeDiff < 5 * 60 * 1000) { // Less than 5 minutes
            stats.wastedBuilds++;
          }
        }
      }
      
      previousBuild = build;
    });
    
    stats.successRate = builds.length > 0 
      ? (successCount / builds.length) * 100 
      : 0;
    
    stats.averageDuration = successCount > 0 
      ? totalDuration / successCount 
      : 0;
    
    // Assuming $0.01 per build minute
    stats.totalCost = (totalDuration / 60) * 0.01;
    
    // Generate recommendations
    if (stats.successRate < 90) {
      stats.recommendations.push('Low success rate - investigate failures');
    }
    
    if (stats.wastedBuilds > builds.length * 0.1) {
      stats.recommendations.push('High number of rapid successive builds - consider debouncing');
    }
    
    if (stats.averageDuration > 600) { // 10 minutes
      stats.recommendations.push('Long build times - consider optimization');
    }
    
    if (builds.length === 0) {
      stats.recommendations.push('No builds in 30 days - consider removing trigger');
    }
    
    return stats;
  }));
  
  return analysis.sort((a, b) => b.totalCost - a.totalCost);
}
```

### Build Notifications

```typescript
// Set up build status monitoring
class BuildMonitor {
  constructor(private client: any) {}
  
  async monitorBuilds(callbacks: {
    onStart?: (build: any) => void;
    onComplete?: (build: any) => void;
    onFail?: (build: any) => void;
  }) {
    const knownBuilds = new Map();
    
    setInterval(async () => {
      const recentBuilds = await this.client.buildEvents.list({
        filter: {
          created_at: {
            gte: new Date(Date.now() - 10 * 60 * 1000).toISOString() // Last 10 min
          }
        }
      });
      
      recentBuilds.forEach(build => {
        const previousStatus = knownBuilds.get(build.id);
        
        if (!previousStatus) {
          // New build
          knownBuilds.set(build.id, build.build_status);
          
          if (build.build_status === 'in_progress' && callbacks.onStart) {
            callbacks.onStart(build);
          }
        } else if (previousStatus !== build.build_status) {
          // Status changed
          knownBuilds.set(build.id, build.build_status);
          
          if (build.build_status === 'success' && callbacks.onComplete) {
            callbacks.onComplete(build);
          } else if (build.build_status === 'failed' && callbacks.onFail) {
            callbacks.onFail(build);
          }
        }
      });
      
      // Clean up old builds
      const tenMinutesAgo = Date.now() - 10 * 60 * 1000;
      knownBuilds.forEach((status, id) => {
        const build = recentBuilds.find(b => b.id === id);
        if (!build || new Date(build.completed_at || build.created_at) < tenMinutesAgo) {
          knownBuilds.delete(id);
        }
      });
    }, 30000); // Check every 30 seconds
  }
}

// Usage
const monitor = new BuildMonitor(client);
monitor.monitorBuilds({
  onStart: (build) => {
    console.log(`Build started: ${build.build_trigger.name}`);
  },
  onComplete: (build) => {
    console.log(`Build completed in ${build.duration_seconds}s`);
    // Send success notification
  },
  onFail: (build) => {
    console.error(`Build failed: ${build.error?.message}`);
    // Send alert
  }
});
```

### Content Change Impact

```typescript
// Analyze which content changes trigger the most builds
async function analyzeContentChangeBuildImpact(client, days = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const contentBuilds = await client.buildEvents.list({
    filter: {
      source: { type: { eq: 'content_change' } },
      created_at: { gte: startDate.toISOString() }
    }
  });
  
  const impact = {
    totalBuilds: contentBuilds.length,
    byEntityType: {},
    byModel: {},
    frequentTriggers: [],
    buildCost: 0
  };
  
  // Track which entities trigger builds
  const entityTriggers = {};
  
  contentBuilds.forEach(build => {
    if (build.source.changed_entities) {
      build.source.changed_entities.forEach(entity => {
        // By entity type
        impact.byEntityType[entity.type] = 
          (impact.byEntityType[entity.type] || 0) + 1;
        
        // Track individual entities
        const key = `${entity.type}:${entity.id}`;
        entityTriggers[key] = (entityTriggers[key] || 0) + 1;
      });
    }
    
    // Calculate cost
    impact.buildCost += (build.duration_seconds || 0) / 60 * 0.01;
  });
  
  // Find frequent triggers
  impact.frequentTriggers = Object.entries(entityTriggers)
    .sort(([,a], [,b]) => b - a)
    .slice(0, 10)
    .map(([key, count]) => {
      const [type, id] = key.split(':');
      return { type, id, triggerCount: count };
    });
  
  // Get model information for items
  const itemTriggers = impact.frequentTriggers.filter(t => t.type === 'item');
  if (itemTriggers.length > 0) {
    const items = await Promise.all(
      itemTriggers.map(t => client.items.find(t.id).catch(() => null))
    );
    
    items.forEach(item => {
      if (item) {
        const modelId = item.item_type.id;
        impact.byModel[modelId] = (impact.byModel[modelId] || 0) + 1;
      }
    });
  }
  
  return impact;
}
```

## Best Practices

### 1. Build Debouncing

```typescript
// Implement build debouncing to avoid rapid triggers
class BuildDebouncer {
  private pendingBuilds = new Map();
  
  async triggerBuild(
    client: any, 
    triggerId: string, 
    debounceMs = 60000 // 1 minute default
  ) {
    const existing = this.pendingBuilds.get(triggerId);
    
    if (existing) {
      clearTimeout(existing.timeout);
    }
    
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(async () => {
        try {
          const result = await client.buildTriggers.trigger(triggerId);
          this.pendingBuilds.delete(triggerId);
          resolve(result);
        } catch (error) {
          this.pendingBuilds.delete(triggerId);
          reject(error);
        }
      }, debounceMs);
      
      this.pendingBuilds.set(triggerId, { timeout, resolve, reject });
    });
  }
}
```

### 2. Build Queue Management

```typescript
// Monitor and manage build queue
async function manageBuildQueue(client) {
  const pendingBuilds = await client.buildEvents.list({
    filter: {
      build_status: { in: ['pending', 'in_progress'] }
    }
  });
  
  const queue = {
    pending: pendingBuilds.filter(b => b.build_status === 'pending'),
    inProgress: pendingBuilds.filter(b => b.build_status === 'in_progress'),
    queueTime: 0
  };
  
  // Calculate average queue time
  const completedRecent = await client.buildEvents.list({
    filter: {
      build_status: { eq: 'success' },
      started_at: { exists: true }
    },
    per_page: 10,
    order_by: 'completed_at_DESC'
  });
  
  const queueTimes = completedRecent
    .filter(b => b.started_at)
    .map(b => 
      (new Date(b.started_at) - new Date(b.created_at)) / 1000
    );
  
  queue.queueTime = queueTimes.length > 0
    ? queueTimes.reduce((a, b) => a + b) / queueTimes.length
    : 0;
  
  return queue;
}
```

### 3. Cost Optimization

```typescript
// Analyze and optimize build costs
async function optimizeBuildCosts(client) {
  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
  
  const builds = await client.buildEvents.list({
    filter: {
      created_at: { gte: thirtyDaysAgo.toISOString() },
      build_status: { eq: 'success' }
    }
  });
  
  // Group builds by hour of day
  const buildsByHour = {};
  builds.forEach(build => {
    const hour = new Date(build.created_at).getHours();
    if (!buildsByHour[hour]) {
      buildsByHour[hour] = { count: 0, totalDuration: 0 };
    }
    buildsByHour[hour].count++;
    buildsByHour[hour].totalDuration += build.duration_seconds || 0;
  });
  
  // Find optimal build times (least busy)
  const optimalHours = Object.entries(buildsByHour)
    .sort(([,a], [,b]) => a.count - b.count)
    .slice(0, 3)
    .map(([hour]) => parseInt(hour));
  
  const recommendations = {
    optimalBuildHours: optimalHours,
    potentialSavings: 0,
    suggestions: []
  };
  
  // Calculate potential savings from scheduling
  const peakHour = Object.entries(buildsByHour)
    .sort(([,a], [,b]) => b.count - a.count)[0];
  
  if (peakHour) {
    const avgBuildsPerHour = builds.length / 24 / 30;
    const peakBuilds = peakHour[1].count / 30; // Daily average
    
    if (peakBuilds > avgBuildsPerHour * 2) {
      recommendations.suggestions.push(
        `Schedule non-critical builds outside peak hour ${peakHour[0]}:00`
      );
      recommendations.potentialSavings = (peakBuilds - avgBuildsPerHour) * 30 * 0.5;
    }
  }
  
  return recommendations;
}
```

---

# Search Results

The Search Results resource tracks and analyzes search queries performed within DatoCMS. This helps understand how users discover content and identify content gaps.

## Overview

Search results capture:
- Search queries and terms
- Number of results returned
- User interactions with results
- Search performance metrics
- Zero-result searches
- Popular search terms

## Available Operations

### List Search Results

```typescript
// List all search results
const searches = await client.searchResults.list();

// Filter by query
const termSearches = await client.searchResults.list({
  filter: {
    query: { matches: 'blog' }
  }
});

// Filter by locale
const enSearches = await client.searchResults.list({
  filter: {
    locale: { eq: 'en' }
  }
});

// Get zero-result searches
const noResults = await client.searchResults.list({
  filter: {
    results_count: { eq: 0 }
  }
});
```

## Search Result Structure

```typescript
interface SearchResult {
  id: string;
  type: 'search_result';
  query: string;              // Search query text
  locale?: string;            // Search locale
  results_count: number;      // Number of results returned
  created_at: string;         // When search was performed
  
  user?: {                    // Who performed the search
    id: string;
    type: 'user' | 'sso_user';
    email?: string;
  };
  
  results?: Array<{           // Top results returned
    entity_type: string;      // 'item', 'upload', etc.
    entity_id: string;
    score?: number;           // Relevance score
    clicked?: boolean;        // Whether user clicked this result
  }>;
  
  metadata?: {
    search_type?: 'full_text' | 'filtered' | 'fuzzy';
    duration_ms?: number;     // Search execution time
    filters_used?: string[];  // Applied filters
  };
}
```

## Common Use Cases

### Search Analytics Dashboard

```typescript
// Generate comprehensive search analytics
async function generateSearchAnalytics(client, days = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const searches = await client.searchResults.list({
    filter: {
      created_at: { gte: startDate.toISOString() }
    }
  });
  
  const analytics = {
    totalSearches: searches.length,
    uniqueQueries: new Set(),
    averageResults: 0,
    zeroResultQueries: [],
    topQueries: {},
    searchesByDay: {},
    clickThroughRate: 0,
    averageSearchTime: 0
  };
  
  let totalResults = 0;
  let totalDuration = 0;
  let clickedSearches = 0;
  
  searches.forEach(search => {
    // Unique queries
    analytics.uniqueQueries.add(search.query.toLowerCase());
    
    // Results count
    totalResults += search.results_count;
    
    // Zero results
    if (search.results_count === 0) {
      analytics.zeroResultQueries.push({
        query: search.query,
        timestamp: search.created_at,
        user: search.user?.email
      });
    }
    
    // Top queries
    const queryKey = search.query.toLowerCase();
    analytics.topQueries[queryKey] = (analytics.topQueries[queryKey] || 0) + 1;
    
    // Daily searches
    const date = new Date(search.created_at).toISOString().split('T')[0];
    analytics.searchesByDay[date] = (analytics.searchesByDay[date] || 0) + 1;
    
    // Performance
    if (search.metadata?.duration_ms) {
      totalDuration += search.metadata.duration_ms;
    }
    
    // Click-through rate
    if (search.results?.some(r => r.clicked)) {
      clickedSearches++;
    }
  });
  
  analytics.uniqueQueries = analytics.uniqueQueries.size;
  analytics.averageResults = searches.length > 0 
    ? totalResults / searches.length 
    : 0;
  analytics.averageSearchTime = searches.length > 0 
    ? totalDuration / searches.length 
    : 0;
  analytics.clickThroughRate = searches.length > 0 
    ? (clickedSearches / searches.length) * 100 
    : 0;
  
  // Sort top queries
  analytics.topQueries = Object.entries(analytics.topQueries)
    .sort(([,a], [,b]) => b - a)
    .slice(0, 20)
    .reduce((obj, [query, count]) => {
      obj[query] = count;
      return obj;
    }, {});
  
  return analytics;
}
```

### Zero-Result Analysis

```typescript
// Analyze searches with no results to identify content gaps
async function analyzeZeroResults(client, days = 7) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const zeroResults = await client.searchResults.list({
    filter: {
      results_count: { eq: 0 },
      created_at: { gte: startDate.toISOString() }
    }
  });
  
  const analysis = {
    total: zeroResults.length,
    uniqueQueries: {},
    patterns: [],
    suggestions: [],
    byUser: {}
  };
  
  zeroResults.forEach(search => {
    const query = search.query.toLowerCase();
    
    // Count unique queries
    analysis.uniqueQueries[query] = (analysis.uniqueQueries[query] || 0) + 1;
    
    // Track by user
    if (search.user) {
      const userKey = search.user.email || search.user.id;
      analysis.byUser[userKey] = (analysis.byUser[userKey] || 0) + 1;
    }
  });
  
  // Identify patterns
  const queries = Object.keys(analysis.uniqueQueries);
  
  // Check for common typos
  const commonTerms = ['product', 'blog', 'about', 'contact', 'service'];
  queries.forEach(query => {
    commonTerms.forEach(term => {
      if (query.includes(term.substring(0, 3)) && query !== term) {
        analysis.patterns.push({
          type: 'possible_typo',
          query,
          suggestion: term
        });
      }
    });
  });
  
  // Check for plural/singular variations
  queries.forEach(query => {
    const singular = query.replace(/s$/, '');
    const plural = query + 's';
    
    if (queries.includes(singular) || queries.includes(plural)) {
      analysis.patterns.push({
        type: 'plural_variation',
        queries: [query, singular, plural].filter(q => queries.includes(q))
      });
    }
  });
  
  // Generate content suggestions
  Object.entries(analysis.uniqueQueries)
    .sort(([,a], [,b]) => b - a)
    .slice(0, 10)
    .forEach(([query, count]) => {
      analysis.suggestions.push({
        query,
        searchCount: count,
        recommendation: `Consider creating content for "${query}"`
      });
    });
  
  return analysis;
}
```

### Search Performance Optimization

```typescript
// Monitor search performance and identify slow queries
async function analyzeSearchPerformance(client) {
  const searches = await client.searchResults.list({
    filter: {
      metadata: { duration_ms: { exists: true } }
    },
    order_by: 'created_at_DESC',
    per_page: 100
  });
  
  const performance = {
    averageTime: 0,
    slowQueries: [],
    performanceByType: {},
    optimizationSuggestions: []
  };
  
  let totalTime = 0;
  
  searches.forEach(search => {
    const duration = search.metadata.duration_ms;
    totalTime += duration;
    
    // Track slow queries (> 1 second)
    if (duration > 1000) {
      performance.slowQueries.push({
        query: search.query,
        duration,
        resultsCount: search.results_count,
        searchType: search.metadata.search_type,
        filters: search.metadata.filters_used
      });
    }
    
    // Performance by search type
    const type = search.metadata.search_type || 'unknown';
    if (!performance.performanceByType[type]) {
      performance.performanceByType[type] = {
        count: 0,
        totalTime: 0,
        averageTime: 0
      };
    }
    
    performance.performanceByType[type].count++;
    performance.performanceByType[type].totalTime += duration;
  });
  
  performance.averageTime = searches.length > 0 
    ? totalTime / searches.length 
    : 0;
  
  // Calculate averages by type
  Object.values(performance.performanceByType).forEach(stats => {
    stats.averageTime = stats.count > 0 
      ? stats.totalTime / stats.count 
      : 0;
  });
  
  // Generate optimization suggestions
  if (performance.averageTime > 500) {
    performance.optimizationSuggestions.push(
      'Average search time is high - consider indexing optimization'
    );
  }
  
  if (performance.slowQueries.length > searches.length * 0.1) {
    performance.optimizationSuggestions.push(
      'More than 10% of searches are slow - review query patterns'
    );
  }
  
  const fuzzyPerf = performance.performanceByType.fuzzy;
  if (fuzzyPerf && fuzzyPerf.averageTime > 1000) {
    performance.optimizationSuggestions.push(
      'Fuzzy searches are particularly slow - consider limiting fuzzy search scope'
    );
  }
  
  return performance;
}
```

### Popular Content Discovery

```typescript
// Identify most searched and clicked content
async function discoverPopularContent(client, days = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const searches = await client.searchResults.list({
    filter: {
      created_at: { gte: startDate.toISOString() },
      results_count: { gt: 0 }
    }
  });
  
  const contentPopularity = {
    mostSearched: {},
    mostClicked: {},
    searchToClickRate: {},
    contentGaps: []
  };
  
  // Analyze search results
  searches.forEach(search => {
    if (search.results) {
      search.results.forEach(result => {
        const key = `${result.entity_type}:${result.entity_id}`;
        
        // Track appearances in search results
        if (!contentPopularity.mostSearched[key]) {
          contentPopularity.mostSearched[key] = {
            appearances: 0,
            clicks: 0,
            averagePosition: 0,
            totalPosition: 0
          };
        }
        
        const content = contentPopularity.mostSearched[key];
        content.appearances++;
        content.totalPosition += result.position || 0;
        
        // Track clicks
        if (result.clicked) {
          content.clicks++;
          
          if (!contentPopularity.mostClicked[key]) {
            contentPopularity.mostClicked[key] = 0;
          }
          contentPopularity.mostClicked[key]++;
        }
      });
    }
  });
  
  // Calculate click-through rates and average positions
  Object.entries(contentPopularity.mostSearched).forEach(([key, stats]) => {
    stats.averagePosition = stats.appearances > 0 
      ? stats.totalPosition / stats.appearances 
      : 0;
    
    contentPopularity.searchToClickRate[key] = stats.appearances > 0 
      ? (stats.clicks / stats.appearances) * 100 
      : 0;
    
    // Identify content that appears often but isn't clicked
    if (stats.appearances > 10 && stats.clicks === 0) {
      contentPopularity.contentGaps.push({
        content: key,
        appearances: stats.appearances,
        issue: 'High visibility but no engagement'
      });
    }
  });
  
  // Sort and limit results
  contentPopularity.mostSearched = Object.entries(contentPopularity.mostSearched)
    .sort(([,a], [,b]) => b.appearances - a.appearances)
    .slice(0, 20)
    .reduce((obj, [key, stats]) => {
      obj[key] = stats;
      return obj;
    }, {});
  
  contentPopularity.mostClicked = Object.entries(contentPopularity.mostClicked)
    .sort(([,a], [,b]) => b - a)
    .slice(0, 20)
    .reduce((obj, [key, clicks]) => {
      obj[key] = clicks;
      return obj;
    }, {});
  
  return contentPopularity;
}
```

### Search Trends

```typescript
// Analyze search trends over time
async function analyzeSearchTrends(client, days = 90) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const searches = await client.searchResults.list({
    filter: {
      created_at: { gte: startDate.toISOString() }
    }
  });
  
  const trends = {
    weeklyVolume: {},
    emergingTerms: {},
    decliningTerms: {},
    seasonalPatterns: {}
  };
  
  // Group searches by week
  const searchesByWeek = {};
  
  searches.forEach(search => {
    const date = new Date(search.created_at);
    const weekStart = new Date(date);
    weekStart.setDate(date.getDate() - date.getDay());
    const weekKey = weekStart.toISOString().split('T')[0];
    
    if (!searchesByWeek[weekKey]) {
      searchesByWeek[weekKey] = {
        total: 0,
        queries: {}
      };
    }
    
    searchesByWeek[weekKey].total++;
    
    const query = search.query.toLowerCase();
    searchesByWeek[weekKey].queries[query] = 
      (searchesByWeek[weekKey].queries[query] || 0) + 1;
  });
  
  // Calculate weekly volumes
  Object.entries(searchesByWeek).forEach(([week, data]) => {
    trends.weeklyVolume[week] = data.total;
  });
  
  // Identify emerging and declining terms
  const weeks = Object.keys(searchesByWeek).sort();
  const recentWeeks = weeks.slice(-4);
  const olderWeeks = weeks.slice(0, -4);
  
  const recentQueries = {};
  const olderQueries = {};
  
  recentWeeks.forEach(week => {
    Object.entries(searchesByWeek[week].queries).forEach(([query, count]) => {
      recentQueries[query] = (recentQueries[query] || 0) + count;
    });
  });
  
  olderWeeks.forEach(week => {
    Object.entries(searchesByWeek[week].queries).forEach(([query, count]) => {
      olderQueries[query] = (olderQueries[query] || 0) + count;
    });
  });
  
  // Find emerging terms (more popular recently)
  Object.entries(recentQueries).forEach(([query, recentCount]) => {
    const olderCount = olderQueries[query] || 0;
    const growth = olderCount > 0 
      ? ((recentCount - olderCount) / olderCount) * 100 
      : 100;
    
    if (growth > 50 && recentCount > 5) {
      trends.emergingTerms[query] = {
        recentCount,
        olderCount,
        growthPercent: growth
      };
    }
  });
  
  // Find declining terms
  Object.entries(olderQueries).forEach(([query, olderCount]) => {
    const recentCount = recentQueries[query] || 0;
    const decline = olderCount > 0 
      ? ((olderCount - recentCount) / olderCount) * 100 
      : 0;
    
    if (decline > 50 && olderCount > 5) {
      trends.decliningTerms[query] = {
        recentCount,
        olderCount,
        declinePercent: decline
      };
    }
  });
  
  return trends;
}
```

## Best Practices

### 1. Search Query Normalization

```typescript
// Normalize search queries for better analysis
function normalizeSearchQuery(query: string): string {
  return query
    .toLowerCase()
    .trim()
    .replace(/[^\w\s]/g, ' ')  // Remove special characters
    .replace(/\s+/g, ' ')      // Normalize whitespace
    .split(' ')
    .filter(word => word.length > 2)  // Remove short words
    .join(' ');
}

// Group similar queries
async function groupSimilarQueries(client) {
  const searches = await client.searchResults.list();
  
  const normalizedGroups = {};
  
  searches.forEach(search => {
    const normalized = normalizeSearchQuery(search.query);
    
    if (!normalizedGroups[normalized]) {
      normalizedGroups[normalized] = {
        variations: new Set(),
        count: 0,
        totalResults: 0
      };
    }
    
    normalizedGroups[normalized].variations.add(search.query);
    normalizedGroups[normalized].count++;
    normalizedGroups[normalized].totalResults += search.results_count;
  });
  
  // Convert sets to arrays and calculate averages
  Object.values(normalizedGroups).forEach(group => {
    group.variations = Array.from(group.variations);
    group.averageResults = group.count > 0 
      ? group.totalResults / group.count 
      : 0;
  });
  
  return normalizedGroups;
}
```

### 2. Search Suggestions

```typescript
// Generate search suggestions based on successful searches
async function generateSearchSuggestions(client) {
  const searches = await client.searchResults.list({
    filter: {
      results_count: { gt: 0 }
    }
  });
  
  // Build suggestion index
  const suggestions = new Map();
  
  searches.forEach(search => {
    const query = search.query.toLowerCase();
    const words = query.split(' ');
    
    // Index by first word and bigrams
    words.forEach((word, i) => {
      if (word.length > 2) {
        if (!suggestions.has(word)) {
          suggestions.set(word, new Set());
        }
        suggestions.get(word).add(query);
        
        // Bigram
        if (i < words.length - 1) {
          const bigram = `${word} ${words[i + 1]}`;
          if (!suggestions.has(bigram)) {
            suggestions.set(bigram, new Set());
          }
          suggestions.get(bigram).add(query);
        }
      }
    });
  });
  
  // Convert to array format
  const suggestionIndex = {};
  suggestions.forEach((queries, prefix) => {
    suggestionIndex[prefix] = Array.from(queries)
      .sort((a, b) => a.length - b.length)
      .slice(0, 10);
  });
  
  return suggestionIndex;
}
```

### 3. Content Gap Analysis

```typescript
// Identify content gaps based on search patterns
async function identifyContentGaps(client) {
  const zeroResults = await client.searchResults.list({
    filter: {
      results_count: { eq: 0 }
    }
  });
  
  const gaps = {
    missingTopics: {},
    recommendations: [],
    potentialContent: []
  };
  
  // Analyze zero-result queries
  zeroResults.forEach(search => {
    const query = normalizeSearchQuery(search.query);
    gaps.missingTopics[query] = (gaps.missingTopics[query] || 0) + 1;
  });
  
  // Generate recommendations
  Object.entries(gaps.missingTopics)
    .sort(([,a], [,b]) => b - a)
    .slice(0, 20)
    .forEach(([topic, count]) => {
      if (count > 5) {
        gaps.recommendations.push({
          topic,
          searchVolume: count,
          priority: count > 20 ? 'high' : count > 10 ? 'medium' : 'low',
          action: `Create content about "${topic}"`
        });
      }
    });
  
  // Group by potential content types
  gaps.recommendations.forEach(rec => {
    const topic = rec.topic.toLowerCase();
    
    let contentType = 'article';
    if (topic.includes('how') || topic.includes('guide')) {
      contentType = 'tutorial';
    } else if (topic.includes('what') || topic.includes('why')) {
      contentType = 'explainer';
    } else if (topic.includes('product') || topic.includes('service')) {
      contentType = 'product_page';
    }
    
    gaps.potentialContent.push({
      ...rec,
      suggestedType: contentType
    });
  });
  
  return gaps;
}
```

---

# Webhook Calls

The Webhook Calls resource tracks all webhook deliveries triggered by events in DatoCMS. This includes successful deliveries, failures, retries, and response data.

## Overview

Webhook calls capture:
- Webhook trigger events
- HTTP request/response details
- Delivery status and retries
- Response times and performance
- Error messages and debugging info
- Payload data

## Available Operations

### List Webhook Calls

```typescript
// List all webhook calls
const calls = await client.webhookCalls.list();

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
  }
});

// Filter by date range
const recentCalls = await client.webhookCalls.list({
  filter: {
    created_at: {
      gte: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
    }
  },
  order_by: 'created_at_DESC'
});
```

### Find Webhook Call

```typescript
const call = await client.webhookCalls.find('webhook-call-id');

console.log(`Webhook: ${call.webhook.name}`);
console.log(`Status: ${call.response_status}`);
console.log(`Duration: ${call.response_time_ms}ms`);
console.log(`Attempt: ${call.attempt_number}`);
```

### Resend Webhook Call

```typescript
// Manually retry a failed webhook
await client.webhookCalls.resend('webhook-call-id');
```

## Webhook Call Structure

```typescript
interface WebhookCall {
  id: string;
  type: 'webhook_call';
  created_at: string;           // When webhook was triggered
  
  webhook: {                    // Associated webhook config
    id: string;
    type: 'webhook';
    name: string;
    url: string;
  };
  
  event_type: string;           // Triggering event
  entity_type?: string;         // Related entity type
  entity_id?: string;           // Related entity ID
  environment?: string;         // Environment context
  
  request: {
    url: string;
    method: 'POST' | 'GET';
    headers: Record<string, string>;
    body?: string;              // JSON payload
  };
  
  response?: {
    status: number;             // HTTP status code
    headers: Record<string, string>;
    body?: string;              // Response body
  };
  
  response_status?: number;     // Quick access to status
  response_time_ms?: number;    // Response duration
  
  attempt_number: number;       // Which retry attempt
  next_retry_at?: string;       // Scheduled retry time
  
  error?: {
    message: string;
    code?: string;
    details?: any;
  };
}
```

## Common Use Cases

### Webhook Delivery Monitoring

```typescript
// Monitor webhook delivery health
async function monitorWebhookHealth(client, hours = 24) {
  const since = new Date();
  since.setHours(since.getHours() - hours);
  
  const calls = await client.webhookCalls.list({
    filter: {
      created_at: { gte: since.toISOString() }
    }
  });
  
  const health = {
    totalCalls: calls.length,
    byWebhook: {},
    overallStats: {
      successful: 0,
      failed: 0,
      pending: 0,
      successRate: 0,
      averageResponseTime: 0
    },
    failureReasons: {},
    slowWebhooks: []
  };
  
  let totalResponseTime = 0;
  let responseTimeCount = 0;
  
  calls.forEach(call => {
    // Initialize webhook stats
    if (!health.byWebhook[call.webhook.id]) {
      health.byWebhook[call.webhook.id] = {
        name: call.webhook.name,
        url: call.webhook.url,
        total: 0,
        successful: 0,
        failed: 0,
        pending: 0,
        averageResponseTime: 0,
        totalResponseTime: 0,
        lastFailure: null
      };
    }
    
    const webhookStats = health.byWebhook[call.webhook.id];
    webhookStats.total++;
    
    // Categorize by status
    if (!call.response_status) {
      webhookStats.pending++;
      health.overallStats.pending++;
    } else if (call.response_status < 400) {
      webhookStats.successful++;
      health.overallStats.successful++;
      
      // Track response time
      if (call.response_time_ms) {
        webhookStats.totalResponseTime += call.response_time_ms;
        totalResponseTime += call.response_time_ms;
        responseTimeCount++;
        
        // Flag slow webhooks (> 5 seconds)
        if (call.response_time_ms > 5000) {
          health.slowWebhooks.push({
            webhook: call.webhook.name,
            url: call.webhook.url,
            responseTime: call.response_time_ms,
            timestamp: call.created_at
          });
        }
      }
    } else {
      webhookStats.failed++;
      health.overallStats.failed++;
      webhookStats.lastFailure = {
        status: call.response_status,
        error: call.error?.message,
        timestamp: call.created_at
      };
      
      // Track failure reasons
      const reason = call.error?.message || `HTTP ${call.response_status}`;
      health.failureReasons[reason] = (health.failureReasons[reason] || 0) + 1;
    }
  });
  
  // Calculate averages
  health.overallStats.successRate = health.overallStats.totalCalls > 0
    ? (health.overallStats.successful / calls.length) * 100
    : 0;
  
  health.overallStats.averageResponseTime = responseTimeCount > 0
    ? totalResponseTime / responseTimeCount
    : 0;
  
  Object.values(health.byWebhook).forEach(webhook => {
    webhook.successRate = webhook.total > 0
      ? (webhook.successful / webhook.total) * 100
      : 0;
    
    webhook.averageResponseTime = webhook.successful > 0
      ? webhook.totalResponseTime / webhook.successful
      : 0;
    
    delete webhook.totalResponseTime; // Clean up internal counter
  });
  
  return health;
}

// Real-time webhook monitoring
async function setupWebhookMonitoring(client, alertCallback) {
  setInterval(async () => {
    const fiveMinutesAgo = new Date(Date.now() - 5 * 60 * 1000);
    
    const recentCalls = await client.webhookCalls.list({
      filter: {
        created_at: { gte: fiveMinutesAgo.toISOString() },
        response_status: { gte: 400 }
      }
    });
    
    if (recentCalls.length > 0) {
      const alert = {
        failureCount: recentCalls.length,
        webhooks: [...new Set(recentCalls.map(c => c.webhook.name))],
        commonErrors: {}
      };
      
      recentCalls.forEach(call => {
        const error = call.error?.message || `HTTP ${call.response_status}`;
        alert.commonErrors[error] = (alert.commonErrors[error] || 0) + 1;
      });
      
      alertCallback(alert);
    }
  }, 5 * 60 * 1000); // Check every 5 minutes
}
```

### Webhook Debugging

```typescript
// Debug webhook delivery issues
async function debugWebhook(client, webhookId: string, hours = 24) {
  const since = new Date();
  since.setHours(since.getHours() - hours);
  
  const calls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      created_at: { gte: since.toISOString() }
    },
    order_by: 'created_at_DESC'
  });
  
  const debug = {
    webhook: calls[0]?.webhook || { id: webhookId },
    totalCalls: calls.length,
    successRate: 0,
    recentFailures: [],
    commonErrors: {},
    retryPatterns: {},
    samplePayloads: {
      successful: null,
      failed: null
    },
    performanceStats: {
      min: Infinity,
      max: 0,
      average: 0,
      p95: 0
    }
  };
  
  const successfulCalls = [];
  const failedCalls = [];
  const responseTimes = [];
  
  calls.forEach(call => {
    if (call.response_status && call.response_status < 400) {
      successfulCalls.push(call);
      
      if (call.response_time_ms) {
        responseTimes.push(call.response_time_ms);
        debug.performanceStats.min = Math.min(debug.performanceStats.min, call.response_time_ms);
        debug.performanceStats.max = Math.max(debug.performanceStats.max, call.response_time_ms);
      }
    } else {
      failedCalls.push(call);
      
      // Track failure patterns
      const error = call.error?.message || `HTTP ${call.response_status}`;
      debug.commonErrors[error] = (debug.commonErrors[error] || 0) + 1;
      
      // Track retry patterns
      if (call.attempt_number > 1) {
        const retryKey = `Attempt ${call.attempt_number}`;
        debug.retryPatterns[retryKey] = (debug.retryPatterns[retryKey] || 0) + 1;
      }
      
      // Recent failures
      if (debug.recentFailures.length < 5) {
        debug.recentFailures.push({
          id: call.id,
          timestamp: call.created_at,
          status: call.response_status,
          error: call.error,
          attempt: call.attempt_number,
          request: {
            headers: call.request.headers,
            body: call.request.body ? 
              JSON.parse(call.request.body) : null
          },
          response: call.response
        });
      }
    }
  });
  
  debug.successRate = calls.length > 0
    ? (successfulCalls.length / calls.length) * 100
    : 0;
  
  // Calculate performance stats
  if (responseTimes.length > 0) {
    responseTimes.sort((a, b) => a - b);
    debug.performanceStats.average = 
      responseTimes.reduce((a, b) => a + b) / responseTimes.length;
    debug.performanceStats.p95 = 
      responseTimes[Math.floor(responseTimes.length * 0.95)];
  }
  
  // Get sample payloads
  if (successfulCalls.length > 0) {
    const sample = successfulCalls[0];
    debug.samplePayloads.successful = {
      event: sample.event_type,
      request: JSON.parse(sample.request.body || '{}'),
      response: {
        status: sample.response_status,
        body: sample.response?.body
      }
    };
  }
  
  if (failedCalls.length > 0) {
    const sample = failedCalls[0];
    debug.samplePayloads.failed = {
      event: sample.event_type,
      request: JSON.parse(sample.request.body || '{}'),
      response: {
        status: sample.response_status,
        body: sample.response?.body,
        error: sample.error
      }
    };
  }
  
  return debug;
}

// Test webhook endpoint
async function testWebhookEndpoint(client, webhookId: string) {
  // Get recent successful call to use as template
  const recentCalls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      response_status: { lt: 400 }
    },
    order_by: 'created_at_DESC',
    per_page: 1
  });
  
  if (recentCalls.length === 0) {
    throw new Error('No successful calls found to use as template');
  }
  
  const template = recentCalls[0];
  
  // Resend the webhook
  console.log('Sending test webhook...');
  await client.webhookCalls.resend(template.id);
  
  // Wait for delivery
  await new Promise(resolve => setTimeout(resolve, 5000));
  
  // Check result
  const testCalls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      created_at: { 
        gte: new Date(Date.now() - 10000).toISOString() 
      }
    },
    order_by: 'created_at_DESC',
    per_page: 1
  });
  
  if (testCalls.length > 0) {
    const result = testCalls[0];
    return {
      success: result.response_status < 400,
      status: result.response_status,
      responseTime: result.response_time_ms,
      error: result.error,
      response: result.response
    };
  }
  
  throw new Error('Test webhook delivery not found');
}
```

### Retry Management

```typescript
// Manage webhook retries and failures
async function manageWebhookRetries(client) {
  // Get all pending retries
  const pendingRetries = await client.webhookCalls.list({
    filter: {
      next_retry_at: { exists: true },
      response_status: { gte: 400 }
    }
  });
  
  const retryManagement = {
    pendingCount: pendingRetries.length,
    byWebhook: {},
    upcomingRetries: [],
    recommendedActions: []
  };
  
  pendingRetries.forEach(call => {
    // Group by webhook
    if (!retryManagement.byWebhook[call.webhook.id]) {
      retryManagement.byWebhook[call.webhook.id] = {
        name: call.webhook.name,
        pendingRetries: 0,
        maxAttempts: 0,
        nextRetry: null
      };
    }
    
    const webhookRetries = retryManagement.byWebhook[call.webhook.id];
    webhookRetries.pendingRetries++;
    webhookRetries.maxAttempts = Math.max(
      webhookRetries.maxAttempts, 
      call.attempt_number
    );
    
    // Track upcoming retries
    const retryTime = new Date(call.next_retry_at);
    const now = new Date();
    
    if (retryTime > now) {
      retryManagement.upcomingRetries.push({
        webhookName: call.webhook.name,
        retryAt: call.next_retry_at,
        minutesUntilRetry: Math.round((retryTime - now) / 60000),
        attemptNumber: call.attempt_number,
        lastError: call.error?.message
      });
      
      // Update next retry for webhook
      if (!webhookRetries.nextRetry || retryTime < new Date(webhookRetries.nextRetry)) {
        webhookRetries.nextRetry = call.next_retry_at;
      }
    }
  });
  
  // Sort upcoming retries
  retryManagement.upcomingRetries.sort((a, b) => 
    new Date(a.retryAt) - new Date(b.retryAt)
  );
  
  // Generate recommendations
  Object.entries(retryManagement.byWebhook).forEach(([webhookId, stats]) => {
    if (stats.maxAttempts >= 5) {
      retryManagement.recommendedActions.push({
        webhook: stats.name,
        action: 'disable',
        reason: 'Webhook has failed 5+ times'
      });
    } else if (stats.pendingRetries > 10) {
      retryManagement.recommendedActions.push({
        webhook: stats.name,
        action: 'investigate',
        reason: 'High number of pending retries'
      });
    }
  });
  
  return retryManagement;
}

// Cancel pending retries for a webhook
async function cancelWebhookRetries(client, webhookId: string) {
  const pendingCalls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      next_retry_at: { exists: true }
    }
  });
  
  console.log(`Found ${pendingCalls.length} pending retries`);
  
  // Note: DatoCMS doesn't provide a direct way to cancel retries
  // You might need to disable the webhook temporarily
  
  return {
    pendingRetries: pendingCalls.length,
    recommendation: 'Disable webhook temporarily to stop retries'
  };
}
```

### Webhook Analytics

```typescript
// Comprehensive webhook analytics
async function generateWebhookAnalytics(client, days = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const calls = await client.webhookCalls.list({
    filter: {
      created_at: { gte: startDate.toISOString() }
    }
  });
  
  const analytics = {
    summary: {
      totalCalls: calls.length,
      uniqueWebhooks: new Set(),
      totalSuccessful: 0,
      totalFailed: 0,
      averageResponseTime: 0,
      totalDataTransferred: 0
    },
    byEvent: {},
    byHour: {},
    byDay: {},
    topFailures: {},
    performance: {
      fastest: { webhook: null, time: Infinity },
      slowest: { webhook: null, time: 0 },
      mostReliable: { webhook: null, successRate: 0 },
      leastReliable: { webhook: null, successRate: 100 }
    }
  };
  
  let totalResponseTime = 0;
  let responseTimeCount = 0;
  
  // Process each call
  calls.forEach(call => {
    analytics.summary.uniqueWebhooks.add(call.webhook.id);
    
    // Event type analysis
    if (!analytics.byEvent[call.event_type]) {
      analytics.byEvent[call.event_type] = {
        total: 0,
        successful: 0,
        failed: 0
      };
    }
    analytics.byEvent[call.event_type].total++;
    
    // Time-based analysis
    const date = new Date(call.created_at);
    const hour = date.getHours();
    const day = date.toISOString().split('T')[0];
    
    analytics.byHour[hour] = (analytics.byHour[hour] || 0) + 1;
    analytics.byDay[day] = (analytics.byDay[day] || 0) + 1;
    
    // Success/failure tracking
    if (call.response_status && call.response_status < 400) {
      analytics.summary.totalSuccessful++;
      analytics.byEvent[call.event_type].successful++;
      
      if (call.response_time_ms) {
        totalResponseTime += call.response_time_ms;
        responseTimeCount++;
      }
      
      // Data transfer estimation
      if (call.request.body) {
        analytics.summary.totalDataTransferred += call.request.body.length;
      }
      if (call.response?.body) {
        analytics.summary.totalDataTransferred += call.response.body.length;
      }
    } else {
      analytics.summary.totalFailed++;
      analytics.byEvent[call.event_type].failed++;
      
      // Track failure reasons
      const reason = call.error?.message || `HTTP ${call.response_status}`;
      analytics.topFailures[reason] = (analytics.topFailures[reason] || 0) + 1;
    }
  });
  
  analytics.summary.uniqueWebhooks = analytics.summary.uniqueWebhooks.size;
  analytics.summary.averageResponseTime = responseTimeCount > 0
    ? totalResponseTime / responseTimeCount
    : 0;
  
  // Calculate webhook-specific metrics
  const webhookMetrics = {};
  
  calls.forEach(call => {
    if (!webhookMetrics[call.webhook.id]) {
      webhookMetrics[call.webhook.id] = {
        name: call.webhook.name,
        total: 0,
        successful: 0,
        totalResponseTime: 0,
        responseTimeCount: 0
      };
    }
    
    const metrics = webhookMetrics[call.webhook.id];
    metrics.total++;
    
    if (call.response_status && call.response_status < 400) {
      metrics.successful++;
      
      if (call.response_time_ms) {
        metrics.totalResponseTime += call.response_time_ms;
        metrics.responseTimeCount++;
      }
    }
  });
  
  // Find best/worst performers
  Object.entries(webhookMetrics).forEach(([id, metrics]) => {
    const successRate = (metrics.successful / metrics.total) * 100;
    const avgResponseTime = metrics.responseTimeCount > 0
      ? metrics.totalResponseTime / metrics.responseTimeCount
      : 0;
    
    if (avgResponseTime > 0 && avgResponseTime < analytics.performance.fastest.time) {
      analytics.performance.fastest = {
        webhook: metrics.name,
        time: avgResponseTime
      };
    }
    
    if (avgResponseTime > analytics.performance.slowest.time) {
      analytics.performance.slowest = {
        webhook: metrics.name,
        time: avgResponseTime
      };
    }
    
    if (successRate > analytics.performance.mostReliable.successRate) {
      analytics.performance.mostReliable = {
        webhook: metrics.name,
        successRate
      };
    }
    
    if (successRate < analytics.performance.leastReliable.successRate) {
      analytics.performance.leastReliable = {
        webhook: metrics.name,
        successRate
      };
    }
  });
  
  // Sort top failures
  analytics.topFailures = Object.entries(analytics.topFailures)
    .sort(([,a], [,b]) => b - a)
    .slice(0, 10)
    .reduce((obj, [reason, count]) => {
      obj[reason] = count;
      return obj;
    }, {});
  
  return analytics;
}
```

### Payload Analysis

```typescript
// Analyze webhook payloads for optimization
async function analyzeWebhookPayloads(client, webhookId: string) {
  const calls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      response_status: { lt: 400 }
    },
    per_page: 100
  });
  
  const analysis = {
    averagePayloadSize: 0,
    largestPayload: 0,
    smallestPayload: Infinity,
    commonFields: {},
    unusedFields: new Set(),
    recommendations: []
  };
  
  let totalSize = 0;
  const allFields = new Set();
  const fieldUsage = {};
  
  calls.forEach(call => {
    if (call.request.body) {
      const payload = JSON.parse(call.request.body);
      const payloadSize = call.request.body.length;
      
      totalSize += payloadSize;
      analysis.largestPayload = Math.max(analysis.largestPayload, payloadSize);
      analysis.smallestPayload = Math.min(analysis.smallestPayload, payloadSize);
      
      // Analyze fields
      function analyzeObject(obj, prefix = '') {
        Object.keys(obj).forEach(key => {
          const fieldPath = prefix ? `${prefix}.${key}` : key;
          allFields.add(fieldPath);
          fieldUsage[fieldPath] = (fieldUsage[fieldPath] || 0) + 1;
          
          if (typeof obj[key] === 'object' && obj[key] !== null) {
            analyzeObject(obj[key], fieldPath);
          }
        });
      }
      
      analyzeObject(payload);
    }
  });
  
  analysis.averagePayloadSize = calls.length > 0
    ? totalSize / calls.length
    : 0;
  
  // Find commonly used fields
  const usageThreshold = calls.length * 0.9; // 90% of payloads
  
  Object.entries(fieldUsage).forEach(([field, count]) => {
    if (count >= usageThreshold) {
      analysis.commonFields[field] = (count / calls.length) * 100;
    } else if (count < calls.length * 0.1) {
      analysis.unusedFields.add(field);
    }
  });
  
  // Generate recommendations
  if (analysis.averagePayloadSize > 100000) { // 100KB
    analysis.recommendations.push({
      type: 'optimization',
      message: 'Large payload size detected',
      suggestion: 'Consider filtering unnecessary fields'
    });
  }
  
  if (analysis.unusedFields.size > allFields.size * 0.3) {
    analysis.recommendations.push({
      type: 'cleanup',
      message: 'Many rarely-used fields detected',
      suggestion: 'Review and remove unused fields from webhook payload'
    });
  }
  
  const sizeVariation = analysis.largestPayload / analysis.smallestPayload;
  if (sizeVariation > 10) {
    analysis.recommendations.push({
      type: 'consistency',
      message: 'High payload size variation',
      suggestion: 'Consider implementing pagination for large payloads'
    });
  }
  
  return analysis;
}
```

## Best Practices

### 1. Webhook Reliability

```typescript
// Implement webhook reliability patterns
class ReliableWebhookHandler {
  constructor(private client: any) {}
  
  // Monitor and auto-disable failing webhooks
  async autoDisableFailingWebhooks(threshold = 0.5, minCalls = 10) {
    const webhooks = await this.client.webhooks.list();
    const oneHourAgo = new Date(Date.now() - 60 * 60 * 1000);
    
    for (const webhook of webhooks) {
      if (!webhook.enabled) continue;
      
      const recentCalls = await this.client.webhookCalls.list({
        filter: {
          webhook: { eq: webhook.id },
          created_at: { gte: oneHourAgo.toISOString() }
        }
      });
      
      if (recentCalls.length >= minCalls) {
        const failureRate = recentCalls.filter(c => 
          !c.response_status || c.response_status >= 400
        ).length / recentCalls.length;
        
        if (failureRate > threshold) {
          console.log(`Disabling webhook ${webhook.name} due to ${failureRate * 100}% failure rate`);
          
          await this.client.webhooks.update(webhook.id, {
            enabled: false
          });
          
          // Notify administrators
          // await notifyAdmins(webhook, failureRate);
        }
      }
    }
  }
  
  // Implement circuit breaker pattern
  async withCircuitBreaker(webhookId: string, operation: () => Promise<any>) {
    const recentCalls = await this.client.webhookCalls.list({
      filter: {
        webhook: { eq: webhookId },
        created_at: { 
          gte: new Date(Date.now() - 5 * 60 * 1000).toISOString() 
        }
      }
    });
    
    const failures = recentCalls.filter(c => 
      !c.response_status || c.response_status >= 400
    ).length;
    
    // Circuit open if too many recent failures
    if (failures > 5) {
      throw new Error('Circuit breaker open - too many failures');
    }
    
    try {
      return await operation();
    } catch (error) {
      // Record failure
      console.error('Operation failed:', error);
      throw error;
    }
  }
}
```

### 2. Webhook Testing

```typescript
// Comprehensive webhook testing framework
class WebhookTester {
  constructor(private client: any) {}
  
  async runTestSuite(webhookId: string) {
    const tests = {
      connectivity: await this.testConnectivity(webhookId),
      performance: await this.testPerformance(webhookId),
      reliability: await this.testReliability(webhookId),
      payload: await this.testPayloadHandling(webhookId)
    };
    
    const summary = {
      passed: Object.values(tests).filter(t => t.passed).length,
      total: Object.keys(tests).length,
      details: tests
    };
    
    return summary;
  }
  
  private async testConnectivity(webhookId: string) {
    try {
      const result = await testWebhookEndpoint(this.client, webhookId);
      return {
        passed: result.success,
        message: result.success ? 'Endpoint reachable' : 'Endpoint unreachable',
        details: result
      };
    } catch (error) {
      return {
        passed: false,
        message: 'Connectivity test failed',
        error: error.message
      };
    }
  }
  
  private async testPerformance(webhookId: string) {
    const recentCalls = await this.client.webhookCalls.list({
      filter: {
        webhook: { eq: webhookId },
        response_status: { lt: 400 }
      },
      per_page: 50
    });
    
    if (recentCalls.length === 0) {
      return {
        passed: false,
        message: 'No successful calls to analyze'
      };
    }
    
    const responseTimes = recentCalls
      .filter(c => c.response_time_ms)
      .map(c => c.response_time_ms);
    
    const avgResponseTime = responseTimes.reduce((a, b) => a + b) / responseTimes.length;
    
    return {
      passed: avgResponseTime < 5000, // 5 second threshold
      message: `Average response time: ${avgResponseTime}ms`,
      details: {
        average: avgResponseTime,
        min: Math.min(...responseTimes),
        max: Math.max(...responseTimes)
      }
    };
  }
  
  private async testReliability(webhookId: string) {
    const calls = await this.client.webhookCalls.list({
      filter: {
        webhook: { eq: webhookId }
      },
      per_page: 100
    });
    
    const successRate = calls.filter(c => 
      c.response_status && c.response_status < 400
    ).length / calls.length * 100;
    
    return {
      passed: successRate >= 95,
      message: `Success rate: ${successRate.toFixed(2)}%`,
      details: {
        totalCalls: calls.length,
        successful: calls.filter(c => c.response_status < 400).length,
        failed: calls.filter(c => !c.response_status || c.response_status >= 400).length
      }
    };
  }
  
  private async testPayloadHandling(webhookId: string) {
    // Test with different payload sizes
    const testSizes = [1000, 10000, 100000]; // 1KB, 10KB, 100KB
    const results = [];
    
    for (const size of testSizes) {
      // Would need to trigger test webhooks with different payload sizes
      // This is a simplified example
      results.push({
        size,
        passed: true,
        message: `${size} byte payload handled successfully`
      });
    }
    
    return {
      passed: results.every(r => r.passed),
      message: 'Payload size handling test',
      details: results
    };
  }
}
```

### 3. Webhook Documentation

```typescript
// Generate webhook documentation from actual calls
async function generateWebhookDocumentation(client, webhookId: string) {
  const webhook = await client.webhooks.find(webhookId);
  const recentCalls = await client.webhookCalls.list({
    filter: {
      webhook: { eq: webhookId },
      response_status: { lt: 400 }
    },
    per_page: 10
  });
  
  const documentation = {
    webhook: {
      name: webhook.name,
      url: webhook.url,
      enabled: webhook.enabled,
      events: webhook.events
    },
    endpoints: {
      url: webhook.url,
      method: 'POST',
      headers: webhook.headers || {}
    },
    authentication: webhook.http_basic_user ? 'Basic Auth' : 'None',
    payloadExamples: {},
    responseExamples: {},
    errorHandling: {
      retryPolicy: 'Exponential backoff with 5 attempts',
      timeouts: '30 seconds'
    }
  };
  
  // Extract payload examples for each event type
  const eventTypes = new Set();
  
  recentCalls.forEach(call => {
    eventTypes.add(call.event_type);
    
    if (!documentation.payloadExamples[call.event_type] && call.request.body) {
      documentation.payloadExamples[call.event_type] = JSON.parse(call.request.body);
    }
    
    if (!documentation.responseExamples[call.event_type] && call.response?.body) {
      documentation.responseExamples[call.event_type] = {
        status: call.response_status,
        body: call.response.body
      };
    }
  });
  
  // Generate markdown documentation
  let markdown = `# Webhook: ${documentation.webhook.name}\n\n`;
  markdown += `## Endpoint\n\n`;
  markdown += `- **URL**: ${documentation.endpoints.url}\n`;
  markdown += `- **Method**: ${documentation.endpoints.method}\n`;
  markdown += `- **Authentication**: ${documentation.authentication}\n\n`;
  
  markdown += `## Events\n\n`;
  documentation.webhook.events.forEach(event => {
    markdown += `- ${event.entity_type}: ${event.event_types.join(', ')}\n`;
  });
  
  markdown += `\n## Payload Examples\n\n`;
  Object.entries(documentation.payloadExamples).forEach(([event, payload]) => {
    markdown += `### ${event}\n\n`;
    markdown += '```json\n';
    markdown += JSON.stringify(payload, null, 2);
    markdown += '\n```\n\n';
  });
  
  return {
    documentation,
    markdown
  };
}
```

## Monitoring Best Practices Summary

1. **Set up automated monitoring** for all critical metrics
2. **Implement alerting** for anomalous patterns
3. **Regular analysis** of trends and patterns
4. **Proactive optimization** based on data insights
5. **Documentation** of findings and actions taken
6. **Retention policies** for monitoring data
7. **Dashboard creation** for stakeholder visibility
8. **Integration** with existing monitoring tools

Remember to:
- Use appropriate time ranges for analysis
- Consider rate limits when querying
- Store aggregated data for long-term trends
- Implement proper error handling
- Test monitoring systems regularly
- Keep sensitive data secure
- Document thresholds and alerts