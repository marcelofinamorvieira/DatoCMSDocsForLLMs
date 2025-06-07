# Context Object

The context object (`ctx`) is the primary interface between your plugin and DatoCMS. It provides access to data, UI controls, and methods for interacting with the CMS. Every hook receives a context object tailored to its specific capabilities.

## Overview

The context object serves as:
- **Data Provider**: Access to current user, project, records, and configuration
- **UI Controller**: Methods to show notifications, open dialogs, and navigate
- **State Manager**: Read and update form values, field states, and plugin parameters
- **Communication Bridge**: Interface between your plugin iframe and DatoCMS

## Base Context Properties

All context objects include these fundamental properties and methods:

### Plugin Information

| Property | Type | Description |
|----------|------|-------------|
| `plugin` | `Plugin` | Current plugin instance with id, attributes, and parameters |

### Authentication & Permissions

| Property | Type | Description |
|----------|------|-------------|
| `currentUser` | `User \| SsoUser \| Account \| Organization` | The current DatoCMS user. It can either be the owner or one of the collaborators (regular or SSO) |
| `currentRole` | `Role` | The role for the current DatoCMS user |
| `currentUserAccessToken` | `string \| undefined` | The access token to perform API calls on behalf of the current user. Only available if `currentUserAccessToken` additional permission is granted |
| `account` | `Account \| undefined` | Deprecated - use `owner` instead |

### Project Information

| Property | Type | Description |
|----------|------|-------------|
| `site` | `Site` | The current DatoCMS project |
| `environment` | `string` | The ID of the current environment |
| `isEnvironmentPrimary` | `boolean` | Whether the current environment is the primary one |
| `owner` | `Account \| Organization` | The account/organization that is the project owner |
| `ui` | `{ locale: string }` | UI preferences of the current user (right now, only the preferred locale is available) |
| `theme` | `Theme` | An object containing the theme colors for the current DatoCMS project |

### Theme Object Structure

The `theme` object contains the color palette for the current DatoCMS project:

```typescript
interface Theme {
  primaryColor: string;               // Primary brand color (e.g., "#ff6900")
  accentColor: string;               // Accent color (e.g., "#ff6900")
  semiTransparentAccentColor: string; // Semi-transparent accent (e.g., "rgba(255, 105, 0, 0.1)")
  lightColor: string;                // Light background color (e.g., "#f0f0f0")
  darkColor: string;                 // Dark text color (e.g., "#333333")
}
```

#### Using Theme Colors

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Apply theme colors to your plugin
    const styles = {
      backgroundColor: ctx.theme.lightColor,
      color: ctx.theme.darkColor,
      accentColor: ctx.theme.primaryColor,
      borderColor: ctx.theme.semiTransparentAccentColor
    };
  }
});
```

### Entity Repositories

These properties provide access to "entity repos", that is, the collection of resources of a particular type that have been loaded by the CMS up to this moment. The entity repos are objects, indexed by the ID of the entity itself.

| Property | Type | Description |
|----------|------|-------------|
| `itemTypes` | `Partial<Record<string, ItemType>>` | All the models of the current DatoCMS project, indexed by ID |
| `fields` | `Partial<Record<string, Field>>` | All the fields currently loaded for the current DatoCMS project, indexed by ID |
| `fieldsets` | `Partial<Record<string, Fieldset>>` | All the fieldsets currently loaded for the current DatoCMS project, indexed by ID |
| `users` | `Partial<Record<string, User>>` | All the regular users currently loaded for the current DatoCMS project, indexed by ID |
| `ssoUsers` | `Partial<Record<string, SsoUser>>` | All the SSO users currently loaded for the current DatoCMS project, indexed by ID |

#### Working with Entity Repositories

```typescript
// Get a specific model by ID
const blogModel = ctx.itemTypes['123456'];

// Convert to array to iterate all models
const allModels = Object.values(ctx.itemTypes);
allModels.forEach(model => {
  console.log(model.attributes.name);
});

// Get all fields for a specific model
const modelFields = Object.values(ctx.fields).filter(
  field => field.relationships.item_type.data.id === modelId
);
```

## Base Context Methods

### Data Loading

These methods can be used to asynchronously load additional information your plugin needs to work:

```typescript
// Loads all the fields for a specific model (or block)
loadItemTypeFields(itemTypeId: string): Promise<Field[]>

// Loads all the fieldsets for a specific model (or block)
loadItemTypeFieldsets(itemTypeId: string): Promise<Fieldset[]>

// Loads all the fields in the project that are currently using the plugin
// for one of its manual field extensions
loadFieldsUsingPlugin(): Promise<Field[]>

// Loads all regular users
loadUsers(): Promise<User[]>

// Loads all SSO users
loadSsoUsers(): Promise<SsoUser[]>
```

#### Data Loading Examples

```typescript
// Load fields for a specific model
const itemTypeId = prompt('Please insert a model ID:');
const fields = await ctx.loadItemTypeFields(itemTypeId);

ctx.notice(
  `Success! ${fields
    .map((field) => field.attributes.api_key)
    .join(', ')}`,
);

// Load all fields using this plugin
const fields = await ctx.loadFieldsUsingPlugin();
console.log(`${fields.length} fields are using this plugin`);
```

### Plugin Configuration

These methods can be used to update both plugin parameters and manual field extensions configuration:

```typescript
// Updates the plugin parameters
updatePluginParameters(params: Record<string, unknown>): Promise<void>

// Performs changes in the appearance of a field
updateFieldAppearance(
  fieldId: string,
  changes: FieldAppearanceChange[]
): Promise<void>
```

#### Field Appearance Change Types

```typescript
type FieldAppearanceChange =
  | { operation: 'removeEditor' }
  | { operation: 'updateEditor'; 
      newFieldExtensionId?: string; 
      newParameters?: Record<string, unknown> }
  | { operation: 'setEditor'; 
      fieldExtensionId: string; 
      parameters: Record<string, unknown> }
  | { operation: 'removeAddon'; 
      index: number }
  | { operation: 'updateAddon'; 
      index: number; 
      newFieldExtensionId?: string; 
      newParameters?: Record<string, unknown> }
  | { operation: 'insertAddon'; 
      index: number; 
      fieldExtensionId: string; 
      parameters: Record<string, unknown> }
```

#### Plugin Configuration Examples

```typescript
// Update plugin parameters
await ctx.updatePluginParameters({ debugMode: true });
await ctx.notice('Plugin parameters successfully updated!');

// Update field appearance
const fields = await ctx.loadFieldsUsingPlugin();

for (const field of fields) {
  const { appearance } = field.attributes;
  const operations = [];

  if (appearance.editor === ctx.plugin.id) {
    operations.push({
      operation: 'updateEditor',
      newParameters: {
        ...appearance.parameters,
        foo: 'bar',
      },
    });
  }

  await ctx.updateFieldAppearance(field.id, operations);
}
```

**Note:** Always check `ctx.currentRole.meta.final_permissions.can_edit_schema` before calling these methods, as the user might not have permission to perform the operation.

### Toast Notifications

```typescript
// Triggers an "error" toast displaying the selected message
alert(message: string): Promise<void>

// Triggers a "success" toast displaying the selected message
notice(message: string): Promise<void>

// Triggers a custom toast displaying the selected message (and optionally a CTA)
customToast<CtaValue = unknown>(toast: Toast<CtaValue>): Promise<CtaValue | null>
```

#### Toast Type Definition

```typescript
type Toast<CtaValue = unknown> = {
  /** Message of the notification */
  message: string;
  /** Type of notification. Will present the toast in a different color accent. */
  type: 'notice' | 'alert' | 'warning';
  /** An optional button to show inside the toast */
  cta?: {
    /** Label for the button */
    label: string;
    /** The value to be returned by the customToast promise if clicked */
    value: CtaValue;
  };
  /** Whether the toast is to be automatically closed if the user changes page */
  dismissOnPageChange?: boolean;
  /** Whether the toast is to be automatically closed after some time */
  dismissAfterTimeout?: boolean | number;
};
```

#### Toast Examples

```typescript
// Simple notifications
await ctx.alert('This is an error message!');
await ctx.notice('Operation completed successfully!');

// Custom toast with CTA
const result = await ctx.customToast({
  type: 'warning',
  message: 'Just a sample warning notification!',
  dismissOnPageChange: true,
  dismissAfterTimeout: 5000,
  cta: {
    label: 'Execute call-to-action',
    value: 'cta',
  },
});

if (result === 'cta') {
  ctx.notice(`Clicked CTA!`);
}
```

### Item (Record) Dialogs

```typescript
// Opens a dialog for creating a new record
createNewItem(itemTypeId: string): Promise<Item | null>

// Opens a dialog for selecting one (or multiple) record(s)
selectItem: {
  (itemTypeId: string, options: { 
    multiple: true; 
    initialLocationQuery?: ItemListLocationQuery 
  }): Promise<Item[] | null>;
  (itemTypeId: string, options?: { 
    multiple: false; 
    initialLocationQuery?: ItemListLocationQuery 
  }): Promise<Item | null>;
}

// Opens a dialog for editing an existing record
editItem(itemId: string): Promise<Item | null>
```

#### ItemListLocationQuery Type

```typescript
type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};
```

#### Item Dialog Examples

```typescript
// Create a new record
const itemTypeId = prompt('Please insert a model ID:');
const item = await ctx.createNewItem(itemTypeId);

if (item) {
  ctx.notice(`Success! Created item ${item.id}`);
}

// Select multiple records with filters
const items = await ctx.selectItem('blog_post', { 
  multiple: true,
  initialLocationQuery: {
    filter: {
      fields: {
        status: { eq: 'published' }
      }
    }
  }
});

if (items) {
  ctx.notice(`Selected ${items.length} items`);
}

// Edit an existing record
const itemId = prompt('Please insert a record ID:');
const editedItem = await ctx.editItem(itemId);

if (editedItem) {
  ctx.notice(`Successfully edited ${editedItem.id}`);
}
```

### Upload (Asset) Dialogs

```typescript
// Opens a dialog for selecting one (or multiple) existing asset(s)
selectUpload: {
  (options: { multiple: true }): Promise<Upload[] | null>;
  (options?: { multiple: false }): Promise<Upload | null>;
}

// Opens a dialog for editing a Media Area asset
editUpload(uploadId: string): Promise<(Upload & { deleted?: true }) | null>

// Opens a dialog for editing a "single asset" field structure
editUploadMetadata(
  fileFieldValue: FileFieldValue,
  locale?: string
): Promise<FileFieldValue | null>
```

#### FileFieldValue Type

```typescript
type FileFieldValue = {
  /** ID of the asset */
  upload_id: string;
  /** Alternate text for the asset */
  alt: string | null;
  /** Title for the asset */
  title: string | null;
  /** Focal point of an asset */
  focal_point: FocalPoint | null;
  /** Object with arbitrary metadata related to the asset */
  custom_data: Record<string, string>;
};

type FocalPoint = {
  /** Horizontal position expressed as float between 0 and 1 */
  x: number;
  /** Vertical position expressed as float between 0 and 1 */
  y: number;
};
```

#### Upload Dialog Examples

```typescript
// Select a single upload
const upload = await ctx.selectUpload({ multiple: false });
if (upload) {
  ctx.notice(`Selected: ${upload.id}`);
}

// Select multiple uploads
const uploads = await ctx.selectUpload({ multiple: true });
if (uploads) {
  ctx.notice(`Selected ${uploads.length} uploads`);
}

// Edit upload metadata
const uploadId = prompt('Please insert an asset ID:');
const result = await ctx.editUploadMetadata({
  upload_id: uploadId,
  alt: null,
  title: null,
  custom_data: {},
  focal_point: null,
});

if (result) {
  ctx.notice(`Success! ${JSON.stringify(result)}`);
}
```

### Custom Dialogs

```typescript
// Opens a custom modal
openModal(modal: Modal): Promise<unknown>

// Opens a UI-consistent confirmation dialog
openConfirm(options: ConfirmOptions): Promise<unknown>
```

#### Modal Type

```typescript
type Modal = {
  /** ID of the modal. Will be the first argument for the renderModal function */
  id: string;
  /** Title for the modal. Ignored by fullWidth modals */
  title?: string;
  /** Whether to present a close button for the modal or not */
  closeDisabled?: boolean;
  /** Width of the modal. Can be a number, or one of the predefined sizes */
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  /** An arbitrary configuration object that will be passed as the parameters property */
  parameters?: Record<string, unknown>;
  /** The initial height to set for the iframe that will render the modal content */
  initialHeight?: number;
};
```

#### ConfirmOptions Type

```typescript
type ConfirmOptions = {
  /** The title to be shown inside the confirmation panel */
  title: string;
  /** The main message to be shown inside the confirmation panel */
  content: string;
  /** The different options the user can choose from */
  choices: ConfirmChoice[];
  /** The cancel option to present to the user */
  cancel: ConfirmChoice;
};

type ConfirmChoice = {
  /** The label to be shown for the choice */
  label: string;
  /** The value to be returned by the openConfirm promise if clicked */
  value: unknown;
  /** The intent of the button. Will present the button in a different color accent */
  intent?: 'positive' | 'negative';
};
```

#### Custom Dialog Examples

```typescript
// Open a custom modal
const result = await ctx.openModal({
  id: 'custom-modal',
  title: 'Custom Modal',
  width: 'l',
  parameters: { foo: 'bar' },
});

if (result) {
  ctx.notice(`Modal returned: ${JSON.stringify(result)}`);
}

// Confirmation dialog
const confirmed = await ctx.openConfirm({
  title: 'Delete this item?',
  content: 'This action cannot be undone. All data will be permanently deleted.',
  choices: [
    {
      label: 'Delete',
      value: 'delete',
      intent: 'negative',
    },
  ],
  cancel: {
    label: 'Cancel',
    value: false,
  },
});

if (confirmed === 'delete') {
  // Perform deletion
}
```

### Navigation

```typescript
// Moves the user to another URL internal to the backend
navigateTo(path: string): Promise<void>
```


## Context Variants

The SDK provides two main context variants based on how the plugin is rendered:

### ImposedSizePluginFrameCtx

Used when the plugin iframe has a fixed size imposed by the container. This is used for full-screen views like pages and boot mode.

```typescript
// From packages/sdk/src/ctx/pluginFrame.ts
export type ImposedSizePluginFrameCtx<
  Mode extends string,
  ExtraProperties extends Record<string, unknown>,
  ExtraMethods extends Record<string, unknown>,
> = Ctx<ExtraProperties, ExtraMethods> & PluginFrameProperties<Mode>;

interface PluginFrameProperties<Mode extends string> {
  mode: Mode;               // Current rendering mode
  bodyPadding: number[];    // [top, right, bottom, left] padding
  getSettings(): Promise<Properties>; // Get current properties
}
```

### SelfResizingPluginFrameCtx  

Used when the plugin can control its own height. This extends ImposedSizePluginFrameCtx with sizing utilities and iframe methods.

```typescript
// From packages/sdk/src/ctx/pluginFrame.ts
export type SelfResizingPluginFrameCtx<
  Mode extends string,
  ExtraProperties extends Record<string, unknown>,
  ExtraMethods extends Record<string, unknown>,
> = ImposedSizePluginFrameCtx<Mode, ExtraProperties, ExtraMethods> &
  SizingUtilities &
  IframeMethods;
```

#### IframeMethods Interface

The `IframeMethods` interface provides methods to control properties of the containing iframe:

```typescript
// From packages/sdk/src/ctx/commonExtras/sizing.ts
export type IframeMethods = {
  /** Sets the height for the iframe */
  setHeight: (number: number) => Promise<void>;
};
```

#### SizingUtilities Interface

The `SizingUtilities` interface provides comprehensive control over iframe sizing:

```typescript
// From packages/sdk/src/ctx/commonExtras/sizing.ts
export type SizingUtilities = {
  /**
   * Listens for DOM changes and automatically calls `setHeight` when it detects
   * a change. If you're using `datocms-react-ui` package, the `<Canvas />`
   * component already takes care of calling this method for you.
   */
  startAutoResizer: () => void;

  /** Stops resizing the iframe automatically */
  stopAutoResizer: () => void;

  /**
   * Triggers a change in the size of the iframe. If you don't explicitly pass
   * a `newHeight` it will be automatically calculated using the iframe content
   * at the moment
   */
  updateHeight: (newHeight?: number) => void;

  /** Whether the auto-resizer is currently active or not */
  isAutoResizerActive(): boolean;
};
```

#### Sizing Implementation Details

The sizing utilities use sophisticated DOM monitoring to ensure proper iframe sizing:

1. **Auto-resizing**: Uses both `ResizeObserver` and `MutationObserver` to detect changes
2. **Height calculation**: Considers multiple factors:
   - `document.body.scrollHeight`
   - `document.body.offsetHeight`
   - `document.documentElement.getBoundingClientRect().height`
   - Bottom position of all elements in the document
3. **Optimization**: Only calls `setHeight` when the height actually changes

#### Available in These Contexts

Self-resizing contexts include:
- `renderFieldExtension`
- `renderConfigScreen`
- `renderPage`
- `renderModal`
- `renderAssetSource`
- `renderItemCollectionOutlet`
- `renderItemFormOutlet`
- `renderItemFormSidebar`
- `renderItemFormSidebarPanel`
- `renderManualFieldExtensionConfigScreen`
- `renderUploadSidebar`
- `renderUploadSidebarPanel`

#### Sizing Examples

```typescript
// Manual height control
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Set explicit height
    ctx.setHeight(300);
    
    // Update height based on content
    ctx.updateHeight(); // Auto-calculate
    ctx.updateHeight(400); // Explicit height
    
    // Start auto-resizing
    ctx.startAutoResizer();
    
    // Check if auto-resizer is active
    if (ctx.isAutoResizerActive()) {
      console.log('Auto-resizer is running');
    }
    
    // Stop auto-resizing for performance
    ctx.stopAutoResizer();
  }
});

// Using Canvas component (auto-resizer built-in)
import { Canvas } from 'datocms-react-ui';

connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    render(
      <Canvas ctx={ctx}>
        {/* Canvas automatically handles resizing */}
        <YourComponent />
      </Canvas>
    );
  }
});
```


### SizingUtilities

These utilities help manage the plugin iframe's height dynamically:

```typescript
// From packages/sdk/src/ctx/commonExtras/sizing.ts
/** A number of methods that you can use to control the size of the plugin frame */
export type SizingUtilities = {
  /**
   * Listens for DOM changes and automatically calls `setHeight` when it detects
   * a change. If you're using `datocms-react-ui` package, the `<Canvas />`
   * component already takes care of calling this method for you.
   */
  startAutoResizer: () => void;

  /** Stops resizing the iframe automatically */
  stopAutoResizer: () => void;

  /**
   * Triggers a change in the size of the iframe. If you don't explicitely pass
   * a `newHeight` it will be automatically calculated using the iframe content
   * at the moment
   */
  updateHeight: (newHeight?: number) => void;

  /** Wheter the auto-resizer is currently active or not */
  isAutoResizerActive(): boolean;
};
```

### IframeMethods

Direct control over iframe properties:

```typescript
// From packages/sdk/src/ctx/commonExtras/sizing.ts
/** These methods can be used to set various properties of the containing iframe */
export type IframeMethods = {
  /** Sets the height for the iframe */
  setHeight: (number: number) => Promise<void>;
};
```

### How Auto-Resizing Works

The auto-resizer uses:
1. **ResizeObserver**: Watches for size changes on the document element
2. **MutationObserver**: Detects DOM mutations (additions, removals, attribute changes)
3. **Height Calculation**: Computes the maximum height needed by:
   - `document.body.scrollHeight`
   - `document.body.offsetHeight`
   - `document.documentElement.getBoundingClientRect().height`
   - Maximum bottom position of all elements

### Usage Examples

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Option 1: Auto-resize (recommended for dynamic content)
    ctx.startAutoResizer();
    
    // The iframe will now automatically adjust its height
    // when content changes
    
    // Check if auto-resizer is active
    if (ctx.isAutoResizerActive()) {
      console.log('Auto-resizing is enabled');
    }
    
    // Option 2: Manual height control
    ctx.stopAutoResizer(); // Stop auto-resizing first
    ctx.setHeight(400);    // Set to exactly 400px
    
    // Option 3: One-time height update
    ctx.updateHeight();    // Calculates and sets current height
    ctx.updateHeight(500); // Or set specific height
  }
});
```

### Best Practices

1. **Use with Canvas Component**: If using `datocms-react-ui`, the `<Canvas />` component automatically calls `startAutoResizer()`
2. **Dynamic Content**: Use `startAutoResizer()` for content that changes frequently
3. **Fixed Layouts**: Use `setHeight()` for predictable, fixed-height interfaces
4. **Performance**: Stop the auto-resizer when not needed to save resources
5. **Transitions**: Call `updateHeight()` after animations complete

## Context Extensions

Additional properties and methods are added based on the hook type:

### Field Extension Properties

When rendering field editors, these properties are added:

```typescript
interface FieldAdditionalProperties {
  disabled: boolean         // Whether field is disabled
  fieldPath: string         // Path in formValues (e.g., 'title.en')
  field: Field              // Field definition
  parentField?: Field       // Parent if nested in modular/structured
  block?: {                 // Block context if inside a block
    id: string
    blockModel: ItemType
  }
}
```

### Item Form Properties

When accessing record forms, these are added:

```typescript
interface ItemFormAdditionalProperties {
  locale: string                    // Active locale
  item: Item | null                 // Current record or null if new
  itemType: ItemType                // Model definition
  formValues: Record<string, any>   // Complete form state
  itemStatus: 'new' | 'draft' | 'updated' | 'published'
  isSubmitting: boolean             // Form submission state
  isFormDirty: boolean              // Unsaved changes
  blocksAnalysis: BlocksAnalysis    // Block usage statistics
}

interface BlocksAnalysis {
  usage: {
    /** Total number of blocks present in form state */
    total: number;
    /** Total number of blocks present in non-localized fields */
    nonLocalized: number;
    /** Total number of blocks present in localized fields, per locale */
    perLocale: Record<string, number>;
  };
  /** Maximum number of blocks per item */
  maximumPerItem: number;
}
```

### Item Form Methods

```typescript
interface ItemFormAdditionalMethods {
  // Field visibility
  toggleField(path: string, show: boolean): Promise<void>
  
  // Field state
  disableField(path: string, disable: boolean): Promise<void>
  
  // Navigation
  scrollToField(path: string, locale?: string): Promise<void>
  
  // Value management
  setFieldValue(path: string, value: unknown): Promise<void>
  
  // Form conversion
  // Returns undefined if required nested blocks are not yet loaded
  formValuesToItem(
    formValues: Record<string, any>, 
    skipUnchanged?: boolean
  ): Promise<Omit<Item, 'id' | 'meta'> | undefined>
  
  itemToFormValues(item: Partial<Item>): Promise<Record<string, any>>
  
  // Save
  saveCurrentItem(showToast?: boolean): Promise<void>
}
```

## Context by Hook

Different hooks receive different combinations of base context and extensions:

### Self-Resizing Contexts

| Hook | Context Features |
|------|------------------|
| `renderFieldExtension` | Base + Self-resizing + Field + ItemForm |
| `renderConfigScreen` | Base + Self-resizing |
| `renderModal` | Base + Self-resizing + Modal params |
| `renderItemFormSidebarPanel` | Base + Self-resizing + ItemForm |
| `renderAssetSource` | Base + Self-resizing + Asset config |
| `renderManualFieldExtensionConfigScreen` | Base + Self-resizing + Extension config |
| `renderUploadSidebarPanel` | Base + Self-resizing + Upload context |
| `renderItemCollectionOutlet` | Base + Self-resizing + Outlet config |
| `renderItemFormOutlet` | Base + Self-resizing + ItemForm + Outlet config |

### Fixed-Size Contexts

| Hook | Context Features |
|------|------------------|
| `onBoot` | Base + Imposed size |
| `renderPage` | Base + Imposed size + Page location |
| `renderItemFormSidebar` | Base + Imposed size + ItemForm |
| `renderUploadSidebar` | Base + Imposed size + Upload context |

### Direct Call Contexts

| Hook Type | Context Features |
|-----------|------------------|
| Action hooks (`executeXXXAction`) | Base + Action parameters |
| Lifecycle hooks (`onBeforeXXX`) | Base + Operation data |
| Configuration hooks | Base + Config requirements |

## Usage Examples

### Accessing User and Project Info

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // User info
    console.log('Current user:', ctx.currentUser);
    console.log('User role:', ctx.currentRole.name);
    console.log('UI locale:', ctx.ui.locale);
    
    // Project info
    console.log('Project:', ctx.site.attributes.name);
    console.log('Environment:', ctx.environment);
    console.log('Is primary env:', ctx.isEnvironmentPrimary);
    console.log('Theme colors:', ctx.theme);
  }
});
```

### Working with Field Values

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Get current value
    const currentValue = ctx.formValues[ctx.fieldPath];
    
    // Update value
    await ctx.setFieldValue(ctx.fieldPath, 'New value');
    
    // Access related fields from entity repos
    const titleField = Object.values(ctx.fields).find(f => f.attributes.api_key === 'title');
    const titleValue = ctx.formValues[titleField.attributes.api_key];
    
    // Disable field conditionally
    if (titleValue === '') {
      await ctx.disableField(ctx.fieldPath, true);
    }
  }
});
```

### Managing Plugin Height

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Auto-resize based on content
    ctx.startAutoResizer();
    
    // Or set specific height
    ctx.setHeight(300);
    
    // Update height after content change
    const addContent = () => {
      // Add elements...
      ctx.updateHeight(); // Auto-detect new height
    };
  }
});
```

### Using Dialogs

```typescript
connect({
  async renderFieldExtension(fieldExtensionId, ctx) {
    // Select a record
    const item = await ctx.selectItem('blog_post', {
      filters: {
        fields: {
          status: { eq: 'published' }
        }
      }
    });
    
    if (item) {
      ctx.notice(`Selected: ${item.title}`);
    }
    
    // Confirm action
    const result = await ctx.openConfirm({
      title: 'Delete this item?',
      content: 'This action cannot be undone.',
      choices: [
        { label: 'Delete', value: true, intent: 'negative' },
      ],
      cancel: { label: 'Cancel', value: false }
    });
    
    if (result) {
      // Perform deletion
    }
  }
});
```

### Accessing Models and Fields

```typescript
connect({
  renderConfigScreen(ctx) {
    // List all models
    Object.values(ctx.itemTypes).forEach(model => {
      console.log(`Model: ${model.attributes.name} (${model.attributes.api_key})`);
      
      // Get fields for this model
      const modelFields = Object.values(ctx.fields).filter(
        f => f.relationships.item_type.data.id === model.id
      );
      
      modelFields.forEach(field => {
        console.log(`  - ${field.attributes.label} (${field.attributes.api_key}): ${field.attributes.field_type}`);
      });
    });
  }
});
```

## TypeScript Support

The SDK provides full TypeScript definitions for all context types:

```typescript
import { 
  RenderFieldExtensionCtx,
  RenderConfigScreenCtx,
  RenderModalCtx,
  OnBootCtx 
} from 'datocms-plugin-sdk';

// Type-safe hook implementations
connect({
  renderFieldExtension(id: string, ctx: RenderFieldExtensionCtx) {
    // ctx is fully typed with all available properties/methods
  },
  
  onBoot(ctx: OnBootCtx) {
    // ctx only has base properties (no field/form extensions)
  }
});
```

## Best Practices

1. **Check Optional Properties**: Some properties like `currentUser` may be null
2. **Handle Async Methods**: Most context methods return Promises
3. **Use Type Guards**: Check field types before accessing type-specific properties
4. **Cache Data**: Pre-loaded collections are available immediately, use them instead of loading
5. **Error Handling**: Wrap dialog and async operations in try-catch blocks
6. **Height Management**: Use `startAutoResizer()` for dynamic content, `setHeight()` for fixed layouts

## Related Documentation

- [connect Function](./connect.md) - How to initialize plugins with hooks
- [Hooks Overview](../02-hooks/README.md) - All available hooks and their contexts
- [Shared Types](./shared-types.md) - Type definitions used throughout the SDK