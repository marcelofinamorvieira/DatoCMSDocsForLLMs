# Dashboard API Client

The Dashboard API client provides programmatic access to manage DatoCMS accounts, organizations, billing, and other account-level features.

## Installation

```bash
npm install @datocms/dashboard-client
```

## Client Initialization

```typescript
import { buildClient } from '@datocms/dashboard-client';

const client = buildClient({ 
  apiToken: 'YOUR_ACCOUNT_API_TOKEN'
});
```

## API Token Requirements

Dashboard API requires an **Account API token** (not a project API token). Create one from:
1. Go to [Account Settings](https://dashboard.datocms.com/account/tokens)
2. Click "Create new API token"
3. Give it a name and appropriate permissions

## Documentation Structure

This documentation has been organized into comprehensive merged files by functional area:

### 📁 [Account Management](./merged-account-management.md)
Complete documentation for account-related operations including:
- Account profile management
- Session management
- OAuth applications and authorizations
- Two-factor authentication (2FA)

### 📁 [Organization Management](./merged-organization-management.md)
Everything related to organization administration:
- Organizations (create, update, delete)
- Organization memberships and roles
- Organization invitations
- Payment mandates for organizations

### 📁 [Billing & Subscriptions](./merged-billing-subscriptions.md)
Comprehensive billing and subscription management:
- Billing profiles (per-site and per-owner pricing)
- Subscriptions and plans
- Invoices and payment processing
- Resource usage monitoring
- Subscription features and limits

### 📁 [Site Management](./merged-site-management.md)
Complete site/project management documentation:
- Site configuration (CMA and Dashboard APIs)
- Site invitations and team management
- Site plans and transfers
- Asynchronous job monitoring

## Common Patterns

### Creating a New Site

```typescript
const newSite = await client.sites.create({
  name: 'My New Project',
  organization: { type: 'organization', id: 'org-id' }
});
```

### Managing Organization Members

```typescript
// List organization members
const members = await client.organizationMemberships.list({
  filter: { organization: { eq: 'org-id' } }
});

// Update member role
await client.organizationMemberships.update('membership-id', {
  role: { type: 'organization_role', id: 'admin-role-id' }
});
```

### Monitoring Usage

```typescript
// Get resource usage for a site
const usage = await client.resourceUsages.find('site-id');
console.log('API calls this month:', usage.api_calls);
console.log('Bandwidth used:', usage.traffic_bytes);
```

## Error Handling

```typescript
import { ApiError } from '@datocms/dashboard-client';

try {
  await client.sites.create({ name: 'Test' });
} catch (error) {
  if (error instanceof ApiError) {
    console.error('API Error:', error.message);
    
    // Handle specific errors
    if (error.response?.status === 422) {
      console.error('Validation errors:', error.errors);
    }
  }
}
```

## Rate Limiting

Dashboard API has different rate limits than CMA:
- **Default**: 30 requests per 3 seconds
- **Burst**: Up to 60 requests allowed with proper spacing

The client handles rate limiting automatically with exponential backoff.

## Related Documentation

- [Authentication Setup](../00-quick-start/authentication.md)
- [CMA Client](../01-cma-client/README.md)
- [Account API Documentation](https://www.datocms.com/docs/account-api)