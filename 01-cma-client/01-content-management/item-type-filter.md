# ItemTypeFilter

The ItemTypeFilter resource allows you to create and manage saved filters for content models in DatoCMS. These filters provide quick access to specific subsets of records with predefined search criteria, custom column configurations, and sorting preferences, enhancing content management efficiency in the admin interface.

## Available Operations

### Create Filter

```javascript
const filter = await client.itemTypeFilters.create({
  name: "Draft Articles",
  item_type: { type: "item_type", id: "article" },
  filter: {
    fields: {
      status: { eq: "draft" }
    }
  },
  columns: [
    { name: "title", width: 0.4 },
    { name: "_status", width: 0.2 },
    { name: "_updated_at", width: 0.4 }
  ],
  order_by: "_updated_at_DESC",
  shared: true
});
```

**Parameters:**
- `name` (string, required): Display name for the filter
- `item_type` (object, required): The model this filter applies to
- `filter` (object, required): Filter criteria object
- `columns` (array, optional): Column configuration for the UI
- `order_by` (string, optional): Sorting criteria
- `shared` (boolean, optional): Whether filter is shared with other users

### List Filters

```javascript
const filters = await client.itemTypeFilters.list();
```

**Returns:** Array of all ItemTypeFilter objects

### Find Filter

```javascript
const filter = await client.itemTypeFilters.find(filterId);
```

**Parameters:**
- `filterId` (string, required): The filter ID

**Returns:** Single ItemTypeFilter object

### Update Filter

```javascript
const updated = await client.itemTypeFilters.update(filterId, {
  name: "Updated Filter Name",
  shared: false
});
```

**Parameters:**
- `filterId` (string, required): The filter ID
- `body` (object, required): Fields to update

**Returns:** Updated ItemTypeFilter object

### Delete Filter

```javascript
await client.itemTypeFilters.destroy(filterId);
```

**Parameters:**
- `filterId` (string, required): The filter ID

**Returns:** Empty response on success

## The ItemTypeFilter Object

```typescript
{
  id: string;                    // Filter UUID
  type: 'item_type_filter';      // Always 'item_type_filter'
  name: string;                  // Display name
  filter: {                      // Filter criteria
    [k: string]: unknown;
  };
  columns: Array<{               // UI column configuration
    name: string;                // Field API key or meta column
    width: number;               // Width (0.0 to 1.0)
  }> | null;
  order_by: string | null;       // Sorting criteria
  shared: boolean;               // Shared with team
  item_type: {                   // Associated model
    id: string;
    type: 'item_type';
  };
}
```

## Filter Syntax

### Field-based Filtering

```javascript
// Simple equality
filter: {
  fields: {
    status: { eq: "published" }
  }
}

// Multiple conditions (AND)
filter: {
  fields: {
    status: { eq: "published" },
    featured: { eq: true },
    category: { eq: "technology" }
  }
}

// OR logic
filter: {
  or: [
    { fields: { priority: { eq: "high" } } },
    { fields: { urgent: { eq: true } } }
  ]
}
```

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `{ status: { eq: "draft" } }` |
| `neq` | Not equals | `{ status: { neq: "archived" } }` |
| `gt` | Greater than | `{ views: { gt: 1000 } }` |
| `gte` | Greater than or equal | `{ rating: { gte: 4 } }` |
| `lt` | Less than | `{ price: { lt: 100 } }` |
| `lte` | Less than or equal | `{ stock: { lte: 10 } }` |
| `in` | In array | `{ category: { in: ["tech", "science"] } }` |
| `notIn` | Not in array | `{ status: { notIn: ["deleted", "spam"] } }` |
| `exists` | Has value | `{ image: { exists: true } }` |

### Array Field Operators

```javascript
// Any value in array field matches
filter: {
  fields: {
    tags: { anyIn: ["javascript", "react", "vue"] }
  }
}

// All values must be in array field
filter: {
  fields: {
    required_skills: { allIn: ["html", "css", "js"] }
  }
}
```

### Date Filtering

```javascript
// Date range
filter: {
  fields: {
    published_at: {
      gte: "2024-01-01T00:00:00Z",
      lt: "2024-02-01T00:00:00Z"
    }
  }
}

// Relative dates (last 30 days)
const thirtyDaysAgo = new Date();
thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);

filter: {
  fields: {
    created_at: {
      gte: thirtyDaysAgo.toISOString()
    }
  }
}
```

### Text Search

```javascript
// Full-text search combined with filters
filter: {
  query: "javascript tutorial",
  fields: {
    status: { eq: "published" }
  }
}
```

## Column Configuration

### Available Meta Columns

| Column Name | Description |
|-------------|-------------|
| `id` | Record ID |
| `_preview` | Preview column |
| `_updated_at` | Last update timestamp |
| `_created_at` | Creation timestamp |
| `_creator` | User who created |
| `_status` | Publication status |
| `_published_at` | Publication timestamp |
| `_first_published_at` | First publication |
| `_publication_scheduled_at` | Scheduled publication |
| `_unpublishing_scheduled_at` | Scheduled unpublishing |
| `position` | For sortable models |

### Column Width Configuration

```javascript
columns: [
  { name: "title", width: 0.3 },      // 30%
  { name: "author", width: 0.2 },     // 20%
  { name: "category", width: 0.15 },  // 15%
  { name: "_status", width: 0.15 },   // 15%
  { name: "_updated_at", width: 0.2 } // 20%
]
```

## Sorting

```javascript
// Single field sorting
order_by: "title_ASC"
order_by: "created_at_DESC"

// Multiple field sorting
order_by: "featured_DESC,published_at_DESC"

// Meta field sorting
order_by: "_updated_at_DESC"
order_by: "position_ASC"
```

## Common Use Cases

### 1. Content Status Filters

```javascript
// Draft content filter
const draftsFilter = await client.itemTypeFilters.create({
  name: "Drafts",
  item_type: { type: "item_type", id: "article" },
  filter: {
    fields: {
      _status: { eq: "draft" }
    }
  },
  columns: [
    { name: "title", width: 0.4 },
    { name: "author", width: 0.2 },
    { name: "_updated_at", width: 0.4 }
  ],
  order_by: "_updated_at_DESC",
  shared: true
});

// Scheduled publications
const scheduledFilter = await client.itemTypeFilters.create({
  name: "Scheduled",
  item_type: { type: "item_type", id: "article" },
  filter: {
    fields: {
      _publication_scheduled_at: { exists: true }
    }
  },
  columns: [
    { name: "title", width: 0.4 },
    { name: "_publication_scheduled_at", width: 0.3 },
    { name: "_status", width: 0.3 }
  ],
  order_by: "_publication_scheduled_at_ASC"
});
```

### 2. Author/User Filters

```javascript
// My content filter
const myContentFilter = await client.itemTypeFilters.create({
  name: "My Articles",
  item_type: { type: "item_type", id: "article" },
  filter: {
    fields: {
      author: { eq: { type: "item", id: currentUserId } }
    }
  },
  columns: [
    { name: "title", width: 0.3 },
    { name: "_status", width: 0.2 },
    { name: "category", width: 0.2 },
    { name: "_updated_at", width: 0.3 }
  ],
  order_by: "_updated_at_DESC",
  shared: false // Personal filter
});
```

### 3. Category/Type Filters

```javascript
// Product category filters
async function createCategoryFilters(client, categories) {
  const filters = [];
  
  for (const category of categories) {
    const filter = await client.itemTypeFilters.create({
      name: `${category.name} Products`,
      item_type: { type: "item_type", id: "product" },
      filter: {
        fields: {
          category: { eq: { type: "item", id: category.id } },
          status: { eq: "active" }
        }
      },
      columns: [
        { name: "name", width: 0.3 },
        { name: "sku", width: 0.15 },
        { name: "price", width: 0.15 },
        { name: "stock", width: 0.15 },
        { name: "_updated_at", width: 0.25 }
      ],
      order_by: "name_ASC",
      shared: true
    });
    
    filters.push(filter);
  }
  
  return filters;
}
```

### 4. Performance Filters

```javascript
// High-traffic content
const popularContentFilter = await client.itemTypeFilters.create({
  name: "Popular Content",
  item_type: { type: "item_type", id: "article" },
  filter: {
    fields: {
      views: { gt: 10000 },
      _status: { eq: "published" }
    }
  },
  columns: [
    { name: "title", width: 0.3 },
    { name: "views", width: 0.15 },
    { name: "engagement_rate", width: 0.15 },
    { name: "author", width: 0.2 },
    { name: "_published_at", width: 0.2 }
  ],
  order_by: "views_DESC"
});

// Low-performing content
const underperformingFilter = await client.itemTypeFilters.create({
  name: "Needs Attention",
  item_type: { type: "item_type", id: "article" },
  filter: {
    fields: {
      views: { lt: 100 },
      _published_at: { 
        lte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString()
      }
    }
  },
  order_by: "views_ASC"
});
```

### 5. Complex Business Logic

```javascript
// Featured content with expiration
const featuredActiveFilter = await client.itemTypeFilters.create({
  name: "Active Featured",
  item_type: { type: "item_type", id: "promotion" },
  filter: {
    fields: {
      featured: { eq: true },
      start_date: { lte: new Date().toISOString() },
      end_date: { gte: new Date().toISOString() },
      status: { eq: "approved" }
    }
  },
  columns: [
    { name: "title", width: 0.25 },
    { name: "start_date", width: 0.2 },
    { name: "end_date", width: 0.2 },
    { name: "discount_percentage", width: 0.15 },
    { name: "_status", width: 0.2 }
  ],
  order_by: "end_date_ASC,priority_DESC"
});

// Content needing review
const reviewQueueFilter = await client.itemTypeFilters.create({
  name: "Review Queue",
  item_type: { type: "item_type", id: "article" },
  filter: {
    or: [
      { fields: { review_status: { eq: "pending" } } },
      { fields: { 
        last_reviewed: { 
          lt: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000).toISOString()
        }
      }},
      { fields: { flagged_for_review: { eq: true } } }
    ]
  },
  order_by: "priority_DESC,_created_at_ASC"
});
```

### 6. Localization Filters

```javascript
// Missing translations
const missingTranslationsFilter = await client.itemTypeFilters.create({
  name: "Missing Spanish Translations",
  item_type: { type: "item_type", id: "article" },
  filter: {
    fields: {
      "_locales": { notIn: ["es"] }
    }
  },
  columns: [
    { name: "title", width: 0.4 },
    { name: "_locales", width: 0.3 },
    { name: "_updated_at", width: 0.3 }
  ]
});
```

## Filter Management

### Bulk Filter Creation

```javascript
async function setupDefaultFilters(client, itemTypeId) {
  const defaultFilters = [
    {
      name: "All Published",
      filter: { fields: { _status: { eq: "published" } } },
      order_by: "_published_at_DESC"
    },
    {
      name: "Drafts",
      filter: { fields: { _status: { eq: "draft" } } },
      order_by: "_updated_at_DESC"
    },
    {
      name: "Recently Updated",
      filter: {
        fields: {
          _updated_at: {
            gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString()
          }
        }
      },
      order_by: "_updated_at_DESC"
    },
    {
      name: "Scheduled",
      filter: {
        fields: {
          _publication_scheduled_at: { exists: true }
        }
      },
      order_by: "_publication_scheduled_at_ASC"
    }
  ];
  
  const createdFilters = [];
  
  for (const filterDef of defaultFilters) {
    const filter = await client.itemTypeFilters.create({
      ...filterDef,
      item_type: { type: "item_type", id: itemTypeId },
      shared: true,
      columns: [
        { name: "title", width: 0.4 },
        { name: "_status", width: 0.2 },
        { name: "_updated_at", width: 0.4 }
      ]
    });
    
    createdFilters.push(filter);
  }
  
  return createdFilters;
}
```

## Best Practices

1. **Naming Convention**: Use clear, descriptive names that indicate the filter's purpose
2. **Column Selection**: Include only essential columns to avoid UI clutter
3. **Performance**: Avoid overly complex filters that might slow down queries
4. **Sharing**: Share filters that benefit the entire team, keep personal filters private
5. **Maintenance**: Regularly review and update filters as content structure evolves

## Error Handling

```javascript
try {
  await client.itemTypeFilters.create({
    name: "Test Filter",
    item_type: { type: "item_type", id: "invalid_id" },
    filter: {}
  });
} catch (error) {
  if (error.message.includes('INVALID_FIELD')) {
    console.error('Invalid item type ID');
  } else if (error.message.includes('VALIDATION_ERROR')) {
    console.error('Filter validation failed:', error.details);
  }
}
```

## Raw Response Access

```javascript
const response = await client.itemTypeFilters.rawList();
console.log(response.data); // Array of filters with full JSON:API structure
```

## Related Resources

- [Items](/01-cma-client/01-content-management/item.md) - Content records
- [ItemTypes](/01-cma-client/01-content-management/item-type.md) - Content models
- [MenuItems](/01-cma-client/04-site-configuration/menu-item.md) - Navigation with filters