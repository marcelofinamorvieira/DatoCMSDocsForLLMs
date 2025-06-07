# Access Control Resources

This comprehensive guide covers all access control and authentication resources in the DatoCMS Content Management API. Access control in DatoCMS is managed through a combination of roles, users, SSO configurations, and access tokens.

## Table of Contents

- [Overview](#overview)
- [Access Tokens](#access-tokens)
- [Roles](#roles)
- [Users](#users)
- [SSO Users](#sso-users)
- [SSO Groups](#sso-groups)
- [SSO Settings](#sso-settings)
- [Sessions](#sessions)
- [Site Invitations](#site-invitations)
- [Editing Sessions](#editing-sessions)

## Overview

DatoCMS provides a comprehensive access control system that supports:

- **Role-Based Access Control (RBAC)**: Define permissions through roles
- **Single Sign-On (SSO)**: SAML/SCIM integration for enterprise authentication
- **API Access Tokens**: Programmatic access with granular permissions
- **User Management**: Both manual and SSO-based user provisioning
- **Session Management**: Track and control active user sessions
- **Collaborative Editing**: Monitor real-time editing sessions

### Authentication Methods

1. **Web App Authentication**: Username/password or SSO
2. **API Authentication**: Access tokens for programmatic access
3. **SSO Authentication**: SAML 2.0 for enterprise identity providers

### Permission Hierarchy

```
Organization
  └── Projects (Sites)
       └── Roles
            └── Users/SSO Users
                 └── Permissions
```

---

# Access Tokens

## Overview

The Access Tokens resource (`client.accessTokens`) manages API tokens for programmatic access to your DatoCMS project. These tokens control access to the Content Management API (CMA), Content Delivery API (CDA), and CMA Migrations API.

**Resource Type**: `access_token`  
**JSON:API Type**: `access_token`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// List all access tokens
const tokens = await client.accessTokens.list();

// Create a new access token
const token = await client.accessTokens.create({
  name: 'CI/CD Pipeline Token',
  can_access_cma: true,
  role: { type: 'role', id: 'editor-role-id' }
});
```

## Available Methods

### Create Access Token

Creates a new API token with specified permissions.

```typescript
// Create a full-access token
const fullAccessToken = await client.accessTokens.create({
  name: 'Admin API Token',
  can_access_cma: true,
  can_access_cda: true,
  can_access_cma_migrations: true,
  role: { type: 'role', id: 'admin-role-id' }
});

// Create a read-only token for content delivery
const cdaToken = await client.accessTokens.create({
  name: 'Frontend CDA Token',
  can_access_cma: false,
  can_access_cda: true,
  can_access_cda_preview: true,
  can_access_cma_migrations: false
});

// Create environment-specific token
const stagingToken = await client.accessTokens.create({
  name: 'Staging Environment Token',
  can_access_cma: true,
  role: { type: 'role', id: 'editor-role-id' },
  environments: ['staging', 'development']
});

// Create token with custom data
const customToken = await client.accessTokens.create({
  name: 'Integration Token',
  can_access_cma: true,
  role: { type: 'role', id: 'custom-role-id' },
  custom_data: {
    service: 'github-actions',
    purpose: 'deployment',
    created_by: 'devops-team'
  }
});
```

**Parameters:**

```typescript
interface SimpleAccessTokenCreateSchema {
  name: string;                              // Token name/description
  can_access_cda?: boolean;                  // Access to published content (default: true)
  can_access_cda_preview?: boolean;          // Access to draft content (default: true)
  can_access_cma?: boolean;                  // Access to management API (default: true)
  can_access_cma_migrations?: boolean;       // Enable Migrations API access
  role?: { type: 'role'; id: string } | null; // Associated role (required if can_access_cma is true)
  environments?: string[];                   // Limit to specific environments
  custom_data?: Record<string, any>;         // Custom metadata
}
```

> ⚠️ **Important**: The `token` field is only returned when creating or regenerating a token. Store it securely as it cannot be retrieved later.

### List Access Tokens

Retrieves all access tokens in the project.

```typescript
// Get all tokens
const tokens = await client.accessTokens.list();

// With filters and pagination
const filteredTokens = await client.accessTokens.list({
  filter: {
    query: 'production',
    ids: ['token1', 'token2']
  },
  page: {
    offset: 0,
    limit: 20
  }
});

// Using iterator for large datasets
for await (const token of client.accessTokens.listPagedIterator()) {
  console.log(token.name);
}

// Filter examples
const cmaTokens = tokens.filter(token => token.can_access_cma);
const cdaTokens = tokens.filter(token => token.can_access_cda && !token.can_access_cma);
const editorTokens = tokens.filter(token => token.role?.id === 'editor-role-id');
const prodTokens = tokens.filter(token => 
  token.environments?.includes('production')
);
```

### Find Access Token

Retrieves details of a specific access token.

```typescript
const token = await client.accessTokens.find('token-id');

console.log(`Name: ${token.name}`);
console.log(`Created: ${token.created_at}`);
console.log(`Last used: ${token.last_used_at || 'Never'}`);
console.log(`CMA access: ${token.can_access_cma}`);
```

### Update Access Token

Modifies access token settings.

```typescript
// Update token name and role
const updated = await client.accessTokens.update('token-id', {
  name: 'Updated Token Name',
  role: { type: 'role', id: 'viewer-role-id' }
});

// Update environment access
const envUpdate = await client.accessTokens.update('token-id', {
  environments: ['production', 'staging']
});

// Update custom data
const metaUpdate = await client.accessTokens.update('token-id', {
  custom_data: {
    last_rotated: new Date().toISOString(),
    rotation_schedule: 'monthly'
  }
});

// Remove role association
const noRoleToken = await client.accessTokens.update('token-id', {
  role: null
});
```

### Regenerate Access Token

Regenerates the token value while keeping the same token ID.

```typescript
// Regenerate token (get new token value)
const regenerated = await client.accessTokens.regenerateToken('token-id');

console.log(`New token value: ${regenerated.token}`);
// Save this token value - it won't be shown again!
```

> ⚠️ **Security Note**: The old token is immediately invalidated. Update your applications with the new token value.

### Destroy Access Token

Permanently deletes an access token.

```typescript
// Simple deletion
await client.accessTokens.destroy('token-id');

// With ownership transfer options
await client.accessTokens.destroy('token-id', {
  transfer_site_to_user_id: '5678',         // Transfer site ownership
  transfer_site_to_organization_id: '9012',  // Or to organization
  force_destroy: true                        // Force deletion even with dependencies
});
```

## Token Permissions

### API Access Control

Access tokens can be configured to access different APIs:

| Permission | Description | Use Case |
|------------|-------------|----------|
| `can_access_cda` | Content Delivery API (published) | Production websites |
| `can_access_cda_preview` | Content Delivery API (drafts) | Preview environments |
| `can_access_cma` | Content Management API | Content operations |
| `can_access_cma_migrations` | Migrations API | Schema migrations |

### Role-Based Permissions

Tokens can be associated with roles to inherit their permissions:

```typescript
// Create token with editor role
const editorToken = await client.accessTokens.create({
  name: 'Editor Token',
  can_access_cma: true,
  role: { type: 'role', id: 'editor-role-id' }
});

// The token inherits all permissions from the editor role
```

## Token Management Patterns

### Token Rotation

Implement automatic token rotation for security:

```typescript
class TokenRotationManager {
  static async rotateToken(
    tokenId: string,
    options: {
      notifyUrl?: string;
      updateConfig?: (token: string) => Promise<void>;
    } = {}
  ) {
    try {
      // Get current token info
      const currentToken = await client.accessTokens.find(tokenId);
      
      // Regenerate token
      const newToken = await client.accessTokens.regenerateToken(tokenId);
      
      // Update token in your system
      if (options.updateConfig) {
        await options.updateConfig(newToken.token);
      }
      
      // Update custom data with rotation info
      await client.accessTokens.update(tokenId, {
        custom_data: {
          ...currentToken.custom_data,
          last_rotated: new Date().toISOString(),
          rotation_count: (currentToken.custom_data?.rotation_count || 0) + 1
        }
      });
      
      // Notify webhook if configured
      if (options.notifyUrl) {
        await fetch(options.notifyUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            event: 'token_rotated',
            tokenId,
            tokenName: currentToken.name,
            timestamp: new Date().toISOString()
          })
        });
      }
      
      return {
        success: true,
        tokenId,
        newToken: newToken.token
      };
    } catch (error) {
      console.error('Token rotation failed:', error);
      throw error;
    }
  }
  
  static async scheduleRotation(tokenIds: string[], intervalDays = 30) {
    const results = [];
    
    for (const tokenId of tokenIds) {
      const token = await client.accessTokens.find(tokenId);
      const lastRotated = new Date(token.custom_data?.last_rotated || token.created_at);
      const daysSinceRotation = Math.floor(
        (Date.now() - lastRotated.getTime()) / (1000 * 60 * 60 * 24)
      );
      
      if (daysSinceRotation >= intervalDays) {
        const result = await this.rotateToken(tokenId);
        results.push(result);
      } else {
        results.push({
          tokenId,
          skipped: true,
          nextRotation: intervalDays - daysSinceRotation
        });
      }
    }
    
    return results;
  }
}

// Usage
const rotationResults = await TokenRotationManager.scheduleRotation(
  ['token1', 'token2'], 
  90 // Rotate every 90 days
);
```

### Environment-Specific Tokens

Create and manage tokens for different environments:

```typescript
async function createEnvironmentTokens() {
  const environments = ['development', 'staging', 'production'];
  const tokens = {};
  
  for (const env of environments) {
    // Different permissions per environment
    const roleId = env === 'production' ? 'viewer-role-id' : 'editor-role-id';
    
    const token = await client.accessTokens.create({
      name: `${env.toUpperCase()} Environment Token`,
      can_access_cma: true,
      can_access_cda: true,
      role: { type: 'role', id: roleId },
      environments: [env],
      custom_data: {
        environment: env,
        purpose: 'deployment',
        restricted: env === 'production'
      }
    });
    
    tokens[env] = {
      id: token.id,
      token: token.token,
      role: roleId
    };
  }
  
  return tokens;
}

// Production token (read-only, published content)
const productionToken = await client.accessTokens.create({
  name: 'Production Website',
  can_access_cda: true,
  can_access_cda_preview: false,
  can_access_cma: false
});

// Preview token (read-only, all content)
const previewToken = await client.accessTokens.create({
  name: 'Preview Environment',
  can_access_cda: true,
  can_access_cda_preview: true,
  can_access_cma: false
});
```

### Service-Specific Tokens

Create tokens for different services and integrations:

```typescript
class ServiceTokenManager {
  static async createServiceToken(service: {
    name: string;
    type: 'ci' | 'cms' | 'frontend' | 'api';
    permissions: {
      cma?: boolean;
      cda?: boolean;
      migrations?: boolean;
    };
    environments?: string[];
    metadata?: Record<string, any>;
  }) {
    // Determine role based on service type
    const roleMap = {
      ci: 'ci-role-id',
      cms: 'editor-role-id',
      frontend: 'viewer-role-id',
      api: 'api-role-id'
    };
    
    const token = await client.accessTokens.create({
      name: `${service.name} (${service.type})`,
      can_access_cma: service.permissions.cma ?? false,
      can_access_cda: service.permissions.cda ?? true,
      can_access_cma_migrations: service.permissions.migrations ?? false,
      role: service.permissions.cma ? 
        { type: 'role', id: roleMap[service.type] } : undefined,
      environments: service.environments,
      custom_data: {
        service_name: service.name,
        service_type: service.type,
        created_at: new Date().toISOString(),
        ...service.metadata
      }
    });
    
    return token;
  }
  
  static async createStandardTokens() {
    const services = [
      {
        name: 'GitHub Actions',
        type: 'ci' as const,
        permissions: { cma: true, migrations: true },
        environments: ['staging', 'production']
      },
      {
        name: 'Next.js Frontend',
        type: 'frontend' as const,
        permissions: { cda: true },
        metadata: { framework: 'nextjs', version: '14' }
      },
      {
        name: 'Content Import Service',
        type: 'api' as const,
        permissions: { cma: true },
        environments: ['staging']
      }
    ];
    
    const tokens = [];
    for (const service of services) {
      const token = await this.createServiceToken(service);
      tokens.push(token);
    }
    
    return tokens;
  }
}
```

### Token Auditing

Track token usage and audit access:

```typescript
async function auditTokenUsage() {
  const tokens = await client.accessTokens.list();
  const audit = {
    total: tokens.length,
    byType: {
      cmaOnly: 0,
      cdaOnly: 0,
      both: 0,
      migrations: 0
    },
    byRole: {},
    unused: [],
    stale: [],
    highPrivilege: []
  };
  
  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
  
  for (const token of tokens) {
    // Categorize by type
    if (token.can_access_cma && token.can_access_cda) {
      audit.byType.both++;
    } else if (token.can_access_cma) {
      audit.byType.cmaOnly++;
    } else if (token.can_access_cda) {
      audit.byType.cdaOnly++;
    }
    
    if (token.can_access_cma_migrations) {
      audit.byType.migrations++;
    }
    
    // Count by role
    if (token.role) {
      audit.byRole[token.role.id] = (audit.byRole[token.role.id] || 0) + 1;
    }
    
    // Find unused tokens
    if (!token.last_used_at) {
      audit.unused.push({
        id: token.id,
        name: token.name,
        created: token.created_at
      });
    }
    
    // Find stale tokens
    if (token.last_used_at && new Date(token.last_used_at) < thirtyDaysAgo) {
      audit.stale.push({
        id: token.id,
        name: token.name,
        lastUsed: token.last_used_at
      });
    }
    
    // Flag high-privilege tokens
    if (token.can_access_cma_migrations || 
        (!token.environments || token.environments.length === 0)) {
      audit.highPrivilege.push({
        id: token.id,
        name: token.name,
        risks: [
          token.can_access_cma_migrations && 'Can run migrations',
          !token.environments && 'Access to all environments'
        ].filter(Boolean)
      });
    }
  }
  
  return audit;
}

// Run audit
const audit = await auditTokenUsage();
console.log(`Total tokens: ${audit.total}`);
console.log(`Unused tokens: ${audit.unused.length}`);
console.log(`High-privilege tokens: ${audit.highPrivilege.length}`);
```

## Security Best Practices

### Secure Token Storage

```typescript
// Good: Environment variable
const client = buildClient({
  apiToken: process.env.DATOCMS_API_TOKEN
});

// Bad: Hardcoded token
const client = buildClient({
  apiToken: 'da123...' // Never do this!
});

// Secrets management service
class SecretsManager {
  static async storeToken(tokenInfo: {
    id: string;
    value: string;
    name: string;
    environment: string;
  }) {
    // Example: AWS Secrets Manager
    // await secretsClient.createSecret({
    //   Name: `datocms/${tokenInfo.environment}/${tokenInfo.name}`,
    //   SecretString: tokenInfo.value,
    //   Tags: [
    //     { Key: 'token-id', Value: tokenInfo.id },
    //     { Key: 'service', Value: 'datocms' }
    //   ]
    // });
    
    console.log(`Token stored securely for ${tokenInfo.environment}`);
  }
}
```

### Best Practices

1. **Token Storage**: Never commit tokens to version control
2. **Minimal Permissions**: Only grant necessary API access
3. **Regular Rotation**: Implement automatic token rotation (90 days recommended)
4. **Role Scoping**: Use roles to limit token permissions
5. **Environment Isolation**: Use environment-specific tokens
6. **Audit Trail**: Regularly review token configurations and usage
7. **Secure Storage**: Use environment variables or secrets management services

## Response Format

### Simplified Response

```typescript
interface AccessToken {
  id: string;
  type: 'access_token';
  name: string;
  token?: string;                          // Only present on create/regenerate
  can_access_cda: boolean;
  can_access_cda_preview: boolean;
  can_access_cma: boolean;
  can_access_cma_migrations: boolean;
  visible_via_admin: boolean;
  created_at: string;                      // ISO 8601
  updated_at: string;                      // ISO 8601
  last_used_at?: string | null;
  environments?: string[];
  custom_data?: Record<string, any>;
  hardcoded_type?: string | null;          // System-defined type
  role?: {
    type: 'role';
    id: string;
  } | null;
}

// Example access token object
{
  id: 'token-123',
  type: 'access_token',
  name: 'Production API Token',
  can_access_cma: true,
  can_access_cda: true,
  can_access_cma_migrations: false,
  visible_via_admin: true,
  created_at: '2024-01-15T10:30:00Z',
  last_used_at: '2024-01-20T15:45:00Z',
  environments: ['production'],
  custom_data: {
    service: 'frontend',
    team: 'web-dev'
  },
  role: {
    type: 'role',
    id: 'viewer-role-id'
  }
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.accessTokens.create({
    name: 'Test Token',
    can_access_cma: true
    // Missing required role when can_access_cma is true
  });
} catch (error) {
  if (error instanceof ApiError) {
    const roleError = error.findError('role');
    if (roleError?.code === 'VALIDATION_REQUIRED') {
      console.log('Role is required when CMA access is enabled');
    }
  }
}

// Handle token not found
try {
  await client.accessTokens.regenerateToken('non-existent-id');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 404) {
    console.log('Access token not found');
  }
}

// Handle permission errors
try {
  await client.accessTokens.create({
    name: 'Unauthorized Token',
    can_access_cma: true,
    role: { type: 'role', id: 'admin-role-id' }
  });
} catch (error) {
  if (error instanceof ApiError && error.response.status === 403) {
    console.log('Insufficient permissions to create access tokens');
  }
}

// Handle deletion with dependencies
try {
  await client.accessTokens.destroy('token-in-use');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 422) {
    // Token is in use, need to transfer ownership or force destroy
    console.log('Token has dependencies, use force_destroy or transfer ownership');
  }
}
```

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| 422 - VALIDATION_REQUIRED | Missing required field (e.g., role when CMA enabled) | Provide missing field |
| 422 - VALIDATION_INVALID | Invalid field value | Check field requirements |
| 404 - NOT_FOUND | Token or role doesn't exist | Verify IDs |
| 403 - FORBIDDEN | Insufficient permissions | Check user permissions |
| 422 - IN_USE | Token has dependencies | Use force_destroy or transfer ownership |

## Important Notes

- The actual token value is only visible when creating or regenerating
- Tokens are immediately active upon creation
- Regenerating a token immediately invalidates the old value
- Deleting tokens may require ownership transfer for dependent resources
- System tokens (`hardcoded_type`) may have special restrictions
- Rate limits apply per token, not per project
- Tokens with CMA access require a role association

---

# Roles

Roles define sets of permissions that can be assigned to users in your DatoCMS project. They control what users can see and do within the CMS interface and API.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a new role
const role = await client.roles.create({
  name: 'Content Editor',
  can_edit_schema: false,
  can_manage_users: false,
  can_publish_to_production: true
});
```

## API Reference

### Create Role

Create a new role with specific permissions.

```typescript
const editorRole = await client.roles.create({
  name: 'Blog Editor',
  // Site-wide permissions
  can_edit_favicon: false,
  can_edit_site: false,
  can_edit_schema: false,
  can_manage_menu: true,
  can_edit_environment: false,
  can_promote_environments: false,
  can_manage_users: false,
  can_manage_shared_filters: true,
  can_manage_upload_collections: true,
  can_manage_build_triggers: false,
  can_manage_webhooks: false,
  can_manage_environments: false,
  can_manage_sso: false,
  can_access_audit_log: false,
  can_manage_workflows: false,
  can_manage_access_tokens: false,
  can_perform_site_search: true,
  can_access_build_events_log: false,
  
  // Environment access
  environments_access: 'all', // or 'primary_only' or 'sandbox_only'
  
  // Item type permissions
  positive_item_type_permissions: [
    {
      item_type: { type: 'item_type', id: 'blog_post' },
      action: 'all'
    },
    {
      item_type: { type: 'item_type', id: 'author' },
      action: 'published' // Can only see published items
    }
  ],
  negative_item_type_permissions: [
    {
      item_type: { type: 'item_type', id: 'secret_page' },
      action: 'all' // No access to this model
    }
  ]
});
```

**Parameters:**
- `name` (string, required): Role name
- Site-wide boolean permissions (all optional, default: false)
- `environments_access` (string, optional): 'all', 'primary_only', or 'sandbox_only'
- `positive_item_type_permissions` (array, optional): Granted permissions
- `negative_item_type_permissions` (array, optional): Denied permissions
- `positive_upload_permissions` (array, optional): Upload access rules
- `negative_upload_permissions` (array, optional): Upload restrictions
- `positive_build_trigger_permissions` (array, optional): Build trigger access
- `negative_build_trigger_permissions` (array, optional): Build trigger restrictions
- `inherits_permissions_from` (object, optional): Base role to inherit from

**Returns:** The created role object

### Update Role

Modify an existing role's permissions.

```typescript
const updated = await client.roles.update('role-id', {
  name: 'Senior Editor',
  can_edit_schema: true,
  can_manage_workflows: true,
  
  // Add permission to new model
  positive_item_type_permissions: [
    {
      item_type: { type: 'item_type', id: 'blog_post' },
      action: 'all'
    },
    {
      item_type: { type: 'item_type', id: 'landing_page' },
      action: 'all'
    }
  ]
});
```

**Parameters:**
- `roleId` (string, required): The role ID
- Update parameters (same as create, all optional)

**Returns:** The updated role object

### List Roles

Get all roles in the project.

```typescript
const roles = await client.roles.list();

// Find admin roles
const adminRoles = roles.filter(role => 
  role.can_edit_schema && role.can_manage_users
);

// Find content-only roles
const contentRoles = roles.filter(role => 
  !role.can_edit_schema && !role.can_manage_users
);
```

**Returns:** Array of role objects

### Find Role

Retrieve a specific role by ID.

```typescript
const role = await client.roles.find('role-id');
```

**Parameters:**
- `roleId` (string, required): The role ID

**Returns:** The role object

### Delete Role

Remove a role. Users with this role will lose their permissions.

```typescript
await client.roles.destroy('role-id');
```

**Parameters:**
- `roleId` (string, required): The role ID

**Note:** You cannot delete a role that is currently assigned to users

### Duplicate Role

Create a copy of an existing role.

```typescript
const duplicate = await client.roles.duplicate('role-id');

// The duplicate will have:
// - Same permissions as the original
// - Name with " (copy)" suffix
// - New ID
```

**Parameters:**
- `roleId` (string, required): The role ID to duplicate

**Returns:** The new role object

## Permission Types

### Site-Wide Permissions

Control access to global features:

```typescript
{
  // Content management
  can_edit_favicon: false,        // Change site favicon
  can_edit_site: false,          // Edit site settings
  can_manage_menu: true,         // Organize navigation menu
  can_perform_site_search: true, // Use global search
  
  // Schema management
  can_edit_schema: false,        // Create/edit models and fields
  can_manage_workflows: false,   // Create/edit workflows
  
  // User management
  can_manage_users: false,       // Invite/remove users
  can_manage_sso: false,        // Configure SSO
  
  // Development features
  can_manage_webhooks: false,    // Create/edit webhooks
  can_manage_build_triggers: false, // Trigger builds
  can_manage_access_tokens: false,  // Create API tokens
  
  // Environment management
  can_edit_environment: false,      // Edit current environment
  can_manage_environments: false,   // Create/delete environments
  can_promote_environments: false,  // Promote to production
  
  // Asset management
  can_manage_upload_collections: true, // Organize uploads
  can_manage_shared_filters: true,     // Save search filters
  
  // Monitoring
  can_access_audit_log: false,        // View audit trail
  can_access_build_events_log: false  // View build logs
}
```

### Environment Access

Control which environments users can access:

```typescript
// Access all environments
{ environments_access: 'all' }

// Only production
{ environments_access: 'primary_only' }

// Only sandboxes (not production)
{ environments_access: 'sandbox_only' }
```

### Item Type Permissions

Fine-grained control over content access:

```typescript
// Positive permissions (grant access)
positive_item_type_permissions: [
  {
    item_type: { type: 'item_type', id: 'article' },
    action: 'all' // Full access
  },
  {
    item_type: { type: 'item_type', id: 'page' },
    action: 'published' // Read published only
  },
  {
    item_type: { type: 'item_type', id: 'product' },
    action: 'create' // Can create but not edit/delete
  }
]

// Negative permissions (deny access)
negative_item_type_permissions: [
  {
    item_type: { type: 'item_type', id: 'financial_report' },
    action: 'all' // No access at all
  }
]
```

**Available actions:**
- `all` - Full CRUD access
- `published` - Read published items only
- `create` - Create new items only
- `update` - Edit existing items only
- `delete` - Delete items only

### Upload Permissions

Control access to media library:

```typescript
// Grant upload permissions
positive_upload_permissions: [
  {
    upload_collection: { type: 'upload_collection', id: 'marketing' },
    action: 'all'
  }
]

// Restrict upload access
negative_upload_permissions: [
  {
    upload_collection: { type: 'upload_collection', id: 'sensitive' },
    action: 'all'
  }
]
```

### Build Trigger Permissions

Control who can trigger builds:

```typescript
positive_build_trigger_permissions: [
  {
    build_trigger: { type: 'build_trigger', id: 'staging-deploy' },
    action: 'trigger'
  }
]
```

## Advanced Usage

### Role Inheritance

Create roles that inherit permissions from a base role:

```typescript
// Create base role
const baseEditor = await client.roles.create({
  name: 'Base Editor',
  can_perform_site_search: true,
  can_manage_shared_filters: true
});

// Create specialized role inheriting from base
const blogEditor = await client.roles.create({
  name: 'Blog Editor',
  inherits_permissions_from: {
    type: 'role',
    id: baseEditor.id
  },
  // Additional permissions
  positive_item_type_permissions: [
    {
      item_type: { type: 'item_type', id: 'blog_post' },
      action: 'all'
    }
  ]
});
```

### Role Templates

Common role configurations:

```typescript
// Read-only viewer
const viewer = await client.roles.create({
  name: 'Viewer',
  can_perform_site_search: true,
  environments_access: 'primary_only',
  positive_item_type_permissions: [
    {
      item_type: { type: 'item_type', id: 'all' },
      action: 'published'
    }
  ]
});

// Content creator (no publishing)
const creator = await client.roles.create({
  name: 'Content Creator',
  can_perform_site_search: true,
  can_manage_shared_filters: true,
  can_manage_upload_collections: true,
  positive_item_type_permissions: [
    {
      item_type: { type: 'item_type', id: 'all' },
      action: 'create'
    },
    {
      item_type: { type: 'item_type', id: 'all' },
      action: 'update'
    }
  ]
});

// Publisher
const publisher = await client.roles.create({
  name: 'Publisher',
  can_perform_site_search: true,
  can_manage_build_triggers: true,
  positive_item_type_permissions: [
    {
      item_type: { type: 'item_type', id: 'all' },
      action: 'all'
    }
  ],
  positive_build_trigger_permissions: [
    {
      build_trigger: { type: 'build_trigger', id: 'production' },
      action: 'trigger'
    }
  ]
});
```

### Dynamic Permission Assignment

Programmatically build permissions based on models:

```typescript
// Get all blog-related models
const itemTypes = await client.itemTypes.list();
const blogModels = itemTypes.filter(it => 
  it.api_key.startsWith('blog_')
);

// Create role with access to all blog models
const blogAdmin = await client.roles.create({
  name: 'Blog Administrator',
  positive_item_type_permissions: blogModels.map(model => ({
    item_type: { type: 'item_type', id: model.id },
    action: 'all'
  }))
});
```

### Audit Role Usage

Check which users have specific roles:

```typescript
// Get all users
const users = await client.users.list();

// Find users with a specific role
const roleId = 'editor-role-id';
const editorsCount = users.filter(user => 
  user.role?.id === roleId
).length;

console.log(`${editorsCount} users have the Editor role`);
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.roles.create({
    name: 'Admin' // May already exist
  });
} catch (error) {
  if (error instanceof ApiError) {
    const nameError = error.findError('name');
    if (nameError?.code === 'VALIDATION_UNIQUE') {
      console.log('Role name must be unique');
    }
  }
}

// Handle deletion of assigned roles
try {
  await client.roles.destroy('role-id');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 422) {
    console.log('Cannot delete role: still assigned to users');
  }
}
```

## Role Object Structure

```typescript
interface Role {
  id: string;
  name: string;
  can_edit_favicon: boolean;
  can_edit_site: boolean;
  can_edit_schema: boolean;
  can_manage_menu: boolean;
  can_edit_environment: boolean;
  can_promote_environments: boolean;
  environments_access: 'all' | 'primary_only' | 'sandbox_only';
  can_manage_users: boolean;
  can_manage_shared_filters: boolean;
  can_manage_upload_collections: boolean;
  can_manage_build_triggers: boolean;
  can_manage_webhooks: boolean;
  can_manage_environments: boolean;
  can_manage_sso: boolean;
  can_access_audit_log: boolean;
  can_manage_workflows: boolean;
  can_manage_access_tokens: boolean;
  can_perform_site_search: boolean;
  can_access_build_events_log: boolean;
  positive_item_type_permissions: Array<{
    item_type: { type: 'item_type'; id: string };
    action: 'all' | 'published' | 'create' | 'update' | 'delete';
  }>;
  negative_item_type_permissions: Array<{
    item_type: { type: 'item_type'; id: string };
    action: 'all' | 'published' | 'create' | 'update' | 'delete';
  }>;
  positive_upload_permissions: Array<{
    upload_collection?: { type: 'upload_collection'; id: string };
    action: string;
  }>;
  negative_upload_permissions: Array<{
    upload_collection?: { type: 'upload_collection'; id: string };
    action: string;
  }>;
  positive_build_trigger_permissions: Array<{
    build_trigger: { type: 'build_trigger'; id: string };
    action: string;
  }>;
  negative_build_trigger_permissions: Array<{
    build_trigger: { type: 'build_trigger'; id: string };
    action: string;
  }>;
  inherits_permissions_from?: {
    type: 'role';
    id: string;
  };
}
```

---

# Users

The Users resource manages regular (non-SSO) users in your DatoCMS project. Users have direct login credentials and can be assigned roles to control their permissions.

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// List all users
const users = await client.users.list();

// Create a new user
const user = await client.users.create({
  email: 'user@example.com',
  first_name: 'John',
  last_name: 'Doe',
  role: { type: 'role', id: 'editor-role-id' }
});
```

## Available Operations

### List Users

```typescript
const users = await client.users.list();

// Filter by role
const editors = users.filter(user => user.role?.id === 'editor-role-id');

// Filter by state
const activeUsers = users.filter(user => user.state === 'active');
const pendingUsers = users.filter(user => user.state === 'pending');
```

### Find User

```typescript
const user = await client.users.find('user-id');

console.log(`Name: ${user.first_name} ${user.last_name}`);
console.log(`Email: ${user.email}`);
console.log(`Role: ${user.role?.id}`);
console.log(`State: ${user.state}`);
```

### Create User

```typescript
// Create a new user (sends invitation email)
const newUser = await client.users.create({
  email: 'newuser@example.com',
  first_name: 'Jane',
  last_name: 'Smith',
  role: { type: 'role', id: 'content-editor-id' },
  locale: 'en' // UI language preference
});

// User will receive an invitation email to set their password
```

### Update User

```typescript
// Update user details
const updated = await client.users.update('user-id', {
  first_name: 'Updated',
  last_name: 'Name',
  role: { type: 'role', id: 'admin-role-id' }
});

// Remove role (user becomes viewer)
const viewer = await client.users.update('user-id', {
  role: null
});
```

### Delete User

```typescript
// Simple deletion
await client.users.destroy('user-id');

// With resource transfer
await client.users.destroy('user-id', {
  transfer_to_user_id: 'another-user-id' // Transfer owned resources
});
```

### Reset Password

```typescript
// Send password reset email
await client.users.resetPassword('user-id');
// User receives email with reset link
```

## User States

Users can be in different states:

- **pending**: User invited but hasn't set password yet
- **active**: User has set password and can log in
- **disabled**: User account is suspended

## User Management Patterns

### Bulk User Operations

```typescript
// Invite multiple users
async function bulkInviteUsers(userList: Array<{
  email: string;
  name: string;
  role: string;
}>) {
  const results = [];
  
  for (const userData of userList) {
    try {
      const [firstName, ...lastNameParts] = userData.name.split(' ');
      const user = await client.users.create({
        email: userData.email,
        first_name: firstName,
        last_name: lastNameParts.join(' ') || '',
        role: { type: 'role', id: userData.role }
      });
      
      results.push({ success: true, user });
    } catch (error) {
      results.push({ 
        success: false, 
        email: userData.email, 
        error: error.message 
      });
    }
  }
  
  return results;
}

// Usage
const invitations = await bulkInviteUsers([
  { email: 'editor1@example.com', name: 'Editor One', role: 'editor-id' },
  { email: 'editor2@example.com', name: 'Editor Two', role: 'editor-id' }
]);
```

### User Activity Monitoring

```typescript
// Check inactive users
async function findInactiveUsers(daysInactive = 90) {
  const users = await client.users.list();
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - daysInactive);
  
  // Note: last_access is not directly available in user object
  // You would need to check audit logs for actual activity
  
  return users.filter(user => {
    // Check if user is pending (never logged in)
    return user.state === 'pending';
  });
}
```

### Role Migration

```typescript
// Migrate users from one role to another
async function migrateUsersToNewRole(
  oldRoleId: string, 
  newRoleId: string
) {
  const users = await client.users.list();
  const affectedUsers = users.filter(user => user.role?.id === oldRoleId);
  
  console.log(`Migrating ${affectedUsers.length} users`);
  
  const results = [];
  for (const user of affectedUsers) {
    try {
      const updated = await client.users.update(user.id, {
        role: { type: 'role', id: newRoleId }
      });
      results.push({ userId: user.id, success: true });
    } catch (error) {
      results.push({ userId: user.id, success: false, error });
    }
  }
  
  return results;
}
```

## User Object Structure

```typescript
interface User {
  id: string;
  type: 'user';
  email: string;
  first_name: string;
  last_name: string;
  locale: string;              // UI language preference
  state: 'pending' | 'active' | 'disabled';
  is_system: boolean;          // System-created user
  created_at: string;          // ISO 8601
  updated_at: string;          // ISO 8601
  role?: {
    type: 'role';
    id: string;
  } | null;
}
```

## Error Handling

```typescript
try {
  await client.users.create({
    email: 'duplicate@example.com',
    first_name: 'Test',
    last_name: 'User',
    role: { type: 'role', id: 'role-id' }
  });
} catch (error) {
  if (error instanceof ApiError) {
    const emailError = error.findError('email');
    if (emailError?.code === 'VALIDATION_UNIQUE') {
      console.log('User with this email already exists');
    }
  }
}
```

---

# SSO Users

The SsoUser resource represents users authenticated through Single Sign-On (SSO) via your Identity Provider. These users are managed externally through SAML/SCIM and have read-only profiles in DatoCMS, with roles determined by their group memberships.

## Available Operations

### List SSO Users

```javascript
const users = await client.ssoUsers.list();
```

**Returns:** Array of all SSO users

### Find SSO User

```javascript
const user = await client.ssoUsers.find(userId);
```

**Parameters:**
- `userId` (string, required): The SSO user ID

**Returns:** Single SsoUser object

### Copy Existing Users

```javascript
await client.ssoUsers.copyUsers();
```

**Purpose:** Convert existing DatoCMS editors to SSO users, useful during SSO migration

### Delete SSO User

```javascript
await client.ssoUsers.destroy(userId, {
  destination_user_type: "user",
  destination_user_id: "new_owner_id"
});
```

**Parameters:**
- `userId` (string, required): The SSO user ID to delete
- `queryParams` (object, optional):
  - `destination_user_type` (string): Type of user to transfer resources to
  - `destination_user_id` (string): ID of user to receive transferred resources

**Returns:** Empty response on success

## The SsoUser Object

```typescript
{
  id: string;                    // Unique user ID
  type: 'sso_user';             // Always 'sso_user'
  username: string;             // Email address
  external_id: string;          // Identity provider user ID
  is_active: boolean;           // Whether user can login
  first_name?: string;          // User's first name
  last_name?: string;           // User's last name
  groups: Array<{               // SSO group memberships
    id: string;
    type: 'sso_group';
  }>;
  role: {                       // Effective role (from highest priority group)
    id: string;
    type: 'role';
  };
  meta: {
    last_access?: string;       // Last login timestamp (ISO 8601)
  }
}
```

## Common Use Cases

### 1. SSO User Directory

```javascript
// Build a comprehensive user directory with group memberships
async function buildUserDirectory(client) {
  const users = await client.ssoUsers.list();
  const groups = await client.ssoGroups.list();
  const roles = await client.roles.list();
  
  // Create lookup maps
  const groupMap = new Map(groups.map(g => [g.id, g]));
  const roleMap = new Map(roles.map(r => [r.id, r]));
  
  const directory = users.map(user => {
    const userGroups = user.groups.map(g => groupMap.get(g.id));
    const effectiveRole = roleMap.get(user.role.id);
    
    return {
      name: `${user.first_name || ''} ${user.last_name || ''}`.trim() || user.username,
      email: user.username,
      status: user.is_active ? 'Active' : 'Deactivated',
      role: effectiveRole?.name || 'Unknown',
      groups: userGroups.map(g => g?.name || 'Unknown').join(', '),
      lastAccess: user.meta.last_access 
        ? new Date(user.meta.last_access).toLocaleDateString()
        : 'Never',
      externalId: user.external_id
    };
  });
  
  // Sort by last access (most recent first)
  directory.sort((a, b) => {
    const dateA = a.lastAccess === 'Never' ? 0 : new Date(a.lastAccess).getTime();
    const dateB = b.lastAccess === 'Never' ? 0 : new Date(b.lastAccess).getTime();
    return dateB - dateA;
  });
  
  return directory;
}

// Display the directory
const directory = await buildUserDirectory(client);
console.table(directory);
```

### 2. User Activity Monitoring

```javascript
// Monitor SSO user activity and identify inactive users
async function analyzeUserActivity(client, inactiveDays = 30) {
  const users = await client.ssoUsers.list();
  const now = new Date();
  const inactiveThreshold = now.getTime() - (inactiveDays * 24 * 60 * 60 * 1000);
  
  const analysis = {
    totalUsers: users.length,
    activeUsers: 0,
    inactiveUsers: 0,
    neverLoggedIn: 0,
    deactivatedUsers: 0,
    inactiveList: [],
    activityByMonth: {}
  };
  
  users.forEach(user => {
    // Check if deactivated
    if (!user.is_active) {
      analysis.deactivatedUsers++;
      return;
    }
    
    // Check last access
    if (!user.meta.last_access) {
      analysis.neverLoggedIn++;
      analysis.inactiveList.push({
        email: user.username,
        name: `${user.first_name || ''} ${user.last_name || ''}`.trim(),
        status: 'Never logged in'
      });
    } else {
      const lastAccess = new Date(user.meta.last_access);
      const lastAccessTime = lastAccess.getTime();
      
      if (lastAccessTime < inactiveThreshold) {
        analysis.inactiveUsers++;
        const daysSinceAccess = Math.floor((now - lastAccess) / (24 * 60 * 60 * 1000));
        
        analysis.inactiveList.push({
          email: user.username,
          name: `${user.first_name || ''} ${user.last_name || ''}`.trim(),
          status: `Inactive ${daysSinceAccess} days`,
          lastAccess: lastAccess.toLocaleDateString()
        });
      } else {
        analysis.activeUsers++;
      }
      
      // Track activity by month
      const monthKey = lastAccess.toISOString().substring(0, 7);
      analysis.activityByMonth[monthKey] = (analysis.activityByMonth[monthKey] || 0) + 1;
    }
  });
  
  // Sort inactive users by last access
  analysis.inactiveList.sort((a, b) => {
    if (a.status === 'Never logged in') return -1;
    if (b.status === 'Never logged in') return 1;
    return new Date(b.lastAccess) - new Date(a.lastAccess);
  });
  
  return analysis;
}

// Generate activity report
const activity = await analyzeUserActivity(client, 60);
console.log(`Total SSO Users: ${activity.totalUsers}`);
console.log(`Active (last 60 days): ${activity.activeUsers}`);
console.log(`Inactive: ${activity.inactiveUsers}`);
console.log(`Never logged in: ${activity.neverLoggedIn}`);
console.log(`Deactivated: ${activity.deactivatedUsers}`);
```

### 3. SSO Migration Helper

```javascript
// Migrate existing users to SSO during initial setup
async function migrateToSSO(client) {
  console.log('Starting SSO migration...');
  
  // Get current non-SSO users
  const regularUsers = await client.users.list();
  const ssoUsers = await client.ssoUsers.list();
  
  // Create a set of existing SSO user emails
  const ssoEmails = new Set(ssoUsers.map(u => u.username.toLowerCase()));
  
  // Find users that need migration
  const usersToMigrate = regularUsers.filter(user => 
    !ssoEmails.has(user.email.toLowerCase()) && 
    user.state === 'active'
  );
  
  console.log(`Found ${usersToMigrate.length} users to migrate`);
  
  if (usersToMigrate.length > 0) {
    // Copy users to SSO
    await client.ssoUsers.copyUsers();
    
    console.log('Users copied to SSO. Next steps:');
    console.log('1. Configure these users in your Identity Provider');
    console.log('2. Assign them to appropriate groups');
    console.log('3. Notify users about the SSO transition');
    
    // List migrated users
    console.log('\nMigrated users:');
    usersToMigrate.forEach(user => {
      console.log(`- ${user.email} (${user.first_name} ${user.last_name})`);
    });
  }
  
  return {
    migrated: usersToMigrate.length,
    alreadySSO: ssoUsers.length,
    users: usersToMigrate.map(u => ({
      email: u.email,
      name: `${u.first_name} ${u.last_name}`.trim()
    }))
  };
}
```

### 4. User Deprovisioning Workflow

```javascript
// Safely deprovision SSO users with resource transfer
async function deprovisionUser(client, userEmail, transferToEmail) {
  try {
    // Find the user to deprovision
    const users = await client.ssoUsers.list();
    const userToRemove = users.find(u => 
      u.username.toLowerCase() === userEmail.toLowerCase()
    );
    
    if (!userToRemove) {
      throw new Error(`SSO user ${userEmail} not found`);
    }
    
    // Find the destination user
    const destinationUser = users.find(u => 
      u.username.toLowerCase() === transferToEmail.toLowerCase()
    );
    
    if (!destinationUser) {
      throw new Error(`Destination user ${transferToEmail} not found`);
    }
    
    // Check what resources the user owns
    console.log(`Checking resources owned by ${userEmail}...`);
    
    // Note: You would need to check items, uploads, etc. that might be owned
    // This is a simplified example
    
    // Transfer ownership and delete user
    console.log(`Transferring resources to ${transferToEmail}...`);
    
    await client.ssoUsers.destroy(userToRemove.id, {
      destination_user_type: 'sso_user',
      destination_user_id: destinationUser.id
    });
    
    console.log(`✅ Successfully deprovisioned ${userEmail}`);
    console.log(`   Resources transferred to ${transferToEmail}`);
    
    return {
      success: true,
      removedUser: userEmail,
      transferredTo: transferToEmail
    };
    
  } catch (error) {
    console.error(`❌ Failed to deprovision user: ${error.message}`);
    throw error;
  }
}
```

### 5. Group Membership Analysis

```javascript
// Analyze user distribution across groups
async function analyzeGroupMembership(client) {
  const users = await client.ssoUsers.list();
  const groups = await client.ssoGroups.list();
  
  const analysis = {
    usersWithoutGroups: [],
    usersInMultipleGroups: [],
    groupSizes: {},
    roleDistribution: {}
  };
  
  // Create group lookup
  const groupMap = new Map(groups.map(g => [g.id, g]));
  
  users.forEach(user => {
    const userGroups = user.groups.map(g => groupMap.get(g.id));
    
    // Track users without groups
    if (userGroups.length === 0) {
      analysis.usersWithoutGroups.push({
        email: user.username,
        role: user.role.id // They still have a role from default_role
      });
    }
    
    // Track users in multiple groups
    if (userGroups.length > 1) {
      analysis.usersInMultipleGroups.push({
        email: user.username,
        groups: userGroups.map(g => ({
          name: g?.name,
          priority: g?.priority
        })).sort((a, b) => (b.priority || 0) - (a.priority || 0)),
        effectiveGroup: userGroups[0]?.name // Highest priority
      });
    }
    
    // Count group sizes
    userGroups.forEach(group => {
      if (group) {
        analysis.groupSizes[group.name] = (analysis.groupSizes[group.name] || 0) + 1;
      }
    });
    
    // Track role distribution
    const roleId = user.role.id;
    analysis.roleDistribution[roleId] = (analysis.roleDistribution[roleId] || 0) + 1;
  });
  
  return analysis;
}
```

### 6. SSO User Search

```javascript
// Search and filter SSO users
class SSOUserSearch {
  constructor(client) {
    this.client = client;
    this.users = null;
    this.groups = null;
    this.roles = null;
  }
  
  async initialize() {
    this.users = await this.client.ssoUsers.list();
    this.groups = await this.client.ssoGroups.list();
    this.roles = await this.client.roles.list();
  }
  
  search(criteria) {
    let results = [...this.users];
    
    // Filter by email/name
    if (criteria.query) {
      const query = criteria.query.toLowerCase();
      results = results.filter(user => 
        user.username.toLowerCase().includes(query) ||
        user.first_name?.toLowerCase().includes(query) ||
        user.last_name?.toLowerCase().includes(query)
      );
    }
    
    // Filter by active status
    if (criteria.isActive !== undefined) {
      results = results.filter(user => user.is_active === criteria.isActive);
    }
    
    // Filter by group
    if (criteria.groupName) {
      const group = this.groups.find(g => g.name === criteria.groupName);
      if (group) {
        results = results.filter(user => 
          user.groups.some(g => g.id === group.id)
        );
      }
    }
    
    // Filter by role
    if (criteria.roleName) {
      const role = this.roles.find(r => r.name === criteria.roleName);
      if (role) {
        results = results.filter(user => user.role.id === role.id);
      }
    }
    
    // Filter by last access
    if (criteria.lastAccessDays) {
      const threshold = Date.now() - (criteria.lastAccessDays * 24 * 60 * 60 * 1000);
      results = results.filter(user => {
        if (!user.meta.last_access) return criteria.includeNeverAccessed || false;
        return new Date(user.meta.last_access).getTime() > threshold;
      });
    }
    
    return results;
  }
}

// Usage
const search = new SSOUserSearch(client);
await search.initialize();

// Find inactive editors
const inactiveEditors = search.search({
  roleName: 'editor',
  isActive: false
});

// Find users who haven't logged in for 90 days
const inactiveUsers = search.search({
  lastAccessDays: 90,
  includeNeverAccessed: true
});
```

## Understanding SSO User Management

### User Lifecycle

1. **Provisioning**: Users are created via SCIM or during SAML authentication
2. **Group Assignment**: IdP assigns users to groups
3. **Role Calculation**: DatoCMS determines role from highest priority group
4. **Access**: Users authenticate via IdP and access DatoCMS
5. **Deprovisioning**: Users are removed via SCIM or manual deletion

### Role Determination

Users get their role from:
1. **Group Membership**: If in groups, highest priority group's role applies
2. **Default Role**: If not in any group, the SSO settings default role applies

## Important Notes

1. **Read-Only Profiles**: SSO user details (name, email) cannot be edited in DatoCMS
2. **External Management**: All user management happens in your Identity Provider
3. **Resource Ownership**: When deleting users, owned resources must be transferred
4. **No Password**: SSO users don't have DatoCMS passwords - they authenticate via IdP
5. **SCIM Sync**: User provisioning/deprovisioning typically happens via SCIM

## Object Structure

SSO User objects have this structure:

```typescript
interface SsoUser {
  id: string;
  type: 'sso_user';
  username: string;                    // Email address from IdP
  external_id: string;                 // Unique ID from Identity Provider
  is_active: boolean;                  // Whether user is active on IdP
  first_name: string | null;           // User's first name
  last_name: string | null;            // User's last name
  groups: SsoGroup[];                  // Associated SSO groups
  role: Role | null;                   // Assigned role (if any)
  meta: {
    created_at: string;                // ISO timestamp
    updated_at: string;                // ISO timestamp
    last_access_at: string | null;     // Last login timestamp
  };
}
```

**Example SSO User:**
```javascript
const ssoUser = {
  id: 'sso-user-123',
  type: 'sso_user',
  username: 'john.doe@company.com',
  external_id: 'idp-user-456',
  is_active: true,
  first_name: 'John',
  last_name: 'Doe',
  groups: [
    { id: 'group-1', type: 'sso_group', name: 'Developers' },
    { id: 'group-2', type: 'sso_group', name: 'Content Editors' }
  ],
  role: { id: 'role-1', type: 'role', name: 'Developer' },
  meta: {
    created_at: '2024-01-15T10:00:00.000Z',
    updated_at: '2024-01-20T14:30:00.000Z',
    last_access_at: '2024-01-20T09:15:00.000Z'
  }
};
```

## Error Handling

```javascript
try {
  await client.ssoUsers.destroy('user-id', {
    destination_user_type: 'invalid_type',
    destination_user_id: 'user-id'
  });
} catch (error) {
  if (error.message.includes('VALIDATION_ERROR')) {
    console.error('Invalid destination user type');
  } else if (error.message.includes('NOT_FOUND')) {
    console.error('User not found');
  }
}
```

---

# SSO Groups

The SsoGroup resource manages groups synchronized from your Identity Provider (IdP) for Single Sign-On authentication. Groups allow you to map collections of users from your IdP to specific DatoCMS roles, enabling efficient role-based access control for SSO users.

## Available Operations

### List SSO Groups

```javascript
const groups = await client.ssoGroups.list();
```

**Returns:** Array of all SSO groups synchronized from the IdP

### Update SSO Group

```javascript
const updated = await client.ssoGroups.update(groupId, {
  priority: 10,
  role: { type: "role", id: "editor_role_id" }
});
```

**Parameters:**
- `groupId` (string, required): The SSO group ID
- `body` (object, required):
  - `priority` (number, optional): Group priority (higher takes precedence)
  - `role` (object, optional): DatoCMS role assignment

**Returns:** Updated SsoGroup object

### Copy Roles

```javascript
await client.ssoGroups.copyRoles(groupId);
```

**Parameters:**
- `groupId` (string, required): The SSO group ID

**Purpose:** Synchronize SSO provider groups to DatoCMS roles

### Delete SSO Group

```javascript
await client.ssoGroups.destroy(groupId);
```

**Parameters:**
- `groupId` (string, required): The SSO group ID

**Returns:** Empty response on success

## The SsoGroup Object

```typescript
{
  id: string;                // Unique group ID
  type: 'sso_group';        // Always 'sso_group'
  name: string;             // Group name from IdP
  priority: number;         // Priority level (0-100+)
  role: {                   // Associated DatoCMS role
    id: string;
    type: 'role';
  };
  users: Array<{            // Group members
    id: string;
    type: 'sso_user';
  }>;
}
```

## Understanding Group Priority

When a user belongs to multiple groups, the group with the highest priority determines their effective role:

- Higher numbers = higher priority
- Default priority is often 0
- Range typically 0-100, but can be higher
- Users get the role from their highest priority group

## Common Use Cases

### 1. Map IdP Groups to DatoCMS Roles

```javascript
// Map organizational groups to appropriate roles
async function setupGroupMappings(client) {
  const groups = await client.ssoGroups.list();
  const roles = await client.roles.list();
  
  // Create role lookup
  const roleMap = new Map(roles.map(r => [r.name.toLowerCase(), r.id]));
  
  const mappings = [
    { groupName: 'Marketing Team', roleName: 'editor', priority: 20 },
    { groupName: 'Development Team', roleName: 'admin', priority: 30 },
    { groupName: 'External Contractors', roleName: 'viewer', priority: 10 },
    { groupName: 'C-Suite', roleName: 'admin', priority: 50 }
  ];
  
  for (const mapping of mappings) {
    const group = groups.find(g => g.name === mapping.groupName);
    const roleId = roleMap.get(mapping.roleName.toLowerCase());
    
    if (group && roleId) {
      await client.ssoGroups.update(group.id, {
        role: { type: 'role', id: roleId },
        priority: mapping.priority
      });
      
      console.log(`✅ Mapped ${mapping.groupName} to ${mapping.roleName} role`);
    } else {
      console.warn(`⚠️ Could not map ${mapping.groupName} - group or role not found`);
    }
  }
}
```

### 2. Priority-Based Access Control

```javascript
// Implement hierarchical access control
async function setupHierarchicalAccess(client) {
  const groups = await client.ssoGroups.list();
  
  // Define hierarchy (higher priority = more access)
  const hierarchy = {
    'All Employees': { priority: 10, role: 'viewer' },
    'Content Team': { priority: 20, role: 'editor' },
    'Department Heads': { priority: 30, role: 'editor_with_publish' },
    'IT Administrators': { priority: 40, role: 'admin' },
    'Executive Team': { priority: 50, role: 'admin' }
  };
  
  for (const [groupName, config] of Object.entries(hierarchy)) {
    const group = groups.find(g => g.name === groupName);
    
    if (group) {
      // Find role by name
      const roles = await client.roles.list();
      const role = roles.find(r => r.name === config.role);
      
      if (role) {
        await client.ssoGroups.update(group.id, {
          priority: config.priority,
          role: { type: 'role', id: role.id }
        });
        
        console.log(`Set ${groupName}: priority=${config.priority}, role=${config.role}`);
      }
    }
  }
}
```

### 3. Group Analytics Dashboard

```javascript
// Analyze SSO group composition and access levels
async function analyzeGroupAccess(client) {
  const groups = await client.ssoGroups.list();
  const roles = await client.roles.list();
  
  // Create role lookup
  const roleMap = new Map(roles.map(r => [r.id, r]));
  
  const analysis = {
    totalGroups: groups.length,
    totalUsers: new Set(),
    groupDetails: [],
    roleDistribution: {},
    priorityConflicts: []
  };
  
  // Analyze each group
  for (const group of groups) {
    const role = roleMap.get(group.role.id);
    const userCount = group.users.length;
    
    // Track unique users
    group.users.forEach(u => analysis.totalUsers.add(u.id));
    
    // Track role distribution
    const roleName = role?.name || 'Unknown';
    analysis.roleDistribution[roleName] = 
      (analysis.roleDistribution[roleName] || 0) + userCount;
    
    analysis.groupDetails.push({
      name: group.name,
      priority: group.priority,
      role: roleName,
      userCount: userCount,
      permissions: role?.can_access_site ? 'Full Access' : 'Limited Access'
    });
  }
  
  // Sort by priority to show hierarchy
  analysis.groupDetails.sort((a, b) => b.priority - a.priority);
  
  // Find users in multiple groups (potential conflicts)
  const userGroupMap = new Map();
  groups.forEach(group => {
    group.users.forEach(user => {
      if (!userGroupMap.has(user.id)) {
        userGroupMap.set(user.id, []);
      }
      userGroupMap.get(user.id).push({
        groupName: group.name,
        priority: group.priority,
        role: roleMap.get(group.role.id)?.name
      });
    });
  });
  
  // Identify priority conflicts
  userGroupMap.forEach((userGroups, userId) => {
    if (userGroups.length > 1) {
      userGroups.sort((a, b) => b.priority - a.priority);
      analysis.priorityConflicts.push({
        userId,
        effectiveGroup: userGroups[0].groupName,
        effectiveRole: userGroups[0].role,
        allGroups: userGroups.map(g => `${g.groupName} (${g.role})`)
      });
    }
  });
  
  analysis.totalUsers = analysis.totalUsers.size;
  
  return analysis;
}

// Display the analysis
const analysis = await analyzeGroupAccess(client);
console.log(`Total Groups: ${analysis.totalGroups}`);
console.log(`Total SSO Users: ${analysis.totalUsers}`);
console.log('\nGroup Hierarchy:');
console.table(analysis.groupDetails);
console.log('\nUsers in Multiple Groups:', analysis.priorityConflicts.length);
```

### 4. Automated Group Synchronization

```javascript
// Sync groups from IdP and auto-assign roles based on naming
async function autoSyncGroups(client) {
  const groups = await client.ssoGroups.list();
  const roles = await client.roles.list();
  
  // Define auto-mapping rules based on group name patterns
  const autoMappingRules = [
    { pattern: /admin|administrator/i, role: 'admin', priority: 40 },
    { pattern: /editor|content/i, role: 'editor', priority: 20 },
    { pattern: /developer|engineering/i, role: 'admin', priority: 35 },
    { pattern: /marketing|sales/i, role: 'editor', priority: 25 },
    { pattern: /viewer|readonly|guest/i, role: 'viewer', priority: 10 },
    { pattern: /.*/, role: 'viewer', priority: 5 } // Default fallback
  ];
  
  const results = {
    mapped: [],
    failed: [],
    unchanged: []
  };
  
  for (const group of groups) {
    // Skip if already mapped
    if (group.role) {
      results.unchanged.push(group.name);
      continue;
    }
    
    // Find matching rule
    const matchingRule = autoMappingRules.find(rule => 
      rule.pattern.test(group.name)
    );
    
    if (matchingRule) {
      const role = roles.find(r => r.name === matchingRule.role);
      
      if (role) {
        try {
          await client.ssoGroups.update(group.id, {
            role: { type: 'role', id: role.id },
            priority: matchingRule.priority
          });
          
          results.mapped.push({
            group: group.name,
            role: matchingRule.role,
            priority: matchingRule.priority
          });
        } catch (error) {
          results.failed.push({
            group: group.name,
            error: error.message
          });
        }
      }
    }
  }
  
  return results;
}
```

### 5. Group Cleanup

```javascript
// Remove empty groups and consolidate similar groups
async function cleanupGroups(client) {
  const groups = await client.ssoGroups.list();
  
  const cleanup = {
    emptyGroups: [],
    duplicateGroups: [],
    consolidated: []
  };
  
  // Find empty groups
  cleanup.emptyGroups = groups.filter(g => g.users.length === 0);
  
  // Find potential duplicates (similar names)
  const groupsByRole = {};
  groups.forEach(group => {
    const roleId = group.role?.id;
    if (roleId) {
      if (!groupsByRole[roleId]) {
        groupsByRole[roleId] = [];
      }
      groupsByRole[roleId].push(group);
    }
  });
  
  // Check for groups with same role and similar names
  Object.entries(groupsByRole).forEach(([roleId, roleGroups]) => {
    if (roleGroups.length > 1) {
      // Simple similarity check (you could use more sophisticated algorithms)
      roleGroups.forEach((group1, i) => {
        roleGroups.slice(i + 1).forEach(group2 => {
          const name1 = group1.name.toLowerCase();
          const name2 = group2.name.toLowerCase();
          
          if (name1.includes(name2) || name2.includes(name1)) {
            cleanup.duplicateGroups.push({
              group1: group1.name,
              group2: group2.name,
              sameRole: true
            });
          }
        });
      });
    }
  });
  
  // Optionally remove empty groups
  for (const group of cleanup.emptyGroups) {
    if (confirm(`Delete empty group "${group.name}"?`)) {
      await client.ssoGroups.destroy(group.id);
      console.log(`Deleted empty group: ${group.name}`);
    }
  }
  
  return cleanup;
}
```

### 6. Role Migration

```javascript
// Migrate users from one role to another via groups
async function migrateGroupRoles(client, fromRoleName, toRoleName) {
  const groups = await client.ssoGroups.list();
  const roles = await client.roles.list();
  
  const fromRole = roles.find(r => r.name === fromRoleName);
  const toRole = roles.find(r => r.name === toRoleName);
  
  if (!fromRole || !toRole) {
    throw new Error('Invalid role names provided');
  }
  
  const affectedGroups = groups.filter(g => g.role.id === fromRole.id);
  
  console.log(`Found ${affectedGroups.length} groups with role "${fromRoleName}"`);
  
  const results = [];
  
  for (const group of affectedGroups) {
    try {
      await client.ssoGroups.update(group.id, {
        role: { type: 'role', id: toRole.id }
      });
      
      results.push({
        group: group.name,
        users: group.users.length,
        status: 'migrated'
      });
      
      console.log(`✅ Migrated ${group.name} (${group.users.length} users)`);
    } catch (error) {
      results.push({
        group: group.name,
        users: group.users.length,
        status: 'failed',
        error: error.message
      });
    }
  }
  
  const totalUsers = results.reduce((sum, r) => sum + r.users, 0);
  console.log(`\nMigration complete: ${totalUsers} users affected`);
  
  return results;
}
```

## Best Practices

### Priority Guidelines

1. **Reserve high priorities (40-50+)** for administrative groups
2. **Use medium priorities (20-30)** for content management groups
3. **Use low priorities (10-20)** for read-only or limited access groups
4. **Leave gaps** between priority levels for future groups

### Group Management

1. **Naming Conventions**: Use clear, descriptive group names from your IdP
2. **Document Mappings**: Keep a record of group-to-role mappings
3. **Regular Audits**: Periodically review group assignments and priorities
4. **Test Changes**: Test role changes with a small group first

## Important Notes

1. **IdP Synchronization**: Groups are synchronized from your Identity Provider
2. **Cannot Create Groups**: Groups must exist in the IdP first
3. **User Assignment**: Users are assigned to groups by the IdP, not via DatoCMS
4. **Role Changes**: Updating a group's role affects all users in that group
5. **Priority Resolution**: Only the highest priority group's role applies to each user

## Object Structure

SSO Group objects have this structure:

```typescript
interface SsoGroup {
  id: string;
  type: 'sso_group';
  name: string;                        // Group name from Identity Provider
  priority: number;                    // Priority for role resolution (higher wins)
  role: Role | null;                   // Associated DatoCMS role
  users: SsoUser[];                    // Users belonging to this group
  meta: {
    created_at: string;                // ISO timestamp
    updated_at: string;                // ISO timestamp
    synchronized_at: string | null;    // Last sync from IdP
  };
}
```

**Example SSO Group:**
```javascript
const ssoGroup = {
  id: 'sso-group-123',
  type: 'sso_group',
  name: 'Content Editors',
  priority: 25,
  role: { 
    id: 'role-456', 
    type: 'role', 
    name: 'Editor',
    can_edit_schema: false,
    can_edit_favicon: true
  },
  users: [
    { id: 'user-1', type: 'sso_user', username: 'editor1@company.com' },
    { id: 'user-2', type: 'sso_user', username: 'editor2@company.com' }
  ],
  meta: {
    created_at: '2024-01-10T08:00:00.000Z',
    updated_at: '2024-01-20T12:15:00.000Z',
    synchronized_at: '2024-01-20T12:00:00.000Z'
  }
};
```

## Error Handling

```javascript
try {
  await client.ssoGroups.update('invalid-id', {
    role: { type: 'role', id: 'role-id' }
  });
} catch (error) {
  if (error.message.includes('NOT_FOUND')) {
    console.error('SSO group not found');
  } else if (error.message.includes('INVALID_FIELD')) {
    console.error('Invalid role ID provided');
  }
}
```

---

# SSO Settings

The SsoSettings resource manages Single Sign-On configuration for your DatoCMS organization. This includes SAML 2.0 settings, SCIM provisioning, and group-to-role mappings.

## Overview

SSO Settings allow you to:
- Configure SAML 2.0 authentication
- Enable SCIM provisioning
- Set default roles for new users
- Map IdP groups to DatoCMS roles
- Customize login behavior

## Available Operations

### Get SSO Settings

```javascript
const settings = await client.ssoSettings.find();
```

**Returns:** Current SSO configuration

### Update SSO Settings

```javascript
const updated = await client.ssoSettings.update({
  idp_saml_metadata_url: 'https://idp.example.com/metadata',
  default_role: { type: 'role', id: 'viewer-role-id' },
  domain_whitelist: ['example.com', 'company.org']
});
```

**Parameters:** See configuration options below

### Request SAML Identity

```javascript
const identity = await client.ssoSettings.requestSamlIdentity();
```

**Returns:** Service Provider metadata for IdP configuration

### Generate SCIM Token

```javascript
const token = await client.ssoSettings.generateToken();
```

**Returns:** New SCIM provisioning token

## Configuration Options

### SAML Settings

```typescript
{
  // Identity Provider metadata URL
  idp_saml_metadata_url: string;
  
  // OR manual configuration:
  idp_saml_sso_target_url: string;
  idp_saml_slo_target_url?: string;
  idp_saml_cert: string;
  
  // Service Provider settings (read-only)
  sp_saml_entity_id: string;
  sp_saml_metadata_url: string;
  sp_saml_acs_url: string;
  sp_saml_sls_url: string;
}
```

### Access Control

```typescript
{
  // Default role for new SSO users
  default_role: {
    type: 'role';
    id: string;
  };
  
  // Restrict access to specific email domains
  domain_whitelist: string[];
  
  // Disable non-SSO login
  sso_enforcement_enabled: boolean;
}
```

### SCIM Settings

```typescript
{
  // SCIM endpoint URL (read-only)
  scim_endpoint_url: string;
  
  // Active SCIM token (write-only)
  scim_token?: string;
}
```

## Common Configurations

### 1. Basic SAML Setup

```javascript
// Configure SAML with metadata URL
async function setupSAML(client, metadataUrl) {
  const settings = await client.ssoSettings.update({
    idp_saml_metadata_url: metadataUrl,
    default_role: { type: 'role', id: 'viewer-role-id' }
  });
  
  console.log('SAML configured successfully');
  console.log(`SP Entity ID: ${settings.sp_saml_entity_id}`);
  console.log(`SP Metadata URL: ${settings.sp_saml_metadata_url}`);
  
  return settings;
}
```

### 2. Manual SAML Configuration

```javascript
// Configure SAML manually (without metadata URL)
async function setupSAMLManually(client, config) {
  const settings = await client.ssoSettings.update({
    idp_saml_sso_target_url: config.ssoUrl,
    idp_saml_slo_target_url: config.sloUrl,
    idp_saml_cert: config.certificate,
    default_role: { type: 'role', id: config.defaultRoleId }
  });
  
  return settings;
}
```

### 3. Enable SCIM Provisioning

```javascript
// Setup SCIM for automatic user provisioning
async function enableSCIM(client) {
  // Generate new SCIM token
  const tokenResponse = await client.ssoSettings.generateToken();
  const scimToken = tokenResponse.scim_token;
  
  // Get current settings to find SCIM endpoint
  const settings = await client.ssoSettings.find();
  
  console.log('SCIM Configuration:');
  console.log(`Endpoint: ${settings.scim_endpoint_url}`);
  console.log(`Token: ${scimToken}`);
  console.log('\n⚠️  Save this token securely - it cannot be retrieved later!');
  
  return {
    endpoint: settings.scim_endpoint_url,
    token: scimToken
  };
}
```

### 4. Domain Whitelisting

```javascript
// Restrict SSO access to specific domains
async function configureDomainWhitelist(client, allowedDomains) {
  const settings = await client.ssoSettings.update({
    domain_whitelist: allowedDomains
  });
  
  console.log(`SSO restricted to domains: ${allowedDomains.join(', ')}`);
  
  return settings;
}

// Example usage
await configureDomainWhitelist(client, ['company.com', 'subsidiary.com']);
```

### 5. Enforce SSO Login

```javascript
// Disable password-based login, force SSO
async function enforceSSO(client) {
  const settings = await client.ssoSettings.update({
    sso_enforcement_enabled: true
  });
  
  console.log('⚠️  SSO enforcement enabled - users can only login via SSO');
  
  return settings;
}
```

## SSO Implementation Workflow

### Step 1: Get Service Provider Information

```javascript
async function getServiceProviderInfo(client) {
  // Request SAML identity to get SP metadata
  const identity = await client.ssoSettings.requestSamlIdentity();
  
  console.log('Service Provider Configuration:');
  console.log(`Entity ID: ${identity.sp_entity_id}`);
  console.log(`Metadata URL: ${identity.sp_metadata_url}`);
  console.log(`ACS URL: ${identity.sp_acs_url}`);
  console.log(`SLS URL: ${identity.sp_sls_url}`);
  
  // These values need to be configured in your IdP
  return identity;
}
```

### Step 2: Configure Identity Provider

```javascript
// After configuring your IdP, update DatoCMS with IdP metadata
async function configureIdentityProvider(client, idpMetadataUrl) {
  const settings = await client.ssoSettings.update({
    idp_saml_metadata_url: idpMetadataUrl,
    default_role: { type: 'role', id: 'appropriate-role-id' }
  });
  
  console.log('Identity Provider configured successfully');
  
  return settings;
}
```

### Step 3: Map Groups to Roles

```javascript
// Configure group mappings after SSO is working
async function mapGroupsToRoles(client) {
  const groups = await client.ssoGroups.list();
  const roles = await client.roles.list();
  
  // Example mappings
  const mappings = [
    { groupName: 'Admins', roleName: 'Admin', priority: 100 },
    { groupName: 'Editors', roleName: 'Editor', priority: 50 },
    { groupName: 'Viewers', roleName: 'Viewer', priority: 10 }
  ];
  
  for (const mapping of mappings) {
    const group = groups.find(g => g.name === mapping.groupName);
    const role = roles.find(r => r.name === mapping.roleName);
    
    if (group && role) {
      await client.ssoGroups.update(group.id, {
        role: { type: 'role', id: role.id },
        priority: mapping.priority
      });
      
      console.log(`Mapped ${mapping.groupName} → ${mapping.roleName}`);
    }
  }
}
```

## Complete SSO Setup Example

```javascript
// Complete SSO implementation
async function implementSSO(client, config) {
  console.log('=== DatoCMS SSO Setup ===\n');
  
  // Step 1: Get SP information for IdP configuration
  console.log('Step 1: Service Provider Information');
  const spInfo = await client.ssoSettings.requestSamlIdentity();
  console.log('Configure your IdP with:');
  console.log(`- SP Entity ID: ${spInfo.sp_entity_id}`);
  console.log(`- SP ACS URL: ${spInfo.sp_acs_url}`);
  console.log(`- SP Metadata URL: ${spInfo.sp_metadata_url}\n`);
  
  // Step 2: Configure SAML settings
  console.log('Step 2: Configure SAML');
  await client.ssoSettings.update({
    idp_saml_metadata_url: config.idpMetadataUrl,
    default_role: { type: 'role', id: config.defaultRoleId },
    domain_whitelist: config.allowedDomains || []
  });
  console.log('✓ SAML configured\n');
  
  // Step 3: Enable SCIM if needed
  if (config.enableSCIM) {
    console.log('Step 3: Enable SCIM Provisioning');
    const scimConfig = await client.ssoSettings.generateToken();
    const settings = await client.ssoSettings.find();
    
    console.log('Configure your IdP SCIM with:');
    console.log(`- SCIM Endpoint: ${settings.scim_endpoint_url}`);
    console.log(`- Bearer Token: ${scimConfig.scim_token}`);
    console.log('⚠️  Save this token securely!\n');
  }
  
  // Step 4: Test SSO login
  console.log('Step 4: Test SSO');
  const loginUrl = `https://${config.organizationId}.admin.datocms.com/sign_in`;
  console.log(`Test SSO at: ${loginUrl}`);
  console.log('Users should see "Sign in with SSO" option\n');
  
  // Step 5: Configure group mappings
  console.log('Step 5: Configure Group Mappings');
  await new Promise(resolve => setTimeout(resolve, 5000)); // Wait for groups to sync
  await mapGroupsToRoles(client);
  
  console.log('\n✅ SSO setup complete!');
}

// Usage
await implementSSO(client, {
  idpMetadataUrl: 'https://idp.example.com/metadata',
  defaultRoleId: 'viewer-role-id',
  allowedDomains: ['company.com'],
  enableSCIM: true,
  organizationId: 'your-org-id'
});
```

## Troubleshooting SSO

### Debug SAML Issues

```javascript
// Check current SSO configuration
async function debugSSO(client) {
  const settings = await client.ssoSettings.find();
  
  console.log('Current SSO Configuration:');
  console.log('========================');
  
  // SAML settings
  console.log('\nSAML Configuration:');
  console.log(`IdP Metadata URL: ${settings.idp_saml_metadata_url || 'Not set'}`);
  console.log(`IdP SSO URL: ${settings.idp_saml_sso_target_url || 'Not set'}`);
  console.log(`IdP Certificate: ${settings.idp_saml_cert ? '✓ Configured' : '✗ Not set'}`);
  
  // Access control
  console.log('\nAccess Control:');
  console.log(`Default Role: ${settings.default_role?.id || 'Not set'}`);
  console.log(`Domain Whitelist: ${settings.domain_whitelist?.join(', ') || 'None'}`);
  console.log(`SSO Enforcement: ${settings.sso_enforcement_enabled ? 'Enabled' : 'Disabled'}`);
  
  // SCIM status
  console.log('\nSCIM Provisioning:');
  console.log(`SCIM Endpoint: ${settings.scim_endpoint_url}`);
  console.log(`SCIM Token: ${settings.scim_token ? '✓ Active' : '✗ Not configured'}`);
  
  // Check groups and users
  const groups = await client.ssoGroups.list();
  const users = await client.ssoUsers.list();
  
  console.log('\nSSO Statistics:');
  console.log(`Total Groups: ${groups.length}`);
  console.log(`Total SSO Users: ${users.length}`);
  
  return settings;
}
```

### Common Issues

1. **Users can't see SSO option**
   - Check if IdP metadata is configured
   - Verify domain whitelist settings

2. **SAML errors during login**
   - Validate IdP certificate
   - Check ACS URL configuration in IdP
   - Ensure clock sync between IdP and DatoCMS

3. **Groups not syncing**
   - Verify SAML attribute mappings
   - Check if groups are included in SAML assertion
   - For SCIM: verify token is valid

4. **Users getting wrong role**
   - Check group priority settings
   - Verify group-to-role mappings
   - Ensure default role is set

## Best Practices

1. **Test in Sandbox**: Always test SSO configuration in a non-production environment first
2. **Document Mappings**: Keep clear documentation of group-to-role mappings
3. **Monitor Access**: Regularly audit SSO users and their permissions
4. **Backup Access**: Keep at least one non-SSO admin account for emergencies
5. **Token Security**: Store SCIM tokens securely and rotate periodically

---

# Sessions

The Session resource represents active user authentication sessions in DatoCMS. Sessions are created when users log in and are used to track active user access.

## Available Operations

### List Sessions

```javascript
const sessions = await client.sessions.list();
```

**Returns:** Array of all active sessions

### Find Session

```javascript
const session = await client.sessions.find(sessionId);
```

**Parameters:**
- `sessionId` (string, required): The session ID

**Returns:** Single session object

### Destroy Session

```javascript
await client.sessions.destroy(sessionId);
```

**Parameters:**
- `sessionId` (string, required): The session ID to terminate

**Returns:** Empty response on success

## The Session Object

```typescript
{
  id: string;                    // Unique session ID
  type: 'session';              // Always 'session'
  created_at: string;           // Session creation time (ISO 8601)
  last_active_at: string;       // Last activity timestamp
  ip_address: string;           // Client IP address
  user_agent: string;           // Browser/client user agent
  user: {                       // Associated user
    id: string;
    type: 'user' | 'sso_user';
  };
}
```

## Common Use Cases

### Monitor Active Sessions

```javascript
// Get all active sessions with user details
async function getActiveSessions(client) {
  const sessions = await client.sessions.list();
  const users = await client.users.list();
  const ssoUsers = await client.ssoUsers.list();
  
  // Create user lookup
  const userMap = new Map([
    ...users.map(u => [u.id, { ...u, userType: 'regular' }]),
    ...ssoUsers.map(u => [u.id, { ...u, userType: 'sso' }])
  ]);
  
  const sessionDetails = sessions.map(session => {
    const user = userMap.get(session.user.id);
    return {
      sessionId: session.id,
      userName: user ? `${user.first_name} ${user.last_name}` : 'Unknown',
      email: user?.email || user?.username || 'Unknown',
      userType: user?.userType || 'unknown',
      createdAt: new Date(session.created_at),
      lastActiveAt: new Date(session.last_active_at),
      duration: Date.now() - new Date(session.created_at).getTime(),
      ipAddress: session.ip_address,
      userAgent: session.user_agent
    };
  });
  
  // Sort by most recent activity
  sessionDetails.sort((a, b) => b.lastActiveAt - a.lastActiveAt);
  
  return sessionDetails;
}
```

### Session Security Monitoring

```javascript
// Detect suspicious session activity
async function monitorSessionSecurity(client) {
  const sessions = await client.sessions.list();
  
  const alerts = {
    multipleIPs: [],
    unusualLocations: [],
    longDuration: [],
    suspiciousAgents: []
  };
  
  // Group sessions by user
  const userSessions = {};
  sessions.forEach(session => {
    const userId = session.user.id;
    if (!userSessions[userId]) {
      userSessions[userId] = [];
    }
    userSessions[userId].push(session);
  });
  
  // Check for suspicious patterns
  Object.entries(userSessions).forEach(([userId, userSessionList]) => {
    // Multiple IPs for same user
    const uniqueIPs = new Set(userSessionList.map(s => s.ip_address));
    if (uniqueIPs.size > 1) {
      alerts.multipleIPs.push({
        userId,
        ips: Array.from(uniqueIPs),
        sessionCount: userSessionList.length
      });
    }
    
    // Long-running sessions (> 24 hours)
    userSessionList.forEach(session => {
      const duration = Date.now() - new Date(session.created_at).getTime();
      const hours = duration / (1000 * 60 * 60);
      
      if (hours > 24) {
        alerts.longDuration.push({
          sessionId: session.id,
          userId,
          hours: Math.round(hours),
          ipAddress: session.ip_address
        });
      }
    });
  });
  
  return alerts;
}

// Usage
const securityAlerts = await monitorSessionSecurity(client);
if (securityAlerts.multipleIPs.length > 0) {
  console.warn('⚠️  Users with multiple IP addresses:', securityAlerts.multipleIPs);
}
```

### Force Logout Users

```javascript
// Terminate all sessions for a specific user
async function forceLogoutUser(client, userEmail) {
  const sessions = await client.sessions.list();
  const users = await client.users.list();
  const ssoUsers = await client.ssoUsers.list();
  
  // Find user by email
  const user = [...users, ...ssoUsers].find(u => 
    (u.email || u.username) === userEmail
  );
  
  if (!user) {
    throw new Error(`User ${userEmail} not found`);
  }
  
  // Find all sessions for this user
  const userSessions = sessions.filter(s => s.user.id === user.id);
  
  console.log(`Found ${userSessions.length} active sessions for ${userEmail}`);
  
  // Terminate each session
  const results = [];
  for (const session of userSessions) {
    try {
      await client.sessions.destroy(session.id);
      results.push({ sessionId: session.id, terminated: true });
    } catch (error) {
      results.push({ sessionId: session.id, terminated: false, error });
    }
  }
  
  return results;
}

// Terminate all sessions except current
async function terminateOtherSessions(client, currentSessionId) {
  const sessions = await client.sessions.list();
  const otherSessions = sessions.filter(s => s.id !== currentSessionId);
  
  for (const session of otherSessions) {
    await client.sessions.destroy(session.id);
  }
  
  console.log(`Terminated ${otherSessions.length} other sessions`);
}
```

### Session Analytics

```javascript
// Analyze session patterns
async function analyzeSessionPatterns(client) {
  const sessions = await client.sessions.list();
  
  const analytics = {
    totalActive: sessions.length,
    byHour: {},
    byUserAgent: {},
    byDuration: {
      under1Hour: 0,
      under8Hours: 0,
      under24Hours: 0,
      over24Hours: 0
    },
    averageDuration: 0
  };
  
  let totalDuration = 0;
  
  sessions.forEach(session => {
    // Sessions by hour of day
    const hour = new Date(session.created_at).getHours();
    analytics.byHour[hour] = (analytics.byHour[hour] || 0) + 1;
    
    // Parse user agent
    const browser = detectBrowser(session.user_agent);
    analytics.byUserAgent[browser] = (analytics.byUserAgent[browser] || 0) + 1;
    
    // Duration analysis
    const duration = Date.now() - new Date(session.created_at).getTime();
    const hours = duration / (1000 * 60 * 60);
    
    if (hours < 1) analytics.byDuration.under1Hour++;
    else if (hours < 8) analytics.byDuration.under8Hours++;
    else if (hours < 24) analytics.byDuration.under24Hours++;
    else analytics.byDuration.over24Hours++;
    
    totalDuration += duration;
  });
  
  analytics.averageDuration = totalDuration / sessions.length / (1000 * 60 * 60); // hours
  
  return analytics;
}

// Simple browser detection
function detectBrowser(userAgent) {
  if (userAgent.includes('Chrome')) return 'Chrome';
  if (userAgent.includes('Firefox')) return 'Firefox';
  if (userAgent.includes('Safari')) return 'Safari';
  if (userAgent.includes('Edge')) return 'Edge';
  return 'Other';
}
```

## Security Best Practices

1. **Regular Monitoring**: Check for unusual session patterns
2. **Session Timeouts**: Implement automatic session expiration
3. **IP Monitoring**: Track sessions from unexpected locations
4. **Force Logout**: Terminate sessions when security concerns arise
5. **Audit Trail**: Log session creation and termination events

---

# Site Invitations

The Site Invitation resource manages pending invitations for users to join a DatoCMS project. Invitations are sent via email and expire after a set period.

## Available Operations

### List Invitations

```javascript
const invitations = await client.siteInvitations.list();
```

**Returns:** Array of all pending invitations

### Create Invitation

```javascript
const invitation = await client.siteInvitations.create({
  email: 'newuser@example.com',
  role: { type: 'role', id: 'editor-role-id' }
});
```

**Parameters:**
- `email` (string, required): Email address to invite
- `role` (object, required): Role to assign

**Returns:** Created invitation object

### Find Invitation

```javascript
const invitation = await client.siteInvitations.find(invitationId);
```

**Parameters:**
- `invitationId` (string, required): The invitation ID

**Returns:** Single invitation object

### Update Invitation

```javascript
const updated = await client.siteInvitations.update(invitationId, {
  role: { type: 'role', id: 'admin-role-id' }
});
```

**Parameters:**
- `invitationId` (string, required): The invitation ID
- `role` (object, required): New role assignment

**Returns:** Updated invitation object

### Resend Invitation

```javascript
await client.siteInvitations.resend(invitationId);
```

**Parameters:**
- `invitationId` (string, required): The invitation ID

**Returns:** Empty response on success

### Delete Invitation

```javascript
await client.siteInvitations.destroy(invitationId);
```

**Parameters:**
- `invitationId` (string, required): The invitation ID

**Returns:** Empty response on success

## The Invitation Object

```typescript
{
  id: string;                    // Unique invitation ID
  type: 'site_invitation';      // Always 'site_invitation'
  email: string;                // Invited email address
  role: {                       // Assigned role
    id: string;
    type: 'role';
  };
  created_at: string;           // When invitation was created
  expires_at: string;           // Expiration timestamp
  accepted_at?: string;         // When accepted (if applicable)
}
```

## Common Use Cases

### Bulk Invitations

```javascript
// Send invitations to multiple users
async function bulkInvite(client, inviteList) {
  const results = {
    sent: [],
    failed: [],
    existing: []
  };
  
  // Get existing users and invitations
  const [users, ssoUsers, pendingInvites] = await Promise.all([
    client.users.list(),
    client.ssoUsers.list(),
    client.siteInvitations.list()
  ]);
  
  // Create lookup sets
  const existingEmails = new Set([
    ...users.map(u => u.email.toLowerCase()),
    ...ssoUsers.map(u => u.username.toLowerCase()),
    ...pendingInvites.map(i => i.email.toLowerCase())
  ]);
  
  // Process each invitation
  for (const invite of inviteList) {
    const email = invite.email.toLowerCase();
    
    if (existingEmails.has(email)) {
      results.existing.push(email);
      continue;
    }
    
    try {
      const invitation = await client.siteInvitations.create({
        email: invite.email,
        role: { type: 'role', id: invite.roleId }
      });
      
      results.sent.push({
        email: invite.email,
        invitationId: invitation.id,
        expiresAt: invitation.expires_at
      });
    } catch (error) {
      results.failed.push({
        email: invite.email,
        error: error.message
      });
    }
  }
  
  console.log(`Invitations sent: ${results.sent.length}`);
  console.log(`Already exists: ${results.existing.length}`);
  console.log(`Failed: ${results.failed.length}`);
  
  return results;
}

// Usage
const invitations = await bulkInvite(client, [
  { email: 'editor1@example.com', roleId: 'editor-role-id' },
  { email: 'editor2@example.com', roleId: 'editor-role-id' },
  { email: 'viewer@example.com', roleId: 'viewer-role-id' }
]);
```

### Invitation Management

```javascript
// Monitor and manage pending invitations
async function manageInvitations(client) {
  const invitations = await client.siteInvitations.list();
  const now = new Date();
  
  const analysis = {
    total: invitations.length,
    expiringSoon: [],
    expired: [],
    byRole: {},
    byDomain: {}
  };
  
  invitations.forEach(invitation => {
    const expiresAt = new Date(invitation.expires_at);
    const hoursUntilExpiry = (expiresAt - now) / (1000 * 60 * 60);
    
    // Check expiration status
    if (expiresAt < now) {
      analysis.expired.push({
        email: invitation.email,
        expiredAt: expiresAt
      });
    } else if (hoursUntilExpiry < 24) {
      analysis.expiringSoon.push({
        email: invitation.email,
        hoursLeft: Math.round(hoursUntilExpiry)
      });
    }
    
    // Group by role
    const roleId = invitation.role.id;
    analysis.byRole[roleId] = (analysis.byRole[roleId] || 0) + 1;
    
    // Group by email domain
    const domain = invitation.email.split('@')[1];
    analysis.byDomain[domain] = (analysis.byDomain[domain] || 0) + 1;
  });
  
  return analysis;
}

// Resend expiring invitations
async function resendExpiringInvitations(client) {
  const analysis = await manageInvitations(client);
  
  for (const expiring of analysis.expiringSoon) {
    const invitation = invitations.find(i => i.email === expiring.email);
    if (invitation) {
      await client.siteInvitations.resend(invitation.id);
      console.log(`Resent invitation to ${expiring.email}`);
    }
  }
}
```

### Clean Up Expired Invitations

```javascript
// Remove expired invitations
async function cleanupExpiredInvitations(client) {
  const invitations = await client.siteInvitations.list();
  const now = new Date();
  const expired = [];
  
  for (const invitation of invitations) {
    if (new Date(invitation.expires_at) < now) {
      try {
        await client.siteInvitations.destroy(invitation.id);
        expired.push(invitation.email);
      } catch (error) {
        console.error(`Failed to delete invitation for ${invitation.email}:`, error);
      }
    }
  }
  
  console.log(`Cleaned up ${expired.length} expired invitations`);
  return expired;
}
```

### Role-Based Invitation Workflow

```javascript
// Invite users with appropriate roles based on email domain
async function smartInvite(client, email) {
  const roles = await client.roles.list();
  
  // Define role assignment rules
  const roleRules = [
    { domain: 'management.company.com', roleName: 'Admin' },
    { domain: 'company.com', roleName: 'Editor' },
    { domain: 'contractors.com', roleName: 'Viewer' }
  ];
  
  // Default role
  const defaultRole = roles.find(r => r.name === 'Viewer');
  
  // Determine role based on email domain
  const emailDomain = email.split('@')[1];
  let assignedRole = defaultRole;
  
  for (const rule of roleRules) {
    if (emailDomain === rule.domain) {
      assignedRole = roles.find(r => r.name === rule.roleName) || defaultRole;
      break;
    }
  }
  
  // Create invitation
  const invitation = await client.siteInvitations.create({
    email,
    role: { type: 'role', id: assignedRole.id }
  });
  
  console.log(`Invited ${email} with role: ${assignedRole.name}`);
  
  return invitation;
}
```

## Best Practices

1. **Check Existing Users**: Always verify if user already exists before inviting
2. **Role Assignment**: Assign appropriate roles based on user responsibilities
3. **Expiration Monitoring**: Track and resend invitations before they expire
4. **Bulk Operations**: Use batch processing for multiple invitations
5. **Audit Trail**: Log all invitation activities for compliance

## Error Handling

```javascript
try {
  await client.siteInvitations.create({
    email: 'existing@example.com',
    role: { type: 'role', id: 'role-id' }
  });
} catch (error) {
  if (error.message.includes('already exists')) {
    console.error('User already exists or has pending invitation');
  } else if (error.message.includes('invalid email')) {
    console.error('Invalid email address format');
  }
}
```

## Object Structure

SSO Settings objects have this structure:

```typescript
interface SsoSettings {
  id: string;
  type: 'sso_settings';
  idp_saml_metadata_url: string | null;          // IdP SAML metadata URL
  idp_saml_metadata_xml: string | null;          // IdP SAML metadata XML
  scim_base_url: string;                         // DatoCMS SCIM base URL (read-only)
  saml_acs_url: string;                          // DatoCMS SAML ACS URL (read-only)
  sp_saml_metadata_url: string;                  // DatoCMS SAML metadata URL (read-only)
  sp_saml_base_url: string;                      // DatoCMS SAML base URL (read-only)
  saml_token: string;                            // DatoCMS SAML token (read-only)
  default_role: Role | null;                     // Default role for new SSO users
  meta: {
    created_at: string;                          // ISO timestamp
    updated_at: string;                          // ISO timestamp
    last_sync_at: string | null;                 // Last SCIM sync timestamp
  };
}
```

**Example SSO Settings:**
```javascript
const ssoSettings = {
  id: 'sso-settings-123',
  type: 'sso_settings',
  idp_saml_metadata_url: 'https://idp.company.com/saml/metadata',
  idp_saml_metadata_xml: null,
  scim_base_url: 'https://site-api.datocms.com/scim/v2',
  saml_acs_url: 'https://site-api.datocms.com/saml/acs',
  sp_saml_metadata_url: 'https://site-api.datocms.com/saml/metadata',
  sp_saml_base_url: 'https://site-api.datocms.com/saml',
  saml_token: 'dat-scim-token-abc123',
  default_role: {
    id: 'role-456',
    type: 'role',
    name: 'Viewer',
    can_edit_schema: false,
    can_publish_items: false
  },
  meta: {
    created_at: '2024-01-05T09:00:00.000Z',
    updated_at: '2024-01-20T15:30:00.000Z',
    last_sync_at: '2024-01-20T14:45:00.000Z'
  }
};
```

**Key Properties Explained:**

- **Read-only URLs**: `scim_base_url`, `saml_acs_url`, `sp_saml_metadata_url`, `sp_saml_base_url`, `saml_token` are generated by DatoCMS
- **IdP Configuration**: Use either `idp_saml_metadata_url` OR `idp_saml_metadata_xml`, not both
- **Default Role**: Applied to new SSO users who don't belong to any mapped groups
- **SCIM Token**: Use the `saml_token` for SCIM provisioning authentication

---

# Editing Sessions (Private API)

> ⚠️ **Warning**: This is a private API that may change without notice. It is not recommended for production use. Use webhooks or polling for monitoring content changes instead.

The EditingSession resource manages collaborative editing sessions in DatoCMS. It tracks which users are currently editing content items and provides locking mechanisms to prevent editing conflicts. This API is primarily used by the DatoCMS web interface.

## Available Operations

### List Editing Sessions

```javascript
// NOT RECOMMENDED - Private API
const sessions = await client.editingSessions.list();
```

**Returns:** Array of active editing sessions

### Update Editing Session

```javascript
// NOT RECOMMENDED - Private API
const response = await client.editingSessions.rawUpdate(sessionId, {
  type: "editing_session_lock_item",
  relationships: {
    active_item: { type: "item", id: "item_id" }
  }
});
```

**Parameters:**
- `sessionId` (string, required): The editing session ID
- `body` (object, required): Update action and relationships

**Update Types:**
- `editing_session_enter_item` - Enter item for editing
- `editing_session_take_over_item` - Take over from another user
- `editing_session_lock_item` - Lock item to prevent others
- `editing_session_unlock_item` - Unlock item

### Delete Editing Session

```javascript
// NOT RECOMMENDED - Private API
await client.editingSessions.destroy(sessionId);
```

**Parameters:**
- `sessionId` (string, required): The editing session ID to delete

## The EditingSession Object

```typescript
{
  id: string;                        // Session identifier
  type: 'editing_session';           // Always 'editing_session'
  last_activity_at?: string | null;  // Last activity timestamp
  locked_at?: string | null;         // Lock timestamp
  relationships: {
    active_item: {                   // Item being edited
      id: string;
      type: 'item';
    };
    active_item_type: {              // Item type being edited
      id: string;
      type: 'item_type';
    };
    editor: {                        // User editing
      id: string;
      type: 'user' | 'sso_user' | 'account' | 'access_token';
    };
  }
}
```

## Why This API is Private

1. **Internal Complexity**: Manages real-time collaborative features
2. **WebSocket Integration**: Tied to internal WebSocket infrastructure
3. **UI Coupling**: Tightly coupled with web interface behavior
4. **Stability**: Subject to change based on UI requirements

## Recommended Alternatives

### 1. Monitor Content Changes via Webhooks

```javascript
// Recommended: Use webhooks to track content changes
const webhook = await client.webhooks.create({
  name: "Content Change Tracker",
  url: "https://your-app.com/webhooks/content",
  events: [
    { entity_type: "item", event_types: ["update", "create"] }
  ]
});
```

### 2. Track User Activity via Audit Logs

```javascript
// Recommended: Use audit logs for activity tracking
const events = await client.auditLogEvents.list({
  filter: {
    entity_type: "item",
    event_type: ["update"]
  },
  order_by: "created_at_DESC"
});

// See who edited what
events.forEach(event => {
  console.log(`${event.user.email} edited ${event.entity.id} at ${event.created_at}`);
});
```

### 3. Implement Custom Locking

```javascript
// Recommended: Implement your own locking mechanism
class ContentLocker {
  constructor(client) {
    this.client = client;
    this.locks = new Map(); // In production, use Redis or similar
  }
  
  async lockItem(itemId, userId, duration = 300000) { // 5 minutes
    const existingLock = this.locks.get(itemId);
    
    if (existingLock && existingLock.expires > Date.now()) {
      if (existingLock.userId !== userId) {
        throw new Error(`Item locked by user ${existingLock.userId}`);
      }
    }
    
    this.locks.set(itemId, {
      userId,
      expires: Date.now() + duration
    });
    
    // Store lock metadata on the item
    await this.client.items.update(itemId, {
      meta: {
        locked_by: userId,
        locked_until: new Date(Date.now() + duration).toISOString()
      }
    });
  }
  
  async unlockItem(itemId, userId) {
    const lock = this.locks.get(itemId);
    
    if (lock && lock.userId === userId) {
      this.locks.delete(itemId);
      
      // Clear lock metadata
      await this.client.items.update(itemId, {
        meta: {
          locked_by: null,
          locked_until: null
        }
      });
    }
  }
}
```

## Use Cases (Internal Only)

The EditingSession API is used internally for:

1. **Real-time Collaboration**: Show who's editing what in the UI
2. **Conflict Prevention**: Prevent simultaneous edits
3. **Edit Takeover**: Allow admins to take over stuck sessions
4. **Activity Tracking**: Monitor user editing patterns

## Understanding the Workflow

### Internal Editing Flow:
1. User opens item in editor → `editing_session_enter_item`
2. System tracks activity → Updates `last_activity_at`
3. User requests lock → `editing_session_lock_item`
4. Other user attempts edit → Sees lock notification
5. Admin takes over → `editing_session_take_over_item`
6. Session cleanup → `destroy()` on close

## Migration Guide

If you need collaborative editing features:

### Option 1: Polling-Based Status
```javascript
// Check if item is being edited (custom implementation)
async function isItemBeingEdited(client, itemId) {
  const recentActivity = await client.auditLogEvents.list({
    filter: {
      entity_id: { eq: itemId },
      entity_type: "item",
      created_at: { gt: new Date(Date.now() - 300000).toISOString() } // Last 5 min
    }
  });
  
  return recentActivity.length > 0;
}
```

### Option 2: WebSocket Integration
```javascript
// Use DatoCMS WebSocket for real-time updates
import { EventSource } from 'eventsource';

const events = new EventSource(
  `https://site-api.datocms.com/events?item_id=${itemId}`,
  { headers: { 'Authorization': `Bearer ${apiToken}` } }
);

events.on('message', (event) => {
  const data = JSON.parse(event.data);
  if (data.entity.type === 'item' && data.event_type === 'update') {
    console.log('Item being edited by:', data.user);
  }
});
```

## Important Notes

1. **No Public Support**: This API is not officially supported
2. **Breaking Changes**: May change or be removed without notice
3. **Rate Limits**: Subject to different rate limits than public APIs
4. **WebSocket Dependency**: Relies on internal WebSocket infrastructure
5. **UI-Specific**: Behavior tied to web interface requirements

## Best Practices

If you absolutely must use this API:

1. **Wrap in Try-Catch**: Always handle failures gracefully
2. **Don't Rely on It**: Have fallback mechanisms
3. **Monitor Changes**: Watch for API changes in DatoCMS updates
4. **Use Sparingly**: Only for non-critical features
5. **Document Usage**: Make it clear this is using private APIs

Remember: For production applications, always use the recommended alternatives instead of private APIs.