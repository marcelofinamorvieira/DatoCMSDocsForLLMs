# renderUploadSidebarPanel Hook

## Purpose

The `renderUploadSidebarPanel` hook renders collapsible panels within the upload/media sidebar. These panels integrate into the existing sidebar structure, allowing you to add focused functionality for media management without replacing the entire sidebar.

## Signature

```typescript
renderUploadSidebarPanel(
  uploadSidebarPanelId: string,
  ctx: RenderUploadSidebarPanelCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

### uploadSidebarPanelId: string
The ID of the panel being rendered

### ctx: RenderUploadSidebarPanelCtx
Context object containing all properties and methods for the upload sidebar panel

## Type Definitions

```typescript
// Complete type definitions for renderUploadSidebarPanel hook
export type RenderUploadSidebarPanelHook = {
  renderUploadSidebarPanel: (
    uploadSidebarPanelId: string,
    ctx: RenderUploadSidebarPanelCtx
  ) => void | {
    render: () => Promise<{ default: React.ComponentType<any> }>;
  };
};

export type RenderUploadSidebarPanelCtx = SelfResizingPluginFrameCtx<
  'renderUploadSidebarPanel',
  {
    uploadSidebarPanelId: string;
    upload: Upload;
    parameters: Record<string, unknown>;
  },
  {
    updateUpload: (payload: Partial<Upload>) => Promise<Upload>;
    editUploadMetadata: (metadata: Record<string, any>) => Promise<void>;
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

The `renderUploadSidebarPanel` hook receives a context object with:

### Base Properties
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `theme`: Theme - Theme colors for the current project

### Upload Properties
- `upload`: Upload - The current upload/media asset
- `uploadSidebarPanelId`: string - The ID of the panel being rendered
- `parameters`: Record<string, unknown> - Configuration parameters for this panel

### Methods
- `updateUpload(payload: Partial<Upload>)`: Promise<Upload> - Update upload metadata
- `editUploadMetadata(metadata: Record<string, any>)`: Promise<void> - Edit custom metadata
- Standard context methods (notice, alert, etc.)

## Use Cases

### 1. Image Analysis Panel

Analyze images using AI/ML services:

```typescript
async renderUploadSidebarPanel(
  uploadSidebarPanelId: string,
  ctx: RenderUploadSidebarPanelCtx
) {
  if (uploadSidebarPanelId === 'imageAnalysis') {
    return {
      render: () => import('./components/ImageAnalysisPanel'),
    };
  }
}

// ImageAnalysisPanel.tsx
import { RenderUploadSidebarPanelCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, Spinner } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaEye, FaTags, FaSmile, FaPalette } from 'react-icons/fa';

interface AnalysisResult {
  labels?: Array<{ name: string; confidence: number }>;
  faces?: Array<{ emotions: Record<string, number>; boundingBox: any }>;
  colors?: Array<{ hex: string; percentage: number; name: string }>;
  text?: Array<{ text: string; boundingBox: any }>;
  objects?: Array<{ name: string; confidence: number; boundingBox: any }>;
}

export default function ImageAnalysisPanel({ ctx }: { ctx: RenderUploadSidebarPanelCtx }) {
  const [analysis, setAnalysis] = useState<AnalysisResult | null>(null);
  const [isAnalyzing, setIsAnalyzing] = useState(false);
  const [hasAnalyzed, setHasAnalyzed] = useState(false);

  const isImage = ctx.upload.mimeType?.startsWith('image/');
  const visionApiKey = ctx.plugin.attributes.parameters.googleVisionApiKey;

  useEffect(() => {
    // Check if we have cached analysis
    const cachedAnalysis = ctx.upload.customData?.imageAnalysis;
    if (cachedAnalysis) {
      setAnalysis(cachedAnalysis);
      setHasAnalyzed(true);
    }
  }, [ctx.upload.id]);

  const analyzeImage = async () => {
    if (!visionApiKey) {
      ctx.alert('Google Vision API key not configured');
      return;
    }

    setIsAnalyzing(true);
    
    try {
      // Google Vision API analysis
      const features = [
        { type: 'LABEL_DETECTION', maxResults: 10 },
        { type: 'FACE_DETECTION', maxResults: 5 },
        { type: 'IMAGE_PROPERTIES', maxResults: 5 },
        { type: 'TEXT_DETECTION', maxResults: 50 },
        { type: 'OBJECT_LOCALIZATION', maxResults: 10 },
      ];

      const response = await fetch(
        `https://vision.googleapis.com/v1/images:annotate?key=${visionApiKey}`,
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            requests: [{
              image: { source: { imageUri: ctx.upload.url } },
              features,
            }],
          }),
        }
      );

      const data = await response.json();
      const result = data.responses[0];

      const analysisResult: AnalysisResult = {
        labels: result.labelAnnotations?.map((label: any) => ({
          name: label.description,
          confidence: Math.round(label.score * 100),
        })),
        faces: result.faceAnnotations?.map((face: any) => ({
          emotions: {
            joy: face.joyLikelihood,
            sorrow: face.sorrowLikelihood,
            anger: face.angerLikelihood,
            surprise: face.surpriseLikelihood,
          },
          boundingBox: face.boundingPoly,
        })),
        colors: result.imagePropertiesAnnotation?.dominantColors?.colors?.map((color: any) => ({
          hex: rgbToHex(color.color),
          percentage: Math.round(color.score * 100),
          name: getColorName(color.color),
        })),
        text: result.textAnnotations?.slice(1), // First item is full text
        objects: result.localizedObjectAnnotations?.map((obj: any) => ({
          name: obj.name,
          confidence: Math.round(obj.score * 100),
          boundingBox: obj.boundingPoly,
        })),
      };

      setAnalysis(analysisResult);
      setHasAnalyzed(true);

      // Save analysis to upload metadata
      await ctx.editUploadMetadata({
        imageAnalysis: analysisResult,
        analyzedAt: new Date().toISOString(),
      });

      // Auto-generate tags from labels
      if (analysisResult.labels && analysisResult.labels.length > 0) {
        const suggestedTags = analysisResult.labels
          .filter(label => label.confidence > 80)
          .map(label => label.name.toLowerCase())
          .slice(0, 5);

        const confirmed = await ctx.openConfirm({
          title: 'Apply Suggested Tags?',
          content: `Add these tags: ${suggestedTags.join(', ')}?`,
          choices: [
            { label: 'Apply Tags', value: true, intent: 'primary' },
            { label: 'Skip', value: false },
          ],
        });

        if (confirmed) {
          await ctx.updateUpload({
            tags: [...(ctx.upload.tags || []), ...suggestedTags],
          });
        }
      }
    } catch (error) {
      ctx.alert('Failed to analyze image: ' + error.message);
    } finally {
      setIsAnalyzing(false);
    }
  };

  const rgbToHex = (color: any): string => {
    const r = Math.round(color.red || 0);
    const g = Math.round(color.green || 0);
    const b = Math.round(color.blue || 0);
    return '#' + [r, g, b].map(x => x.toString(16).padStart(2, '0')).join('');
  };

  const getColorName = (color: any): string => {
    // Simple color naming - in production, use a proper color naming library
    const r = color.red || 0;
    const g = color.green || 0;
    const b = color.blue || 0;
    
    if (r > 200 && g < 100 && b < 100) return 'Red';
    if (r < 100 && g > 200 && b < 100) return 'Green';
    if (r < 100 && g < 100 && b > 200) return 'Blue';
    if (r > 200 && g > 200 && b < 100) return 'Yellow';
    if (r > 200 && g < 100 && b > 200) return 'Magenta';
    if (r < 100 && g > 200 && b > 200) return 'Cyan';
    if (r > 200 && g > 200 && b > 200) return 'White';
    if (r < 50 && g < 50 && b < 50) return 'Black';
    return 'Gray';
  };

  if (!isImage) {
    return (
      <Canvas ctx={ctx}>
        <div style={{ padding: '16px', textAlign: 'center', color: '#666' }}>
          <FaEye size={24} style={{ opacity: 0.5, marginBottom: '8px' }} />
          <p style={{ fontSize: '13px' }}>
            Image analysis is only available for image files.
          </p>
        </div>
      </Canvas>
    );
  }

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {!hasAnalyzed && (
          <div style={{ textAlign: 'center' }}>
            <p style={{ marginBottom: '12px', fontSize: '13px', color: '#666' }}>
              Analyze this image to detect objects, text, faces, and colors.
            </p>
            <Button
              onClick={analyzeImage}
              buttonType="primary"
              disabled={isAnalyzing}
              fullWidth
            >
              {isAnalyzing ? 'Analyzing...' : 'Analyze Image'}
            </Button>
          </div>
        )}

        {isAnalyzing && (
          <div style={{ textAlign: 'center', padding: '20px' }}>
            <Spinner />
            <p style={{ marginTop: '10px', fontSize: '13px', color: '#666' }}>
              Analyzing image content...
            </p>
          </div>
        )}

        {analysis && (
          <div>
            {/* Labels/Tags */}
            {analysis.labels && analysis.labels.length > 0 && (
              <div style={{ marginBottom: '20px' }}>
                <h4 style={{ display: 'flex', alignItems: 'center', marginBottom: '10px' }}>
                  <FaTags style={{ marginRight: '8px' }} />
                  Detected Labels
                </h4>
                <div style={{ display: 'flex', flexWrap: 'wrap', gap: '5px' }}>
                  {analysis.labels.map((label, i) => (
                    <span
                      key={i}
                      style={{
                        padding: '4px 8px',
                        backgroundColor: '#e3f2fd',
                        borderRadius: '12px',
                        fontSize: '12px',
                      }}
                    >
                      {label.name} ({label.confidence}%)
                    </span>
                  ))}
                </div>
              </div>
            )}

            {/* Colors */}
            {analysis.colors && analysis.colors.length > 0 && (
              <div style={{ marginBottom: '20px' }}>
                <h4 style={{ display: 'flex', alignItems: 'center', marginBottom: '10px' }}>
                  <FaPalette style={{ marginRight: '8px' }} />
                  Dominant Colors
                </h4>
                <div style={{ display: 'flex', flexWrap: 'wrap', gap: '8px' }}>
                  {analysis.colors.slice(0, 6).map((color, i) => (
                    <div
                      key={i}
                      style={{
                        display: 'flex',
                        alignItems: 'center',
                        gap: '6px',
                        padding: '4px 8px',
                        backgroundColor: '#f5f5f5',
                        borderRadius: '4px',
                        fontSize: '12px',
                      }}
                    >
                      <div
                        style={{
                          width: '20px',
                          height: '20px',
                          backgroundColor: color.hex,
                          borderRadius: '3px',
                          border: '1px solid #ddd',
                        }}
                      />
                      <span>{color.name}</span>
                    </div>
                  ))}
                </div>
              </div>
            )}

            {/* Objects */}
            {analysis.objects && analysis.objects.length > 0 && (
              <div style={{ marginBottom: '20px' }}>
                <h4 style={{ marginBottom: '10px' }}>Detected Objects</h4>
                <ul style={{ margin: 0, paddingLeft: '20px', fontSize: '13px' }}>
                  {analysis.objects.map((obj, i) => (
                    <li key={i}>
                      {obj.name} ({obj.confidence}% confidence)
                    </li>
                  ))}
                </ul>
              </div>
            )}

            {/* Faces */}
            {analysis.faces && analysis.faces.length > 0 && (
              <div style={{ marginBottom: '20px' }}>
                <h4 style={{ display: 'flex', alignItems: 'center', marginBottom: '10px' }}>
                  <FaSmile style={{ marginRight: '8px' }} />
                  Detected Faces
                </h4>
                <p style={{ fontSize: '13px', color: '#666' }}>
                  {analysis.faces.length} face{analysis.faces.length !== 1 ? 's' : ''} detected
                </p>
              </div>
            )}

            {/* Re-analyze Button */}
            <div style={{ marginTop: '20px', paddingTop: '20px', borderTop: '1px solid #e0e0e0' }}>
              <Button
                onClick={analyzeImage}
                buttonSize="s"
                fullWidth
              >
                Re-analyze Image
              </Button>
            </div>
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

### 2. Usage Rights Panel

Manage licensing and usage rights:

```typescript
async renderUploadSidebarPanel(
  uploadSidebarPanelId: string,
  ctx: RenderUploadSidebarPanelCtx
) {
  if (uploadSidebarPanelId === 'usageRights') {
    return {
      render: () => import('./components/UsageRightsPanel'),
    };
  }
}

// UsageRightsPanel.tsx
import { RenderUploadSidebarPanelCtx } from 'datocms-plugin-sdk';
import { Canvas, SelectField, TextField, TextareaField, SwitchField, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaShieldAlt, FaCalendarAlt, FaGlobe } from 'react-icons/fa';

interface UsageRights {
  license?: string;
  customLicense?: string;
  copyright?: string;
  attribution?: string;
  restrictions?: string[];
  expiryDate?: string;
  territories?: string[];
  exclusivity?: boolean;
  modelRelease?: boolean;
  propertyRelease?: boolean;
}

export default function UsageRightsPanel({ ctx }: { ctx: RenderUploadSidebarPanelCtx }) {
  const [rights, setRights] = useState<UsageRights>({});
  const [isSaving, setIsSaving] = useState(false);

  const licenses = [
    { label: 'All Rights Reserved', value: 'all-rights-reserved' },
    { label: 'Creative Commons CC0', value: 'cc0' },
    { label: 'Creative Commons BY', value: 'cc-by' },
    { label: 'Creative Commons BY-SA', value: 'cc-by-sa' },
    { label: 'Creative Commons BY-NC', value: 'cc-by-nc' },
    { label: 'Creative Commons BY-NC-SA', value: 'cc-by-nc-sa' },
    { label: 'Creative Commons BY-ND', value: 'cc-by-nd' },
    { label: 'Creative Commons BY-NC-ND', value: 'cc-by-nc-nd' },
    { label: 'Royalty Free', value: 'royalty-free' },
    { label: 'Rights Managed', value: 'rights-managed' },
    { label: 'Editorial Use Only', value: 'editorial' },
    { label: 'Custom License', value: 'custom' },
  ];

  const commonRestrictions = [
    'No commercial use',
    'No derivative works',
    'Editorial use only',
    'Internal use only',
    'Web use only',
    'Print use only',
    'Single use only',
    'Geographic restrictions apply',
  ];

  useEffect(() => {
    const existingRights = ctx.upload.customData?.usageRights || {};
    setRights(existingRights);
  }, [ctx.upload.id]);

  const toggleRestriction = (restriction: string) => {
    const current = rights.restrictions || [];
    const updated = current.includes(restriction)
      ? current.filter(r => r !== restriction)
      : [...current, restriction];
    
    setRights({ ...rights, restrictions: updated });
  };

  const saveRights = async () => {
    setIsSaving(true);
    
    try {
      await ctx.editUploadMetadata({
        usageRights: rights,
        rightsUpdatedAt: new Date().toISOString(),
        rightsUpdatedBy: ctx.currentUser.email,
      });
      
      // Also update the notes field with license info
      const licenseText = rights.license === 'custom' 
        ? rights.customLicense 
        : licenses.find(l => l.value === rights.license)?.label;
      
      if (licenseText) {
        await ctx.updateUpload({
          notes: `License: ${licenseText}${rights.copyright ? `\nCopyright: ${rights.copyright}` : ''}`,
        });
      }
      
      ctx.notice('Usage rights saved successfully');
    } catch (error) {
      ctx.alert('Failed to save usage rights');
    } finally {
      setIsSaving(false);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {/* License Type */}
        <div style={{ marginBottom: '20px' }}>
          <SelectField
            id="license"
            name="license"
            label="License Type"
            value={rights.license || ''}
            selectInputProps={{
              options: licenses,
              onChange: (value) => setRights({ ...rights, license: value }),
            }}
            hint="Select the license that applies to this asset"
          />
          
          {rights.license === 'custom' && (
            <TextareaField
              id="customLicense"
              name="customLicense"
              label="Custom License Terms"
              value={rights.customLicense || ''}
              onChange={(value) => setRights({ ...rights, customLicense: value })}
              hint="Describe the custom license terms"
              textareaInputProps={{ rows: 3 }}
              style={{ marginTop: '10px' }}
            />
          )}
        </div>

        {/* Copyright & Attribution */}
        <div style={{ marginBottom: '20px' }}>
          <TextField
            id="copyright"
            name="copyright"
            label="Copyright Holder"
            value={rights.copyright || ''}
            onChange={(value) => setRights({ ...rights, copyright: value })}
            placeholder="© 2024 Company Name"
            leftIcon={<FaShieldAlt />}
          />
          
          <TextField
            id="attribution"
            name="attribution"
            label="Required Attribution"
            value={rights.attribution || ''}
            onChange={(value) => setRights({ ...rights, attribution: value })}
            placeholder="Photo by John Doe"
            hint="Attribution text if required by license"
            style={{ marginTop: '10px' }}
          />
        </div>

        {/* Restrictions */}
        <div style={{ marginBottom: '20px' }}>
          <label style={{ display: 'block', marginBottom: '8px', fontSize: '13px', fontWeight: 'bold' }}>
            Usage Restrictions
          </label>
          <div style={{ display: 'flex', flexWrap: 'wrap', gap: '8px' }}>
            {commonRestrictions.map(restriction => (
              <label
                key={restriction}
                style={{
                  display: 'flex',
                  alignItems: 'center',
                  padding: '6px 10px',
                  backgroundColor: (rights.restrictions || []).includes(restriction) ? '#e3f2fd' : '#f5f5f5',
                  borderRadius: '4px',
                  cursor: 'pointer',
                  fontSize: '12px',
                  border: `1px solid ${(rights.restrictions || []).includes(restriction) ? '#2196F3' : '#e0e0e0'}`,
                }}
              >
                <input
                  type="checkbox"
                  checked={(rights.restrictions || []).includes(restriction)}
                  onChange={() => toggleRestriction(restriction)}
                  style={{ marginRight: '6px' }}
                />
                {restriction}
              </label>
            ))}
          </div>
        </div>

        {/* Expiry Date */}
        <div style={{ marginBottom: '20px' }}>
          <TextField
            id="expiryDate"
            name="expiryDate"
            label="License Expiry Date"
            value={rights.expiryDate || ''}
            onChange={(value) => setRights({ ...rights, expiryDate: value })}
            textInputProps={{ type: 'date' }}
            leftIcon={<FaCalendarAlt />}
            hint="Leave empty for perpetual license"
          />
        </div>

        {/* Releases */}
        <div style={{ marginBottom: '20px' }}>
          <label style={{ display: 'block', marginBottom: '8px', fontSize: '13px', fontWeight: 'bold' }}>
            Releases
          </label>
          <SwitchField
            id="modelRelease"
            name="modelRelease"
            label="Model Release Available"
            value={rights.modelRelease || false}
            onChange={(value) => setRights({ ...rights, modelRelease: value })}
            hint="Confirms permission from people in the image"
          />
          
          <SwitchField
            id="propertyRelease"
            name="propertyRelease"
            label="Property Release Available"
            value={rights.propertyRelease || false}
            onChange={(value) => setRights({ ...rights, propertyRelease: value })}
            hint="Confirms permission for recognizable property"
            style={{ marginTop: '10px' }}
          />
          
          <SwitchField
            id="exclusivity"
            name="exclusivity"
            label="Exclusive License"
            value={rights.exclusivity || false}
            onChange={(value) => setRights({ ...rights, exclusivity: value })}
            hint="This asset is exclusively licensed"
            style={{ marginTop: '10px' }}
          />
        </div>

        {/* Save Button */}
        <Button
          onClick={saveRights}
          buttonType="primary"
          fullWidth
          disabled={isSaving}
        >
          {isSaving ? 'Saving...' : 'Save Usage Rights'}
        </Button>

        {/* License Info */}
        {rights.license && rights.license !== 'custom' && (
          <div style={{
            marginTop: '16px',
            padding: '12px',
            backgroundColor: '#f5f5f5',
            borderRadius: '4px',
            fontSize: '12px',
            color: '#666',
          }}>
            <strong>License Info:</strong>
            {rights.license === 'cc0' && ' Public Domain - No rights reserved'}
            {rights.license === 'cc-by' && ' Attribution required'}
            {rights.license === 'cc-by-sa' && ' Attribution + ShareAlike'}
            {rights.license === 'cc-by-nc' && ' Attribution + NonCommercial'}
            {rights.license === 'editorial' && ' Editorial use only - not for commercial use'}
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

### 3. Performance Optimization Panel

Optimize images for web delivery:

```typescript
async renderUploadSidebarPanel(
  uploadSidebarPanelId: string,
  ctx: RenderUploadSidebarPanelCtx
) {
  if (uploadSidebarPanelId === 'imageOptimization') {
    return {
      render: () => import('./components/ImageOptimizationPanel'),
    };
  }
}

// ImageOptimizationPanel.tsx
import { RenderUploadSidebarPanelCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, SelectField, SwitchField, Spinner } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaRocket, FaImage, FaCompress } from 'react-icons/fa';

interface OptimizationSettings {
  format?: 'webp' | 'avif' | 'auto';
  quality?: number;
  resize?: {
    width?: number;
    height?: number;
    fit?: 'cover' | 'contain' | 'fill' | 'inside' | 'outside';
  };
  generateSrcSet?: boolean;
  lazyLoad?: boolean;
}

export default function ImageOptimizationPanel({ ctx }: { ctx: RenderUploadSidebarPanelCtx }) {
  const [settings, setSettings] = useState<OptimizationSettings>({
    format: 'auto',
    quality: 85,
    generateSrcSet: true,
    lazyLoad: true,
  });
  const [optimizedVersions, setOptimizedVersions] = useState<any[]>([]);
  const [isOptimizing, setIsOptimizing] = useState(false);

  const isImage = ctx.upload.mimeType?.startsWith('image/');
  const originalSize = ctx.upload.size;

  const commonSizes = [
    { label: 'Thumbnail (150px)', width: 150 },
    { label: 'Small (300px)', width: 300 },
    { label: 'Medium (600px)', width: 600 },
    { label: 'Large (1200px)', width: 1200 },
    { label: 'Full HD (1920px)', width: 1920 },
  ];

  useEffect(() => {
    // Load existing optimization settings
    const saved = ctx.upload.customData?.optimizationSettings;
    if (saved) {
      setSettings(saved);
    }
  }, [ctx.upload.id]);

  const generateOptimizedVersions = async () => {
    setIsOptimizing(true);
    const versions = [];

    try {
      // Generate different sizes
      for (const size of commonSizes) {
        if (size.width < ctx.upload.width) {
          const optimizedUrl = buildOptimizedUrl(ctx.upload.url, {
            width: size.width,
            format: settings.format,
            quality: settings.quality,
          });

          // Estimate file size reduction
          const ratio = (size.width / ctx.upload.width) ** 2;
          const estimatedSize = Math.round(originalSize * ratio * (settings.quality / 100));

          versions.push({
            label: size.label,
            width: size.width,
            url: optimizedUrl,
            estimatedSize,
            reduction: Math.round((1 - estimatedSize / originalSize) * 100),
          });
        }
      }

      setOptimizedVersions(versions);

      // Save settings
      await ctx.editUploadMetadata({
        optimizationSettings: settings,
        optimizedVersions: versions,
      });

      ctx.notice('Optimization settings saved');
    } catch (error) {
      ctx.alert('Failed to generate optimized versions');
    } finally {
      setIsOptimizing(false);
    }
  };

  const buildOptimizedUrl = (baseUrl: string, params: any): string => {
    // This would use your CDN's URL transformation API
    const url = new URL(baseUrl);
    if (params.width) url.searchParams.set('w', params.width);
    if (params.format) url.searchParams.set('fm', params.format);
    if (params.quality) url.searchParams.set('q', params.quality);
    return url.toString();
  };

  const generateSrcSetCode = () => {
    const srcset = optimizedVersions
      .map(v => `${v.url} ${v.width}w`)
      .join(',\n  ');

    const code = `<img
  src="${ctx.upload.url}"
  srcset="${srcset}"
  sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 33vw"
  alt="${ctx.upload.alt || ''}"
  loading="${settings.lazyLoad ? 'lazy' : 'eager'}"
/>`;

    navigator.clipboard.writeText(code);
    ctx.notice('Srcset code copied to clipboard!');
  };

  const formatFileSize = (bytes: number): string => {
    if (bytes < 1024) return bytes + ' B';
    if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
    return (bytes / (1024 * 1024)).toFixed(2) + ' MB';
  };

  if (!isImage) {
    return (
      <Canvas ctx={ctx}>
        <div style={{ padding: '16px', textAlign: 'center', color: '#666' }}>
          <FaImage size={24} style={{ opacity: 0.5, marginBottom: '8px' }} />
          <p style={{ fontSize: '13px' }}>
            Optimization is only available for image files.
          </p>
        </div>
      </Canvas>
    );
  }

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {/* Original Image Info */}
        <div style={{
          padding: '12px',
          backgroundColor: '#f5f5f5',
          borderRadius: '6px',
          marginBottom: '20px',
          fontSize: '13px',
        }}>
          <div style={{ marginBottom: '4px' }}>
            <strong>Original:</strong> {ctx.upload.width} × {ctx.upload.height}px
          </div>
          <div>
            <strong>Size:</strong> {formatFileSize(originalSize)}
          </div>
        </div>

        {/* Optimization Settings */}
        <div style={{ marginBottom: '20px' }}>
          <SelectField
            id="format"
            name="format"
            label="Output Format"
            value={settings.format || 'auto'}
            selectInputProps={{
              options: [
                { label: 'Auto (Best for browser)', value: 'auto' },
                { label: 'WebP', value: 'webp' },
                { label: 'AVIF', value: 'avif' },
              ],
              onChange: (value) => setSettings({ ...settings, format: value }),
            }}
            hint="Modern formats provide better compression"
          />

          <div style={{ marginTop: '15px' }}>
            <label style={{ display: 'block', marginBottom: '5px', fontSize: '13px' }}>
              Quality: {settings.quality}%
            </label>
            <input
              type="range"
              min="10"
              max="100"
              step="5"
              value={settings.quality}
              onChange={(e) => setSettings({ ...settings, quality: Number(e.target.value) })}
              style={{ width: '100%' }}
            />
          </div>

          <SwitchField
            id="srcset"
            name="srcset"
            label="Generate Responsive Images"
            value={settings.generateSrcSet || false}
            onChange={(value) => setSettings({ ...settings, generateSrcSet: value })}
            hint="Create multiple sizes for responsive designs"
            style={{ marginTop: '15px' }}
          />

          <SwitchField
            id="lazyload"
            name="lazyload"
            label="Enable Lazy Loading"
            value={settings.lazyLoad || false}
            onChange={(value) => setSettings({ ...settings, lazyLoad: value })}
            hint="Load images only when visible"
            style={{ marginTop: '10px' }}
          />
        </div>

        {/* Generate Button */}
        <Button
          onClick={generateOptimizedVersions}
          buttonType="primary"
          fullWidth
          disabled={isOptimizing}
          leftIcon={<FaRocket />}
        >
          {isOptimizing ? 'Optimizing...' : 'Generate Optimized Versions'}
        </Button>

        {/* Optimized Versions */}
        {optimizedVersions.length > 0 && (
          <div style={{ marginTop: '20px' }}>
            <h4 style={{ marginBottom: '12px', display: 'flex', alignItems: 'center' }}>
              <FaCompress style={{ marginRight: '8px' }} />
              Optimized Versions
            </h4>
            
            <div style={{ display: 'flex', flexDirection: 'column', gap: '8px' }}>
              {optimizedVersions.map((version, i) => (
                <div
                  key={i}
                  style={{
                    padding: '10px',
                    backgroundColor: '#f5f5f5',
                    borderRadius: '4px',
                    fontSize: '12px',
                  }}
                >
                  <div style={{ display: 'flex', justifyContent: 'space-between' }}>
                    <strong>{version.label}</strong>
                    <span style={{ color: '#4CAF50' }}>
                      -{version.reduction}%
                    </span>
                  </div>
                  <div style={{ color: '#666', marginTop: '2px' }}>
                    {formatFileSize(version.estimatedSize)}
                  </div>
                </div>
              ))}
            </div>

            {settings.generateSrcSet && (
              <Button
                onClick={generateSrcSetCode}
                buttonSize="s"
                fullWidth
                style={{ marginTop: '12px' }}
              >
                Copy Srcset Code
              </Button>
            )}
          </div>
        )}

        {/* Tips */}
        <div style={{
          marginTop: '20px',
          padding: '12px',
          backgroundColor: '#e3f2fd',
          borderRadius: '6px',
          fontSize: '12px',
        }}>
          <strong>Optimization Tips:</strong>
          <ul style={{ margin: '5px 0 0 0', paddingLeft: '20px' }}>
            <li>Use WebP/AVIF for better compression</li>
            <li>85% quality is usually indistinguishable from 100%</li>
            <li>Generate multiple sizes for responsive images</li>
            <li>Enable lazy loading for better performance</li>
          </ul>
        </div>
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Focused Functionality**: Each panel should have a single, clear purpose
2. **Compact UI**: Design for limited sidebar space
3. **Performance**: Keep panels lightweight and responsive
4. **Visual Feedback**: Show loading states and operation results
5. **Error Handling**: Handle errors gracefully in the constrained space
6. **Data Persistence**: Save panel data to upload metadata
7. **Progressive Enhancement**: Show basic info first, advanced features on demand
8. **Responsive Values**: Use relative units for better adaptability
9. **Keyboard Navigation**: Ensure all controls are keyboard accessible
10. **Help Text**: Provide context and guidance for complex features

## Related Hooks

- `uploadSidebarPanels` - Define which panels your plugin provides
- `renderUploadSidebar` - Render complete custom sidebars
- `renderAssetSource` - Add custom asset sources
- `uploadsDropdownActions` - Add actions to upload dropdowns

## Important Notes

- Panels integrate into the existing upload sidebar
- Multiple panels can be active simultaneously
- The Canvas component ensures consistent styling
- Check file types before offering format-specific features
- Save panel state to upload metadata for persistence
- Consider file size when performing client-side operations