# renderUploadSidebar Hook

## Purpose

The `renderUploadSidebar` hook renders custom sidebar components for the upload/media view. This allows you to create comprehensive tools for managing media assets, including metadata editing, image analysis, and asset organization features.

## Signature

```typescript
renderUploadSidebar(
  uploadSidebarId: string,
  ctx: RenderUploadSidebarCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

### uploadSidebarId: string
The ID of the sidebar being rendered

### ctx: RenderUploadSidebarCtx
Context object containing all properties and methods for the upload sidebar

## Type Definitions

```typescript
// Complete type definitions for renderUploadSidebar hook
export type RenderUploadSidebarHook = {
  renderUploadSidebar: (
    uploadSidebarId: string,
    ctx: RenderUploadSidebarCtx
  ) => void | {
    render: () => Promise<{ default: React.ComponentType<any> }>;
  };
};

export type RenderUploadSidebarCtx = SelfResizingPluginFrameCtx<
  'renderUploadSidebar',
  {
    uploadSidebarId: string;
    upload: Upload;
    parameters: Record<string, unknown>;
    locale: string;
  },
  {
    updateUpload: (payload: Partial<Upload>) => Promise<Upload>;
    editUploadMetadata: (metadata: Record<string, any>) => Promise<void>;
    regenerateUrl: () => Promise<string>;
  }
>;

// Base context type with self-resizing capabilities
type SelfResizingPluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = ImposedSizePluginFrameCtx<Mode, AdditionalProperties, AdditionalMethods> &
  SizingUtilities &
  IframeMethods;

// Imposed size context
type ImposedSizePluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BasePluginFrameCtx<Mode, AdditionalProperties, AdditionalMethods>;

// Base plugin frame context
type BasePluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BaseProperties &
  BaseMethods &
  AdditionalMethods &
  AdditionalProperties;

// Sizing utilities for self-resizing contexts
type SizingUtilities = {
  startAutoResizer: () => void;
  stopAutoResizer: () => void;
  updateHeight: (height: number) => void;
};

// Iframe-specific methods
type IframeMethods = {
  getSettings: () => Promise<{ mode: string }>;
};

// Base properties included in all contexts
type BaseProperties = {
  plugin: Plugin;
  currentUserAccessToken: string | null;
  environment: string;
  site: Site;
  currentUser: User | SsoUser | null;
  currentRole: Role;
  account: Account;
  mainLocale: string;
  theme: 'dark' | 'light';
  currentUserMenuLocale: string;
  owner: Account | Organization;
  itemTypes: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
  users: User[];
  ssoUsers: SsoUser[];
};

// Base methods included in all contexts
type BaseMethods = {
  createClient(options?: {
    apiToken?: string;
    environment?: string;
  }): CmaClient;
  loadItemTypeFields(itemTypeId: string): Promise<Field[]>;
  loadItemTypeFieldsets(itemTypeId: string): Promise<Fieldset[]>;
  loadFieldsUsingPlugin(): Promise<Field[]>;
  loadUsers(): Promise<User[]>;
  loadSsoUsers(): Promise<SsoUser[]>;
  navigateTo(path: string): void;
  scrollToField(path: string): void;
  openUrl(url: string): void;
  copyToClipboard(text: string): void;
  getFieldValue(path: string): unknown;
  toggleField(path: string, show: boolean): void;
  disableField(path: string, disable: boolean): void;
  setFieldValue(path: string, value: unknown): void;
  notice(message: string): void;
  alert(message: string): void;
  customToast(options: {
    message: string;
    type?: 'notice' | 'alert' | 'warning';
    dismissAfterTimeout?: number | false;
    cta?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  openConfirm(options: {
    title: string;
    content?: ReactNode;
    choices?: Array<{
      label: string;
      value: unknown;
      intent?: 'positive' | 'negative';
    }>;
    cancel?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  updatePluginParameters(params: Record<string, unknown>): Promise<void>;
  selectItem(itemTypeId: string, options?: {
    multiple?: boolean;
    locale?: string;
    filters?: ItemListFilter;
  }): Promise<Item | Item[] | null>;
  editItem(itemId: string, options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
  }): Promise<Item | null>;
  createNewItem(itemTypeId: string, options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
  }): Promise<Item | null>;
  openModal(modalId: string, options?: {
    title?: string;
    width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth';
    parameters?: Record<string, unknown>;
  }): Promise<unknown>;
  renderModal(modalId: string): void;
};

// Upload type definition
type Upload = {
  id: string;
  type: string;
  attributes: {
    path: string;
    basename: string;
    filename: string;
    url: string;
    size: number;
    width?: number;
    height?: number;
    format?: string;
    blurhash?: string;
    duration?: number;
    frame_rate?: number;
    mux_playback_id?: string;
    mux_mp4_highest_res?: string;
    thumbhash?: string;
    mime_type?: string;
    default_field_metadata: {
      [locale: string]: {
        alt: string | null;
        title: string | null;
        custom_data: Record<string, unknown>;
        focal_point?: {
          x: number;
          y: number;
        } | null;
      };
    };
    author?: string;
    notes?: string;
    copyright?: string;
    tags?: string[];
    smart_tags?: string[];
    custom_data?: Record<string, unknown>;
    created_at: string;
    updated_at: string;
    is_image: boolean;
  };
  relationships: {
    created_by?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
    upload_collection?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
  };
};

// Supporting types
type Plugin = {
  id: string;
  type: string;
  attributes: {
    name: string;
    description?: string;
    homepage_url?: string;
    package_name?: string;
    package_version?: string;
    parameters: Record<string, any>;
  };
  relationships?: {
    installed_by?: {
      data: {
        id: string;
        type: string;
      };
    };
  };
};

type Site = {
  id: string;
  type: string;
  attributes: {
    name: string;
    internal_subdomain: string;
    production_frontend_url?: string;
    primary_locale: string;
    locales: string[];
    timezone: string;
    global_seo?: Record<string, any>;
    favicon?: any;
    no_index: boolean;
  };
};

type User = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
    is_admin: boolean;
  };
};

type SsoUser = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    is_admin: boolean;
  };
};

type Role = {
  id: string;
  type: string;
  attributes: {
    name: string;
    can_edit_content: boolean;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_duplicate_records: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_edit_environment: boolean;
    can_access_build_triggers: boolean;
    environments_access: 'all' | 'primary_only' | string[];
    positive_item_type_permissions: ItemTypePermission[];
    negative_item_type_permissions: ItemTypePermission[];
  };
};

type ItemTypePermission = {
  action: 'create' | 'update' | 'delete' | 'take_over' | 'edit_metadata' | 'edit_schema' | 'publish';
  item_type?: string;
  on_environment?: string;
  locales?: string[];
  item_id?: string;
};

type Account = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    company?: string;
  };
};

type Organization = {
  id: string;
  type: string;
  attributes: {
    name: string;
  };
};

type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    tree: boolean;
    modular_block: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'tabular';
    ordering_direction?: 'asc' | 'desc';
    has_singleton_item: boolean;
  };
  relationships: {
    fields: {
      data: Array<{
        id: string;
        type: string;
      }>;
    };
    fieldsets: {
      data: Array<{
        id: string;
        type: string;
      }>;
    };
    ordering_field?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
    title_field?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
    singleton_item?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
  };
};

type Field = {
  id: string;
  type: string;
  attributes: {
    label: string;
    api_key: string;
    field_type: string;
    hint?: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
      field_size: 12;
      groups?: Array<{
        id: string;
        title: string;
        description?: string;
      }>;
    };
    default_value?: any;
    validators: Record<string, any>;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
    fieldset?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
  };
};

type Fieldset = {
  id: string;
  type: string;
  attributes: {
    title: string;
    hint?: string;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
  };
};

type CmaClient = any; // Simplified for this context
type ItemListFilter = any; // Simplified for this context
type Item = any; // Simplified for this context
type ReactNode = any; // Simplified for this context
type FullConnectParameters = any; // Simplified for this context
```

## Context Properties

The `renderUploadSidebar` hook receives a context object with:

### Base Properties
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `environment`: string - The ID of the current environment
- `theme`: Theme - Theme colors for the current project

### Upload Properties
- `upload`: Upload - The current upload/media asset
- `uploadSidebarId`: string - The ID of the sidebar being rendered
- `parameters`: Record<string, unknown> - Configuration parameters for this sidebar
- `locale`: string - Current locale

### Methods
- `updateUpload(payload: Partial<Upload>)`: Promise<Upload> - Update upload metadata
- `editUploadMetadata(metadata: Record<string, any>)`: Promise<void> - Edit custom metadata
- `regenerateUrl()`: Promise<string> - Regenerate the upload URL
- Standard context methods (notice, alert, openModal, etc.)

## Use Cases

### 1. Advanced Image Editor Sidebar

Provide comprehensive image editing capabilities:

```typescript
async renderUploadSidebar(
  uploadSidebarId: string,
  ctx: RenderUploadSidebarCtx
) {
  if (uploadSidebarId === 'imageEditor') {
    return {
      render: () => import('./components/ImageEditorSidebar'),
    };
  }
}

// ImageEditorSidebar.tsx
import { RenderUploadSidebarCtx } from 'datocms-plugin-sdk';
import { Canvas, Section, Button, TextField, SelectField } from 'datocms-react-ui';
import { useState, useEffect, useRef } from 'react';
import { 
  FaCrop, 
  FaAdjust, 
  FaImage, 
  FaSave,
  FaUndo,
  FaRedo,
  FaMagic 
} from 'react-icons/fa';

interface ImageAdjustments {
  brightness: number;
  contrast: number;
  saturation: number;
  blur: number;
  rotation: number;
}

export default function ImageEditorSidebar({ ctx }: { ctx: RenderUploadSidebarCtx }) {
  const [adjustments, setAdjustments] = useState<ImageAdjustments>({
    brightness: 100,
    contrast: 100,
    saturation: 100,
    blur: 0,
    rotation: 0,
  });
  
  const [cropMode, setCropMode] = useState(false);
  const [cropData, setCropData] = useState({ x: 0, y: 0, width: 0, height: 0 });
  const [previewUrl, setPreviewUrl] = useState(ctx.upload.url);
  const [isProcessing, setIsProcessing] = useState(false);
  const [history, setHistory] = useState<ImageAdjustments[]>([]);
  const [historyIndex, setHistoryIndex] = useState(-1);
  
  const canvasRef = useRef<HTMLCanvasElement>(null);

  const isImage = ctx.upload.mimeType?.startsWith('image/');

  useEffect(() => {
    if (isImage) {
      applyAdjustments();
    }
  }, [adjustments]);

  const applyAdjustments = () => {
    if (!canvasRef.current) return;
    
    const canvas = canvasRef.current;
    const ctx2d = canvas.getContext('2d');
    if (!ctx2d) return;

    const img = new Image();
    img.crossOrigin = 'anonymous';
    img.onload = () => {
      canvas.width = img.width;
      canvas.height = img.height;
      
      // Apply filters
      ctx2d.filter = `
        brightness(${adjustments.brightness}%)
        contrast(${adjustments.contrast}%)
        saturate(${adjustments.saturation}%)
        blur(${adjustments.blur}px)
      `;
      
      // Apply rotation
      if (adjustments.rotation !== 0) {
        ctx2d.translate(canvas.width / 2, canvas.height / 2);
        ctx2d.rotate((adjustments.rotation * Math.PI) / 180);
        ctx2d.translate(-canvas.width / 2, -canvas.height / 2);
      }
      
      ctx2d.drawImage(img, 0, 0);
      
      // Update preview
      canvas.toBlob((blob) => {
        if (blob) {
          setPreviewUrl(URL.createObjectURL(blob));
        }
      });
    };
    
    img.src = ctx.upload.url;
  };

  const updateAdjustment = (key: keyof ImageAdjustments, value: number) => {
    const newAdjustments = { ...adjustments, [key]: value };
    setAdjustments(newAdjustments);
    
    // Add to history
    const newHistory = history.slice(0, historyIndex + 1);
    newHistory.push(newAdjustments);
    setHistory(newHistory);
    setHistoryIndex(newHistory.length - 1);
  };

  const undo = () => {
    if (historyIndex > 0) {
      setHistoryIndex(historyIndex - 1);
      setAdjustments(history[historyIndex - 1]);
    }
  };

  const redo = () => {
    if (historyIndex < history.length - 1) {
      setHistoryIndex(historyIndex + 1);
      setAdjustments(history[historyIndex + 1]);
    }
  };

  const autoEnhance = () => {
    // Simple auto-enhance algorithm
    setAdjustments({
      brightness: 110,
      contrast: 115,
      saturation: 120,
      blur: 0,
      rotation: 0,
    });
  };

  const saveEdits = async () => {
    if (!canvasRef.current) return;
    
    setIsProcessing(true);
    
    try {
      // Convert canvas to blob
      const blob = await new Promise<Blob>((resolve) => {
        canvasRef.current!.toBlob((blob) => resolve(blob!));
      });
      
      // Upload the edited image
      const formData = new FormData();
      formData.append('file', blob, `edited-${ctx.upload.filename}`);
      
      // This would need actual implementation with your upload endpoint
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData,
      });
      
      const newUpload = await response.json();
      
      // Update the upload record
      await ctx.updateUpload({
        url: newUpload.url,
        metadata: {
          ...ctx.upload.metadata,
          edited: true,
          editHistory: {
            adjustments,
            editedAt: new Date().toISOString(),
            editor: ctx.currentUser.email,
          },
        },
      });
      
      ctx.notice('Image saved successfully!');
    } catch (error) {
      ctx.alert('Failed to save edited image');
    } finally {
      setIsProcessing(false);
    }
  };

  if (!isImage) {
    return (
      <Canvas ctx={ctx}>
        <div style={{ padding: '20px', textAlign: 'center' }}>
          <FaImage size={48} style={{ opacity: 0.3, marginBottom: '10px' }} />
          <p>Image editing is only available for image files.</p>
        </div>
      </Canvas>
    );
  }

  return (
    <Canvas ctx={ctx}>
      <div style={{ height: '100vh', overflowY: 'auto' }}>
        {/* Preview */}
        <div style={{ 
          padding: '20px',
          backgroundColor: '#f5f5f5',
          textAlign: 'center',
        }}>
          <img
            src={previewUrl}
            alt={ctx.upload.filename}
            style={{
              maxWidth: '100%',
              maxHeight: '300px',
              objectFit: 'contain',
            }}
          />
          <canvas
            ref={canvasRef}
            style={{ display: 'none' }}
          />
        </div>

        {/* Toolbar */}
        <div style={{
          padding: '10px',
          backgroundColor: '#fff',
          borderBottom: '1px solid #e0e0e0',
          display: 'flex',
          gap: '5px',
          justifyContent: 'center',
        }}>
          <Button
            onClick={undo}
            buttonSize="s"
            disabled={historyIndex <= 0}
            leftIcon={<FaUndo />}
          >
            Undo
          </Button>
          <Button
            onClick={redo}
            buttonSize="s"
            disabled={historyIndex >= history.length - 1}
            leftIcon={<FaRedo />}
          >
            Redo
          </Button>
          <Button
            onClick={autoEnhance}
            buttonSize="s"
            leftIcon={<FaMagic />}
          >
            Auto Enhance
          </Button>
        </div>

        {/* Adjustments */}
        <Section title="Adjustments">
          <div style={{ padding: '15px' }}>
            <div style={{ marginBottom: '20px' }}>
              <label style={{ display: 'block', marginBottom: '5px', fontSize: '13px' }}>
                Brightness: {adjustments.brightness}%
              </label>
              <input
                type="range"
                min="0"
                max="200"
                value={adjustments.brightness}
                onChange={(e) => updateAdjustment('brightness', Number(e.target.value))}
                style={{ width: '100%' }}
              />
            </div>

            <div style={{ marginBottom: '20px' }}>
              <label style={{ display: 'block', marginBottom: '5px', fontSize: '13px' }}>
                Contrast: {adjustments.contrast}%
              </label>
              <input
                type="range"
                min="0"
                max="200"
                value={adjustments.contrast}
                onChange={(e) => updateAdjustment('contrast', Number(e.target.value))}
                style={{ width: '100%' }}
              />
            </div>

            <div style={{ marginBottom: '20px' }}>
              <label style={{ display: 'block', marginBottom: '5px', fontSize: '13px' }}>
                Saturation: {adjustments.saturation}%
              </label>
              <input
                type="range"
                min="0"
                max="200"
                value={adjustments.saturation}
                onChange={(e) => updateAdjustment('saturation', Number(e.target.value))}
                style={{ width: '100%' }}
              />
            </div>

            <div style={{ marginBottom: '20px' }}>
              <label style={{ display: 'block', marginBottom: '5px', fontSize: '13px' }}>
                Blur: {adjustments.blur}px
              </label>
              <input
                type="range"
                min="0"
                max="10"
                step="0.1"
                value={adjustments.blur}
                onChange={(e) => updateAdjustment('blur', Number(e.target.value))}
                style={{ width: '100%' }}
              />
            </div>

            <div style={{ marginBottom: '20px' }}>
              <label style={{ display: 'block', marginBottom: '5px', fontSize: '13px' }}>
                Rotation: {adjustments.rotation}°
              </label>
              <input
                type="range"
                min="-180"
                max="180"
                value={adjustments.rotation}
                onChange={(e) => updateAdjustment('rotation', Number(e.target.value))}
                style={{ width: '100%' }}
              />
            </div>

            <Button
              onClick={() => setAdjustments({
                brightness: 100,
                contrast: 100,
                saturation: 100,
                blur: 0,
                rotation: 0,
              })}
              fullWidth
              buttonType="muted"
            >
              Reset All
            </Button>
          </div>
        </Section>

        {/* Image Info */}
        <Section title="Image Information" startOpen={false}>
          <div style={{ padding: '15px', fontSize: '13px' }}>
            <div style={{ marginBottom: '8px' }}>
              <strong>Filename:</strong> {ctx.upload.filename}
            </div>
            <div style={{ marginBottom: '8px' }}>
              <strong>Dimensions:</strong> {ctx.upload.width} × {ctx.upload.height}px
            </div>
            <div style={{ marginBottom: '8px' }}>
              <strong>Size:</strong> {(ctx.upload.size / 1024 / 1024).toFixed(2)} MB
            </div>
            <div style={{ marginBottom: '8px' }}>
              <strong>Format:</strong> {ctx.upload.format}
            </div>
          </div>
        </Section>

        {/* Save Button */}
        <div style={{ padding: '20px' }}>
          <Button
            onClick={saveEdits}
            buttonType="primary"
            fullWidth
            leftIcon={<FaSave />}
            disabled={isProcessing}
          >
            {isProcessing ? 'Saving...' : 'Save Edited Image'}
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

### 2. Asset Metadata Manager

Manage comprehensive metadata for assets:

```typescript
async renderUploadSidebar(
  uploadSidebarId: string,
  ctx: RenderUploadSidebarCtx
) {
  if (uploadSidebarId === 'metadataManager') {
    return {
      render: () => import('./components/MetadataManagerSidebar'),
    };
  }
}

// MetadataManagerSidebar.tsx
import { RenderUploadSidebarCtx } from 'datocms-plugin-sdk';
import { Canvas, Section, TextField, TextareaField, SelectField, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaTags, FaUser, FaCopyright, FaCalendar } from 'react-icons/fa';

interface ImageMetadata {
  title?: string;
  description?: string;
  tags?: string[];
  author?: string;
  copyright?: string;
  dateTaken?: string;
  location?: {
    latitude?: number;
    longitude?: number;
    placeName?: string;
  };
  camera?: {
    make?: string;
    model?: string;
    lens?: string;
    settings?: {
      aperture?: string;
      shutterSpeed?: string;
      iso?: number;
    };
  };
}

export default function MetadataManagerSidebar({ ctx }: { ctx: RenderUploadSidebarCtx }) {
  const [metadata, setMetadata] = useState<ImageMetadata>({});
  const [exifData, setExifData] = useState<any>(null);
  const [tagInput, setTagInput] = useState('');
  const [isSaving, setIsSaving] = useState(false);

  useEffect(() => {
    loadMetadata();
    if (ctx.upload.format === 'jpg' || ctx.upload.format === 'jpeg') {
      extractExifData();
    }
  }, [ctx.upload.id]);

  const loadMetadata = () => {
    const existingMeta = ctx.upload.customData || {};
    setMetadata({
      title: ctx.upload.title || '',
      description: ctx.upload.alt || '',
      tags: existingMeta.tags || [],
      author: existingMeta.author || '',
      copyright: existingMeta.copyright || '',
      dateTaken: existingMeta.dateTaken || '',
      location: existingMeta.location || {},
      camera: existingMeta.camera || {},
    });
  };

  const extractExifData = async () => {
    try {
      // In a real implementation, you would extract EXIF data
      // This is a mock example
      const mockExif = {
        Make: 'Canon',
        Model: 'EOS R5',
        LensModel: 'RF 24-70mm F2.8 L IS USM',
        FNumber: 2.8,
        ExposureTime: '1/250',
        ISO: 400,
        DateTimeOriginal: '2024:01:15 14:30:00',
        GPSLatitude: 40.7128,
        GPSLongitude: -74.0060,
      };
      
      setExifData(mockExif);
      
      // Auto-populate from EXIF
      if (mockExif.DateTimeOriginal && !metadata.dateTaken) {
        setMetadata(prev => ({
          ...prev,
          dateTaken: mockExif.DateTimeOriginal.replace(/:/g, '-').replace(' ', 'T'),
        }));
      }
      
      if (mockExif.Make && !metadata.camera?.make) {
        setMetadata(prev => ({
          ...prev,
          camera: {
            ...prev.camera,
            make: mockExif.Make,
            model: mockExif.Model,
            lens: mockExif.LensModel,
            settings: {
              aperture: `f/${mockExif.FNumber}`,
              shutterSpeed: mockExif.ExposureTime,
              iso: mockExif.ISO,
            },
          },
        }));
      }
      
      if (mockExif.GPSLatitude && !metadata.location?.latitude) {
        setMetadata(prev => ({
          ...prev,
          location: {
            ...prev.location,
            latitude: mockExif.GPSLatitude,
            longitude: mockExif.GPSLongitude,
          },
        }));
      }
    } catch (error) {
      console.error('Failed to extract EXIF data:', error);
    }
  };

  const addTag = () => {
    if (tagInput.trim() && !metadata.tags?.includes(tagInput.trim())) {
      setMetadata({
        ...metadata,
        tags: [...(metadata.tags || []), tagInput.trim()],
      });
      setTagInput('');
    }
  };

  const removeTag = (tag: string) => {
    setMetadata({
      ...metadata,
      tags: metadata.tags?.filter(t => t !== tag) || [],
    });
  };

  const saveMetadata = async () => {
    setIsSaving(true);
    
    try {
      // Update the upload with new metadata
      await ctx.updateUpload({
        title: metadata.title,
        alt: metadata.description,
        customData: {
          tags: metadata.tags,
          author: metadata.author,
          copyright: metadata.copyright,
          dateTaken: metadata.dateTaken,
          location: metadata.location,
          camera: metadata.camera,
        },
      });
      
      // Also update any custom metadata
      await ctx.editUploadMetadata({
        ...metadata,
        lastModified: new Date().toISOString(),
        modifiedBy: ctx.currentUser.email,
      });
      
      ctx.notice('Metadata saved successfully');
    } catch (error) {
      ctx.alert('Failed to save metadata');
    } finally {
      setIsSaving(false);
    }
  };

  const generateAltText = async () => {
    // This would integrate with an AI service for alt text generation
    ctx.notice('Generating alt text...');
    
    try {
      // Mock AI-generated alt text
      const generatedAlt = `A ${ctx.upload.format} image showing [generated description]`;
      setMetadata({
        ...metadata,
        description: generatedAlt,
      });
      ctx.notice('Alt text generated');
    } catch (error) {
      ctx.alert('Failed to generate alt text');
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ height: '100vh', overflowY: 'auto', padding: '20px' }}>
        {/* Basic Information */}
        <Section title="Basic Information">
          <div style={{ padding: '15px' }}>
            <TextField
              id="title"
              name="title"
              label="Title"
              value={metadata.title || ''}
              onChange={(value) => setMetadata({ ...metadata, title: value })}
              hint="A descriptive title for this asset"
            />
            
            <TextareaField
              id="description"
              name="description"
              label="Alt Text / Description"
              value={metadata.description || ''}
              onChange={(value) => setMetadata({ ...metadata, description: value })}
              hint="Important for accessibility and SEO"
              textareaInputProps={{ rows: 3 }}
            />
            
            <Button
              onClick={generateAltText}
              buttonSize="s"
              style={{ marginTop: '5px' }}
            >
              Generate Alt Text with AI
            </Button>
          </div>
        </Section>

        {/* Tags */}
        <Section title="Tags" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <div style={{ display: 'flex', gap: '10px', marginBottom: '10px' }}>
              <TextField
                id="tagInput"
                name="tagInput"
                label=""
                value={tagInput}
                onChange={setTagInput}
                placeholder="Add a tag..."
                textInputProps={{
                  onKeyPress: (e) => {
                    if (e.key === 'Enter') {
                      e.preventDefault();
                      addTag();
                    }
                  },
                }}
              />
              <Button onClick={addTag} buttonSize="s">
                Add
              </Button>
            </div>
            
            <div style={{ display: 'flex', flexWrap: 'wrap', gap: '5px' }}>
              {metadata.tags?.map(tag => (
                <span
                  key={tag}
                  style={{
                    padding: '4px 8px',
                    backgroundColor: '#e0e0e0',
                    borderRadius: '12px',
                    fontSize: '12px',
                    display: 'inline-flex',
                    alignItems: 'center',
                    gap: '5px',
                  }}
                >
                  {tag}
                  <button
                    onClick={() => removeTag(tag)}
                    style={{
                      background: 'none',
                      border: 'none',
                      cursor: 'pointer',
                      padding: 0,
                      fontSize: '16px',
                      lineHeight: 1,
                    }}
                  >
                    ×
                  </button>
                </span>
              ))}
            </div>
          </div>
        </Section>

        {/* Copyright & Attribution */}
        <Section title="Copyright & Attribution" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <TextField
              id="author"
              name="author"
              label="Author/Photographer"
              value={metadata.author || ''}
              onChange={(value) => setMetadata({ ...metadata, author: value })}
              leftIcon={<FaUser />}
            />
            
            <TextField
              id="copyright"
              name="copyright"
              label="Copyright"
              value={metadata.copyright || ''}
              onChange={(value) => setMetadata({ ...metadata, copyright: value })}
              placeholder="© 2024 Company Name"
              leftIcon={<FaCopyright />}
            />
            
            <TextField
              id="dateTaken"
              name="dateTaken"
              label="Date Taken"
              value={metadata.dateTaken || ''}
              onChange={(value) => setMetadata({ ...metadata, dateTaken: value })}
              textInputProps={{ type: 'datetime-local' }}
              leftIcon={<FaCalendar />}
            />
          </div>
        </Section>

        {/* Location */}
        <Section title="Location" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <TextField
              id="placeName"
              name="placeName"
              label="Place Name"
              value={metadata.location?.placeName || ''}
              onChange={(value) => setMetadata({
                ...metadata,
                location: { ...metadata.location, placeName: value },
              })}
              placeholder="New York City, USA"
            />
            
            <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '10px' }}>
              <TextField
                id="latitude"
                name="latitude"
                label="Latitude"
                value={String(metadata.location?.latitude || '')}
                onChange={(value) => setMetadata({
                  ...metadata,
                  location: { ...metadata.location, latitude: parseFloat(value) || undefined },
                })}
                textInputProps={{ type: 'number', step: 'any' }}
              />
              
              <TextField
                id="longitude"
                name="longitude"
                label="Longitude"
                value={String(metadata.location?.longitude || '')}
                onChange={(value) => setMetadata({
                  ...metadata,
                  location: { ...metadata.location, longitude: parseFloat(value) || undefined },
                })}
                textInputProps={{ type: 'number', step: 'any' }}
              />
            </div>
          </div>
        </Section>

        {/* Camera Information */}
        {exifData && (
          <Section title="Camera Information" startOpen={false}>
            <div style={{ padding: '15px', fontSize: '13px' }}>
              <div style={{ marginBottom: '8px' }}>
                <strong>Camera:</strong> {metadata.camera?.make} {metadata.camera?.model}
              </div>
              {metadata.camera?.lens && (
                <div style={{ marginBottom: '8px' }}>
                  <strong>Lens:</strong> {metadata.camera.lens}
                </div>
              )}
              {metadata.camera?.settings && (
                <>
                  <div style={{ marginBottom: '8px' }}>
                    <strong>Aperture:</strong> {metadata.camera.settings.aperture}
                  </div>
                  <div style={{ marginBottom: '8px' }}>
                    <strong>Shutter Speed:</strong> {metadata.camera.settings.shutterSpeed}
                  </div>
                  <div style={{ marginBottom: '8px' }}>
                    <strong>ISO:</strong> {metadata.camera.settings.iso}
                  </div>
                </>
              )}
            </div>
          </Section>
        )}

        {/* Save Button */}
        <div style={{ padding: '20px', paddingBottom: '40px' }}>
          <Button
            onClick={saveMetadata}
            buttonType="primary"
            fullWidth
            disabled={isSaving}
          >
            {isSaving ? 'Saving...' : 'Save Metadata'}
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Performance**: Optimize for large files and avoid blocking operations
2. **Visual Feedback**: Show loading states and progress for long operations
3. **Error Handling**: Handle file processing errors gracefully
4. **Responsive Design**: Ensure sidebar works well at different heights
5. **File Type Support**: Check file types before offering specific features
6. **Metadata Preservation**: Preserve existing metadata when updating
7. **Undo/Redo**: Implement history for destructive operations
8. **Auto-Save**: Consider auto-saving changes to prevent data loss
9. **Keyboard Shortcuts**: Add shortcuts for common operations
10. **Accessibility**: Ensure all controls are keyboard accessible

## Related Hooks

- `uploadSidebars` - Define which upload sidebars your plugin provides
- `uploadSidebarPanels` - Add panels to the default upload sidebar
- `renderUploadSidebarPanel` - Render individual sidebar panels
- `renderAssetSource` - Add custom asset sources

## Important Notes

- Upload sidebars replace the entire sidebar when active
- Always check file type before offering format-specific features
- Use the Canvas component for consistent styling
- Consider file size when processing images client-side
- Changes to uploads should be saved through the provided methods
- Large file operations should show progress indicators