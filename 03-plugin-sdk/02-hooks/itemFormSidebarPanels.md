# itemFormSidebarPanels Hook

## Purpose

The `itemFormSidebarPanels` hook allows plugins to add custom panels to the sidebar of the record editing interface. These panels appear on the right side of the form and can display supplementary information, provide quick actions, show related data, or offer additional tools while editing records.

## Hook Registration

```typescript
import type { 
  ItemFormSidebarPanelsHook,
  ItemFormSidebarPanelsCtx,
  ItemFormSidebarPanel,
  ItemFormSidebarPanelPlacement,
  ItemType,
  Field,
  Site,
  Plugin,
  Role,
  User,
  Account,
  Organization,
  SsoUser,
  ButtonOptions,
  ButtonChoices,
  ModalOptions
} from 'datocms-plugin-sdk';

const plugin: ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels(
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx
  ): ItemFormSidebarPanel[] {
    // Implementation
    return [];
  }
};
```

## Complete Type Definitions

```typescript
// Hook function type
type ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels: (
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx,
  ) => ItemFormSidebarPanel[];
};

// Context type with full type definitions
type ItemFormSidebarPanelsCtx = {
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken?: string;
  site: Site;
  environment: string;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  menuItems: MenuItem[];
  schema: Schema;
  users: Partial<Record<string, User>>;
  ui: {
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
  siteId: string;
};

// Return type
type ItemFormSidebarPanel = {
  id: string;                                     // Unique identifier for the panel
  label: string;                                  // Panel title displayed in sidebar
  parameters?: Record<string, unknown>;           // Parameters passed to panel render
  startOpen?: boolean;                            // Whether panel starts expanded (default: false)
  placement?: ItemFormSidebarPanelPlacement;      // Position relative to other panels
  rank?: number;                                  // Display order (lower numbers appear first)
  initialHeight?: number;                         // Initial iframe height in pixels
};

// Placement options
type ItemFormSidebarPanelPlacement = [
  'before' | 'after',
  'info' | 'publishedVersion' | 'schedule' | 'links' | 'history',
];

// ItemType with complete structure
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    singleton: boolean;
    sortable: boolean;
    api_key: string;
    tree: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    modular_block: boolean;
    inverse_relationships_enabled: boolean;
    workflow_enabled?: boolean;
    created_at: string;
    updated_at: string;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: 'field' }> };
    fieldsets: { data: Array<{ id: string; type: 'fieldset' }> };
    singleton_item?: { data: { id: string; type: 'item' } | null };
    ordering_field?: { data: { id: string; type: 'field' } | null };
    ordering_direction?: 'asc' | 'desc';
    title_field?: { data: { id: string; type: 'field' } | null };
    image_preview_field?: { data: { id: string; type: 'field' } | null };
    excerpt_field?: { data: { id: string; type: 'field' } | null };
  };
  meta: {
    created_at: string;
    updated_at: string;
  };
};

// Field with complete structure
type Field = {
  id: string;
  type: 'field';
  attributes: {
    api_key: string;
    field_type: string;
    label: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    hint?: string;
    default_value?: any;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
      fieldset_appearance?: {
        id: string;
      };
    };
    created_at: string;
    updated_at: string;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fieldset?: { data: { id: string; type: 'fieldset' } | null };
    inverse_of?: { data: { id: string; type: 'field' } | null };
  };
  meta: {
    created_at: string;
    updated_at: string;
  };
};

// Supporting types for context
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description?: string;
    url: string;
    parameters: Record<string, any>;
    field_types: string[];
    plugin_type: 'field_editor' | 'field_addon' | 'sidebar';
    tags: string[];
    permissions: {
      can_create_new_locale: boolean;
      can_edit_favicon: boolean;
      can_edit_site: boolean;
      can_edit_schema: boolean;
      can_manage_menu: boolean;
      can_manage_users: boolean;
      can_manage_access_tokens: boolean;
      can_dump_data: boolean;
      can_import_and_export: boolean;
      can_manage_webhooks: boolean;
      can_manage_build_triggers: boolean;
      can_manage_environments: boolean;
    };
  };
};

type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    locales: string[];
    theme: Record<string, any>;
    domain?: string;
    internal_domain: string;
    global_seo: Record<string, any>;
    favicon?: any;
    no_index: boolean;
    frontend_url?: string;
    ssg?: string;
    created_at: string;
    updated_at: string;
  };
};

type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_favicon: boolean;
    can_edit_site: boolean;
    can_manage_menu: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_dump_data: boolean;
    can_import_and_export: boolean;
    can_manage_webhooks: boolean;
    can_manage_build_triggers: boolean;
    can_manage_environments: boolean;
    positive_item_type_permissions: string[];
    negative_item_type_permissions: string[];
    item_type_permissions: Record<string, any>;
  };
};

type User = {
  id: string;
  type: 'user';
  attributes: {
    first_name: string;
    last_name: string;
    email: string;
    is_sso: boolean;
    state: 'active' | 'pending' | 'inactive';
  };
  relationships: {
    role: { data: { id: string; type: 'role' } };
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

### 1. SEO Analysis Panel

Show real-time SEO analysis and suggestions:

```typescript
import type { 
  ItemFormSidebarPanelsHook, 
  ItemFormSidebarPanelsCtx, 
  ItemFormSidebarPanel, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels(
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx
  ): ItemFormSidebarPanel[] {
    // Only show for content types with SEO fields
    const seoModels: string[] = ['article', 'page', 'product', 'landing_page'];
    
    if (!seoModels.includes(itemType.attributes.api_key)) {
      return [];
    }

    const panels: ItemFormSidebarPanel[] = [
      {
        id: 'seo-analysis',
        label: 'SEO Analysis',
        startOpen: true,
        placement: ['after', 'info'],
        rank: 10
      }
    ];

    return panels;
  }
};

// In renderItemFormSidebarPanel hook:
if (sidebarPanelId === 'seo-analysis') {
  return {
    render: () => import('./panels/SeoAnalysisPanel'),
  };
}

// SeoAnalysisPanel.tsx
export default function SeoAnalysisPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [analysis, setAnalysis] = useState(null);
  
  useEffect(() => {
    analyzeSeo();
  }, [ctx.formValues.title, ctx.formValues.content, ctx.formValues.seo_title]);

  const analyzeSeo = () => {
    const title = ctx.formValues.title || '';
    const content = ctx.formValues.content || '';
    const seoTitle = ctx.formValues.seo_title || '';
    const metaDesc = ctx.formValues.meta_description || '';
    
    const issues = [];
    const suggestions = [];
    
    // Title analysis
    if (!seoTitle) {
      issues.push('Missing SEO title');
    } else if (seoTitle.length < 30) {
      issues.push('SEO title too short (min 30 chars)');
    } else if (seoTitle.length > 60) {
      issues.push('SEO title too long (max 60 chars)');
    }
    
    // Content analysis
    const wordCount = content.split(/\s+/).filter(Boolean).length;
    if (wordCount < 300) {
      suggestions.push(`Add ${300 - wordCount} more words for better SEO`);
    }
    
    // Keyword density
    if (ctx.formValues.focus_keywords) {
      const keywords = ctx.formValues.focus_keywords.split(',');
      // Analyze keyword usage...
    }
    
    setAnalysis({ issues, suggestions, score: calculateScore(issues) });
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {analysis && (
          <>
            <ScoreIndicator score={analysis.score} />
            
            {analysis.issues.length > 0 && (
              <div style={{ marginTop: '16px' }}>
                <h4>Issues</h4>
                <ul>
                  {analysis.issues.map((issue, i) => (
                    <li key={i} style={{ color: '#dc3545' }}>{issue}</li>
                  ))}
                </ul>
              </div>
            )}
            
            {analysis.suggestions.length > 0 && (
              <div style={{ marginTop: '16px' }}>
                <h4>Suggestions</h4>
                <ul>
                  {analysis.suggestions.map((suggestion, i) => (
                    <li key={i} style={{ color: '#ffc107' }}>{suggestion}</li>
                  ))}
                </ul>
              </div>
            )}
          </>
        )}
      </div>
    </Canvas>
  );
}
```

### 2. Related Content Panel

Display and manage related records:

```typescript
import type { 
  ItemFormSidebarPanelsHook, 
  ItemFormSidebarPanelsCtx, 
  ItemFormSidebarPanel, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels(
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx
  ): ItemFormSidebarPanel[] {
    const panels: ItemFormSidebarPanel[] = [];

    // Show for articles to display related content
    if (itemType.attributes.api_key === 'article') {
      panels.push({
        id: 'related-articles',
        label: 'Related Articles',
        startOpen: false,
        placement: ['after', 'info'],
        rank: 20
      });
    }

    // Show for products to display related products
    if (itemType.attributes.api_key === 'product') {
      panels.push(
        {
          id: 'related-products',
          label: 'Related Products',
          startOpen: true,
          placement: ['after', 'info'],
          rank: 15
        },
        {
          id: 'cross-sells',
          label: 'Cross-sell Products',
          startOpen: false,
          placement: ['after', 'info'],
          rank: 16
        }
      );
    }

    return panels;
  }
};

// RelatedProductsPanel.tsx
export default function RelatedProductsPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [relatedProducts, setRelatedProducts] = useState([]);
  const [suggestions, setSuggestions] = useState([]);

  useEffect(() => {
    loadRelatedProducts();
    loadSuggestions();
  }, [ctx.item?.id]);

  const loadRelatedProducts = async () => {
    if (!ctx.item) return;
    
    const related = await ctx.searchItems({
      filter: {
        type: 'product',
        fields: {
          related_to: { any_in: [ctx.item.id] }
        }
      }
    });
    
    setRelatedProducts(related);
  };

  const loadSuggestions = async () => {
    // AI-based suggestions
    const category = ctx.formValues.category;
    const tags = ctx.formValues.tags || [];
    
    const suggestions = await ctx.searchItems({
      filter: {
        type: 'product',
        fields: {
          category: { eq: category },
          tags: { any_in: tags }
        }
      },
      page: { limit: 5 }
    });
    
    setSuggestions(suggestions.filter(p => p.id !== ctx.item?.id));
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        <div>
          <h4>Current Related Products ({relatedProducts.length})</h4>
          {relatedProducts.map(product => (
            <RelatedProductItem
              key={product.id}
              product={product}
              onRemove={() => removeRelated(product.id)}
              onView={() => ctx.navigateTo(`/editor/item_types/product/items/${product.id}/edit`)}
            />
          ))}
        </div>

        <div style={{ marginTop: '20px' }}>
          <h4>Suggestions</h4>
          {suggestions.map(product => (
            <SuggestionItem
              key={product.id}
              product={product}
              onAdd={() => addRelated(product.id)}
            />
          ))}
        </div>

        <Button
          onClick={() => ctx.openModal({
            id: 'product-search',
            title: 'Add Related Products',
            width: 'l'
          })}
          buttonType="primary"
          buttonSize="s"
          fullWidth
        >
          Browse All Products
        </Button>
      </div>
    </Canvas>
  );
}
```

### 3. Workflow Status Panel

Show workflow information and actions:

```typescript
import type { 
  ItemFormSidebarPanelsHook, 
  ItemFormSidebarPanelsCtx, 
  ItemFormSidebarPanel, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels(
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx
  ): ItemFormSidebarPanel[] {
    // Check if workflow is enabled for this model
    const workflowEnabled: boolean = itemType.attributes.workflow_enabled ?? false;
    
    if (!workflowEnabled) {
      return [];
    }

    const panels: ItemFormSidebarPanel[] = [
      {
        id: 'workflow-status',
        label: 'Workflow',
        startOpen: true,
        placement: ['before', 'info'],
        rank: 1
      }
    ];

    return panels;
  }
};

// WorkflowStatusPanel.tsx
export default function WorkflowStatusPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [workflow, setWorkflow] = useState(null);
  const [history, setHistory] = useState([]);

  useEffect(() => {
    loadWorkflowData();
  }, [ctx.item?.id]);

  const loadWorkflowData = async () => {
    if (!ctx.item) return;
    
    // Load workflow state and history
    const workflowData = await fetchWorkflowStatus(ctx.item.id);
    setWorkflow(workflowData);
    setHistory(workflowData.history);
  };

  const handleTransition = async (action: string) => {
    try {
      await performWorkflowTransition(ctx.item.id, action);
      await loadWorkflowData();
      ctx.notice(`Workflow updated: ${action}`);
    } catch (error) {
      ctx.alert(`Workflow error: ${error.message}`);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {workflow && (
          <>
            <WorkflowDiagram
              currentState={workflow.state}
              availableTransitions={workflow.transitions}
            />

            <div style={{ marginTop: '16px' }}>
              <strong>Current State:</strong> {workflow.state}
            </div>

            <div style={{ marginTop: '8px' }}>
              <strong>Assigned to:</strong> {workflow.assignee || 'Unassigned'}
            </div>

            <div style={{ marginTop: '16px' }}>
              <h4>Available Actions</h4>
              {workflow.transitions.map(transition => (
                <Button
                  key={transition.action}
                  onClick={() => handleTransition(transition.action)}
                  buttonType={transition.type === 'approve' ? 'primary' : 'muted'}
                  buttonSize="s"
                  fullWidth
                  style={{ marginBottom: '8px' }}
                >
                  {transition.label}
                </Button>
              ))}
            </div>

            <div style={{ marginTop: '20px' }}>
              <h4>History</h4>
              <Timeline items={history} />
            </div>
          </>
        )}
      </div>
    </Canvas>
  );
}
```

### 4. AI Content Assistant Panel

Provide AI-powered content suggestions:

```typescript
import type { 
  ItemFormSidebarPanelsHook, 
  ItemFormSidebarPanelsCtx, 
  ItemFormSidebarPanel, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels(
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx
  ): ItemFormSidebarPanel[] {
    // Only show for content types that can benefit from AI
    const aiEnabledTypes: string[] = ['article', 'blog_post', 'product_description', 'page'];
    
    if (!aiEnabledTypes.includes(itemType.attributes.api_key)) {
      return [];
    }

    const parameters: Record<string, any> = ctx.plugin.attributes.parameters;
    if (!parameters.aiApiKey) {
      return [];
    }

    const panels: ItemFormSidebarPanel[] = [
      {
        id: 'ai-assistant',
        label: 'AI Assistant',
        startOpen: false,
        placement: ['after', 'info'],
        rank: 10
      }
    ];

    return panels;
  }
};

// AiAssistantPanel.tsx
export default function AiAssistantPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [suggestions, setSuggestions] = useState({});
  const [generating, setGenerating] = useState(false);

  const generateSuggestions = async (field: string) => {
    setGenerating(true);
    try {
      const prompt = buildPrompt(field, ctx.formValues);
      const suggestion = await callAiApi(prompt);
      
      setSuggestions({
        ...suggestions,
        [field]: suggestion
      });
    } catch (error) {
      ctx.alert('Failed to generate suggestion');
    } finally {
      setGenerating(false);
    }
  };

  const applySuggestion = (field: string) => {
    ctx.setFieldValue(field, suggestions[field]);
    ctx.notice('Suggestion applied');
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        <h4>AI Content Assistant</h4>
        
        <div style={{ marginTop: '16px' }}>
          <label>Generate Title</label>
          <Button
            onClick={() => generateSuggestions('title')}
            disabled={generating}
            buttonSize="s"
            fullWidth
          >
            {generating ? 'Generating...' : 'Generate Title'}
          </Button>
          {suggestions.title && (
            <SuggestionBox
              suggestion={suggestions.title}
              onApply={() => applySuggestion('title')}
            />
          )}
        </div>

        <div style={{ marginTop: '16px' }}>
          <label>Generate Meta Description</label>
          <Button
            onClick={() => generateSuggestions('meta_description')}
            disabled={generating}
            buttonSize="s"
            fullWidth
          >
            Generate Meta Description
          </Button>
          {suggestions.meta_description && (
            <SuggestionBox
              suggestion={suggestions.meta_description}
              onApply={() => applySuggestion('meta_description')}
            />
          )}
        </div>

        <div style={{ marginTop: '16px' }}>
          <label>Content Ideas</label>
          <Button
            onClick={() => generateContentIdeas()}
            disabled={generating}
            buttonSize="s"
            fullWidth
          >
            Get Content Ideas
          </Button>
        </div>
      </div>
    </Canvas>
  );
}
```

### 5. Preview Panel

Show live preview of content:

```typescript
import type { 
  ItemFormSidebarPanelsHook, 
  ItemFormSidebarPanelsCtx, 
  ItemFormSidebarPanel, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels(
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx
  ): ItemFormSidebarPanel[] {
    // Show preview for previewable content types
    const previewableTypes: string[] = ['article', 'page', 'email_template'];
    
    if (!previewableTypes.includes(itemType.attributes.api_key)) {
      return [];
    }

    const panels: ItemFormSidebarPanel[] = [
      {
        id: 'content-preview',
        label: 'Preview',
        startOpen: true,
        placement: ['after', 'info'],
        rank: 5
      }
    ];

    return panels;
  }
};

// ContentPreviewPanel.tsx
export default function ContentPreviewPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [previewMode, setPreviewMode] = useState('desktop');
  const [previewUrl, setPreviewUrl] = useState('');

  useEffect(() => {
    generatePreviewUrl();
  }, [ctx.formValues, ctx.locale]);

  const generatePreviewUrl = async () => {
    if (!ctx.item?.id) return;
    
    const previewToken = await generatePreviewToken(ctx.item.id);
    const baseUrl = ctx.plugin.attributes.parameters.previewBaseUrl;
    
    setPreviewUrl(`${baseUrl}/preview/${ctx.item.id}?token=${previewToken}&locale=${ctx.locale}`);
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        <div style={{ marginBottom: '12px' }}>
          <ButtonGroup>
            <Button
              onClick={() => setPreviewMode('desktop')}
              buttonSize="s"
              buttonType={previewMode === 'desktop' ? 'primary' : 'muted'}
            >
              Desktop
            </Button>
            <Button
              onClick={() => setPreviewMode('tablet')}
              buttonSize="s"
              buttonType={previewMode === 'tablet' ? 'primary' : 'muted'}
            >
              Tablet
            </Button>
            <Button
              onClick={() => setPreviewMode('mobile')}
              buttonSize="s"
              buttonType={previewMode === 'mobile' ? 'primary' : 'muted'}
            >
              Mobile
            </Button>
          </ButtonGroup>
        </div>

        <PreviewFrame
          url={previewUrl}
          mode={previewMode}
          onRefresh={generatePreviewUrl}
        />

        <Button
          onClick={() => window.open(previewUrl, '_blank')}
          buttonType="muted"
          buttonSize="s"
          fullWidth
          style={{ marginTop: '12px' }}
        >
          Open in New Tab
        </Button>
      </div>
    </Canvas>
  );
}
```

### 6. Version History Panel

Show record version history:

```typescript
import type { 
  ItemFormSidebarPanelsHook, 
  ItemFormSidebarPanelsCtx, 
  ItemFormSidebarPanel, 
  ItemType 
} from 'datocms-plugin-sdk';

const plugin: ItemFormSidebarPanelsHook = {
  itemFormSidebarPanels(
    itemType: ItemType,
    ctx: ItemFormSidebarPanelsCtx
  ): ItemFormSidebarPanel[] {
    // Show for all content types
    const panels: ItemFormSidebarPanel[] = [
      {
        id: 'version-history',
        label: 'Version History',
        startOpen: false,
        placement: ['after', 'info'],
        rank: 100
      }
    ];

    return panels;
  }
};

// VersionHistoryPanel.tsx
export default function VersionHistoryPanel({ ctx }: { ctx: RenderItemFormSidebarPanelCtx }) {
  const [versions, setVersions] = useState([]);
  const [comparing, setComparing] = useState(false);

  useEffect(() => {
    if (ctx.item?.id) {
      loadVersionHistory();
    }
  }, [ctx.item?.id]);

  const loadVersionHistory = async () => {
    const history = await ctx.loadItemVersions(ctx.item.id);
    setVersions(history);
  };

  const compareVersions = async (versionId: string) => {
    setComparing(true);
    const comparison = await ctx.compareVersions(ctx.item.id, versionId);
    
    await ctx.openModal({
      id: 'version-comparison',
      title: 'Version Comparison',
      width: 'xl',
      parameters: { comparison }
    });
    
    setComparing(false);
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '16px' }}>
        {versions.length === 0 ? (
          <p>No version history available</p>
        ) : (
          <div>
            <p style={{ marginBottom: '12px' }}>
              {versions.length} versions available
            </p>
            
            {versions.slice(0, 10).map(version => (
              <VersionItem
                key={version.id}
                version={version}
                onCompare={() => compareVersions(version.id)}
                onRestore={() => restoreVersion(version.id)}
                current={version.id === ctx.item.meta.current_version}
              />
            ))}
            
            {versions.length > 10 && (
              <Button
                onClick={() => ctx.openModal({
                  id: 'all-versions',
                  title: 'All Versions',
                  width: 'l',
                  parameters: { versions }
                })}
                buttonType="muted"
                buttonSize="s"
                fullWidth
              >
                View All {versions.length} Versions
              </Button>
            )}
          </div>
        )}
      </div>
    </Canvas>
  );
}
```

## Placement Options

Control where panels appear in the sidebar:

```typescript
// Before the info panel (very top)
placement: ['before', 'info']

// After the info panel
placement: ['after', 'info']

// Custom ordering with rank
placement: ['after', 'info'],
rank: 10 // Lower rank appears first
```

## Best Practices

1. **Conditional Display**: Only show panels relevant to the content type
2. **Start State**: Use `startOpen: true` for frequently-used panels
3. **Performance**: Load data asynchronously and show loading states
4. **Responsive**: Design panels to work in narrow sidebar width
5. **Clear Labels**: Use concise, descriptive panel titles
6. **Rank Configuration**: Make rank configurable to avoid conflicts

## Related Hooks

- **renderItemFormSidebarPanel**: Implements the panel content
- **itemFormSidebars**: Add complete custom sidebars
- **itemFormOutlets**: Add content to other areas of the form

## Important Notes

- Panels appear in the right sidebar of the record editing form
- The panel ID must be unique and match the ID in `renderItemFormSidebarPanel`
- Panels can be collapsed/expanded by users
- Multiple panels can be open simultaneously
- Panel content has access to form state and can modify field values
- Consider performance impact of real-time updates in panels

## Validation Functions

```typescript
// Type guard for ItemFormSidebarPanel
export function isItemFormSidebarPanel(value: unknown): value is ItemFormSidebarPanel {
  return (
    typeof value === 'object' &&
    value !== null &&
    typeof (value as any).id === 'string' &&
    typeof (value as any).label === 'string' &&
    ((value as any).parameters === undefined || typeof (value as any).parameters === 'object') &&
    ((value as any).startOpen === undefined || typeof (value as any).startOpen === 'boolean') &&
    ((value as any).placement === undefined || 
     (Array.isArray((value as any).placement) && 
      (value as any).placement.length === 2 &&
      typeof (value as any).placement[0] === 'string' &&
      typeof (value as any).placement[1] === 'string')) &&
    ((value as any).rank === undefined || typeof (value as any).rank === 'number') &&
    ((value as any).initialHeight === undefined || typeof (value as any).initialHeight === 'number')
  );
}

// Type guard for hook return value
export function isReturnTypeOfItemFormSidebarPanelsHook(
  value: unknown,
): value is ItemFormSidebarPanel[] {
  return Array.isArray(value) && value.every(isItemFormSidebarPanel);
}
```