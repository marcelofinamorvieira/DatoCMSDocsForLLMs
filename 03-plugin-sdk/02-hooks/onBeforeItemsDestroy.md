# onBeforeItemsDestroy Hook

The `onBeforeItemsDestroy` hook is triggered before one or more records are deleted, allowing plugins to validate the deletion, check for dependencies, or prevent the operation entirely. This hook is essential for maintaining referential integrity and preventing accidental data loss.

## Quick Reference

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  onBeforeItemsDestroy(items, ctx) {
    // Validate deletion
    if (hasActiveDependencies(items)) {
      ctx.alert('Cannot delete items with dependencies');
      return false; // Prevent deletion
    }
    return true; // Allow deletion
  }
});
```

## Purpose

Use this hook to:
- Prevent deletion of records with dependencies
- Enforce business rules around deletions
- Implement soft delete/archive patterns
- Create backups before deletion
- Require approval or reasons for critical deletions
- Check cascading delete impacts
- Maintain referential integrity

## Signature

```typescript
// The hook function signature
onBeforeItemsDestroy(
  items: Item[],
  ctx: OnBeforeItemsDestroyCtx
): MaybePromise<boolean>

// Used in plugin definition
connect({
  async onBeforeItemsDestroy(items, ctx): Promise<boolean> {
    // Check if deletion is allowed
    if (hasActiveDependencies(items)) {
      ctx.alert('Cannot delete items with active dependencies');
      return false; // Prevent deletion
    }
    return true; // Allow deletion
  }
});
```

## Parameters

### items: Item[]

Array of records that are about to be deleted:

```typescript
interface Item {
  id: string;
  type: 'item';
  attributes: Record<string, any>;           // All field values
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
    creator?: {
      data: {
        id: string;
        type: 'user' | 'account' | 'sso_user';
      } | null;
    };
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at?: string | null;
    publication_scheduled_at?: string | null;
    unpublishing_scheduled_at?: string | null;
    status: 'draft' | 'published' | 'updated';
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    current_version: string;
  };
}
```

### ctx: OnBeforeItemsDestroyCtx

The context object providing access to DatoCMS data and methods. This type extends the base `Ctx` type.

## Type Definitions

### Hook Type Definition

```typescript
// From packages/sdk/src/hooks/onBeforeItemsDestroy.ts
import type { SchemaTypes } from '@datocms/cma-client';
import { Ctx } from '../ctx/base';
import { MaybePromise } from '../utils';

type Item = SchemaTypes.Item;

export type OnBeforeItemsDestroyHook = {
  /**
   * This function will be called before destroying records. You can stop the
   * action by returning `false`
   *
   * @tag beforeHooks
   */
  onBeforeItemsDestroy: (
    items: Item[],
    ctx: OnBeforeItemsDestroyCtx,
  ) => MaybePromise<boolean>;
};

export type OnBeforeItemsDestroyCtx = Ctx;
```

### Complete Context Type Expansion

```typescript
// OnBeforeItemsDestroyCtx expands to:
type OnBeforeItemsDestroyCtx = BaseProperties & BaseMethods;

// BaseProperties includes:
type BaseProperties = PluginProperties &
  AuthenticationProperties &
  ProjectProperties &
  EntityReposProperties;

// BaseMethods includes:
type BaseMethods = LoadDataMethods &
  UpdatePluginParametersMethods &
  ToastMethods &
  ItemDialogMethods &
  UploadDialogMethods &
  CustomDialogMethods &
  NavigateMethods;
```

### Full Context Interface

```typescript
interface OnBeforeItemsDestroyCtx {
  // Plugin Properties
  /** The current plugin */
  plugin: Plugin;

  // Authentication Properties
  /**
   * The current DatoCMS user. It can either be the owner or one of the
   * collaborators (regular or SSO).
   */
  currentUser: User | SsoUser | Account | Organization;
  /** The role for the current DatoCMS user */
  currentRole: Role;
  /**
   * The access token to perform API calls on behalf of the current user. Only
   * available if `currentUserAccessToken` additional permission is granted
   */
  currentUserAccessToken: string | undefined;

  // Project Properties
  /** The current DatoCMS project */
  site: Site;
  /** The ID of the current environment */
  environment: string;
  /** Whether the current environment is the primary one */
  isEnvironmentPrimary: boolean;
  /** The account/organization that is the project owner */
  owner: Account | Organization;
  /**
   * The account that is the project owner
   * @deprecated Please use `.owner` instead, as the project owner can also be
   *   an organization
   */
  account: Account | undefined;
  /**
   * UI preferences of the current user (right now, only the preferred locale is
   * available)
   */
  ui: {
    /** Preferred locale */
    locale: string;
  };
  /** An object containing the theme colors for the current DatoCMS project */
  theme: Theme;

  // Entity Repositories Properties
  /** All the models of the current DatoCMS project, indexed by ID */
  itemTypes: Partial<Record<string, ItemType>>;
  /**
   * All the fields currently loaded for the current DatoCMS project, indexed by
   * ID. If some fields you need are not present, use the `loadItemTypeFields`
   * function to load them.
   */
  fields: Partial<Record<string, Field>>;
  /**
   * All the fieldsets currently loaded for the current DatoCMS project, indexed
   * by ID. If some fields you need are not present, use the
   * `loadItemTypeFieldsets` function to load them.
   */
  fieldsets: Partial<Record<string, Fieldset>>;
  /**
   * All the regular users currently loaded for the current DatoCMS project,
   * indexed by ID. It will always contain the current user. If some users you
   * need are not present, use the `loadUsers` function to load them.
   */
  users: Partial<Record<string, User>>;
  /**
   * All the SSO users currently loaded for the current DatoCMS project, indexed
   * by ID. It will always contain the current user. If some users you need are
   * not present, use the `loadSsoUsers` function to load them.
   */
  ssoUsers: Partial<Record<string, SsoUser>>;

  // Load Data Methods
  /**
   * Loads all the fields for a specific model (or block). Fields will be
   * returned and will also be available in the the `fields` property.
   */
  loadItemTypeFields: (itemTypeId: string) => Promise<Field[]>;
  /**
   * Loads all the fieldsets for a specific model (or block). Fieldsets will be
   * returned and will also be available in the the `fieldsets` property.
   */
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<Fieldset[]>;
  /**
   * Loads all the fields in the project that are currently using the plugin for
   * one of its manual field extensions.
   */
  loadFieldsUsingPlugin: () => Promise<Field[]>;
  /**
   * Loads all regular users. Users will be returned and will also be available
   * in the the `users` property.
   */
  loadUsers: () => Promise<User[]>;
  /**
   * Loads all SSO users. Users will be returned and will also be available in
   * the the `ssoUsers` property.
   */
  loadSsoUsers: () => Promise<SsoUser[]>;

  // Update Plugin Parameters Methods
  /**
   * Updates the plugin parameters.
   *
   * Always check `ctx.currentRole.meta.final_permissions.can_edit_schema`
   * before calling this, as the user might not have the permission to perform
   * the operation.
   */
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  /**
   * Performs changes in the appearance of a field. You can install/remove a
   * manual field extension, or tweak their parameters. If multiple changes are
   * passed, they will be applied sequencially.
   *
   * Always check `ctx.currentRole.meta.final_permissions.can_edit_schema`
   * before calling this, as the user might not have the permission to perform
   * the operation.
   */
  updateFieldAppearance: (
    fieldId: string,
    changes: FieldAppearanceChange[],
  ) => Promise<void>;

  // Toast Methods
  /**
   * Triggers an "error" toast displaying the selected message
   */
  alert: (message: string) => Promise<void>;
  /**
   * Triggers a "success" toast displaying the selected message
   */
  notice: (message: string) => Promise<void>;
  /**
   * Triggers a custom toast displaying the selected message (and optionally a
   * CTA)
   */
  customToast: <CtaValue = unknown>(
    toast: Toast<CtaValue>,
  ) => Promise<CtaValue | null>;

  // Item Dialog Methods
  /**
   * Opens a dialog for creating a new record. It returns a promise resolved
   * with the newly created record or `null` if the user closes the dialog
   * without creating anything.
   */
  createNewItem: (itemTypeId: string) => Promise<Item | null>;
  /**
   * Opens a dialog for selecting one (or multiple) record(s) from a list of
   * existing records of type `itemTypeId`. It returns a promise resolved with
   * the selected record(s), or `null` if the user closes the dialog without
   * choosing any record.
   */
  selectItem: {
    (
      itemTypeId: string,
      options: { multiple: true; initialLocationQuery?: ItemListLocationQuery },
    ): Promise<Item[] | null>;
    (
      itemTypeId: string,
      options?: {
        multiple: false;
        initialLocationQuery?: ItemListLocationQuery;
      },
    ): Promise<Item | null>;
  };
  /**
   * Opens a dialog for editing an existing record. It returns a promise
   * resolved with the edited record, or `null` if the user closes the dialog
   * without persisting any change.
   */
  editItem: (itemId: string) => Promise<Item | null>;

  // Upload Dialog Methods
  /**
   * Opens a dialog for selecting one (or multiple) existing asset(s). It
   * returns a promise resolved with the selected asset(s), or `null` if the
   * user closes the dialog without selecting any upload.
   */
  selectUpload: {
    (options: { multiple: true }): Promise<Upload[] | null>;
    (options?: { multiple: false }): Promise<Upload | null>;
  };
  /**
   * Opens a dialog for editing a Media Area asset. It returns a promise
   * resolved with:
   *
   * - The updated asset, if the user persists some changes to the asset itself
   * - `null`, if the user closes the dialog without persisting any change
   * - An asset structure with an additional `deleted` property set to true, if
   *   the user deletes the asset
   */
  editUpload: (
    uploadId: string,
  ) => Promise<(Upload & { deleted?: true }) | null>;
  /**
   * Opens a dialog for editing a "single asset" field structure. It returns a
   * promise resolved with the updated structure, or `null` if the user closes
   * the dialog without persisting any change.
   */
  editUploadMetadata: (
    /** The "single asset" field structure */
    fileFieldValue: FileFieldValue,
    /** Shows metadata information for a specific locale */
    locale?: string,
  ) => Promise<FileFieldValue | null>;

  // Custom Dialog Methods
  /**
   * Opens a custom modal. Returns a promise resolved with what the modal itself
   * returns calling the `resolve()` function
   */
  openModal: (modal: Modal) => Promise<unknown>;
  /**
   * Opens a UI-consistent confirmation dialog. Returns a promise resolved with
   * the value of the choice made by the user
   */
  openConfirm: (options: ConfirmOptions) => Promise<unknown>;

  // Navigate Methods
  /**
   * Moves the user to another URL internal to the backend
   */
  navigateTo: (path: string) => Promise<void>;
}
```

### Supporting Type Definitions

```typescript
/** An object containing the theme colors for the current DatoCMS project */
type Theme = {
  primaryColor: string;
  accentColor: string;
  semiTransparentAccentColor: string;
  lightColor: string;
  darkColor: string;
};

type FieldAppearanceChange =
  | {
      operation: 'removeEditor';
    }
  | {
      operation: 'updateEditor';
      newFieldExtensionId?: string;
      newParameters?: Record<string, unknown>;
    }
  | {
      operation: 'setEditor';
      fieldExtensionId: string;
      parameters: Record<string, unknown>;
    }
  | {
      operation: 'removeAddon';
      index: number;
    }
  | {
      operation: 'updateAddon';
      index: number;
      newFieldExtensionId?: string;
      newParameters?: Record<string, unknown>;
    }
  | {
      operation: 'insertAddon';
      index: number;
      fieldExtensionId: string;
      parameters: Record<string, unknown>;
    };

type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};

/** A toast notification to present to the user */
type Toast<CtaValue = unknown> = {
  /** Message of the notification */
  message: string;
  /** Type of notification. Will present the toast in a different color accent. */
  type: 'notice' | 'alert' | 'warning';
  /** An optional button to show inside the toast */
  cta?: {
    /** Label for the button */
    label: string;
    /**
     * The value to be returned by the `customToast` promise if the button is
     * clicked by the user
     */
    value: CtaValue;
  };
  /** Whether the toast is to be automatically closed if the user changes page */
  dismissOnPageChange?: boolean;
  /**
   * Whether the toast is to be automatically closed after some time (`true`
   * will use the default DatoCMS time interval)
   */
  dismissAfterTimeout?: boolean | number;
};

/** The structure contained in a "single asset" field */
type FileFieldValue = {
  /** ID of the asset */
  // eslint-disable-next-line camelcase
  upload_id: string;
  /** Alternate text for the asset */
  alt: string | null;
  /** Title for the asset */
  title: string | null;
  /** Focal point of an asset */
  // eslint-disable-next-line camelcase
  focal_point: FocalPoint | null;
  /** Object with arbitrary metadata related to the asset */
  // eslint-disable-next-line camelcase
  custom_data: Record<string, string>;
};

/** A modal to present to the user */
type Modal = {
  /** ID of the modal. Will be the first argument for the `renderModal` function */
  id: string;
  /** Title for the modal. Ignored by `fullWidth` modals */
  title?: string;
  /** Whether to present a close button for the modal or not */
  closeDisabled?: boolean;
  /** Width of the modal. Can be a number, or one of the predefined sizes */
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  /**
   * An arbitrary configuration object that will be passed as the `parameters`
   * property of the second argument of the `renderModal` function
   */
  parameters?: Record<string, unknown>;
  /** The initial height to set for the iframe that will render the modal content */
  initialHeight?: number;
};

/** Focal point of an image asset */
type FocalPoint = {
  /** Horizontal position expressed as float between 0 and 1 */
  x: number;
  /** Vertical position expressed as float between 0 and 1 */
  y: number;
};

/** Options for the `openConfirm` function */
type ConfirmOptions = {
  /** The title to be shown inside the confirmation panel */
  title: string;
  /** The main message to be shown inside the confirmation panel */
  content: string;
  /** The different options the user can choose from */
  choices: ConfirmChoice[];
  /** The cancel option to present to the user */
  cancel: ConfirmChoice;
};

/** A choice presented in a `openConfirm` panel */
type ConfirmChoice = {
  /** The label to be shown for the choice */
  label: string;
  /**
   * The value to be returned by the `openConfirm` promise if the button is
   * clicked by the user
   */
  value: unknown;
  /**
   * The intent of the button. Will present the button in a different color
   * accent.
   */
  intent?: 'positive' | 'negative';
};

// Utility type from packages/sdk/src/utils.ts
type MaybePromise<T> = T | Promise<T>;
```

## Return Value

- `true` - Allow the deletion to proceed
- `false` - Cancel the deletion operation (show error messages first)

## Complete Examples

### 1. Referential Integrity Protection

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsDestroyCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsDestroy(
    items: Item[],
    ctx: OnBeforeItemsDestroyCtx
  ): Promise<boolean> {
    const client = ctx.createClient();
    
    for (const item of items) {
      const modelId = item.relationships.item_type.data.id;
      const itemType = ctx.itemTypes.find(it => it.id === modelId);
      
      if (!itemType) continue;
      
      // Check category dependencies
      if (itemType.attributes.api_key === 'category') {
        // Find all products using this category
        const products = await client.items.list({
          filter: {
            type: 'product',
            fields: {
              category: {
                eq: item.id
              }
            }
          }
        });
        
        if (products.length > 0) {
          // Build helpful error message
          const productList = products
            .slice(0, 5)
            .map(p => `• ${p.attributes.name}`)
            .join('\n');
          
          const moreText = products.length > 5 
            ? `\n\n...and ${products.length - 5} more products` 
            : '';
          
          await ctx.alert(
            `Cannot delete category "${item.attributes.name}".\n\n` +
            `It is currently used by:\n${productList}${moreText}\n\n` +
            `Please reassign these products to another category first.`
          );
          
          return false;
        }
      }
      
      // Check author dependencies with options
      if (itemType.attributes.api_key === 'author') {
        const articles = await client.items.list({
          filter: {
            type: 'article',
            fields: {
              author: {
                eq: item.id
              }
            }
          }
        });
        
        if (articles.length > 0) {
          const action = await ctx.openConfirm({
            title: 'Author Has Articles',
            content: 
              `${item.attributes.name} has written ${articles.length} article(s).\n\n` +
              `What would you like to do with these articles?`,
            choices: [
              { 
                label: 'Cancel Deletion', 
                value: 'cancel',
                intent: 'positive'
              },
              { 
                label: 'Delete Author Only (Orphan Articles)', 
                value: 'orphan',
                intent: 'negative'
              },
              { 
                label: `Delete Author and All ${articles.length} Articles`, 
                value: 'cascade',
                intent: 'negative'
              }
            ]
          });
          
          switch (action) {
            case 'cancel':
              return false;
              
            case 'orphan':
              // Update articles to remove author reference
              for (const article of articles) {
                await client.items.update(article.id, {
                  author: null
                });
              }
              ctx.notice('Articles have been orphaned');
              break;
              
            case 'cascade':
              // Delete all articles
              await client.items.bulkDestroy({
                items: articles.map(a => ({ id: a.id }))
              });
              ctx.notice(`Deleted ${articles.length} articles`);
              break;
          }
        }
      }
      
      // Check for circular dependencies
      if (itemType.attributes.api_key === 'page') {
        // Check if page has children
        const children = await client.items.list({
          filter: {
            type: 'page',
            fields: {
              parent_page: {
                eq: item.id
              }
            }
          }
        });
        
        if (children.length > 0) {
          await ctx.alert(
            `Cannot delete page "${item.attributes.title}".\n\n` +
            `It has ${children.length} child page(s). ` +
            `Delete or reassign child pages first.`
          );
          return false;
        }
      }
    }
    
    return true;
  }
});
```

### 2. Business Rule Enforcement

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsDestroyCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsDestroy(
    items: Item[],
    ctx: OnBeforeItemsDestroyCtx
  ): Promise<boolean> {
    // Check permissions for critical models
    const criticalModels = ['contract', 'invoice', 'legal_document', 'financial_record'];
    const criticalItems = items.filter(item => {
      const itemType = ctx.itemTypes.find(
        it => it.id === item.relationships.item_type.data.id
      );
      return criticalModels.includes(itemType?.attributes.api_key || '');
    });
    
    if (criticalItems.length > 0) {
      // Check user role
      if (!ctx.currentRole.attributes.can_manage_environments) {
        await ctx.alert(
          'Insufficient Permissions\n\n' +
          'Only administrators can delete contracts, invoices, or financial records.\n' +
          'Please contact your system administrator.'
        );
        return false;
      }
      
      // Require deletion reason for audit trail
      const reason = await ctx.openModal('deletion-reason', {
        title: 'Deletion Reason Required',
        width: 'm',
        parameters: {
          items: criticalItems.map(item => {
            const itemType = ctx.itemTypes.find(
              it => it.id === item.relationships.item_type.data.id
            );
            return {
              id: item.id,
              type: itemType?.attributes.name,
              name: item.attributes.name || item.attributes.title || item.id
            };
          }),
          requireApproval: criticalItems.length > 5
        }
      }) as { reason: string; approved: boolean } | null;
      
      if (!reason) {
        return false; // User cancelled
      }
      
      if (!reason.approved && criticalItems.length > 5) {
        await ctx.alert(
          'Bulk deletion of critical records requires additional approval.\n' +
          'Please contact your supervisor.'
        );
        return false;
      }
      
      // Log the deletion attempt
      await logCriticalDeletion(criticalItems, reason.reason, ctx);
    }
    
    // Prevent deletion of active/published items
    const activeItems = items.filter(item => {
      const itemType = ctx.itemTypes.find(
        it => it.id === item.relationships.item_type.data.id
      );
      
      // Check different "active" conditions per model
      switch (itemType?.attributes.api_key) {
        case 'subscription':
          return item.attributes.status === 'active';
        case 'employee':
          return item.attributes.employment_status === 'active';
        case 'campaign':
          return item.attributes.is_running === true;
        case 'service':
          return item.attributes.is_enabled === true;
        default:
          // Check if item is published
          return item.meta.status === 'published';
      }
    });
    
    if (activeItems.length > 0) {
      const itemDescriptions = activeItems.map(item => {
        const itemType = ctx.itemTypes.find(
          it => it.id === item.relationships.item_type.data.id
        );
        const name = item.attributes.name || item.attributes.title || item.id;
        return `• ${itemType?.attributes.name}: ${name}`;
      }).join('\n');
      
      await ctx.alert(
        'Cannot Delete Active Items\n\n' +
        'The following items are currently active:\n' +
        `${itemDescriptions}\n\n` +
        'Please deactivate or unpublish them before deletion.'
      );
      
      return false;
    }
    
    // Check for time-based restrictions
    const timeRestrictedModels = ['promotion', 'event', 'announcement'];
    const currentDate = new Date();
    
    for (const item of items) {
      const itemType = ctx.itemTypes.find(
        it => it.id === item.relationships.item_type.data.id
      );
      
      if (timeRestrictedModels.includes(itemType?.attributes.api_key || '')) {
        const startDate = item.attributes.start_date ? new Date(item.attributes.start_date) : null;
        const endDate = item.attributes.end_date ? new Date(item.attributes.end_date) : null;
        
        // Can't delete ongoing items
        if (startDate && endDate && currentDate >= startDate && currentDate <= endDate) {
          await ctx.alert(
            `Cannot delete active ${itemType?.attributes.name}.\n\n` +
            `"${item.attributes.name}" is currently running.\n` +
            `It will end on ${endDate.toLocaleDateString()}.`
          );
          return false;
        }
        
        // Warn about future items
        if (startDate && currentDate < startDate) {
          const proceed = await ctx.openConfirm({
            title: 'Delete Scheduled Item?',
            content: 
              `"${item.attributes.name}" is scheduled to start on ${startDate.toLocaleDateString()}.\n\n` +
              'Are you sure you want to delete it?',
            choices: [
              { label: 'Keep Item', value: false },
              { label: 'Delete Anyway', value: true, intent: 'negative' }
            ]
          });
          
          if (!proceed) {
            return false;
          }
        }
      }
    }
    
    return true;
  }
});

// Helper function to log deletions
async function logCriticalDeletion(
  items: Item[],
  reason: string,
  ctx: OnBeforeItemsDestroyCtx
): Promise<void> {
  const client = ctx.createClient();
  
  // Create audit log entry
  await client.items.create({
    itemType: 'audit_log',
    action: 'delete',
    user: ctx.currentUser?.id,
    timestamp: new Date().toISOString(),
    reason,
    environment: ctx.environment,
    affected_items: items.map(item => ({
      id: item.id,
      type: item.relationships.item_type.data.id,
      name: item.attributes.name || item.attributes.title || item.id,
      data_snapshot: JSON.stringify(item.attributes)
    }))
  });
}
```

### 3. Cascade Deletion Management

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsDestroyCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsDestroy(
    items: Item[],
    ctx: OnBeforeItemsDestroyCtx
  ): Promise<boolean> {
    // Define cascade relationships
    const cascadeDefinitions: Record<string, Array<{
      model: string;
      field: string;
      label: string;
    }>> = {
      'project': [
        { model: 'task', field: 'project', label: 'tasks' },
        { model: 'milestone', field: 'project', label: 'milestones' },
        { model: 'project_file', field: 'project', label: 'files' },
        { model: 'project_comment', field: 'project', label: 'comments' }
      ],
      'course': [
        { model: 'lesson', field: 'course', label: 'lessons' },
        { model: 'quiz', field: 'course', label: 'quizzes' },
        { model: 'enrollment', field: 'course', label: 'enrollments' },
        { model: 'course_material', field: 'course', label: 'materials' }
      ],
      'event': [
        { model: 'ticket_type', field: 'event', label: 'ticket types' },
        { model: 'registration', field: 'event', label: 'registrations' },
        { model: 'speaker', field: 'events', label: 'speaker assignments' }
      ]
    };
    
    const client = ctx.createClient();
    
    for (const item of items) {
      const itemType = ctx.itemTypes.find(
        it => it.id === item.relationships.item_type.data.id
      );
      
      if (!itemType) continue;
      
      const cascades = cascadeDefinitions[itemType.attributes.api_key];
      
      if (cascades) {
        const dependencies: Array<{
          label: string;
          count: number;
          items: Item[];
        }> = [];
        
        // Check all cascade relationships
        for (const cascade of cascades) {
          const relatedItems = await client.items.list({
            filter: {
              type: cascade.model,
              fields: {
                [cascade.field]: {
                  eq: item.id
                }
              }
            },
            page: {
              limit: 100
            }
          });
          
          if (relatedItems.length > 0) {
            dependencies.push({
              label: cascade.label,
              count: relatedItems.length,
              items: relatedItems
            });
          }
        }
        
        if (dependencies.length > 0) {
          // Build summary
          const summary = dependencies
            .map(dep => `• ${dep.count} ${dep.label}`)
            .join('\n');
          
          const totalItems = dependencies.reduce((sum, dep) => sum + dep.count, 0);
          
          const action = await ctx.openConfirm({
            title: `Delete ${itemType.attributes.name}?`,
            content: 
              `Deleting "${item.attributes.name || item.attributes.title}" will also delete:\n\n` +
              `${summary}\n\n` +
              `Total: ${totalItems} related items will be deleted.\n` +
              'This action cannot be undone.',
            choices: [
              { 
                label: 'Cancel', 
                value: 'cancel'
              },
              { 
                label: `Delete All (${totalItems + 1} items)`, 
                value: 'cascade',
                intent: 'negative'
              }
            ]
          });
          
          if (action !== 'cascade') {
            return false;
          }
          
          // Perform cascade deletion
          ctx.notice('Deleting related items...');
          
          for (const dep of dependencies) {
            await client.items.bulkDestroy({
              items: dep.items.map(i => ({ id: i.id }))
            });
          }
          
          ctx.notice(`Deleted ${totalItems} related items`);
        }
      }
    }
    
    return true;
  }
});
```

### 4. Soft Delete Implementation

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsDestroyCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsDestroy(
    items: Item[],
    ctx: OnBeforeItemsDestroyCtx
  ): Promise<boolean> {
    // Models that support soft delete
    const softDeleteModels = [
      'customer',
      'vendor', 
      'employee',
      'contract',
      'product'
    ];
    
    const softDeleteItems = items.filter(item => {
      const itemType = ctx.itemTypes.find(
        it => it.id === item.relationships.item_type.data.id
      );
      return softDeleteModels.includes(itemType?.attributes.api_key || '');
    });
    
    if (softDeleteItems.length > 0) {
      // Group by type for better UX
      const itemsByType = softDeleteItems.reduce((acc, item) => {
        const itemType = ctx.itemTypes.find(
          it => it.id === item.relationships.item_type.data.id
        );
        const typeName = itemType?.attributes.name || 'Unknown';
        
        if (!acc[typeName]) {
          acc[typeName] = [];
        }
        acc[typeName].push(item);
        return acc;
      }, {} as Record<string, Item[]>);
      
      // Build summary
      const summary = Object.entries(itemsByType)
        .map(([type, items]) => `• ${items.length} ${type}(s)`)
        .join('\n');
      
      const action = await ctx.openConfirm({
        title: 'Archive or Delete?',
        content: 
          'These records can be archived instead of permanently deleted:\n\n' +
          `${summary}\n\n` +
          'Archived records:\n' +
          '• Are hidden from normal views\n' +
          '• Can be restored later if needed\n' +
          '• Maintain data integrity\n\n' +
          'What would you like to do?',
        choices: [
          { 
            label: 'Archive Records', 
            value: 'archive',
            intent: 'positive'
          },
          { 
            label: 'Delete Permanently', 
            value: 'delete',
            intent: 'negative'
          },
          { 
            label: 'Cancel', 
            value: 'cancel'
          }
        ]
      });
      
      if (action === 'cancel') {
        return false;
      }
      
      if (action === 'archive') {
        const client = ctx.createClient();
        let archivedCount = 0;
        
        // Archive each item
        for (const item of softDeleteItems) {
          try {
            await client.items.update(item.id, {
              archived: true,
              archived_at: new Date().toISOString(),
              archived_by: ctx.currentUser?.id,
              archived_reason: 'User requested deletion'
            });
            archivedCount++;
          } catch (error) {
            console.error(`Failed to archive item ${item.id}:`, error);
          }
        }
        
        ctx.notice(
          `Successfully archived ${archivedCount} record(s).\n` +
          'They can be restored from the Archive section.'
        );
        
        return false; // Prevent actual deletion
      }
      
      // If delete was chosen, ask for confirmation on permanent deletion
      if (action === 'delete') {
        const confirmDelete = await ctx.openConfirm({
          title: '⚠️ Permanent Deletion Warning',
          content: 
            'You are about to permanently delete these records.\n\n' +
            'This action:\n' +
            '• Cannot be undone\n' +
            '• Will remove all data permanently\n' +
            '• May affect related records\n\n' +
            'Are you absolutely sure?',
          choices: [
            { 
              label: 'Cancel', 
              value: false
            },
            { 
              label: 'Yes, Delete Permanently', 
              value: true,
              intent: 'negative'
            }
          ]
        });
        
        if (!confirmDelete) {
          return false;
        }
      }
    }
    
    return true;
  }
});
```

### 5. Pre-deletion Backup

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsDestroyCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsDestroy(
    items: Item[],
    ctx: OnBeforeItemsDestroyCtx
  ): Promise<boolean> {
    // Models requiring backup before deletion
    const backupRequired = [
      'financial_record',
      'tax_document',
      'legal_contract',
      'compliance_report',
      'audit_trail'
    ];
    
    const criticalItems = items.filter(item => {
      const itemType = ctx.itemTypes.find(
        it => it.id === item.relationships.item_type.data.id
      );
      return backupRequired.includes(itemType?.attributes.api_key || '');
    });
    
    if (criticalItems.length > 0) {
      // Notify user about backup process
      const proceed = await ctx.openConfirm({
        title: 'Backup Required',
        content: 
          `${criticalItems.length} critical record(s) will be backed up before deletion.\n\n` +
          'This is required for compliance and audit purposes.\n' +
          'The backup process may take a few moments.\n\n' +
          'Continue?',
        choices: [
          { label: 'Cancel', value: false },
          { label: 'Backup and Delete', value: true, intent: 'negative' }
        ]
      });
      
      if (!proceed) {
        return false;
      }
      
      ctx.notice('Creating backup...');
      
      try {
        // Create comprehensive backup
        const backup = {
          id: `backup_${Date.now()}`,
          timestamp: new Date().toISOString(),
          user: {
            id: ctx.currentUser?.id,
            email: ctx.currentUser?.attributes.email,
            name: ctx.currentUser?.attributes.full_name
          },
          environment: ctx.environment,
          reason: 'pre_deletion_backup',
          items: await Promise.all(
            criticalItems.map(async item => {
              const itemType = ctx.itemTypes.find(
                it => it.id === item.relationships.item_type.data.id
              );
              
              // Get all relationships
              const client = ctx.createClient();
              const fullItem = await client.items.find(item.id, {
                nested: true
              });
              
              return {
                id: item.id,
                type: itemType?.attributes.api_key,
                type_name: itemType?.attributes.name,
                attributes: fullItem.attributes,
                relationships: fullItem.relationships,
                meta: fullItem.meta,
                checksum: generateChecksum(fullItem)
              };
            })
          )
        };
        
        // Store backup in multiple locations
        const backupStored = await storeBackup(backup, ctx);
        
        if (!backupStored) {
          await ctx.alert(
            'Backup Failed\n\n' +
            'Could not create backup of critical records.\n' +
            'Deletion has been cancelled for safety.\n\n' +
            'Please contact system administrator.'
          );
          return false;
        }
        
        // Create backup record in DatoCMS
        const client = ctx.createClient();
        await client.items.create({
          itemType: 'deletion_backup',
          backup_id: backup.id,
          items_count: criticalItems.length,
          backup_location: 'external_storage',
          created_by: ctx.currentUser?.id,
          created_at: backup.timestamp,
          expiry_date: new Date(Date.now() + 7 * 365 * 24 * 60 * 60 * 1000).toISOString(), // 7 years
          items_summary: criticalItems.map(item => ({
            id: item.id,
            type: item.relationships.item_type.data.id,
            name: item.attributes.name || item.attributes.title || item.id
          }))
        });
        
        ctx.notice(
          `Backup completed successfully.\n` +
          `Backup ID: ${backup.id}\n` +
          `${criticalItems.length} record(s) backed up.`
        );
        
      } catch (error) {
        console.error('Backup error:', error);
        
        await ctx.alert(
          'Backup Error\n\n' +
          'An error occurred while creating the backup.\n' +
          'Deletion has been cancelled.\n\n' +
          `Error: ${error.message}`
        );
        
        return false;
      }
    }
    
    return true;
  }
});

// Helper functions
function generateChecksum(item: any): string {
  // Simple checksum for verification
  const content = JSON.stringify(item);
  let hash = 0;
  for (let i = 0; i < content.length; i++) {
    const char = content.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash;
  }
  return Math.abs(hash).toString(36);
}

async function storeBackup(
  backup: any,
  ctx: OnBeforeItemsDestroyCtx
): Promise<boolean> {
  try {
    // Example: Store to S3 or external service
    const response = await fetch(ctx.plugin.attributes.parameters.backupApiUrl as string, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${ctx.plugin.attributes.parameters.backupApiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(backup)
    });
    
    return response.ok;
  } catch (error) {
    console.error('Failed to store backup:', error);
    return false;
  }
}
```

### 6. Multi-environment Validation

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsDestroyCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsDestroy(
    items: Item[],
    ctx: OnBeforeItemsDestroyCtx
  ): Promise<boolean> {
    // Models that might be shared across environments
    const environmentSensitiveModels = [
      'global_config',
      'feature_flag',
      'api_configuration',
      'integration_settings'
    ];
    
    const sensitiveItems = items.filter(item => {
      const itemType = ctx.itemTypes.find(
        it => it.id === item.relationships.item_type.data.id
      );
      return environmentSensitiveModels.includes(itemType?.attributes.api_key || '');
    });
    
    if (sensitiveItems.length > 0) {
      // Check current environment
      const isProduction = ctx.environment === 'main';
      const environments = ['development', 'staging', 'main'];
      const otherEnvironments = environments.filter(env => env !== ctx.environment);
      
      if (isProduction) {
        // Extra caution for production
        const itemList = sensitiveItems.map(item => {
          const itemType = ctx.itemTypes.find(
            it => it.id === item.relationships.item_type.data.id
          );
          return `• ${itemType?.attributes.name}: ${item.attributes.name || item.id}`;
        }).join('\n');
        
        const confirmation = await ctx.openConfirm({
          title: '⚠️ Production Deletion Warning',
          content: 
            'You are about to delete items from the PRODUCTION environment:\n\n' +
            `${itemList}\n\n` +
            'This may affect:\n' +
            '• Live website functionality\n' +
            '• Active user experiences\n' +
            '• API integrations\n\n' +
            'Have you tested this deletion in staging?',
          choices: [
            { label: 'Cancel', value: 'cancel' },
            { 
              label: 'Yes, I have tested in staging', 
              value: 'tested',
              intent: 'negative'
            }
          ]
        });
        
        if (confirmation !== 'tested') {
          return false;
        }
        
        // Double confirmation for production
        const finalConfirm = await ctx.openConfirm({
          title: 'Final Confirmation',
          content: 'Type "DELETE" to confirm production deletion:',
          choices: [
            { label: 'Cancel', value: false },
            { label: 'DELETE', value: true, intent: 'negative' }
          ]
        });
        
        if (!finalConfirm) {
          return false;
        }
      } else {
        // Non-production environment warning
        const warning = await ctx.openConfirm({
          title: 'Environment-Specific Deletion',
          content: 
            `You are deleting items in the ${ctx.environment.toUpperCase()} environment.\n\n` +
            'Note:\n' +
            `• This will NOT affect items in: ${otherEnvironments.join(', ')}\n` +
            '• Each environment maintains separate data\n' +
            '• You may need to repeat this in other environments\n\n' +
            'Continue?',
          choices: [
            { label: 'Cancel', value: false },
            { 
              label: `Delete in ${ctx.environment}`, 
              value: true,
              intent: 'negative'
            }
          ]
        });
        
        if (!warning) {
          return false;
        }
      }
    }
    
    // Check for items that shouldn't be deleted in any environment
    const protectedItems = items.filter(item => {
      const attrs = item.attributes;
      
      // Never delete system defaults
      if (attrs.is_system_default === true) {
        return true;
      }
      
      // Never delete if marked as protected
      if (attrs.deletion_protected === true) {
        return true;
      }
      
      // Check for special flags
      if (attrs.do_not_delete === true) {
        return true;
      }
      
      return false;
    });
    
    if (protectedItems.length > 0) {
      const protectedList = protectedItems.map(item => {
        const reason = item.attributes.is_system_default ? 'System Default' :
                      item.attributes.deletion_protected ? 'Protected' :
                      'Marked as Do Not Delete';
        return `• ${item.attributes.name || item.id} (${reason})`;
      }).join('\n');
      
      await ctx.alert(
        'Protected Items Cannot Be Deleted\n\n' +
        'The following items are protected from deletion:\n' +
        `${protectedList}\n\n` +
        'Contact your system administrator if you need to modify these items.'
      );
      
      return false;
    }
    
    return true;
  }
});
```

## Best Practices

### 1. Clear Error Messages
```typescript
// Provide specific, actionable feedback
await ctx.alert(
  `Cannot delete category "${item.attributes.name}".\n\n` +
  `It is used by ${count} products.\n\n` +
  'Options:\n' +
  '1. Reassign products to another category\n' +
  '2. Delete the products first\n' +
  '3. Archive the category instead'
);
```

### 2. Offer Alternatives
```typescript
// Don't just block - provide options
const action = await ctx.openConfirm({
  title: 'Choose Action',
  choices: [
    { label: 'Cancel', value: 'cancel' },
    { label: 'Archive Instead', value: 'archive' },
    { label: 'Reassign Dependencies', value: 'reassign' },
    { label: 'Force Delete', value: 'force', intent: 'negative' }
  ]
});
```

### 3. Batch Processing
```typescript
// Handle multiple items efficiently
const itemsByType = items.reduce((acc, item) => {
  const typeId = item.relationships.item_type.data.id;
  if (!acc[typeId]) acc[typeId] = [];
  acc[typeId].push(item);
  return acc;
}, {} as Record<string, Item[]>);

// Process by type for efficiency
for (const [typeId, typeItems] of Object.entries(itemsByType)) {
  // Batch checks for same type
}
```

### 4. Performance Optimization
```typescript
// Use efficient queries
const hasDepenencies = await client.items.list({
  filter: {
    type: 'dependent_type',
    fields: {
      parent_id: {
        in: items.map(i => i.id)  // Check all at once
      }
    }
  },
  page: {
    limit: 1  // We only need to know if any exist
  }
});
```

## Common Patterns

### Dependency Checking
```typescript
// Generic dependency checker
async function checkDependencies(
  item: Item,
  dependencyMap: Record<string, { model: string; field: string }>,
  ctx: OnBeforeItemsDestroyCtx
): Promise<{ model: string; count: number }[]> {
  const client = ctx.createClient();
  const results = [];
  
  for (const [key, dep] of Object.entries(dependencyMap)) {
    const items = await client.items.list({
      filter: {
        type: dep.model,
        fields: {
          [dep.field]: { eq: item.id }
        }
      },
      page: { limit: 0 }  // Just get count
    });
    
    if (items.meta.total_count > 0) {
      results.push({ 
        model: dep.model, 
        count: items.meta.total_count 
      });
    }
  }
  
  return results;
}
```

### Soft Delete Pattern
```typescript
// Convert delete to archive
await client.items.update(item.id, {
  deleted: true,
  deleted_at: new Date().toISOString(),
  deleted_by: ctx.currentUser?.id
});

return false; // Prevent actual deletion
```

## Related Hooks

- [`onBeforeItemUpsert`](./onBeforeItemUpsert.md) - Validate single item operations
- [`onBeforeItemsPublish`](./onBeforeItemsPublish.md) - Validate before publishing
- [`onBeforeItemsUnpublish`](./onBeforeItemsUnpublish.md) - Validate before unpublishing
- [`itemsDropdownActions`](./itemsDropdownActions.md) - Add bulk actions

## Troubleshooting

### Hook Not Triggering
1. Ensure hook is registered in plugin
2. Check for JavaScript errors
3. Verify plugin is active
4. Test with console.log

### Performance Issues
1. Use efficient queries with limits
2. Batch operations where possible
3. Cache repeated lookups
4. Consider async/parallel checks

### User Experience
1. Show progress for long operations
2. Provide clear, actionable messages
3. Offer alternatives to deletion
4. Allow graceful cancellation