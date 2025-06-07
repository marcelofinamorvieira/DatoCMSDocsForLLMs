# Dashboard API - Organization Management (Merged Documentation)

This file contains merged documentation for all organization-related resources in the Dashboard API:
- Organizations
- Organization Memberships
- Organization Invitations
- Organization Roles
- Organization Mandates
- Organization Mandate Requests

## Overview

Organizations in DatoCMS allow teams to collaborate across multiple projects with centralized billing and member management. Organizations enable shared access control, consolidated billing, and multi-site management.

## Client Initialization

```typescript
import { buildClient } from '@datocms/dashboard-client';

const dashboardClient = buildClient({
  apiToken: 'YOUR_ACCOUNT_API_TOKEN', // Account API token required
  environment: 'production'
});
```

## Organizations Resource

### Resource: `dashboardClient.organizations`

**JSON:API Type**: `organization`  
**Purpose**: Manage organizations and their settings

### Create Organization

Creates a new organization with optional company data.

```typescript
const organization = await dashboardClient.organizations.create({
  name: 'Acme Corporation',
  slug: 'acme-corp', // Optional, auto-generated if not provided
  billing_email: 'billing@acme.com',
  company_data: {
    company_name: 'Acme Corporation Ltd.',
    country: 'US',
    vat_number: 'US123456789',
    address_line_1: '123 Main St',
    city: 'New York',
    zip_code: '10001',
    state: 'NY'
  }
});

// Returns:
{
  id: '123',
  type: 'organization',
  name: 'Acme Corporation',
  slug: 'acme-corp',
  billing_email: 'billing@acme.com',
  created_at: '2024-01-01T00:00:00Z',
  updated_at: '2024-01-01T00:00:00Z',
  can_create_new_sites: true,
  sites_count: 0,
  members_count: 1,
  pending_invitations_count: 0,
  owner: { type: 'account', id: '456' },
  billing_profile: { type: 'per_owner_pricing_billing_profile', id: '789' }
}
```

**Parameters:**

```typescript
interface SimpleOrganizationCreateSchema {
  name: string;
  slug?: string;                    // Auto-generated from name if not provided
  billing_email?: string;
  company_data?: {
    company_name?: string;
    country?: string;              // ISO 3166-1 alpha-2 code
    vat_number?: string;
    address_line_1?: string;
    address_line_2?: string;
    city?: string;
    zip_code?: string;
    state?: string;                // For US/CA addresses
  };
}
```

### List Organizations

Lists all organizations the authenticated user has access to.

```typescript
// List all accessible organizations
const organizations = await dashboardClient.organizations.list();

// Filter by role
const myOrganizations = await dashboardClient.organizations.list({
  filter: { role: { eq: 'owner' } }
});

// Include related data
const detailedOrgs = await dashboardClient.organizations.list({
  include: ['owner', 'billing_profile', 'sites'],
  per_page: 50,
  page: 1
});
```

### Find Organization

Retrieves a single organization by ID.

```typescript
const organization = await dashboardClient.organizations.find('org-id', {
  include: ['owner', 'billing_profile', 'sites']
});
```

### Update Organization

Updates an existing organization's details.

```typescript
const updated = await dashboardClient.organizations.update('org-id', {
  name: 'Acme Corp International',
  billing_email: 'finance@acme.com',
  company_data: {
    vat_number: 'US987654321'
  }
});
```

### Delete Organization

Deletes an organization. All sites must be transferred or deleted first.

```typescript
await dashboardClient.organizations.destroy('org-id');
```

### Organization Attributes

```typescript
interface Organization {
  id: string;
  type: 'organization';
  name: string;
  slug: string;
  billing_email: string | null;
  created_at: string;
  updated_at: string;
  can_create_new_sites: boolean;
  sites_count: number;
  members_count: number;
  pending_invitations_count: number;
  
  // Company data
  company_name?: string;
  country?: string;
  vat_number?: string;
  address_line_1?: string;
  address_line_2?: string;
  city?: string;
  zip_code?: string;
  state?: string;
  
  // Relationships
  owner?: Account;
  billing_profile?: BillingProfile;
  sites?: Site[];
  memberships?: OrganizationMembership[];
  invitations?: OrganizationInvitation[];
}
```

## Organization Memberships Resource

### Resource: `dashboardClient.organizationMemberships`

**JSON:API Type**: `organization_membership`  
**Purpose**: Manage organization members and their roles

### List Organization Members

```typescript
// List all members of an organization
const memberships = await dashboardClient.organizationMemberships.list({
  filter: { organization: { eq: 'org-id' } },
  include: ['user', 'role', 'organization'],
  per_page: 100
});

// Filter by role
const admins = await dashboardClient.organizationMemberships.list({
  filter: {
    organization: { eq: 'org-id' },
    role: { eq: 'admin' }
  }
});
```

### Find Membership

Retrieves a specific membership by ID.

```typescript
const membership = await dashboardClient.organizationMemberships.find('membership-id', {
  include: ['user', 'role']
});
```

### Find Current User's Membership

Gets the authenticated user's membership in an organization.

```typescript
const myMembership = await dashboardClient.organizationMemberships.findMe({
  filter: { organization: { eq: 'org-id' } }
});

// Check permissions
const role = await dashboardClient.organizationRoles.find(
  myMembership.relationships.role.data.id
);
console.log('Can manage organization:', role.can_manage_organization);
```

### Update Member Role

Changes a member's role within the organization.

```typescript
const updated = await dashboardClient.organizationMemberships.update('membership-id', {
  role: { type: 'organization_role', id: 'admin' }
});
```

### Remove Member

Removes a member from the organization.

```typescript
await dashboardClient.organizationMemberships.destroy('membership-id');
```

### Membership Attributes

```typescript
interface OrganizationMembership {
  id: string;
  type: 'organization_membership';
  created_at: string;
  updated_at: string;
  is_owner: boolean;
  joined_at: string;
  
  // Relationships
  organization?: Organization;
  user?: Account;
  role?: OrganizationRole;
}
```

## Organization Invitations Resource

### Resource: `dashboardClient.organizationInvitations`

**JSON:API Type**: `organization_invitation`  
**Purpose**: Invite new members to organizations

### Create Invitation

Invites a new member to the organization.

```typescript
const invitation = await dashboardClient.organizationInvitations.create({
  email: 'newmember@example.com',
  organization: { type: 'organization', id: 'org-id' },
  role: { type: 'organization_role', id: 'developer' },
  message: 'Welcome to our team! Looking forward to collaborating with you.'
});

// Returns:
{
  id: 'inv-789',
  type: 'organization_invitation',
  email: 'newmember@example.com',
  status: 'pending',
  created_at: '2024-07-06T10:00:00.000Z',
  sent_at: '2024-07-06T10:00:15.000Z',
  expires_at: '2024-07-13T10:00:00.000Z',
  invitation_url: 'https://dashboard.datocms.com/invite/abc123...'
}
```

**Parameters:**

```typescript
interface SimpleOrganizationInvitationCreateSchema {
  email: string;
  organization: { type: 'organization', id: string };
  role: { type: 'organization_role', id: string };
  message?: string;  // Optional custom message
}
```

### List Invitations

Lists all invitations for an organization.

```typescript
// All invitations
const allInvitations = await dashboardClient.organizationInvitations.list({
  filter: { organization: { eq: 'org-id' } }
});

// Pending invitations only
const pendingInvitations = await dashboardClient.organizationInvitations.list({
  filter: {
    organization: { eq: 'org-id' },
    status: { eq: 'pending' }
  }
});

// Include related data
const detailedInvitations = await dashboardClient.organizationInvitations.list({
  filter: { organization: { eq: 'org-id' } },
  include: ['organization', 'role', 'invited_by']
});
```

### Resend Invitation

Resends an invitation email to the recipient.

```typescript
const resent = await dashboardClient.organizationInvitations.update('inv-id', {
  resend: true
});

console.log('Invitation resent at:', resent.sent_at);
```

### Update Invitation Role

Changes the role for a pending invitation.

```typescript
const updated = await dashboardClient.organizationInvitations.update('inv-id', {
  role: { type: 'organization_role', id: 'admin' }
});
```

### Cancel Invitation

Cancels a pending invitation.

```typescript
await dashboardClient.organizationInvitations.destroy('invitation-id');
```

### Invitation Attributes

```typescript
interface OrganizationInvitation {
  id: string;
  type: 'organization_invitation';
  email: string;
  status: 'pending' | 'accepted' | 'expired';
  created_at: string;
  sent_at: string;
  expires_at: string;
  accepted_at: string | null;
  invitation_url: string;
  
  // Relationships
  organization?: Organization;
  role?: OrganizationRole;
  invited_by?: Account;
  user?: Account;  // Populated after acceptance
}
```

## Organization Roles Resource

### Resource: `dashboardClient.organizationRoles`

**JSON:API Type**: `organization_role`  
**Purpose**: Define permissions for organization members

### List Available Roles

```typescript
const roles = await dashboardClient.organizationRoles.list();

// Returns predefined roles:
[
  {
    id: 'owner',
    type: 'organization_role',
    name: 'Owner',
    description: 'Full control over the organization',
    can_create_sites: true,
    can_manage_organization: true,
    can_manage_billing: true,
    can_invite_members: true,
    can_remove_members: true,
    can_manage_member_roles: true,
    can_delete_organization: true,
    can_transfer_sites: true,
    can_access_all_sites: true
  },
  {
    id: 'admin',
    type: 'organization_role',
    name: 'Admin',
    description: 'Administrative access to organization',
    can_create_sites: true,
    can_manage_organization: true,
    can_manage_billing: false,
    can_invite_members: true,
    can_remove_members: true,
    can_manage_member_roles: true,
    can_delete_organization: false,
    can_transfer_sites: true,
    can_access_all_sites: true
  },
  {
    id: 'developer',
    type: 'organization_role',
    name: 'Developer',
    description: 'Can create and manage sites',
    can_create_sites: true,
    can_manage_organization: false,
    can_manage_billing: false,
    can_invite_members: false,
    can_remove_members: false,
    can_manage_member_roles: false,
    can_delete_organization: false,
    can_transfer_sites: false,
    can_access_all_sites: false
  },
  {
    id: 'viewer',
    type: 'organization_role',
    name: 'Viewer',
    description: 'Read-only access to organization',
    can_create_sites: false,
    can_manage_organization: false,
    can_manage_billing: false,
    can_invite_members: false,
    can_remove_members: false,
    can_manage_member_roles: false,
    can_delete_organization: false,
    can_transfer_sites: false,
    can_access_all_sites: false
  }
]
```

### Find Role Details

```typescript
const role = await dashboardClient.organizationRoles.find('admin');
```

### Role Permissions Matrix

| Permission | Owner | Admin | Developer | Viewer |
|------------|-------|-------|-----------|--------|
| `can_create_sites` | ✓ | ✓ | ✓ | ✗ |
| `can_manage_organization` | ✓ | ✓ | ✗ | ✗ |
| `can_manage_billing` | ✓ | ✗ | ✗ | ✗ |
| `can_invite_members` | ✓ | ✓ | ✗ | ✗ |
| `can_remove_members` | ✓ | ✓ | ✗ | ✗ |
| `can_manage_member_roles` | ✓ | ✓ | ✗ | ✗ |
| `can_delete_organization` | ✓ | ✗ | ✗ | ✗ |
| `can_transfer_sites` | ✓ | ✓ | ✗ | ✗ |
| `can_access_all_sites` | ✓ | ✓ | ✗ | ✗ |

### Role Attributes

```typescript
interface OrganizationRole {
  id: string;
  type: 'organization_role';
  name: string;
  description: string;
  can_create_sites: boolean;
  can_manage_organization: boolean;
  can_manage_billing: boolean;
  can_invite_members: boolean;
  can_remove_members: boolean;
  can_manage_member_roles: boolean;
  can_delete_organization: boolean;
  can_transfer_sites: boolean;
  can_access_all_sites: boolean;
}
```

## Organization Mandates Resource

### Resource: `dashboardClient.organizationMandates`

**JSON:API Type**: `organization_mandate`  
**Purpose**: Manage payment mandates for SEPA Direct Debit and other mandate-based payment methods

### Create Mandate

Initiates a new payment mandate (returns an async job).

```typescript
const job = await dashboardClient.organizationMandates.create('org-id', {
  email: 'billing@company.com',
  name: 'Company Billing Department',
  scheme: 'sepa_core' // Options: 'sepa_core' | 'bacs' | 'ach'
});

// Monitor job completion
const result = await dashboardClient.jobResults.find(job.id);
```

**Parameters:**

```typescript
interface SimpleOrganizationMandateCreateSchema {
  email: string;
  name: string;
  scheme: 'sepa_core' | 'bacs' | 'ach';
}
```

### List Mandates

Retrieves all mandates for an organization.

```typescript
const mandates = await dashboardClient.organizationMandates.list('org-id');

// Filter active mandates
const activeMandates = mandates.filter(m => m.status === 'active');
```

### Cancel Mandate

Cancels an active mandate.

```typescript
await dashboardClient.organizationMandates.destroy('mandate-id');
```

### Mandate Statuses

- `pending_submission` - Created but not yet submitted to bank
- `submitted` - Submitted to bank, awaiting confirmation
- `active` - Confirmed and can be used for charges
- `failed` - Setup failed
- `cancelled` - Cancelled by user
- `expired` - Mandate expired

### Mandate Attributes

```typescript
interface OrganizationMandate {
  id: string;
  type: 'organization_mandate';
  reference: string;
  scheme: 'sepa_core' | 'bacs' | 'ach';
  status: 'pending_submission' | 'submitted' | 'active' | 'failed' | 'cancelled' | 'expired';
  created_at: string;
  activated_at: string | null;
  cancelled_at: string | null;
  
  // Bank account details (masked)
  account_name: string;
  account_number_ending: string;
  bank_name: string;
  
  // Relationships
  organization?: Organization;
}
```

## Organization Mandate Requests Resource

### Resource: `dashboardClient.organizationMandateRequests`

**JSON:API Type**: `organization_mandate_request`  
**Purpose**: Handle the initial setup flow for payment mandates with user interaction

### Create Mandate Request

Initiates a mandate setup request that requires user interaction.

```typescript
const request = await dashboardClient.organizationMandateRequests.create({
  organization: { type: 'organization', id: 'org-id' },
  email: 'finance@company.com',
  success_url: 'https://app.example.com/billing/success',
  cancel_url: 'https://app.example.com/billing/cancel'
});

// Redirect user to complete setup
window.location.href = request.checkout_url;
```

**Parameters:**

```typescript
interface SimpleOrganizationMandateRequestCreateSchema {
  organization: { type: 'organization', id: string };
  email: string;
  success_url: string;  // Where to redirect on success
  cancel_url: string;   // Where to redirect on cancel
}
```

### Find Request Status

Check the status of a mandate request.

```typescript
const request = await dashboardClient.organizationMandateRequests.find('request-id');

if (request.status === 'completed') {
  console.log('Mandate created:', request.mandate);
}
```

### Request Statuses

- `pending` - Awaiting user action
- `in_progress` - User is completing setup
- `completed` - Mandate created successfully
- `cancelled` - User cancelled the process
- `expired` - Request expired before completion

### Request Attributes

```typescript
interface OrganizationMandateRequest {
  id: string;
  type: 'organization_mandate_request';
  email: string;
  status: 'pending' | 'in_progress' | 'completed' | 'cancelled' | 'expired';
  checkout_url: string;
  success_url: string;
  cancel_url: string;
  created_at: string;
  expires_at: string;
  completed_at: string | null;
  
  // Relationships
  organization?: Organization;
  mandate?: OrganizationMandate;  // Available after completion
}
```

## Common Patterns

### Complete Organization Setup

```typescript
async function setupOrganization(client: DashboardClient) {
  try {
    // 1. Create organization
    const org = await client.organizations.create({
      name: 'Tech Startup Inc.',
      slug: 'tech-startup',
      billing_email: 'billing@techstartup.com',
      company_data: {
        company_name: 'Tech Startup Inc.',
        country: 'US',
        vat_number: 'US123456789',
        address_line_1: '456 Innovation Blvd',
        city: 'San Francisco',
        zip_code: '94105',
        state: 'CA'
      }
    });

    console.log('Organization created:', org.id);

    // 2. Invite team members
    const teamMembers = [
      { email: 'cto@techstartup.com', role: 'admin' },
      { email: 'lead@techstartup.com', role: 'developer' },
      { email: 'dev1@techstartup.com', role: 'developer' },
      { email: 'intern@techstartup.com', role: 'viewer' }
    ];

    for (const member of teamMembers) {
      await client.organizationInvitations.create({
        email: member.email,
        organization: { type: 'organization', id: org.id },
        role: { type: 'organization_role', id: member.role },
        message: 'Welcome to Tech Startup! Please join our organization.'
      });
    }

    // 3. Set up billing mandate (for SEPA)
    const mandateRequest = await client.organizationMandateRequests.create({
      organization: { type: 'organization', id: org.id },
      email: org.billing_email,
      success_url: 'https://app.techstartup.com/billing/success',
      cancel_url: 'https://app.techstartup.com/billing/cancel'
    });

    console.log('Billing setup URL:', mandateRequest.checkout_url);

    return { org, mandateRequest };

  } catch (error) {
    console.error('Organization setup failed:', error);
    throw error;
  }
}
```

### Member Management

```typescript
async function manageOrganizationMembers(client: DashboardClient, orgId: string) {
  // 1. List current members
  const memberships = await client.organizationMemberships.list({
    filter: { organization: { eq: orgId } },
    include: ['user', 'role']
  });

  console.log(`Current members: ${memberships.length}`);

  // 2. Check pending invitations
  const pendingInvitations = await client.organizationInvitations.list({
    filter: {
      organization: { eq: orgId },
      status: { eq: 'pending' }
    }
  });

  console.log(`Pending invitations: ${pendingInvitations.length}`);

  // 3. Promote developer to admin
  const developer = memberships.find(m => 
    m.relationships.role.data.id === 'developer'
  );

  if (developer) {
    await client.organizationMemberships.update(developer.id, {
      role: { type: 'organization_role', id: 'admin' }
    });
    console.log('Developer promoted to admin');
  }

  // 4. Remove inactive members
  const threeMonthsAgo = new Date();
  threeMonthsAgo.setMonth(threeMonthsAgo.getMonth() - 3);

  for (const membership of memberships) {
    const lastActive = new Date(membership.updated_at);
    if (lastActive < threeMonthsAgo && !membership.is_owner) {
      await client.organizationMemberships.destroy(membership.id);
      console.log('Removed inactive member:', membership.id);
    }
  }

  // 5. Resend expired invitations
  for (const invitation of pendingInvitations) {
    const expires = new Date(invitation.expires_at);
    if (expires < new Date()) {
      await client.organizationInvitations.update(invitation.id, {
        resend: true
      });
      console.log('Resent expired invitation to:', invitation.email);
    }
  }
}
```

### Permission Checking

```typescript
async function checkUserPermissions(client: DashboardClient, orgId: string) {
  try {
    // Get current user's membership
    const membership = await client.organizationMemberships.findMe({
      filter: { organization: { eq: orgId } }
    });

    // Get role details
    const role = await client.organizationRoles.find(
      membership.relationships.role.data.id
    );

    // Create permission object
    const permissions = {
      canCreateSites: role.can_create_sites,
      canManageOrganization: role.can_manage_organization,
      canManageBilling: role.can_manage_billing,
      canInviteMembers: role.can_invite_members,
      canRemoveMembers: role.can_remove_members,
      canManageMemberRoles: role.can_manage_member_roles,
      canDeleteOrganization: role.can_delete_organization,
      canTransferSites: role.can_transfer_sites,
      canAccessAllSites: role.can_access_all_sites,
      isOwner: membership.is_owner,
      roleName: role.name
    };

    console.log('User permissions:', permissions);
    return permissions;

  } catch (error) {
    console.error('Failed to check permissions:', error);
    return null;
  }
}
```

### Billing Setup with Mandates

```typescript
async function setupBillingMandate(client: DashboardClient, orgId: string) {
  // 1. Check existing mandates
  const existingMandates = await client.organizationMandates.list(orgId);
  const activeMandate = existingMandates.find(m => m.status === 'active');

  if (activeMandate) {
    console.log('Active mandate already exists:', activeMandate.reference);
    return activeMandate;
  }

  // 2. Create mandate request for user interaction
  const request = await client.organizationMandateRequests.create({
    organization: { type: 'organization', id: orgId },
    email: 'billing@company.com',
    success_url: 'https://app.example.com/billing/success',
    cancel_url: 'https://app.example.com/billing/cancel'
  });

  console.log('Direct user to:', request.checkout_url);

  // 3. Poll for completion (in real app, use webhooks)
  let completed = false;
  let attempts = 0;

  while (!completed && attempts < 60) {
    await new Promise(resolve => setTimeout(resolve, 5000)); // Wait 5 seconds

    const status = await client.organizationMandateRequests.find(request.id);
    
    if (status.status === 'completed') {
      completed = true;
      console.log('Mandate setup completed!');
      return status.mandate;
    } else if (status.status === 'cancelled' || status.status === 'expired') {
      throw new Error(`Mandate setup ${status.status}`);
    }

    attempts++;
  }

  throw new Error('Mandate setup timeout');
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/dashboard-client';

async function handleOrganizationErrors(client: DashboardClient) {
  try {
    await client.organizations.create({
      name: 'Test Org',
      slug: 'existing-slug' // This might already exist
    });
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.response?.status) {
        case 422:
          // Validation error
          console.error('Validation errors:', error.errors);
          // Example: [{ detail: "Slug has already been taken", source: { pointer: "/data/attributes/slug" } }]
          break;
        case 403:
          // Permission denied
          console.error('Not authorized to create organizations');
          break;
        case 402:
          // Payment required
          console.error('Upgrade required to create more organizations');
          break;
        case 409:
          // Conflict
          console.error('Organization with this slug already exists');
          break;
        default:
          console.error('API error:', error.message);
      }
    }
  }
}

// Common error scenarios
async function handleCommonErrors(client: DashboardClient, orgId: string) {
  // Cannot delete organization with sites
  try {
    await client.organizations.destroy(orgId);
  } catch (error) {
    if (error.response?.status === 422) {
      console.error('Must transfer or delete all sites first');
    }
  }

  // Cannot remove last owner
  try {
    await client.organizationMemberships.destroy('owner-membership-id');
  } catch (error) {
    if (error.response?.status === 422) {
      console.error('Cannot remove the last owner');
    }
  }

  // User already member
  try {
    await client.organizationInvitations.create({
      email: 'existing@member.com',
      organization: { type: 'organization', id: orgId },
      role: { type: 'organization_role', id: 'developer' }
    });
  } catch (error) {
    if (error.response?.status === 422) {
      console.error('User is already a member or has pending invitation');
    }
  }
}
```

## Rate Limiting

Organization API endpoints have specific rate limits:
- **Organization operations**: 30 requests per 3 seconds
- **Member operations**: 60 requests per 3 seconds
- **Invitation operations**: 30 requests per 3 seconds
- **Mandate operations**: 10 requests per 3 seconds

The client automatically handles rate limiting with exponential backoff.

## Best Practices

1. **Always check permissions** before attempting operations
2. **Handle invitation expiry** by resending or creating new invitations
3. **Monitor mandate status** for billing continuity
4. **Use pagination** when listing large numbers of members
5. **Include relationships** to reduce API calls
6. **Validate email addresses** before sending invitations
7. **Set up billing early** in the organization lifecycle
8. **Regularly audit** memberships and permissions

## Related Resources

- [Account Management](./merged-account-management.md)
- [Billing & Subscriptions](./merged-billing-subscriptions.md)
- [Site Management](./merged-site-management.md)
- [Authentication Setup](../00-quick-start/authentication.md)