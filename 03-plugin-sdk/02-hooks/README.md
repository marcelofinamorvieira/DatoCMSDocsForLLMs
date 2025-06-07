# Plugin Hooks Reference

DatoCMS plugins use hooks to integrate with the interface. There are 44 hooks organized into categories based on their functionality.

## Documentation Status

✅ **Completed**: 44 hooks documented  
✅ **Remaining**: 0 hooks to document

## Hook Categories

### 🚀 Lifecycle Hooks
Control and validate content operations:

- ✅ **[onBoot](./onBoot.md)** - Plugin initialization and setup
- ✅ **[onBeforeItemUpsert](./onBeforeItemUpsert.md)** - Validate/transform before item create/update
- ✅ **[onBeforeItemsDestroy](./onBeforeItemsDestroy.md)** - Validate before item deletion
- ✅ **[onBeforeItemsPublish](./onBeforeItemsPublish.md)** - Validate before publishing
- ✅ **[onBeforeItemsUnpublish](./onBeforeItemsUnpublish.md)** - Validate before unpublishing

### 🎨 Rendering Hooks
Create custom UI components:

#### Field Extensions
- ✅ **[renderFieldExtension](./renderFieldExtension.md)** - Custom field editors
- ✅ **[manualFieldExtensions](./manualFieldExtensions.md)** - Define available field extensions
- ✅ **[overrideFieldExtensions](./overrideFieldExtensions.md)** - Override default field editors
- ✅ **[validateManualFieldExtensionParameters](./validateManualFieldExtensionParameters.md)** - Validate extension config
- ✅ **[renderManualFieldExtensionConfigScreen](./renderManualFieldExtensionConfigScreen.md)** - Extension settings UI

#### Pages & Modals
- ✅ **[renderPage](./renderPage.md)** - Full custom pages
- ✅ **[renderModal](./renderModal.md)** - Modal dialogs
- ✅ **[renderConfigScreen](./renderConfigScreen.md)** - Plugin configuration UI

#### Item Form Customization
- ✅ **[renderItemFormSidebar](./renderItemFormSidebar.md)** - Custom item form sidebar
- ✅ **[renderItemFormSidebarPanel](./renderItemFormSidebarPanel.md)** - Sidebar panel content
- ✅ **[itemFormSidebarPanels](./itemFormSidebarPanels.md)** - Define sidebar panels
- ✅ **[renderItemFormOutlet](./renderItemFormOutlet.md)** - Inject content into item form
- ✅ **[itemFormOutlets](./itemFormOutlets.md)** - Define injection points

#### Upload Customization
- ✅ **[renderUploadSidebar](./renderUploadSidebar.md)** - Custom upload sidebar
- ✅ **[renderUploadSidebarPanel](./renderUploadSidebarPanel.md)** - Upload sidebar panel content
- ✅ **[uploadSidebarPanels](./uploadSidebarPanels.md)** - Define upload sidebar panels

#### Collection Customization
- ✅ **[renderItemCollectionOutlet](./renderItemCollectionOutlet.md)** - Inject content into item lists
- ✅ **[itemCollectionOutlets](./itemCollectionOutlets.md)** - Define collection injection points

#### Asset Management
- ✅ **[renderAssetSource](./renderAssetSource.md)** - Custom asset source UI
- ✅ **[assetSources](./assetSources.md)** - Define external asset sources

### 🎯 Action Hooks
Add custom actions and behaviors:

#### Dropdown Actions
- ✅ **[executeItemsDropdownAction](./executeItemsDropdownAction.md)** - Execute bulk item actions
- ✅ **[itemsDropdownActions](./itemsDropdownActions.md)** - Define bulk item actions
- ✅ **[executeItemFormDropdownAction](./executeItemFormDropdownAction.md)** - Execute item form actions
- ✅ **[itemFormDropdownActions](./itemFormDropdownActions.md)** - Define item form actions
- ✅ **[executeFieldDropdownAction](./executeFieldDropdownAction.md)** - Execute field actions
- ✅ **[fieldDropdownActions](./fieldDropdownActions.md)** - Define field actions
- ✅ **[executeUploadsDropdownAction](./executeUploadsDropdownAction.md)** - Execute upload actions
- ✅ **[uploadsDropdownActions](./uploadsDropdownActions.md)** - Define upload actions
- ✅ **[executeSchemaItemTypeDropdownAction](./executeSchemaItemTypeDropdownAction.md)** - Execute model actions
- ✅ **[schemaItemTypeDropdownActions](./schemaItemTypeDropdownActions.md)** - Define model actions

### 🔧 Configuration Hooks
Customize DatoCMS interface:

#### Navigation
- ✅ **[mainNavigationTabs](./mainNavigationTabs.md)** - Add main navigation tabs
- ✅ **[settingsAreaSidebarItemGroups](./settingsAreaSidebarItemGroups.md)** - Add settings menu items
- ✅ **[contentAreaSidebarItems](./contentAreaSidebarItems.md)** - Add content area sidebar items

#### Content Presentation
- ✅ **[buildItemPresentationInfo](./buildItemPresentationInfo.md)** - Customize item preview
- ✅ **[initialLocationQueryForItemSelector](./initialLocationQueryForItemSelector.md)** - Set item selector defaults

#### Structured Text
- ✅ **[customBlockStylesForStructuredTextField](./customBlockStylesForStructuredTextField.md)** - Add block styles
- ✅ **[customMarksForStructuredTextField](./customMarksForStructuredTextField.md)** - Add text marks

### 📦 Other Hooks
- ✅ **[itemFormSidebars](./itemFormSidebars.md)** - Legacy sidebar definition
- ✅ **[uploadSidebars](./uploadSidebars.md)** - Legacy upload sidebar definition

## Hook Execution Order

Understanding when hooks execute helps you design better plugins:

```
1. Plugin Load
   └─ onBoot

2. Page Navigation
   ├─ mainNavigationTabs
   ├─ settingsAreaSidebarItemGroups
   └─ contentAreaSidebarItems

3. Item List View
   ├─ itemCollectionOutlets
   ├─ renderItemCollectionOutlet
   └─ itemsDropdownActions

4. Item Form View
   ├─ itemFormSidebars / itemFormSidebarPanels
   ├─ renderItemFormSidebar / renderItemFormSidebarPanel
   ├─ itemFormOutlets
   ├─ renderItemFormOutlet
   ├─ manualFieldExtensions
   ├─ overrideFieldExtensions
   ├─ renderFieldExtension
   └─ itemFormDropdownActions

5. Item Operations
   ├─ onBeforeItemUpsert (create/update)
   ├─ onBeforeItemsPublish
   ├─ onBeforeItemsUnpublish
   └─ onBeforeItemsDestroy

6. User Actions
   ├─ executeItemsDropdownAction
   ├─ executeItemFormDropdownAction
   ├─ executeFieldDropdownAction
   └─ renderModal
```

## TypeScript Support

All hooks are fully typed. Import types from the SDK:

```typescript
import { 
  RenderFieldExtensionCtx,
  OnBootCtx,
  ItemFormSidebarPanelCtx,
  // ... other context types
} from 'datocms-plugin-sdk';
```

## Common Patterns

### Conditional Hook Registration

```typescript
connect({
  // Only add SEO panel for article models
  itemFormSidebarPanels(ctx) {
    if (ctx.itemType.attributes.api_key === 'article') {
      return [{
        id: 'seo-panel',
        label: 'SEO Analysis'
      }];
    }
    return [];
  }
});
```

### Async Operations in Hooks

```typescript
connect({
  async onBeforeItemUpsert(item, ctx) {
    // Async validation
    const isValid = await validateWithExternalService(item);
    if (!isValid) {
      ctx.cancel('Validation failed');
    }
    return item;
  }
});
```

### Error Handling

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    try {
      // Render logic
    } catch (error) {
      ctx.alert('Error rendering field: ' + error.message);
      console.error(error);
    }
  }
});
```

## Hook Limitations

- **No direct DOM manipulation** outside of render hooks
- **Async operations** must complete within reasonable time
- **Memory management** - clean up event listeners and timers
- **API rate limits** apply to plugin API calls

## Getting Started

1. Choose hooks based on what you want to accomplish
2. Implement only the hooks you need
3. Test thoroughly in sandbox environment
4. Handle errors gracefully

## Related Documentation

- [Context Object](../01-core-concepts/context-object.md)
- [Plugin Manifest](../01-core-concepts/plugin-manifest.md)
- [UI Components](../03-ui-components/README.md)