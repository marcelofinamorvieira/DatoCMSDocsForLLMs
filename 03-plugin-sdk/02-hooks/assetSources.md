# assetSources Hook

## Purpose

The `assetSources` hook allows plugins to register additional media sources that appear alongside the default DatoCMS media library when users upload assets. This enables integration with external asset management systems, stock photo libraries, or custom asset repositories.

## Signature

```typescript
assetSources(ctx: AssetSourcesCtx): AssetSource[] | undefined
```

## Context Properties

The `assetSources` hook receives the standard base context object (`Ctx`) with no additional properties or methods.

## Return Value

The hook should return an array of `AssetSource` objects, each with:

```typescript
type AssetSource = {
  id: string;                // Unique identifier for the asset source
  name: string;              // Display name shown to users
  icon: Icon;               // FontAwesome icon name or custom SVG
  modal?: {                 // Optional modal configuration
    width?: 's' | 'm' | 'l' | 'xl' | number;
    initialHeight?: number;
  };
}
```

## Use Cases

### 1. Stock Photo Library Integration

Integrate with services like Unsplash, Pexels, or Getty Images:

```typescript
assetSources(ctx: AssetSourcesCtx): AssetSource[] {
  return [
    {
      id: 'unsplash',
      name: 'Unsplash Photos',
      icon: 'camera',
      modal: {
        width: 'xl',
        initialHeight: 600
      }
    },
    {
      id: 'pexels',
      name: 'Pexels Stock Photos',
      icon: 'images',
      modal: {
        width: 'l',
        initialHeight: 500
      }
    }
  ];
}
```

### 2. Company Asset Repository

Connect to internal DAM systems or branded asset libraries:

```typescript
assetSources(ctx: AssetSourcesCtx): AssetSource[] {
  // Only show company assets to authenticated users
  if (!ctx.currentUserAccessToken) {
    return [];
  }

  const sources: AssetSource[] = [
    {
      id: 'brand-assets',
      name: 'Brand Assets',
      icon: 'bookmark',
      modal: {
        width: 'xl',
        initialHeight: 700
      }
    }
  ];

  // Add department-specific sources based on user role
  if (ctx.currentRole.attributes.name.includes('Marketing')) {
    sources.push({
      id: 'marketing-assets',
      name: 'Marketing Materials',
      icon: 'bullhorn',
      modal: {
        width: 'l',
        initialHeight: 600
      }
    });
  }

  if (ctx.currentRole.attributes.name.includes('Product')) {
    sources.push({
      id: 'product-images',
      name: 'Product Images',
      icon: 'shopping-bag',
      modal: {
        width: 'xl',
        initialHeight: 650
      }
    });
  }

  return sources;
}
```

### 3. Cloud Storage Integration

Connect to cloud storage providers:

```typescript
assetSources(ctx: AssetSourcesCtx): AssetSource[] {
  const sources: AssetSource[] = [];

  // Check plugin configuration for enabled services
  const { enabledServices } = ctx.plugin.attributes.parameters;

  if (enabledServices?.dropbox) {
    sources.push({
      id: 'dropbox',
      name: 'Dropbox',
      icon: {
        type: 'svg',
        viewBox: '0 0 24 24',
        content: '<path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"/>'
      },
      modal: {
        width: 'l',
        initialHeight: 500
      }
    });
  }

  if (enabledServices?.googleDrive) {
    sources.push({
      id: 'google-drive',
      name: 'Google Drive',
      icon: 'cloud',
      modal: {
        width: 'l',
        initialHeight: 500
      }
    });
  }

  if (enabledServices?.s3) {
    sources.push({
      id: 'aws-s3',
      name: 'AWS S3 Bucket',
      icon: 'database',
      modal: {
        width: 'm',
        initialHeight: 400
      }
    });
  }

  return sources;
}
```

### 4. AI-Generated Images

Integrate AI image generation services:

```typescript
assetSources(ctx: AssetSourcesCtx): AssetSource[] {
  const aiServices: AssetSource[] = [];

  // Only show if API keys are configured
  if (ctx.plugin.attributes.parameters.openaiApiKey) {
    aiServices.push({
      id: 'dall-e',
      name: 'DALL-E Image Generator',
      icon: 'magic',
      modal: {
        width: 'l',
        initialHeight: 600
      }
    });
  }

  if (ctx.plugin.attributes.parameters.midjourenyApiKey) {
    aiServices.push({
      id: 'midjourney',
      name: 'Midjourney',
      icon: 'palette',
      modal: {
        width: 'xl',
        initialHeight: 700
      }
    });
  }

  if (ctx.plugin.attributes.parameters.stableDiffusionEndpoint) {
    aiServices.push({
      id: 'stable-diffusion',
      name: 'Stable Diffusion',
      icon: 'wand-magic-sparkles',
      modal: {
        width: 'l',
        initialHeight: 650
      }
    });
  }

  return aiServices;
}
```

### 5. Screenshot and Design Tools

Integrate with design and screenshot services:

```typescript
assetSources(ctx: AssetSourcesCtx): AssetSource[] {
  const designTools: AssetSource[] = [
    {
      id: 'figma',
      name: 'Figma Designs',
      icon: 'pen-ruler',
      modal: {
        width: 'xl',
        initialHeight: 700
      }
    },
    {
      id: 'screenshot-api',
      name: 'Website Screenshots',
      icon: 'desktop',
      modal: {
        width: 'm',
        initialHeight: 400
      }
    },
    {
      id: 'canva',
      name: 'Canva Designs',
      icon: 'paint-brush',
      modal: {
        width: 'xl',
        initialHeight: 700
      }
    }
  ];

  // Filter based on user permissions or configuration
  return designTools.filter(tool => {
    const toolConfig = ctx.plugin.attributes.parameters[tool.id];
    return toolConfig?.enabled && toolConfig?.apiKey;
  });
}
```

### 6. Dynamic Asset Sources

Load asset sources based on project configuration:

```typescript
assetSources(ctx: AssetSourcesCtx): AssetSource[] | undefined {
  // Load configuration from a specific model
  const configModel = Object.values(ctx.itemTypes).find(
    model => model.attributes.api_key === 'asset_source_config'
  );

  if (!configModel) {
    return undefined;
  }

  // In a real implementation, you would fetch the actual records
  // This is a simplified example
  const dynamicSources: AssetSource[] = [
    {
      id: 'partner-assets',
      name: `${ctx.site.attributes.name} Partner Assets`,
      icon: 'handshake',
      modal: {
        width: 'l',
        initialHeight: 500
      }
    }
  ];

  // Add environment-specific sources
  if (ctx.environment !== 'main') {
    dynamicSources.push({
      id: 'test-assets',
      name: 'Test Assets (Non-Production)',
      icon: 'flask',
      modal: {
        width: 'm',
        initialHeight: 400
      }
    });
  }

  return dynamicSources;
}
```

## Working with Modal Sizes

The modal configuration allows you to control the display of your asset source:

```typescript
// Predefined sizes
const modalSizes = {
  small: { width: 's', initialHeight: 300 },      // ~400px wide
  medium: { width: 'm', initialHeight: 400 },     // ~600px wide
  large: { width: 'l', initialHeight: 500 },      // ~800px wide
  extraLarge: { width: 'xl', initialHeight: 600 } // ~1000px wide
};

// Custom size with exact pixels
const customModal = {
  width: 750,        // Exact width in pixels
  initialHeight: 450 // Initial height before auto-resize
};
```

## Icon Options

You can use either FontAwesome icons or custom SVGs:

```typescript
// FontAwesome icon (recommended for consistency)
const faIcon = {
  id: 'my-source',
  name: 'My Source',
  icon: 'folder-open' // Any valid FontAwesome icon name
};

// Custom SVG icon
const svgIcon = {
  id: 'custom-source',
  name: 'Custom Source',
  icon: {
    type: 'svg',
    viewBox: '0 0 100 100',
    content: '<circle cx="50" cy="50" r="40" fill="currentColor"/>'
  }
};
```

## Best Practices

1. **Conditional Display**: Only show asset sources that are properly configured:
   ```typescript
   if (!ctx.plugin.attributes.parameters.apiKey) {
     return [];
   }
   ```

2. **User Permissions**: Consider user roles when displaying sources:
   ```typescript
   if (!ctx.currentRole.attributes.can_edit_schema) {
     return sources.filter(s => s.id !== 'admin-assets');
   }
   ```

3. **Environment Awareness**: Provide different sources for different environments:
   ```typescript
   if (ctx.environment === 'production') {
     return productionSources;
   }
   return testingSources;
   ```

4. **Clear Naming**: Use descriptive names that clearly indicate the source:
   ```typescript
   {
     name: 'Unsplash (Free Stock Photos)',
     // vs
     name: 'Images' // Too generic
   }
   ```

5. **Appropriate Icons**: Choose icons that represent the source type:
   ```typescript
   'camera' for photo libraries
   'video' for video sources
   'cloud' for cloud storage
   'palette' for design tools
   ```

## Related Hooks

- **renderAssetSource**: Implements the UI for selecting assets from your custom source
- **uploadSidebarPanels**: Add panels to the upload sidebar for asset metadata
- **uploadSidebars**: Create custom sidebars for uploaded assets

## Important Notes

- Asset sources appear in the media picker dialog alongside the default DatoCMS media library
- Each asset source must have a unique ID that will be passed to `renderAssetSource`
- The modal size should be chosen based on the complexity of your asset picker interface
- Icons should maintain visual consistency with DatoCMS interface
- Asset sources are available globally across all models and fields that accept media
- The hook is called once when the media picker is opened, not on every render

## Type Definitions

### Hook Signature

```typescript
// From packages/sdk/src/hooks/assetSources.ts
export type AssetSourcesHook = {
  /**
   * Use this function to declare additional sources to be shown when users want
   * to upload new assets
   *
   * @tag assetSources
   */
  assetSources: (ctx: AssetSourcesCtx) => AssetSource[] | undefined;
};
```

### Context Type

```typescript
export type AssetSourcesCtx = Ctx;
```

### AssetSource Type

```typescript
// From packages/sdk/src/hooks/assetSources.ts
export type AssetSource = {
  /**
   * ID of the asset source. Will be the first argument for the
   * `renderAssetSource` function
   */
  id: string;
  /** Name of the asset that will be shown to the user */
  name: string;
  /**
   * Icon to be shown alongside the name. Can be a FontAwesome icon name (ie.
   * `"address-book"`) or a custom SVG definition. To maintain visual
   * consistency with the rest of the interface, try to use FontAwesome icons
   * whenever possible.
   */
  icon: Icon;
  /**
   * Configuration options for the modal that will be opened to select a media
   * file from this source
   */
  modal?: {
    /** Width of the modal. Can be a number, or one of the predefined sizes */
    width?: 's' | 'm' | 'l' | 'xl' | number;
    /**
     * The initial height to set for the iframe that will render the modal
     * content
     */
    initialHeight?: number;
  };
};
```

### Full Context Properties

The `AssetSourcesCtx` is the base `Ctx` type, which includes:

#### Base Properties
```typescript
// Plugin information
plugin: Plugin;
currentUserAccessToken: string | null;

// Authentication & Permissions  
currentUser: User | SsoUser | null;
currentRole: Role;
account: Account;

// Project Information
site: Site;
environment: string;
environmentId: string;
mainLocale: string;
currentUserMenuLocale: string;
theme: 'dark' | 'light';
ui: Record<string, unknown>;

// Entity Repositories
itemTypes: ItemType[];
fields: Field[];
fieldsets: Fieldset[];
users: User[];
ssoUsers: SsoUser[];
```

#### Base Methods
```typescript
// Data Loading
async loadItemTypes(ids: string[]): Promise<ItemType[]>;
async loadFields(ids: string[]): Promise<Field[]>;
async loadFieldsets(ids: string[]): Promise<Fieldset[]>;
async loadUsers(ids: string[]): Promise<User[]>;
async loadSsoUsers(ids: string[]): Promise<SsoUser[]>;

// Plugin Configuration
async updatePluginParameters(params: Record<string, unknown>): Promise<void>;
async updateFieldAppearance(
  fieldId: string,
  appearance: Record<string, unknown>
): Promise<void>;

// Toast Notifications
notice(message: string): void;
alert(message: string): void;
customToast(toast: Toast): void;

// Item (Record) Dialogs
async selectItem(
  itemTypeId: string,
  options?: {
    multiple?: boolean;
    locale?: string;
    filters?: ItemListFilter;
    initialLocationQuery?: LocationQuery;
  }
): Promise<Item | Item[] | null>;

async createNewItem(
  itemTypeId: string,
  options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
    finalizeCallback?: FinalizeCallback;
  }
): Promise<Item | null>;

async editItem(
  itemId: string,
  options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
    finalizeCallback?: FinalizeCallback;
  }
): Promise<Item | null>;

// Upload (Asset) Dialogs
async selectUpload(options?: {
  multiple?: boolean;
  locale?: string;
  filters?: UploadListFilter;
}): Promise<Upload | Upload[] | null>;

async editUpload(
  uploadId: string,
  options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
    finalizeCallback?: FinalizeCallback;
  }
): Promise<Upload | null>;

async editUploads(
  uploadIds: string[],
  options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
    finalizeCallback?: FinalizeCallback;
  }
): Promise<void>;

// Custom Dialogs
async openModal(
  modalId: string,
  options?: {
    title?: string;
    width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
    height?: 's' | 'm' | 'l' | number;
    initialHeight?: number;
    closeDisabled?: boolean;
    parameters?: Record<string, unknown>;
  }
): Promise<unknown>;

async openConfirm(options: {
  title: string;
  content?: ReactElement | string;
  choices?: Array<{
    label: string;
    value: unknown;
    icon?: Icon;
    intent?: 'positive' | 'negative';
  }>;
  cancel?: {
    label: string;
    value: unknown;
  };
}): Promise<unknown>;

// Navigation
async navigateTo(
  location: 
    | string // URL path
    | { 
        pageId: string;
        params?: Record<string, string>;
        query?: Record<string, string>;
        locale?: string;
      }
): Promise<void>;

// Utility Methods
getFieldIntlTitle(
  field: Field,
  locale?: string,
  returnUntranslatedTitle?: boolean
): string | null;

createClient(options?: { 
  apiToken?: string;
  environment?: string;
}): SiteApiClient;
```

### Type Guards

The SDK provides type guard functions for validation:

```typescript
// From packages/sdk/src/hooks/assetSources.ts
export function isAssetSource(value: unknown): value is AssetSource {
  if (isNullish(value)) return false;
  if (!isRecord(value)) return false;

  const { id, name, icon, modal } = value;

  return (
    isString(id) &&
    isString(name) &&
    isIcon(icon) &&
    (isNullish(modal) ||
      (isRecord(modal) &&
        (isNullish(modal.width) ||
          (isString(modal.width) &&
            ['s', 'm', 'l', 'xl'].includes(modal.width)) ||
          isNumber(modal.width)) &&
        (isNullish(modal.initialHeight) || isNumber(modal.initialHeight))))
  );
}

export function isReturnTypeOfAssetSourcesHook(
  value: unknown,
): value is AssetSource[] | undefined {
  return isNullish(value) || isArray(value, isAssetSource);
}