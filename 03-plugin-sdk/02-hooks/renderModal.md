# renderModal Hook

## Purpose

The `renderModal` hook is responsible for rendering modal dialogs when triggered by `ctx.openModal()` calls from other hooks or components. Modals provide a focused interface for user interactions, confirmations, data input, or displaying detailed information without leaving the current context.

## Signature

```typescript
// Hook function signature
renderModal(modalId: string, ctx: RenderModalCtx): void

// Full type definition
type RenderModalHook = {
  /**
   * This function will be called when the plugin requested to open a modal (see
   * the `openModal` hook)
   *
   * @tag modals
   */
  renderModal: (modalId: string, ctx: RenderModalCtx) => void;
};
```

## Parameters

- `modalId`: string - The unique identifier of the modal to render
- `ctx`: RenderModalCtx - The context object containing all necessary data and methods

## Type Definitions

```typescript
// Hook signature
export type RenderModalHook = {
  /**
   * This function will be called when the plugin requested to open a modal (see
   * the `openModal` hook)
   *
   * @tag modals
   */
  renderModal: (modalId: string, ctx: RenderModalCtx) => void;
};

// Main context type
type RenderModalCtx = SelfResizingPluginFrameCtx<
  'renderModal',
  {
    /** The ID of the modal that needs to be rendered */
    modalId: string;
    /**
     * The arbitrary `parameters` of the modal declared in the `openModal`
     * function
     */
    parameters: Record<string, unknown>;
  },
  {
    /**
     * A function to be called by the plugin to close the modal. The `openModal`
     * call will be resolved with the passed return value
     */
    resolve: (returnValue: unknown) => Promise<void>;
  }
>;

// Which expands to include:
type RenderModalCtx = {
  // Mode and frame properties
  mode: 'renderModal';
  bodyPadding: [number, number, number, number];
  
  // Modal-specific properties
  modalId: string;
  parameters: Record<string, unknown>;
  
  // Base properties
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken: string | undefined;
  site: Site;
  environment: string;
  isEnvironmentPrimary: boolean;
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
  createNewItem: (itemTypeId: string) => Promise<Item | null>;
  selectItem: {
    (itemTypeId: string, options: { multiple: true; initialLocationQuery?: ItemListLocationQuery }): Promise<Item[] | null>;
    (itemTypeId: string, options?: { multiple: false; initialLocationQuery?: ItemListLocationQuery }): Promise<Item | null>;
  };
  editItem: (itemId: string) => Promise<Item | null>;
  alert: (message: string) => Promise<void>;
  notice: (message: string) => Promise<void>;
  customToast: <CtaValue = unknown>(toast: Toast<CtaValue>) => Promise<CtaValue | null>;
  selectUpload: {
    (options: { multiple: true }): Promise<Upload[] | null>;
    (options?: { multiple: false }): Promise<Upload | null>;
  };
  editUpload: (uploadId: string) => Promise<(Upload & { deleted?: true }) | null>;
  editUploadMetadata: (fileFieldValue: FileFieldValue, locale?: string) => Promise<FileFieldValue | null>;
  openModal: (modal: Modal) => Promise<unknown>;
  openConfirm: (options: ConfirmOptions) => Promise<unknown>;
  navigateTo: (path: string) => Promise<void>;
  getSettings: () => Promise<RenderModalCtx>;
  
  // Modal-specific methods
  resolve: (returnValue: unknown) => Promise<void>;
  
  // Sizing utilities
  updateHeight: (height: number) => Promise<void>;
  updateAutoHeight: (autoHeight: boolean) => Promise<void>;
  startAutoResizer: () => Promise<void>;
  
  // Iframe methods
  close: () => Promise<void>;
  reject: (error?: unknown) => Promise<void>;
};

// Supporting types
type Theme = {
  primaryColor: string;
  accentColor: string;
  semiTransparentAccentColor: string;
  lightColor: string;
  darkColor: string;
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

The hook receives a context object that includes:

### Base Properties
All standard context properties like `plugin`, `currentUser`, `site`, etc.

### Modal-Specific Properties
- `modalId`: string - The ID of the modal being rendered
- `parameters`: Record<string, unknown> - Parameters passed when opening the modal

### Modal-Specific Methods
- `resolve(returnValue: unknown)`: Promise<void> - Close modal and return value
- `close()`: Promise<void> - Close modal without returning value
- `reject(error?: unknown)`: Promise<void> - Close modal with error

### Frame Properties
- Modal rendering context with configurable size
- Automatic height adjustment capabilities

## Use Cases

### 1. Data Input Modal

Create a form modal for data collection:

```typescript
renderModal(modalId: string, ctx: RenderModalCtx) {
  switch (modalId) {
    case 'collect-metadata':
      return {
        render: () => import('./modals/MetadataModal'),
      };
  }
}

// MetadataModal.tsx
import { RenderModalCtx } from 'datocms-plugin-sdk';
import { Canvas, Form, TextField, TextareaField, Button } from 'datocms-react-ui';
import { useState } from 'react';

// Component types from datocms-react-ui
type CanvasProps = {
  ctx: RenderModalCtx;
  children: React.ReactNode;
  noAutoHeight?: boolean;
  theme?: 'light' | 'dark' | 'auto';
};

type TextFieldProps = {
  id: string;
  name: string;
  label: React.ReactNode;
  hint?: React.ReactNode;
  placeholder?: string;
  error?: React.ReactNode;
  required?: boolean;
  value: string;
  onChange: (value: string) => void;
  textInputProps?: TextInputProps;
};

type TextareaFieldProps = {
  id: string;
  name: string;
  label: React.ReactNode;
  hint?: React.ReactNode;
  placeholder?: string;
  error?: React.ReactNode;
  required?: boolean;
  value: string;
  onChange: (value: string) => void;
  textareaInputProps?: { rows?: number };
};

type ButtonProps = {
  children?: React.ReactNode;
  type?: 'button' | 'submit' | 'reset';
  disabled?: boolean;
  onClick?: React.MouseEventHandler;
  buttonType?: 'primary' | 'muted' | 'negative';
  buttonSize?: 'xxs' | 'xs' | 's' | 'm' | 'l' | 'xl';
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  fullWidth?: boolean;
};

// Modal component props type
type MetadataModalProps = {
  ctx: RenderModalCtx;
};

export default function MetadataModal({ ctx }: MetadataModalProps) {
  const [formData, setFormData] = useState({
    title: ctx.parameters.defaultTitle || '',
    description: '',
    keywords: [],
    author: ctx.currentUser.attributes.full_name
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Validate form
    if (!formData.title) {
      ctx.alert('Title is required');
      return;
    }

    if (formData.description.length < 50) {
      const proceed = await ctx.openConfirm({
        title: 'Short Description',
        content: 'The description is quite short. Continue anyway?',
        choices: [
          { label: 'Edit', value: false },
          { label: 'Continue', value: true }
        ]
      });
      
      if (!proceed) return;
    }

    // Return the data
    ctx.resolve(formData);
  };

  const handleCancel = () => {
    ctx.close();
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px' }}>
        <h2>Add Metadata</h2>
        
        <Form onSubmit={handleSubmit}>
          <TextField
            id="title"
            label="Title"
            value={formData.title}
            onChange={(value) => setFormData({ ...formData, title: value })}
            required
            hint="This will be used as the SEO title"
          />

          <TextareaField
            id="description"
            label="Description"
            value={formData.description}
            onChange={(value) => setFormData({ ...formData, description: value })}
            hint="Recommended: 50-160 characters"
            textareaInputProps={{
              rows: 3
            }}
          />

          <TagInput
            id="keywords"
            label="Keywords"
            value={formData.keywords}
            onChange={(value) => setFormData({ ...formData, keywords: value })}
            placeholder="Add keywords..."
          />

          <TextField
            id="author"
            label="Author"
            value={formData.author}
            onChange={(value) => setFormData({ ...formData, author: value })}
            disabled
          />

          <div style={{ marginTop: '20px', display: 'flex', gap: '10px' }}>
            <Button type="submit" buttonType="primary">
              Save Metadata
            </Button>
            <Button type="button" onClick={handleCancel} buttonType="muted">
              Cancel
            </Button>
          </div>
        </Form>
      </div>
    </Canvas>
  );
}
```

### 2. Confirmation Dialog with Options

Create a custom confirmation modal:

```typescript
renderModal(modalId: string, ctx: RenderModalCtx) {
  if (modalId === 'delete-confirmation') {
    return {
      render: () => import('./modals/DeleteConfirmation'),
    };
  }
}

// DeleteConfirmation.tsx
import { RenderModalCtx } from 'datocms-plugin-sdk';
import { Canvas, Button } from 'datocms-react-ui';
import { useState } from 'react';

type DeleteConfirmationProps = {
  ctx: RenderModalCtx;
};

type DeleteParameters = {
  items: Array<{ id: string; title?: string; name?: string }>;
  cascade?: Array<{ type: string; count: number }>;
};

export default function DeleteConfirmation({ ctx }: DeleteConfirmationProps) {
  const { items, cascade } = ctx.parameters as DeleteParameters;
  const { items, cascade } = ctx.parameters;
  const [deleteOption, setDeleteOption] = useState('soft');
  const [confirmed, setConfirmed] = useState(false);

  const handleDelete = () => {
    if (!confirmed) {
      ctx.alert('Please confirm the deletion');
      return;
    }

    ctx.resolve({
      action: 'delete',
      option: deleteOption,
      items: items.map(i => i.id)
    });
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px', maxWidth: '500px' }}>
        <h2 style={{ color: '#dc3545' }}>⚠️ Delete Confirmation</h2>
        
        <p>You are about to delete {items.length} item(s):</p>
        
        <ul style={{ maxHeight: '150px', overflow: 'auto' }}>
          {items.map(item => (
            <li key={item.id}>{item.title || item.name || item.id}</li>
          ))}
        </ul>

        {cascade && cascade.length > 0 && (
          <div style={{ 
            background: '#fff3cd', 
            padding: '10px', 
            borderRadius: '4px',
            marginTop: '10px'
          }}>
            <strong>Warning:</strong> This will also delete:
            <ul>
              {cascade.map(c => (
                <li key={c.type}>{c.count} {c.type}(s)</li>
              ))}
            </ul>
          </div>
        )}

        <div style={{ marginTop: '20px' }}>
          <label>
            <input
              type="radio"
              value="soft"
              checked={deleteOption === 'soft'}
              onChange={(e) => setDeleteOption(e.target.value)}
            />
            Soft Delete (can be restored)
          </label>
          <label style={{ display: 'block', marginTop: '10px' }}>
            <input
              type="radio"
              value="hard"
              checked={deleteOption === 'hard'}
              onChange={(e) => setDeleteOption(e.target.value)}
            />
            Permanent Delete (cannot be restored)
          </label>
        </div>

        <div style={{ marginTop: '20px' }}>
          <label style={{ color: '#dc3545' }}>
            <input
              type="checkbox"
              checked={confirmed}
              onChange={(e) => setConfirmed(e.target.checked)}
            />
            I understand this action cannot be undone
          </label>
        </div>

        <div style={{ marginTop: '20px', display: 'flex', gap: '10px' }}>
          <Button
            onClick={handleDelete}
            buttonType="negative"
            disabled={!confirmed}
          >
            Delete {items.length} Item(s)
          </Button>
          <Button
            onClick={() => ctx.close()}
            buttonType="muted"
          >
            Cancel
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

### 3. Selection Modal

Create a modal for selecting from options:

```typescript
renderModal(modalId: string, ctx: RenderModalCtx) {
  if (modalId === 'select-template') {
    return {
      render: () => import('./modals/TemplateSelector'),
    };
  }
}

// TemplateSelector.tsx
import { RenderModalCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, Spinner } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

type TemplateSelectorProps = {
  ctx: RenderModalCtx;
};

type Template = {
  id: string;
  name: string;
  description: string;
  thumbnail?: string;
  category: string;
};

type TemplateParameters = {
  category?: string;
};

type SpinnerProps = {
  size?: 'xs' | 's' | 'm' | 'l' | 'xl';
  placement?: 'inline' | 'centered';
};

export default function TemplateSelector({ ctx }: TemplateSelectorProps) {
  const parameters = ctx.parameters as TemplateParameters;
  const [selectedTemplate, setSelectedTemplate] = useState(null);
  const [templates, setTemplates] = useState([]);
  const [loading, setLoading] = useState(true);
  const [preview, setPreview] = useState(null);

  useEffect(() => {
    loadTemplates();
  }, []);

  const loadTemplates = async () => {
    try {
      const templates = await fetchTemplates(
        ctx.parameters.category,
        ctx.plugin.attributes.parameters.templateApiKey
      );
      setTemplates(templates);
    } catch (error) {
      ctx.alert('Failed to load templates');
    } finally {
      setLoading(false);
    }
  };

  const handleSelect = () => {
    if (!selectedTemplate) {
      ctx.alert('Please select a template');
      return;
    }

    ctx.resolve(selectedTemplate);
  };

  const handlePreview = async (template) => {
    const previewData = await generatePreview(template);
    setPreview(previewData);
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', height: '600px' }}>
        {/* Template List */}
        <div style={{ width: '300px', borderRight: '1px solid #e0e0e0', padding: '20px' }}>
          <h3>Select Template</h3>
          
          {loading ? (
            <Spinner />
          ) : (
            <div style={{ marginTop: '20px' }}>
              {templates.map(template => (
                <TemplateCard
                  key={template.id}
                  template={template}
                  selected={selectedTemplate?.id === template.id}
                  onClick={() => {
                    setSelectedTemplate(template);
                    handlePreview(template);
                  }}
                />
              ))}
            </div>
          )}
        </div>

        {/* Preview Area */}
        <div style={{ flex: 1, padding: '20px' }}>
          {preview ? (
            <div>
              <h3>{preview.title}</h3>
              <div style={{ 
                border: '1px solid #e0e0e0', 
                borderRadius: '4px',
                padding: '20px',
                marginTop: '10px'
              }}>
                <TemplatePreview data={preview} />
              </div>
            </div>
          ) : (
            <div style={{ 
              display: 'flex', 
              alignItems: 'center', 
              justifyContent: 'center',
              height: '100%',
              color: '#999'
            }}>
              Select a template to preview
            </div>
          )}
        </div>
      </div>

      {/* Footer */}
      <div style={{ 
        borderTop: '1px solid #e0e0e0', 
        padding: '20px',
        display: 'flex',
        justifyContent: 'flex-end',
        gap: '10px'
      }}>
        <Button onClick={() => ctx.close()} buttonType="muted">
          Cancel
        </Button>
        <Button 
          onClick={handleSelect} 
          buttonType="primary"
          disabled={!selectedTemplate}
        >
          Use Template
        </Button>
      </div>
    </Canvas>
  );
}
```

### 4. Progress Modal

Show progress for long-running operations:

```typescript
renderModal(modalId: string, ctx: RenderModalCtx) {
  if (modalId === 'import-progress') {
    return {
      render: () => import('./modals/ImportProgress'),
    };
  }
}

// ImportProgress.tsx
import { RenderModalCtx } from 'datocms-plugin-sdk';
import { Canvas, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

type ImportProgressProps = {
  ctx: RenderModalCtx;
};

type ImportParameters = {
  totalItems: number;
  items: Array<{ name?: string; [key: string]: any }>;
  importFunction: (item: any) => Promise<void>;
  onProgress?: (progress: number) => void;
};

type ImportError = {
  item: string;
  error: string;
};

export default function ImportProgress({ ctx }: ImportProgressProps) {
  const { totalItems, onProgress } = ctx.parameters as ImportParameters;
  const { totalItems, onProgress } = ctx.parameters;
  const [progress, setProgress] = useState(0);
  const [currentItem, setCurrentItem] = useState('');
  const [errors, setErrors] = useState([]);
  const [completed, setCompleted] = useState(false);

  useEffect(() => {
    startImport();
  }, []);

  const startImport = async () => {
    const { items, importFunction } = ctx.parameters;

    for (let i = 0; i < items.length; i++) {
      const item = items[i];
      setCurrentItem(item.name || `Item ${i + 1}`);
      
      try {
        await importFunction(item);
        setProgress(((i + 1) / items.length) * 100);
      } catch (error) {
        setErrors(prev => [...prev, {
          item: item.name,
          error: error.message
        }]);
      }
    }

    setCompleted(true);
  };

  const handleClose = () => {
    if (!completed) {
      ctx.alert('Import is still in progress');
      return;
    }

    ctx.resolve({
      success: errors.length === 0,
      imported: totalItems - errors.length,
      errors
    });
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px', minHeight: '300px' }}>
        <h2>Import Progress</h2>
        
        <div style={{ marginTop: '20px' }}>
          <ProgressBar value={progress} />
          <p style={{ marginTop: '10px', fontSize: '14px', color: '#666' }}>
            {completed ? 'Import completed!' : `Importing: ${currentItem}`}
          </p>
        </div>

        <div style={{ marginTop: '20px' }}>
          <strong>Status:</strong>
          <ul>
            <li>Total items: {totalItems}</li>
            <li>Processed: {Math.floor((progress / 100) * totalItems)}</li>
            <li>Errors: {errors.length}</li>
          </ul>
        </div>

        {errors.length > 0 && (
          <div style={{ 
            marginTop: '20px',
            padding: '10px',
            background: '#fee',
            borderRadius: '4px'
          }}>
            <strong>Errors:</strong>
            <ul style={{ marginTop: '5px', fontSize: '12px' }}>
              {errors.map((error, i) => (
                <li key={i}>{error.item}: {error.error}</li>
              ))}
            </ul>
          </div>
        )}

        <div style={{ marginTop: '20px' }}>
          <Button
            onClick={handleClose}
            buttonType={completed ? 'primary' : 'muted'}
            disabled={!completed}
          >
            {completed ? 'Done' : 'Please wait...'}
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

### 5. Configuration Modal

Create a settings/configuration modal:

```typescript
renderModal(modalId: string, ctx: RenderModalCtx) {
  if (modalId === 'field-config') {
    return {
      render: () => import('./modals/FieldConfiguration'),
    };
  }
}

// FieldConfiguration.tsx
import { RenderModalCtx } from 'datocms-plugin-sdk';
import { Canvas, Button } from 'datocms-react-ui';
import { useState } from 'react';

type FieldConfigurationProps = {
  ctx: RenderModalCtx;
};

type FieldConfigParameters = {
  field: {
    id: string;
    label: string;
    api_key: string;
    field_type: string;
  };
  currentConfig?: Record<string, any>;
};

export default function FieldConfiguration({ ctx }: FieldConfigurationProps) {
  const { field, currentConfig } = ctx.parameters as FieldConfigParameters;
  const { field, currentConfig } = ctx.parameters;
  const [config, setConfig] = useState(currentConfig || getDefaultConfig(field));
  const [activeTab, setActiveTab] = useState('general');

  const handleSave = () => {
    // Validate configuration
    const validation = validateFieldConfig(config, field);
    
    if (!validation.valid) {
      ctx.alert(validation.errors.join('\n'));
      return;
    }

    ctx.resolve(config);
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ minHeight: '500px' }}>
        <h2>Configure {field.label}</h2>
        
        <Tabs value={activeTab} onChange={setActiveTab}>
          <Tab value="general" label="General" />
          <Tab value="validation" label="Validation" />
          <Tab value="appearance" label="Appearance" />
          <Tab value="advanced" label="Advanced" />
        </Tabs>

        <div style={{ padding: '20px' }}>
          {activeTab === 'general' && (
            <GeneralSettings
              config={config}
              onChange={setConfig}
              field={field}
            />
          )}

          {activeTab === 'validation' && (
            <ValidationSettings
              config={config}
              onChange={setConfig}
              field={field}
            />
          )}

          {activeTab === 'appearance' && (
            <AppearanceSettings
              config={config}
              onChange={setConfig}
              field={field}
            />
          )}

          {activeTab === 'advanced' && (
            <AdvancedSettings
              config={config}
              onChange={setConfig}
              field={field}
            />
          )}
        </div>

        <div style={{ 
          borderTop: '1px solid #e0e0e0',
          padding: '20px',
          display: 'flex',
          justifyContent: 'space-between'
        }}>
          <Button
            onClick={() => setConfig(getDefaultConfig(field))}
            buttonType="muted"
          >
            Reset to Defaults
          </Button>
          
          <div style={{ display: 'flex', gap: '10px' }}>
            <Button onClick={() => ctx.close()} buttonType="muted">
              Cancel
            </Button>
            <Button onClick={handleSave} buttonType="primary">
              Save Configuration
            </Button>
          </div>
        </div>
      </div>
    </Canvas>
  );
}
```

### 6. Media Preview Modal

Display media with editing capabilities:

```typescript
renderModal(modalId: string, ctx: RenderModalCtx) {
  if (modalId === 'media-preview') {
    return {
      render: () => import('./modals/MediaPreview'),
    };
  }
}

// MediaPreview.tsx
import { RenderModalCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, Form, TextField, TextareaField } from 'datocms-react-ui';
import { useState } from 'react';

type MediaPreviewProps = {
  ctx: RenderModalCtx;
};

type Asset = {
  id: string;
  url: string;
  alt?: string;
  title?: string;
  caption?: string;
  focalPoint?: { x: number; y: number };
  width: number;
  height: number;
  size: number;
  format: string;
};

type MediaParameters = {
  asset: Asset;
};

export default function MediaPreview({ ctx }: MediaPreviewProps) {
  const { asset } = ctx.parameters as MediaParameters;
  const { asset } = ctx.parameters;
  const [metadata, setMetadata] = useState({
    alt: asset.alt || '',
    title: asset.title || '',
    caption: asset.caption || ''
  });
  const [focal, setFocal] = useState(asset.focalPoint || { x: 0.5, y: 0.5 });

  const handleSave = () => {
    ctx.resolve({
      ...asset,
      ...metadata,
      focalPoint: focal
    });
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', height: '600px' }}>
        {/* Image Preview */}
        <div style={{ flex: 1, position: 'relative', background: '#f0f0f0' }}>
          <FocalPointSelector
            imageUrl={asset.url}
            focalPoint={focal}
            onChange={setFocal}
          />
          
          <div style={{ 
            position: 'absolute',
            bottom: '20px',
            left: '20px',
            background: 'rgba(0,0,0,0.7)',
            color: 'white',
            padding: '10px',
            borderRadius: '4px'
          }}>
            <p>Dimensions: {asset.width} × {asset.height}</p>
            <p>Size: {formatFileSize(asset.size)}</p>
            <p>Format: {asset.format}</p>
          </div>
        </div>

        {/* Metadata Form */}
        <div style={{ width: '350px', padding: '20px', borderLeft: '1px solid #e0e0e0' }}>
          <h3>Image Details</h3>
          
          <Form>
            <TextField
              id="alt"
              label="Alt Text"
              value={metadata.alt}
              onChange={(value) => setMetadata({ ...metadata, alt: value })}
              hint="Describe the image for accessibility"
            />

            <TextField
              id="title"
              label="Title"
              value={metadata.title}
              onChange={(value) => setMetadata({ ...metadata, title: value })}
            />

            <TextareaField
              id="caption"
              label="Caption"
              value={metadata.caption}
              onChange={(value) => setMetadata({ ...metadata, caption: value })}
              textareaInputProps={{ rows: 3 }}
            />

            <div style={{ marginTop: '20px' }}>
              <label>Focal Point</label>
              <p style={{ fontSize: '12px', color: '#666' }}>
                Click on the image to set the focal point
              </p>
              <p>X: {(focal.x * 100).toFixed(0)}%, Y: {(focal.y * 100).toFixed(0)}%</p>
            </div>
          </Form>

          <div style={{ marginTop: '30px', display: 'flex', gap: '10px' }}>
            <Button onClick={() => ctx.close()} buttonType="muted">
              Cancel
            </Button>
            <Button onClick={handleSave} buttonType="primary">
              Save Changes
            </Button>
          </div>
        </div>
      </div>
    </Canvas>
  );
}
```

## Modal Sizes and Configuration

```typescript
// Complete Modal type definition
type Modal = {
  /** ID of the modal. Will be the first argument for the `renderModal` function */
  id: string;
  /** Title for the modal. Ignored by `fullWidth` modals */
  title?: string;
  /** Whether to present a close button for the modal or not */
  closeDisabled?: boolean;
  /** Width of the modal. Can be a number, or one of the predefined sizes */
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  /**
   * An arbitrary configuration object that will be passed as the `parameters`
   * property of the second argument of the `renderModal` function
   */
  parameters?: Record<string, unknown>;
  /** The initial height to set for the iframe that will render the modal content */
  initialHeight?: number;
};

// Modal size examples
// Small modal (400px)
ctx.openModal({
  id: 'my-modal',
  title: 'Small Modal',
  width: 's'
});

// Medium modal (600px)
ctx.openModal({
  id: 'my-modal',
  title: 'Medium Modal',
  width: 'm'
});

// Large modal (800px)
ctx.openModal({
  id: 'my-modal',
  title: 'Large Modal',
  width: 'l'
});

// Extra large modal (1000px)
ctx.openModal({
  id: 'my-modal',
  title: 'Extra Large Modal',
  width: 'xl'
});

// Custom width
ctx.openModal({
  id: 'my-modal',
  title: 'Custom Width Modal',
  width: 650
});
```

## Best Practices

1. **Always Use Canvas**: Wrap modal content in Canvas for consistent styling
2. **Handle Resolution**: Use `ctx.resolve()` to return values
3. **Handle Cancellation**: Use `ctx.close()` or `ctx.reject()` appropriately
4. **Validate Input**: Validate user input before resolving
5. **Loading States**: Show loading indicators for async operations
6. **Error Handling**: Display clear error messages
7. **Responsive Design**: Ensure modals work at different sizes

## Related Hooks

- **renderPage**: Full page rendering
- **renderConfigScreen**: Plugin configuration modals
- **renderManualFieldExtensionConfigScreen**: Field extension configuration

## Return Value and Hook Registration

```typescript
// Hook registration in plugin entry
import { connect, RenderModalCtx } from 'datocms-plugin-sdk';

connect({
  renderModal(modalId: string, ctx: RenderModalCtx) {
    switch (modalId) {
      case 'my-modal':
        return {
          render: () => import('./modals/MyModal')
        };
      case 'another-modal':
        return {
          render: () => import('./modals/AnotherModal')
        };
      default:
        return;
    }
  }
});

// The render function should return a module with a default export
// that is a React component accepting { ctx: RenderModalCtx }
```

## Important Notes

- Modals are rendered in an iframe overlay
- The modal ID must be unique within your plugin
- Parameters passed to `openModal()` are available in `ctx.parameters`
- Modals can open other modals (nested modals)
- Always resolve or reject to close the modal properly
- Modal state is not preserved after closing
- The modal context includes all base context methods for API calls