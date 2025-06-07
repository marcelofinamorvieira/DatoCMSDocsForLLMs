# settingsAreaSidebarItemGroups Hook

## Purpose

The `settingsAreaSidebarItemGroups` hook allows you to add custom item groups to the settings area sidebar in DatoCMS. This enables you to create organized sections of custom settings pages, integrations, or administrative tools within the project settings.

## Signature

```typescript
settingsAreaSidebarItemGroups(
  ctx: SettingsAreaSidebarItemGroupsCtx
): SettingsAreaSidebarItemGroup[]
```

## Context Properties

The `settingsAreaSidebarItemGroups` hook receives a context object with:

### Base Properties
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `environment`: string - The ID of the current environment
- `theme`: Theme - Theme colors for the current project

## Type Definitions

```typescript
// Hook signature
export type SettingsAreaSidebarItemGroupsHook = {
  /**
   * Use this function to declare new item groups to be displayed in the
   * Settings Area sidebar
   *
   * @tag settingsAreaSidebarItemGroups
   */
  settingsAreaSidebarItemGroups: (
    ctx: SettingsAreaSidebarItemGroupsCtx,
  ) => SettingsAreaSidebarItemGroup[];
};

// Context type
export type SettingsAreaSidebarItemGroupsCtx = Ctx;

// Return type
export type SettingsAreaSidebarItemGroup = {
  /** Label for the group */
  label: string;
  /** Items within the group */
  items: SettingsAreaSidebarItem[];
  /**
   * Expresses where you want the group to be placed inside the sidebar.
   * If not specified, the item will be placed after the standard groups
   * provided by DatoCMS itself.
   */
  placement?: [
    'before' | 'after',
    'itemTypes' | 'permissions' | 'languages' | 'webhooks' | 'deployment',
  ];
};

export type SettingsAreaSidebarItem = {
  /** Label for the item */
  label: string;
  /** Icon to display alongside the label */
  icon: ReactElement;
  /** The destination for the link */
  pointsTo: {
    /** The page ID to navigate to */
    pageId: string;
  };
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
type Icon = string | { type: 'svg'; viewBox: string; content: string };
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
export function isSettingsAreaSidebarItemGroup(
  value: unknown,
): value is SettingsAreaSidebarItemGroup {
  return (
    isRecord(value) &&
    isString(value.label) &&
    isArray(value.items, isSettingsAreaSidebarItem) &&
    (isNullish(value.placement) ||
      (isArray(value.placement) &&
        value.placement.length === 2 &&
        ['before', 'after'].includes(value.placement[0]) &&
        ['itemTypes', 'permissions', 'languages', 'webhooks', 'deployment'].includes(
          value.placement[1],
        )))
  );
}

export function isSettingsAreaSidebarItem(
  value: unknown,
): value is SettingsAreaSidebarItem {
  return (
    isRecord(value) &&
    isString(value.label) &&
    isReactElement(value.icon) &&
    isRecord(value.pointsTo) &&
    isString(value.pointsTo.pageId)
  );
}

export function isReturnTypeOfSettingsAreaSidebarItemGroupsHook(
  value: unknown,
): value is SettingsAreaSidebarItemGroup[] {
  return isArray(value, isSettingsAreaSidebarItemGroup);
}

// Helper functions  
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

function isString(value: unknown): value is string {
  return typeof value === 'string';
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

function isReactElement(value: unknown): value is ReactElement {
  return (
    isRecord(value) &&
    (isString(value.type) || typeof value.type === 'function') &&
    isRecord(value.props)
  );
}
```

## Return Value

Returns an array of `SettingsAreaSidebarItemGroup` objects:

```typescript
interface SettingsAreaSidebarItemGroup {
  label: string;
  items: Array<{
    label: string;
    icon: ReactElement;
    pointsTo: {
      pageId: string;
    };
  }>;
  placement?: ['before' | 'after', string];
}
```

## Use Cases

### 1. Plugin Configuration Group

Add a comprehensive configuration section for your plugin:

```typescript
import { FaCog, FaKey, FaWebhook, FaDatabase, FaChartBar } from 'react-icons/fa';

settingsAreaSidebarItemGroups(ctx: SettingsAreaSidebarItemGroupsCtx) {
  return [
    {
      label: 'My Plugin Settings',
      placement: ['after', 'permissions'],
      items: [
        {
          label: 'General Configuration',
          icon: <FaCog />,
          pointsTo: {
            pageId: 'pluginGeneralConfig',
          },
        },
        {
          label: 'API Keys & Authentication',
          icon: <FaKey />,
          pointsTo: {
            pageId: 'pluginApiKeys',
          },
        },
        {
          label: 'Webhooks',
          icon: <FaWebhook />,
          pointsTo: {
            pageId: 'pluginWebhooks',
          },
        },
        {
          label: 'Data Sync',
          icon: <FaDatabase />,
          pointsTo: {
            pageId: 'pluginDataSync',
          },
        },
        {
          label: 'Analytics & Reports',
          icon: <FaChartBar />,
          pointsTo: {
            pageId: 'pluginAnalytics',
          },
        },
      ],
    },
  ];
}
```

### 2. Integration Management Group

Create a section for managing external integrations:

```typescript
import { 
  FaShopify, 
  FaWordpress, 
  FaSlack, 
  FaMailchimp,
  FaGoogle,
  FaAws 
} from 'react-icons/fa';

settingsAreaSidebarItemGroups(ctx: SettingsAreaSidebarItemGroupsCtx) {
  const integrations = [
    {
      label: 'E-commerce Integrations',
      items: [
        {
          label: 'Shopify',
          icon: <FaShopify />,
          pointsTo: { pageId: 'shopifyIntegration' },
          enabled: ctx.plugin.attributes.parameters.shopifyEnabled,
        },
        {
          label: 'WooCommerce',
          icon: <FaWordpress />,
          pointsTo: { pageId: 'woocommerceIntegration' },
          enabled: ctx.plugin.attributes.parameters.woocommerceEnabled,
        },
      ],
    },
    {
      label: 'Marketing Tools',
      items: [
        {
          label: 'Mailchimp',
          icon: <FaMailchimp />,
          pointsTo: { pageId: 'mailchimpIntegration' },
          enabled: ctx.plugin.attributes.parameters.mailchimpEnabled,
        },
        {
          label: 'Google Analytics',
          icon: <FaGoogle />,
          pointsTo: { pageId: 'googleAnalyticsIntegration' },
          enabled: ctx.plugin.attributes.parameters.gaEnabled,
        },
      ],
    },
    {
      label: 'Communication',
      items: [
        {
          label: 'Slack Notifications',
          icon: <FaSlack />,
          pointsTo: { pageId: 'slackIntegration' },
          enabled: ctx.plugin.attributes.parameters.slackEnabled,
        },
      ],
    },
    {
      label: 'Infrastructure',
      items: [
        {
          label: 'AWS S3 Backup',
          icon: <FaAws />,
          pointsTo: { pageId: 'awsBackupIntegration' },
          enabled: ctx.plugin.attributes.parameters.awsEnabled,
        },
      ],
    },
  ];

  // Only show groups that have enabled integrations
  return integrations
    .filter(group => group.items.some(item => item.enabled))
    .map(group => ({
      label: group.label,
      placement: ['before', 'deployment'] as ['before', 'deployment'],
      items: group.items
        .filter(item => item.enabled)
        .map(({ enabled, ...item }) => item),
    }));
}
```

### 3. Advanced Tools Group

Add power user tools and utilities:

```typescript
import { 
  FaTools, 
  FaCode, 
  FaRocket, 
  FaSearch,
  FaClipboardCheck,
  FaBug 
} from 'react-icons/fa';

settingsAreaSidebarItemGroups(ctx: SettingsAreaSidebarItemGroupsCtx) {
  // Only show for admin users
  if (ctx.currentRole.attributes.name !== 'Admin') {
    return [];
  }

  return [
    {
      label: 'Developer Tools',
      placement: ['after', 'deployment'],
      items: [
        {
          label: 'API Explorer',
          icon: <FaCode />,
          pointsTo: {
            pageId: 'apiExplorer',
          },
        },
        {
          label: 'GraphQL Playground',
          icon: <FaRocket />,
          pointsTo: {
            pageId: 'graphqlPlayground',
          },
        },
        {
          label: 'Content Search',
          icon: <FaSearch />,
          pointsTo: {
            pageId: 'advancedSearch',
          },
        },
        {
          label: 'Data Validation',
          icon: <FaClipboardCheck />,
          pointsTo: {
            pageId: 'dataValidation',
          },
        },
        {
          label: 'Debug Console',
          icon: <FaBug />,
          pointsTo: {
            pageId: 'debugConsole',
          },
        },
      ],
    },
  ];
}
```

### 4. Content Management Tools

Add content-specific management tools:

```typescript
import { 
  FaFileImport, 
  FaFileExport, 
  FaLanguage,
  FaHistory,
  FaTrash,
  FaMagic 
} from 'react-icons/fa';

settingsAreaSidebarItemGroups(ctx: SettingsAreaSidebarItemGroupsCtx) {
  const groups = [];

  // Import/Export tools
  groups.push({
    label: 'Import & Export',
    placement: ['after', 'media'] as ['after', 'media'],
    items: [
      {
        label: 'Bulk Import',
        icon: <FaFileImport />,
        pointsTo: {
          pageId: 'bulkImport',
        },
      },
      {
        label: 'Export Manager',
        icon: <FaFileExport />,
        pointsTo: {
          pageId: 'exportManager',
        },
      },
      {
        label: 'Migration Tools',
        icon: <FaHistory />,
        pointsTo: {
          pageId: 'migrationTools',
        },
      },
    ],
  });

  // Localization tools (only if multi-locale)
  if (ctx.site.locales.length > 1) {
    groups.push({
      label: 'Localization',
      placement: ['after', 'locales'] as ['after', 'locales'],
      items: [
        {
          label: 'Translation Manager',
          icon: <FaLanguage />,
          pointsTo: {
            pageId: 'translationManager',
          },
        },
        {
          label: 'Locale Sync',
          icon: <FaMagic />,
          pointsTo: {
            pageId: 'localeSync',
          },
        },
      ],
    });
  }

  // Maintenance tools
  groups.push({
    label: 'Maintenance',
    placement: ['after', 'deployment'] as ['after', 'deployment'],
    items: [
      {
        label: 'Cleanup Tools',
        icon: <FaTrash />,
        pointsTo: {
          pageId: 'cleanupTools',
        },
      },
      {
        label: 'Audit Log',
        icon: <FaHistory />,
        pointsTo: {
          pageId: 'auditLog',
        },
      },
    ],
  });

  return groups;
}
```

### 5. Dynamic Groups Based on Configuration

Create groups dynamically based on plugin configuration:

```typescript
import { FaPlug, FaPuzzlePiece, FaCogs } from 'react-icons/fa';

settingsAreaSidebarItemGroups(ctx: SettingsAreaSidebarItemGroupsCtx) {
  const groups: SettingsAreaSidebarItemGroup[] = [];
  const config = ctx.plugin.attributes.parameters;

  // Add groups based on enabled features
  if (config.features?.workflows) {
    groups.push({
      label: 'Workflow Management',
      items: [
        {
          label: 'Workflow Designer',
          icon: <FaCogs />,
          pointsTo: { pageId: 'workflowDesigner' },
        },
        {
          label: 'Approval Rules',
          icon: <FaClipboardCheck />,
          pointsTo: { pageId: 'approvalRules' },
        },
        {
          label: 'Workflow Reports',
          icon: <FaChartBar />,
          pointsTo: { pageId: 'workflowReports' },
        },
      ],
    });
  }

  if (config.features?.automations) {
    groups.push({
      label: 'Automations',
      items: [
        {
          label: 'Automation Rules',
          icon: <FaCogs />,
          pointsTo: { pageId: 'automationRules' },
        },
        {
          label: 'Scheduled Tasks',
          icon: <FaClock />,
          pointsTo: { pageId: 'scheduledTasks' },
        },
        {
          label: 'Event Triggers',
          icon: <FaBolt />,
          pointsTo: { pageId: 'eventTriggers' },
        },
      ],
    });
  }

  // Add extension marketplace if enabled
  if (config.features?.marketplace) {
    groups.push({
      label: 'Extensions',
      placement: ['before', 'deployment'] as ['before', 'deployment'],
      items: [
        {
          label: 'Browse Extensions',
          icon: <FaPlug />,
          pointsTo: { pageId: 'extensionMarketplace' },
        },
        {
          label: 'Installed Extensions',
          icon: <FaPuzzlePiece />,
          pointsTo: { pageId: 'installedExtensions' },
        },
      ],
    });
  }

  return groups;
}
```

## Best Practices

1. **Logical Grouping**: Group related settings pages together
2. **Clear Labels**: Use descriptive labels for groups and items
3. **Meaningful Icons**: Choose icons that represent the content
4. **Placement**: Use placement to position groups logically
5. **Permission Checks**: Only show groups based on user permissions
6. **Dynamic Groups**: Create groups based on configuration or features
7. **Consistent Naming**: Use consistent naming patterns across groups
8. **Limited Items**: Keep groups focused with 3-7 items each
9. **User Context**: Consider the user's role and needs
10. **Environment Awareness**: Adjust groups based on environment

## Related Hooks

- `renderPage` - Render the actual pages referenced by these items
- `mainNavigationTabs` - Add tabs to the main navigation
- `contentAreaSidebarItems` - Add items to the content area sidebar

## Important Notes

- Groups appear in the settings area sidebar
- Each item must point to a valid pageId
- The pageId must be handled by a `renderPage` hook
- Groups can be positioned relative to existing groups
- Consider user permissions when showing groups
- Groups should enhance, not clutter, the settings area