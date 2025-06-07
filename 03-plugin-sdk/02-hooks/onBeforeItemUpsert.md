# onBeforeItemUpsert Hook

The `onBeforeItemUpsert` hook is triggered before a record is created or updated, allowing plugins to validate the data, perform transformations, or prevent the save operation entirely. This is a critical hook for implementing custom business logic, data validation, and maintaining data integrity across your content.

## Quick Reference

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(payload, ctx) {
    // Validate the data
    if (!isValid(payload)) {
      ctx.alert('Validation failed');
      return false; // Prevent save
    }
    return true; // Allow save
  }
});
```

## Purpose

Use this hook to:
- Validate field values beyond built-in validators
- Enforce complex business rules
- Check data consistency across fields
- Integrate with external validation services
- Transform or enrich data before saving
- Prevent saves based on custom conditions
- Guide users to fix validation issues

## Signature

```typescript
// The hook function signature
onBeforeItemUpsert(
  createOrUpdateItemPayload: ItemUpdateSchema | ItemCreateSchema,
  ctx: OnBeforeItemUpsertCtx
): MaybePromise<boolean>

// Used in plugin definition
connect({
  async onBeforeItemUpsert(payload, ctx): Promise<boolean> {
    // Validate the data
    if (!isValid(payload)) {
      ctx.alert('Validation failed');
      return false; // Prevent save
    }
    return true; // Allow save
  }
});
```

## Parameters

### createOrUpdateItemPayload

The data being saved, either for creating a new record or updating existing:

```typescript
// For new items (ItemCreateSchema)
interface ItemCreateSchema {
  type: 'item';
  attributes: Record<string, any>;          // Field values
  relationships: {
    item_type: {
      data: {
        type: 'item_type';
        id: string;                         // Model ID
      };
    };
  };
  meta?: {
    created_at?: string;
    updated_at?: string;
    status?: 'draft' | 'published' | 'updated';
    publication_scheduled_at?: string | null;
    unpublishing_scheduled_at?: string | null;
  };
}

// For existing items (ItemUpdateSchema)
interface ItemUpdateSchema extends ItemCreateSchema {
  id: string;                               // Item ID when updating
}
```

### ctx: OnBeforeItemUpsertCtx

The context object providing access to DatoCMS data and methods. This type extends the base `Ctx` type with additional methods specific to this hook.

## Type Definitions

### Hook Type Definition

```typescript
// From packages/sdk/src/hooks/onBeforeItemUpsert.ts
import type { SchemaTypes } from '@datocms/cma-client';
import { Ctx } from '../ctx/base';
import { MaybePromise } from '../utils';

type ItemUpdateSchema = SchemaTypes.ItemUpdateSchema;
type ItemCreateSchema = SchemaTypes.ItemCreateSchema;

export type OnBeforeItemUpsertHook = {
  /**
   * This function will be called before saving a new version of a record. You
   * can stop the action by returning `false`
   *
   * @tag beforeHooks
   */
  onBeforeItemUpsert: (
    createOrUpdateItemPayload: ItemUpdateSchema | ItemCreateSchema,
    ctx: OnBeforeItemUpsertCtx,
  ) => MaybePromise<boolean>;
};

export type OnBeforeItemUpsertCtx = Ctx<
  {},
  {
    /**
     * Smoothly navigates to a specific field in the form. If the field is
     * localized it will switch language tab and then navigate to the chosen
     * field.
     *
     * @example
     *
     * ```js
     * const fieldPath = prompt(
     *   'Please insert the path of a field in the form',
     *   ctx.fieldPath,
     * );
     *
     * await ctx.scrollToField(fieldPath);
     * ```
     */
    scrollToField: (path: string, locale?: string) => Promise<void>;
  }
>;
```

### Complete Context Type Expansion

```typescript
// OnBeforeItemUpsertCtx expands to:
type OnBeforeItemUpsertCtx = BaseProperties & BaseMethods & {
  scrollToField: (path: string, locale?: string) => Promise<void>;
};

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
interface OnBeforeItemUpsertCtx {
  // Unique to this hook
  /**
   * Smoothly navigates to a specific field in the form. If the field is
   * localized it will switch language tab and then navigate to the chosen
   * field.
   */
  scrollToField: (path: string, locale?: string) => Promise<void>;
  
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

// Note: Item type is SchemaTypes.Item from @datocms/cma-client
interface Item {
  id: string;
  type: 'item';
  attributes: Record<string, any>;
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

## Return Value

- `true` - Allow the save operation to proceed
- `false` - Cancel the save operation (show error messages first)

## Complete Examples

### 1. Comprehensive Field Validation

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { 
  ItemUpdateSchema, 
  ItemCreateSchema, 
  OnBeforeItemUpsertCtx 
} from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(
    payload: ItemUpdateSchema | ItemCreateSchema,
    ctx: OnBeforeItemUpsertCtx
  ): Promise<boolean> {
    // Get the model being saved
    const modelId = payload.relationships?.item_type?.data?.id;
    if (!modelId) return true;
    
    const itemType = ctx.itemTypes.find(it => it.id === modelId);
    if (!itemType) return true;
    
    // Article validation
    if (itemType.attributes.api_key === 'article') {
      const attrs = payload.attributes || {};
      const errors: string[] = [];
      
      // Title validation
      if (!attrs.title) {
        errors.push('Title is required');
      } else {
        const title = attrs.title as string;
        
        // Length check
        if (title.length < 10 || title.length > 100) {
          errors.push('Title must be between 10 and 100 characters');
        }
        
        // Format check
        if (!/^[A-Z]/.test(title)) {
          errors.push('Title must start with a capital letter');
        }
        
        // Duplicate check (only for new articles)
        if (!('id' in payload)) {
          const client = ctx.createClient();
          const existing = await client.items.list({
            filter: {
              type: 'article',
              fields: {
                title: { eq: title }
              }
            }
          });
          
          if (existing.length > 0) {
            errors.push('An article with this title already exists');
          }
        }
      }
      
      // Content validation
      if (!attrs.content) {
        errors.push('Content is required');
      } else {
        const content = attrs.content as string;
        const wordCount = content.split(/\s+/).filter(Boolean).length;
        
        if (wordCount < 300) {
          errors.push(`Content too short: ${wordCount} words (minimum 300)`);
        }
        
        if (wordCount > 5000) {
          errors.push(`Content too long: ${wordCount} words (maximum 5000)`);
        }
      }
      
      // Category validation
      if (!attrs.category_id) {
        errors.push('Category is required');
      }
      
      // Author validation for published articles
      if (payload.meta?.status === 'published') {
        if (!attrs.author_id) {
          errors.push('Published articles must have an author');
        }
        
        if (!attrs.featured_image) {
          errors.push('Published articles must have a featured image');
        }
        
        if (!attrs.excerpt) {
          errors.push('Published articles must have an excerpt');
        }
      }
      
      // Show errors if any
      if (errors.length > 0) {
        await ctx.alert(`Please fix the following issues:\n\n${errors.join('\n• ')}`);
        
        // Navigate to first problematic field
        if (!attrs.title) {
          await ctx.scrollToField('title');
        } else if (!attrs.content) {
          await ctx.scrollToField('content');
        } else if (!attrs.category_id) {
          await ctx.scrollToField('category_id');
        }
        
        return false;
      }
    }
    
    return true;
  }
});
```

### 2. Business Rule Enforcement

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { OnBeforeItemUpsertCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(payload, ctx: OnBeforeItemUpsertCtx): Promise<boolean> {
    const modelId = payload.relationships?.item_type?.data?.id;
    const itemType = ctx.itemTypes.find(it => it.id === modelId);
    
    // E-commerce order validation
    if (itemType?.attributes.api_key === 'order') {
      const attrs = payload.attributes || {};
      
      // Status transition rules
      if ('id' in payload) {
        // Fetch current order state
        const client = ctx.createClient();
        const currentOrder = await client.items.find(payload.id);
        const currentStatus = currentOrder.attributes.status;
        const newStatus = attrs.status;
        
        // Define allowed transitions
        const allowedTransitions: Record<string, string[]> = {
          'pending': ['processing', 'cancelled'],
          'processing': ['shipped', 'cancelled'],
          'shipped': ['delivered', 'returned'],
          'delivered': ['returned'],
          'cancelled': [],
          'returned': []
        };
        
        if (newStatus && newStatus !== currentStatus) {
          const allowed = allowedTransitions[currentStatus] || [];
          
          if (!allowed.includes(newStatus)) {
            await ctx.alert(
              `Invalid status transition: ${currentStatus} → ${newStatus}\n` +
              `Allowed transitions: ${allowed.join(', ') || 'none'}`
            );
            await ctx.scrollToField('status');
            return false;
          }
        }
      }
      
      // Inventory validation
      if (attrs.items) {
        const orderItems = attrs.items as Array<{
          product_id: string;
          quantity: number;
        }>;
        
        for (const item of orderItems) {
          const product = await ctx.createClient().items.find(item.product_id);
          const availableStock = product.attributes.stock_quantity as number;
          
          if (item.quantity > availableStock) {
            await ctx.alert(
              `Insufficient stock for ${product.attributes.name}:\n` +
              `Requested: ${item.quantity}, Available: ${availableStock}`
            );
            return false;
          }
        }
      }
      
      // Price validation
      if (attrs.total_amount && attrs.items) {
        const calculatedTotal = await calculateOrderTotal(attrs.items, ctx);
        
        if (Math.abs(attrs.total_amount - calculatedTotal) > 0.01) {
          const proceed = await ctx.openConfirm({
            title: 'Price Mismatch',
            content: `Entered total: $${attrs.total_amount}\nCalculated total: $${calculatedTotal}\n\nUse calculated total?`,
            choices: [
              { label: 'Use Calculated', value: true, intent: 'positive' },
              { label: 'Keep Entered', value: false }
            ]
          });
          
          if (proceed) {
            attrs.total_amount = calculatedTotal;
          }
        }
      }
    }
    
    return true;
  }
});

async function calculateOrderTotal(items: any[], ctx: OnBeforeItemUpsertCtx): Promise<number> {
  const client = ctx.createClient();
  let total = 0;
  
  for (const item of items) {
    const product = await client.items.find(item.product_id);
    const price = product.attributes.price as number;
    total += price * item.quantity;
  }
  
  return total;
}
```

### 3. SEO and Content Quality Checks

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { OnBeforeItemUpsertCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(payload, ctx: OnBeforeItemUpsertCtx): Promise<boolean> {
    const modelId = payload.relationships?.item_type?.data?.id;
    const itemType = ctx.itemTypes.find(it => it.id === modelId);
    
    // Only check SEO for published content
    if (payload.meta?.status !== 'published') {
      return true;
    }
    
    const seoModels = ['article', 'page', 'landing_page', 'product'];
    
    if (itemType && seoModels.includes(itemType.attributes.api_key)) {
      const attrs = payload.attributes || {};
      const issues: Array<{ field: string; message: string; severity: 'error' | 'warning' }> = [];
      
      // SEO title checks
      if (!attrs.seo_title) {
        issues.push({
          field: 'seo_title',
          message: 'SEO title is required for published content',
          severity: 'error'
        });
      } else {
        const seoTitle = attrs.seo_title as string;
        
        if (seoTitle.length < 30) {
          issues.push({
            field: 'seo_title',
            message: `SEO title too short (${seoTitle.length}/30 minimum)`,
            severity: 'warning'
          });
        }
        
        if (seoTitle.length > 60) {
          issues.push({
            field: 'seo_title',
            message: `SEO title too long (${seoTitle.length}/60 maximum)`,
            severity: 'warning'
          });
        }
        
        // Check for keyword presence
        if (attrs.target_keyword && !seoTitle.toLowerCase().includes(attrs.target_keyword.toLowerCase())) {
          issues.push({
            field: 'seo_title',
            message: 'Target keyword not found in SEO title',
            severity: 'warning'
          });
        }
      }
      
      // Meta description checks
      if (!attrs.meta_description) {
        issues.push({
          field: 'meta_description',
          message: 'Meta description is required for published content',
          severity: 'error'
        });
      } else {
        const metaDesc = attrs.meta_description as string;
        
        if (metaDesc.length < 120) {
          issues.push({
            field: 'meta_description',
            message: `Meta description too short (${metaDesc.length}/120 minimum)`,
            severity: 'warning'
          });
        }
        
        if (metaDesc.length > 160) {
          issues.push({
            field: 'meta_description',
            message: `Meta description too long (${metaDesc.length}/160 maximum)`,
            severity: 'warning'
          });
        }
      }
      
      // URL slug checks
      if (attrs.slug) {
        const slug = attrs.slug as string;
        
        if (!/^[a-z0-9-]+$/.test(slug)) {
          issues.push({
            field: 'slug',
            message: 'URL slug contains invalid characters (use only lowercase letters, numbers, and hyphens)',
            severity: 'error'
          });
        }
        
        if (slug.length > 50) {
          issues.push({
            field: 'slug',
            message: `URL slug too long (${slug.length}/50 maximum)`,
            severity: 'warning'
          });
        }
        
        // Check for duplicate slugs
        const client = ctx.createClient();
        const duplicates = await client.items.list({
          filter: {
            type: itemType.attributes.api_key,
            fields: {
              slug: { eq: slug }
            }
          }
        });
        
        const otherItems = duplicates.filter(item => item.id !== payload.id);
        if (otherItems.length > 0) {
          issues.push({
            field: 'slug',
            message: 'This URL slug is already in use',
            severity: 'error'
          });
        }
      }
      
      // Content quality checks
      if (attrs.content) {
        const content = attrs.content as string;
        const wordCount = content.split(/\s+/).filter(Boolean).length;
        
        // Readability score (simplified)
        const sentences = content.split(/[.!?]+/).filter(Boolean).length;
        const avgWordsPerSentence = wordCount / sentences;
        
        if (avgWordsPerSentence > 25) {
          issues.push({
            field: 'content',
            message: `Average sentence too long (${Math.round(avgWordsPerSentence)} words). Aim for 15-20 words.`,
            severity: 'warning'
          });
        }
        
        // Check keyword density
        if (attrs.target_keyword) {
          const keyword = attrs.target_keyword as string;
          const keywordCount = (content.toLowerCase().match(new RegExp(keyword.toLowerCase(), 'g')) || []).length;
          const keywordDensity = (keywordCount / wordCount) * 100;
          
          if (keywordDensity < 0.5) {
            issues.push({
              field: 'content',
              message: `Low keyword density (${keywordDensity.toFixed(1)}%). Target keyword appears only ${keywordCount} times.`,
              severity: 'warning'
            });
          }
          
          if (keywordDensity > 3) {
            issues.push({
              field: 'content',
              message: `High keyword density (${keywordDensity.toFixed(1)}%). Avoid keyword stuffing.`,
              severity: 'warning'
            });
          }
        }
      }
      
      // Handle issues
      const errors = issues.filter(i => i.severity === 'error');
      const warnings = issues.filter(i => i.severity === 'warning');
      
      if (errors.length > 0) {
        await ctx.alert(
          'SEO Errors (must fix):\n\n' +
          errors.map(e => `• ${e.message}`).join('\n')
        );
        
        await ctx.scrollToField(errors[0].field);
        return false;
      }
      
      if (warnings.length > 0) {
        const proceed = await ctx.openConfirm({
          title: 'SEO Warnings',
          content: warnings.map(w => `• ${w.message}`).join('\n'),
          choices: [
            { label: 'Fix Issues', value: false },
            { label: 'Publish Anyway', value: true, intent: 'negative' }
          ]
        });
        
        if (!proceed) {
          await ctx.scrollToField(warnings[0].field);
          return false;
        }
      }
    }
    
    return true;
  }
});
```

### 4. Multi-locale Content Validation

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { OnBeforeItemUpsertCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(payload, ctx: OnBeforeItemUpsertCtx): Promise<boolean> {
    // Only validate when publishing
    if (payload.meta?.status !== 'published') {
      return true;
    }
    
    const modelId = payload.relationships?.item_type?.data?.id;
    const itemType = ctx.itemTypes.find(it => it.id === modelId);
    
    // Check if model requires all locales
    if (itemType?.attributes.all_locales_required) {
      const attrs = payload.attributes || {};
      const projectLocales = ctx.site.attributes.locales;
      const issues: Array<{ field: string; locales: string[] }> = [];
      
      // Get all localized fields for this model
      const localizedFields = ctx.fields.filter(
        field => field.attributes.localized && 
        field.relationships.item_type.data.id === modelId
      );
      
      // Check each localized field
      for (const field of localizedFields) {
        const fieldValue = attrs[field.attributes.api_key];
        
        if (!fieldValue || typeof fieldValue !== 'object') {
          issues.push({
            field: field.attributes.api_key,
            locales: projectLocales
          });
          continue;
        }
        
        // Check which locales are missing
        const missingLocales = projectLocales.filter(locale => {
          const value = fieldValue[locale];
          
          // Different validation based on field type
          switch (field.attributes.field_type) {
            case 'string':
            case 'text':
              return !value || value.trim() === '';
            case 'file':
            case 'gallery':
              return !value || (Array.isArray(value) && value.length === 0);
            case 'link':
            case 'links':
              return !value || (Array.isArray(value) && value.length === 0);
            default:
              return !value;
          }
        });
        
        if (missingLocales.length > 0) {
          issues.push({
            field: field.attributes.api_key,
            locales: missingLocales
          });
        }
      }
      
      // Check required non-localized fields too
      const requiredFields = ctx.fields.filter(
        field => !field.attributes.localized &&
        field.relationships.item_type.data.id === modelId &&
        field.attributes.validators.required
      );
      
      for (const field of requiredFields) {
        const value = attrs[field.attributes.api_key];
        if (!value || (typeof value === 'string' && value.trim() === '')) {
          issues.push({
            field: field.attributes.api_key,
            locales: ['all'] // Indicates non-localized field
          });
        }
      }
      
      if (issues.length > 0) {
        // Format the report
        const report = issues.map(issue => {
          const field = ctx.fields.find(f => f.attributes.api_key === issue.field);
          const fieldLabel = field?.attributes.label || issue.field;
          
          if (issue.locales.includes('all')) {
            return `${fieldLabel}: Missing value (required field)`;
          } else {
            const localeNames = issue.locales.map(locale => {
              // Convert locale codes to readable names
              const localeMap: Record<string, string> = {
                'en': 'English',
                'it': 'Italian',
                'es': 'Spanish',
                'fr': 'French',
                'de': 'German',
                'pt': 'Portuguese',
                'ja': 'Japanese',
                'zh': 'Chinese'
              };
              return localeMap[locale] || locale;
            });
            
            return `${fieldLabel}: Missing translations for ${localeNames.join(', ')}`;
          }
        }).join('\n');
        
        const proceed = await ctx.openConfirm({
          title: 'Incomplete Translations',
          content: `The following fields have missing translations:\n\n${report}\n\nPublishing will make incomplete content visible.`,
          choices: [
            { label: 'Complete Translations', value: false, intent: 'positive' },
            { label: 'Publish Incomplete', value: true, intent: 'negative' }
          ]
        });
        
        if (!proceed) {
          // Navigate to first incomplete field
          const firstIssue = issues[0];
          const firstMissingLocale = firstIssue.locales[0] !== 'all' ? firstIssue.locales[0] : undefined;
          await ctx.scrollToField(firstIssue.field, firstMissingLocale);
          return false;
        }
      }
    }
    
    return true;
  }
});
```

### 5. Data Enrichment and Auto-Generation

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { OnBeforeItemUpsertCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(payload, ctx: OnBeforeItemUpsertCtx): Promise<boolean> {
    const modelId = payload.relationships?.item_type?.data?.id;
    const itemType = ctx.itemTypes.find(it => it.id === modelId);
    
    if (itemType?.attributes.api_key === 'product') {
      const attrs = payload.attributes || {};
      
      // Auto-generate SKU for new products
      if (!('id' in payload) && !attrs.sku) {
        const name = attrs.name as string || 'PRODUCT';
        const category = attrs.category as string || 'GEN';
        const timestamp = Date.now().toString(36).toUpperCase();
        
        attrs.sku = `${category.substring(0, 3).toUpperCase()}-${name.substring(0, 3).toUpperCase()}-${timestamp}`;
        
        ctx.notice(`Generated SKU: ${attrs.sku}`);
      }
      
      // Calculate and validate pricing
      if (attrs.base_price && attrs.tax_rate) {
        const basePrice = parseFloat(attrs.base_price);
        const taxRate = parseFloat(attrs.tax_rate);
        
        // Calculate total price
        attrs.total_price = basePrice * (1 + taxRate / 100);
        attrs.tax_amount = basePrice * (taxRate / 100);
        
        // Validate pricing logic
        if (attrs.sale_price) {
          const salePrice = parseFloat(attrs.sale_price);
          
          if (salePrice >= basePrice) {
            await ctx.alert('Sale price must be less than base price');
            await ctx.scrollToField('sale_price');
            return false;
          }
          
          attrs.discount_percentage = ((basePrice - salePrice) / basePrice * 100).toFixed(2);
          attrs.savings_amount = (basePrice - salePrice).toFixed(2);
        }
      }
      
      // Auto-generate SEO fields from product data
      if (!attrs.seo_title && attrs.name) {
        attrs.seo_title = `${attrs.name} - ${attrs.brand || 'Our Store'}`;
      }
      
      if (!attrs.meta_description && attrs.description) {
        const description = attrs.description as string;
        attrs.meta_description = description.substring(0, 155) + '...';
      }
      
      if (!attrs.slug && attrs.name) {
        attrs.slug = (attrs.name as string)
          .toLowerCase()
          .replace(/[^a-z0-9]+/g, '-')
          .replace(/^-|-$/g, '');
      }
      
      // Validate inventory
      if (attrs.track_inventory) {
        if (!attrs.stock_quantity && attrs.stock_quantity !== 0) {
          await ctx.alert('Stock quantity is required when inventory tracking is enabled');
          await ctx.scrollToField('stock_quantity');
          return false;
        }
        
        if (attrs.stock_quantity < attrs.low_stock_threshold) {
          const proceed = await ctx.openConfirm({
            title: 'Low Stock Warning',
            content: `Stock quantity (${attrs.stock_quantity}) is below the low stock threshold (${attrs.low_stock_threshold}).`,
            choices: [
              { label: 'Continue', value: true },
              { label: 'Adjust Stock', value: false }
            ]
          });
          
          if (!proceed) {
            await ctx.scrollToField('stock_quantity');
            return false;
          }
        }
      }
      
      // Set metadata
      attrs.last_updated_by = ctx.currentUser?.id;
      attrs.last_updated_at = new Date().toISOString();
      
      if (!('id' in payload)) {
        attrs.created_by = ctx.currentUser?.id;
        attrs.created_at = new Date().toISOString();
      }
    }
    
    return true;
  }
});
```

### 6. External API Integration

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { OnBeforeItemUpsertCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemUpsert(payload, ctx: OnBeforeItemUpsertCtx): Promise<boolean> {
    const modelId = payload.relationships?.item_type?.data?.id;
    const itemType = ctx.itemTypes.find(it => it.id === modelId);
    
    if (itemType?.attributes.api_key === 'location') {
      const attrs = payload.attributes || {};
      
      // Validate and enrich address data
      if (attrs.street_address && attrs.city && attrs.country) {
        try {
          // Geocode the address
          const geocodeUrl = `https://maps.googleapis.com/maps/api/geocode/json`;
          const params = new URLSearchParams({
            address: `${attrs.street_address}, ${attrs.city}, ${attrs.country}`,
            key: ctx.plugin.attributes.parameters.googleMapsApiKey as string
          });
          
          const geocodeResponse = await fetch(`${geocodeUrl}?${params}`);
          const geocodeData = await geocodeResponse.json();
          
          if (geocodeData.status === 'OK' && geocodeData.results.length > 0) {
            const result = geocodeData.results[0];
            
            // Set coordinates
            attrs.latitude = result.geometry.location.lat;
            attrs.longitude = result.geometry.location.lng;
            
            // Extract and validate components
            const components = result.address_components;
            
            // Set postal code if found
            const postalCode = components.find((c: any) => 
              c.types.includes('postal_code')
            );
            if (postalCode && !attrs.postal_code) {
              attrs.postal_code = postalCode.long_name;
            }
            
            // Set state/province if found
            const state = components.find((c: any) => 
              c.types.includes('administrative_area_level_1')
            );
            if (state && !attrs.state) {
              attrs.state = state.long_name;
            }
            
            // Store the formatted address
            attrs.formatted_address = result.formatted_address;
            
            ctx.notice('Address validated and geocoded successfully');
          } else {
            await ctx.alert('Could not validate address. Please check the address details.');
            await ctx.scrollToField('street_address');
            return false;
          }
        } catch (error) {
          console.error('Geocoding error:', error);
          
          // Ask user if they want to continue without geocoding
          const proceed = await ctx.openConfirm({
            title: 'Geocoding Failed',
            content: 'Could not validate the address with Google Maps. Save without coordinates?',
            choices: [
              { label: 'Save Without Coordinates', value: true, intent: 'negative' },
              { label: 'Fix Address', value: false }
            ]
          });
          
          if (!proceed) {
            await ctx.scrollToField('street_address');
            return false;
          }
        }
      }
      
      // Validate coordinates if manually entered
      if (attrs.latitude && attrs.longitude) {
        const lat = parseFloat(attrs.latitude);
        const lng = parseFloat(attrs.longitude);
        
        if (lat < -90 || lat > 90) {
          await ctx.alert('Latitude must be between -90 and 90');
          await ctx.scrollToField('latitude');
          return false;
        }
        
        if (lng < -180 || lng > 180) {
          await ctx.alert('Longitude must be between -180 and 180');
          await ctx.scrollToField('longitude');
          return false;
        }
      }
    }
    
    // Customer validation with external CRM
    if (itemType?.attributes.api_key === 'customer') {
      const attrs = payload.attributes || {};
      
      if (attrs.email) {
        try {
          // Validate email format and existence
          const emailValidationUrl = 'https://api.email-validator.net/api/verify';
          const validationResponse = await fetch(emailValidationUrl, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${ctx.plugin.attributes.parameters.emailValidatorKey}`
            },
            body: JSON.stringify({ email: attrs.email })
          });
          
          const validation = await validationResponse.json();
          
          if (!validation.status || validation.status === 'invalid') {
            await ctx.alert(`Invalid email address: ${validation.reason || 'Unknown error'}`);
            await ctx.scrollToField('email');
            return false;
          }
          
          if (validation.disposable) {
            const allowDisposable = await ctx.openConfirm({
              title: 'Disposable Email Detected',
              content: 'This appears to be a disposable email address. Allow anyway?',
              choices: [
                { label: 'Reject', value: false, intent: 'positive' },
                { label: 'Allow', value: true, intent: 'negative' }
              ]
            });
            
            if (!allowDisposable) {
              await ctx.scrollToField('email');
              return false;
            }
          }
          
          // Check CRM for existing customer
          if (!('id' in payload) && ctx.plugin.attributes.parameters.crmApiUrl) {
            const crmResponse = await fetch(
              `${ctx.plugin.attributes.parameters.crmApiUrl}/customers/check`,
              {
                method: 'POST',
                headers: {
                  'Content-Type': 'application/json',
                  'X-API-Key': ctx.plugin.attributes.parameters.crmApiKey as string
                },
                body: JSON.stringify({ email: attrs.email })
              }
            );
            
            if (crmResponse.ok) {
              const crmData = await crmResponse.json();
              
              if (crmData.exists) {
                await ctx.alert(
                  'A customer with this email already exists in the CRM.\n' +
                  `Customer ID: ${crmData.customerId}\n` +
                  `Name: ${crmData.name}`
                );
                return false;
              }
            }
          }
        } catch (error) {
          console.error('Email validation error:', error);
          
          // Non-blocking warning
          ctx.notice('Could not validate email address - proceeding anyway');
        }
      }
    }
    
    return true;
  }
});
```

## Best Practices

### 1. Performance Optimization
```typescript
// Cache repeated lookups
const itemTypeCache = new Map();

async onBeforeItemUpsert(payload, ctx) {
  const modelId = payload.relationships?.item_type?.data?.id;
  
  let itemType = itemTypeCache.get(modelId);
  if (!itemType) {
    itemType = ctx.itemTypes.find(it => it.id === modelId);
    itemTypeCache.set(modelId, itemType);
  }
  
  // Use cached itemType...
}
```

### 2. User-Friendly Error Messages
```typescript
// Provide clear, actionable feedback
if (!isValid) {
  await ctx.alert(
    'Please fix the following issues:\n\n' +
    '• Title must be at least 10 characters\n' +
    '• Featured image is required\n' +
    '• Select at least one category'
  );
  
  // Guide to the first issue
  await ctx.scrollToField('title');
  return false;
}
```

### 3. Progressive Enhancement
```typescript
// Validate based on save context
if (payload.meta?.status === 'draft') {
  // Relaxed validation for drafts
  if (!attrs.title) {
    ctx.notice('Remember to add a title before publishing');
  }
} else {
  // Strict validation for published content
  if (!attrs.title || attrs.title.length < 10) {
    await ctx.alert('Published content requires a title of at least 10 characters');
    return false;
  }
}
```

### 4. Error Recovery
```typescript
// Handle external API failures gracefully
try {
  const response = await fetch(externalApi);
  // Process response...
} catch (error) {
  const proceed = await ctx.openConfirm({
    title: 'External Service Unavailable',
    content: 'Could not validate with external service. Save anyway?',
    choices: [
      { label: 'Save Anyway', value: true, intent: 'negative' },
      { label: 'Cancel', value: false }
    ]
  });
  
  if (!proceed) return false;
}
```

## Common Patterns

### Model Type Checking
```typescript
const modelId = payload.relationships?.item_type?.data?.id;
const itemType = ctx.itemTypes.find(it => it.id === modelId);

if (itemType?.attributes.api_key === 'target_model') {
  // Model-specific validation
}
```

### Field Navigation
```typescript
// Navigate to problematic field
await ctx.scrollToField('field_name', 'locale_code');
```

### Conditional Validation
```typescript
// Only validate certain conditions
if (payload.meta?.status === 'published') {
  // Published-only validation
}

if (!('id' in payload)) {
  // Creation-only validation
}

if ('id' in payload) {
  // Update-only validation
}
```

## Related Hooks

- [`onBeforeItemsPublish`](./onBeforeItemsPublish.md) - Validate before bulk publishing
- [`onBeforeItemsDestroy`](./onBeforeItemsDestroy.md) - Validate before deletion
- [`renderFieldExtension`](./renderFieldExtension.md) - Add custom validation UI
- [`renderItemFormSidebarPanel`](./renderItemFormSidebarPanel.md) - Add validation panels

## Troubleshooting

### Hook Not Triggering
1. Ensure hook is registered in plugin
2. Check for JavaScript errors
3. Verify plugin is installed and active
4. Test with simple console.log first

### Validation Bypassed
1. Always return explicit boolean
2. Check all code paths return true/false
3. Handle async operations properly
4. Verify error messages shown before false

### Performance Issues
1. Minimize API calls in validation
2. Cache repeated lookups
3. Use efficient queries
4. Consider debouncing for field-level validation