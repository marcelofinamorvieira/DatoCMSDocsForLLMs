# Filtering Guide - Complete Reference

This guide provides comprehensive documentation for filtering in DatoCMS APIs, covering all available operators, type-specific patterns, and best practices.

## Table of Contents

1. [Filter Operator Reference](#filter-operator-reference)
2. [Type-Specific Filtering](#type-specific-filtering)
3. [Nested Filtering Patterns](#nested-filtering-patterns)
4. [Complex Query Construction](#complex-query-construction)
5. [Performance Optimization](#performance-optimization)
6. [Common Filtering Patterns](#common-filtering-patterns)
7. [Filter Validation](#filter-validation)
8. [Error Handling](#error-handling)
9. [Best Practices](#best-practices)

## Filter Operator Reference

### Comparison Operators

#### `_eq` - Equals

Matches records where the field equals the specified value.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// String equality
const publishedItems = await client.items.list({
  filter: {
    'fields.status.en': { _eq: 'published' }
  }
});

// Number equality
const specificPrice = await client.items.list({
  filter: {
    'fields.price.en': { _eq: 99.99 }
  }
});

// Boolean equality
const featuredItems = await client.items.list({
  filter: {
    'fields.featured.en': { _eq: true }
  }
});

// Date equality (exact match)
const specificDate = await client.items.list({
  filter: {
    _created_at: { _eq: '2024-01-01T00:00:00Z' }
  }
});
```

#### `_neq` - Not Equals

Matches records where the field does not equal the specified value.

```typescript
// Exclude draft items
const nonDraftItems = await client.items.list({
  filter: {
    'fields.status.en': { _neq: 'draft' }
  }
});

// Exclude specific price
const notFreeItems = await client.items.list({
  filter: {
    'fields.price.en': { _neq: 0 }
  }
});
```

#### `_lt` - Less Than

Matches records where the field is less than the specified value.

```typescript
// Items with price less than 50
const affordableItems = await client.items.list({
  filter: {
    'fields.price.en': { _lt: 50 }
  }
});

// Items created before a date
const oldItems = await client.items.list({
  filter: {
    _created_at: { _lt: '2023-01-01T00:00:00Z' }
  }
});
```

#### `_lte` - Less Than or Equal

Matches records where the field is less than or equal to the specified value.

```typescript
// Items with stock <= 10
const lowStockItems = await client.items.list({
  filter: {
    'fields.stock_quantity.en': { _lte: 10 }
  }
});
```

#### `_gt` - Greater Than

Matches records where the field is greater than the specified value.

```typescript
// Items with high ratings
const highRatedItems = await client.items.list({
  filter: {
    'fields.rating.en': { _gt: 4.5 }
  }
});

// Recent items
const recentItems = await client.items.list({
  filter: {
    _created_at: { _gt: '2024-01-01T00:00:00Z' }
  }
});
```

#### `_gte` - Greater Than or Equal

Matches records where the field is greater than or equal to the specified value.

```typescript
// Premium items
const premiumItems = await client.items.list({
  filter: {
    'fields.price.en': { _gte: 100 }
  }
});
```

### String Operators

#### `_matches` - Pattern Matching

Matches records using a regular expression pattern.

```typescript
// Find items with titles containing "guide"
const guideItems = await client.items.list({
  filter: {
    'fields.title.en': { _matches: { pattern: 'guide', flags: 'i' } }
  }
});

// Find items with specific URL patterns
const blogPosts = await client.items.list({
  filter: {
    'fields.slug.en': { _matches: { pattern: '^blog/', flags: '' } }
  }
});

// Complex pattern matching
const productCodes = await client.items.list({
  filter: {
    'fields.sku.en': { _matches: { pattern: '^PRD-[0-9]{4}$', flags: '' } }
  }
});
```

#### `_starts_with` - String Prefix

Matches records where the field starts with the specified string.

```typescript
// Find items with titles starting with "How to"
const howToItems = await client.items.list({
  filter: {
    'fields.title.en': { _starts_with: 'How to' }
  }
});
```

#### `_ends_with` - String Suffix

Matches records where the field ends with the specified string.

```typescript
// Find items with URLs ending in .pdf
const pdfDocuments = await client.items.list({
  filter: {
    'fields.file_url.en': { _ends_with: '.pdf' }
  }
});
```

### Array Operators

#### `_in` - In Array

Matches records where the field value is in the specified array.

```typescript
// Find items with specific statuses
const activeItems = await client.items.list({
  filter: {
    'fields.status.en': { _in: ['published', 'scheduled'] }
  }
});

// Find items by multiple IDs
const specificItems = await client.items.list({
  filter: {
    id: { _in: ['item1', 'item2', 'item3'] }
  }
});

// Filter uploads by format
const webImages = await client.uploads.list({
  filter: {
    format: { _in: ['jpg', 'png', 'webp', 'gif'] }
  }
});
```

#### `_not_in` - Not In Array

Matches records where the field value is not in the specified array.

```typescript
// Exclude specific categories
const nonTechItems = await client.items.list({
  filter: {
    'fields.category.en': { _not_in: ['technology', 'software'] }
  }
});
```

#### `_all_in` - All In Array (for array fields)

Matches records where the array field contains all specified values.

```typescript
// Find items with all required tags
const taggedItems = await client.items.list({
  filter: {
    'fields.tags.en': { _all_in: ['featured', 'premium'] }
  }
});
```

#### `_any_in` - Any In Array (for array fields)

Matches records where the array field contains any of the specified values.

```typescript
// Find items with any of the specified tags
const taggedItems = await client.items.list({
  filter: {
    'fields.tags.en': { _any_in: ['sale', 'discount', 'clearance'] }
  }
});
```

### Existence Operators

#### `_exists` - Field Exists

Matches records where the field exists (is not null).

```typescript
// Find items with descriptions
const itemsWithDescriptions = await client.items.list({
  filter: {
    'fields.description.en': { _exists: true }
  }
});

// Find items without descriptions
const itemsWithoutDescriptions = await client.items.list({
  filter: {
    'fields.description.en': { _exists: false }
  }
});
```

#### `_blank` - Field is Blank

Matches records where the field is blank (empty string or null).

```typescript
// Find items with blank titles
const blankTitleItems = await client.items.list({
  filter: {
    'fields.title.en': { _blank: true }
  }
});
```

#### `_present` - Field is Present

Matches records where the field has a non-blank value.

```typescript
// Find items with non-empty descriptions
const itemsWithContent = await client.items.list({
  filter: {
    'fields.description.en': { _present: true }
  }
});
```

## Type-Specific Filtering

### String Field Filtering

```typescript
// Multiple string operators combined
const filteredItems = await client.items.list({
  filter: {
    'fields.title.en': {
      _matches: { pattern: 'guide', flags: 'i' },
      _neq: 'User Guide'
    }
  }
});

// Case-insensitive search
const searchResults = await client.items.list({
  filter: {
    'fields.title.en': {
      _matches: { pattern: searchTerm, flags: 'i' }
    }
  }
});
```

### Number Field Filtering

```typescript
// Range filtering
const priceRangeItems = await client.items.list({
  filter: {
    'fields.price.en': {
      _gte: 10,
      _lte: 100
    }
  }
});

// Multiple numeric conditions
const qualityItems = await client.items.list({
  filter: {
    'fields.rating.en': { _gte: 4 },
    'fields.reviews_count.en': { _gt: 10 }
  }
});
```

### Date Field Filtering

```typescript
// Date range filtering
const dateRangeItems = await client.items.list({
  filter: {
    _created_at: {
      _gte: '2024-01-01T00:00:00Z',
      _lt: '2024-02-01T00:00:00Z'
    }
  }
});

// Relative date filtering (last 30 days)
const thirtyDaysAgo = new Date();
thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

const recentItems = await client.items.list({
  filter: {
    _updated_at: {
      _gte: thirtyDaysAgo.toISOString()
    }
  }
});

// Publishing date filters
const scheduledItems = await client.items.list({
  filter: {
    _first_published_at: { _exists: false },
    'fields.publish_date.en': {
      _gte: new Date().toISOString()
    }
  }
});
```

### Boolean Field Filtering

```typescript
// Simple boolean filtering
const featuredItems = await client.items.list({
  filter: {
    'fields.is_featured.en': { _eq: true },
    'fields.is_archived.en': { _eq: false }
  }
});

// Combined with other filters
const activeFeaturedItems = await client.items.list({
  filter: {
    'fields.is_featured.en': { _eq: true },
    'fields.status.en': { _eq: 'active' }
  }
});
```

### Array Field Filtering

```typescript
// Array contains specific value
const taggedItems = await client.items.list({
  filter: {
    'fields.tags.en': { _in: ['javascript'] }
  }
});

// Array contains all values
const multiTaggedItems = await client.items.list({
  filter: {
    'fields.tags.en': { _all_in: ['javascript', 'typescript'] }
  }
});

// Array contains any value
const categoryItems = await client.items.list({
  filter: {
    'fields.categories.en': { _any_in: ['tech', 'programming', 'tutorial'] }
  }
});

// Empty array check
const untaggedItems = await client.items.list({
  filter: {
    'fields.tags.en': { _eq: [] }
  }
});
```

### JSON Field Filtering

```typescript
// Filter by JSON field properties
const configuredItems = await client.items.list({
  filter: {
    'fields.config.en.enabled': { _eq: true },
    'fields.config.en.version': { _gte: 2 }
  }
});

// Nested JSON filtering
const specificSettings = await client.items.list({
  filter: {
    'fields.settings.en.display.theme': { _eq: 'dark' },
    'fields.settings.en.display.layout': { _in: ['grid', 'list'] }
  }
});

// JSON field existence
const hasMetadata = await client.items.list({
  filter: {
    'fields.metadata.en': { _exists: true }
  }
});
```

### Link Field Filtering (References)

```typescript
// Filter by linked item
const authoredItems = await client.items.list({
  filter: {
    'fields.author.en': { _eq: 'author_id_123' }
  }
});

// Filter by multiple linked items
const multiAuthorItems = await client.items.list({
  filter: {
    'fields.authors.en': { _any_in: ['author1', 'author2', 'author3'] }
  }
});

// Filter by absence of link
const orphanedItems = await client.items.list({
  filter: {
    'fields.category.en': { _exists: false }
  }
});
```

## Nested Filtering Patterns

### Filter by Related Item Properties

```typescript
// Filter uploads by collection
const collectionUploads = await client.uploads.list({
  filter: {
    upload_collection: {
      id: { _eq: 'collection_id' }
    }
  }
});

// Filter items by creator
const userItems = await client.items.list({
  filter: {
    creator: {
      id: { _eq: 'user_id' }
    }
  }
});
```

### Deep Nested Filtering

```typescript
// Complex nested structure filtering
const complexFilter = await client.items.list({
  filter: {
    type: 'article',
    'fields.author.en': {
      id: { _in: ['author1', 'author2'] }
    },
    'fields.metadata.en.seo.title': {
      _matches: { pattern: 'SEO', flags: 'i' }
    }
  }
});
```

## Complex Query Construction

### Combining Multiple Filters

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Complex product filter
async function findProducts(params: {
  minPrice?: number;
  maxPrice?: number;
  categories?: string[];
  inStock?: boolean;
  searchTerm?: string;
}) {
  const filter: any = {};

  // Price range
  if (params.minPrice !== undefined || params.maxPrice !== undefined) {
    filter['fields.price.en'] = {};
    if (params.minPrice !== undefined) {
      filter['fields.price.en']._gte = params.minPrice;
    }
    if (params.maxPrice !== undefined) {
      filter['fields.price.en']._lte = params.maxPrice;
    }
  }

  // Categories
  if (params.categories && params.categories.length > 0) {
    filter['fields.category.en'] = { _in: params.categories };
  }

  // Stock status
  if (params.inStock !== undefined) {
    filter['fields.in_stock.en'] = { _eq: params.inStock };
  }

  // Search term
  if (params.searchTerm) {
    filter['fields.title.en'] = {
      _matches: { pattern: params.searchTerm, flags: 'i' }
    };
  }

  const products = await client.items.list({
    filter: filter,
    orderBy: ['fields.price.en_ASC'],
    page: { limit: 100 }
  });

  return products;
}
```

### Dynamic Filter Building

```typescript
// Build filters dynamically based on user input
function buildDynamicFilter(userFilters: Record<string, any>) {
  const filter: Record<string, any> = {};

  Object.entries(userFilters).forEach(([key, value]) => {
    if (value === null || value === undefined) return;

    // Handle different value types
    if (Array.isArray(value)) {
      filter[key] = { _in: value };
    } else if (typeof value === 'object' && value.min !== undefined) {
      // Range filter
      filter[key] = {};
      if (value.min !== undefined) filter[key]._gte = value.min;
      if (value.max !== undefined) filter[key]._lte = value.max;
    } else if (typeof value === 'string' && value.includes('*')) {
      // Wildcard to regex
      const pattern = value.replace(/\*/g, '.*');
      filter[key] = { _matches: { pattern, flags: 'i' } };
    } else {
      filter[key] = { _eq: value };
    }
  });

  return filter;
}

// Usage
const userInput = {
  'fields.status.en': 'published',
  'fields.price.en': { min: 10, max: 100 },
  'fields.tags.en': ['featured', 'sale'],
  'fields.title.en': '*guide*'
};

const dynamicFilter = buildDynamicFilter(userInput);
const results = await client.items.list({ filter: dynamicFilter });
```

### Conditional Filter Application

```typescript
interface SearchParams {
  query?: string;
  status?: string;
  author?: string;
  dateFrom?: string;
  dateTo?: string;
  tags?: string[];
  excludeArchived?: boolean;
}

async function searchItems(params: SearchParams) {
  const filters: any[] = [];

  // Text search
  if (params.query) {
    filters.push({
      or: [
        { 'fields.title.en': { _matches: { pattern: params.query, flags: 'i' } } },
        { 'fields.description.en': { _matches: { pattern: params.query, flags: 'i' } } },
        { 'fields.content.en': { _matches: { pattern: params.query, flags: 'i' } } }
      ]
    });
  }

  // Status filter
  if (params.status) {
    filters.push({ 'fields.status.en': { _eq: params.status } });
  }

  // Author filter
  if (params.author) {
    filters.push({ 'fields.author.en': { _eq: params.author } });
  }

  // Date range
  if (params.dateFrom || params.dateTo) {
    const dateFilter: any = {};
    if (params.dateFrom) dateFilter._gte = params.dateFrom;
    if (params.dateTo) dateFilter._lte = params.dateTo;
    filters.push({ _created_at: dateFilter });
  }

  // Tags filter
  if (params.tags && params.tags.length > 0) {
    filters.push({ 'fields.tags.en': { _any_in: params.tags } });
  }

  // Exclude archived
  if (params.excludeArchived) {
    filters.push({ 'fields.is_archived.en': { _neq: true } });
  }

  // Combine all filters
  const filter = filters.length > 0 ? { and: filters } : {};

  return await client.items.list({
    filter,
    orderBy: '_created_at_DESC',
    page: { limit: 50 }
  });
}
```

## Performance Optimization

### Efficient Filter Design

```typescript
// ❌ Inefficient: Multiple API calls
async function inefficientApproach(categoryIds: string[]) {
  const results = [];
  for (const categoryId of categoryIds) {
    const items = await client.items.list({
      filter: { 'fields.category.en': { _eq: categoryId } }
    });
    results.push(...items);
  }
  return results;
}

// ✅ Efficient: Single API call with _in operator
async function efficientApproach(categoryIds: string[]) {
  return await client.items.list({
    filter: { 'fields.category.en': { _in: categoryIds } }
  });
}
```

### Pagination for Large Result Sets

```typescript
async function* paginateResults(filter: any) {
  let offset = 0;
  const limit = 100;
  let hasMore = true;

  while (hasMore) {
    const response = await client.items.rawList({
      filter,
      page: { offset, limit }
    });

    yield response.data;

    hasMore = response.data.length === limit;
    offset += limit;
  }
}

// Usage
async function processAllItems(filter: any) {
  for await (const batch of paginateResults(filter)) {
    // Process batch of items
    console.log(`Processing ${batch.length} items`);
    await processBatch(batch);
  }
}
```

### Selective Field Loading

```typescript
// Load only necessary fields for better performance
const lightweightItems = await client.items.list({
  filter: { type: 'article' },
  fields: {
    item: ['title', 'slug', 'status'],
    author: ['name']
  }
});
```

### Index-Friendly Filtering

```typescript
// Use indexed fields for better performance
// ✅ Good: Filter by indexed fields like id, type, _created_at
const efficientFilter = await client.items.list({
  filter: {
    type: 'article',
    _created_at: { _gte: '2024-01-01T00:00:00Z' }
  }
});

// ❌ Avoid: Complex regex on large text fields
const inefficientFilter = await client.items.list({
  filter: {
    'fields.content.en': {
      _matches: { pattern: 'complex.*regex.*pattern', flags: 'i' }
    }
  }
});
```

## Common Filtering Patterns

### Search Implementation

```typescript
// Full-text search across multiple fields
async function searchContent(searchTerm: string) {
  const searchPattern = { pattern: searchTerm, flags: 'i' };
  
  return await client.items.list({
    filter: {
      or: [
        { 'fields.title.en': { _matches: searchPattern } },
        { 'fields.subtitle.en': { _matches: searchPattern } },
        { 'fields.description.en': { _matches: searchPattern } },
        { 'fields.tags.en': { _in: [searchTerm.toLowerCase()] } }
      ]
    },
    orderBy: '_updated_at_DESC'
  });
}
```

### Date-Based Filtering

```typescript
// Get items published in the last week
async function getRecentlyPublished() {
  const oneWeekAgo = new Date();
  oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);

  return await client.items.list({
    filter: {
      _first_published_at: {
        _gte: oneWeekAgo.toISOString()
      }
    }
  });
}

// Get items scheduled for future publication
async function getScheduledItems() {
  const now = new Date().toISOString();
  
  return await client.items.list({
    filter: {
      'fields.publish_date.en': {
        _gt: now
      },
      _first_published_at: { _exists: false }
    }
  });
}
```

### Status-Based Workflows

```typescript
// Content review workflow
async function getItemsForReview() {
  return await client.items.list({
    filter: {
      'fields.status.en': { _in: ['pending_review', 'in_review'] },
      'fields.assigned_reviewer.en': { _exists: true }
    },
    orderBy: ['fields.priority.en_DESC', '_updated_at_ASC']
  });
}

// Archived content management
async function getArchivedContent(beforeDate?: string) {
  const filter: any = {
    'fields.status.en': { _eq: 'archived' }
  };

  if (beforeDate) {
    filter['fields.archived_date.en'] = { _lt: beforeDate };
  }

  return await client.items.list({
    filter,
    orderBy: 'fields.archived_date.en_DESC'
  });
}
```

### Media Asset Filtering

```typescript
// Find unused assets
async function findUnusedAssets() {
  return await client.uploads.list({
    filter: {
      in_use: false,
      _created_at: {
        _lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString()
      }
    }
  });
}

// Find large media files
async function findLargeFiles(sizeInMB: number) {
  const sizeInBytes = sizeInMB * 1024 * 1024;
  
  return await client.uploads.list({
    filter: {
      size: { _gt: sizeInBytes },
      type: { _in: ['image', 'video'] }
    },
    orderBy: 'size_DESC'
  });
}

// Find assets by metadata
async function findAssetsByMetadata() {
  return await client.uploads.list({
    filter: {
      'default_field_metadata.en.alt': { _present: true },
      'default_field_metadata.en.custom_data.project': { _eq: 'summer-campaign' }
    }
  });
}
```

## Filter Validation

### Input Validation

```typescript
// Validate and sanitize filter inputs
function validateFilterInput(input: any): Record<string, any> {
  const validatedFilter: Record<string, any> = {};

  // Whitelist allowed fields
  const allowedFields = ['title', 'status', 'category', 'price', 'created_at'];
  
  Object.entries(input).forEach(([key, value]) => {
    // Check if field is allowed
    if (!allowedFields.includes(key)) {
      console.warn(`Ignoring invalid filter field: ${key}`);
      return;
    }

    // Validate value types
    if (key === 'price' && typeof value === 'object') {
      if (value.min !== undefined && typeof value.min !== 'number') {
        throw new Error('Price min must be a number');
      }
      if (value.max !== undefined && typeof value.max !== 'number') {
        throw new Error('Price max must be a number');
      }
    }

    // Sanitize string inputs
    if (typeof value === 'string') {
      value = value.trim();
      if (value.length === 0) return;
    }

    // Build field path
    const fieldPath = `fields.${key}.en`;
    validatedFilter[fieldPath] = value;
  });

  return validatedFilter;
}
```

### Type-Safe Filter Building

```typescript
// Type-safe filter builder
interface ProductFilter {
  title?: string;
  category?: string[];
  priceRange?: { min?: number; max?: number };
  inStock?: boolean;
  tags?: string[];
}

function buildProductFilter(input: ProductFilter) {
  const filter: Record<string, any> = {};

  if (input.title) {
    filter['fields.title.en'] = {
      _matches: { pattern: input.title, flags: 'i' }
    };
  }

  if (input.category && input.category.length > 0) {
    filter['fields.category.en'] = { _in: input.category };
  }

  if (input.priceRange) {
    filter['fields.price.en'] = {};
    if (input.priceRange.min !== undefined) {
      filter['fields.price.en']._gte = input.priceRange.min;
    }
    if (input.priceRange.max !== undefined) {
      filter['fields.price.en']._lte = input.priceRange.max;
    }
  }

  if (input.inStock !== undefined) {
    filter['fields.in_stock.en'] = { _eq: input.inStock };
  }

  if (input.tags && input.tags.length > 0) {
    filter['fields.tags.en'] = { _any_in: input.tags };
  }

  return filter;
}
```

## Error Handling

### Common Filter Errors

```typescript
// Handle common filtering errors
async function safeFilterQuery(filter: any) {
  try {
    return await client.items.list({ filter });
  } catch (error: any) {
    if (error.response?.status === 422) {
      // Validation error
      console.error('Invalid filter:', error.response.data);
      
      // Parse specific errors
      const errors = error.response.data.errors || [];
      errors.forEach((err: any) => {
        if (err.detail.includes('Invalid filter')) {
          console.error(`Filter error on field ${err.source.pointer}: ${err.detail}`);
        }
      });
      
      throw new Error('Invalid filter parameters');
    }
    
    if (error.response?.status === 400) {
      // Bad request
      console.error('Bad request:', error.response.data);
      throw new Error('Malformed filter query');
    }
    
    // Re-throw other errors
    throw error;
  }
}
```

### Graceful Degradation

```typescript
// Implement fallback strategies
async function searchWithFallback(searchTerm: string) {
  try {
    // Try complex search first
    return await client.items.list({
      filter: {
        or: [
          { 'fields.title.en': { _matches: { pattern: searchTerm, flags: 'i' } } },
          { 'fields.content.en': { _matches: { pattern: searchTerm, flags: 'i' } } }
        ]
      }
    });
  } catch (error) {
    console.warn('Complex search failed, trying simple search');
    
    try {
      // Fallback to simpler search
      return await client.items.list({
        filter: {
          'fields.title.en': { _matches: { pattern: searchTerm, flags: 'i' } }
        }
      });
    } catch (fallbackError) {
      console.error('Search failed completely');
      return [];
    }
  }
}
```

### Filter Debugging

```typescript
// Debug filter construction
function debugFilter(filter: any) {
  console.log('Filter Debug:');
  console.log(JSON.stringify(filter, null, 2));
  
  // Check for common issues
  const warnings: string[] = [];
  
  // Check for empty arrays
  JSON.stringify(filter, (key, value) => {
    if (Array.isArray(value) && value.length === 0) {
      warnings.push(`Empty array in filter: ${key}`);
    }
    return value;
  });
  
  // Check for null/undefined values
  JSON.stringify(filter, (key, value) => {
    if (value === null || value === undefined) {
      warnings.push(`Null/undefined value in filter: ${key}`);
    }
    return value;
  });
  
  if (warnings.length > 0) {
    console.warn('Filter warnings:', warnings);
  }
  
  return filter;
}

// Usage
const filter = debugFilter({
  'fields.category.en': { _in: [] }, // Warning: empty array
  'fields.title.en': null, // Warning: null value
  'fields.status.en': { _eq: 'published' }
});
```

## Best Practices

### 1. Use Specific Filters

```typescript
// ❌ Avoid: Fetching all items and filtering in memory
const allItems = await client.items.list();
const publishedItems = allItems.filter(item => 
  item.attributes.fields.status.en === 'published'
);

// ✅ Better: Filter at the API level
const publishedItems = await client.items.list({
  filter: {
    'fields.status.en': { _eq: 'published' }
  }
});
```

### 2. Combine Filters Efficiently

```typescript
// ❌ Avoid: Multiple separate queries
const draftItems = await client.items.list({
  filter: { 'fields.status.en': { _eq: 'draft' } }
});
const pendingItems = await client.items.list({
  filter: { 'fields.status.en': { _eq: 'pending' } }
});
const allItems = [...draftItems, ...pendingItems];

// ✅ Better: Single query with _in operator
const items = await client.items.list({
  filter: {
    'fields.status.en': { _in: ['draft', 'pending'] }
  }
});
```

### 3. Use Appropriate Operators

```typescript
// ❌ Avoid: Using regex for exact matches
const item = await client.items.list({
  filter: {
    'fields.slug.en': { _matches: { pattern: '^exact-slug$', flags: '' } }
  }
});

// ✅ Better: Use _eq for exact matches
const item = await client.items.list({
  filter: {
    'fields.slug.en': { _eq: 'exact-slug' }
  }
});
```

### 4. Handle Edge Cases

```typescript
// Handle empty filter values
function buildSafeFilter(params: any) {
  const filter: any = {};
  
  // Only add filters with actual values
  if (params.search?.trim()) {
    filter['fields.title.en'] = {
      _matches: { pattern: params.search.trim(), flags: 'i' }
    };
  }
  
  if (params.categories?.length > 0) {
    filter['fields.category.en'] = { _in: params.categories };
  }
  
  if (params.minPrice != null && !isNaN(params.minPrice)) {
    filter['fields.price.en'] = filter['fields.price.en'] || {};
    filter['fields.price.en']._gte = Number(params.minPrice);
  }
  
  return filter;
}
```

### 5. Document Filter Logic

```typescript
/**
 * Find products matching the given criteria
 * @param criteria - Search criteria
 * @param criteria.query - Text to search in title and description
 * @param criteria.minPrice - Minimum price (inclusive)
 * @param criteria.maxPrice - Maximum price (inclusive)
 * @param criteria.categories - Array of category IDs
 * @param criteria.inStock - If true, only return items in stock
 * @returns Filtered list of products
 */
async function findProducts(criteria: {
  query?: string;
  minPrice?: number;
  maxPrice?: number;
  categories?: string[];
  inStock?: boolean;
}) {
  const filter: any = {};
  
  // Full-text search across multiple fields
  if (criteria.query) {
    const searchPattern = { pattern: criteria.query, flags: 'i' };
    filter.or = [
      { 'fields.title.en': { _matches: searchPattern } },
      { 'fields.description.en': { _matches: searchPattern } }
    ];
  }
  
  // Price range filter
  if (criteria.minPrice !== undefined || criteria.maxPrice !== undefined) {
    filter['fields.price.en'] = {};
    if (criteria.minPrice !== undefined) {
      filter['fields.price.en']._gte = criteria.minPrice;
    }
    if (criteria.maxPrice !== undefined) {
      filter['fields.price.en']._lte = criteria.maxPrice;
    }
  }
  
  // Category filter
  if (criteria.categories && criteria.categories.length > 0) {
    filter['fields.category.en'] = { _in: criteria.categories };
  }
  
  // Stock filter
  if (criteria.inStock === true) {
    filter['fields.stock_quantity.en'] = { _gt: 0 };
  }
  
  return await client.items.list({
    filter,
    orderBy: ['fields.relevance.en_DESC', '_created_at_DESC']
  });
}
```

### 6. Optimize for Performance

```typescript
// Cache frequently used filters
const filterCache = new Map<string, any>();

async function getCachedFilterResults(filterKey: string, filter: any) {
  const cacheKey = `${filterKey}:${JSON.stringify(filter)}`;
  
  if (filterCache.has(cacheKey)) {
    const cached = filterCache.get(cacheKey);
    if (cached.timestamp > Date.now() - 5 * 60 * 1000) { // 5 min cache
      return cached.data;
    }
  }
  
  const results = await client.items.list({ filter });
  filterCache.set(cacheKey, {
    data: results,
    timestamp: Date.now()
  });
  
  return results;
}
```

## Cross-Reference Examples

### Items (Records)
- See [Item Methods](../01-cma-client/01-content-management/item-methods.md) for content filtering
- See [Item Type Methods](../01-cma-client/01-content-management/item-type-methods.md) for model filtering

### Media Assets
- See [Upload Methods](../01-cma-client/02-media-management/upload-methods.md) for asset filtering
- See [Upload Filter Methods](../01-cma-client/02-media-management/upload-filter-methods.md) for saved filters

### Administrative
- See [Audit Log Event Methods](../01-cma-client/06-monitoring/audit-log-event-methods.md) for activity filtering
- See [User Methods](../01-cma-client/05-access-control/user-methods.md) for user filtering

### Publishing
- See [Scheduled Publication Methods](../01-cma-client/03-publishing-workflow/scheduled-publication-methods.md) for workflow filtering

This comprehensive guide covers all aspects of filtering in DatoCMS APIs. Remember to always test your filters thoroughly and handle edge cases appropriately for robust applications.