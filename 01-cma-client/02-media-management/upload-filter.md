# Upload Filter Resource

## Overview

The Upload Filter resource allows you to create and manage saved filters for media uploads in DatoCMS. These filters help organize and quickly access specific subsets of uploads in the media library, improving asset management efficiency for content teams.

## API Client Section

`client.uploadFilters`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create an upload filter
const filter = await client.uploadFilters.create({
  name: "Unused Images",
  filter: {
    in_use: { eq: false },
    type: { eq: "image" }
  },
  shared: true
});

// List all filters
const filters = await client.uploadFilters.list();

// Update a filter
const updated = await client.uploadFilters.update(filter.id, {
  name: "Updated Filter Name"
});

// Delete a filter
await client.uploadFilters.destroy(filter.id);
```

## API Reference

### list() - List Upload Filters

Retrieve all upload filters for the project.

**Signature**: `list(queryParams): Promise<UploadFilter[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `orderBy` | string | Sort order (e.g., 'name_ASC') |

**Example**:
```typescript
// Get all upload filters
const filters = await client.uploadFilters.list();

// Get filters with ordering
const orderedFilters = await client.uploadFilters.list({
  orderBy: 'name_ASC'
});

// Get raw response with metadata
const response = await client.uploadFilters.rawList();
// response.meta contains pagination info
```

### find() - Get Single Upload Filter

Retrieve a single upload filter by ID.

**Signature**: `find(uploadFilterId): Promise<UploadFilter>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadFilterId` | string | The filter ID (required) |

**Example**:
```typescript
// Get upload filter by ID
const filter = await client.uploadFilters.find('filter_id');

// Upload filter structure includes:
// {
//   id: 'filter_id',
//   type: 'upload_filter',
//   name: 'Product Images',
//   shared: true,
//   filter: {
//     type: { eq: 'image' },
//     upload_collection: { eq: 'products_collection_id' }
//   }
// }

// Error handling
try {
  const filter = await client.uploadFilters.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Upload filter not found');
  }
}
```

### create() - Create New Upload Filter

Creates a new upload filter.

**Signature**: `create(body): Promise<UploadFilter>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Display name for the filter (required) |
| `filter` | object | Filter criteria object (required) |
| `shared` | boolean | Whether filter is shared with other users (required) |

**Basic Example**:
```javascript
// Create basic filter
const imageFilter = await client.uploadFilters.create({
  name: 'Images Only',
  filter: {
    type: { eq: 'image' }
  },
  shared: true
});

// Create filter for specific collection
const collectionFilter = await client.uploadFilters.create({
  name: 'Marketing Assets',
  filter: {
    upload_collection: { eq: 'marketing_collection_id' }
  },
  shared: true
});
```

**Advanced Example**:
```javascript
// Create filter with multiple conditions
const complexFilter = await client.uploadFilters.create({
  name: 'Large HD Images',
  filter: {
    type: { eq: 'image' },
    width: { gte: 1920 },
    height: { gte: 1080 },
    size: { gt: 1000000 } // > 1MB
  },
  shared: false // Private filter
});

// Create filter for unused assets
const unusedFilter = await client.uploadFilters.create({
  name: 'Unused Assets',
  filter: {
    in_use: { eq: false }
  },
  shared: true
});

// Create filter with date range
const recentFilter = await client.uploadFilters.create({
  name: 'Recent Uploads (30 days)',
  filter: {
    created_at: { 
      gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString() 
    }
  },
  shared: true
});

// Error handling
try {
  const filter = await client.uploadFilters.create({
    name: '', // Empty name not allowed
    filter: {}
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation error:', error.response.data.errors);
  }
}
```

### update() - Update Upload Filter

Updates an existing upload filter.

**Signature**: `update(uploadFilterId, body): Promise<UploadFilter>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadFilterId` | string | The filter ID (required) |
| `name` | string | New display name (optional) |
| `filter` | object | New filter criteria (optional) |
| `shared` | boolean | New sharing status (optional) |

**Example**:
```javascript
// Update filter name
const updated = await client.uploadFilters.update(filterId, {
  name: "Updated Filter Name",
  shared: false
});

// Update filter criteria
const updatedQuery = await client.uploadFilters.update('filter_id', {
  filter: {
    type: { eq: 'image' },
    size: { lte: 5000000 } // <= 5MB
  }
});

// Make filter shared/private
const shared = await client.uploadFilters.update('filter_id', {
  shared: true
});

// Error handling
try {
  const updated = await client.uploadFilters.update('filter_id', {
    name: '' // Empty name not allowed
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Name cannot be empty');
  }
}
```

### destroy() - Delete Upload Filter

Deletes an upload filter.

**Signature**: `destroy(uploadFilterId): Promise<void>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadFilterId` | string | The filter ID (required) |

**Example**:
```javascript
// Delete upload filter
await client.uploadFilters.destroy(filterId);

// Delete with error handling
try {
  await client.uploadFilters.destroy('filter_id');
  console.log('Upload filter deleted successfully');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Upload filter not found');
  } else if (error.response?.status === 403) {
    console.error('Cannot delete shared filter created by another user');
  }
}

// Bulk delete pattern
async function deleteMultipleFilters(filterIds) {
  const results = await Promise.allSettled(
    filterIds.map(id => client.uploadFilters.destroy(id))
  );
  
  return {
    successful: results.filter(r => r.status === 'fulfilled').length,
    failed: results.filter(r => r.status === 'rejected').length
  };
}
```

## Filter Syntax

### Common Filter Fields

| Field | Description | Type |
|-------|-------------|------|
| `in_use` | Whether upload is used in records | boolean |
| `type` | File type category | string |
| `format` | Specific file format | string |
| `size` | File size in bytes | number |
| `tags` | Assigned tags | array |
| `author` | User who uploaded | reference |
| `upload_collection` | Collection ID | reference |
| `created_at` | Creation timestamp | datetime |
| `updated_at` | Last update timestamp | datetime |

### File Type Values

- `image` - Image files (jpg, png, gif, etc.)
- `video` - Video files
- `audio` - Audio files
- `document` - Documents (pdf, doc, etc.)
- `archive` - Archive files (zip, rar, etc.)
- `other` - Other file types

### Filter Operators

```javascript
// Equality
filter: { in_use: { eq: true } }

// Inequality
filter: { type: { neq: "video" } }

// Comparison
filter: { size: { gt: 1048576 } }  // > 1MB
filter: { size: { lte: 10485760 } } // <= 10MB

// Array operations
filter: { tags: { any_in: ["banner", "hero"] } }
filter: { tags: { all_in: ["approved", "published"] } }

// Date filtering
filter: { 
  created_at: { 
    gte: "2024-01-01T00:00:00Z",
    lt: "2024-02-01T00:00:00Z"
  } 
}

// Existence
filter: { alt: { exists: true } }
```

## Common Use Cases

### 1. Media Cleanup Filters

```javascript
// Find unused uploads
const unusedFilter = await client.uploadFilters.create({
  name: "Unused Media",
  filter: {
    in_use: { eq: false }
  },
  shared: true
});

// Find old unused uploads
const oldUnusedFilter = await client.uploadFilters.create({
  name: "Old Unused Media (6+ months)",
  filter: {
    in_use: { eq: false },
    updated_at: { 
      lt: new Date(Date.now() - 180 * 24 * 60 * 60 * 1000).toISOString()
    }
  },
  shared: true
});

// Find large unused files
const largeUnusedFilter = await client.uploadFilters.create({
  name: "Large Unused Files (>50MB)",
  filter: {
    in_use: { eq: false },
    size: { gt: 52428800 } // 50MB
  },
  shared: true
});
```

### 2. Content Type Filters

```javascript
// Image library filters
const imageFilters = [
  {
    name: "High-res Images",
    filter: {
      type: { eq: "image" },
      width: { gte: 1920 }
    }
  },
  {
    name: "PNG Images",
    filter: {
      format: { eq: "png" }
    }
  },
  {
    name: "Small Images (<500KB)",
    filter: {
      type: { eq: "image" },
      size: { lt: 512000 }
    }
  }
];

for (const filterDef of imageFilters) {
  await client.uploadFilters.create({
    ...filterDef,
    shared: true
  });
}

// Document filters
const documentFilter = await client.uploadFilters.create({
  name: "PDF Documents",
  filter: {
    format: { eq: "pdf" }
  },
  shared: true
});

// Video content
const videoFilter = await client.uploadFilters.create({
  name: "Video Files",
  filter: {
    type: { eq: "video" }
  },
  shared: true
});
```

### 3. Tag-based Organization

```javascript
// Marketing assets
const marketingFilter = await client.uploadFilters.create({
  name: "Marketing Assets",
  filter: {
    tags: { any_in: ["marketing", "campaign", "social"] }
  },
  shared: true
});

// Approved content
const approvedFilter = await client.uploadFilters.create({
  name: "Approved Media",
  filter: {
    tags: { all_in: ["approved", "reviewed"] }
  },
  shared: true
});

// Product images
const productImagesFilter = await client.uploadFilters.create({
  name: "Product Images",
  filter: {
    type: { eq: "image" },
    tags: { any_in: ["product", "catalog", "ecommerce"] }
  },
  shared: true
});
```

### 4. Collection-based Filters

```javascript
// First, get upload collections
const collections = await client.uploadCollections.list();

// Create filters for each collection
for (const collection of collections) {
  await client.uploadFilters.create({
    name: `Collection: ${collection.name}`,
    filter: {
      upload_collection: { 
        eq: collection.id
      }
    },
    shared: true
  });
}
```

### 5. Author/Team Filters

```javascript
// My uploads
const myUploadsFilter = await client.uploadFilters.create({
  name: "My Uploads",
  filter: {
    author: { 
      eq: currentUserId
    }
  },
  shared: false // Personal filter
});

// Team uploads (last 30 days)
const recentTeamUploads = await client.uploadFilters.create({
  name: "Recent Team Uploads",
  filter: {
    created_at: {
      gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString()
    }
  },
  shared: true
});
```

### 6. Complex Filtering Scenarios

```javascript
// Missing metadata
const missingMetadataFilter = await client.uploadFilters.create({
  name: "Images Missing Alt Text",
  filter: {
    type: { eq: "image" },
    alt: { exists: false }
  },
  shared: true
});

// Multi-criteria filter
const priorityContentFilter = await client.uploadFilters.create({
  name: "Priority Marketing Images",
  filter: {
    type: { eq: "image" },
    tags: { any_in: ["priority", "featured"] },
    in_use: { eq: true },
    size: { lte: 5242880 }, // <= 5MB for web performance
    created_at: {
      gte: new Date(Date.now() - 90 * 24 * 60 * 60 * 1000).toISOString()
    }
  },
  shared: true
});

// Format-specific with size constraints
const webOptimizedImages = await client.uploadFilters.create({
  name: "Web-Optimized Images",
  filter: {
    or: [
      {
        format: { eq: "webp" }
      },
      {
        format: { eq: "jpg" },
        size: { lte: 1048576 } // <= 1MB
      },
      {
        format: { eq: "png" },
        size: { lte: 512000 } // <= 500KB
      }
    ]
  },
  shared: true
});
```

## Filter Management

### Bulk Filter Creation

```javascript
async function setupDefaultUploadFilters(client) {
  const defaultFilters = [
    {
      name: "All Images",
      filter: { type: { eq: "image" } }
    },
    {
      name: "All Videos",
      filter: { type: { eq: "video" } }
    },
    {
      name: "All Documents",
      filter: { type: { eq: "document" } }
    },
    {
      name: "In Use",
      filter: { in_use: { eq: true } }
    },
    {
      name: "Not In Use",
      filter: { in_use: { eq: false } }
    },
    {
      name: "Recent Uploads (7 days)",
      filter: {
        created_at: {
          gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString()
        }
      }
    }
  ];
  
  const createdFilters = [];
  
  for (const filterDef of defaultFilters) {
    try {
      const filter = await client.uploadFilters.create({
        ...filterDef,
        shared: true
      });
      createdFilters.push(filter);
      console.log(`Created filter: ${filter.name}`);
    } catch (error) {
      console.error(`Failed to create filter ${filterDef.name}:`, error);
    }
  }
  
  return createdFilters;
}
```

### Filter Maintenance

```javascript
// Update all personal filters to shared
async function shareAllFilters(client) {
  const filters = await client.uploadFilters.list();
  const personalFilters = filters.filter(f => !f.shared);
  
  for (const filter of personalFilters) {
    await client.uploadFilters.update(filter.id, {
      shared: true
    });
    console.log(`Shared filter: ${filter.name}`);
  }
}

// Clean up duplicate filters
async function removeDuplicateFilters(client) {
  const filters = await client.uploadFilters.list();
  const seen = new Map();
  const duplicates = [];
  
  for (const filter of filters) {
    const key = JSON.stringify(filter.filter);
    
    if (seen.has(key)) {
      duplicates.push(filter);
    } else {
      seen.set(key, filter);
    }
  }
  
  for (const duplicate of duplicates) {
    await client.uploadFilters.destroy(duplicate.id);
    console.log(`Removed duplicate: ${duplicate.name}`);
  }
  
  return duplicates.length;
}
```

### Filter Templates

```javascript
const filterTemplates = {
  imageQuality: {
    hd: {
      name: 'HD Images',
      filter: {
        type: { eq: 'image' },
        width: { gte: 1920 },
        height: { gte: 1080 }
      }
    },
    '4k': {
      name: '4K Images',
      filter: {
        type: { eq: 'image' },
        width: { gte: 3840 },
        height: { gte: 2160 }
      }
    },
    thumbnail: {
      name: 'Thumbnails',
      filter: {
        type: { eq: 'image' },
        width: { lte: 500 },
        height: { lte: 500 }
      }
    }
  },
  
  fileTypes: {
    images: {
      name: 'Web Images',
      filter: {
        type: { eq: 'image' },
        format: { in: ['jpg', 'png', 'webp', 'gif'] }
      }
    },
    documents: {
      name: 'Office Documents',
      filter: {
        mime_type: { 
          in: [
            'application/pdf',
            'application/msword',
            'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
            'application/vnd.ms-excel',
            'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
          ]
        }
      }
    },
    videos: {
      name: 'Video Files',
      filter: {
        type: { eq: 'video' },
        format: { in: ['mp4', 'webm', 'mov'] }
      }
    }
  },
  
  maintenance: {
    unused: {
      name: 'Unused Assets (30+ days)',
      filter: {
        in_use: { eq: false },
        created_at: { 
          lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString() 
        }
      }
    },
    large: {
      name: 'Large Unused Files',
      filter: {
        in_use: { eq: false },
        size: { gt: 10000000 }
      }
    },
    untagged: {
      name: 'Untagged Assets',
      filter: {
        tags: { exists: false }
      }
    }
  }
};

// Apply template
async function createFilterFromTemplate(category, template) {
  const filterData = filterTemplates[category][template];
  if (!filterData) {
    throw new Error(`Template ${category}.${template} not found`);
  }
  
  return client.uploadFilters.create({
    ...filterData,
    shared: true
  });
}
```

### Filter Analysis

```javascript
async function analyzeFilterUsage() {
  const filters = await client.uploadFilters.list();
  
  const analysis = {
    total: filters.length,
    shared: 0,
    private: 0,
    byCreator: new Map(),
    mostRestrictive: null,
    leastRestrictive: null
  };
  
  for (const filter of filters) {
    // Count shared vs private
    if (filter.shared) {
      analysis.shared++;
    } else {
      analysis.private++;
    }
    
    // Count filter conditions
    const conditionCount = Object.keys(filter.filter || {}).length;
    
    if (!analysis.mostRestrictive || conditionCount > analysis.mostRestrictive.conditions) {
      analysis.mostRestrictive = {
        filter,
        conditions: conditionCount
      };
    }
    
    if (!analysis.leastRestrictive || conditionCount < analysis.leastRestrictive.conditions) {
      analysis.leastRestrictive = {
        filter,
        conditions: conditionCount
      };
    }
  }
  
  return analysis;
}
```

## Best Practices

### Naming Conventions
1. Use clear, descriptive names
2. Include criteria in the name when helpful
3. Use prefixes for grouping (e.g., "Type: Images", "Tag: Marketing")

### Performance
1. Avoid overly complex filters that might slow queries
2. Use specific criteria to narrow results effectively
3. Combine filters logically to reduce processing

### Team Collaboration
1. Share filters that benefit the whole team
2. Keep personal experimental filters private
3. Document complex filter logic

## Differences from ItemTypeFilter

| Feature | UploadFilter | ItemTypeFilter |
|---------|--------------|----------------|
| **Scope** | All uploads | Specific content model |
| **Columns** | Not configurable | Configurable |
| **Sorting** | Not configurable | Configurable |
| **Linked to** | Nothing | Specific item type |
| **Filter fields** | Upload-specific | Model field-specific |

## Error Handling

```javascript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.uploadFilters.create({
    name: "Test Filter",
    filter: { invalid_field: { eq: true } }
  });
} catch (error) {
  if (error instanceof ApiError) {
    if (error.message.includes('INVALID_FIELD')) {
      console.error('Invalid filter field specified');
    } else if (error.message.includes('VALIDATION_ERROR')) {
      console.error('Filter validation failed:', error.details);
    }
  }
}

// Comprehensive error handling
async function safeFilterOperation() {
  try {
    const filter = await client.uploadFilters.create({
      name: 'Test Filter',
      filter: { type: { eq: 'image' } },
      shared: true
    });
    return filter;
  } catch (error) {
    if (error.response) {
      switch (error.response.status) {
        case 422:
          const validationErrors = error.response.data.errors;
          validationErrors.forEach(err => {
            if (err.source?.pointer?.includes('name')) {
              console.error('Name error:', err.detail);
            } else if (err.source?.pointer?.includes('filter')) {
              console.error('Query error:', err.detail);
            }
          });
          break;
        case 403:
          console.error('Insufficient permissions to create filters');
          break;
        case 409:
          console.error('Filter with this name already exists');
          break;
        default:
          console.error('Unexpected error:', error.response.status);
      }
    }
    throw error;
  }
}
```

## The UploadFilter Object

```typescript
{
  id: string;                  // Filter UUID
  type: 'upload_filter';       // Always 'upload_filter'
  name: string;                // Display name
  filter: {                    // Filter criteria
    [k: string]: unknown;
  };
  shared: boolean;             // Shared with team
}
```

## Related Resources

- [Uploads](./upload.md) - Upload management
- [Upload Collections](./upload-collection.md) - Organize uploads
- [Upload Tags](./upload-tag.md) - Tag management
- [ItemTypeFilter](../01-content-management/item-type-filter.md) - Content filters