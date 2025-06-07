# renderAssetSource Hook

The `renderAssetSource` hook is responsible for rendering custom asset source interfaces. Asset sources allow users to browse and select media from external services (like Unsplash, Pexels, or custom DAMs) directly within DatoCMS's media picker.

## Quick Reference

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  assetSources() {
    return [
      {
        id: 'myAssetSource',
        name: 'My Asset Library',
        icon: 'images'
      }
    ];
  },

  renderAssetSource(assetSourceId, ctx) {
    if (assetSourceId === 'myAssetSource') {
      return {
        render: () => import('./components/MyAssetSource')
      };
    }
  }
});
```

## Purpose

The `renderAssetSource` hook enables:
- Integration with external media libraries
- Custom asset selection interfaces
- Stock photo service connections
- Digital Asset Management (DAM) integration
- Icon and illustration libraries
- AI-generated image services

## Signature

```typescript
renderAssetSource(
  assetSourceId: string,
  ctx: RenderAssetSourceCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

### assetSourceId: string
The ID of the asset source being rendered (matches the ID defined in `assetSources()` hook)

### ctx: RenderAssetSourceCtx
Context object containing all properties and methods for the asset source interface

## Type Definitions

```typescript
// Complete type definitions for renderAssetSource hook
export type RenderAssetSourceHook = {
  renderAssetSource: (assetSourceId: string, ctx: RenderAssetSourceCtx) => void | {
    render: () => Promise<{ default: React.ComponentType<any> }>;
  };
};

export type RenderAssetSourceCtx = SelfResizingPluginFrameCtx<
  'renderAssetSource',
  {
    assetSourceId: string;
  },
  {
    select: (newUpload: NewUpload) => void;
    selectMultiple?: (newUploads: NewUpload[]) => void;
    cancel: () => void;
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

// New upload type
export type NewUpload = {
  resource: NewUploadResourceAsUrl | NewUploadResourceAsBase64;
  copyright?: string;
  author?: string;
  notes?: string;
  tags?: string[];
  default_field_metadata?: {
    [k: string]: {
      alt: string | null;
      title: string | null;
      custom_data: {
        [k: string]: unknown;
      };
      focal_point?: {
        x: number;
        y: number;
      } | null;
    };
  };
};

// Resource types
export type NewUploadResourceAsUrl = {
  url: string;
  headers?: Record<string, string>;
  filename?: string;
};

export type NewUploadResourceAsBase64 = {
  base64: string;
  filename: string;
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

The `renderAssetSource` hook receives a context object with:

### Base Properties
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `environment`: string - The ID of the current environment
- `theme`: Theme - Theme colors for the current project

### Specific Properties
- `assetSourceId`: string - The ID of the asset source being rendered
- `parameters`: Record<string, unknown> - Configuration parameters for this asset source

### Methods

#### Asset Selection
- `select(newUpload: NewUpload): void` - Select a single asset to import
- `selectMultiple?(newUploads: NewUpload[]): void` - Select multiple assets (if available)
- `cancel(): void` - Cancel the asset selection

#### Standard Context Methods
- `notice(message: string): void` - Show success notification
- `alert(message: string): void` - Show error notification
- `updatePluginParameters(params: Record<string, unknown>): Promise<void>`
- All other standard context methods

## Use Cases

### 1. Unsplash Integration

Connect to Unsplash API for stock photos:

```typescript
async renderAssetSource(assetSourceId: string, ctx: RenderAssetSourceCtx) {
  if (assetSourceId === 'unsplash') {
    return {
      render: () => import('./components/UnsplashAssetSource'),
    };
  }
}

// UnsplashAssetSource.tsx
import { RenderAssetSourceCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField, Button, Spinner } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

interface UnsplashPhoto {
  id: string;
  urls: {
    regular: string;
    full: string;
    thumb: string;
  };
  description: string;
  user: {
    name: string;
    links: {
      html: string;
    };
  };
}

export default function UnsplashAssetSource({ ctx }: { ctx: RenderAssetSourceCtx }) {
  const [query, setQuery] = useState('');
  const [photos, setPhotos] = useState<UnsplashPhoto[]>([]);
  const [loading, setLoading] = useState(false);
  const [selectedPhotos, setSelectedPhotos] = useState<Set<string>>(new Set());
  
  const accessKey = ctx.plugin.attributes.parameters.unsplashAccessKey as string;

  const searchPhotos = async () => {
    if (!query.trim()) return;
    
    setLoading(true);
    try {
      const response = await fetch(
        `https://api.unsplash.com/search/photos?query=${encodeURIComponent(query)}&per_page=30`,
        {
          headers: {
            'Authorization': `Client-ID ${accessKey}`,
          },
        }
      );
      
      const data = await response.json();
      setPhotos(data.results);
    } catch (error) {
      ctx.alert('Failed to search Unsplash');
    } finally {
      setLoading(false);
    }
  };

  const handleSelect = (photo: UnsplashPhoto) => {
    // Track download for Unsplash API requirements
    fetch(`https://api.unsplash.com/photos/${photo.id}/download`, {
      headers: {
        'Authorization': `Client-ID ${accessKey}`,
      },
    });

    // Select the full resolution image with metadata
    ctx.select({
      resource: {
        url: photo.urls.full,
        filename: `unsplash-${photo.id}.jpg`
      },
      copyright: `Photo by ${photo.user.name} on Unsplash`,
      author: photo.user.name,
      notes: photo.description || undefined,
      tags: ['unsplash', 'stock-photo'],
      default_field_metadata: {
        en: {
          alt: photo.description || `Photo by ${photo.user.name}`,
          title: photo.description || null,
          custom_data: {
            unsplash_id: photo.id,
            photographer_url: photo.user.links.html
          }
        }
      }
    });
  };

  const handleMultiSelect = () => {
    const uploads = Array.from(selectedPhotos).map(id => {
      const photo = photos.find(p => p.id === id);
      if (photo) {
        // Track download
        fetch(`https://api.unsplash.com/photos/${photo.id}/download`, {
          headers: {
            'Authorization': `Client-ID ${accessKey}`,
          },
        });
        
        return {
          resource: {
            url: photo.urls.full,
            filename: `unsplash-${photo.id}.jpg`
          },
          copyright: `Photo by ${photo.user.name} on Unsplash`,
          author: photo.user.name,
          notes: photo.description || undefined,
          tags: ['unsplash', 'stock-photo'],
          default_field_metadata: {
            en: {
              alt: photo.description || `Photo by ${photo.user.name}`,
              title: photo.description || null,
              custom_data: {
                unsplash_id: photo.id,
                photographer_url: photo.user.links.html
              }
            }
          }
        };
      }
      return null;
    }).filter(Boolean) as NewUpload[];

    ctx.selectMultiple(uploads);
  };

  const toggleSelection = (photoId: string) => {
    const newSelected = new Set(selectedPhotos);
    if (newSelected.has(photoId)) {
      newSelected.delete(photoId);
    } else {
      newSelected.add(photoId);
    }
    setSelectedPhotos(newSelected);
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px' }}>
        <div style={{ marginBottom: '20px' }}>
          <div style={{ display: 'flex', gap: '10px' }}>
            <TextField
              id="search"
              name="search"
              label=""
              value={query}
              onChange={setQuery}
              placeholder="Search for photos..."
              textInputProps={{
                onKeyPress: (e) => {
                  if (e.key === 'Enter') {
                    searchPhotos();
                  }
                },
              }}
            />
            <Button onClick={searchPhotos} disabled={loading}>
              Search
            </Button>
          </div>
        </div>

        {selectedPhotos.size > 0 && (
          <div style={{ marginBottom: '20px', display: 'flex', justifyContent: 'space-between' }}>
            <span>{selectedPhotos.size} photos selected</span>
            <Button onClick={handleMultiSelect} buttonType="primary">
              Import Selected
            </Button>
          </div>
        )}

        {loading && (
          <div style={{ textAlign: 'center', padding: '40px' }}>
            <Spinner />
          </div>
        )}

        <div style={{
          display: 'grid',
          gridTemplateColumns: 'repeat(auto-fill, minmax(200px, 1fr))',
          gap: '15px',
        }}>
          {photos.map((photo) => (
            <div
              key={photo.id}
              style={{
                position: 'relative',
                cursor: 'pointer',
                borderRadius: '4px',
                overflow: 'hidden',
                border: selectedPhotos.has(photo.id) ? '3px solid #0084ff' : '3px solid transparent',
              }}
            >
              <img
                src={photo.urls.thumb}
                alt={photo.description || 'Unsplash photo'}
                style={{
                  width: '100%',
                  height: '150px',
                  objectFit: 'cover',
                }}
                onClick={() => handleSelect(photo)}
              />
              
              <div style={{
                position: 'absolute',
                top: '10px',
                right: '10px',
              }}>
                <input
                  type="checkbox"
                  checked={selectedPhotos.has(photo.id)}
                  onChange={() => toggleSelection(photo.id)}
                  onClick={(e) => e.stopPropagation()}
                  style={{
                    width: '20px',
                    height: '20px',
                    cursor: 'pointer',
                  }}
                />
              </div>

              <div style={{
                position: 'absolute',
                bottom: 0,
                left: 0,
                right: 0,
                padding: '10px',
                background: 'linear-gradient(to top, rgba(0,0,0,0.8), transparent)',
                color: 'white',
                fontSize: '12px',
              }}>
                by {photo.user.name}
              </div>
            </div>
          ))}
        </div>

        <div style={{
          position: 'fixed',
          bottom: '20px',
          right: '20px',
        }}>
          <Button onClick={() => ctx.cancel()} buttonType="muted">
            Cancel
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

### 2. Custom DAM Integration

Connect to a company's Digital Asset Management system:

```typescript
async renderAssetSource(assetSourceId: string, ctx: RenderAssetSourceCtx) {
  if (assetSourceId === 'companyDAM') {
    return {
      render: () => import('./components/CompanyDAMSource'),
    };
  }
}

// CompanyDAMSource.tsx
import { RenderAssetSourceCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField, SelectField, Button, Spinner } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

interface DAMAsset {
  id: string;
  name: string;
  url: string;
  thumbnail: string;
  type: string;
  size: number;
  tags: string[];
  metadata: Record<string, any>;
}

export default function CompanyDAMSource({ ctx }: { ctx: RenderAssetSourceCtx }) {
  const [assets, setAssets] = useState<DAMAsset[]>([]);
  const [filteredAssets, setFilteredAssets] = useState<DAMAsset[]>([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState('');
  const [typeFilter, setTypeFilter] = useState('all');
  const [selectedAssets, setSelectedAssets] = useState<Set<string>>(new Set());

  const damApiUrl = ctx.parameters.damApiUrl as string;
  const damApiKey = ctx.plugin.attributes.parameters.damApiKey as string;

  useEffect(() => {
    fetchAssets();
  }, []);

  useEffect(() => {
    filterAssets();
  }, [search, typeFilter, assets]);

  const fetchAssets = async () => {
    try {
      const response = await fetch(`${damApiUrl}/assets`, {
        headers: {
          'Authorization': `Bearer ${damApiKey}`,
        },
      });
      
      const data = await response.json();
      setAssets(data.assets);
      setFilteredAssets(data.assets);
    } catch (error) {
      ctx.alert('Failed to load assets from DAM');
    } finally {
      setLoading(false);
    }
  };

  const filterAssets = () => {
    let filtered = assets;

    if (search) {
      filtered = filtered.filter(asset =>
        asset.name.toLowerCase().includes(search.toLowerCase()) ||
        asset.tags.some(tag => tag.toLowerCase().includes(search.toLowerCase()))
      );
    }

    if (typeFilter !== 'all') {
      filtered = filtered.filter(asset => asset.type === typeFilter);
    }

    setFilteredAssets(filtered);
  };

  const formatFileSize = (bytes: number): string => {
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    if (bytes === 0) return '0 Bytes';
    const i = Math.floor(Math.log(bytes) / Math.log(1024));
    return Math.round(bytes / Math.pow(1024, i) * 100) / 100 + ' ' + sizes[i];
  };

  const handleImport = async (assetIds: string[]) => {
    const assetsToImport = assets.filter(a => assetIds.includes(a.id));
    
    // Log usage in DAM system
    try {
      await fetch(`${damApiUrl}/usage`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${damApiKey}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          assetIds,
          purpose: 'datocms-import',
          user: ctx.currentUser.email,
        }),
      });
    } catch (error) {
      console.error('Failed to log usage:', error);
    }

    // Import to DatoCMS with metadata
    const uploads = assetsToImport.map(asset => ({
      resource: {
        url: asset.url,
        filename: asset.name
      },
      tags: asset.tags,
      notes: `Imported from company DAM (ID: ${asset.id})`,
      default_field_metadata: {
        en: {
          alt: asset.metadata.alt_text || asset.name,
          title: asset.metadata.title || null,
          custom_data: {
            dam_id: asset.id,
            original_size: asset.size,
            ...asset.metadata
          }
        }
      }
    }));
    
    if (uploads.length === 1) {
      ctx.select(uploads[0]);
    } else {
      ctx.selectMultiple(uploads);
    }
  };

  const assetTypes = [
    { label: 'All Types', value: 'all' },
    { label: 'Images', value: 'image' },
    { label: 'Videos', value: 'video' },
    { label: 'Documents', value: 'document' },
    { label: 'Audio', value: 'audio' },
  ];

  if (loading) {
    return (
      <Canvas ctx={ctx}>
        <div style={{ textAlign: 'center', padding: '40px' }}>
          <Spinner />
          <p>Loading assets from DAM...</p>
        </div>
      </Canvas>
    );
  }

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', height: '100vh' }}>
        {/* Sidebar */}
        <div style={{
          width: '250px',
          padding: '20px',
          borderRight: '1px solid #e0e0e0',
          overflowY: 'auto',
        }}>
          <TextField
            id="search"
            name="search"
            label="Search"
            value={search}
            onChange={setSearch}
            placeholder="Search by name or tag..."
          />
          
          <div style={{ marginTop: '20px' }}>
            <SelectField
              id="type"
              name="type"
              label="Asset Type"
              value={typeFilter}
              selectInputProps={{
                options: assetTypes,
                onChange: setTypeFilter,
              }}
            />
          </div>

          {selectedAssets.size > 0 && (
            <div style={{ marginTop: '20px' }}>
              <Button
                onClick={() => handleImport(Array.from(selectedAssets))}
                buttonType="primary"
                fullWidth
              >
                Import {selectedAssets.size} Assets
              </Button>
            </div>
          )}
        </div>

        {/* Main content */}
        <div style={{ flex: 1, padding: '20px', overflowY: 'auto' }}>
          <div style={{
            display: 'grid',
            gridTemplateColumns: 'repeat(auto-fill, minmax(250px, 1fr))',
            gap: '20px',
          }}>
            {filteredAssets.map((asset) => (
              <div
                key={asset.id}
                style={{
                  border: '1px solid #e0e0e0',
                  borderRadius: '8px',
                  overflow: 'hidden',
                  cursor: 'pointer',
                  transition: 'transform 0.2s',
                  backgroundColor: selectedAssets.has(asset.id) ? '#f0f8ff' : 'white',
                }}
                onClick={() => {
                  const newSelected = new Set(selectedAssets);
                  if (newSelected.has(asset.id)) {
                    newSelected.delete(asset.id);
                  } else {
                    newSelected.add(asset.id);
                  }
                  setSelectedAssets(newSelected);
                }}
                onDoubleClick={() => handleImport([asset.id])}
              >
                {asset.type === 'image' || asset.type === 'video' ? (
                  <img
                    src={asset.thumbnail}
                    alt={asset.name}
                    style={{
                      width: '100%',
                      height: '200px',
                      objectFit: 'cover',
                    }}
                  />
                ) : (
                  <div style={{
                    height: '200px',
                    display: 'flex',
                    alignItems: 'center',
                    justifyContent: 'center',
                    backgroundColor: '#f5f5f5',
                    fontSize: '48px',
                  }}>
                    {asset.type === 'document' ? '📄' : '🎵'}
                  </div>
                )}
                
                <div style={{ padding: '15px' }}>
                  <h4 style={{ margin: '0 0 10px 0', fontSize: '14px' }}>
                    {asset.name}
                  </h4>
                  
                  <div style={{ fontSize: '12px', color: '#666' }}>
                    <p style={{ margin: '5px 0' }}>
                      Type: {asset.type} • Size: {formatFileSize(asset.size)}
                    </p>
                    
                    {asset.tags.length > 0 && (
                      <div style={{ marginTop: '10px' }}>
                        {asset.tags.map(tag => (
                          <span
                            key={tag}
                            style={{
                              display: 'inline-block',
                              padding: '2px 8px',
                              margin: '2px',
                              backgroundColor: '#e0e0e0',
                              borderRadius: '12px',
                              fontSize: '11px',
                            }}
                          >
                            {tag}
                          </span>
                        ))}
                      </div>
                    )}
                  </div>
                </div>
              </div>
            ))}
          </div>
        </div>
      </div>

      <div style={{
        position: 'fixed',
        bottom: '20px',
        right: '20px',
      }}>
        <Button onClick={() => ctx.cancel()} buttonType="muted">
          Cancel
        </Button>
      </div>
    </Canvas>
  );
}
```

### 3. Icon Library Integration

Provide access to an icon library:

```typescript
async renderAssetSource(assetSourceId: string, ctx: RenderAssetSourceCtx) {
  if (assetSourceId === 'iconLibrary') {
    return {
      render: () => import('./components/IconLibrarySource'),
    };
  }
}

// IconLibrarySource.tsx
import { RenderAssetSourceCtx } from 'datocms-plugin-sdk';
import { Canvas, TextField, SelectField, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import * as Icons from 'react-icons/fa';

export default function IconLibrarySource({ ctx }: { ctx: RenderAssetSourceCtx }) {
  const [search, setSearch] = useState('');
  const [category, setCategory] = useState('all');
  const [selectedIcon, setSelectedIcon] = useState<string | null>(null);
  const [iconColor, setIconColor] = useState('#000000');
  const [iconSize, setIconSize] = useState(256);

  const iconCategories = [
    { label: 'All Icons', value: 'all' },
    { label: 'Social Media', value: 'social' },
    { label: 'UI Elements', value: 'ui' },
    { label: 'File Types', value: 'file' },
    { label: 'Arrows', value: 'arrow' },
  ];

  const categorizedIcons = {
    social: ['FaFacebook', 'FaTwitter', 'FaInstagram', 'FaLinkedin', 'FaYoutube'],
    ui: ['FaHome', 'FaSearch', 'FaBell', 'FaCog', 'FaUser'],
    file: ['FaFile', 'FaFileAlt', 'FaFilePdf', 'FaFileImage', 'FaFileVideo'],
    arrow: ['FaArrowUp', 'FaArrowDown', 'FaArrowLeft', 'FaArrowRight', 'FaArrowCircleUp'],
  };

  const allIcons = Object.keys(Icons).filter(name => name.startsWith('Fa'));
  
  const getFilteredIcons = () => {
    let icons = category === 'all' 
      ? allIcons 
      : categorizedIcons[category] || [];
    
    if (search) {
      icons = icons.filter(name => 
        name.toLowerCase().includes(search.toLowerCase())
      );
    }
    
    return icons;
  };

  const generateSVG = (iconName: string) => {
    const IconComponent = Icons[iconName];
    if (!IconComponent) return null;

    // Create SVG string
    const svgString = `
      <svg xmlns="http://www.w3.org/2000/svg" width="${iconSize}" height="${iconSize}" viewBox="0 0 24 24" fill="${iconColor}">
        ${IconComponent({}).props.children}
      </svg>
    `;

    // Convert to data URL
    const blob = new Blob([svgString], { type: 'image/svg+xml' });
    return URL.createObjectURL(blob);
  };

  const handleSelectIcon = async () => {
    if (!selectedIcon) return;

    const svgUrl = generateSVG(selectedIcon);
    if (svgUrl) {
      // Convert to PNG for better compatibility
      const img = new Image();
      img.src = svgUrl;
      
      img.onload = () => {
        const canvas = document.createElement('canvas');
        canvas.width = iconSize;
        canvas.height = iconSize;
        const canvasCtx = canvas.getContext('2d');
        
        if (canvasCtx) {
          canvasCtx.drawImage(img, 0, 0);
          canvas.toBlob((blob) => {
            if (blob) {
              const reader = new FileReader();
              reader.onloadend = () => {
                const base64 = reader.result as string;
                ctx.select({
                  resource: {
                    base64: base64.split(',')[1],
                    filename: `${selectedIcon.toLowerCase()}-${iconSize}px.png`
                  },
                  tags: ['icon', category],
                  default_field_metadata: {
                    en: {
                      alt: selectedIcon.replace('Fa', ''),
                      title: `${selectedIcon.replace('Fa', '')} Icon`,
                      custom_data: {
                        icon_name: selectedIcon,
                        icon_color: iconColor,
                        icon_size: iconSize
                      }
                    }
                  }
                });
              };
              reader.readAsDataURL(blob);
            }
          }, 'image/png');
        }
      };
    }
  };

  const filteredIcons = getFilteredIcons();

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px', height: '100vh', display: 'flex', flexDirection: 'column' }}>
        <div style={{ marginBottom: '20px' }}>
          <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '10px', marginBottom: '10px' }}>
            <TextField
              id="search"
              name="search"
              label="Search Icons"
              value={search}
              onChange={setSearch}
              placeholder="Search..."
            />
            
            <SelectField
              id="category"
              name="category"
              label="Category"
              value={category}
              selectInputProps={{
                options: iconCategories,
                onChange: setCategory,
              }}
            />
          </div>

          <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr 1fr', gap: '10px' }}>
            <TextField
              id="color"
              name="color"
              label="Icon Color"
              value={iconColor}
              onChange={setIconColor}
              textInputProps={{
                type: 'color',
              }}
            />
            
            <TextField
              id="size"
              name="size"
              label="Size (px)"
              value={iconSize.toString()}
              onChange={(val) => setIconSize(parseInt(val) || 256)}
              textInputProps={{
                type: 'number',
                min: 16,
                max: 1024,
              }}
            />

            <div style={{ display: 'flex', alignItems: 'flex-end' }}>
              <Button
                onClick={handleSelectIcon}
                buttonType="primary"
                fullWidth
                disabled={!selectedIcon}
              >
                Use Icon
              </Button>
            </div>
          </div>
        </div>

        <div style={{
          flex: 1,
          overflowY: 'auto',
          display: 'grid',
          gridTemplateColumns: 'repeat(auto-fill, minmax(100px, 1fr))',
          gap: '10px',
        }}>
          {filteredIcons.map((iconName) => {
            const Icon = Icons[iconName];
            if (!Icon) return null;

            return (
              <div
                key={iconName}
                style={{
                  display: 'flex',
                  flexDirection: 'column',
                  alignItems: 'center',
                  padding: '15px',
                  border: selectedIcon === iconName ? '2px solid #0084ff' : '1px solid #e0e0e0',
                  borderRadius: '8px',
                  cursor: 'pointer',
                  transition: 'all 0.2s',
                }}
                onClick={() => setSelectedIcon(iconName)}
                onDoubleClick={handleSelectIcon}
              >
                <Icon size={32} color={selectedIcon === iconName ? iconColor : '#666'} />
                <span style={{
                  fontSize: '11px',
                  marginTop: '8px',
                  textAlign: 'center',
                  wordBreak: 'break-word',
                }}>
                  {iconName.replace('Fa', '')}
                </span>
              </div>
            );
          })}
        </div>

        <div style={{
          position: 'fixed',
          bottom: '20px',
          right: '20px',
        }}>
          <Button onClick={() => ctx.cancel()} buttonType="muted">
            Cancel
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Loading States**: Always show loading indicators when fetching external data
2. **Error Handling**: Gracefully handle API failures and show meaningful error messages
3. **Selection Feedback**: Clearly indicate which assets are selected
4. **Batch Operations**: Support both single and multiple asset selection
5. **Attribution**: Include proper attribution for stock photo services
6. **Performance**: Implement pagination or lazy loading for large asset libraries
7. **Search**: Provide robust search and filtering capabilities
8. **Preview**: Show adequate previews of assets before selection
9. **Metadata**: Include comprehensive metadata with uploads (alt text, copyright, tags)
10. **Cancel Option**: Always provide a way to cancel the operation
11. **API Rate Limits**: Respect and handle rate limits from external services
12. **Localization**: Support multiple locales in default_field_metadata
13. **Accessibility**: Ensure keyboard navigation and screen reader support
14. **Resource Types**: Use appropriate resource type (URL vs base64) based on asset
15. **Filename Convention**: Use descriptive filenames for better organization

## Related Hooks

- `assetSources` - Define which asset sources your plugin provides
- `uploadSidebarPanels` - Add sidebar panels to the upload view
- `renderUploadSidebar` - Render custom upload sidebars

## Important Notes

- Asset sources appear in the media picker's source selector
- The selected URLs should be publicly accessible for DatoCMS to import them
- Consider rate limits when integrating with external APIs
- Respect licensing terms of external asset providers
- The `select()` method immediately closes the picker and imports the asset
- Use `selectMultiple()` for batch imports
- Always provide proper error handling for network requests