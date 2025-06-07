# manualFieldExtensions Hook

The `manualFieldExtensions` hook allows plugins to declare custom field extensions that users can manually install on fields. These extensions can either replace the default field editor (type: 'editor') or add supplementary functionality below the field (type: 'addon'). This is the primary way to create reusable field components that enhance DatoCMS's field editing capabilities.

## Purpose

Create custom field extensions to:
- Replace default field editors with specialized interfaces
- Add supplementary tools and visualizations to fields
- Provide enhanced editing experiences for specific content types
- Create reusable field components across projects
- Build domain-specific input methods
- Add analytics, validation, or helper tools to any field

## Signature

```typescript
// The hook function signature
manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[]

// Used in plugin definition
connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    return [
      {
        id: 'my-extension',
        name: 'My Custom Extension',
        type: 'editor',
        fieldTypes: ['text']
      }
    ];
  }
});
```

## Context Object (ManualFieldExtensionsCtx)

The context object provides access to DatoCMS data and methods. The `ManualFieldExtensionsCtx` is an alias for the base `Ctx` type.

## Type Definitions

```typescript
// Hook signature
export type ManualFieldExtensionsHook = {
  /**
   * Use this function to declare custom field extensions that users can manually
   * install on fields
   *
   * @tag fieldExtensions
   */
  manualFieldExtensions: (ctx: ManualFieldExtensionsCtx) => ManualFieldExtension[];
};

// Context type
export type ManualFieldExtensionsCtx = Ctx;

// Return type
export type ManualFieldExtension = {
  /** Unique ID for the field extension */
  id: string;
  /** Display name in field settings */
  name: string;
  /** Extension type */
  type: 'editor' | 'addon';
  /** Compatible field types */
  fieldTypes: 'all' | FieldType[];
  /** Whether the extension needs configuration */
  configurable?: boolean | {
    /** Initial height of the configuration screen */
    initialHeight: number;
  };
  /** Initial height of the extension iframe */
  initialHeight?: number;
  /** Whether to show as sidebar panel (editor only) */
  asSidebarPanel?: boolean | {
    /** Whether panel starts open */
    startOpen: boolean;
  };
};

// Field types
type FieldType = 
  | 'boolean'          // Checkbox fields
  | 'color'            // Color picker fields
  | 'date_time'        // Date and time fields
  | 'date'             // Date only fields
  | 'file'             // Single file/media fields
  | 'float'            // Decimal number fields
  | 'gallery'          // Multiple file/media fields
  | 'integer'          // Whole number fields
  | 'json'             // JSON object fields
  | 'lat_lon'          // Geographic coordinate fields
  | 'link'             // Single record reference fields
  | 'links'            // Multiple record reference fields
  | 'rich_text'        // Rich text editor fields (deprecated)
  | 'seo'              // SEO meta fields
  | 'slug'             // URL slug fields
  | 'string'           // Single line text fields
  | 'structured_text'  // Structured text fields
  | 'text'             // Multi-line text fields
  | 'video';           // External video fields

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
```

## Complete Examples

### 1. Advanced Text Editing Suite

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { ManualFieldExtensionsCtx, ManualFieldExtension } from 'datocms-plugin-sdk';

connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    const extensions: ManualFieldExtension[] = [];
    
    // Markdown editor with live preview
    extensions.push({
      id: 'markdown-editor',
      name: 'Markdown Editor with Preview',
      type: 'editor',
      fieldTypes: ['text', 'string'],
      configurable: {
        initialHeight: 300  // Config screen for toolbar options
      },
      initialHeight: 500
    });
    
    // Code editor with syntax highlighting
    extensions.push({
      id: 'code-editor',
      name: 'Code Editor',
      type: 'editor',
      fieldTypes: ['text', 'json', 'string'],
      configurable: true,  // For language selection
      initialHeight: 600
    });
    
    // Rich text editor replacement
    extensions.push({
      id: 'custom-rich-text',
      name: 'Enhanced Rich Text Editor',
      type: 'editor',
      fieldTypes: ['structured_text', 'text'],
      configurable: {
        initialHeight: 400  // Configure allowed blocks
      },
      initialHeight: 700
    });
    
    // Character/word counter addon
    extensions.push({
      id: 'text-counter',
      name: 'Character & Word Counter',
      type: 'addon',
      fieldTypes: ['text', 'string', 'structured_text'],
      initialHeight: 80
    });
    
    // AI writing assistant
    extensions.push({
      id: 'ai-assistant',
      name: 'AI Writing Assistant',
      type: 'addon',
      fieldTypes: ['text', 'structured_text'],
      configurable: true,  // API key configuration
      initialHeight: 200
    });
    
    return extensions;
  },

  // Render the field extensions
  renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
    switch (fieldExtensionId) {
      case 'markdown-editor':
        return {
          component: () => <MarkdownEditor ctx={ctx} />
        };
      case 'code-editor':
        return {
          component: () => <CodeEditor ctx={ctx} />
        };
      case 'text-counter':
        return {
          component: () => <TextCounter ctx={ctx} />
        };
      // ... other cases
    }
  },
  
  // Configuration screens
  renderManualFieldExtensionConfigScreen(fieldExtensionId: string, ctx) {
    switch (fieldExtensionId) {
      case 'markdown-editor':
        return {
          component: () => <MarkdownEditorConfig ctx={ctx} />
        };
      case 'code-editor':
        return {
          component: () => <CodeEditorConfig ctx={ctx} />
        };
      // ... other cases
    }
  }
});
```

### 2. Media Management Extensions

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { ManualFieldExtensionsCtx, ManualFieldExtension } from 'datocms-plugin-sdk';

connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    const extensions: ManualFieldExtension[] = [];
    
    // Advanced image editor
    extensions.push({
      id: 'image-editor',
      name: 'Image Editor & Cropper',
      type: 'editor',
      fieldTypes: ['file'],
      configurable: {
        initialHeight: 350  // Configure aspect ratios, filters
      },
      initialHeight: 600
    });
    
    // Gallery organizer with drag & drop
    extensions.push({
      id: 'gallery-organizer',
      name: 'Gallery Organizer',
      type: 'editor',
      fieldTypes: ['gallery'],
      asSidebarPanel: {
        startOpen: true  // Better for managing many images
      },
      initialHeight: 700
    });
    
    // External asset picker (DAM integration)
    extensions.push({
      id: 'dam-picker',
      name: 'Digital Asset Manager',
      type: 'editor',
      fieldTypes: ['file', 'gallery'],
      configurable: true,  // DAM credentials
      initialHeight: 500
    });
    
    // Image optimization addon
    extensions.push({
      id: 'image-optimizer',
      name: 'Image Optimization Info',
      type: 'addon',
      fieldTypes: ['file', 'gallery'],
      initialHeight: 150
    });
    
    // Video preview and metadata
    extensions.push({
      id: 'video-preview',
      name: 'Video Preview & Chapters',
      type: 'addon',
      fieldTypes: ['video', 'file'],
      configurable: {
        initialHeight: 200  // Configure preview options
      },
      initialHeight: 300
    });
    
    return extensions;
  }
});
```

### 3. Data Input Enhancements

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { ManualFieldExtensionsCtx, ManualFieldExtension } from 'datocms-plugin-sdk';

connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    return [
      // Interactive map for location picking
      {
        id: 'map-picker',
        name: 'Interactive Map Picker',
        type: 'editor',
        fieldTypes: ['lat_lon'],
        configurable: {
          initialHeight: 300  // Configure map provider, style
        },
        initialHeight: 450
      },
      
      // Advanced color picker with palettes
      {
        id: 'color-palette',
        name: 'Color Palette Picker',
        type: 'editor',
        fieldTypes: ['color'],
        configurable: true,  // Define custom palettes
        initialHeight: 400
      },
      
      // JSON schema editor
      {
        id: 'json-schema-editor',
        name: 'JSON Schema Editor',
        type: 'editor',
        fieldTypes: ['json'],
        configurable: {
          initialHeight: 400  // Upload/define schema
        },
        initialHeight: 500
      },
      
      // Date range picker
      {
        id: 'date-range',
        name: 'Date Range Picker',
        type: 'editor',
        fieldTypes: ['json'],  // Stores as {start: date, end: date}
        initialHeight: 350
      },
      
      // Star rating input
      {
        id: 'star-rating',
        name: 'Star Rating',
        type: 'editor',
        fieldTypes: ['integer', 'float'],
        configurable: {
          initialHeight: 200  // Configure max stars, allow half
        },
        initialHeight: 100
      },
      
      // Tag input with autocomplete
      {
        id: 'tag-input',
        name: 'Tag Input',
        type: 'editor',
        fieldTypes: ['json', 'string'],
        configurable: true,  // Configure suggestions
        initialHeight: 200
      }
    ];
  }
});
```

### 4. SEO and Content Optimization

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { ManualFieldExtensionsCtx, ManualFieldExtension } from 'datocms-plugin-sdk';

connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    const extensions: ManualFieldExtension[] = [];
    
    // SEO analyzer addon
    extensions.push({
      id: 'seo-analyzer',
      name: 'SEO Analysis',
      type: 'addon',
      fieldTypes: ['seo', 'text', 'structured_text'],
      configurable: {
        initialHeight: 250  // Configure target keywords
      },
      initialHeight: 300
    });
    
    // Readability score
    extensions.push({
      id: 'readability',
      name: 'Readability Score',
      type: 'addon',
      fieldTypes: ['text', 'structured_text'],
      initialHeight: 150
    });
    
    // Social media preview
    extensions.push({
      id: 'social-preview',
      name: 'Social Media Preview',
      type: 'addon',
      fieldTypes: ['seo'],
      initialHeight: 400
    });
    
    // Content suggestions based on SEO
    extensions.push({
      id: 'content-suggestions',
      name: 'SEO Content Suggestions',
      type: 'addon',
      fieldTypes: ['text', 'structured_text'],
      configurable: true,  // API keys, target audience
      initialHeight: 250
    });
    
    // Schema.org markup helper
    extensions.push({
      id: 'schema-markup',
      name: 'Schema.org Markup',
      type: 'addon',
      fieldTypes: ['json', 'seo'],
      configurable: {
        initialHeight: 300
      },
      initialHeight: 350
    });
    
    return extensions;
  }
});
```

### 5. Relationship and Reference Management

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { ManualFieldExtensionsCtx, ManualFieldExtension } from 'datocms-plugin-sdk';

connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    return [
      // Advanced record browser
      {
        id: 'record-browser',
        name: 'Advanced Record Browser',
        type: 'editor',
        fieldTypes: ['link', 'links'],
        configurable: {
          initialHeight: 300  // Configure filters, display
        },
        initialHeight: 500
      },
      
      // Relationship visualizer
      {
        id: 'relationship-graph',
        name: 'Relationship Graph',
        type: 'addon',
        fieldTypes: ['link', 'links'],
        initialHeight: 400
      },
      
      // Bulk link manager
      {
        id: 'bulk-linker',
        name: 'Bulk Link Manager',
        type: 'editor',
        fieldTypes: ['links'],
        asSidebarPanel: true,  // Better for bulk operations
        initialHeight: 600
      },
      
      // Related content suggestions
      {
        id: 'related-content',
        name: 'AI Content Suggestions',
        type: 'addon',
        fieldTypes: ['link', 'links'],
        configurable: true,  // AI service configuration
        initialHeight: 250
      },
      
      // Broken link checker
      {
        id: 'link-validator',
        name: 'Link Validator',
        type: 'addon',
        fieldTypes: ['link', 'links'],
        initialHeight: 100
      }
    ];
  }
});
```

### 6. Universal Field Addons

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { ManualFieldExtensionsCtx, ManualFieldExtension } from 'datocms-plugin-sdk';

connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    // These work with any field type
    return [
      // Field notes and documentation
      {
        id: 'field-notes',
        name: 'Field Notes',
        type: 'addon',
        fieldTypes: 'all',
        configurable: {
          initialHeight: 200  // Configure note categories
        },
        initialHeight: 150
      },
      
      // Field revision history
      {
        id: 'field-history',
        name: 'Field History',
        type: 'addon',
        fieldTypes: 'all',
        initialHeight: 200
      },
      
      // Contextual help
      {
        id: 'field-help',
        name: 'Help & Guidelines',
        type: 'addon',
        fieldTypes: 'all',
        configurable: true,  // Custom help content
        initialHeight: 100
      },
      
      // Field validation status
      {
        id: 'validation-status',
        name: 'Validation Status',
        type: 'addon',
        fieldTypes: 'all',
        initialHeight: 80
      },
      
      // Translation status (for localized fields)
      {
        id: 'translation-status',
        name: 'Translation Status',
        type: 'addon',
        fieldTypes: 'all',
        initialHeight: 120
      }
    ];
  }
});
```

### 7. Conditional Extensions Based on Context

```typescript
import { connect } from 'datocms-plugin-sdk';
import type { ManualFieldExtensionsCtx, ManualFieldExtension } from 'datocms-plugin-sdk';

connect({
  manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
    const extensions: ManualFieldExtension[] = [];
    const config = ctx.plugin.attributes.parameters as {
      enabledModules?: string[];
      advancedMode?: boolean;
    };
    
    // Only add if module is enabled
    if (config.enabledModules?.includes('markdown')) {
      extensions.push({
        id: 'markdown-editor',
        name: 'Markdown Editor',
        type: 'editor',
        fieldTypes: ['text'],
        initialHeight: 500
      });
    }
    
    // Only for admin users
    if (ctx.currentRole.attributes.can_edit_schema) {
      extensions.push({
        id: 'field-debugger',
        name: 'Field Debugger',
        type: 'addon',
        fieldTypes: 'all',
        initialHeight: 200
      });
    }
    
    // Advanced features
    if (config.advancedMode) {
      extensions.push({
        id: 'json-schema-editor',
        name: 'JSON Schema Editor',
        type: 'editor',
        fieldTypes: ['json'],
        configurable: true,
        initialHeight: 600
      });
      
      extensions.push({
        id: 'regex-validator',
        name: 'Regex Validator',
        type: 'addon',
        fieldTypes: ['string', 'text'],
        configurable: true,
        initialHeight: 150
      });
    }
    
    // Multi-locale sites only
    if (ctx.site.attributes.locales.length > 1) {
      extensions.push({
        id: 'translation-helper',
        name: 'Translation Helper',
        type: 'addon',
        fieldTypes: ['string', 'text', 'structured_text'],
        asSidebarPanel: {
          startOpen: false
        },
        initialHeight: 400
      });
    }
    
    return extensions;
  }
});
```

## Configuration Patterns

### Basic Non-Configurable Extension
```typescript
{
  id: 'simple-addon',
  name: 'Simple Addon',
  type: 'addon',
  fieldTypes: ['string'],
  initialHeight: 100
}
```

### Configurable Extension
```typescript
{
  id: 'configurable-editor',
  name: 'Configurable Editor',
  type: 'editor',
  fieldTypes: ['text'],
  configurable: true,  // Boolean for default height
  initialHeight: 400
}
```

### Configurable with Custom Height
```typescript
{
  id: 'custom-config',
  name: 'Custom Config Screen',
  type: 'editor',
  fieldTypes: ['json'],
  configurable: {
    initialHeight: 350  // Custom config screen height
  },
  initialHeight: 500   // Field extension height
}
```

### Sidebar Panel Options
```typescript
// Simple sidebar (starts collapsed)
{
  id: 'sidebar-simple',
  name: 'Sidebar Tool',
  type: 'editor',
  fieldTypes: 'all',
  asSidebarPanel: true,
  initialHeight: 500
}

// Sidebar with initial state
{
  id: 'sidebar-open',
  name: 'Sidebar Tool (Open)',
  type: 'editor',
  fieldTypes: 'all',
  asSidebarPanel: {
    startOpen: true  // Starts expanded
  },
  initialHeight: 500
}
```

## Best Practices

### 1. Extension Design
- Use descriptive IDs and names
- Choose appropriate extension type (editor vs addon)
- Set reasonable initial heights
- Make extensions configurable when needed

### 2. Field Type Compatibility
```typescript
// Target specific field types
{
  id: 'text-tool',
  name: 'Text Tool',
  type: 'addon',
  fieldTypes: ['text', 'string', 'structured_text']
}

// Use 'all' sparingly
{
  id: 'universal-tool',
  name: 'Universal Tool',
  type: 'addon',
  fieldTypes: 'all'  // Only if truly universal
}
```

### 3. Performance Considerations
```typescript
manualFieldExtensions(ctx: ManualFieldExtensionsCtx): ManualFieldExtension[] {
  // Don't perform heavy operations in this hook
  // Just return extension definitions
  
  // Good: Quick config check
  const enabledExtensions = ctx.plugin.attributes.parameters.enabled || [];
  
  // Bad: Don't make API calls here
  // const data = await fetch(...) // DON'T DO THIS
  
  return enabledExtensions.map(id => extensionMap[id]);
}
```

### 4. User Experience
```typescript
// Provide clear, helpful names
{
  id: 'markdown',
  name: 'Markdown Editor with Live Preview',  // Clear about features
  type: 'editor',
  fieldTypes: ['text']
}

// Group related extensions
const textExtensions = [
  { id: 'markdown', name: 'Markdown Editor', ... },
  { id: 'code', name: 'Code Editor', ... },
  { id: 'rich-text', name: 'Rich Text Editor', ... }
];
```

## Common Patterns

### Extension Factory Pattern
```typescript
function createExtension(
  id: string,
  name: string,
  fieldTypes: FieldType[],
  options?: Partial<ManualFieldExtension>
): ManualFieldExtension {
  return {
    id,
    name,
    type: 'addon',
    fieldTypes,
    initialHeight: 200,
    ...options
  };
}

manualFieldExtensions(ctx): ManualFieldExtension[] {
  return [
    createExtension('addon1', 'Addon 1', ['text']),
    createExtension('addon2', 'Addon 2', ['string'], {
      type: 'editor',
      initialHeight: 400
    })
  ];
}
```

### Conditional Loading
```typescript
manualFieldExtensions(ctx): ManualFieldExtension[] {
  const extensions: ManualFieldExtension[] = [];
  const config = ctx.plugin.attributes.parameters;
  
  // Load based on feature flags
  if (config.features?.markdown) {
    extensions.push(markdownExtension);
  }
  
  // Load based on user role
  if (ctx.currentRole.attributes.can_edit_schema) {
    extensions.push(adminExtension);
  }
  
  // Load based on field types in use
  const hasJsonFields = ctx.fields.some(f => f.attributes.field_type === 'json');
  if (hasJsonFields) {
    extensions.push(jsonExtension);
  }
  
  return extensions;
}
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
    field_types: FieldType[];
    parameter_definitions: Array<{
      type: string;
      // ... other parameter definition fields
    }>;
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
type BaseCtx = Record<string, any>;
```

## Related Hooks

- [`renderFieldExtension`](./renderFieldExtension.md) - Implements the UI for extensions
- [`renderManualFieldExtensionConfigScreen`](./renderManualFieldExtensionConfigScreen.md) - Configuration UI
- [`validateManualFieldExtensionParameters`](./validateManualFieldExtensionParameters.md) - Validate config
- [`overrideFieldExtensions`](./overrideFieldExtensions.md) - Auto-override field editors

## Troubleshooting

### Extension Not Appearing
1. Verify the extension ID is unique
2. Check fieldTypes match the target field
3. Ensure hook returns valid array
4. Check browser console for errors

### Configuration Issues
1. Set `configurable: true` or object with height
2. Implement `renderManualFieldExtensionConfigScreen`
3. Implement `validateManualFieldExtensionParameters`
4. Check configuration is saved properly

### Height Problems
1. Set appropriate `initialHeight`
2. For config screens, use `configurable: { initialHeight: X }`
3. Consider content requirements
4. Test on different screen sizes

### Sidebar Panel Issues
1. Use `asSidebarPanel` only for editor type
2. Consider if sidebar is appropriate
3. Set `startOpen` based on use case
4. Test panel collapse/expand behavior