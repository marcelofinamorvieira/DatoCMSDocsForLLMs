# executeSchemaItemTypeDropdownAction Hook

## Purpose

The `executeSchemaItemTypeDropdownAction` hook executes custom actions on models (item types) from the schema editor dropdown menu. This hook handles operations like model migrations, field management, validation rule updates, or any schema-level operations you need to perform on content models.

## Signature

```typescript
executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: SchemaTypes.ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
): Promise<void>
```

## Parameters

- `actionId`: string - The unique identifier of the action being executed
- `itemType`: SchemaTypes.ItemType - The model/block model on which the action should be executed

## Complete Type Definitions

```typescript
import type { SchemaTypes } from '@datocms/cma-client';

// Main hook type
export type ExecuteSchemaItemTypeDropdownActionHook = {
  executeSchemaItemTypeDropdownAction: (
    actionId: string,
    itemType: SchemaTypes.ItemType,
    ctx: ExecuteSchemaItemTypeDropdownActionCtx,
  ) => Promise<void>;
};

// Field creation parameters
type FieldCreationParams = {
  label: string;
  field_type: string;
  api_key: string;
  hint?: string;
  localized?: boolean;
  validators?: Record<string, unknown>;
  appearance?: {
    type?: string;
    parameters?: Record<string, unknown>;
    addons?: Array<{
      id: string;
      parameters?: Record<string, unknown>;
    }>;
  };
  position?: number;
  fieldset?: string;
  default_value?: unknown;
  required?: boolean;
};

// Field update parameters
type FieldUpdateParams = Partial<FieldCreationParams> & {
  id?: never; // Cannot update ID
};

// Fieldset creation parameters
type FieldsetCreationParams = {
  title: string;
  hint?: string;
  collapsible?: boolean;
  start_collapsed?: boolean;
  position?: number;
};

// Item type update parameters
type ItemTypeUpdateParams = {
  name?: string;
  api_key?: string;
  singleton?: boolean;
  sortable?: boolean;
  tree?: boolean;
  modular_block?: boolean;
  draft_enabled?: boolean;
  publishing_enabled?: boolean;
  auto_publish?: boolean;
  require_approval?: boolean;
  approval_roles?: string[];
  workflow_stages?: Array<{
    id: string;
    label: string;
    color: string;
  }>;
  api_enabled?: boolean;
  api_read_only?: boolean;
  allowed_api_operations?: string[];
  rate_limit?: number;
  title_field?: string;
  excerpt_field?: string;
  image_preview_field?: string;
  ordering_direction?: 'asc' | 'desc';
  ordering_field?: string;
  inverse_relationships_enabled?: boolean;
};

// Item list location query for item selectors
type ItemListLocationQuery = {
  filter?: {
    type?: string;
    query?: string;
    ids?: string[];
  };
  order_by?: string;
  order_direction?: 'asc' | 'desc';
};

// Full context type with all properties and methods
export type ExecuteSchemaItemTypeDropdownActionCtx = {
  // === Base Properties ===
  
  // Plugin information
  plugin: SchemaTypes.Plugin;
  
  // Authentication
  currentUser: SchemaTypes.User | SchemaTypes.SsoUser | SchemaTypes.Account | SchemaTypes.Organization;
  currentRole: SchemaTypes.Role;
  currentUserAccessToken: string | undefined;
  
  // Project information
  site: SchemaTypes.Site;
  environment: string;
  isEnvironmentPrimary: boolean;
  owner: SchemaTypes.Account | SchemaTypes.Organization;
  account: SchemaTypes.Account | undefined; // Deprecated - use owner
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
  itemTypes: Partial<Record<string, SchemaTypes.ItemType>>;
  fields: Partial<Record<string, SchemaTypes.Field>>;
  fieldsets: Partial<Record<string, SchemaTypes.Fieldset>>;
  users: Partial<Record<string, SchemaTypes.User>>;
  ssoUsers: Partial<Record<string, SchemaTypes.SsoUser>>;
  
  // === Schema-Specific Properties ===
  
  /** The model being acted upon (also passed as second parameter) */
  itemType: SchemaTypes.ItemType;
  /** All fields belonging to this model */
  fields: SchemaTypes.Field[];
  /** Fieldsets for organizing fields in this model */
  fieldsets: SchemaTypes.Fieldset[];
  /** Parameters passed from the action definition */
  parameters: Record<string, unknown> | undefined;
  
  // === Base Methods ===
  
  // Data loading
  loadItemTypeFields: (itemTypeId: string) => Promise<SchemaTypes.Field[]>;
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<SchemaTypes.Fieldset[]>;
  loadFieldsUsingPlugin: () => Promise<SchemaTypes.Field[]>;
  loadUsers: () => Promise<SchemaTypes.User[]>;
  loadSsoUsers: () => Promise<SchemaTypes.SsoUser[]>;
  
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
  createNewItem: (itemTypeId: string) => Promise<SchemaTypes.Item | null>;
  selectItem: {
    (itemTypeId: string, options: { multiple: true; initialLocationQuery?: ItemListLocationQuery }): Promise<SchemaTypes.Item[] | null>;
    (itemTypeId: string, options?: { multiple: false; initialLocationQuery?: ItemListLocationQuery }): Promise<SchemaTypes.Item | null>;
  };
  editItem: (itemId: string) => Promise<SchemaTypes.Item | null>;
  
  // Upload dialogs
  selectUpload: {
    (options: { multiple: true }): Promise<SchemaTypes.Upload[] | null>;
    (options?: { multiple: false }): Promise<SchemaTypes.Upload | null>;
  };
  editUpload: (uploadId: string) => Promise<(SchemaTypes.Upload & { deleted?: true }) | null>;
  editUploadMetadata: (
    fileFieldValue: {
      upload_id: string;
      alt?: string;
      title?: string;
      custom_data?: Record<string, unknown>;
      focal_point?: { x: number; y: number };
    },
  ) => Promise<{
    upload_id: string;
    alt?: string;
    title?: string;
    custom_data?: Record<string, unknown>;
    focal_point?: { x: number; y: number };
  } | null>;
  
  // File uploads
  uploadFile: (file: File) => Promise<SchemaTypes.Upload>;
  
  // Navigation
  navigateTo: (url: string) => Promise<void>;
  
  // User confirmations and dialogs
  openConfirm: <Value = boolean>(options: {
    title: string;
    content: string;
    choices: Array<{
      label: string;
      value: Value;
      intent?: 'positive' | 'negative';
    }>;
    cancel?: {
      label: string;
      value: Value;
    };
  }) => Promise<Value>;
  
  openModal: <ReturnValue = unknown>(modal: {
    id: string;
    title?: string;
    width?: 'xs' | 's' | 'm' | 'l' | 'xl' | 'xxl' | 'fullWidth';
    initialHeight?: number;
    parameters?: Record<string, unknown>;
  }) => Promise<ReturnValue | null>;
  
  // === Schema Management Methods ===
  
  /** Create a new field in this model */
  createField: (field: FieldCreationParams) => Promise<SchemaTypes.Field>;
  
  /** Update an existing field */
  updateField: (fieldId: string, updates: FieldUpdateParams) => Promise<SchemaTypes.Field>;
  
  /** Delete a field from this model */
  deleteField: (fieldId: string) => Promise<void>;
  
  /** Update model settings */
  updateItemType: (updates: ItemTypeUpdateParams) => Promise<SchemaTypes.ItemType>;
  
  /** Create a new fieldset for organizing fields */
  createFieldset: (fieldset: FieldsetCreationParams) => Promise<SchemaTypes.Fieldset>;
  
  /** Update an existing fieldset */
  updateFieldset: (fieldsetId: string, updates: Partial<FieldsetCreationParams>) => Promise<SchemaTypes.Fieldset>;
  
  /** Delete a fieldset */
  deleteFieldset: (fieldsetId: string) => Promise<void>;
  
  // === Item Management Methods ===
  
  /** Get all items of this model type */
  loadItems: (options?: {
    filter?: Record<string, unknown>;
    order_by?: string;
    order_direction?: 'asc' | 'desc';
    page?: number;
    per_page?: number;
  }) => Promise<SchemaTypes.Item[]>;
  
  /** Get a specific item by ID */
  loadItem: (itemId: string) => Promise<SchemaTypes.Item | null>;
  
  /** Create a new item of this model type */
  createItem: (attributes: Record<string, unknown>) => Promise<SchemaTypes.Item>;
  
  /** Update an existing item */
  updateItem: (itemId: string, attributes: Record<string, unknown>) => Promise<SchemaTypes.Item>;
  
  /** Delete an item */
  deleteItem: (itemId: string) => Promise<void>;
  
  /** Publish items */
  publishItems: (itemIds: string[]) => Promise<void>;
  
  /** Unpublish items */
  unpublishItems: (itemIds: string[]) => Promise<void>;
  
  /** Duplicate an item */
  duplicateItem: (itemId: string, options?: {
    copyReferences?: boolean;
    parentId?: string;
  }) => Promise<SchemaTypes.Item>;
};
```

## Context Properties

The context object includes all standard plugin properties plus schema-specific properties and methods. See the complete type definition above for all available properties and methods, including:

- **Schema data**: `itemType`, `fields`, `fieldsets` for the current model
- **Schema management**: Methods to create, update, and delete fields and fieldsets  
- **Item management**: Methods to work with items of this model type
- **UI dialogs**: Methods for user interaction, confirmations, and modals
- **Standard context**: Plugin info, authentication, site data, and utility methods

## Use Cases

### 1. Field Bulk Operations

Manage multiple fields at once:

```typescript
async executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
): Promise<void> {
  if (actionId === 'add-seo-fields') {
    const { itemType, fields } = ctx;

    // Check if SEO fields already exist
    const existingSeoFields = fields.filter(f => 
      ['seo_title', 'meta_description', 'og_image'].includes(f.attributes.api_key)
    );

    if (existingSeoFields.length > 0) {
      const proceed = await ctx.openConfirm({
        title: 'SEO Fields Exist',
        content: `Found ${existingSeoFields.length} existing SEO fields. Add remaining fields?`,
        choices: [
          { label: 'Cancel', value: false },
          { label: 'Add Missing Fields', value: true }
        ]
      });

      if (!proceed) return;
    }

    try {
      ctx.notice('Adding SEO fields...');

      // Create SEO fieldset first
      let seoFieldset = ctx.fieldsets.find(fs => fs.attributes.title === 'SEO');
      if (!seoFieldset) {
        seoFieldset = await ctx.createFieldset({
          title: 'SEO',
          hint: 'Search engine optimization settings',
          collapsible: true,
          start_collapsed: true
        });
      }

      // Define SEO fields
      const seoFieldDefinitions = [
        {
          label: 'SEO Title',
          field_type: 'string',
          api_key: 'seo_title',
          hint: 'Title for search engines (50-60 characters)',
          validators: {
            length: { max: 60 }
          },
          appearance: {
            addons: [
              {
                id: 'charCounter',
                parameters: { limit: 60 }
              }
            ]
          },
          fieldset: seoFieldset.id
        },
        {
          label: 'Meta Description',
          field_type: 'text',
          api_key: 'meta_description',
          hint: 'Description for search engines (150-160 characters)',
          validators: {
            length: { max: 160 }
          },
          appearance: {
            type: 'textarea',
            parameters: { rows: 3 }
          },
          fieldset: seoFieldset.id
        },
        {
          label: 'Open Graph Image',
          field_type: 'file',
          api_key: 'og_image',
          hint: 'Image for social media sharing (1200x630px recommended)',
          validators: {
            accept_only_specified_extensions: true,
            specified_extensions: ['jpg', 'jpeg', 'png', 'webp']
          },
          fieldset: seoFieldset.id
        },
        {
          label: 'Focus Keywords',
          field_type: 'string',
          api_key: 'focus_keywords',
          hint: 'Comma-separated keywords for SEO analysis',
          appearance: {
            type: 'tags'
          },
          fieldset: seoFieldset.id
        }
      ];

      // Create missing fields
      const createdFields = [];
      for (const fieldDef of seoFieldDefinitions) {
        const exists = fields.find(f => f.attributes.api_key === fieldDef.api_key);
        if (!exists) {
          const field = await ctx.createField(fieldDef);
          createdFields.push(field);
        }
      }

      ctx.notice(`✅ Added ${createdFields.length} SEO fields to ${itemType.attributes.name}`);
    } catch (error) {
      ctx.alert(`Failed to add SEO fields: ${error.message}`);
    }
  }

  if (actionId === 'configure-localization') {
    const { fields } = ctx;

    // Get all string/text fields
    const localizableFields = fields.filter(f => 
      ['string', 'text', 'structured_text'].includes(f.attributes.field_type) &&
      !f.attributes.localized
    );

    if (localizableFields.length === 0) {
      ctx.alert('No fields available for localization');
      return;
    }

    // Let user select fields to localize
    const selectedFields = await ctx.openModal({
      id: 'field-selector',
      title: 'Configure Localization',
      width: 'l',
      parameters: {
        fields: localizableFields,
        locales: ctx.site.attributes.locales
      }
    });

    if (!selectedFields || selectedFields.length === 0) return;

    try {
      ctx.notice('Configuring localization...');

      for (const fieldId of selectedFields) {
        await ctx.updateField(fieldId, {
          localized: true
        });
      }

      ctx.notice(`✅ Enabled localization for ${selectedFields.length} fields`);

      // Offer to set default locale values
      const setDefaults = await ctx.openConfirm({
        title: 'Set Default Values',
        content: 'Would you like to copy current values to all locales as defaults?',
        choices: [
          { label: 'Skip', value: false },
          { label: 'Copy Values', value: true }
        ]
      });

      if (setDefaults) {
        // This would require accessing and updating existing records
        await copyDefaultLocaleValues(ctx.itemType.id, selectedFields);
      }
    } catch (error) {
      ctx.alert(`Localization setup failed: ${error.message}`);
    }
  }
}
```

### 2. Model Configuration and Settings

Update model-level configurations:

```typescript
executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: SchemaTypes.ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
) {
  if (actionId === 'configure-workflow') {
    const { itemType } = ctx;

    // Show workflow configuration modal
    const workflowConfig = await ctx.openModal({
      id: 'workflow-config',
      title: `Configure Workflow for ${itemType.attributes.name}`,
      width: 'l',
      parameters: {
        currentSettings: {
          draftEnabled: itemType.attributes.draft_enabled,
          publishingEnabled: itemType.attributes.publishing_enabled,
          workflowStages: itemType.attributes.workflow_stages
        }
      }
    });

    if (!workflowConfig) return;

    try {
      // Update model settings
      await ctx.updateItemType({
        draft_enabled: workflowConfig.draftEnabled,
        publishing_enabled: workflowConfig.publishingEnabled,
        workflow_stages: workflowConfig.stages,
        // Additional workflow settings
        auto_publish: workflowConfig.autoPublish,
        require_approval: workflowConfig.requireApproval,
        approval_roles: workflowConfig.approvalRoles
      });

      // Create workflow status field if needed
      if (workflowConfig.addStatusField) {
        const statusField = await ctx.createField({
          label: 'Workflow Status',
          field_type: 'string',
          api_key: 'workflow_status',
          validators: {
            enum: workflowConfig.stages.map(s => s.id)
          },
          appearance: {
            type: 'select',
            parameters: {
              options: workflowConfig.stages.map(s => ({
                label: s.label,
                value: s.id
              }))
            }
          }
        });
      }

      ctx.notice('Workflow configuration updated successfully');
    } catch (error) {
      ctx.alert(`Failed to update workflow: ${error.message}`);
    }
  }

  if (actionId === 'configure-api-access') {
    const { itemType } = ctx;

    const apiConfig = await ctx.openModal({
      id: 'api-access-config',
      title: 'Configure API Access',
      width: 'm',
      parameters: {
        current: {
          apiKey: itemType.attributes.api_key,
          enabled: itemType.attributes.api_enabled,
          readOnly: itemType.attributes.api_read_only
        }
      }
    });

    if (!apiConfig) return;

    try {
      await ctx.updateItemType({
        api_key: apiConfig.apiKey,
        api_enabled: apiConfig.enabled,
        api_read_only: apiConfig.readOnly,
        allowed_api_operations: apiConfig.allowedOperations,
        rate_limit: apiConfig.rateLimit
      });

      ctx.notice('API access configuration updated');
    } catch (error) {
      ctx.alert(`Failed to update API access: ${error.message}`);
    }
  }
}
```

### 3. Schema Migration and Transformation

Transform model structure:

```typescript
executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: SchemaTypes.ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
) {
  if (actionId === 'split-name-field') {
    const { fields } = ctx;

    // Find name field
    const nameField = fields.find(f => 
      f.attributes.api_key === 'name' || 
      f.attributes.api_key === 'full_name'
    );

    if (!nameField) {
      ctx.alert('No name field found to split');
      return;
    }

    const confirmed = await ctx.openConfirm({
      title: 'Split Name Field',
      content: 'This will create first_name and last_name fields. The original field will be kept but can be removed later.',
      choices: [
        { label: 'Cancel', value: false },
        { label: 'Split Field', value: true }
      ]
    });

    if (!confirmed) return;

    try {
      // Create new fields
      const firstNameField = await ctx.createField({
        label: 'First Name',
        field_type: 'string',
        api_key: 'first_name',
        position: nameField.attributes.position,
        validators: nameField.attributes.validators
      });

      const lastNameField = await ctx.createField({
        label: 'Last Name',
        field_type: 'string',
        api_key: 'last_name',
        position: nameField.attributes.position + 1,
        validators: {}
      });

      // Offer to migrate existing data
      const migrateData = await ctx.openConfirm({
        title: 'Migrate Data',
        content: 'Would you like to split existing name data into the new fields?',
        choices: [
          { label: 'Skip', value: false },
          { label: 'Migrate', value: true }
        ]
      });

      if (migrateData) {
        await runNameSplitMigration(ctx.itemType.id, nameField.id, firstNameField.id, lastNameField.id);
      }

      ctx.notice('Name field split successfully');
    } catch (error) {
      ctx.alert(`Failed to split field: ${error.message}`);
    }
  }

  if (actionId === 'merge-duplicate-fields') {
    const { fields } = ctx;

    // Detect potential duplicate fields
    const duplicates = detectDuplicateFields(fields);

    if (duplicates.length === 0) {
      ctx.alert('No duplicate fields detected');
      return;
    }

    // Show merge options
    const mergeOptions = await ctx.openModal({
      id: 'merge-fields',
      title: 'Merge Duplicate Fields',
      width: 'l',
      parameters: { duplicates, fields }
    });

    if (!mergeOptions) return;

    try {
      for (const merge of mergeOptions.merges) {
        // Update target field if needed
        if (merge.updateTarget) {
          await ctx.updateField(merge.targetId, merge.updates);
        }

        // Migrate data from source to target
        await migrateFieldData(
          ctx.itemType.id,
          merge.sourceId,
          merge.targetId,
          merge.strategy
        );

        // Delete source field
        if (merge.deleteSource) {
          await ctx.deleteField(merge.sourceId);
        }
      }

      ctx.notice(`Merged ${mergeOptions.merges.length} field pairs`);
    } catch (error) {
      ctx.alert(`Merge failed: ${error.message}`);
    }
  }
}
```

### 4. Validation and Constraints

Add or update validation rules:

```typescript
executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: SchemaTypes.ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
) {
  if (actionId === 'add-validation-rules') {
    const { fields } = ctx;

    // Group fields by type for appropriate validations
    const fieldsByType = {
      string: fields.filter(f => f.attributes.field_type === 'string'),
      text: fields.filter(f => f.attributes.field_type === 'text'),
      integer: fields.filter(f => f.attributes.field_type === 'integer'),
      float: fields.filter(f => f.attributes.field_type === 'float'),
      date: fields.filter(f => f.attributes.field_type === 'date')
    };

    const validationPlan = await ctx.openModal({
      id: 'validation-planner',
      title: 'Add Validation Rules',
      width: 'xl',
      parameters: { fieldsByType }
    });

    if (!validationPlan) return;

    try {
      let updatedCount = 0;

      for (const rule of validationPlan.rules) {
        const field = fields.find(f => f.id === rule.fieldId);
        if (!field) continue;

        const validators = { ...field.attributes.validators };

        // Apply validation based on type
        switch (rule.type) {
          case 'required':
            validators.required = true;
            break;
          
          case 'length':
            validators.length = {
              min: rule.min,
              max: rule.max
            };
            break;
          
          case 'pattern':
            validators.pattern = {
              regex: rule.regex,
              message: rule.message
            };
            break;
          
          case 'range':
            validators.number_range = {
              min: rule.min,
              max: rule.max
            };
            break;
          
          case 'unique':
            validators.unique = true;
            break;
          
          case 'format':
            validators.format = rule.format; // email, url, etc.
            break;
        }

        await ctx.updateField(field.id, { validators });
        updatedCount++;
      }

      ctx.notice(`Added validation rules to ${updatedCount} fields`);
    } catch (error) {
      ctx.alert(`Failed to add validations: ${error.message}`);
    }
  }

  if (actionId === 'enforce-required-fields') {
    const { fields, itemType } = ctx;

    // Suggest fields that should be required
    const suggestions = suggestRequiredFields(fields, itemType);

    const selectedFields = await ctx.openModal({
      id: 'required-field-selector',
      title: 'Enforce Required Fields',
      width: 'l',
      parameters: {
        suggestions,
        fields: fields.filter(f => !f.attributes.validators?.required)
      }
    });

    if (!selectedFields || selectedFields.length === 0) return;

    try {
      // Check existing records for empty values
      const emptyRecords = await checkEmptyRequiredFields(
        ctx.itemType.id,
        selectedFields
      );

      if (emptyRecords.length > 0) {
        const strategy = await ctx.openModal({
          id: 'empty-records-strategy',
          title: 'Handle Empty Values',
          width: 'm',
          parameters: {
            emptyRecords,
            fields: selectedFields
          }
        });

        if (!strategy) return;

        // Handle empty records based on strategy
        await handleEmptyRecords(emptyRecords, strategy);
      }

      // Make fields required
      for (const fieldId of selectedFields) {
        const field = fields.find(f => f.id === fieldId);
        await ctx.updateField(fieldId, {
          validators: {
            ...field.attributes.validators,
            required: true
          }
        });
      }

      ctx.notice(`Made ${selectedFields.length} fields required`);
    } catch (error) {
      ctx.alert(`Failed to enforce required fields: ${error.message}`);
    }
  }
}
```

### 5. Import/Export Schema

Import or export model schemas:

```typescript
executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: SchemaTypes.ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
) {
  if (actionId === 'export-schema') {
    const { itemType, fields, fieldsets } = ctx;

    const exportOptions = await ctx.openModal({
      id: 'export-options',
      title: 'Export Model Schema',
      width: 's',
      parameters: {
        formats: ['json', 'yaml', 'typescript']
      }
    });

    if (!exportOptions) return;

    try {
      const schema = {
        model: {
          name: itemType.attributes.name,
          api_key: itemType.attributes.api_key,
          singleton: itemType.attributes.singleton,
          sortable: itemType.attributes.sortable,
          tree: itemType.attributes.tree,
          settings: {
            draft_enabled: itemType.attributes.draft_enabled,
            publishing_enabled: itemType.attributes.publishing_enabled
          }
        },
        fieldsets: fieldsets.map(fs => ({
          title: fs.attributes.title,
          hint: fs.attributes.hint,
          collapsible: fs.attributes.collapsible
        })),
        fields: fields.map(f => ({
          label: f.attributes.label,
          api_key: f.attributes.api_key,
          field_type: f.attributes.field_type,
          hint: f.attributes.hint,
          localized: f.attributes.localized,
          validators: f.attributes.validators,
          appearance: f.attributes.appearance,
          position: f.attributes.position,
          fieldset: f.relationships?.fieldset?.data?.id
        }))
      };

      let content;
      let filename;
      let mimeType;

      switch (exportOptions.format) {
        case 'json':
          content = JSON.stringify(schema, null, 2);
          filename = `${itemType.attributes.api_key}-schema.json`;
          mimeType = 'application/json';
          break;
        
        case 'yaml':
          content = toYAML(schema);
          filename = `${itemType.attributes.api_key}-schema.yml`;
          mimeType = 'text/yaml';
          break;
        
        case 'typescript':
          content = generateTypeScript(schema);
          filename = `${itemType.attributes.api_key}.types.ts`;
          mimeType = 'text/typescript';
          break;
      }

      // Download file
      const blob = new Blob([content], { type: mimeType });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = filename;
      link.click();
      URL.revokeObjectURL(url);

      ctx.notice('Schema exported successfully');
    } catch (error) {
      ctx.alert(`Export failed: ${error.message}`);
    }
  }

  if (actionId === 'import-schema') {
    // Show import modal
    const importData = await ctx.openModal({
      id: 'schema-importer',
      title: 'Import Model Schema',
      width: 'l'
    });

    if (!importData) return;

    try {
      const { schema, options } = importData;

      // Validate schema
      const validation = validateSchema(schema);
      if (!validation.valid) {
        ctx.alert(`Invalid schema: ${validation.errors.join(', ')}`);
        return;
      }

      // Show preview of changes
      const changes = compareSchemas(
        { itemType, fields, fieldsets },
        schema
      );

      const confirmed = await ctx.openModal({
        id: 'import-preview',
        title: 'Preview Schema Changes',
        width: 'xl',
        parameters: { changes, options }
      });

      if (!confirmed) return;

      // Apply schema changes
      ctx.notice('Importing schema...');

      // Update model settings
      if (options.updateModel && changes.modelChanges) {
        await ctx.updateItemType(changes.modelChanges);
      }

      // Create/update fieldsets
      if (options.importFieldsets) {
        for (const fieldset of changes.newFieldsets) {
          await ctx.createFieldset(fieldset);
        }
      }

      // Create/update fields
      if (options.importFields) {
        for (const field of changes.newFields) {
          await ctx.createField(field);
        }
        
        for (const update of changes.fieldUpdates) {
          await ctx.updateField(update.id, update.changes);
        }
      }

      ctx.notice('Schema imported successfully');
    } catch (error) {
      ctx.alert(`Import failed: ${error.message}`);
    }
  }
}
```

### 6. Schema Analysis and Optimization

Analyze and optimize model structure:

```typescript
executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: SchemaTypes.ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
) {
  if (actionId === 'analyze-schema') {
    const { itemType, fields } = ctx;

    ctx.notice('Analyzing model schema...');

    const analysis = {
      fieldCount: fields.length,
      fieldTypes: {},
      issues: [],
      suggestions: [],
      complexity: 0
    };

    // Analyze field distribution
    fields.forEach(field => {
      const type = field.attributes.field_type;
      analysis.fieldTypes[type] = (analysis.fieldTypes[type] || 0) + 1;
    });

    // Check for issues
    // 1. Too many fields
    if (fields.length > 50) {
      analysis.issues.push({
        severity: 'warning',
        message: `Model has ${fields.length} fields. Consider splitting into related models.`
      });
    }

    // 2. Missing title field
    if (!itemType.attributes.title_field) {
      analysis.issues.push({
        severity: 'warning',
        message: 'No title field configured for record display'
      });
    }

    // 3. Duplicate field types
    const titleFields = fields.filter(f => 
      ['title', 'name', 'heading'].includes(f.attributes.api_key)
    );
    if (titleFields.length > 1) {
      analysis.issues.push({
        severity: 'info',
        message: 'Multiple potential title fields found'
      });
    }

    // 4. Check for optimization opportunities
    const unindexedSearchFields = fields.filter(f =>
      ['string', 'text'].includes(f.attributes.field_type) &&
      !f.attributes.appearance?.parameters?.indexed_for_search
    );
    
    if (unindexedSearchFields.length > 0) {
      analysis.suggestions.push({
        type: 'performance',
        message: `${unindexedSearchFields.length} text fields could be indexed for search`,
        fields: unindexedSearchFields.map(f => f.attributes.api_key)
      });
    }

    // 5. Localization check
    const localizableButNot = fields.filter(f =>
      ['string', 'text', 'structured_text'].includes(f.attributes.field_type) &&
      !f.attributes.localized &&
      ctx.site.attributes.locales.length > 1
    );

    if (localizableButNot.length > 0) {
      analysis.suggestions.push({
        type: 'i18n',
        message: `${localizableButNot.length} fields could be localized`,
        fields: localizableButNot.map(f => f.attributes.api_key)
      });
    }

    // Calculate complexity score
    analysis.complexity = calculateSchemaComplexity(itemType, fields);

    // Show analysis results
    await ctx.openModal({
      id: 'schema-analysis',
      title: 'Schema Analysis Report',
      width: 'xl',
      parameters: { analysis, itemType, fields }
    });
  }

  if (actionId === 'optimize-performance') {
    const { fields } = ctx;

    // Find optimization opportunities
    const optimizations = [];

    // 1. String fields that could be enums
    const enumCandidates = await findEnumCandidates(ctx.itemType.id, fields);
    optimizations.push(...enumCandidates);

    // 2. Large text fields without lazy loading
    const largeTextFields = fields.filter(f =>
      f.attributes.field_type === 'text' &&
      !f.attributes.appearance?.parameters?.lazy_load
    );
    if (largeTextFields.length > 0) {
      optimizations.push({
        type: 'lazy_load',
        fields: largeTextFields,
        benefit: 'Faster record loading'
      });
    }

    // 3. Unused fields
    const unusedFields = await findUnusedFields(ctx.itemType.id, fields);
    if (unusedFields.length > 0) {
      optimizations.push({
        type: 'remove_unused',
        fields: unusedFields,
        benefit: 'Cleaner schema'
      });
    }

    if (optimizations.length === 0) {
      ctx.alert('No optimization opportunities found');
      return;
    }

    // Show optimization options
    const selected = await ctx.openModal({
      id: 'optimization-selector',
      title: 'Schema Optimizations',
      width: 'xl',
      parameters: { optimizations }
    });

    if (!selected || selected.length === 0) return;

    try {
      // Apply selected optimizations
      for (const opt of selected) {
        switch (opt.type) {
          case 'convert_to_enum':
            await convertToEnumField(ctx, opt.field, opt.values);
            break;
          
          case 'lazy_load':
            await enableLazyLoading(ctx, opt.fields);
            break;
          
          case 'remove_unused':
            await removeUnusedFields(ctx, opt.fields);
            break;
        }
      }

      ctx.notice('Optimizations applied successfully');
    } catch (error) {
      ctx.alert(`Optimization failed: ${error.message}`);
    }
  }
}
```

## Best Practices

1. **Preview Changes**: Always show what will be changed before applying
2. **Backup Important Data**: Offer export before destructive operations
3. **Validate Schema Changes**: Ensure changes won't break existing functionality
4. **Handle Existing Data**: Consider migration when changing field types
5. **Permission Checks**: Verify user can modify schema
6. **Atomic Operations**: Group related changes together
7. **Progress Feedback**: Show progress for long-running operations

## Error Handling

```typescript
executeSchemaItemTypeDropdownAction(
  actionId: string,
  itemType: SchemaTypes.ItemType,
  ctx: ExecuteSchemaItemTypeDropdownActionCtx
) {
  try {
    // Check permissions
    if (!ctx.currentRole.attributes.can_edit_schema) {
      ctx.alert('You do not have permission to modify the schema');
      return;
    }

    // Validate environment
    if (ctx.environment === 'main') {
      const confirmed = await ctx.openConfirm({
        title: 'Production Environment',
        content: 'You are modifying the production schema. Are you sure?',
        choices: [
          { label: 'Cancel', value: false },
          { label: 'Continue', value: true, intent: 'negative' }
        ]
      });
      
      if (!confirmed) return;
    }

    // Execute action
    await performSchemaAction(actionId, itemType, ctx);
    
  } catch (error) {
    console.error('Schema action failed:', error);
    ctx.alert(`Operation failed: ${error.message}`);
  }
}
```

## Related Hooks

- **schemaItemTypeDropdownActions**: Defines available schema actions
- **itemFormDropdownActions**: Actions for record forms
- **fieldDropdownActions**: Actions for individual fields

## Important Notes

- Schema changes can affect existing data - always consider migration
- Some operations may require reindexing or cache clearing
- Changes to required fields need careful handling of existing records
- Field deletions are usually irreversible
- Consider the impact on API consumers when changing field names
- Test schema changes in non-production environments first
- Model and field API keys should follow naming conventions