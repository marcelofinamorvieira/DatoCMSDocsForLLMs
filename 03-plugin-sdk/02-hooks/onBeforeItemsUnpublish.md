# onBeforeItemsUnpublish Hook

The `onBeforeItemsUnpublish` hook is triggered before one or more records are unpublished, allowing plugins to validate dependencies, check business rules, or prevent the unpublish operation. This hook helps maintain content integrity and ensures that unpublishing content won't break live websites or violate business requirements.

## Quick Reference

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsUnpublish(items, ctx) {
    // Check for published dependencies
    for (const item of items) {
      const dependents = await findPublishedDependents(item, ctx);
      if (dependents.length > 0) {
        ctx.alert(`Cannot unpublish: ${dependents.length} published items depend on this`);
        return false; // Prevent unpublishing
      }
    }
    return true; // Allow unpublishing
  }
});
```

## Purpose

The `onBeforeItemsUnpublish` hook validates before unpublishing to:
- Prevent breaking published content dependencies
- Maintain minimum published content requirements
- Enforce business rules about content availability
- Check SEO and traffic impact
- Coordinate with external systems

## Signature

```typescript
onBeforeItemsUnpublish(
  items: Item[],
  ctx: OnBeforeItemsUnpublishCtx
): MaybePromise<boolean>
```

## Parameters

### items: Item[]

Array of records that are about to be unpublished:

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

### ctx: OnBeforeItemsUnpublishCtx

The context object providing access to DatoCMS data and methods. This type extends the base `Ctx` type.

## Type Definitions

### Hook Type Definition

```typescript
// From packages/sdk/src/hooks/onBeforeItemsUnpublish.ts
import type { SchemaTypes } from '@datocms/cma-client';
import { Ctx } from '../ctx/base';
import { MaybePromise } from '../utils';

type Item = SchemaTypes.Item;

export type OnBeforeItemsUnpublishHook = {
  /**
   * This function will be called before unpublishing records. You can stop the
   * action by returning `false`
   *
   * @tag beforeHooks
   */
  onBeforeItemsUnpublish: (
    items: Item[],
    ctx: OnBeforeItemsUnpublishCtx,
  ) => MaybePromise<boolean>;
};

export type OnBeforeItemsUnpublishCtx = Ctx;
```

### Complete Context Type Expansion

```typescript
// OnBeforeItemsUnpublishCtx expands to:
type OnBeforeItemsUnpublishCtx = BaseProperties & BaseMethods;

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
interface OnBeforeItemsUnpublishCtx {
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

- `true`: Allow the unpublish operation to proceed
- `false`: Cancel the unpublish operation

## Complete Examples

### 1. Dependency Validation

Prevent unpublishing content that other published content depends on:

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsUnpublishCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsUnpublish(
    items: Item[],
    ctx: OnBeforeItemsUnpublishCtx
  ): Promise<boolean> {
    const criticalDependencies: string[] = [];
    
    for (const item of items) {
      const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
      
      // Check if unpublishing a page that's linked in navigation
      if (itemType?.attributes.api_key === 'page') {
        const menuItems = await ctx.searchItems({
          filter: {
            type: 'menu_item',
            fields: {
              linked_page: { eq: item.id }
            }
          }
        });
        
        const publishedMenuItems = menuItems.filter(mi => mi.meta.status === 'published');
        
        if (publishedMenuItems.length > 0) {
          criticalDependencies.push(
            `Page "${item.attributes.title}" is linked in ${publishedMenuItems.length} published menu items`
          );
        }
        
        // Check if it's a landing page for campaigns
        const campaigns = await ctx.searchItems({
          filter: {
            type: 'campaign',
            fields: {
              landing_page: { eq: item.id },
              status: { eq: 'active' }
            }
          }
        });
        
        if (campaigns.length > 0) {
          criticalDependencies.push(
            `Page "${item.attributes.title}" is the landing page for ${campaigns.length} active campaigns`
          );
        }
      }
      
      // Check if unpublishing a category with published products
      if (itemType?.attributes.api_key === 'category') {
        const publishedProducts = await ctx.searchItems({
          filter: {
            type: 'product',
            fields: {
              category: { eq: item.id }
            }
          }
        });
        
        // Filter for published products
        const actuallyPublished = publishedProducts.filter(
          p => p.meta.status === 'published'
        );
        
        if (actuallyPublished.length > 0) {
          criticalDependencies.push(
            `Category "${item.attributes.name}" has ${actuallyPublished.length} published products`
          );
        }
      }
      
      // Check if unpublishing an author with published articles
      if (itemType?.attributes.api_key === 'author') {
        const publishedArticles = await ctx.searchItems({
          filter: {
            type: 'article',
            fields: {
              author: { eq: item.id }
            }
          }
        });
        
        // Filter for published articles
        const actuallyPublished = publishedArticles.filter(
          a => a.meta.status === 'published'
        );
        
        if (actuallyPublished.length > 0) {
          criticalDependencies.push(
            `Author "${item.attributes.name}" has ${actuallyPublished.length} published articles`
          );
        }
      }
    }
    
    if (criticalDependencies.length > 0) {
      const proceed = await ctx.openConfirm({
        title: 'Critical Dependencies Found',
        content: 
          'Unpublishing these items will affect:\n\n' +
          criticalDependencies.map(d => `• ${d}`).join('\n') +
          '\n\nThis may break your live website. Continue?',
        choices: [
          { label: 'Cancel', value: false },
          { label: 'Unpublish Anyway', value: true, intent: 'negative' }
        ]
      });
      
      return proceed;
    }
    
    return true;
  }
});
```

### 2. Business Rule Enforcement

Enforce business rules around content availability:

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsUnpublishCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsUnpublish(
    items: Item[],
    ctx: OnBeforeItemsUnpublishCtx
  ): Promise<boolean> {
    for (const item of items) {
      const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
      
      // Prevent unpublishing active promotions
      if (itemType?.attributes.api_key === 'promotion') {
        const startDate = new Date(item.attributes.start_date as string);
        const endDate = new Date(item.attributes.end_date as string);
        const now = new Date();
        
        if (now >= startDate && now <= endDate) {
          await ctx.alert(
            `Cannot unpublish active promotion "${item.attributes.title}".\n\n` +
            `This promotion is currently running until ${endDate.toLocaleDateString()}.\n` +
            'Please wait until it ends or update the end date first.'
          );
          return false;
        }
      }
      
      // Prevent unpublishing required legal pages
      if (itemType?.attributes.api_key === 'legal_page') {
        const requiredPages = ['privacy-policy', 'terms-of-service', 'cookie-policy'];
        const slug = item.attributes.slug as string;
        
        if (requiredPages.includes(slug)) {
          await ctx.alert(
            `Cannot unpublish required legal page "${item.attributes.title}".\n\n` +
            'This page is legally required to be publicly accessible.'
          );
          return false;
        }
      }
      
      // Check minimum published content requirements
      if (itemType?.attributes.api_key === 'product') {
        const publishedProducts = await ctx.searchItems({
          filter: {
            type: 'product',
            meta: { status: { eq: 'published' } }
          }
        });
        
        // Ensure at least 10 products remain published
        const remainingPublished = publishedProducts.filter(
          p => !items.find(i => i.id === p.id)
        ).length;
        
        if (remainingPublished < 10) {
          await ctx.alert(
            `Cannot unpublish these products.\n\n` +
            `The store requires at least 10 published products.\n` +
            `After unpublishing, only ${remainingPublished} would remain.`
          );
          return false;
        }
      }
    }
    
    return true;
  }
});
```

### 3. Scheduled Content Management

Handle scheduled content and embargo rules:

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsUnpublishCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsUnpublish(
    items: Item[],
    ctx: OnBeforeItemsUnpublishCtx
  ): Promise<boolean> {
    const warnings: string[] = [];
    
    for (const item of items) {
      const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
      
      // Check for scheduled unpublish dates
      if (item.attributes.unpublish_at) {
        const scheduledUnpublish = new Date(item.attributes.unpublish_at as string);
        const now = new Date();
        
        if (scheduledUnpublish > now) {
          const daysUntil = Math.ceil(
            (scheduledUnpublish.getTime() - now.getTime()) / (1000 * 60 * 60 * 24)
          );
          
          warnings.push(
            `"${item.attributes.title}" is scheduled to unpublish in ${daysUntil} days`
          );
        }
      }
      
      // Check content contracts
      if (item.attributes.content_contract_until) {
        const contractEnd = new Date(item.attributes.content_contract_until as string);
        
        if (contractEnd > new Date()) {
          warnings.push(
            `"${item.attributes.title}" has a content contract until ${contractEnd.toLocaleDateString()}`
          );
        }
      }
      
      // Check seasonal content
      if (itemType?.attributes.api_key === 'seasonal_content') {
        const season = item.attributes.season as string;
        const currentSeason = getCurrentSeason();
        
        if (season === currentSeason) {
          warnings.push(
            `"${item.attributes.title}" is content for the current season (${season})`
          );
        }
      }
    }
    
    if (warnings.length > 0) {
      const proceed = await ctx.openConfirm({
        title: 'Timing Warnings',
        content: 
          'Please note:\n\n' +
          warnings.map(w => `• ${w}`).join('\n') +
          '\n\nUnpublish now?',
        choices: [
          { label: 'Wait', value: false },
          { label: 'Unpublish Now', value: true }
        ]
      });
      
      return proceed;
    }
    
    return true;
  }
});

function getCurrentSeason(): string {
  const month = new Date().getMonth();
  if (month >= 2 && month <= 4) return 'spring';
  if (month >= 5 && month <= 7) return 'summer';
  if (month >= 8 && month <= 10) return 'fall';
  return 'winter';
}
```

### 4. SEO and Traffic Impact

Warn about SEO and traffic implications:

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsUnpublishCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsUnpublish(
    items: Item[],
    ctx: OnBeforeItemsUnpublishCtx
  ): Promise<boolean> {
    const highTrafficWarnings: string[] = [];
    
    for (const item of items) {
      const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
      
      // Check analytics data (if available)
      if (item.attributes.page_views_last_30_days) {
        const pageViews = item.attributes.page_views_last_30_days as number;
        
        if (pageViews > 1000) {
          highTrafficWarnings.push(
            `"${item.attributes.title}" has ${pageViews.toLocaleString()} views in the last 30 days`
          );
        }
      }
      
      // Check SEO rankings
      if (item.attributes.seo_ranking_keywords) {
        const keywords = item.attributes.seo_ranking_keywords as any[];
        const topKeywords = keywords.filter(k => k.position <= 10);
        
        if (topKeywords.length > 0) {
          highTrafficWarnings.push(
            `"${item.attributes.title}" ranks in top 10 for ${topKeywords.length} keywords`
          );
        }
      }
      
      // Check backlinks
      if (item.attributes.external_backlinks_count) {
        const backlinks = item.attributes.external_backlinks_count as number;
        
        if (backlinks > 10) {
          highTrafficWarnings.push(
            `"${item.attributes.title}" has ${backlinks} external backlinks`
          );
        }
      }
      
      // Check if it's a cornerstone content
      if (item.attributes.is_cornerstone_content) {
        highTrafficWarnings.push(
          `"${item.attributes.title}" is marked as cornerstone content`
        );
      }
    }
    
    if (highTrafficWarnings.length > 0) {
      const proceed = await ctx.openConfirm({
        title: 'High Traffic Content',
        content: 
          'You are about to unpublish high-value content:\n\n' +
          highTrafficWarnings.map(w => `• ${w}`).join('\n') +
          '\n\nThis will impact SEO and user experience. Consider redirects.',
        choices: [
          { label: 'Setup Redirects First', value: false },
          { label: 'Unpublish', value: true, intent: 'negative' }
        ]
      });
      
      if (!proceed) {
        // Optionally open redirect setup
        await ctx.navigateTo('/admin/redirects/new');
        return false;
      }
    }
    
    return true;
  }
});
```

### 5. Multi-locale Considerations

Handle multi-locale unpublishing logic:

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsUnpublishCtx } from 'datocms-plugin-sdk';

connect({
  async onBeforeItemsUnpublish(
    items: Item[],
    ctx: OnBeforeItemsUnpublishCtx
  ): Promise<boolean> {
    const localeIssues: string[] = [];
    
    for (const item of items) {
      const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
      
      // Check if content exists in all locales
      if (itemType?.attributes.all_locales_required) {
        const localizedFields = Object.values(ctx.fields).filter(
          f => f.attributes.localized && 
          f.relationships.item_type.data.id === itemType.id
        );
        
        const localesWithContent: Set<string> = new Set();
        
        for (const field of localizedFields) {
          const fieldValue = item.attributes[field.attributes.api_key];
          if (fieldValue && typeof fieldValue === 'object') {
            Object.keys(fieldValue).forEach(locale => {
              if (fieldValue[locale]) localesWithContent.add(locale);
            });
          }
        }
        
        // Check if unpublishing will leave some locales without alternatives
        const affectedLocales = Array.from(localesWithContent);
        
        if (affectedLocales.length !== ctx.site.attributes.locales.length) {
          localeIssues.push(
            `"${item.attributes.title}" is only available in: ${affectedLocales.join(', ')}`
          );
        }
        
        // Check for locale-specific dependencies
        for (const locale of affectedLocales) {
          const localeSpecificLinks = await ctx.searchItems({
            filter: {
              type: 'navigation_item',
              fields: {
                linked_item: { eq: item.id },
                locale: { eq: locale }
              }
            }
          });
          
          if (localeSpecificLinks.length > 0) {
            localeIssues.push(
              `"${item.attributes.title}" is linked in ${locale} navigation`
            );
          }
        }
      }
    }
    
    if (localeIssues.length > 0) {
      const proceed = await ctx.openConfirm({
        title: 'Multi-locale Impact',
        content: 
          'Locale-specific issues found:\n\n' +
          localeIssues.map(i => `• ${i}`).join('\n') +
          '\n\nUnpublish from all locales?',
        choices: [
          { label: 'Review Locales', value: false },
          { label: 'Unpublish All', value: true, intent: 'negative' }
        ]
      });
      
      return proceed;
    }
    
    return true;
  }
});
```

### 6. External System Synchronization

Coordinate with external systems:

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { Item, OnBeforeItemsUnpublishCtx } from 'datocms-plugin-sdk';

// Helper types for external system integration
interface ExternalSystemStatus {
  hasActiveProcesses: boolean;
  processes: string[];
}

interface NotificationPayload {
  action: 'unpublish';
  itemId: string;
  itemType: string;
  scheduledFor: string;
}

interface NotificationResponse {
  acknowledged: boolean;
  message?: string;
}

// Helper function to check external system status
async function checkExternalSystemStatus(
  externalId: string,
  ctx: OnBeforeItemsUnpublishCtx
): Promise<ExternalSystemStatus> {
  try {
    const response = await fetch(
      `${ctx.plugin.attributes.parameters.externalApiUrl}/items/${externalId}/status`,
      {
        headers: {
          'Authorization': `Bearer ${ctx.plugin.attributes.parameters.externalApiKey}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    if (!response.ok) {
      throw new Error(`Status check failed: ${response.statusText}`);
    }
    
    const data = await response.json();
    return {
      hasActiveProcesses: data.activeProcesses?.length > 0,
      processes: data.activeProcesses || []
    };
  } catch (error) {
    // Default to safe state if external system is unreachable
    return {
      hasActiveProcesses: false,
      processes: []
    };
  }
}

// Helper function to notify external system
async function notifyExternalSystem(
  payload: NotificationPayload,
  ctx: OnBeforeItemsUnpublishCtx
): Promise<NotificationResponse> {
  try {
    const response = await fetch(
      `${ctx.plugin.attributes.parameters.externalApiUrl}/notifications`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${ctx.plugin.attributes.parameters.externalApiKey}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(payload)
      }
    );
    
    if (!response.ok) {
      return { acknowledged: false, message: response.statusText };
    }
    
    const data = await response.json();
    return {
      acknowledged: data.acknowledged || false,
      message: data.message
    };
  } catch (error) {
    return { 
      acknowledged: false, 
      message: (error as Error).message 
    };
  }
}

connect({
  async onBeforeItemsUnpublish(
    items: Item[],
    ctx: OnBeforeItemsUnpublishCtx
  ): Promise<boolean> {
    const externalSyncRequired = ['product', 'event', 'course'];
    
    for (const item of items) {
      const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
      
      if (externalSyncRequired.includes(itemType?.attributes.api_key || '')) {
        const externalId = item.attributes.external_id as string;
        
        if (externalId) {
          try {
            // Check external system status
            const externalStatus = await checkExternalSystemStatus(externalId, ctx);
            
            if (externalStatus.hasActiveProcesses) {
              await ctx.alert(
                `Cannot unpublish "${item.attributes.title}".\n\n` +
                `External system has active processes:\n` +
                externalStatus.processes.map(p => `• ${p}`).join('\n')
              );
              return false;
            }
            
            // Notify external system
            const notification = await notifyExternalSystem({
              action: 'unpublish',
              itemId: externalId,
              itemType: itemType.attributes.api_key,
              scheduledFor: new Date().toISOString()
            }, ctx);
            
            if (!notification.acknowledged) {
              const proceed = await ctx.openConfirm({
                title: 'External System Not Ready',
                content: 
                  'The external system did not acknowledge the unpublish request.\n' +
                  'Unpublishing may cause synchronization issues.\n\n' +
                  'Continue anyway?',
                choices: [
                  { label: 'Wait', value: false },
                  { label: 'Force Unpublish', value: true, intent: 'negative' }
                ]
              });
              
              if (!proceed) return false;
            }
          } catch (error) {
            ctx.notice(`External system check failed: ${(error as Error).message}`);
            // Continue anyway - don't block unpublishing due to external system issues
          }
        }
      }
    }
    
    return true;
  }
});
```

## Best Practices

1. **Consider Dependencies**: Always check if other published content depends on items being unpublished
2. **Traffic Awareness**: Warn about high-traffic content to prevent SEO damage
3. **Business Rules**: Enforce minimum content requirements and legal obligations
4. **Clear Communication**: Explain the impact of unpublishing
5. **Offer Alternatives**: Suggest redirects or content archiving instead
6. **External Systems**: Coordinate with external systems but don't let them block critical operations
7. **Audit Trail**: Log unpublish operations for compliance

## Related Hooks

- **onBeforeItemsPublish**: Validate before publishing
- **onBeforeItemsDestroy**: Validate before deletion
- **onBeforeItemUpsert**: Validate individual item operations
- **itemsDropdownActions**: Add bulk unpublish actions

## Important Notes

- This hook runs for both single and bulk unpublish operations
- Returning `false` cancels the entire unpublish operation
- The hook cannot selectively unpublish some items and not others
- Consider the impact on live websites and SEO
- Unpublishing doesn't delete content, it just removes it from public view
- Always provide clear reasons when blocking unpublish operations