# onBeforeItemsPublish Hook

The `onBeforeItemsPublish` hook is triggered before one or more records are published, allowing plugins to validate content quality, check completeness, ensure compliance, or prevent publication based on custom business rules. This hook is crucial for maintaining content standards and workflow requirements.

## Quick Reference

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsPublish(items, ctx) {
    // Validate content before publishing
    for (const item of items) {
      if (!item.attributes.title || !item.attributes.content) {
        ctx.alert('All items must have title and content');
        return false; // Prevent publishing
      }
    }
    return true; // Allow publishing
  }
});
```

## Purpose

The `onBeforeItemsPublish` hook validates records before they are published to ensure:
- Content quality standards are met
- Required fields are populated
- SEO requirements are satisfied
- Approval workflows are completed
- Dependencies are resolved

## Signature

```typescript
onBeforeItemsPublish(
  items: Item[],
  ctx: OnBeforeItemsPublishCtx
): MaybePromise<boolean>
```

## Parameters

- `items`: Item[] - Array of records that are about to be published
- `ctx`: OnBeforeItemsPublishCtx - The context object containing project information, current user details, and utility methods

## Type Definitions

```typescript
import type { SchemaTypes } from '@datocms/cma-client';
import { Ctx } from '../ctx/base';
import { MaybePromise } from '../utils';

// Core schema types from DatoCMS
type Account = SchemaTypes.Account;
type Field = SchemaTypes.Field;
type Fieldset = SchemaTypes.Fieldset;
type Item = SchemaTypes.Item;
type ItemType = SchemaTypes.ItemType;
type Organization = SchemaTypes.Organization;
type Plugin = SchemaTypes.Plugin;
type Role = SchemaTypes.Role;
type Site = SchemaTypes.Site;
type SsoUser = SchemaTypes.SsoUser;
type Upload = SchemaTypes.Upload;
type User = SchemaTypes.User;

// Hook definition
export type OnBeforeItemsPublishHook = {
  /**
   * This function will be called before publishing records. You can stop the
   * action by returning `false`.
   *
   * @tag beforeHooks
   */
  onBeforeItemsPublish: (
    items: Item[],
    ctx: OnBeforeItemsPublishCtx,
  ) => MaybePromise<boolean>;
};

// Context type for this hook
export type OnBeforeItemsPublishCtx = Ctx;

// Base context type
export type Ctx<
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BaseProperties & AdditionalProperties & BaseMethods & AdditionalMethods;

// Utility type for functions that can return promises or direct values
type MaybePromise<T> = T | Promise<T>;

// Item structure - the main record type in DatoCMS
export type Item = {
  type: 'item';
  id: string;
  attributes: {
    // Dynamic field values based on the model configuration
    [fieldApiKey: string]: unknown;
  };
  relationships: {
    // The record's model
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
    // The entity who created the record
    creator?: {
      data: {
        id: string;
        type: 'account' | 'access_token' | 'user' | 'sso_user' | 'organization';
      } | null;
    };
  };
  meta: {
    // Date of creation
    created_at: string;
    // Last update time
    updated_at: string;
    // Date of last publication
    published_at: string | null;
    // Date of first publication
    first_published_at: string | null;
    // Date of future publication
    publication_scheduled_at: string | null;
    // Date of future unpublishing
    unpublishing_scheduled_at: string | null;
    // Status
    status: 'draft' | 'updated' | 'published' | null;
    // Whether the current record is valid or not
    is_valid: boolean;
    // Whether the current version of the record is valid or not
    is_current_version_valid: boolean | null;
    // Whether the published version of record is valid or not
    is_published_version_valid: boolean | null;
    // The ID of the current record version
    current_version: string;
    // Workflow stage in which the item is
    stage: string | null;
  };
};

// Base properties available in all contexts
export type BaseProperties = {
  // Plugin information
  plugin: Plugin;
  
  // Authentication properties
  // The current DatoCMS user (owner or collaborator)
  currentUser: User | SsoUser | Account | Organization;
  // The role for the current DatoCMS user
  currentRole: Role;
  // Access token to perform API calls (if permission granted)
  currentUserAccessToken: string | undefined;
  
  // Project properties
  // The current DatoCMS project
  site: Site;
  // The ID of the current environment
  environment: string;
  // Whether the current environment is the primary one
  isEnvironmentPrimary: boolean;
  // The account/organization that is the project owner
  owner: Account | Organization;
  // Deprecated: use .owner instead
  account: Account | undefined;
  // UI preferences
  ui: {
    locale: string;
  };
  // Theme colors for the current project
  theme: {
    primaryColor: string;
    accentColor: string;
    semiTransparentAccentColor: string;
    lightColor: string;
    darkColor: string;
  };
  
  // Entity repositories (indexed by ID)
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  users: Partial<Record<string, User>>;
  ssoUsers: Partial<Record<string, SsoUser>>;
};

// Base methods available in all contexts
export type BaseMethods = {
  // Load data methods
  loadItemTypeFields: (itemTypeId: string) => Promise<Field[]>;
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<Fieldset[]>;
  loadFieldsUsingPlugin: () => Promise<Field[]>;
  loadUsers: () => Promise<User[]>;
  loadSsoUsers: () => Promise<SsoUser[]>;
  
  // Update plugin parameters methods
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  updateFieldAppearance: (
    fieldId: string,
    changes: FieldAppearanceChange[],
  ) => Promise<void>;
  
  // Item dialog methods
  createNewItem: (itemTypeId: string) => Promise<Item | null>;
  selectItem: {
    (itemTypeId: string, options: { multiple: true; initialLocationQuery?: ItemListLocationQuery }): Promise<Item[] | null>;
    (itemTypeId: string, options?: { multiple: false; initialLocationQuery?: ItemListLocationQuery }): Promise<Item | null>;
  };
  editItem: (itemId: string) => Promise<Item | null>;
  
  // Toast methods
  alert: (message: string) => Promise<void>;
  notice: (message: string) => Promise<void>;
  customToast: <CtaValue = unknown>(toast: Toast<CtaValue>) => Promise<CtaValue | null>;
  
  // Upload dialog methods
  selectUpload: {
    (options: { multiple: true }): Promise<Upload[] | null>;
    (options?: { multiple: false }): Promise<Upload | null>;
  };
  editUpload: (uploadId: string) => Promise<(Upload & { deleted?: true }) | null>;
  editUploadMetadata: (
    fileFieldValue: FileFieldValue,
    locale?: string,
  ) => Promise<FileFieldValue | null>;
  
  // Custom dialog methods
  openModal: (modal: Modal) => Promise<unknown>;
  openConfirm: (options: ConfirmOptions) => Promise<unknown>;
  
  // Navigation methods
  navigateTo: (path: string) => Promise<void>;
};

// Supporting type definitions

// Field appearance change operations
export type FieldAppearanceChange =
  | { operation: 'removeEditor' }
  | { operation: 'updateEditor'; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'setEditor'; fieldExtensionId: string; parameters: Record<string, unknown> }
  | { operation: 'removeAddon'; index: number }
  | { operation: 'updateAddon'; index: number; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'insertAddon'; index: number; fieldExtensionId: string; parameters: Record<string, unknown> };

// Item list query parameters
export type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};

// Toast notification configuration
export type Toast<CtaValue = unknown> = {
  message: string;
  type: 'notice' | 'alert' | 'warning';
  cta?: {
    label: string;
    value: CtaValue;
  };
  dismissOnPageChange?: boolean;
  dismissAfterTimeout?: boolean | number;
};

// Single asset field structure
export type FileFieldValue = {
  upload_id: string;
  alt: string | null;
  title: string | null;
  focal_point: FocalPoint | null;
  custom_data: Record<string, string>;
};

// Modal configuration
export type Modal = {
  id: string;
  title?: string;
  closeDisabled?: boolean;
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  parameters?: Record<string, unknown>;
  initialHeight?: number;
};

// Focal point for images
export type FocalPoint = {
  x: number; // Horizontal position (0-1)
  y: number; // Vertical position (0-1)
};

// Confirmation dialog options
export type ConfirmOptions = {
  title: string;
  content: string;
  choices: ConfirmChoice[];
  cancel: ConfirmChoice;
};

// Choice in confirmation dialog
export type ConfirmChoice = {
  label: string;
  value: unknown;
  intent?: 'positive' | 'negative';
};

// Additional type definitions for comprehensive support

// Plugin entity structure
export type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    package_name: string;
    description: string | null;
    url: string | null;
    homepage: string | null;
    entry_point_url: string;
    permissions: string[];
    version: string;
    parameter_definitions: Record<string, unknown>;
    parameters: Record<string, unknown>;
  };
};

// User entity structure
export type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    is_2fa_active: boolean;
    locale: string;
  };
  relationships: {
    role: {
      data: {
        id: string;
        type: 'role';
      };
    };
  };
};

// SSO User entity structure
export type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
  };
  relationships: {
    role: {
      data: {
        id: string;
        type: 'role';
      };
    };
  };
};

// Role entity structure
export type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_manage_environments: boolean;
    can_manage_build_triggers: boolean;
    can_manage_sso: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_access_site_reports: boolean;
  };
  meta: {
    final_permissions: {
      can_edit_schema: boolean;
      can_manage_environments: boolean;
      can_manage_build_triggers: boolean;
      can_manage_sso: boolean;
      can_access_audit_log: boolean;
      can_manage_webhooks: boolean;
      can_manage_access_tokens: boolean;
      can_perform_site_search: boolean;
      can_access_site_reports: boolean;
      can_manage_build_environments: boolean;
      can_manage_build_environment_variables: boolean;
    };
  };
};

// Site (project) entity structure
export type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    locales: string[];
    timezone: string;
    global_seo: Record<string, unknown> | null;
    favicon: Record<string, unknown> | null;
    no_index: boolean;
    frontend_url: string | null;
    ssg: string | null;
    imgix_host: string | null;
  };
};

// ItemType (model) entity structure
export type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    modular_block: boolean;
    tree: boolean;
    ordering_direction: 'asc' | 'desc' | null;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'table';
    hint: string | null;
  };
  relationships: {
    fields: {
      data: Array<{
        id: string;
        type: 'field';
      }>;
    };
    fieldsets: {
      data: Array<{
        id: string;
        type: 'fieldset';
      }>;
    };
  };
};

// Field entity structure
export type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    api_key: string;
    hint: string | null;
    placeholder: string | null;
    appearance: {
      editor: string;
      parameters: Record<string, unknown>;
      addons: Array<{
        id: string;
        parameters: Record<string, unknown>;
      }>;
    };
    position: number;
    disabled: boolean;
    localized: boolean;
    field_type: string;
    validators: {
      required?: boolean;
      unique?: boolean;
      [key: string]: unknown;
    };
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
  };
};

// Upload (asset) entity structure
export type Upload = {
  id: string;
  type: 'upload';
  attributes: {
    path: string;
    format: string | null;
    size: number;
    width: number | null;
    height: number | null;
    alt: string | null;
    title: string | null;
    custom_data: Record<string, string>;
    focal_point: FocalPoint | null;
    url: string;
    copyright: string | null;
    tags: string[];
    smart_tags: string[];
    filename: string;
    mime_type: string;
    default_field_metadata: {
      [locale: string]: {
        alt: string | null;
        title: string | null;
        custom_data: Record<string, string>;
        focal_point: FocalPoint | null;
      };
    };
  };
};
```

## Return Value

- `true`: Allow the publish operation to proceed
- `false`: Cancel the publish operation

## Use Cases

### 1. Content Quality Validation

Ensure content meets quality standards before publishing:

```typescript
// Helper function for grammar checking
async function checkGrammar(content: string): Promise<string[]> {
  // Example implementation - replace with actual service
  try {
    const response = await fetch('https://api.grammarcheck.com/check', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text: content })
    });
    const result = await response.json();
    return result.issues || [];
  } catch {
    return [];
  }
}

async onBeforeItemsPublish(
  items: Item[],
  ctx: OnBeforeItemsPublishCtx
): Promise<boolean> {
  const issues: string[] = [];
  
  for (const item of items) {
    const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
    
    if (itemType?.attributes.api_key === 'article') {
      const title = item.attributes.title as string;
      const content = item.attributes.content as string;
      const excerpt = item.attributes.excerpt as string;
      const featuredImage = item.attributes.featured_image;
      
      // Check title quality
      if (!title || title.length < 10) {
        issues.push(`Article "${title || 'Untitled'}" has a title that's too short`);
      }
      
      // Check content length
      const wordCount = content ? content.split(/\s+/).filter(Boolean).length : 0;
      if (wordCount < 300) {
        issues.push(`Article "${title}" has only ${wordCount} words (minimum: 300)`);
      }
      
      // Check for excerpt
      if (!excerpt || excerpt.length < 50) {
        issues.push(`Article "${title}" needs a longer excerpt for SEO`);
      }
      
      // Check for featured image
      if (!featuredImage) {
        issues.push(`Article "${title}" is missing a featured image`);
      }
      
      // Check for proper categorization
      if (!item.attributes.category || !item.attributes.tags) {
        issues.push(`Article "${title}" needs proper categorization`);
      }
      
      // Grammar and spell check (using external service)
      if (ctx.plugin.attributes.parameters.grammarCheckEnabled) {
        const grammarIssues = await checkGrammar(content);
        if (grammarIssues.length > 0) {
          issues.push(`Article "${title}" has ${grammarIssues.length} grammar issues`);
        }
      }
    }
    
    if (itemType?.attributes.api_key === 'product') {
      const name = item.attributes.name as string;
      const price = item.attributes.price as number;
      const description = item.attributes.description as string;
      const images = item.attributes.images as any[];
      
      // Product-specific validations
      if (!price || price <= 0) {
        issues.push(`Product "${name}" has invalid pricing`);
      }
      
      if (!images || images.length < 2) {
        issues.push(`Product "${name}" needs at least 2 images`);
      }
      
      if (!description || description.length < 100) {
        issues.push(`Product "${name}" needs a detailed description`);
      }
      
      // Check inventory
      if (item.attributes.track_inventory && !item.attributes.stock_quantity) {
        issues.push(`Product "${name}" has inventory tracking enabled but no stock quantity`);
      }
    }
  }
  
  if (issues.length > 0) {
    const proceed = await ctx.openConfirm({
      title: 'Content Quality Issues',
      content: 
        'The following issues were found:\n\n' +
        issues.map(i => `• ${i}`).join('\n') +
        '\n\nPublish anyway?',
      choices: [
        { label: 'Fix Issues', value: false },
        { label: 'Publish Anyway', value: true, intent: 'negative' }
      ]
    });
    
    return proceed;
  }
  
  return true;
}
```

### 2. Workflow and Approval Requirements

Enforce approval workflows before publication:

```typescript
async onBeforeItemsPublish(
  items: Item[],
  ctx: OnBeforeItemsPublishCtx
): Promise<boolean> {
  const requiresApproval = ['press_release', 'legal_document', 'financial_report'];
  
  for (const item of items) {
    const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
    
    if (requiresApproval.includes(itemType?.attributes.api_key || '')) {
      // Check approval status
      const approvalStatus = item.attributes.approval_status as string;
      const approvedBy = item.attributes.approved_by as string;
      const approvalDate = item.attributes.approval_date as string;
      
      if (approvalStatus !== 'approved') {
        await ctx.alert(
          `${itemType?.attributes.name} "${item.attributes.title}" requires approval before publishing.\n\n` +
          `Current status: ${approvalStatus || 'Not submitted'}`
        );
        return false;
      }
      
      // Verify approval is recent (within 30 days)
      if (approvalDate) {
        const daysSinceApproval = Math.floor(
          (Date.now() - new Date(approvalDate).getTime()) / (1000 * 60 * 60 * 24)
        );
        
        if (daysSinceApproval > 30) {
          const proceed = await ctx.openConfirm({
            title: 'Stale Approval',
            content: 
              `This ${itemType?.attributes.name} was approved ${daysSinceApproval} days ago.\n` +
              'It may need re-approval. Continue publishing?',
            choices: [
              { label: 'Request Re-approval', value: false },
              { label: 'Publish', value: true, intent: 'negative' }
            ]
          });
          
          if (!proceed) {
            // Reset approval status
            await ctx.updateItem(item.id, {
              attributes: {
                approval_status: 'pending_reapproval'
              }
            });
            return false;
          }
        }
      }
      
      // Check approver permissions
      if (!ctx.currentRole.attributes.can_publish_content && approvedBy !== ctx.currentUser.id) {
        await ctx.alert(
          'You don\'t have permission to publish content approved by others.\n' +
          'Please ask an administrator to publish this item.'
        );
        return false;
      }
    }
  }
  
  return true;
}
```

### 3. SEO and Marketing Requirements

Ensure SEO compliance before publishing:

```typescript
async onBeforeItemsPublish(
  items: Item[],
  ctx: OnBeforeItemsPublishCtx
): Promise<boolean> {
  const seoRequiredModels = ['article', 'product', 'landing_page', 'case_study'];
  const seoIssues: Record<string, string[]> = {};
  
  for (const item of items) {
    const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
    
    if (seoRequiredModels.includes(itemType?.attributes.api_key || '')) {
      const issues: string[] = [];
      const title = item.attributes.title || item.attributes.name;
      
      // Check SEO title
      const seoTitle = item.attributes.seo_title as string;
      if (!seoTitle) {
        issues.push('Missing SEO title');
      } else if (seoTitle.length < 30 || seoTitle.length > 60) {
        issues.push(`SEO title length: ${seoTitle.length} (should be 30-60)`);
      }
      
      // Check meta description
      const metaDescription = item.attributes.meta_description as string;
      if (!metaDescription) {
        issues.push('Missing meta description');
      } else if (metaDescription.length < 120 || metaDescription.length > 160) {
        issues.push(`Meta description length: ${metaDescription.length} (should be 120-160)`);
      }
      
      // Check URL slug
      const slug = item.attributes.slug as string;
      if (!slug) {
        issues.push('Missing URL slug');
      } else {
        // Check for SEO-friendly URL
        if (slug.includes('_') || slug.includes(' ') || /[A-Z]/.test(slug)) {
          issues.push('URL slug should use hyphens and lowercase only');
        }
        
        // Check slug uniqueness
        const duplicates = await ctx.searchItems({
          filter: {
            type: itemType.attributes.api_key,
            fields: { slug: { eq: slug } },
            id: { neq: item.id }
          }
        });
        
        if (duplicates.length > 0) {
          issues.push('URL slug is already in use');
        }
      }
      
      // Check Open Graph data
      if (!item.attributes.og_image && !item.attributes.featured_image) {
        issues.push('Missing Open Graph image');
      }
      
      // Check structured data requirements
      if (itemType.attributes.api_key === 'product') {
        if (!item.attributes.sku) issues.push('Missing SKU for structured data');
        if (!item.attributes.brand) issues.push('Missing brand information');
        if (!item.attributes.availability) issues.push('Missing availability status');
      }
      
      if (issues.length > 0) {
        seoIssues[`${title} (${item.id})`] = issues;
      }
    }
  }
  
  if (Object.keys(seoIssues).length > 0) {
    const report = Object.entries(seoIssues)
      .map(([item, issues]) => `${item}:\n${issues.map(i => `  • ${i}`).join('\n')}`)
      .join('\n\n');
    
    const proceed = await ctx.openConfirm({
      title: 'SEO Issues Detected',
      content: `The following SEO issues were found:\n\n${report}\n\nPublish anyway?`,
      choices: [
        { label: 'Fix SEO Issues', value: false },
        { label: 'Publish with Issues', value: true, intent: 'negative' }
      ]
    });
    
    return proceed;
  }
  
  return true;
}
```

### 4. Scheduled Publishing Validation

Validate scheduled publishing constraints:

```typescript
async onBeforeItemsPublish(
  items: Item[],
  ctx: OnBeforeItemsPublishCtx
): Promise<boolean> {
  for (const item of items) {
    const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
    
    // Check embargo dates
    if (item.attributes.embargo_until) {
      const embargoDate = new Date(item.attributes.embargo_until as string);
      
      if (embargoDate > new Date()) {
        await ctx.alert(
          `"${item.attributes.title}" is embargoed until ${embargoDate.toLocaleDateString()}.\n` +
          'Publishing before this date is not allowed.'
        );
        return false;
      }
    }
    
    // Check publication windows
    if (itemType?.attributes.api_key === 'announcement') {
      const now = new Date();
      const dayOfWeek = now.getDay();
      const hour = now.getHours();
      
      // Only allow publishing on weekdays during business hours
      if (dayOfWeek === 0 || dayOfWeek === 6) {
        const override = await ctx.openConfirm({
          title: 'Weekend Publishing',
          content: 'Announcements are typically published on weekdays. Publish on weekend?',
          choices: [
            { label: 'Wait for Monday', value: false },
            { label: 'Publish Now', value: true, intent: 'negative' }
          ]
        });
        
        if (!override) return false;
      }
      
      if (hour < 9 || hour > 17) {
        const override = await ctx.openConfirm({
          title: 'Off-Hours Publishing',
          content: 'Publishing outside business hours may reduce visibility. Continue?',
          choices: [
            { label: 'Schedule for Tomorrow', value: false },
            { label: 'Publish Now', value: true }
          ]
        });
        
        if (!override) return false;
      }
    }
    
    // Check content freshness
    const lastModified = new Date(item.meta.updated_at);
    const daysSinceUpdate = Math.floor(
      (Date.now() - lastModified.getTime()) / (1000 * 60 * 60 * 24)
    );
    
    if (daysSinceUpdate > 180) {
      const proceed = await ctx.openConfirm({
        title: 'Stale Content Warning',
        content: 
          `"${item.attributes.title}" hasn't been updated in ${daysSinceUpdate} days.\n` +
          'Consider reviewing the content before publishing.',
        choices: [
          { label: 'Review First', value: false },
          { label: 'Publish As-Is', value: true, intent: 'negative' }
        ]
      });
      
      if (!proceed) return false;
    }
  }
  
  return true;
}
```

### 5. Multi-locale Publishing Requirements

Ensure content is ready across all required locales:

```typescript
// Helper function for translation quality checking
interface TranslationQuality {
  score: number;
  issues?: string[];
}

async function checkTranslationQuality(
  attributes: Record<string, unknown>,
  sourceLocale: string,
  targetLocale: string
): Promise<TranslationQuality> {
  // Example implementation - replace with actual translation service
  try {
    const localizedFields = Object.entries(attributes)
      .filter(([_, value]) => 
        typeof value === 'object' && 
        value !== null &&
        sourceLocale in value &&
        targetLocale in value
      );
    
    // Simple quality check based on field completion
    const completedFields = localizedFields.filter(([_, value]) => {
      const source = (value as any)[sourceLocale];
      const target = (value as any)[targetLocale];
      return source && target && target.length > 0;
    });
    
    const score = localizedFields.length > 0 
      ? completedFields.length / localizedFields.length 
      : 1;
    
    return { score };
  } catch {
    return { score: 1 }; // Assume quality is good if check fails
  }
}

async onBeforeItemsPublish(
  items: Item[],
  ctx: OnBeforeItemsPublishCtx
): Promise<boolean> {
  const requiredLocales = ctx.site.attributes.locales;
  const primaryLocale = requiredLocales[0];
  const localizationIssues: Record<string, string[]> = {};
  
  for (const item of items) {
    const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
    
    // Skip if model doesn't require full localization
    if (!itemType?.attributes.all_locales_required) {
      continue;
    }
    
    const missingLocales: string[] = [];
    const incompleteLocales: string[] = [];
    
    // Check each localized field
    const localizedFields = Object.values(ctx.fields).filter(
      f => f.attributes.localized && 
      f.relationships.item_type.data.id === itemType.id
    );
    
    for (const locale of requiredLocales) {
      let hasContent = false;
      let isComplete = true;
      
      for (const field of localizedFields) {
        const fieldValue = item.attributes[field.attributes.api_key];
        
        if (fieldValue && typeof fieldValue === 'object') {
          if (fieldValue[locale]) {
            hasContent = true;
          } else if (field.attributes.validators.required) {
            isComplete = false;
          }
        }
      }
      
      if (!hasContent) {
        missingLocales.push(locale);
      } else if (!isComplete) {
        incompleteLocales.push(locale);
      }
    }
    
    // Check translation quality
    if (ctx.plugin.attributes.parameters.translationQualityCheck) {
      for (const locale of requiredLocales) {
        if (locale === primaryLocale) continue;
        
        const quality = await checkTranslationQuality(
          item.attributes,
          primaryLocale,
          locale
        );
        
        if (quality.score < 0.7) {
          incompleteLocales.push(`${locale} (low quality: ${Math.round(quality.score * 100)}%)`);
        }
      }
    }
    
    if (missingLocales.length > 0 || incompleteLocales.length > 0) {
      const issues = [];
      if (missingLocales.length > 0) {
        issues.push(`Missing translations: ${missingLocales.join(', ')}`);
      }
      if (incompleteLocales.length > 0) {
        issues.push(`Incomplete translations: ${incompleteLocales.join(', ')}`);
      }
      
      localizationIssues[item.attributes.title || item.id] = issues;
    }
  }
  
  if (Object.keys(localizationIssues).length > 0) {
    const report = Object.entries(localizationIssues)
      .map(([item, issues]) => `${item}:\n${issues.map(i => `  • ${i}`).join('\n')}`)
      .join('\n\n');
    
    const proceed = await ctx.openConfirm({
      title: 'Incomplete Translations',
      content: `The following items have translation issues:\n\n${report}\n\nPublish anyway?`,
      choices: [
        { label: 'Complete Translations', value: false },
        { label: 'Publish Incomplete', value: true, intent: 'negative' }
      ]
    });
    
    return proceed;
  }
  
  return true;
}
```

### 6. External System Synchronization

Validate with external systems before publishing:

```typescript
// Helper types for external system integration
interface ExternalSystemResponse {
  success: boolean;
  id?: string;
  error?: string;
}

interface ExternalValidation {
  valid: boolean;
  differences: string[];
}

// Helper function to create record in external system
async function createInExternalSystem(
  item: Item,
  ctx: OnBeforeItemsPublishCtx
): Promise<ExternalSystemResponse> {
  try {
    const response = await fetch(`${ctx.plugin.attributes.parameters.externalApiUrl}/records`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${ctx.plugin.attributes.parameters.externalApiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        datocmsId: item.id,
        type: item.relationships.item_type.data.id,
        data: item.attributes
      })
    });
    
    if (!response.ok) {
      const error = await response.text();
      return { success: false, error };
    }
    
    const result = await response.json();
    return { success: true, id: result.id };
  } catch (error) {
    return { success: false, error: (error as Error).message };
  }
}

// Helper function to validate external record
async function validateExternalRecord(
  externalId: string,
  item: Item,
  ctx: OnBeforeItemsPublishCtx
): Promise<ExternalValidation> {
  try {
    const response = await fetch(`${ctx.plugin.attributes.parameters.externalApiUrl}/records/${externalId}/validate`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${ctx.plugin.attributes.parameters.externalApiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        datocmsData: item.attributes
      })
    });
    
    if (!response.ok) {
      return { valid: false, differences: ['External system unavailable'] };
    }
    
    const result = await response.json();
    return result;
  } catch {
    return { valid: false, differences: ['Cannot connect to external system'] };
  }
}

async onBeforeItemsPublish(
  items: Item[],
  ctx: OnBeforeItemsPublishCtx
): Promise<boolean> {
  const syncRequired = ['product', 'event', 'course'];
  
  for (const item of items) {
    const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
    
    if (syncRequired.includes(itemType?.attributes.api_key || '')) {
      try {
        // Check external system readiness
        const externalId = item.attributes.external_id as string;
        
        if (!externalId) {
          // Create in external system first
          const externalData = await createInExternalSystem(item, ctx);
          
          if (!externalData.success) {
            await ctx.alert(
              `Failed to create in external system: ${externalData.error}\n\n` +
              'Publishing cancelled.'
            );
            return false;
          }
          
          // Update item with external ID
          await ctx.updateItem(item.id, {
            attributes: {
              external_id: externalData.id,
              external_sync_status: 'synced',
              external_sync_date: new Date().toISOString()
            }
          });
        } else {
          // Validate existing external record
          const validation = await validateExternalRecord(externalId, item, ctx);
          
          if (!validation.valid) {
            const proceed = await ctx.openConfirm({
              title: 'External System Out of Sync',
              content: 
                `The external system has different data:\n\n` +
                validation.differences.map(d => `• ${d}`).join('\n') +
                '\n\nPublish anyway?',
              choices: [
                { label: 'Sync First', value: false },
                { label: 'Publish', value: true, intent: 'negative' }
              ]
            });
            
            if (!proceed) {
              return false;
            }
          }
        }
      } catch (error) {
        await ctx.alert(
          `External system error: ${error.message}\n\n` +
          'Please try again later or contact support.'
        );
        return false;
      }
    }
  }
  
  return true;
}
```

## Best Practices

1. **Batch Validation**: Process all items before showing errors to avoid multiple dialogs
2. **Clear Messaging**: Explain why publication is being prevented
3. **Offer Solutions**: Provide options to fix issues or override with caution
4. **Performance**: Keep validations fast, especially for bulk publishing
5. **Progressive Enhancement**: Allow overrides for non-critical issues
6. **Audit Trail**: Log publication attempts and validation results
7. **Respect Workflows**: Integrate with existing approval processes

## Related Hooks

- **onBeforeItemUpsert**: Validate individual item saves
- **onBeforeItemsUnpublish**: Validate before unpublishing
- **onBeforeItemsDestroy**: Validate before deletion
- **itemsDropdownActions**: Add bulk publishing actions

## Important Notes

- This hook runs for both single and bulk publish operations
- Returning `false` cancels the entire publish operation
- The hook cannot selectively publish some items and not others
- Consider caching validation results for performance
- External API calls should have timeouts to prevent blocking
- Publishing cannot be partially completed - it's all or nothing