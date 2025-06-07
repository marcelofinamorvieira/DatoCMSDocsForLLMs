# itemFormSidebars Hook

## Purpose

The `itemFormSidebars` hook allows plugins to add entire custom sidebars to record editing forms. Unlike sidebar panels which add sections to the existing sidebar, this hook creates completely new sidebars that appear as additional tabs alongside the default sidebars, providing dedicated spaces for complex plugin functionality.

## Signature

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[]
```

## Parameters

- `itemType`: ItemType - The model type for which to show sidebars
- `ctx`: ItemFormSidebarsCtx - The context object with all standard properties

## Type Definitions

```typescript
// Main hook type
export type ItemFormSidebarsHook = {
  itemFormSidebars: (
    itemType: ItemType,
    ctx: ItemFormSidebarsCtx,
  ) => ItemFormSidebar[];
};

// Context type extends base context
export type ItemFormSidebarsCtx = Ctx;

// Return type for each sidebar definition
export type ItemFormSidebar = {
  id: string;                          // Unique identifier for the sidebar
  label: string;                       // Tab label shown to users
  icon?: Icon;                         // Optional FontAwesome icon name or custom SVG object
  rank?: number;                       // Display order (lower numbers appear first)
  preferredWidth?: number;             // Preferred sidebar width in pixels (default: 350)
  parameters?: Record<string, unknown>; // Optional parameters passed to render hook
};

// Icon type for sidebar tabs
type Icon = string | {
  type: 'svg';
  viewBox: string;
  content: string;
};

// Base context type with all properties and methods
type Ctx<AdditionalProperties = {}> = BaseProperties & BaseMethods & AdditionalProperties;

// Base properties available in all contexts
type BaseProperties = {
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken?: string;
  site: Site;
  environment: string;
  theme: Theme;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  menuItems: MenuItem[];
  schema: Schema;
  users: Partial<Record<string, User>>;
};

// Base UI methods available in all contexts
type BaseMethods = {
  notice: (message: string) => void;
  alert: (message: string, options?: { cancel?: ButtonOptions }) => Promise<void>;
  confirm: (message: string, options?: { 
    cancel?: ButtonOptions; 
    choices?: ButtonChoices 
  }) => Promise<boolean | number | null>;
  prompt: (message: string, options?: { 
    cancel?: ButtonOptions; 
    placeholder?: string; 
    initialValue?: string 
  }) => Promise<string | null>;
  openModal: (options: ModalOptions) => Promise<void>;
  navigateTo: (path: string) => void;
  openUrl: (url: string) => void;
};

// Plugin configuration
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description?: string;
    url: string;
    parameters: Record<string, unknown>;
    field_types: string[];
    plugin_type: 'field_editor' | 'field_addon' | 'sidebar' | 'asset_source' | 'page' | 'modal';
  };
  relationships: {
    installed_at_site?: { data: { id: string; type: 'site' } };
  };
};

// User type definitions
type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    avatar_url?: string;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_publish_to_production: boolean;
    two_factor_authentication_active?: boolean;
  };
  relationships: {
    role: { data: { id: string; type: 'role' } };
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
  };
  relationships: Record<string, any>;
};

type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
    company?: string;
  };
  relationships: Record<string, any>;
};

type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
    logo_url?: string;
  };
  relationships: Record<string, any>;
};

// Role permissions
type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_publish_to_production: boolean;
    can_manage_webhooks: boolean;
    can_manage_environments: boolean;
    can_access_build_triggers: boolean;
    can_manage_seo_readability_analysis: boolean;
    positive_item_type_permissions: string[];
    negative_item_type_permissions: string[];
    meta_permissions: Record<string, any>;
  };
  relationships: Record<string, any>;
};

// Site configuration
type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    domain?: string;
    internal_domain: string;
    locales: string[];
    timezone: string;
    theme: Record<string, any>;
    no_index: boolean;
    global_seo?: Record<string, any>;
    favicon?: any;
    favicon_meta_tags?: any[];
  };
  relationships: Record<string, any>;
};

// Theme configuration
type Theme = {
  primary_color?: string;
  dark_color?: string;
  light_color?: string;
  accent_color?: string;
};

// ItemType (model) definition
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    tree: boolean;
    modular_block: boolean;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'table';
    has_singleton_item: boolean;
    hint?: string;
    preview_field?: string;
    title_field?: string;
    image_preview_field?: string;
    excerpt_field?: string;
    workflow?: any;
    ordering_direction?: 'asc' | 'desc';
    ordering_field?: any;
    some_locales_required?: boolean;
    versioning_enabled?: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: 'field' }> };
    fieldsets: { data: Array<{ id: string; type: 'fieldset' }> };
    singleton_item?: { data: { id: string; type: 'item' } | null };
    ordering_field?: { data: { id: string; type: 'field' } | null };
    title_field?: { data: { id: string; type: 'field' } | null };
    image_preview_field?: { data: { id: string; type: 'field' } | null };
    excerpt_field?: { data: { id: string; type: 'field' } | null };
  };
};

// Field definition
type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    field_type: 'boolean' | 'color' | 'date' | 'date_time' | 'file' | 'float' | 'gallery' | 'integer' | 'json' | 'lat_lon' | 'link' | 'links' | 'rich_text' | 'seo' | 'single_line_string' | 'slug' | 'structured_text' | 'text' | 'video';
    api_key: string;
    localized: boolean;
    validators: Record<string, any>;
    position: number;
    hint?: string;
    default_value?: any;
    appearance: {
      addons: any[];
      editor: string;
      parameters: Record<string, any>;
      type: 'addon' | 'editor' | 'presentation';
    };
    deep_filtering_enabled?: boolean;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fieldset?: { data: { id: string; type: 'fieldset' } | null };
  };
};

// Fieldset (field group)
type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    hint?: string;
    collapsible: boolean;
    start_collapsed: boolean;
    position: number;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fields: { data: Array<{ id: string; type: 'field' }> };
  };
};

// Menu structure
type MenuItem = {
  id: string;
  type: 'menu_item';
  attributes: {
    label: string;
    position: number;
    external_url?: string;
    open_in_new_tab?: boolean;
  };
  relationships: {
    item_type?: { data: { id: string; type: 'item_type' } | null };
    parent?: { data: { id: string; type: 'menu_item' } | null };
    children?: { data: Array<{ id: string; type: 'menu_item' }> };
  };
};

// Schema information
type Schema = {
  models: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
};

// UI interaction types
type ButtonOptions = {
  label: string;
  value: boolean;
  intent?: 'primary' | 'secondary' | 'negative';
};

type ButtonChoices = Array<{
  label: string;
  value: boolean | number;
  intent?: 'primary' | 'secondary' | 'negative';
}>;

type ModalOptions = {
  id: string;
  title: string;
  width?: 'xs' | 's' | 'm' | 'l' | 'xl' | 'fullWidth';
  parameters?: Record<string, unknown>;
};
```

## Return Value

The hook should return an array of `ItemFormSidebar` objects with the following structure:

```typescript
type ItemFormSidebar = {
  id: string;               // Unique identifier for the sidebar
  label: string;            // Tab label shown to users
  icon?: Icon;              // Optional FontAwesome icon name or custom SVG object
  rank?: number;            // Display order (lower numbers appear first)
  preferredWidth?: number;  // Preferred sidebar width in pixels (default: 350)
}

type Icon = string | {
  type: 'svg';
  viewBox: string;
  content: string;
}
```

## Use Cases

### 1. AI Writing Assistant

Add a comprehensive AI assistant sidebar:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Only for content-heavy models
  if (!['blog_post', 'article', 'page', 'product'].includes(itemType.attributes.api_key)) {
    return sidebars;
  }

  sidebars.push({
    id: 'ai-assistant',
    label: 'AI Assistant',
    icon: 'robot',
    rank: 10,
    preferredWidth: 400
  });

  return sidebars;
}
```

### 2. Version Control System

Add a dedicated version history sidebar:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Add version control for all models
  sidebars.push({
    id: 'version-control',
    label: 'Version History',
    icon: 'code-branch',
    rank: 20,
    preferredWidth: 350
  });

  // Add comparison tool for specific models
  if (itemType.attributes.versioning_enabled) {
    sidebars.push({
      id: 'version-compare',
      label: 'Compare Versions',
      icon: 'code-compare',
      rank: 25,
      preferredWidth: 600
    });
  }

  return sidebars;
}
```

### 3. SEO & Marketing Tools

Create a comprehensive SEO sidebar:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Check if model has SEO fields
  const hasSeoFields = Object.values(ctx.fields).some(
    f => f.relationships.item_type.data.id === itemType.id &&
         (f.attributes.api_key.includes('seo') || f.attributes.api_key.includes('meta'))
  );

  if (!hasSeoFields) {
    return sidebars;
  }

  sidebars.push({
    id: 'seo-tools',
    label: 'SEO & Marketing',
    icon: 'search',
    rank: 15,
    preferredWidth: 450
  });

  // Add social media preview for published content
  if (['blog_post', 'article', 'page'].includes(itemType.attributes.api_key)) {
    sidebars.push({
      id: 'social-preview',
      label: 'Social Media',
      icon: 'share-alt',
      rank: 30,
      preferredWidth: 400
    });
  }

  return sidebars;
}
```

### 4. Translation Management

Add translation tools for multilingual content:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Only for sites with multiple locales
  if (ctx.site.attributes.locales.length <= 1) {
    return sidebars;
  }

  // Translation sidebar for all localized models
  if (itemType.attributes.all_locales_required || itemType.attributes.some_locales_required) {
    sidebars.push({
      id: 'translations',
      label: 'Translations',
      icon: 'language',
      rank: 25,
      preferredWidth: 500
    });
  }

  // Machine translation for specific models
  if (['blog_post', 'article', 'product'].includes(itemType.attributes.api_key)) {
    sidebars.push({
      id: 'auto-translate',
      label: 'Auto Translate',
      icon: 'globe',
      rank: 26,
      preferredWidth: 400
    });
  }

  return sidebars;
}
```

### 5. Media & Asset Management

Create a media management sidebar:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Check if model has file/gallery fields
  const hasMediaFields = Object.values(ctx.fields).some(
    f => f.relationships.item_type.data.id === itemType.id &&
         ['file', 'gallery'].includes(f.attributes.field_type)
  );

  if (!hasMediaFields) {
    return sidebars;
  }

  sidebars.push({
    id: 'media-manager',
    label: 'Media Library',
    icon: 'images',
    rank: 35,
    preferredWidth: 450
  });

  // Advanced image editing for specific models
  if (['product', 'gallery_item', 'portfolio'].includes(itemType.attributes.api_key)) {
    sidebars.push({
      id: 'image-editor',
      label: 'Image Editor',
      icon: 'paint-brush',
      rank: 36,
      preferredWidth: 500
    });
  }

  return sidebars;
}
```

### 6. Analytics & Insights

Add analytics dashboard sidebar:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Analytics for published content
  if (['blog_post', 'article', 'page', 'product'].includes(itemType.attributes.api_key)) {
    sidebars.push({
      id: 'analytics',
      label: 'Analytics',
      icon: 'chart-line',
      rank: 40,
      preferredWidth: 450
    });
  }

  // E-commerce specific analytics
  if (itemType.attributes.api_key === 'product') {
    sidebars.push({
      id: 'sales-analytics',
      label: 'Sales Data',
      icon: 'shopping-cart',
      rank: 41,
      preferredWidth: 400
    });
  }

  // User engagement for interactive content
  if (['quiz', 'survey', 'form'].includes(itemType.attributes.api_key)) {
    sidebars.push({
      id: 'engagement-metrics',
      label: 'Engagement',
      icon: 'users',
      rank: 42,
      preferredWidth: 380
    });
  }

  return sidebars;
}
```

### 7. Workflow & Collaboration

Create workflow management sidebar:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Check for workflow fields
  const hasWorkflow = Object.values(ctx.fields).some(
    f => f.relationships.item_type.data.id === itemType.id &&
         ['workflow_stage', 'assigned_to', 'due_date'].includes(f.attributes.api_key)
  );

  if (!hasWorkflow) {
    return sidebars;
  }

  sidebars.push({
    id: 'workflow',
    label: 'Workflow',
    icon: 'tasks',
    rank: 50,
    preferredWidth: 400
  });

  // Add collaboration features
  sidebars.push({
    id: 'collaboration',
    label: 'Comments',
    icon: 'comments',
    rank: 51,
    preferredWidth: 350
  });

  // Task management for complex workflows
  if (ctx.currentRole.attributes.can_edit_schema) {
    sidebars.push({
      id: 'task-manager',
      label: 'Tasks',
      icon: 'check-square',
      rank: 52,
      preferredWidth: 380
    });
  }

  return sidebars;
}
```

## Conditional Display

Control sidebar visibility based on conditions:

```typescript
itemFormSidebars(
  itemType: ItemType,
  ctx: ItemFormSidebarsCtx
): ItemFormSidebar[] {
  const sidebars: ItemFormSidebar[] = [];

  // Environment-specific sidebars
  if (ctx.environment !== 'main') {
    sidebars.push({
      id: 'debug-tools',
      label: 'Debug',
      icon: 'bug',
      rank: 100,
      preferredWidth: 400
    });
  }

  // Role-based sidebars
  if (ctx.currentRole.attributes.can_publish_to_production) {
    sidebars.push({
      id: 'publishing-tools',
      label: 'Publishing',
      icon: 'upload',
      rank: 60,
      preferredWidth: 350
    });
  }

  // Plugin configuration based
  const config = ctx.plugin.attributes.parameters;
  
  if (config.enableAdvancedFeatures) {
    sidebars.push({
      id: 'advanced-tools',
      label: 'Advanced',
      icon: 'cog',
      rank: 90,
      preferredWidth: 450
    });
  }

  // API access based
  if (ctx.currentUserAccessToken) {
    sidebars.push({
      id: 'api-tools',
      label: 'API Tools',
      icon: 'terminal',
      rank: 95,
      preferredWidth: 500
    });
  }

  return sidebars;
}
```

## Icon Customization

Use FontAwesome icons or custom SVGs:

```typescript
// FontAwesome icon
{
  id: 'analytics',
  label: 'Analytics',
  icon: 'chart-line'  // Any valid FontAwesome icon
}

// Custom SVG icon
{
  id: 'custom-tools',
  label: 'Custom Tools',
  icon: {
    type: 'svg',
    viewBox: '0 0 24 24',
    content: '<path d="M12 2L2 7l10 5 10-5-10-5z"/>'
  }
}
```

## Best Practices

1. **Clear Labels**: Use concise, descriptive labels:
   ```typescript
   // Good
   label: 'SEO Tools'
   label: 'Version History'
   
   // Avoid
   label: 'Tools'
   label: 'History'
   ```

2. **Appropriate Icons**: Choose icons that represent functionality:
   ```typescript
   'robot' for AI features
   'search' for SEO
   'language' for translations
   'chart-line' for analytics
   ```

3. **Width Optimization**: Set appropriate widths:
   ```typescript
   // Simple list/form
   preferredWidth: 300
   
   // Complex dashboard
   preferredWidth: 500
   
   // Comparison view
   preferredWidth: 600
   ```

4. **Selective Display**: Only show relevant sidebars:
   ```typescript
   // Check model relevance
   if (!relevantModels.includes(itemType.attributes.api_key)) {
     return [];
   }
   ```

5. **Performance**: Limit sidebars to avoid UI clutter:
   ```typescript
   // Maximum recommended sidebars
   const MAX_SIDEBARS = 5;
   return sidebars.slice(0, MAX_SIDEBARS);
   ```

## Related Hooks

- **renderItemFormSidebar**: Implements the UI for each sidebar
- **itemFormSidebarPanels**: Add panels to existing sidebars
- **itemFormOutlets**: Add inline sections within forms

## Important Notes

- Sidebars appear as tabs in the form's right panel
- Each sidebar has its own scrollable content area
- The sidebar width can be adjusted by users but starts at `preferredWidth`
- Too many sidebars can overwhelm the interface - be selective
- Sidebars are available in both create and edit modes
- Consider mobile/responsive behavior when designing sidebar content
- The render hook receives the full form context including current values

## Validation Functions

```typescript
// Type guard for ItemFormSidebar
export function isItemFormSidebar(value: unknown): value is ItemFormSidebar {
  return (
    typeof value === 'object' &&
    value !== null &&
    typeof (value as any).id === 'string' &&
    typeof (value as any).label === 'string'
  );
}

// Type guard for hook return value
export function isReturnTypeOfItemFormSidebarsHook(
  value: unknown,
): value is ItemFormSidebar[] {
  return Array.isArray(value) && value.every(isItemFormSidebar);
}
```