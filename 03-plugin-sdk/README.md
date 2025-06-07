# Plugin SDK

Build custom plugins to extend the DatoCMS interface with your own functionality.

## Installation

```bash
npm install datocms-plugin-sdk datocms-react-ui
```

## Quick Start

```typescript
import { connect, RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

connect({
  renderFieldExtension(fieldExtensionId: string, ctx: RenderFieldExtensionCtx) {
    // Your custom field editor logic
  }
});
```

## Documentation Structure

### [01. Core Concepts](./01-core-concepts/)
- **[context-object.md](./01-core-concepts/context-object.md)**: Understanding the `ctx` object available in all hooks
- **[plugin-manifest.md](./01-core-concepts/plugin-manifest.md)**: Plugin configuration and manifest.json
- **[css-system-and-theming.md](./01-core-concepts/css-system-and-theming.md)**: CSS variables, theming, and styling guidelines
- **[plugin-lifecycle.md](./01-core-concepts/plugin-lifecycle.md)**: Plugin bootstrapping and initialization
- **[sdk-manifest.md](./01-core-concepts/sdk-manifest.md)**: SDK manifest generation system
- **[utility-functions.md](./01-core-concepts/utility-functions.md)**: Type guards and utility functions

### [02. Hooks](./02-hooks/)
Complete documentation for all 44 plugin hooks, organized by category:

#### Lifecycle Hooks
- **[onBoot.md](./02-hooks/onBoot.md)**: Plugin initialization
- **[onBeforeItemUpsert.md](./02-hooks/onBeforeItemUpsert.md)**: Validate/transform before save
- **[onBeforeItemsDestroy.md](./02-hooks/onBeforeItemsDestroy.md)**: Validate before delete
- **[onBeforeItemsPublish.md](./02-hooks/onBeforeItemsPublish.md)**: Validate before publish
- **[onBeforeItemsUnpublish.md](./02-hooks/onBeforeItemsUnpublish.md)**: Validate before unpublish

#### Rendering Hooks
- **[renderFieldExtension.md](./02-hooks/renderFieldExtension.md)**: Custom field editors
- **[renderPage.md](./02-hooks/renderPage.md)**: Custom pages
- **[renderModal.md](./02-hooks/renderModal.md)**: Modal dialogs
- **[renderItemFormSidebar.md](./02-hooks/renderItemFormSidebar.md)**: Item form sidebars
- **[renderConfigScreen.md](./02-hooks/renderConfigScreen.md)**: Plugin settings

#### Action Hooks
- **[executeItemsDropdownAction.md](./02-hooks/executeItemsDropdownAction.md)**: Bulk actions on items
- **[executeFieldDropdownAction.md](./02-hooks/executeFieldDropdownAction.md)**: Field-specific actions
- **[executeItemFormDropdownAction.md](./02-hooks/executeItemFormDropdownAction.md)**: Item form actions

[See all 44 hooks →](./02-hooks/README.md)

### [03. UI Components](./03-ui-components/)
React components for building plugin interfaces:

#### Form Components
- **[Button.md](./03-ui-components/Button.md)**: Buttons with variants
- **[TextField.md](./03-ui-components/TextField.md)**: Text input fields
- **[SelectField.md](./03-ui-components/SelectField.md)**: Dropdown selects
- **[SwitchField.md](./03-ui-components/SwitchField.md)**: Toggle switches

#### Layout Components
- **[Canvas.md](./03-ui-components/Canvas.md)**: Main content container
- **[SplitView.md](./03-ui-components/SplitView.md)**: Resizable split panels
- **[SidebarPanel.md](./03-ui-components/SidebarPanel.md)**: Sidebar sections

[See all components →](./03-ui-components/README.md)

## Common Patterns

### Basic Field Extension

```typescript
import { connect } from 'datocms-plugin-sdk';
import { Canvas, TextField } from 'datocms-react-ui';

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    const value = ctx.formValues[ctx.fieldPath] || '';
    
    ReactDOM.render(
      <Canvas ctx={ctx}>
        <TextField
          value={value}
          onChange={(newValue) => ctx.setFieldValue(ctx.fieldPath, newValue)}
        />
      </Canvas>,
      document.getElementById('root')
    );
  }
});
```

### Sidebar Panel

```typescript
connect({
  itemFormSidebarPanels() {
    return [
      {
        id: 'seo-analysis',
        label: 'SEO Analysis',
        startOpen: true
      }
    ];
  },
  
  renderItemFormSidebarPanel(sidebarPanelId, ctx) {
    if (sidebarPanelId === 'seo-analysis') {
      // Render SEO analysis panel
    }
  }
});
```

### Modal Actions

```typescript
connect({
  itemsDropdownActions(ctx) {
    return [
      {
        id: 'export-csv',
        label: 'Export to CSV',
        icon: 'download'
      }
    ];
  },
  
  async executeItemsDropdownAction(actionId, ctx) {
    if (actionId === 'export-csv') {
      const result = await ctx.openModal({
        id: 'export-options',
        title: 'Export Options',
        width: 'medium'
      });
      
      if (result) {
        // Perform export with selected options
      }
    }
  }
});
```

## Plugin Development Workflow

1. **Initialize Plugin**
   ```bash
   npm init datocms-plugin my-plugin
   cd my-plugin
   npm install
   ```

2. **Configure Manifest**
   Edit `package.json`:
   ```json
   {
     "datoCmsPlugin": {
       "title": "My Plugin",
       "previewImage": "preview.png",
       "entryPoint": "./dist/index.js",
       "fieldTypes": ["string", "text"],
       "pluginType": "field_editor",
       "parameters": {
         "global": [],
         "instance": []
       }
     }
   }
   ```

3. **Develop Locally**
   ```bash
   npm run dev
   ```

4. **Test in DatoCMS**
   Use the sandbox environment for testing

5. **Deploy**
   ```bash
   npm run build
   npm run publish
   ```

## TypeScript Support

The SDK includes comprehensive TypeScript definitions:

```typescript
import { 
  RenderFieldExtensionCtx,
  RenderModalCtx,
  ItemFormSidebarPanelCtx 
} from 'datocms-plugin-sdk';

// All contexts are fully typed
function myHook(ctx: RenderFieldExtensionCtx) {
  // TypeScript knows all available properties
  ctx.field.api_key;
  ctx.currentUser.email;
  ctx.formValues[ctx.fieldPath];
}
```

## Best Practices

1. **Performance**: Keep renders fast, debounce updates
2. **Error Handling**: Always handle errors gracefully
3. **Accessibility**: Use semantic HTML and ARIA attributes
4. **Localization**: Support multiple locales when applicable
5. **Testing**: Test with different field types and edge cases

## Related Documentation

- [CMA Client](../01-cma-client/README.md) - Access DatoCMS APIs from plugins
- [UI Components Guide](./03-ui-components/README.md) - Complete component reference
- [Hooks Reference](./02-hooks/README.md) - All available plugin hooks
- [Advanced Topics](../04-advanced-topics/) - Performance and patterns