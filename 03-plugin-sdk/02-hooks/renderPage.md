# renderPage Hook

## Purpose

The `renderPage` hook is responsible for rendering custom pages within the DatoCMS interface. These pages can be accessed through navigation items defined in `mainNavigationTabs`, `settingsAreaSidebarItemGroups`, or `contentAreaSidebarItems` hooks. This hook enables plugins to create full-featured administrative interfaces, dashboards, tools, and custom workflows.

## Signature

```typescript
renderPage(pageId: string, ctx: RenderPageCtx): void
```

## Parameters

- `pageId`: string - The unique identifier of the page to render (as defined in navigation hooks)
- `ctx`: RenderPageCtx - The context object containing all necessary data and methods

## Type Definitions

```typescript
// Hook signature
export type RenderPageHook = {
  /**
   * This function will be called when the plugin needs to render a page (see
   * the `mainNavigationTabs`, `settingsAreaSidebarItemGroups` and
   * `contentAreaSidebarItems` functions)
   *
   * @tag pages
   */
  renderPage: (pageId: string, ctx: RenderPageCtx) => void;
};

// Main context type
export type RenderPageCtx = SelfResizingPluginFrameCtx<
  'renderPage',
  {
    /** The ID of the page that needs to be rendered */
    pageId: string;
  }
>;

// Expanded full context type
type RenderPageCtx = {
  // Mode and frame properties
  mode: 'renderPage';
  bodyPadding: [number, number, number, number];
  
  // Page-specific properties
  pageId: string;
  location: {
    pathname: string;
    search: string;
    hash: string;
  };
  
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
  searchItems: (query: ItemSearchQuery) => Promise<Item[]>;
  
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
  navigateTo: (path: string | NavigationTarget) => Promise<void>;
  
  // Sizing utilities
  startAutoResizer: () => Promise<void>;
  stopAutoResizer: () => Promise<void>;
  updateHeight: (height: number) => Promise<void>;
  isAutoResizerActive: () => boolean;
  setHeight: (height: number) => Promise<void>;
  
  // Frame communication
  getSettings: () => Promise<RenderPageCtx>;
};

// Supporting types
type Theme = {
  primaryColor: string;
  accentColor: string;
  semiTransparentAccentColor: string;
  lightColor: string;
  darkColor: string;
};

type Plugin = {
  id: string;
  attributes: {
    name: string;
    description?: string;
    homepage_url?: string;
    preview_image?: string;
    parameters: Record<string, unknown>;
    field_types?: string[];
    icon?: string;
  };
};

type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    full_name: string;
    is_2fa_active: boolean;
    created_at: string;
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    full_name: string;
    created_at: string;
  };
};

type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    full_name: string;
    created_at: string;
    company: string | null;
  };
};

type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
    created_at: string;
  };
};

type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_site: boolean;
    can_manage_users: boolean;
    can_publish_to_production: boolean;
    can_manage_webhooks: boolean;
    can_manage_access_tokens: boolean;
    environments_access: 'all' | 'primary_only' | 'sandbox_only';
    positive_item_type_permissions: Array<{
      item_type: string;
      environment: string;
      action: 'all' | 'read' | 'edit' | 'create' | 'delete' | 'publish';
    }>;
    negative_item_type_permissions: Array<{
      item_type: string;
      environment: string;
      action: 'all' | 'read' | 'edit' | 'create' | 'delete' | 'publish';
    }>;
  };
};

type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    domain: string | null;
    frontend_url: string | null;
    global_seo: object | null;
    favicon: object | null;
    locales: string[];
    timezone: string;
  };
};

type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    collection_appearance: 'table' | 'gallery';
    all_locales_required: boolean;
    has_singleton_item: boolean;
    sortable: boolean;
    modular_block: boolean;
    tree: boolean;
    draft_mode_active: boolean;
  };
};

type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    api_key: string;
    field_type: string;
    appearance: Record<string, unknown>;
    validators: Record<string, unknown>;
    position: number;
    hint: string | null;
    default_value: unknown;
    localized: boolean;
  };
};

type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    hint: string | null;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
  };
};

type Item = {
  id: string;
  type: 'item';
  attributes: Record<string, unknown>;
  meta: {
    created_at: string;
    updated_at: string;
    published_at: string | null;
    first_published_at: string | null;
    publication_scheduled_at: string | null;
    unpublishing_scheduled_at: string | null;
    status: 'draft' | 'published' | 'updated';
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    current_version: string;
    stage: string | null;
  };
};

type Upload = {
  id: string;
  type: 'upload';
  attributes: {
    path: string;
    basename: string;
    filename: string;
    url: string;
    size: number;
    width: number | null;
    height: number | null;
    format: string;
    alt: string | null;
    title: string | null;
    notes: string | null;
    md5: string;
    created_at: string;
    tags: string[];
    smart_tags: string[];
    colors: Array<{ hex: string; alpha: number }>;
    blurhash: string | null;
    thumbhash: string | null;
    is_image: boolean;
  };
};

type ItemSearchQuery = {
  filter?: {
    type?: string;
    query?: string;
    fields?: Record<string, unknown>;
  };
  page?: {
    offset?: number;
    limit?: number;
  };
  orderBy?: string | string[];
};

type NavigationTarget = {
  pageId: string;
  params?: Record<string, string>;
  query?: Record<string, string>;
  locale?: string;
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

type SiteApiClient = {
  items: {
    list(params?: object): Promise<Item[]>;
    find(id: string): Promise<Item>;
    create(params: object): Promise<Item>;
    update(id: string, params: object): Promise<Item>;
    destroy(id: string): Promise<void>;
    publish(id: string): Promise<Item>;
    unpublish(id: string): Promise<Item>;
  };
  uploads: {
    list(params?: object): Promise<Upload[]>;
    find(id: string): Promise<Upload>;
    create(params: object): Promise<Upload>;
    update(id: string, params: object): Promise<Upload>;
    destroy(id: string): Promise<void>;
  };
};
```

## Context Properties

The hook receives an extensive context object that includes:

### Base Properties
All standard context properties like `plugin`, `currentUser`, `site`, etc.

### Page-Specific Properties
- `pageId`: string - The ID of the current page
- `location`: Object containing:
  - `pathname`: string - The current path
  - `search`: string - Query string parameters
  - `hash`: string - URL hash fragment

### Frame Properties
- `mode`: 'renderPage' - The current rendering mode
- Frame sizing and positioning methods

## Use Cases

### 1. Analytics Dashboard

Create a comprehensive analytics dashboard:

```typescript
renderPage(pageId: string, ctx: RenderPageCtx) {
  switch (pageId) {
    case 'analytics-dashboard':
      return {
        render: () => import('./pages/AnalyticsDashboard'),
      };
  }
}

// AnalyticsDashboard.tsx
import { RenderPageCtx } from 'datocms-plugin-sdk';
import { Canvas, Button, DateRangePicker } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { LineChart, BarChart, PieChart } from 'recharts';

export default function AnalyticsDashboard({ ctx }: { ctx: RenderPageCtx }) {
  const [dateRange, setDateRange] = useState({ start: null, end: null });
  const [metrics, setMetrics] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadAnalytics();
  }, [dateRange]);

  const loadAnalytics = async () => {
    setLoading(true);
    try {
      // Fetch content metrics
      const [articles, products, pageviews] = await Promise.all([
        ctx.searchItems({
          filter: { type: 'article' },
          page: { limit: 500 }
        }),
        ctx.searchItems({
          filter: { type: 'product' },
          page: { limit: 500 }
        }),
        fetchPageviews(dateRange, ctx.plugin.attributes.parameters.analyticsApiKey)
      ]);

      const publishedArticles = articles.filter(a => a.meta.status === 'published');
      const draftArticles = articles.filter(a => a.meta.status === 'draft');

      setMetrics({
        content: {
          totalArticles: articles.length,
          publishedArticles: publishedArticles.length,
          draftArticles: draftArticles.length,
          totalProducts: products.length,
          avgArticleLength: calculateAvgLength(articles),
          popularCategories: getPopularCategories(articles)
        },
        traffic: pageviews,
        authors: getAuthorStats(articles)
      });
    } catch (error) {
      ctx.alert('Failed to load analytics data');
    } finally {
      setLoading(false);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px', maxWidth: '1200px', margin: '0 auto' }}>
        <h1>Content Analytics Dashboard</h1>
        
        <div style={{ marginBottom: '20px' }}>
          <DateRangePicker
            value={dateRange}
            onChange={setDateRange}
            presets={[
              { label: 'Last 7 days', days: 7 },
              { label: 'Last 30 days', days: 30 },
              { label: 'Last 90 days', days: 90 }
            ]}
          />
          <Button onClick={loadAnalytics} disabled={loading}>
            Refresh Data
          </Button>
        </div>

        {loading ? (
          <Spinner />
        ) : (
          <div style={{ display: 'grid', gap: '20px' }}>
            {/* Summary Cards */}
            <div style={{ display: 'grid', gridTemplateColumns: 'repeat(4, 1fr)', gap: '15px' }}>
              <MetricCard
                title="Total Articles"
                value={metrics.content.totalArticles}
                change="+12%"
                icon="file-alt"
              />
              <MetricCard
                title="Published"
                value={metrics.content.publishedArticles}
                change="+8%"
                icon="check-circle"
              />
              <MetricCard
                title="Drafts"
                value={metrics.content.draftArticles}
                change="-3%"
                icon="edit"
              />
              <MetricCard
                title="Avg. Length"
                value={`${metrics.content.avgArticleLength} words`}
                icon="align-left"
              />
            </div>

            {/* Charts */}
            <div style={{ display: 'grid', gridTemplateColumns: '2fr 1fr', gap: '20px' }}>
              <div style={{ background: 'white', padding: '20px', borderRadius: '8px' }}>
                <h3>Content Publication Trend</h3>
                <LineChart width={600} height={300} data={metrics.traffic}>
                  <Line type="monotone" dataKey="published" stroke="#8884d8" />
                  <Line type="monotone" dataKey="views" stroke="#82ca9d" />
                </LineChart>
              </div>

              <div style={{ background: 'white', padding: '20px', borderRadius: '8px' }}>
                <h3>Content by Category</h3>
                <PieChart width={300} height={300}>
                  <Pie data={metrics.content.popularCategories} dataKey="count" nameKey="name" />
                </PieChart>
              </div>
            </div>

            {/* Author Performance */}
            <div style={{ background: 'white', padding: '20px', borderRadius: '8px' }}>
              <h3>Top Authors</h3>
              <AuthorTable authors={metrics.authors} />
            </div>
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

### 2. Content Calendar

Build an editorial calendar interface:

```typescript
renderPage(pageId: string, ctx: RenderPageCtx) {
  if (pageId === 'content-calendar') {
    return {
      render: () => import('./pages/ContentCalendar'),
    };
  }
}

// ContentCalendar.tsx
import { RenderPageCtx } from 'datocms-plugin-sdk';
import { Canvas } from 'datocms-react-ui';
import { Calendar, momentLocalizer } from 'react-big-calendar';
import moment from 'moment';
import { useState, useEffect } from 'react';

const localizer = momentLocalizer(moment);

export default function ContentCalendar({ ctx }: { ctx: RenderPageCtx }) {
  const [events, setEvents] = useState([]);
  const [view, setView] = useState('month');

  useEffect(() => {
    loadScheduledContent();
  }, []);

  const loadScheduledContent = async () => {
    // Load all content with publish dates
    const contentTypes = ['article', 'event', 'campaign', 'product_launch'];
    const allContent = [];

    for (const type of contentTypes) {
      const items = await ctx.searchItems({
        filter: { type },
        page: { limit: 500 }
      });

      items.forEach(item => {
        // Scheduled publish
        if (item.attributes.publish_at) {
          allContent.push({
            id: item.id,
            title: `📅 Publish: ${item.attributes.title}`,
            start: new Date(item.attributes.publish_at),
            end: new Date(item.attributes.publish_at),
            type: 'publish',
            itemType: type,
            item
          });
        }

        // Events
        if (type === 'event' && item.attributes.start_date) {
          allContent.push({
            id: `event-${item.id}`,
            title: `🎉 ${item.attributes.title}`,
            start: new Date(item.attributes.start_date),
            end: new Date(item.attributes.end_date || item.attributes.start_date),
            type: 'event',
            item
          });
        }

        // Deadlines
        if (item.attributes.deadline) {
          allContent.push({
            id: `deadline-${item.id}`,
            title: `⏰ Deadline: ${item.attributes.title}`,
            start: new Date(item.attributes.deadline),
            end: new Date(item.attributes.deadline),
            type: 'deadline',
            item
          });
        }
      });
    }

    setEvents(allContent);
  };

  const handleSelectEvent = async (event) => {
    const actions = await ctx.openModal({
      id: 'event-actions',
      title: event.title,
      width: 's',
      parameters: { event }
    });

    if (actions === 'edit') {
      await ctx.navigateTo(`/editor/item_types/${event.item.relationships.item_type.data.id}/items/${event.item.id}/edit`);
    }
  };

  const handleSelectSlot = async ({ start, end }) => {
    const result = await ctx.openModal({
      id: 'create-scheduled-content',
      title: 'Schedule New Content',
      width: 'm',
      parameters: { 
        defaultDate: start.toISOString(),
        contentTypes 
      }
    });

    if (result) {
      await loadScheduledContent();
    }
  };

  const eventStyleGetter = (event) => {
    const colors = {
      publish: '#28a745',
      event: '#007bff',
      deadline: '#dc3545',
      draft: '#6c757d'
    };

    return {
      style: {
        backgroundColor: colors[event.type] || '#6c757d',
        borderRadius: '5px',
        opacity: 0.8,
        color: 'white',
        border: '0px',
        display: 'block'
      }
    };
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ height: 'calc(100vh - 100px)', padding: '20px' }}>
        <div style={{ marginBottom: '20px', display: 'flex', justifyContent: 'space-between' }}>
          <h1>Content Calendar</h1>
          <div>
            <Button onClick={() => setView('month')}>Month</Button>
            <Button onClick={() => setView('week')}>Week</Button>
            <Button onClick={() => setView('agenda')}>Agenda</Button>
          </div>
        </div>

        <Calendar
          localizer={localizer}
          events={events}
          startAccessor="start"
          endAccessor="end"
          style={{ height: '100%' }}
          onSelectEvent={handleSelectEvent}
          onSelectSlot={handleSelectSlot}
          selectable
          view={view}
          onView={setView}
          eventPropGetter={eventStyleGetter}
          components={{
            event: EventComponent,
            toolbar: CustomToolbar
          }}
        />
      </div>
    </Canvas>
  );
}
```

### 3. Import/Export Tool

Create a data management interface:

```typescript
renderPage(pageId: string, ctx: RenderPageCtx) {
  if (pageId === 'import-export-tool') {
    return {
      render: () => import('./pages/ImportExportTool'),
    };
  }
}

// ImportExportTool.tsx
export default function ImportExportTool({ ctx }: { ctx: RenderPageCtx }) {
  const [activeTab, setActiveTab] = useState('import');
  const [importFile, setImportFile] = useState(null);
  const [exportConfig, setExportConfig] = useState({
    models: [],
    format: 'json',
    includeAssets: false
  });

  const handleImport = async () => {
    if (!importFile) {
      ctx.alert('Please select a file to import');
      return;
    }

    ctx.notice('Starting import...');

    try {
      const data = await parseImportFile(importFile);
      const results = await importData(data, ctx);

      await ctx.openModal({
        id: 'import-results',
        title: 'Import Complete',
        width: 'l',
        parameters: { results }
      });
    } catch (error) {
      ctx.alert(`Import failed: ${error.message}`);
    }
  };

  const handleExport = async () => {
    ctx.notice('Preparing export...');

    try {
      const data = await exportData(exportConfig, ctx);
      const file = formatExportData(data, exportConfig.format);

      // Download file
      const blob = new Blob([file], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `datocms-export-${Date.now()}.${exportConfig.format}`;
      a.click();

      ctx.notice('Export completed');
    } catch (error) {
      ctx.alert(`Export failed: ${error.message}`);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px', maxWidth: '800px', margin: '0 auto' }}>
        <h1>Import/Export Tool</h1>

        <Tabs value={activeTab} onChange={setActiveTab}>
          <Tab value="import" label="Import" />
          <Tab value="export" label="Export" />
          <Tab value="templates" label="Templates" />
        </Tabs>

        {activeTab === 'import' && (
          <ImportTab 
            file={importFile}
            onFileChange={setImportFile}
            onImport={handleImport}
            ctx={ctx}
          />
        )}

        {activeTab === 'export' && (
          <ExportTab
            config={exportConfig}
            onConfigChange={setExportConfig}
            onExport={handleExport}
            ctx={ctx}
          />
        )}

        {activeTab === 'templates' && (
          <TemplatesTab ctx={ctx} />
        )}
      </div>
    </Canvas>
  );
}
```

### 4. SEO Analysis Tool

Build an SEO analysis dashboard:

```typescript
renderPage(pageId: string, ctx: RenderPageCtx) {
  if (pageId === 'seo-analyzer') {
    return {
      render: () => import('./pages/SeoAnalyzer'),
    };
  }
}

// SeoAnalyzer.tsx
export default function SeoAnalyzer({ ctx }: { ctx: RenderPageCtx }) {
  const [selectedModel, setSelectedModel] = useState('');
  const [analysisResults, setAnalysisResults] = useState(null);
  const [analyzing, setAnalyzing] = useState(false);

  const analyzeContent = async () => {
    setAnalyzing(true);
    const results = {
      totalItems: 0,
      issues: [],
      warnings: [],
      opportunities: []
    };

    try {
      const items = await ctx.searchItems({
        filter: { type: selectedModel },
        page: { limit: 500 }
      });

      results.totalItems = items.length;

      for (const item of items) {
        const itemIssues = analyzeSEO(item);
        results.issues.push(...itemIssues.issues);
        results.warnings.push(...itemIssues.warnings);
        results.opportunities.push(...itemIssues.opportunities);
      }

      setAnalysisResults(results);
    } catch (error) {
      ctx.alert('Analysis failed');
    } finally {
      setAnalyzing(false);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px', maxWidth: '1000px', margin: '0 auto' }}>
        <h1>SEO Analysis Tool</h1>

        <div style={{ marginBottom: '20px' }}>
          <SelectField
            label="Select Content Type"
            value={selectedModel}
            onChange={setSelectedModel}
            selectInputProps={{
              options: Object.values(ctx.itemTypes)
                .filter(m => m.attributes.has_seo_fields)
                .map(m => ({
                  label: m.attributes.name,
                  value: m.attributes.api_key
                }))
            }}
          />
          <Button 
            onClick={analyzeContent}
            disabled={!selectedModel || analyzing}
            buttonType="primary"
          >
            {analyzing ? 'Analyzing...' : 'Analyze SEO'}
          </Button>
        </div>

        {analysisResults && (
          <SeoResultsDisplay results={analysisResults} ctx={ctx} />
        )}
      </div>
    </Canvas>
  );
}
```

### 5. Workflow Management

Create a workflow dashboard:

```typescript
renderPage(pageId: string, ctx: RenderPageCtx) {
  if (pageId === 'workflow-dashboard') {
    return {
      render: () => import('./pages/WorkflowDashboard'),
    };
  }
}

// WorkflowDashboard.tsx
export default function WorkflowDashboard({ ctx }: { ctx: RenderPageCtx }) {
  const [workflows, setWorkflows] = useState([]);
  const [filter, setFilter] = useState('all');

  useEffect(() => {
    loadWorkflows();
  }, [filter]);

  const loadWorkflows = async () => {
    // Load items in various workflow states
    const workflowItems = await ctx.searchItems({
      filter: {
        fields: {
          workflow_status: { exists: true }
        }
      }
    });

    const grouped = groupByWorkflowStatus(workflowItems);
    setWorkflows(grouped);
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px' }}>
        <h1>Workflow Management</h1>

        <KanbanBoard
          columns={[
            { id: 'draft', title: 'Draft', color: '#6c757d' },
            { id: 'review', title: 'In Review', color: '#ffc107' },
            { id: 'approved', title: 'Approved', color: '#28a745' },
            { id: 'published', title: 'Published', color: '#007bff' }
          ]}
          items={workflows}
          onItemMove={handleItemMove}
          onItemClick={handleItemClick}
        />
      </div>
    </Canvas>
  );
}
```

### 6. URL Parameter Handling

Handle navigation with query parameters:

```typescript
renderPage(pageId: string, ctx: RenderPageCtx) {
  if (pageId === 'search-results') {
    return {
      render: () => import('./pages/SearchResults'),
    };
  }
}

// SearchResults.tsx
export default function SearchResults({ ctx }: { ctx: RenderPageCtx }) {
  const searchParams = new URLSearchParams(ctx.location.search);
  const query = searchParams.get('q') || '';
  const type = searchParams.get('type') || 'all';
  const [results, setResults] = useState([]);

  useEffect(() => {
    if (query) {
      performSearch();
    }
  }, [query, type]);

  const performSearch = async () => {
    const searchResults = await ctx.searchItems({
      filter: {
        query: query,
        type: type !== 'all' ? type : undefined
      }
    });

    setResults(searchResults);
  };

  const updateSearch = (newQuery: string, newType: string) => {
    const params = new URLSearchParams();
    params.set('q', newQuery);
    params.set('type', newType);
    
    // Update URL without reload
    ctx.navigateTo(`${ctx.location.pathname}?${params.toString()}`);
  };

  return (
    <Canvas ctx={ctx}>
      <SearchInterface
        query={query}
        type={type}
        results={results}
        onSearch={updateSearch}
      />
    </Canvas>
  );
}
```

## Best Practices

1. **Use Canvas Component**: Always wrap content in the Canvas component for consistent styling
2. **Handle Loading States**: Show loading indicators during data fetching
3. **Error Handling**: Implement proper error handling with user feedback
4. **Responsive Design**: Ensure pages work well at different viewport sizes
5. **URL Parameters**: Use location.search for stateful navigation
6. **Performance**: Implement pagination for large data sets
7. **Consistent Navigation**: Use ctx.navigateTo() for internal navigation

## Related Hooks

- **mainNavigationTabs**: Add pages to main navigation
- **contentAreaSidebarItems**: Add pages to content sidebar
- **settingsAreaSidebarItemGroups**: Add pages to settings sidebar
- **renderModal**: Create modal dialogs from pages

## Important Notes

- Pages render in a full-screen iframe context
- The pageId must match the ID used in navigation hooks
- Pages have access to the full plugin context and API
- Use React and datocms-react-ui components for consistency
- Pages can navigate to other pages or DatoCMS routes
- State is not preserved between page navigations unless stored