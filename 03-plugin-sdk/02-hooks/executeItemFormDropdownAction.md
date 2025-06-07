# executeItemFormDropdownAction Hook

## Purpose

The `executeItemFormDropdownAction` hook executes custom actions defined by the `itemFormDropdownActions` hook when users click on them in the record form dropdown menu. This hook handles the actual logic and operations for each custom action, such as data validation, API calls, record modifications, or triggering external processes.

## Signature

```typescript
executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
): Promise<void>
```

## Parameters

- `actionId`: string - The unique identifier of the action being executed

## Complete Type Definitions

```typescript
// Main hook type
export type ExecuteItemFormDropdownActionHook = {
  executeItemFormDropdownAction: (
    actionId: string,
    ctx: ExecuteItemFormDropdownActionCtx,
  ) => Promise<void>;
};

// Full context type with all properties and methods
export type ExecuteItemFormDropdownActionCtx = {
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
  
  /** Parameters passed from the action definition */
  parameters: Record<string, unknown> | undefined;
  
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

// Supporting types
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

// Plugin type
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

// User types
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

// Role type
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

// Site type
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

// ItemType type
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

// Field type
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

// Fieldset type
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

// Item type
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

// Upload type
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

### 1. AI Content Generation

Generate content using AI and populate form fields:

```typescript
async executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
) {
  if (actionId === 'ai-generate-content') {
    const { item, itemType, formValues, locale } = ctx;

    // Check if AI is configured
    const aiKey = ctx.plugin.attributes.parameters.openaiApiKey;
    if (!aiKey) {
      ctx.alert('AI is not configured. Please add your OpenAI API key in plugin settings.');
      return;
    }

    // Get context from existing fields
    const title = formValues.title || '';
    const keywords = formValues.seo_keywords || [];
    
    if (!title) {
      ctx.alert('Please add a title first to generate content');
      return;
    }

    // Show progress
    ctx.notice('Generating content with AI...');

    try {
      // Generate content based on title and keywords
      const generatedContent = await generateAIContent({
        title,
        keywords,
        locale,
        model: itemType.attributes.api_key,
        apiKey: aiKey
      });

      // Preview before applying
      const confirmed = await ctx.openConfirm({
        title: 'AI Generated Content',
        content: (
          <div>
            <p>Review the generated content:</p>
            <div style={{ 
              maxHeight: '300px', 
              overflow: 'auto',
              padding: '10px',
              background: '#f5f5f5',
              borderRadius: '4px',
              marginTop: '10px'
            }}>
              <div dangerouslySetInnerHTML={{ __html: generatedContent.html }} />
            </div>
          </div>
        ),
        choices: [
          { label: 'Cancel', value: false },
          { label: 'Use This Content', value: true, intent: 'positive' }
        ]
      });

      if (confirmed) {
        // Update multiple fields
        await ctx.setFieldValue('content', generatedContent.html);
        
        if (generatedContent.excerpt) {
          await ctx.setFieldValue('excerpt', generatedContent.excerpt);
        }
        
        if (generatedContent.metaDescription) {
          await ctx.setFieldValue('meta_description', generatedContent.metaDescription);
        }

        ctx.notice('Content generated and applied successfully!');
      }
    } catch (error) {
      ctx.alert(`Failed to generate content: ${error.message}`);
    }
  }
}
```

### 2. SEO Analysis and Optimization

Analyze and optimize SEO fields:

```typescript
async executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
) {
  if (actionId === 'analyze-seo') {
    const { formValues, locale } = ctx;

    // Gather content for analysis
    const content = {
      title: formValues.title || '',
      content: formValues.content || '',
      seoTitle: formValues.seo_title || '',
      metaDescription: formValues.meta_description || '',
      focusKeyword: formValues.focus_keyword || ''
    };

    // Validate minimum content
    if (!content.title || !content.content) {
      ctx.alert('Please add title and content before analyzing SEO');
      return;
    }

    // Show analysis modal
    const result = await ctx.openModal({
      id: 'seo-analysis',
      title: 'SEO Analysis',
      width: 'l',
      parameters: { content, locale }
    });

    if (result && result.suggestions) {
      // Apply suggestions
      for (const suggestion of result.suggestions) {
        if (suggestion.apply) {
          await ctx.setFieldValue(suggestion.field, suggestion.value);
        }
      }

      ctx.notice('SEO optimizations applied');
    }
  }

  if (actionId === 'optimize-meta') {
    const title = ctx.formValues.title || '';
    const content = ctx.formValues.content || '';

    if (!title || !content) {
      ctx.alert('Title and content are required for optimization');
      return;
    }

    try {
      // Generate optimized meta
      const optimized = await generateOptimizedMeta({ title, content });

      // Show preview
      await ctx.openModal({
        id: 'meta-preview',
        title: 'Optimized Meta Tags',
        parameters: {
          current: {
            seoTitle: ctx.formValues.seo_title,
            metaDescription: ctx.formValues.meta_description
          },
          optimized,
          onApply: async (fields) => {
            if (fields.seoTitle) {
              await ctx.setFieldValue('seo_title', optimized.seoTitle);
            }
            if (fields.metaDescription) {
              await ctx.setFieldValue('meta_description', optimized.metaDescription);
            }
          }
        }
      });
    } catch (error) {
      ctx.alert('Failed to optimize meta tags');
    }
  }
}
```

### 3. Data Import/Export

Import or export record data:

```typescript
async executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
) {
  if (actionId === 'export-json') {
    const { item, formValues, itemType } = ctx;

    // Prepare export data
    const exportData = {
      model: itemType.attributes.api_key,
      locale: ctx.locale,
      exported_at: new Date().toISOString(),
      data: formValues,
      meta: item ? {
        id: item.id,
        created_at: item.meta.created_at,
        updated_at: item.meta.updated_at,
        published_at: item.meta.published_at,
        status: item.meta.status
      } : null
    };

    // Download as JSON
    const blob = new Blob([JSON.stringify(exportData, null, 2)], {
      type: 'application/json'
    });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${itemType.attributes.api_key}-${item?.id || 'new'}-${Date.now()}.json`;
    a.click();
    URL.revokeObjectURL(url);

    ctx.notice('Data exported successfully');
  }

  if (actionId === 'import-json') {
    // Show file picker modal
    const file = await ctx.openModal({
      id: 'file-picker',
      title: 'Import JSON Data',
      width: 's'
    });

    if (!file) return;

    try {
      const text = await file.text();
      const importData = JSON.parse(text);

      // Validate import
      if (importData.model !== ctx.itemType.attributes.api_key) {
        ctx.alert(`Invalid import: Expected model '${ctx.itemType.attributes.api_key}', got '${importData.model}'`);
        return;
      }

      // Confirm import
      const confirmed = await ctx.openConfirm({
        title: 'Import Data',
        content: `This will overwrite current form values with data from ${importData.exported_at}. Continue?`,
        choices: [
          { label: 'Cancel', value: false },
          { label: 'Import', value: true, intent: 'negative' }
        ]
      });

      if (confirmed) {
        // Apply imported data
        for (const [field, value] of Object.entries(importData.data)) {
          if (ctx.fields[field]) { // Only import valid fields
            await ctx.setFieldValue(field, value);
          }
        }

        ctx.notice('Data imported successfully');
      }
    } catch (error) {
      ctx.alert(`Import failed: ${error.message}`);
    }
  }
}
```

### 4. Field Validation and Cleanup

Validate and clean up form data:

```typescript
async executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
) {
  if (actionId === 'validate-all-fields') {
    const issues = [];
    let fixedCount = 0;

    // Check each field
    for (const [fieldPath, value] of Object.entries(ctx.formValues)) {
      const field = ctx.fields[fieldPath];
      if (!field) continue;

      // Check for common issues
      if (field.attributes.field_type === 'string' && typeof value === 'string') {
        // Check for extra whitespace
        const trimmed = value.trim();
        if (trimmed !== value) {
          await ctx.setFieldValue(fieldPath, trimmed);
          fixedCount++;
        }

        // Check for required fields
        if (field.attributes.validators?.required && !trimmed) {
          issues.push({
            field: field.attributes.label,
            issue: 'Required field is empty'
          });
        }

        // Check length constraints
        const length = field.attributes.validators?.length;
        if (length) {
          if (length.min && trimmed.length < length.min) {
            issues.push({
              field: field.attributes.label,
              issue: `Too short (min ${length.min} chars)`
            });
          }
          if (length.max && trimmed.length > length.max) {
            issues.push({
              field: field.attributes.label,
              issue: `Too long (max ${length.max} chars)`
            });
          }
        }
      }

      // Check URLs
      if (field.attributes.validators?.format === 'url' && value) {
        try {
          new URL(value);
        } catch {
          issues.push({
            field: field.attributes.label,
            issue: 'Invalid URL format'
          });
        }
      }

      // Check emails
      if (field.attributes.validators?.format === 'email' && value) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(value)) {
          issues.push({
            field: field.attributes.label,
            issue: 'Invalid email format'
          });
        }
      }
    }

    // Show results
    if (issues.length === 0 && fixedCount === 0) {
      ctx.notice('✅ All fields are valid!');
    } else {
      const message = [];
      if (fixedCount > 0) {
        message.push(`Fixed ${fixedCount} field(s) with whitespace issues`);
      }
      if (issues.length > 0) {
        message.push(`Found ${issues.length} validation issue(s)`);
      }

      await ctx.openModal({
        id: 'validation-results',
        title: 'Validation Results',
        width: 'm',
        parameters: { issues, fixedCount }
      });
    }
  }

  if (actionId === 'clean-html') {
    let cleanedCount = 0;

    for (const [fieldPath, value] of Object.entries(ctx.formValues)) {
      const field = ctx.fields[fieldPath];
      
      if (field?.attributes.field_type === 'text' && 
          field.attributes.appearance?.editor === 'wysiwyg' && 
          value) {
        
        const cleaned = await cleanHTML(value, {
          allowedTags: ['p', 'h2', 'h3', 'h4', 'strong', 'em', 'a', 'ul', 'ol', 'li'],
          allowedAttributes: {
            'a': ['href', 'target', 'rel']
          }
        });

        if (cleaned !== value) {
          await ctx.setFieldValue(fieldPath, cleaned);
          cleanedCount++;
        }
      }
    }

    ctx.notice(`Cleaned ${cleanedCount} HTML field(s)`);
  }
}
```

### 5. Template Application

Apply predefined templates to the record:

```typescript
async executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
) {
  if (actionId === 'apply-template') {
    const { itemType, locale } = ctx;

    // Load available templates
    const templates = await loadTemplatesForModel(itemType.attributes.api_key);
    
    if (templates.length === 0) {
      ctx.alert('No templates available for this content type');
      return;
    }

    // Let user select template
    const selectedTemplate = await ctx.openModal({
      id: 'template-selector',
      title: 'Select Template',
      width: 'l',
      parameters: { 
        templates,
        currentValues: ctx.formValues,
        locale
      }
    });

    if (!selectedTemplate) return;

    // Confirm before applying
    const confirmed = await ctx.openConfirm({
      title: 'Apply Template',
      content: `Apply "${selectedTemplate.name}" template? This will overwrite current values.`,
      choices: [
        { label: 'Cancel', value: false },
        { label: 'Apply Template', value: true, intent: 'negative' }
      ]
    });

    if (confirmed) {
      // Apply template values
      for (const [field, value] of Object.entries(selectedTemplate.values)) {
        // Handle locale-specific values
        if (typeof value === 'object' && value[locale]) {
          await ctx.setFieldValue(field, value[locale]);
        } else {
          await ctx.setFieldValue(field, value);
        }
      }

      ctx.notice(`Template "${selectedTemplate.name}" applied successfully`);
    }
  }

  if (actionId === 'save-as-template') {
    const templateName = await ctx.openModal({
      id: 'template-name-input',
      title: 'Save as Template',
      width: 's'
    });

    if (!templateName) return;

    try {
      await saveTemplate({
        name: templateName,
        model: ctx.itemType.attributes.api_key,
        values: ctx.formValues,
        locale: ctx.locale
      });

      ctx.notice('Template saved successfully');
    } catch (error) {
      ctx.alert(`Failed to save template: ${error.message}`);
    }
  }
}
```

### 6. External Service Integration

Integrate with external services:

```typescript
async executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
) {
  if (actionId === 'sync-to-crm') {
    const { item, formValues, itemType } = ctx;

    // Check CRM configuration
    const crmConfig = ctx.plugin.attributes.parameters.crm;
    if (!crmConfig?.apiKey) {
      ctx.alert('CRM integration not configured');
      return;
    }

    // Map fields to CRM format
    const crmData = mapToCRMFormat(formValues, itemType.attributes.api_key);

    try {
      ctx.notice('Syncing to CRM...');

      const result = await syncToCRM({
        data: crmData,
        recordId: item?.id,
        config: crmConfig
      });

      // Store CRM ID
      if (result.crmId && item) {
        await ctx.setFieldValue('crm_id', result.crmId);
        await ctx.saveCurrentItem(false);
      }

      ctx.notice(`✅ Synced to CRM (ID: ${result.crmId})`);
    } catch (error) {
      ctx.alert(`CRM sync failed: ${error.message}`);
    }
  }

  if (actionId === 'translate-content') {
    const { locale, itemType } = ctx;

    // Get translatable fields
    const translatableFields = Object.values(ctx.fields).filter(
      field => field.attributes.localized && 
               ['string', 'text'].includes(field.attributes.field_type)
    );

    if (translatableFields.length === 0) {
      ctx.alert('No translatable fields found');
      return;
    }

    // Select target locale
    const targetLocale = await ctx.openModal({
      id: 'locale-selector',
      title: 'Translate Content',
      parameters: {
        currentLocale: locale,
        availableLocales: ctx.site.attributes.locales.filter(l => l !== locale)
      }
    });

    if (!targetLocale) return;

    try {
      ctx.notice('Translating content...');

      // Translate each field
      for (const field of translatableFields) {
        const sourceValue = ctx.formValues[field.attributes.api_key];
        if (!sourceValue) continue;

        const translated = await translateText({
          text: sourceValue,
          from: locale,
          to: targetLocale,
          apiKey: ctx.plugin.attributes.parameters.translationApiKey
        });

        // Switch to target locale and update
        await ctx.switchLocale(targetLocale);
        await ctx.setFieldValue(field.attributes.api_key, translated);
      }

      ctx.notice(`Content translated to ${targetLocale}`);
    } catch (error) {
      ctx.alert(`Translation failed: ${error.message}`);
    }
  }
}
```

## Error Handling

Always handle errors gracefully:

```typescript
async executeItemFormDropdownAction(
  actionId: string,
  ctx: ExecuteItemFormDropdownActionCtx
) {
  try {
    switch (actionId) {
      case 'my-action':
        await performAction(ctx);
        break;
      default:
        console.warn(`Unknown action: ${actionId}`);
    }
  } catch (error) {
    console.error('Action failed:', error);
    ctx.alert(`Action failed: ${error.message}`);
  }
}
```

## Best Practices

1. **Validate Before Acting**: Check required fields and conditions
2. **Show Progress**: Use `ctx.notice()` for long-running operations
3. **Confirm Destructive Actions**: Use `ctx.openConfirm()` for irreversible changes
4. **Handle Errors**: Always wrap in try-catch and show user-friendly messages
5. **Preserve User Work**: Save or confirm before making major changes
6. **Provide Feedback**: Always inform the user of success or failure
7. **Check Permissions**: Verify user has permission for the action

## Related Hooks

- **itemFormDropdownActions**: Defines the dropdown actions
- **fieldDropdownActions**: Actions for individual fields
- **executeFieldDropdownAction**: Executes field-specific actions
- **onBeforeItemUpsert**: Validate before saving

## Important Notes

- Actions should complete reasonably quickly or show progress
- Changes made with `setFieldValue` are immediate but not saved
- Use `saveCurrentItem()` to persist changes
- Actions can open modals for complex interactions
- Consider the current form state (new vs existing record)
- Respect field validations and constraints
- Actions can be async but should handle errors properly