# renderItemFormSidebarPanel Hook

## Purpose

The `renderItemFormSidebarPanel` hook renders collapsible panels within the item form sidebar. Unlike full sidebars, these panels integrate into the existing sidebar structure, allowing you to add focused functionality without replacing the entire sidebar interface.

## Signature

```typescript
renderItemFormSidebarPanel(
  itemFormSidebarPanelId: string,
  ctx: RenderItemFormSidebarPanelCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

- `itemFormSidebarPanelId: string` - The ID of the panel being rendered
- `ctx: RenderItemFormSidebarPanelCtx` - The context object containing all item form data and methods

## Type Definitions

```typescript
// Main hook type
export type RenderItemFormSidebarPanelHook = {
  renderItemFormSidebarPanel: (
    itemFormSidebarPanelId: string,
    ctx: RenderItemFormSidebarPanelCtx
  ) => void | { render: () => Promise<{ default: React.ComponentType<any> }> };
};

// Context type extends base context with item form properties and methods
type RenderItemFormSidebarPanelCtx = Ctx<ItemFormAdditionalProperties & ItemFormAdditionalMethods & {
  itemFormSidebarPanelId: string;
  parameters?: Record<string, unknown>;
}>;

// Base context type with all properties and methods
type Ctx<AdditionalProperties = {}> = BaseProperties & BaseMethods & AdditionalProperties;

// Base properties available in all contexts
type BaseProperties = {
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken?: string;
  site: Site;
  environment: string;
  theme: Theme;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  menuItems: MenuItem[];
  schema: Schema;
  users: Partial<Record<string, User>>;
};

// Base UI methods available in all contexts
type BaseMethods = {
  notice: (message: string) => void;
  alert: (message: string, options?: { cancel?: ButtonOptions }) => Promise<void>;
  confirm: (message: string, options?: { 
    cancel?: ButtonOptions; 
    choices?: ButtonChoices 
  }) => Promise<boolean | number | null>;
  prompt: (message: string, options?: { 
    cancel?: ButtonOptions; 
    placeholder?: string; 
    initialValue?: string 
  }) => Promise<string | null>;
  openModal: (options: ModalOptions) => Promise<void>;
  navigateTo: (path: string) => void;
  openUrl: (url: string) => void;
};

// Additional properties specific to item form contexts
type ItemFormAdditionalProperties = {
  item: Item | null;
  itemId: string | null;
  itemType: ItemType;
  itemStatus: 'new' | 'draft' | 'updated' | 'published';
  isSubmitting: boolean;
  isFormDirty: boolean;
  locale: string;
  formValues: Record<string, unknown>;
  fieldHavingError: string[];
};

// Additional methods specific to item form contexts
type ItemFormAdditionalMethods = {
  setFieldValue: (fieldPath: string, value: unknown) => void;
  setMultipleFieldValues: (values: Record<string, unknown>) => void;
  scrollToField: (fieldPath: string) => void;
  toggleField: (fieldPath: string, show: boolean) => void;
  disableField: (fieldPath: string, disable: boolean) => void;
  saveCurrentItem: () => Promise<void>;
  getFieldValue: (fieldPath: string) => unknown;
};

// Plugin configuration
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description?: string;
    url: string;
    parameters: Record<string, unknown>;
    field_types: string[];
    plugin_type: 'field_editor' | 'field_addon' | 'sidebar' | 'asset_source' | 'page' | 'modal';
  };
  relationships: {
    installed_at_site?: { data: { id: string; type: 'site' } };
  };
};

// User type definitions
type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    avatar_url?: string;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_publish_to_production: boolean;
    two_factor_authentication_active?: boolean;
  };
  relationships: {
    role: { data: { id: string; type: 'role' } };
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
  };
  relationships: Record<string, any>;
};

type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
    company?: string;
  };
  relationships: Record<string, any>;
};

type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
    logo_url?: string;
  };
  relationships: Record<string, any>;
};

// Role permissions
type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_publish_to_production: boolean;
    can_manage_webhooks: boolean;
    can_manage_environments: boolean;
    can_access_build_triggers: boolean;
    can_manage_seo_readability_analysis: boolean;
    positive_item_type_permissions: string[];
    negative_item_type_permissions: string[];
    meta_permissions: Record<string, any>;
  };
  relationships: Record<string, any>;
};

// Site configuration
type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    domain?: string;
    internal_domain: string;
    locales: string[];
    timezone: string;
    theme: Record<string, any>;
    no_index: boolean;
    global_seo?: Record<string, any>;
    favicon?: any;
    favicon_meta_tags?: any[];
  };
  relationships: Record<string, any>;
};

// Theme configuration
type Theme = {
  primary_color?: string;
  dark_color?: string;
  light_color?: string;
  accent_color?: string;
};

// ItemType (model) definition
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    tree: boolean;
    modular_block: boolean;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'table';
    has_singleton_item: boolean;
    hint?: string;
    preview_field?: string;
    title_field?: string;
    image_preview_field?: string;
    excerpt_field?: string;
    workflow?: any;
    ordering_direction?: 'asc' | 'desc';
    ordering_field?: any;
    some_locales_required?: boolean;
    versioning_enabled?: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: 'field' }> };
    fieldsets: { data: Array<{ id: string; type: 'fieldset' }> };
    singleton_item?: { data: { id: string; type: 'item' } | null };
    ordering_field?: { data: { id: string; type: 'field' } | null };
    title_field?: { data: { id: string; type: 'field' } | null };
    image_preview_field?: { data: { id: string; type: 'field' } | null };
    excerpt_field?: { data: { id: string; type: 'field' } | null };
  };
};

// Field definition
type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    field_type: 'boolean' | 'color' | 'date' | 'date_time' | 'file' | 'float' | 'gallery' | 'integer' | 'json' | 'lat_lon' | 'link' | 'links' | 'rich_text' | 'seo' | 'single_line_string' | 'slug' | 'structured_text' | 'text' | 'video';
    api_key: string;
    localized: boolean;
    validators: Record<string, any>;
    position: number;
    hint?: string;
    default_value?: any;
    appearance: {
      addons: any[];
      editor: string;
      parameters: Record<string, any>;
      type: 'addon' | 'editor' | 'presentation';
    };
    deep_filtering_enabled?: boolean;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fieldset?: { data: { id: string; type: 'fieldset' } | null };
  };
};

// Fieldset (field group)
type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    hint?: string;
    collapsible: boolean;
    start_collapsed: boolean;
    position: number;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fields: { data: Array<{ id: string; type: 'field' }> };
  };
};

// Item definition
type Item = {
  id: string;
  type: 'item';
  attributes: Record<string, any>;
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at?: string;
    publication_scheduled_at?: string;
    unpublishing_scheduled_at?: string;
    first_published_at?: string;
    is_valid: boolean;
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    status: 'draft' | 'updated' | 'published';
    current_version: string;
    stage?: string;
  };
};

// Menu structure
type MenuItem = {
  id: string;
  type: 'menu_item';
  attributes: {
    label: string;
    position: number;
    external_url?: string;
    open_in_new_tab?: boolean;
  };
  relationships: {
    item_type?: { data: { id: string; type: 'item_type' } | null };
    parent?: { data: { id: string; type: 'menu_item' } | null };
    children?: { data: Array<{ id: string; type: 'menu_item' }> };
  };
};

// Schema information
type Schema = {
  models: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
};

// UI interaction types
type ButtonOptions = {
  label: string;
  value: boolean;
  intent?: 'primary' | 'secondary' | 'negative';
};

type ButtonChoices = Array<{
  label: string;
  value: boolean | number;
  intent?: 'primary' | 'secondary' | 'negative';
}>;

type ModalOptions = {
  id: string;
  title: string;
  width?: 'xs' | 's' | 'm' | 'l' | 'xl' | 'fullWidth';
  parameters?: Record<string, unknown>;
};
```

## Use Cases

### 1. Item Metadata Panel

Display and edit metadata about the content item:

```typescript
async renderItemFormSidebarPanel(
  itemFormSidebarPanelId: string,
  ctx: RenderItemFormSidebarPanelCtx
) {
  if (itemFormSidebarPanelId === 'itemMetadata') {
    return {
      render: () => import('./components/ItemMetadataPanel'),
    };
  }
}

// ItemMetadataPanel.tsx
import { RenderItemFormSidebarPanelCtx } from 'datocms-plugin-sdk';
import { Canvas } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaInfo, FaClock, FaUser, FaHistory } from 'react-icons/fa';

export default function ItemMetadataPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [wordCount, setWordCount] = useState(0);
  const [readingTime, setReadingTime] = useState(0);
  const [lastSaved, setLastSaved] = useState<Date | null>(null);

  useEffect(() => {
    calculateMetrics();
  }, [ctx.formValues]);

  const calculateMetrics = () => {
    // Calculate word count from text fields
    let totalWords = 0;
    
    Object.values(ctx.itemType.attributes.fields).forEach(field => {
      if (field.fieldType === 'text' || field.fieldType === 'string') {
        const value = ctx.formValues[field.apiKey];
        if (typeof value === 'string') {
          totalWords += value.split(/\s+/).filter(word => word.length > 0).length;
        }
      }
    });

    setWordCount(totalWords);
    setReadingTime(Math.ceil(totalWords / 200)); // Average reading speed

    if (ctx.item?.meta.updatedAt) {
      setLastSaved(new Date(ctx.item.meta.updatedAt));
    }
  };

  const formatDate = (date: Date) => {
    const now = new Date();
    const diff = now.getTime() - date.getTime();
    const minutes = Math.floor(diff / 60000);
    const hours = Math.floor(minutes / 60);
    const days = Math.floor(hours / 24);

    if (minutes < 1) return 'Just now';
    if (minutes < 60) return `${minutes} minute${minutes > 1 ? 's' : ''} ago`;
    if (hours < 24) return `${hours} hour${hours > 1 ? 's' : ''} ago`;
    return `${days} day${days > 1 ? 's' : ''} ago`;
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {/* Basic Info */}
        <div style={{ marginBottom: '20px' }}>
          <div style={{ 
            display: 'flex', 
            alignItems: 'center',
            marginBottom: '12px',
            color: '#666'
          }}>
            <FaInfo style={{ marginRight: '8px' }} />
            <strong>Item Information</strong>
          </div>
          
          <div style={{ fontSize: '13px', lineHeight: '1.6' }}>
            <div style={{ marginBottom: '4px' }}>
              <strong>ID:</strong> {ctx.itemId || 'New Item'}
            </div>
            <div style={{ marginBottom: '4px' }}>
              <strong>Type:</strong> {ctx.itemType.attributes.name}
            </div>
            <div style={{ marginBottom: '4px' }}>
              <strong>Status:</strong> 
              <span style={{
                marginLeft: '5px',
                padding: '2px 6px',
                backgroundColor: ctx.itemStatus === 'published' ? '#4CAF50' : '#FF9800',
                color: 'white',
                borderRadius: '3px',
                fontSize: '11px'
              }}>
                {ctx.itemStatus.toUpperCase()}
              </span>
            </div>
          </div>
        </div>

        {/* Content Metrics */}
        <div style={{ marginBottom: '20px' }}>
          <div style={{ 
            display: 'flex', 
            alignItems: 'center',
            marginBottom: '12px',
            color: '#666'
          }}>
            <FaClock style={{ marginRight: '8px' }} />
            <strong>Content Metrics</strong>
          </div>
          
          <div style={{
            display: 'grid',
            gridTemplateColumns: '1fr 1fr',
            gap: '10px',
          }}>
            <div style={{
              padding: '10px',
              backgroundColor: '#f5f5f5',
              borderRadius: '6px',
              textAlign: 'center',
            }}>
              <div style={{ fontSize: '24px', fontWeight: 'bold', color: '#333' }}>
                {wordCount.toLocaleString()}
              </div>
              <div style={{ fontSize: '12px', color: '#666' }}>Words</div>
            </div>
            
            <div style={{
              padding: '10px',
              backgroundColor: '#f5f5f5',
              borderRadius: '6px',
              textAlign: 'center',
            }}>
              <div style={{ fontSize: '24px', fontWeight: 'bold', color: '#333' }}>
                {readingTime}
              </div>
              <div style={{ fontSize: '12px', color: '#666' }}>Min Read</div>
            </div>
          </div>
        </div>

        {/* Author & Dates */}
        {ctx.item && (
          <div style={{ marginBottom: '20px' }}>
            <div style={{ 
              display: 'flex', 
              alignItems: 'center',
              marginBottom: '12px',
              color: '#666'
            }}>
              <FaUser style={{ marginRight: '8px' }} />
              <strong>Author & Dates</strong>
            </div>
            
            <div style={{ fontSize: '13px', lineHeight: '1.8' }}>
              <div style={{ marginBottom: '4px' }}>
                <strong>Created by:</strong> {ctx.item.meta.createdBy || 'Unknown'}
              </div>
              <div style={{ marginBottom: '4px' }}>
                <strong>Created:</strong> {new Date(ctx.item.meta.createdAt).toLocaleDateString()}
              </div>
              {lastSaved && (
                <div style={{ marginBottom: '4px' }}>
                  <strong>Last saved:</strong> {formatDate(lastSaved)}
                </div>
              )}
              {ctx.item.meta.firstPublishedAt && (
                <div>
                  <strong>First published:</strong> {new Date(ctx.item.meta.firstPublishedAt).toLocaleDateString()}
                </div>
              )}
            </div>
          </div>
        )}

        {/* Quick Actions */}
        <div style={{ 
          paddingTop: '16px',
          borderTop: '1px solid #e0e0e0'
        }}>
          <button
            onClick={() => {
              const url = `${window.location.origin}/editor/item-types/${ctx.itemType.id}/items/${ctx.itemId}/history`;
              window.open(url, '_blank');
            }}
            style={{
              display: 'flex',
              alignItems: 'center',
              width: '100%',
              padding: '8px 12px',
              backgroundColor: '#f5f5f5',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer',
              fontSize: '13px',
            }}
          >
            <FaHistory style={{ marginRight: '8px' }} />
            View Version History
          </button>
        </div>
      </div>
    </Canvas>
  );
}
```

### 2. Related Content Panel

Show and manage related content items:

```typescript
async renderItemFormSidebarPanel(
  itemFormSidebarPanelId: string,
  ctx: RenderItemFormSidebarPanelCtx
) {
  if (itemFormSidebarPanelId === 'relatedContent') {
    return {
      render: () => import('./components/RelatedContentPanel'),
    };
  }
}

// RelatedContentPanel.tsx
import { RenderItemFormSidebarPanelCtx } from 'datocms-plugin-sdk';
import { Canvas, Spinner } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaLink, FaExternalLinkAlt, FaPlus } from 'react-icons/fa';

interface RelatedItem {
  id: string;
  title: string;
  type: string;
  status: string;
}

export default function RelatedContentPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [relatedItems, setRelatedItems] = useState<RelatedItem[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    if (ctx.itemId) {
      findRelatedContent();
    } else {
      setLoading(false);
    }
  }, [ctx.itemId]);

  const findRelatedContent = async () => {
    setLoading(true);
    const related: RelatedItem[] = [];

    try {
      const client = ctx.createClient();

      // Find items that link to this item
      for (const itemType of Object.values(ctx.itemTypes)) {
        const linkFields = Object.values(itemType.attributes.fields).filter(
          field => field.fieldType === 'link' || field.fieldType === 'links'
        );

        if (linkFields.length > 0) {
          const items = await client.items.list({
            filter: {
              type: itemType.id,
              fields: linkFields.reduce((acc, field) => {
                acc[field.apiKey] = { anyIn: [ctx.itemId] };
                return acc;
              }, {}),
            },
            page: { limit: 5 },
          });

          items.forEach(item => {
            related.push({
              id: item.id,
              title: item.title || item.name || `${itemType.attributes.name} #${item.id}`,
              type: itemType.attributes.name,
              status: item.meta.status,
            });
          });
        }
      }

      // Find items referenced by this item
      Object.entries(ctx.formValues).forEach(([fieldKey, value]) => {
        const field = Object.values(ctx.itemType.attributes.fields).find(
          f => f.apiKey === fieldKey
        );

        if (field && (field.fieldType === 'link' || field.fieldType === 'links')) {
          const linkedIds = Array.isArray(value) ? value : [value];
          
          linkedIds.filter(Boolean).forEach(linkedId => {
            // In a real implementation, you'd fetch these items
            related.push({
              id: linkedId,
              title: `Referenced Item ${linkedId}`,
              type: 'Unknown',
              status: 'unknown',
            });
          });
        }
      });

      setRelatedItems(related);
    } catch (error) {
      console.error('Failed to find related content:', error);
    } finally {
      setLoading(false);
    }
  };

  const openItem = (itemId: string) => {
    ctx.navigateTo(`/editor/items/${itemId}/edit`);
  };

  if (loading) {
    return (
      <Canvas ctx={ctx}>
        <div style={{ padding: '16px', textAlign: 'center' }}>
          <Spinner />
          <p style={{ marginTop: '8px', fontSize: '13px', color: '#666' }}>
            Finding related content...
          </p>
        </div>
      </Canvas>
    );
  }

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {relatedItems.length === 0 ? (
          <div style={{ textAlign: 'center', color: '#666', fontSize: '13px' }}>
            <FaLink size={24} style={{ marginBottom: '8px', opacity: 0.5 }} />
            <p>No related content found</p>
          </div>
        ) : (
          <>
            <div style={{ marginBottom: '12px', fontSize: '13px', color: '#666' }}>
              Found {relatedItems.length} related item{relatedItems.length !== 1 ? 's' : ''}
            </div>
            
            {relatedItems.map(item => (
              <div
                key={item.id}
                style={{
                  padding: '10px',
                  marginBottom: '8px',
                  backgroundColor: '#f5f5f5',
                  borderRadius: '6px',
                  cursor: 'pointer',
                  transition: 'background-color 0.2s',
                }}
                onClick={() => openItem(item.id)}
                onMouseEnter={(e) => {
                  e.currentTarget.style.backgroundColor = '#e8e8e8';
                }}
                onMouseLeave={(e) => {
                  e.currentTarget.style.backgroundColor = '#f5f5f5';
                }}
              >
                <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
                  <div style={{ flex: 1 }}>
                    <div style={{ fontWeight: 'bold', fontSize: '13px', marginBottom: '2px' }}>
                      {item.title}
                    </div>
                    <div style={{ fontSize: '11px', color: '#666' }}>
                      {item.type} • {item.status}
                    </div>
                  </div>
                  <FaExternalLinkAlt size={12} style={{ color: '#666' }} />
                </div>
              </div>
            ))}
          </>
        )}

        {/* Add Related Content Button */}
        <div style={{ marginTop: '16px', paddingTop: '16px', borderTop: '1px solid #e0e0e0' }}>
          <button
            onClick={() => {
              ctx.openModal({
                id: 'itemSelector',
                title: 'Select Related Items',
                width: 'medium',
                parameters: {
                  itemTypes: Object.keys(ctx.itemTypes),
                  multiple: true,
                },
              }).then((selectedIds) => {
                if (selectedIds && selectedIds.length > 0) {
                  // Handle adding related items
                  ctx.notice(`Added ${selectedIds.length} related items`);
                  findRelatedContent();
                }
              });
            }}
            style={{
              display: 'flex',
              alignItems: 'center',
              width: '100%',
              padding: '8px 12px',
              backgroundColor: '#f5f5f5',
              border: 'none',
              borderRadius: '4px',
              cursor: 'pointer',
              fontSize: '13px',
            }}
          >
            <FaPlus style={{ marginRight: '8px' }} />
            Add Related Content
          </button>
        </div>
      </div>
    </Canvas>
  );
}
```

### 3. Publishing Schedule Panel

Manage publishing schedule and status:

```typescript
async renderItemFormSidebarPanel(
  itemFormSidebarPanelId: string,
  ctx: RenderItemFormSidebarPanelCtx
) {
  if (itemFormSidebarPanelId === 'publishingSchedule') {
    return {
      render: () => import('./components/PublishingSchedulePanel'),
    };
  }
}

// PublishingSchedulePanel.tsx
import { RenderItemFormSidebarPanelCtx } from 'datocms-plugin-sdk';
import { Canvas, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaCalendarAlt, FaClock, FaRocket } from 'react-icons/fa';

export default function PublishingSchedulePanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [scheduledDate, setScheduledDate] = useState('');
  const [scheduledTime, setScheduledTime] = useState('');
  const [timezone, setTimezone] = useState(Intl.DateTimeFormat().resolvedOptions().timeZone);

  useEffect(() => {
    // Load existing schedule if any
    const schedule = ctx.formValues._publishSchedule;
    if (schedule) {
      const date = new Date(schedule);
      setScheduledDate(date.toISOString().split('T')[0]);
      setScheduledTime(date.toTimeString().substring(0, 5));
    }
  }, [ctx.formValues._publishSchedule]);

  const schedulePublication = async () => {
    if (!scheduledDate || !scheduledTime) {
      ctx.alert('Please select both date and time');
      return;
    }

    const scheduledDateTime = new Date(`${scheduledDate}T${scheduledTime}:00`);
    
    if (scheduledDateTime <= new Date()) {
      ctx.alert('Scheduled time must be in the future');
      return;
    }

    ctx.setFieldValue('_publishSchedule', scheduledDateTime.toISOString());
    await ctx.saveCurrentItem();
    
    ctx.notice(`Publication scheduled for ${scheduledDateTime.toLocaleString()}`);
  };

  const cancelSchedule = async () => {
    ctx.setFieldValue('_publishSchedule', null);
    await ctx.saveCurrentItem();
    setScheduledDate('');
    setScheduledTime('');
    ctx.notice('Publishing schedule cancelled');
  };

  const publishNow = async () => {
    const confirmed = await ctx.openConfirm({
      title: 'Publish Now',
      content: 'Are you sure you want to publish this item immediately?',
      cancel: { label: 'Cancel', value: false },
      choices: [{ label: 'Publish', value: true, intent: 'primary' }],
    });

    if (confirmed) {
      try {
        const client = ctx.createClient();
        await client.items.publish(ctx.itemId!);
        ctx.notice('Item published successfully');
        // Reload to update status
        window.location.reload();
      } catch (error) {
        ctx.alert('Failed to publish: ' + error.message);
      }
    }
  };

  const isScheduled = !!ctx.formValues._publishSchedule;

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {/* Current Status */}
        <div style={{ marginBottom: '20px' }}>
          <div style={{ fontSize: '13px', color: '#666', marginBottom: '8px' }}>
            Current Status
          </div>
          <div style={{
            padding: '12px',
            backgroundColor: ctx.itemStatus === 'published' ? '#E8F5E9' : '#FFF3E0',
            borderRadius: '6px',
            textAlign: 'center',
          }}>
            <div style={{
              fontSize: '16px',
              fontWeight: 'bold',
              color: ctx.itemStatus === 'published' ? '#2E7D32' : '#F57C00',
            }}>
              {ctx.itemStatus.toUpperCase()}
            </div>
            {ctx.item?.meta.publishedAt && (
              <div style={{ fontSize: '11px', marginTop: '4px' }}>
                Published: {new Date(ctx.item.meta.publishedAt).toLocaleString()}
              </div>
            )}
          </div>
        </div>

        {/* Schedule Section */}
        {!isScheduled ? (
          <div>
            <div style={{ 
              display: 'flex', 
              alignItems: 'center',
              marginBottom: '12px',
              fontSize: '13px',
              color: '#666'
            }}>
              <FaCalendarAlt style={{ marginRight: '8px' }} />
              <strong>Schedule Publishing</strong>
            </div>

            <div style={{ marginBottom: '12px' }}>
              <label style={{ display: 'block', marginBottom: '4px', fontSize: '12px' }}>
                Date
              </label>
              <input
                type="date"
                value={scheduledDate}
                onChange={(e) => setScheduledDate(e.target.value)}
                min={new Date().toISOString().split('T')[0]}
                style={{
                  width: '100%',
                  padding: '6px',
                  border: '1px solid #ddd',
                  borderRadius: '4px',
                  fontSize: '13px',
                }}
              />
            </div>

            <div style={{ marginBottom: '12px' }}>
              <label style={{ display: 'block', marginBottom: '4px', fontSize: '12px' }}>
                Time
              </label>
              <input
                type="time"
                value={scheduledTime}
                onChange={(e) => setScheduledTime(e.target.value)}
                style={{
                  width: '100%',
                  padding: '6px',
                  border: '1px solid #ddd',
                  borderRadius: '4px',
                  fontSize: '13px',
                }}
              />
            </div>

            <div style={{ marginBottom: '16px' }}>
              <label style={{ display: 'block', marginBottom: '4px', fontSize: '12px' }}>
                Timezone
              </label>
              <div style={{
                padding: '6px',
                backgroundColor: '#f5f5f5',
                borderRadius: '4px',
                fontSize: '13px',
              }}>
                {timezone}
              </div>
            </div>

            <Button
              onClick={schedulePublication}
              buttonType="primary"
              fullWidth
              leftIcon={<FaClock />}
              disabled={!scheduledDate || !scheduledTime}
            >
              Schedule Publication
            </Button>
          </div>
        ) : (
          <div>
            <div style={{ 
              display: 'flex', 
              alignItems: 'center',
              marginBottom: '12px',
              fontSize: '13px',
              color: '#666'
            }}>
              <FaClock style={{ marginRight: '8px' }} />
              <strong>Scheduled Publication</strong>
            </div>

            <div style={{
              padding: '12px',
              backgroundColor: '#E3F2FD',
              borderRadius: '6px',
              marginBottom: '12px',
            }}>
              <div style={{ fontSize: '14px', fontWeight: 'bold', marginBottom: '4px' }}>
                {new Date(ctx.formValues._publishSchedule).toLocaleDateString()}
              </div>
              <div style={{ fontSize: '13px' }}>
                {new Date(ctx.formValues._publishSchedule).toLocaleTimeString()}
              </div>
            </div>

            <Button
              onClick={cancelSchedule}
              buttonType="negative"
              fullWidth
            >
              Cancel Schedule
            </Button>
          </div>
        )}

        {/* Quick Publish */}
        {ctx.itemStatus !== 'published' && (
          <div style={{ marginTop: '16px', paddingTop: '16px', borderTop: '1px solid #e0e0e0' }}>
            <Button
              onClick={publishNow}
              fullWidth
              leftIcon={<FaRocket />}
            >
              Publish Now
            </Button>
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Compact Design**: Design for limited space in sidebar panels
2. **Clear Purpose**: Each panel should have a single, clear purpose
3. **Collapsible Content**: Panels are collapsible by default, organize content accordingly
4. **Performance**: Keep panels lightweight for fast loading
5. **Visual Hierarchy**: Use proper spacing and typography for readability
6. **Interactive Elements**: Ensure all controls are easily clickable
7. **Loading States**: Show loading indicators for async data
8. **Error Handling**: Handle errors gracefully in limited space
9. **Responsive Content**: Content should adapt to different sidebar widths
10. **Meaningful Titles**: Panel titles should clearly indicate content

## Related Hooks

- `itemFormSidebarPanels` - Define which panels your plugin provides
- `renderItemFormSidebar` - Render complete custom sidebars
- `renderItemFormOutlet` - Add outlets to other areas of the form
- `renderFieldExtension` - Create custom field editors

## Important Notes

- Panels integrate into the existing sidebar structure
- Multiple panels can be active simultaneously
- Panels are collapsible to save space
- The Canvas component ensures consistent styling
- Panel content has access to full form context
- Changes made in panels immediately affect form state