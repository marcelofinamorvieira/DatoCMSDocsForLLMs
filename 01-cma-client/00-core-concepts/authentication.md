# Authentication

This guide covers how to authenticate with DatoCMS APIs using the Node.js clients.

## Overview

DatoCMS uses API tokens for authentication. You'll need different tokens depending on which API you're accessing:

- **CMA (Content Management API) Token**: For managing content, models, and site configuration
- **Dashboard API Token**: For account-level operations like billing and organization management

## Obtaining API Tokens

### CMA Token

1. Log into your DatoCMS project dashboard
2. Navigate to Settings → API Tokens
3. Create a new token with appropriate permissions:
   - **Read-only**: For fetching data only
   - **Read & Write**: For creating and updating content
   - **Full Access**: For schema and configuration changes

### Dashboard API Token

1. Log into your DatoCMS account (not project-specific)
2. Go to Account Settings → API Tokens
3. Create a token with necessary account permissions

## Setting Up Authentication

### Environment Variables

Store your tokens securely in environment variables:

```bash
# .env file
DATOCMS_API_TOKEN=your-cma-token-here
DATOCMS_DASHBOARD_TOKEN=your-dashboard-token-here
```

### CMA Client Authentication

```typescript
import { buildClient } from '@datocms/cma-client';

// Initialize the client with your token
const client = buildClient({
  apiToken: process.env.DATOCMS_API_TOKEN || ''
});

// Test the connection
async function testConnection() {
  try {
    const site = await client.site.find();
    console.log('Connected to site:', site.name);
  } catch (error) {
    console.error('Authentication failed:', error);
  }
}
```

### Dashboard Client Authentication

```typescript
import { buildClient } from '@datocms/dashboard-client';

// Initialize dashboard client
const dashboardClient = buildClient({
  apiToken: process.env.DATOCMS_DASHBOARD_TOKEN || ''
});

// Test dashboard access
async function testDashboardAccess() {
  try {
    const account = await dashboardClient.account.find();
    console.log('Authenticated as:', account.email);
  } catch (error) {
    console.error('Dashboard authentication failed:', error);
  }
}
```

## Authentication Best Practices

### 1. Token Security

- **Never commit tokens**: Always use environment variables
- **Use minimal permissions**: Grant only necessary access
- **Rotate tokens regularly**: Update tokens periodically
- **Monitor token usage**: Check audit logs for suspicious activity

### 2. Error Handling

```typescript
import { ApiError } from '@datocms/cma-client';

async function safeApiCall() {
  try {
    const items = await client.items.list();
    return items;
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.statusCode) {
        case 401:
          console.error('Invalid API token');
          break;
        case 403:
          console.error('Token lacks required permissions');
          break;
        case 429:
          console.error('Rate limit exceeded');
          break;
        default:
          console.error('API error:', error.message);
      }
    }
    throw error;
  }
}
```

### 3. Environment-Specific Tokens

Use different tokens for different environments:

```typescript
const getApiToken = () => {
  switch (process.env.NODE_ENV) {
    case 'production':
      return process.env.DATOCMS_PROD_TOKEN;
    case 'staging':
      return process.env.DATOCMS_STAGING_TOKEN;
    default:
      return process.env.DATOCMS_DEV_TOKEN;
  }
};

const client = buildClient({
  apiToken: getApiToken() || ''
});
```

## Token Permissions Reference

### CMA Token Permissions

| Permission | Description | Use Cases |
|------------|-------------|-----------|
| Read content | Fetch records and assets | Content delivery, exports |
| Create/edit content | Modify records and assets | Content management |
| Create/edit models | Change schema structure | Development, migrations |
| Create/edit schema menu items | Modify navigation | UI customization |
| Edit site settings | Change project configuration | Admin tasks |
| Manage webhooks | Configure webhooks | Integrations |
| Manage users | Invite/remove collaborators | Team management |
| Manage SSO settings | Configure single sign-on | Enterprise features |
| Create/edit roles | Define access permissions | Access control |
| Manage environments | Handle staging/production | Multi-environment setup |
| Manage build triggers | Configure deployments | CI/CD integration |
| Access site search | Use internal search | Content discovery |
| Access audit logs | View activity history | Security monitoring |

### Dashboard Token Permissions

| Permission | Description | Use Cases |
|------------|-------------|-----------|
| Read account data | View billing and usage | Monitoring |
| Manage billing | Update payment methods | Financial admin |
| Manage sites | Create/delete projects | Project management |
| Manage organizations | Admin org settings | Enterprise management |

## Common Authentication Patterns

### 1. Server-Side Authentication

For backend applications and scripts:

```typescript
// server.ts
import { buildClient } from '@datocms/cma-client';

const client = buildClient({
  apiToken: process.env.DATOCMS_API_TOKEN!,
  environment: process.env.DATOCMS_ENVIRONMENT || 'main'
});

export { client };
```

### 2. Client-Side Authentication

For browser applications (use read-only tokens):

```typescript
// Warning: Only use read-only tokens in browser code!
const publicClient = buildClient({
  apiToken: process.env.NEXT_PUBLIC_DATOCMS_READONLY_TOKEN!
});
```

### 3. Multi-Environment Setup

```typescript
class DatoCMSService {
  private clients: Map<string, any> = new Map();

  getClient(environment: string = 'main') {
    if (!this.clients.has(environment)) {
      this.clients.set(environment, buildClient({
        apiToken: process.env.DATOCMS_API_TOKEN!,
        environment
      }));
    }
    return this.clients.get(environment);
  }
}

const datoCMS = new DatoCMSService();
const mainClient = datoCMS.getClient('main');
const stagingClient = datoCMS.getClient('staging');
```

## Next Steps

- Learn about [making your first API calls](./README.md)
- Explore [content management operations](../01-cma-client/01-content-management/item.md)
- Set up [error handling](../04-advanced-topics/error-handling-patterns.md)
- Configure [webhooks for real-time updates](../01-cma-client/04-site-configuration/webhook.md)

## Related Resources

- [API Token Management](../01-cma-client/05-access-control/access-token.md)
- [Role-Based Access Control](../01-cma-client/05-access-control/role.md)
- [Security Best Practices](../05-reference/security-best-practices.md)
- [Environment Management](../01-cma-client/04-site-configuration/environment.md)