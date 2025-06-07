# itemFormOutlets Hook

## Purpose

The `itemFormOutlets` hook allows plugins to declare custom UI sections (outlets) that appear within record editing forms. These outlets can be positioned at various locations in the form layout, enabling plugins to add custom functionality, visualizations, or tools directly within the content editing interface.

## Hook Registration

```typescript
import type { 
  ItemFormOutletsHook,
  ItemFormOutletsCtx,
  ItemFormOutlet,
  ItemFormOutletPlacement,
  ItemType,
  Field,
  Site,
  Plugin,
  Role,
  User,
  Account,
  Organization,
  SsoUser,
  ButtonOptions,
  ButtonChoices,
  ModalOptions
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    // Implementation
    return [];
  }
};
```

## Complete Type Definitions

```typescript
// Hook function type
type ItemFormOutletsHook = {
  itemFormOutlets: (
    itemType: ItemType,
    ctx: ItemFormOutletsCtx,
  ) => ItemFormOutlet[];
};

// Context type with full type definitions
type ItemFormOutletsCtx = {
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken?: string;
  site: Site;
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  menuItems: MenuItem[];
  schema: Schema;
  users: Partial<Record<string, User>>;
  ui: {
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
  siteId: string;
};

// Return type
type ItemFormOutlet = {
  id: string;                           // Unique identifier for the outlet
  placement: ItemFormOutletPlacement;   // Where to show the outlet
  rank?: number;                        // Display order (lower numbers appear first)  
  initialHeight?: number;               // Initial iframe height in pixels
};

// Placement options
type ItemFormOutletPlacement = 
  | 'before_fields'                     // Above all fields
  | 'after_fields'                      // Below all fields
  | ['before_field', string]            // Before specific field (by API key)
  | ['after_field', string]             // After specific field (by API key)
  | ['before_fieldset', string]         // Before fieldset (by ID)
  | ['after_fieldset', string]          // After fieldset (by ID)
  | ['before_block', string]            // Before block type (by API key)
  | ['after_block', string];            // After block type (by API key)

// ItemType with complete structure
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    modular_block: boolean;
    inverse_relationships_enabled: boolean;
    created_at: string;
    updated_at: string;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: 'field' }> };
    fieldsets: { data: Array<{ id: string; type: 'fieldset' }> };
    singleton_item?: { data: { id: string; type: 'item' } | null };
    ordering_field?: { data: { id: string; type: 'field' } | null };
    ordering_direction?: 'asc' | 'desc';
    title_field?: { data: { id: string; type: 'field' } | null };
    image_preview_field?: { data: { id: string; type: 'field' } | null };
    excerpt_field?: { data: { id: string; type: 'field' } | null };
  };
  meta: {
    created_at: string;
    updated_at: string;
  };
};

// Field with complete structure
type Field = {
  id: string;
  type: 'field';
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
      fieldset_appearance?: {
        id: string;
      };
    };
    created_at: string;
    updated_at: string;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fieldset?: { data: { id: string; type: 'fieldset' } | null };
    inverse_of?: { data: { id: string; type: 'field' } | null };
  };
  meta: {
    created_at: string;
    updated_at: string;
  };
};

// Supporting types for context
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description?: string;
    url: string;
    parameters: Record<string, any>;
    field_types: string[];
    plugin_type: 'field_editor' | 'field_addon' | 'sidebar';
    tags: string[];
    permissions: {
      can_create_new_locale: boolean;
      can_edit_favicon: boolean;
      can_edit_site: boolean;
      can_edit_schema: boolean;
      can_manage_menu: boolean;
      can_manage_users: boolean;
      can_manage_access_tokens: boolean;
      can_dump_data: boolean;
      can_import_and_export: boolean;
      can_manage_webhooks: boolean;
      can_manage_build_triggers: boolean;
      can_manage_environments: boolean;
    };
  };
};

type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    locales: string[];
    theme: Record<string, any>;
    domain?: string;
    internal_domain: string;
    global_seo: Record<string, any>;
    favicon?: any;
    no_index: boolean;
    frontend_url?: string;
    ssg?: string;
    created_at: string;
    updated_at: string;
  };
};

type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_favicon: boolean;
    can_edit_site: boolean;
    can_manage_menu: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_dump_data: boolean;
    can_import_and_export: boolean;
    can_manage_webhooks: boolean;
    can_manage_build_triggers: boolean;
    can_manage_environments: boolean;
    positive_item_type_permissions: string[];
    negative_item_type_permissions: string[];
    item_type_permissions: Record<string, any>;
  };
};

type User = {
  id: string;
  type: 'user';
  attributes: {
    first_name: string;
    last_name: string;
    email: string;
    is_sso: boolean;
    state: 'active' | 'pending' | 'inactive';
  };
  relationships: {
    role: { data: { id: string; type: 'role' } };
  };
};

type SsoUser = User;
type Account = User;
type Organization = User;

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
  };
};

type MenuItem = {
  id: string;
  type: 'menu_item';
  attributes: {
    label: string;
    position: number;
    external_url?: string;
    open_in_new_tab: boolean;
  };
  relationships: {
    item_type?: { data: { id: string; type: 'item_type' } | null };
    parent?: { data: { id: string; type: 'menu_item' } | null };
    children?: { data: Array<{ id: string; type: 'menu_item' }> };
  };
};

type Schema = {
  item_types: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
  menu_items: MenuItem[];
};

type ButtonOptions = {
  label: string;
  value: any;
  intent?: 'positive' | 'negative';
};

type ButtonChoices = ButtonOptions[];

type ModalOptions = {
  id: string;
  title: string;
  width?: 'xs' | 's' | 'm' | 'l' | 'xl' | 'fullWidth';
  parameters?: Record<string, any>;
  closeDisabled?: boolean;
};
```

## Parameters

- `itemType: ItemType` - The model type for which to show outlets  
- `ctx: ItemFormOutletsCtx` - The context object containing site data, current user, and UI utilities

## Use Cases

### 1. Field-Specific Help Content

Add contextual help next to complex fields:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType, 
  Field 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Add help for SEO fields
    if (itemType.attributes.api_key === 'blog_post') {
      outlets.push({
        id: 'seo-helper',
        placement: ['before_field', 'seo_title'],
        rank: 10,
        initialHeight: 150
      });
    }

    // Add pricing calculator for products
    if (itemType.attributes.api_key === 'product') {
      outlets.push({
        id: 'pricing-calculator',
        placement: ['after_field', 'base_price'],
        rank: 5,
        initialHeight: 200
      });
    }

    // Add markdown preview for content fields
    const markdownFields: Field[] = Object.values(ctx.fields)
      .filter((f): f is Field => f !== undefined)
      .filter((f: Field) => f.relationships.item_type.data.id === itemType.id)
      .filter((f: Field) => f.attributes.appearance?.editor === 'markdown');

    markdownFields.forEach((field: Field) => {
      outlets.push({
        id: `markdown-preview-${field.attributes.api_key}`,
        placement: ['after_field', field.attributes.api_key],
        rank: 20,
        initialHeight: 300
      });
    });

    return outlets;
  }
};
```

### 2. Form-Wide Analytics

Add analytics dashboard at the top of forms:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Content performance metrics
    if (['blog_post', 'article', 'page'].includes(itemType.attributes.api_key)) {
      outlets.push({
        id: 'content-analytics',
        placement: 'before_fields',
        rank: 1,
        initialHeight: 250
      });
    }

    // E-commerce metrics for products
    if (itemType.attributes.api_key === 'product') {
      outlets.push({
        id: 'sales-metrics',
        placement: 'before_fields',
        rank: 1,
        initialHeight: 300
      });
    }

    return outlets;
  }
};
```

### 3. Fieldset Enhancements

Add tools around fieldsets:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType,
  Field 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Image optimization tools before media fieldset
    const hasMediaFieldset: boolean = Object.values(ctx.fields).some(
      (f): f is Field => f !== undefined &&
        f.relationships.item_type.data.id === itemType.id &&
        f.attributes.appearance?.fieldset_appearance?.id === 'media'
    );

    if (hasMediaFieldset) {
      outlets.push({
        id: 'image-optimization',
        placement: ['before_fieldset', 'media'],
        rank: 10,
        initialHeight: 180
      });
    }

    // Translation tools after localization fieldset
    if (itemType.attributes.all_locales_required) {
      outlets.push({
        id: 'translation-helper',
        placement: ['after_fieldset', 'localization'],
        rank: 15,
        initialHeight: 220
      });
    }

    return outlets;
  }
};
```

### 4. Structured Text Block Helpers

Add helpers for specific block types:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType,
  Field 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Check for structured text fields
    const hasStructuredText: boolean = Object.values(ctx.fields).some(
      (f): f is Field => f !== undefined &&
        f.relationships.item_type.data.id === itemType.id &&
        f.attributes.field_type === 'structured_text'
    );

    if (!hasStructuredText) {
      return outlets;
    }

    // Code block formatter
    outlets.push({
      id: 'code-formatter',
      placement: ['after_block', 'code'],
      rank: 10,
      initialHeight: 100
    });

    // Image block SEO helper
    outlets.push({
      id: 'image-alt-helper',
      placement: ['after_block', 'image'],
      rank: 15,
      initialHeight: 120
    });

    // Quote attribution helper
    outlets.push({
      id: 'quote-attribution',
      placement: ['after_block', 'quote'],
      rank: 20,
      initialHeight: 150
    });

    return outlets;
  }
};
```

### 5. Validation Summary

Add validation feedback sections:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType,
  Field 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // SEO validation summary
    const hasSeoFields: boolean = Object.values(ctx.fields).some(
      (f): f is Field => f !== undefined &&
        f.relationships.item_type.data.id === itemType.id &&
        f.attributes.api_key.includes('seo')
    );

    if (hasSeoFields) {
      outlets.push({
        id: 'seo-validation',
        placement: 'after_fields',
        rank: 90,
        initialHeight: 200
      });
    }

    // Required fields checker
    outlets.push({
      id: 'required-fields-status',
      placement: 'before_fields',
      rank: 5,
      initialHeight: 80
    });

    // Content quality score
    if (['blog_post', 'article'].includes(itemType.attributes.api_key)) {
      outlets.push({
        id: 'content-quality-score',
        placement: 'after_fields',
        rank: 85,
        initialHeight: 180
      });
    }

    return outlets;
  }
};
```

### 6. Related Content Suggestions

Show related content recommendations:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Related articles for blog posts
    if (itemType.attributes.api_key === 'blog_post') {
      outlets.push({
        id: 'related-articles',
        placement: ['after_field', 'tags'],
        rank: 25,
        initialHeight: 250
      });
    }

    // Cross-sell products
    if (itemType.attributes.api_key === 'product') {
      outlets.push({
        id: 'cross-sell-suggestions',
        placement: ['after_field', 'category'],
        rank: 30,
        initialHeight: 300
      });
    }

    // Similar FAQs
    if (itemType.attributes.api_key === 'faq') {
      outlets.push({
        id: 'similar-questions',
        placement: 'after_fields',
        rank: 80,
        initialHeight: 200
      });
    }

    return outlets;
  }
};
```

### 7. Workflow Integration

Add workflow-related outlets:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType,
  Field 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Check for workflow fields
    const hasWorkflow: boolean = Object.values(ctx.fields).some(
      (f): f is Field => f !== undefined &&
        f.relationships.item_type.data.id === itemType.id &&
        f.attributes.api_key === 'workflow_stage'
    );

    if (!hasWorkflow) {
      return outlets;
    }

    // Approval history
    outlets.push({
      id: 'approval-history',
      placement: ['after_field', 'workflow_stage'],
      rank: 10,
      initialHeight: 200
    });

    // Task assignments
    outlets.push({
      id: 'task-assignments',
      placement: 'before_fields',
      rank: 2,
      initialHeight: 150
    });

    // Comments thread
    outlets.push({
      id: 'workflow-comments',
      placement: 'after_fields',
      rank: 95,
      initialHeight: 300
    });

    return outlets;
  }
};
```

## Conditional Display

Control outlet visibility based on various conditions:

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Environment-specific outlets
    if (ctx.environment !== 'main') {
      outlets.push({
        id: 'test-data-generator',
        placement: 'before_fields',
        rank: 100,
        initialHeight: 150
      });
    }

    // Role-based outlets
    if (ctx.currentRole.attributes.can_edit_schema) {
      outlets.push({
        id: 'admin-debug-panel',
        placement: 'after_fields',
        rank: 100,
        initialHeight: 200
      });
    }

    // Locale-specific outlets
    if (ctx.site.attributes.locales.length > 1) {
      outlets.push({
        id: 'translation-status',
        placement: 'before_fields',
        rank: 3,
        initialHeight: 100
      });
    }

    // Feature flags
    const features: Record<string, any> = ctx.plugin.attributes.parameters.features || {};
    
    if (features.aiAssistant) {
      outlets.push({
        id: 'ai-writing-assistant',
        placement: ['after_field', 'content'],
        rank: 15,
        initialHeight: 250
      });
    }

    return outlets;
  }
};
```

## Best Practices

1. **Strategic Placement**: Place outlets where they're most useful:
   ```typescript
   // Near related fields
   const outlet: ItemFormOutlet = {
     id: 'pricing-calculator',
     placement: ['after_field', 'price'],  // Pricing calculator
     rank: 10,
     initialHeight: 200
   };
   
   // At form boundaries for summaries
   const summaryOutlet: ItemFormOutlet = {
     id: 'validation-summary',
     placement: 'after_fields',  // Validation summary
     rank: 90,
     initialHeight: 150
   };
   ```

2. **Unique IDs**: Include context in outlet IDs:
   ```typescript
   const placementString: string = Array.isArray(placement) ? placement.join('-') : placement;
   const outletId: string = `${itemType.attributes.api_key}-${placementString}-${purpose}`;
   ```

3. **Appropriate Heights**: Set realistic initial heights:
   ```typescript
   // Small widget
   const smallOutlet: ItemFormOutlet = {
     id: 'status-indicator',
     placement: 'before_fields',
     initialHeight: 100
   };
   
   // Complex dashboard
   const dashboardOutlet: ItemFormOutlet = {
     id: 'analytics-dashboard', 
     placement: 'after_fields',
     initialHeight: 400
   };
   ```

4. **Performance**: Limit outlets to necessary models:
   ```typescript
   const relevantModels: string[] = ['blog_post', 'article', 'product'];
   
   // Check model type first
   if (!relevantModels.includes(itemType.attributes.api_key)) {
     return [];
   }
   ```

5. **Field Verification**: Verify fields exist before using field-specific placement:
   ```typescript
   import type { Field } from 'datocms-plugin-sdk';
   
   const fieldExists: boolean = Object.values(ctx.fields).some(
     (f): f is Field => f !== undefined &&
       f.attributes.api_key === 'target_field' &&
       f.relationships.item_type.data.id === itemType.id
   );
   
   if (fieldExists) {
     outlets.push({
       id: 'field-specific-helper',
       placement: ['after_field', 'target_field'],
       rank: 10,
       initialHeight: 150
     });
   }
   ```

## Related Hooks

- **renderItemFormOutlet**: Implements the UI for each outlet
- **itemCollectionOutlets**: Add outlets to record collection pages
- **itemFormSidebarPanels**: Add panels to the form sidebar

## Important Notes

- Outlets appear inline within the form at specified placements
- Multiple outlets at the same placement are sorted by rank
- The iframe will automatically resize based on content after initial load
- Invalid field references in placement will cause the outlet to not appear
- Outlets are visible in both create and edit modes
- Consider form layout and user experience when adding multiple outlets

## Complete Working Example

```typescript
import type { 
  ItemFormOutletsHook, 
  ItemFormOutletsCtx, 
  ItemFormOutlet, 
  ItemType,
  Field 
} from 'datocms-plugin-sdk';

const plugin: ItemFormOutletsHook = {
  itemFormOutlets(
    itemType: ItemType,
    ctx: ItemFormOutletsCtx
  ): ItemFormOutlet[] {
    const outlets: ItemFormOutlet[] = [];

    // Add SEO helper for blog posts
    if (itemType.attributes.api_key === 'blog_post') {
      // Verify SEO title field exists
      const hasSeoTitleField: boolean = Object.values(ctx.fields).some(
        (f): f is Field => f !== undefined &&
          f.relationships.item_type.data.id === itemType.id &&
          f.attributes.api_key === 'seo_title'
      );

      if (hasSeoTitleField) {
        outlets.push({
          id: 'blog-seo-helper',
          placement: ['after_field', 'seo_title'],
          rank: 10,
          initialHeight: 200
        });
      }

      // Content analysis at the end
      outlets.push({
        id: 'blog-content-analysis',
        placement: 'after_fields',
        rank: 80,
        initialHeight: 250
      });
    }

    // Add pricing tools for products
    if (itemType.attributes.api_key === 'product') {
      outlets.push({
        id: 'product-pricing-calculator',
        placement: ['after_field', 'price'],
        rank: 5,
        initialHeight: 180
      });

      outlets.push({
        id: 'product-inventory-status',
        placement: 'before_fields',
        rank: 1,
        initialHeight: 100
      });
    }

    // Environment-specific debugging for admins
    if (ctx.environment !== 'main' && ctx.currentRole.attributes.can_edit_schema) {
      outlets.push({
        id: 'debug-panel',
        placement: 'after_fields',
        rank: 100,
        initialHeight: 300
      });
    }

    return outlets;
  }
};

export default plugin;
```

## Validation Functions

```typescript
// Type guard for ItemFormOutlet
export function isItemFormOutlet(value: unknown): value is ItemFormOutlet {
  return (
    typeof value === 'object' &&
    value !== null &&
    typeof (value as any).id === 'string' &&
    (
      typeof (value as any).placement === 'string' ||
      (Array.isArray((value as any).placement) && 
       (value as any).placement.length === 2 &&
       typeof (value as any).placement[0] === 'string' &&
       typeof (value as any).placement[1] === 'string')
    ) &&
    ((value as any).rank === undefined || typeof (value as any).rank === 'number') &&
    ((value as any).initialHeight === undefined || typeof (value as any).initialHeight === 'number')
  );
}

// Type guard for hook return value
export function isReturnTypeOfItemFormOutletsHook(
  value: unknown,
): value is ItemFormOutlet[] {
  return Array.isArray(value) && value.every(isItemFormOutlet);
}
```