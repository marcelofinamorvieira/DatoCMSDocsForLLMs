# renderFieldExtension Hook

## Purpose

The `renderFieldExtension` hook is responsible for rendering custom field editors and addons in DatoCMS. This is one of the most important hooks as it allows you to create completely custom field editing experiences, from simple text inputs with special formatting to complex editors with external data integration. It works in conjunction with `manualFieldExtensions` and `overrideFieldExtensions` to provide custom field UIs.

## Signature

```typescript
renderFieldExtension(
  fieldExtensionId: string,
  ctx: RenderFieldExtensionCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

- `fieldExtensionId`: string - The unique identifier of the field extension to render
- `ctx`: RenderFieldExtensionCtx - The context object containing all necessary data and methods

## Type Definitions

```typescript
// Hook signature
export type RenderFieldExtensionHook = {
  /**
   * This function will be called when the plugin needs to render a field
   * extension (see the `manualFieldExtensions` and `overrideFieldExtensions`
   * functions)
   *
   * @tag forcedFieldExtensions
   */
  renderFieldExtension: (
    fieldExtensionId: string,
    ctx: RenderFieldExtensionCtx,
  ) => void;
};

// Main context type
export type RenderFieldExtensionCtx = SelfResizingPluginFrameCtx<
  'renderFieldExtension',
  ItemFormAdditionalProperties &
    FieldAdditionalProperties & {
      /** The ID of the field extension that needs to be rendered */
      fieldExtensionId: string;
      /** The arbitrary `parameters` of the field extension */
      parameters: Record<string, unknown>;
    },
  ItemFormAdditionalMethods
>;

// Expanded full context type
type RenderFieldExtensionCtx = {
  // Mode and frame properties
  mode: 'renderFieldExtension';
  bodyPadding: [number, number, number, number];
  
  // Field extension specific
  fieldExtensionId: string;
  parameters: Record<string, unknown>;
  
  // Field properties
  disabled: boolean;
  fieldPath: string;
  field: Field;
  parentField: Field | undefined;
  block: undefined | { id: string | undefined; blockModel: ItemType };
  
  // Item form properties
  locale: string;
  item: Item | null;
  itemId: string | null;
  itemType: ItemType;
  formValues: Record<string, unknown>;
  itemStatus: 'new' | 'draft' | 'updated' | 'published';
  isSubmitting: boolean;
  isFormDirty: boolean;
  isCollapsedBlock: boolean;
  blocksAnalysis: BlocksAnalysis;
  fieldHavingError: string[];
  
  // Localization
  locales: string[];
  internalLocales: InternalLocale[];
  lastPublishedVersion: Item | null;
  
  // Base properties
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken: string | undefined;
  site: Site;
  environment: string;
  environmentId: string;
  isEnvironmentPrimary: boolean;
  mainLocale: string;
  currentUserMenuLocale: string;
  owner: Account | Organization;
  account: Account | undefined; // deprecated
  ui: { locale: string };
  theme: Theme;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  users: Partial<Record<string, User>>;
  ssoUsers: Partial<Record<string, SsoUser>>;
  
  // Base methods
  loadItemTypeFields: (itemTypeId: string) => Promise<Field[]>;
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<Fieldset[]>;
  loadFieldsUsingPlugin: () => Promise<Field[]>;
  loadUsers: () => Promise<User[]>;
  loadSsoUsers: () => Promise<SsoUser[]>;
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  updateFieldAppearance: (fieldId: string, changes: FieldAppearanceChange[]) => Promise<void>;
  getFieldIntlTitle: (field: Field, locale?: string, returnUntranslatedTitle?: boolean) => string | null;
  createClient: (options?: { apiToken?: string; environment?: string }) => SiteApiClient;
  
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
  editUploadMetadata: (fileFieldValue: FileFieldValue, locale?: string) => Promise<FileFieldValue | null>;
  
  // Notifications
  alert: (message: string) => Promise<void>;
  notice: (message: string) => Promise<void>;
  customToast: <CtaValue = unknown>(toast: Toast<CtaValue>) => Promise<CtaValue | null>;
  
  // Modals and navigation
  openModal: (modal: Modal) => Promise<unknown>;
  openConfirm: (options: ConfirmOptions) => Promise<unknown>;
  navigateTo: (path: string) => Promise<void>;
  
  // Item form methods
  toggleField: (path: string, show: boolean) => Promise<void>;
  disableField: (path: string, disable: boolean) => Promise<void>;
  scrollToField: (path: string, locale?: string) => Promise<void>;
  setFieldValue: (path: string, value: unknown) => Promise<void>;
  setMultipleFieldValues: (values: Record<string, unknown>) => Promise<void>;
  setFieldError: (path: string, error: string | null) => Promise<void>;
  getFieldValue: (path: string) => unknown;
  isFieldDisabled: (path: string) => boolean;
  isFieldVisible: (path: string) => boolean;
  formValuesToItem: (
    formValues: Record<string, unknown>,
    skipUnchangedFields?: boolean,
  ) => Promise<Omit<Item, 'id' | 'meta'> | undefined>;
  itemToFormValues: (
    item: Omit<Item, 'id' | 'meta'>,
  ) => Promise<Record<string, unknown>>;
  saveCurrentItem: (showToast?: boolean) => Promise<void>;
  
  // Sizing utilities
  startAutoResizer: () => Promise<void>;
  stopAutoResizer: () => Promise<void>;
  updateHeight: (height: number) => Promise<void>;
  isAutoResizerActive: () => boolean;
  setHeight: (height: number) => Promise<void>;
  
  // Frame communication
  getSettings: () => Promise<RenderFieldExtensionCtx>;
};

// Supporting types
type Theme = {
  primaryColor: string;
  accentColor: string;
  semiTransparentAccentColor: string;
  lightColor: string;
  darkColor: string;
};

type BlocksAnalysis = {
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

type InternalLocale = {
  id: string;
  name: string;
  code: string;
};

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

type Modal = {
  id: string;
  title?: string;
  closeDisabled?: boolean;
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  parameters?: Record<string, unknown>;
  initialHeight?: number;
};

type ConfirmOptions = {
  title: string;
  content: string;
  choices: ConfirmChoice[];
  cancel: ConfirmChoice;
};

type ConfirmChoice = {
  label: string;
  value: unknown;
  intent?: 'positive' | 'negative';
};

type FileFieldValue = {
  upload_id: string;
  alt: string | null;
  title: string | null;
  focal_point: FocalPoint | null;
  custom_data: Record<string, string>;
};

type FocalPoint = {
  x: number;
  y: number;
};

type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};

type FieldAppearanceChange =
  | { operation: 'removeEditor' }
  | { operation: 'updateEditor'; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'setEditor'; fieldExtensionId: string; parameters: Record<string, unknown> }
  | { operation: 'removeAddon'; index: number }
  | { operation: 'updateAddon'; index: number; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'insertAddon'; index: number; fieldExtensionId: string; parameters: Record<string, unknown> };
```

## Context Properties

The `renderFieldExtension` hook receives a rich context object that extends the base plugin context with field-specific and form-specific properties:

### Base Properties
All standard context properties are available:
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `environment`: string - The ID of the current environment
- `theme`: Theme - Theme colors for the current project
- `itemTypes`: Partial<Record<string, ItemType>> - Models indexed by ID
- `fields`: Partial<Record<string, Field>> - Fields indexed by ID

### Field-Specific Properties
- `fieldExtensionId`: string - The ID of the field extension being rendered
- `parameters`: Record<string, unknown> - Configuration parameters for this field extension instance
- `field`: Field - The field where the extension is installed
- `fieldPath`: string - The path in the formValues object to access the field value (e.g., "title" or "blocks[0].text")
- `disabled`: boolean - Whether the field is currently disabled
- `parentField`: Field | undefined - If in a block, the parent Modular Content/Structured Text field
- `block`: { id: string | undefined; blockModel: ItemType } | undefined - Block information if nested

### Item Form Properties
- `item`: Item | null - The current item being edited (null for new items)
- `itemId`: string | null - The ID of the current item
- `itemType`: ItemType - The model of the current item
- `itemStatus`: 'new' | 'draft' | 'updated' | 'published' - Current item status
- `isSubmitting`: boolean - Whether the form is currently being submitted
- `isFormDirty`: boolean - Whether the form has unsaved changes
- `blocksAnalysis`: Record<string, BlocksAnalysis> - Analysis of blocks in modular content fields

### Form State Properties
- `formValues`: Record<string, unknown> - All current form values
- `fieldHavingError`: string[] - List of field paths with validation errors
- `locale`: string - Current editing locale
- `locales`: string[] - All available locales
- `internalLocales`: InternalLocale[] - Detailed locale information
- `lastPublishedVersion`: Item | null - The last published version of the item

### Methods
All standard context methods plus field-specific methods:
- `setFieldValue(fieldPath: string, value: unknown)`: void - Update a field's value
- `setMultipleFieldValues(values: Record<string, unknown>)`: void - Update multiple fields
- `setFieldError(fieldPath: string, error: string | null)`: void - Set validation error
- `scrollToField(fieldPath: string)`: void - Scroll to a specific field
- `toggleField(fieldPath: string, show: boolean)`: void - Show/hide a field
- `disableField(fieldPath: string, disable: boolean)`: void - Enable/disable a field
- `getFieldValue(fieldPath: string)`: unknown - Get a field's current value
- `isFieldDisabled(fieldPath: string)`: boolean - Check if a field is disabled
- `isFieldVisible(fieldPath: string)`: boolean - Check if a field is visible

### Sizing and Layout
- `self`: PluginFrameRecord<'renderFieldExtension'> - Frame information
- `setHeight(height: number)`: void - Set the plugin frame height
- `startAutoResizer()`: void - Start automatic height adjustment
- `stopAutoResizer()`: void - Stop automatic height adjustment
- `updateHeight(height: number)`: void - Update frame height

## Use Cases

### 1. Custom Text Editor with Formatting

Create a rich text editor with custom formatting options:

```typescript
async renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
  if (fieldExtensionId === 'richTextEditor') {
    return {
      render: () => import('./components/RichTextEditor'),
    };
  }
}

// RichTextEditor.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, ButtonGroup } from 'datocms-react-ui';
import { useState, useRef } from 'react';

export default function RichTextEditor({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [content, setContent] = useState(ctx.formValues[ctx.fieldPath] || '');
  const editorRef = useRef<HTMLDivElement>(null);

  const applyFormat = (format: string) => {
    document.execCommand(format, false);
    updateContent();
  };

  const updateContent = () => {
    if (editorRef.current) {
      const newContent = editorRef.current.innerHTML;
      setContent(newContent);
      ctx.setFieldValue(ctx.fieldPath, newContent);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', flexDirection: 'column', gap: '10px' }}>
        <ButtonGroup>
          <Button onClick={() => applyFormat('bold')} buttonSize="s">Bold</Button>
          <Button onClick={() => applyFormat('italic')} buttonSize="s">Italic</Button>
          <Button onClick={() => applyFormat('underline')} buttonSize="s">Underline</Button>
        </ButtonGroup>
        
        <div
          ref={editorRef}
          contentEditable={!ctx.disabled}
          onInput={updateContent}
          dangerouslySetInnerHTML={{ __html: content }}
          style={{
            border: '1px solid #ddd',
            borderRadius: '4px',
            padding: '12px',
            minHeight: '100px',
            backgroundColor: ctx.disabled ? '#f5f5f5' : '#fff',
          }}
        />
        
        {ctx.field.attributes.validators.length?.max && (
          <div style={{ fontSize: '12px', color: '#666' }}>
            {content.length} / {ctx.field.attributes.validators.length.max} characters
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

### 2. External API Data Selector

Create a field that fetches and displays data from an external API:

```typescript
async renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
  if (fieldExtensionId === 'apiDataSelector') {
    return {
      render: () => import('./components/ApiDataSelector'),
    };
  }
}

// ApiDataSelector.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, SelectField, Spinner } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

interface ApiOption {
  id: string;
  name: string;
  description?: string;
}

export default function ApiDataSelector({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [options, setOptions] = useState<ApiOption[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  // Access configuration from field parameters
  const apiEndpoint = ctx.parameters.apiEndpoint as string;
  const valueField = ctx.parameters.valueField as string || 'id';
  const labelField = ctx.parameters.labelField as string || 'name';
  const apiKey = ctx.plugin.attributes.parameters.apiKey;

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(apiEndpoint, {
          headers: {
            'Authorization': `Bearer ${apiKey}`,
            'Content-Type': 'application/json',
          },
        });
        
        if (!response.ok) {
          throw new Error(`API returned ${response.status}`);
        }
        
        const data = await response.json();
        setOptions(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Failed to fetch data');
        ctx.alert('Failed to load options from API');
      } finally {
        setLoading(false);
      }
    };

    if (apiEndpoint) {
      fetchData();
    }
  }, [apiEndpoint, apiKey]);

  if (loading) {
    return (
      <Canvas ctx={ctx}>
        <div style={{ padding: '20px', textAlign: 'center' }}>
          <Spinner />
          <p>Loading options...</p>
        </div>
      </Canvas>
    );
  }

  const selectOptions = options.map(option => ({
    label: option[labelField] || option.name,
    value: option[valueField] || option.id,
  }));

  return (
    <Canvas ctx={ctx}>
      <SelectField
        id={ctx.fieldPath}
        name={ctx.fieldPath}
        label={ctx.field.attributes.label}
        value={ctx.formValues[ctx.fieldPath]}
        hint={ctx.field.attributes.hint}
        error={error}
        required={ctx.field.attributes.validators.required}
        selectInputProps={{
          isDisabled: ctx.disabled,
          options: selectOptions,
          onChange: (newValue) => {
            ctx.setFieldValue(ctx.fieldPath, newValue);
          },
          placeholder: 'Select an option...',
        }}
      />
    </Canvas>
  );
}
```

### 3. Color Picker with Presets

Create a custom color picker with predefined brand colors:

```typescript
async renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
  if (fieldExtensionId === 'brandColorPicker') {
    return {
      render: () => import('./components/BrandColorPicker'),
    };
  }
}

// BrandColorPicker.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, TextField } from 'datocms-react-ui';
import { useState } from 'react';

export default function BrandColorPicker({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [color, setColor] = useState(ctx.formValues[ctx.fieldPath] || '#000000');
  
  // Brand colors from parameters or defaults
  const brandColors = ctx.parameters.brandColors as string[] || [
    '#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8', '#F7DC6F'
  ];

  const handleColorChange = (newColor: string) => {
    setColor(newColor);
    ctx.setFieldValue(ctx.fieldPath, newColor);
    
    // Validate hex format
    if (!/^#[0-9A-F]{6}$/i.test(newColor)) {
      ctx.setFieldError(ctx.fieldPath, 'Please enter a valid hex color');
    } else {
      ctx.setFieldError(ctx.fieldPath, null);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', flexDirection: 'column', gap: '16px' }}>
        <TextField
          id="colorInput"
          name="colorInput"
          label={ctx.field.attributes.label}
          value={color}
          onChange={handleColorChange}
          placeholder="#000000"
          hint="Enter a hex color code"
          required={ctx.field.attributes.validators.required}
          textInputProps={{
            disabled: ctx.disabled,
            style: {
              borderLeft: `40px solid ${color}`,
            },
          }}
        />
        
        <div>
          <p style={{ fontSize: '14px', marginBottom: '8px' }}>Brand Colors:</p>
          <div style={{ display: 'flex', gap: '8px', flexWrap: 'wrap' }}>
            {brandColors.map((brandColor) => (
              <Button
                key={brandColor}
                onClick={() => handleColorChange(brandColor)}
                buttonSize="s"
                style={{
                  backgroundColor: brandColor,
                  width: '40px',
                  height: '40px',
                  padding: 0,
                  border: color === brandColor ? '2px solid #000' : 'none',
                }}
                aria-label={`Select color ${brandColor}`}
              />
            ))}
          </div>
        </div>
        
        <div style={{ display: 'flex', alignItems: 'center', gap: '12px' }}>
          <div
            style={{
              width: '60px',
              height: '60px',
              backgroundColor: color,
              borderRadius: '4px',
              border: '1px solid #ddd',
            }}
          />
          <div>
            <p style={{ fontSize: '14px', fontWeight: 'bold' }}>Preview</p>
            <p style={{ fontSize: '12px', color: '#666' }}>{color}</p>
          </div>
        </div>
      </div>
    </Canvas>
  );
}
```

### 4. Multi-field Coordinator

Create a field extension that coordinates values across multiple fields:

```typescript
async renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
  if (fieldExtensionId === 'addressCoordinator') {
    return {
      render: () => import('./components/AddressCoordinator'),
    };
  }
}

// AddressCoordinator.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

interface AddressComponents {
  street: string;
  city: string;
  state: string;
  zip: string;
  country: string;
}

export default function AddressCoordinator({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [isExpanded, setIsExpanded] = useState(false);
  const fullAddress = ctx.formValues[ctx.fieldPath] as string || '';
  
  // Parse address components from the full address
  const parseAddress = (address: string): AddressComponents => {
    const parts = address.split(', ');
    return {
      street: parts[0] || '',
      city: parts[1] || '',
      state: parts[2] || '',
      zip: parts[3] || '',
      country: parts[4] || '',
    };
  };
  
  const [components, setComponents] = useState<AddressComponents>(
    parseAddress(fullAddress)
  );

  const updateFullAddress = (newComponents: AddressComponents) => {
    const { street, city, state, zip, country } = newComponents;
    const nonEmpty = [street, city, state, zip, country].filter(Boolean);
    const newAddress = nonEmpty.join(', ');
    
    ctx.setFieldValue(ctx.fieldPath, newAddress);
    
    // Also update related fields if they exist
    if (ctx.formValues.city !== undefined) {
      ctx.setMultipleFieldValues({
        city: city,
        state: state,
        zip: zip,
        country: country,
      });
    }
  };

  const handleComponentChange = (field: keyof AddressComponents, value: string) => {
    const newComponents = { ...components, [field]: value };
    setComponents(newComponents);
    updateFullAddress(newComponents);
  };

  const lookupAddress = async () => {
    // Example: Geocoding API integration
    if (components.zip && ctx.parameters.geocodingApiKey) {
      try {
        const response = await fetch(
          `https://api.geocoding.example.com/zip/${components.zip}?key=${ctx.parameters.geocodingApiKey}`
        );
        const data = await response.json();
        
        const newComponents = {
          ...components,
          city: data.city,
          state: data.state,
          country: data.country,
        };
        
        setComponents(newComponents);
        updateFullAddress(newComponents);
        ctx.notice('Address components updated from ZIP code');
      } catch (error) {
        ctx.alert('Failed to lookup address');
      }
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', flexDirection: 'column', gap: '12px' }}>
        <TextField
          id="fullAddress"
          name="fullAddress"
          label={ctx.field.attributes.label}
          value={fullAddress}
          onChange={(value) => {
            ctx.setFieldValue(ctx.fieldPath, value);
            setComponents(parseAddress(value));
          }}
          hint="Full address or use the form below"
          required={ctx.field.attributes.validators.required}
          textInputProps={{
            disabled: ctx.disabled,
          }}
        />
        
        <Button
          onClick={() => setIsExpanded(!isExpanded)}
          buttonType="muted"
          buttonSize="s"
        >
          {isExpanded ? 'Hide' : 'Show'} Address Components
        </Button>
        
        {isExpanded && (
          <div style={{ 
            padding: '16px', 
            backgroundColor: '#f8f9fa', 
            borderRadius: '4px',
            display: 'flex',
            flexDirection: 'column',
            gap: '12px'
          }}>
            <TextField
              id="street"
              name="street"
              label="Street"
              value={components.street}
              onChange={(value) => handleComponentChange('street', value)}
              textInputProps={{ disabled: ctx.disabled }}
            />
            
            <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '12px' }}>
              <TextField
                id="city"
                name="city"
                label="City"
                value={components.city}
                onChange={(value) => handleComponentChange('city', value)}
                textInputProps={{ disabled: ctx.disabled }}
              />
              
              <TextField
                id="state"
                name="state"
                label="State"
                value={components.state}
                onChange={(value) => handleComponentChange('state', value)}
                textInputProps={{ disabled: ctx.disabled }}
              />
            </div>
            
            <div style={{ display: 'grid', gridTemplateColumns: '1fr 2fr', gap: '12px' }}>
              <TextField
                id="zip"
                name="zip"
                label="ZIP"
                value={components.zip}
                onChange={(value) => handleComponentChange('zip', value)}
                textInputProps={{ disabled: ctx.disabled }}
              />
              
              <TextField
                id="country"
                name="country"
                label="Country"
                value={components.country}
                onChange={(value) => handleComponentChange('country', value)}
                textInputProps={{ disabled: ctx.disabled }}
              />
            </div>
            
            {ctx.parameters.geocodingApiKey && (
              <Button
                onClick={lookupAddress}
                buttonSize="s"
                disabled={!components.zip || ctx.disabled}
              >
                Lookup from ZIP
              </Button>
            )}
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

### 5. Conditional Field Display

Create a field that shows/hides based on other field values:

```typescript
async renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
  if (fieldExtensionId === 'conditionalField') {
    return {
      render: () => import('./components/ConditionalField'),
    };
  }
}

// ConditionalField.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField, SelectField } from 'datocms-react-ui';
import { useEffect } from 'react';

export default function ConditionalField({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  // Configuration for conditional logic
  const dependsOnField = ctx.parameters.dependsOnField as string;
  const showWhenValue = ctx.parameters.showWhenValue as string;
  const hideOtherFields = ctx.parameters.hideOtherFields as string[] || [];
  
  // Get the value of the field we depend on
  const dependentValue = ctx.formValues[dependsOnField];
  const shouldShow = dependentValue === showWhenValue;
  
  useEffect(() => {
    // Hide this field if condition is not met
    ctx.toggleField(ctx.fieldPath, shouldShow);
    
    // Optionally hide/show other related fields
    hideOtherFields.forEach(fieldPath => {
      ctx.toggleField(fieldPath, !shouldShow);
    });
    
    // Clear value when hidden
    if (!shouldShow && ctx.formValues[ctx.fieldPath]) {
      ctx.setFieldValue(ctx.fieldPath, null);
    }
  }, [dependentValue, shouldShow]);
  
  // Don't render anything if we shouldn't show
  if (!shouldShow) {
    return null;
  }
  
  // Example: Show different input based on another field's value
  const inputType = ctx.formValues.input_type as string;
  
  if (inputType === 'select') {
    return (
      <Canvas ctx={ctx}>
        <SelectField
          id={ctx.fieldPath}
          name={ctx.fieldPath}
          label={ctx.field.attributes.label}
          value={ctx.formValues[ctx.fieldPath]}
          hint="This field is shown because the condition was met"
          selectInputProps={{
            options: [
              { label: 'Option A', value: 'a' },
              { label: 'Option B', value: 'b' },
              { label: 'Option C', value: 'c' },
            ],
            onChange: (value) => ctx.setFieldValue(ctx.fieldPath, value),
          }}
        />
      </Canvas>
    );
  }
  
  return (
    <Canvas ctx={ctx}>
      <TextField
        id={ctx.fieldPath}
        name={ctx.fieldPath}
        label={ctx.field.attributes.label}
        value={ctx.formValues[ctx.fieldPath] || ''}
        onChange={(value) => ctx.setFieldValue(ctx.fieldPath, value)}
        hint="This field is shown because the condition was met"
        required={ctx.field.attributes.validators.required}
      />
    </Canvas>
  );
}
```

### 6. Field with Live Validation

Create a field with real-time validation and feedback:

```typescript
async renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
  if (fieldExtensionId === 'validatedUrlField') {
    return {
      render: () => import('./components/ValidatedUrlField'),
    };
  }
}

// ValidatedUrlField.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField, Spinner } from 'datocms-react-ui';
import { useState, useCallback } from 'react';
import debounce from 'lodash.debounce';

export default function ValidatedUrlField({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [url, setUrl] = useState(ctx.formValues[ctx.fieldPath] as string || '');
  const [validating, setValidating] = useState(false);
  const [validationResult, setValidationResult] = useState<{
    valid: boolean;
    message: string;
    details?: any;
  } | null>(null);

  const validateUrl = async (urlToValidate: string) => {
    if (!urlToValidate) {
      setValidationResult(null);
      ctx.setFieldError(ctx.fieldPath, null);
      return;
    }

    setValidating(true);
    
    try {
      // Basic URL validation
      const urlObj = new URL(urlToValidate);
      
      // Check if URL is reachable
      const response = await fetch(`/api/validate-url?url=${encodeURIComponent(urlToValidate)}`);
      const result = await response.json();
      
      if (result.valid) {
        setValidationResult({
          valid: true,
          message: 'URL is valid and reachable',
          details: {
            statusCode: result.statusCode,
            contentType: result.contentType,
            title: result.pageTitle,
          },
        });
        ctx.setFieldError(ctx.fieldPath, null);
      } else {
        setValidationResult({
          valid: false,
          message: result.error || 'URL is not reachable',
        });
        ctx.setFieldError(ctx.fieldPath, result.error);
      }
    } catch (error) {
      setValidationResult({
        valid: false,
        message: 'Invalid URL format',
      });
      ctx.setFieldError(ctx.fieldPath, 'Please enter a valid URL');
    } finally {
      setValidating(false);
    }
  };

  // Debounce validation to avoid too many requests
  const debouncedValidate = useCallback(
    debounce(validateUrl, 500),
    []
  );

  const handleChange = (value: string) => {
    setUrl(value);
    ctx.setFieldValue(ctx.fieldPath, value);
    debouncedValidate(value);
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', flexDirection: 'column', gap: '12px' }}>
        <TextField
          id={ctx.fieldPath}
          name={ctx.fieldPath}
          label={ctx.field.attributes.label}
          value={url}
          onChange={handleChange}
          placeholder="https://example.com"
          hint={ctx.field.attributes.hint}
          required={ctx.field.attributes.validators.required}
          error={validationResult && !validationResult.valid ? validationResult.message : undefined}
          textInputProps={{
            disabled: ctx.disabled,
          }}
          suffix={validating && <Spinner size={16} />}
        />
        
        {validationResult && validationResult.valid && validationResult.details && (
          <div style={{
            padding: '12px',
            backgroundColor: '#e7f5e7',
            borderRadius: '4px',
            fontSize: '13px',
          }}>
            <p style={{ color: '#2e7d2e', fontWeight: 'bold', marginBottom: '4px' }}>
              ✓ {validationResult.message}
            </p>
            {validationResult.details.title && (
              <p style={{ color: '#666' }}>
                Page title: {validationResult.details.title}
              </p>
            )}
            <p style={{ color: '#666' }}>
              Status: {validationResult.details.statusCode} • 
              Type: {validationResult.details.contentType}
            </p>
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

## Integration with React and datocms-react-ui Components

The `renderFieldExtension` hook is designed to work seamlessly with the `datocms-react-ui` component library:

### Using Canvas Component

Always wrap your field extension in the `Canvas` component to ensure proper styling and theme integration:

```typescript
import { Canvas } from 'datocms-react-ui';

export default function MyFieldExtension({ ctx }: Props) {
  return (
    <Canvas ctx={ctx}>
      {/* Your field implementation */}
    </Canvas>
  );
}
```

### Available UI Components

The `datocms-react-ui` library provides a comprehensive set of components:

- **Form Components**: TextField, TextareaField, SelectField, SwitchField
- **Layout Components**: Form, FieldGroup, Section, SplitView, VerticalSplit
- **Feedback Components**: FieldError, FieldHint, Spinner
- **Interactive Components**: Button, ButtonGroup, Dropdown
- **Display Components**: ContextInspector, SidebarPanel

### Maintaining Consistent Styling

```typescript
import { TextField, Button, FieldGroup } from 'datocms-react-ui';
import 'datocms-react-ui/styles'; // Import default styles

export default function StyledFieldExtension({ ctx }: Props) {
  return (
    <Canvas ctx={ctx}>
      <Form>
        <FieldGroup>
          <TextField
            id="field1"
            name="field1"
            label="Field 1"
            value={value1}
            onChange={handleChange1}
          />
          <TextField
            id="field2"
            name="field2"
            label="Field 2"
            value={value2}
            onChange={handleChange2}
          />
        </FieldGroup>
        
        <Button onClick={handleSubmit} buttonType="primary">
          Save
        </Button>
      </Form>
    </Canvas>
  );
}
```

## Best Practices for Performance

### 1. Lazy Loading

Use dynamic imports for better initial load performance:

```typescript
renderFieldExtension(fieldExtensionId: string) {
  switch (fieldExtensionId) {
    case 'heavyEditor':
      return {
        render: () => import('./components/HeavyEditor'),
      };
    case 'lightEditor':
      return {
        render: () => import('./components/LightEditor'),
      };
  }
}
```

### 2. Memoization

Use React hooks to prevent unnecessary re-renders:

```typescript
import { useMemo, useCallback, memo } from 'react';

const MyFieldExtension = memo(({ ctx }: Props) => {
  const expensiveOptions = useMemo(() => {
    return generateOptions(ctx.field.attributes);
  }, [ctx.field.attributes]);

  const handleChange = useCallback((value: string) => {
    ctx.setFieldValue(ctx.fieldPath, value);
  }, [ctx.fieldPath]);

  return (
    <Canvas ctx={ctx}>
      {/* Component implementation */}
    </Canvas>
  );
});
```

### 3. Debouncing Updates

Prevent excessive API calls or validations:

```typescript
import { useMemo } from 'react';
import debounce from 'lodash.debounce';

export default function DebouncedField({ ctx }: Props) {
  const debouncedSetValue = useMemo(
    () => debounce((value: string) => {
      ctx.setFieldValue(ctx.fieldPath, value);
    }, 300),
    [ctx.fieldPath]
  );

  return (
    <Canvas ctx={ctx}>
      <input
        type="text"
        onChange={(e) => debouncedSetValue(e.target.value)}
      />
    </Canvas>
  );
}
```

### 4. Virtual Scrolling

For fields displaying large lists:

```typescript
import { FixedSizeList } from 'react-window';

export default function VirtualizedListField({ ctx }: Props) {
  const items = ctx.parameters.items as string[] || [];
  
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index]}
    </div>
  );

  return (
    <Canvas ctx={ctx}>
      <FixedSizeList
        height={400}
        itemCount={items.length}
        itemSize={35}
        width="100%"
      >
        {Row}
      </FixedSizeList>
    </Canvas>
  );
}
```

### 5. Conditional Rendering

Avoid rendering complex components when not needed:

```typescript
export default function ConditionalComplexField({ ctx }: Props) {
  const [showAdvanced, setShowAdvanced] = useState(false);
  
  return (
    <Canvas ctx={ctx}>
      <Button onClick={() => setShowAdvanced(!showAdvanced)}>
        {showAdvanced ? 'Hide' : 'Show'} Advanced Options
      </Button>
      
      {showAdvanced && (
        <Suspense fallback={<Spinner />}>
          <AdvancedOptions ctx={ctx} />
        </Suspense>
      )}
    </Canvas>
  );
}
```

## Related Hooks

- **manualFieldExtensions**: Define which field extensions your plugin provides
- **overrideFieldExtensions**: Override default field editors with custom ones
- **validateManualFieldExtensionParameters**: Validate configuration for field extensions
- **renderManualFieldExtensionConfigScreen**: Render configuration UI for field extensions
- **onBeforeItemUpsert**: Perform validation or transformation before saving
- **fieldDropdownActions**: Add custom actions to field dropdowns

## Important Notes

- Field extensions must handle all field types they declare support for
- Always use the `Canvas` component as the root element for proper theming
- The context provides both read and write access to form values
- Field extensions can interact with other fields through `setMultipleFieldValues`
- Use `startAutoResizer()` for dynamically sized content
- Handle the `disabled` state appropriately to prevent edits when needed
- Field extensions in blocks have access to both the block and parent field context
- Validation errors set with `setFieldError` will prevent form submission
- The plugin frame automatically adjusts height, but you can control it manually
- Field extensions should be responsive and work well at different viewport sizes

