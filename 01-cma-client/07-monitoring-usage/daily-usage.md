# DailyUsage

The DailyUsage resource provides access to daily usage statistics for your DatoCMS project, including API calls, traffic, and media usage metrics. This data is essential for monitoring resource consumption, managing costs, and capacity planning.

## Available Operations

### List Daily Usage

```javascript
const usages = await client.dailyUsages.list();
```

**Returns:** An array of DailyUsage objects containing usage statistics for each day

## The DailyUsage Object

```typescript
{
  id: string;                                    // Unique identifier
  type: 'daily_usage';                          // Always 'daily_usage'
  date: string;                                 // Date in ISO format (YYYY-MM-DD)
  cda_api_calls: number;                        // Content Delivery API calls count
  cma_api_calls: number;                        // Content Management API calls count
  cda_traffic_bytes: number;                    // Content Delivery API traffic in bytes
  cma_traffic_bytes: number;                    // Content Management API traffic in bytes
  assets_traffic_bytes: number;                 // Upload/asset traffic in bytes
  mux_delivered_seconds: number;                // Video streaming (≤1080p) in seconds
  mux_high_resolution_delivered_seconds: number; // High-res video streaming (>1080p)
  mux_encoded_seconds: number;                  // Video encoding time in seconds
}
```

## Metrics Explained

### API Calls
- **cda_api_calls**: Number of requests to the Content Delivery API (published content)
- **cma_api_calls**: Number of requests to the Content Management API (content editing)

### Traffic (in bytes)
- **cda_traffic_bytes**: Data transferred via Content Delivery API
- **cma_traffic_bytes**: Data transferred via Content Management API  
- **assets_traffic_bytes**: Data transferred for file uploads and downloads

### Video Metrics (Mux)
- **mux_delivered_seconds**: Standard quality video streaming (up to 1080p)
- **mux_high_resolution_delivered_seconds**: High quality video streaming (above 1080p)
- **mux_encoded_seconds**: Time spent encoding/processing videos

## Common Use Cases

### 1. Monitor API Usage Trends

```javascript
const usages = await client.dailyUsages.list();

// Calculate total API calls for the period
const totalCalls = usages.reduce((sum, day) => 
  sum + day.cda_api_calls + day.cma_api_calls, 0
);

// Find peak usage day
const peakDay = usages.reduce((max, day) => 
  (day.cda_api_calls + day.cma_api_calls) > (max.cda_api_calls + max.cma_api_calls) ? day : max
);

console.log(`Total API calls: ${totalCalls}`);
console.log(`Peak usage on ${peakDay.date}: ${peakDay.cda_api_calls + peakDay.cma_api_calls} calls`);
```

### 2. Calculate Bandwidth Usage

```javascript
const usages = await client.dailyUsages.list();

// Calculate total bandwidth in GB
const totalBandwidthBytes = usages.reduce((sum, day) => 
  sum + day.cda_traffic_bytes + day.cma_traffic_bytes + day.assets_traffic_bytes, 0
);

const totalBandwidthGB = (totalBandwidthBytes / (1024 * 1024 * 1024)).toFixed(2);

console.log(`Total bandwidth used: ${totalBandwidthGB} GB`);

// Breakdown by service
usages.forEach(day => {
  const cdaGB = (day.cda_traffic_bytes / (1024 * 1024 * 1024)).toFixed(2);
  const cmaGB = (day.cma_traffic_bytes / (1024 * 1024 * 1024)).toFixed(2);
  const assetsGB = (day.assets_traffic_bytes / (1024 * 1024 * 1024)).toFixed(2);
  
  console.log(`${day.date}: CDA: ${cdaGB}GB, CMA: ${cmaGB}GB, Assets: ${assetsGB}GB`);
});
```

### 3. Video Usage Analysis

```javascript
const usages = await client.dailyUsages.list();

// Calculate total video streaming hours
const totalStreamingSeconds = usages.reduce((sum, day) => 
  sum + day.mux_delivered_seconds + day.mux_high_resolution_delivered_seconds, 0
);

const totalStreamingHours = (totalStreamingSeconds / 3600).toFixed(2);

// Calculate encoding costs
const totalEncodingMinutes = usages.reduce((sum, day) => 
  sum + day.mux_encoded_seconds, 0
) / 60;

console.log(`Total video streamed: ${totalStreamingHours} hours`);
console.log(`Total video encoded: ${totalEncodingMinutes.toFixed(2)} minutes`);
```

### 4. Cost Monitoring Dashboard

```javascript
// Create a usage report for the current month
const usages = await client.dailyUsages.list();
const currentMonth = new Date().toISOString().substring(0, 7);

const monthlyUsage = usages
  .filter(day => day.date.startsWith(currentMonth))
  .reduce((report, day) => {
    report.apiCalls += day.cda_api_calls + day.cma_api_calls;
    report.bandwidth += day.cda_traffic_bytes + day.cma_traffic_bytes + day.assets_traffic_bytes;
    report.videoStreaming += day.mux_delivered_seconds + day.mux_high_resolution_delivered_seconds;
    report.videoEncoding += day.mux_encoded_seconds;
    return report;
  }, { apiCalls: 0, bandwidth: 0, videoStreaming: 0, videoEncoding: 0 });

console.log('Monthly Usage Report:', {
  apiCalls: monthlyUsage.apiCalls.toLocaleString(),
  bandwidthGB: (monthlyUsage.bandwidth / (1024**3)).toFixed(2),
  videoStreamingHours: (monthlyUsage.videoStreaming / 3600).toFixed(2),
  videoEncodingMinutes: (monthlyUsage.videoEncoding / 60).toFixed(2)
});
```

### 5. Usage Alerts

```javascript
// Set up usage threshold alerts
const DAILY_API_THRESHOLD = 100000;
const DAILY_BANDWIDTH_GB_THRESHOLD = 50;

const usages = await client.dailyUsages.list();
const today = usages[usages.length - 1]; // Most recent day

const todayApiCalls = today.cda_api_calls + today.cma_api_calls;
const todayBandwidthGB = (today.cda_traffic_bytes + today.cma_traffic_bytes + today.assets_traffic_bytes) / (1024**3);

if (todayApiCalls > DAILY_API_THRESHOLD) {
  console.warn(`⚠️ High API usage: ${todayApiCalls} calls on ${today.date}`);
}

if (todayBandwidthGB > DAILY_BANDWIDTH_GB_THRESHOLD) {
  console.warn(`⚠️ High bandwidth usage: ${todayBandwidthGB.toFixed(2)}GB on ${today.date}`);
}
```

## Dashboard Client Usage

The Dashboard client provides additional filtering options:

```javascript
import { DashboardClient } from '@datocms/dashboard-client';

const client = new DashboardClient({ apiToken: 'YOUR_TOKEN' });

// Get usage for specific month
const marchUsage = await client.dailyUsages.list({
  year: '2024',
  month: '03'
});

// Get usage for entire year
const yearlyUsage = await client.dailyUsages.list({
  year: '2024'
});
```

## Important Notes

1. **Data Availability**: Usage data may have a slight delay (typically a few hours)
2. **Historical Data**: The amount of historical data available depends on your plan
3. **Time Zone**: All dates are in UTC
4. **Read-Only**: This is a read-only resource - usage data cannot be modified
5. **No Pagination**: The list method returns all available records at once

## Raw Response Access

For accessing the raw API response with metadata:

```javascript
const response = await client.dailyUsages.rawList();
console.log(response.data); // Array of daily usage records
```

## Related Resources

- [Subscription Limits](/01-cma-client/07-monitoring-usage/subscription-limit.md) - View plan limits
- [Subscription Features](/01-cma-client/07-monitoring-usage/subscription-feature.md) - Check available features
- [Usage Counters](/01-cma-client/07-monitoring-usage/usage-counter.md) - Real-time usage counters

---

# Daily Usage Methods

This document provides comprehensive documentation for the Daily Usage resource methods in the DatoCMS Content Management API, including examples for time series analysis, usage trend visualization, and cost estimation workflows.

## Available Methods

### list()

Lists daily usage statistics for your DatoCMS project with detailed metrics.

```typescript
// Method signature
client.dailyUsages.list(queryParams?: {
  filter?: {
    date?: {
      gte?: string;  // ISO 8601 date (YYYY-MM-DD)
      lte?: string;  // ISO 8601 date (YYYY-MM-DD)
    };
  };
  page?: {
    offset?: number;
    limit?: number;
  };
}): Promise<DailyUsage[]>;
```

## Response Structure

```typescript
interface DailyUsage {
  type: 'daily_usage';
  id: string;  // Format: "YYYY-MM-DD"
  attributes: {
    date: string;  // ISO 8601 date
    
    // API Usage Metrics
    cma_api_calls: number;           // Content Management API calls
    cda_api_calls: number;           // Content Delivery API calls (REST)
    
    // CDN Traffic
    assets_traffic: number;          // Asset bandwidth in bytes
    
    // Build Minutes
    build_minutes: number;           // Total build minutes used
    
    // Media Processing
    assets_transformations: number;  // Number of asset transformations
    assets_mux_encodings_seconds: number;  // Video encoding seconds
    assets_mux_streaming_seconds: number;  // Video streaming seconds
    
    // Uploads
    total_assets_size: number;       // Total asset storage in bytes
    assets_uploaded: number;         // Number of assets uploaded
    
    // GraphQL Usage
    graphql_requests_preview: number;    // Preview endpoint requests
    graphql_requests_published: number;  // Published endpoint requests
    graphql_complexity_preview: number;  // Preview API complexity points
    graphql_complexity_published: number; // Published API complexity points
  };
}
```

## Basic Usage Examples

### List Recent Usage (Last 30 Days)

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function getRecentUsage() {
  try {
    // Calculate date range for last 30 days
    const endDate = new Date();
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - 30);
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      },
      page: {
        limit: 100  // Max 100 records per page
      }
    });
    
    console.log(`Retrieved ${usageData.length} days of usage data`);
    
    // Display summary
    usageData.forEach(day => {
      console.log(`${day.attributes.date}:`, {
        apiCalls: day.attributes.cma_api_calls + day.attributes.cda_api_calls,
        buildMinutes: day.attributes.build_minutes,
        assetsSize: (day.attributes.total_assets_size / 1024 / 1024).toFixed(2) + ' MB'
      });
    });
    
    return usageData;
  } catch (error) {
    console.error('Error fetching usage data:', error);
    throw error;
  }
}
```

### Current Month Usage

```typescript
async function getCurrentMonthUsage() {
  try {
    const now = new Date();
    const firstDayOfMonth = new Date(now.getFullYear(), now.getMonth(), 1);
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: firstDayOfMonth.toISOString().split('T')[0],
          lte: now.toISOString().split('T')[0]
        }
      }
    });
    
    // Calculate totals
    const totals = usageData.reduce((acc, day) => ({
      totalApiCalls: acc.totalApiCalls + day.attributes.cma_api_calls + day.attributes.cda_api_calls,
      totalBuildMinutes: acc.totalBuildMinutes + day.attributes.build_minutes,
      totalAssetTraffic: acc.totalAssetTraffic + day.attributes.assets_traffic,
      totalTransformations: acc.totalTransformations + day.attributes.assets_transformations,
      totalGraphQLRequests: acc.totalGraphQLRequests + 
        day.attributes.graphql_requests_preview + 
        day.attributes.graphql_requests_published
    }), {
      totalApiCalls: 0,
      totalBuildMinutes: 0,
      totalAssetTraffic: 0,
      totalTransformations: 0,
      totalGraphQLRequests: 0
    });
    
    console.log('Current Month Totals:', {
      apiCalls: totals.totalApiCalls.toLocaleString(),
      buildMinutes: totals.totalBuildMinutes,
      assetTraffic: (totals.totalAssetTraffic / 1024 / 1024 / 1024).toFixed(2) + ' GB',
      transformations: totals.totalTransformations.toLocaleString(),
      graphQLRequests: totals.totalGraphQLRequests.toLocaleString()
    });
    
    return { daily: usageData, totals };
  } catch (error) {
    console.error('Error fetching monthly usage:', error);
    throw error;
  }
}
```

## Time Series Analysis Patterns

### Calculate Weekly Rollups

```typescript
interface WeeklyUsage {
  weekStart: string;
  weekEnd: string;
  metrics: {
    totalApiCalls: number;
    totalBuildMinutes: number;
    totalAssetTraffic: number;
    totalTransformations: number;
    avgDailyApiCalls: number;
    peakDayApiCalls: number;
    peakDate: string;
  };
}

async function calculateWeeklyRollups(weeks = 4): Promise<WeeklyUsage[]> {
  try {
    const endDate = new Date();
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - (weeks * 7));
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      },
      page: { limit: 100 }
    });
    
    // Group by week
    const weeklyData: Map<string, DailyUsage[]> = new Map();
    
    usageData.forEach(day => {
      const date = new Date(day.attributes.date);
      const weekStart = new Date(date);
      weekStart.setDate(date.getDate() - date.getDay()); // Start of week (Sunday)
      const weekKey = weekStart.toISOString().split('T')[0];
      
      if (!weeklyData.has(weekKey)) {
        weeklyData.set(weekKey, []);
      }
      weeklyData.get(weekKey)!.push(day);
    });
    
    // Calculate weekly metrics
    const weeklyRollups: WeeklyUsage[] = [];
    
    weeklyData.forEach((days, weekStart) => {
      const weekEnd = new Date(weekStart);
      weekEnd.setDate(weekEnd.getDate() + 6);
      
      const metrics = days.reduce((acc, day) => {
        const dailyApiCalls = day.attributes.cma_api_calls + day.attributes.cda_api_calls;
        
        return {
          totalApiCalls: acc.totalApiCalls + dailyApiCalls,
          totalBuildMinutes: acc.totalBuildMinutes + day.attributes.build_minutes,
          totalAssetTraffic: acc.totalAssetTraffic + day.attributes.assets_traffic,
          totalTransformations: acc.totalTransformations + day.attributes.assets_transformations,
          peakDayApiCalls: Math.max(acc.peakDayApiCalls, dailyApiCalls),
          peakDate: dailyApiCalls > acc.peakDayApiCalls ? day.attributes.date : acc.peakDate
        };
      }, {
        totalApiCalls: 0,
        totalBuildMinutes: 0,
        totalAssetTraffic: 0,
        totalTransformations: 0,
        peakDayApiCalls: 0,
        peakDate: ''
      });
      
      weeklyRollups.push({
        weekStart,
        weekEnd: weekEnd.toISOString().split('T')[0],
        metrics: {
          ...metrics,
          avgDailyApiCalls: Math.round(metrics.totalApiCalls / days.length)
        }
      });
    });
    
    // Sort by week
    weeklyRollups.sort((a, b) => a.weekStart.localeCompare(b.weekStart));
    
    // Display results
    weeklyRollups.forEach(week => {
      console.log(`Week ${week.weekStart} to ${week.weekEnd}:`, {
        totalApiCalls: week.metrics.totalApiCalls.toLocaleString(),
        avgDailyApiCalls: week.metrics.avgDailyApiCalls.toLocaleString(),
        peakDay: `${week.metrics.peakDate} (${week.metrics.peakDayApiCalls.toLocaleString()} calls)`,
        buildMinutes: week.metrics.totalBuildMinutes,
        assetTraffic: (week.metrics.totalAssetTraffic / 1024 / 1024 / 1024).toFixed(2) + ' GB'
      });
    });
    
    return weeklyRollups;
  } catch (error) {
    console.error('Error calculating weekly rollups:', error);
    throw error;
  }
}
```

### Detect Usage Spikes

```typescript
interface UsageSpike {
  date: string;
  metric: string;
  value: number;
  percentageIncrease: number;
  movingAverage: number;
}

async function detectUsageSpikes(days = 30, threshold = 50): Promise<UsageSpike[]> {
  try {
    const endDate = new Date();
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - days);
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      },
      page: { limit: 100 }
    });
    
    const spikes: UsageSpike[] = [];
    const windowSize = 7; // 7-day moving average
    
    // Metrics to monitor
    const metrics = [
      { name: 'apiCalls', getter: (d: DailyUsage) => d.attributes.cma_api_calls + d.attributes.cda_api_calls },
      { name: 'buildMinutes', getter: (d: DailyUsage) => d.attributes.build_minutes },
      { name: 'assetTraffic', getter: (d: DailyUsage) => d.attributes.assets_traffic },
      { name: 'transformations', getter: (d: DailyUsage) => d.attributes.assets_transformations },
      { name: 'graphQLRequests', getter: (d: DailyUsage) => 
        d.attributes.graphql_requests_preview + d.attributes.graphql_requests_published }
    ];
    
    metrics.forEach(metric => {
      for (let i = windowSize; i < usageData.length; i++) {
        const currentValue = metric.getter(usageData[i]);
        
        // Calculate moving average for previous days
        let sum = 0;
        for (let j = i - windowSize; j < i; j++) {
          sum += metric.getter(usageData[j]);
        }
        const movingAverage = sum / windowSize;
        
        // Check if current value exceeds threshold
        if (movingAverage > 0) {
          const percentageIncrease = ((currentValue - movingAverage) / movingAverage) * 100;
          
          if (percentageIncrease > threshold) {
            spikes.push({
              date: usageData[i].attributes.date,
              metric: metric.name,
              value: currentValue,
              percentageIncrease: Math.round(percentageIncrease),
              movingAverage: Math.round(movingAverage)
            });
          }
        }
      }
    });
    
    // Sort spikes by percentage increase
    spikes.sort((a, b) => b.percentageIncrease - a.percentageIncrease);
    
    console.log(`Found ${spikes.length} usage spikes (>${threshold}% increase):`);
    spikes.forEach(spike => {
      console.log(`${spike.date}: ${spike.metric} spiked to ${spike.value.toLocaleString()} ` +
        `(+${spike.percentageIncrease}% vs avg ${spike.movingAverage.toLocaleString()})`);
    });
    
    return spikes;
  } catch (error) {
    console.error('Error detecting usage spikes:', error);
    throw error;
  }
}
```

## Usage Trend Visualization

### Generate CSV for Charting

```typescript
async function exportUsageToCSV(days = 30): Promise<string> {
  try {
    const endDate = new Date();
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - days);
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      },
      page: { limit: 100 }
    });
    
    // CSV header
    const csv = [
      'Date,API Calls,Build Minutes,Asset Traffic (MB),Transformations,GraphQL Requests,Video Encoding (min),Video Streaming (min)'
    ];
    
    // Add data rows
    usageData.forEach(day => {
      const row = [
        day.attributes.date,
        day.attributes.cma_api_calls + day.attributes.cda_api_calls,
        day.attributes.build_minutes,
        (day.attributes.assets_traffic / 1024 / 1024).toFixed(2),
        day.attributes.assets_transformations,
        day.attributes.graphql_requests_preview + day.attributes.graphql_requests_published,
        (day.attributes.assets_mux_encodings_seconds / 60).toFixed(2),
        (day.attributes.assets_mux_streaming_seconds / 60).toFixed(2)
      ].join(',');
      
      csv.push(row);
    });
    
    const csvContent = csv.join('\n');
    
    // Save to file
    const fs = require('fs').promises;
    const filename = `datocms-usage-${startDate.toISOString().split('T')[0]}-to-${endDate.toISOString().split('T')[0]}.csv`;
    await fs.writeFile(filename, csvContent);
    
    console.log(`Usage data exported to ${filename}`);
    return csvContent;
  } catch (error) {
    console.error('Error exporting usage data:', error);
    throw error;
  }
}
```

### Calculate Growth Rates

```typescript
interface GrowthMetrics {
  metric: string;
  currentPeriodTotal: number;
  previousPeriodTotal: number;
  growthRate: number;
  trend: 'increasing' | 'decreasing' | 'stable';
}

async function calculateGrowthRates(periodDays = 30): Promise<GrowthMetrics[]> {
  try {
    const endDate = new Date();
    const midDate = new Date();
    midDate.setDate(midDate.getDate() - periodDays);
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - (periodDays * 2));
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      },
      page: { limit: 100 }
    });
    
    // Split into current and previous periods
    const midDateStr = midDate.toISOString().split('T')[0];
    const currentPeriod = usageData.filter(d => d.attributes.date > midDateStr);
    const previousPeriod = usageData.filter(d => d.attributes.date <= midDateStr);
    
    // Define metrics to analyze
    const metricsConfig = [
      { name: 'API Calls', getter: (d: DailyUsage) => d.attributes.cma_api_calls + d.attributes.cda_api_calls },
      { name: 'Build Minutes', getter: (d: DailyUsage) => d.attributes.build_minutes },
      { name: 'Asset Traffic', getter: (d: DailyUsage) => d.attributes.assets_traffic },
      { name: 'Transformations', getter: (d: DailyUsage) => d.attributes.assets_transformations },
      { name: 'GraphQL Requests', getter: (d: DailyUsage) => 
        d.attributes.graphql_requests_preview + d.attributes.graphql_requests_published }
    ];
    
    const growthMetrics: GrowthMetrics[] = metricsConfig.map(config => {
      const currentTotal = currentPeriod.reduce((sum, d) => sum + config.getter(d), 0);
      const previousTotal = previousPeriod.reduce((sum, d) => sum + config.getter(d), 0);
      
      const growthRate = previousTotal > 0 
        ? ((currentTotal - previousTotal) / previousTotal) * 100 
        : 0;
      
      let trend: 'increasing' | 'decreasing' | 'stable';
      if (growthRate > 5) trend = 'increasing';
      else if (growthRate < -5) trend = 'decreasing';
      else trend = 'stable';
      
      return {
        metric: config.name,
        currentPeriodTotal: currentTotal,
        previousPeriodTotal: previousTotal,
        growthRate: Math.round(growthRate * 10) / 10,
        trend
      };
    });
    
    // Display results
    console.log(`Growth rates for ${periodDays}-day periods:`);
    growthMetrics.forEach(metric => {
      const arrow = metric.trend === 'increasing' ? '↑' : 
                   metric.trend === 'decreasing' ? '↓' : '→';
      console.log(`${metric.metric}: ${arrow} ${metric.growthRate}%`, {
        current: metric.currentPeriodTotal.toLocaleString(),
        previous: metric.previousPeriodTotal.toLocaleString()
      });
    });
    
    return growthMetrics;
  } catch (error) {
    console.error('Error calculating growth rates:', error);
    throw error;
  }
}
```

## Cost Estimation Workflows

### Estimate Monthly Costs

```typescript
interface CostEstimate {
  metric: string;
  usage: number;
  estimatedCost: number;
  notes: string;
}

async function estimateMonthlyCosts(): Promise<{
  breakdown: CostEstimate[];
  totalEstimated: number;
}> {
  try {
    // Get current month usage
    const now = new Date();
    const firstDayOfMonth = new Date(now.getFullYear(), now.getMonth(), 1);
    const daysInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
    const daysElapsed = now.getDate();
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: firstDayOfMonth.toISOString().split('T')[0],
          lte: now.toISOString().split('T')[0]
        }
      }
    });
    
    // Calculate current totals
    const currentTotals = usageData.reduce((acc, day) => ({
      apiCalls: acc.apiCalls + day.attributes.cma_api_calls + day.attributes.cda_api_calls,
      buildMinutes: acc.buildMinutes + day.attributes.build_minutes,
      assetTraffic: acc.assetTraffic + day.attributes.assets_traffic,
      transformations: acc.transformations + day.attributes.assets_transformations,
      videoEncoding: acc.videoEncoding + day.attributes.assets_mux_encodings_seconds,
      videoStreaming: acc.videoStreaming + day.attributes.assets_mux_streaming_seconds,
      graphqlRequests: acc.graphqlRequests + 
        day.attributes.graphql_requests_preview + 
        day.attributes.graphql_requests_published
    }), {
      apiCalls: 0,
      buildMinutes: 0,
      assetTraffic: 0,
      transformations: 0,
      videoEncoding: 0,
      videoStreaming: 0,
      graphqlRequests: 0
    });
    
    // Project to full month
    const projectionFactor = daysInMonth / daysElapsed;
    const projectedTotals = {
      apiCalls: Math.round(currentTotals.apiCalls * projectionFactor),
      buildMinutes: Math.round(currentTotals.buildMinutes * projectionFactor),
      assetTraffic: Math.round(currentTotals.assetTraffic * projectionFactor),
      transformations: Math.round(currentTotals.transformations * projectionFactor),
      videoEncoding: Math.round(currentTotals.videoEncoding * projectionFactor),
      videoStreaming: Math.round(currentTotals.videoStreaming * projectionFactor),
      graphqlRequests: Math.round(currentTotals.graphqlRequests * projectionFactor)
    };
    
    // Example pricing (adjust based on your plan)
    const pricing = {
      apiCallsPer1M: 5.00,        // $5 per 1M API calls over included
      buildMinutesPer1K: 10.00,   // $10 per 1K build minutes over included
      assetTrafficPerGB: 0.15,    // $0.15 per GB over included
      transformationsPer1K: 2.00,  // $2 per 1K transformations over included
      videoEncodingPerHour: 4.00,  // $4 per hour of encoding
      videoStreamingPerHour: 0.50  // $0.50 per hour of streaming
    };
    
    // Example included limits (adjust based on your plan)
    const included = {
      apiCalls: 1000000,       // 1M included
      buildMinutes: 300,       // 300 minutes included
      assetTrafficGB: 100,     // 100 GB included
      transformations: 50000   // 50K included
    };
    
    // Calculate costs
    const breakdown: CostEstimate[] = [];
    
    // API Calls
    const apiCallsOverage = Math.max(0, projectedTotals.apiCalls - included.apiCalls);
    if (apiCallsOverage > 0) {
      breakdown.push({
        metric: 'API Calls',
        usage: apiCallsOverage,
        estimatedCost: (apiCallsOverage / 1000000) * pricing.apiCallsPer1M,
        notes: `${(apiCallsOverage / 1000000).toFixed(2)}M calls over included ${(included.apiCalls / 1000000)}M`
      });
    }
    
    // Build Minutes
    const buildMinutesOverage = Math.max(0, projectedTotals.buildMinutes - included.buildMinutes);
    if (buildMinutesOverage > 0) {
      breakdown.push({
        metric: 'Build Minutes',
        usage: buildMinutesOverage,
        estimatedCost: (buildMinutesOverage / 1000) * pricing.buildMinutesPer1K,
        notes: `${buildMinutesOverage} minutes over included ${included.buildMinutes}`
      });
    }
    
    // Asset Traffic
    const assetTrafficGB = projectedTotals.assetTraffic / 1024 / 1024 / 1024;
    const assetTrafficOverage = Math.max(0, assetTrafficGB - included.assetTrafficGB);
    if (assetTrafficOverage > 0) {
      breakdown.push({
        metric: 'Asset Traffic',
        usage: assetTrafficOverage,
        estimatedCost: assetTrafficOverage * pricing.assetTrafficPerGB,
        notes: `${assetTrafficOverage.toFixed(2)} GB over included ${included.assetTrafficGB} GB`
      });
    }
    
    // Transformations
    const transformationsOverage = Math.max(0, projectedTotals.transformations - included.transformations);
    if (transformationsOverage > 0) {
      breakdown.push({
        metric: 'Transformations',
        usage: transformationsOverage,
        estimatedCost: (transformationsOverage / 1000) * pricing.transformationsPer1K,
        notes: `${(transformationsOverage / 1000).toFixed(1)}K over included ${(included.transformations / 1000)}K`
      });
    }
    
    // Video Encoding
    const videoEncodingHours = projectedTotals.videoEncoding / 3600;
    if (videoEncodingHours > 0) {
      breakdown.push({
        metric: 'Video Encoding',
        usage: videoEncodingHours,
        estimatedCost: videoEncodingHours * pricing.videoEncodingPerHour,
        notes: `${videoEncodingHours.toFixed(2)} hours total`
      });
    }
    
    // Video Streaming
    const videoStreamingHours = projectedTotals.videoStreaming / 3600;
    if (videoStreamingHours > 0) {
      breakdown.push({
        metric: 'Video Streaming',
        usage: videoStreamingHours,
        estimatedCost: videoStreamingHours * pricing.videoStreamingPerHour,
        notes: `${videoStreamingHours.toFixed(2)} hours total`
      });
    }
    
    const totalEstimated = breakdown.reduce((sum, item) => sum + item.estimatedCost, 0);
    
    // Display results
    console.log(`\nEstimated costs for ${now.toLocaleString('default', { month: 'long', year: 'numeric' })}:`);
    console.log(`(Based on ${daysElapsed} days of usage, projected to ${daysInMonth} days)\n`);
    
    breakdown.forEach(item => {
      console.log(`${item.metric}: $${item.estimatedCost.toFixed(2)}`);
      console.log(`  ${item.notes}\n`);
    });
    
    console.log(`Total Estimated Overage: $${totalEstimated.toFixed(2)}`);
    
    return { breakdown, totalEstimated };
  } catch (error) {
    console.error('Error estimating costs:', error);
    throw error;
  }
}
```

## Multi-Metric Comparison

### Compare Multiple Metrics

```typescript
interface MetricComparison {
  date: string;
  metrics: {
    [key: string]: {
      value: number;
      percentOfPeak: number;
    };
  };
}

async function compareMetrics(days = 7): Promise<{
  comparisons: MetricComparison[];
  peaks: { [key: string]: { value: number; date: string } };
}> {
  try {
    const endDate = new Date();
    const startDate = new Date();
    startDate.setDate(startDate.getDate() - days);
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      }
    });
    
    // Define metrics to compare
    const metricsConfig = {
      apiCalls: (d: DailyUsage) => d.attributes.cma_api_calls + d.attributes.cda_api_calls,
      buildMinutes: (d: DailyUsage) => d.attributes.build_minutes,
      assetTrafficMB: (d: DailyUsage) => d.attributes.assets_traffic / 1024 / 1024,
      transformations: (d: DailyUsage) => d.attributes.assets_transformations,
      graphQLRequests: (d: DailyUsage) => 
        d.attributes.graphql_requests_preview + d.attributes.graphql_requests_published
    };
    
    // Find peak values
    const peaks: { [key: string]: { value: number; date: string } } = {};
    
    Object.entries(metricsConfig).forEach(([metricName, getter]) => {
      let peakValue = 0;
      let peakDate = '';
      
      usageData.forEach(day => {
        const value = getter(day);
        if (value > peakValue) {
          peakValue = value;
          peakDate = day.attributes.date;
        }
      });
      
      peaks[metricName] = { value: peakValue, date: peakDate };
    });
    
    // Build comparisons
    const comparisons: MetricComparison[] = usageData.map(day => {
      const metrics: { [key: string]: { value: number; percentOfPeak: number } } = {};
      
      Object.entries(metricsConfig).forEach(([metricName, getter]) => {
        const value = getter(day);
        const percentOfPeak = peaks[metricName].value > 0 
          ? (value / peaks[metricName].value) * 100 
          : 0;
        
        metrics[metricName] = {
          value: Math.round(value),
          percentOfPeak: Math.round(percentOfPeak)
        };
      });
      
      return {
        date: day.attributes.date,
        metrics
      };
    });
    
    // Display comparison table
    console.log('\nMetric Comparison (% of peak in parentheses):\n');
    console.log('Date'.padEnd(12) + 
      Object.keys(metricsConfig).map(m => m.padEnd(20)).join(''));
    console.log('-'.repeat(12 + Object.keys(metricsConfig).length * 20));
    
    comparisons.forEach(comp => {
      const row = comp.date.padEnd(12) + 
        Object.entries(comp.metrics).map(([_, data]) => 
          `${data.value.toLocaleString()} (${data.percentOfPeak}%)`.padEnd(20)
        ).join('');
      console.log(row);
    });
    
    // Display peaks
    console.log('\nPeak Values:');
    Object.entries(peaks).forEach(([metric, peak]) => {
      console.log(`${metric}: ${peak.value.toLocaleString()} on ${peak.date}`);
    });
    
    return { comparisons, peaks };
  } catch (error) {
    console.error('Error comparing metrics:', error);
    throw error;
  }
}
```

## Error Handling Examples

### Comprehensive Error Handling

```typescript
async function robustUsageFetch(retries = 3): Promise<DailyUsage[]> {
  let lastError: Error | null = null;
  
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      console.log(`Fetching usage data (attempt ${attempt}/${retries})...`);
      
      const endDate = new Date();
      const startDate = new Date();
      startDate.setDate(startDate.getDate() - 30);
      
      const usageData = await client.dailyUsages.list({
        filter: {
          date: {
            gte: startDate.toISOString().split('T')[0],
            lte: endDate.toISOString().split('T')[0]
          }
        },
        page: { limit: 100 }
      });
      
      // Validate data
      if (!Array.isArray(usageData)) {
        throw new Error('Invalid response format: expected array');
      }
      
      if (usageData.length === 0) {
        console.warn('No usage data found for the specified period');
      }
      
      // Validate data integrity
      usageData.forEach((day, index) => {
        if (!day.attributes || !day.attributes.date) {
          throw new Error(`Invalid data at index ${index}: missing attributes or date`);
        }
        
        // Check for required numeric fields
        const numericFields = [
          'cma_api_calls', 'cda_api_calls', 'build_minutes',
          'assets_traffic', 'assets_transformations'
        ];
        
        numericFields.forEach(field => {
          if (typeof day.attributes[field] !== 'number') {
            throw new Error(`Invalid data for ${day.attributes.date}: ${field} is not a number`);
          }
        });
      });
      
      console.log(`Successfully fetched ${usageData.length} days of usage data`);
      return usageData;
      
    } catch (error) {
      lastError = error as Error;
      console.error(`Attempt ${attempt} failed:`, error);
      
      if (attempt < retries) {
        // Exponential backoff
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Waiting ${delay}ms before retry...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  // All retries failed
  throw new Error(`Failed to fetch usage data after ${retries} attempts: ${lastError?.message}`);
}

// Usage with error handling
async function safeUsageAnalysis() {
  try {
    const usageData = await robustUsageFetch();
    
    // Process data...
    console.log('Analysis completed successfully');
    
  } catch (error) {
    console.error('Usage analysis failed:', error);
    
    // Log error details for debugging
    if (error instanceof Error) {
      console.error('Error details:', {
        message: error.message,
        stack: error.stack,
        timestamp: new Date().toISOString()
      });
    }
    
    // Notify administrators or take corrective action
    // await notifyAdmins('Usage analysis failed', error);
  }
}
```

## Best Practices for Usage Analytics

### 1. Regular Monitoring

```typescript
// Set up automated daily usage checks
async function dailyUsageCheck() {
  try {
    const yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1);
    const dateStr = yesterday.toISOString().split('T')[0];
    
    const usage = await client.dailyUsages.list({
      filter: {
        date: {
          gte: dateStr,
          lte: dateStr
        }
      }
    });
    
    if (usage.length > 0) {
      const day = usage[0];
      const totalApiCalls = day.attributes.cma_api_calls + day.attributes.cda_api_calls;
      
      // Check against thresholds
      const thresholds = {
        apiCalls: 100000,      // Alert if > 100K calls/day
        buildMinutes: 60,      // Alert if > 60 minutes/day
        assetTrafficGB: 10     // Alert if > 10 GB/day
      };
      
      const alerts = [];
      
      if (totalApiCalls > thresholds.apiCalls) {
        alerts.push(`High API usage: ${totalApiCalls.toLocaleString()} calls`);
      }
      
      if (day.attributes.build_minutes > thresholds.buildMinutes) {
        alerts.push(`High build usage: ${day.attributes.build_minutes} minutes`);
      }
      
      const trafficGB = day.attributes.assets_traffic / 1024 / 1024 / 1024;
      if (trafficGB > thresholds.assetTrafficGB) {
        alerts.push(`High asset traffic: ${trafficGB.toFixed(2)} GB`);
      }
      
      if (alerts.length > 0) {
        console.warn(`Usage alerts for ${dateStr}:`, alerts);
        // Send notifications
      } else {
        console.log(`Usage for ${dateStr} is within normal limits`);
      }
    }
  } catch (error) {
    console.error('Daily usage check failed:', error);
  }
}
```

### 2. Efficient Data Fetching

```typescript
// Fetch only required date ranges
async function getUsageForDateRange(startDate: Date, endDate: Date): Promise<DailyUsage[]> {
  const allData: DailyUsage[] = [];
  let offset = 0;
  const limit = 100; // Maximum allowed
  
  while (true) {
    const batch = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      },
      page: { offset, limit }
    });
    
    allData.push(...batch);
    
    if (batch.length < limit) {
      break; // No more data
    }
    
    offset += limit;
  }
  
  return allData;
}
```

### 3. Caching Strategy

```typescript
// Cache usage data to reduce API calls
class UsageCache {
  private cache: Map<string, DailyUsage> = new Map();
  private lastFetch: Date | null = null;
  
  async getUsage(startDate: Date, endDate: Date): Promise<DailyUsage[]> {
    const now = new Date();
    const cacheExpiry = 3600000; // 1 hour
    
    // Check if cache needs refresh
    if (!this.lastFetch || now.getTime() - this.lastFetch.getTime() > cacheExpiry) {
      await this.refreshCache(startDate, endDate);
    }
    
    // Return cached data for date range
    const result: DailyUsage[] = [];
    const current = new Date(startDate);
    
    while (current <= endDate) {
      const dateKey = current.toISOString().split('T')[0];
      const cached = this.cache.get(dateKey);
      if (cached) {
        result.push(cached);
      }
      current.setDate(current.getDate() + 1);
    }
    
    return result;
  }
  
  private async refreshCache(startDate: Date, endDate: Date): Promise<void> {
    const client = buildClient({ apiToken: process.env.DATOCMS_API_TOKEN! });
    
    const usageData = await client.dailyUsages.list({
      filter: {
        date: {
          gte: startDate.toISOString().split('T')[0],
          lte: endDate.toISOString().split('T')[0]
        }
      },
      page: { limit: 100 }
    });
    
    // Update cache
    this.cache.clear();
    usageData.forEach(day => {
      this.cache.set(day.attributes.date, day);
    });
    
    this.lastFetch = new Date();
  }
}
```

## Related Resources

- [Subscription Features](./subscription-feature.md) - Available features and limits
- [Subscription Limits](./subscription-limit.md) - Current usage limits
- [Usage Counters](./usage-counter.md) - Real-time usage tracking
- [Job Results](./job-result.md) - Background job monitoring
- [Build Events](../06-monitoring/build-event.md) - Build pipeline monitoring
- [Webhook Calls](../06-monitoring/webhook-call.md) - Webhook delivery tracking

## Common Integration Patterns

### Slack Notifications for Usage Alerts

```typescript
async function sendUsageAlertToSlack(webhookUrl: string) {
  const spike = await detectUsageSpikes(7, 100); // 100% increase threshold
  
  if (spike.length > 0) {
    const message = {
      text: 'DatoCMS Usage Alert',
      blocks: [
        {
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `*Usage Spike Detected* :warning:\n${spike.length} metrics exceeded normal levels`
          }
        },
        ...spike.map(s => ({
          type: 'section',
          text: {
            type: 'mrkdwn',
            text: `• *${s.metric}* on ${s.date}: ${s.value.toLocaleString()} (+${s.percentageIncrease}%)`
          }
        }))
      ]
    };
    
    await fetch(webhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(message)
    });
  }
}
```

### Dashboard Data Export

```typescript
async function generateDashboardData(): Promise<{
  summary: any;
  daily: any[];
  weekly: any[];
}> {
  const [daily, weekly, growth] = await Promise.all([
    getCurrentMonthUsage(),
    calculateWeeklyRollups(4),
    calculateGrowthRates(30)
  ]);
  
  return {
    summary: {
      currentMonth: daily.totals,
      growth: growth,
      lastUpdated: new Date().toISOString()
    },
    daily: daily.daily,
    weekly: weekly
  };
}
```