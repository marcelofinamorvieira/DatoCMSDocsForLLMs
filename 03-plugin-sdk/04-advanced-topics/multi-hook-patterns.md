# Multi-Hook Patterns

## Overview

DatoCMS plugins can implement multiple hooks to create sophisticated functionality that spans different areas of the CMS. This guide covers patterns and best practices for building plugins that leverage multiple hooks effectively.

## Common Multi-Hook Patterns

### 1. Field Extension with Configuration Screen

A common pattern is combining a field extension with a configuration screen for setup.

```typescript
import { connect, RenderConfigScreenCtx, RenderFieldExtensionCtx } from 'datocms-plugin-sdk';
import React, { useState } from 'react';

// Shared configuration interface
interface PluginConfig {
  apiEndpoint: string;
  apiKey: string;
  defaultLanguage: string;
  enableCache: boolean;
}

// Configuration screen component
function ConfigScreen({ ctx }: { ctx: RenderConfigScreenCtx }) {
  const [config, setConfig] = useState<PluginConfig>(
    ctx.plugin.attributes.parameters as PluginConfig || {
      apiEndpoint: '',
      apiKey: '',
      defaultLanguage: 'en',
      enableCache: true
    }
  );

  const handleSave = async () => {
    await ctx.updatePluginParameters(config);
    ctx.notice('Configuration saved successfully!');
  };

  return (
    <div>
      <h2>Plugin Configuration</h2>
      <input
        placeholder="API Endpoint"
        value={config.apiEndpoint}
        onChange={(e) => setConfig({ ...config, apiEndpoint: e.target.value })}
      />
      <input
        placeholder="API Key"
        type="password"
        value={config.apiKey}
        onChange={(e) => setConfig({ ...config, apiKey: e.target.value })}
      />
      <button onClick={handleSave}>Save Configuration</button>
    </div>
  );
}

// Field extension component
function FieldExtension({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const config = ctx.plugin.attributes.parameters as PluginConfig;
  
  if (!config?.apiEndpoint || !config?.apiKey) {
    return (
      <div>
        <p>Plugin not configured. Please configure it in the plugin settings.</p>
        <button onClick={() => ctx.navigateTo(`/admin/plugins/${ctx.plugin.id}/edit`)}>
          Configure Plugin
        </button>
      </div>
    );
  }

  // Main field extension logic
  return <div>Field extension UI using {config.apiEndpoint}</div>;
}

// Connect multiple hooks
connect({
  renderConfigScreen(ctx) {
    render(<ConfigScreen ctx={ctx} />);
  },
  
  manualFieldExtensions() {
    return [
      {
        id: 'myExtension',
        name: 'My Field Extension',
        type: 'editor',
        fieldTypes: ['text', 'string']
      }
    ];
  },
  
  renderFieldExtension(fieldExtensionId, ctx) {
    render(<FieldExtension ctx={ctx} />);
  }
});
```

### 2. Global Navigation with Custom Pages

Create a complete feature with navigation and multiple pages.

```typescript
import { connect } from 'datocms-plugin-sdk';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// Main navigation component
function Navigation() {
  return (
    <nav>
      <Link to="/">Dashboard</Link>
      <Link to="/reports">Reports</Link>
      <Link to="/settings">Settings</Link>
    </nav>
  );
}

// Page components
function Dashboard({ ctx }) {
  const [stats, setStats] = useState(null);
  
  useEffect(() => {
    loadDashboardStats(ctx).then(setStats);
  }, []);
  
  return (
    <div>
      <h1>Analytics Dashboard</h1>
      {stats && <StatsDisplay stats={stats} />}
    </div>
  );
}

function Reports({ ctx }) {
  return (
    <div>
      <h1>Reports</h1>
      <ReportGenerator ctx={ctx} />
    </div>
  );
}

// Connect with navigation and page rendering
connect({
  mainNavigationTabs() {
    return [
      {
        label: 'Analytics',
        icon: 'chart-line',
        pointsTo: {
          pageId: 'analytics'
        }
      }
    ];
  },
  
  renderPage(pageId, ctx) {
    if (pageId === 'analytics') {
      render(
        <BrowserRouter>
          <Navigation />
          <Routes>
            <Route path="/" element={<Dashboard ctx={ctx} />} />
            <Route path="/reports" element={<Reports ctx={ctx} />} />
            <Route path="/settings" element={<Settings ctx={ctx} />} />
          </Routes>
        </BrowserRouter>
      );
    }
  },
  
  contentAreaSidebarItems(ctx) {
    // Add quick stats to content area
    return [
      {
        label: 'Quick Stats',
        icon: 'chart-pie',
        placement: ['after', 'menuBarSearch'],
        action: async () => {
          const stats = await loadQuickStats(ctx);
          ctx.openModal({
            id: 'quickStats',
            title: 'Quick Statistics',
            width: 's',
            parameters: { stats }
          });
        }
      }
    ];
  },
  
  renderModal(modalId, ctx) {
    if (modalId === 'quickStats') {
      render(<QuickStatsModal stats={ctx.parameters.stats} />);
    }
  }
});
```

### 3. Item Form Enhancement Suite

Enhance the item editing experience with multiple integration points.

```typescript
import { connect } from 'datocms-plugin-sdk';

// Shared state manager for form enhancements
class FormEnhancementManager {
  private subscribers = new Map<string, Set<Function>>();
  private state = {
    wordCount: 0,
    readingTime: 0,
    seoScore: 0,
    validationErrors: []
  };
  
  subscribe(key: string, callback: Function) {
    if (!this.subscribers.has(key)) {
      this.subscribers.set(key, new Set());
    }
    this.subscribers.get(key)!.add(callback);
    
    return () => {
      this.subscribers.get(key)?.delete(callback);
    };
  }
  
  updateState(updates: Partial<typeof this.state>) {
    this.state = { ...this.state, ...updates };
    this.notify();
  }
  
  private notify() {
    this.subscribers.forEach(callbacks => {
      callbacks.forEach(cb => cb(this.state));
    });
  }
}

const manager = new FormEnhancementManager();

connect({
  // Add sidebar panel for content analysis
  itemFormSidebarPanels() {
    return [
      {
        id: 'contentAnalysis',
        label: 'Content Analysis',
        icon: 'chart-bar'
      }
    ];
  },
  
  renderItemFormSidebarPanel(sidebarPanelId, ctx) {
    if (sidebarPanelId === 'contentAnalysis') {
      function ContentAnalysisPanel() {
        const [state, setState] = useState(manager.state);
        
        useEffect(() => {
          return manager.subscribe('panel', setState);
        }, []);
        
        return (
          <div>
            <h3>Content Metrics</h3>
            <p>Word Count: {state.wordCount}</p>
            <p>Reading Time: {state.readingTime} min</p>
            <p>SEO Score: {state.seoScore}/100</p>
          </div>
        );
      }
      
      render(<ContentAnalysisPanel />);
    }
  },
  
  // Add toolbar actions
  itemFormDropdownActions(ctx) {
    return [
      {
        label: 'Analyze Content',
        icon: 'search',
        action: async () => {
          const analysis = await analyzeContent(ctx.formValues);
          manager.updateState(analysis);
          ctx.notice('Content analyzed successfully');
        }
      },
      {
        label: 'Export as PDF',
        icon: 'download',
        action: async () => {
          await exportItemAsPdf(ctx.item);
          ctx.notice('PDF exported successfully');
        }
      }
    ];
  },
  
  // Add validation before save
  async onBeforeItemUpsert(item, ctx) {
    const errors = [];
    
    // Check content requirements
    if (item.attributes.title?.length < 10) {
      errors.push('Title must be at least 10 characters');
    }
    
    if (item.attributes.content?.length < 100) {
      errors.push('Content must be at least 100 characters');
    }
    
    if (errors.length > 0) {
      ctx.alert(errors.join('\n'));
      return false; // Prevent save
    }
    
    // Add metadata
    return {
      ...item,
      attributes: {
        ...item.attributes,
        last_analyzed: new Date().toISOString(),
        word_count: countWords(item.attributes.content)
      }
    };
  },
  
  // Add custom outlets
  itemFormOutlets() {
    return [
      {
        id: 'seoPreview',
        initialHeight: 120
      }
    ];
  },
  
  renderItemFormOutlet(outletId, ctx) {
    if (outletId === 'seoPreview') {
      function SeoPreview() {
        const title = ctx.formValues.title || 'Untitled';
        const description = ctx.formValues.meta_description || 
                          ctx.formValues.content?.substring(0, 160) || '';
        
        return (
          <div style={{ padding: '16px', background: '#f5f5f5' }}>
            <h4 style={{ color: '#1a0dab', margin: 0 }}>{title}</h4>
            <p style={{ color: '#006621', margin: '4px 0' }}>
              {window.location.origin}/posts/{ctx.formValues.slug || 'untitled'}
            </p>
            <p style={{ color: '#545454', margin: 0 }}>{description}...</p>
          </div>
        );
      }
      
      render(<SeoPreview />);
    }
  }
});
```

### 4. Upload Management System

Comprehensive upload handling with multiple touchpoints.

```typescript
import { connect } from 'datocms-plugin-sdk';

// Shared upload processor
class UploadProcessor {
  async processImage(file: File): Promise<ProcessedImage> {
    // Resize, optimize, generate thumbnails
    const optimized = await optimizeImage(file);
    const thumbnail = await generateThumbnail(file);
    const metadata = await extractMetadata(file);
    
    return { optimized, thumbnail, metadata };
  }
}

const processor = new UploadProcessor();

connect({
  // Custom asset source
  assetSources() {
    return [
      {
        id: 'stockPhotos',
        name: 'Stock Photos',
        icon: 'image',
        modal: { width: 'l', height: 'l' }
      }
    ];
  },
  
  renderAssetSource(sourceId, ctx) {
    if (sourceId === 'stockPhotos') {
      function StockPhotoSearch() {
        const [query, setQuery] = useState('');
        const [results, setResults] = useState([]);
        
        const handleSearch = async () => {
          const photos = await searchStockPhotos(query);
          setResults(photos);
        };
        
        const handleSelect = async (photo) => {
          const processed = await processor.processImage(photo.url);
          ctx.select({
            resource: processed.optimized,
            thumbnail: processed.thumbnail,
            metadata: processed.metadata
          });
        };
        
        return (
          <div>
            <input
              value={query}
              onChange={(e) => setQuery(e.target.value)}
              placeholder="Search stock photos..."
            />
            <button onClick={handleSearch}>Search</button>
            <div>
              {results.map(photo => (
                <img
                  key={photo.id}
                  src={photo.thumbnail}
                  onClick={() => handleSelect(photo)}
                  style={{ cursor: 'pointer' }}
                />
              ))}
            </div>
          </div>
        );
      }
      
      render(<StockPhotoSearch />);
    }
  },
  
  // Upload sidebar for metadata
  uploadSidebars() {
    return [
      {
        id: 'imageMetadata',
        label: 'Image Metadata'
      }
    ];
  },
  
  renderUploadSidebar(sidebarId, ctx) {
    if (sidebarId === 'imageMetadata') {
      function ImageMetadataEditor() {
        const [metadata, setMetadata] = useState({
          alt: ctx.upload.attributes.alt || '',
          title: ctx.upload.attributes.title || '',
          credits: ctx.upload.attributes.custom_data?.credits || ''
        });
        
        const handleSave = async () => {
          await ctx.updateUpload({
            alt: metadata.alt,
            title: metadata.title,
            custom_data: {
              ...ctx.upload.attributes.custom_data,
              credits: metadata.credits
            }
          });
          ctx.notice('Metadata updated');
        };
        
        return (
          <div>
            <input
              placeholder="Alt text"
              value={metadata.alt}
              onChange={(e) => setMetadata({ ...metadata, alt: e.target.value })}
            />
            <input
              placeholder="Title"
              value={metadata.title}
              onChange={(e) => setMetadata({ ...metadata, title: e.target.value })}
            />
            <input
              placeholder="Credits"
              value={metadata.credits}
              onChange={(e) => setMetadata({ ...metadata, credits: e.target.value })}
            />
            <button onClick={handleSave}>Save Metadata</button>
          </div>
        );
      }
      
      render(<ImageMetadataEditor />);
    }
  },
  
  // Bulk operations on uploads
  uploadsDropdownActions(ctx) {
    return [
      {
        label: 'Optimize Selected',
        icon: 'compress',
        action: async (uploads) => {
          ctx.notice('Optimizing images...');
          
          for (const upload of uploads) {
            if (upload.attributes.mime_type?.startsWith('image/')) {
              const optimized = await processor.processImage(upload.attributes.url);
              await ctx.updateUpload(upload.id, {
                custom_data: {
                  ...upload.attributes.custom_data,
                  optimized_url: optimized.url
                }
              });
            }
          }
          
          ctx.notice(`Optimized ${uploads.length} images`);
        }
      }
    ];
  }
});
```

### 5. Complete Workflow Integration

Integrate with DatoCMS workflows using multiple hooks.

```typescript
import { connect } from 'datocms-plugin-sdk';

// Workflow state manager
class WorkflowManager {
  async checkRequirements(item: any, ctx: any): Promise<ValidationResult> {
    const results = {
      seo: await this.checkSeo(item),
      content: await this.checkContent(item),
      media: await this.checkMedia(item)
    };
    
    return {
      passed: Object.values(results).every(r => r.passed),
      results
    };
  }
}

const workflow = new WorkflowManager();

connect({
  // Boot hook for initialization
  onBoot(ctx) {
    console.log('Workflow plugin initialized');
    
    // Set up any global state or connections
    if (ctx.plugin.attributes.parameters.webhookUrl) {
      setupWebhookConnection(ctx.plugin.attributes.parameters.webhookUrl);
    }
  },
  
  // Intercept publish action
  async onBeforeItemsPublish(items, ctx) {
    const failedItems = [];
    
    for (const item of items) {
      const validation = await workflow.checkRequirements(item, ctx);
      
      if (!validation.passed) {
        failedItems.push({
          item,
          errors: validation.results
        });
      }
    }
    
    if (failedItems.length > 0) {
      const errors = failedItems.map(f => 
        `${f.item.attributes.title}: ${Object.entries(f.errors)
          .filter(([_, v]) => !v.passed)
          .map(([k, v]) => `${k} - ${v.message}`)
          .join(', ')}`
      ).join('\n');
      
      ctx.alert(`Cannot publish items with errors:\n${errors}`);
      return false;
    }
    
    // Log publish event
    await logWorkflowEvent('publish', items, ctx);
    
    return items;
  },
  
  // Add workflow status indicator
  buildItemPresentationInfo(item, ctx) {
    const status = item.meta.status;
    const hasErrors = item.attributes.custom_data?.workflow_errors?.length > 0;
    
    return {
      title: item.attributes.title || 'Untitled',
      imageUrl: item.attributes.featured_image?.url,
      badge: hasErrors ? {
        text: 'Has Issues',
        type: 'negative'
      } : status === 'published' ? {
        text: 'Published',
        type: 'positive'
      } : {
        text: 'Draft',
        type: 'neutral'
      }
    };
  },
  
  // Settings panel for workflow configuration
  settingsAreaSidebarItemGroups() {
    return [
      {
        label: 'Workflow',
        items: [
          {
            label: 'Workflow Settings',
            icon: 'cog',
            pointsTo: {
              pageId: 'workflowSettings'
            }
          },
          {
            label: 'Validation Rules',
            icon: 'check-circle',
            pointsTo: {
              pageId: 'validationRules'
            }
          }
        ]
      }
    ];
  },
  
  renderPage(pageId, ctx) {
    switch (pageId) {
      case 'workflowSettings':
        render(<WorkflowSettings ctx={ctx} />);
        break;
      case 'validationRules':
        render(<ValidationRules ctx={ctx} />);
        break;
    }
  },
  
  // Bulk operations
  itemsDropdownActions(ctx) {
    return [
      {
        label: 'Validate Selected',
        icon: 'check',
        action: async (items) => {
          const results = await Promise.all(
            items.map(item => workflow.checkRequirements(item, ctx))
          );
          
          const summary = results.reduce((acc, result, index) => {
            if (!result.passed) {
              acc.push(`${items[index].attributes.title}: Failed`);
            }
            return acc;
          }, []);
          
          if (summary.length > 0) {
            ctx.alert(`Validation failed for:\n${summary.join('\n')}`);
          } else {
            ctx.notice('All items passed validation!');
          }
        }
      }
    ];
  }
});
```

## State Management Across Hooks

### 1. Shared State Pattern

```typescript
// Shared state manager
class PluginStateManager {
  private state = new Map<string, any>();
  private listeners = new Map<string, Set<Function>>();
  
  get<T>(key: string, defaultValue?: T): T {
    return this.state.get(key) ?? defaultValue;
  }
  
  set(key: string, value: any) {
    this.state.set(key, value);
    this.notify(key, value);
  }
  
  subscribe(key: string, callback: Function) {
    if (!this.listeners.has(key)) {
      this.listeners.set(key, new Set());
    }
    this.listeners.get(key)!.add(callback);
    
    return () => {
      this.listeners.get(key)?.delete(callback);
    };
  }
  
  private notify(key: string, value: any) {
    this.listeners.get(key)?.forEach(cb => cb(value));
  }
}

const pluginState = new PluginStateManager();

// Use in multiple hooks
connect({
  renderConfigScreen(ctx) {
    function ConfigScreen() {
      const [apiKey, setApiKey] = useState(pluginState.get('apiKey', ''));
      
      useEffect(() => {
        return pluginState.subscribe('apiKey', setApiKey);
      }, []);
      
      const handleSave = () => {
        pluginState.set('apiKey', apiKey);
        ctx.updatePluginParameters({ apiKey });
      };
      
      return (
        <div>
          <input value={apiKey} onChange={e => setApiKey(e.target.value)} />
          <button onClick={handleSave}>Save</button>
        </div>
      );
    }
    
    render(<ConfigScreen />);
  },
  
  renderFieldExtension(fieldExtensionId, ctx) {
    function FieldExtension() {
      const [apiKey, setApiKey] = useState(pluginState.get('apiKey', ''));
      
      useEffect(() => {
        return pluginState.subscribe('apiKey', setApiKey);
      }, []);
      
      if (!apiKey) {
        return <div>Please configure API key</div>;
      }
      
      return <div>Using API key: {apiKey.substring(0, 8)}...</div>;
    }
    
    render(<FieldExtension />);
  }
});
```

### 2. Event Bus Pattern

```typescript
class EventBus {
  private events = new Map<string, Set<Function>>();
  
  on(event: string, handler: Function) {
    if (!this.events.has(event)) {
      this.events.set(event, new Set());
    }
    this.events.get(event)!.add(handler);
    
    return () => this.off(event, handler);
  }
  
  off(event: string, handler: Function) {
    this.events.get(event)?.delete(handler);
  }
  
  emit(event: string, ...args: any[]) {
    this.events.get(event)?.forEach(handler => handler(...args));
  }
}

const eventBus = new EventBus();

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    function FieldExtension() {
      useEffect(() => {
        const unsubscribe = eventBus.on('configUpdated', (config) => {
          console.log('Config updated:', config);
        });
        
        return unsubscribe;
      }, []);
      
      const handleChange = (value) => {
        ctx.setFieldValue(ctx.fieldPath, value);
        eventBus.emit('fieldChanged', { field: ctx.fieldPath, value });
      };
      
      return <input onChange={e => handleChange(e.target.value)} />;
    }
    
    render(<FieldExtension />);
  },
  
  renderItemFormSidebarPanel(sidebarPanelId, ctx) {
    function SidebarPanel() {
      const [lastChange, setLastChange] = useState(null);
      
      useEffect(() => {
        return eventBus.on('fieldChanged', setLastChange);
      }, []);
      
      return (
        <div>
          Last change: {lastChange?.field} = {lastChange?.value}
        </div>
      );
    }
    
    render(<SidebarPanel />);
  }
});
```

## Best Practices

### 1. Hook Organization

```typescript
// Organize hooks by feature
const fieldExtensionHooks = {
  manualFieldExtensions() {
    return [/* ... */];
  },
  renderFieldExtension(id, ctx) {
    // ...
  }
};

const navigationHooks = {
  mainNavigationTabs() {
    return [/* ... */];
  },
  renderPage(pageId, ctx) {
    // ...
  }
};

const validationHooks = {
  onBeforeItemUpsert(item, ctx) {
    // ...
  },
  onBeforeItemsPublish(items, ctx) {
    // ...
  }
};

// Combine all hooks
connect({
  ...fieldExtensionHooks,
  ...navigationHooks,
  ...validationHooks
});
```

### 2. Error Handling

```typescript
// Centralized error handler
class ErrorHandler {
  handle(error: Error, ctx: any, context: string) {
    console.error(`Error in ${context}:`, error);
    
    if (ctx.alert) {
      ctx.alert(`An error occurred: ${error.message}`);
    }
    
    // Report to error tracking service
    if (window.Sentry) {
      window.Sentry.captureException(error, {
        tags: { plugin: ctx.plugin.id, context }
      });
    }
  }
}

const errorHandler = new ErrorHandler();

connect({
  async onBeforeItemUpsert(item, ctx) {
    try {
      return await validateAndTransformItem(item);
    } catch (error) {
      errorHandler.handle(error, ctx, 'onBeforeItemUpsert');
      return false; // Prevent save on error
    }
  }
});
```

### 3. Performance Optimization

```typescript
// Lazy load heavy components
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Cache expensive operations
const cache = new Map();

function getCachedData(key: string, fetcher: () => Promise<any>) {
  if (cache.has(key)) {
    return cache.get(key);
  }
  
  const promise = fetcher().then(data => {
    cache.set(key, data);
    return data;
  });
  
  cache.set(key, promise);
  return promise;
}
```

## Testing Multi-Hook Plugins

```typescript
// Test individual hooks
describe('Multi-hook Plugin', () => {
  it('should register field extensions', () => {
    const extensions = plugin.manualFieldExtensions();
    expect(extensions).toHaveLength(1);
    expect(extensions[0].id).toBe('myExtension');
  });
  
  it('should validate before publish', async () => {
    const items = [{ attributes: { title: '' } }];
    const result = await plugin.onBeforeItemsPublish(items, mockCtx);
    expect(result).toBe(false);
  });
  
  it('should share state between hooks', () => {
    // Test state sharing logic
  });
});
```

## Common Pitfalls

1. **Memory leaks**: Clean up subscriptions and listeners
2. **Race conditions**: Handle async operations carefully
3. **State sync**: Keep shared state consistent
4. **Error propagation**: Handle errors in each hook
5. **Performance**: Don't block the UI with heavy operations

## Related Documentation

- [Plugin State Management](./plugin-state-management.md)
- [Plugin Performance](./plugin-performance.md)
- [Hook Documentation](../02-hooks/README.md)