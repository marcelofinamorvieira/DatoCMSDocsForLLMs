# Dashboard API - Billing & Subscriptions (Merged Documentation)

This file contains merged documentation for all billing and subscription-related resources in the Dashboard API.

## Overview

DatoCMS offers two pricing models:
- **Per-Site Pricing** (Legacy): Each site has its own subscription
- **Per-Owner Pricing** (Modern): Organization-level billing with shared resources

The Dashboard API provides a unified interface through the `subscriptions` resource that works with both models.

## Client Initialization

```typescript
import { buildClient } from '@datocms/dashboard-client';

const dashboardClient = buildClient({
  apiToken: 'YOUR_ACCOUNT_API_TOKEN',
  environment: 'production'
});
```

## Pricing Models

### Per-Site Pricing (Legacy)
- Each site has individual subscription
- Billing tied directly to sites
- Limited resource sharing
- Used for older accounts

### Per-Owner Pricing (Modern)
- Organization-level billing
- Resources shared across all sites
- More flexible and cost-effective
- Default for new accounts

## Core Resources

| Resource | Type | Description |
|----------|------|-------------|
| `billingProfiles` | `per_site_pricing_billing_profile` | Legacy billing profiles |
| `perOwnerPricingBillingProfiles` | `per_owner_pricing_billing_profile` | Modern billing profiles |
| `subscriptions` | `subscription` | Unified subscription interface |
| `siteSubscriptions` | `per_site_pricing_subscription` | Legacy site subscriptions |
| `perOwnerPricingSubscriptions` | `per_owner_pricing_subscription` | Modern org subscriptions |
| `plans` | `plan` | Legacy pricing plans |
| `perOwnerPricingPlans` | `per_owner_pricing_plan` | Modern pricing plans |
| `invoices` | `invoice` | Billing invoices |
| `paymentIntents` | `payment_intent` | Stripe payment processing |
| `resourceUsages` | `resource_usage` | Site usage metrics |
| `dailyUsages` | `daily_usage` | Daily usage breakdown |
| `subscriptionFeatures` | `subscription_feature` | Plan features |
| `subscriptionLimits` | `subscription_limit` | Resource limits |

## Billing Profiles

### Per-Site Pricing Billing Profile

**Resource**: `dashboardClient.billingProfiles`  
**Type**: `per_site_pricing_billing_profile`

```typescript
// Create billing profile
const profile = await dashboardClient.billingProfiles.create({
  company_data: {
    company_name: 'Acme Inc.',
    country: 'US',
    vat_number: 'US123456789',
    address_line_1: '123 Main St',
    city: 'New York',
    zip_code: '10001',
    state: 'NY'
  },
  billing_email: 'billing@acme.com'
});

// Update billing profile
const updated = await dashboardClient.billingProfiles.update('profile-id', {
  billing_email: 'finance@acme.com'
});

// Add payment method
const paymentIntent = await dashboardClient.paymentIntents.create({
  billing_profile: { type: 'per_site_pricing_billing_profile', id: 'profile-id' }
});
// Redirect to paymentIntent.payment_url
```

### Per-Owner Pricing Billing Profile

**Resource**: `dashboardClient.perOwnerPricingBillingProfiles`  
**Type**: `per_owner_pricing_billing_profile`

```typescript
// List organization billing profiles
const profiles = await dashboardClient.perOwnerPricingBillingProfiles.list({
  filter: { organization: { eq: 'org-id' } }
});

// Find specific profile
const profile = await dashboardClient.perOwnerPricingBillingProfiles.find('profile-id');

// Update profile
const updated = await dashboardClient.perOwnerPricingBillingProfiles.update('profile-id', {
  company_data: {
    vat_number: 'US987654321'
  }
});
```

### Billing Profile Attributes

```typescript
interface BillingProfile {
  id: string;
  type: string;
  company_name: string;
  country: string;
  vat_number?: string;
  address_line_1: string;
  address_line_2?: string;
  city: string;
  zip_code: string;
  state?: string;
  billing_email: string;
  has_payment_method: boolean;
  payment_method_brand?: 'visa' | 'mastercard' | 'amex' | 'discover';
  payment_method_last4?: string;
  payment_method_exp_month?: number;
  payment_method_exp_year?: number;
  created_at: string;
  updated_at: string;
}
```

## Subscriptions (Unified Interface)

### Resource: `dashboardClient.subscriptions`

**Type**: `subscription`  
**Purpose**: Unified interface for both pricing models

```typescript
// List all subscriptions
const subscriptions = await dashboardClient.subscriptions.list();

// Find specific subscription
const subscription = await dashboardClient.subscriptions.find('sub-id');

// Activate subscription
const activated = await dashboardClient.subscriptions.activate('sub-id');

// Cancel subscription
const cancelled = await dashboardClient.subscriptions.cancel('sub-id');

// Returns:
{
  id: 'sub-123',
  type: 'subscription',
  plan_name: 'Professional',
  status: 'active', // 'active' | 'canceled' | 'past_due' | 'unpaid' | 'trialing'
  current_period_start: '2024-01-01T00:00:00Z',
  current_period_end: '2024-02-01T00:00:00Z',
  trial_end: null,
  cancel_at_period_end: false,
  
  // Polymorphic relationship
  owner: {
    type: 'site', // or 'organization'
    id: '456'
  }
}
```

## Site Subscriptions (Per-Site Pricing)

### Resource: `dashboardClient.siteSubscriptions`

**Type**: `per_site_pricing_subscription`

```typescript
// Create subscription for a site
const subscription = await dashboardClient.siteSubscriptions.create({
  site_id: 'site-123',
  plan_id: 'professional',
  billing_profile_id: 'profile-456'
});

// Update subscription plan
const upgraded = await dashboardClient.siteSubscriptions.update('sub-id', {
  plan_id: 'enterprise'
});

// Cancel subscription
await dashboardClient.siteSubscriptions.cancel('sub-id');

// Reactivate cancelled subscription
await dashboardClient.siteSubscriptions.reactivate('sub-id');
```

## Organization Subscriptions (Per-Owner Pricing)

### Resource: `dashboardClient.perOwnerPricingSubscriptions`

**Type**: `per_owner_pricing_subscription`

```typescript
// Create organization subscription
const subscription = await dashboardClient.perOwnerPricingSubscriptions.create({
  organization_id: 'org-123',
  plan_id: 'professional',
  billing_profile_id: 'profile-789'
});

// Update subscription
const updated = await dashboardClient.perOwnerPricingSubscriptions.update('sub-id', {
  plan_id: 'enterprise'
});

// Manage extras (add-ons)
const withExtras = await dashboardClient.perOwnerPricingSubscriptions.updateExtras('sub-id', {
  extra_records: 100000,
  extra_build_minutes: 5000,
  extra_traffic_bytes: 1000000000 // 1GB
});
```

### Subscription Attributes

```typescript
interface Subscription {
  id: string;
  type: string;
  status: 'active' | 'canceled' | 'past_due' | 'unpaid' | 'trialing';
  plan_name: string;
  current_period_start: string;
  current_period_end: string;
  trial_end: string | null;
  cancel_at_period_end: boolean;
  canceled_at: string | null;
  
  // For per-owner pricing
  extra_records?: number;
  extra_build_minutes?: number;
  extra_traffic_bytes?: number;
  
  // Relationships
  plan?: Plan;
  billing_profile?: BillingProfile;
  invoices?: Invoice[];
}
```

## Plans

### Legacy Plans (Per-Site)

**Resource**: `dashboardClient.plans`  
**Type**: `plan`

```typescript
// List available plans
const plans = await dashboardClient.plans.list();

// Common plans:
[
  {
    id: 'free',
    name: 'Free',
    monthly_price: 0,
    yearly_price: 0,
    limits: {
      records: 300,
      build_minutes: 100,
      traffic_bytes: 1073741824, // 1GB
      api_calls: 10000,
      sites: 1
    }
  },
  {
    id: 'basic',
    name: 'Basic',
    monthly_price: 1900, // $19 in cents
    yearly_price: 18000,
    limits: {
      records: 3000,
      build_minutes: 500,
      traffic_bytes: 10737418240, // 10GB
      api_calls: 100000,
      sites: 1
    }
  },
  {
    id: 'professional',
    name: 'Professional',
    monthly_price: 9900,
    yearly_price: 99000,
    limits: {
      records: 50000,
      build_minutes: 2500,
      traffic_bytes: 107374182400, // 100GB
      api_calls: 1000000,
      sites: 1
    }
  }
]
```

### Modern Plans (Per-Owner)

**Resource**: `dashboardClient.perOwnerPricingPlans`  
**Type**: `per_owner_pricing_plan`

```typescript
// List available plans
const plans = await dashboardClient.perOwnerPricingPlans.list();

// Find specific plan
const plan = await dashboardClient.perOwnerPricingPlans.find('professional');

// Plan structure:
{
  id: 'professional',
  type: 'per_owner_pricing_plan',
  name: 'Professional',
  base_monthly_price: 10000, // $100
  base_yearly_price: 100000,
  included_resources: {
    sites: 10,
    records: 100000,
    build_minutes: 5000,
    traffic_bytes: 214748364800, // 200GB
    api_calls: 5000000,
    users: 10
  },
  extra_prices: {
    record: 0.02, // $0.02 per 1k records
    build_minute: 0.05,
    traffic_byte: 0.00000001 // per byte
  }
}
```

## Invoices

### Resource: `dashboardClient.invoices`

**Type**: `invoice`

```typescript
// List invoices
const invoices = await dashboardClient.invoices.list({
  filter: { subscription: { eq: 'sub-id' } },
  order_by: 'created_at_DESC'
});

// Find specific invoice
const invoice = await dashboardClient.invoices.find('inv-id');

// Download invoice PDF
const pdfUrl = invoice.pdf_url;

// Invoice structure:
{
  id: 'inv-123',
  type: 'invoice',
  number: 'INV-2024-001',
  status: 'paid', // 'draft' | 'open' | 'paid' | 'void' | 'uncollectible'
  amount_total: 10000, // $100.00
  amount_paid: 10000,
  amount_due: 0,
  currency: 'usd',
  created_at: '2024-01-01T00:00:00Z',
  due_date: '2024-01-15T00:00:00Z',
  paid_at: '2024-01-05T00:00:00Z',
  pdf_url: 'https://...',
  hosted_invoice_url: 'https://...',
  
  line_items: [
    {
      description: 'Professional Plan - Jan 2024',
      amount: 9900,
      quantity: 1
    },
    {
      description: 'Extra records (50k)',
      amount: 100,
      quantity: 50
    }
  ],
  
  // Relationships
  subscription?: Subscription;
  billing_profile?: BillingProfile;
}
```

## Payment Intents

### Resource: `dashboardClient.paymentIntents`

**Type**: `payment_intent`  
**Purpose**: Handle Stripe payment setup

```typescript
// Create payment intent for new payment method
const intent = await dashboardClient.paymentIntents.create({
  billing_profile: { 
    type: 'per_owner_pricing_billing_profile', 
    id: 'profile-id' 
  }
});

// Redirect user to complete payment
window.location.href = intent.payment_url;

// Payment intent structure:
{
  id: 'pi-123',
  type: 'payment_intent',
  status: 'requires_payment_method', // Stripe status
  payment_url: 'https://checkout.stripe.com/...',
  success_url: 'https://app.example.com/billing/success',
  cancel_url: 'https://app.example.com/billing/cancel',
  created_at: '2024-01-01T00:00:00Z'
}
```

## Usage Monitoring

### Resource Usage

**Resource**: `dashboardClient.resourceUsages`  
**Type**: `resource_usage`

```typescript
// Get current usage for a site
const usage = await dashboardClient.resourceUsages.find('site-id');

// Usage structure:
{
  id: 'site-id',
  type: 'resource_usage',
  
  // Current billing period
  records_count: 45000,
  build_minutes_used: 1250,
  traffic_bytes_used: 53687091200, // 50GB
  api_calls_used: 450000,
  
  // Limits (based on subscription)
  records_limit: 50000,
  build_minutes_limit: 2500,
  traffic_bytes_limit: 107374182400, // 100GB
  api_calls_limit: 1000000,
  
  // Usage percentages
  records_percentage: 90,
  build_minutes_percentage: 50,
  traffic_percentage: 50,
  api_calls_percentage: 45,
  
  // Overage status
  is_over_records_limit: false,
  is_over_build_minutes_limit: false,
  is_over_traffic_limit: false,
  is_over_api_calls_limit: false
}
```

### Daily Usage

**Resource**: `dashboardClient.dailyUsages`  
**Type**: `daily_usage`

```typescript
// Get daily usage breakdown
const dailyUsages = await dashboardClient.dailyUsages.list({
  filter: {
    site: { eq: 'site-id' },
    date: { gte: '2024-01-01', lte: '2024-01-31' }
  }
});

// Daily usage structure:
{
  id: '2024-01-15',
  type: 'daily_usage',
  date: '2024-01-15',
  
  // Daily metrics
  api_calls: 15000,
  build_minutes: 45,
  traffic_bytes: 1073741824, // 1GB
  records_created: 150,
  records_deleted: 50,
  
  // Cumulative for billing period
  cumulative_api_calls: 225000,
  cumulative_build_minutes: 675,
  cumulative_traffic_bytes: 16106127360 // 15GB
}
```

## Subscription Features

### Resource: `dashboardClient.subscriptionFeatures`

**Type**: `subscription_feature`

```typescript
// List features for a subscription
const features = await dashboardClient.subscriptionFeatures.list({
  filter: { subscription: { eq: 'sub-id' } }
});

// Common features:
[
  {
    id: 'api-access',
    type: 'subscription_feature',
    name: 'API Access',
    enabled: true
  },
  {
    id: 'custom-domains',
    type: 'subscription_feature', 
    name: 'Custom Domains',
    enabled: true,
    limit: 5
  },
  {
    id: 'environments',
    type: 'subscription_feature',
    name: 'Environments',
    enabled: true,
    limit: 2
  },
  {
    id: 'sso',
    type: 'subscription_feature',
    name: 'Single Sign-On',
    enabled: false // Enterprise only
  }
]
```

## Subscription Limits

### Resource: `dashboardClient.subscriptionLimits`

**Type**: `subscription_limit`

```typescript
// Get limits for a subscription
const limits = await dashboardClient.subscriptionLimits.find('sub-id');

// Limits structure:
{
  id: 'sub-id',
  type: 'subscription_limit',
  
  // Resource limits
  records: 50000,
  build_minutes: 2500,
  traffic_bytes: 107374182400, // 100GB
  api_calls: 1000000,
  
  // Feature limits
  sites: 10,
  users: 25,
  roles: 10,
  environments: 2,
  locales: 10,
  
  // Storage limits
  upload_size_bytes: 536870912, // 500MB per file
  total_upload_size_bytes: 10737418240, // 10GB total
  
  // API limits
  rate_limit_per_second: 30,
  graphql_complexity_limit: 10000
}
```

## Common Workflows

### Setting Up a New Subscription

```typescript
async function setupSubscription(orgId: string) {
  // 1. Create billing profile
  const profile = await dashboardClient.perOwnerPricingBillingProfiles.create({
    organization: { type: 'organization', id: orgId },
    company_data: {
      company_name: 'Tech Corp',
      country: 'US',
      address_line_1: '123 Tech St',
      city: 'San Francisco',
      zip_code: '94105',
      state: 'CA'
    },
    billing_email: 'billing@techcorp.com'
  });

  // 2. Set up payment method
  const paymentIntent = await dashboardClient.paymentIntents.create({
    billing_profile: { 
      type: 'per_owner_pricing_billing_profile', 
      id: profile.id 
    }
  });

  // 3. Redirect to Stripe
  console.log('Complete payment at:', paymentIntent.payment_url);
  
  // 4. After payment completion, create subscription
  const subscription = await dashboardClient.perOwnerPricingSubscriptions.create({
    organization_id: orgId,
    plan_id: 'professional',
    billing_profile_id: profile.id
  });

  return subscription;
}
```

### Upgrading a Subscription

```typescript
async function upgradeSubscription(subId: string) {
  // 1. Get current subscription
  const current = await dashboardClient.subscriptions.find(subId);
  
  // 2. Check available plans
  const plans = await dashboardClient.perOwnerPricingPlans.list();
  const enterprisePlan = plans.find(p => p.id === 'enterprise');
  
  // 3. Calculate prorated amount
  const daysRemaining = Math.ceil(
    (new Date(current.current_period_end).getTime() - Date.now()) / 
    (1000 * 60 * 60 * 24)
  );
  
  // 4. Update subscription
  const upgraded = await dashboardClient.perOwnerPricingSubscriptions.update(subId, {
    plan_id: 'enterprise'
  });
  
  // 5. Add extras if needed
  if (needsMoreResources) {
    await dashboardClient.perOwnerPricingSubscriptions.updateExtras(subId, {
      extra_records: 500000,
      extra_build_minutes: 10000
    });
  }
  
  return upgraded;
}
```

### Monitoring Usage and Costs

```typescript
async function monitorUsageAndCosts(siteId: string) {
  // 1. Get current usage
  const usage = await dashboardClient.resourceUsages.find(siteId);
  
  // 2. Check for overages
  const overages = {
    records: usage.is_over_records_limit,
    buildMinutes: usage.is_over_build_minutes_limit,
    traffic: usage.is_over_traffic_limit,
    apiCalls: usage.is_over_api_calls_limit
  };
  
  if (Object.values(overages).some(v => v)) {
    console.warn('Resource limits exceeded:', overages);
  }
  
  // 3. Get daily breakdown for current month
  const startOfMonth = new Date();
  startOfMonth.setDate(1);
  
  const dailyUsages = await dashboardClient.dailyUsages.list({
    filter: {
      site: { eq: siteId },
      date: { gte: startOfMonth.toISOString().split('T')[0] }
    }
  });
  
  // 4. Calculate projected monthly usage
  const daysInMonth = new Date(
    startOfMonth.getFullYear(), 
    startOfMonth.getMonth() + 1, 
    0
  ).getDate();
  
  const daysPassed = new Date().getDate();
  const latestDaily = dailyUsages[dailyUsages.length - 1];
  
  const projectedUsage = {
    apiCalls: (latestDaily.cumulative_api_calls / daysPassed) * daysInMonth,
    buildMinutes: (latestDaily.cumulative_build_minutes / daysPassed) * daysInMonth,
    trafficGB: (latestDaily.cumulative_traffic_bytes / daysPassed * daysInMonth) / 1073741824
  };
  
  // 5. Get recent invoices
  const invoices = await dashboardClient.invoices.list({
    filter: { 
      subscription: { eq: usage.subscription.id },
      created_at: { gte: startOfMonth.toISOString() }
    }
  });
  
  const totalCost = invoices.reduce((sum, inv) => sum + inv.amount_total, 0);
  
  return {
    currentUsage: usage,
    projectedUsage,
    monthlySpend: totalCost / 100, // Convert from cents
    overages
  };
}
```

### Handling Trial Periods

```typescript
async function manageTrial(orgId: string) {
  // 1. Start trial
  const subscription = await dashboardClient.perOwnerPricingSubscriptions.create({
    organization_id: orgId,
    plan_id: 'professional',
    trial_days: 14 // 14-day trial
  });
  
  // 2. Monitor trial status
  const checkTrialStatus = async () => {
    const sub = await dashboardClient.subscriptions.find(subscription.id);
    
    if (sub.status === 'trialing') {
      const trialEnd = new Date(sub.trial_end);
      const daysLeft = Math.ceil(
        (trialEnd.getTime() - Date.now()) / (1000 * 60 * 60 * 24)
      );
      
      console.log(`Trial ends in ${daysLeft} days`);
      
      // Send reminder when trial is ending
      if (daysLeft <= 3) {
        await sendTrialEndingReminder(orgId);
      }
    }
  };
  
  // 3. Convert trial to paid
  const convertToPaid = async () => {
    // Add payment method
    const paymentIntent = await dashboardClient.paymentIntents.create({
      billing_profile: { 
        type: 'per_owner_pricing_billing_profile', 
        id: subscription.billing_profile.id 
      }
    });
    
    // After payment method added, subscription automatically converts
    return paymentIntent.payment_url;
  };
  
  return {
    subscription,
    checkTrialStatus,
    convertToPaid
  };
}
```

### Cancellation and Reactivation

```typescript
async function handleCancellation(subId: string) {
  // 1. Cancel at period end (recommended)
  const cancelled = await dashboardClient.subscriptions.cancel(subId);
  
  console.log('Subscription will end on:', cancelled.current_period_end);
  
  // 2. Immediate cancellation (if needed)
  const immediate = await dashboardClient.subscriptions.cancel(subId, {
    immediate: true
  });
  
  // 3. Reactivate before period end
  if (cancelled.cancel_at_period_end && !cancelled.canceled_at) {
    const reactivated = await dashboardClient.subscriptions.activate(subId);
    console.log('Subscription reactivated');
  }
  
  // 4. Handle downgrade to free plan
  const downgradedToFree = async (siteId: string) => {
    // Check if within free limits
    const usage = await dashboardClient.resourceUsages.find(siteId);
    const freeLimits = {
      records: 300,
      buildMinutes: 100,
      trafficGB: 1
    };
    
    if (usage.records_count > freeLimits.records) {
      console.warn(`Reduce records from ${usage.records_count} to ${freeLimits.records}`);
    }
    
    return usage;
  };
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/dashboard-client';

async function handleBillingErrors() {
  try {
    await dashboardClient.subscriptions.create({
      site_id: 'site-123',
      plan_id: 'invalid-plan'
    });
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.response?.status) {
        case 402:
          // Payment required
          console.error('Payment method required');
          // Redirect to add payment method
          break;
          
        case 422:
          // Validation error
          if (error.errors?.some(e => e.detail.includes('payment'))) {
            console.error('Invalid payment method');
          } else if (error.errors?.some(e => e.detail.includes('plan'))) {
            console.error('Invalid plan selected');
          }
          break;
          
        case 409:
          // Conflict
          console.error('Subscription already exists');
          break;
          
        case 429:
          // Rate limited
          console.error('Too many requests, retry later');
          break;
      }
    }
  }
}

// Handle payment failures
async function handlePaymentFailure(subId: string) {
  const subscription = await dashboardClient.subscriptions.find(subId);
  
  if (subscription.status === 'past_due') {
    // Payment failed, update payment method
    const intent = await dashboardClient.paymentIntents.create({
      billing_profile: subscription.billing_profile
    });
    
    console.log('Update payment method:', intent.payment_url);
    
    // Check for retry
    const checkRetry = setInterval(async () => {
      const updated = await dashboardClient.subscriptions.find(subId);
      if (updated.status === 'active') {
        console.log('Payment successful!');
        clearInterval(checkRetry);
      }
    }, 60000); // Check every minute
  }
}
```

## Rate Limiting

Billing endpoints have specific limits:
- **Subscription operations**: 10 requests per 3 seconds
- **Invoice operations**: 30 requests per 3 seconds
- **Usage queries**: 60 requests per 3 seconds
- **Payment operations**: 5 requests per 3 seconds

## Best Practices

### 1. Payment Method Management
```typescript
// Always verify payment method before subscription changes
async function verifyPaymentMethod(profileId: string) {
  const profile = await dashboardClient.billingProfiles.find(profileId);
  
  if (!profile.has_payment_method) {
    throw new Error('Payment method required');
  }
  
  // Check expiration
  const expYear = profile.payment_method_exp_year;
  const expMonth = profile.payment_method_exp_month;
  const now = new Date();
  
  if (expYear < now.getFullYear() || 
      (expYear === now.getFullYear() && expMonth < now.getMonth() + 1)) {
    console.warn('Payment method expired');
  }
}
```

### 2. Usage Monitoring
```typescript
// Set up usage alerts
async function setupUsageAlerts(siteId: string) {
  const checkUsage = async () => {
    const usage = await dashboardClient.resourceUsages.find(siteId);
    
    // Alert at 80% usage
    const alerts = [];
    if (usage.records_percentage > 80) {
      alerts.push(`Records at ${usage.records_percentage}%`);
    }
    if (usage.traffic_percentage > 80) {
      alerts.push(`Traffic at ${usage.traffic_percentage}%`);
    }
    
    if (alerts.length > 0) {
      await sendUsageAlert(alerts);
    }
  };
  
  // Check daily
  setInterval(checkUsage, 24 * 60 * 60 * 1000);
}
```

### 3. Cost Optimization
```typescript
// Analyze usage patterns for optimization
async function optimizeCosts(orgId: string) {
  const sites = await dashboardClient.sites.list({
    filter: { organization: { eq: orgId } }
  });
  
  const usageData = await Promise.all(
    sites.map(site => dashboardClient.resourceUsages.find(site.id))
  );
  
  // Find underutilized resources
  const recommendations = [];
  
  usageData.forEach((usage, i) => {
    if (usage.records_percentage < 20 && usage.records_limit > 10000) {
      recommendations.push({
        site: sites[i].name,
        action: 'Consider downgrading plan',
        savings: '$50/month'
      });
    }
  });
  
  return recommendations;
}
```

### 4. Subscription Lifecycle Events
```typescript
// Handle subscription events
async function subscriptionLifecycle(subId: string) {
  const events = {
    onUpgrade: async (oldPlan: string, newPlan: string) => {
      console.log(`Upgraded from ${oldPlan} to ${newPlan}`);
      await notifyTeam('subscription.upgraded', { oldPlan, newPlan });
    },
    
    onDowngrade: async (oldPlan: string, newPlan: string) => {
      console.log(`Downgraded from ${oldPlan} to ${newPlan}`);
      await checkResourceLimits(subId);
    },
    
    onCancel: async () => {
      console.log('Subscription cancelled');
      await exportData(subId);
      await notifyTeam('subscription.cancelled');
    },
    
    onPaymentFailed: async () => {
      console.log('Payment failed');
      await notifyBilling('payment.failed', { subId });
    }
  };
  
  return events;
}
```

## Related Resources

- [Account Management](./merged-account-management.md)
- [Organization Management](./merged-organization-management.md)
- [Site Management](./merged-site-management.md)
- [Authentication Setup](../00-quick-start/authentication.md)