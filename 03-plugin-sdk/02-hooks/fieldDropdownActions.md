# fieldDropdownActions Hook

## Purpose

The `fieldDropdownActions` hook allows plugins to add custom actions to the dropdown menu of individual fields in the record editing form. These actions appear in the field's context menu (three dots icon) and can perform various operations on the field's value. The actual execution of these actions is handled by the `executeFieldDropdownAction` hook.

## Signature

```typescript
fieldDropdownActions(
  field: Field,
  ctx: FieldDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup>
```

## Parameters

- `field`: Field - The field for which to provide dropdown actions
- `ctx`: FieldDropdownActionsCtx - Context object with form data and utilities

## Type Definitions

### Complete Hook Signature

```typescript
export type FieldDropdownActionsHook = {
  fieldDropdownActions: (
    field: Field,
    ctx: FieldDropdownActionsCtx,
  ) => Array<DropdownAction | DropdownActionGroup>;
};
```

### Field Type

```typescript
interface Field {
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
    validators: {
      required?: boolean;
      unique?: boolean;
      format?: string;
      pattern?: { pattern: string; flags?: string };
      length?: { min?: number; max?: number };
      enum?: { values: string[] };
      number_range?: { min?: number; max?: number };
      items_item_type?: { item_types: string[] };
      rich_text_blocks?: { item_types: string[] };
      slug_format?: { predefined?: string; custom?: string };
      slug_title_field?: { title_field_id: string };
      date_range?: { min?: string; max?: string };
      datetime_range?: { min?: string; max?: string };
      file_size?: { min?: number; max?: number };
      image_dimensions?: { min_width?: number; max_width?: number; min_height?: number; max_height?: number };
      number_of_items?: { min?: number; max?: number };
    };
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
      addons?: Array<{
        id: string;
        parameters: Record<string, any>;
      }>;
    };
    hint?: string | null;
    placeholder?: string | null;
    appeareance?: Record<string, any>; // Legacy, use appearance
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fieldset?: { data: { id: string; type: 'fieldset' } | null };
  };
  meta: {
    valid_validators: Array<{
      type: string;
      config: Record<string, any>;
    }>;
  };
}
```

### Context Type Definition

```typescript
export type FieldDropdownActionsCtx = {
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

  // === Field-Specific Properties ===
  
  /** Whether the field is currently disabled */
  disabled: boolean;
  /** The path to this field in the form values object */
  fieldPath: string;
  /** The field being acted upon (also passed as first parameter) */
  field: Field;
  /** The parent field if this field is nested inside a modular content/block */
  parentField?: Field;
  /** Information about the block if this field is inside a modular content block */
  block?: {
    id: string | undefined;
    blockModel: ItemType;
  };

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
  id: string;                  // Unique identifier for the action
  label: string;               // Display text
  icon?: Icon;                 // FontAwesome icon name or custom SVG
  active?: boolean;            // Show as active/selected
  alert?: boolean;             // Show in alert style
  disabled?: boolean;          // Disable the action
  closeMenuOnClick?: boolean;  // Close dropdown after click
  parameters?: Record<string, unknown>; // Pass data to execute hook
  rank?: number;               // Display order (ascending)
}

interface DropdownActionGroup {
  label: string;               // Group label
  icon?: Icon;                 // Group icon
  actions: DropdownAction[];   // Actions in the group
  rank?: number;               // Display order (ascending)
}

// Icon can be a FontAwesome identifier or custom SVG
type Icon = string | {
  type: 'svg';
  viewBox: string;
  content: string;
};
```

### Supporting Types

```typescript
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

// Other Types
type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};

type FileFieldValue = {
  upload_id: string;
  alt: string | null;
  title: string | null;
  focal_point: { x: number; y: number } | null;
  custom_data: Record<string, string>;
};
```

## Use Cases

### 1. Text Field Actions

Common text manipulation actions:

```typescript
fieldDropdownActions(
  field: Field,
  ctx: FieldDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  // Only for text fields
  if (!['string', 'text'].includes(field.attributes.field_type)) {
    return [];
  }

  const actions: DropdownAction[] = [];
  const currentValue = ctx.formValues[ctx.fieldPath] as string || '';

  // Text transformation group
  const transformActions: DropdownActionGroup = {
    label: 'Transform Text',
    actions: [
      {
        id: 'to-uppercase',
        label: 'Convert to UPPERCASE',
        icon: 'font',
        disabled: !currentValue
      },
      {
        id: 'to-lowercase',
        label: 'Convert to lowercase',
        icon: 'font',
        disabled: !currentValue
      },
      {
        id: 'to-title-case',
        label: 'Convert to Title Case',
        icon: 'heading',
        disabled: !currentValue
      },
      {
        id: 'slugify',
        label: 'Convert to slug',
        icon: 'link',
        disabled: !currentValue
      }
    ]
  };

  // Analysis actions
  const analysisActions: DropdownActionGroup = {
    label: 'Analyze',
    actions: [
      {
        id: 'count-words',
        label: 'Count Words',
        icon: 'calculator',
        disabled: !currentValue
      },
      {
        id: 'check-readability',
        label: 'Check Readability',
        icon: 'glasses',
        disabled: !currentValue || currentValue.length < 100
      }
    ]
  };

  // Cleanup actions
  const cleanupActions: DropdownActionGroup = {
    label: 'Clean Up',
    actions: [
      {
        id: 'remove-html',
        label: 'Remove HTML Tags',
        icon: 'code',
        disabled: !currentValue || !/<[^>]*>/.test(currentValue)
      },
      {
        id: 'trim-whitespace',
        label: 'Trim Whitespace',
        icon: 'eraser',
        disabled: !currentValue || currentValue.trim() === currentValue
      },
      {
        id: 'remove-duplicates',
        label: 'Remove Duplicate Lines',
        icon: 'copy',
        disabled: !currentValue || !currentValue.includes('\n')
      }
    ]
  };

  return [transformActions, analysisActions, cleanupActions];
}
```

### 2. SEO Field Actions

Actions specific to SEO-related fields:

```typescript
fieldDropdownActions(
  field: Field,
  ctx: FieldDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];

  // SEO title actions
  if (field.attributes.api_key === 'seo_title') {
    actions.push({
      id: 'generate-from-title',
      label: 'Generate from Title',
      icon: 'magic',
      disabled: !ctx.formValues.title
    });

    actions.push({
      id: 'check-length',
      label: 'Check SEO Length',
      icon: 'ruler',
      disabled: !ctx.formValues[ctx.fieldPath]
    });

    actions.push({
      id: 'preview-google',
      label: 'Preview in Google',
      icon: 'google',
      disabled: !ctx.formValues[ctx.fieldPath]
    });
  }

  // Meta description actions
  if (field.attributes.api_key === 'meta_description') {
    actions.push({
      id: 'generate-from-content',
      label: 'Generate from Content',
      icon: 'magic',
      disabled: !ctx.formValues.content
    });

    actions.push({
      id: 'optimize-keywords',
      label: 'Optimize for Keywords',
      icon: 'search',
      parameters: {
        keywords: ctx.formValues.seo_keywords
      }
    });
  }

  // URL/Slug actions
  if (field.attributes.api_key === 'slug') {
    actions.push({
      id: 'check-uniqueness',
      label: 'Check URL Uniqueness',
      icon: 'fingerprint'
    });

    actions.push({
      id: 'redirect-old-urls',
      label: 'Setup Redirects',
      icon: 'directions',
      disabled: ctx.itemStatus === 'new'
    });
  }

  return actions;
}
```

### 3. Media Field Actions

Actions for image and file fields:

```typescript
fieldDropdownActions(
  field: Field,
  ctx: FieldDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  if (!['file', 'gallery'].includes(field.attributes.field_type)) {
    return [];
  }

  const hasValue = !!ctx.formValues[ctx.fieldPath];
  const isGallery = field.attributes.field_type === 'gallery';

  const imageActions: DropdownActionGroup = {
    label: 'Image Operations',
    actions: [
      {
        id: 'optimize-image',
        label: 'Optimize Image',
        icon: 'compress',
        disabled: !hasValue,
        parameters: {
          quality: 85,
          format: 'auto'
        }
      },
      {
        id: 'generate-alt-text',
        label: 'Generate Alt Text (AI)',
        icon: 'robot',
        disabled: !hasValue
      },
      {
        id: 'crop-focal-point',
        label: 'Set Focal Point',
        icon: 'crosshairs',
        disabled: !hasValue
      },
      {
        id: 'extract-colors',
        label: 'Extract Color Palette',
        icon: 'palette',
        disabled: !hasValue
      }
    ]
  };

  const galleryActions: DropdownActionGroup = {
    label: 'Gallery Operations',
    actions: [
      {
        id: 'bulk-upload',
        label: 'Bulk Upload',
        icon: 'cloud-upload-alt'
      },
      {
        id: 'sort-by-name',
        label: 'Sort by Name',
        icon: 'sort-alpha-down',
        disabled: !hasValue
      },
      {
        id: 'sort-by-date',
        label: 'Sort by Date',
        icon: 'sort-numeric-down',
        disabled: !hasValue
      },
      {
        id: 'generate-thumbnails',
        label: 'Regenerate Thumbnails',
        icon: 'images',
        disabled: !hasValue
      }
    ]
  };

  const actions = [imageActions];
  if (isGallery) {
    actions.push(galleryActions);
  }

  return actions;
}
```

### 4. Localization Actions

Actions for multi-locale fields:

```typescript
fieldDropdownActions(
  field: Field,
  ctx: FieldDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  // Only for localized fields
  if (!field.attributes.localized) {
    return [];
  }

  const currentValue = ctx.formValues[ctx.fieldPath];
  const hasValue = !!currentValue;
  const otherLocales = ctx.site.attributes.locales.filter(l => l !== ctx.locale);

  const copyActions = otherLocales.map(locale => ({
    id: `copy-to-${locale}`,
    label: `Copy to ${locale.toUpperCase()}`,
    icon: 'copy',
    disabled: !hasValue,
    parameters: { targetLocale: locale }
  }));

  const translateActions = otherLocales.map(locale => ({
    id: `translate-to-${locale}`,
    label: `Translate to ${locale.toUpperCase()} (AI)`,
    icon: 'language',
    disabled: !hasValue || !ctx.plugin.attributes.parameters.translationApiKey,
    parameters: { targetLocale: locale }
  }));

  return [
    {
      label: 'Copy to Other Locales',
      actions: copyActions
    },
    {
      label: 'Translate',
      actions: translateActions
    },
    {
      id: 'copy-to-all-locales',
      label: 'Copy to All Locales',
      icon: 'globe',
      disabled: !hasValue,
      destructive: true
    },
    {
      id: 'translate-all',
      label: 'Translate to All Locales (AI)',
      icon: 'magic',
      disabled: !hasValue || !ctx.plugin.attributes.parameters.translationApiKey
    }
  ];
}
```

### 5. Conditional Actions

Show actions based on field state and configuration:

```typescript
fieldDropdownActions(
  field: Field,
  ctx: FieldDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const actions: Array<DropdownAction | DropdownActionGroup> = [];
  const value = ctx.formValues[ctx.fieldPath];

  // Validation actions
  if (field.attributes.validators.required && !value) {
    actions.push({
      id: 'fill-required',
      label: 'Fill with Default Value',
      icon: 'fill',
      parameters: {
        defaultValue: field.attributes.default_value
      }
    });
  }

  // Format validation actions
  if (field.attributes.validators.format) {
    const format = field.attributes.validators.format;
    
    if (format === 'email') {
      actions.push({
        id: 'validate-email',
        label: 'Validate Email',
        icon: 'envelope',
        disabled: !value
      });
    }
    
    if (format === 'url') {
      actions.push({
        id: 'check-url',
        label: 'Check URL Status',
        icon: 'link',
        disabled: !value
      });
      
      actions.push({
        id: 'shorten-url',
        label: 'Shorten URL',
        icon: 'compress',
        disabled: !value
      });
    }
  }

  // JSON field actions
  if (field.attributes.field_type === 'json') {
    actions.push({
      label: 'JSON Operations',
      actions: [
        {
          id: 'validate-json',
          label: 'Validate & Format',
          icon: 'check-circle',
          disabled: !value
        },
        {
          id: 'minify-json',
          label: 'Minify',
          icon: 'compress',
          disabled: !value
        },
        {
          id: 'json-to-yaml',
          label: 'Convert to YAML',
          icon: 'exchange-alt',
          disabled: !value
        }
      ]
    });
  }

  // Rich text actions
  if (field.attributes.field_type === 'text' && field.attributes.appearance.editor === 'markdown') {
    actions.push({
      label: 'Markdown Tools',
      actions: [
        {
          id: 'preview-markdown',
          label: 'Preview',
          icon: 'eye',
          disabled: !value
        },
        {
          id: 'convert-to-html',
          label: 'Convert to HTML',
          icon: 'code',
          disabled: !value
        },
        {
          id: 'add-toc',
          label: 'Add Table of Contents',
          icon: 'list',
          disabled: !value || !(value as string).includes('#')
        }
      ]
    });
  }

  return actions;
}
```

### 6. Field-Specific Development Tools

Actions for development and debugging:

```typescript
fieldDropdownActions(
  field: Field,
  ctx: FieldDropdownActionsCtx
): Array<DropdownAction | DropdownActionGroup> {
  const devActions: DropdownAction[] = [];

  // Only show in development environment
  if (ctx.environment !== 'main') {
    devActions.push({
      id: 'inspect-field',
      label: 'Inspect Field Config',
      icon: 'info-circle',
      closeMenuOnClick: false
    });

    devActions.push({
      id: 'copy-field-path',
      label: 'Copy Field Path',
      icon: 'clipboard',
      parameters: {
        path: ctx.fieldPath
      }
    });

    devActions.push({
      id: 'log-field-value',
      label: 'Log Value to Console',
      icon: 'terminal'
    });

    if (ctx.formValues[ctx.fieldPath]) {
      devActions.push({
        id: 'generate-test-data',
        label: 'Generate Test Data',
        icon: 'dice',
        parameters: {
          fieldType: field.attributes.field_type
        }
      });
    }
  }

  // Admin-only actions
  if (ctx.currentRole.attributes.admin) {
    devActions.push({
      id: 'export-field-data',
      label: 'Export Field Data',
      icon: 'download'
    });
  }

  return devActions.length > 0 ? [{
    label: 'Developer Tools',
    actions: devActions
  }] : [];
}
```

## Best Practices

1. **Conditional Display**: Only show relevant actions based on field type and state
2. **Disable When Appropriate**: Use the `disabled` property to prevent invalid operations
3. **Group Related Actions**: Use `DropdownActionGroup` to organize many actions
4. **Clear Labels**: Use descriptive labels that explain what the action does
5. **Icons**: Choose appropriate FontAwesome icons for visual clarity
6. **Destructive Actions**: Mark dangerous operations with `destructive: true`
7. **Parameters**: Pass necessary data via `parameters` to the execute hook

## Related Hooks

- **executeFieldDropdownAction**: Handles the execution of these actions
- **itemFormDropdownActions**: Similar hook for form-level actions
- **itemsDropdownActions**: Hook for collection-level actions

## Important Notes

- Action IDs must be unique within your plugin
- The same action IDs must be handled in `executeFieldDropdownAction`
- Actions are displayed in the order they are returned
- Empty groups are automatically hidden
- The dropdown is only shown if there are actions to display
- Consider field permissions when showing actions

## Type Definitions

```typescript
// Main hook type
export type FieldDropdownActionsHook = {
  fieldDropdownActions: (
    field: Field,
    ctx: FieldDropdownActionsCtx,
  ) => Array<DropdownAction | DropdownActionGroup>;
};

// Supporting types referenced in examples

// Plugin type
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description?: string;
    homepage?: string;
    package_name?: string;
    package_version?: string;
    parameters: Record<string, unknown>;
  };
};

// User type
type User = {
  id: string;
  type: 'user';
  attributes: {
    first_name: string;
    last_name: string;
    email: string;
    locale: string;
    is_2fa_active: boolean;
  };
};

// Account type
type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    company?: string;
    profile_type: string;
  };
};

// Organization type
type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
    is_enterprise: boolean;
  };
};

// SsoUser type
type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    first_name: string;
    last_name: string;
    email: string;
  };
};

// Site type
type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    primary_environment: string;
    locales: string[];
    timezone: string;
  };
};

// Role type
type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    positive_item_type_permissions: Array<{
      item_type: string;
      action: string;
    }>;
    negative_item_type_permissions: Array<{
      item_type: string;
      action: string;
    }>;
    positive_field_permissions: Array<{
      field: string;
      action: string;
    }>;
    negative_field_permissions: Array<{
      field: string;
      action: string;
    }>;
    admin: boolean;
  };
  meta: {
    final_permissions: {
      can_edit_schema: boolean;
      can_edit_site: boolean;
      can_edit_favicon: boolean;
      can_manage_environments: boolean;
      can_manage_build_triggers: boolean;
      can_manage_sso: boolean;
      can_manage_users: boolean;
      can_manage_webhooks: boolean;
      can_manage_access_tokens: boolean;
      can_perform_site_search: boolean;
      can_dump_data: boolean;
      can_import_and_export: boolean;
      can_manage_site_notifications: boolean;
      environments_access: 'all' | 'main' | 'sandbox' | Array<string>;
    };
  };
};

// Item type
type Item = {
  id: string;
  type: 'item';
  attributes: Record<string, unknown>;
  relationships: Record<string, any>;
  meta: {
    created_at: string;
    updated_at: string;
    published_at?: string;
    publication_scheduled_at?: string;
    unpublishing_scheduled_at?: string;
    first_published_at?: string;
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    current_version?: string;
    stage?: string;
  };
};

// ItemType type
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
    modular_block: boolean;
    ordering_direction?: 'asc' | 'desc';
    ordering_meta?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'table';
    has_singleton_item: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: 'field' }> };
    fieldsets: { data: Array<{ id: string; type: 'fieldset' }> };
    singleton_item?: { data: { id: string; type: 'item' } | null };
    ordering_field?: { data: { id: string; type: 'field' } | null };
    title_field?: { data: { id: string; type: 'field' } | null };
    image_preview_field?: { data: { id: string; type: 'field' } | null };
    excerpt_field?: { data: { id: string; type: 'field' } | null };
    modular_block_blocks?: { data: Array<{ id: string; type: 'item_type' }> };
    workflow?: { data: { id: string; type: 'workflow' } | null };
  };
};

// Fieldset type
type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
    hint?: string;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
  };
};

// Upload type
type Upload = {
  id: string;
  type: 'upload';
  attributes: {
    size: number;
    width?: number;
    height?: number;
    path: string;
    basename: string;
    filename: string;
    url: string;
    format?: string;
    author?: string;
    copyright?: string;
    notes?: string;
    default_field_metadata: {
      [locale: string]: {
        alt: string | null;
        title: string | null;
        custom_data: Record<string, string>;
        focal_point: { x: number; y: number } | null;
      };
    };
    is_image: boolean;
    created_at: string;
    updated_at: string;
    mime_type: string;
    tags: string[];
    smart_tags: string[];
    colors: Array<{ red: number; green: number; blue: number; alpha: number }>;
    blurhash?: string;
  };
};

// FileFieldValue type
type FileFieldValue = {
  upload_id: string;
  alt: string | null;
  title: string | null;
  focal_point: { x: number; y: number } | null;
  custom_data: Record<string, string>;
};

// Toast type
type Toast<CtaValue = unknown> = {
  message: string;
  type: 'notice' | 'alert' | 'warning';
  cta?: {
    label: string;
    value: CtaValue;
  };
  dismissOnPageChange?: boolean;
  dismissAfterTimeout?: boolean | number;
};

// Modal type
type Modal = {
  id: string;
  title?: string;
  closeDisabled?: boolean;
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  parameters?: Record<string, unknown>;
  initialHeight?: number;
};

// ConfirmOptions type
type ConfirmOptions = {
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
};

// FieldAppearanceChange type
type FieldAppearanceChange =
  | { operation: 'removeEditor' }
  | { operation: 'updateEditor'; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'setEditor'; fieldExtensionId: string; parameters: Record<string, unknown> }
  | { operation: 'removeAddon'; index: number }
  | { operation: 'updateAddon'; index: number; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'insertAddon'; index: number; fieldExtensionId: string; parameters: Record<string, unknown> };

// ItemListLocationQuery type
type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};
```