# Fieldset Resource

## Overview

Fieldsets allow you to group related fields together in the CMS interface, making forms more organized and easier to navigate. They can be collapsible to save space and improve the editing experience. Fieldsets help organize complex models by grouping fields into logical sections like "SEO Settings", "Media", or "Advanced Options".

## API Client Section

`client.fieldsets`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a fieldset for SEO fields
const fieldset = await client.fieldsets.create('item-type-id', {
  title: 'SEO Settings',
  hint: 'Search engine optimization fields',
  collapsible: true,
  start_collapsed: true
});

// List all fieldsets for a model
const fieldsets = await client.fieldsets.list('item-type-id');

// Update a fieldset
const updated = await client.fieldsets.update('fieldset-id', {
  title: 'Updated SEO Settings'
});

// Delete a fieldset (fields are ungrouped, not deleted)
await client.fieldsets.destroy('fieldset-id');
```

## API Reference

### list() - List Fieldsets

Retrieve all fieldsets for a specific item type (model).

**Signature**: `list(itemTypeId, queryParams?): Promise<Fieldset[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID (required) |
| `orderBy` | string | Sort order (e.g., 'position_ASC') |

**Basic Example**:
```javascript
// Get all fieldsets for a model
const fieldsets = await client.fieldsets.list('model_id');

// Get fieldsets with ordering
const orderedFieldsets = await client.fieldsets.list('model_id', {
  orderBy: 'position_ASC'
});

// Get raw response with metadata
const response = await client.fieldsets.rawList('model_id');
// response.meta contains pagination info
```

**Advanced Example**:
```javascript
// List and analyze fieldset structure
const fieldsets = await client.fieldsets.list('item-type-id');

// Fieldsets are returned in position order
fieldsets.forEach(fieldset => {
  console.log(`${fieldset.position}: ${fieldset.title}`);
  if (fieldset.hint) {
    console.log(`  Hint: ${fieldset.hint}`);
  }
  console.log(`  Collapsible: ${fieldset.collapsible}`);
  console.log(`  Start collapsed: ${fieldset.start_collapsed}`);
});
```

### find() - Get Single Fieldset

Retrieve a specific fieldset by ID.

**Signature**: `find(fieldsetId): Promise<Fieldset>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `fieldsetId` | string | The fieldset ID (required) |

**Basic Example**:
```javascript
// Get fieldset by ID
const fieldset = await client.fieldsets.find('fieldset_id');

// Check fieldset configuration
console.log(`Fieldset: ${fieldset.title}`);
console.log(`Collapsible: ${fieldset.collapsible}`);
console.log(`Item Type: ${fieldset.item_type.id}`);
```

**Error Handling Example**:
```javascript
try {
  const fieldset = await client.fieldsets.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Fieldset not found');
  }
}
```

### create() - Create Fieldset

Create a new fieldset within an item type to group related fields.

**Signature**: `create(itemTypeId, body): Promise<Fieldset>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID (required) |
| `title` | string | Fieldset display title (required) |
| `hint` | string | Help text for the fieldset |
| `position` | number | Order position among fieldsets |
| `collapsible` | boolean | Can be collapsed (default: false) |
| `start_collapsed` | boolean | Initially collapsed (default: false) |

**Basic Example**:
```javascript
// Create basic fieldset
const fieldset = await client.fieldsets.create('model_id', {
  title: 'SEO Settings',
  hint: 'Search engine optimization fields',
  collapsible: true,
  start_collapsed: false
});

// Create with specific position
const positionedFieldset = await client.fieldsets.create('model_id', {
  title: 'Media',
  hint: 'Images and videos',
  position: 2
});
```

**Advanced Example (With Fields)**:
```javascript
// Create fieldset and add fields
const fieldset = await client.fieldsets.create('product-model-id', {
  title: 'Pricing Information',
  hint: 'Set product pricing and discount options',
  position: 1,
  collapsible: true,
  start_collapsed: false
});

// Create fields within the fieldset
await client.fields.create('product-model-id', {
  label: 'Price',
  field_type: 'float',
  api_key: 'price',
  fieldset: {
    type: 'fieldset',
    id: fieldset.id
  }
});

await client.fields.create('product-model-id', {
  label: 'Sale Price',
  field_type: 'float',
  api_key: 'sale_price',
  fieldset: {
    type: 'fieldset',
    id: fieldset.id
  }
});
```

**Error Handling Example**:
```javascript
try {
  const fieldset = await client.fieldsets.create('model_id', {
    title: '', // Empty title not allowed
    collapsible: true
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation error:', error.response.data.errors);
  }
}
```

### update() - Update Fieldset

Modify an existing fieldset's properties.

**Signature**: `update(fieldsetId, body): Promise<Fieldset>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `fieldsetId` | string | The fieldset ID (required) |
| `title` | string | New display title |
| `hint` | string | Updated help text |
| `position` | number | New order position |
| `collapsible` | boolean | Toggle collapsibility |
| `start_collapsed` | boolean | Default collapsed state |

**Basic Example**:
```javascript
// Update fieldset title
const updated = await client.fieldsets.update('fieldset_id', {
  title: 'Updated SEO Settings'
});

// Make fieldset collapsible
const collapsible = await client.fieldsets.update('fieldset_id', {
  collapsible: true,
  start_collapsed: false
});
```

**Advanced Example**:
```javascript
// Update multiple properties
const fullyUpdated = await client.fieldsets.update('fieldset-id', {
  title: 'Advanced SEO Settings',
  hint: 'Configure meta tags and social sharing',
  collapsible: true,
  start_collapsed: true,
  position: 3
});

// Error handling
try {
  const updated = await client.fieldsets.update('fieldset_id', {
    title: '' // Empty title not allowed
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Title cannot be empty');
  }
}
```

### destroy() - Delete Fieldset

Remove a fieldset. Fields within the fieldset are not deleted but become ungrouped.

**Signature**: `destroy(fieldsetId): Promise<void>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `fieldsetId` | string | The fieldset ID (required) |

**Basic Example**:
```javascript
// Delete fieldset
await client.fieldsets.destroy('fieldset_id');

// Delete with error handling
try {
  await client.fieldsets.destroy('fieldset_id');
  console.log('Fieldset deleted successfully');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Fieldset not found');
  }
}
```

**Advanced Example (Safe Delete)**:
```javascript
// Move fields out of fieldset first (optional)
async function safeDeleteFieldset(fieldsetId) {
  try {
    // First, move all fields out of fieldset
    const fieldset = await client.fieldsets.find(fieldsetId);
    const modelId = fieldset.relationships.item_type.data.id;
    
    const fields = await client.fields.list(modelId);
    const fieldsInFieldset = fields.filter(
      f => f.relationships.fieldset?.data?.id === fieldsetId
    );
    
    // Move fields out of fieldset
    for (const field of fieldsInFieldset) {
      await client.fields.update(field.id, {
        fieldset: null
      });
    }
    
    // Then delete the fieldset
    await client.fieldsets.destroy(fieldsetId);
    console.log('Fieldset deleted successfully');
  } catch (error) {
    console.error('Failed to delete fieldset:', error);
  }
}
```

## Working with Fields

### Adding Fields to Fieldset

```javascript
// Create fieldset
const fieldset = await client.fieldsets.create('model_id', {
  title: 'Contact Information',
  collapsible: true
});

// Add new field directly to fieldset
const emailField = await client.fields.create('model_id', {
  label: 'Email',
  api_key: 'email',
  field_type: 'string',
  fieldset: {
    type: 'fieldset',
    id: fieldset.id
  },
  position: 1 // Position within fieldset
});

// Add another field
const phoneField = await client.fields.create('model_id', {
  label: 'Phone',
  api_key: 'phone',
  field_type: 'string',
  fieldset: {
    type: 'fieldset',
    id: fieldset.id
  },
  position: 2
});
```

### Moving Existing Fields

```javascript
// Move existing field into fieldset
await client.fields.update('existing_field_id', {
  fieldset: {
    type: 'fieldset',
    id: 'fieldset_id'
  }
});

// Move field out of fieldset
await client.fields.update('field_id', {
  fieldset: null
});

// Move multiple fields to fieldset
async function moveFieldsToFieldset(fieldIds, fieldsetId) {
  const results = await Promise.allSettled(
    fieldIds.map((fieldId, index) => 
      client.fields.update(fieldId, {
        fieldset: {
          type: 'fieldset',
          id: fieldsetId
        },
        position: index + 1
      })
    )
  );
  
  const successful = results.filter(r => r.status === 'fulfilled').length;
  const failed = results.filter(r => r.status === 'rejected').length;
  
  return { successful, failed };
}
```

## Common Patterns

### SEO Fieldset Template

Group SEO-related fields together:

```javascript
// Create SEO fieldset
const seoFieldset = await client.fieldsets.create('page-model', {
  title: 'SEO & Social',
  hint: 'Configure how this page appears in search results and social media',
  collapsible: true,
  start_collapsed: true,
  position: 99 // Put at the end
});

// Add SEO field
await client.fields.create('page-model', {
  label: 'SEO',
  field_type: 'seo',
  api_key: 'seo',
  fieldset: {
    type: 'fieldset',
    id: seoFieldset.id
  }
});

// Add Open Graph image
await client.fields.create('page-model', {
  label: 'Social Share Image',
  field_type: 'file',
  api_key: 'og_image',
  validators: {
    extension: { predefined_list: 'image' },
    dimension: { 
      width: { min: 1200 },
      height: { min: 630 }
    }
  },
  fieldset: {
    type: 'fieldset',
    id: seoFieldset.id
  }
});
```

### Media Fieldset

Organize media-related fields:

```javascript
const mediaFieldset = await client.fieldsets.create('article-model', {
  title: 'Media & Images',
  hint: 'Featured images and media content',
  collapsible: true,
  start_collapsed: false
});

// Add media fields
const mediaFields = [
  {
    label: 'Featured Image',
    field_type: 'file',
    api_key: 'featured_image',
    validators: { required: {} }
  },
  {
    label: 'Image Caption',
    field_type: 'string',
    api_key: 'image_caption'
  },
  {
    label: 'Image Gallery',
    field_type: 'gallery',
    api_key: 'gallery'
  },
  {
    label: 'Video Embed',
    field_type: 'string',
    api_key: 'video_url'
  }
];

for (const fieldConfig of mediaFields) {
  await client.fields.create('article-model', {
    ...fieldConfig,
    fieldset: {
      type: 'fieldset',
      id: mediaFieldset.id
    }
  });
}
```

### Settings Fieldset

Group configuration options:

```javascript
const settingsFieldset = await client.fieldsets.create('component-model', {
  title: 'Display Settings',
  hint: 'Control how this component appears',
  collapsible: true,
  start_collapsed: true
});

// Layout options
await client.fields.create('component-model', {
  label: 'Layout',
  field_type: 'string',
  api_key: 'layout',
  appearance: {
    editor: 'string_select',
    parameters: {
      options: [
        { label: 'Full Width', value: 'full' },
        { label: 'Contained', value: 'contained' },
        { label: 'Narrow', value: 'narrow' }
      ]
    }
  },
  fieldset: { type: 'fieldset', id: settingsFieldset.id }
});

// Color scheme
await client.fields.create('component-model', {
  label: 'Color Scheme',
  field_type: 'string',
  api_key: 'color_scheme',
  appearance: {
    editor: 'string_radio_group',
    parameters: {
      radios: [
        { label: 'Light', value: 'light' },
        { label: 'Dark', value: 'dark' },
        { label: 'Auto', value: 'auto' }
      ]
    }
  },
  fieldset: { type: 'fieldset', id: settingsFieldset.id }
});
```

### Localization Fieldset

Group fields that need translation:

```javascript
const i18nFieldset = await client.fieldsets.create('product-model', {
  title: 'Localized Content',
  hint: 'Content that needs translation for each market',
  collapsible: false // Always visible for translators
});

// Add localized fields
const localizedFields = [
  { label: 'Product Name', api_key: 'name', field_type: 'string' },
  { label: 'Description', api_key: 'description', field_type: 'text' },
  { label: 'Features', api_key: 'features', field_type: 'text' }
];

for (const field of localizedFields) {
  await client.fields.create('product-model', {
    ...field,
    localized: true, // Enable localization
    fieldset: { type: 'fieldset', id: i18nFieldset.id }
  });
}
```

## Advanced Usage

### Dynamic Fieldset Management

Programmatically organize fields into fieldsets:

```javascript
async function reorganizeFields(itemTypeId: string) {
  // Get all fields
  const fields = await client.fields.list(itemTypeId);
  
  // Group fields by category
  const categories = {
    content: ['title', 'body', 'excerpt'],
    media: ['image', 'gallery', 'video'],
    seo: ['meta_title', 'meta_description', 'slug'],
    settings: ['status', 'featured', 'template']
  };
  
  // Create fieldsets for each category
  for (const [category, fieldApiKeys] of Object.entries(categories)) {
    const categoryFields = fields.filter(f => 
      fieldApiKeys.includes(f.api_key)
    );
    
    if (categoryFields.length > 0) {
      // Create fieldset
      const fieldset = await client.fieldsets.create(itemTypeId, {
        title: category.charAt(0).toUpperCase() + category.slice(1),
        collapsible: category !== 'content',
        start_collapsed: category === 'settings'
      });
      
      // Assign fields to fieldset
      for (const field of categoryFields) {
        await client.fields.update(field.id, {
          fieldset: { type: 'fieldset', id: fieldset.id }
        });
      }
    }
  }
}
```

### Fieldset Templates

Create reusable fieldset configurations:

```javascript
class FieldsetTemplates {
  static async createSeoFieldset(client: any, itemTypeId: string) {
    const fieldset = await client.fieldsets.create(itemTypeId, {
      title: 'SEO',
      hint: 'Search engine optimization',
      collapsible: true,
      start_collapsed: true,
      position: 100
    });
    
    await client.fields.create(itemTypeId, {
      label: 'SEO',
      field_type: 'seo',
      api_key: 'seo',
      fieldset: { type: 'fieldset', id: fieldset.id }
    });
    
    return fieldset;
  }
  
  static async createTimestampFieldset(client: any, itemTypeId: string) {
    const fieldset = await client.fieldsets.create(itemTypeId, {
      title: 'Timestamps',
      hint: 'Publication and scheduling dates',
      collapsible: true,
      start_collapsed: true
    });
    
    const timestampFields = [
      { label: 'Publish Date', api_key: 'publish_date', field_type: 'datetime' },
      { label: 'Expiry Date', api_key: 'expiry_date', field_type: 'datetime' },
      { label: 'Last Review', api_key: 'last_review', field_type: 'date' }
    ];
    
    for (const field of timestampFields) {
      await client.fields.create(itemTypeId, {
        ...field,
        fieldset: { type: 'fieldset', id: fieldset.id }
      });
    }
    
    return fieldset;
  }
}

// Usage
await FieldsetTemplates.createSeoFieldset(client, 'article');
await FieldsetTemplates.createTimestampFieldset(client, 'article');
```

### Fieldset Order Management

Reorder fieldsets for better UX:

```javascript
async function reorderFieldsets(itemTypeId: string, order: string[]) {
  const fieldsets = await client.fieldsets.list(itemTypeId);
  
  // Create a map of title to fieldset
  const fieldsetMap = new Map(
    fieldsets.map(fs => [fs.title.toLowerCase(), fs])
  );
  
  // Update positions based on desired order
  for (let i = 0; i < order.length; i++) {
    const fieldset = fieldsetMap.get(order[i].toLowerCase());
    if (fieldset) {
      await client.fieldsets.update(fieldset.id, {
        position: i + 1
      });
    }
  }
}

// Set specific order
await reorderFieldsets('page-model', [
  'Content',      // Main content first
  'Media',        // Images and videos
  'Layout',       // Display options
  'SEO',          // SEO last
]);
```

### Creating Organized Model Structure

```javascript
async function createOrganizedModel(modelId) {
  // Create fieldsets for organization
  const basicFieldset = await client.fieldsets.create(modelId, {
    title: 'Basic Information',
    hint: 'Core content fields',
    collapsible: false,
    position: 1
  });
  
  const mediaFieldset = await client.fieldsets.create(modelId, {
    title: 'Media',
    hint: 'Images and files',
    collapsible: true,
    start_collapsed: false,
    position: 2
  });
  
  const seoFieldset = await client.fieldsets.create(modelId, {
    title: 'SEO',
    hint: 'Search engine optimization',
    collapsible: true,
    start_collapsed: true,
    position: 3
  });
  
  const advancedFieldset = await client.fieldsets.create(modelId, {
    title: 'Advanced Settings',
    hint: 'Additional configuration',
    collapsible: true,
    start_collapsed: true,
    position: 4
  });
  
  // Create fields in appropriate fieldsets
  await client.fields.create(modelId, {
    label: 'Title',
    api_key: 'title',
    field_type: 'string',
    validators: { required: {} },
    fieldset: { type: 'fieldset', id: basicFieldset.id },
    position: 1
  });
  
  await client.fields.create(modelId, {
    label: 'Content',
    api_key: 'content',
    field_type: 'structured_text',
    validators: { required: {} },
    fieldset: { type: 'fieldset', id: basicFieldset.id },
    position: 2
  });
  
  await client.fields.create(modelId, {
    label: 'Featured Image',
    api_key: 'featured_image',
    field_type: 'file',
    fieldset: { type: 'fieldset', id: mediaFieldset.id },
    position: 1
  });
  
  await client.fields.create(modelId, {
    label: 'Gallery',
    api_key: 'gallery',
    field_type: 'gallery',
    fieldset: { type: 'fieldset', id: mediaFieldset.id },
    position: 2
  });
  
  await client.fields.create(modelId, {
    label: 'SEO Title',
    api_key: 'seo_title',
    field_type: 'string',
    fieldset: { type: 'fieldset', id: seoFieldset.id },
    position: 1
  });
  
  await client.fields.create(modelId, {
    label: 'SEO Description',
    api_key: 'seo_description',
    field_type: 'text',
    fieldset: { type: 'fieldset', id: seoFieldset.id },
    position: 2
  });
  
  return {
    fieldsets: [basicFieldset, mediaFieldset, seoFieldset, advancedFieldset]
  };
}
```

### Cloning Fieldset Structure

```javascript
async function cloneFieldsetStructure(sourceModelId, targetModelId) {
  // Get source fieldsets
  const sourceFieldsets = await client.fieldsets.list(sourceModelId);
  const fieldsetMapping = new Map();
  
  // Clone fieldsets
  for (const sourceFieldset of sourceFieldsets) {
    const newFieldset = await client.fieldsets.create(targetModelId, {
      title: sourceFieldset.title,
      hint: sourceFieldset.hint,
      collapsible: sourceFieldset.collapsible,
      start_collapsed: sourceFieldset.start_collapsed,
      position: sourceFieldset.position
    });
    
    fieldsetMapping.set(sourceFieldset.id, newFieldset.id);
  }
  
  // Get source fields
  const sourceFields = await client.fields.list(sourceModelId);
  
  // Clone fields with fieldset relationships
  for (const sourceField of sourceFields) {
    const fieldData = {
      label: sourceField.label,
      api_key: sourceField.api_key,
      field_type: sourceField.field_type,
      validators: sourceField.validators,
      appearance: sourceField.appearance,
      position: sourceField.position,
      localized: sourceField.localized,
      hint: sourceField.hint
    };
    
    // Map fieldset if exists
    if (sourceField.relationships.fieldset?.data?.id) {
      const targetFieldsetId = fieldsetMapping.get(
        sourceField.relationships.fieldset.data.id
      );
      if (targetFieldsetId) {
        fieldData.fieldset = {
          type: 'fieldset',
          id: targetFieldsetId
        };
      }
    }
    
    await client.fields.create(targetModelId, fieldData);
  }
  
  return {
    fieldsets: fieldsetMapping.size,
    fields: sourceFields.length
  };
}
```

### Fieldset Analytics

```javascript
async function analyzeFieldsetUsage(modelId) {
  const fieldsets = await client.fieldsets.list(modelId);
  const fields = await client.fields.list(modelId);
  
  const analysis = {
    totalFieldsets: fieldsets.length,
    totalFields: fields.length,
    fieldsets: []
  };
  
  for (const fieldset of fieldsets) {
    const fieldsInFieldset = fields.filter(
      f => f.relationships.fieldset?.data?.id === fieldset.id
    );
    
    analysis.fieldsets.push({
      id: fieldset.id,
      title: fieldset.title,
      fieldCount: fieldsInFieldset.length,
      fields: fieldsInFieldset.map(f => ({
        id: f.id,
        label: f.label,
        api_key: f.api_key,
        field_type: f.field_type
      })),
      collapsible: fieldset.collapsible,
      start_collapsed: fieldset.start_collapsed
    });
  }
  
  // Find orphaned fields (not in any fieldset)
  const orphanedFields = fields.filter(
    f => !f.relationships.fieldset
  );
  
  analysis.orphanedFields = {
    count: orphanedFields.length,
    fields: orphanedFields.map(f => ({
      id: f.id,
      label: f.label,
      api_key: f.api_key
    }))
  };
  
  return analysis;
}
```

### Reorganizing Model Fields

```javascript
async function reorganizeModelFields(modelId) {
  const fields = await client.fields.list(modelId);
  
  // Categorize fields
  const categories = {
    content: [],
    media: [],
    seo: [],
    metadata: [],
    other: []
  };
  
  for (const field of fields) {
    if (['title', 'content', 'body', 'description'].includes(field.api_key)) {
      categories.content.push(field);
    } else if (['file', 'gallery'].includes(field.field_type)) {
      categories.media.push(field);
    } else if (field.api_key.includes('seo') || field.field_type === 'seo') {
      categories.seo.push(field);
    } else if (['created_at', 'updated_at', 'tags', 'status'].includes(field.api_key)) {
      categories.metadata.push(field);
    } else {
      categories.other.push(field);
    }
  }
  
  // Create fieldsets for non-empty categories
  const fieldsets = {};
  
  if (categories.content.length > 0) {
    fieldsets.content = await client.fieldsets.create(modelId, {
      title: 'Content',
      collapsible: false,
      position: 1
    });
  }
  
  if (categories.media.length > 0) {
    fieldsets.media = await client.fieldsets.create(modelId, {
      title: 'Media',
      collapsible: true,
      position: 2
    });
  }
  
  if (categories.seo.length > 0) {
    fieldsets.seo = await client.fieldsets.create(modelId, {
      title: 'SEO',
      collapsible: true,
      start_collapsed: true,
      position: 3
    });
  }
  
  if (categories.metadata.length > 0) {
    fieldsets.metadata = await client.fieldsets.create(modelId, {
      title: 'Metadata',
      collapsible: true,
      start_collapsed: true,
      position: 4
    });
  }
  
  // Move fields to appropriate fieldsets
  for (const [category, categoryFields] of Object.entries(categories)) {
    if (fieldsets[category] && categoryFields.length > 0) {
      for (const [index, field] of categoryFields.entries()) {
        await client.fields.update(field.id, {
          fieldset: {
            type: 'fieldset',
            id: fieldsets[category].id
          },
          position: index + 1
        });
      }
    }
  }
  
  return {
    fieldsets: Object.keys(fieldsets).length,
    organized: fields.length - categories.other.length,
    unorganized: categories.other.length
  };
}
```

## Common Fieldset Templates

```javascript
// Common fieldset templates
const fieldsetTemplates = {
  seo: {
    title: 'SEO',
    hint: 'Search engine optimization settings',
    collapsible: true,
    start_collapsed: true,
    fields: [
      {
        label: 'SEO Title',
        api_key: 'seo_title',
        field_type: 'string',
        validators: {
          length: { max: 60 }
        }
      },
      {
        label: 'SEO Description',
        api_key: 'seo_description',
        field_type: 'text',
        validators: {
          length: { max: 160 }
        }
      },
      {
        label: 'SEO Image',
        api_key: 'seo_image',
        field_type: 'file',
        validators: {
          file: {
            required_image_dimensions: true,
            image_dimensions: {
              width: { min: 1200 },
              height: { min: 630 }
            }
          }
        }
      }
    ]
  },
  
  social: {
    title: 'Social Media',
    hint: 'Social media sharing settings',
    collapsible: true,
    start_collapsed: true,
    fields: [
      {
        label: 'Facebook Title',
        api_key: 'og_title',
        field_type: 'string'
      },
      {
        label: 'Facebook Description',
        api_key: 'og_description',
        field_type: 'text'
      },
      {
        label: 'Twitter Card Type',
        api_key: 'twitter_card',
        field_type: 'string',
        appearance: {
          editor: 'single_line',
          parameters: {
            placeholder: 'summary_large_image'
          }
        }
      }
    ]
  },
  
  metadata: {
    title: 'Metadata',
    hint: 'Additional metadata',
    collapsible: true,
    start_collapsed: true,
    fields: [
      {
        label: 'Tags',
        api_key: 'tags',
        field_type: 'string',
        localized: false
      },
      {
        label: 'Custom Data',
        api_key: 'custom_data',
        field_type: 'json'
      }
    ]
  }
};

// Apply template
async function applyFieldsetTemplate(modelId, templateName) {
  const template = fieldsetTemplates[templateName];
  if (!template) {
    throw new Error(`Template ${templateName} not found`);
  }
  
  // Create fieldset
  const fieldset = await client.fieldsets.create(modelId, {
    title: template.title,
    hint: template.hint,
    collapsible: template.collapsible,
    start_collapsed: template.start_collapsed
  });
  
  // Create fields
  for (const [index, fieldData] of template.fields.entries()) {
    await client.fields.create(modelId, {
      ...fieldData,
      fieldset: {
        type: 'fieldset',
        id: fieldset.id
      },
      position: index + 1
    });
  }
  
  return fieldset;
}
```

## Error Handling

```javascript
import { ApiError } from '@datocms/rest-client-utils';

// Comprehensive error handling
async function safeFieldsetOperation() {
  try {
    const fieldset = await client.fieldsets.create('model_id', {
      title: 'Test Fieldset',
      collapsible: true
    });
    return fieldset;
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.response.status) {
        case 422:
          const validationErrors = error.response.data.errors;
          validationErrors.forEach(err => {
            if (err.source?.pointer?.includes('title')) {
              console.error('Title error:', err.detail);
            }
          });
          break;
        case 404:
          console.error('Model not found');
          break;
        case 403:
          console.error('Insufficient permissions to create fieldsets');
          break;
        default:
          console.error('Unexpected error:', error.response.status);
      }
    }
    throw error;
  }
}

// Validate before creation
async function validateFieldsetCreation(modelId, fieldsetData) {
  const errors = [];
  
  // Check title
  if (!fieldsetData.title || fieldsetData.title.trim() === '') {
    errors.push('Title is required');
  }
  
  // Check if model exists
  try {
    await client.itemTypes.find(modelId);
  } catch (error) {
    if (error.response?.status === 404) {
      errors.push('Model not found');
    }
  }
  
  return errors;
}
```

## Fieldset Object Structure

```typescript
interface Fieldset {
  id: string;
  title: string;
  hint?: string;
  position: number;
  collapsible: boolean;
  start_collapsed: boolean;
  item_type: {
    type: 'item_type';
    id: string;
  };
}
```

## Best Practices

1. **Organization**: Use fieldsets to logically group related fields
2. **Collapsibility**: Make optional or advanced sections collapsible
3. **Start Collapsed**: Collapse less frequently used fieldsets by default
4. **Clear Titles**: Use descriptive titles that clearly indicate content
5. **Helpful Hints**: Provide hints to explain the purpose of each fieldset
6. **Position Management**: Order fieldsets by importance and workflow
7. **Consistent Structure**: Maintain consistent fieldset organization across similar models
8. **Performance**: Use collapsible fieldsets to improve form performance with many fields

## Related Resources

- [Fields](./field.md) - Create and manage fields within fieldsets
- [Item Types](./item-type.md) - Models that contain fieldsets
- [Field Configuration](./field.md#appearance-configuration) - Configure field appearance