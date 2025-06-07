# onBoot Hook

The `onBoot` hook is executed once when the plugin is first loaded, allowing initialization of services, data fetching, setup tasks, and early configuration validation. This is the first hook that runs in a plugin's lifecycle, making it ideal for bootstrapping operations.

## Quick Reference

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  async onBoot(ctx) {
    // Initialize plugin services
    await initializeServices(ctx);
    
    // Validate configuration
    if (!ctx.plugin.attributes.parameters.apiKey) {
      ctx.alert('API key is required. Please configure the plugin.');
      return;
    }
    
    // Preload data
    const fields = await ctx.loadFieldsUsingPlugin();
    console.log(`Plugin loaded for ${fields.length} fields`);
  }
});
```

## Purpose

The `onBoot` hook is used for:
- Service initialization (analytics, error tracking, feature flags)
- Configuration validation and setup
- Data preloading and caching
- Environment-specific setup
- Permission checks and user preference loading
- WebSocket or real-time connection establishment

## Signature

```typescript
async onBoot(ctx: OnBootCtx): Promise<void>
```

## Type Definitions

```typescript
// Hook signature
export type OnBootHook = {
  /**
   * Use this function to execute boot operations (e.g. add event listeners, starting timers, etc.)
   *
   * @tag onBoot
   */
  onBoot: (ctx: OnBootCtx) => void;
};

// Context type
export type OnBootCtx = ImposedSizePluginFrameCtx<'onBoot'>;

// ImposedSizePluginFrameCtx base type
type ImposedSizePluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BaseProperties &
  PluginFrameAdditionalProperties<Mode> &
  AdditionalProperties &
  BaseMethods &
  PluginFrameAdditionalMethods<
    BaseProperties &
      PluginFrameAdditionalProperties<Mode> &
      AdditionalProperties
  > &
  AdditionalMethods;

// Additional properties for plugin frame
type PluginFrameAdditionalProperties<Mode extends keyof FullConnectParameters> = {
  mode: Mode;
  bodyPadding: [number, number, number, number];
};

// Additional methods for plugin frame
type PluginFrameAdditionalMethods<Properties extends Record<string, unknown>> = {
  getSettings: () => Promise<Properties>;
};

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
type FullConnectParameters = Record<string, any>;

// Type guard functions
export function isReturnTypeOfOnBootHook(value: unknown): value is void {
  return isNullish(value);
}

// Helper functions  
function isNullish(value: unknown): value is null | undefined {
  return value === null || value === undefined;
}
```

## Context Properties

The `onBoot` hook receives a context object with all base properties and methods:

### Properties
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `currentUserAccessToken`: string | undefined - API token for the current user (if permission granted)
- `site`: Site - The current DatoCMS project
- `environment`: string - The ID of the current environment
- `isEnvironmentPrimary`: boolean - Whether the current environment is primary
- `owner`: Account | Organization - The project owner
- `ui`: { locale: string } - UI preferences (currently only locale)
- `theme`: Theme - Theme colors for the current project
- `itemTypes`: Partial<Record<string, ItemType>> - Models indexed by ID
- `fields`: Partial<Record<string, Field>> - Fields indexed by ID
- `fieldsets`: Partial<Record<string, Fieldset>> - Fieldsets indexed by ID
- `users`: Partial<Record<string, User>> - Regular users indexed by ID
- `ssoUsers`: Partial<Record<string, SsoUser>> - SSO users indexed by ID
- `mode`: 'onBoot' - The current hook mode
- `bodyPadding`: [number, number, number, number] - Frame padding values

### Methods
All standard context methods are available, including:
- Data loading: `loadItemTypeFields()`, `loadFieldsUsingPlugin()`, `loadUsers()`, etc.
- Plugin configuration: `updatePluginParameters()`, `updateFieldAppearance()`
- Notifications: `alert()`, `notice()`, `customToast()`
- Dialogs: `openModal()`, `openConfirm()`
- Navigation: `navigateTo()`

## Use Cases

### 1. Service Initialization

Initialize third-party services and APIs that the plugin depends on:

```typescript
async onBoot(ctx: OnBootCtx) {
  // Initialize analytics service
  if (ctx.plugin.attributes.parameters.analyticsKey) {
    await initializeAnalytics({
      apiKey: ctx.plugin.attributes.parameters.analyticsKey,
      projectId: ctx.site.id,
      environment: ctx.environment
    });
  }
  
  // Set up error monitoring
  if (ctx.plugin.attributes.parameters.sentryDsn) {
    setupErrorTracking({
      dsn: ctx.plugin.attributes.parameters.sentryDsn,
      user: ctx.currentUser.id,
      environment: ctx.environment
    });
  }
  
  // Initialize feature flags
  await initializeFeatureFlags({
    apiKey: ctx.plugin.attributes.parameters.featureFlagKey,
    userId: ctx.currentUser.id
  });
  
  ctx.notice('Plugin services initialized successfully');
}
```

### 2. Configuration Validation

Validate plugin configuration and alert users to missing or invalid settings:

```typescript
async onBoot(ctx: OnBootCtx) {
  const params = ctx.plugin.attributes.parameters;
  const errors: string[] = [];
  
  // Check required configuration
  if (!params.apiEndpoint) {
    errors.push('API endpoint is not configured');
  } else {
    // Validate endpoint format
    try {
      new URL(params.apiEndpoint);
    } catch {
      errors.push('API endpoint is not a valid URL');
    }
  }
  
  if (!params.apiKey) {
    errors.push('API key is missing');
  }
  
  // Verify API connectivity
  if (params.apiEndpoint && params.apiKey) {
    try {
      const response = await fetch(`${params.apiEndpoint}/health`, {
        headers: { 'Authorization': `Bearer ${params.apiKey}` }
      });
      
      if (!response.ok) {
        errors.push('Failed to connect to API - please check your credentials');
      }
    } catch (error) {
      errors.push('Cannot reach API endpoint - please check the URL');
    }
  }
  
  // Report configuration issues
  if (errors.length > 0) {
    await ctx.alert(
      'Plugin configuration issues:\n' + 
      errors.map(e => `• ${e}`).join('\n') +
      '\n\nPlease update the plugin settings.'
    );
    
    // Optionally navigate to settings
    if (ctx.currentRole.attributes.can_edit_schema) {
      await ctx.navigateTo(`/admin/plugins/${ctx.plugin.id}/edit`);
    }
  }
}
```

### 3. Data Preloading and Caching

Load and cache frequently used data to improve plugin performance:

```typescript
async onBoot(ctx: OnBootCtx) {
  // Load all fields using this plugin
  const fieldsUsingPlugin = await ctx.loadFieldsUsingPlugin();
  
  // Cache field configurations
  const fieldConfigs = new Map();
  for (const field of fieldsUsingPlugin) {
    if (field.attributes.appearance.editor === ctx.plugin.id) {
      fieldConfigs.set(field.id, {
        fieldType: field.attributes.field_type,
        validators: field.attributes.validators,
        parameters: field.attributes.appearance.parameters
      });
    }
  }
  
  // Store in global cache
  window.__pluginFieldCache = fieldConfigs;
  
  // Preload common data
  const [users, models] = await Promise.all([
    ctx.loadUsers(),
    loadFrequentlyUsedModels(ctx)
  ]);
  
  // Initialize local storage
  localStorage.setItem(`plugin_${ctx.plugin.id}_initialized`, Date.now().toString());
  
  ctx.notice(`Plugin initialized with ${fieldConfigs.size} configured fields`);
}

async function loadFrequentlyUsedModels(ctx: OnBootCtx) {
  // Load models that are commonly accessed
  const modelIds = ctx.plugin.attributes.parameters.cachedModels || [];
  const models = [];
  
  for (const modelId of modelIds) {
    if (ctx.itemTypes[modelId]) {
      const fields = await ctx.loadItemTypeFields(modelId);
      models.push({
        model: ctx.itemTypes[modelId],
        fields
      });
    }
  }
  
  return models;
}
```

### 4. Environment Setup

Set up environment-specific configurations and features:

```typescript
async onBoot(ctx: OnBootCtx) {
  // Configure based on environment
  const config = {
    apiUrl: ctx.isEnvironmentPrimary 
      ? ctx.plugin.attributes.parameters.productionApi
      : ctx.plugin.attributes.parameters.stagingApi,
    debugMode: !ctx.isEnvironmentPrimary,
    features: {}
  };
  
  // Enable environment-specific features
  if (ctx.environment === 'development') {
    config.features = {
      experimentalEditor: true,
      debugPanel: true,
      verboseLogging: true
    };
  }
  
  // Set up logging
  if (config.debugMode) {
    window.__pluginDebug = {
      enabled: true,
      startTime: Date.now(),
      environment: ctx.environment,
      user: ctx.currentUser.id
    };
    
    console.log('[Plugin Debug] Initialized in debug mode', {
      environment: ctx.environment,
      site: ctx.site.id,
      plugin: ctx.plugin.id
    });
  }
  
  // Store configuration
  window.__pluginConfig = config;
  
  // Show environment indicator
  if (!ctx.isEnvironmentPrimary) {
    ctx.customToast({
      type: 'warning',
      message: `Plugin running in ${ctx.environment} environment`,
      dismissAfterTimeout: 5000
    });
  }
}
```

### 5. Permission Checks

Verify user permissions and adjust plugin behavior accordingly:

```typescript
async onBoot(ctx: OnBootCtx) {
  const permissions = ctx.currentRole.attributes.can_edit_schema;
  const userType = ctx.currentUser.type;
  
  // Store permission state
  const pluginPermissions = {
    canEditSchema: permissions,
    canEditContent: ctx.currentRole.attributes.can_edit_content,
    canPublish: ctx.currentRole.attributes.can_publish_content,
    isOwner: userType === 'account' || userType === 'organization',
    isAdmin: ctx.currentRole.attributes.admin
  };
  
  window.__pluginPermissions = pluginPermissions;
  
  // Warn about limited permissions
  if (!pluginPermissions.canEditSchema && hasSchemaEditingFeatures(ctx.plugin)) {
    ctx.alert(
      'You have limited permissions. Some plugin features may not be available.'
    );
  }
  
  // Load user-specific data
  if (userType === 'user' || userType === 'sso_user') {
    const preferences = await loadUserPreferences(ctx.currentUser.id);
    applyUserPreferences(preferences);
  }
  
  // Initialize role-based features
  initializeFeatures(pluginPermissions);
}

function hasSchemaEditingFeatures(plugin: Plugin): boolean {
  // Check if plugin has features that require schema editing
  return plugin.attributes.field_types.length > 0 || 
         plugin.attributes.parameter_definitions.some(p => p.type === 'model');
}
```

### 6. WebSocket or Real-time Connections

Establish persistent connections for real-time features:

```typescript
async onBoot(ctx: OnBootCtx) {
  if (!ctx.plugin.attributes.parameters.enableRealtime) {
    return;
  }
  
  try {
    // Initialize WebSocket connection
    const ws = new WebSocket(ctx.plugin.attributes.parameters.websocketUrl);
    
    ws.onopen = () => {
      // Authenticate connection
      ws.send(JSON.stringify({
        type: 'auth',
        token: ctx.currentUserAccessToken,
        site: ctx.site.id,
        environment: ctx.environment
      }));
      
      ctx.notice('Real-time connection established');
    };
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      handleRealtimeMessage(data);
    };
    
    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
      ctx.alert('Failed to establish real-time connection');
    };
    
    // Store connection reference
    window.__pluginWebSocket = ws;
    
    // Clean up on unload
    window.addEventListener('beforeunload', () => {
      ws.close();
    });
    
  } catch (error) {
    ctx.alert(`Real-time initialization failed: ${error.message}`);
  }
}
```

## Best Practices

1. **Fast Initialization**: Keep the onBoot hook fast to avoid delaying plugin startup. Use async operations wisely and consider deferring non-critical tasks.

2. **Error Handling**: Always wrap initialization code in try-catch blocks and provide meaningful error messages to users.

3. **Configuration Validation**: Validate all required configuration early and guide users to fix any issues.

4. **Resource Cleanup**: If initializing resources that need cleanup (WebSockets, intervals, etc.), set up proper cleanup handlers.

5. **Progressive Enhancement**: Design the plugin to work with minimal configuration and enhance functionality as more settings are provided.

6. **Respect Permissions**: Check user permissions before initializing features that require specific access levels.

7. **Environment Awareness**: Adapt behavior based on the current environment (development, staging, production).

8. **Caching Strategy**: Cache expensive operations but implement cache invalidation strategies.

## Common Patterns

### Conditional Initialization

```typescript
async onBoot(ctx: OnBootCtx) {
  // Only initialize if properly configured
  if (!isPluginConfigured(ctx.plugin)) {
    ctx.alert('Please configure the plugin before use');
    return;
  }
  
  // Initialize based on feature flags
  const features = ctx.plugin.attributes.parameters.features || {};
  
  if (features.analytics) {
    await initAnalytics(ctx);
  }
  
  if (features.customValidation) {
    await loadValidationRules(ctx);
  }
  
  if (features.aiIntegration) {
    await connectToAIService(ctx);
  }
}
```

### Multi-step Initialization

```typescript
async onBoot(ctx: OnBootCtx) {
  const steps = [
    { name: 'config', fn: validateConfiguration },
    { name: 'auth', fn: authenticateServices },
    { name: 'data', fn: preloadData },
    { name: 'ui', fn: setupUIComponents }
  ];
  
  for (const step of steps) {
    try {
      await step.fn(ctx);
    } catch (error) {
      ctx.alert(`Initialization failed at ${step.name}: ${error.message}`);
      // Decide whether to continue or abort
      if (step.name === 'config' || step.name === 'auth') {
        return; // Critical steps - abort
      }
      // Non-critical steps - continue with degraded functionality
    }
  }
  
  ctx.notice('Plugin initialized successfully');
}
```

## Related Hooks

- **renderConfigScreen**: Often used together with onBoot to handle initial configuration
- **onBeforeItemUpsert**: May use data cached during onBoot
- **manualFieldExtensions**: Fields configured here might be initialized in onBoot
- **renderFieldExtension**: The actual field rendering that follows initialization

## Important Notes

- The onBoot hook runs only once when the plugin loads, not on every page navigation
- It has access to the full plugin context including parameters and user information
- Long-running operations in onBoot will delay the plugin's availability to users
- Any global state initialized here persists for the plugin session
- Errors thrown in onBoot will prevent the plugin from loading properly
- The hook cannot be cancelled like some other lifecycle hooks