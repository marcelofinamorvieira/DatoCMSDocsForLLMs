# contentAreaSidebarItems Hook

## Purpose

The `contentAreaSidebarItems` hook allows plugins to add custom navigation items to the content area sidebar in DatoCMS. This enables plugins to provide quick access to custom pages, tools, or views that content editors need while working with records.

## Signature

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[]
```

## Context Properties

The hook receives the standard context object with base properties:

- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `environment`: string - The ID of the current environment
- `itemTypes`: Partial<Record<string, ItemType>> - Models indexed by ID
- `fields`: Partial<Record<string, Field>> - Fields indexed by ID

## Type Definitions

```typescript
// Hook signature
export type ContentAreaSidebarItemsHook = {
  /**
   * Use this function to add custom sidebar item groups in the content area
   *
   * @tag contentAreaSidebarItems
   */
  contentAreaSidebarItems: (ctx: ContentAreaSidebarItemsCtx) => ContentAreaSidebarItem[];
};

// Context type
export type ContentAreaSidebarItemsCtx = Ctx;

// Return type
export type ContentAreaSidebarItem = {
  /** Unique label shown in the sidebar */
  label: string;
  /** FontAwesome icon name or custom SVG */
  icon: Icon;
  /** Link destination */
  pointsTo: {
    /** ID of the custom page to navigate to */
    pageId: string;
  };
  /** Control where the item appears in the sidebar */
  placement?: ['before' | 'after', 'menuItems' | 'seoPreferences'];
  /** Priority when multiple items have same placement */
  rank?: number;
};

// Icon type
export type Icon = AwesomeFontIconIdentifier | SvgIcon;

export type AwesomeFontIconIdentifier = string;

export type SvgIcon = {
  type: 'svg';
  viewBox: string;
  content: string;
};

// Base context type (Ctx) - includes all properties and methods
export type Ctx = BaseCtx & BaseProperties & BaseMethods;

// Base properties included in all contexts
type BaseProperties = {
  // Plugin information
  plugin: Plugin;
  currentUserAccessToken: string | null;
  
  // Authentication & Permissions  
  currentUser: User | SsoUser | null;
  currentRole: Role;
  account: Account;
  owner: Account | Organization;
  
  // Project Information
  site: Site;
  environment: string;
  environmentId: string;
  isEnvironmentPrimary: boolean;
  mainLocale: string;
  currentUserMenuLocale: string;
  theme: 'dark' | 'light';
  ui: { locale: string };
  
  // Entity Repositories
  itemTypes: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
  users: User[];
  ssoUsers: SsoUser[];
};

// Base methods included in all contexts
type BaseMethods = {
  // Data Loading
  loadItemTypes: (ids: string[]) => Promise<ItemType[]>;
  loadFields: (ids: string[]) => Promise<Field[]>;
  loadFieldsets: (ids: string[]) => Promise<Fieldset[]>;
  loadUsers: (ids: string[]) => Promise<User[]>;
  loadSsoUsers: (ids: string[]) => Promise<SsoUser[]>;
  loadItemTypeFields: (itemTypeId: string) => Promise<Field[]>;
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<Fieldset[]>;
  loadFieldsUsingPlugin: () => Promise<Field[]>;
  
  // Plugin Configuration
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  updateFieldAppearance: (
    fieldId: string,
    changes: FieldAppearanceChange[]
  ) => Promise<void>;
  
  // Toast Notifications
  notice: (message: string) => void;
  alert: (message: string) => void;
  customToast: (options: {
    message: string;
    type?: 'notice' | 'alert' | 'warning';
    cta?: { label: string; value: unknown; };
    dismissAfterTimeout?: number;
  }) => Promise<unknown>;
  
  // Item (Record) Dialogs
  selectItem: (
    itemTypeId: string,
    options?: {
      multiple?: boolean;
      locale?: string;
      filters?: ItemListFilter;
      initialLocationQuery?: LocationQuery;
    }
  ) => Promise<Item | Item[] | null>;
  
  createNewItem: (
    itemTypeId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<Item | null>;
  
  editItem: (
    itemId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<Item | null>;
  
  // Upload (Asset) Dialogs
  selectUpload: (options?: {
    multiple?: boolean;
    locale?: string;
    filters?: UploadListFilter;
  }) => Promise<Upload | Upload[] | null>;
  
  editUpload: (
    uploadId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<Upload | null>;
  
  editUploads: (
    uploadIds: string[],
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<void>;
  
  // Custom Dialogs
  openModal: (
    modalId: string,
    options?: {
      title?: string;
      width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
      height?: 's' | 'm' | 'l' | number;
      initialHeight?: number;
      closeDisabled?: boolean;
      parameters?: Record<string, unknown>;
    }
  ) => Promise<unknown>;
  
  openConfirm: (options: {
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
  }) => Promise<unknown>;
  
  // Navigation
  navigateTo: (
    location: 
      | string // URL path
      | { 
          pageId: string;
          params?: Record<string, string>;
          query?: Record<string, string>;
          locale?: string;
        }
  ) => Promise<void>;
  
  // Utility Methods
  getFieldIntlTitle: (
    field: Field,
    locale?: string,
    returnUntranslatedTitle?: boolean
  ) => string | null;
  
  createClient: (options?: { 
    apiToken?: string;
    environment?: string;
  }) => SiteApiClient;
};

// Supporting types

// Plugin type
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description: string;
    parameters: Record<string, unknown>;
    // ... other attributes
  };
};

// User types
type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    // ... other attributes
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    // ... other attributes
  };
};

// Organization type
type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
    // ... other attributes
  };
};

// Role type
type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_content: boolean;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_environments: boolean;
    can_access_audit_log: boolean;
    can_publish_content: boolean;
    // ... other permissions
  };
};

// Account type
type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    name: string;
    // ... other attributes
  };
};

// Site type
type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    locales: string[];
    // ... other attributes
  };
};

// ItemType (Model) type
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    // ... other attributes
  };
};

// Field type
type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    api_key: string;
    field_type: FieldType;
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
    // ... other attributes
  };
};

// Fieldset type
type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    // ... other attributes
  };
};

// FieldAppearanceChange type
type FieldAppearanceChange = {
  path: string;
  value: any;
};

// Item type
type Item = {
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
    // ... other relationships
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at: string | null;
    first_published_at: string | null;
    publication_scheduled_at: string | null;
    unpublishing_scheduled_at: string | null;
    status: 'draft' | 'updated' | 'published';
    // ... other meta
  };
};

// Other referenced types
type ItemListFilter = Record<string, any>;
type UploadListFilter = Record<string, any>;
type LocationQuery = Record<string, string>;
type FinalizeCallback = (item: Item | Upload) => void;
type Upload = Record<string, any>;
type SiteApiClient = any; // DatoCMS client instance
type ReactElement = any; // React element type
type FieldType = 
  | 'boolean'
  | 'color' 
  | 'date'
  | 'date_time'
  | 'file'
  | 'gallery'
  | 'integer'
  | 'json'
  | 'lat_lon'
  | 'links'
  | 'link'
  | 'float'
  | 'rich_text'
  | 'seo'
  | 'slug'
  | 'string'
  | 'text'
  | 'video'
  | 'structured_text';
type BaseCtx = Record<string, any>;

// Type guard functions
export function isContentAreaSidebarItem(
  value: unknown,
): value is ContentAreaSidebarItem {
  return (
    isRecord(value) &&
    isString(value.label) &&
    isIcon(value.icon) &&
    isRecord(value.pointsTo) &&
    isString(value.pointsTo.pageId) &&
    (isNullish(value.placement) ||
      (isArray(value.placement) &&
        value.placement.length === 2 &&
        ['before', 'after'].includes(value.placement[0]) &&
        ['menuItems', 'seoPreferences'].includes(value.placement[1]))) &&
    (isNullish(value.rank) || isNumber(value.rank))
  );
}

export function isReturnTypeOfContentAreaSidebarItemsHook(
  value: unknown,
): value is ContentAreaSidebarItem[] {
  return isArray(value, isContentAreaSidebarItem);
}

// Helper functions  
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}

function isNullish(value: unknown): value is null | undefined {
  return value === null || value === undefined;
}

function isArray<T>(
  value: unknown,
  itemGuard?: (item: unknown) => item is T
): value is T[] {
  if (!Array.isArray(value)) return false;
  if (!itemGuard) return true;
  return value.every(itemGuard);
}

function isIcon(value: unknown): value is Icon {
  return (
    isString(value) ||
    (isRecord(value) &&
      value.type === 'svg' &&
      isString(value.viewBox) &&
      isString(value.content))
  );
}
```

## Return Value

The hook must return an array of `ContentAreaSidebarItem` objects:

```typescript
type ContentAreaSidebarItem = {
  label: string;                    // Unique label shown in the sidebar
  icon: Icon;                      // FontAwesome icon name or custom SVG
  pointsTo: {                      // Link destination
    pageId: string;                // ID of the custom page to navigate to
  };
  placement?: ['before' | 'after', 'menuItems' | 'seoPreferences'];
  rank?: number;                   // Priority when multiple items have same placement
}
```

## Placement Options

Control where your items appear in the sidebar:

- `['before', 'menuItems']` - Before the standard menu items
- `['after', 'menuItems']` - After the standard menu items (default)
- `['before', 'seoPreferences']` - Before SEO preferences
- `['after', 'seoPreferences']` - After SEO preferences

## Use Cases

### 1. Content Analytics Dashboard

Add a link to view content performance metrics:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  return [
    {
      label: 'Content Analytics',
      icon: 'chart-line',
      pointsTo: {
        pageId: 'analytics-dashboard'
      },
      placement: ['after', 'menuItems'],
      rank: 10
    }
  ];
}
```

### 2. Translation Management

Provide quick access to translation tools:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  const items: ContentAreaSidebarItem[] = [];

  // Only show if site has multiple locales
  if (ctx.site.attributes.locales.length > 1) {
    items.push({
      label: 'Translation Center',
      icon: 'language',
      pointsTo: {
        pageId: 'translation-center'
      },
      placement: ['before', 'seoPreferences'],
      rank: 5
    });

    items.push({
      label: 'Missing Translations',
      icon: 'exclamation-triangle',
      pointsTo: {
        pageId: 'missing-translations'
      },
      placement: ['after', 'menuItems'],
      rank: 20
    });
  }

  return items;
}
```

### 3. Workflow Management

Add workflow-related navigation items:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  const items: ContentAreaSidebarItem[] = [];

  // Add workflow items based on user role
  if (ctx.currentRole.attributes.can_edit_content) {
    items.push({
      label: 'My Tasks',
      icon: 'tasks',
      pointsTo: {
        pageId: 'workflow-tasks'
      },
      placement: ['before', 'menuItems'],
      rank: 1
    });
  }

  if (ctx.currentRole.attributes.can_publish_content) {
    items.push({
      label: 'Approval Queue',
      icon: 'clipboard-check',
      pointsTo: {
        pageId: 'approval-queue'
      },
      placement: ['before', 'menuItems'],
      rank: 2
    });
  }

  // Add workflow overview for everyone
  items.push({
    label: 'Workflow Overview',
    icon: 'sitemap',
    pointsTo: {
      pageId: 'workflow-overview'
    },
    placement: ['after', 'menuItems'],
    rank: 15
  });

  return items;
}
```

### 4. Import/Export Tools

Provide data management utilities:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  const items: ContentAreaSidebarItem[] = [];

  // Only show to users with appropriate permissions
  if (ctx.currentRole.attributes.can_edit_schema) {
    items.push({
      label: 'Import Data',
      icon: 'file-import',
      pointsTo: {
        pageId: 'import-wizard'
      },
      placement: ['after', 'menuItems'],
      rank: 30
    });

    items.push({
      label: 'Export Center',
      icon: 'file-export',
      pointsTo: {
        pageId: 'export-center'
      },
      placement: ['after', 'menuItems'],
      rank: 31
    });
  }

  return items;
}
```

### 5. Custom Reports and Tools

Add various utility pages based on configuration:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  const items: ContentAreaSidebarItem[] = [];
  const enabledTools = ctx.plugin.attributes.parameters.enabledTools || {};

  if (enabledTools.seoChecker) {
    items.push({
      label: 'SEO Checker',
      icon: 'search',
      pointsTo: {
        pageId: 'seo-checker'
      },
      placement: ['after', 'seoPreferences'],
      rank: 1
    });
  }

  if (enabledTools.contentCalendar) {
    items.push({
      label: 'Content Calendar',
      icon: 'calendar-alt',
      pointsTo: {
        pageId: 'content-calendar'
      },
      placement: ['after', 'menuItems'],
      rank: 10
    });
  }

  if (enabledTools.brokenLinks) {
    items.push({
      label: 'Broken Links',
      icon: 'link-slash',
      pointsTo: {
        pageId: 'broken-links-scanner'
      },
      placement: ['after', 'menuItems'],
      rank: 25
    });
  }

  if (enabledTools.aiAssistant) {
    items.push({
      label: 'AI Writing Assistant',
      icon: 'robot',
      pointsTo: {
        pageId: 'ai-assistant'
      },
      placement: ['before', 'menuItems'],
      rank: 5
    });
  }

  return items;
}
```

### 6. Environment-Specific Tools

Show different tools based on the current environment:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  const items: ContentAreaSidebarItem[] = [];

  if (ctx.environment === 'development') {
    items.push({
      label: 'Dev Tools',
      icon: 'wrench',
      pointsTo: {
        pageId: 'development-tools'
      },
      placement: ['after', 'menuItems'],
      rank: 100
    });

    items.push({
      label: 'Test Data Generator',
      icon: 'dice',
      pointsTo: {
        pageId: 'test-data-generator'
      },
      placement: ['after', 'menuItems'],
      rank: 101
    });
  }

  if (ctx.environment === 'main') {
    items.push({
      label: 'Production Monitor',
      icon: 'heartbeat',
      pointsTo: {
        pageId: 'production-monitor'
      },
      placement: ['before', 'menuItems'],
      rank: 1
    });
  }

  // Available in all environments
  items.push({
    label: 'Environment Sync',
    icon: 'sync',
    pointsTo: {
      pageId: 'environment-sync'
    },
    placement: ['after', 'menuItems'],
    rank: 50
  });

  return items;
}
```

### 7. Dynamic Items Based on Models

Create navigation items based on available content models:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  const items: ContentAreaSidebarItem[] = [];

  // Check for specific models and add relevant tools
  const hasProducts = Object.values(ctx.itemTypes).some(
    model => model.attributes.api_key === 'product'
  );

  if (hasProducts) {
    items.push({
      label: 'Price Manager',
      icon: 'tag',
      pointsTo: {
        pageId: 'price-manager'
      },
      placement: ['after', 'menuItems'],
      rank: 20
    });

    items.push({
      label: 'Inventory',
      icon: 'boxes',
      pointsTo: {
        pageId: 'inventory-manager'
      },
      placement: ['after', 'menuItems'],
      rank: 21
    });
  }

  const hasBlog = Object.values(ctx.itemTypes).some(
    model => model.attributes.api_key === 'blog_post'
  );

  if (hasBlog) {
    items.push({
      label: 'Editorial Calendar',
      icon: 'newspaper',
      pointsTo: {
        pageId: 'editorial-calendar'
      },
      placement: ['after', 'menuItems'],
      rank: 15
    });
  }

  return items;
}
```

## Using Custom SVG Icons

While FontAwesome icons are recommended for consistency, you can use custom SVGs:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  return [
    {
      label: 'Custom Tool',
      icon: {
        type: 'svg',
        viewBox: '0 0 24 24',
        content: `
          <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
        `
      },
      pointsTo: {
        pageId: 'custom-tool'
      }
    }
  ];
}
```

## Configurable Sidebar Items

Make sidebar items configurable through plugin parameters:

```typescript
contentAreaSidebarItems(ctx: ContentAreaSidebarItemsCtx): ContentAreaSidebarItem[] {
  const customItems = ctx.plugin.attributes.parameters.sidebarItems || [];
  
  return customItems
    .filter(item => {
      // Validate item configuration
      return item.enabled && item.label && item.pageId;
    })
    .map(item => ({
      label: item.label,
      icon: item.icon || 'cube',
      pointsTo: {
        pageId: item.pageId
      },
      placement: item.placement || ['after', 'menuItems'],
      rank: item.rank || 10
    }));
}
```

## Best Practices

1. **Unique Labels**: Each item must have a unique label across all plugins
2. **Meaningful Icons**: Choose icons that clearly represent the functionality
3. **Configurable Rank**: Allow users to customize the rank to avoid conflicts
4. **Permission Checks**: Only show items relevant to the user's permissions
5. **Environment Awareness**: Consider showing different items per environment
6. **Consistent Placement**: Group related items together using similar placement

## Related Hooks

- **renderPage**: Implements the actual page content for the sidebar items
- **mainNavigationTabs**: Add items to the main navigation instead of sidebar
- **settingsAreaSidebarItemGroups**: Add items to the settings area sidebar

## Important Notes

- Labels must be unique across all plugins to avoid conflicts
- The `pageId` must correspond to a page registered via the `renderPage` hook
- Items are sorted by rank within their placement group
- The sidebar automatically handles overflow with scrolling
- Icons should be chosen to maintain visual consistency with DatoCMS
- Users can collapse/expand the sidebar, so items should have clear labels

