# renderItemCollectionOutlet Hook

## Purpose

The `renderItemCollectionOutlet` hook renders custom UI components in designated outlet areas within the content browser. These outlets allow you to add functionality like bulk actions, custom filters, or data visualizations to the item collection views.

## Signature

```typescript
renderItemCollectionOutlet(
  itemCollectionOutletId: string,
  ctx: RenderItemCollectionOutletCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

### itemCollectionOutletId: string
The ID of the outlet being rendered

### ctx: RenderItemCollectionOutletCtx
Context object containing all properties and methods for the outlet

## Type Definitions

```typescript
// Complete type definitions for renderItemCollectionOutlet hook
export type RenderItemCollectionOutletHook = {
  renderItemCollectionOutlet: (
    itemCollectionOutletId: string,
    ctx: RenderItemCollectionOutletCtx
  ) => void | {
    render: () => Promise<{ default: React.ComponentType<any> }>;
  };
};

export type RenderItemCollectionOutletCtx = SelfResizingPluginFrameCtx<
  'renderItemCollectionOutlet',
  {
    itemCollectionOutletId: string;
    itemType: ItemType;
    items: Item[];
    totalItems: number;
    locale: string;
    page: number;
    perPage: number;
    filters: Record<string, any>;
    ordering: Array<[string, 'asc' | 'desc']>;
  },
  {
    reloadItems: () => Promise<void>;
    setFilter: (field: string, value: any) => void;
    clearFilters: () => void;
    setOrdering: (field: string, direction: 'asc' | 'desc') => void;
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

// Item type definition
type Item = {
  id: string;
  type: string;
  attributes: Record<string, any>;
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
    [key: string]: any;
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at?: string;
    first_published_at?: string;
    publication_scheduled_at?: string;
    unpublishing_scheduled_at?: string;
    stage?: 'draft' | 'updated' | 'published';
    current_version: string;
    status: 'draft' | 'updated' | 'published';
    is_valid: boolean;
    is_current_version_valid: boolean;
    created_by?: string;
    updated_by?: string;
  };
};

// ItemType definition
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
type ReactNode = any; // Simplified for this context
type FullConnectParameters = any; // Simplified for this context
```

## Context Properties

The `renderItemCollectionOutlet` hook receives a context object with:

### Base Properties
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `environment`: string - The ID of the current environment
- `theme`: Theme - Theme colors for the current project
- `itemTypes`: Record<string, ItemType> - All models indexed by ID

### Specific Properties
- `itemCollectionOutletId`: string - The ID of the outlet being rendered
- `itemType`: ItemType - The current item type/model being viewed
- `items`: Item[] - Currently displayed items in the collection
- `totalItems`: number - Total number of items (including pagination)
- `locale`: string - Current content locale
- `page`: number - Current page number
- `perPage`: number - Items per page
- `filters`: Record<string, any> - Active filters
- `ordering`: Array<[string, 'asc' | 'desc']> - Active sorting

### Methods
- Standard context methods (notice, alert, openModal, etc.)
- `reloadItems()`: Promise<void> - Refresh the item collection
- `setFilter(field: string, value: any)`: void - Apply a filter
- `clearFilters()`: void - Clear all filters
- `setOrdering(field: string, direction: 'asc' | 'desc')`: void - Change sorting

## Use Cases

### 1. Collection Statistics Dashboard

Display statistics about the current collection:

```typescript
async renderItemCollectionOutlet(
  itemCollectionOutletId: string,
  ctx: RenderItemCollectionOutletCtx
) {
  if (itemCollectionOutletId === 'collectionStats') {
    return {
      render: () => import('./components/CollectionStatsDashboard'),
    };
  }
}

// CollectionStatsDashboard.tsx
import { RenderItemCollectionOutletCtx } from 'datocms-plugin-sdk';
import { Canvas, Section } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { BarChart, Bar, PieChart, Pie, Cell, XAxis, YAxis, CartesianGrid, Tooltip, Legend } from 'recharts';

export default function CollectionStatsDashboard({ ctx }: { ctx: RenderItemCollectionOutletCtx }) {
  const [stats, setStats] = useState({
    byStatus: {},
    byAuthor: {},
    byMonth: {},
    fieldCompleteness: {},
  });

  useEffect(() => {
    calculateStats();
  }, [ctx.items]);

  const calculateStats = () => {
    const newStats = {
      byStatus: {},
      byAuthor: {},
      byMonth: {},
      fieldCompleteness: {},
    };

    // Count by status
    ctx.items.forEach(item => {
      const status = item.meta.status;
      newStats.byStatus[status] = (newStats.byStatus[status] || 0) + 1;

      // Count by author
      const author = item.meta.createdBy;
      newStats.byAuthor[author] = (newStats.byAuthor[author] || 0) + 1;

      // Count by creation month
      const month = new Date(item.meta.createdAt).toLocaleDateString('en-US', { 
        year: 'numeric', 
        month: 'short' 
      });
      newStats.byMonth[month] = (newStats.byMonth[month] || 0) + 1;

      // Check field completeness
      Object.values(ctx.itemType.attributes.fields).forEach(field => {
        if (!newStats.fieldCompleteness[field.label]) {
          newStats.fieldCompleteness[field.label] = { filled: 0, empty: 0 };
        }
        
        if (item[field.apiKey]) {
          newStats.fieldCompleteness[field.label].filled++;
        } else {
          newStats.fieldCompleteness[field.label].empty++;
        }
      });
    });

    setStats(newStats);
  };

  const statusColors = {
    draft: '#FFA07A',
    published: '#90EE90',
    updated: '#87CEEB',
  };

  const statusData = Object.entries(stats.byStatus).map(([status, count]) => ({
    name: status,
    value: count as number,
  }));

  const monthData = Object.entries(stats.byMonth)
    .sort((a, b) => new Date(a[0]).getTime() - new Date(b[0]).getTime())
    .slice(-6) // Last 6 months
    .map(([month, count]) => ({
      name: month,
      items: count as number,
    }));

  const completenessData = Object.entries(stats.fieldCompleteness).map(([field, data]: [string, any]) => ({
    field,
    completeness: Math.round((data.filled / (data.filled + data.empty)) * 100),
  }));

  return (
    <Canvas ctx={ctx}>
      <Section title={`${ctx.itemType.attributes.name} Collection Statistics`}>
        <div style={{ 
          display: 'grid', 
          gridTemplateColumns: 'repeat(auto-fit, minmax(300px, 1fr))', 
          gap: '20px',
          padding: '20px'
        }}>
          {/* Status Distribution */}
          <div style={{ 
            backgroundColor: 'white', 
            padding: '20px', 
            borderRadius: '8px',
            boxShadow: '0 2px 4px rgba(0,0,0,0.1)'
          }}>
            <h3 style={{ marginBottom: '15px' }}>Status Distribution</h3>
            <PieChart width={280} height={200}>
              <Pie
                data={statusData}
                cx={140}
                cy={100}
                innerRadius={40}
                outerRadius={80}
                paddingAngle={5}
                dataKey="value"
              >
                {statusData.map((entry, index) => (
                  <Cell key={`cell-${index}`} fill={statusColors[entry.name] || '#8884d8'} />
                ))}
              </Pie>
              <Tooltip />
              <Legend />
            </PieChart>
            <div style={{ marginTop: '10px', fontSize: '14px' }}>
              Total items on this page: {ctx.items.length}
              <br />
              Total items in collection: {ctx.totalItems}
            </div>
          </div>

          {/* Items Over Time */}
          <div style={{ 
            backgroundColor: 'white', 
            padding: '20px', 
            borderRadius: '8px',
            boxShadow: '0 2px 4px rgba(0,0,0,0.1)'
          }}>
            <h3 style={{ marginBottom: '15px' }}>Items Created Over Time</h3>
            <BarChart width={280} height={200} data={monthData}>
              <CartesianGrid strokeDasharray="3 3" />
              <XAxis dataKey="name" />
              <YAxis />
              <Tooltip />
              <Bar dataKey="items" fill="#8884d8" />
            </BarChart>
          </div>

          {/* Field Completeness */}
          <div style={{ 
            backgroundColor: 'white', 
            padding: '20px', 
            borderRadius: '8px',
            boxShadow: '0 2px 4px rgba(0,0,0,0.1)'
          }}>
            <h3 style={{ marginBottom: '15px' }}>Field Completeness</h3>
            <div style={{ maxHeight: '200px', overflowY: 'auto' }}>
              {completenessData.map(({ field, completeness }) => (
                <div key={field} style={{ marginBottom: '10px' }}>
                  <div style={{ 
                    display: 'flex', 
                    justifyContent: 'space-between',
                    marginBottom: '2px',
                    fontSize: '12px'
                  }}>
                    <span>{field}</span>
                    <span>{completeness}%</span>
                  </div>
                  <div style={{ 
                    height: '20px', 
                    backgroundColor: '#e0e0e0',
                    borderRadius: '10px',
                    overflow: 'hidden'
                  }}>
                    <div style={{
                      height: '100%',
                      width: `${completeness}%`,
                      backgroundColor: completeness > 80 ? '#4CAF50' : completeness > 50 ? '#FFC107' : '#F44336',
                      transition: 'width 0.3s ease'
                    }} />
                  </div>
                </div>
              ))}
            </div>
          </div>
        </div>
      </Section>
    </Canvas>
  );
}
```

### 2. Advanced Filter Panel

Add custom filtering options:

```typescript
async renderItemCollectionOutlet(
  itemCollectionOutletId: string,
  ctx: RenderItemCollectionOutletCtx
) {
  if (itemCollectionOutletId === 'advancedFilters') {
    return {
      render: () => import('./components/AdvancedFilterPanel'),
    };
  }
}

// AdvancedFilterPanel.tsx
import { RenderItemCollectionOutletCtx } from 'datocms-plugin-sdk';
import { Canvas, Section, TextField, SelectField, Button, SwitchField } from 'datocms-react-ui';
import { useState } from 'react';

export default function AdvancedFilterPanel({ ctx }: { ctx: RenderItemCollectionOutletCtx }) {
  const [dateRange, setDateRange] = useState({ start: '', end: '' });
  const [missingFields, setMissingFields] = useState<string[]>([]);
  const [hasImages, setHasImages] = useState<boolean | null>(null);
  const [minWordCount, setMinWordCount] = useState('');

  const applyFilters = async () => {
    ctx.notice('Applying advanced filters...');

    // Filter by date range
    if (dateRange.start || dateRange.end) {
      ctx.setFilter('created_at', {
        gte: dateRange.start || undefined,
        lte: dateRange.end || undefined,
      });
    }

    // Filter by missing fields
    if (missingFields.length > 0) {
      missingFields.forEach(fieldApiKey => {
        ctx.setFilter(fieldApiKey, { exists: false });
      });
    }

    // Filter by image presence
    if (hasImages !== null) {
      const imageFields = Object.values(ctx.itemType.attributes.fields)
        .filter(f => f.fieldType === 'file' || f.fieldType === 'gallery')
        .map(f => f.apiKey);

      imageFields.forEach(fieldApiKey => {
        ctx.setFilter(fieldApiKey, { exists: hasImages });
      });
    }

    await ctx.reloadItems();
  };

  const clearAllFilters = async () => {
    ctx.clearFilters();
    setDateRange({ start: '', end: '' });
    setMissingFields([]);
    setHasImages(null);
    setMinWordCount('');
    await ctx.reloadItems();
    ctx.notice('All filters cleared');
  };

  const textFields = Object.values(ctx.itemType.attributes.fields)
    .filter(f => f.fieldType === 'text' || f.fieldType === 'string');

  return (
    <Canvas ctx={ctx}>
      <Section title="Advanced Filters" startOpen={false}>
        <div style={{ padding: '15px' }}>
          {/* Date Range Filter */}
          <div style={{ marginBottom: '20px' }}>
            <h4 style={{ marginBottom: '10px' }}>Date Range</h4>
            <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '10px' }}>
              <TextField
                id="startDate"
                name="startDate"
                label="From"
                value={dateRange.start}
                onChange={(value) => setDateRange({ ...dateRange, start: value })}
                textInputProps={{ type: 'date' }}
              />
              <TextField
                id="endDate"
                name="endDate"
                label="To"
                value={dateRange.end}
                onChange={(value) => setDateRange({ ...dateRange, end: value })}
                textInputProps={{ type: 'date' }}
              />
            </div>
          </div>

          {/* Missing Fields Filter */}
          <div style={{ marginBottom: '20px' }}>
            <h4 style={{ marginBottom: '10px' }}>Items Missing Fields</h4>
            <div style={{ maxHeight: '150px', overflowY: 'auto' }}>
              {Object.values(ctx.itemType.attributes.fields).map(field => (
                <label
                  key={field.id}
                  style={{ 
                    display: 'block', 
                    padding: '5px',
                    cursor: 'pointer'
                  }}
                >
                  <input
                    type="checkbox"
                    checked={missingFields.includes(field.apiKey)}
                    onChange={(e) => {
                      if (e.target.checked) {
                        setMissingFields([...missingFields, field.apiKey]);
                      } else {
                        setMissingFields(missingFields.filter(f => f !== field.apiKey));
                      }
                    }}
                    style={{ marginRight: '8px' }}
                  />
                  {field.label}
                </label>
              ))}
            </div>
          </div>

          {/* Image Presence Filter */}
          <div style={{ marginBottom: '20px' }}>
            <h4 style={{ marginBottom: '10px' }}>Media Assets</h4>
            <SelectField
              id="hasImages"
              name="hasImages"
              label="Has Images/Media"
              value={hasImages === null ? '' : hasImages.toString()}
              selectInputProps={{
                options: [
                  { label: 'Any', value: '' },
                  { label: 'With Images', value: 'true' },
                  { label: 'Without Images', value: 'false' },
                ],
                onChange: (value) => {
                  setHasImages(value === '' ? null : value === 'true');
                },
              }}
            />
          </div>

          {/* Word Count Filter */}
          {textFields.length > 0 && (
            <div style={{ marginBottom: '20px' }}>
              <h4 style={{ marginBottom: '10px' }}>Content Length</h4>
              <TextField
                id="minWords"
                name="minWords"
                label="Minimum Word Count"
                value={minWordCount}
                onChange={setMinWordCount}
                textInputProps={{ 
                  type: 'number',
                  min: '0'
                }}
                hint="Filter items by minimum word count in text fields"
              />
            </div>
          )}

          {/* Action Buttons */}
          <div style={{ 
            display: 'flex', 
            gap: '10px',
            marginTop: '20px',
            paddingTop: '20px',
            borderTop: '1px solid #e0e0e0'
          }}>
            <Button
              onClick={applyFilters}
              buttonType="primary"
              fullWidth
            >
              Apply Filters
            </Button>
            <Button
              onClick={clearAllFilters}
              buttonType="muted"
              fullWidth
            >
              Clear All
            </Button>
          </div>
        </div>
      </Section>
    </Canvas>
  );
}
```

### 3. Bulk Operations Toolbar

Add bulk operations for the collection:

```typescript
async renderItemCollectionOutlet(
  itemCollectionOutletId: string,
  ctx: RenderItemCollectionOutletCtx
) {
  if (itemCollectionOutletId === 'bulkOperations') {
    return {
      render: () => import('./components/BulkOperationsToolbar'),
    };
  }
}

// BulkOperationsToolbar.tsx
import { RenderItemCollectionOutletCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, ButtonGroup, SelectField } from 'datocms-react-ui';
import { useState } from 'react';
import { FaEdit, FaCopy, FaTrash, FaFileExport } from 'react-icons/fa';

export default function BulkOperationsToolbar({ ctx }: { ctx: RenderItemCollectionOutletCtx }) {
  const [selectedItems, setSelectedItems] = useState<Set<string>>(new Set());
  const [bulkEditField, setBulkEditField] = useState('');
  const [bulkEditValue, setBulkEditValue] = useState('');

  const selectAll = () => {
    const allIds = new Set(ctx.items.map(item => item.id));
    setSelectedItems(allIds);
  };

  const deselectAll = () => {
    setSelectedItems(new Set());
  };

  const bulkUpdate = async () => {
    if (!bulkEditField || selectedItems.size === 0) {
      ctx.alert('Please select items and a field to update');
      return;
    }

    const confirmed = await ctx.openConfirm({
      title: 'Bulk Update',
      content: `Update ${selectedItems.size} items?`,
      cancel: { label: 'Cancel', value: false },
      choices: [{ label: 'Update', value: true, intent: 'primary' }]
    });

    if (!confirmed) return;

    ctx.notice('Updating items...');
    
    const client = ctx.createClient();
    const results = await Promise.allSettled(
      Array.from(selectedItems).map(async (itemId) => {
        const item = ctx.items.find(i => i.id === itemId);
        if (!item) return;

        const updatedData = {
          ...item,
          [bulkEditField]: bulkEditValue,
        };

        return client.items.update(itemId, updatedData);
      })
    );

    const succeeded = results.filter(r => r.status === 'fulfilled').length;
    const failed = results.filter(r => r.status === 'rejected').length;

    if (failed > 0) {
      ctx.alert(`Updated ${succeeded} items, ${failed} failed`);
    } else {
      ctx.notice(`Successfully updated ${succeeded} items`);
    }

    await ctx.reloadItems();
    setSelectedItems(new Set());
  };

  const duplicateItems = async () => {
    if (selectedItems.size === 0) {
      ctx.alert('Please select items to duplicate');
      return;
    }

    const confirmed = await ctx.openConfirm({
      title: 'Duplicate Items',
      content: `Duplicate ${selectedItems.size} items?`,
      cancel: { label: 'Cancel', value: false },
      choices: [{ label: 'Duplicate', value: true, intent: 'primary' }]
    });

    if (!confirmed) return;

    ctx.notice('Duplicating items...');
    
    const client = ctx.createClient();
    const results = await Promise.allSettled(
      Array.from(selectedItems).map(async (itemId) => {
        const item = ctx.items.find(i => i.id === itemId);
        if (!item) return;

        const { id, meta, ...itemData } = item;
        const duplicatedData = {
          ...itemData,
          // Add suffix to title if it exists
          title: itemData.title ? `${itemData.title} (Copy)` : undefined,
        };

        return client.items.create({
          itemType: ctx.itemType.id,
          ...duplicatedData,
        });
      })
    );

    const succeeded = results.filter(r => r.status === 'fulfilled').length;
    ctx.notice(`Successfully duplicated ${succeeded} items`);
    
    await ctx.reloadItems();
    setSelectedItems(new Set());
  };

  const exportSelected = () => {
    const itemsToExport = ctx.items.filter(item => selectedItems.has(item.id));
    
    const csvContent = [
      // Headers
      ['ID', 'Status', ...Object.values(ctx.itemType.attributes.fields).map(f => f.label)],
      // Data rows
      ...itemsToExport.map(item => [
        item.id,
        item.meta.status,
        ...Object.values(ctx.itemType.attributes.fields).map(field => {
          const value = item[field.apiKey];
          return value ? JSON.stringify(value) : '';
        })
      ])
    ].map(row => row.map(cell => `"${String(cell).replace(/"/g, '""')}"`).join(','))
     .join('\n');

    const blob = new Blob([csvContent], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${ctx.itemType.attributes.name}-export-${new Date().toISOString()}.csv`;
    a.click();
    
    ctx.notice(`Exported ${itemsToExport.length} items`);
  };

  const editableFields = Object.values(ctx.itemType.attributes.fields)
    .filter(f => !['id', 'created_at', 'updated_at'].includes(f.apiKey))
    .map(f => ({ label: f.label, value: f.apiKey }));

  return (
    <Canvas ctx={ctx}>
      <div style={{ 
        padding: '15px',
        backgroundColor: '#f8f9fa',
        borderRadius: '8px',
        marginBottom: '20px'
      }}>
        {/* Selection Controls */}
        <div style={{ marginBottom: '15px' }}>
          <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center' }}>
            <div>
              <strong>{selectedItems.size}</strong> of {ctx.items.length} items selected
              {ctx.totalItems > ctx.items.length && (
                <span style={{ fontSize: '12px', color: '#666', marginLeft: '10px' }}>
                  ({ctx.totalItems} total in collection)
                </span>
              )}
            </div>
            <ButtonGroup>
              <Button onClick={selectAll} buttonSize="s">
                Select All
              </Button>
              <Button onClick={deselectAll} buttonSize="s">
                Deselect All
              </Button>
            </ButtonGroup>
          </div>
        </div>

        {/* Item Selection */}
        <div style={{ 
          maxHeight: '200px', 
          overflowY: 'auto',
          marginBottom: '15px',
          border: '1px solid #e0e0e0',
          borderRadius: '4px',
          padding: '10px',
          backgroundColor: 'white'
        }}>
          {ctx.items.map(item => (
            <label
              key={item.id}
              style={{ 
                display: 'block', 
                padding: '5px',
                cursor: 'pointer',
                borderBottom: '1px solid #f0f0f0'
              }}
            >
              <input
                type="checkbox"
                checked={selectedItems.has(item.id)}
                onChange={(e) => {
                  const newSelected = new Set(selectedItems);
                  if (e.target.checked) {
                    newSelected.add(item.id);
                  } else {
                    newSelected.delete(item.id);
                  }
                  setSelectedItems(newSelected);
                }}
                style={{ marginRight: '8px' }}
              />
              {item.title || item.name || item.id}
            </label>
          ))}
        </div>

        {/* Bulk Edit Section */}
        <div style={{ 
          marginBottom: '15px',
          padding: '15px',
          backgroundColor: 'white',
          borderRadius: '4px',
          border: '1px solid #e0e0e0'
        }}>
          <h4 style={{ marginBottom: '10px' }}>Bulk Edit</h4>
          <div style={{ display: 'grid', gridTemplateColumns: '1fr 2fr auto', gap: '10px', alignItems: 'end' }}>
            <SelectField
              id="field"
              name="field"
              label="Field"
              value={bulkEditField}
              selectInputProps={{
                options: editableFields,
                onChange: setBulkEditField,
              }}
            />
            <TextField
              id="value"
              name="value"
              label="New Value"
              value={bulkEditValue}
              onChange={setBulkEditValue}
            />
            <Button
              onClick={bulkUpdate}
              buttonType="primary"
              leftIcon={<FaEdit />}
              disabled={selectedItems.size === 0}
            >
              Update
            </Button>
          </div>
        </div>

        {/* Action Buttons */}
        <div style={{ display: 'flex', gap: '10px' }}>
          <Button
            onClick={duplicateItems}
            leftIcon={<FaCopy />}
            disabled={selectedItems.size === 0}
          >
            Duplicate Selected
          </Button>
          <Button
            onClick={exportSelected}
            leftIcon={<FaFileExport />}
            disabled={selectedItems.size === 0}
          >
            Export Selected
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Performance**: Be mindful of performance when processing large collections
2. **Visual Hierarchy**: Design outlets to complement, not compete with, the main UI
3. **Responsive Design**: Ensure outlets work well at different screen sizes
4. **State Management**: Handle state changes efficiently, especially with large datasets
5. **Error Handling**: Gracefully handle errors in data processing
6. **Loading States**: Show appropriate loading indicators for async operations
7. **User Feedback**: Provide clear feedback for user actions
8. **Accessibility**: Ensure custom controls are keyboard accessible
9. **Batch Operations**: Use Promise.allSettled for resilient batch operations
10. **Data Refresh**: Use ctx.reloadItems() after modifying data

## Related Hooks

- `itemCollectionOutlets` - Define which outlets your plugin provides
- `itemFormOutlets` - Add outlets to individual item forms
- `itemsDropdownActions` - Add dropdown actions for multiple items
- `renderPage` - Render full-page views

## Important Notes

- Outlets render in specific areas of the content browser UI
- Consider the available space when designing outlet components
- Use the Canvas component for consistent styling
- The context provides access to the current collection state
- Filters and sorting can be modified programmatically
- Changes to items require reloading the collection