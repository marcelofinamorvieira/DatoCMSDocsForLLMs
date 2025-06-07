# executeFieldDropdownAction Hook

## Purpose

The `executeFieldDropdownAction` hook handles the execution of custom dropdown actions defined for fields via the `fieldDropdownActions` hook. When a user clicks on a custom action in a field's dropdown menu, this hook receives the action ID and executes the corresponding functionality, allowing plugins to perform custom operations on field values.

## Signature

```typescript
async executeFieldDropdownAction(
  actionId: string,
  ctx: ExecuteFieldDropdownActionCtx
): Promise<void>
```

## Parameters

- `actionId`: string - The ID of the action that was clicked by the user (as defined in `fieldDropdownActions`)
- `ctx`: ExecuteFieldDropdownActionCtx - The context object containing field information, form state, and methods to interact with the form

## Type Definitions

```typescript
import type { SchemaTypes } from '@datocms/cma-client';

// Core schema types from DatoCMS
type Account = SchemaTypes.Account;
type Field = SchemaTypes.Field;
type Fieldset = SchemaTypes.Fieldset;
type Item = SchemaTypes.Item;
type ItemType = SchemaTypes.ItemType;
type Organization = SchemaTypes.Organization;
type Plugin = SchemaTypes.Plugin;
type Role = SchemaTypes.Role;
type Site = SchemaTypes.Site;
type SsoUser = SchemaTypes.SsoUser;
type Upload = SchemaTypes.Upload;
type User = SchemaTypes.User;

// Hook definition
export type ExecuteFieldDropdownActionHook = {
  executeFieldDropdownAction: (
    actionId: string,
    ctx: ExecuteFieldDropdownActionCtx,
  ) => Promise<void>;
};

// Context type for this hook
export type ExecuteFieldDropdownActionCtx = Ctx<
  ItemFormAdditionalProperties &
    FieldAdditionalProperties & {
      parameters: Record<string, unknown> | undefined;
    },
  ItemFormAdditionalMethods
>;

// Base context type
export type Ctx<
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BaseProperties & AdditionalProperties & BaseMethods & AdditionalMethods;

// Base properties available in all contexts
export type BaseProperties = {
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
  account: Account | undefined; // Deprecated, use owner
  ui: { locale: string };
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
};

// Base methods available in all contexts
export type BaseMethods = {
  // Load data methods
  loadItemTypeFields: (itemTypeId: string) => Promise<Field[]>;
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<Fieldset[]>;
  loadFieldsUsingPlugin: () => Promise<Field[]>;
  loadUsers: () => Promise<User[]>;
  loadSsoUsers: () => Promise<SsoUser[]>;
  
  // Update plugin parameters
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  updateFieldAppearance: (
    fieldId: string,
    changes: FieldAppearanceChange[],
  ) => Promise<void>;
  
  // Item dialogs
  createNewItem: (itemTypeId: string) => Promise<Item | null>;
  selectItem: {
    (itemTypeId: string, options: { multiple: true; initialLocationQuery?: ItemListLocationQuery }): Promise<Item[] | null>;
    (itemTypeId: string, options?: { multiple: false; initialLocationQuery?: ItemListLocationQuery }): Promise<Item | null>;
  };
  editItem: (itemId: string) => Promise<Item | null>;
  
  // Toast notifications
  alert: (message: string) => Promise<void>;
  notice: (message: string) => Promise<void>;
  customToast: <CtaValue = unknown>(toast: Toast<CtaValue>) => Promise<CtaValue | null>;
  
  // Upload dialogs
  selectUpload: {
    (options: { multiple: true }): Promise<Upload[] | null>;
    (options?: { multiple: false }): Promise<Upload | null>;
  };
  editUpload: (uploadId: string) => Promise<(Upload & { deleted?: true }) | null>;
  editUploadMetadata: (
    fileFieldValue: FileFieldValue,
    locale?: string,
  ) => Promise<FileFieldValue | null>;
  
  // Custom dialogs
  openModal: (modal: Modal) => Promise<unknown>;
  openConfirm: (options: ConfirmOptions) => Promise<unknown>;
  
  // Navigation
  navigateTo: (path: string) => Promise<void>;
};

// Field-specific properties included in context
export type FieldAdditionalProperties = {
  /** Whether the field is currently disabled or not */
  disabled: boolean;
  /** The path in the formValues object where to find the current value for the field */
  fieldPath: string;
  /** The field where the field extension is installed to */
  field: Field;
  /** If the field extension is installed in a field of a block, returns the top level Modular Content/Structured Text field containing the block itself */
  parentField: Field | undefined;
  /** If the field extension is installed in a field of a block, returns the ID of the block — or undefined if the block is still not persisted — and the block model. */
  block: undefined | { id: string | undefined; blockModel: ItemType };
};

// Form-specific properties included in context
export type ItemFormAdditionalProperties = {
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
  blocksAnalysis: BlocksAnalysis;
};

// Form-specific methods included in context
export type ItemFormAdditionalMethods = {
  /** Hides/shows a specific field in the form */
  toggleField: (path: string, show: boolean) => Promise<void>;
  /** Disables/re-enables a specific field in the form */
  disableField: (path: string, disable: boolean) => Promise<void>;
  /** Smoothly navigates to a specific field in the form */
  scrollToField: (path: string, locale?: string) => Promise<void>;
  /** Changes a specific path of the formValues object */
  setFieldValue: (path: string, value: unknown) => Promise<void>;
  /** Takes the internal form state and transforms it into an Item entity compatible with DatoCMS API */
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
export type BlocksAnalysis = {
  usage: {
    /** Total number of blocks present in form state */
    total: number;
    /** Total number of blocks present in non-localized fields */
    nonLocalized: number;
    /** Total number of blocks present in localized fields, per locale */
    perLocale: Record<string, number>;
  };
  /** Maximum number of blocks per item */
  maximumPerItem: number;
};

export type FieldAppearanceChange =
  | { operation: 'removeEditor' }
  | { operation: 'updateEditor'; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'setEditor'; fieldExtensionId: string; parameters: Record<string, unknown> }
  | { operation: 'removeAddon'; index: number }
  | { operation: 'updateAddon'; index: number; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'insertAddon'; index: number; fieldExtensionId: string; parameters: Record<string, unknown> };

export type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};

export type Toast<CtaValue = unknown> = {
  /** Message of the notification */
  message: string;
  /** Type of notification. Will present the toast in a different color accent. */
  type: 'notice' | 'alert' | 'warning';
  /** An optional button to show inside the toast */
  cta?: {
    /** Label for the button */
    label: string;
    /** The value to be returned by the customToast promise if the button is clicked by the user */
    value: CtaValue;
  };
  /** Whether the toast is to be automatically closed if the user changes page */
  dismissOnPageChange?: boolean;
  /** Whether the toast is to be automatically closed after some time (true will use the default DatoCMS time interval) */
  dismissAfterTimeout?: boolean | number;
};

export type FileFieldValue = {
  /** ID of the asset */
  upload_id: string;
  /** Alternate text for the asset */
  alt: string | null;
  /** Title for the asset */
  title: string | null;
  /** Focal point of an asset */
  focal_point: FocalPoint | null;
  /** Object with arbitrary metadata related to the asset */
  custom_data: Record<string, string>;
};

export type FocalPoint = {
  /** Horizontal position expressed as float between 0 and 1 */
  x: number;
  /** Vertical position expressed as float between 0 and 1 */
  y: number;
};

export type Modal = {
  /** ID of the modal. Will be the first argument for the renderModal function */
  id: string;
  /** Title for the modal. Ignored by fullWidth modals */
  title?: string;
  /** Whether to present a close button for the modal or not */
  closeDisabled?: boolean;
  /** Width of the modal. Can be a number, or one of the predefined sizes */
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  /** An arbitrary configuration object that will be passed as the parameters property of the second argument of the renderModal function */
  parameters?: Record<string, unknown>;
  /** The initial height to set for the iframe that will render the modal content */
  initialHeight?: number;
};

export type ConfirmOptions = {
  /** The title to be shown inside the confirmation panel */
  title: string;
  /** The main message to be shown inside the confirmation panel */
  content: string;
  /** The different options the user can choose from */
  choices: ConfirmChoice[];
  /** The cancel option to present to the user */
  cancel: ConfirmChoice;
};

export type ConfirmChoice = {
  /** The label to be shown for the choice */
  label: string;
  /** The value to be returned by the openConfirm promise if the button is clicked by the user */
  value: unknown;
  /** The intent of the button. Will present the button in a different color accent. */
  intent?: 'positive' | 'negative';
};
```

## Context Properties

The hook receives an extensive context object that includes:

### Base Context Properties
All standard properties like `plugin`, `currentUser`, `site`, etc.

### Item Form Properties
- `locale`: string - Currently active locale
- `item`: Item | null - The record being edited (null for new records)
- `itemType`: ItemType - The model of the record
- `formValues`: Record<string, unknown> - All current form values
- `itemStatus`: 'new' | 'draft' | 'updated' | 'published' - Record status
- `isSubmitting`: boolean - Whether form is being submitted
- `isFormDirty`: boolean - Whether form has unsaved changes
- `blocksAnalysis`: BlocksAnalysis - Analysis of blocks in the form

### Field-Specific Properties
- `disabled`: boolean - Whether the field is disabled
- `fieldPath`: string - Path to the field value in formValues
- `field`: Field - The field where the action was triggered
- `parentField`: Field | undefined - Parent field if in a block
- `block`: { id: string | undefined; blockModel: ItemType } | undefined - Block info if nested
- `parameters`: Record<string, unknown> | undefined - Action-specific parameters

### Methods
All item form methods including:
- `toggleField(path: string, show: boolean)` - Show/hide fields
- `disableField(path: string, disable: boolean)` - Enable/disable fields
- `scrollToField(path: string, locale?: string)` - Navigate to fields
- `setFieldValue(path: string, value: unknown)` - Update field values
- `formValuesToItem()` - Convert form to API format
- `itemToFormValues()` - Convert API format to form
- `saveCurrentItem(showToast?: boolean)` - Save the record

## Use Cases

### 1. Text Transformation Actions

Transform field content in various ways:

```typescript
async executeFieldDropdownAction(
  actionId: string,
  ctx: ExecuteFieldDropdownActionCtx
): Promise<void> {
  const currentValue = ctx.formValues[ctx.fieldPath] as string || '';

  switch (actionId) {
    case 'to-uppercase':
      await ctx.setFieldValue(ctx.fieldPath, currentValue.toUpperCase());
      ctx.notice('Text converted to uppercase');
      break;

    case 'to-lowercase':
      await ctx.setFieldValue(ctx.fieldPath, currentValue.toLowerCase());
      ctx.notice('Text converted to lowercase');
      break;

    case 'to-title-case':
      const titleCase = currentValue.replace(/\w\S*/g, (txt) => 
        txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase()
      );
      await ctx.setFieldValue(ctx.fieldPath, titleCase);
      ctx.notice('Text converted to title case');
      break;

    case 'slugify':
      const slug = currentValue
        .toLowerCase()
        .replace(/[^\w\s-]/g, '')
        .replace(/\s+/g, '-')
        .replace(/-+/g, '-')
        .trim();
      await ctx.setFieldValue(ctx.fieldPath, slug);
      ctx.notice('Text converted to slug format');
      break;

    case 'remove-html':
      const textOnly = currentValue.replace(/<[^>]*>/g, '');
      await ctx.setFieldValue(ctx.fieldPath, textOnly);
      ctx.notice('HTML tags removed');
      break;

    case 'count-words':
      const wordCount = currentValue.split(/\s+/).filter(Boolean).length;
      await ctx.alert(`Word count: ${wordCount}`);
      break;
  }
}
```

### 2. AI-Powered Content Enhancement

Integrate AI services to enhance content:

```typescript
async executeFieldDropdownAction(
  actionId: string,
  ctx: ExecuteFieldDropdownActionCtx
): Promise<void> {
  const currentValue = ctx.formValues[ctx.fieldPath] as string;
  
  // Show loading state
  await ctx.disableField(ctx.fieldPath, true);
  
  try {
    switch (actionId) {
      case 'ai-improve-writing':
        const improved = await callAIService({
          action: 'improve',
          content: currentValue,
          apiKey: ctx.plugin.attributes.parameters.openaiKey
        });
        
        const confirm = await ctx.openConfirm({
          title: 'AI Writing Improvement',
          content: `Original:\n${currentValue}\n\nImproved:\n${improved}\n\nApply the improvement?`,
          choices: [
            { label: 'Apply', value: true, intent: 'positive' },
            { label: 'Cancel', value: false }
          ]
        });
        
        if (confirm) {
          await ctx.setFieldValue(ctx.fieldPath, improved);
          ctx.notice('Writing improved by AI');
        }
        break;

      case 'ai-translate':
        const targetLocale = ctx.parameters?.targetLocale || 'es';
        const translated = await callAIService({
          action: 'translate',
          content: currentValue,
          targetLanguage: targetLocale,
          apiKey: ctx.plugin.attributes.parameters.openaiKey
        });
        
        // Set value in target locale field
        const targetFieldPath = ctx.fieldPath.replace(ctx.locale, targetLocale);
        await ctx.setFieldValue(targetFieldPath, translated);
        ctx.notice(`Translated to ${targetLocale}`);
        break;

      case 'ai-summarize':
        const summary = await callAIService({
          action: 'summarize',
          content: currentValue,
          maxLength: 200,
          apiKey: ctx.plugin.attributes.parameters.openaiKey
        });
        
        // Find summary field and update it
        const summaryFieldPath = ctx.fieldPath.replace('content', 'summary');
        if (ctx.formValues[summaryFieldPath] !== undefined) {
          await ctx.setFieldValue(summaryFieldPath, summary);
          ctx.notice('Summary generated');
        }
        break;
    }
  } catch (error) {
    ctx.alert(`AI service error: ${error.message}`);
  } finally {
    await ctx.disableField(ctx.fieldPath, false);
  }
}
```

### 3. Media and Asset Operations

Perform operations on media fields:

```typescript
async executeFieldDropdownAction(
  actionId: string,
  ctx: ExecuteFieldDropdownActionCtx
): Promise<void> {
  const assetData = ctx.formValues[ctx.fieldPath] as any;

  switch (actionId) {
    case 'optimize-image':
      if (!assetData?.upload_id) {
        ctx.alert('No image selected');
        return;
      }

      try {
        // Call image optimization service
        const optimized = await optimizeImage(assetData.upload_id, {
          quality: ctx.parameters?.quality || 85,
          format: ctx.parameters?.format || 'auto'
        });

        // Update with optimized version
        await ctx.setFieldValue(ctx.fieldPath, {
          upload_id: optimized.id,
          alt: assetData.alt || '',
          title: assetData.title || ''
        });

        ctx.notice('Image optimized successfully');
      } catch (error) {
        ctx.alert('Failed to optimize image');
      }
      break;

    case 'generate-alt-text':
      if (!assetData?.upload_id) {
        ctx.alert('No image selected');
        return;
      }

      const altText = await generateAltText(assetData.upload_id);
      await ctx.setFieldValue(ctx.fieldPath, {
        ...assetData,
        alt: altText
      });
      ctx.notice('Alt text generated');
      break;

    case 'bulk-upload':
      const files = await ctx.selectFiles({
        multiple: true,
        accept: 'image/*'
      });

      if (files.length > 0) {
        // Handle multiple file uploads
        const uploads = await Promise.all(
          files.map(file => uploadFile(file))
        );

        // If it's a gallery field, append to existing
        if (ctx.field.attributes.field_type === 'gallery') {
          const currentGallery = ctx.formValues[ctx.fieldPath] as any[] || [];
          await ctx.setFieldValue(ctx.fieldPath, [
            ...currentGallery,
            ...uploads.map(u => ({ upload_id: u.id }))
          ]);
        }

        ctx.notice(`${files.length} files uploaded`);
      }
      break;
  }
}
```

### 4. Field Synchronization

Synchronize values between related fields:

```typescript
async executeFieldDropdownAction(
  actionId: string,
  ctx: ExecuteFieldDropdownActionCtx
): Promise<void> {
  switch (actionId) {
    case 'sync-to-seo':
      const title = ctx.formValues[ctx.fieldPath] as string;
      
      // Update SEO title if it exists and is empty
      if (ctx.formValues.seo_title === '') {
        await ctx.setFieldValue('seo_title', title);
      }

      // Generate meta description from content
      if (ctx.fieldPath === 'content' && ctx.formValues.meta_description === '') {
        const description = (title as string)
          .substring(0, 160)
          .replace(/\s\S*$/, '...');
        await ctx.setFieldValue('meta_description', description);
      }

      ctx.notice('SEO fields synchronized');
      break;

    case 'copy-to-all-locales':
      const value = ctx.formValues[ctx.fieldPath];
      const confirmed = await ctx.openConfirm({
        title: 'Copy to All Locales',
        content: 'This will overwrite the field value in all other locales. Continue?',
        choices: [
          { label: 'Copy', value: true, intent: 'negative' },
          { label: 'Cancel', value: false }
        ]
      });

      if (confirmed) {
        // Apply value to all locales
        for (const locale of ctx.site.attributes.locales) {
          if (locale !== ctx.locale) {
            const localePath = ctx.fieldPath.replace(ctx.locale, locale);
            await ctx.setFieldValue(localePath, value);
          }
        }
        ctx.notice('Copied to all locales');
      }
      break;

    case 'generate-from-title':
      const sourceTitle = ctx.formValues.title as string;
      
      if (!sourceTitle) {
        ctx.alert('Please enter a title first');
        return;
      }

      // Generate slug
      const slug = sourceTitle
        .toLowerCase()
        .replace(/[^\w\s-]/g, '')
        .replace(/\s+/g, '-');

      // Generate excerpt
      const excerpt = `Read about ${sourceTitle} in this comprehensive guide.`;

      // Set multiple field values sequentially
      await ctx.setFieldValue('slug', slug);
      await ctx.setFieldValue('excerpt', excerpt);

      ctx.notice('Fields generated from title');
      break;
  }
}
```

### 5. Data Validation and Formatting

Validate and format field data:

```typescript
async executeFieldDropdownAction(
  actionId: string,
  ctx: ExecuteFieldDropdownActionCtx
): Promise<void> {
  const value = ctx.formValues[ctx.fieldPath];

  switch (actionId) {
    case 'validate-json':
      try {
        const json = JSON.parse(value as string);
        const formatted = JSON.stringify(json, null, 2);
        await ctx.setFieldValue(ctx.fieldPath, formatted);
        ctx.notice('JSON is valid and has been formatted');
      } catch (error) {
        ctx.alert(`Invalid JSON: ${error.message}`);
      }
      break;

    case 'validate-email-list':
      const emails = (value as string).split(/[,;\n]/).map(e => e.trim());
      const invalid: string[] = [];
      const valid: string[] = [];

      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      
      emails.forEach(email => {
        if (email && emailRegex.test(email)) {
          valid.push(email);
        } else if (email) {
          invalid.push(email);
        }
      });

      if (invalid.length > 0) {
        const fix = await ctx.openConfirm({
          title: 'Invalid Emails Found',
          content: `Invalid: ${invalid.join(', ')}\n\nRemove invalid emails?`,
          choices: [
            { label: 'Remove Invalid', value: true },
            { label: 'Keep All', value: false }
          ]
        });

        if (fix) {
          await ctx.setFieldValue(ctx.fieldPath, valid.join(', '));
        }
      } else {
        ctx.notice('All emails are valid');
      }
      break;

    case 'format-phone':
      const phone = (value as string).replace(/\D/g, '');
      
      if (phone.length === 10) {
        const formatted = `(${phone.substr(0, 3)}) ${phone.substr(3, 3)}-${phone.substr(6)}`;
        await ctx.setFieldValue(ctx.fieldPath, formatted);
      } else if (phone.length === 11 && phone.startsWith('1')) {
        const formatted = `+1 (${phone.substr(1, 3)}) ${phone.substr(4, 3)}-${phone.substr(7)}`;
        await ctx.setFieldValue(ctx.fieldPath, formatted);
      } else {
        ctx.alert('Please enter a valid 10-digit phone number');
      }
      break;
  }
}
```

### 6. Integration with External Services

Connect to external APIs and services:

```typescript
async executeFieldDropdownAction(
  actionId: string,
  ctx: ExecuteFieldDropdownActionCtx
): Promise<void> {
  switch (actionId) {
    case 'import-from-wordpress':
      const wpUrl = await ctx.openModal({
        id: 'wp-importer',
        title: 'Import from WordPress',
        width: 's',
        parameters: { fieldPath: ctx.fieldPath }
      });

      if (wpUrl) {
        try {
          const content = await fetchWordPressContent(wpUrl);
          await ctx.setFieldValue(ctx.fieldPath, content);
          ctx.notice('Content imported from WordPress');
        } catch (error) {
          ctx.alert('Failed to import content');
        }
      }
      break;

    case 'preview-markdown':
      const markdown = ctx.formValues[ctx.fieldPath] as string;
      
      await ctx.openModal({
        id: 'markdown-preview',
        title: 'Markdown Preview',
        width: 'l',
        parameters: {
          markdown: markdown,
          theme: ctx.theme
        }
      });
      break;

    case 'check-broken-links':
      const content = ctx.formValues[ctx.fieldPath] as string;
      const links = extractLinks(content);
      
      ctx.notice('Checking links...');
      
      const results = await checkLinks(links);
      const broken = results.filter(r => !r.valid);
      
      if (broken.length > 0) {
        await ctx.openModal({
          id: 'broken-links-report',
          title: 'Broken Links Found',
          width: 'm',
          parameters: { brokenLinks: broken }
        });
      } else {
        ctx.notice('All links are valid!');
      }
      break;
  }
}
```

## Best Practices

1. **Action ID Validation**: Always validate the actionId and handle unknown actions
2. **Error Handling**: Wrap operations in try-catch blocks and provide user feedback
3. **User Confirmation**: Use `openConfirm` for destructive operations
4. **Loading States**: Disable fields during long operations
5. **Undo Support**: Consider implementing undo for complex transformations
6. **Progress Feedback**: Use `notice` and `alert` to keep users informed
7. **Type Safety**: Always type-check field values before operations

## Related Hooks

- **fieldDropdownActions**: Defines the dropdown actions available for fields
- **executeItemFormDropdownAction**: Similar hook for form-level actions
- **executeItemsDropdownAction**: Hook for collection-level actions
- **renderFieldExtension**: Create custom field editors with built-in actions

## Important Notes

- This hook must handle all action IDs defined in `fieldDropdownActions`
- Operations should be idempotent when possible
- Consider field permissions and validation rules
- The hook is async, so long-running operations are supported
- Changes made via `setFieldValue` trigger form validation
- The context provides access to the entire form state, not just the current field