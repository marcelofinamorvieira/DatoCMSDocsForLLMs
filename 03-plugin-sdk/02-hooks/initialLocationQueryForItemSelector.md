# initialLocationQueryForItemSelector Hook

## Purpose

The `initialLocationQueryForItemSelector` hook allows plugins to customize the initial filters and query parameters when users open a record selector dialog from "Single link" or "Multiple links" fields. This enables contextual filtering based on the current field, model, or other criteria to help users find relevant records more quickly.

## Hook Signature

```typescript
initialLocationQueryForItemSelector(
  openerField: Field,
  itemType: ItemType,
  ctx: InitialLocationQueryForItemSelectorCtx
): Promise<LocationQueryResult | undefined> | LocationQueryResult | undefined
```

## Parameters

### openerField: Field
The link field that triggered the record selector

### itemType: ItemType
The model type of records being selected  

### ctx: InitialLocationQueryForItemSelectorCtx
The context object with plugin data, current user, site info, and available fields

## Type Definitions

```typescript
// Complete type definitions for initialLocationQueryForItemSelector hook
export type InitialLocationQueryForItemSelectorHook = {
  initialLocationQueryForItemSelector: (
    openerField: Field,
    itemType: ItemType,
    ctx: InitialLocationQueryForItemSelectorCtx
  ) => Promise<LocationQueryResult | undefined> | LocationQueryResult | undefined;
};

export type InitialLocationQueryForItemSelectorCtx = BasePluginFrameCtx<
  'initialLocationQueryForItemSelector',
  {
    formValues?: Record<string, unknown>;
    locale?: string;
    item?: Item;
  }
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
export type LocationQueryResult = {
  locationQuery: {
    filter?: {
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
    orderBy?: Array<{
      field: string;
      direction: 'asc' | 'desc';
    }>;
    page?: {
      limit?: number;
      offset?: number;
    };
  };
  rank?: number;
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
```

## Return Value

Returns an object with:
- **locationQuery**: Query parameters including filters, ordering, and pagination
- **rank** (optional): Priority when multiple plugins implement this hook (lower numbers = higher priority)

## Use Cases

### 1. Contextual Filtering by Category

Filter related records based on the current item's category:

```typescript
initialLocationQueryForItemSelector(
  openerField: {
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
      fieldset?: { data: { id: string; type: string } };
    };
  },
  itemType: {
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
      singleton_item?: { data: { id: string; type: string } | null };
    };
  },
  ctx: {
    plugin: { id: string; type: string; attributes: { name: string; package_name: string; parameters: Record<string, any>; url?: string } };
    currentUser: { id: string; type: 'user' | 'account' | 'sso_user'; attributes: { email: string; first_name?: string; last_name?: string; avatar_url?: string } };
    site: { id: string; type: string; attributes: { name: string; locale: string; locales: string[]; domain?: string; internal_domain: string } };
    environment: string;
    formValues?: Record<string, unknown>;
    [key: string]: any;
  }
): {
  locationQuery: {
    filter?: {
      fields?: Record<string, {
        eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
        exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
        matches?: { pattern: string; caseSensitive?: boolean };
      }>;
      query?: string;
      ids?: string[];
    };
    orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
    page?: { limit?: number; offset?: number };
  };
  rank?: number;
} | undefined {
  // Only apply to specific fields
  if (openerField.attributes.api_key !== 'related_articles') {
    return undefined;
  }

  // Get the current item being edited (if available in context)
  const currentCategoryId = ctx.formValues?.category;
  
  if (!currentCategoryId) {
    return undefined;
  }

  return {
    locationQuery: {
      filter: {
        fields: {
          category: {
            eq: currentCategoryId
          }
        }
      },
      orderBy: [
        { field: 'published_at', direction: 'desc' }
      ]
    },
    rank: 10
  };
}
```

### 2. Language-based Filtering

Filter records by the current editing locale:

```typescript
initialLocationQueryForItemSelector(
  openerField: {
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
      fieldset?: { data: { id: string; type: string } };
    };
  },
  itemType: {
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
      singleton_item?: { data: { id: string; type: string } | null };
    };
  },
  ctx: {
    plugin: { id: string; type: string; attributes: { name: string; package_name: string; parameters: Record<string, any>; url?: string } };
    currentUser: { id: string; type: 'user' | 'account' | 'sso_user'; attributes: { email: string; first_name?: string; last_name?: string; avatar_url?: string } };
    site: { id: string; type: string; attributes: { name: string; locale: string; locales: string[]; domain?: string; internal_domain: string } };
    environment: string;
    locale?: string;
    [key: string]: any;
  }
): {
  locationQuery: {
    filter?: {
      fields?: Record<string, {
        eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
        exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
        matches?: { pattern: string; caseSensitive?: boolean };
      }>;
      query?: string;
      ids?: string[];
    };
    orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
    page?: { limit?: number; offset?: number };
  };
  rank?: number;
} | undefined {
  // Apply to all link fields in localized models
  if (!itemType.attributes.all_locales_required) {
    return undefined;
  }

  const currentLocale = ctx.locale || ctx.site.attributes.locales[0];

  return {
    locationQuery: {
      filter: {
        fields: {
          _locales: {
            anyIn: [currentLocale]
          }
        }
      }
    },
    rank: 20
  };
}
```

### 3. Status-based Filtering

Show only published items when selecting related content:

```typescript
initialLocationQueryForItemSelector(
  openerField: {
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
      fieldset?: { data: { id: string; type: string } };
    };
  },
  itemType: {
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
      singleton_item?: { data: { id: string; type: string } | null };
    };
  },
  ctx: {
    plugin: { id: string; type: string; attributes: { name: string; package_name: string; parameters: Record<string, any>; url?: string } };
    currentUser: { id: string; type: 'user' | 'account' | 'sso_user'; attributes: { email: string; first_name?: string; last_name?: string; avatar_url?: string } };
    site: { id: string; type: string; attributes: { name: string; locale: string; locales: string[]; domain?: string; internal_domain: string } };
    environment: string;
    fields: Partial<Record<string, {
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
        fieldset?: { data: { id: string; type: string } };
      };
    }>>;
    [key: string]: any;
  }
): {
  locationQuery: {
    filter?: {
      fields?: Record<string, {
        eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
        exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
        matches?: { pattern: string; caseSensitive?: boolean };
      }>;
      query?: string;
      ids?: string[];
    };
    orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
    page?: { limit?: number; offset?: number };
  };
  rank?: number;
} | undefined {
  // Only for production environment
  if (ctx.environment !== 'main') {
    return undefined;
  }

  // Check if the target model has a status field
  const statusField = Object.values(ctx.fields).find(
    f => f.attributes.api_key === 'status' && 
        f.relationships.item_type.data.id === itemType.id
  );

  if (!statusField) {
    return undefined;
  }

  return {
    locationQuery: {
      filter: {
        fields: {
          status: {
            eq: 'published'
          }
        }
      },
      orderBy: [
        { field: 'title', direction: 'asc' }
      ]
    },
    rank: 15
  };
}
```

### 4. Date Range Filtering

Filter events or time-sensitive content:

```typescript
initialLocationQueryForItemSelector(
  openerField: {
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
      fieldset?: { data: { id: string; type: string } };
    };
  },
  itemType: {
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
      singleton_item?: { data: { id: string; type: string } | null };
    };
  },
  ctx: {
    plugin: { id: string; type: string; attributes: { name: string; package_name: string; parameters: Record<string, any>; url?: string } };
    currentUser: { id: string; type: 'user' | 'account' | 'sso_user'; attributes: { email: string; first_name?: string; last_name?: string; avatar_url?: string } };
    site: { id: string; type: string; attributes: { name: string; locale: string; locales: string[]; domain?: string; internal_domain: string } };
    environment: string;
    [key: string]: any;
  }
): {
  locationQuery: {
    filter?: {
      fields?: Record<string, {
        eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
        exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
        matches?: { pattern: string; caseSensitive?: boolean };
      }>;
      query?: string;
      ids?: string[];
    };
    orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
    page?: { limit?: number; offset?: number };
  };
  rank?: number;
} | undefined {
  // Only for event-related fields
  if (!['upcoming_events', 'related_events'].includes(openerField.attributes.api_key)) {
    return undefined;
  }

  const today = new Date().toISOString().split('T')[0];

  return {
    locationQuery: {
      filter: {
        fields: {
          event_date: {
            gte: today  // Only future events
          },
          status: {
            eq: 'confirmed'
          }
        }
      },
      orderBy: [
        { field: 'event_date', direction: 'asc' }
      ],
      page: {
        limit: 50  // Show more results for event selection
      }
    },
    rank: 5
  };
}
```

### 5. User-based Filtering

Filter content based on the current user:

```typescript
initialLocationQueryForItemSelector(
  openerField: {
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
      fieldset?: { data: { id: string; type: string } };
    };
  },
  itemType: {
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
      singleton_item?: { data: { id: string; type: string } | null };
    };
  },
  ctx: {
    plugin: { id: string; type: string; attributes: { name: string; package_name: string; parameters: Record<string, any>; url?: string } };
    currentUser: {
      id: string;
      type: 'user' | 'account' | 'sso_user';
      attributes: {
        email: string;
        first_name?: string;
        last_name?: string;
        avatar_url?: string;
      };
    };
    site: { id: string; type: string; attributes: { name: string; locale: string; locales: string[]; domain?: string; internal_domain: string } };
    environment: string;
    [key: string]: any;
  }
): {
  locationQuery: {
    filter?: {
      fields?: Record<string, {
        eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
        exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
        matches?: { pattern: string; caseSensitive?: boolean };
      }>;
      query?: string;
      ids?: string[];
    };
    orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
    page?: { limit?: number; offset?: number };
  };
  rank?: number;
} | undefined {
  // Only for author/assignee fields
  if (openerField.attributes.api_key !== 'assigned_articles') {
    return undefined;
  }

  // Filter by current user's ID or email
  const userEmail = ctx.currentUser.type === 'Account' 
    ? ctx.currentUser.attributes.email
    : ctx.currentUser.attributes.email;

  return {
    locationQuery: {
      filter: {
        fields: {
          author_email: {
            eq: userEmail
          },
          status: {
            in: ['draft', 'in_review']
          }
        }
      },
      orderBy: [
        { field: 'updated_at', direction: 'desc' }
      ]
    },
    rank: 10
  };
}
```

### 6. Hierarchical Filtering

Filter based on parent-child relationships:

```typescript
initialLocationQueryForItemSelector(
  openerField: {
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
      fieldset?: { data: { id: string; type: string } };
    };
  },
  itemType: {
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
      singleton_item?: { data: { id: string; type: string } | null };
    };
  },
  ctx: {
    plugin: { id: string; type: string; attributes: { name: string; package_name: string; parameters: Record<string, any>; url?: string } };
    currentUser: { id: string; type: 'user' | 'account' | 'sso_user'; attributes: { email: string; first_name?: string; last_name?: string; avatar_url?: string } };
    site: { id: string; type: string; attributes: { name: string; locale: string; locales: string[]; domain?: string; internal_domain: string } };
    environment: string;
    item?: {
      id: string;
      type: string;
      attributes: Record<string, any>;
      relationships?: Record<string, any>;
      meta: {
        created_at: string;
        updated_at: string;
        published_at?: string;
        first_published_at?: string;
        publication_scheduled_at?: string;
        unpublishing_scheduled_at?: string;
        stage: 'draft' | 'updated' | 'published';
        current_version: string;
        status: 'draft' | 'updated' | 'published';
      };
    };
    [key: string]: any;
  }
): {
  locationQuery: {
    filter?: {
      fields?: Record<string, {
        eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
        exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
        matches?: { pattern: string; caseSensitive?: boolean };
      }>;
      query?: string;
      ids?: string[];
    };
    orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
    page?: { limit?: number; offset?: number };
  };
  rank?: number;
} | undefined {
  // For selecting child categories
  if (openerField.attributes.api_key !== 'subcategories') {
    return undefined;
  }

  // Get current item ID (if editing existing record)
  const currentItemId = ctx.item?.id;
  
  if (!currentItemId) {
    return undefined;
  }

  return {
    locationQuery: {
      filter: {
        fields: {
          parent_category: {
            eq: currentItemId
          },
          level: {
            lte: 3  // Maximum nesting level
          }
        }
      },
      orderBy: [
        { field: 'position', direction: 'asc' },
        { field: 'name', direction: 'asc' }
      ]
    },
    rank: 5
  };
}
```

### 7. Full-text Search Default

Provide a default search query:

```typescript
initialLocationQueryForItemSelector(
  openerField: {
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
      fieldset?: { data: { id: string; type: string } };
    };
  },
  itemType: {
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
      singleton_item?: { data: { id: string; type: string } | null };
    };
  },
  ctx: {
    plugin: { id: string; type: string; attributes: { name: string; package_name: string; parameters: Record<string, any>; url?: string } };
    currentUser: { id: string; type: 'user' | 'account' | 'sso_user'; attributes: { email: string; first_name?: string; last_name?: string; avatar_url?: string } };
    site: { id: string; type: string; attributes: { name: string; locale: string; locales: string[]; domain?: string; internal_domain: string } };
    environment: string;
    formValues?: Record<string, unknown>;
    [key: string]: any;
  }
): {
  locationQuery: {
    filter?: {
      fields?: Record<string, {
        eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
        exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
        matches?: { pattern: string; caseSensitive?: boolean };
      }>;
      query?: string;
      ids?: string[];
    };
    orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
    page?: { limit?: number; offset?: number };
  };
  rank?: number;
} | undefined {
  // For product recommendation fields
  if (openerField.attributes.api_key !== 'recommended_products') {
    return undefined;
  }

  // Get current product name or category
  const currentProductName = ctx.formValues?.name;
  const currentCategory = ctx.formValues?.category;

  const searchQuery = currentProductName || currentCategory || '';

  return {
    locationQuery: {
      filter: {
        query: searchQuery,  // Pre-fill search with related terms
        fields: {
          in_stock: {
            eq: true
          }
        }
      }
    },
    rank: 10
  };
}
```

## Handling Multiple Plugins

When multiple plugins implement this hook, the one with the lowest `rank` value takes precedence:

```typescript
// Plugin A
return {
  locationQuery: { /* ... */ },
  rank: 10  // Higher priority
};

// Plugin B  
return {
  locationQuery: { /* ... */ },
  rank: 20  // Lower priority
};

// Plugin A's configuration will be used
```

## Best Practices

1. **Check Field Specificity**: Always verify you're operating on the intended field:
   ```typescript
   if (openerField.attributes.api_key !== 'target_field') {
     return undefined;
   }
   ```

2. **Handle Missing Data**: Gracefully handle cases where expected data isn't available:
   ```typescript
   const category = ctx.formValues?.category;
   if (!category) {
     return undefined;  // Don't apply filter if no category
   }
   ```

3. **Performance Considerations**: Avoid overly complex filters that might slow down the query:
   ```typescript
   // Good: Simple, indexed field filter
   filter: { fields: { status: { eq: 'published' } } }
   
   // Avoid: Multiple complex conditions without indexes
   ```

4. **User Experience**: Set reasonable defaults for pagination:
   ```typescript
   page: {
     limit: 20  // Reasonable default
   }
   ```

5. **Plugin Settings**: Make rank configurable through plugin parameters:
   ```typescript
   const rank = ctx.plugin.attributes.parameters.selectorFilterRank || 10;
   return {
     locationQuery: { /* ... */ },
     rank
   };
   ```

## Related Hooks

- **itemFormDropdownActions**: Add custom actions to the record form
- **itemFormOutlets**: Add custom UI sections to record forms
- **manualFieldExtensions**: Create custom field editors with their own selectors

## Important Notes

- This hook is called when the record selector dialog opens, not when the page loads
- The filter is applied as an initial state - users can still modify or clear filters
- Complex filters might impact performance on large datasets
- The hook affects all instances of the specified field across the project
- Return `undefined` to skip filtering and use default behavior
- Always test with various data scenarios to ensure filters work as expected

## Complete Implementation Example

```typescript
// Import the connect function
import { connect } from 'datocms-plugin-sdk';

// Define the plugin
connect({
  initialLocationQueryForItemSelector(
    openerField: {
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
        fieldset?: { data: { id: string; type: string } };
      };
    },
    itemType: {
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
        singleton_item?: { data: { id: string; type: string } | null };
      };
    },
    ctx: {
      plugin: {
        id: string;
        type: string;
        attributes: {
          name: string;
          package_name: string;
          parameters: Record<string, any>;
          url?: string;
        };
      };
      currentUser: {
        id: string;
        type: 'user' | 'account' | 'sso_user';
        attributes: {
          email: string;
          first_name?: string;
          last_name?: string;
          avatar_url?: string;
        };
      };
      site: {
        id: string;
        type: string;
        attributes: {
          name: string;
          locale: string;
          locales: string[];
          domain?: string;
          internal_domain: string;
        };
      };
      environment: string;
      formValues?: Record<string, unknown>;
      [key: string]: any;
    }
  ): {
    locationQuery: {
      filter?: {
        fields?: Record<string, {
          eq?: any; neq?: any; lt?: any; lte?: any; gt?: any; gte?: any;
          exists?: boolean; in?: any[]; notIn?: any[]; anyIn?: any[]; allIn?: any[];
          matches?: { pattern: string; caseSensitive?: boolean };
        }>;
        query?: string;
        ids?: string[];
      };
      orderBy?: Array<{ field: string; direction: 'asc' | 'desc' }>;
      page?: { limit?: number; offset?: number };
    };
    rank?: number;
  } | undefined {
    // Implementation logic here
    return {
      locationQuery: {
        filter: {
          fields: {
            status: { eq: 'published' }
          }
        },
        orderBy: [{ field: 'created_at', direction: 'desc' }]
      },
      rank: 10
    };
  }
});
```