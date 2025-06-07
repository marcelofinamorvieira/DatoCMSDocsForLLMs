# mainNavigationTabs Hook

The `mainNavigationTabs` hook allows plugins to add custom tabs to the main navigation bar in DatoCMS. These tabs appear alongside the default tabs (Content, Media, Schema, etc.) and provide quick access to plugin functionality, making it ideal for frequently-used tools, dashboards, or administrative interfaces.

## Purpose

Add custom tabs to the main DatoCMS navigation bar for:
- Analytics dashboards and reporting tools
- Workflow management systems
- E-commerce administration
- Translation and localization centers
- Developer tools and API explorers
- Custom administrative interfaces

## Signature

```typescript
// The hook function signature
mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[]

// Used in plugin definition
connect({
  mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
    return [
      {
        label: 'My Tab',
        icon: 'FaRocket',
        pointsTo: { pageId: 'my-page' }
      }
    ];
  }
});
```

## Context Object (MainNavigationTabsCtx)

The context object provides access to DatoCMS information for determining which tabs to show. The `MainNavigationTabsCtx` is an alias for the base `Ctx` type.

## Type Definitions

```typescript
// Hook signature
export type MainNavigationTabsHook = {
  /**
   * Use this function to declare new tabs to be shown in the main navigation bar
   *
   * @tag navigationTabs
   */
  mainNavigationTabs: (ctx: MainNavigationTabsCtx) => MainNavigationTab[];
};

// Context type
export type MainNavigationTabsCtx = Ctx;

// Return type
export type MainNavigationTab = {
  /** Label to be shown. Must be unique. */
  label: string;
  /**
   * Icon to be shown alongside the label. Can be a FontAwesome icon name (ie.
   * "address-book") or a custom SVG definition. To maintain visual
   * consistency with the rest of the interface, try to use FontAwesome icons
   * whenever possible.
   */
  icon: Icon;
  /** ID of the page linked to the tab */
  pointsTo: {
    pageId: string;
  };
  /**
   * Expresses where you want the tab to be placed inside the navigation bar.
   * If not specified, the tab will be placed after the standard tabs provided
   * by DatoCMS itself.
   */
  placement?: [
    'before' | 'after',
    'content' | 'media' | 'schema' | 'configuration' | 'cdaPlayground'
  ];
  /**
   * If different plugins specify the same `placement` for their tabs, they
   * will be displayed by ascending `rank`. If you want to specify an explicit
   * value for `rank`, make sure to offer a way for final users to customize it
   * inside the plugin's settings form, otherwise the hardcoded value you choose
   * might clash with the one of another plugin!
   */
  rank?: number;
};

// Icon type
export type Icon = AwesomeFontIconIdentifier | SvgIcon;

export type SvgIcon = {
  type: 'svg';
  viewBox: string;
  content: string;
};

// FontAwesome icon identifier is a string matching valid icon names
export type AwesomeFontIconIdentifier = string;

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
  
  // Project Information
  site: Site;
  environment: string;
  environmentId: string;
  mainLocale: string;
  currentUserMenuLocale: string;
  theme: 'dark' | 'light';
  ui: Record<string, unknown>;
  
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
    appearance: Record<string, unknown>
  ) => Promise<void>;
  
  // Toast Notifications
  notice: (message: string) => void;
  alert: (message: string) => void;
  customToast: (toast: Toast) => void;
  
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
```

## Complete Examples

### 1. Analytics Dashboard with Multiple Tabs

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { MainNavigationTabsCtx, MainNavigationTab } from 'datocms-plugin-sdk';

connect({
  mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
    const tabs: MainNavigationTab[] = [];
    
    // Main analytics tab - available to all users
    tabs.push({
      label: 'Analytics',
      icon: 'FaChartLine',
      pointsTo: { pageId: 'analytics-overview' },
      placement: ['after', 'content'],
      rank: 10
    });
    
    // Advanced reports - only for admins
    if (ctx.currentRole.attributes.can_access_audit_log) {
      tabs.push({
        label: 'Reports',
        icon: 'FaChartBar',
        pointsTo: { pageId: 'advanced-reports' },
        placement: ['after', 'content'],
        rank: 11
      });
    }
    
    // Performance metrics - development only
    if (ctx.environment !== 'main') {
      tabs.push({
        label: 'Performance',
        icon: 'FaTachometerAlt',
        pointsTo: { pageId: 'performance-metrics' },
        placement: ['after', 'content'],
        rank: 12
      });
    }
    
    return tabs;
  },

  renderPage(pageId, ctx) {
    switch (pageId) {
      case 'analytics-overview':
        return {
          title: 'Analytics Overview',
          component: () => <AnalyticsOverview ctx={ctx} />
        };
      
      case 'advanced-reports':
        return {
          title: 'Advanced Reports',
          component: () => <AdvancedReports ctx={ctx} />
        };
        
      case 'performance-metrics':
        return {
          title: 'Performance Metrics',
          component: () => <PerformanceMetrics ctx={ctx} />
        };
    }
  }
});
```

### 2. E-commerce Management System

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { MainNavigationTabsCtx, MainNavigationTab } from 'datocms-plugin-sdk';

connect({
  mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
    const tabs: MainNavigationTab[] = [];
    
    // Check for e-commerce models
    const hasProducts = ctx.itemTypes.some(
      model => model.attributes.api_key === 'product'
    );
    const hasOrders = ctx.itemTypes.some(
      model => model.attributes.api_key === 'order'
    );
    const hasCustomers = ctx.itemTypes.some(
      model => model.attributes.api_key === 'customer'
    );
    
    if (!hasProducts) {
      return []; // No e-commerce features
    }
    
    // Main store dashboard
    tabs.push({
      label: 'Store',
      icon: 'FaStore',
      pointsTo: { pageId: 'store-dashboard' },
      placement: ['after', 'content'],
      rank: 20
    });
    
    // Orders management (admins and editors)
    if (hasOrders && ctx.currentRole.attributes.can_edit_content) {
      tabs.push({
        label: 'Orders',
        icon: 'FaShoppingCart',
        pointsTo: { pageId: 'order-management' },
        placement: ['after', 'content'],
        rank: 21
      });
    }
    
    // Customer insights
    if (hasCustomers) {
      tabs.push({
        label: 'Customers',
        icon: 'FaUsers',
        pointsTo: { pageId: 'customer-insights' },
        placement: ['after', 'content'],
        rank: 22
      });
    }
    
    // Inventory management
    tabs.push({
      label: 'Inventory',
      icon: 'FaBoxes',
      pointsTo: { pageId: 'inventory-management' },
      placement: ['after', 'content'],
      rank: 23
    });
    
    // Financial reports (admin only)
    if (ctx.currentRole.attributes.can_manage_environments) {
      tabs.push({
        label: 'Finance',
        icon: 'FaDollarSign',
        pointsTo: { pageId: 'financial-reports' },
        placement: ['before', 'configuration'],
        rank: 5
      });
    }
    
    return tabs;
  }
});
```

### 3. Multi-locale Translation Center

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { MainNavigationTabsCtx, MainNavigationTab } from 'datocms-plugin-sdk';

connect({
  mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
    // Only show for multi-locale sites
    const locales = ctx.site.attributes.locales;
    if (locales.length <= 1) {
      return [];
    }
    
    const tabs: MainNavigationTab[] = [];
    
    // Translation center
    tabs.push({
      label: 'Translations',
      icon: 'FaLanguage',
      pointsTo: { pageId: 'translation-center' },
      placement: ['after', 'content'],
      rank: 30
    });
    
    // Locale coverage reports
    tabs.push({
      label: 'Coverage',
      icon: 'FaGlobe',
      pointsTo: { pageId: 'locale-coverage' },
      placement: ['after', 'content'],
      rank: 31
    });
    
    // Translation workflow (if user can publish)
    if (ctx.currentRole.attributes.can_publish_to_production) {
      tabs.push({
        label: 'Translation Queue',
        icon: 'FaTasks',
        pointsTo: { pageId: 'translation-queue' },
        placement: ['after', 'content'],
        rank: 32
      });
    }
    
    // Machine translation settings (admin only)
    if (ctx.currentRole.attributes.can_edit_environment) {
      tabs.push({
        label: 'MT Settings',
        icon: 'FaRobot',
        pointsTo: { pageId: 'mt-settings' },
        placement: ['after', 'schema'],
        rank: 40
      });
    }
    
    return tabs;
  }
});
```

### 4. Developer Tools Suite

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { MainNavigationTabsCtx, MainNavigationTab } from 'datocms-plugin-sdk';

connect({
  mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
    const tabs: MainNavigationTab[] = [];
    const isDev = ctx.environment !== 'main';
    const isAdmin = ctx.currentRole.attributes.can_edit_schema;
    
    // API Explorer (dev environments)
    if (isDev) {
      tabs.push({
        label: 'API Explorer',
        icon: 'FaTerminal',
        pointsTo: { pageId: 'api-explorer' },
        placement: ['after', 'schema'],
        rank: 50
      });
      
      // GraphQL Playground
      tabs.push({
        label: 'GraphQL',
        icon: 'FaCode',
        pointsTo: { pageId: 'graphql-playground' },
        placement: ['after', 'schema'],
        rank: 51
      });
    }
    
    // Schema analyzer (schema editors)
    if (isAdmin) {
      tabs.push({
        label: 'Schema Tools',
        icon: 'FaDatabase',
        pointsTo: { pageId: 'schema-analyzer' },
        placement: ['after', 'schema'],
        rank: 52
      });
    }
    
    // Webhook debugger
    if (ctx.currentRole.attributes.can_manage_webhooks) {
      tabs.push({
        label: 'Webhooks',
        icon: 'FaBolt',
        pointsTo: { pageId: 'webhook-debugger' },
        placement: ['after', 'schema'],
        rank: 53
      });
    }
    
    // Activity logs (audit access)
    if (ctx.currentRole.attributes.can_access_audit_log) {
      tabs.push({
        label: 'Activity Logs',
        icon: 'FaHistory',
        pointsTo: { pageId: 'activity-logs' },
        placement: ['before', 'configuration'],
        rank: 10
      });
    }
    
    return tabs;
  }
});
```

### 5. Dynamic Module System

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { MainNavigationTabsCtx, MainNavigationTab } from 'datocms-plugin-sdk';

interface PluginModules {
  seo?: { enabled: boolean; advanced?: boolean };
  calendar?: { enabled: boolean; showWeekends?: boolean };
  workflows?: { enabled: boolean; approvalLevels?: number };
  import?: { enabled: boolean; formats?: string[] };
}

connect({
  mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
    const modules = ctx.plugin.attributes.parameters.modules as PluginModules || {};
    const tabs: MainNavigationTab[] = [];
    
    // SEO Module
    if (modules.seo?.enabled) {
      tabs.push({
        label: 'SEO',
        icon: 'FaSearch',
        pointsTo: { pageId: 'seo-dashboard' },
        placement: ['after', 'content'],
        rank: 60
      });
      
      if (modules.seo.advanced && ctx.currentRole.attributes.can_edit_schema) {
        tabs.push({
          label: 'SEO Config',
          icon: 'FaCog',
          pointsTo: { pageId: 'seo-configuration' },
          placement: ['after', 'schema'],
          rank: 61
        });
      }
    }
    
    // Calendar Module
    if (modules.calendar?.enabled) {
      tabs.push({
        label: 'Calendar',
        icon: 'FaCalendarAlt',
        pointsTo: { pageId: 'content-calendar' },
        placement: ['after', 'content'],
        rank: 70
      });
    }
    
    // Workflow Module
    if (modules.workflows?.enabled) {
      tabs.push({
        label: 'Workflows',
        icon: 'FaProjectDiagram',
        pointsTo: { pageId: 'workflow-dashboard' },
        placement: ['after', 'content'],
        rank: 80
      });
      
      // Approval queue for publishers
      if (ctx.currentRole.attributes.can_publish_to_production) {
        tabs.push({
          label: 'Approvals',
          icon: 'FaCheckCircle',
          pointsTo: { pageId: 'approval-queue' },
          placement: ['after', 'content'],
          rank: 81
        });
      }
    }
    
    // Import/Export Module
    if (modules.import?.enabled) {
      tabs.push({
        label: 'Import/Export',
        icon: 'FaExchangeAlt',
        pointsTo: { pageId: 'import-export' },
        placement: ['after', 'media'],
        rank: 90
      });
    }
    
    return tabs;
  }
});
```

### 6. Custom SVG Icons

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { MainNavigationTabsCtx, MainNavigationTab } from 'datocms-plugin-sdk';

connect({
  mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
    return [
      // FontAwesome icon
      {
        label: 'Dashboard',
        icon: 'FaTachometerAlt',
        pointsTo: { pageId: 'dashboard' }
      },
      
      // Custom SVG icon
      {
        label: 'Custom Tool',
        icon: {
          type: 'svg',
          viewBox: '0 0 24 24',
          content: `
            <path d="M12 2L2 7l10 5 10-5-10-5z"/>
            <path d="M2 17l10 5 10-5M2 12l10 5 10-5"/>
          `
        },
        pointsTo: { pageId: 'custom-tool' }
      },
      
      // Another custom SVG
      {
        label: 'Integration',
        icon: {
          type: 'svg',
          viewBox: '0 0 100 100',
          content: `
            <circle cx="30" cy="50" r="20" fill="currentColor"/>
            <circle cx="70" cy="50" r="20" fill="currentColor"/>
            <rect x="30" y="40" width="40" height="20" fill="currentColor"/>
          `
        },
        pointsTo: { pageId: 'integration-hub' },
        placement: ['after', 'media']
      }
    ];
  }
});
```

## Placement Options Reference

### Available Placement Positions

```typescript
// All possible placement combinations
const placementExamples: MainNavigationTab[] = [
  // Before content tab
  {
    label: 'Pre-Content',
    icon: 'FaLayerGroup',
    pointsTo: { pageId: 'pre-content' },
    placement: ['before', 'content']
  },
  
  // After content tab (most common)
  {
    label: 'Post-Content',
    icon: 'FaFolderPlus',
    pointsTo: { pageId: 'post-content' },
    placement: ['after', 'content']
  },
  
  // Around media tab
  {
    label: 'Media Tools',
    icon: 'FaImages',
    pointsTo: { pageId: 'media-tools' },
    placement: ['after', 'media']
  },
  
  // Around schema tab
  {
    label: 'Schema Utilities',
    icon: 'FaDatabase',
    pointsTo: { pageId: 'schema-utils' },
    placement: ['before', 'schema']
  },
  
  // Around configuration (settings)
  {
    label: 'Admin Panel',
    icon: 'FaUserShield',
    pointsTo: { pageId: 'admin' },
    placement: ['before', 'configuration']
  },
  
  // Around CDA Playground
  {
    label: 'API Tools',
    icon: 'FaCode',
    pointsTo: { pageId: 'api-tools' },
    placement: ['after', 'cdaPlayground']
  }
];
```

### Rank Priority System

```typescript
// Tabs with same placement are sorted by rank
const rankedTabs: MainNavigationTab[] = [
  {
    label: 'First',
    icon: 'Fa1',
    pointsTo: { pageId: 'first' },
    placement: ['after', 'content'],
    rank: 10  // Appears first
  },
  {
    label: 'Second',
    icon: 'Fa2',
    pointsTo: { pageId: 'second' },
    placement: ['after', 'content'],
    rank: 20  // Appears second
  },
  {
    label: 'Third',
    icon: 'Fa3',
    pointsTo: { pageId: 'third' },
    placement: ['after', 'content'],
    rank: 30  // Appears third
  }
];
```

## Best Practices

### 1. Tab Design
- Keep labels short (1-2 words) to fit in navigation
- Use clear, recognizable icons
- Group related functionality into single tabs
- Consider mobile/responsive views

### 2. Permission Checks
```typescript
mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
  const tabs: MainNavigationTab[] = [];
  
  // Basic tab for all users
  tabs.push({
    label: 'Dashboard',
    icon: 'FaTachometerAlt',
    pointsTo: { pageId: 'dashboard' }
  });
  
  // Advanced features based on permissions
  if (ctx.currentRole.attributes.can_edit_schema) {
    tabs.push({
      label: 'Schema Tools',
      icon: 'FaDatabase',
      pointsTo: { pageId: 'schema-tools' }
    });
  }
  
  return tabs;
}
```

### 3. Environment Awareness
```typescript
mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
  // Show different tabs per environment
  if (ctx.environment === 'main') {
    return [{
      label: 'Production',
      icon: 'FaServer',
      pointsTo: { pageId: 'prod-dashboard' }
    }];
  }
  
  return [{
    label: 'Dev Tools',
    icon: 'FaCode',
    pointsTo: { pageId: 'dev-dashboard' }
  }];
}
```

### 4. Feature Detection
```typescript
mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
  // Only show tabs if required models exist
  const hasRequiredModels = ctx.itemTypes.some(
    model => ['product', 'order'].includes(model.attributes.api_key)
  );
  
  if (!hasRequiredModels) {
    return [];
  }
  
  return [{
    label: 'E-commerce',
    icon: 'FaShoppingCart',
    pointsTo: { pageId: 'ecommerce' }
  }];
}
```

### 5. Configuration-Based Tabs
```typescript
mainNavigationTabs(ctx: MainNavigationTabsCtx): MainNavigationTab[] {
  const config = ctx.plugin.attributes.parameters;
  const tabs: MainNavigationTab[] = [];
  
  // Make rank configurable to avoid conflicts
  const baseRank = config.navigationRank || 100;
  
  if (config.showAnalytics) {
    tabs.push({
      label: 'Analytics',
      icon: 'FaChartLine',
      pointsTo: { pageId: 'analytics' },
      rank: baseRank
    });
  }
  
  return tabs;
}
```

## Common Icon Choices

```typescript
// Commonly used FontAwesome icons for navigation
const iconReference = {
  // Analytics & Reports
  analytics: 'FaChartLine',
  reports: 'FaChartBar',
  dashboard: 'FaTachometerAlt',
  metrics: 'FaChartPie',
  
  // Content & Media
  content: 'FaFileAlt',
  media: 'FaImages',
  calendar: 'FaCalendarAlt',
  tasks: 'FaTasks',
  
  // Tools & Settings
  tools: 'FaTools',
  settings: 'FaCog',
  config: 'FaSlidersH',
  admin: 'FaUserShield',
  
  // Development
  code: 'FaCode',
  api: 'FaPlug',
  terminal: 'FaTerminal',
  database: 'FaDatabase',
  
  // E-commerce
  store: 'FaStore',
  cart: 'FaShoppingCart',
  orders: 'FaBoxes',
  customers: 'FaUsers',
  
  // Workflows
  workflow: 'FaProjectDiagram',
  approval: 'FaCheckCircle',
  queue: 'FaListOl',
  
  // Import/Export
  import: 'FaFileImport',
  export: 'FaFileExport',
  sync: 'FaSync',
  exchange: 'FaExchangeAlt'
};
```

## Supporting Types

```typescript
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
    can_manage_webhooks: boolean;
    can_access_audit_log: boolean;
    can_publish_to_production: boolean;
    can_publish_content: boolean;
    can_edit_environment: boolean;
    admin: boolean;
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

// Toast type
type Toast = {
  message: string;
  type?: 'notice' | 'alert' | 'warning';
  dismissAfterTimeout?: number;
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
```

## Related Hooks

- [`renderPage`](./renderPage.md) - Implements the pages that tabs point to
- [`contentAreaSidebarItems`](./contentAreaSidebarItems.md) - Add items to content sidebar
- [`settingsAreaSidebarItemGroups`](./settingsAreaSidebarItemGroups.md) - Add settings items

## Troubleshooting

### Tab Not Appearing
1. Verify the hook returns a non-empty array
2. Check placement references valid tab names
3. Ensure pageId matches renderPage implementation
4. Verify no permission checks blocking display

### Icon Not Showing
1. Use valid FontAwesome icon names (with 'Fa' prefix)
2. For custom SVGs, ensure valid viewBox and content
3. Check browser console for errors

### Navigation Cluttered
1. Limit tabs to essential functionality
2. Consolidate related features into single tabs
3. Use submenu patterns within pages
4. Consider using sidebar items instead

### Responsive Issues
1. Test with long labels on narrow screens
2. Keep labels to 1-2 words maximum
3. Use icons that remain clear at small sizes
4. Consider hiding less critical tabs on mobile