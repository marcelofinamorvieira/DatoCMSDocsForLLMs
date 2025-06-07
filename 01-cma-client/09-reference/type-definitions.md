# Type Definitions Guide

## Overview

DatoCMS provides comprehensive TypeScript support across all SDKs. This guide explains the type systems and how to use them effectively.

## Schema Types vs Simple Schema Types

The CMA client provides two type systems:

### SchemaTypes
Full API response types including metadata and relationships:

```typescript
import { SchemaTypes } from '@datocms/cma-client';

// Full item type with all metadata
const item: SchemaTypes.Item = {
  id: '12345',
  type: 'item',
  attributes: {
    title: 'My Article',
    created_at: '2024-01-01T00:00:00Z',
    updated_at: '2024-01-01T00:00:00Z',
    // ... all fields
  },
  relationships: {
    item_type: { data: { type: 'item_type', id: 'article' } },
    creator: { data: { type: 'user', id: '456' } }
  },
  meta: {
    created_at: '2024-01-01T00:00:00Z',
    updated_at: '2024-01-01T00:00:00Z',
    published_at: '2024-01-01T00:00:00Z',
    publication_scheduled_at: null,
    unpublishing_scheduled_at: null,
    first_published_at: '2024-01-01T00:00:00Z',
    is_current_version_valid: true,
    is_published_version_valid: true,
    current_version: '789',
    stage: 'published'
  }
};
```

### SimpleSchemaTypes
Simplified types for easier manipulation:

```typescript
import { SimpleSchemaTypes } from '@datocms/cma-client';

// Simplified item for creation
const newItem: SimpleSchemaTypes.ItemCreateSchema = {
  item_type: { type: 'item_type', id: 'article' },
  title: 'My Article',
  slug: 'my-article',
  content: 'Article content...'
};

// Simplified item response
const item: SimpleSchemaTypes.Item = {
  id: '12345',
  title: 'My Article',
  slug: 'my-article',
  content: 'Article content...',
  // Metadata flattened
  created_at: '2024-01-01T00:00:00Z',
  updated_at: '2024-01-01T00:00:00Z'
};
```

## Field Value Types

### Basic Field Types

```typescript
// String fields
type StringField = string;

// Text fields  
type TextField = string;

// Boolean fields
type BooleanField = boolean;

// Integer fields
type IntegerField = number;

// Float fields
type FloatField = number;

// Date fields (ISO 8601 string)
type DateField = string; // "2024-01-01"

// DateTime fields (ISO 8601 string)
type DateTimeField = string; // "2024-01-01T12:00:00Z"

// Color fields
type ColorField = {
  red: number;   // 0-255
  green: number; // 0-255
  blue: number;  // 0-255
  alpha: number; // 0-255
};

// JSON fields
type JsonField = any; // Can be any valid JSON
```

### Complex Field Types

```typescript
// Link field (single reference)
type LinkField = {
  id: string;
  type: 'item';
} | null;

// Links field (multiple references)
type LinksField = Array<{
  id: string;
  type: 'item';
}>;

// File field (single upload)
type FileField = {
  upload_id: string;
  title?: string;
  alt?: string;
  focal_point?: {
    x: number; // 0-1
    y: number; // 0-1
  };
} | null;

// Gallery field (multiple uploads)
type GalleryField = Array<{
  upload_id: string;
  title?: string;
  alt?: string;
}>;

// SEO field
type SeoField = {
  title?: string;
  description?: string;
  image?: {
    upload_id: string;
  };
  twitter_card?: 'summary' | 'summary_large_image';
};

// Geolocation field
type LatLonField = {
  latitude: number;
  longitude: number;
};

// Structured text field
type StructuredTextField = {
  schema: 'dast';
  document: DastDocument;
};
```

## Structured Text (DAST) Types

```typescript
import { Document, Node, Block, Inline } from 'datocms-structured-text-utils';

// Document structure
interface DastDocument {
  schema: 'dast';
  document: {
    type: 'root';
    children: Array<Block | Inline>;
  };
}

// Block nodes
type Block = 
  | { type: 'paragraph'; children: Inline[] }
  | { type: 'heading'; level: 1 | 2 | 3 | 4 | 5 | 6; children: Inline[] }
  | { type: 'list'; style: 'bulleted' | 'numbered'; children: ListItem[] }
  | { type: 'code'; language?: string; code: string }
  | { type: 'blockquote'; children: Block[] }
  | { type: 'block'; item: string } // DatoCMS record
  | { type: 'image'; upload: string }; // DatoCMS upload

// Inline nodes
type Inline =
  | { type: 'span'; value: string; marks?: Mark[] }
  | { type: 'link'; url: string; children: Inline[] }
  | { type: 'itemLink'; item: string; children: Inline[] }
  | { type: 'inlineItem'; item: string };

// Text marks
type Mark =
  | { type: 'emphasis' }
  | { type: 'strong' }
  | { type: 'underline' }
  | { type: 'strikethrough' }
  | { type: 'code' }
  | { type: 'highlight' };
```

## Localized Fields

```typescript
// Single locale
type LocalizedString = string;

// Multiple locales
type LocalizedStringMulti = {
  en: string;
  it: string;
  // ... other locales
};

// Usage in items
interface LocalizedItem {
  title: LocalizedStringMulti;
  content: LocalizedStringMulti;
  // Non-localized fields
  created_at: string;
  id: string;
}
```

## Plugin SDK Types

```typescript
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

// Context types for hooks
interface RenderFieldExtensionCtx {
  fieldPath: string;
  field: Field;
  item: Item | null;
  itemType: ItemType;
  currentUser: User;
  disabled: boolean;
  locale: string;
  // ... many more properties
}

// Hook function types
type RenderFieldExtensionHook = (
  fieldExtensionId: string,
  ctx: RenderFieldExtensionCtx
) => void;
```

## Utility Types

```typescript
// API response wrapper
interface CollectionResponse<T> {
  data: T[];
  meta: {
    total_count: number;
  };
}

// Pagination parameters
interface PageParams {
  offset?: number;
  limit?: number;
}

// Filter parameters
interface FilterParams<T> {
  ids?: string[];
  query?: string;
  filter?: Partial<T>;
}

// Reference type
interface Reference {
  id: string;
  type: string;
}
```

## Type Guards

```typescript
// Check if value is an ApiError
import { ApiError } from '@datocms/cma-client';

function isApiError(error: unknown): error is ApiError {
  return error instanceof ApiError;
}

// Check if field is localized
function isLocalizedField(value: any): value is Record<string, any> {
  return typeof value === 'object' && 
         value !== null && 
         !Array.isArray(value) &&
         Object.keys(value).every(key => /^[a-z]{2}(-[A-Z]{2})?$/.test(key));
}
```

## Generic Type Patterns

```typescript
// Create type-safe resource methods
type ResourceMethods<T> = {
  find(id: string): Promise<T>;
  list(params?: FilterParams<T>): Promise<T[]>;
  create(data: Omit<T, 'id' | 'created_at' | 'updated_at'>): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  destroy(id: string): Promise<void>;
};

// Type-safe field updates
type FieldUpdate<T> = {
  [K in keyof T]?: T[K] extends LocalizedStringMulti
    ? LocalizedStringMulti | { [locale: string]: string }
    : T[K];
};
```

## Using Types with Clients

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_TOKEN' });

// Type inference works automatically
const items = await client.items.list(); // items: SimpleSchemaTypes.Item[]

// Explicit typing for clarity
const newItem: SimpleSchemaTypes.ItemCreateSchema = {
  item_type: { type: 'item_type', id: 'article' },
  title: 'My Article'
};

const created = await client.items.create(newItem); // created: SimpleSchemaTypes.Item
```

## Related Documentation

- [CMA Client](../01-cma-client/README.md)
- [Plugin SDK](../03-plugin-sdk/README.md)
- [Field Types Matrix](./field-type-matrix.md)