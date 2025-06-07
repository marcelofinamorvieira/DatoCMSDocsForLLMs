# buildItemPresentationInfo Hook

## Purpose

The `buildItemPresentationInfo` hook allows plugins to customize how records are displayed throughout the DatoCMS interface. This affects record presentation in collection views, relationship fields (Single/Multiple links), and anywhere else records are referenced. You can provide custom titles and preview images that better represent your content than the default display.

## Signature

```typescript
buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): MaybePromise<ItemPresentationInfo | undefined>
```

## Context Properties

The hook receives the standard context object (`BuildItemPresentationInfoCtx` which is an alias for `Ctx`). The context includes all base properties and methods available to plugin hooks.

## Parameters

- `item`: Item - The record to generate presentation info for

## Type Definitions

```typescript
// Hook signature
export type BuildItemPresentationInfoHook = {
  /**
   * Use this function to customize the presentation of a record in records
   * collections and "Single link" or "Multiple links" field
   *
   * @tag presentation
   */
  buildItemPresentationInfo: (
    item: Item,
    ctx: BuildItemPresentationInfoCtx,
  ) => MaybePromise<ItemPresentationInfo | undefined>;
};

// Context type
export type BuildItemPresentationInfoCtx = Ctx;

// Return type
export type ItemPresentationInfo = {
  /** The title to present the record */
  title: string;
  /** An image representative of the record */
  imageUrl?: string;
  /**
   * If different plugins implement the `buildItemPresentationInfo` hook, the
   * one with the lowest `rank` will be used. If you want to specify an explicit
   * value for `rank`, make sure to offer a way for final users to customize it
   * inside the plugin's settings form, otherwise the hardcoded value you choose
   * might clash with the one of another plugin!
   */
  rank?: number;
};

// Item type from @datocms/cma-client
type Item = {
  id: string;
  type: 'item';
  attributes: Record<string, any>; // Field values keyed by field API key
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
    creator: {
      data: {
        id: string;
        type: 'account' | 'user' | 'sso_user';
      };
    };
    // Additional relationships may include:
    parent?: { data: { id: string; type: 'item' } | null };
    children?: { data: Array<{ id: string; type: 'item' }> };
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at: string | null;
    first_published_at: string | null;
    publication_scheduled_at: string | null;
    unpublishing_scheduled_at: string | null;
    status: 'draft' | 'updated' | 'published';
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    current_version: string;
    stage: string | null;
  };
};

// Helper type
export type MaybePromise<T> = T | Promise<T>;

// Base context type (Ctx) - includes all properties and methods
export type Ctx = BaseCtx & BaseProperties & BaseMethods;

// Base properties included in all contexts
type BaseProperties = {
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
};

// Base methods included in all contexts
type BaseMethods = {
  // Data Loading
  loadItemTypes: (ids: string[]) => Promise<ItemType[]>;
  loadFields: (ids: string[]) => Promise<Field[]>;
  loadFieldsets: (ids: string[]) => Promise<Fieldset[]>;
  loadUsers: (ids: string[]) => Promise<User[]>;
  loadSsoUsers: (ids: string[]) => Promise<SsoUser[]>;
  
  // Plugin Configuration
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  updateFieldAppearance: (
    fieldId: string,
    appearance: Record<string, unknown>
  ) => Promise<void>;
  
  // Toast Notifications
  notice: (message: string) => void;
  alert: (message: string) => void;
  customToast: (toast: Toast) => void;
  
  // Item (Record) Dialogs
  selectItem: (
    itemTypeId: string,
    options?: {
      multiple?: boolean;
      locale?: string;
      filters?: ItemListFilter;
      initialLocationQuery?: LocationQuery;
    }
  ) => Promise<Item | Item[] | null>;
  
  createNewItem: (
    itemTypeId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<Item | null>;
  
  editItem: (
    itemId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<Item | null>;
  
  // Upload (Asset) Dialogs
  selectUpload: (options?: {
    multiple?: boolean;
    locale?: string;
    filters?: UploadListFilter;
  }) => Promise<Upload | Upload[] | null>;
  
  editUpload: (
    uploadId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<Upload | null>;
  
  editUploads: (
    uploadIds: string[],
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ) => Promise<void>;
  
  // Custom Dialogs
  openModal: (
    modalId: string,
    options?: {
      title?: string;
      width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
      height?: 's' | 'm' | 'l' | number;
      initialHeight?: number;
      closeDisabled?: boolean;
      parameters?: Record<string, unknown>;
    }
  ) => Promise<unknown>;
  
  openConfirm: (options: {
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
  }) => Promise<unknown>;
  
  // Navigation
  navigateTo: (
    location: 
      | string // URL path
      | { 
          pageId: string;
          params?: Record<string, string>;
          query?: Record<string, string>;
          locale?: string;
        }
  ) => Promise<void>;
  
  // Utility Methods
  getFieldIntlTitle: (
    field: Field,
    locale?: string,
    returnUntranslatedTitle?: boolean
  ) => string | null;
  
  createClient: (options?: { 
    apiToken?: string;
    environment?: string;
  }) => SiteApiClient;
};
```

## Return Value

The hook should return an `ItemPresentationInfo` object or `undefined`:

## Use Cases

### 1. E-commerce Product Display

Show product information with price and availability:

```typescript
async buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): Promise<ItemPresentationInfo | undefined> {
  const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
  
  if (itemType?.attributes.api_key !== 'product') {
    return undefined;
  }

  const price = item.attributes.price as number;
  const currency = item.attributes.currency as string || 'USD';
  const inStock = item.attributes.in_stock as boolean;
  const name = item.attributes.name as string;
  
  // Format title with availability
  const stockIndicator = inStock ? '✓' : '✗';
  const formattedPrice = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency
  }).format(price);
  
  const title = `${stockIndicator} ${name} - ${formattedPrice}`;
  
  // Use the first product image as preview
  const images = item.attributes.images as any[];
  const imageUrl = images?.[0]?.url;
  
  return {
    title,
    imageUrl: imageUrl ? `${imageUrl}?w=50&h=50&fit=crop` : undefined,
    rank: ctx.plugin.attributes.parameters.presentationRank || 10
  };
}
```

### 2. Blog Post with Reading Time

Display blog posts with author and estimated reading time:

```typescript
async buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): Promise<ItemPresentationInfo | undefined> {
  const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
  
  if (itemType?.attributes.api_key !== 'blog_post') {
    return undefined;
  }

  // Calculate reading time
  const content = item.attributes.content as string || '';
  const wordCount = content.split(/\s+/).length;
  const readingTime = Math.ceil(wordCount / 200); // 200 words per minute
  
  // Get author information
  const authorId = item.relationships.author?.data?.id;
  let authorName = 'Unknown Author';
  
  if (authorId) {
    try {
      // Load author data if not in context
      const author = await ctx.loadItem(authorId);
      authorName = author.attributes.name as string;
    } catch (e) {
      // Handle error gracefully
    }
  }
  
  // Format date
  const publishedAt = item.attributes.published_at as string;
  const dateStr = new Date(publishedAt).toLocaleDateString('en-US', {
    month: 'short',
    day: 'numeric',
    year: 'numeric'
  });
  
  const title = `${item.attributes.title} • ${authorName} • ${readingTime} min read • ${dateStr}`;
  
  // Use featured image
  const featuredImage = item.attributes.featured_image as any;
  
  return {
    title,
    imageUrl: featuredImage?.url ? `${featuredImage.url}?w=60&h=60&fit=crop` : undefined
  };
}
```

### 3. Event with Status Indicators

Show events with visual status indicators:

```typescript
async buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): Promise<ItemPresentationInfo | undefined> {
  const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
  
  if (itemType?.attributes.api_key !== 'event') {
    return undefined;
  }

  const name = item.attributes.name as string;
  const startDate = new Date(item.attributes.start_date as string);
  const endDate = new Date(item.attributes.end_date as string);
  const maxAttendees = item.attributes.max_attendees as number;
  const currentAttendees = item.attributes.current_attendees as number;
  
  // Determine event status
  const now = new Date();
  let status = '';
  let statusEmoji = '';
  
  if (now < startDate) {
    status = 'Upcoming';
    statusEmoji = '🔜';
  } else if (now >= startDate && now <= endDate) {
    status = 'Live';
    statusEmoji = '🔴';
  } else {
    status = 'Ended';
    statusEmoji = '✓';
  }
  
  // Calculate availability
  const availability = maxAttendees - currentAttendees;
  const availabilityText = availability > 0 
    ? `${availability} spots left` 
    : 'SOLD OUT';
  
  // Format dates
  const dateFormat: Intl.DateTimeFormatOptions = {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  };
  
  const startStr = startDate.toLocaleDateString('en-US', dateFormat);
  const endStr = endDate.toLocaleDateString('en-US', dateFormat);
  
  const title = `${statusEmoji} ${name} | ${status} | ${startStr} - ${endStr} | ${availabilityText}`;
  
  // Use event banner
  const banner = item.attributes.banner_image as any;
  
  return {
    title,
    imageUrl: banner?.url ? `${banner.url}?w=80&h=45&fit=crop` : undefined
  };
}
```

### 4. Multi-locale Content

Display content with locale indicators:

```typescript
async buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): Promise<ItemPresentationInfo | undefined> {
  const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
  
  // Only handle specific content types
  if (!['article', 'page', 'product'].includes(itemType?.attributes.api_key || '')) {
    return undefined;
  }

  // Get the title in the current UI locale or fallback
  const uiLocale = ctx.ui.locale;
  const titleField = item.attributes.title;
  
  let title = '';
  let localeIndicator = '';
  
  if (typeof titleField === 'object' && titleField !== null) {
    // Localized field
    title = titleField[uiLocale] || titleField.en || Object.values(titleField)[0] || 'Untitled';
    
    // Show which locales have content
    const availableLocales = Object.keys(titleField).filter(locale => titleField[locale]);
    localeIndicator = ` [${availableLocales.join(', ')}]`;
  } else {
    // Non-localized field
    title = titleField as string || 'Untitled';
  }
  
  // Add status indicators
  const isDraft = item.meta.status === 'draft';
  const isUpdated = item.meta.status === 'updated';
  
  let statusPrefix = '';
  if (isDraft) statusPrefix = '📝 ';
  else if (isUpdated) statusPrefix = '✏️ ';
  
  return {
    title: `${statusPrefix}${title}${localeIndicator}`
  };
}
```

### 5. User Profile Display

Show user records with role and status:

```typescript
async buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): Promise<ItemPresentationInfo | undefined> {
  const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
  
  if (itemType?.attributes.api_key !== 'user_profile') {
    return undefined;
  }

  const firstName = item.attributes.first_name as string;
  const lastName = item.attributes.last_name as string;
  const email = item.attributes.email as string;
  const role = item.attributes.role as string;
  const isActive = item.attributes.is_active as boolean;
  const lastLogin = item.attributes.last_login as string;
  
  // Format name
  const fullName = `${firstName} ${lastName}`.trim() || email;
  
  // Add role badge
  const roleBadge = {
    admin: '👑',
    editor: '✏️',
    viewer: '👁️',
    guest: '👤'
  }[role] || '👤';
  
  // Calculate last seen
  let lastSeenText = 'Never logged in';
  if (lastLogin) {
    const lastLoginDate = new Date(lastLogin);
    const daysAgo = Math.floor((Date.now() - lastLoginDate.getTime()) / (1000 * 60 * 60 * 24));
    
    if (daysAgo === 0) lastSeenText = 'Today';
    else if (daysAgo === 1) lastSeenText = 'Yesterday';
    else if (daysAgo < 7) lastSeenText = `${daysAgo} days ago`;
    else if (daysAgo < 30) lastSeenText = `${Math.floor(daysAgo / 7)} weeks ago`;
    else lastSeenText = `${Math.floor(daysAgo / 30)} months ago`;
  }
  
  // Status indicator
  const statusIcon = isActive ? '🟢' : '🔴';
  
  const title = `${statusIcon} ${roleBadge} ${fullName} (${role}) • ${lastSeenText}`;
  
  // Use avatar if available
  const avatar = item.attributes.avatar as any;
  
  return {
    title,
    imageUrl: avatar?.url ? `${avatar.url}?w=40&h=40&fit=crop&crop=faces` : undefined
  };
}
```

### 6. Dynamic Presentation Based on Field Values

Adapt presentation based on record content:

```typescript
async buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): Promise<ItemPresentationInfo | undefined> {
  const itemType = ctx.itemTypes[item.relationships.item_type.data.id];
  
  // Configuration from plugin parameters
  const config = ctx.plugin.attributes.parameters.presentationConfig?.[itemType?.attributes.api_key];
  
  if (!config) {
    return undefined;
  }
  
  // Build title from configured fields
  const titleParts: string[] = [];
  
  for (const fieldPath of config.titleFields || []) {
    const value = getNestedValue(item.attributes, fieldPath);
    if (value) {
      titleParts.push(String(value));
    }
  }
  
  // Add computed values
  if (config.showId) {
    titleParts.unshift(`#${item.id}`);
  }
  
  if (config.showStatus) {
    titleParts.push(`[${item.meta.status}]`);
  }
  
  if (config.showCreatedAt) {
    const created = new Date(item.meta.created_at);
    titleParts.push(created.toLocaleDateString());
  }
  
  // Handle image field
  let imageUrl: string | undefined;
  if (config.imageField) {
    const imageValue = getNestedValue(item.attributes, config.imageField);
    if (imageValue?.url) {
      imageUrl = `${imageValue.url}?${config.imageParams || 'w=50&h=50&fit=crop'}`;
    }
  }
  
  return {
    title: titleParts.join(config.separator || ' • '),
    imageUrl,
    rank: config.rank || 10
  };
}

function getNestedValue(obj: any, path: string): any {
  return path.split('.').reduce((current, key) => current?.[key], obj);
}
```

## Handling Multiple Plugins

When multiple plugins implement `buildItemPresentationInfo` for the same record type, the `rank` property determines which one is used:

```typescript
return {
  title: 'My Custom Title',
  imageUrl: 'https://example.com/image.jpg',
  rank: 5 // Lower rank = higher priority
};
```

Best practice: Make rank configurable in plugin settings:

```typescript
async buildItemPresentationInfo(
  item: Item,
  ctx: BuildItemPresentationInfoCtx
): Promise<ItemPresentationInfo | undefined> {
  // ... generate title and imageUrl ...
  
  return {
    title,
    imageUrl,
    rank: ctx.plugin.attributes.parameters.presentationRank || 10
  };
}
```

## Performance Considerations

1. **Async Operations**: The hook supports async operations, but keep them minimal:
   ```typescript
   // Good: Only load data when necessary
   if (needsAuthorData && !authorInCache) {
     const author = await ctx.loadItem(authorId);
   }
   
   // Avoid: Loading unnecessary data
   const allUsers = await ctx.loadUsers(); // Don't do this
   ```

2. **Image Optimization**: Always use image transformation parameters:
   ```typescript
   imageUrl: imageUrl ? `${imageUrl}?w=50&h=50&fit=crop&auto=format` : undefined
   ```

3. **Caching**: Cache expensive computations when possible:
   ```typescript
   const cacheKey = `presentation_${item.id}_${item.meta.updated_at}`;
   const cached = sessionStorage.getItem(cacheKey);
   if (cached) {
     return JSON.parse(cached);
   }
   ```

## Best Practices

1. **Return undefined for non-handled types**: Don't try to handle all record types
2. **Keep titles concise**: Long titles get truncated in the UI
3. **Use meaningful emojis sparingly**: They can help with quick visual recognition
4. **Include key information**: Status, dates, or other critical data
5. **Handle missing data gracefully**: Always provide fallbacks
6. **Optimize images**: Use small thumbnails with appropriate cropping
7. **Make rank configurable**: Allow users to control priority
8. **Consider localization**: Respect the user's UI locale when formatting

## Related Hooks

- **itemCollectionOutlets**: Add custom UI elements to record lists
- **renderItemFormSidebarPanel**: Add information panels to record editing views
- **itemFormSidebarPanels**: Define sidebar panels for record forms

## Important Notes

- This hook is called frequently, so performance is critical
- The presentation info affects all record displays across DatoCMS
- Return `undefined` to let other plugins or default behavior handle the record
- Image URLs should be absolute and accessible
- The hook runs in the browser context with access to modern JavaScript APIs
- Presentation info is not persisted - it's calculated on demand

## Type Guards

The SDK provides type guard functions for validation:

```typescript
export function isItemPresentationInfo(
  value: unknown,
): value is ItemPresentationInfo {
  return (
    isRecord(value) &&
    isString(value.title) &&
    (isNullish(value.imageUrl) || isString(value.imageUrl)) &&
    (isNullish(value.rank) || isNumber(value.rank))
  );
}

export function isReturnTypeOfBuildItemPresentationInfoHook(
  value: unknown,
): value is ItemPresentationInfo | undefined {
  return isNullish(value) || isItemPresentationInfo(value);
}
```

## Supporting Types

```typescript
// Plugin type
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description: string;
    parameters: Record<string, unknown>;
    // ... other attributes
  };
};

// User types
type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    // ... other attributes
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    // ... other attributes
  };
};

// Role type
type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_content: boolean;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_environments: boolean;
    can_access_audit_log: boolean;
    // ... other permissions
  };
};

// Account type
type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    name: string;
    // ... other attributes
  };
};

// Site type
type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    locales: string[];
    // ... other attributes
  };
};

// ItemType (Model) type
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    // ... other attributes
  };
};

// Field type
type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    api_key: string;
    field_type: FieldType;
    localized: boolean;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      addons: Array<{
        id: string;
        parameters: Record<string, any>;
      }>;
    };
    // ... other attributes
  };
};

// Fieldset type
type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    // ... other attributes
  };
};

// Toast type
type Toast = {
  message: string;
  type?: 'notice' | 'alert' | 'warning';
  dismissAfterTimeout?: number;
};

// Other referenced types
type ItemListFilter = Record<string, any>;
type UploadListFilter = Record<string, any>;
type LocationQuery = Record<string, string>;
type FinalizeCallback = (item: Item | Upload) => void;
type Upload = Record<string, any>;
type SiteApiClient = any; // DatoCMS client instance
type Icon = string | { type: 'svg'; viewBox: string; content: string };
type ReactElement = any; // React element type
type FieldType = 
  | 'boolean'
  | 'color' 
  | 'date'
  | 'date_time'
  | 'file'
  | 'gallery'
  | 'integer'
  | 'json'
  | 'lat_lon'
  | 'links'
  | 'link'
  | 'float'
  | 'rich_text'
  | 'seo'
  | 'slug'
  | 'string'
  | 'text'
  | 'video'
  | 'structured_text';

// Helper functions  
function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}

function isNullish(value: unknown): value is null | undefined {
  return value === null || value === undefined;
}
```