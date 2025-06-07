# itemsDropdownActions Hook

The `itemsDropdownActions` hook allows plugins to add custom dropdown actions to the content area when multiple items are selected. This enables batch operations on multiple content items at once, such as bulk updates, exports, or custom workflows.

## Purpose

Add custom batch actions for multiple selected items in the content area. Perfect for bulk operations like:
- Exporting multiple items to CSV/JSON
- Bulk publishing/unpublishing
- Applying templates to multiple items
- Sending items to external services
- Generating reports from selected data

## Signature

```typescript
// The hook function signature
itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] | DropdownActionGroup[] | undefined

// Used in plugin definition
connect({
  itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] | DropdownActionGroup[] | undefined {
    // Return dropdown actions based on context
    return [
      {
        id: 'my-action',
        label: 'My Custom Action',
        icon: 'FaRocket'
      }
    ];
  }
});
```

## Parameters

### ctx: ItemsDropdownActionsCtx
The context object containing all properties and methods for the dropdown actions

## Type Definitions

```typescript
// Complete type definitions for itemsDropdownActions hook
export type ItemsDropdownActionsHook = {
  itemsDropdownActions: (ctx: ItemsDropdownActionsCtx) => DropdownAction[] | DropdownActionGroup[] | undefined;
};

export type ItemsDropdownActionsCtx = BasePluginFrameCtx<
  'itemsDropdownActions',
  {
    locale: string;
    selectedItemIds: string[];
    selectedItems: Item[];
  }
>;

// Base plugin frame context
type BasePluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BaseProperties &
  BaseMethods &
  (Mode extends 'renderManualFieldExtensionConfigScreen' | 'validateManualFieldExtensionParameters'
    ? ManualFieldExtensionConfigMethods
    : Record<string, never>) &
  (Mode extends
    | 'renderItemFormSidebar'
    | 'renderItemFormSidebarPanel'
    | 'renderFieldExtension'
    | 'renderItemFormOutlet'
    | 'itemFormDropdownActions'
    | 'fieldDropdownActions'
    | 'executeFieldDropdownAction'
    | 'executeItemFormDropdownAction'
    | 'customBlockStylesForStructuredTextField'
    | 'customMarksForStructuredTextField'
    ? FieldMethods & ItemFormMethods
    : Record<string, never>) &
  AdditionalMethods &
  AdditionalProperties;

// Base properties included in all contexts
type BaseProperties = {
  plugin: Plugin;
  currentUserAccessToken: string | null;
  environment: string;
  site: Site;
  currentUser: User | SsoUser | null;
  currentRole: Role;
  account: Account;
  mainLocale: string;
  theme: 'dark' | 'light';
  currentUserMenuLocale: string;
  owner: Account | Organization;
  itemTypes: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
  users: User[];
  ssoUsers: SsoUser[];
};

// Base methods included in all contexts
type BaseMethods = {
  createClient(options?: {
    apiToken?: string;
    environment?: string;
  }): CmaClient;
  loadItemTypeFields(itemTypeId: string): Promise<Field[]>;
  loadItemTypeFieldsets(itemTypeId: string): Promise<Fieldset[]>;
  loadFieldsUsingPlugin(): Promise<Field[]>;
  loadUsers(): Promise<User[]>;
  loadSsoUsers(): Promise<SsoUser[]>;
  navigateTo(path: string): void;
  scrollToField(path: string): void;
  openUrl(url: string): void;
  copyToClipboard(text: string): void;
  getFieldValue(path: string): unknown;
  toggleField(path: string, show: boolean): void;
  disableField(path: string, disable: boolean): void;
  setFieldValue(path: string, value: unknown): void;
  startAutoResizer(): void;
  stopAutoResizer(): void;
  updateHeight(height: number): void;
  notice(message: string): void;
  alert(message: string): void;
  customToast(options: {
    message: string;
    type?: 'notice' | 'alert' | 'warning';
    dismissAfterTimeout?: number | false;
    cta?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  openConfirm(options: {
    title: string;
    content?: ReactNode;
    choices?: Array<{
      label: string;
      value: unknown;
      intent?: 'positive' | 'negative';
    }>;
    cancel?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  updatePluginParameters(params: Record<string, unknown>): Promise<void>;
  selectItem(itemTypeId: string, options?: {
    multiple?: boolean;
    locale?: string;
    filters?: ItemListFilter;
  }): Promise<Item | Item[] | null>;
  editItem(itemId: string, options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
  }): Promise<Item | null>;
  createNewItem(itemTypeId: string, options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
  }): Promise<Item | null>;
  openModal(modalId: string, options?: {
    title?: string;
    width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth';
    parameters?: Record<string, unknown>;
  }): Promise<unknown>;
  renderModal(modalId: string): void;
  refreshItems(): void;
};

// Item type definition
type Item = {
  id: string;
  type: string;
  attributes: Record<string, any>;
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
    [key: string]: any;
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at?: string;
    first_published_at?: string;
    publication_scheduled_at?: string;
    unpublishing_scheduled_at?: string;
    stage?: 'draft' | 'updated' | 'published';
    current_version: string;
    status: 'draft' | 'updated' | 'published';
    is_valid: boolean;
    is_current_version_valid: boolean;
    created_by?: string;
    updated_by?: string;
  };
};

// Return value types
export type DropdownAction = {
  id: string;
  label: string;
  icon?: string | CustomIcon;
  group?: string;
  rank?: number;
  disabled?: boolean;
  tooltip?: string;
  alert?: boolean;
  active?: boolean;
  closeMenuOnClick?: boolean;
  parameters?: Record<string, unknown>;
};

export type DropdownActionGroup = {
  label: string;
  icon?: string | CustomIcon;
  actions: DropdownAction[];
  rank?: number;
};

export type CustomIcon = {
  type: 'svg';
  viewBox: string;
  content: string;
};

// Supporting types
type Plugin = {
  id: string;
  type: string;
  attributes: {
    name: string;
    description?: string;
    homepage_url?: string;
    package_name?: string;
    package_version?: string;
    parameters: Record<string, any>;
  };
  relationships?: {
    installed_by?: {
      data: {
        id: string;
        type: string;
      };
    };
  };
};

type Site = {
  id: string;
  type: string;
  attributes: {
    name: string;
    internal_subdomain: string;
    production_frontend_url?: string;
    primary_locale: string;
    locales: string[];
    timezone: string;
    global_seo?: Record<string, any>;
    favicon?: any;
    no_index: boolean;
  };
};

type User = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
    is_admin: boolean;
  };
};

type SsoUser = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    is_admin: boolean;
  };
};

type Role = {
  id: string;
  type: string;
  attributes: {
    name: string;
    can_edit_content: boolean;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_duplicate_records: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_edit_environment: boolean;
    can_access_build_triggers: boolean;
    environments_access: 'all' | 'primary_only' | string[];
    positive_item_type_permissions: ItemTypePermission[];
    negative_item_type_permissions: ItemTypePermission[];
  };
};

type ItemTypePermission = {
  action: 'create' | 'update' | 'delete' | 'take_over' | 'edit_metadata' | 'edit_schema' | 'publish';
  item_type?: string;
  on_environment?: string;
  locales?: string[];
  item_id?: string;
};

type Account = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    company?: string;
  };
};

type Organization = {
  id: string;
  type: string;
  attributes: {
    name: string;
  };
};

type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    tree: boolean;
    modular_block: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'tabular';
    ordering_direction?: 'asc' | 'desc';
    has_singleton_item: boolean;
  };
  relationships: {
    fields: {
      data: Array<{
        id: string;
        type: string;
      }>;
    };
    fieldsets: {
      data: Array<{
        id: string;
        type: string;
      }>;
    };
    ordering_field?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
    title_field?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
    singleton_item?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
  };
};

type Field = {
  id: string;
  type: string;
  attributes: {
    label: string;
    api_key: string;
    field_type: string;
    hint?: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
      field_size: 12;
      groups?: Array<{
        id: string;
        title: string;
        description?: string;
      }>;
    };
    default_value?: any;
    validators: Record<string, any>;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
    fieldset?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
  };
};

type Fieldset = {
  id: string;
  type: string;
  attributes: {
    title: string;
    hint?: string;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
  };
};

type CmaClient = {
  items: {
    create(params: any): Promise<Item>;
    update(itemId: string, params: any): Promise<Item>;
    destroy(itemId: string): Promise<void>;
    publish(itemId: string): Promise<Item>;
    unpublish(itemId: string): Promise<Item>;
    list(params?: any): AsyncIterator<Item>;
  };
  uploads: {
    list(params?: any): AsyncIterator<any>;
  };
  // ... other resources
};

type ItemListFilter = {
  type?: string | string[];
  fields?: Record<string, {
    eq?: any;
    neq?: any;
    lt?: any;
    lte?: any;
    gt?: any;
    gte?: any;
    exists?: boolean;
    in?: any[];
    notIn?: any[];
    anyIn?: any[];
    allIn?: any[];
    matches?: {
      pattern: string;
      caseSensitive?: boolean;
    };
  }>;
  query?: string;
  ids?: string[];
};
```

## Context Object (ItemsDropdownActionsCtx)

The context object provides comprehensive information about the selected items and DatoCMS environment:

```typescript
interface ItemsDropdownActionsCtx {
  // Selection Information
  locale: string;                    // Current content locale (e.g., 'en', 'it')
  selectedItemIds: string[];         // Array of selected item IDs
  selectedItems: Item[];             // Full item objects with all data
  
  // Base Properties
  plugin: Plugin;                    // Current plugin configuration
  currentUserAccessToken: string | null;  // API token for requests
  environment: string;               // Current environment ID
  
  // User & Permissions
  currentUser: User | SsoUser | null;  // Current user info
  currentRole: Role;                   // User's role and permissions
  account: Account;                    // Account info
  
  // Project Information
  site: Site;                        // Current project settings
  mainLocale: string;                // Primary locale
  owner: Account | Organization;     // Project owner
  
  // UI State
  theme: 'dark' | 'light';          // Current UI theme
  currentUserMenuLocale: string;     // UI language preference
  
  // Entity Repositories (all available)
  itemTypes: ItemType[];            // All models in project
  fields: Field[];                  // All fields
  fieldsets: Fieldset[];           // All fieldsets
  users: User[];                   // All regular users
  ssoUsers: SsoUser[];             // All SSO users
  
  // Methods (see Methods section below)
}
```

## Available Methods

The context object provides numerous methods for interacting with DatoCMS:

### Data Loading Methods

```typescript
// Load fields for a specific model
async loadItemTypeFields(itemTypeId: string): Promise<Field[]>

// Load fieldsets for a specific model  
async loadItemTypeFieldsets(itemTypeId: string): Promise<Fieldset[]>

// Load fields using the plugin
async loadFieldsUsingPlugin(): Promise<Field[]>

// Load all users
async loadUsers(): Promise<User[]>
async loadSsoUsers(): Promise<SsoUser[]>
```

### UI Methods

```typescript
// Show notifications
notice(message: string): void              // Green success message
alert(message: string): void               // Red error message
async customToast(options: {              // Custom toast with action
  message: string;
  type?: 'notice' | 'alert' | 'warning';
  cta?: {
    label: string;
    value: unknown;
  };
}): Promise<unknown>

// Open dialogs
async openConfirm(options: {
  title: string;
  content?: string;
  choices?: Array<{
    label: string;
    value: unknown;
    intent?: 'positive' | 'negative';
  }>;
}): Promise<unknown>

async openModal(modalId: string, options?: {
  title?: string;
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth';
  parameters?: Record<string, unknown>;
}): Promise<unknown>

// Update UI
refreshItems(): void  // Refresh items list after changes
```

### Item Management

```typescript
// Select items
async selectItem(itemTypeId: string, options?: {
  multiple?: boolean;
  locale?: string;
  filters?: ItemListFilter;
}): Promise<Item | Item[] | null>

// Edit items
async editItem(itemId: string, options?: {
  locale?: string;
  initialValues?: Record<string, unknown>;
}): Promise<Item | null>

// Create items
async createNewItem(itemTypeId: string, options?: {
  locale?: string;
  initialValues?: Record<string, unknown>;
}): Promise<Item | null>
```

### API Access

```typescript
// Create API client for custom operations
createClient(options?: {
  apiToken?: string;
  environment?: string;
}): CmaClient
```

## Return Value Types

### DropdownAction

```typescript
interface DropdownAction {
  id: string;                        // Unique action identifier
  label: string;                     // Display text
  icon?: string | CustomIcon;        // FontAwesome icon or custom SVG
  group?: string;                    // Group name for organization
  rank?: number;                     // Sort order (lower = higher)
  disabled?: boolean;                // Whether action is disabled
  tooltip?: string;                  // Hover tooltip text
  alert?: boolean;                   // Style as destructive (red)
  active?: boolean;                  // Show as active/selected
  closeMenuOnClick?: boolean;        // Close menu on click (default: true)
  parameters?: Record<string, unknown>;  // Custom data for handler
}
```

### DropdownActionGroup

```typescript
interface DropdownActionGroup {
  label: string;                     // Group label
  icon?: string | CustomIcon;        // Group icon
  actions: DropdownAction[];         // Actions in this group
  rank?: number;                     // Group sort order
}
```

## Complete Examples

### 1. CSV Export with Field Selection

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { 
  ItemsDropdownActionsCtx, 
  DropdownAction,
  ExecuteItemsDropdownActionCtx,
  Item,
  Field 
} from 'datocms-plugin-sdk';

connect({
  itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] {
    // Only show when items are selected
    if (ctx.selectedItemIds.length === 0) {
      return [];
    }

    return [
      {
        id: 'export-csv',
        label: `Export ${ctx.selectedItemIds.length} items to CSV`,
        icon: 'FaFileExport',
        group: 'export',
        rank: 10,
        tooltip: 'Download selected items as CSV file'
      }
    ];
  },

  async executeItemsDropdownAction(
    actionId: string,
    items: Item[],
    ctx: ExecuteItemsDropdownActionCtx
  ): Promise<void> {
    if (actionId === 'export-csv') {
      // Get item type info
      const firstItem = items[0];
      const itemType = ctx.itemTypes.find(
        it => it.id === firstItem.relationships.item_type.data.id
      );
      
      if (!itemType) {
        ctx.alert('Could not determine item type');
        return;
      }

      // Load fields for this model
      const fields = await ctx.loadItemTypeFields(itemType.id);
      
      // Build CSV headers
      const headers = [
        'ID',
        'Status',
        'Created',
        'Updated',
        'Published',
        ...fields.map(f => f.attributes.label)
      ];
      
      // Build CSV rows
      const rows = items.map(item => {
        const baseData = [
          item.id,
          item.meta.status,
          new Date(item.meta.created_at).toLocaleString(),
          new Date(item.meta.updated_at).toLocaleString(),
          item.meta.published_at ? new Date(item.meta.published_at).toLocaleString() : ''
        ];
        
        const fieldData = fields.map(field => {
          const value = item.attributes[field.attributes.api_key];
          
          // Handle different field types
          if (value === null || value === undefined) {
            return '';
          }
          
          // Handle complex types
          if (typeof value === 'object') {
            // File fields
            if (value.upload_id) {
              const upload = ctx.uploads?.find(u => u.id === value.upload_id);
              return upload ? upload.attributes.url : value.upload_id;
            }
            // JSON fields
            return JSON.stringify(value);
          }
          
          // Simple values
          return String(value);
        });
        
        return [...baseData, ...fieldData];
      });
      
      // Convert to CSV format
      const csvContent = [
        headers.join(','),
        ...rows.map(row => 
          row.map(cell => {
            // Escape quotes and wrap in quotes if contains comma
            const escaped = String(cell).replace(/"/g, '""');
            return escaped.includes(',') ? `"${escaped}"` : escaped;
          }).join(',')
        )
      ].join('\n');
      
      // Create download
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${itemType.attributes.api_key}-export-${new Date().toISOString().split('T')[0]}.csv`;
      link.click();
      URL.revokeObjectURL(url);
      
      ctx.notice(`Exported ${items.length} items to CSV`);
    }
  }
});
```

### 2. Bulk Status Management with Progress

```typescript
import { connect } from 'datocms-plugin-sdk';
import { buildClient } from '@datocms/cma-client';
import type { 
  ItemsDropdownActionsCtx, 
  DropdownActionGroup,
  ExecuteItemsDropdownActionCtx,
  Item 
} from 'datocms-plugin-sdk';

connect({
  itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownActionGroup[] {
    if (ctx.selectedItemIds.length === 0) {
      return [];
    }

    // Analyze selected items
    const draftCount = ctx.selectedItems.filter(
      item => item.meta.status === 'draft'
    ).length;
    const publishedCount = ctx.selectedItems.filter(
      item => item.meta.status === 'published'
    ).length;

    return [
      {
        label: 'Status Actions',
        icon: 'FaLayerGroup',
        rank: 1,
        actions: [
          {
            id: 'bulk-publish',
            label: `Publish ${draftCount > 0 ? `${draftCount} draft` : 'all'} items`,
            icon: 'FaCheck',
            disabled: draftCount === 0 && publishedCount === ctx.selectedItems.length,
            tooltip: 'Publish all selected draft items'
          },
          {
            id: 'bulk-unpublish',
            label: `Unpublish ${publishedCount > 0 ? `${publishedCount}` : 'all'} items`,
            icon: 'FaTimes',
            disabled: publishedCount === 0,
            alert: true,
            tooltip: 'Unpublish all selected published items'
          },
          {
            id: 'schedule-publishing',
            label: 'Schedule Publishing',
            icon: 'FaClock',
            tooltip: 'Schedule items to be published at a specific time'
          }
        ]
      }
    ];
  },

  async executeItemsDropdownAction(
    actionId: string,
    items: Item[],
    ctx: ExecuteItemsDropdownActionCtx
  ): Promise<void> {
    const client = buildClient({
      apiToken: ctx.currentUserAccessToken || '',
      environment: ctx.environment
    });

    switch (actionId) {
      case 'bulk-publish': {
        const confirmed = await ctx.openConfirm({
          title: 'Publish Items',
          content: `Are you sure you want to publish ${items.length} items?`,
          choices: [
            { 
              label: 'Publish All', 
              value: true, 
              intent: 'positive',
              icon: 'FaCheck'
            }
          ],
          cancel: { label: 'Cancel', value: false }
        });
        
        if (!confirmed) return;
        
        // Show progress toast
        const toastResult = await ctx.customToast({
          message: `Publishing ${items.length} items...`,
          type: 'notice',
          dismissAfterTimeout: false
        });
        
        let successCount = 0;
        let errorCount = 0;
        
        // Process in batches of 10
        const batchSize = 10;
        for (let i = 0; i < items.length; i += batchSize) {
          const batch = items.slice(i, i + batchSize);
          
          const results = await Promise.allSettled(
            batch.map(item => client.items.publish(item.id))
          );
          
          successCount += results.filter(r => r.status === 'fulfilled').length;
          errorCount += results.filter(r => r.status === 'rejected').length;
          
          // Update progress
          const progress = Math.round(((i + batch.length) / items.length) * 100);
          ctx.notice(`Progress: ${progress}%`);
        }
        
        // Show final result
        if (errorCount > 0) {
          ctx.alert(`Published ${successCount} items, ${errorCount} failed`);
        } else {
          ctx.notice(`Successfully published ${successCount} items`);
        }
        
        ctx.refreshItems();
        break;
      }

      case 'bulk-unpublish': {
        const confirmed = await ctx.openConfirm({
          title: 'Unpublish Items',
          content: `This will unpublish ${items.length} items. Continue?`,
          choices: [
            { 
              label: 'Unpublish All', 
              value: true, 
              intent: 'negative',
              icon: 'FaTimes'
            }
          ],
          cancel: { label: 'Cancel', value: false }
        });
        
        if (!confirmed) return;
        
        ctx.notice('Unpublishing items...');
        
        const results = await Promise.allSettled(
          items.map(item => client.items.unpublish(item.id))
        );
        
        const succeeded = results.filter(r => r.status === 'fulfilled').length;
        const failed = results.filter(r => r.status === 'rejected').length;
        
        if (failed > 0) {
          ctx.alert(`Unpublished ${succeeded} items, ${failed} failed`);
        } else {
          ctx.notice(`Successfully unpublished ${succeeded} items`);
        }
        
        ctx.refreshItems();
        break;
      }

      case 'schedule-publishing': {
        // Open modal to get scheduling details
        const schedule = await ctx.openModal('schedulePublishing', {
          title: 'Schedule Publishing',
          width: 'm',
          parameters: {
            itemCount: items.length,
            itemIds: items.map(i => i.id)
          }
        }) as { date: string; time: string } | null;
        
        if (!schedule) return;
        
        const scheduledAt = new Date(`${schedule.date}T${schedule.time}`).toISOString();
        
        ctx.notice('Scheduling items...');
        
        try {
          const results = await Promise.allSettled(
            items.map(item => 
              client.items.update(item.id, {
                ...item.attributes,
                meta: {
                  ...item.meta,
                  publication_scheduled_at: scheduledAt
                }
              })
            )
          );
          
          const succeeded = results.filter(r => r.status === 'fulfilled').length;
          ctx.notice(`Scheduled ${succeeded} items for publishing`);
          ctx.refreshItems();
        } catch (error: any) {
          ctx.alert(`Failed to schedule: ${error.message}`);
        }
        break;
      }
    }
  }
});
```

### 3. Smart Field Updates with Validation

```typescript
import { connect } from 'datocms-plugin-sdk';
import { buildClient } from '@datocms/cma-client';
import type { 
  ItemsDropdownActionsCtx, 
  DropdownAction,
  ExecuteItemsDropdownActionCtx,
  Item,
  Field,
  ItemType
} from 'datocms-plugin-sdk';

connect({
  itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] {
    if (ctx.selectedItemIds.length === 0) {
      return [];
    }

    // Check if all selected items are same type
    const itemTypeIds = new Set(
      ctx.selectedItems.map(item => item.relationships.item_type.data.id)
    );
    
    const sameType = itemTypeIds.size === 1;

    return [
      {
        id: 'smart-field-update',
        label: 'Update Field Values',
        icon: 'FaEdit',
        group: 'edit',
        rank: 20,
        disabled: !sameType,
        tooltip: sameType 
          ? 'Update field values across selected items' 
          : 'Select items of the same type'
      },
      {
        id: 'find-replace',
        label: 'Find & Replace',
        icon: 'FaSearchPlus',
        group: 'edit',
        rank: 21,
        disabled: !sameType,
        tooltip: 'Find and replace text in fields'
      }
    ];
  },

  async executeItemsDropdownAction(
    actionId: string,
    items: Item[],
    ctx: ExecuteItemsDropdownActionCtx
  ): Promise<void> {
    if (actionId === 'smart-field-update') {
      // Get item type and fields
      const itemTypeId = items[0].relationships.item_type.data.id;
      const itemType = ctx.itemTypes.find(it => it.id === itemTypeId);
      if (!itemType) return;

      const fields = await ctx.loadItemTypeFields(itemTypeId);
      
      // Filter to editable fields
      const editableFields = fields.filter(field => {
        const fieldType = field.attributes.field_type;
        // Skip readonly or complex fields
        return !['slug', 'created_at', 'updated_at'].includes(field.attributes.api_key) &&
               ['string', 'text', 'boolean', 'integer', 'float', 'date', 'datetime', 'color', 'json'].includes(fieldType);
      });

      // Open modal to select field and value
      const updateConfig = await ctx.openModal('fieldUpdateModal', {
        title: 'Update Field Values',
        width: 'l',
        parameters: {
          itemType,
          fields: editableFields,
          itemCount: items.length,
          sampleValues: items.slice(0, 3).map(item => ({
            id: item.id,
            values: editableFields.reduce((acc, field) => ({
              ...acc,
              [field.attributes.api_key]: item.attributes[field.attributes.api_key]
            }), {})
          }))
        }
      }) as {
        fieldApiKey: string;
        value: any;
        operation: 'replace' | 'append' | 'prepend';
      } | null;

      if (!updateConfig) return;

      const field = fields.find(f => f.attributes.api_key === updateConfig.fieldApiKey);
      if (!field) return;

      ctx.notice(`Updating ${field.attributes.label} field...`);

      const client = buildClient({
        apiToken: ctx.currentUserAccessToken || '',
        environment: ctx.environment
      });

      let successCount = 0;
      let validationErrors = 0;

      try {
        const results = await Promise.allSettled(
          items.map(async item => {
            let newValue = updateConfig.value;

            // Handle different operations
            if (updateConfig.operation === 'append' || updateConfig.operation === 'prepend') {
              const currentValue = item.attributes[updateConfig.fieldApiKey];
              if (typeof currentValue === 'string' && typeof newValue === 'string') {
                newValue = updateConfig.operation === 'append' 
                  ? currentValue + newValue 
                  : newValue + currentValue;
              }
            }

            // Validate against field validators
            const validators = field.attributes.validators || {};
            
            // Check required
            if (validators.required && !newValue) {
              throw new Error('Field is required');
            }

            // Check length
            if (validators.length) {
              const length = String(newValue).length;
              if (validators.length.min && length < validators.length.min) {
                throw new Error(`Minimum length is ${validators.length.min}`);
              }
              if (validators.length.max && length > validators.length.max) {
                throw new Error(`Maximum length is ${validators.length.max}`);
              }
            }

            // Check number range
            if (validators.number_range && typeof newValue === 'number') {
              if (validators.number_range.min && newValue < validators.number_range.min) {
                throw new Error(`Minimum value is ${validators.number_range.min}`);
              }
              if (validators.number_range.max && newValue > validators.number_range.max) {
                throw new Error(`Maximum value is ${validators.number_range.max}`);
              }
            }

            // Update item
            return client.items.update(item.id, {
              ...item.attributes,
              [updateConfig.fieldApiKey]: newValue
            });
          })
        );

        results.forEach(result => {
          if (result.status === 'fulfilled') {
            successCount++;
          } else {
            validationErrors++;
            console.error('Update failed:', result.reason);
          }
        });

        if (validationErrors > 0) {
          ctx.alert(`Updated ${successCount} items, ${validationErrors} failed validation`);
        } else {
          ctx.notice(`Successfully updated ${successCount} items`);
        }

        ctx.refreshItems();
      } catch (error: any) {
        ctx.alert(`Update failed: ${error.message}`);
      }
    }

    if (actionId === 'find-replace') {
      // Implementation for find & replace
      const config = await ctx.openModal('findReplaceModal', {
        title: 'Find & Replace',
        width: 'm',
        parameters: {
          itemCount: items.length
        }
      }) as {
        find: string;
        replace: string;
        fieldApiKeys: string[];
        caseSensitive: boolean;
      } | null;

      if (!config) return;

      ctx.notice('Processing find & replace...');

      const client = buildClient({
        apiToken: ctx.currentUserAccessToken || '',
        environment: ctx.environment
      });

      let totalReplacements = 0;
      let itemsModified = 0;

      try {
        for (const item of items) {
          let modified = false;
          const updates: Record<string, any> = {};

          for (const fieldApiKey of config.fieldApiKeys) {
            const value = item.attributes[fieldApiKey];
            if (typeof value === 'string') {
              const regex = new RegExp(
                config.find.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'), 
                config.caseSensitive ? 'g' : 'gi'
              );
              
              const newValue = value.replace(regex, config.replace);
              if (newValue !== value) {
                updates[fieldApiKey] = newValue;
                modified = true;
                totalReplacements += (value.match(regex) || []).length;
              }
            }
          }

          if (modified) {
            await client.items.update(item.id, {
              ...item.attributes,
              ...updates
            });
            itemsModified++;
          }
        }

        ctx.notice(
          `Replaced ${totalReplacements} occurrences in ${itemsModified} items`
        );
        ctx.refreshItems();
      } catch (error: any) {
        ctx.alert(`Find & replace failed: ${error.message}`);
      }
    }
  }
});
```

### 4. Advanced Reporting with Charts

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { 
  ItemsDropdownActionsCtx, 
  DropdownAction,
  ExecuteItemsDropdownActionCtx,
  Item 
} from 'datocms-plugin-sdk';

connect({
  itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] {
    if (ctx.selectedItemIds.length === 0) {
      return [];
    }

    return [
      {
        id: 'generate-report',
        label: 'Generate Analytics Report',
        icon: 'FaChartBar',
        group: 'analytics',
        rank: 30,
        tooltip: 'Generate detailed analytics for selected items'
      },
      {
        id: 'export-json',
        label: 'Export as JSON',
        icon: 'FaFileCode',
        group: 'export',
        rank: 31,
        tooltip: 'Export data in JSON format'
      }
    ];
  },

  async executeItemsDropdownAction(
    actionId: string,
    items: Item[],
    ctx: ExecuteItemsDropdownActionCtx
  ): Promise<void> {
    if (actionId === 'generate-report') {
      const itemType = ctx.itemTypes.find(
        it => it.id === items[0].relationships.item_type.data.id
      );
      
      if (!itemType) return;

      // Analyze data
      const analytics = {
        total: items.length,
        byStatus: {} as Record<string, number>,
        byMonth: {} as Record<string, number>,
        fieldStats: {} as Record<string, any>,
        performance: {
          avgUpdateFrequency: 0,
          mostActiveDay: '',
          leastActiveDay: ''
        }
      };

      // Analyze each item
      items.forEach(item => {
        // Status distribution
        analytics.byStatus[item.meta.status] = 
          (analytics.byStatus[item.meta.status] || 0) + 1;

        // Monthly distribution
        const month = new Date(item.meta.created_at).toISOString().slice(0, 7);
        analytics.byMonth[month] = (analytics.byMonth[month] || 0) + 1;

        // Analyze fields
        Object.entries(item.attributes).forEach(([key, value]) => {
          if (!analytics.fieldStats[key]) {
            analytics.fieldStats[key] = {
              filled: 0,
              empty: 0,
              avgLength: 0,
              uniqueValues: new Set()
            };
          }

          const stat = analytics.fieldStats[key];
          if (value !== null && value !== '') {
            stat.filled++;
            if (typeof value === 'string') {
              stat.avgLength += value.length;
            }
            stat.uniqueValues.add(JSON.stringify(value));
          } else {
            stat.empty++;
          }
        });
      });

      // Calculate averages
      Object.values(analytics.fieldStats).forEach(stat => {
        if (stat.filled > 0) {
          stat.avgLength = Math.round(stat.avgLength / stat.filled);
        }
        stat.uniqueCount = stat.uniqueValues.size;
        delete stat.uniqueValues; // Remove set from final output
      });

      // Generate HTML report with charts
      const reportHtml = `
<!DOCTYPE html>
<html>
<head>
  <title>${itemType.attributes.name} Analytics Report</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@3.9.1/dist/chart.min.js"></script>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      padding: 40px;
      background: #f5f5f5;
    }
    .container {
      max-width: 1200px;
      margin: 0 auto;
      background: white;
      padding: 30px;
      border-radius: 8px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    h1 {
      color: #333;
      margin-bottom: 10px;
    }
    .subtitle {
      color: #666;
      margin-bottom: 30px;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 30px;
      margin: 30px 0;
    }
    .chart-container {
      position: relative;
      height: 300px;
    }
    .stat-card {
      background: #f8f9fa;
      padding: 20px;
      border-radius: 8px;
      border: 1px solid #e9ecef;
    }
    .stat-value {
      font-size: 2em;
      font-weight: bold;
      color: #007bff;
    }
    .stat-label {
      color: #6c757d;
      margin-top: 5px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    th, td {
      padding: 12px;
      text-align: left;
      border-bottom: 1px solid #dee2e6;
    }
    th {
      background: #f8f9fa;
      font-weight: 600;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>${itemType.attributes.name} Analytics Report</h1>
    <p class="subtitle">Generated on ${new Date().toLocaleString()} • ${analytics.total} items analyzed</p>
    
    <div class="grid">
      <div class="stat-card">
        <div class="stat-value">${analytics.total}</div>
        <div class="stat-label">Total Items</div>
      </div>
      <div class="stat-card">
        <div class="stat-value">${analytics.byStatus.published || 0}</div>
        <div class="stat-label">Published</div>
      </div>
      <div class="stat-card">
        <div class="stat-value">${analytics.byStatus.draft || 0}</div>
        <div class="stat-label">Drafts</div>
      </div>
    </div>

    <div class="grid">
      <div class="chart-container">
        <canvas id="statusChart"></canvas>
      </div>
      <div class="chart-container">
        <canvas id="monthlyChart"></canvas>
      </div>
    </div>

    <h2>Field Analysis</h2>
    <table>
      <thead>
        <tr>
          <th>Field</th>
          <th>Filled</th>
          <th>Empty</th>
          <th>Fill Rate</th>
          <th>Unique Values</th>
          <th>Avg Length</th>
        </tr>
      </thead>
      <tbody>
        ${Object.entries(analytics.fieldStats).map(([field, stats]: [string, any]) => `
          <tr>
            <td>${field}</td>
            <td>${stats.filled}</td>
            <td>${stats.empty}</td>
            <td>${Math.round((stats.filled / (stats.filled + stats.empty)) * 100)}%</td>
            <td>${stats.uniqueCount}</td>
            <td>${stats.avgLength || '-'}</td>
          </tr>
        `).join('')}
      </tbody>
    </table>
  </div>

  <script>
    // Status Chart
    new Chart(document.getElementById('statusChart'), {
      type: 'doughnut',
      data: {
        labels: ${JSON.stringify(Object.keys(analytics.byStatus))},
        datasets: [{
          data: ${JSON.stringify(Object.values(analytics.byStatus))},
          backgroundColor: ['#28a745', '#ffc107', '#dc3545']
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          title: {
            display: true,
            text: 'Status Distribution'
          }
        }
      }
    });

    // Monthly Chart
    new Chart(document.getElementById('monthlyChart'), {
      type: 'line',
      data: {
        labels: ${JSON.stringify(Object.keys(analytics.byMonth))},
        datasets: [{
          label: 'Items Created',
          data: ${JSON.stringify(Object.values(analytics.byMonth))},
          borderColor: '#007bff',
          tension: 0.1
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          title: {
            display: true,
            text: 'Creation Timeline'
          }
        }
      }
    });
  </script>
</body>
</html>
      `;

      // Create and download report
      const blob = new Blob([reportHtml], { type: 'text/html' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${itemType.attributes.api_key}-analytics-${new Date().toISOString().split('T')[0]}.html`;
      link.click();
      URL.revokeObjectURL(url);
      
      ctx.notice('Analytics report generated successfully');
    }

    if (actionId === 'export-json') {
      // Export as structured JSON
      const itemType = ctx.itemTypes.find(
        it => it.id === items[0].relationships.item_type.data.id
      );

      const exportData = {
        export: {
          date: new Date().toISOString(),
          environment: ctx.environment,
          locale: ctx.locale,
          itemType: itemType?.attributes.api_key,
          count: items.length
        },
        items: items.map(item => ({
          id: item.id,
          ...item.attributes,
          _meta: item.meta
        }))
      };

      const blob = new Blob(
        [JSON.stringify(exportData, null, 2)], 
        { type: 'application/json' }
      );
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `${itemType?.attributes.api_key || 'export'}-${new Date().toISOString().split('T')[0]}.json`;
      link.click();
      URL.revokeObjectURL(url);
      
      ctx.notice(`Exported ${items.length} items as JSON`);
    }
  }
});
```

## Best Practices

### 1. Action Design
- Use clear, action-oriented labels
- Group related actions together
- Provide helpful tooltips
- Use appropriate icons from FontAwesome

### 2. Performance
- Handle large datasets efficiently
- Use batch processing for bulk operations
- Show progress for long-running tasks
- Implement proper error handling

### 3. User Experience
- Always confirm destructive actions
- Provide clear feedback on success/failure
- Refresh the UI after modifications
- Disable actions when not applicable

### 4. Error Handling
```typescript
try {
  // Perform operation
  const results = await Promise.allSettled(
    items.map(item => performOperation(item))
  );
  
  const succeeded = results.filter(r => r.status === 'fulfilled').length;
  const failed = results.filter(r => r.status === 'rejected').length;
  
  if (failed > 0) {
    ctx.alert(`Operation completed: ${succeeded} succeeded, ${failed} failed`);
  } else {
    ctx.notice(`Successfully processed ${succeeded} items`);
  }
} catch (error: any) {
  ctx.alert(`Operation failed: ${error.message}`);
}
```

### 5. Security Considerations
- Validate user permissions before operations
- Use the user's access token for API calls
- Sanitize data before external requests
- Handle sensitive data appropriately

## Common Patterns

### Permission-Based Actions
```typescript
itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] {
  const actions: DropdownAction[] = [];
  
  // Only show publish action if user can publish
  if (ctx.currentRole.attributes.can_publish_to_production) {
    actions.push({
      id: 'bulk-publish',
      label: 'Publish Items',
      icon: 'FaCheck'
    });
  }
  
  return actions;
}
```

### Dynamic Action Labels
```typescript
itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] {
  const count = ctx.selectedItemIds.length;
  
  return [{
    id: 'export',
    label: count === 1 
      ? 'Export item' 
      : `Export ${count} items`,
    icon: 'FaDownload'
  }];
}
```

### Conditional Actions
```typescript
itemsDropdownActions(ctx: ItemsDropdownActionsCtx): DropdownAction[] {
  // Check if all items are same type
  const types = new Set(
    ctx.selectedItems.map(i => i.relationships.item_type.data.id)
  );
  
  if (types.size > 1) {
    return [{
      id: 'mixed-warning',
      label: 'Select items of same type',
      disabled: true,
      icon: 'FaExclamationTriangle'
    }];
  }
  
  // Return type-specific actions
  return getActionsForType(Array.from(types)[0]);
}
```

## Related Hooks

- [`executeItemsDropdownAction`](./executeItemsDropdownAction.md) - Handles execution of these actions
- [`itemFormDropdownActions`](./itemFormDropdownActions.md) - Actions for single items
- [`fieldDropdownActions`](./fieldDropdownActions.md) - Actions for individual fields
- [`onBeforeItemsPublish`](./onBeforeItemsPublish.md) - Hook into bulk publishing
- [`onBeforeItemsDestroy`](./onBeforeItemsDestroy.md) - Hook into bulk deletion

## Troubleshooting

### Actions Not Showing
1. Ensure items are selected
2. Check return value is not empty array
3. Verify hook is properly registered
4. Check for JavaScript errors

### Performance Issues
1. Limit data processing in the hook itself
2. Use pagination for large datasets
3. Implement progress indicators
4. Consider background processing

### API Errors
1. Verify access token is valid
2. Check API rate limits
3. Validate data before sending
4. Handle network failures gracefully