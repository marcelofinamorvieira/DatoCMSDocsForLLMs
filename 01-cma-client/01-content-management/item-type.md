# Item Type Resource

## Overview

Item Types (also called Models) define the structure and behavior of content in your DatoCMS project. They act as templates that specify which fields are available for content editors when creating items. Models are the foundation of your content structure, defining not just what fields are available but also how content behaves - whether it's a singleton page, a hierarchical tree structure, or reusable modular blocks.

## API Client Section

`client.itemTypes`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a new model
const itemType = await client.itemTypes.create({
  name: 'Blog Post',
  api_key: 'blog_post',
  singleton: false
});

// Fetch a single model
const model = await client.itemTypes.find('blog_post');

// Update a model
const updated = await client.itemTypes.update('blog_post', {
  hint: 'Blog articles and news posts'
});

// Delete a model
await client.itemTypes.destroy('blog_post');
```

## API Reference

### list() - List Item Types

Retrieve a list of item types (models) from your DatoCMS project.

**Signature**: `list(queryParams): Promise<ItemType[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `filter` | object | Filtering options |
| `filter.singleton` | boolean | Filter by singleton status |
| `filter.modular_block` | boolean | Filter by modular block status |
| `filter.tree` | boolean | Filter by tree status |
| `filter.sortable` | boolean | Filter by sortable status |
| `filter.draft_mode_active` | boolean | Filter by draft mode status |
| `filter.name` | object | Filter by name pattern |
| `orderBy` | string | Sorting criteria (e.g., 'name_ASC', '_created_at_DESC') |

**Basic Example**:
```javascript
// Get all item types
const allModels = await client.itemTypes.list();

// Get item types with filters
const sortableModels = await client.itemTypes.list({
  filter: { sortable: true }
});

// Get item types with ordering
const orderedModels = await client.itemTypes.list({
  orderBy: 'name_ASC'
});

// Get raw response with metadata
const response = await client.itemTypes.rawList();
// response.meta contains pagination info
```

**Advanced Example (Complex Filtering)**:
```javascript
// Filter by type
const singletons = await client.itemTypes.list({
  filter: { singleton: true }
});

const modularBlocks = await client.itemTypes.list({
  filter: { modular_block: true }
});

const treeModels = await client.itemTypes.list({
  filter: { tree: true }
});

// Filter by features
const sortableModels = await client.itemTypes.list({
  filter: { sortable: true }
});

const draftEnabledModels = await client.itemTypes.list({
  filter: { draft_mode_active: true }
});

// Filter by name pattern
const newsModels = await client.itemTypes.list({
  filter: {
    name: { _matches: 'News' }
  }
});

// Combined filters
const complexFilter = await client.itemTypes.list({
  filter: {
    singleton: false,
    modular_block: false,
    draft_mode_active: true
  }
});
```

### find() - Get Single Item Type

Retrieve a single item type by ID or API key.

**Signature**: `find(itemTypeId): Promise<ItemType>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID or api_key |

**Example**:
```javascript
// Get by ID
const articleModel = await client.itemTypes.find('article_model_id');

// Get by API key
const blogPost = await client.itemTypes.find('blog_post');

// Error handling
try {
  const model = await client.itemTypes.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Model not found');
  }
}
```

### create() - Create New Item Type

Create a new item type (model) in your project.

**Signature**: `create(body): Promise<ItemType>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Human-readable name (required) |
| `api_key` | string | API identifier (required) |
| `singleton` | boolean | Single instance model (default: false) |
| `sortable` | boolean | Allow manual sorting (default: false) |
| `tree` | boolean | Enable tree hierarchy (default: false) |
| `modular_block` | boolean | Use as block in modular content (default: false) |
| `draft_mode_active` | boolean | Enable draft/published system (default: false) |
| `draft_saving_active` | boolean | Auto-save drafts (default: true when draft_mode_active is true) |
| `all_locales_required` | boolean | Require all locales (default: false) |
| `collection_appearance` | string | How to display in UI ('table', 'grid', 'compact') |
| `hint` | string | Help text for editors |
| `inverse_relationships_enabled` | boolean | Show reverse links (default: false) |
| `ordering_field` | object | Default ordering field |
| `ordering_direction` | string | Default sort order ('asc' or 'desc') |
| `ordering_meta` | string | Sort by metadata field |

**Basic Example**:
```javascript
// Create basic model
const newModel = await client.itemTypes.create({
  name: 'Article',
  api_key: 'article',
  singleton: false,
  modular_block: false,
  tree: false,
  sortable: true,
  draft_mode_active: true,
  all_locales_required: false,
  collection_appearance: 'table',
  hint: 'Blog articles and news posts'
});

// Create singleton model (single instance)
const homepageModel = await client.itemTypes.create({
  name: 'Homepage',
  api_key: 'homepage',
  singleton: true,
  sortable: false,
  draft_mode_active: true
});

// Create modular block (for use in modular content fields)
const ctaBlock = await client.itemTypes.create({
  name: 'Call to Action',
  api_key: 'cta_block',
  modular_block: true,
  sortable: false
});
```

**Advanced Examples**:
```javascript
// Create tree model (hierarchical)
const pageModel = await client.itemTypes.create({
  name: 'Page',
  api_key: 'page',
  tree: true,
  ordering_field: { 
    type: 'field',
    id: 'position_field_id'
  },
  ordering_direction: 'asc'
});

// Create with custom collection appearance
const productModel = await client.itemTypes.create({
  name: 'Product',
  api_key: 'product',
  collection_appearance: 'compact',
  ordering_meta: 'created_at',
  ordering_direction: 'desc'
});

// Create with field ordering
const modelWithOrdering = await client.itemTypes.create({
  name: 'News',
  api_key: 'news',
  ordering_field: {
    type: 'field',
    id: 'published_at_field_id'
  },
  ordering_direction: 'desc'
});

// Create with inverse relationships disabled
const tagModel = await client.itemTypes.create({
  name: 'Tag',
  api_key: 'tag',
  inverse_relationships_enabled: false
});

// Error handling
try {
  const model = await client.itemTypes.create({
    name: 'Invalid Model',
    api_key: 'invalid-key!' // Invalid characters
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation error:', error.response.data.errors);
  }
}
```

### update() - Update Item Type

Update an existing item type.

**Signature**: `update(itemTypeId, body): Promise<ItemType>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID (required) |
| `name` | string | Human-readable name |
| `hint` | string | Help text for editors |
| `draft_mode_active` | boolean | Enable draft/published system |
| `all_locales_required` | boolean | Require all locales |
| `collection_appearance` | string | How to display in UI |
| `ordering_direction` | string | Default sort order |
| `ordering_field` | object | Field to sort by |
| `ordering_meta` | string | Sort by metadata |
| `title_field` | object | Field to use as item title |
| `presentation_title_field` | object | Alternative title for presentation |
| `image_preview_field` | object | Field for thumbnail |
| `presentation_image_field` | object | Alternative image for presentation |
| `excerpt_field` | object | Field for excerpt/preview |
| `workflow` | object | Assign workflow for approvals |
| `tree` | boolean | Enable tree structure |
| `sortable` | boolean | Allow manual sorting |
| `inverse_relationships_enabled` | boolean | Show reverse links |

**Example**:
```javascript
// Update basic properties
const updated = await client.itemTypes.update('model_id', {
  name: 'Updated Article',
  hint: 'Updated description'
});

// Enable/disable features
const withDrafts = await client.itemTypes.update('model_id', {
  draft_mode_active: true,
  all_locales_required: true
});

// Update collection settings
const collectionUpdate = await client.itemTypes.update('model_id', {
  collection_appearance: 'table',
  ordering_meta: 'updated_at',
  ordering_direction: 'desc'
});

// Update ordering field
const orderingUpdate = await client.itemTypes.update('model_id', {
  ordering_field: {
    type: 'field',
    id: 'position_field_id'
  },
  ordering_direction: 'asc'
});

// Update presentation fields
const presentationUpdate = await client.itemTypes.update('model_id', {
  title_field: {
    type: 'field',
    id: 'title_field_id'
  },
  image_preview_field: {
    type: 'field',
    id: 'featured_image_field_id'
  },
  excerpt_field: {
    type: 'field',
    id: 'excerpt_field_id'
  }
});

// Assign workflow
const workflowUpdate = await client.itemTypes.update('model_id', {
  workflow: {
    type: 'workflow',
    id: 'workflow_id'
  }
});

// Enable tree structure
const treeUpdate = await client.itemTypes.update('model_id', {
  tree: true
});

// Error handling
try {
  const updated = await client.itemTypes.update('model_id', {
    api_key: 'new-invalid-key!'
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Cannot change API key with invalid characters');
  }
}
```

### destroy() - Delete Item Type

Delete an item type and all its records.

**Signature**: `destroy(itemTypeId): Promise<void>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID |

**Example**:
```javascript
// Delete item type
await client.itemTypes.destroy('model_id');

// Delete with error handling
try {
  await client.itemTypes.destroy('model_id');
  console.log('Model deleted successfully');
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Cannot delete model with existing records');
  } else if (error.response?.status === 404) {
    console.error('Model not found');
  }
}

// Safe delete pattern (delete all items first)
async function safeDeleteModel(modelId) {
  // First, delete all items of this type
  const items = await client.items.list({
    filter: { type: modelId }
  });
  
  await Promise.all(
    items.map(item => client.items.destroy(item.id))
  );
  
  // Then delete the model
  await client.itemTypes.destroy(modelId);
}
```

**Warning:** This permanently deletes the model and all its content.

### duplicate() - Duplicate Item Type

Create a copy of an existing item type including all its fields.

**Signature**: `duplicate(itemTypeId, body): Promise<ItemType>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID to duplicate |
| `name` | string | Name for the duplicate (optional) |
| `api_key` | string | API key for the duplicate (optional) |
| `duplicate_records` | boolean | Whether to copy records (optional) |

**Example**:
```javascript
// Simple duplicate
const duplicated = await client.itemTypes.duplicate('original_model_id');

// Duplicate with new properties
const customDuplicate = await client.itemTypes.duplicate('original_model_id', {
  name: 'Article Copy',
  api_key: 'article_copy'
});

// Duplicate without records
const structureOnly = await client.itemTypes.duplicate('original_model_id', {
  duplicate_records: false
});
```

## Common Patterns

### Standard Models

Regular content models for creating multiple items:

```javascript
const article = await client.itemTypes.create({
  name: 'Article',
  api_key: 'article',
  singleton: false,
  sortable: true,
  draft_mode_active: true,
  collection_appearance: 'table'
});
```

### Singleton Models

For unique pages or global settings with only one instance:

```javascript
const settings = await client.itemTypes.create({
  name: 'Site Settings',
  api_key: 'site_settings',
  singleton: true,
  hint: 'Global website configuration'
});

// Singleton items are accessed differently
const settingsItem = await client.items.list({
  filter: { type: 'site_settings' }
}).then(items => items[0]);
```

### Tree Models

For hierarchical content like navigation or categories:

```javascript
const category = await client.itemTypes.create({
  name: 'Category',
  api_key: 'category',
  tree: true,
  ordering_direction: 'asc',
  ordering_field: { type: 'field', id: 'name-field-id' }
});

// Items can have parent-child relationships
const childCategory = await client.items.create({
  item_type: { type: 'item_type', id: 'category' },
  name: 'Sub Category',
  parent: { type: 'item', id: 'parent-category-id' }
});
```

### Modular Blocks

Components for use within modular content or structured text fields. Setting `modular_block: true` makes the model available as a block that can be used in Modular Content and Structured Text fields. For comprehensive information about working with modular blocks, see the [Modular Content Guide](./modular-content.md).

```javascript
const heroBlock = await client.itemTypes.create({
  name: 'Hero Section',
  api_key: 'hero_section',
  modular_block: true // This makes it available as a block
});

// Blocks are not directly accessible via items.list()
// They're embedded within other items' modular content fields
```

### Sortable Models

Enable manual ordering with drag-and-drop:

```javascript
const navigationItem = await client.itemTypes.create({
  name: 'Navigation Item',
  api_key: 'navigation_item',
  sortable: true
});

// After adding fields, set the position field
const positionField = await client.fields.create(navigationItem.id, {
  label: 'Position',
  field_type: 'integer',
  api_key: 'position'
});

await client.itemTypes.update(navigationItem.id, {
  ordering_field: {
    type: 'field',
    id: positionField.id
  }
});
```

## Advanced Configuration

### Presentation Settings

Configure how items appear in the CMS interface:

```javascript
await client.itemTypes.update('article', {
  // Set title field for item display
  title_field: {
    type: 'field',
    id: 'headline-field-id'
  },
  
  // Set image for thumbnails
  image_preview_field: {
    type: 'field',
    id: 'featured-image-field-id'
  },
  
  // Set excerpt for previews
  excerpt_field: {
    type: 'field',
    id: 'summary-field-id'
  },
  
  // Grid view with image previews
  collection_appearance: 'grid'
});
```

### Ordering Configuration

Set default sorting for items:

```javascript
// Sort by a specific field
await client.itemTypes.update('product', {
  ordering_direction: 'asc',
  ordering_field: {
    type: 'field',
    id: 'price-field-id'
  }
});

// Sort by metadata
await client.itemTypes.update('blog_post', {
  ordering_direction: 'desc',
  ordering_meta: 'published_at'
});

// Manual sorting (drag & drop)
await client.itemTypes.update('navigation_item', {
  sortable: true,
  ordering_field: {
    type: 'field',
    id: 'position-field-id' // Integer field
  }
});
```

### Draft/Published System

Enable content versioning and publishing workflow:

```javascript
const itemType = await client.itemTypes.create({
  name: 'Page',
  api_key: 'page',
  draft_mode_active: true,
  draft_saving_active: true // Auto-save drafts
});

// Items now have draft and published versions
const item = await client.items.create({
  item_type: { type: 'item_type', id: 'page' },
  title: 'Draft Page'
});

// Publish when ready
await client.items.publish(item.id);
```

### Workflow Integration

Assign approval workflows to models:

```javascript
// First create a workflow
const workflow = await client.workflows.create({
  name: 'Editorial Review',
  stages: [
    { name: 'Draft', color: '#999999' },
    { name: 'Review', color: '#FFA500' },
    { name: 'Approved', color: '#00FF00' }
  ]
});

// Assign to item type
await client.itemTypes.update('article', {
  workflow: {
    type: 'workflow',
    id: workflow.id
  }
});
```

### Localization Settings

Configure locale requirements:

```javascript
await client.itemTypes.create({
  name: 'Product',
  api_key: 'product',
  all_locales_required: true // All locales must be filled
});

// With all_locales_required: true, items cannot be saved
// unless all fields have values for all enabled locales
```

### Inverse Relationships

Show items that link to the current item:

```javascript
await client.itemTypes.update('author', {
  inverse_relationships_enabled: true
});

// Now when viewing an author, you'll see all articles
// that link to this author via their "author" field
```

## Working with Fields

After creating an item type, add fields to define its structure:

```javascript
// Create the model
const itemType = await client.itemTypes.create({
  name: 'Product',
  api_key: 'product'
});

// Add fields
const titleField = await client.fields.create(itemType.id, {
  label: 'Product Name',
  field_type: 'string',
  api_key: 'name',
  validators: { required: {} }
});

const priceField = await client.fields.create(itemType.id, {
  label: 'Price',
  field_type: 'float',
  api_key: 'price',
  validators: { 
    required: {},
    number_range: { min: 0 }
  }
});

// Set the title field
await client.itemTypes.update(itemType.id, {
  title_field: {
    type: 'field',
    id: titleField.id
  }
});
```

## Complete Example: Blog System

Create a full blog content structure:

```javascript
async function createBlogStructure() {
  // 1. Create Author model
  const authorModel = await client.itemTypes.create({
    name: 'Author',
    api_key: 'author',
    collection_appearance: 'table'
  });

  const authorNameField = await client.fields.create(authorModel.id, {
    label: 'Name',
    field_type: 'string',
    api_key: 'name',
    validators: { required: {} }
  });

  await client.itemTypes.update(authorModel.id, {
    title_field: { type: 'field', id: authorNameField.id }
  });

  // 2. Create Category model (tree structure)
  const categoryModel = await client.itemTypes.create({
    name: 'Category',
    api_key: 'category',
    tree: true
  });

  const categoryNameField = await client.fields.create(categoryModel.id, {
    label: 'Name',
    field_type: 'string',
    api_key: 'name',
    validators: { required: {} }
  });

  await client.itemTypes.update(categoryModel.id, {
    title_field: { type: 'field', id: categoryNameField.id }
  });

  // 3. Create Blog Post model
  const blogPostModel = await client.itemTypes.create({
    name: 'Blog Post',
    api_key: 'blog_post',
    draft_mode_active: true,
    collection_appearance: 'table',
    ordering_direction: 'desc',
    ordering_meta: 'created_at'
  });

  // Add fields to blog post
  const titleField = await client.fields.create(blogPostModel.id, {
    label: 'Title',
    field_type: 'string',
    api_key: 'title',
    validators: { required: {} }
  });

  const contentField = await client.fields.create(blogPostModel.id, {
    label: 'Content',
    field_type: 'structured_text',
    api_key: 'content'
  });

  const authorField = await client.fields.create(blogPostModel.id, {
    label: 'Author',
    field_type: 'link',
    api_key: 'author',
    validators: {
      item_item_type: { item_types: [authorModel.id] },
      required: {}
    }
  });

  const categoriesField = await client.fields.create(blogPostModel.id, {
    label: 'Categories',
    field_type: 'links',
    api_key: 'categories',
    validators: {
      items_item_type: { item_types: [categoryModel.id] }
    }
  });

  const featuredImageField = await client.fields.create(blogPostModel.id, {
    label: 'Featured Image',
    field_type: 'file',
    api_key: 'featured_image',
    validators: {
      required: {},
      file_size: { max: 5242880 }, // 5MB
      extension: { extensions: ['jpg', 'jpeg', 'png', 'webp'] }
    }
  });

  const excerptField = await client.fields.create(blogPostModel.id, {
    label: 'Excerpt',
    field_type: 'text',
    api_key: 'excerpt',
    validators: {
      length: { max: 200 }
    }
  });

  // Configure blog post presentation
  await client.itemTypes.update(blogPostModel.id, {
    title_field: { type: 'field', id: titleField.id },
    image_preview_field: { type: 'field', id: featuredImageField.id },
    excerpt_field: { type: 'field', id: excerptField.id }
  });

  // 4. Create CTA block for structured text
  // Setting modular_block: true makes this available as a block in modular content fields
  // See ./modular-content.md for complete block documentation
  const ctaBlock = await client.itemTypes.create({
    name: 'Call to Action',
    api_key: 'cta_block',
    modular_block: true
  });

  await client.fields.create(ctaBlock.id, {
    label: 'Button Text',
    field_type: 'string',
    api_key: 'button_text',
    validators: { required: {} }
  });

  await client.fields.create(ctaBlock.id, {
    label: 'Button URL',
    field_type: 'string',
    api_key: 'button_url',
    validators: { 
      required: {},
      format: { predefined_pattern: 'url' }
    }
  });

  return {
    author: authorModel,
    category: categoryModel,
    blogPost: blogPostModel,
    ctaBlock: ctaBlock
  };
}
```

## Model Naming Conventions

### API Key Rules
- Lowercase letters, numbers, underscores only
- Must start with a letter
- No spaces or special characters
- Examples: `article`, `blog_post`, `product_category`

### Display Name Best Practices
- Use singular form for standard models
- Use descriptive names
- Include context when needed
- Examples: "Article", "Blog Post", "Product Category"

### Common Naming Patterns
```javascript
// Content types
'article', 'page', 'post', 'product'

// Settings/Configuration
'site_settings', 'global_config', 'theme_settings'

// Navigation/Structure
'menu_item', 'navigation', 'breadcrumb'

// Media/Assets
'gallery', 'media_item', 'download'

// Users/Permissions
'author', 'contributor', 'team_member'

// E-commerce
'product', 'category', 'variant', 'order'
```

## Model Limits and Quotas

Check your project's limits:

```javascript
// Get current usage
const site = await client.site.find();
const usage = site.relationships.usage.data.attributes;

console.log(`Models: ${usage.itemTypes.used}/${usage.itemTypes.limit}`);
console.log(`Items: ${usage.items.used}/${usage.items.limit}`);

// Check before creating
if (usage.itemTypes.used >= usage.itemTypes.limit) {
  throw new Error('Model limit reached');
}
```

## Error Handling

```javascript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.itemTypes.create({
    name: 'Test Model',
    api_key: 'test-model' // May already exist
  });
} catch (error) {
  if (error instanceof ApiError) {
    const apiKeyError = error.findError('api_key');
    if (apiKeyError?.code === 'VALIDATION_UNIQUE') {
      console.log('Model with this API key already exists');
    }
    
    // Handle validation errors
    if (error.response.status === 422) {
      error.errors.forEach(err => {
        console.log(`Field: ${err.attributes.field}`);
        console.log(`Error: ${err.attributes.details}`);
      });
    }
  }
}

// Safe model creation with existence check
async function createModelIfNotExists(config: any) {
  const models = await client.itemTypes.list();
  const exists = models.some(m => m.api_key === config.api_key);
  
  if (exists) {
    console.log(`Model ${config.api_key} already exists`);
    return models.find(m => m.api_key === config.api_key);
  }
  
  return await client.itemTypes.create(config);
}
```

## Item Type Object Structure

```javascript
// ItemType object structure
{
  id: string;
  name: string;
  api_key: string;
  singleton: boolean;
  sortable: boolean;
  tree: boolean;
  modular_block: boolean;
  draft_mode_active: boolean;
  draft_saving_active: boolean;
  all_locales_required: boolean;
  collection_appearance: 'table' | 'grid';
  hint?: string;
  inverse_relationships_enabled: boolean;
  ordering_direction?: 'asc' | 'desc';
  ordering_meta?: 'created_at' | 'updated_at' | 'published_at' | 'first_published_at' | 'unpublished_at';
  ordering_field?: {
    type: 'field';
    id: string;
  };
  title_field?: {
    type: 'field';
    id: string;
  };
  presentation_title_field?: {
    type: 'field';
    id: string;
  };
  image_preview_field?: {
    type: 'field';
    id: string;
  };
  presentation_image_field?: {
    type: 'field';
    id: string;
  };
  excerpt_field?: {
    type: 'field';
    id: string;
  };
  workflow?: {
    type: 'workflow';
    id: string;
  };
  fields_count: number;
  items_count?: number; // Not available for modular blocks
}
  id: string;
  name: string;
  api_key: string;
  singleton: boolean;
  sortable: boolean;
  tree: boolean;
  modular_block: boolean;
  draft_mode_active: boolean;
  draft_saving_active: boolean;
  all_locales_required: boolean;
  collection_appearance: 'table' | 'grid';
  hint?: string;
  inverse_relationships_enabled: boolean;
  ordering_direction?: 'asc' | 'desc';
  ordering_meta?: 'created_at' | 'updated_at' | 'published_at' | 'first_published_at' | 'unpublished_at';
  ordering_field?: {
    type: 'field';
    id: string;
  };
  title_field?: {
    type: 'field';
    id: string;
  };
  presentation_title_field?: {
    type: 'field';
    id: string;
  };
  image_preview_field?: {
    type: 'field';
    id: string;
  };
  presentation_image_field?: {
    type: 'field';
    id: string;
  };
  excerpt_field?: {
    type: 'field';
    id: string;
  };
  workflow?: {
    type: 'workflow';
    id: string;
  };
  fields_count: number;
  items_count?: number; // Not available for modular blocks
}
```

## Advanced Patterns

### Model Configuration Patterns

#### Blog Model Setup

```javascript
async function createBlogModel() {
  // Create the model
  const blogPost = await client.itemTypes.create({
    name: 'Blog Post',
    api_key: 'blog_post',
    singleton: false,
    sortable: false,
    draft_mode_active: true,
    all_locales_required: false,
    collection_appearance: 'table',
    hint: 'Blog posts and articles'
  });
  
  // Add fields
  await client.fields.create(blogPost.id, {
    label: 'Title',
    api_key: 'title',
    field_type: 'string',
    validators: { required: {} },
    localized: true
  });
  
  await client.fields.create(blogPost.id, {
    label: 'Slug',
    api_key: 'slug',
    field_type: 'slug',
    validators: { 
      required: {},
      unique: {},
      slug_format: {}
    }
  });
  
  await client.fields.create(blogPost.id, {
    label: 'Content',
    api_key: 'content',
    field_type: 'structured_text',
    validators: { required: {} },
    localized: true
  });
  
  await client.fields.create(blogPost.id, {
    label: 'Featured Image',
    api_key: 'featured_image',
    field_type: 'file',
    validators: {
      required: {},
      file: { 
        required_image_dimensions: true,
        image_dimensions: { 
          width: { min: 1200 },
          height: { min: 630 }
        }
      }
    }
  });
  
  await client.fields.create(blogPost.id, {
    label: 'Author',
    api_key: 'author',
    field_type: 'link',
    validators: { 
      required: {},
      item_item_type: { item_types: ['author_model_id'] }
    }
  });
  
  await client.fields.create(blogPost.id, {
    label: 'Published Date',
    api_key: 'published_date',
    field_type: 'date',
    validators: { required: {} }
  });
  
  await client.fields.create(blogPost.id, {
    label: 'SEO',
    api_key: 'seo',
    field_type: 'seo',
    validators: {}
  });
  
  return blogPost;
}
```

#### E-commerce Product Model

```javascript
async function createProductModel() {
  const product = await client.itemTypes.create({
    name: 'Product',
    api_key: 'product',
    draft_mode_active: true,
    collection_appearance: 'compact'
  });
  
  // Basic fields
  await client.fields.create(product.id, {
    label: 'Name',
    api_key: 'name',
    field_type: 'string',
    validators: { required: {} },
    localized: true
  });
  
  await client.fields.create(product.id, {
    label: 'SKU',
    api_key: 'sku',
    field_type: 'string',
    validators: { 
      required: {},
      unique: {},
      format: { pattern: '^[A-Z0-9-]+$' }
    }
  });
  
  await client.fields.create(product.id, {
    label: 'Price',
    api_key: 'price',
    field_type: 'float',
    validators: { 
      required: {},
      number_range: { min: 0 }
    },
    appearance: {
      editor: 'float',
      parameters: { prefix: '$' }
    }
  });
  
  await client.fields.create(product.id, {
    label: 'Images',
    api_key: 'images',
    field_type: 'gallery',
    validators: {
      size: { min: 1, max: 10 }
    }
  });
  
  await client.fields.create(product.id, {
    label: 'Categories',
    api_key: 'categories',
    field_type: 'links',
    validators: {
      size: { min: 1 },
      items_item_type: { item_types: ['category_model_id'] }
    }
  });
  
  await client.fields.create(product.id, {
    label: 'Variants',
    api_key: 'variants',
    field_type: 'modular_content',
    validators: {
      rich_text_blocks: { item_types: ['product_variant_block_id'] }
    }
  });
  
  return product;
}
```

#### Page Builder Model

```javascript
async function createPageModel() {
  // Create modular blocks first
  const heroBlock = await client.itemTypes.create({
    name: 'Hero Block',
    api_key: 'hero_block',
    modular_block: true
  });
  
  const contentBlock = await client.itemTypes.create({
    name: 'Content Block',
    api_key: 'content_block',
    modular_block: true
  });
  
  const galleryBlock = await client.itemTypes.create({
    name: 'Gallery Block',
    api_key: 'gallery_block',
    modular_block: true
  });
  
  // Create page model
  const page = await client.itemTypes.create({
    name: 'Landing Page',
    api_key: 'landing_page',
    tree: true,
    draft_mode_active: true
  });
  
  // Add page builder field
  await client.fields.create(page.id, {
    label: 'Page Sections',
    api_key: 'page_sections',
    field_type: 'modular_content',
    validators: {
      rich_text_blocks: { 
        item_types: [heroBlock.id, contentBlock.id, galleryBlock.id] 
      }
    }
  });
  
  return { page, blocks: [heroBlock, contentBlock, galleryBlock] };
}
```

### Model Management Patterns

#### Find Models by Field Type

```javascript
async function findModelsWithFieldType(fieldType) {
  const allModels = await client.itemTypes.list();
  const modelsWithField = [];
  
  for (const model of allModels) {
    const fields = await client.fields.list(model.id);
    const hasFieldType = fields.some(field => field.field_type === fieldType);
    
    if (hasFieldType) {
      modelsWithField.push(model);
    }
  }
  
  return modelsWithField;
}

// Usage
const modelsWithSeo = await findModelsWithFieldType('seo');
const modelsWithModularContent = await findModelsWithFieldType('modular_content');
```

#### Model Dependencies

```javascript
async function getModelDependencies(modelId) {
  const allModels = await client.itemTypes.list();
  const dependencies = {
    referencedBy: [],
    references: []
  };
  
  // Find models that reference this model
  for (const model of allModels) {
    const fields = await client.fields.list(model.id);
    
    for (const field of fields) {
      if (field.field_type === 'link' || field.field_type === 'links') {
        const allowedModels = field.validators?.item_item_type?.item_types || 
                            field.validators?.items_item_type?.item_types || [];
        
        if (allowedModels.includes(modelId)) {
          dependencies.referencedBy.push({
            model: model,
            field: field
          });
        }
      }
    }
  }
  
  // Find models this model references
  const thisModelFields = await client.fields.list(modelId);
  
  for (const field of thisModelFields) {
    if (field.field_type === 'link' || field.field_type === 'links') {
      const allowedModels = field.validators?.item_item_type?.item_types || 
                          field.validators?.items_item_type?.item_types || [];
      
      dependencies.references.push({
        field: field,
        models: allowedModels
      });
    }
  }
  
  return dependencies;
}
```

#### Bulk Model Operations

```javascript
// Export all models structure
async function exportModelsStructure() {
  const models = await client.itemTypes.list();
  const exportData = [];
  
  for (const model of models) {
    const fields = await client.fields.list(model.id);
    const fieldsets = await client.fieldsets.list(model.id);
    
    exportData.push({
      model: model,
      fields: fields,
      fieldsets: fieldsets
    });
  }
  
  return exportData;
}

// Clone model structure to another project
async function cloneModelToProject(modelId, targetClient) {
  const sourceModel = await client.itemTypes.find(modelId);
  const sourceFields = await client.fields.list(modelId);
  const sourceFieldsets = await client.fieldsets.list(modelId);
  
  // Create model in target project
  const newModel = await targetClient.itemTypes.create({
    name: sourceModel.name,
    api_key: sourceModel.api_key + '_copy',
    singleton: sourceModel.singleton,
    modular_block: sourceModel.modular_block,
    tree: sourceModel.tree,
    sortable: sourceModel.sortable,
    draft_mode_active: sourceModel.draft_mode_active
  });
  
  // Create fieldsets
  const fieldsetMap = new Map();
  for (const fieldset of sourceFieldsets) {
    const newFieldset = await targetClient.fieldsets.create(newModel.id, {
      title: fieldset.title,
      hint: fieldset.hint,
      collapsible: fieldset.collapsible,
      start_collapsed: fieldset.start_collapsed
    });
    fieldsetMap.set(fieldset.id, newFieldset.id);
  }
  
  // Create fields
  for (const field of sourceFields) {
    const fieldData = {
      label: field.label,
      api_key: field.api_key,
      field_type: field.field_type,
      validators: field.validators,
      localized: field.localized,
      hint: field.hint,
      default_value: field.default_value,
      appearance: field.appearance
    };
    
    // Update fieldset reference
    if (field.fieldset) {
      fieldData.fieldset = {
        type: 'fieldset',
        id: fieldsetMap.get(field.fieldset.id)
      };
    }
    
    await targetClient.fields.create(newModel.id, fieldData);
  }
  
  return newModel;
}
```

### Comprehensive Error Handling

```javascript
// Comprehensive error handling
async function safeModelOperation() {
  try {
    const model = await client.itemTypes.create({
      name: 'Test Model',
      api_key: 'test_model'
    });
    return model;
  } catch (error) {
    if (error.response) {
      switch (error.response.status) {
        case 422:
          const validationErrors = error.response.data.errors;
          validationErrors.forEach(err => {
            if (err.source?.pointer?.includes('api_key')) {
              console.error('API key error:', err.detail);
            }
          });
          break;
        case 409:
          console.error('Model with this API key already exists');
          break;
        case 403:
          console.error('Insufficient permissions to create models');
          break;
        default:
          console.error('Unexpected error:', error.response.status);
      }
    }
    throw error;
  }
}

// Validate before creation
async function validateModelCreation(modelData) {
  const errors = [];
  
  // Check API key format
  if (!/^[a-z_][a-z0-9_]*$/.test(modelData.api_key)) {
    errors.push('API key must contain only lowercase letters, numbers, and underscores');
  }
  
  // Check if API key exists
  const existingModels = await client.itemTypes.list();
  if (existingModels.some(m => m.api_key === modelData.api_key)) {
    errors.push('Model with this API key already exists');
  }
  
  // Check singleton with tree
  if (modelData.singleton && modelData.tree) {
    errors.push('Model cannot be both singleton and tree');
  }
  
  return errors;
}
```

## Related Resources

- [Fields](./field.md) - Define model structure
- [Fieldsets](./fieldset.md) - Group fields
- [Items](./item.md) - Create content
- [Workflows](../03-publishing-workflow/workflow.md) - Content approval
- [Schema Menu Items](../04-site-configuration/schema-menu-item.md) - Organize models in UI