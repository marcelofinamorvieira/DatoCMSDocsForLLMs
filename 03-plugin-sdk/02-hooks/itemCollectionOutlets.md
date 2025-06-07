# itemCollectionOutlets Hook

## Purpose

The `itemCollectionOutlets` hook allows plugins to declare custom UI sections (outlets) that appear at the top of record collection pages for specific models. This enables adding custom functionality, visualizations, or tools directly above the record list, enhancing the content management experience for specific content types.

## Hook Signature

```typescript
itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[]
```

## Parameters

### itemType: ItemType
The model/content type for which outlets should be displayed

### ctx: ItemCollectionOutletsCtx
Context object containing plugin configuration, current user, site data, and UI utilities

## Type Definitions

```typescript
// Complete type definitions for itemCollectionOutlets hook
export type ItemCollectionOutletsHook = {
  itemCollectionOutlets: (
    itemType: ItemType,
    ctx: ItemCollectionOutletsCtx,
  ) => ItemCollectionOutlet[];
};

export type ItemCollectionOutletsCtx = BasePluginFrameCtx<
  'itemCollectionOutlets',
  Record<string, never>
>;

// Base plugin frame context
type BasePluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BaseProperties &
  BaseMethods &
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
};

// Return value type
export type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// ItemType definition
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

// Field type definition
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

type ReactNode = any; // Simplified for this context
type Item = any; // Simplified for this context
```

**Returns:** Array of outlet configurations that define where custom UI sections should appear

## Use Cases

### 1. Analytics Dashboard

Display content performance metrics above the record list:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type Field = {
  id: string;
  type: string;
  attributes: {
    api_key: string;
    field_type: string;
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
    };
  };
  relationships: {
    item_type: { data: { id: string; type: string } };
    fieldset?: { data: { id: string; type: string } | null };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      locales: string[];
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // Add analytics for blog posts
  if (itemType.attributes.api_key === 'blog_post') {
    outlets.push({
      id: 'blog-analytics',
      rank: 10,
      initialHeight: 300
    });
  }

  // Add analytics for products
  if (itemType.attributes.api_key === 'product') {
    outlets.push({
      id: 'product-performance',
      rank: 10,
      initialHeight: 350
    });
  }

  // Add general content metrics for any model with views tracking
  const hasViewsField = Object.values(ctx.fields).some(
    field => field && 
             field.attributes.api_key === 'view_count' &&
             field.relationships.item_type.data.id === itemType.id
  );

  if (hasViewsField) {
    outlets.push({
      id: 'content-metrics',
      rank: 20,
      initialHeight: 250
    });
  }

  return outlets;
}
```

### 2. Bulk Operations Toolbar

Add bulk actions for specific content types:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
      can_access_build_triggers: boolean;
      can_access_audit_log: boolean;
      can_edit_favicon: boolean;
      can_edit_global_seo: boolean;
      can_edit_site: boolean;
      can_manage_access_tokens: boolean;
      can_manage_webhooks: boolean;
      can_manage_environment: boolean;
      can_manage_sso: boolean;
      can_manage_users: boolean;
      can_perform_site_search: boolean;
      can_publish_to_production: boolean;
      can_access_private_environments: boolean;
      positive_item_type_permissions: string[];
      negative_item_type_permissions: string[];
      item_type_permissions: Record<string, string[]>;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      internal_domain: string;
      deploy_adapter: string;
      frontend_url?: string;
      locales: string[];
      timezone: string;
      imgix_host?: string;
      global_seo: Record<string, any>;
      favicon?: any;
      theme: Record<string, any>;
      no_index: boolean;
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

type Field = {
  id: string;
  type: string;
  attributes: {
    api_key: string;
    field_type: string;
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
    };
  };
  relationships: {
    item_type: { data: { id: string; type: string } };
    fieldset?: { data: { id: string; type: string } | null };
  };
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // Bulk operations for e-commerce products
  if (itemType.attributes.api_key === 'product') {
    outlets.push({
      id: 'bulk-price-update',
      rank: 5,
      initialHeight: 120
    });
    
    outlets.push({
      id: 'bulk-inventory-management',
      rank: 6,
      initialHeight: 150
    });
  }

  // Bulk operations for media assets
  if (itemType.attributes.api_key === 'media_asset') {
    outlets.push({
      id: 'bulk-resize-images',
      rank: 10,
      initialHeight: 180
    });
  }

  // General bulk operations for all models
  if (ctx.currentRole.attributes.can_edit_schema) {
    outlets.push({
      id: 'advanced-bulk-operations',
      rank: 100,
      initialHeight: 200
    });
  }

  return outlets;
}
```

### 3. Import/Export Tools

Provide data import/export functionality:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type Field = {
  id: string;
  type: string;
  attributes: {
    api_key: string;
    field_type: 'boolean' | 'color' | 'date' | 'date_time' | 'file' | 'float' | 'gallery' | 'integer' | 'json' | 'lat_lon' | 'link' | 'links' | 'rich_text' | 'seo' | 'single_line_string' | 'slug' | 'text' | 'video';
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
    };
  };
  relationships: {
    item_type: { data: { id: string; type: string } };
    fieldset?: { data: { id: string; type: string } | null };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      locales: string[];
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // CSV import/export for structured data
  const itemTypeFields = Object.values(ctx.fields)
    .filter(f => f && f.relationships.item_type.data.id === itemType.id);
  
  const hasSimpleFields = itemTypeFields.length > 0 && itemTypeFields
    .every(f => ['single_line_string', 'text', 'integer', 'float', 'boolean', 'date']
      .includes(f.attributes.field_type));

  if (hasSimpleFields) {
    outlets.push({
      id: 'csv-import-export',
      rank: 30,
      initialHeight: 200
    });
  }

  // JSON import/export for complex models
  if (itemType.attributes.api_key === 'configuration') {
    outlets.push({
      id: 'json-import-export',
      rank: 25,
      initialHeight: 220
    });
  }

  // Translation export for localized content
  if (itemType.attributes.all_locales_required) {
    outlets.push({
      id: 'translation-export',
      rank: 35,
      initialHeight: 180
    });
  }

  return outlets;
}
```

### 4. Content Calendar

Show a calendar view for date-based content:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type Field = {
  id: string;
  type: string;
  attributes: {
    api_key: string;
    field_type: 'boolean' | 'color' | 'date' | 'date_time' | 'file' | 'float' | 'gallery' | 'integer' | 'json' | 'lat_lon' | 'link' | 'links' | 'rich_text' | 'seo' | 'single_line_string' | 'slug' | 'text' | 'video';
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
    };
  };
  relationships: {
    item_type: { data: { id: string; type: string } };
    fieldset?: { data: { id: string; type: string } | null };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      locales: string[];
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // Check if model has date fields
  const dateFields = Object.values(ctx.fields)
    .filter(f => f && f.relationships.item_type.data.id === itemType.id)
    .filter(f => ['date', 'date_time'].includes(f.attributes.field_type));

  if (dateFields.length > 0) {
    // Event calendar for event models
    if (itemType.attributes.api_key === 'event') {
      outlets.push({
        id: 'event-calendar',
        rank: 1,
        initialHeight: 500
      });
    }

    // Editorial calendar for content
    if (['blog_post', 'article', 'news'].includes(itemType.attributes.api_key)) {
      outlets.push({
        id: 'editorial-calendar',
        rank: 5,
        initialHeight: 450
      });
    }

    // Generic calendar for any date-based model
    outlets.push({
      id: 'content-calendar',
      rank: 50,
      initialHeight: 400
    });
  }

  return outlets;
}
```

### 5. AI Content Assistant

Provide AI-powered content suggestions:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type Field = {
  id: string;
  type: string;
  attributes: {
    api_key: string;
    field_type: string;
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
    };
  };
  relationships: {
    item_type: { data: { id: string; type: string } };
    fieldset?: { data: { id: string; type: string } | null };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      locales: string[];
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // Only show if AI features are enabled
  if (!ctx.plugin.attributes.parameters.aiEnabled) {
    return outlets;
  }

  // Content suggestions for creative content
  if (['blog_post', 'article', 'landing_page'].includes(itemType.attributes.api_key)) {
    outlets.push({
      id: 'ai-content-ideas',
      rank: 15,
      initialHeight: 300
    });
  }

  // SEO optimization for any model with SEO fields
  const hasSeoFields = Object.values(ctx.fields).some(
    f => f && 
         f.attributes.api_key.includes('seo') &&
         f.relationships.item_type.data.id === itemType.id
  );

  if (hasSeoFields) {
    outlets.push({
      id: 'ai-seo-analyzer',
      rank: 20,
      initialHeight: 250
    });
  }

  return outlets;
}
```

### 6. Workflow Status Overview

Display workflow statistics and pending tasks:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type Field = {
  id: string;
  type: string;
  attributes: {
    api_key: string;
    field_type: string;
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
    };
  };
  relationships: {
    item_type: { data: { id: string; type: string } };
    fieldset?: { data: { id: string; type: string } | null };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      locales: string[];
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // Check if model has workflow fields
  const hasWorkflowField = Object.values(ctx.fields).some(
    f => f && 
         f.attributes.api_key === 'workflow_stage' &&
         f.relationships.item_type.data.id === itemType.id
  );

  if (hasWorkflowField) {
    outlets.push({
      id: 'workflow-overview',
      rank: 2,
      initialHeight: 200
    });

    // Add approval queue for editors
    if (ctx.currentRole.attributes.can_edit_schema) {
      outlets.push({
        id: 'approval-queue',
        rank: 3,
        initialHeight: 250
      });
    }
  }

  return outlets;
}
```

### 7. Quick Actions Bar

Provide frequently used actions:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      locales: string[];
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // Model-specific quick actions
  const quickActions: Record<string, { id: string; height: number }> = {
    product: { id: 'product-quick-actions', height: 100 },
    blog_post: { id: 'blog-quick-actions', height: 120 },
    page: { id: 'page-quick-actions', height: 100 },
    faq: { id: 'faq-quick-actions', height: 80 }
  };

  const modelAction = quickActions[itemType.attributes.api_key];
  if (modelAction) {
    outlets.push({
      id: modelAction.id,
      rank: 1,
      initialHeight: modelAction.height
    });
  }

  // Universal quick actions based on plugin config
  if (ctx.plugin.attributes.parameters.enableQuickActions) {
    outlets.push({
      id: 'universal-quick-actions',
      rank: 90,
      initialHeight: 100
    });
  }

  return outlets;
}
```

## Conditional Display

Control outlet visibility based on various conditions:

```typescript
// Complete type definitions for this example
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: string }> };
    fieldsets: { data: Array<{ id: string; type: string }> };
  };
};

type ItemCollectionOutletsCtx = {
  plugin: {
    id: string;
    type: string;
    attributes: {
      name: string;
      package_name: string;
      url: string;
      options_schema: Record<string, any>;
      parameters: Record<string, any>;
      plugin_type: string;
    };
  };
  currentUser: {
    id: string;
    type: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      is_sso: boolean;
    };
  };
  currentRole: {
    id: string;
    type: string;
    attributes: {
      name: string;
      can_edit_schema: boolean;
      can_edit_environment: boolean;
      can_access_build_triggers: boolean;
      can_access_audit_log: boolean;
      can_edit_favicon: boolean;
      can_edit_global_seo: boolean;
      can_edit_site: boolean;
      can_manage_access_tokens: boolean;
      can_manage_webhooks: boolean;
      can_manage_environment: boolean;
      can_manage_sso: boolean;
      can_manage_users: boolean;
      can_perform_site_search: boolean;
      can_publish_to_production: boolean;
      can_access_private_environments: boolean;
      positive_item_type_permissions: string[];
      negative_item_type_permissions: string[];
      item_type_permissions: Record<string, string[]>;
    };
  };
  site: {
    id: string;
    type: string;
    attributes: {
      name: string;
      domain: string;
      internal_domain: string;
      deploy_adapter: string;
      frontend_url?: string;
      locales: string[];
      timezone: string;
      imgix_host?: string;
      global_seo: Record<string, any>;
      favicon?: any;
      theme: Record<string, any>;
      no_index: boolean;
    };
  };
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  siteId: string;
};

type Field = {
  id: string;
  type: string;
  attributes: {
    api_key: string;
    field_type: string;
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
    };
  };
  relationships: {
    item_type: { data: { id: string; type: string } };
    fieldset?: { data: { id: string; type: string } | null };
  };
};

type ItemCollectionOutlet = {
  id: string;              // Unique identifier for the outlet
  rank?: number;           // Display order (lower numbers appear first)
  initialHeight?: number;  // Initial iframe height in pixels
};

// Hook implementation
function itemCollectionOutlets(
  itemType: ItemType,
  ctx: ItemCollectionOutletsCtx
): ItemCollectionOutlet[] {
  const outlets: ItemCollectionOutlet[] = [];

  // Environment-specific outlets
  if (ctx.environment === 'main') {
    outlets.push({
      id: 'production-checks',
      rank: 5,
      initialHeight: 150
    });
  } else {
    outlets.push({
      id: 'development-tools',
      rank: 5,
      initialHeight: 200
    });
  }

  // Role-based outlets
  if (ctx.currentRole.attributes.can_edit_schema) {
    outlets.push({
      id: 'admin-tools',
      rank: 95,
      initialHeight: 180
    });
  }

  // Feature flag based outlets
  const features = ctx.plugin.attributes.parameters.features || {};
  
  if (features.betaAnalytics) {
    outlets.push({
      id: 'beta-analytics',
      rank: 50,
      initialHeight: 300
    });
  }

  return outlets;
}
```

## Best Practices

1. **Unique IDs**: Ensure each outlet has a unique ID:
   ```typescript
   {
     id: `${itemType.attributes.api_key}-analytics`,
     // Includes model key for uniqueness
   }
   ```

2. **Appropriate Heights**: Set realistic initial heights:
   ```typescript
   // Simple toolbar
   initialHeight: 100
   
   // Complex dashboard
   initialHeight: 500
   ```

3. **Logical Ranking**: Use consistent ranking patterns:
   ```typescript
   // 1-10: Critical/primary features
   // 11-50: Standard features  
   // 51-90: Optional features
   // 91-100: Admin/debug tools
   ```

4. **Performance**: Only add outlets when necessary:
   ```typescript
   // Check if feature is enabled
   if (!ctx.plugin.attributes.parameters.outletEnabled) {
     return [];
   }
   ```

5. **User Experience**: Consider the cumulative height:
   ```typescript
   // Limit total outlets to prevent excessive scrolling
   const maxOutlets = 3;
   return outlets.slice(0, maxOutlets);
   ```

## Related Hooks

- **renderItemCollectionOutlet**: Implements the UI for each outlet
- **itemFormOutlets**: Add outlets to individual record forms
- **itemsDropdownActions**: Add actions to the collection page dropdown

## Important Notes

- Outlets appear at the top of the collection page, above the record list
- Multiple outlets are displayed in order of their rank (ascending)
- The iframe will automatically resize based on content after initial load
- Outlets are visible to all users who can access the model
- The hook is called when the collection page loads
- Performance impact should be considered when adding multiple outlets

## Summary

The `itemCollectionOutlets` hook enables plugins to add custom UI sections at the top of model collection pages. Key features:

- **Flexible positioning**: Use `rank` to control display order
- **Context-aware**: Access to current user, role, and site configuration
- **Model-specific**: Show different outlets based on the content type
- **Conditional display**: Control visibility based on permissions, environment, or feature flags
- **Performance considerate**: Set appropriate `initialHeight` and limit total outlets

Each example above includes complete type definitions and demonstrates real-world use cases for enhancing the content management experience with custom tools and visualizations.