# connect

The `connect` function is the main entry point for all DatoCMS plugins. It establishes communication between your plugin and the DatoCMS interface, registers your hook implementations, and sets up the plugin lifecycle.

## Purpose

The `connect` function:
- Establishes a secure communication channel with DatoCMS using [Penpal](https://github.com/Aaronius/penpal)
- Registers all the hooks your plugin implements
- Initializes the plugin based on the current rendering context
- Manages the plugin lifecycle and state updates

## Signature

```typescript
import { connect } from 'datocms-plugin-sdk';

async function connect(
  configuration?: Partial<FullConnectParameters>
): Promise<void>
```

## Parameters

| Prop | Type | Description |
|------|------|-------------|
| configuration | `Partial<FullConnectParameters>` | An object where keys are hook names and values are the hook implementations |

The `FullConnectParameters` type is a union of all available hook types in the SDK. You can implement any combination of the following hooks:

### Field Extensions
- `manualFieldExtensions` - Define custom field editors
- `overrideFieldExtensions` - Override default field editors
- `fieldDropdownActions` - Add dropdown actions to fields
- `executeFieldDropdownAction` - Handle field dropdown action clicks

### Item Form Customization
- `itemFormSidebars` - Add custom sidebars to record forms
- `itemFormSidebarPanels` - Add panels to record form sidebars
- `itemFormOutlets` - Add custom areas to record forms
- `itemFormDropdownActions` - Add dropdown actions to record forms
- `executeItemFormDropdownAction` - Handle record form dropdown action clicks

### Content Management
- `itemCollectionOutlets` - Add custom areas to record listings
- `itemsDropdownActions` - Add dropdown actions to record listings
- `executeItemsDropdownAction` - Handle record listing dropdown action clicks
- `buildItemPresentationInfo` - Customize how records are presented
- `initialLocationQueryForItemSelector` - Set initial filters for record selectors

### Asset Management
- `assetSources` - Define custom asset sources
- `uploadSidebars` - Add sidebars to asset detail views
- `uploadSidebarPanels` - Add panels to asset sidebars
- `uploadsDropdownActions` - Add dropdown actions to asset listings
- `executeUploadsDropdownAction` - Handle asset dropdown action clicks

### Navigation & Settings
- `mainNavigationTabs` - Add custom navigation tabs
- `settingsAreaSidebarItemGroups` - Add custom settings sections
- `contentAreaSidebarItems` - Add items to content area sidebar

### Schema Management
- `schemaItemTypeDropdownActions` - Add actions to model dropdowns
- `executeSchemaItemTypeDropdownAction` - Handle model dropdown clicks

### Structured Text
- `customMarksForStructuredTextField` - Add custom marks (inline styles)
- `customBlockStylesForStructuredTextField` - Add custom block styles

### Lifecycle Hooks
- `onBoot` - Execute code when plugin loads
- `onBeforeItemUpsert` - Validate/modify records before save
- `onBeforeItemsDestroy` - Handle before records are deleted
- `onBeforeItemsPublish` - Handle before records are published
- `onBeforeItemsUnpublish` - Handle before records are unpublished

### Rendering Hooks
- `renderConfigScreen` - Render plugin configuration UI
- `renderFieldExtension` - Render custom field editors
- `renderItemFormSidebar` - Render sidebar content
- `renderItemFormSidebarPanel` - Render sidebar panel content
- `renderItemFormOutlet` - Render outlet content
- `renderItemCollectionOutlet` - Render collection outlet content
- `renderPage` - Render full-page plugin views
- `renderModal` - Render modal dialogs
- `renderAssetSource` - Render custom asset source UI
- `renderUploadSidebar` - Render asset sidebar content
- `renderUploadSidebarPanel` - Render asset sidebar panel content
- `renderManualFieldExtensionConfigScreen` - Render field extension config

### Validation
- `validateManualFieldExtensionParameters` - Validate field extension parameters

## How It Works

1. **Connection Setup**: The function uses Penpal to establish a connection with the parent DatoCMS window
2. **Hook Registration**: It introspects your configuration object to determine which hooks you've implemented
3. **Context Initialization**: Based on the rendering mode, it initializes the appropriate context and bootstraps the UI
4. **Event Handling**: It sets up listeners for state changes and method calls from DatoCMS

## Basic Usage

### Minimal Plugin (Field Extension)

```typescript
import { connect, RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

connect({
  renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
    // Render your custom field editor
    const container = document.getElementById('root');
    
    // Access current field value
    const currentValue = ctx.formValues[ctx.fieldPath];
    
    // Update field value
    ctx.setFieldValue(ctx.fieldPath, 'new value');
    
    // Render your UI here
  }
});
```

### Multi-Hook Plugin

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  // Initialize plugin
  onBoot(ctx) {
    ctx.notice('Plugin loaded successfully!');
  },
  
  // Add navigation tab
  mainNavigationTabs(ctx) {
    return [
      {
        id: 'my-plugin',
        label: 'My Plugin',
        icon: 'pencil',
        pointsTo: {
          pageId: 'my-page'
        }
      }
    ];
  },
  
  // Render the page
  renderPage(pageId, ctx) {
    if (pageId === 'my-page') {
      // Render your page content
    }
  },
  
  // Add field editor
  manualFieldExtensions(ctx) {
    return [
      {
        id: 'my-editor',
        name: 'My Custom Editor',
        type: 'editor',
        fieldTypes: ['string', 'text']
      }
    ];
  },
  
  renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'my-editor') {
      // Render custom field editor
    }
  }
});
```

### Lifecycle-Only Plugin

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  // Validate records before saving
  onBeforeItemUpsert(createOrUpdateItemPayload, ctx) {
    const { item_type, data } = createOrUpdateItemPayload;
    
    // Validate title is not empty
    if (data.title === '') {
      return {
        valid: false,
        errors: [
          {
            field: 'title',
            message: 'Title cannot be empty'
          }
        ]
      };
    }
    
    // Auto-generate slug from title
    if (!data.slug && data.title) {
      return {
        valid: true,
        overriddenData: {
          ...data,
          slug: data.title.toLowerCase().replace(/\s+/g, '-')
        }
      };
    }
    
    return { valid: true };
  },
  
  // Log when items are deleted
  onBeforeItemsDestroy(items, ctx) {
    console.log(`Deleting ${items.length} items`);
    return { valid: true };
  }
});
```

## Advanced Examples

### Plugin with Configuration Screen

```typescript
import { connect, RenderConfigScreenCtx } from 'datocms-plugin-sdk';

connect({
  renderConfigScreen(ctx: RenderConfigScreenCtx) {
    const container = document.getElementById('root');
    
    // Get current configuration
    const config = ctx.plugin.attributes.parameters;
    
    // Create configuration form
    const form = document.createElement('form');
    
    const apiKeyInput = document.createElement('input');
    apiKeyInput.value = config.apiKey || '';
    apiKeyInput.placeholder = 'Enter API key';
    
    const saveButton = document.createElement('button');
    saveButton.textContent = 'Save';
    
    form.appendChild(apiKeyInput);
    form.appendChild(saveButton);
    
    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      
      // Update plugin configuration
      await ctx.updatePluginParameters({
        apiKey: apiKeyInput.value
      });
      
      ctx.notice('Configuration saved!');
    });
    
    container.appendChild(form);
  },
  
  renderFieldExtension(fieldExtensionId, ctx) {
    // Access configuration in other hooks
    const apiKey = ctx.plugin.attributes.parameters.apiKey;
    
    if (!apiKey) {
      ctx.alert('Please configure the plugin first');
      return;
    }
    
    // Use the API key...
  }
});
```

### Asset Source Plugin

```typescript
import { connect } from 'datocms-plugin-sdk';

connect({
  assetSources(ctx) {
    return [
      {
        id: 'unsplash',
        name: 'Unsplash',
        icon: {
          type: 'svg',
          viewBox: '0 0 24 24', 
          content: '<path d="..."/>'
        }
      }
    ];
  },
  
  renderAssetSource(assetSourceId, ctx) {
    if (assetSourceId === 'unsplash') {
      const container = document.getElementById('root');
      
      // Render search interface
      const searchInput = document.createElement('input');
      searchInput.placeholder = 'Search Unsplash...';
      
      searchInput.addEventListener('input', async (e) => {
        const query = e.target.value;
        
        // Search Unsplash API
        const results = await searchUnsplash(query);
        
        // Display results
        results.forEach(photo => {
          const img = document.createElement('img');
          img.src = photo.urls.thumb;
          
          img.addEventListener('click', () => {
            // Select this asset
            ctx.select({
              resource: {
                url: photo.urls.full,
                filename: `${photo.id}.jpg`
              }
            });
          });
          
          container.appendChild(img);
        });
      });
      
      container.appendChild(searchInput);
    }
  }
});
```

## Best Practices

1. **Type Safety**: Always import and use the proper TypeScript types for hook parameters and context objects
2. **Error Handling**: Wrap your hook implementations in try-catch blocks to handle errors gracefully
3. **Performance**: For render hooks, avoid heavy computations in the render path - use memoization when needed
4. **State Management**: Use a proper state management solution (React, Vue, etc.) for complex UIs
5. **Configuration**: Store plugin configuration using `updatePluginParameters` for persistence
6. **User Feedback**: Use `ctx.notice()` for success messages and `ctx.alert()` for errors

## Common Patterns

### Conditional Hook Registration

```typescript
const isDevelopment = process.env.NODE_ENV === 'development';

connect({
  // Always register these
  renderFieldExtension() { /* ... */ },
  
  // Only in development
  ...(isDevelopment && {
    onBoot(ctx) {
      console.log('Plugin context:', ctx);
    }
  })
});
```

### Hook Composition

```typescript
// Separate hook implementations
const fieldExtensions = {
  renderFieldExtension() { /* ... */ },
  manualFieldExtensions() { /* ... */ }
};

const sidebarFeatures = {
  itemFormSidebars() { /* ... */ },
  renderItemFormSidebar() { /* ... */ }
};

// Compose them
connect({
  ...fieldExtensions,
  ...sidebarFeatures,
  onBoot() { /* ... */ }
});
```

### Async Initialization

```typescript
// Since connect returns a Promise, you can await it
async function initializePlugin() {
  try {
    await connect({
      async onBoot(ctx) {
        // Perform async initialization
        const config = await fetchConfiguration();
        
        // Store in plugin parameters
        await ctx.updatePluginParameters(config);
      }
    });
  } catch (error) {
    console.error('Failed to initialize plugin:', error);
  }
}

initializePlugin();
```

## Related

- [Plugin Manifest](./plugin-manifest.md) - Structure of the plugin.json file
- [Context Object](./context-object.md) - The ctx parameter passed to all hooks
- [Hooks Overview](../02-hooks/README.md) - Detailed documentation for each hook