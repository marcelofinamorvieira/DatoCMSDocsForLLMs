# Item Resource

## Overview

Items are the individual content entries in your DatoCMS project. Each item belongs to a specific model (item type) and contains the actual content data for your website or application. Items are the core building blocks of your content, representing everything from blog posts and products to pages and any custom content types you define.

## API Client Section

`client.items`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a new item
const item = await client.items.create({
  item_type: { type: 'item_type', id: 'article' },
  title: 'My First Article',
  content: 'This is the article content',
  published_at: new Date().toISOString()
});

// Fetch a single item
const article = await client.items.find('12345');

// Update an item
const updated = await client.items.update('12345', {
  title: 'Updated Title'
});

// Delete an item
await client.items.destroy('12345');
```

## API Reference

### list() - List Items

Retrieve a paginated list of items with powerful filtering and sorting options.

**Signature**: `list(queryParams): Promise<Item[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `filter` | object | Complex filtering options |
| `filter.type` | string | Filter by item type API key |
| `filter.ids` | string | Comma-separated list of IDs |
| `filter.query` | string | Full-text search |
| `filter.fields` | object | Filter by field values |
| `filter.first_published_at` | object | Filter by first publication date |
| `filter.published_at` | object | Filter by publication date |
| `order_by` | array | Sorting criteria (prefix with `-` for descending) |
| `page` | object | Pagination options |
| `page.offset` | number | Starting offset (default: 0) |
| `page.limit` | number | Number of items (default: 30, max: 500) |
| `locale` | string | Filter by locale |
| `nested` | boolean | Include nested blocks (default: true) |
| `version` | string | API version |

**Basic Example**:
```javascript
// Simple list call
const items = await client.items.list({
  filter: { type: 'article' }
});

// Get first page with default size (30 items)
const firstPage = await client.items.list({
  filter: { type: 'article' }
});

// Get specific page with custom size
const page = await client.items.list({
  filter: { type: 'article' },
  page: {
    offset: 0,    // Start from first item
    limit: 50     // Return 50 items
  }
});
```

**Advanced Example (Complex Filtering)**:
```javascript
// Complex filter example with multiple conditions
const publishedArticles = await client.items.list({
  filter: {
    type: 'article',
    fields: {
      published: { eq: true },
      category: { in: ['tech', 'news'] },
      publish_date: { 
        gte: '2024-01-01T00:00:00Z',
        lt: '2024-12-31T23:59:59Z'
      },
      views: { gt: 1000 },
      author: { eq: 'author-123' },
      tags: { anyIn: ['javascript', 'react'] }
    }
  },
  page: { limit: 20, offset: 0 },
  order_by: ['-created_at', 'title']
});

// Full-text search
const searchResults = await client.items.list({
  filter: {
    query: 'javascript react',
    type: 'article'
  }
});

// Filter with OR logic
const items = await client.items.list({
  filter: {
    type: 'article',
    or: [
      { fields: { featured: { eq: true } } },
      { fields: { views: { gt: 1000 } } },
      { fields: { author: { eq: 'author-vip' } } }
    ]
  }
});
```

**Automatic Pagination with Iterators**:
```javascript
// Iterate through all items automatically
for await (const item of client.items.listPagedIterator({
  filter: { type: 'article' }
})) {
  console.log(item.title);
  // Process each item
}

// With custom page size
for await (const item of client.items.listPagedIterator(
  { filter: { type: 'article' } },
  { perPage: 100 }  // Fetch 100 items per request
)) {
  console.log(item.title);
}
```

### create() - Create New Item

Create a new content item with field values matching your model schema.

**Signature**: `create(body): Promise<Item>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `item_type` | object | Reference to the item type (required) |
| `item_type.type` | string | Must be `'item_type'` |
| `item_type.id` | string | The item type ID or API key |
| Field values | various | As defined in your model |
| `meta` | object | Item metadata (optional) |
| `meta.status` | string | Publication status |
| `meta.created_at` | string | Override creation date |
| `meta.updated_at` | string | Override update date |
| `meta.published_at` | string | Override publication date |
| `meta.first_published_at` | string | Override first publication date |
| `meta.publication_scheduled_at` | string | Schedule publication |
| `meta.unpublishing_scheduled_at` | string | Schedule unpublishing |
| `meta.creator` | object | Override creator |

**Basic Example**:
```javascript
const item = await client.items.create({
  item_type: 'article', // Model API key or ID
  title: 'My Article',
  content: 'Article content here',
  published: true
});

console.log('Created item:', item.id);
```

**Comprehensive Example with All Field Types**:
```javascript
const article = await client.items.create({
  item_type: 'article',
  
  // Text fields
  title: 'Complete Article Example',
  slug: 'complete-article-example',
  excerpt: 'This article demonstrates all field types',
  content: 'Full article content...',
  
  // Boolean fields
  featured: true,
  published: false,
  
  // Number fields
  read_time: 5,
  priority: 1.5,
  
  // Date fields
  publish_date: '2024-01-15',
  event_datetime: '2024-01-15T14:30:00Z',
  
  // JSON field
  metadata: {
    views: 0,
    source: 'api',
    tags: ['tutorial', 'example']
  },
  
  // SEO field
  seo: {
    title: 'SEO Title',
    description: 'Meta description',
    image: { upload_id: 'upload-123' }
  },
  
  // Color field
  brand_color: '#FF5733', // or { red: 255, green: 87, blue: 51, alpha: 1 }
  
  // Location field
  location: {
    latitude: 40.7128,
    longitude: -74.0060
  },
  
  // Single asset field
  featured_image: {
    upload_id: 'upload-456',
    alt: 'Featured image',
    title: 'Image title'
  },
  
  // Gallery field (multiple assets)
  gallery: [
    { upload_id: 'upload-1', alt: 'Image 1' },
    { upload_id: 'upload-2', alt: 'Image 2' }
  ]
});
```

**Creating with Relationships**:
```javascript
// Single reference (belongs_to)
const article = await client.items.create({
  item_type: 'article',
  title: 'Technical Guide',
  author: '67890', // Author item ID
  category: 'category-456',
  content: 'Guide content...'
});

// Multiple references (has_many)
const article = await client.items.create({
  item_type: 'article',
  title: 'Multi-category Article',
  categories: ['cat-123', 'cat-456', 'cat-789'], // Array of category IDs
  tags: ['tag-001', 'tag-002'] // Another multi-reference field
});

// With modular content (inline blocks)
const page = await client.items.create({
  item_type: 'landing_page',
  title: 'Product Launch',
  sections: [
    {
      // Inline hero section
      item_type: 'hero_section',
      headline: 'Introducing Our New Product',
      subtitle: 'Revolutionary features',
      cta_text: 'Learn More',
      cta_link: '/products/new'
    },
    {
      // Reference to existing block
      type: 'item',
      id: 'reusable-block-456'
    }
  ]
});
```

**Creating with Structured Text**:
```javascript
const item = await client.items.create({
  item_type: 'article',
  title: 'Article with Rich Content',
  content: {
    schema: 'dast',
    document: {
      type: 'root',
      children: [
        {
          type: 'heading',
          level: 1,
          children: [{ type: 'span', value: 'Main Heading' }]
        },
        {
          type: 'paragraph',
          children: [
            { type: 'span', value: 'Check out our ' },
            {
              type: 'itemLink',
              item: 'product-123', // Link to product
              children: [{ type: 'span', value: 'latest product' }]
            },
            { type: 'span', value: ' for more details.' }
          ]
        },
        {
          type: 'block',
          item: {
            type: 'item',
            attributes: {
              text: 'Call to action text',
              url: 'https://example.com'
            },
            relationships: {
              item_type: { data: { type: 'item_type', id: 'cta_block' } }
            }
          }
        }
      ]
    }
  }
});
```

**Batch Creation**:
```javascript
// Create multiple items efficiently
const articles = [
  { title: 'Article 1', content: 'Content 1', item_type: 'article' },
  { title: 'Article 2', content: 'Content 2', item_type: 'article' },
  { title: 'Article 3', content: 'Content 3', item_type: 'article' }
];

// Simple concurrent creation
const createdItems = await Promise.all(
  articles.map(data => client.items.create(data))
);

// Controlled batch processing
async function createInBatches(items, batchSize = 20) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    
    // Create batch concurrently
    const batchResults = await Promise.all(
      batch.map(item => client.items.create(item))
    );
    
    results.push(...batchResults);
    
    // Progress update
    console.log(`Created ${results.length}/${items.length} items`);
    
    // Delay between batches to respect rate limits
    if (i + batchSize < items.length) {
      await new Promise(resolve => setTimeout(resolve, 200));
    }
  }
  
  return results;
}
```

### find() - Get Single Item by ID

Retrieve a single item by its ID.

**Signature**: `find(itemId, queryParams?): Promise<Item>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID (required) |
| `locale` | string | Specific locale to retrieve |
| `version` | string | Version to retrieve ('current' or 'published') |
| `nested` | boolean | Include nested blocks |

**Basic Example**:
```javascript
// Fetch a blog post by ID
const blogPost = await client.items.find('12345');

console.log(blogPost.attributes.title);
```

**Advanced Examples**:
```javascript
// Get the current draft version (even if published)
const draftPost = await client.items.find('12345', {
  version: 'current'
});

// Get the published version explicitly
const publishedPost = await client.items.find('12345', {
  version: 'published'
});

// Fetch from a sandbox environment
const sandboxPost = await client.items.find('12345', {
  'environment': 'sandbox'
});

// Include nested resources
const postWithType = await client.items.find('12345', {
  nested: true
});
```

### update() - Update Existing Item

Update an existing item's field values.

**Signature**: `update(itemId, body): Promise<Item>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID (required) |
| Field values | various | Fields to update |
| `meta` | object | Update metadata (optional) |

**Basic Example**:
```javascript
const updatedItem = await client.items.update('12345678', {
  title: 'Updated Title',
  content: 'Updated content',
  meta: {
    updated_at: new Date().toISOString()
  }
});
```

**Partial Updates**:
```javascript
// Update only specific fields
const item = await client.items.update('item-id-123', {
  // Only update these fields
  status: 'reviewed',
  last_reviewed_at: new Date().toISOString()
  // Other fields remain unchanged
});

// Conditional updates
const currentItem = await client.items.find('item-id');

if (currentItem.title !== newTitle) {
  await client.items.update('item-id', {
    title: newTitle,
    last_title_change: new Date().toISOString()
  });
}
```

**Updating Different Field Types**:
```javascript
await client.items.update('item-id', {
  // Text fields
  title: 'New Title',
  description: 'Updated description',
  excerpt: null, // Clear field value
  
  // Number fields
  price: 29.99,
  quantity: 100,
  discount_percentage: 15.5,
  
  // Boolean fields
  published: true,
  featured: false,
  archived: false,
  
  // Date/Time fields
  publish_date: '2024-12-25',
  event_datetime: new Date('2024-12-25T18:00:00Z').toISOString(),
  last_modified: new Date().toISOString(),
  
  // JSON fields
  metadata: {
    version: 2,
    tags: ['updated', 'revised'],
    custom: { foo: 'bar' }
  },
  
  // Single reference
  author: 'new-author-id',
  category: null, // Remove reference
  
  // Multiple references
  tags: ['tag-1', 'tag-2', 'tag-3'], // Replace all
  related_articles: [], // Clear all references
  
  // Media fields
  featured_image: {
    upload_id: 'new-upload-id',
    alt: 'Updated alt text',
    title: 'New image title'
  },
  
  // Gallery
  gallery: [
    { upload_id: 'upload-1', alt: 'Image 1' },
    { upload_id: 'upload-2', alt: 'Image 2' },
    { upload_id: 'upload-3', alt: 'Image 3' }
  ]
});
```

**Bulk Updates**:
```javascript
// Update multiple items with same values
const itemIds = ['12345', '12346', '12347'];

const updatedItems = await Promise.all(
  itemIds.map(id => 
    client.items.update(id, {
      published: true,
      featured: false,
      updated_by: 'bulk-update'
    })
  )
);

// Controlled batch updates
async function bulkUpdate(updates, batchSize = 30) {
  const results = [];
  
  for (let i = 0; i < updates.length; i += batchSize) {
    const batch = updates.slice(i, i + batchSize);
    
    // Process batch
    const batchResults = await Promise.all(
      batch.map(({ id, data }) => 
        client.items.update(id, data)
      )
    );
    
    results.push(...batchResults);
    console.log(`Updated ${results.length}/${updates.length} items`);
    
    // Delay between batches
    if (i + batchSize < updates.length) {
      await new Promise(resolve => setTimeout(resolve, 150));
    }
  }
  
  return results;
}
```

### destroy() - Delete Item

Delete an item permanently. This operation is asynchronous and returns a job.

**Signature**: `destroy(itemId): Promise<Job>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID (required) |

**Basic Example**:
```javascript
const job = await client.items.destroy('12345678');

// Wait for completion
const result = await client.jobResults.wait(job.id);
```

**With Error Handling**:
```javascript
try {
  await client.items.destroy('item-id-123');
  console.log('Item deleted');
} catch (error) {
  if (error.statusCode === 404) {
    console.error('Item already deleted or not found');
  } else if (error.statusCode === 403) {
    console.error('No permission to delete');
  } else if (error.statusCode === 422) {
    console.error('Cannot delete - item has dependencies');
  } else {
    console.error('Delete failed:', error.message);
  }
}
```

**Check Before Delete**:
```javascript
async function safeDelete(itemId) {
  try {
    // Check if item exists
    const item = await client.items.find(itemId);
    
    // Confirm it's safe to delete
    if (item.meta.status === 'published') {
      console.warn('Warning: Deleting published item');
    }
    
    // Perform delete
    await client.items.destroy(itemId);
    return true;
  } catch (error) {
    if (error.statusCode === 404) {
      console.log('Item already deleted');
      return true;
    }
    throw error;
  }
}
```

### duplicate() - Duplicate Item

Create a copy of an existing item. This operation is asynchronous.

**Signature**: `duplicate(itemId): Promise<Job>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID to duplicate (required) |

**Example**:
```javascript
const job = await client.items.duplicate('12345678');

// Get the duplicated item ID from the job result
const result = await client.jobResults.wait(job.id);
const duplicatedItemId = result.payload.data.id;
```

### publish() - Publish Item

Publish an item, making it available in the published version of your content.

**Signature**: `publish(itemId, body?): Promise<Item>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID (required) |
| `content_in_locales` | object | Selective publishing (optional) |
| `recursive` | boolean | Also publish nested blocks (optional) |

**Examples**:
```javascript
// Publish entire item
await client.items.publish('12345678');

// Selective publishing (only specific fields/locales)
await client.items.publish('12345678', {
  content_in_locales: {
    title: ['en', 'it'],
    content: ['en']
  },
  recursive: true
});
```

### unpublish() - Unpublish Item

Unpublish an item, removing it from the published version.

**Signature**: `unpublish(itemId, body?): Promise<Item>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID (required) |
| `content_in_locales` | object | Selective unpublishing (optional) |
| `recursive` | boolean | Also unpublish nested blocks (optional) |

**Examples**:
```javascript
// Unpublish entire item
await client.items.unpublish('12345678');

// Selective unpublishing
await client.items.unpublish('12345678', {
  content_in_locales: {
    title: ['it'], // Only unpublish Italian title
  }
});
```

### references() - Get References

Find all items that reference a specific item through link/links fields.

**Signature**: `references(itemId, queryParams?): Promise<Item[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID (required) |
| Query parameters | object | Same as `list()` method |

**Example**:
```javascript
const references = await client.items.references('12345678', {
  filter: {
    type: 'blog_post'
  },
  page: { limit: 100 }
});
```

### rawList() - Get Items with Metadata

Returns the raw JSON:API response including pagination metadata.

**Signature**: `rawList(queryParams): Promise<RawListResponse>`

**Parameters**: Same as `list()` method

**Returns**:
```typescript
{
  data: Item[],
  meta: {
    total_count: number,
    page_count: number,
    per_page: number,
    page: number
  }
}
```

**Example**:
```javascript
const response = await client.items.rawList({
  filter: { type: 'article' },
  page: { limit: 20 }
});

console.log(`Total items: ${response.meta.total_count}`);
console.log(`Total pages: ${response.meta.page_count}`);
```

### listPagedIterator() - Automatic Pagination Iterator

Iterates through all items automatically handling pagination.

**Signature**: `listPagedIterator(queryParams, iteratorOptions?): AsyncIterator<Item>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `queryParams` | object | Same as `list()` method |
| `iteratorOptions.perPage` | number | Items per page (default: 100) |

**Example**:
```javascript
// Iterate through all items automatically
for await (const item of client.items.listPagedIterator({
  filter: { type: 'article' }
})) {
  console.log(item.title);
}

// With custom page size
for await (const item of client.items.listPagedIterator(
  { filter: { type: 'article' } },
  { perPage: 200 }
)) {
  console.log(item.title);
}
```

### validate() - Validate Item

Validates an item without saving it.

**Signature**: `validate(itemId): Promise<ValidationResult>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID (required) |

**Returns**:
```typescript
{
  is_valid: boolean,
  errors: ValidationError[]
}
```

**Example**:
```javascript
const validation = await client.items.validate('item-id-123');

if (validation.is_valid) {
  console.log('Item is valid');
} else {
  console.log('Validation errors:', validation.errors);
}

// Validate before update
const validation = await client.items.validate('item-id');
if (validation.is_valid) {
  await client.items.update('item-id', { status: 'published' });
}
```

### Bulk Operations

Perform operations on multiple items at once. All bulk operations are asynchronous and return a job.

#### Bulk Publish (`client.items.bulkPublish`)

**Example**:
```javascript
const job = await client.items.bulkPublish({
  items: [
    { type: 'item', id: '123' },
    { type: 'item', id: '456' }
  ],
  content_in_locales: {
    title: ['en', 'it']
  }
});
```

#### Bulk Unpublish (`client.items.bulkUnpublish`)

**Example**:
```javascript
const job = await client.items.bulkUnpublish({
  items: [
    { type: 'item', id: '123' },
    { type: 'item', id: '456' }
  ]
});
```

#### Bulk Delete (`client.items.bulkDestroy`)

**Example**:
```javascript
const job = await client.items.bulkDestroy({
  items: [
    { type: 'item', id: '123' },
    { type: 'item', id: '456' }
  ]
});
```

#### Bulk Move to Stage (`client.items.bulkMoveToStage`)

Move items to a different workflow stage.

**Example**:
```javascript
const job = await client.items.bulkMoveToStage({
  items: [
    { type: 'item', id: '123' },
    { type: 'item', id: '456' }
  ],
  stage: { type: 'workflow_stage', id: 'review' }
});
```

## Filtering, Ordering, and Pagination

### Filter Operators

DatoCMS provides powerful filtering capabilities with various operators:

| Operator | Description | Example |
|----------|-------------|---------|
| `_eq` | Equals | `{ title: { _eq: 'Hello' } }` |
| `_neq` | Not equals | `{ status: { _neq: 'archived' } }` |
| `_matches` | Pattern match (regex) | `{ title: { _matches: '^Breaking' } }` |
| `_not_matches` | Does not match pattern | `{ content: { _not_matches: 'spam' } }` |
| `_lt` | Less than | `{ price: { _lt: 100 } }` |
| `_lte` | Less than or equal | `{ date: { _lte: '2024-12-31' } }` |
| `_gt` | Greater than | `{ views: { _gt: 1000 } }` |
| `_gte` | Greater than or equal | `{ created_at: { _gte: '2024-01-01' } }` |
| `_in` | In array | `{ id: { _in: ['123', '456'] } }` |
| `_not_in` | Not in array | `{ status: { _not_in: ['deleted', 'archived'] } }` |
| `_all_in` | All items in array | `{ tags: { _all_in: ['news', 'tech'] } }` |
| `_any_in` | Any item in array | `{ categories: { _any_in: ['tech', 'science'] } }` |
| `_is_blank` | Is null/empty | `{ description: { _is_blank: true } }` |
| `_is_present` | Is not null/empty | `{ image: { _is_present: true } }` |

### Advanced Filtering Examples

#### String Pattern Matching
```javascript
// Case-sensitive regex
const items = await client.items.list({
  filter: {
    'fields.title.en': { _matches: '^Breaking' }
  }
});

// Case-insensitive matching
const items = await client.items.list({
  filter: {
    'fields.title.en': { 
      _matches: { 
        pattern: 'javascript', 
        case_sensitive: false 
      } 
    }
  }
});
```

#### Date Range Filtering
```javascript
const recentItems = await client.items.list({
  filter: {
    _created_at: {
      _gte: '2024-01-01T00:00:00Z',
      _lte: '2024-12-31T23:59:59Z'
    }
  }
});
```

#### Relationship Filtering
```javascript
// Filter by related item
const itemsByAuthor = await client.items.list({
  filter: {
    'fields.author.en': { id: { _eq: 'author-123' } }
  }
});

// Filter by related item field
const itemsByAuthorEmail = await client.items.list({
  filter: {
    'fields.author.en.email': { _matches: '@company.com$' }
  }
});
```

#### Complex Logic with OR
```javascript
// OR conditions
const draftOrScheduled = await client.items.list({
  filter: {
    _or: [
      { 'fields.status.en': 'draft' },
      { 'fields.status.en': 'scheduled' }
    ]
  }
});

// Nested OR with AND
const complexFilter = await client.items.list({
  filter: {
    type: 'article',
    _or: [
      { 'fields.status.en': 'published' },
      {
        'fields.status.en': 'draft',
        'fields.author.en': { id: { _eq: 'current-user' } }
      }
    ]
  }
});
```

### Ordering

Items can be ordered by multiple fields:

```javascript
// Single field ordering
const newestFirst = await client.items.list({
  orderBy: '_created_at_DESC'
});

// Multiple field ordering
const ordered = await client.items.list({
  orderBy: ['position_ASC', '_created_at_DESC']
});

// Order by field value
const alphabetical = await client.items.list({
  orderBy: 'fields.title.en_ASC'
});

// Random ordering
const randomItems = await client.items.list({
  orderBy: '_random',
  page: { limit: 10 }
});
```

### Pagination Best Practices

```javascript
// Manual pagination
async function getAllItems(filter = {}) {
  const allItems = [];
  let offset = 0;
  const limit = 100;

  while (true) {
    const items = await client.items.list({
      filter,
      page: { offset, limit }
    });

    if (items.length === 0) break;
    
    allItems.push(...items);
    offset += limit;
  }

  return allItems;
}

// Using the iterator (recommended)
async function processAllItems(filter = {}) {
  for await (const item of client.items.listPagedIterator({ filter })) {
    // Process each item
    console.log(item.id);
  }
}
```

## Common Patterns

### Working with Locales

DatoCMS supports multi-locale content. Here's how to work with localized fields:

```javascript
// Create item with multiple locales
const item = await client.items.create({
  item_type: { type: 'item_type', id: 'article' },
  title: {
    en: 'English Title',
    it: 'Titolo Italiano',
    es: 'Título Español'
  }
});

// Query specific locale
const englishItems = await client.items.list({
  locale: 'en',
  fallback_locales: ['en-US', 'en-GB']
});

// Update specific locale
await client.items.update('12345', {
  title: {
    fr: 'Titre Français' // Only updates French locale
  }
});
```

### Working with Relationships

#### Loading Related Data
```javascript
// Fetch article with multiple related items
const article = await client.items.find('12345');

// Collect all relationship IDs
const relatedIds = [
  article.attributes.author,
  ...(article.attributes.categories || []),
  ...(article.attributes.tags || [])
].filter(Boolean);

// Fetch all related items in one request
const relatedItems = await client.items.list({
  filter: {
    ids: {
      in: relatedIds
    }
  },
  page: {
    limit: relatedIds.length
  }
});

// Map relationships
const itemsById = new Map(relatedItems.map(item => [item.id, item]));
const author = itemsById.get(article.attributes.author);
const categories = (article.attributes.categories || [])
  .map(id => itemsById.get(id))
  .filter(Boolean);
```

#### Deep Loading Pattern
```javascript
async function loadItemWithRelationships(itemId, relationships = {}) {
  // Fetch main item
  const item = await client.items.find(itemId);
  const result = { ...item, _loaded: {} };
  
  // Process each relationship
  for (const [field, options] of Object.entries(relationships)) {
    const value = item.attributes[field];
    
    if (!value) continue;
    
    if (Array.isArray(value)) {
      // Multiple reference field
      result._loaded[field] = await loadMultipleItems(value, options);
    } else {
      // Single reference field
      result._loaded[field] = await loadSingleItem(value, options);
    }
  }
  
  return result;
}

// Usage example
const post = await loadItemWithRelationships('12345', {
  author: {
    include: {
      profile_image: {}
    }
  },
  categories: {},
  featured_image: {}
});
```

### Pagination Patterns

#### Manual Pagination Loop
```javascript
async function getAllItems(itemType) {
  const allItems = [];
  const pageSize = 100;
  let offset = 0;
  let hasMore = true;

  while (hasMore) {
    const items = await client.items.list({
      filter: { type: itemType },
      page: { offset, limit: pageSize }
    });

    allItems.push(...items);
    
    // Check if we got a full page
    hasMore = items.length === pageSize;
    offset += pageSize;
    
    console.log(`Fetched ${allItems.length} items so far...`);
  }

  return allItems;
}
```

#### Pagination with Progress
```javascript
async function fetchWithProgress(filter, onProgress) {
  // First get total count
  const countResponse = await client.items.rawList({
    filter,
    page: { limit: 1 }
  });
  
  const total = countResponse.meta.total_count;
  const results = [];
  
  for await (const item of client.items.listPagedIterator(
    { filter },
    { perPage: 100 }
  )) {
    results.push(item);
    
    if (onProgress) {
      onProgress({
        current: results.length,
        total,
        percentage: Math.round((results.length / total) * 100)
      });
    }
  }
  
  return results;
}
```

### Bulk Operations with Error Handling

```javascript
async function bulkUpdateWithErrorTracking(updates) {
  const results = {
    successful: [],
    failed: [],
    stats: {
      total: updates.length,
      successCount: 0,
      failureCount: 0
    }
  };
  
  // Process each update
  const promises = updates.map(async ({ id, data }, index) => {
    try {
      const updated = await client.items.update(id, data);
      results.successful.push({ id, updated });
      results.stats.successCount++;
    } catch (error) {
      results.failed.push({
        id,
        index,
        error: error.message,
        statusCode: error.statusCode,
        validationErrors: error.errors
      });
      results.stats.failureCount++;
    }
  });
  
  await Promise.all(promises);
  
  // Log summary
  console.log(`Success: ${results.stats.successCount}/${results.stats.total}`);
  console.log(`Failed: ${results.stats.failureCount}/${results.stats.total}`);
  
  return results;
}
```

### Field Update Patterns

#### Increment/Decrement Values
```javascript
async function incrementField(itemId, field, amount = 1) {
  const item = await client.items.find(itemId);
  const currentValue = item.attributes[field] || 0;
  
  return await client.items.update(itemId, {
    [field]: currentValue + amount
  });
}

// Usage
await incrementField('12345', 'view_count');
await incrementField('12345', 'likes', 5);
await incrementField('12345', 'stock', -1); // Decrement
```

#### Array Field Operations
```javascript
// Add to array without duplicates
async function addToArray(itemId, field, values) {
  const item = await client.items.find(itemId);
  const current = item.attributes[field] || [];
  const unique = [...new Set([...current, ...values])];
  
  return await client.items.update(itemId, {
    [field]: unique
  });
}

// Remove from array
async function removeFromArray(itemId, field, values) {
  const item = await client.items.find(itemId);
  const current = item.attributes[field] || [];
  const toRemove = new Set(values);
  
  return await client.items.update(itemId, {
    [field]: current.filter(v => !toRemove.has(v))
  });
}
```

### Soft Delete Pattern

```javascript
async function softDelete(itemId) {
  // Instead of destroying, mark as deleted
  await client.items.update(itemId, {
    deleted: true,
    deleted_at: new Date().toISOString(),
    status: 'archived'
  });
  
  console.log('Item archived (soft deleted)');
}

async function restore(itemId) {
  // Restore soft-deleted item
  await client.items.update(itemId, {
    deleted: false,
    deleted_at: null,
    status: 'draft'
  });
}
```

### Raw Methods

For advanced use cases requiring direct JSON:API format:

```javascript
// Using raw methods
const rawResponse = await client.items.rawList({
  filter: { type: { eq: 'article' } },
  page: { offset: 0, limit: 30 }
});

// Access JSON:API structure
console.log(rawResponse.data); // Array of items in JSON:API format
console.log(rawResponse.included); // Included relationships
console.log(rawResponse.meta); // Metadata including pagination

// Raw create with full JSON:API structure
const rawItem = await client.items.rawCreate({
  data: {
    type: 'item',
    attributes: {
      title: 'My Title'
    },
    relationships: {
      item_type: {
        data: { type: 'item_type', id: 'article' }
      }
    }
  }
});
```

## Error Handling

The API uses standard HTTP status codes and returns detailed error information:

```javascript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.items.create({
    item_type: { type: 'item_type', id: 'article' },
    title: '' // Invalid: empty required field
  });
} catch (error) {
  if (error instanceof ApiError) {
    // Handle validation errors
    const titleError = error.findError('title');
    if (titleError) {
      console.log(titleError.code); // 'VALIDATION_REQUIRED'
    }
    
    // Check error response
    console.log(error.response.status); // 422
    console.log(error.errors); // Array of all errors
  }
}
```

### Common Errors

| Status | Error | Cause | Solution |
|--------|-------|-------|----------|
| 401 | UNAUTHORIZED | Invalid or missing API token | Check API token permissions |
| 403 | FORBIDDEN | No permission for operation | Ensure token has required scope |
| 404 | NOT_FOUND | Item or resource not found | Verify ID exists |
| 422 | VALIDATION_REQUIRED | Missing required field | Include all required fields |
| 422 | VALIDATION_FORMAT | Invalid field format | Check field validators |
| 422 | VALIDATION_UNIQUE | Duplicate unique field | Use different value |
| 429 | RATE_LIMITED | Too many requests | Implement retry with backoff |

### Validation Error Handling

```javascript
async function handleValidationErrors(itemsData) {
  const results = await Promise.allSettled(
    itemsData.map(data => client.items.create(data))
  );
  
  results.forEach((result, index) => {
    if (result.status === 'rejected') {
      const error = result.reason;
      
      if (error.statusCode === 422) {
        console.error(`Item ${index} validation errors:`);
        error.errors?.forEach(err => {
          console.error(`- Field '${err.attributes.field}': ${err.attributes.code}`);
          console.error(`  Details: ${err.attributes.details}`);
        });
      }
    }
  });
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Rate Limit Handling

```javascript
async function createWithRateLimitRetry(itemsData, maxRetries = 3) {
  const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));
  
  async function attemptCreate(data, retryCount = 0) {
    try {
      return await client.items.create(data);
    } catch (error) {
      if (error.statusCode === 429 && retryCount < maxRetries) {
        // Rate limited - exponential backoff
        const waitTime = Math.pow(2, retryCount) * 1000;
        console.log(`Rate limited, waiting ${waitTime}ms...`);
        await delay(waitTime);
        return attemptCreate(data, retryCount + 1);
      }
      throw error;
    }
  }
  
  // Process with controlled concurrency
  const results = [];
  const concurrency = 5;
  
  for (let i = 0; i < itemsData.length; i += concurrency) {
    const batch = itemsData.slice(i, i + concurrency);
    const batchResults = await Promise.all(
      batch.map(data => attemptCreate(data))
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

## Object Structure

Items follow the JSON:API specification with this structure:

```typescript
interface Item {
  id: string;
  type: 'item';
  attributes: {
    // Field values as defined in your model
    [fieldName: string]: any;
  };
  relationships: {
    item_type: {
      data: {
        type: 'item_type';
        id: string;
      };
    };
    creator?: {
      data: {
        type: 'account' | 'user';
        id: string;
      };
    };
  };
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
    status: 'draft' | 'published' | 'updated';
    current_version: string;
    stage: string | null;
  };
}
```

### Field Types

Common field types and their value formats:

| Field Type | Value Format | Example |
|------------|--------------|---------|
| Single-line string | string | `"Hello world"` |
| Multi-line text | string | `"Line 1\nLine 2"` |
| Integer | number | `42` |
| Float | number | `3.14` |
| Boolean | boolean | `true` |
| Date | ISO 8601 date | `"2024-01-15"` |
| DateTime | ISO 8601 datetime | `"2024-01-15T10:30:00Z"` |
| Color | Hex string or RGB object | `"#FF5733"` or `{ red: 255, green: 87, blue: 51 }` |
| JSON | Any valid JSON | `{ "key": "value" }` |
| Location | Lat/lng object | `{ latitude: 40.7128, longitude: -74.0060 }` |
| SEO | SEO object | `{ title: "SEO Title", description: "...", image: "..." }` |
| Single asset | Asset reference | `{ upload_id: "123", alt: "...", title: "..." }` |
| Asset gallery | Array of assets | `[{ upload_id: "123", alt: "..." }, ...]` |
| Single link | Item ID | `"12345"` |
| Multiple links | Array of IDs | `["123", "456", "789"]` |
| Structured text | DAST document | `{ schema: "dast", document: { ... } }` |

## Performance Optimization

### Batch Operations
- Use batch sizes of 20-50 for simple items
- Reduce to 10-20 for complex items with many fields
- Use 5-10 for items with media uploads

### Pagination
- Default page size is 30, max is 500
- Use iterators for automatic pagination
- Process items as you receive them to manage memory

### Rate Limiting
- API allows 300 requests per minute
- Implement exponential backoff for 429 errors
- Add delays between batches (100-200ms)

### Query Optimization
- Use specific filters to reduce dataset size
- Always filter by item type first
- Request only needed fields when possible
- Use indexes for frequently filtered fields

### Search Implementation

```javascript
// Full-text search across multiple fields
async function searchItems(query, options = {}) {
  const searchFields = options.fields || ['title', 'content', 'description'];
  const locale = options.locale || 'en';
  
  const orConditions = searchFields.map(field => ({
    [`fields.${field}.${locale}`]: { 
      _matches: { 
        pattern: query, 
        case_sensitive: false 
      } 
    }
  }));
  
  return client.items.list({
    filter: {
      type: options.type || null,
      _or: orConditions
    },
    orderBy: options.orderBy || '_updated_at_DESC',
    page: options.page || { limit: 20 }
  });
}

// Usage
const results = await searchItems('javascript', {
  type: 'article',
  fields: ['title', 'content', 'tags'],
  locale: 'en'
});
```

### Draft/Published Management

```javascript
// Get all draft items
async function getDraftItems(itemType) {
  const allItems = await client.items.list({
    filter: { type: itemType },
    nested: true
  });
  
  return allItems.filter(item => item.meta.status === 'draft');
}

// Get items pending publication
async function getPendingItems(itemType) {
  const allItems = await client.items.list({
    filter: { type: itemType },
    nested: true
  });
  
  return allItems.filter(item => 
    item.meta.status === 'updated' && 
    item.meta.is_current_version_valid
  );
}

// Publish all valid drafts
async function publishAllDrafts(itemType) {
  const draftItems = await getDraftItems(itemType);
  const validDrafts = draftItems.filter(item => item.attributes.is_valid);
  
  const publishResults = await Promise.allSettled(
    validDrafts.map(item => client.items.publish(item.id))
  );
  
  return {
    total: validDrafts.length,
    successful: publishResults.filter(r => r.status === 'fulfilled').length,
    failed: publishResults.filter(r => r.status === 'rejected').length
  };
}
```

### Tree Structure Management

```javascript
// Build tree structure from flat items
async function buildItemTree(itemType, parentField = 'parent') {
  const allItems = await client.items.list({
    filter: { type: itemType }
  });
  
  const itemMap = new Map(allItems.map(item => [item.id, item]));
  const tree = [];
  
  allItems.forEach(item => {
    if (!item[parentField]) {
      tree.push({ ...item, children: [] });
    } else {
      const parent = itemMap.get(item[parentField].id);
      if (parent) {
        if (!parent.children) parent.children = [];
        parent.children.push(item);
      }
    }
  });
  
  return tree;
}

// Get item ancestry
async function getAncestry(itemId, parentField = 'parent') {
  const ancestry = [];
  let currentItem = await client.items.find(itemId);
  
  while (currentItem[parentField]) {
    const parentId = currentItem[parentField].id;
    currentItem = await client.items.find(parentId);
    ancestry.unshift(currentItem);
  }
  
  return ancestry;
}
```

### Localization Patterns

```javascript
// Copy content between locales
async function copyLocaleContent(itemId, fromLocale, toLocale) {
  const item = await client.items.find(itemId);
  const updates = {};
  
  Object.keys(item.attributes).forEach(key => {
    if (typeof item.attributes[key] === 'object' && item.attributes[key][fromLocale]) {
      updates[key] = {
        ...item.attributes[key],
        [toLocale]: item.attributes[key][fromLocale]
      };
    }
  });
  
  return client.items.update(itemId, updates);
}

// Get items missing translations
async function getUntranslatedItems(itemType, locale, requiredFields = ['title']) {
  const items = await client.items.list({
    filter: { type: itemType }
  });
  
  return items.filter(item => {
    return requiredFields.some(field => {
      const fieldValue = item.attributes[field];
      return !fieldValue || !fieldValue[locale] || fieldValue[locale] === '';
    });
  });
}
```

## EditingSession Resource (Private API)

⚠️ **Warning**: The EditingSession resource is a private API that may change without notice. It is primarily used internally by the DatoCMS web interface for collaborative editing features.

### Overview

EditingSessions manage collaborative editing state in DatoCMS, tracking which users are currently editing content items and providing locking mechanisms to prevent editing conflicts during real-time collaboration.

### API Client Section

`client.editingSessions`

### Installation

```bash
npm install @datocms/cma-client
```

### Basic Usage

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// List current editing sessions
const sessions = await client.editingSessions.list();

// Enter editing mode for an item
await client.editingSessions.rawUpdate('session-id', {
  data: {
    type: 'editing_session_enter_item',
    relationships: {
      item: {
        data: { type: 'item', id: 'item-123' }
      }
    }
  }
});

// Destroy editing session (exit editing mode)
await client.editingSessions.destroy('session-id');
```

### API Reference

#### list() - List Active Editing Sessions

Retrieve all active editing sessions across the project.

**Signature**: `list(): Promise<EditingSession[]>`

**Parameters**: None

**Returns**: Array of EditingSession objects

**Example**:
```javascript
const sessions = await client.editingSessions.list();
console.log(`Found ${sessions.length} active editing sessions`);

sessions.forEach(session => {
  console.log(`User ${session.editor.id} editing item ${session.active_item.id}`);
  console.log(`Last activity: ${session.last_activity_at}`);
  console.log(`Locked since: ${session.locked_at || 'Not locked'}`);
});
```

#### rawUpdate() - Update Editing Session State

Update an editing session to perform collaborative editing operations.

**Signature**: `rawUpdate(sessionId: string, body: EditingSessionUpdateSchema): Promise<EditingSession | [EditingSession, FormData]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `sessionId` | string | The editing session ID |
| `body` | object | Update operation payload |

**Update Operations**:

1. **Enter Item Editing**:
```javascript
await client.editingSessions.rawUpdate('session-id', {
  data: {
    type: 'editing_session_enter_item',
    relationships: {
      item: {
        data: { type: 'item', id: 'item-123' }
      }
    }
  }
});
```

2. **Take Over Item Editing**:
```javascript
await client.editingSessions.rawUpdate('session-id', {
  data: {
    type: 'editing_session_take_over_item',
    relationships: {
      item: {
        data: { type: 'item', id: 'item-123' }
      }
    }
  }
});
```

3. **Lock Item for Exclusive Editing**:
```javascript
await client.editingSessions.rawUpdate('session-id', {
  data: {
    type: 'editing_session_lock_item',
    relationships: {
      item: {
        data: { type: 'item', id: 'item-123' }
      }
    }
  }
});
```

#### destroy() - End Editing Session

Terminate an editing session and release any locks.

**Signature**: `destroy(sessionId: string | EditingSession): Promise<EditingSession>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `sessionId` | string \| EditingSession | Session ID or session object |

**Example**:
```javascript
// End session by ID
const destroyedSession = await client.editingSessions.destroy('session-123');

// End session using object
const session = await client.editingSessions.list()[0];
await client.editingSessions.destroy(session);
```

### Object Structure

EditingSession objects have this structure:

```typescript
interface EditingSession {
  id: string;                           // Session identifier
  type: 'editing_session';              // Always 'editing_session'
  last_activity_at: string | null;      // ISO timestamp of last activity
  locked_at: string | null;             // ISO timestamp when locked
  active_item: {                        // Item being edited
    id: string;
    type: string;
  };
  active_item_type: {                   // Type of item being edited
    id: string;
    type: 'item_type';
  };
  editor: {                             // Editor information
    id: string;
    type: 'account' | 'user' | 'sso_user' | 'access_token' | 'organization';
  };
}
```

### Real-time Collaboration Patterns

#### Detecting Active Editors

```javascript
async function getActiveEditors(itemId) {
  const sessions = await client.editingSessions.list();
  
  return sessions
    .filter(session => session.active_item?.id === itemId)
    .map(session => ({
      editorId: session.editor.id,
      editorType: session.editor.type,
      lastActivity: session.last_activity_at,
      isLocked: !!session.locked_at,
      sessionId: session.id
    }));
}

// Usage
const editors = await getActiveEditors('item-123');
if (editors.length > 0) {
  console.log(`Item is being edited by ${editors.length} user(s)`);
  editors.forEach(editor => {
    console.log(`- ${editor.editorType} ${editor.editorId}`);
  });
}
```

#### Collaborative Editing Workflow

```javascript
class CollaborativeEditor {
  constructor(client, itemId) {
    this.client = client;
    this.itemId = itemId;
    this.sessionId = null;
  }

  async startEditing() {
    try {
      // Check if anyone else is editing
      const activeEditors = await this.getActiveEditors();
      
      if (activeEditors.length > 0) {
        console.log('Warning: Other users are editing this item');
        // Ask user if they want to take over or wait
      }

      // Create new session (this would typically be done by the backend)
      // For demonstration purposes only - actual session creation is internal
      console.log('Starting collaborative editing session...');
      
    } catch (error) {
      console.error('Failed to start editing session:', error);
    }
  }

  async getActiveEditors() {
    const sessions = await this.client.editingSessions.list();
    return sessions.filter(s => s.active_item?.id === this.itemId);
  }

  async endEditing() {
    if (this.sessionId) {
      await this.client.editingSessions.destroy(this.sessionId);
      this.sessionId = null;
    }
  }
}

// Usage
const editor = new CollaborativeEditor(client, 'item-123');
await editor.startEditing();
// ... perform editing operations ...
await editor.endEditing();
```

### Conflict Resolution

```javascript
async function handleEditingConflicts(itemId) {
  const sessions = await client.editingSessions.list();
  const itemSessions = sessions.filter(s => s.active_item?.id === itemId);
  
  if (itemSessions.length > 1) {
    console.log(`Editing conflict detected for item ${itemId}`);
    
    // Find the session with earliest activity
    const oldestSession = itemSessions.reduce((oldest, current) => {
      const oldestTime = new Date(oldest.last_activity_at || 0);
      const currentTime = new Date(current.last_activity_at || 0);
      return currentTime < oldestTime ? current : oldest;
    });

    // Check if any session has a lock
    const lockedSession = itemSessions.find(s => s.locked_at);
    
    if (lockedSession) {
      console.log(`Item is locked by session ${lockedSession.id}`);
      return { status: 'locked', session: lockedSession };
    }

    return { 
      status: 'conflict', 
      sessions: itemSessions,
      oldestSession 
    };
  }

  return { status: 'available' };
}
```

### Error Handling

```javascript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.editingSessions.rawUpdate('invalid-session', {
    data: {
      type: 'editing_session_enter_item',
      relationships: {
        item: { data: { type: 'item', id: 'item-123' } }
      }
    }
  });
} catch (error) {
  if (error instanceof ApiError) {
    if (error.statusCode === 404) {
      console.error('Editing session not found or expired');
    } else if (error.statusCode === 422) {
      console.error('Invalid editing session operation');
    }
  }
}
```

### Common Editing Session Errors

| Status | Error | Cause | Solution |
|--------|-------|-------|----------|
| 404 | NOT_FOUND | Session expired or invalid | Create new session |
| 422 | VALIDATION_ERROR | Invalid operation type | Check operation payload |
| 403 | FORBIDDEN | No permission to edit item | Verify user permissions |
| 409 | CONFLICT | Item locked by another user | Wait or request takeover |

### Important Notes

1. **Private API**: This resource is not part of the public API and may change without notice
2. **Internal Use**: Primarily used by DatoCMS web interface for collaborative features
3. **Session Management**: Sessions are typically managed automatically by the DatoCMS frontend
4. **Real-time Updates**: For production collaborative editing, use WebSocket connections with REST API Events
5. **Permissions**: Users must have edit permissions on items to create editing sessions
6. **Automatic Cleanup**: Sessions may be automatically cleaned up after periods of inactivity

## Related Resources

- [Item Types](./item-type.md) - Define content models
- [Fields](./field.md) - Configure model fields
- [Workflows](../03-publishing-workflow/workflow.md) - Content approval workflows
- Job Results - Handle async operations
- [Uploads](../02-media-management/upload.md) - Manage media assets
- [Webhooks](../04-site-configuration/webhook.md) - React to content changes
- [Environments](../04-site-configuration/environment.md) - Manage content environments
- Localization - Multi-locale content