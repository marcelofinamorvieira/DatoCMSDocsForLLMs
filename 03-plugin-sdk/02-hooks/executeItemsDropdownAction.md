# executeItemsDropdownAction Hook

## Purpose

The `executeItemsDropdownAction` hook executes bulk actions on multiple records when users select them from the dropdown menu in the content index page. This hook handles operations like bulk updates, exports, deletions, or any custom batch processing you need to perform on selected records.

## Signature

```typescript
async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
): Promise<void>
```

## Parameters

- `actionId`: string - The unique identifier of the action being executed
- `items`: Item[] - Array of selected records on which the action should be executed

## Type Definitions

### Complete Hook Signature

```typescript
export type ExecuteItemsDropdownActionHook = {
  executeItemsDropdownAction: (
    actionId: string,
    items: Item[],
    ctx: ExecuteItemsDropdownActionCtx,
  ) => Promise<void>;
};
```

### Context Type Definition

```typescript
export type ExecuteItemsDropdownActionCtx = {
  // Base Properties
  /** The current plugin configuration */
  plugin: Plugin;
  /** The current token (if a user is set it's a user token, otherwise it's the account token) */
  currentUserAccessToken: string | null;
  
  // Authentication & Permissions
  /** The DatoCMS user that is currently taking the action */
  currentUser: User | SsoUser | null;
  currentRole: Role;
  account: Account;
  
  // Project Information
  /** The current DatoCMS project */
  site: Site;
  /** The ID of the current environment */
  environment: string;
  environmentId: string;
  /** The account/organization that's the project owner */
  owner: Account | Organization;
  /** The account/organization that's the project owner (deprecated, use owner) */
  organization: Account | Organization;
  /** The primary locale for the current DatoCMS project */
  mainLocale: string;
  /** UI preferences of the current user */
  currentUserMenuLocale: string;
  theme: 'dark' | 'light';
  ui: Record<string, unknown>;
  
  // Entity Repositories  
  /** All the models of the current DatoCMS project, indexed by ID */
  itemTypes: ItemType[];
  /** All the fields currently cached, indexed by ID */
  fields: Field[];
  /** All the fieldsets currently cached, indexed by ID */
  fieldsets: Fieldset[];
  /** All the regular users currently cached, indexed by ID */
  users: User[];
  /** All the SSO users currently cached, indexed by ID */
  ssoUsers: SsoUser[];
  
  // Additional properties for this hook
  /** The arbitrary parameters of the dropdown action */
  parameters: Record<string, unknown> | undefined;
  
  // Base Methods
  /** Loads fields for a specific model */
  async loadItemTypeFields(itemTypeId: string): Promise<Field[]>;
  /** Loads fieldsets for a specific model */
  async loadItemTypeFieldsets(itemTypeId: string): Promise<Fieldset[]>;
  /** Loads all the fields that are using the plugin for one of its manual field extensions */
  async loadFieldsUsingPlugin(): Promise<Field[]>;
  /** Loads all regular users */
  async loadUsers(): Promise<User[]>;
  /** Loads all SSO users */
  async loadSsoUsers(): Promise<SsoUser[]>;
  
  /** Updates the plugin parameters */
  async updatePluginParameters(params: Record<string, unknown>): Promise<void>;
  /** Updates the manual field extensions configuration */
  async updateFieldAppearance(
    fieldId: string,
    changes: FieldAppearanceChange[]
  ): Promise<void>;
  
  // Toast Notifications
  /** Shows a custom toast with call-to-action */
  async customToast(toast: Toast): Promise<unknown>;
  /** Shows a notice-level toast (green) */
  notice(message: string): void;
  /** Shows an alert-level toast (red) */
  alert(message: string): void;
  
  // Item (Record) Dialogs
  /** Opens a dialog for selecting one (or multiple) record(s) */
  async selectItem(
    itemTypeId: string,
    options?: {
      multiple?: boolean;
      locale?: string;
      filters?: ItemListFilter;
      initialLocationQuery?: LocationQuery;
    }
  ): Promise<Item | Item[] | null>;
  
  /** Opens a dialog for creating a new record */
  async createNewItem(
    itemTypeId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<Item | null>;
  
  /** Opens a dialog for editing a record */
  async editItem(
    itemId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<Item | null>;
  
  // Upload (Asset) Dialogs  
  /** Opens a dialog for selecting one (or multiple) upload(s) */
  async selectUpload(options?: {
    multiple?: boolean;
    locale?: string;
    filters?: UploadListFilter;
  }): Promise<Upload | Upload[] | null>;
  
  /** Opens a dialog for editing an upload */
  async editUpload(
    uploadId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<Upload | null>;
  
  /** Opens a dialog for editing upload metadata */
  async editUploadMetadata(
    fileFieldValue: FileFieldValue,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalValues?: FileFieldValue;
    }
  ): Promise<FileFieldValue | null>;
  
  /** Opens a dialog for editing multiple uploads */
  async editUploads(
    uploadIds: string[],
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<void>;
  
  // Custom Dialogs
  /** Opens a custom modal. Returns the value passed as the modal's result */
  async openModal(
    modalId: string,
    options?: {
      title?: string;
      width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
      height?: 's' | 'm' | 'l' | number;
      initialHeight?: number;
      closeDisabled?: boolean;
      parameters?: Record<string, unknown>;
    }
  ): Promise<unknown>;
  
  /** Opens a confirm dialog. Returns the selected choice */
  async openConfirm(options: {
    title: string;
    content?: ReactElement | string;
    choices?: Array<{
      label: string;
      value: unknown;
      icon?: Icon;
      intent?: 'positive' | 'negative';
    }>;
    cancel?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  
  // Navigation
  /** Navigates to a specific location in the DatoCMS interface */
  async navigateTo(
    location: 
      | string // URL path
      | { 
          pageId: string;
          params?: Record<string, string>;
          query?: Record<string, string>;
          locale?: string;
        }
  ): Promise<void>;
  
  // Utility Methods
  /** Returns the internationalized title of a field */
  getFieldIntlTitle(
    field: Field,
    locale?: string,
    returnUntranslatedTitle?: boolean
  ): string | null;
  
  /** Creates a new API client configured for the current environment */
  createClient(options?: { 
    apiToken?: string;
    environment?: string;
  }): SiteApiClient;
};
```

### Item Type

```typescript
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
        type: 'account' | 'user' | 'sso_user';
      } | null;
    };
    fields?: {
      data: Array<{
        id: string;
        type: 'field';
      }>;
    };
    parent?: {
      data: {
        id: string;
        type: 'item';
      } | null;
    };
    children?: {
      data: Array<{
        id: string;
        type: 'item';
      }>;
    };
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at: string | null;
    publication_scheduled_at: string | null;
    unpublishing_scheduled_at: string | null;
    first_published_at: string | null;
    is_valid: boolean;
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    current_version: string;
    stage: string | null;
    status: 'draft' | 'published' | 'updated';
  };
}
```

### Supporting Types

```typescript
// Plugin Type
interface Plugin {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description: string;
    homepage_url: string;
    version: string;
    icon: string;
    parameters: Record<string, unknown>;
    permissions: string[];
    version: string;
  };
}

// User Types
interface User {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    created_at: string;
    is_2fa_active: boolean;
    last_sign_in_at: string | null;
  };
}

interface SsoUser {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    created_at: string;
  };
}

interface Account {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    company: string | null;
  };
}

interface Organization {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
  };
}

// Site Type
interface Site {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    domain: string | null;
    global_seo: Record<string, any>;
    favicon: Upload | null;
    locales: string[];
    timezone: string;
  };
}

// Role Type
interface Role {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_site: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_manage_build_triggers: boolean;
    can_manage_menu_items: boolean;
    can_edit_favicon: boolean;
    can_edit_seo_settings: boolean;
    can_manage_workflows: boolean;
    environments_access: 'primary_only' | 'sandbox_only' | 'all';
    positive_item_type_permissions: Array<{
      item_type_id: string;
      action: 'all' | 'create' | 'update' | 'delete' | 'publish' | 'edit_metadata';
    }>;
    negative_item_type_permissions: Array<{
      item_type_id: string;
      action: 'all' | 'create' | 'update' | 'delete' | 'publish' | 'edit_metadata';
    }>;
  };
}

// ItemType Type
interface ItemType {
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
    ordering_field: string | null;
    all_locales_required: boolean;
    collection_appearance: 'table' | 'gallery';
    hint?: string | null;
    inverse_relationships_enabled: boolean;
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
  meta: {
    has_singleton_item: boolean;
  };
}

// Field Type
interface Field {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    field_type: string;
    api_key: string;
    localized: boolean;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      addons: Array<{
        id: string;
        parameters: Record<string, any>;
      }>;
    };
    default_value: any;
    hint?: string | null;
    placeholder?: string | null;
    appeareance?: Record<string, any>; // Legacy, use appearance
    position: number;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
    fieldset?: {
      data: {
        id: string;
        type: 'fieldset';
      } | null;
    };
  };
  meta: {
    valid_validators: Array<{
      type: string;
      config: Record<string, any>;
    }>;
  };
}

// Fieldset Type
interface Fieldset {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
    hint?: string | null;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
  };
}

// Upload Type
interface Upload {
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
    focal_point: { x: number; y: number } | null;
    url: string;
    blurhash?: string;
    colors?: Array<{ hex: string; alpha: number }>;
    thumbhash?: string | null;
    basename: string;
    is_image: boolean;
    mime_type: string;
    default_field_metadata: {
      [locale: string]: {
        alt: string | null;
        title: string | null;
        custom_data: Record<string, string>;
        focal_point: { x: number; y: number } | null;
      };
    };
  };
  relationships: {
    creator?: {
      data: {
        id: string;
        type: 'account' | 'user' | 'sso_user';
      } | null;
    };
    upload_collection?: {
      data: {
        id: string;
        type: 'upload_collection';
      } | null;
    };
  };
  meta: {
    created_at: string;
    updated_at: string;
  };
}

// Toast Type
interface Toast {
  message: string;
  type: 'notice' | 'alert' | 'warning' | 'primary';
  dismissOnPageChange?: boolean;
  dismissAfterMs?: number;
  cta?: {
    label: string;
    value: unknown;
  };
}

// Modal Type
interface Modal {
  id: string;
  title?: string;
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  height?: 's' | 'm' | 'l' | number;
  initialHeight?: number;
  closeDisabled?: boolean;
  parameters?: Record<string, unknown>;
}

// Icon Type
type Icon = string | {
  type: 'svg';
  viewBox: string;
  content: string;
};

// Filter Types
interface ItemListFilter {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
}

interface LocationQuery {
  locale?: string;
  page?: number;
  perPage?: number;
  sortBy?: string;
  sortDirection?: 'asc' | 'desc';
  filter?: Record<string, unknown>;
}

interface UploadListFilter {
  type?: 'all' | 'image' | 'video' | 'audio' | 'document' | 'archive' | 'other';
  query?: string;
}

// File Field Types
interface FileFieldValue {
  upload_id: string;
  alt: string | null;
  title: string | null;
  focal_point: FocalPoint | null;
  custom_data: Record<string, string>;
}

interface FileFieldMetadata {
  alt: string | null;
  title: string | null;
  focal_point: FocalPoint | null;
  custom_data: Record<string, string>;
}

interface FocalPoint {
  x: number;
  y: number;
}

// Callback Types
interface FinalizeCallback {
  item?: Item;
  itemDidChange: boolean;
}

// Field Appearance Change Type
type FieldAppearanceChange =
  | { operation: 'removeEditor' }
  | { operation: 'updateEditor'; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'setEditor'; fieldExtensionId: string; parameters: Record<string, unknown> }
  | { operation: 'removeAddon'; index: number }
  | { operation: 'updateAddon'; index: number; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'insertAddon'; index: number; fieldExtensionId: string; parameters: Record<string, unknown> };

// API Client Type
interface SiteApiClient {
  // Full typed client from @datocms/cma-client
  items: {
    list(params?: ItemListParams): Promise<Item[]>;
    find(itemId: string): Promise<Item>;
    create(params: ItemCreateParams): Promise<Item>;
    update(itemId: string, params: ItemUpdateParams): Promise<Item>;
    destroy(itemId: string): Promise<void>;
    publish(itemId: string): Promise<Item>;
    unpublish(itemId: string): Promise<Item>;
    // ... other methods
  };
  itemTypes: {
    list(): Promise<ItemType[]>;
    find(itemTypeId: string): Promise<ItemType>;
    // ... other methods
  };
  fields: {
    list(itemTypeId: string): Promise<Field[]>;
    find(fieldId: string): Promise<Field>;
    // ... other methods
  };
  uploads: {
    list(params?: UploadListParams): Promise<Upload[]>;
    find(uploadId: string): Promise<Upload>;
    create(params: UploadCreateParams): Promise<Upload>;
    update(uploadId: string, params: UploadUpdateParams): Promise<Upload>;
    destroy(uploadId: string): Promise<void>;
    // ... other methods
  };
  // ... other resources
}

// Parameter types for API operations
interface ItemListParams {
  filter?: {
    type?: string;
    fields?: Record<string, unknown>;
    query?: string;
  };
  order_by?: string[];
  page?: {
    offset?: number;
    limit?: number;
  };
  locale?: string;
  nested?: boolean;
}

interface ItemCreateParams {
  type: 'item';
  attributes: Record<string, any>;
  relationships?: {
    item_type: {
      data: {
        type: 'item_type';
        id: string;
      };
    };
  };
  meta?: {
    created_at?: string;
    updated_at?: string;
  };
}

interface ItemUpdateParams {
  type: 'item';
  id: string;
  attributes?: Record<string, any>;
  meta?: {
    updated_at?: string;
  };
}

interface UploadListParams {
  filter?: {
    type?: 'all' | 'image' | 'video' | 'audio' | 'document' | 'archive' | 'other';
    query?: string;
  };
  order_by?: string[];
  page?: {
    offset?: number;
    limit?: number;
  };
}

interface UploadCreateParams {
  path: string | File | Buffer;
  default_field_metadata?: Record<string, FileFieldMetadata>;
  author?: string;
  copyright?: string;
  notes?: string;
}

interface UploadUpdateParams {
  default_field_metadata?: Record<string, FileFieldMetadata>;
  author?: string;
  copyright?: string;
  notes?: string;
}
```

## Context Properties

The hook receives a rich context object (`ExecuteItemsDropdownActionCtx`) that provides access to all DatoCMS functionality:

### Important Notes
- The `items` parameter contains the full array of selected records with all their attributes, metadata, and relationships
- The context does NOT provide direct CMA client methods - you need to use `buildClient` from `@datocms/cma-client` for API operations
- Consider using batching for operations on large datasets to avoid timeouts
- The `parameters` property contains custom parameters passed when defining the action via `itemsDropdownActions`

## Use Cases

### 1. Bulk Content Updates

Update multiple records at once:

```typescript
import { buildClient } from '@datocms/cma-client';

async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
) {
  if (actionId === 'bulk-update-status') {
    if (items.length === 0) {
      await ctx.alert('No items selected');
      return;
    }

    // Show update options modal
    const updateOptions = await ctx.openModal({
      id: 'bulk-status-update',
      title: `Update Status for ${items.length} Items`,
      width: 's',
      parameters: {
        currentStatuses: [...new Set(items.map(i => i.attributes.status))],
        itemCount: items.length
      }
    }) as { status: string; reviewStatus: string } | null;

    if (!updateOptions) return;

    try {
      await ctx.notice(`Updating ${items.length} items...`);

      // Initialize CMA client
      const client = buildClient({
        apiToken: ctx.currentUserAccessToken || '',
        environment: ctx.environment
      });

      // Update in batches to avoid timeouts
      const batchSize = 50;
      
      for (let i = 0; i < items.length; i += batchSize) {
        const batch = items.slice(i, i + batchSize);
        const progress = Math.round((i / items.length) * 100);
        
        // Update progress in modal if needed
        await ctx.openModal({
          id: 'progress-indicator',
          title: 'Updating Items',
          width: 's',
          parameters: {
            progress,
            message: `Processing batch ${Math.floor(i / batchSize) + 1}...`
          }
        });
        
        // Update each item in the batch
        await Promise.all(
          batch.map(item =>
            client.items.update(item.id, {
              ...item.attributes,
              status: updateOptions.status,
              review_status: updateOptions.reviewStatus,
              updated_by: ctx.currentUser.id
            })
          )
        );
      }

      await ctx.notice(`✅ Successfully updated ${items.length} items`);
    } catch (error: any) {
      await ctx.alert(`Failed to update items: ${error.message}`);
    }
  }

  if (actionId === 'bulk-categorize') {
    // Get the item type from the first item's relationship
    const itemTypeId = items[0]?.relationships.item_type.data.id;
    if (!itemTypeId) return;

    // Get category field
    const categoryField = Object.values(ctx.fields).find(
      f => f.attributes.api_key === 'category' && f.relationships.item_type.data.id === itemTypeId
    );

    if (!categoryField) {
      await ctx.alert('No category field found in this model');
      return;
    }

    // Let user select categories
    const categories = await ctx.openModal({
      id: 'category-selector',
      title: 'Bulk Categorize',
      width: 'm',
      parameters: {
        items,
        field: categoryField,
        multiple: true
      }
    }) as string[] | null;

    if (!categories || categories.length === 0) return;

    try {
      // Initialize CMA client
      const client = buildClient({
        apiToken: ctx.currentUserAccessToken || '',
        environment: ctx.environment
      });

      // Group items by existing categories for smart updates
      const itemsByCategory: Record<string, typeof items> = {};
      items.forEach(item => {
        const key = item.attributes.category || 'uncategorized';
        if (!itemsByCategory[key]) itemsByCategory[key] = [];
        itemsByCategory[key].push(item);
      });

      // Update each group
      for (const [oldCategory, groupItems] of Object.entries(itemsByCategory)) {
        const action = await ctx.openConfirm({
          title: 'Update Strategy',
          content: `${groupItems.length} items currently in "${oldCategory}". How to update?`,
          choices: [
            { label: 'Replace', value: 'replace' },
            { label: 'Append', value: 'append' },
            { label: 'Skip', value: 'skip' }
          ],
          cancel: { label: 'Cancel', value: 'cancel' }
        }) as string;

        if (action === 'skip' || action === 'cancel') continue;

        const updateData = action === 'replace' 
          ? { category: categories }
          : { category: [...new Set([...(groupItems[0].attributes.category || []), ...categories])] };

        // Update items in this group
        await Promise.all(
          groupItems.map(item =>
            client.items.update(item.id, {
              ...item.attributes,
              ...updateData
            })
          )
        );
      }

      await ctx.notice('Categories updated successfully');
    } catch (error: any) {
      await ctx.alert(`Categorization failed: ${error.message}`);
    }
  }
}
```

### 2. Bulk Export Operations

Export selected records in various formats:

```typescript
async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
) {
  if (actionId === 'export-csv') {
    if (items.length === 0) {
      await ctx.alert('No items selected for export');
      return;
    }
    
    // Get the item type from the first item
    const itemTypeId = items[0]?.relationships.item_type.data.id;
    const itemType = ctx.itemTypes.find(it => it.id === itemTypeId);
    if (!itemType) return;

    try {
      await ctx.notice('Preparing CSV export...');

      // Get exportable fields
      const fields = Object.values(ctx.fields).filter(
        f => ['string', 'text', 'integer', 'float', 'boolean', 'date', 'datetime'].includes(f.attributes.field_type)
      );

      // Build CSV
      const csv = [];
      
      // Header row
      const headers = ['ID', 'Created', 'Updated', 'Status'];
      fields.forEach(field => {
        headers.push(field.attributes.label);
      });
      csv.push(headers);

      // Data rows
      for (let index = 0; index < items.length; index++) {
        const item = items[index];
        // Update progress in modal if needed
        await ctx.openModal({
          id: 'progress-indicator',
          title: 'Processing Items',
          width: 's',
          closeDisabled: true,
          parameters: {
            progress: Math.round((index / items.length) * 100),
            message: `Processing item ${index + 1} of ${items.length}`
          }
        });

        const row = [
          item.id,
          item.meta.created_at,
          item.meta.updated_at,
          item.meta.status
        ];

        fields.forEach(field => {
          const value = item[field.attributes.api_key];
          // Handle different field types
          if (value === null || value === undefined) {
            row.push('');
          } else if (typeof value === 'object') {
            row.push(JSON.stringify(value));
          } else {
            row.push(String(value).replace(/"/g, '""')); // Escape quotes
          }
        });

        csv.push(row);
      }

      // Convert to CSV string
      const csvContent = csv.map(row => 
        row.map(cell => `"${cell}"`).join(',')
      ).join('\n');

      // Download
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${itemType.attributes.api_key}-export-${Date.now()}.csv`;
      link.click();
      URL.revokeObjectURL(url);

      await ctx.notice(`Exported ${items.length} items to CSV`);
    } catch (error: any) {
      await ctx.alert(`Export failed: ${error.message}`);
    }
  }

  if (actionId === 'export-json') {
    if (items.length === 0) return;
    
    // Get the item type from the first item
    const itemTypeId = items[0]?.relationships.item_type.data.id;
    const itemType = ctx.itemTypes.find(it => it.id === itemTypeId);
    if (!itemType) return;

    // Options for JSON export
    const options = await ctx.openModal({
      id: 'json-export-options',
      title: 'JSON Export Options',
      width: 's',
      parameters: {
        itemCount: items.length,
        hasLocalizedFields: Object.values(ctx.fields).some(f => f.attributes.localized)
      }
    });

    if (!options) return;

    try {
      const exportData = {
        exported_at: new Date().toISOString(),
        model: itemType.attributes.api_key,
        locale: options.includeAllLocales ? 'all' : ctx.ui.locale,
        count: items.length,
        items: []
      };

      // Process each item
      for (let i = 0; i < items.length; i++) {
        // Show progress
        await ctx.openModal({
          id: 'export-progress',
          title: 'Exporting Items',
          width: 's',
          closeDisabled: true,
          parameters: {
            progress: Math.round((i / items.length) * 100),
            message: `Exporting item ${i + 1} of ${items.length}`
          }
        });

        const item = items[i];
        const itemData = {
          id: item.id,
          ...item.attributes
        };

        if (options.includeMeta) {
          itemData._meta = item.meta;
        }

        if (options.includeRelationships) {
          // You would need to fetch full item with relationships using CMA client
          const client = buildClient({
            apiToken: ctx.currentUserAccessToken || '',
            environment: ctx.environment
          });
          const fullItem = await client.items.find(item.id, { include: 'all' });
          itemData._relationships = fullItem.relationships;
        }

        exportData.items.push(itemData);
      }

      // Download JSON
      const blob = new Blob([JSON.stringify(exportData, null, 2)], {
        type: 'application/json'
      });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${itemType.attributes.api_key}-export-${Date.now()}.json`;
      link.click();
      URL.revokeObjectURL(url);

      await ctx.notice(`Exported ${items.length} items to JSON`);
    } catch (error: any) {
      await ctx.alert(`Export failed: ${error.message}`);
    }
  }
}
```

### 3. Bulk Publishing Operations

Handle publishing workflows for multiple items:

```typescript
import { buildClient } from '@datocms/cma-client';

async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
) {
  if (actionId === 'publish-with-validation') {
    const unpublishedItems = items.filter(i => i.meta.status !== 'published');
    
    if (unpublishedItems.length === 0) {
      await ctx.alert('All selected items are already published');
      return;
    }

    // Initialize CMA client
    const client = buildClient({
      apiToken: ctx.currentUserAccessToken || '',
      environment: ctx.environment
    });

    // Validate items before publishing
    await ctx.notice('Validating items...');
    const validationResults: Array<{ item: Item; issues: string[] }> = [];

    for (let i = 0; i < unpublishedItems.length; i++) {
      const item = unpublishedItems[i];
      
      // Show progress in modal
      await ctx.openModal({
        id: 'validation-progress',
        title: 'Validating Items',
        width: 's',
        closeDisabled: true,
        parameters: {
          progress: Math.round((i / unpublishedItems.length) * 50),
          message: `Validating ${item.id}`
        }
      });

      const issues = await validateItemForPublishing(item, ctx);
      if (issues.length > 0) {
        validationResults.push({ item, issues });
      }
    }

    if (validationResults.length > 0) {
      // Show validation issues
      const proceed = await ctx.openModal({
        id: 'validation-issues',
        title: 'Validation Issues Found',
        width: 'l',
        parameters: {
          results: validationResults,
          totalItems: unpublishedItems.length
        }
      }) as boolean;

      if (!proceed) return;
    }

    // Publish items
    try {
      const validItems = unpublishedItems
        .filter(item => !validationResults.find(r => r.item.id === item.id));

      for (let i = 0; i < validItems.length; i += 20) {
        const batch = validItems.slice(i, i + 20);
        
        await ctx.openModal({
          id: 'publish-progress',
          title: 'Publishing Items',
          width: 's',
          closeDisabled: true,
          parameters: {
            progress: 50 + Math.round((i / validItems.length) * 50),
            message: `Publishing batch ${Math.floor(i / 20) + 1}`
          }
        });
        
        // Publish each item in the batch
        await Promise.all(
          batch.map(item => client.items.publish(item.id))
        );
      }

      await ctx.notice(`✅ Published ${validItems.length} items successfully`);
    } catch (error: any) {
      await ctx.alert(`Publishing failed: ${error.message}`);
    }
  }

  if (actionId === 'schedule-publishing') {
    const { items } = ctx;

    const unpublishedItems = items.filter(i => i.meta.status !== 'published');
    
    if (unpublishedItems.length === 0) {
      ctx.alert('All selected items are already published');
      return;
    }

    // Get scheduling options
    const schedule = await ctx.openModal({
      id: 'schedule-publisher',
      title: `Schedule Publishing for ${unpublishedItems.length} Items`,
      width: 'm'
    });

    if (!schedule) return;

    try {
      // Create scheduled jobs
      const jobs = [];
      
      for (const item of unpublishedItems) {
        const job = await createScheduledPublishJob({
          itemId: item.id,
          publishAt: schedule.datetime,
          timezone: schedule.timezone,
          notifyEmail: schedule.notifyEmail
        });
        jobs.push(job);
      }

      ctx.notice(`Scheduled ${jobs.length} items for publishing at ${schedule.datetime}`);
      
      // Optionally show summary
      await ctx.openModal({
        id: 'schedule-summary',
        title: 'Publishing Scheduled',
        width: 's',
        parameters: { jobs, schedule }
      });
    } catch (error) {
      ctx.alert(`Scheduling failed: ${error.message}`);
    }
  }
}
```

### 4. Bulk Delete with Safeguards

Safely delete multiple items with confirmation:

```typescript
import { buildClient } from '@datocms/cma-client';

async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
) {
  if (actionId === 'bulk-delete') {
    if (items.length === 0) return;

    // Initialize CMA client
    const client = buildClient({
      apiToken: ctx.currentUserAccessToken || '',
      environment: ctx.environment
    });

    // Get item type details
    const itemTypeId = items[0]?.relationships.item_type.data.id;
    const itemType = ctx.itemTypes[itemTypeId];

    // Check for published items
    const publishedItems = items.filter(i => i.meta.status === 'published');
    const hasPublished = publishedItems.length > 0;

    // Build confirmation message
    let message = `You are about to delete ${items.length} ${itemType?.attributes.name || 'records'}.`;
    
    if (hasPublished) {
      message += `\n\n⚠️ Warning: ${publishedItems.length} items are currently published.`;
    }

    // Check for relationships (you would implement this function)
    const relationships = await checkItemRelationships(items.map(i => i.id), client);
    if (relationships.length > 0) {
      message += `\n\n🔗 These items are referenced by:`;
      relationships.forEach(rel => {
        message += `\n- ${rel.count} ${rel.model} records`;
      });
    }

    // Multi-step confirmation for safety
    const firstConfirm = await ctx.openConfirm({
      title: 'Confirm Deletion',
      content: message,
      choices: [
        { label: 'Continue', value: true, intent: 'negative' }
      ],
      cancel: { label: 'Cancel', value: false }
    }) as boolean;

    if (!firstConfirm) return;

    // Second confirmation with type-to-confirm
    const deleteConfirmation = await ctx.openModal({
      id: 'delete-confirmation',
      title: 'Final Confirmation',
      width: 's',
      parameters: {
        itemCount: items.length,
        requireTyping: true,
        confirmText: `delete ${items.length} items`
      }
    }) as boolean;

    if (!deleteConfirmation) return;

    try {
      // Unpublish first if needed
      if (hasPublished) {
        await ctx.notice('Unpublishing items first...');
        
        await Promise.all(
          publishedItems.map(item => client.items.unpublish(item.id))
        );
      }

      // Delete in batches
      const batchSize = 25;
      
      for (let i = 0; i < items.length; i += batchSize) {
        const batch = items.slice(i, i + batchSize);
        
        await ctx.openModal({
          id: 'delete-progress',
          title: 'Deleting Items',
          width: 's',
          closeDisabled: true,
          parameters: {
            progress: Math.round((i / items.length) * 100),
            message: `Deleting items ${i + 1} to ${Math.min(i + batchSize, items.length)}`
          }
        });
        
        // Delete each item in the batch
        await Promise.all(
          batch.map(item => client.items.destroy(item.id))
        );
      }

      await ctx.notice(`✅ Successfully deleted ${items.length} items`);
    } catch (error: any) {
      await ctx.alert(`Deletion failed: ${error.message}`);
    }
  }

  if (actionId === 'archive-items') {
    // Move to archive instead of deleting
    const archiveField = ctx.fields.find(
      f => f.attributes.api_key === 'archived' || f.attributes.api_key === 'is_archived'
    );
    
    if (!archiveField) {
      await ctx.alert('No archive field found. Add an "archived" boolean field to use this feature.');
      return;
    }

    const confirmed = await ctx.openConfirm({
      title: 'Archive Items',
      content: `Archive ${items.length} items? They will be hidden but can be restored later.`,
      choices: [
        { label: 'Cancel', value: false },
        { label: 'Archive', value: true }
      ]
    });

    if (!confirmed) return;

    try {
      // Initialize CMA client
      const client = buildClient({
        apiToken: ctx.currentUserAccessToken || '',
        environment: ctx.environment
      });
      
      // Update items to archive them
      await Promise.all(
        items.map(item =>
          client.items.update(item.id, {
            ...item.attributes,
            [archiveField.attributes.api_key]: true
          })
        )
      );

      await ctx.notice(`Archived ${items.length} items`);
    } catch (error: any) {
      await ctx.alert(`Archive failed: ${error.message}`);
    }
  }
}
```

### 5. Content Analysis and Reporting

Analyze selected content and generate reports:

```typescript
async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
) {
  if (actionId === 'content-analysis') {
    if (items.length === 0) return;
    
    // Get the item type from the first item
    const itemTypeId = items[0]?.relationships.item_type.data.id;
    const itemType = ctx.itemTypes.find(it => it.id === itemTypeId);
    if (!itemType) return;

    await ctx.notice('Analyzing content...');

    const analysis = {
      totalItems: items.length,
      byStatus: {},
      byAuthor: {},
      contentLength: [],
      missingFields: {},
      seoScores: []
    };

    // Analyze each item
    for (let i = 0; i < items.length; i++) {
      const item = items[i];
      // Show progress
      await ctx.openModal({
        id: 'analysis-progress',
        title: 'Analyzing Content',
        width: 's',
        closeDisabled: true,
        parameters: {
          progress: Math.round((i / items.length) * 100),
          message: `Analyzing item ${i + 1} of ${items.length}`
        }
      });

      // Status distribution
      analysis.byStatus[item.meta.status] = (analysis.byStatus[item.meta.status] || 0) + 1;

      // Author distribution
      const author = item.meta.created_by || 'Unknown';
      analysis.byAuthor[author] = (analysis.byAuthor[author] || 0) + 1;

      // Content length analysis
      const contentField = item.content || item.body || item.description;
      if (contentField) {
        const wordCount = contentField.split(/\s+/).filter(Boolean).length;
        analysis.contentLength.push({ id: item.id, wordCount });
      }

      // Check for missing required fields
      Object.values(ctx.fields).forEach(field => {
        if (field.attributes.validators?.required && !item[field.attributes.api_key]) {
          if (!analysis.missingFields[field.attributes.api_key]) {
            analysis.missingFields[field.attributes.api_key] = [];
          }
          analysis.missingFields[field.attributes.api_key].push(item.id);
        }
      });

      // SEO analysis
      if (item.seo_title || item.meta_description) {
        const seoScore = calculateSEOScore(item);
        analysis.seoScores.push({ id: item.id, score: seoScore });
      }
    }

    // Show analysis results
    await ctx.openModal({
      id: 'analysis-results',
      title: 'Content Analysis Report',
      width: 'xl',
      parameters: { analysis, itemType }
    });
  }

  if (actionId === 'generate-report') {
    if (items.length === 0) return;
    
    // Get the item type from the first item
    const itemTypeId = items[0]?.relationships.item_type.data.id;
    const itemType = ctx.itemTypes.find(it => it.id === itemTypeId);
    if (!itemType) return;

    // Get report options
    const reportOptions = await ctx.openModal({
      id: 'report-options',
      title: 'Generate Report',
      width: 'm',
      parameters: {
        itemCount: items.length,
        availableFormats: ['pdf', 'excel', 'html']
      }
    });

    if (!reportOptions) return;

    try {
      await ctx.notice('Generating report...');

      const reportData = await generateReport({
        items,
        format: reportOptions.format,
        includeCharts: reportOptions.includeCharts,
        groupBy: reportOptions.groupBy,
        fields: reportOptions.selectedFields
      });

      // Download report
      const blob = new Blob([reportData.content], { type: reportData.mimeType });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${itemType.attributes.api_key}-report-${Date.now()}.${reportOptions.format}`;
      link.click();
      URL.revokeObjectURL(url);

      await ctx.notice('Report generated successfully');
    } catch (error: any) {
      await ctx.alert(`Report generation failed: ${error.message}`);
    }
  }
}
```

### 6. Integration and Sync Operations

Sync selected items with external systems:

```typescript
async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
) {
  if (actionId === 'sync-to-external') {

    // Check integration config
    const integration = ctx.plugin.attributes.parameters.integration;
    if (!integration?.enabled) {
      await ctx.alert('External integration not configured');
      return;
    }

    // Preview sync operation
    const syncPreview = await ctx.openModal({
      id: 'sync-preview',
      title: 'Sync Preview',
      width: 'l',
      parameters: {
        items,
        destination: integration.name,
        lastSync: integration.lastSync
      }
    });

    if (!syncPreview?.confirm) return;

    try {
      const results = {
        success: 0,
        failed: 0,
        errors: []
      };

      // Sync each item
      for (let i = 0; i < items.length; i++) {
        const item = items[i];
        // Show progress
        await ctx.openModal({
          id: 'sync-progress',
          title: 'Syncing Items',
          width: 's',
          closeDisabled: true,
          parameters: {
            progress: Math.round((i / items.length) * 100),
            message: `Syncing ${item.id}`
          }
        });

        try {
          await syncItemToExternal(item, integration);
          results.success++;
        } catch (error) {
          results.failed++;
          results.errors.push({
            itemId: item.id,
            error: error.message
          });
        }
      }

      // Show results
      if (results.failed > 0) {
        await ctx.openModal({
          id: 'sync-errors',
          title: 'Sync Completed with Errors',
          width: 'm',
          parameters: results
        });
      } else {
        await ctx.notice(`✅ Successfully synced ${results.success} items`);
      }

      // Update last sync timestamp
      await ctx.updatePluginParameters({
        ...ctx.plugin.attributes.parameters,
        integration: {
          ...integration,
          lastSync: new Date().toISOString()
        }
      });
    } catch (error) {
      await ctx.alert(`Sync failed: ${error.message}`);
    }
  }

  if (actionId === 'queue-for-translation') {
    // Get current locale from UI context
    const locale = ctx.ui.locale || ctx.mainLocale;

    // Get target locales
    const targetLocales = await ctx.openModal({
      id: 'locale-selector',
      title: 'Queue for Translation',
      width: 'm',
      parameters: {
        items,
        sourceLocale: locale,
        availableLocales: ctx.site.attributes.locales.filter((l: string) => l !== locale)
      }
    });

    if (!targetLocales || targetLocales.length === 0) return;

    try {
      const queue = [];

      for (const item of items) {
        for (const targetLocale of targetLocales) {
          const itemTypeId = item.relationships.item_type.data.id;
          const itemType = ctx.itemTypes.find(it => it.id === itemTypeId);
          
          queue.push({
            itemId: item.id,
            itemType: itemType?.attributes.api_key || 'unknown',
            sourceLocale: locale,
            targetLocale,
            fields: getTranslatableFields(item, ctx.fields),
            priority: item.meta.status === 'published' ? 'high' : 'normal'
          });
        }
      }

      // Send to translation service
      const response = await queueTranslations(queue);

      await ctx.notice(`Queued ${queue.length} translations. Job ID: ${response.jobId}`);

      // Show tracking info
      await ctx.openModal({
        id: 'translation-tracking',
        title: 'Translation Queued',
        width: 's',
        parameters: {
          jobId: response.jobId,
          estimatedTime: response.estimatedTime,
          trackingUrl: response.trackingUrl
        }
      });
    } catch (error) {
      await ctx.alert(`Translation queue failed: ${error.message}`);
    }
  }
}
```

## Best Practices

1. **Handle Large Datasets**: Process items in batches to avoid timeouts
2. **Show Progress**: Use `reportProgress()` for long operations
3. **Validate Before Acting**: Check permissions and data validity
4. **Confirm Destructive Actions**: Always confirm before deleting/modifying
5. **Handle Errors Gracefully**: Catch and report errors clearly
6. **Respect Rate Limits**: Don't overwhelm the API with requests
7. **Provide Undo Options**: When possible, allow users to reverse actions

## Error Handling Pattern

```typescript
async executeItemsDropdownAction(
  actionId: string,
  items: Item[],
  ctx: ExecuteItemsDropdownActionCtx
) {
  try {
    // Validate selection
    if (items.length === 0) {
      await ctx.alert('No items selected');
      return;
    }

    // Check permissions based on role
    if (!ctx.currentRole.attributes.can_edit_schema) {
      await ctx.alert('You do not have permission to perform this action');
      return;
    }

    // Execute action
    await performBulkAction(items, ctx);
    
  } catch (error: any) {
    console.error('Bulk action failed:', error);
    await ctx.alert(`Operation failed: ${error.message}`);
  }
}
```

## Related Hooks

- **itemsDropdownActions**: Defines available bulk actions
- **executeItemFormDropdownAction**: Single item actions
- **onBeforeItemsDestroy**: Validate before bulk delete
- **onBeforeItemsPublish**: Validate before bulk publish

## Important Notes

- Actions operate on arrays of items, handle empty selections
- Use batching for operations on many items (>50)
- Progress reporting helps users understand long operations
- Some operations may require elevated permissions
- Always provide feedback on operation success/failure
- Consider performance impact of operations on large datasets
- Respect the current locale context for localized fields