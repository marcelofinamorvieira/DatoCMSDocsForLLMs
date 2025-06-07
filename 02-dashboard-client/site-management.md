# Dashboard API - Site Management (Merged Documentation)

This file contains merged documentation for all site-related resources in both the CMA and Dashboard APIs:
- Sites
- Site Invitations
- Site Plans
- Site Transfers
- Job Results (for async operations)

## Overview

Site management in DatoCMS involves both project-level configuration (via CMA) and cross-organization site management (via Dashboard API). The CMA provides detailed site configuration while the Dashboard API handles site creation, transfers, and billing.

## Client Initialization

```typescript
// CMA Client - for site configuration
import { buildClient } from '@datocms/cma-client';

const cmaClient = buildClient({
  apiToken: 'YOUR_PROJECT_API_TOKEN',
  environment: 'main'
});

// Dashboard Client - for cross-org management
import { buildClient as buildDashboardClient } from '@datocms/dashboard-client';

const dashboardClient = buildDashboardClient({
  apiToken: 'YOUR_ACCOUNT_API_TOKEN'
});
```

## Sites Resource

### CMA Site Resource (Project Configuration)

**Resource**: `cmaClient.site`  
**Type**: `site` (singleton)  
**Purpose**: Configure project-level settings

#### Get Site Configuration

```typescript
const site = await cmaClient.site.find();

// Returns:
{
  id: '123',
  type: 'site',
  name: 'My Project',
  domain: null, // Custom domain if configured
  internal_subdomain: 'my-project',
  frontend_url: 'https://my-project.admin.datocms.com',
  
  // Localization
  locales: ['en', 'es', 'it'],
  timezone: 'America/New_York',
  
  // Security
  require_2fa: true,
  ip_whitelist: ['192.168.1.0/24'],
  sso_settings: {
    enabled: true,
    provider: 'google',
    domain_restrictions: ['company.com']
  },
  
  // SEO & Meta
  global_seo_settings: {
    site_name: 'My Company',
    title_suffix: ' | My Company',
    twitter_account: '@mycompany',
    facebook_page_url: 'https://facebook.com/mycompany',
    fallback_seo: {
      title: 'Welcome',
      description: 'Default description',
      image: { upload_id: '123' }
    }
  },
  
  // Branding
  favicon: { upload_id: '456' },
  theme: {
    primary_color: '#FF6B6B',
    light_color: '#FFE66D',
    dark_color: '#4ECDC4',
    accent_color: '#1A535C'
  },
  
  // Features
  no_index: false, // true prevents search indexing
  google_tag_manager_id: 'GTM-XXXXX',
  
  created_at: '2024-01-01T00:00:00Z',
  updated_at: '2024-07-01T00:00:00Z'
}
```

#### Update Site Configuration

```typescript
const updatedSite = await cmaClient.site.update({
  name: 'My Updated Project',
  locales: ['en', 'es', 'it', 'fr'], // Add French
  timezone: 'Europe/Rome',
  require_2fa: true,
  
  // Update SEO settings
  global_seo_settings: {
    site_name: 'My Company International',
    fallback_seo: {
      title: 'Welcome to Our Site',
      description: 'We provide amazing services worldwide'
    }
  },
  
  // Configure SSO
  sso_settings: {
    enabled: true,
    provider: 'okta',
    idp_url: 'https://company.okta.com',
    certificate: 'MIIDxTCCAq2gAwIBAgIQ...',
    email_attribute: 'email',
    first_name_attribute: 'given_name',
    last_name_attribute: 'family_name'
  }
});
```

### Dashboard Sites Resource (Cross-Organization)

**Resource**: `dashboardClient.sites`  
**Type**: `site`  
**Purpose**: Manage sites across organizations

#### List Sites

```typescript
// List all accessible sites
const sites = await dashboardClient.sites.list();

// Filter by organization
const orgSites = await dashboardClient.sites.list({
  filter: { organization: { eq: 'org-id' } }
});

// Include usage data
const sitesWithUsage = await dashboardClient.sites.list({
  include: ['usage_counters', 'owner'],
  per_page: 100
});
```

#### Create Site

```typescript
const newSite = await dashboardClient.sites.create({
  name: 'New Project',
  organization: { type: 'organization', id: 'org-id' },
  // Optional: specify template
  template_id: 'blog-template'
});

// Returns:
{
  id: '789',
  type: 'site',
  name: 'New Project',
  internal_subdomain: 'new-project-789',
  frontend_url: 'https://new-project-789.admin.datocms.com',
  
  // Usage counters
  usage_counters: {
    records: 0,
    uploads: 0,
    bandwidth: 0,
    api_calls: 0
  },
  
  // Relationships
  owner: { type: 'organization', id: 'org-id' },
  plan: { type: 'site_plan', id: 'free' }
}
```

#### Find Site

```typescript
const site = await dashboardClient.sites.find('site-id', {
  include: ['owner', 'plan', 'usage_counters']
});
```

#### Delete Site

```typescript
await dashboardClient.sites.destroy('site-id');
```

### Site Attributes

```typescript
interface Site {
  id: string;
  type: 'site';
  name: string;
  domain: string | null;
  internal_subdomain: string;
  frontend_url: string;
  locales: string[];
  timezone: string;
  
  // Security
  require_2fa: boolean;
  ip_whitelist: string[];
  sso_settings?: SsoSettings;
  
  // SEO
  global_seo_settings?: GlobalSeoSettings;
  favicon?: { upload_id: string };
  
  // Usage (Dashboard only)
  usage_counters?: {
    records: number;
    uploads: number;
    bandwidth: number;
    api_calls: number;
  };
  
  // Timestamps
  created_at: string;
  updated_at: string;
  
  // Relationships
  owner?: Organization | Account;
  plan?: SitePlan;
}
```

## Site Invitations Resource

### Resource: `cmaClient.siteInvitations`

**Type**: `site_invitation`  
**Purpose**: Invite users to collaborate on projects

### Create Invitation

```typescript
const invitation = await cmaClient.siteInvitations.create({
  email: 'developer@example.com',
  role: { type: 'role', id: 'editor-role-id' },
  message: 'Welcome to our content team!'
});

// Returns:
{
  id: 'inv-123',
  type: 'site_invitation',
  email: 'developer@example.com',
  status: 'pending',
  token: 'abc123...',
  expires_at: '2024-07-13T00:00:00Z',
  sent_at: '2024-07-06T00:00:00Z',
  accepted_at: null,
  
  // Relationships
  site: { type: 'site', id: 'site-123' },
  role: { type: 'role', id: 'editor-role-id' },
  inviter: { type: 'account', id: 'user-456' }
}
```

### List Invitations

```typescript
// All invitations
const invitations = await cmaClient.siteInvitations.list();

// Filter by status
const pendingInvitations = await cmaClient.siteInvitations.list({
  filter: { status: { eq: 'pending' } }
});

// Include related data
const detailedInvitations = await cmaClient.siteInvitations.list({
  include: ['role', 'inviter']
});
```

### Resend Invitation

```typescript
const resent = await cmaClient.siteInvitations.update('inv-id', {
  resend: true
});

// Or use convenience method
await cmaClient.siteInvitations.resend('inv-id');
```

### Cancel Invitation

```typescript
await cmaClient.siteInvitations.destroy('inv-id');
```

### Invitation Statuses

- `pending` - Sent, awaiting response
- `accepted` - User joined the site
- `expired` - 7 days passed without acceptance
- `cancelled` - Manually cancelled

### Invitation Attributes

```typescript
interface SiteInvitation {
  id: string;
  type: 'site_invitation';
  email: string;
  status: 'pending' | 'accepted' | 'expired' | 'cancelled';
  token: string;
  expires_at: string;
  sent_at: string;
  accepted_at: string | null;
  
  // Relationships
  site?: Site;
  role?: Role;
  inviter?: Account;
  account?: Account; // After acceptance
}
```

## Site Plans Resource

### Resource: `dashboardClient.sitePlans`

**Type**: `site_plan`  
**Purpose**: View available pricing plans and limits

### List Available Plans

```typescript
const plans = await dashboardClient.sitePlans.list();

// Returns:
[
  {
    id: 'free',
    type: 'site_plan',
    name: 'Free',
    code: 'free',
    amount: 0,
    currency: 'usd',
    billing_period: 'monthly',
    active: true,
    popular: false,
    
    // Resource limits
    limits: {
      items: 300,
      item_types: 15,
      locales: 2,
      upload_size: 10485760, // 10MB
      bandwidth: 1073741824, // 1GB
      api_calls: 10000,
      users: 1,
      roles: 2,
      environments: 1
    },
    
    // Features
    features: {
      webhooks: false,
      sso: false,
      workflows: false,
      audit_log: false,
      scheduled_publishing: false,
      content_delivery_api: true,
      management_api: true,
      images_api: true
    }
  },
  {
    id: 'professional',
    type: 'site_plan',
    name: 'Professional',
    code: 'professional',
    amount: 9900, // $99 in cents
    currency: 'usd',
    billing_period: 'monthly',
    active: true,
    popular: true,
    
    limits: {
      items: 50000,
      item_types: 100,
      locales: 10,
      upload_size: 104857600, // 100MB
      bandwidth: 107374182400, // 100GB
      api_calls: 1000000,
      users: 10,
      roles: 20,
      environments: 3
    },
    
    features: {
      webhooks: true,
      sso: true,
      workflows: true,
      audit_log: true,
      scheduled_publishing: true,
      content_delivery_api: true,
      management_api: true,
      images_api: true
    }
  }
]
```

### Find Specific Plan

```typescript
const plan = await dashboardClient.sitePlans.find('professional');
```

### Plan Comparison

```typescript
async function comparePlans() {
  const plans = await dashboardClient.sitePlans.list();
  
  const comparison = plans.map(plan => ({
    name: plan.name,
    price: `$${plan.amount / 100}/${plan.billing_period}`,
    records: plan.limits.items.toLocaleString(),
    storage: `${plan.limits.upload_size / 1048576}MB`,
    bandwidth: `${plan.limits.bandwidth / 1073741824}GB`,
    apiCalls: plan.limits.api_calls.toLocaleString(),
    hasSSO: plan.features.sso,
    hasWorkflows: plan.features.workflows
  }));
  
  return comparison;
}
```

## Site Transfers Resource

### Resource: `dashboardClient.siteTransfers`

**Type**: `site_transfer`  
**Purpose**: Transfer site ownership between accounts/organizations

### Initiate Transfer

```typescript
// Transfer returns a job that must be monitored
const job = await dashboardClient.siteTransfers.create({
  site_id: 'site-123',
  target_account_id: 'account-456', // or target_organization_id
  transfer_settings: {
    transfer_users: true,  // Include site users
    transfer_roles: true,  // Include custom roles
    keep_subscription: false // Target pays for new subscription
  }
});

// Monitor job completion
const result = await pollJobCompletion(dashboardClient, job.id);

// Transfer object from completed job:
{
  id: 'transfer-789',
  type: 'site_transfer',
  status: 'pending',
  transfer_token: 'tk_abc123...',
  expires_at: '2024-07-13T00:00:00Z',
  
  site: { type: 'site', id: 'site-123' },
  source_account: { type: 'account', id: 'account-123' },
  target_account: { type: 'account', id: 'account-456' }
}
```

### Accept Transfer (Target Account)

```typescript
// Target account must accept the transfer
const acceptedTransfer = await dashboardClient.siteTransfers.accept('transfer-id', {
  transfer_token: 'tk_abc123...' // Security token
});
```

### Decline Transfer

```typescript
await dashboardClient.siteTransfers.decline('transfer-id', {
  reason: 'Organization does not accept external sites'
});
```

### Cancel Transfer (Source Account)

```typescript
await dashboardClient.siteTransfers.cancel('transfer-id');
```

### List Transfers

```typescript
// Incoming transfers
const incoming = await dashboardClient.siteTransfers.list({
  filter: { 
    target_account: { eq: 'my-account-id' },
    status: { eq: 'pending' }
  }
});

// Outgoing transfers
const outgoing = await dashboardClient.siteTransfers.list({
  filter: { 
    source_account: { eq: 'my-account-id' },
    status: { eq: 'pending' }
  }
});
```

### Transfer States

- `pending` - Awaiting acceptance
- `accepted` - Transfer accepted, processing
- `completed` - Transfer successful
- `declined` - Target declined transfer
- `cancelled` - Source cancelled transfer
- `expired` - 7 days passed without acceptance

## Job Results Resource

### Resource: `dashboardClient.jobResults`

**Type**: `job_result`  
**Purpose**: Monitor async operations like transfers

### Find Job Result

```typescript
const job = await dashboardClient.jobResults.find('job-id');

// Job structure:
{
  id: 'job-123',
  type: 'job_result',
  status: 202, // Processing
  progress: 45, // 0-100
  created_at: '2024-07-06T00:00:00Z',
  completed_at: null,
  
  // On completion
  status: 200, // Success
  payload: { /* operation-specific result */ },
  completed_at: '2024-07-06T00:05:00Z'
  
  // On error
  status: 422, // or 500
  error: {
    message: 'Transfer failed',
    details: ['Target organization at member limit']
  }
}
```

### Poll Job Until Completion

```typescript
async function pollJobCompletion(client: DashboardClient, jobId: string) {
  const maxAttempts = 60;
  const baseDelay = 1000; // 1 second
  
  for (let attempt = 0; attempt < maxAttempts; attempt++) {
    const job = await client.jobResults.find(jobId);
    
    // Check completion
    if (job.status !== 202) {
      if (job.status === 200) {
        return job.payload;
      } else {
        throw new Error(`Job failed: ${job.error?.message}`);
      }
    }
    
    // Show progress
    if (job.progress) {
      console.log(`Progress: ${job.progress}%`);
    }
    
    // Exponential backoff
    const delay = Math.min(baseDelay * Math.pow(1.5, attempt), 30000);
    await new Promise(resolve => setTimeout(resolve, delay));
  }
  
  throw new Error('Job timeout');
}
```

### Job Status Codes

- `200` - Success
- `202` - Processing
- `422` - Validation error
- `500` - Server error

## Common Workflows

### Complete Site Setup

```typescript
async function setupNewSite(orgId: string, projectName: string) {
  // 1. Create site via Dashboard API
  const site = await dashboardClient.sites.create({
    name: projectName,
    organization: { type: 'organization', id: orgId }
  });
  
  console.log('Site created:', site.frontend_url);
  
  // 2. Get API token for CMA access
  const apiToken = await getProjectApiToken(site.id);
  
  // 3. Initialize CMA client for configuration
  const projectClient = buildClient({ apiToken });
  
  // 4. Configure site settings
  await projectClient.site.update({
    locales: ['en', 'es', 'fr'],
    timezone: 'Europe/London',
    require_2fa: true,
    
    global_seo_settings: {
      site_name: projectName,
      fallback_seo: {
        title: `${projectName} - Welcome`,
        description: `Welcome to ${projectName}`
      }
    }
  });
  
  // 5. Create roles
  const editorRole = await projectClient.roles.create({
    name: 'Editor',
    can_edit_schema: false,
    can_manage_users: false,
    can_publish_to_production: true
  });
  
  // 6. Invite team members
  const teamMembers = [
    { email: 'editor@company.com', role: editorRole.id },
    { email: 'writer@company.com', role: editorRole.id }
  ];
  
  for (const member of teamMembers) {
    await projectClient.siteInvitations.create({
      email: member.email,
      role: { type: 'role', id: member.role }
    });
  }
  
  return site;
}
```

### Site Transfer Workflow

```typescript
async function transferSite(siteId: string, targetOrgId: string) {
  try {
    // 1. Check transfer eligibility
    const site = await dashboardClient.sites.find(siteId);
    if (site.owner.type !== 'account') {
      throw new Error('Can only transfer sites owned by accounts');
    }
    
    // 2. Initiate transfer
    console.log('Initiating transfer...');
    const job = await dashboardClient.siteTransfers.create({
      site_id: siteId,
      target_organization_id: targetOrgId,
      transfer_settings: {
        transfer_users: true,
        transfer_roles: true,
        keep_subscription: false
      }
    });
    
    // 3. Monitor job
    console.log('Processing transfer...');
    const transfer = await pollJobCompletion(dashboardClient, job.id);
    
    console.log('Transfer pending acceptance');
    console.log('Transfer token:', transfer.transfer_token);
    console.log('Expires:', transfer.expires_at);
    
    // 4. Wait for acceptance (in real app, this would be async)
    // Target organization must run:
    // await dashboardClient.siteTransfers.accept(transfer.id, {
    //   transfer_token: transfer.transfer_token
    // });
    
    return transfer;
    
  } catch (error) {
    console.error('Transfer failed:', error);
    throw error;
  }
}
```

### Team Management

```typescript
async function manageTeam(apiToken: string) {
  const client = buildClient({ apiToken });
  
  // 1. List current team
  const users = await client.users.list({
    include: ['role']
  });
  
  console.log(`Current team: ${users.length} members`);
  
  // 2. Check pending invitations
  const invitations = await client.siteInvitations.list({
    filter: { status: { eq: 'pending' } }
  });
  
  // 3. Resend expiring invitations
  const threeDaysFromNow = new Date();
  threeDaysFromNow.setDate(threeDaysFromNow.getDate() + 3);
  
  for (const invitation of invitations) {
    const expiresAt = new Date(invitation.expires_at);
    if (expiresAt < threeDaysFromNow) {
      await client.siteInvitations.resend(invitation.id);
      console.log(`Resent invitation to ${invitation.email}`);
    }
  }
  
  // 4. Clean up expired invitations
  const expired = invitations.filter(inv => 
    new Date(inv.expires_at) < new Date()
  );
  
  for (const invitation of expired) {
    await client.siteInvitations.destroy(invitation.id);
    console.log(`Removed expired invitation for ${invitation.email}`);
  }
  
  // 5. Bulk invite new members
  const newMembers = [
    'designer@company.com',
    'developer@company.com',
    'manager@company.com'
  ];
  
  const role = users[0].role.id; // Use existing role
  
  for (const email of newMembers) {
    try {
      await client.siteInvitations.create({
        email,
        role: { type: 'role', id: role }
      });
      console.log(`Invited ${email}`);
    } catch (error) {
      if (error.code === 'ALREADY_A_USER') {
        console.log(`${email} is already a member`);
      }
    }
  }
}
```

### Plan Usage Monitoring

```typescript
async function monitorPlanUsage(siteId: string) {
  // 1. Get current site and plan
  const site = await dashboardClient.sites.find(siteId, {
    include: ['plan', 'usage_counters']
  });
  
  const plan = site.plan;
  const usage = site.usage_counters;
  
  // 2. Calculate usage percentages
  const usageMetrics = {
    records: {
      used: usage.records,
      limit: plan.limits.items,
      percentage: (usage.records / plan.limits.items) * 100
    },
    bandwidth: {
      used: usage.bandwidth,
      limit: plan.limits.bandwidth,
      percentage: (usage.bandwidth / plan.limits.bandwidth) * 100
    },
    apiCalls: {
      used: usage.api_calls,
      limit: plan.limits.api_calls,
      percentage: (usage.api_calls / plan.limits.api_calls) * 100
    }
  };
  
  // 3. Check for overages
  const warnings = [];
  for (const [metric, data] of Object.entries(usageMetrics)) {
    if (data.percentage > 80) {
      warnings.push({
        metric,
        usage: `${data.percentage.toFixed(1)}%`,
        recommendation: data.percentage > 90 ? 'urgent' : 'warning'
      });
    }
  }
  
  // 4. Recommend plan upgrade if needed
  if (warnings.some(w => w.recommendation === 'urgent')) {
    const plans = await dashboardClient.sitePlans.list();
    const nextPlan = plans.find(p => 
      p.limits.items > plan.limits.items && 
      p.code !== 'enterprise'
    );
    
    if (nextPlan) {
      console.log(`Consider upgrading to ${nextPlan.name}`);
      console.log(`Cost: $${nextPlan.amount / 100}/month`);
    }
  }
  
  return { usageMetrics, warnings };
}
```

### Bulk Site Operations

```typescript
async function bulkSiteOperations(orgId: string) {
  // 1. Get all organization sites
  const sites = await dashboardClient.sites.list({
    filter: { organization: { eq: orgId } },
    per_page: 100
  });
  
  // 2. Analyze sites
  const analysis = {
    total: sites.length,
    byPlan: {},
    inactive: [],
    overLimit: []
  };
  
  for (const site of sites) {
    // Group by plan
    const planName = site.plan?.name || 'Unknown';
    analysis.byPlan[planName] = (analysis.byPlan[planName] || 0) + 1;
    
    // Check activity (via CMA)
    try {
      const apiToken = await getProjectApiToken(site.id);
      const client = buildClient({ apiToken });
      
      const recentItems = await client.items.list({
        filter: { updated_at: { gt: '2024-01-01' } },
        page: { limit: 1 }
      });
      
      if (recentItems.length === 0) {
        analysis.inactive.push(site);
      }
      
      // Check usage
      const usage = site.usage_counters;
      const limits = site.plan?.limits;
      
      if (limits && usage.records > limits.items * 0.9) {
        analysis.overLimit.push(site);
      }
    } catch (error) {
      console.error(`Failed to analyze ${site.name}:`, error);
    }
  }
  
  // 3. Generate report
  console.log('Site Analysis:', analysis);
  
  // 4. Bulk updates
  for (const site of analysis.inactive) {
    // Add "inactive" tag or notify
    console.log(`Inactive site: ${site.name}`);
  }
  
  return analysis;
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/cma-client';
import { ApiError as DashboardApiError } from '@datocms/dashboard-client';

async function handleSiteErrors() {
  try {
    await dashboardClient.sites.create({
      name: 'Test Site',
      organization: { type: 'organization', id: 'invalid' }
    });
  } catch (error) {
    if (error instanceof DashboardApiError) {
      switch (error.response?.status) {
        case 401:
          console.error('Invalid API token');
          break;
        case 403:
          console.error('No permission to create sites');
          break;
        case 404:
          console.error('Organization not found');
          break;
        case 422:
          console.error('Validation error:', error.errors);
          // E.g., organization at site limit
          break;
        case 429:
          console.error('Rate limit exceeded');
          break;
      }
    }
  }
}

// Handle invitation errors
async function handleInvitationErrors(client: CmaClient) {
  try {
    await client.siteInvitations.create({
      email: 'existing@user.com',
      role: { type: 'role', id: 'role-id' }
    });
  } catch (error) {
    if (error.code === 'ALREADY_A_USER') {
      console.log('User is already a member');
    } else if (error.code === 'INVITATION_ALREADY_SENT') {
      console.log('Invitation already pending');
    }
  }
}

// Handle transfer errors
async function handleTransferErrors() {
  try {
    const job = await dashboardClient.siteTransfers.create({
      site_id: 'site-id',
      target_organization_id: 'org-id'
    });
    
    const result = await pollJobCompletion(dashboardClient, job.id);
  } catch (error) {
    if (error.message.includes('organization at member limit')) {
      console.error('Target organization cannot accept more sites');
    } else if (error.message.includes('billing')) {
      console.error('Billing issue prevents transfer');
    }
  }
}
```

## Rate Limiting

Different endpoints have different rate limits:
- **Site operations**: 30 requests per 3 seconds
- **Invitation operations**: 60 requests per 3 seconds
- **Transfer operations**: 10 requests per 3 seconds
- **Job polling**: 120 requests per 3 seconds

## Best Practices

### 1. Site Configuration
```typescript
// Always set up essential settings immediately after creation
async function configureNewSite(apiToken: string) {
  const client = buildClient({ apiToken });
  
  await client.site.update({
    // Security first
    require_2fa: true,
    ip_whitelist: process.env.ALLOWED_IPS?.split(',') || [],
    
    // Localization
    locales: ['en'],
    timezone: 'UTC',
    
    // SEO defaults
    global_seo_settings: {
      site_name: 'My Site',
      fallback_seo: {
        title: 'Welcome',
        description: 'Default description'
      }
    }
  });
}
```

### 2. Invitation Management
```typescript
// Implement invitation lifecycle management
class InvitationManager {
  async sendBulkInvitations(emails: string[], roleId: string) {
    const results = { sent: [], failed: [] };
    
    for (const email of emails) {
      try {
        await this.client.siteInvitations.create({
          email,
          role: { type: 'role', id: roleId }
        });
        results.sent.push(email);
      } catch (error) {
        results.failed.push({ email, error: error.message });
      }
    }
    
    return results;
  }
  
  async cleanupInvitations() {
    const invitations = await this.client.siteInvitations.list();
    const now = new Date();
    
    for (const inv of invitations) {
      if (new Date(inv.expires_at) < now) {
        await this.client.siteInvitations.destroy(inv.id);
      }
    }
  }
}
```

### 3. Transfer Safety
```typescript
// Implement safe transfer workflow
async function safeTransfer(siteId: string, targetOrgId: string) {
  // Pre-transfer checklist
  const checks = {
    siteActive: false,
    targetReady: false,
    noActiveUsers: false,
    billingClear: false
  };
  
  // Verify site status
  const site = await dashboardClient.sites.find(siteId);
  checks.siteActive = site.usage_counters.api_calls > 0;
  
  // Verify target
  const org = await dashboardClient.organizations.find(targetOrgId);
  checks.targetReady = org.can_create_new_sites;
  
  // Check active users
  const apiToken = await getProjectApiToken(siteId);
  const client = buildClient({ apiToken });
  const sessions = await client.accessTokens.list();
  checks.noActiveUsers = sessions.length === 0;
  
  // Verify billing
  checks.billingClear = site.billing_status === 'active';
  
  // Only proceed if all checks pass
  if (Object.values(checks).every(v => v)) {
    return await transferSite(siteId, targetOrgId);
  } else {
    throw new Error('Pre-transfer checks failed: ' + 
      JSON.stringify(checks));
  }
}
```

### 4. Job Monitoring
```typescript
// Robust job monitoring with progress callback
async function monitorJob(
  client: DashboardClient,
  jobId: string,
  onProgress?: (progress: number) => void
) {
  const startTime = Date.now();
  const timeout = 300000; // 5 minutes
  let lastProgress = 0;
  
  while (true) {
    const job = await client.jobResults.find(jobId);
    
    // Handle progress updates
    if (job.progress && job.progress !== lastProgress) {
      lastProgress = job.progress;
      onProgress?.(job.progress);
    }
    
    // Check completion
    if (job.status !== 202) {
      if (job.status === 200) {
        return job.payload;
      }
      throw new Error(job.error?.message || 'Job failed');
    }
    
    // Check timeout
    if (Date.now() - startTime > timeout) {
      throw new Error('Job timeout');
    }
    
    // Adaptive delay
    const elapsed = Date.now() - startTime;
    const delay = elapsed < 10000 ? 1000 : 
                  elapsed < 60000 ? 2000 : 5000;
    
    await new Promise(resolve => setTimeout(resolve, delay));
  }
}
```

## Related Resources

- [Account Management](./merged-account-management.md)
- [Organization Management](./merged-organization-management.md)
- [Billing & Subscriptions](./merged-billing-subscriptions.md)
- [CMA Client Documentation](../01-cma-client/README.md)