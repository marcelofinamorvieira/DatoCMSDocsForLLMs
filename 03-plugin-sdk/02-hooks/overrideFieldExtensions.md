# overrideFieldExtensions Hook

The `overrideFieldExtensions` hook allows you to override the default field editors in DatoCMS with custom implementations. This hook enables you to replace the standard UI for specific field types across your entire project, providing a consistent custom editing experience.

## Quick Reference

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  overrideFieldExtensions(field, ctx) {
    // Override color fields with custom picker
    if (field.attributes.field_type === 'color') {
      return {
        editor: { id: 'myColorPicker' }
      };
    }
    
    // Override JSON fields with code editor
    if (field.attributes.field_type === 'json') {
      return {
        editor: { id: 'myJsonEditor' }
      };
    }
  }
});
```

## Purpose

The `overrideFieldExtensions` hook enables:
- Replacing default field editors with custom implementations
- Providing consistent editing experience across all fields of a type
- Adding specialized editors for specific field types
- Enhancing user experience with richer interfaces
- Implementing company-specific UI standards

## Signature

```typescript
overrideFieldExtensions(
  field: Field,
  ctx: OverrideFieldExtensionsCtx
): FieldExtensionOverride | undefined
```

## Parameters

### field: Field
The field being rendered, containing:
- `id`: string - The field's unique identifier
- `type`: 'field' 
- `attributes`:
  - `label`: string - Field label
  - `field_type`: FieldType - The type of field
  - `api_key`: string - Field's API key
  - `hint`: string | null - Help text
  - `localized`: boolean - Whether field is localized
  - `validators`: object - Validation rules
  - `appearance`: object - UI configuration
  - `position`: number - Order in form

### ctx: OverrideFieldExtensionsCtx
Context object containing:
- `itemType`: ItemType - The model this field belongs to
- All standard context properties (plugin, user, site, etc.)

## Return Value

Returns a `FieldExtensionOverride` object or `undefined`:

```typescript
export type FieldExtensionOverride = {
  editor?: EditorOverride;
  addons?: AddonOverride[];
};

export type EditorOverride = {
  id: string;
  asSidebarPanel?:
    | boolean
    | { startOpen?: boolean; placement?: ItemFormSidebarPanelPlacement };
  parameters?: Record<string, unknown>;
  rank?: number;
  initialHeight?: number;
};

export type AddonOverride = {
  id: string;
  parameters?: Record<string, unknown>;
  rank?: number;
  initialHeight?: number;
};

type ItemFormSidebarPanelPlacement = [
  'before' | 'after',
  'info' | 'publishedVersion' | 'schedule' | 'links' | 'history',
];

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
  | 'seo'
  | 'slug'
  | 'string'
  | 'text'
  | 'video'
  | 'structured_text';
```

## Type Definitions

```typescript
// Hook signature
export type OverrideFieldExtensionsHook = {
  /**
   * Use this function to automatically force one or more field extensions
   * to a particular field
   *
   * @tag fieldExtensions
   */
  overrideFieldExtensions: (
    field: Field,
    ctx: OverrideFieldExtensionsCtx,
  ) => FieldExtensionOverride | undefined;
};

// Context type - includes itemType in addition to base context
export type OverrideFieldExtensionsCtx = Ctx & {
  /** The model containing this field */
  itemType: ItemType;
};

// Field extension override type
export type FieldExtensionOverride = {
  /** Override the field editor */
  editor?: EditorOverride;
  /** Add or override field addons */
  addons?: AddonOverride[];
};

// Editor override type
export type EditorOverride = {
  /** ID of the field extension to use as editor */
  id: string;
  /** Show editor as sidebar panel instead of inline */
  asSidebarPanel?:
    | boolean
    | { 
        /** Whether panel starts open */
        startOpen?: boolean; 
        /** Where to place the panel */
        placement?: ItemFormSidebarPanelPlacement;
      };
  /** Parameters to pass to the field extension */
  parameters?: Record<string, unknown>;
  /** Priority when multiple overrides exist */
  rank?: number;
  /** Initial iframe height */
  initialHeight?: number;
};

// Addon override type
export type AddonOverride = {
  /** ID of the field extension to use as addon */
  id: string;
  /** Parameters to pass to the addon */
  parameters?: Record<string, unknown>;
  /** Priority when multiple addons exist */
  rank?: number;
  /** Initial iframe height */
  initialHeight?: number;
};

// Placement type for sidebar panels
type ItemFormSidebarPanelPlacement = [
  'before' | 'after',
  'info' | 'publishedVersion' | 'schedule' | 'links' | 'history',
];

// Field type
type Field = {
  id: string;
  type: 'field';
  attributes: {
    /** Field label */
    label: string;
    /** Field API key */
    api_key: string;
    /** Field type */
    field_type: FieldType;
    /** Whether field is localized */
    localized: boolean;
    /** Field validators */
    validators: Record<string, any>;
    /** Field appearance configuration */
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      addons: Array<{
        id: string;
        parameters: Record<string, any>;
      }>;
    };
    /** Field position in form */
    position: number;
    /** Field hint text */
    hint: string | null;
    /** Whether field is required */
    required: boolean;
    // ... other attributes
  };
  relationships: {
    /** The fieldset this field belongs to (if any) */
    fieldset?: {
      data: {
        id: string;
        type: 'fieldset';
      } | null;
    };
    /** The model this field belongs to */
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
  };
};

// ItemType (Model) type
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    /** Model name */
    name: string;
    /** Model API key */
    api_key: string;
    /** Whether model is singleton */
    singleton: boolean;
    /** Whether model is modular block */
    modular_block: boolean;
    /** Model ordering configuration */
    ordering_direction: 'asc' | 'desc' | null;
    ordering_field: string | null;
    /** Whether model has tree structure */
    tree: boolean;
    /** Draft/published system */
    draft_mode_active: boolean;
    // ... other attributes
  };
  relationships: {
    /** Fields belonging to this model */
    fields: {
      data: Array<{
        id: string;
        type: 'field';
      }>;
    };
    /** Fieldsets belonging to this model */
    fieldsets: {
      data: Array<{
        id: string;
        type: 'fieldset';
      }>;
    };
    /** Singleton item (if applicable) */
    singleton_item?: {
      data: {
        id: string;
        type: 'item';
      } | null;
    };
  };
};

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

// Fieldset type
type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    collapsible: boolean;
    start_collapsed: boolean;
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

// Type guard functions
export function isFieldExtensionOverride(
  value: unknown,
): value is FieldExtensionOverride {
  return (
    isRecord(value) &&
    (isNullish(value.editor) || isEditorOverride(value.editor)) &&
    (isNullish(value.addons) || isArray(value.addons, isAddonOverride))
  );
}

export function isEditorOverride(value: unknown): value is EditorOverride {
  return (
    isRecord(value) &&
    isString(value.id) &&
    (isNullish(value.parameters) || isRecord(value.parameters)) &&
    (isNullish(value.rank) || isNumber(value.rank)) &&
    (isNullish(value.initialHeight) || isNumber(value.initialHeight))
  );
}

export function isAddonOverride(value: unknown): value is AddonOverride {
  return (
    isRecord(value) &&
    isString(value.id) &&
    (isNullish(value.parameters) || isRecord(value.parameters)) &&
    (isNullish(value.rank) || isNumber(value.rank)) &&
    (isNullish(value.initialHeight) || isNumber(value.initialHeight))
  );
}

// Helper functions  
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}

function isNullish(value: unknown): value is null | undefined {
  return value === null || value === undefined;
}

function isArray<T>(
  value: unknown,
  itemGuard?: (item: unknown) => item is T
): value is T[] {
  if (!Array.isArray(value)) return false;
  if (!itemGuard) return true;
  return value.every(itemGuard);
}

## Use Cases

### 1. Custom Color Picker

Replace the default color field with a brand-specific color picker:

```typescript
// In plugin definition
connect({
  overrideFieldExtensions(field, ctx) {
    if (field.attributes.field_type === 'color') {
      return {
        editor: {
          id: 'brandColorPicker',
          parameters: {
            brandColors: ['#FF6B6B', '#4ECDC4', '#45B7D1']
          }
        }
      };
    }
  },

  // Render the custom field extension
  renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'brandColorPicker') {
      return {
        render: () => import('./components/BrandColorPicker')
      };
    }
  }
});

// BrandColorPicker.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, Button } from 'datocms-react-ui';

export default function BrandColorPicker({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const brandColors = [
    { name: 'Primary', hex: '#FF6B6B' },
    { name: 'Secondary', hex: '#4ECDC4' },
    { name: 'Accent', hex: '#45B7D1' },
    { name: 'Success', hex: '#95E1D3' },
    { name: 'Warning', hex: '#F38181' },
    { name: 'Info', hex: '#786FA6' }
  ];

  const currentColor = ctx.formValues[ctx.fieldPath] || '#000000';

  return (
    <Canvas ctx={ctx}>
      <div>
        <h4>Brand Colors</h4>
        <div style={{ display: 'grid', gridTemplateColumns: 'repeat(3, 1fr)', gap: '10px' }}>
          {brandColors.map(({ name, hex }) => (
            <Button
              key={hex}
              onClick={() => ctx.setFieldValue(ctx.fieldPath, hex)}
              style={{
                backgroundColor: hex,
                color: '#fff',
                border: currentColor === hex ? '3px solid #000' : 'none'
              }}
            >
              {name}
            </Button>
          ))}
        </div>
        
        <div style={{ marginTop: '20px' }}>
          <label>Custom Color:</label>
          <input
            type="color"
            value={currentColor}
            onChange={(e) => ctx.setFieldValue(ctx.fieldPath, e.target.value)}
            style={{ width: '100%', height: '40px' }}
          />
        </div>
        
        <div style={{
          marginTop: '10px',
          padding: '20px',
          backgroundColor: currentColor,
          color: '#fff',
          textAlign: 'center'
        }}>
          Preview: {currentColor}
        </div>
      </div>
    </Canvas>
  );
}
```

### 2. Enhanced JSON Editor

Replace the default JSON field with a Monaco Editor:

```typescript
// In plugin definition
connect({
  overrideFieldExtensions(field, ctx) {
    if (field.attributes.field_type === 'json') {
      return {
        editor: {
          id: 'monacoJsonEditor',
          initialHeight: 400
        }
      };
    }
  },

  renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'monacoJsonEditor') {
      return {
        render: () => import('./components/MonacoJsonEditor')
      };
    }
  }
});

// MonacoJsonEditor.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas } from 'datocms-react-ui';
import Editor from '@monaco-editor/react';
import { useState, useEffect } from 'react';

export default function MonacoJsonEditor({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [value, setValue] = useState(
    JSON.stringify(ctx.formValues[ctx.fieldPath] || {}, null, 2)
  );
  const [error, setError] = useState<string | null>(null);

  const handleEditorChange = (newValue: string | undefined) => {
    if (!newValue) return;
    
    setValue(newValue);
    
    try {
      const parsed = JSON.parse(newValue);
      ctx.setFieldValue(ctx.fieldPath, parsed);
      setError(null);
      ctx.setFieldError(ctx.fieldPath, null);
    } catch (e) {
      const errorMessage = e instanceof Error ? e.message : 'Invalid JSON';
      setError(errorMessage);
      ctx.setFieldError(ctx.fieldPath, errorMessage);
    }
  };

  useEffect(() => {
    ctx.startAutoResizer();
    return () => ctx.stopAutoResizer();
  }, []);

  return (
    <Canvas ctx={ctx}>
      <div>
        <div style={{ 
          border: '1px solid #ddd', 
          borderRadius: '4px',
          overflow: 'hidden'
        }}>
          <Editor
            height="400px"
            defaultLanguage="json"
            value={value}
            onChange={handleEditorChange}
            theme={ctx.theme.lightMode ? 'light' : 'vs-dark'}
            options={{
              minimap: { enabled: false },
              fontSize: 14,
              readOnly: ctx.disabled,
              scrollBeyondLastLine: false,
              formatOnPaste: true,
              formatOnType: true
            }}
          />
        </div>
        
        {error && (
          <div style={{
            color: '#ff0000',
            fontSize: '12px',
            marginTop: '5px'
          }}>
            Error: {error}
          </div>
        )}
        
        <div style={{
          marginTop: '10px',
          fontSize: '12px',
          color: '#666'
        }}>
          {ctx.field.attributes.hint || 'Enter valid JSON data'}
        </div>
      </div>
    </Canvas>
  );
}
```

### 3. Advanced Date/Time Picker

Replace default date_time field with a feature-rich picker:

```typescript
// In plugin definition  
connect({
  overrideFieldExtensions(field, ctx) {
    if (field.attributes.field_type === 'date_time') {
      return {
        editor: {
          id: 'advancedDateTimePicker',
          parameters: {
            showQuickSelections: true,
            timeIntervals: 15
          }
        }
      };
    }
  },

  renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'advancedDateTimePicker') {
      return {
        render: () => import('./components/AdvancedDateTimePicker')
      };
    }
  }
});

// AdvancedDateTimePicker.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, Button } from 'datocms-react-ui';
import DatePicker from 'react-datepicker';
import 'react-datepicker/dist/react-datepicker.css';
import { format, addDays, startOfWeek, endOfWeek, startOfMonth } from 'date-fns';

export default function AdvancedDateTimePicker({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const value = ctx.formValues[ctx.fieldPath] 
    ? new Date(ctx.formValues[ctx.fieldPath] as string)
    : null;

  const quickSelections = [
    { label: 'Now', getValue: () => new Date() },
    { label: 'Tomorrow', getValue: () => addDays(new Date(), 1) },
    { label: 'Next Week', getValue: () => addDays(new Date(), 7) },
    { label: 'Start of Week', getValue: () => startOfWeek(new Date()) },
    { label: 'End of Week', getValue: () => endOfWeek(new Date()) },
    { label: 'Start of Month', getValue: () => startOfMonth(new Date()) }
  ];

  const handleChange = (date: Date | null) => {
    ctx.setFieldValue(
      ctx.fieldPath, 
      date ? date.toISOString() : null
    );
  };

  return (
    <Canvas ctx={ctx}>
      <div>
        <label>{ctx.field.attributes.label}</label>
        
        <DatePicker
          selected={value}
          onChange={handleChange}
          showTimeSelect
          timeFormat="HH:mm"
          timeIntervals={15}
          dateFormat="MMMM d, yyyy h:mm aa"
          disabled={ctx.disabled}
          required={ctx.field.attributes.validators.required}
          placeholderText="Select date and time"
          className="form-control"
          calendarClassName="custom-calendar"
          showMonthDropdown
          showYearDropdown
          dropdownMode="select"
        />
        
        <div style={{ marginTop: '10px' }}>
          <p style={{ fontSize: '12px', marginBottom: '5px' }}>Quick Select:</p>
          <div style={{ display: 'flex', gap: '5px', flexWrap: 'wrap' }}>
            {quickSelections.map(({ label, getValue }) => (
              <Button
                key={label}
                onClick={() => handleChange(getValue())}
                buttonSize="s"
                buttonType="muted"
              >
                {label}
              </Button>
            ))}
          </div>
        </div>
        
        {value && (
          <div style={{
            marginTop: '10px',
            padding: '10px',
            backgroundColor: '#f5f5f5',
            borderRadius: '4px',
            fontSize: '12px'
          }}>
            <strong>Selected:</strong> {format(value, 'PPpp')}
            <br />
            <strong>ISO:</strong> {value.toISOString()}
          </div>
        )}
        
        {ctx.field.attributes.hint && (
          <p style={{ fontSize: '12px', color: '#666', marginTop: '5px' }}>
            {ctx.field.attributes.hint}
          </p>
        )}
      </div>
    </Canvas>
  );
}
```

### 4. Enhanced Slug Field

Replace default slug field with auto-generation features:

```typescript
// In plugin definition
connect({
  overrideFieldExtensions(field, ctx) {
    if (field.attributes.field_type === 'slug') {
      return {
        editor: {
          id: 'enhancedSlugField',
          parameters: {
            autoGenerate: true,
            checkAvailability: true
          }
        }
      };
    }
  },

  renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'enhancedSlugField') {
      return {
        render: () => import('./components/EnhancedSlugField')
      };
    }
  }
});

// EnhancedSlugField.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

export default function EnhancedSlugField({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [slug, setSlug] = useState(ctx.formValues[ctx.fieldPath] || '');
  const [isLocked, setIsLocked] = useState(false);
  
  // Get title field for auto-generation
  const titleField = ctx.field.attributes.validators.slugTitleField;
  const titleValue = titleField ? ctx.formValues[titleField.id] : '';

  const generateSlug = (text: string): string => {
    return text
      .toLowerCase()
      .trim()
      .replace(/[^\w\s-]/g, '') // Remove special characters
      .replace(/[\s_-]+/g, '-') // Replace spaces with -
      .replace(/^-+|-+$/g, ''); // Remove leading/trailing -
  };

  useEffect(() => {
    if (!isLocked && titleValue && !ctx.formValues[ctx.fieldPath]) {
      const newSlug = generateSlug(titleValue as string);
      setSlug(newSlug);
      ctx.setFieldValue(ctx.fieldPath, newSlug);
    }
  }, [titleValue, isLocked]);

  const handleChange = (value: string) => {
    const cleaned = value.replace(/[^a-z0-9-]/g, '-');
    setSlug(cleaned);
    ctx.setFieldValue(ctx.fieldPath, cleaned);
  };

  const handleRegenerate = () => {
    if (titleValue) {
      const newSlug = generateSlug(titleValue as string);
      setSlug(newSlug);
      ctx.setFieldValue(ctx.fieldPath, newSlug);
    }
  };

  const checkAvailability = async () => {
    const client = ctx.createClient();
    
    try {
      const items = await client.items.list({
        filter: {
          type: ctx.itemType.id,
          fields: {
            [ctx.field.apiKey]: { eq: slug }
          }
        }
      });
      
      const isAvailable = items.length === 0 || 
        (items.length === 1 && items[0].id === ctx.item?.id);
      
      if (isAvailable) {
        ctx.notice('Slug is available!');
        ctx.setFieldError(ctx.fieldPath, null);
      } else {
        ctx.alert('Slug is already in use');
        ctx.setFieldError(ctx.fieldPath, 'This slug is already taken');
      }
    } catch (error) {
      ctx.alert('Failed to check slug availability');
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div>
        <TextField
          id="slug"
          name="slug"
          label={ctx.field.attributes.label}
          value={slug}
          onChange={handleChange}
          hint="URL-friendly version of the title"
          required={ctx.field.attributes.validators.required}
          disabled={ctx.disabled}
          prefix={ctx.site.attributes.internalDomain + '/'}
          suffix={
            <div style={{ display: 'flex', gap: '5px' }}>
              <Button
                onClick={() => setIsLocked(!isLocked)}
                buttonSize="s"
                buttonType="muted"
                title={isLocked ? 'Unlock auto-generation' : 'Lock slug'}
              >
                {isLocked ? '🔒' : '🔓'}
              </Button>
              <Button
                onClick={handleRegenerate}
                buttonSize="s"
                buttonType="muted"
                disabled={!titleValue || isLocked}
                title="Regenerate from title"
              >
                ↻
              </Button>
              <Button
                onClick={checkAvailability}
                buttonSize="s"
                buttonType="muted"
                disabled={!slug}
                title="Check if available"
              >
                ✓
              </Button>
            </div>
          }
        />
        
        <div style={{
          marginTop: '10px',
          fontSize: '12px',
          color: '#666'
        }}>
          {isLocked 
            ? '🔒 Slug is locked and won\'t auto-update'
            : '🔓 Slug will auto-generate from title'}
        </div>
      </div>
    </Canvas>
  );
}
```

### 5. Rich Text Editor Override

Replace structured text with a custom rich text editor:

```typescript
// In plugin definition
connect({
  overrideFieldExtensions(field, ctx) {
    if (field.attributes.field_type === 'structured_text') {
      return {
        editor: {
          id: 'customRichTextEditor',
          initialHeight: 500,
          parameters: {
            toolbar: 'full'
          }
        }
      };
    }
  },

  renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'customRichTextEditor') {
      return {
        render: () => import('./components/CustomRichTextEditor')
      };
    }
  }
});

// CustomRichTextEditor.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas } from 'datocms-react-ui';
import { Editor } from '@tinymce/tinymce-react';
import { useRef } from 'react';

export default function CustomRichTextEditor({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const editorRef = useRef(null);
  const value = ctx.formValues[ctx.fieldPath];
  
  // Convert structured text to HTML for editing
  const structuredTextToHtml = (doc: any): string => {
    if (!doc || !doc.document) return '';
    
    // Simplified conversion - in real app, use proper converter
    return doc.document.children
      .map((node: any) => {
        switch (node.type) {
          case 'paragraph':
            return `<p>${node.children.map((c: any) => c.value).join('')}</p>`;
          case 'heading':
            return `<h${node.level}>${node.children.map((c: any) => c.value).join('')}</h${node.level}>`;
          default:
            return '';
        }
      })
      .join('\n');
  };
  
  // Convert HTML back to structured text
  const htmlToStructuredText = (html: string): any => {
    // Simplified conversion - in real app, use proper parser
    const parser = new DOMParser();
    const doc = parser.parseFromString(html, 'text/html');
    const children = [];
    
    doc.body.childNodes.forEach(node => {
      if (node.nodeName === 'P') {
        children.push({
          type: 'paragraph',
          children: [{ type: 'span', value: node.textContent }]
        });
      } else if (node.nodeName.match(/^H[1-6]$/)) {
        children.push({
          type: 'heading',
          level: parseInt(node.nodeName[1]),
          children: [{ type: 'span', value: node.textContent }]
        });
      }
    });
    
    return {
      schema: 'dast',
      document: {
        type: 'root',
        children
      }
    };
  };

  const handleEditorChange = (content: string) => {
    const structuredText = htmlToStructuredText(content);
    ctx.setFieldValue(ctx.fieldPath, structuredText);
  };

  return (
    <Canvas ctx={ctx}>
      <div>
        <label>{ctx.field.attributes.label}</label>
        
        <Editor
          apiKey={ctx.plugin.attributes.parameters.tinymceApiKey}
          onInit={(evt, editor) => editorRef.current = editor}
          initialValue={structuredTextToHtml(value)}
          init={{
            height: 500,
            menubar: false,
            plugins: [
              'advlist', 'autolink', 'lists', 'link', 'image', 'charmap',
              'searchreplace', 'visualblocks', 'code', 'fullscreen',
              'insertdatetime', 'media', 'table', 'preview', 'help', 'wordcount'
            ],
            toolbar: 'undo redo | blocks | ' +
              'bold italic forecolor | alignleft aligncenter ' +
              'alignright alignjustify | bullist numlist outdent indent | ' +
              'removeformat | help',
            content_style: 'body { font-family:Helvetica,Arial,sans-serif; font-size:14px }',
            readonly: ctx.disabled
          }}
          onEditorChange={handleEditorChange}
        />
        
        {ctx.field.attributes.hint && (
          <p style={{ fontSize: '12px', color: '#666', marginTop: '10px' }}>
            {ctx.field.attributes.hint}
          </p>
        )}
      </div>
    </Canvas>
  );
}
```

### 6. Location Field with Map

Replace lat_lon field with an interactive map:

```typescript
// In plugin definition
connect({
  overrideFieldExtensions(field, ctx) {
    if (field.attributes.field_type === 'lat_lon') {
      return {
        editor: {
          id: 'interactiveMapLocation',
          initialHeight: 450,
          parameters: {
            defaultZoom: 13,
            showAddress: true
          }
        }
      };
    }
  },

  renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'interactiveMapLocation') {
      return {
        render: () => import('./components/InteractiveMapLocation')
      };
    }
  }
});

// InteractiveMapLocation.tsx
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField } from 'datocms-react-ui';
import { MapContainer, TileLayer, Marker, useMapEvents } from 'react-leaflet';
import 'leaflet/dist/leaflet.css';
import { useState, useEffect } from 'react';

export default function InteractiveMapLocation({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const value = ctx.formValues[ctx.fieldPath] as { latitude: number; longitude: number } | null;
  const [position, setPosition] = useState<[number, number]>(
    value ? [value.latitude, value.longitude] : [51.505, -0.09]
  );
  const [address, setAddress] = useState('');

  const LocationMarker = () => {
    const map = useMapEvents({
      click(e) {
        if (!ctx.disabled) {
          const { lat, lng } = e.latlng;
          setPosition([lat, lng]);
          ctx.setFieldValue(ctx.fieldPath, { 
            latitude: lat, 
            longitude: lng 
          });
          reverseGeocode(lat, lng);
        }
      }
    });

    return position ? <Marker position={position} /> : null;
  };

  const reverseGeocode = async (lat: number, lng: number) => {
    try {
      const response = await fetch(
        `https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lng}`
      );
      const data = await response.json();
      setAddress(data.display_name || 'Unknown location');
    } catch (error) {
      setAddress('Failed to get address');
    }
  };

  const handleCoordinateChange = (type: 'latitude' | 'longitude', value: string) => {
    const numValue = parseFloat(value);
    if (!isNaN(numValue)) {
      const newPosition: [number, number] = type === 'latitude' 
        ? [numValue, position[1]]
        : [position[0], numValue];
      
      setPosition(newPosition);
      ctx.setFieldValue(ctx.fieldPath, {
        latitude: newPosition[0],
        longitude: newPosition[1]
      });
    }
  };

  useEffect(() => {
    if (value) {
      reverseGeocode(value.latitude, value.longitude);
    }
  }, []);

  return (
    <Canvas ctx={ctx}>
      <div>
        <label>{ctx.field.attributes.label}</label>
        
        <div style={{ height: '400px', marginBottom: '10px' }}>
          <MapContainer 
            center={position} 
            zoom={13} 
            style={{ height: '100%', width: '100%' }}
          >
            <TileLayer
              attribution='&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
              url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
            />
            <LocationMarker />
          </MapContainer>
        </div>
        
        <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '10px' }}>
          <TextField
            id="latitude"
            name="latitude"
            label="Latitude"
            value={position[0].toString()}
            onChange={(val) => handleCoordinateChange('latitude', val)}
            disabled={ctx.disabled}
          />
          <TextField
            id="longitude"
            name="longitude"
            label="Longitude"
            value={position[1].toString()}
            onChange={(val) => handleCoordinateChange('longitude', val)}
            disabled={ctx.disabled}
          />
        </div>
        
        {address && (
          <div style={{
            marginTop: '10px',
            padding: '10px',
            backgroundColor: '#f5f5f5',
            borderRadius: '4px',
            fontSize: '12px'
          }}>
            <strong>Address:</strong> {address}
          </div>
        )}
        
        <p style={{ fontSize: '12px', color: '#666', marginTop: '10px' }}>
          Click on the map to set location or enter coordinates manually
        </p>
      </div>
    </Canvas>
  );
}
```

## Advanced Override Examples

### Conditional Overrides

Override fields based on model or field configuration:

```typescript
connect({
  overrideFieldExtensions(field, ctx) {
    // Override only for specific models
    if (field.attributes.field_type === 'string' && 
        ctx.itemType.attributes.api_key === 'product') {
      return {
        editor: {
          id: 'productNameEditor',
          parameters: {
            validateSKU: true
          }
        }
      };
    }

    // Override based on field validators
    if (field.attributes.field_type === 'text' &&
        field.attributes.validators.format?.pattern === '^#[A-Fa-f0-9]{6}$') {
      return {
        editor: { id: 'hexColorEditor' }
      };
    }

    // Override with sidebar panel
    if (field.attributes.field_type === 'json' &&
        field.attributes.api_key === 'metadata') {
      return {
        editor: {
          id: 'metadataEditor',
          asSidebarPanel: {
            startOpen: true,
            placement: ['after', 'info']
          }
        }
      };
    }
  }
});
```

### Adding Field Addons

Enhance fields with additional functionality:

```typescript
connect({
  overrideFieldExtensions(field, ctx) {
    if (field.attributes.field_type === 'string') {
      return {
        // Keep default editor
        addons: [
          {
            id: 'characterCounter',
            parameters: {
              maxLength: field.attributes.validators.length?.max || 255
            }
          },
          {
            id: 'stringFormatter',
            rank: 10
          }
        ]
      };
    }

    if (field.attributes.field_type === 'structured_text') {
      return {
        editor: {
          id: 'richTextEditor'
        },
        addons: [
          {
            id: 'readingTime',
            initialHeight: 50
          },
          {
            id: 'aiWritingAssistant',
            parameters: {
              model: 'gpt-4'
            }
          }
        ]
      };
    }
  }
});
```

## Best Practices

1. **Maintain Compatibility**: Ensure your override handles all features of the original field type
2. **Preserve Validation**: Respect the field's validation rules (required, min/max, etc.)
3. **Handle All States**: Support disabled, error, and loading states
4. **Follow UI Patterns**: Use datocms-react-ui components for consistency
5. **Lazy Load**: Use dynamic imports for heavy dependencies
6. **Accessibility**: Maintain proper ARIA labels and keyboard navigation
7. **Data Format**: Store data in the expected format for the field type
8. **Performance**: Consider performance impact of replacing core fields
9. **Error Handling**: Properly handle and display validation errors
10. **Theme Support**: Respect the user's theme preferences
11. **Conditional Logic**: Only override when necessary based on context
12. **User Choice**: Consider allowing users to switch back to default editors

## Related Hooks

- `manualFieldExtensions` - Define custom field extensions
- `renderFieldExtension` - Render the actual field extension UI
- `validateManualFieldExtensionParameters` - Validate field extension parameters
- `fieldDropdownActions` - Add actions to field dropdowns
- `renderManualFieldExtensionConfigScreen` - Configuration UI for extensions

## Important Notes

- Overrides apply globally to all fields of the specified type
- The override must handle all aspects of the field type it replaces
- Consider performance implications when overriding frequently-used field types
- Test thoroughly as overrides affect the entire project
- Users can still manually select different field extensions if needed
- Overrides should maintain backward compatibility with existing data
- The original field editor is completely replaced, not extended