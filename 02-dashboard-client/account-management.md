# Dashboard API - Account Management (Merged Documentation)

This file contains merged documentation for all account-related resources in the Dashboard API:
- Account
- Session
- OAuth Applications
- Two-Factor Authentication

## Client Initialization

```typescript
import { buildClient } from '@datocms/dashboard-client';

const dashboardClient = buildClient({
  apiToken: 'YOUR_ACCOUNT_API_TOKEN', // Different from CMA token!
  environment: 'production' // Optional, defaults to production
});
```

## Account Resource

### Resource: `dashboardClient.account`

**JSON:API Type**: `account`  
**Endpoint**: `/account`

The account resource represents the authenticated user's account information.

### Get Current Account

Retrieves information about the authenticated account.

```typescript
const account = await dashboardClient.account.find();

// Returns:
{
  id: '123',
  type: 'account',
  email: 'user@example.com',
  full_name: 'John Doe',
  company: 'Acme Corp',
  country: 'US',
  timezone: 'America/New_York',
  is_2fa_active: true,
  is_sso_user: false,
  created_at: '2023-01-01T00:00:00Z',
  updated_at: '2024-01-01T00:00:00Z'
}
```

### Update Account

Updates the current account information.

```typescript
const updatedAccount = await dashboardClient.account.update({
  full_name: 'Jane Doe',
  company: 'New Company',
  country: 'US',
  timezone: 'America/Los_Angeles'
});

// Returns updated account object
```

**Parameters:**

```typescript
interface SimpleAccountUpdateSchema {
  full_name?: string;
  company?: string;
  country?: string;      // ISO 3166-1 alpha-2 code
  timezone?: string;     // IANA timezone identifier
}
```

### Change Email

Request an email change for the current account.

```typescript
await dashboardClient.account.requestEmailChange({
  email: 'newemail@example.com',
  password: 'current-password'
});

// User will receive a confirmation email to complete the change
```

### Change Password

Update the account password.

```typescript
await dashboardClient.account.changePassword({
  old_password: 'current-password',
  new_password: 'new-secure-password'
});
```

### Account Attributes

```typescript
interface Account {
  id: string;
  type: 'account';
  email: string;
  full_name: string;
  company: string | null;
  country: string | null;
  timezone: string;
  is_2fa_active: boolean;
  is_sso_user: boolean;
  created_at: string;
  updated_at: string;
  
  // Relationships
  active_sessions?: Session[];
  oauth_applications?: OauthApplication[];
  organizations?: Organization[];
  sites?: Site[];
}
```

## Session Resource

### Resource: `dashboardClient.sessions`

**JSON:API Type**: `session`  
**Purpose**: Manage active login sessions

### List Active Sessions

Get all active sessions for the current account.

```typescript
const sessions = await dashboardClient.sessions.list();

// Returns:
[
  {
    id: '789',
    type: 'session',
    ip_address: '192.168.1.1',
    user_agent: 'Mozilla/5.0...',
    created_at: '2024-01-01T00:00:00Z',
    last_active_at: '2024-01-02T00:00:00Z',
    device_name: 'Chrome on macOS',
    location: 'New York, US'
  }
]
```

### Destroy Session

Terminate a specific session (logout from a device).

```typescript
await dashboardClient.sessions.destroy('session-id');
```

### Destroy All Sessions

Terminate all sessions except the current one.

```typescript
await dashboardClient.sessions.destroyAll();
```

### Session Attributes

```typescript
interface Session {
  id: string;
  type: 'session';
  ip_address: string;
  user_agent: string;
  created_at: string;
  last_active_at: string;
  device_name: string;
  location: string | null;
  
  // Relationships
  account?: Account;
}
```

## OAuth Application Resource

### Resource: `dashboardClient.oauthApplications`

**JSON:API Type**: `oauth_application`  
**Purpose**: Manage OAuth applications for third-party integrations

### List OAuth Applications

Get all OAuth applications created by the account.

```typescript
const apps = await dashboardClient.oauthApplications.list();

// Returns:
[
  {
    id: '456',
    type: 'oauth_application',
    name: 'My Integration',
    description: 'Custom integration for workflow automation',
    client_id: 'abc123...',
    client_secret: 'secret123...',
    redirect_uris: ['https://myapp.com/callback'],
    created_at: '2024-01-01T00:00:00Z',
    updated_at: '2024-01-01T00:00:00Z'
  }
]
```

### Create OAuth Application

Create a new OAuth application.

```typescript
const app = await dashboardClient.oauthApplications.create({
  name: 'My Integration',
  description: 'Custom integration for workflow automation',
  redirect_uris: ['https://myapp.com/callback', 'https://myapp.com/callback2']
});

// Returns created application with client_id and client_secret
```

**Parameters:**

```typescript
interface SimpleOauthApplicationCreateSchema {
  name: string;
  description?: string;
  redirect_uris: string[];  // Must be HTTPS URLs
}
```

### Update OAuth Application

Update an existing OAuth application.

```typescript
const updated = await dashboardClient.oauthApplications.update('app-id', {
  name: 'Updated Integration',
  redirect_uris: ['https://myapp.com/new-callback']
});
```

### Regenerate Client Secret

Generate a new client secret for an OAuth application.

```typescript
const result = await dashboardClient.oauthApplications.regenerateSecret('app-id');

// Returns:
{
  client_secret: 'new-secret-123...'
}
```

### Delete OAuth Application

Delete an OAuth application.

```typescript
await dashboardClient.oauthApplications.destroy('app-id');
```

### OAuth Application Attributes

```typescript
interface OauthApplication {
  id: string;
  type: 'oauth_application';
  name: string;
  description: string | null;
  client_id: string;
  client_secret: string;        // Only visible on creation
  redirect_uris: string[];
  created_at: string;
  updated_at: string;
  
  // Relationships
  owner?: Account;
  authorizations?: OauthApplicationAuthorization[];
}
```

## OAuth Application Authorization Resource

### Resource: `dashboardClient.oauthApplicationAuthorizations`

**JSON:API Type**: `oauth_application_authorization`  
**Purpose**: Manage OAuth authorizations granted to applications

### List Authorizations

Get all OAuth authorizations for the current account.

```typescript
const authorizations = await dashboardClient.oauthApplicationAuthorizations.list();

// Returns:
[
  {
    id: '789',
    type: 'oauth_application_authorization',
    scopes: ['read:sites', 'write:sites'],
    authorized_at: '2024-01-01T00:00:00Z',
    last_used_at: '2024-01-02T00:00:00Z',
    oauth_application: {
      type: 'oauth_application',
      id: '456'
    }
  }
]
```

### Revoke Authorization

Revoke an OAuth authorization.

```typescript
await dashboardClient.oauthApplicationAuthorizations.destroy('authorization-id');
```

### Authorization Attributes

```typescript
interface OauthApplicationAuthorization {
  id: string;
  type: 'oauth_application_authorization';
  scopes: string[];
  authorized_at: string;
  last_used_at: string | null;
  
  // Relationships
  oauth_application?: OauthApplication;
  account?: Account;
}
```

## Two-Factor Authentication (TFA) Resource

### Resource: `dashboardClient.tfaDeactivateRequests`

**JSON:API Type**: `tfa_deactivate_request`  
**Purpose**: Handle two-factor authentication deactivation

### Request TFA Deactivation

Request to deactivate two-factor authentication (requires password).

```typescript
const request = await dashboardClient.tfaDeactivateRequests.create({
  password: 'current-password'
});

// Returns:
{
  id: '123',
  type: 'tfa_deactivate_request',
  status: 'pending',
  created_at: '2024-01-01T00:00:00Z'
}

// User will receive an email to confirm deactivation
```

### Complete TFA Deactivation

Complete the deactivation using the token from email.

```typescript
await dashboardClient.tfaDeactivateRequests.confirm('request-id', {
  token: 'token-from-email'
});
```

### TFA Deactivate Request Attributes

```typescript
interface TfaDeactivateRequest {
  id: string;
  type: 'tfa_deactivate_request';
  status: 'pending' | 'completed' | 'expired';
  created_at: string;
  expires_at: string;
  
  // Relationships
  account?: Account;
}
```

## Common Patterns

### Complete Account Setup Example

```typescript
import { buildClient } from '@datocms/dashboard-client';

async function setupAccount() {
  const client = buildClient({
    apiToken: 'YOUR_ACCOUNT_API_TOKEN'
  });

  try {
    // 1. Get current account info
    const account = await client.account.find();
    console.log('Current account:', account.email);

    // 2. Update account details
    const updated = await client.account.update({
      full_name: 'John Doe',
      company: 'Acme Corp',
      country: 'US',
      timezone: 'America/New_York'
    });

    // 3. Create OAuth application for integrations
    const app = await client.oauthApplications.create({
      name: 'Acme Integration',
      description: 'Internal workflow automation',
      redirect_uris: ['https://internal.acme.com/oauth/callback']
    });

    console.log('OAuth Client ID:', app.client_id);

    // 4. Check active sessions
    const sessions = await client.sessions.list();
    console.log(`Active sessions: ${sessions.length}`);

    // 5. Enable 2FA if not active
    if (!account.is_2fa_active) {
      console.log('Consider enabling 2FA for security');
    }

  } catch (error) {
    console.error('Setup failed:', error);
  }
}
```

### Security Best Practices

```typescript
// Monitor and manage sessions
async function securityAudit(client: DashboardClient) {
  // 1. Review active sessions
  const sessions = await client.sessions.list();
  
  // Identify suspicious sessions
  const suspicious = sessions.filter(session => {
    const lastActive = new Date(session.last_active_at);
    const daysSinceActive = (Date.now() - lastActive.getTime()) / (1000 * 60 * 60 * 24);
    
    return daysSinceActive > 30 || session.location?.includes('Unexpected Location');
  });

  // 2. Terminate suspicious sessions
  for (const session of suspicious) {
    await client.sessions.destroy(session.id);
    console.log(`Terminated session from ${session.location}`);
  }

  // 3. Review OAuth authorizations
  const authorizations = await client.oauthApplicationAuthorizations.list();
  
  // Check for unused authorizations
  const unused = authorizations.filter(auth => {
    if (!auth.last_used_at) return true;
    
    const lastUsed = new Date(auth.last_used_at);
    const daysSinceUsed = (Date.now() - lastUsed.getTime()) / (1000 * 60 * 60 * 24);
    
    return daysSinceUsed > 90;
  });

  // 4. Revoke unused authorizations
  for (const auth of unused) {
    await client.oauthApplicationAuthorizations.destroy(auth.id);
    console.log(`Revoked unused authorization: ${auth.id}`);
  }
}
```

### OAuth Application Management

```typescript
// Complete OAuth app lifecycle
async function manageOAuthApp(client: DashboardClient) {
  // 1. Create app
  const app = await client.oauthApplications.create({
    name: 'Production API Client',
    description: 'Main production integration',
    redirect_uris: [
      'https://app.example.com/oauth/callback',
      'https://staging.example.com/oauth/callback'
    ]
  });

  console.log('Client ID:', app.client_id);
  console.log('Client Secret:', app.client_secret); // Save this!

  // 2. Update app
  const updated = await client.oauthApplications.update(app.id, {
    name: 'Production API Client v2',
    redirect_uris: [
      'https://app.example.com/oauth/callback',
      'https://app.example.com/oauth/callback/v2'
    ]
  });

  // 3. Regenerate secret if compromised
  const newSecret = await client.oauthApplications.regenerateSecret(app.id);
  console.log('New secret:', newSecret.client_secret);

  // 4. Monitor usage
  const authorizations = await client.oauthApplicationAuthorizations.list({
    filter: { oauth_application: { eq: app.id } }
  });
  
  console.log(`Active authorizations: ${authorizations.length}`);

  // 5. Clean up when done
  // await client.oauthApplications.destroy(app.id);
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/dashboard-client';

async function handleAccountOperations(client: DashboardClient) {
  try {
    await client.account.update({
      email: 'invalid-email' // This will fail
    });
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.response?.status) {
        case 401:
          console.error('Invalid API token');
          break;
        case 422:
          console.error('Validation error:', error.errors);
          // error.errors might contain:
          // [{ detail: "Email is invalid", source: { pointer: "/data/attributes/email" } }]
          break;
        case 429:
          console.error('Rate limit exceeded');
          break;
        default:
          console.error('API error:', error.message);
      }
    }
  }
}
```

## Rate Limiting

Dashboard API has specific rate limits:
- **Account operations**: 30 requests per 3 seconds
- **OAuth operations**: 10 requests per 3 seconds
- **Session operations**: 20 requests per 3 seconds

The client automatically handles rate limiting with exponential backoff.

## Related Resources

- [Organization Management](./merged-organization-management.md)
- [Billing & Subscriptions](./merged-billing-subscriptions.md)
- [Site Management](./merged-site-management.md)
- [Authentication Setup](../00-quick-start/authentication.md)