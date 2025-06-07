# Upload Collection Resource

## Overview

Upload Collections allow you to organize your media assets into a hierarchical folder structure, making it easier to manage large media libraries. Collections work like folders in a traditional file system.

## API Client Section

`client.uploadCollections`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a top-level collection
const marketingAssets = await client.uploadCollections.create({
  label: 'Marketing Assets',
  position: 1
});

// Create a sub-collection
const socialMedia = await client.uploadCollections.create({
  label: 'Social Media',
  parent: { type: 'upload_collection', id: marketingAssets.id }
});

// List all collections
const collections = await client.uploadCollections.list();

// Update a collection
const updated = await client.uploadCollections.update('collection-id', {
  label: 'Product Photography'
});

// Delete a collection
await client.uploadCollections.destroy('collection-id');
```

## API Reference

### list() - List Upload Collections

Retrieve all collections in your project with optional filtering.

**Signature**: `list(queryParams): Promise<UploadCollection[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `filter` | object | Filtering options |
| `filter.ids` | string | Comma-separated collection IDs |
| `page` | object | Pagination options |
| `page.offset` | number | Starting offset (default: 0) |
| `page.limit` | number | Number of items (default: 30) |
| `orderBy` | string | Sort order (e.g., 'name_ASC') |

**Basic Example**:
```typescript
// Get all collections
const collections = await client.uploadCollections.list();

// Get specific collections by IDs
const specificCollections = await client.uploadCollections.list({
  filter: { ids: 'id1,id2,id3' }
});

// With pagination
const paginatedCollections = await client.uploadCollections.list({
  page: { offset: 0, limit: 30 }
});
```

**Advanced Example (Building Tree Structure)**:
```typescript
// Build tree structure
const allCollections = await client.uploadCollections.list();
const rootCollections = allCollections.filter(c => !c.parent);

const buildTree = (parentId: string | null) => {
  return allCollections
    .filter(c => c.parent?.id === parentId)
    .sort((a, b) => a.position - b.position);
};

// Get raw response with metadata
const rawResponse = await client.uploadCollections.rawList({
  page: { limit: 20 }
});
// rawResponse structure:
// {
//   data: UploadCollection[],
//   meta: {
//     total_count: number
//   }
// }
```

### find() - Get Single Upload Collection

Get a specific collection by ID.

**Signature**: `find(uploadCollectionId): Promise<UploadCollection>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadCollectionId` | string | Collection ID (required) |

**Example**:
```typescript
const collection = await client.uploadCollections.find('collection-id');

console.log(`Collection: ${collection.label}`);
console.log(`Parent: ${collection.parent?.id || 'root'}`);
console.log(`Children: ${collection.children.length}`);

// Collection structure includes:
// {
//   id: 'collection_id_123',
//   type: 'upload_collection',
//   label: 'Product Images',
//   position: 1,
//   parent?: {
//     type: 'upload_collection',
//     id: 'parent_collection_id'
//   },
//   children: [
//     { type: 'upload_collection', id: 'child_collection_1' },
//     { type: 'upload_collection', id: 'child_collection_2' }
//   ]
// }

// Error handling
try {
  const collection = await client.uploadCollections.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Collection not found');
  }
}
```

### create() - Create Upload Collection

Create a new collection to organize uploads.

**Signature**: `create(body): Promise<UploadCollection>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | string | Collection name (required) |
| `position` | number | Order within parent (optional) |
| `parent` | object | Parent collection reference (optional) |

**Basic Example**:
```typescript
// Create root-level collection
const productsCollection = await client.uploadCollections.create({
  label: 'Product Images',
  position: 1
});

// Create nested collection
const summerProducts = await client.uploadCollections.create({
  label: 'Summer 2024',
  position: 1,
  parent: { type: 'upload_collection', id: productsCollection.id }
});
```

**Advanced Example (Creating Nested Structure)**:
```typescript
// Create deeply nested structure
const categories = await client.uploadCollections.create({
  label: 'Categories',
  parent: { type: 'upload_collection', id: summerProducts.id }
});

// Create nested collection structure
const parentCollection = await client.uploadCollections.create({
  label: 'Media Library',
  position: 1
});

const childCollection = await client.uploadCollections.create({
  label: 'Videos',
  parent: { 
    type: 'upload_collection', 
    id: parentCollection.id 
  }
});

// Error handling
try {
  const collection = await client.uploadCollections.create({
    label: '' // Empty name not allowed
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation error:', error.response.data.errors);
  }
}
```

### update() - Update Upload Collection

Rename or reorganize existing collections.

**Signature**: `update(uploadCollectionId, body): Promise<UploadCollection>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadCollectionId` | string | Collection ID (required) |
| `label` | string | New name (optional) |
| `position` | number | New order position (optional) |
| `parent` | object | New parent or null (optional) |
| `children` | array | Reorder child collections (optional) |

**Basic Example**:
```typescript
// Rename collection
await client.uploadCollections.update('collection-id', {
  label: 'Product Photography'
});

// Move to different parent
await client.uploadCollections.update('collection-id', {
  parent: { type: 'upload_collection', id: 'new-parent-id' },
  position: 2
});
```

**Advanced Example (Reorganizing Collections)**:
```typescript
// Move to root level
await client.uploadCollections.update('collection-id', {
  parent: null,
  position: 1
});

// Reorder children
await client.uploadCollections.update('parent-id', {
  children: [
    { type: 'upload_collection', id: 'child-1' },
    { type: 'upload_collection', id: 'child-2' },
    { type: 'upload_collection', id: 'child-3' }
  ]
});

// Error handling
try {
  const updated = await client.uploadCollections.update('collection_id', {
    label: '' // Empty name not allowed
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Name cannot be empty');
  }
}
```

### destroy() - Delete Upload Collection

Remove a collection. Uploads in the collection are not deleted but moved to parent or root.

**Signature**: `destroy(uploadCollectionId): Promise<void>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadCollectionId` | string | Collection ID (required) |

**Example**:
```typescript
// Delete collection (uploads are moved to parent or root)
await client.uploadCollections.destroy('collection-id');

// Delete collection and check for uploads first
const uploads = await client.uploads.list({
  filter: { upload_collection: 'collection-id' }
});

if (uploads.length > 0) {
  console.log(`Collection contains ${uploads.length} uploads`);
  // Optionally move uploads before deletion
}

await client.uploadCollections.destroy('collection-id');

// Safe delete with content check
async function safeDeleteCollection(collectionId) {
  const collection = await client.uploadCollections.find(collectionId);
  
  if (collection.meta?.uploads_count > 0) {
    console.warn(`Collection has ${collection.meta.uploads_count} uploads`);
    return false;
  }
  
  if (collection.children.length > 0) {
    console.warn('Collection has child collections');
    return false;
  }
  
  await client.uploadCollections.destroy(collectionId);
  return true;
}
```

## Collection Organization Patterns

### Department-Based Structure

Organize assets by department or team:

```typescript
async function createDepartmentStructure() {
  const departments = [
    { name: 'Marketing', subfolders: ['Campaigns', 'Social Media', 'Print'] },
    { name: 'Product', subfolders: ['Photos', 'Videos', '360 Views'] },
    { name: 'Content', subfolders: ['Blog', 'Tutorials', 'Infographics'] },
    { name: 'Brand', subfolders: ['Logos', 'Colors', 'Templates'] }
  ];
  
  for (const dept of departments) {
    // Create department folder
    const deptCollection = await client.uploadCollections.create({
      label: dept.name
    });
    
    // Create subfolders
    for (let i = 0; i < dept.subfolders.length; i++) {
      await client.uploadCollections.create({
        label: dept.subfolders[i],
        position: i + 1,
        parent: { type: 'upload_collection', id: deptCollection.id }
      });
    }
  }
}
```

### Date-Based Archives

Create time-based organization:

```typescript
async function createDateBasedStructure() {
  const currentYear = new Date().getFullYear();
  
  // Create year folders for last 3 years
  for (let year = currentYear; year >= currentYear - 2; year--) {
    const yearCollection = await client.uploadCollections.create({
      label: year.toString(),
      position: currentYear - year + 1
    });
    
    // Create month folders
    const months = [
      'January', 'February', 'March', 'April', 'May', 'June',
      'July', 'August', 'September', 'October', 'November', 'December'
    ];
    
    for (let i = 0; i < months.length; i++) {
      await client.uploadCollections.create({
        label: months[i],
        position: i + 1,
        parent: { type: 'upload_collection', id: yearCollection.id }
      });
    }
  }
}
```

### Project-Based Collections

Organize by project or campaign:

```typescript
async function createProjectCollection(projectName: string, structure: any) {
  // Create main project folder
  const project = await client.uploadCollections.create({
    label: projectName
  });
  
  // Standard project structure
  const folders = [
    { label: 'Assets', children: ['Raw', 'Processed', 'Final'] },
    { label: 'Documentation', children: ['Briefs', 'Guidelines'] },
    { label: 'Deliverables', children: ['Web', 'Print', 'Social'] }
  ];
  
  for (const folder of folders) {
    const parent = await client.uploadCollections.create({
      label: folder.label,
      parent: { type: 'upload_collection', id: project.id }
    });
    
    // Create subfolders
    for (const child of folder.children) {
      await client.uploadCollections.create({
        label: child,
        parent: { type: 'upload_collection', id: parent.id }
      });
    }
  }
  
  return project;
}
```

## Working with Uploads and Collections

### Upload to Collection

Upload files directly to a collection:

```typescript
// Upload to specific collection
const upload = await client.uploads.createFromLocalFile({
  localPath: './product-photo.jpg',
  upload_collection: {
    type: 'upload_collection',
    id: 'collection-id'
  },
  tags: ['product', '2024'],
  default_field_metadata: {
    en: {
      alt: 'Product photo',
      title: 'Summer collection item'
    }
  }
});
```

### Move Uploads Between Collections

Bulk move uploads to different collections:

```typescript
// Move single upload
await client.uploads.update('upload-id', {
  upload_collection: {
    type: 'upload_collection',
    id: 'new-collection-id'
  }
});

// Bulk move uploads
const uploads = await client.uploads.list({
  filter: { 
    tags: { any_in: ['archive'] }
  }
});

const archiveCollection = await client.uploadCollections.create({
  label: 'Archive'
});

await client.uploads.bulkSetUploadCollection({
  uploads: uploads.map(u => ({ type: 'upload', id: u.id })),
  upload_collection: {
    type: 'upload_collection',
    id: archiveCollection.id
  }
});
```

### Collection Statistics

Get insights about collection usage:

```typescript
async function getCollectionStats(collectionId: string) {
  // Get uploads in collection
  const uploads = await client.uploads.list({
    filter: { upload_collection: collectionId }
  });
  
  // Calculate statistics
  const stats = {
    totalFiles: uploads.length,
    totalSize: uploads.reduce((sum, u) => sum + u.size, 0),
    fileTypes: {},
    avgFileSize: 0
  };
  
  // Count file types
  uploads.forEach(upload => {
    const ext = upload.format || 'unknown';
    stats.fileTypes[ext] = (stats.fileTypes[ext] || 0) + 1;
  });
  
  stats.avgFileSize = stats.totalFiles > 0 
    ? Math.round(stats.totalSize / stats.totalFiles) 
    : 0;
  
  // Get subcollections
  const allCollections = await client.uploadCollections.list();
  const subCollections = allCollections.filter(c => 
    c.parent?.id === collectionId
  );
  
  return {
    ...stats,
    subCollections: subCollections.length,
    formattedSize: formatBytes(stats.totalSize)
  };
}

function formatBytes(bytes: number): string {
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  if (bytes === 0) return '0 Bytes';
  const i = Math.floor(Math.log(bytes) / Math.log(1024));
  return Math.round(bytes / Math.pow(1024, i) * 100) / 100 + ' ' + sizes[i];
}
```

## Advanced Usage

### Collection Templates

Create reusable collection structures:

```typescript
class CollectionTemplates {
  static async createWebsiteAssets(client: any) {
    const website = await client.uploadCollections.create({
      label: 'Website Assets'
    });
    
    const structure = {
      'Images': ['Hero', 'Products', 'Blog', 'Team'],
      'Videos': ['Background', 'Tutorials', 'Testimonials'],
      'Documents': ['PDFs', 'Downloads', 'Forms'],
      'Icons': ['SVG', 'PNG', 'Favicons']
    };
    
    for (const [main, subs] of Object.entries(structure)) {
      const mainCollection = await client.uploadCollections.create({
        label: main,
        parent: { type: 'upload_collection', id: website.id }
      });
      
      for (const sub of subs) {
        await client.uploadCollections.create({
          label: sub,
          parent: { type: 'upload_collection', id: mainCollection.id }
        });
      }
    }
    
    return website;
  }
  
  static async createEcommerceAssets(client: any) {
    const ecommerce = await client.uploadCollections.create({
      label: 'E-commerce'
    });
    
    const categories = ['Electronics', 'Clothing', 'Home', 'Sports'];
    for (const category of categories) {
      const cat = await client.uploadCollections.create({
        label: category,
        parent: { type: 'upload_collection', id: ecommerce.id }
      });
      
      // Add standard subfolders
      for (const type of ['Products', 'Lifestyle', 'Details']) {
        await client.uploadCollections.create({
          label: type,
          parent: { type: 'upload_collection', id: cat.id }
        });
      }
    }
    
    return ecommerce;
  }
}
```

### Collection Migration

Move entire collection structures:

```typescript
async function migrateCollection(
  sourceId: string, 
  targetParentId: string | null
) {
  const allCollections = await client.uploadCollections.list();
  
  // Get source collection and all descendants
  const source = await client.uploadCollections.find(sourceId);
  const descendants = [];
  
  const getDescendants = (parentId: string) => {
    const children = allCollections.filter(c => c.parent?.id === parentId);
    children.forEach(child => {
      descendants.push(child);
      getDescendants(child.id);
    });
  };
  
  getDescendants(sourceId);
  
  // Move source to new parent
  await client.uploadCollections.update(sourceId, {
    parent: targetParentId 
      ? { type: 'upload_collection', id: targetParentId }
      : null
  });
  
  console.log(`Migrated ${source.label} and ${descendants.length} sub-collections`);
}
```

### Collection Cleanup

Remove empty collections:

```typescript
async function cleanupEmptyCollections() {
  const collections = await client.uploadCollections.list();
  const emptyCollections = [];
  
  // Check each collection for uploads
  for (const collection of collections) {
    const uploads = await client.uploads.list({
      filter: { upload_collection: collection.id },
      page: { limit: 1 }
    });
    
    if (uploads.length === 0) {
      // Check if it has child collections
      const hasChildren = collections.some(c => c.parent?.id === collection.id);
      if (!hasChildren) {
        emptyCollections.push(collection);
      }
    }
  }
  
  // Delete empty collections (from deepest level up)
  emptyCollections.sort((a, b) => {
    const aDepth = getDepth(a, collections);
    const bDepth = getDepth(b, collections);
    return bDepth - aDepth;
  });
  
  for (const collection of emptyCollections) {
    console.log(`Deleting empty collection: ${collection.label}`);
    await client.uploadCollections.destroy(collection.id);
  }
  
  return emptyCollections.length;
}

function getDepth(collection: any, allCollections: any[]): number {
  let depth = 0;
  let current = collection;
  
  while (current.parent) {
    depth++;
    current = allCollections.find(c => c.id === current.parent.id);
  }
  
  return depth;
}
```

### Collection Search

Find collections by name or description:

```typescript
async function searchCollections(query) {
  const allCollections = await client.uploadCollections.list();
  
  // Search in name and description
  const results = allCollections.filter(collection => {
    const nameMatch = collection.label.toLowerCase().includes(query.toLowerCase());
    return nameMatch;
  });
  
  // Enhance results with path and upload count
  const enhancedResults = await Promise.all(
    results.map(async (collection) => {
      const path = await getCollectionPath(collection.id);
      
      const uploads = await client.uploads.list({
        filter: {
          upload_collection: collection.id
        },
        page: { limit: 1 }
      });
      
      return {
        ...collection,
        path: path.map(c => c.label).join(' > '),
        uploadCount: uploads.length,
        hasChildren: collection.children.length > 0
      };
    })
  );
  
  return enhancedResults.sort((a, b) => b.uploadCount - a.uploadCount);
}

// Get full collection path
async function getCollectionPath(collectionId) {
  const path = [];
  let currentId = collectionId;
  
  while (currentId) {
    const collection = await client.uploadCollections.find(currentId);
    path.unshift(collection);
    currentId = collection.parent?.id;
  }
  
  return path;
}
```

### Collection Permissions Setup

Organize collections for different teams:

```typescript
async function setupCollectionPermissions() {
  // Note: Permissions are managed at the role level
  // This example shows how to organize collections for different teams
  
  const departments = [
    { name: 'Marketing Team', icon: '📢' },
    { name: 'Product Team', icon: '🛍️' },
    { name: 'Engineering Team', icon: '⚙️' },
    { name: 'HR Team', icon: '👥' }
  ];
  
  const collections = await Promise.all(
    departments.map(dept =>
      client.uploadCollections.create({
        label: dept.name
      })
    )
  );
  
  // Create shared collection
  const sharedCollection = await client.uploadCollections.create({
    label: 'Shared Resources'
  });
  
  return {
    departmentCollections: collections,
    shared: sharedCollection
  };
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.uploadCollections.create({
    label: '', // Invalid: empty label
    parent: { type: 'upload_collection', id: 'invalid-id' }
  });
} catch (error) {
  if (error instanceof ApiError) {
    const labelError = error.findError('label');
    if (labelError?.code === 'VALIDATION_REQUIRED') {
      console.log('Collection label is required');
    }
    
    const parentError = error.findError('parent');
    if (parentError?.code === 'VALIDATION_INVALID') {
      console.log('Invalid parent collection');
    }
  }
}

// Comprehensive error handling
async function safeCollectionOperation() {
  try {
    const collection = await client.uploadCollections.create({
      label: 'New Collection',
      parent: { type: 'upload_collection', id: 'parent_id' }
    });
    return collection;
  } catch (error) {
    if (error.response) {
      switch (error.response.status) {
        case 422:
          const validationErrors = error.response.data.errors;
          validationErrors.forEach(err => {
            if (err.source?.pointer?.includes('label')) {
              console.error('Name validation failed:', err.detail);
            } else if (err.source?.pointer?.includes('parent')) {
              console.error('Invalid parent collection:', err.detail);
            }
          });
          break;
        case 404:
          console.error('Parent collection not found');
          break;
        case 403:
          console.error('Insufficient permissions');
          break;
        default:
          console.error('Unexpected error:', error.response.status);
      }
    }
    throw error;
  }
}
```

## Best Practices

1. **Hierarchical Structure**: Organize collections in a logical hierarchy that matches your content structure
2. **Naming Convention**: Use clear, descriptive names that indicate the collection's purpose
3. **Regular Maintenance**: Periodically review and clean up empty or unused collections
4. **Permissions Planning**: Structure collections with access control in mind
5. **Avoid Deep Nesting**: Keep hierarchy levels manageable (3-4 levels max)

## Upload Collection Object Structure

```typescript
interface UploadCollection {
  id: string;
  type: 'upload_collection';
  label: string;
  position: number;
  parent?: {
    type: 'upload_collection';
    id: string;
  };
  children: Array<{
    type: 'upload_collection';
    id: string;
  }>;
}
```

## Related Resources

- [Uploads](./upload.md) - Upload files to collections
- [Upload Tags](./upload-tag.md) - Alternative organization method
- [Roles](../05-access-control/role.md) - Control collection management permissions