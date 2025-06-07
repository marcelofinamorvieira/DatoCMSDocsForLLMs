# Build Event

Build Events track the status and progress of builds triggered by DatoCMS. They provide real-time information about build deployments, including success/failure status, duration, and logs.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// List recent build events
const events = await client.buildEvents.list();

// Get a specific build event
const event = await client.buildEvents.find('event-id');
```

## API Reference

### List Build Events

Retrieve build events with filtering options.

```typescript
// Get all build events
const allEvents = await client.buildEvents.list();

// Filter by build trigger
const triggerEvents = await client.buildEvents.list({
  filter: {
    build_trigger: { eq: 'trigger-id' }
  }
});

// Filter by status
const failedBuilds = await client.buildEvents.list({
  filter: {
    status: { eq: 'failed' }
  },
  order_by: ['created_at_DESC']
});

// Get recent builds
const recentBuilds = await client.buildEvents.list({
  page: { offset: 0, limit: 20 },
  order_by: ['created_at_DESC']
});
```

**Parameters:**
- `filter` (object, optional): Filter conditions
  - `build_trigger`: Filter by trigger ID
  - `status`: Filter by build status
  - `created_at`: Date range filters
- `order_by` (array, optional): Sort order
- `page` (object, optional): Pagination

**Returns:** Array of build event objects

### Find Build Event

Get details of a specific build event.

```typescript
const event = await client.buildEvents.find('event-id');

console.log(`Build #${event.id}`);
console.log(`Status: ${event.status}`);
console.log(`Duration: ${event.duration}ms`);
console.log(`Trigger: ${event.build_trigger.id}`);
```

**Parameters:**
- `eventId` (string, required): The build event ID

**Returns:** Build event object

## Build Status Tracking

### Monitor Build Progress

```typescript
async function monitorBuildProgress(eventId: string) {
  let event = await client.buildEvents.find(eventId);
  
  while (event.status === 'pending' || event.status === 'building') {
    console.log(`Build ${eventId}: ${event.status}`);
    
    // Wait 5 seconds before checking again
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    event = await client.buildEvents.find(eventId);
  }
  
  console.log(`Build completed: ${event.status}`);
  console.log(`Duration: ${event.duration}ms`);
  
  return event;
}
```

### Build Analytics

```typescript
async function getBuildAnalytics(triggerId: string, days = 7) {
  const since = new Date();
  since.setDate(since.getDate() - days);
  
  const events = await client.buildEvents.list({
    filter: {
      build_trigger: { eq: triggerId },
      created_at: { gte: since.toISOString() }
    }
  });
  
  const stats = {
    total: events.length,
    successful: events.filter(e => e.status === 'success').length,
    failed: events.filter(e => e.status === 'failed').length,
    avgDuration: 0,
    maxDuration: 0,
    minDuration: Infinity
  };
  
  const durations = events
    .filter(e => e.duration && e.status === 'success')
    .map(e => e.duration);
  
  if (durations.length > 0) {
    stats.avgDuration = durations.reduce((a, b) => a + b, 0) / durations.length;
    stats.maxDuration = Math.max(...durations);
    stats.minDuration = Math.min(...durations);
  }
  
  return stats;
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.buildEvents.find('invalid-id');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 404) {
    console.log('Build event not found');
  }
}
```

## Build Event Object Structure

```typescript
interface BuildEvent {
  id: string;
  type: 'build_event';
  status: 'pending' | 'building' | 'success' | 'failed';
  created_at: string;
  started_at?: string;
  completed_at?: string;
  duration?: number; // milliseconds
  error_message?: string;
  build_trigger: {
    type: 'build_trigger';
    id: string;
  };
}
```

## Related Resources

- [Build Triggers](../04-site-configuration/build-trigger.md) - Configure build webhooks
- [Webhooks](./webhook.md) - Alternative event notifications
- [Webhook Calls](./webhook-call.md) - Track webhook deliveries

---

# Build Event Methods

This guide covers the methods available for Build Event resources in the DatoCMS Content Management API. Build events track the status and details of site builds triggered through DatoCMS.

## Available Methods

### list()

List all build events for the current environment.

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

// Basic usage - list all build events
const buildEvents = await client.buildEvents.list();

// Returns: SimpleSchemaBuildEvent[]
/*
{
  id: string;
  type: 'build_event';
  attributes: {
    status: 'triggered' | 'success' | 'failed';
    triggered_at: string; // ISO 8601 datetime
    completed_at: string | null; // ISO 8601 datetime or null if still running
    duration: number | null; // Duration in seconds or null if still running
    error_message: string | null; // Error details if status is 'failed'
    deploy_url: string | null; // Deployment URL if available
    environment: string; // Environment name
    build_trigger: {
      id: string;
      type: 'build_trigger';
    } | null;
    creator: {
      id: string;
      type: 'account' | 'user' | 'sso_user' | 'access_token';
    } | null;
  }
}
*/
```

## Build Tracking Patterns

### Monitor Active Builds

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function getActiveBulds() {
  try {
    const allBuilds = await client.buildEvents.list();
    
    // Filter for currently running builds
    const activeBuilds = allBuilds.filter(build => 
      build.attributes.status === 'triggered' && 
      !build.attributes.completed_at
    );
    
    console.log(`Active builds: ${activeBuilds.length}`);
    
    activeBuilds.forEach(build => {
      const startTime = new Date(build.attributes.triggered_at);
      const elapsed = Math.floor((Date.now() - startTime.getTime()) / 1000);
      console.log(`Build ${build.id}: Running for ${elapsed} seconds`);
    });
    
    return activeBuilds;
  } catch (error) {
    console.error('Error fetching active builds:', error);
    throw error;
  }
}
```

### Track Build Success Rate

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function calculateBuildSuccessRate(days: number = 7) {
  try {
    const builds = await client.buildEvents.list();
    
    // Filter builds from last N days
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);
    
    const recentBuilds = builds.filter(build => 
      new Date(build.attributes.triggered_at) > cutoffDate
    );
    
    const stats = recentBuilds.reduce((acc, build) => {
      acc.total++;
      acc[build.attributes.status]++;
      return acc;
    }, { total: 0, success: 0, failed: 0, triggered: 0 });
    
    const successRate = stats.total > 0 
      ? (stats.success / stats.total * 100).toFixed(2)
      : 0;
    
    console.log(`Build statistics (last ${days} days):`);
    console.log(`- Total builds: ${stats.total}`);
    console.log(`- Successful: ${stats.success}`);
    console.log(`- Failed: ${stats.failed}`);
    console.log(`- In progress: ${stats.triggered}`);
    console.log(`- Success rate: ${successRate}%`);
    
    return stats;
  } catch (error) {
    console.error('Error calculating build stats:', error);
    throw error;
  }
}
```

## Filtering by Status and Trigger

### Filter Builds by Status

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function getBuildsByStatus(status: 'triggered' | 'success' | 'failed') {
  try {
    const allBuilds = await client.buildEvents.list();
    
    const filteredBuilds = allBuilds.filter(build => 
      build.attributes.status === status
    );
    
    console.log(`Found ${filteredBuilds.length} builds with status: ${status}`);
    
    return filteredBuilds;
  } catch (error) {
    console.error(`Error fetching ${status} builds:`, error);
    throw error;
  }
}

// Get all failed builds
const failedBuilds = await getBuildsByStatus('failed');
failedBuilds.forEach(build => {
  console.log(`Failed build ${build.id}:`);
  console.log(`- Triggered at: ${build.attributes.triggered_at}`);
  console.log(`- Error: ${build.attributes.error_message}`);
});
```

### Group Builds by Trigger Source

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function groupBuildsByTrigger() {
  try {
    const builds = await client.buildEvents.list();
    
    const buildsByTrigger = builds.reduce((acc, build) => {
      const triggerId = build.attributes.build_trigger?.id || 'manual';
      if (!acc[triggerId]) {
        acc[triggerId] = [];
      }
      acc[triggerId].push(build);
      return acc;
    }, {} as Record<string, typeof builds>);
    
    // Get build trigger details for better reporting
    const triggers = await client.buildTriggers.list();
    const triggerMap = new Map(triggers.map(t => [t.id, t]));
    
    console.log('Builds grouped by trigger:');
    for (const [triggerId, triggerBuilds] of Object.entries(buildsByTrigger)) {
      if (triggerId === 'manual') {
        console.log(`\nManual builds: ${triggerBuilds.length}`);
      } else {
        const trigger = triggerMap.get(triggerId);
        console.log(`\n${trigger?.attributes.name || 'Unknown trigger'} (${triggerId}): ${triggerBuilds.length} builds`);
      }
    }
    
    return buildsByTrigger;
  } catch (error) {
    console.error('Error grouping builds:', error);
    throw error;
  }
}
```

## Build Duration Analysis

### Analyze Build Performance

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface BuildMetrics {
  averageDuration: number;
  medianDuration: number;
  minDuration: number;
  maxDuration: number;
  totalBuilds: number;
}

async function analyzeBuildPerformance(days: number = 30): Promise<BuildMetrics> {
  try {
    const builds = await client.buildEvents.list();
    
    // Filter completed builds from last N days
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - days);
    
    const completedBuilds = builds.filter(build => 
      build.attributes.status !== 'triggered' &&
      build.attributes.duration !== null &&
      new Date(build.attributes.triggered_at) > cutoffDate
    );
    
    if (completedBuilds.length === 0) {
      return {
        averageDuration: 0,
        medianDuration: 0,
        minDuration: 0,
        maxDuration: 0,
        totalBuilds: 0
      };
    }
    
    const durations = completedBuilds
      .map(b => b.attributes.duration!)
      .sort((a, b) => a - b);
    
    const metrics: BuildMetrics = {
      averageDuration: durations.reduce((sum, d) => sum + d, 0) / durations.length,
      medianDuration: durations[Math.floor(durations.length / 2)],
      minDuration: durations[0],
      maxDuration: durations[durations.length - 1],
      totalBuilds: completedBuilds.length
    };
    
    console.log(`Build performance metrics (last ${days} days):`);
    console.log(`- Total completed builds: ${metrics.totalBuilds}`);
    console.log(`- Average duration: ${formatDuration(metrics.averageDuration)}`);
    console.log(`- Median duration: ${formatDuration(metrics.medianDuration)}`);
    console.log(`- Fastest build: ${formatDuration(metrics.minDuration)}`);
    console.log(`- Slowest build: ${formatDuration(metrics.maxDuration)}`);
    
    return metrics;
  } catch (error) {
    console.error('Error analyzing build performance:', error);
    throw error;
  }
}

function formatDuration(seconds: number): string {
  const minutes = Math.floor(seconds / 60);
  const remainingSeconds = seconds % 60;
  return `${minutes}m ${remainingSeconds}s`;
}
```

### Identify Slow Builds

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function identifySlowBuilds(thresholdMinutes: number = 10) {
  try {
    const builds = await client.buildEvents.list();
    const thresholdSeconds = thresholdMinutes * 60;
    
    const slowBuilds = builds.filter(build => 
      build.attributes.duration !== null &&
      build.attributes.duration > thresholdSeconds
    );
    
    console.log(`Found ${slowBuilds.length} builds exceeding ${thresholdMinutes} minutes:`);
    
    slowBuilds
      .sort((a, b) => b.attributes.duration! - a.attributes.duration!)
      .slice(0, 10) // Top 10 slowest
      .forEach(build => {
        console.log(`\nBuild ${build.id}:`);
        console.log(`- Duration: ${formatDuration(build.attributes.duration!)}`);
        console.log(`- Triggered: ${build.attributes.triggered_at}`);
        console.log(`- Status: ${build.attributes.status}`);
        if (build.attributes.build_trigger) {
          console.log(`- Trigger ID: ${build.attributes.build_trigger.id}`);
        }
      });
    
    return slowBuilds;
  } catch (error) {
    console.error('Error identifying slow builds:', error);
    throw error;
  }
}
```

## Deployment Monitoring Workflows

### Monitor Recent Deployments

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface DeploymentInfo {
  build: any; // SimpleSchemaBuildEvent
  trigger?: any; // Build trigger details
  creator?: any; // User/token that triggered the build
}

async function monitorRecentDeployments(hours: number = 24): Promise<DeploymentInfo[]> {
  try {
    const builds = await client.buildEvents.list();
    const cutoffTime = new Date();
    cutoffTime.setHours(cutoffTime.getHours() - hours);
    
    // Filter recent successful builds with deploy URLs
    const recentDeployments = builds.filter(build => 
      new Date(build.attributes.triggered_at) > cutoffTime &&
      build.attributes.status === 'success' &&
      build.attributes.deploy_url
    );
    
    // Enrich with trigger information
    const triggers = await client.buildTriggers.list();
    const triggerMap = new Map(triggers.map(t => [t.id, t]));
    
    const deploymentInfo: DeploymentInfo[] = recentDeployments.map(build => ({
      build,
      trigger: build.attributes.build_trigger 
        ? triggerMap.get(build.attributes.build_trigger.id)
        : undefined
    }));
    
    console.log(`Recent deployments (last ${hours} hours):`);
    deploymentInfo.forEach(({ build, trigger }) => {
      console.log(`\nDeployment ${build.id}:`);
      console.log(`- Deployed at: ${build.attributes.completed_at}`);
      console.log(`- Duration: ${formatDuration(build.attributes.duration!)}`);
      console.log(`- Deploy URL: ${build.attributes.deploy_url}`);
      console.log(`- Trigger: ${trigger?.attributes.name || 'Manual'}`);
    });
    
    return deploymentInfo;
  } catch (error) {
    console.error('Error monitoring deployments:', error);
    throw error;
  }
}
```

### Create Build Status Dashboard

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface BuildDashboard {
  currentStatus: 'idle' | 'building' | 'failed';
  activeBuilds: any[];
  recentBuilds: any[];
  stats24h: {
    total: number;
    successful: number;
    failed: number;
    averageDuration: number;
  };
  lastDeployment?: {
    build: any;
    url: string;
    timestamp: string;
  };
}

async function getBuildDashboard(): Promise<BuildDashboard> {
  try {
    const builds = await client.buildEvents.list();
    const now = new Date();
    const oneDayAgo = new Date(now.getTime() - 24 * 60 * 60 * 1000);
    
    // Current active builds
    const activeBuilds = builds.filter(b => 
      b.attributes.status === 'triggered'
    );
    
    // Recent builds (last 10)
    const recentBuilds = builds
      .sort((a, b) => 
        new Date(b.attributes.triggered_at).getTime() - 
        new Date(a.attributes.triggered_at).getTime()
      )
      .slice(0, 10);
    
    // 24h stats
    const last24hBuilds = builds.filter(b => 
      new Date(b.attributes.triggered_at) > oneDayAgo
    );
    
    const stats24h = {
      total: last24hBuilds.length,
      successful: last24hBuilds.filter(b => b.attributes.status === 'success').length,
      failed: last24hBuilds.filter(b => b.attributes.status === 'failed').length,
      averageDuration: last24hBuilds
        .filter(b => b.attributes.duration !== null)
        .reduce((sum, b) => sum + b.attributes.duration!, 0) / 
        (last24hBuilds.filter(b => b.attributes.duration !== null).length || 1)
    };
    
    // Last successful deployment
    const lastDeployment = builds
      .filter(b => b.attributes.status === 'success' && b.attributes.deploy_url)
      .sort((a, b) => 
        new Date(b.attributes.completed_at!).getTime() - 
        new Date(a.attributes.completed_at!).getTime()
      )[0];
    
    // Determine current status
    let currentStatus: BuildDashboard['currentStatus'] = 'idle';
    if (activeBuilds.length > 0) {
      currentStatus = 'building';
    } else if (recentBuilds[0]?.attributes.status === 'failed') {
      currentStatus = 'failed';
    }
    
    const dashboard: BuildDashboard = {
      currentStatus,
      activeBuilds,
      recentBuilds,
      stats24h,
      lastDeployment: lastDeployment ? {
        build: lastDeployment,
        url: lastDeployment.attributes.deploy_url!,
        timestamp: lastDeployment.attributes.completed_at!
      } : undefined
    };
    
    // Display dashboard
    console.log('=== Build Status Dashboard ===');
    console.log(`Current Status: ${currentStatus.toUpperCase()}`);
    console.log(`Active Builds: ${activeBuilds.length}`);
    console.log('\n24 Hour Statistics:');
    console.log(`- Total builds: ${stats24h.total}`);
    console.log(`- Successful: ${stats24h.successful}`);
    console.log(`- Failed: ${stats24h.failed}`);
    console.log(`- Average duration: ${formatDuration(stats24h.averageDuration)}`);
    
    if (dashboard.lastDeployment) {
      console.log('\nLast Deployment:');
      console.log(`- Time: ${dashboard.lastDeployment.timestamp}`);
      console.log(`- URL: ${dashboard.lastDeployment.url}`);
    }
    
    return dashboard;
  } catch (error) {
    console.error('Error creating build dashboard:', error);
    throw error;
  }
}
```

## Error Handling Examples

### Handle Build Failures

```typescript
import { buildCmaClient } from '@datocms/cma-client';
import { ApiError } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function analyzeBuildFailures() {
  try {
    const builds = await client.buildEvents.list();
    const failedBuilds = builds.filter(b => b.attributes.status === 'failed');
    
    if (failedBuilds.length === 0) {
      console.log('No failed builds found');
      return [];
    }
    
    // Group failures by error message
    const errorGroups = failedBuilds.reduce((acc, build) => {
      const error = build.attributes.error_message || 'Unknown error';
      if (!acc[error]) {
        acc[error] = [];
      }
      acc[error].push(build);
      return acc;
    }, {} as Record<string, typeof failedBuilds>);
    
    console.log('Build failure analysis:');
    for (const [error, errorBuilds] of Object.entries(errorGroups)) {
      console.log(`\nError: "${error}"`);
      console.log(`Occurrences: ${errorBuilds.length}`);
      console.log('Recent instances:');
      errorBuilds.slice(0, 3).forEach(build => {
        console.log(`  - ${build.id} at ${build.attributes.triggered_at}`);
      });
    }
    
    return errorGroups;
  } catch (error) {
    if (error instanceof ApiError) {
      console.error('API Error:', error.message);
      console.error('Error details:', error.errorWithCode);
    } else {
      console.error('Unexpected error:', error);
    }
    throw error;
  }
}
```

### Retry Failed Builds

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function retryFailedBuilds(hoursBack: number = 1) {
  try {
    const builds = await client.buildEvents.list();
    const cutoffTime = new Date();
    cutoffTime.setHours(cutoffTime.getHours() - hoursBack);
    
    // Find recent failed builds
    const recentFailedBuilds = builds.filter(build => 
      build.attributes.status === 'failed' &&
      new Date(build.attributes.triggered_at) > cutoffTime
    );
    
    if (recentFailedBuilds.length === 0) {
      console.log('No recent failed builds to retry');
      return;
    }
    
    console.log(`Found ${recentFailedBuilds.length} failed builds to retry`);
    
    // Get unique build triggers from failed builds
    const failedTriggerIds = new Set(
      recentFailedBuilds
        .map(b => b.attributes.build_trigger?.id)
        .filter(Boolean)
    );
    
    // Trigger new builds for each failed trigger
    const retryResults = [];
    for (const triggerId of failedTriggerIds) {
      try {
        console.log(`Retrying build trigger: ${triggerId}`);
        // Note: You would need to use the appropriate method to trigger a build
        // This is just an example structure
        // const result = await client.buildTriggers.trigger(triggerId);
        // retryResults.push({ triggerId, success: true });
      } catch (error) {
        console.error(`Failed to retry trigger ${triggerId}:`, error);
        retryResults.push({ triggerId, success: false, error });
      }
    }
    
    return retryResults;
  } catch (error) {
    console.error('Error retrying failed builds:', error);
    throw error;
  }
}
```

## Best Practices for CI/CD Monitoring

### 1. Implement Build Notifications

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface BuildNotification {
  type: 'started' | 'completed' | 'failed';
  build: any;
  message: string;
  severity: 'info' | 'success' | 'error';
}

async function setupBuildMonitoring(
  onNotification: (notification: BuildNotification) => void,
  pollInterval: number = 30000 // 30 seconds
) {
  let knownBuilds = new Map<string, string>(); // id -> status
  
  async function checkBuilds() {
    try {
      const builds = await client.buildEvents.list();
      
      for (const build of builds) {
        const previousStatus = knownBuilds.get(build.id);
        const currentStatus = build.attributes.status;
        
        // New build detected
        if (!previousStatus && currentStatus === 'triggered') {
          onNotification({
            type: 'started',
            build,
            message: `Build ${build.id} started`,
            severity: 'info'
          });
        }
        
        // Build completed
        else if (previousStatus === 'triggered' && currentStatus === 'success') {
          onNotification({
            type: 'completed',
            build,
            message: `Build ${build.id} completed successfully in ${formatDuration(build.attributes.duration!)}`,
            severity: 'success'
          });
        }
        
        // Build failed
        else if (previousStatus === 'triggered' && currentStatus === 'failed') {
          onNotification({
            type: 'failed',
            build,
            message: `Build ${build.id} failed: ${build.attributes.error_message}`,
            severity: 'error'
          });
        }
        
        knownBuilds.set(build.id, currentStatus);
      }
      
      // Clean up old builds from memory
      const currentBuildIds = new Set(builds.map(b => b.id));
      for (const [id] of knownBuilds) {
        if (!currentBuildIds.has(id)) {
          knownBuilds.delete(id);
        }
      }
    } catch (error) {
      console.error('Error checking builds:', error);
    }
  }
  
  // Initial check
  await checkBuilds();
  
  // Set up polling
  const intervalId = setInterval(checkBuilds, pollInterval);
  
  // Return cleanup function
  return () => clearInterval(intervalId);
}

// Usage example
const cleanup = await setupBuildMonitoring((notification) => {
  console.log(`[${notification.severity.toUpperCase()}] ${notification.message}`);
  
  // Send to external notification service
  if (notification.type === 'failed') {
    // sendSlackNotification(notification.message);
    // sendEmailAlert(notification.message);
  }
});
```

### 2. Track Build Trends

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface BuildTrend {
  date: string;
  total: number;
  successful: number;
  failed: number;
  averageDuration: number;
}

async function analyzeBuildTrends(days: number = 7): Promise<BuildTrend[]> {
  try {
    const builds = await client.buildEvents.list();
    const trends: BuildTrend[] = [];
    
    for (let i = 0; i < days; i++) {
      const date = new Date();
      date.setDate(date.getDate() - i);
      date.setHours(0, 0, 0, 0);
      
      const nextDate = new Date(date);
      nextDate.setDate(nextDate.getDate() + 1);
      
      const dayBuilds = builds.filter(build => {
        const buildDate = new Date(build.attributes.triggered_at);
        return buildDate >= date && buildDate < nextDate;
      });
      
      const completedBuilds = dayBuilds.filter(b => b.attributes.duration !== null);
      const averageDuration = completedBuilds.length > 0
        ? completedBuilds.reduce((sum, b) => sum + b.attributes.duration!, 0) / completedBuilds.length
        : 0;
      
      trends.push({
        date: date.toISOString().split('T')[0],
        total: dayBuilds.length,
        successful: dayBuilds.filter(b => b.attributes.status === 'success').length,
        failed: dayBuilds.filter(b => b.attributes.status === 'failed').length,
        averageDuration
      });
    }
    
    // Display trend summary
    console.log(`Build trends (last ${days} days):`);
    trends.reverse().forEach(trend => {
      const successRate = trend.total > 0 
        ? (trend.successful / trend.total * 100).toFixed(1)
        : 'N/A';
      console.log(`${trend.date}: ${trend.total} builds, ${successRate}% success, avg ${formatDuration(trend.averageDuration)}`);
    });
    
    return trends;
  } catch (error) {
    console.error('Error analyzing build trends:', error);
    throw error;
  }
}
```

### 3. Monitor Build Queue Health

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface QueueHealth {
  status: 'healthy' | 'warning' | 'critical';
  activeBuilds: number;
  queuedBuilds: number;
  averageWaitTime: number;
  oldestBuildAge: number;
  recommendations: string[];
}

async function assessBuildQueueHealth(): Promise<QueueHealth> {
  try {
    const builds = await client.buildEvents.list();
    const now = new Date();
    
    // Get active builds
    const activeBuilds = builds.filter(b => b.attributes.status === 'triggered');
    
    // Calculate metrics
    const queueMetrics = activeBuilds.map(build => {
      const startTime = new Date(build.attributes.triggered_at);
      const ageInMinutes = (now.getTime() - startTime.getTime()) / (1000 * 60);
      return { build, ageInMinutes };
    });
    
    const oldestBuildAge = queueMetrics.length > 0
      ? Math.max(...queueMetrics.map(m => m.ageInMinutes))
      : 0;
    
    const averageWaitTime = queueMetrics.length > 0
      ? queueMetrics.reduce((sum, m) => sum + m.ageInMinutes, 0) / queueMetrics.length
      : 0;
    
    // Determine health status
    let status: QueueHealth['status'] = 'healthy';
    const recommendations: string[] = [];
    
    if (activeBuilds.length > 5) {
      status = 'warning';
      recommendations.push('High number of concurrent builds detected');
    }
    
    if (oldestBuildAge > 30) {
      status = 'critical';
      recommendations.push(`Build running for over ${Math.floor(oldestBuildAge)} minutes - may be stuck`);
    }
    
    if (averageWaitTime > 15) {
      status = status === 'healthy' ? 'warning' : status;
      recommendations.push('Long average build times - consider optimizing build process');
    }
    
    const health: QueueHealth = {
      status,
      activeBuilds: activeBuilds.length,
      queuedBuilds: 0, // Would need webhook data to determine actual queue
      averageWaitTime,
      oldestBuildAge,
      recommendations
    };
    
    console.log(`Build Queue Health: ${status.toUpperCase()}`);
    console.log(`- Active builds: ${health.activeBuilds}`);
    console.log(`- Average build age: ${health.averageWaitTime.toFixed(1)} minutes`);
    console.log(`- Oldest build: ${health.oldestBuildAge.toFixed(1)} minutes`);
    
    if (recommendations.length > 0) {
      console.log('\nRecommendations:');
      recommendations.forEach(rec => console.log(`- ${rec}`));
    }
    
    return health;
  } catch (error) {
    console.error('Error assessing queue health:', error);
    throw error;
  }
}
```

## Integration with Build Triggers

### Link Builds to Triggers

```typescript
import { buildCmaClient } from '@datocms/cma-client';

const client = buildCmaClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function analyzeBuildsPerTrigger() {
  try {
    const [builds, triggers] = await Promise.all([
      client.buildEvents.list(),
      client.buildTriggers.list()
    ]);
    
    // Create trigger lookup map
    const triggerMap = new Map(triggers.map(t => [t.id, t]));
    
    // Analyze builds per trigger
    const triggerStats = new Map<string, {
      trigger: any;
      builds: any[];
      successRate: number;
      averageDuration: number;
    }>();
    
    // Initialize stats for all triggers
    triggers.forEach(trigger => {
      triggerStats.set(trigger.id, {
        trigger,
        builds: [],
        successRate: 0,
        averageDuration: 0
      });
    });
    
    // Group builds by trigger
    builds.forEach(build => {
      const triggerId = build.attributes.build_trigger?.id;
      if (triggerId && triggerStats.has(triggerId)) {
        triggerStats.get(triggerId)!.builds.push(build);
      }
    });
    
    // Calculate statistics
    for (const [triggerId, stats] of triggerStats) {
      const triggerBuilds = stats.builds;
      if (triggerBuilds.length > 0) {
        const successfulBuilds = triggerBuilds.filter(b => b.attributes.status === 'success');
        stats.successRate = (successfulBuilds.length / triggerBuilds.length) * 100;
        
        const completedBuilds = triggerBuilds.filter(b => b.attributes.duration !== null);
        stats.averageDuration = completedBuilds.length > 0
          ? completedBuilds.reduce((sum, b) => sum + b.attributes.duration!, 0) / completedBuilds.length
          : 0;
      }
    }
    
    // Display results
    console.log('Build Trigger Analysis:');
    for (const [triggerId, stats] of triggerStats) {
      console.log(`\n${stats.trigger.attributes.name}:`);
      console.log(`- Total builds: ${stats.builds.length}`);
      console.log(`- Success rate: ${stats.successRate.toFixed(1)}%`);
      console.log(`- Average duration: ${formatDuration(stats.averageDuration)}`);
      console.log(`- Webhook URL: ${stats.trigger.attributes.webhook_url || 'N/A'}`);
    }
    
    return triggerStats;
  } catch (error) {
    console.error('Error analyzing builds per trigger:', error);
    throw error;
  }
}
```

## Related Resources

- [Build Trigger Methods](./build-trigger-methods.md) - Create and manage build triggers
- [Webhook Methods](./webhook-methods.md) - Set up webhooks for build events
- [Environment Methods](../04-site-configuration/environment-methods.md) - Manage deployment environments
- [Maintenance Mode Methods](../04-site-configuration/maintenance-mode-methods.md) - Control site availability during deployments

## Build Event Object Structure

```typescript
interface SimpleSchemaBuildEvent {
  id: string;
  type: 'build_event';
  attributes: {
    // Build status
    status: 'triggered' | 'success' | 'failed';
    
    // Timestamps
    triggered_at: string; // ISO 8601 datetime when build was triggered
    completed_at: string | null; // ISO 8601 datetime when build completed (null if still running)
    
    // Build metrics
    duration: number | null; // Duration in seconds (null if still running)
    
    // Build details
    error_message: string | null; // Error details if status is 'failed'
    deploy_url: string | null; // Deployment URL if available
    environment: string; // Environment name (e.g., 'main', 'staging')
    
    // Related entities
    build_trigger: {
      id: string;
      type: 'build_trigger';
    } | null; // The trigger that initiated this build
    
    creator: {
      id: string;
      type: 'account' | 'user' | 'sso_user' | 'access_token';
    } | null; // Who/what triggered the build
  };
}
```

## Common Patterns Summary

1. **Real-time Monitoring**: Poll build events regularly to track active builds and notify on status changes
2. **Performance Analysis**: Calculate build duration metrics to identify optimization opportunities
3. **Failure Analysis**: Group failed builds by error message to identify common issues
4. **Trend Tracking**: Analyze build patterns over time to spot degradation or improvements
5. **Queue Health**: Monitor concurrent builds and build age to prevent bottlenecks
6. **Trigger Analytics**: Link builds to their triggers for detailed deployment source analysis