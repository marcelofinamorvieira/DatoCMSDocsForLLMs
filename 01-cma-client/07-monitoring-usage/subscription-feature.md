# SubscriptionFeature

The SubscriptionFeature resource provides information about which features are available and in use for your DatoCMS project based on its subscription plan. While SubscriptionLimit tracks quantitative usage (how much), SubscriptionFeature tracks qualitative capabilities (what features are enabled).

## Available Operations

### List All Subscription Features

```javascript
const features = await client.subscriptionFeatures.list();
```

**Returns:** An array of all subscription features for your project

## The SubscriptionFeature Object

```typescript
{
  id: string;                    // Feature identifier (same as code)
  type: 'subscription_feature';  // Always 'subscription_feature'
  code: string;                  // The feature codename
  enabled: boolean;              // Whether available on current plan
  in_use?: boolean;             // Whether actively used (optional)
}
```

## Common Feature Types

### Authentication & Security
| Feature Code | Description |
|--------------|-------------|
| `sso` | Single Sign-On integration |
| `two_factor_authentication` | Two-factor authentication |
| `ip_whitelist` | IP address whitelisting |

### Development & Testing
| Feature Code | Description |
|--------------|-------------|
| `sandbox_environments` | Multiple sandbox environments |
| `draft_content_api` | Access draft content via API |
| `environment_fork` | Fork environments |

### Publishing & Workflows
| Feature Code | Description |
|--------------|-------------|
| `scheduled_publishing` | Schedule content publication |
| `workflows` | Publishing workflows |
| `bulk_publish` | Bulk publish operations |

### Analytics & Monitoring
| Feature Code | Description |
|--------------|-------------|
| `traffic_analytics` | Built-in traffic analytics |
| `audit_logs` | Detailed audit logging |
| `activity_log` | User activity tracking |

### API & Integrations
| Feature Code | Description |
|--------------|-------------|
| `graphql_api` | GraphQL API access |
| `webhooks` | Webhook notifications |
| `plugins` | Custom plugin installation |
| `build_triggers` | Build trigger integrations |

### Advanced Features
| Feature Code | Description |
|--------------|-------------|
| `locales` | Multi-locale support |
| `custom_forms` | Custom form builder |
| `white_label` | White-label customization |
| `advanced_media_area` | Advanced media management |

## Common Use Cases

### 1. Feature Availability Check

```javascript
// Check if specific features are available
async function checkFeatureAvailability(client, requiredFeatures) {
  const features = await client.subscriptionFeatures.list();
  const featureMap = new Map(features.map(f => [f.code, f]));
  
  const results = requiredFeatures.map(featureCode => {
    const feature = featureMap.get(featureCode);
    return {
      feature: featureCode,
      available: feature?.enabled || false,
      inUse: feature?.in_use || false
    };
  });
  
  const unavailable = results.filter(r => !r.available);
  
  if (unavailable.length > 0) {
    console.warn('Unavailable features:', unavailable.map(r => r.feature));
  }
  
  return results;
}

// Usage
const required = ['sso', 'sandbox_environments', 'webhooks'];
const availability = await checkFeatureAvailability(client, required);
```

### 2. Feature-Gated Functionality

```javascript
// Implement feature gates in your application
class FeatureGate {
  constructor(client) {
    this.client = client;
    this.features = null;
  }
  
  async initialize() {
    this.features = await this.client.subscriptionFeatures.list();
    this.featureMap = new Map(this.features.map(f => [f.code, f]));
  }
  
  isEnabled(featureCode) {
    const feature = this.featureMap.get(featureCode);
    return feature?.enabled || false;
  }
  
  isInUse(featureCode) {
    const feature = this.featureMap.get(featureCode);
    return feature?.in_use || false;
  }
  
  requireFeature(featureCode) {
    if (!this.isEnabled(featureCode)) {
      throw new Error(
        `Feature '${featureCode}' is not available on your current plan. ` +
        `Please upgrade to access this feature.`
      );
    }
  }
}

// Usage
const featureGate = new FeatureGate(client);
await featureGate.initialize();

// Before using SSO
featureGate.requireFeature('sso');
// Proceed with SSO configuration...
```

### 3. Plan Comparison Dashboard

```javascript
// Create a feature comparison view
async function generateFeatureReport(client) {
  const features = await client.subscriptionFeatures.list();
  
  const categories = {
    'Authentication': ['sso', 'two_factor_authentication', 'ip_whitelist'],
    'Development': ['sandbox_environments', 'draft_content_api', 'environment_fork'],
    'Publishing': ['scheduled_publishing', 'workflows', 'bulk_publish'],
    'Analytics': ['traffic_analytics', 'audit_logs', 'activity_log'],
    'Integrations': ['graphql_api', 'webhooks', 'plugins', 'build_triggers']
  };
  
  const report = {};
  
  Object.entries(categories).forEach(([category, featureCodes]) => {
    report[category] = featureCodes.map(code => {
      const feature = features.find(f => f.code === code);
      return {
        feature: code.replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase()),
        status: !feature ? 'Unknown' :
                !feature.enabled ? '❌ Not Available' :
                feature.in_use === true ? '✅ Active' :
                feature.in_use === false ? '⭕ Available' : '✓ Enabled'
      };
    });
  });
  
  return report;
}

// Display the report
const report = await generateFeatureReport(client);
Object.entries(report).forEach(([category, features]) => {
  console.log(`\n${category}:`);
  features.forEach(f => console.log(`  ${f.feature}: ${f.status}`));
});
```

### 4. Feature Adoption Metrics

```javascript
// Analyze feature utilization across your project
async function analyzeFeatureAdoption(client) {
  const features = await client.subscriptionFeatures.list();
  
  const enabledFeatures = features.filter(f => f.enabled);
  const trackableFeatures = enabledFeatures.filter(f => f.in_use !== undefined);
  const activeFeatures = trackableFeatures.filter(f => f.in_use === true);
  
  const metrics = {
    totalAvailable: enabledFeatures.length,
    trackable: trackableFeatures.length,
    activelyUsed: activeFeatures.length,
    adoptionRate: trackableFeatures.length > 0 
      ? `${Math.round((activeFeatures.length / trackableFeatures.length) * 100)}%`
      : 'N/A',
    
    unusedFeatures: trackableFeatures
      .filter(f => f.in_use === false)
      .map(f => ({
        code: f.code,
        name: f.code.replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase())
      })),
    
    recommendations: []
  };
  
  // Add recommendations for unused features
  if (metrics.unusedFeatures.length > 0) {
    metrics.recommendations.push(
      `Consider exploring these unused features: ${
        metrics.unusedFeatures.map(f => f.name).join(', ')
      }`
    );
  }
  
  return metrics;
}
```

### 5. Upgrade Prompts

```javascript
// Smart upgrade suggestions based on feature requests
class UpgradeAdvisor {
  constructor(client) {
    this.client = client;
    this.attemptedFeatures = new Set();
  }
  
  async checkAndSuggest(featureCode) {
    const features = await this.client.subscriptionFeatures.list();
    const feature = features.find(f => f.code === featureCode);
    
    if (!feature?.enabled) {
      this.attemptedFeatures.add(featureCode);
      
      return {
        available: false,
        message: `The '${featureCode}' feature requires a plan upgrade.`,
        suggestion: await this.getUpgradeSuggestion()
      };
    }
    
    return { available: true };
  }
  
  async getUpgradeSuggestion() {
    if (this.attemptedFeatures.size >= 3) {
      return `You've tried to use ${this.attemptedFeatures.size} features ` +
             `not available on your current plan. Consider upgrading for full access.`;
    }
    return null;
  }
}

// Usage
const advisor = new UpgradeAdvisor(client);

const ssoCheck = await advisor.checkAndSuggest('sso');
if (!ssoCheck.available) {
  console.log(ssoCheck.message);
  if (ssoCheck.suggestion) {
    console.log(ssoCheck.suggestion);
  }
}
```

### 6. Dashboard Client Usage

When using the Dashboard client, you need to specify the site ID:

```javascript
import { DashboardClient } from '@datocms/dashboard-client';

const dashboardClient = new DashboardClient({ apiToken: 'YOUR_TOKEN' });

// Get features for a specific site
const siteId = 'your-site-id';
const features = await dashboardClient.subscriptionFeatures.list(siteId);

// Compare features across multiple sites
async function compareSiteFeatures(dashboardClient, siteIds) {
  const comparison = {};
  
  for (const siteId of siteIds) {
    const features = await dashboardClient.subscriptionFeatures.list(siteId);
    comparison[siteId] = features.reduce((acc, f) => {
      acc[f.code] = f.enabled;
      return acc;
    }, {});
  }
  
  return comparison;
}
```

## Understanding Feature States

### Feature Status Combinations

| `enabled` | `in_use` | Meaning |
|-----------|----------|---------|
| `true` | `true` | Feature is available and actively used |
| `true` | `false` | Feature is available but not currently used |
| `true` | `undefined` | Feature is available (usage not tracked) |
| `false` | Any | Feature not available on current plan |

### Best Practices

1. **Cache Features**: Feature availability rarely changes, so cache the results
2. **Graceful Degradation**: Design your app to work without optional features
3. **Clear Messaging**: Provide clear upgrade prompts when features aren't available
4. **Track Attempts**: Log when users try to use unavailable features

## Raw Response Access

For accessing the complete API response:

```javascript
// CMA Client
const response = await client.subscriptionFeatures.rawList();
console.log(response.data); // Array of features with full JSON:API structure

// Dashboard Client
const response = await dashboardClient.subscriptionFeatures.rawList(siteId);
console.log(response.data); // Features for specific site
```

## Important Notes

1. **Read-Only Resource**: Features cannot be modified via API
2. **Plan Updates**: Feature availability updates when plans change
3. **Usage Tracking**: Not all features track the `in_use` status
4. **Feature Evolution**: New features may be added over time

## Related Resources

- [Subscription Limits](/01-cma-client/07-monitoring-usage/subscription-limit.md) - Quantitative usage limits
- [Site](/01-cma-client/04-site-configuration/site.md) - Current plan information
- [Plans](/02-dashboard-client/plan.md) - Available plans and features

---

# Subscription Feature Methods

The Subscription Feature resource provides methods to retrieve and analyze features available in your DatoCMS subscription plan. This includes both boolean features (enabled/disabled) and numeric limits.

## Available Methods

### list()

Lists all subscription features for the current project, showing which features are enabled and their limits.

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

// List all subscription features
async function listSubscriptionFeatures() {
  try {
    const features = await client.subscriptionFeatures.list();
    
    console.log('Subscription Features:', features);
    
    // Example response structure
    /*
    [
      {
        id: "123",
        type: "subscription_feature",
        attributes: {
          feature_type: "boolean",              // Type of feature: "boolean" or "numeric"
          enabled: true,                        // Whether the feature is enabled
          api_id: "graphql_content_api",       // Unique identifier for the feature
          name: "GraphQL Content API",         // Human-readable feature name
          description: "Access to GraphQL API", // Feature description
          value: null                          // Numeric value (only for numeric features)
        }
      },
      {
        id: "124",
        type: "subscription_feature",
        attributes: {
          feature_type: "numeric",
          enabled: true,
          api_id: "monthly_api_calls",
          name: "Monthly API Calls",
          description: "Number of API calls per month",
          value: 1000000                       // Numeric limit
        }
      },
      {
        id: "125",
        type: "subscription_feature",
        attributes: {
          feature_type: "boolean",
          enabled: false,                      // Feature not available in current plan
          api_id: "advanced_media_area",
          name: "Advanced Media Area",
          description: "Advanced media management features",
          value: null
        }
      }
    ]
    */
    
    return features;
  } catch (error) {
    console.error('Error listing subscription features:', error);
    throw error;
  }
}
```

## Feature Availability Checking

Check if specific features are available in your subscription:

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

// Check if a specific feature is available
async function isFeatureAvailable(featureApiId: string): Promise<boolean> {
  try {
    const features = await client.subscriptionFeatures.list();
    
    const feature = features.find(f => f.attributes.api_id === featureApiId);
    
    if (!feature) {
      console.warn(`Feature ${featureApiId} not found`);
      return false;
    }
    
    return feature.attributes.enabled;
  } catch (error) {
    console.error('Error checking feature availability:', error);
    return false;
  }
}

// Example usage
async function checkFeatures() {
  const hasGraphQL = await isFeatureAvailable('graphql_content_api');
  const hasWebhooks = await isFeatureAvailable('webhooks');
  const hasSSO = await isFeatureAvailable('sso');
  
  console.log('GraphQL API:', hasGraphQL ? 'Available' : 'Not available');
  console.log('Webhooks:', hasWebhooks ? 'Available' : 'Not available');
  console.log('SSO:', hasSSO ? 'Available' : 'Not available');
}
```

## Plan Comparison Patterns

Compare features across different subscription plans:

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

interface FeatureComparison {
  apiId: string;
  name: string;
  currentPlan: {
    enabled: boolean;
    value?: number;
  };
  requiredForFeature?: string;
}

// Create a feature comparison report
async function generateFeatureComparison(): Promise<FeatureComparison[]> {
  try {
    const features = await client.subscriptionFeatures.list();
    
    const comparison: FeatureComparison[] = features.map(feature => ({
      apiId: feature.attributes.api_id,
      name: feature.attributes.name,
      currentPlan: {
        enabled: feature.attributes.enabled,
        value: feature.attributes.value || undefined
      }
    }));
    
    // Identify features that might require upgrades
    const limitedFeatures = comparison.filter(f => !f.currentPlan.enabled);
    
    if (limitedFeatures.length > 0) {
      console.log('Features requiring plan upgrade:');
      limitedFeatures.forEach(f => {
        console.log(`- ${f.name} (${f.apiId})`);
      });
    }
    
    return comparison;
  } catch (error) {
    console.error('Error generating feature comparison:', error);
    throw error;
  }
}
```

## Feature Flags Implementation

Implement feature flags based on subscription features:

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

class FeatureFlags {
  private features: Map<string, SimpleSchemaTypes.SubscriptionFeature> = new Map();
  private loaded: boolean = false;
  
  // Load features from API
  async load(): Promise<void> {
    try {
      const featureList = await client.subscriptionFeatures.list();
      
      this.features.clear();
      featureList.forEach(feature => {
        this.features.set(feature.attributes.api_id, feature);
      });
      
      this.loaded = true;
    } catch (error) {
      console.error('Error loading feature flags:', error);
      throw error;
    }
  }
  
  // Check if a feature is enabled
  isEnabled(featureApiId: string): boolean {
    if (!this.loaded) {
      console.warn('Feature flags not loaded. Call load() first.');
      return false;
    }
    
    const feature = this.features.get(featureApiId);
    return feature?.attributes.enabled || false;
  }
  
  // Get numeric limit for a feature
  getLimit(featureApiId: string): number | null {
    if (!this.loaded) {
      console.warn('Feature flags not loaded. Call load() first.');
      return null;
    }
    
    const feature = this.features.get(featureApiId);
    if (feature?.attributes.feature_type === 'numeric') {
      return feature.attributes.value;
    }
    
    return null;
  }
  
  // Check if usage is within limits
  isWithinLimit(featureApiId: string, currentUsage: number): boolean {
    const limit = this.getLimit(featureApiId);
    if (limit === null) return true; // No limit or not a numeric feature
    
    return currentUsage < limit;
  }
}

// Example usage
async function useFeatureFlags() {
  const flags = new FeatureFlags();
  await flags.load();
  
  // Check boolean features
  if (flags.isEnabled('webhooks')) {
    console.log('Webhooks are available');
    // Enable webhook functionality
  }
  
  // Check numeric limits
  const apiCallLimit = flags.getLimit('monthly_api_calls');
  console.log(`API call limit: ${apiCallLimit || 'Unlimited'}`);
  
  // Check if within limits
  const currentApiCalls = 500000;
  if (!flags.isWithinLimit('monthly_api_calls', currentApiCalls)) {
    console.warn('Approaching API call limit!');
  }
}
```

## Upgrade Prompts Workflows

Create upgrade prompts based on feature limitations:

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

interface UpgradePrompt {
  feature: string;
  message: string;
  currentLimit?: number;
  requiredAction: string;
}

// Generate upgrade prompts for disabled features
async function generateUpgradePrompts(): Promise<UpgradePrompt[]> {
  try {
    const features = await client.subscriptionFeatures.list();
    const prompts: UpgradePrompt[] = [];
    
    features.forEach(feature => {
      if (!feature.attributes.enabled) {
        prompts.push({
          feature: feature.attributes.name,
          message: `${feature.attributes.name} is not available in your current plan`,
          requiredAction: 'Upgrade your plan to access this feature'
        });
      }
    });
    
    // Check for features nearing limits
    const numericFeatures = features.filter(f => 
      f.attributes.feature_type === 'numeric' && 
      f.attributes.enabled
    );
    
    // You would need to get actual usage data here
    // This is a placeholder for demonstration
    for (const feature of numericFeatures) {
      const limit = feature.attributes.value;
      // const currentUsage = await getUsageForFeature(feature.attributes.api_id);
      // if (currentUsage > limit * 0.8) {
      //   prompts.push({
      //     feature: feature.attributes.name,
      //     message: `You're approaching the limit for ${feature.attributes.name}`,
      //     currentLimit: limit,
      //     requiredAction: 'Consider upgrading for higher limits'
      //   });
      // }
    }
    
    return prompts;
  } catch (error) {
    console.error('Error generating upgrade prompts:', error);
    throw error;
  }
}

// Display upgrade prompts to users
async function displayUpgradePrompts() {
  const prompts = await generateUpgradePrompts();
  
  if (prompts.length > 0) {
    console.log('⚠️  Plan Limitations:');
    prompts.forEach(prompt => {
      console.log(`\n📌 ${prompt.feature}`);
      console.log(`   ${prompt.message}`);
      if (prompt.currentLimit) {
        console.log(`   Current limit: ${prompt.currentLimit}`);
      }
      console.log(`   Action: ${prompt.requiredAction}`);
    });
  }
}
```

## Feature Usage Tracking

Track feature usage against subscription limits:

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

interface FeatureUsageTracker {
  featureApiId: string;
  featureName: string;
  limit: number | null;
  currentUsage: number;
  percentageUsed: number | null;
  status: 'ok' | 'warning' | 'critical' | 'exceeded';
}

// Create a comprehensive usage tracking system
class UsageTracker {
  private features: SimpleSchemaTypes.SubscriptionFeature[] = [];
  
  async initialize(): Promise<void> {
    this.features = await client.subscriptionFeatures.list();
  }
  
  // Get usage status for numeric features
  async getUsageStatus(): Promise<FeatureUsageTracker[]> {
    const trackers: FeatureUsageTracker[] = [];
    
    const numericFeatures = this.features.filter(f => 
      f.attributes.feature_type === 'numeric' && 
      f.attributes.enabled
    );
    
    for (const feature of numericFeatures) {
      // In real implementation, you would fetch actual usage data
      // This is a mock implementation
      const currentUsage = await this.getCurrentUsage(feature.attributes.api_id);
      const limit = feature.attributes.value || 0;
      
      const percentageUsed = limit > 0 ? (currentUsage / limit) * 100 : 0;
      
      let status: 'ok' | 'warning' | 'critical' | 'exceeded' = 'ok';
      if (percentageUsed >= 100) status = 'exceeded';
      else if (percentageUsed >= 90) status = 'critical';
      else if (percentageUsed >= 75) status = 'warning';
      
      trackers.push({
        featureApiId: feature.attributes.api_id,
        featureName: feature.attributes.name,
        limit: limit,
        currentUsage: currentUsage,
        percentageUsed: Math.round(percentageUsed * 100) / 100,
        status: status
      });
    }
    
    return trackers;
  }
  
  // Mock method - replace with actual usage fetching
  private async getCurrentUsage(featureApiId: string): Promise<number> {
    // This would typically fetch from usage counters or other APIs
    const mockUsage: Record<string, number> = {
      'monthly_api_calls': 750000,
      'storage_gb': 45,
      'bandwidth_gb': 120,
      'build_minutes': 800
    };
    
    return mockUsage[featureApiId] || 0;
  }
  
  // Generate usage alerts
  async getUsageAlerts(): Promise<string[]> {
    const alerts: string[] = [];
    const usageStatus = await this.getUsageStatus();
    
    usageStatus.forEach(tracker => {
      if (tracker.status === 'exceeded') {
        alerts.push(`🚨 ${tracker.featureName} limit exceeded! Used: ${tracker.currentUsage} / ${tracker.limit}`);
      } else if (tracker.status === 'critical') {
        alerts.push(`⚠️  ${tracker.featureName} usage critical: ${tracker.percentageUsed}% used`);
      } else if (tracker.status === 'warning') {
        alerts.push(`📊 ${tracker.featureName} usage high: ${tracker.percentageUsed}% used`);
      }
    });
    
    return alerts;
  }
}

// Example usage monitoring
async function monitorFeatureUsage() {
  const tracker = new UsageTracker();
  await tracker.initialize();
  
  // Get current usage status
  const usageStatus = await tracker.getUsageStatus();
  console.log('Feature Usage Status:');
  usageStatus.forEach(status => {
    const emoji = status.status === 'ok' ? '✅' : 
                  status.status === 'warning' ? '⚠️' : 
                  status.status === 'critical' ? '🔴' : '🚨';
    
    console.log(`${emoji} ${status.featureName}: ${status.currentUsage}/${status.limit || '∞'} (${status.percentageUsed || 0}%)`);
  });
  
  // Check for alerts
  const alerts = await tracker.getUsageAlerts();
  if (alerts.length > 0) {
    console.log('\nAlerts:');
    alerts.forEach(alert => console.log(alert));
  }
}
```

## Error Handling Examples

Handle various error scenarios when working with subscription features:

```typescript
import { buildClient, SimpleSchemaTypes, ApiError } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

// Comprehensive error handling for subscription features
async function safeListSubscriptionFeatures() {
  try {
    const features = await client.subscriptionFeatures.list();
    return { success: true, data: features, error: null };
  } catch (error) {
    if (error instanceof ApiError) {
      // Handle specific API errors
      switch (error.statusCode) {
        case 401:
          return { 
            success: false, 
            data: null, 
            error: 'Authentication failed. Check your API token.' 
          };
        
        case 403:
          return { 
            success: false, 
            data: null, 
            error: 'Access forbidden. You may not have permission to view subscription features.' 
          };
        
        case 429:
          const retryAfter = error.headers?.['x-ratelimit-reset'];
          return { 
            success: false, 
            data: null, 
            error: `Rate limit exceeded. Retry after ${retryAfter}` 
          };
        
        default:
          return { 
            success: false, 
            data: null, 
            error: `API error: ${error.message}` 
          };
      }
    }
    
    // Handle network or other errors
    return { 
      success: false, 
      data: null, 
      error: `Unexpected error: ${error instanceof Error ? error.message : 'Unknown error'}` 
    };
  }
}

// Retry mechanism for transient failures
async function listSubscriptionFeaturesWithRetry(maxRetries: number = 3) {
  let lastError: Error | null = null;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const features = await client.subscriptionFeatures.list();
      return features;
    } catch (error) {
      lastError = error as Error;
      
      if (error instanceof ApiError && error.statusCode === 429) {
        // Rate limited - wait before retry
        const retryAfter = parseInt(error.headers?.['x-ratelimit-reset'] || '60');
        console.log(`Rate limited. Waiting ${retryAfter} seconds before retry ${attempt}/${maxRetries}`);
        await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
      } else if (attempt < maxRetries) {
        // Exponential backoff for other errors
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Attempt ${attempt} failed. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw lastError || new Error('Failed to list subscription features after retries');
}
```

## Best Practices for Feature Management

### 1. Cache Feature Data

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: '<YOUR_API_TOKEN>' 
});

class FeatureCache {
  private cache: Map<string, SimpleSchemaTypes.SubscriptionFeature> = new Map();
  private lastFetch: Date | null = null;
  private cacheDuration: number = 3600000; // 1 hour in milliseconds
  
  async getFeatures(): Promise<SimpleSchemaTypes.SubscriptionFeature[]> {
    const now = new Date();
    
    // Check if cache is valid
    if (this.lastFetch && 
        (now.getTime() - this.lastFetch.getTime()) < this.cacheDuration) {
      return Array.from(this.cache.values());
    }
    
    // Refresh cache
    const features = await client.subscriptionFeatures.list();
    this.cache.clear();
    
    features.forEach(feature => {
      this.cache.set(feature.attributes.api_id, feature);
    });
    
    this.lastFetch = now;
    return features;
  }
  
  async getFeature(apiId: string): Promise<SimpleSchemaTypes.SubscriptionFeature | null> {
    await this.getFeatures(); // Ensures cache is populated
    return this.cache.get(apiId) || null;
  }
  
  invalidate(): void {
    this.cache.clear();
    this.lastFetch = null;
  }
}
```

### 2. Feature-Based Access Control

```typescript
// Implement feature-based access control
class FeatureGate {
  private featureCache: FeatureCache;
  
  constructor() {
    this.featureCache = new FeatureCache();
  }
  
  // Gate function execution based on features
  async requireFeature<T>(
    featureApiId: string, 
    fn: () => Promise<T>
  ): Promise<T> {
    const feature = await this.featureCache.getFeature(featureApiId);
    
    if (!feature || !feature.attributes.enabled) {
      throw new Error(
        `Feature "${featureApiId}" is not available in your subscription plan. ` +
        `Please upgrade to access this functionality.`
      );
    }
    
    return fn();
  }
  
  // Check multiple features
  async requireAllFeatures<T>(
    featureApiIds: string[], 
    fn: () => Promise<T>
  ): Promise<T> {
    const features = await this.featureCache.getFeatures();
    const missingFeatures: string[] = [];
    
    for (const apiId of featureApiIds) {
      const feature = features.find(f => f.attributes.api_id === apiId);
      if (!feature || !feature.attributes.enabled) {
        missingFeatures.push(apiId);
      }
    }
    
    if (missingFeatures.length > 0) {
      throw new Error(
        `The following features are required but not available: ${missingFeatures.join(', ')}. ` +
        `Please upgrade your plan.`
      );
    }
    
    return fn();
  }
}

// Example usage
const featureGate = new FeatureGate();

// Gate webhook creation
async function createWebhook(data: any) {
  return featureGate.requireFeature('webhooks', async () => {
    // Webhook creation logic here
    return client.webhooks.create(data);
  });
}

// Gate multiple features
async function advancedOperation() {
  return featureGate.requireAllFeatures(
    ['graphql_content_api', 'webhooks', 'custom_roles'],
    async () => {
      // Advanced operation requiring multiple features
      console.log('Performing advanced operation...');
    }
  );
}
```

### 3. Progressive Feature Enhancement

```typescript
// Build UI/functionality that adapts to available features
class ProgressiveEnhancement {
  private features: Map<string, boolean> = new Map();
  
  async initialize() {
    const featureList = await client.subscriptionFeatures.list();
    
    featureList.forEach(feature => {
      this.features.set(
        feature.attributes.api_id, 
        feature.attributes.enabled
      );
    });
  }
  
  // Build configuration based on available features
  getAvailableOperations() {
    const operations = {
      basic: ['read', 'write', 'delete'],
      advanced: []
    };
    
    if (this.features.get('webhooks')) {
      operations.advanced.push('webhook_notifications');
    }
    
    if (this.features.get('graphql_content_api')) {
      operations.advanced.push('graphql_queries');
    }
    
    if (this.features.get('scheduled_publishing')) {
      operations.advanced.push('schedule_content');
    }
    
    if (this.features.get('custom_roles')) {
      operations.advanced.push('role_management');
    }
    
    return operations;
  }
  
  // Generate UI configuration
  getUIConfiguration() {
    return {
      showWebhooksTab: this.features.get('webhooks') || false,
      showAdvancedMedia: this.features.get('advanced_media_area') || false,
      showScheduling: this.features.get('scheduled_publishing') || false,
      showSSO: this.features.get('sso') || false,
      maxUploadSize: this.getMaxUploadSize()
    };
  }
  
  private getMaxUploadSize(): number {
    // Different plans might have different upload limits
    if (this.features.get('enterprise_uploads')) {
      return 5 * 1024 * 1024 * 1024; // 5GB
    } else if (this.features.get('pro_uploads')) {
      return 1024 * 1024 * 1024; // 1GB
    }
    return 250 * 1024 * 1024; // 250MB default
  }
}
```

## Related Resources

- [Subscription Limits](./subscription-limit.md) - Detailed numeric limits for features
- [Usage Counters](./usage-counter.md) - Track actual usage against limits
- [Daily Usage](./daily-usage.md) - Monitor daily consumption patterns
- [Site Resource](../04-site-configuration/site.md) - Site-level subscription information
- [API Error Handling](../../04-advanced-topics/error-handling-patterns.md) - Comprehensive error handling guide

## Common Feature API IDs

Here are some common feature API IDs you might encounter:

- `graphql_content_api` - GraphQL Content Delivery API
- `webhooks` - Webhook notifications
- `scheduled_publishing` - Schedule content publication
- `custom_roles` - Custom user roles and permissions
- `sso` - Single Sign-On
- `advanced_media_area` - Advanced media management
- `sandbox_environments` - Development/staging environments
- `build_triggers` - Build triggers integration
- `plugins` - Custom plugins
- `locales` - Multi-language support
- `monthly_api_calls` - API call limits
- `storage_gb` - Storage limits in GB
- `bandwidth_gb` - Bandwidth limits in GB
- `build_minutes` - Build time limits