# itemFormDropdownActions Hook

## Purpose

The `itemFormDropdownActions` hook allows plugins to add custom actions or groups of actions to the dropdown menu when editing a specific record. These actions appear in the record form's action menu and can trigger custom functionality through the `executeItemFormDropdownAction` hook.

## Signature

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup>
```

## Complete Type Definitions

### Main Hook Signature

```typescript
export type ItemFormDropdownActionsHook = {
  itemFormDropdownActions: (
    itemType: ItemType,
    ctx: ItemFormDropdownActionsCtx,
  ) => Array<DropdownAction | DropdownActionGroup>;
};
```

### Context Type Definition

```typescript

export type ItemFormDropdownActionsCtx = {
  // === Base Properties ===
  
  // Plugin information
  plugin: Plugin;
  
  // Authentication
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken: string | undefined;
  
  // Project information
  site: Site;
  environment: string;
  isEnvironmentPrimary: boolean;
  owner: Account | Organization;
  account: Account | undefined; // Deprecated - use owner
  ui: {
    locale: string;
  };
  theme: {
    primaryColor: string;
    accentColor: string;
    semiTransparentAccentColor: string;
    lightColor: string;
    darkColor: string;
  };
  
  // Entity repositories
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  users: Partial<Record<string, User>>;
  ssoUsers: Partial<Record<string, SsoUser>>;
  
  // === Form-Specific Properties ===
  
  /** The currently active locale for the record */
  locale: string;
  /** If an already persisted record is being edited, returns the full record entity */
  item: Item | null;
  /** The model for the record being edited */
  itemType: ItemType;
  /** The complete internal form state */
  formValues: Record<string, unknown>;
  /** The current status of the record being edited */
  itemStatus: 'new' | 'draft' | 'updated' | 'published';
  /** Whether the form is currently submitting itself or not */
  isSubmitting: boolean;
  /** Whether the form has some non-persisted changes or not */
  isFormDirty: boolean;
  /** Provides information on how many blocks are currently present in the form */
  blocksAnalysis: {
    usage: {
      total: number;
      nonLocalized: number;
      perLocale: Record<string, number>;
    };
    maximumPerItem: number;
  };
  
  // Legacy properties that may be present for backward compatibility
  itemId?: string | null;
  fieldPath?: string;
  disabled?: boolean;
  
  // === Base Methods ===
  
  // Data loading
  loadItemTypeFields: (itemTypeId: string) => Promise<Field[]>;
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<Fieldset[]>;
  loadFieldsUsingPlugin: () => Promise<Field[]>;
  loadUsers: () => Promise<User[]>;
  loadSsoUsers: () => Promise<SsoUser[]>;
  
  // Plugin parameters
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  updateFieldAppearance: (
    fieldId: string,
    changes: Array<
      | { operation: 'removeEditor' }
      | { operation: 'updateEditor'; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
      | { operation: 'setEditor'; fieldExtensionId: string; parameters: Record<string, unknown> }
      | { operation: 'removeAddon'; index: number }
      | { operation: 'updateAddon'; index: number; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
      | { operation: 'insertAddon'; index: number; fieldExtensionId: string; parameters: Record<string, unknown> }
    >,
  ) => Promise<void>;
  
  // Toast notifications
  alert: (message: string) => Promise<void>;
  notice: (message: string) => Promise<void>;
  customToast: <CtaValue = unknown>(toast: {
    message: string;
    type: 'notice' | 'alert' | 'warning';
    cta?: {
      label: string;
      value: CtaValue;
    };
    dismissOnPageChange?: boolean;
    dismissAfterTimeout?: boolean | number;
  }) => Promise<CtaValue | null>;
  
  // Item dialogs
  createNewItem: (itemTypeId: string) => Promise<Item | null>;
  selectItem: {
    (itemTypeId: string, options: { multiple: true; initialLocationQuery?: ItemListLocationQuery }): Promise<Item[] | null>;
    (itemTypeId: string, options?: { multiple: false; initialLocationQuery?: ItemListLocationQuery }): Promise<Item | null>;
  };
  editItem: (itemId: string) => Promise<Item | null>;
  
  // Upload dialogs
  selectUpload: {
    (options: { multiple: true }): Promise<Upload[] | null>;
    (options?: { multiple: false }): Promise<Upload | null>;
  };
  editUpload: (uploadId: string) => Promise<(Upload & { deleted?: true }) | null>;
  editUploadMetadata: (
    fileFieldValue: {
      upload_id: string;
      alt: string | null;
      title: string | null;
      focal_point: { x: number; y: number } | null;
      custom_data: Record<string, string>;
    },
    locale?: string,
  ) => Promise<FileFieldValue | null>;
  
  // Custom dialogs
  openModal: (modal: {
    id: string;
    title?: string;
    closeDisabled?: boolean;
    width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
    parameters?: Record<string, unknown>;
    initialHeight?: number;
  }) => Promise<unknown>;
  openConfirm: (options: {
    title: string;
    content: string;
    choices: Array<{
      label: string;
      value: unknown;
      intent?: 'positive' | 'negative';
    }>;
    cancel: {
      label: string;
      value: unknown;
    };
  }) => Promise<unknown>;
  
  // Navigation
  navigateTo: (path: string) => Promise<void>;
  
  // === Form-Specific Methods ===
  
  /** Hides/shows a specific field in the form */
  toggleField: (path: string, show: boolean) => Promise<void>;
  /** Disables/re-enables a specific field in the form */
  disableField: (path: string, disable: boolean) => Promise<void>;
  /** Smoothly navigates to a specific field in the form */
  scrollToField: (path: string, locale?: string) => Promise<void>;
  /** Changes a specific path of the formValues object */
  setFieldValue: (path: string, value: unknown) => Promise<void>;
  /** Takes the internal form state and transforms it into an Item entity */
  formValuesToItem: (
    formValues: Record<string, unknown>,
    skipUnchangedFields?: boolean,
  ) => Promise<Omit<Item, 'id' | 'meta'> | undefined>;
  /** Takes an Item entity and converts it into the internal form state */
  itemToFormValues: (
    item: Omit<Item, 'id' | 'meta'>,
  ) => Promise<Record<string, unknown>>;
  /** Triggers a submit form for current record */
  saveCurrentItem: (showToast?: boolean) => Promise<void>;
};
```

### Return Types

```typescript
interface DropdownAction {
  /** Unique action identifier passed to executeItemFormDropdownAction */
  id: string;
  /** Display text shown to the user */
  label: string;
  /** Optional FontAwesome icon name or custom SVG */
  icon?: string;
  /** Whether this action is destructive (shows in red) */
  destructive?: boolean;
  /** Whether the action is disabled */
  disabled?: boolean;
  /** Whether to close the dropdown menu when clicked (default: true) */
  closeMenuOnClick?: boolean;
  /** Custom parameters passed to the action handler */
  parameters?: Record<string, unknown>;
  /** Display order priority (lower numbers appear first) */
  rank?: number;
}

interface DropdownActionGroup {
  /** Unique group identifier */
  id: string;
  /** Group header text */
  label: string;
  /** Actions within this group */
  actions: DropdownAction[];
  /** Group display order priority (lower numbers appear first) */
  rank?: number;
  /** Optional icon for the group header */
  icon?: string;
}
```

### Supporting Types

```typescript
// Location Query Types
interface ItemListLocationQuery {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
}

interface FileFieldValue {
  upload_id: string;
  alt: string | null;
  title: string | null;
  focal_point: { x: number; y: number } | null;
  custom_data: Record<string, string>;
}

// Plugin Type
interface Plugin {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description: string;
    homepage_url: string;
    icon: string;
    parameters: Record<string, unknown>;
    permissions: string[];
    version: string;
  };
}

// User Types
interface User {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    created_at: string;
    is_2fa_active: boolean;
    last_sign_in_at: string | null;
  };
}

interface SsoUser {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    created_at: string;
  };
}

interface Account {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    company: string | null;
  };
}

interface Organization {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
  };
}

// Role Type
interface Role {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_site: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_manage_build_triggers: boolean;
    can_manage_menu_items: boolean;
    can_edit_favicon: boolean;
    can_edit_seo_settings: boolean;
    can_manage_workflows: boolean;
    environments_access: 'primary_only' | 'sandbox_only' | 'all';
    positive_item_type_permissions: Array<{
      item_type_id: string;
      action: 'all' | 'create' | 'update' | 'delete' | 'publish' | 'edit_metadata';
    }>;
    negative_item_type_permissions: Array<{
      item_type_id: string;
      action: 'all' | 'create' | 'update' | 'delete' | 'publish' | 'edit_metadata';
    }>;
  };
}

// Site Type
interface Site {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    domain: string | null;
    global_seo: Record<string, any>;
    favicon: Upload | null;
    locales: string[];
    timezone: string;
  };
}

// ItemType Type
interface ItemType {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    modular_block: boolean;
    tree: boolean;
    ordering_direction: 'asc' | 'desc' | null;
    ordering_field: string | null;
    all_locales_required: boolean;
    collection_appearance: 'table' | 'gallery';
    hint?: string | null;
    inverse_relationships_enabled: boolean;
  };
  relationships: {
    fields: {
      data: Array<{
        id: string;
        type: 'field';
      }>;
    };
    fieldsets: {
      data: Array<{
        id: string;
        type: 'fieldset';
      }>;
    };
    singleton_item?: {
      data: {
        id: string;
        type: 'item';
      } | null;
    };
  };
  meta: {
    has_singleton_item: boolean;
  };
}

// Field Type
interface Field {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    field_type: string;
    api_key: string;
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
    default_value: any;
    hint?: string | null;
    placeholder?: string | null;
    appeareance?: Record<string, any>; // Legacy, use appearance
    position: number;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
    fieldset?: {
      data: {
        id: string;
        type: 'fieldset';
      } | null;
    };
  };
  meta: {
    valid_validators: Array<{
      type: string;
      config: Record<string, any>;
    }>;
  };
}

// Fieldset Type
interface Fieldset {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
    hint?: string | null;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
  };
}

// Item Type
interface Item {
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
    creator?: {
      data: {
        id: string;
        type: 'account' | 'user' | 'sso_user';
      } | null;
    };
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at: string | null;
    first_published_at: string | null;
    publication_scheduled_at: string | null;
    unpublishing_scheduled_at: string | null;
    status: 'draft' | 'updated' | 'published';
    is_current_version_valid: boolean;
    is_published_version_valid: boolean | null;
    current_version: string;
    stage: string | null;
  };
}

// Upload Type
interface Upload {
  id: string;
  type: 'upload';
  attributes: {
    path: string;
    format: string | null;
    size: number;
    width: number | null;
    height: number | null;
    alt: string | null;
    title: string | null;
    custom_data: Record<string, string>;
    focal_point: { x: number; y: number } | null;
    url: string;
    blurhash?: string;
    colors?: Array<{ hex: string; alpha: number }>;
    thumbhash?: string | null;
    basename: string;
    is_image: boolean;
    mime_type: string;
    default_field_metadata: {
      [locale: string]: {
        alt: string | null;
        title: string | null;
        custom_data: Record<string, string>;
        focal_point: { x: number; y: number } | null;
      };
    };
  };
  relationships: {
    creator?: {
      data: {
        id: string;
        type: 'account' | 'user' | 'sso_user';
      } | null;
    };
    upload_collection?: {
      data: {
        id: string;
        type: 'upload_collection';
      } | null;
    };
  };
  meta: {
    created_at: string;
    updated_at: string;
  };
}
```

## Use Cases

### 1. AI Content Enhancement

Add AI-powered content improvement actions:

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // Only for content-heavy models
  if (!['blog_post', 'article', 'page'].includes(itemType.attributes.api_key)) {
    return actions;
  }

  // AI actions group
  actions.push({
    id: 'ai-tools',
    label: 'AI Assistant',
    icon: 'robot',
    rank: 10,
    actions: [
      {
        id: 'improve-seo',
        label: 'Optimize for SEO',
        icon: 'search',
        disabled: !ctx.formValues.title || !ctx.formValues.content
      },
      {
        id: 'generate-summary',
        label: 'Generate Summary',
        icon: 'align-left',
        disabled: !ctx.formValues.content
      },
      {
        id: 'check-readability',
        label: 'Check Readability',
        icon: 'spell-check'
      },
      {
        id: 'translate-content',
        label: 'Translate to Other Locales',
        icon: 'language',
        disabled: ctx.site.attributes.locales.length === 1
      }
    ]
  });

  return actions;
}
```

### 2. Workflow Management

Add workflow-related actions:

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // Check if model has workflow
  const workflowField = Object.values(ctx.fields).find(
    f => f.attributes.api_key === 'workflow_stage' &&
         f.relationships.item_type.data.id === itemType.id
  );

  if (!workflowField) {
    return actions;
  }

  const currentStage = ctx.formValues.workflow_stage as string;

  // Workflow actions based on current stage
  const workflowActions: DropdownAction[] = [];

  if (currentStage === 'draft') {
    workflowActions.push({
      id: 'submit-for-review',
      label: 'Submit for Review',
      icon: 'paper-plane'
    });
  }

  if (currentStage === 'in_review') {
    workflowActions.push({
      id: 'approve-content',
      label: 'Approve',
      icon: 'check-circle',
      disabled: !ctx.currentRole.attributes.can_publish_to_production
    });
    
    workflowActions.push({
      id: 'request-changes',
      label: 'Request Changes',
      icon: 'edit',
      destructive: true
    });
  }

  if (workflowActions.length > 0) {
    actions.push({
      id: 'workflow',
      label: 'Workflow',
      icon: 'tasks',
      rank: 5,
      actions: workflowActions
    });
  }

  // Quick status changes
  actions.push({
    id: 'quick-status-change',
    label: 'Change Status to Draft',
    icon: 'file',
    rank: 20,
    disabled: currentStage === 'draft'
  });

  return actions;
}
```

### 3. External Integrations

Connect with external services:

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // E-commerce integration for products
  if (itemType.attributes.api_key === 'product' && ctx.item) {
    const syncActions: DropdownAction[] = [
      {
        id: 'sync-to-shopify',
        label: 'Sync to Shopify',
        icon: 'shopping-cart',
        disabled: !ctx.plugin.attributes.parameters.shopifyApiKey
      },
      {
        id: 'update-inventory',
        label: 'Update Inventory',
        icon: 'boxes'
      },
      {
        id: 'calculate-shipping',
        label: 'Calculate Shipping Rates',
        icon: 'truck'
      }
    ];

    actions.push({
      id: 'ecommerce',
      label: 'E-commerce',
      icon: 'store',
      rank: 15,
      actions: syncActions
    });
  }

  // Social media for blog posts
  if (itemType.attributes.api_key === 'blog_post' && ctx.item?.meta.published_at) {
    actions.push({
      id: 'social-media',
      label: 'Share on Social Media',
      icon: 'share-alt',
      rank: 25,
      actions: [
        {
          id: 'share-twitter',
          label: 'Share on Twitter',
          icon: 'twitter'
        },
        {
          id: 'share-linkedin',
          label: 'Share on LinkedIn',
          icon: 'linkedin'
        },
        {
          id: 'schedule-social',
          label: 'Schedule Social Posts',
          icon: 'calendar'
        }
      ]
    });
  }

  return actions;
}
```

### 4. Content Validation

Add validation and quality check actions:

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // Validation actions for any content
  const validationActions: DropdownAction[] = [];

  // Check for broken links
  if (ctx.formValues.content || ctx.formValues.body) {
    validationActions.push({
      id: 'check-links',
      label: 'Check for Broken Links',
      icon: 'link'
    });
  }

  // Image optimization
  const hasImageFields = Object.values(ctx.fields).some(
    f => f.attributes.field_type === 'file' &&
         f.relationships.item_type.data.id === itemType.id
  );

  if (hasImageFields) {
    validationActions.push({
      id: 'optimize-images',
      label: 'Optimize Images',
      icon: 'image'
    });
  }

  // SEO validation
  if (ctx.formValues.seo_title || ctx.formValues.meta_description) {
    validationActions.push({
      id: 'validate-seo',
      label: 'Validate SEO',
      icon: 'search'
    });
  }

  if (validationActions.length > 0) {
    actions.push({
      id: 'validation',
      label: 'Quality Checks',
      icon: 'clipboard-check',
      rank: 30,
      actions: validationActions
    });
  }

  return actions;
}
```

### 5. Version Control

Add version management actions:

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // Only for existing records
  if (!ctx.itemId) {
    return actions;
  }

  actions.push({
    id: 'versions',
    label: 'Version Control',
    icon: 'code-branch',
    rank: 40,
    actions: [
      {
        id: 'create-version',
        label: 'Create Version Snapshot',
        icon: 'camera'
      },
      {
        id: 'compare-versions',
        label: 'Compare with Previous Version',
        icon: 'code-compare'
      },
      {
        id: 'restore-version',
        label: 'Restore Previous Version',
        icon: 'undo',
        destructive: true,
        disabled: ctx.disabled
      },
      {
        id: 'view-history',
        label: 'View Full History',
        icon: 'history'
      }
    ]
  });

  return actions;
}
```

### 6. Conditional Actions

Show actions based on field values:

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // Product-specific actions based on type
  if (itemType.attributes.api_key === 'product') {
    const productType = ctx.formValues.product_type as string;

    if (productType === 'digital') {
      actions.push({
        id: 'generate-license',
        label: 'Generate License Keys',
        icon: 'key',
        rank: 10
      });
    }

    if (productType === 'physical') {
      actions.push({
        id: 'print-label',
        label: 'Print Shipping Label',
        icon: 'tag',
        rank: 10
      });
    }

    if (productType === 'subscription') {
      actions.push({
        id: 'manage-subscription',
        label: 'Manage Subscription Settings',
        icon: 'sync',
        rank: 10
      });
    }
  }

  // Event actions based on status
  if (itemType.attributes.api_key === 'event') {
    const eventDate = new Date(ctx.formValues.event_date as string);
    const now = new Date();

    if (eventDate > now) {
      actions.push({
        id: 'send-reminders',
        label: 'Send Reminder Emails',
        icon: 'envelope',
        rank: 15
      });
    } else {
      actions.push({
        id: 'generate-report',
        label: 'Generate Event Report',
        icon: 'chart-bar',
        rank: 15
      });
    }
  }

  return actions;
}
```

### 7. Developer Tools

Add development and debugging actions:

```typescript
itemFormDropdownActions(
  itemType: ItemType,
  ctx: ItemFormDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // Only show in non-production or for admins
  if (ctx.environment === 'main' && !ctx.currentRole.attributes.can_edit_schema) {
    return actions;
  }

  const devActions: DropdownAction[] = [
    {
      id: 'copy-json',
      label: 'Copy as JSON',
      icon: 'code'
    },
    {
      id: 'copy-api-call',
      label: 'Copy API Call',
      icon: 'terminal'
    },
    {
      id: 'validate-schema',
      label: 'Validate Against Schema',
      icon: 'check-double'
    }
  ];

  if (ctx.itemId) {
    devActions.push({
      id: 'view-api-response',
      label: 'View API Response',
      icon: 'eye'
    });
  }

  actions.push({
    id: 'developer',
    label: 'Developer Tools',
    icon: 'wrench',
    rank: 100,
    actions: devActions
  });

  return actions;
}
```

## Best Practices

1. **Unique IDs**: Ensure all action IDs are unique:
   ```typescript
   {
     id: `${itemType.attributes.api_key}-${actionType}`,
     // Prefix with context for uniqueness
   }
   ```

2. **Clear Labels**: Use descriptive, action-oriented labels:
   ```typescript
   // Good
   label: 'Export to PDF'
   
   // Avoid
   label: 'PDF'
   ```

3. **Appropriate Icons**: Choose relevant FontAwesome icons:
   ```typescript
   'save' for save actions
   'share' for sharing
   'trash' for delete (with destructive: true)
   'cog' for settings
   ```

4. **Disable When Appropriate**: Disable actions that can't be performed:
   ```typescript
   disabled: !ctx.itemId || // No ID for new records
            !ctx.formValues.required_field || // Missing required data
            ctx.disabled // Form is read-only
   ```

5. **Group Related Actions**: Use groups for better organization:
   ```typescript
   // Group by functionality
   { id: 'workflow', label: 'Workflow', actions: [...] }
   { id: 'sharing', label: 'Sharing', actions: [...] }
   { id: 'tools', label: 'Tools', actions: [...] }
   ```

## Related Hooks

- **executeItemFormDropdownAction**: Handles the execution of these actions
- **itemFormOutlets**: Add custom UI sections to forms
- **fieldDropdownActions**: Add actions specific to fields

## Important Notes

- Actions appear in the form's dropdown menu (usually in the top toolbar)
- The `executeItemFormDropdownAction` hook must handle the action execution
- Actions can be disabled but still visible to indicate unavailable functionality
- Destructive actions are shown in red as a warning
- Groups help organize many actions into logical sections
- The rank determines display order (lower numbers appear first)

