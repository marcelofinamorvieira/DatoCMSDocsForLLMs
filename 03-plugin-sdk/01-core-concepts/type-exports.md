# Type Exports

The DatoCMS Plugin SDK re-exports several key types from `@datocms/cma-client` for convenience. These types represent the core entities you'll work with when building plugins.

## Overview

These types are imported from the Content Management API client and represent the data structures used throughout DatoCMS. They are fully typed with TypeScript and include all properties available through the API.

## Import

```typescript
import type {
  Account,
  Field,
  Item,
  ItemType,
  Plugin,
  Role,
  Site,
  SsoUser,
  Upload,
  User
} from 'datocms-plugin-sdk';
```

## Core Types

### Account

Represents the organization account that owns projects.

```typescript
interface Account {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    name: string;
    company: string | null;
    avatar_url: string | null;
  };
}
```

**Usage Example:**
```typescript
connect({
  renderConfigScreen(ctx) {
    console.log(`Organization: ${ctx.account.attributes.name}`);
    console.log(`Account email: ${ctx.account.attributes.email}`);
  }
});
```

### Field

Represents a field within a model.

```typescript
interface Field {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    api_key: string;
    field_type: FieldType;
    appearance: FieldAppearance;
    validators: Record<string, any>;
    position: number;
    hint: string | null;
    default_value: any;
    localized: boolean;
    required: boolean;
    unique: boolean;
    private: boolean;
    item_type: ItemTypeReference;
  };
}
```

**Common Field Types:**
- `string` - Single-line text
- `text` - Multi-line text
- `boolean` - Boolean (true/false)
- `integer` - Integer number
- `float` - Decimal number
- `date` - Date only
- `datetime` - Date and time
- `color` - Color picker
- `json` - JSON data
- `seo` - SEO metadata
- `slug` - URL slug
- `file` - Single asset
- `gallery` - Multiple assets
- `link` - Single record reference
- `links` - Multiple record references
- `structured_text` - Rich text with blocks
- `modular_content` - Dynamic blocks

**Usage Example:**
```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    const field = ctx.field;
    console.log(`Field: ${field.attributes.label}`);
    console.log(`Type: ${field.attributes.field_type}`);
    console.log(`Required: ${field.attributes.required}`);
    console.log(`Localized: ${field.attributes.localized}`);
  }
});
```

### Item

Represents a content record.

```typescript
interface Item {
  id: string;
  type: 'item';
  attributes: Record<string, any>; // Field values
  meta: {
    created_at: string;
    updated_at: string;
    published_at: string | null;
    publication_scheduled_at: string | null;
    unpublishing_scheduled_at: string | null;
    first_published_at: string | null;
    is_valid: boolean;
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    status: 'draft' | 'updated' | 'published';
    current_version: string;
    stage: string | null;
  };
  relationships: {
    item_type: ItemTypeReference;
    creator: UserReference;
    last_editor: UserReference | null;
  };
}
```

**Usage Example:**
```typescript
connect({
  async renderFieldExtension(fieldExtensionId, ctx) {
    // Select a related item
    const item = await ctx.selectItem('blog_post');
    
    if (item) {
      console.log(`Title: ${item.attributes.title}`);
      console.log(`Status: ${item.meta.status}`);
      console.log(`Created: ${item.meta.created_at}`);
      console.log(`Published: ${item.meta.published_at || 'Not published'}`);
    }
  }
});
```

### ItemType

Represents a model (content type).

```typescript
interface ItemType {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    modular_block: boolean;
    tree: boolean;
    collection_appearance: 'compact' | 'table';
    hint: string | null;
    inverse_relationships_enabled: boolean;
  };
}
```

**Usage Example:**
```typescript
connect({
  renderConfigScreen(ctx) {
    // List all models
    ctx.itemTypes.forEach(itemType => {
      console.log(`Model: ${itemType.attributes.name}`);
      console.log(`API Key: ${itemType.attributes.api_key}`);
      console.log(`Singleton: ${itemType.attributes.singleton}`);
      console.log(`Block: ${itemType.attributes.modular_block}`);
    });
  }
});
```

### Plugin

Represents a plugin installation.

```typescript
interface Plugin {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description: string | null;
    url: string | null;
    homepage: string | null;
    tags: string[];
    version: string | null;
    package_name: string | null;
    package_version: string | null;
    parameters: Record<string, any>;
    permissions: PluginPermission[];
  };
}
```

**Usage Example:**
```typescript
connect({
  async renderConfigScreen(ctx) {
    const plugin = ctx.plugin;
    
    console.log(`Plugin: ${plugin.attributes.name}`);
    console.log(`Version: ${plugin.attributes.version}`);
    console.log(`Parameters:`, plugin.attributes.parameters);
    
    // Update plugin configuration
    await ctx.updatePluginParameters({
      apiKey: 'new-key',
      enabled: true
    });
  }
});
```

### Role

Represents a user role with permissions.

```typescript
interface Role {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_site: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_manage_environments: boolean;
    can_manage_sso: boolean;
    positive_item_type_permissions: ItemTypePermission[];
    negative_item_type_permissions: ItemTypePermission[];
    positive_upload_permissions: UploadPermission[];
    negative_upload_permissions: UploadPermission[];
  };
}
```

**Usage Example:**
```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    const role = ctx.currentRole;
    
    if (!role.attributes.can_edit_schema) {
      ctx.alert('You do not have permission to edit the schema');
      return;
    }
    
    console.log(`Role: ${role.attributes.name}`);
    console.log(`Can edit content: ${role.attributes.can_edit_schema}`);
  }
});
```

### Site

Represents a DatoCMS project.

```typescript
interface Site {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    domain: string | null;
    internal_domain: string;
    global_seo_settings: GlobalSeoSettings | null;
    favicon: Upload | null;
    locales: string[];
    timezone: string;
    no_index: boolean;
    frontend_url: string | null;
    ssg: string | null;
    imgix_host: string | null;
  };
}
```

**Usage Example:**
```typescript
connect({
  renderConfigScreen(ctx) {
    const site = ctx.site;
    
    console.log(`Project: ${site.attributes.name}`);
    console.log(`Domain: ${site.attributes.internal_domain}`);
    console.log(`Locales: ${site.attributes.locales.join(', ')}`);
    console.log(`Timezone: ${site.attributes.timezone}`);
  }
});
```

### SsoUser

Represents a Single Sign-On user.

```typescript
interface SsoUser {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    is_active: boolean;
  };
  relationships: {
    role: RoleReference;
  };
}
```

**Usage Example:**
```typescript
connect({
  renderConfigScreen(ctx) {
    const currentUser = ctx.currentUser;
    
    if (currentUser && currentUser.type === 'sso_user') {
      const ssoUser = currentUser as SsoUser;
      console.log(`SSO User: ${ssoUser.attributes.email}`);
      console.log(`Active: ${ssoUser.attributes.is_active}`);
    }
  }
});
```

### Upload

Represents a media asset.

```typescript
interface Upload {
  id: string;
  type: 'upload';
  attributes: {
    path: string;
    basename: string;
    format: string | null;
    url: string;
    copyright: string | null;
    notes: string | null;
    tags: string[];
    smart_tags: string[];
    alt: string | null;
    title: string | null;
    focal_point: { x: number; y: number } | null;
    upload_id: string;
    is_image: boolean;
    default_field_metadata: {
      [locale: string]: {
        alt: string | null;
        title: string | null;
        focal_point: { x: number; y: number } | null;
      };
    };
    size: number;
    width: number | null;
    height: number | null;
    mime_type: string;
    created_at: string;
    updated_at: string;
  };
}
```

**Usage Example:**
```typescript
connect({
  async renderFieldExtension(fieldExtensionId, ctx) {
    // Select an upload
    const upload = await ctx.selectUpload({
      multiple: false,
      filters: {
        type: 'image'
      }
    });
    
    if (upload) {
      console.log(`File: ${upload.attributes.basename}`);
      console.log(`URL: ${upload.attributes.url}`);
      console.log(`Size: ${upload.attributes.size} bytes`);
      console.log(`Dimensions: ${upload.attributes.width}x${upload.attributes.height}`);
    }
  }
});
```

### User

Represents a standard user account.

```typescript
interface User {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    is_active: boolean;
    avatar_url: string | null;
    created_at: string;
    updated_at: string;
    state: 'pending' | 'active';
  };
  relationships: {
    role: RoleReference;
  };
}
```

**Usage Example:**
```typescript
connect({
  renderItemFormSidebar(sidebarId, ctx) {
    const currentUser = ctx.currentUser;
    
    if (currentUser && currentUser.type === 'user') {
      const user = currentUser as User;
      console.log(`User: ${user.attributes.first_name} ${user.attributes.last_name}`);
      console.log(`Email: ${user.attributes.email}`);
      console.log(`State: ${user.attributes.state}`);
    }
  }
});
```

## Type Guards

When working with union types like `User | SsoUser`, use type guards:

```typescript
function isUser(user: User | SsoUser): user is User {
  return user.type === 'user';
}

function isSsoUser(user: User | SsoUser): user is SsoUser {
  return user.type === 'sso_user';
}

// Usage
if (ctx.currentUser && isUser(ctx.currentUser)) {
  console.log('Regular user:', ctx.currentUser.attributes.email);
} else if (ctx.currentUser && isSsoUser(ctx.currentUser)) {
  console.log('SSO user:', ctx.currentUser.attributes.email);
}
```

## Working with Relationships

Many types include relationship references:

```typescript
// Get the model for a field
const field = ctx.field;
const itemTypeId = field.attributes.item_type.id;
const itemType = ctx.itemTypes.find(it => it.id === itemTypeId);

// Get the creator of an item
const item = ctx.item;
const creatorId = item.relationships.creator.id;
const creator = ctx.users.find(u => u.id === creatorId);
```

## Best Practices

1. **Use Type Imports**: Import types with `import type` for better tree-shaking
2. **Check for Null**: Many properties can be null, always check before accessing
3. **Use Type Guards**: When dealing with union types, create proper type guards
4. **Reference by ID**: Use relationship IDs to look up entities in collections
5. **Handle Missing Data**: Not all properties are always populated

## Related Documentation

- [Context Object](./context-object.md) - How to access these types through context
- [CMA Client Documentation](../../01-cma-client/README.md) - Full API client docs
- [Shared Types](./shared-types.md) - Additional type definitions