# Field Resource

## Overview

Fields define the structure and content types within your DatoCMS models (item types). Each field has a specific type, validation rules, and appearance configuration that determines how content editors interact with it. Fields are the building blocks that define what kind of data can be stored in your content items - from simple text and numbers to complex relationships and structured content.

## API Client Section

`client.fields`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a new field in an item type
const field = await client.fields.create('article-model-id', {
  label: 'Article Title',
  field_type: 'string',
  api_key: 'title',
  validators: {
    required: {}
  }
});
```

## API Reference

### list() - List Fields

Retrieve all fields for a specific item type (model).

**Signature**: `list(itemTypeId, queryParams?): Promise<Field[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID (required) |
| `orderBy` | string | Sort order (e.g., 'position_ASC') |

**Example**:
```typescript
// Get all fields for a model
const fields = await client.fields.list('model_id');

// Get fields with ordering
const orderedFields = await client.fields.list('model_id', {
  orderBy: 'position_ASC'
});

// Get raw response with metadata
const response = await client.fields.rawList('model_id');
// response.meta contains pagination info
```

### find() - Get Single Field

Retrieve a single field by ID.

**Signature**: `find(fieldId): Promise<Field>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `fieldId` | string | The field ID (required) |

**Example**:
```typescript
const field = await client.fields.find('field_id');

// Error handling
try {
  const field = await client.fields.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Field not found');
  }
}
```

### create() - Create Field

Create a new field within an item type. This operation is asynchronous and returns a job.

**Signature**: `create(itemTypeId, fieldData): Promise<Job>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemTypeId` | string | The item type ID (required) |
| `label` | string | Human-readable field label (required) |
| `field_type` | string | The field type (required) |
| `api_key` | string | API identifier for the field (required) |
| `localized` | boolean | Enable localization for this field |
| `validators` | object | Validation rules |
| `appearance` | object | Editor configuration |
| `position` | number | Field order position |
| `hint` | string | Help text for editors |
| `default_value` | any | Default value(s) |
| `deep_filtering_enabled` | boolean | Enable deep filtering for relationships |
| `fieldset` | object | Group field in a fieldset |

```typescript
// Create basic text field
const textFieldJob = await client.fields.create('model_id', {
  label: 'Title',
  api_key: 'title',
  field_type: 'string',
  validators: {
    required: {},
    length: { max: 200 }
  },
  appearance: {
    editor: 'single_line',
    parameters: { heading: false }
  }
});
const textField = await client.jobResults.wait(textFieldJob.id);

// Create localized field
const localizedFieldJob = await client.fields.create('model_id', {
  label: 'Description',
  api_key: 'description',
  field_type: 'text',
  localized: true,
  validators: { required: {} },
  appearance: {
    editor: 'wysiwyg',
    parameters: {}
  }
});
const localizedField = await client.jobResults.wait(localizedFieldJob.id);

// Create relationship field
const linkFieldJob = await client.fields.create('model_id', {
  label: 'Author',
  api_key: 'author',
  field_type: 'link',
  validators: {
    required: {},
    item_item_type: { item_types: ['author_model_id'] }
  },
  appearance: {
    editor: 'link_select',
    parameters: {}
  }
});
const linkField = await client.jobResults.wait(linkFieldJob.id);

// Create modular content field
const modularFieldJob = await client.fields.create('model_id', {
  label: 'Page Sections',
  api_key: 'page_sections',
  field_type: 'modular_content',
  validators: {
    rich_text_blocks: {
      item_types: ['hero_block', 'content_block', 'gallery_block']
    }
  }
});
const modularField = await client.jobResults.wait(modularFieldJob.id);

// Create field with fieldset
const fieldInFieldsetJob = await client.fields.create('model_id', {
  label: 'SEO Title',
  api_key: 'seo_title',
  field_type: 'string',
  fieldset: {
    type: 'fieldset',
    id: 'fieldset_id'
  }
});
const fieldInFieldset = await client.jobResults.wait(fieldInFieldsetJob.id);

// Create with position
const positionedFieldJob = await client.fields.create('model_id', {
  label: 'Featured',
  api_key: 'featured',
  field_type: 'boolean',
  position: 1 // Place at top
});
const positionedField = await client.jobResults.wait(positionedFieldJob.id);

// Error handling
try {
  const job = await client.fields.create('model_id', {
    label: 'Invalid Field',
    api_key: 'invalid-key!', // Invalid characters
    field_type: 'string'
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation error:', error.response.data.errors);
  }
}
```

**Returns**: Job object for tracking the operation

### update() - Update Field

Update an existing field. This operation is asynchronous.

**Signature**: `update(fieldId, updateData): Promise<Job>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `fieldId` | string | The field ID (required) |
| `label` | string | Updated field label |
| `hint` | string | Updated help text |
| `validators` | object | Updated validation rules |
| `appearance` | object | Updated editor configuration |
| `position` | number | Updated field position |
| `localized` | boolean | Enable/disable localization |
| `fieldset` | object | Move to different fieldset |
| `default_value` | any | Updated default value |

**Note**: You cannot change the `field_type` of an existing field

**Example**:

```typescript
// Update field label
const updateLabelJob = await client.fields.update('field_id', {
  label: 'Updated Title'
});
const updatedLabel = await client.jobResults.wait(updateLabelJob.id);

// Update validators
const updateValidatorsJob = await client.fields.update('field_id', {
  validators: {
    required: {},
    length: { min: 10, max: 500 }
  }
});
const updatedValidators = await client.jobResults.wait(updateValidatorsJob.id);

// Update appearance
const updateAppearanceJob = await client.fields.update('field_id', {
  appearance: {
    editor: 'wysiwyg',
    parameters: {
      toolbar: ['bold', 'italic', 'link']
    }
  }
});
const updatedAppearance = await client.jobResults.wait(updateAppearanceJob.id);

// Move to fieldset
const moveToFieldsetJob = await client.fields.update('field_id', {
  fieldset: {
    type: 'fieldset',
    id: 'new_fieldset_id'
  }
});
const movedField = await client.jobResults.wait(moveToFieldsetJob.id);

// Update position
const updatePositionJob = await client.fields.update('field_id', {
  position: 5
});
const repositioned = await client.jobResults.wait(updatePositionJob.id);

// Enable localization
const enableLocalizationJob = await client.fields.update('field_id', {
  localized: true
});
const localized = await client.jobResults.wait(enableLocalizationJob.id);

// Error handling
try {
  const job = await client.fields.update('field_id', {
    field_type: 'text' // Cannot change field type
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Cannot change field type after creation');
  }
}
```

### destroy() - Delete Field

Delete a field and all its data. This operation is asynchronous.

**Signature**: `destroy(fieldId): Promise<Job>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `fieldId` | string | The field ID (required) |

**Warning**: Deleting a field will permanently remove all data stored in that field across all items

**Example**:

```typescript
// Delete field
const deleteJob = await client.fields.destroy('field_id');
await client.jobResults.wait(deleteJob.id);

// Delete with error handling
try {
  const job = await client.fields.destroy('field_id');
  await client.jobResults.wait(job.id);
  console.log('Field deleted successfully');
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Cannot delete field with existing data');
  } else if (error.response?.status === 404) {
    console.error('Field not found');
  }
}

// Safe delete pattern
async function safeDeleteField(fieldId) {
  try {
    // First, check if field has data
    const field = await client.fields.find(fieldId);
    
    // Clear field data from all items if needed
    const items = await client.items.list({
      filter: { type: field.relationships.item_type.data.id }
    });
    
    for (const item of items) {
      if (item[field.api_key]) {
        await client.items.update(item.id, {
          [field.api_key]: null
        });
      }
    }
    
    // Then delete the field
    const job = await client.fields.destroy(fieldId);
    await client.jobResults.wait(job.id);
  } catch (error) {
    console.error('Failed to delete field:', error);
  }
}
```

**Returns**: Job object for tracking the operation

### duplicate() - Duplicate Field

Create a copy of an existing field. This operation is asynchronous.

**Signature**: `duplicate(fieldId): Promise<Job>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `fieldId` | string | The field ID to duplicate (required) |

**Example**:

```typescript
const duplicateJob = await client.fields.duplicate('field_id');

// Get the new field
const result = await client.jobResults.wait(duplicateJob.id);
const duplicatedField = result.payload.data;

// Clone field to different model
async function cloneField(sourceFieldId, targetModelId) {
  const sourceField = await client.fields.find(sourceFieldId);
  
  // Remove ID and relationships
  const { id, type, attributes, relationships, ...rest } = sourceField;
  
  const createJob = await client.fields.create(targetModelId, {
    label: sourceField.label + ' (Copy)',
    api_key: sourceField.api_key + '_copy',
    field_type: sourceField.field_type,
    validators: sourceField.validators,
    localized: sourceField.localized,
    default_value: sourceField.default_value,
    hint: sourceField.hint,
    appearance: sourceField.appearance
  });
  
  const newField = await client.jobResults.wait(createJob.id);
  return newField;
}
```

**Returns**: Job object containing the new field

## Field Types

### Text Fields

#### Single-line String (`string`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Name',
  field_type: 'string',
  api_key: 'name',
  appearance: {
    editor: 'single_line',
    parameters: {
      placeholder: 'Enter name...',
      prefix: '@',
      suffix: '.com'
    }
  }
});
```

**Value Type:** `string`
**Example:** `"Hello World"`

#### Multi-line Text (`text`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Description',
  field_type: 'text',
  api_key: 'description',
  appearance: {
    editor: 'textarea',
    parameters: {
      rows: 5
    }
  }
});
```

**Value Type:** `string`
**Example:** `"Long text content with multiple lines..."`

#### Slug (`slug`)
```typescript
await client.fields.create('item-type-id', {
  label: 'URL Slug',
  field_type: 'slug',
  api_key: 'slug',
  validators: {
    required: {},
    unique: {},
    slug_title_field: {
      title_field_id: 'title-field-id' // Auto-generate from title
    }
  },
  appearance: {
    editor: 'slug',
    parameters: {
      prefix: '/blog/'
    }
  }
});
```

**Value Type:** `string`
**Example:** `"hello-world"`

### Number Fields

#### Integer (`integer`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Quantity',
  field_type: 'integer',
  api_key: 'quantity',
  validators: {
    number_range: { min: 0, max: 1000 }
  }
});
```

**Value Type:** `number`
**Example:** `42`

#### Float (`float`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Price',
  field_type: 'float',
  api_key: 'price',
  validators: {
    number_range: { min: 0 }
  },
  appearance: {
    editor: 'float',
    parameters: {
      prefix: '$',
      precision: 2
    }
  }
});
```

**Value Type:** `number`
**Example:** `19.99`

### Boolean Field (`boolean`)

```typescript
await client.fields.create('item-type-id', {
  label: 'Published',
  field_type: 'boolean',
  api_key: 'published',
  default_value: { en: false },
  appearance: {
    editor: 'boolean_switch' // or 'boolean' for checkbox
  }
});
```

**Value Type:** `boolean`
**Example:** `true` or `false`

### Date/Time Fields

#### Date (`date`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Event Date',
  field_type: 'date',
  api_key: 'event_date',
  validators: {
    date_range: { 
      min: '2024-01-01',
      max: '2025-12-31'
    }
  }
});
```

**Value Type:** `string` (ISO 8601 date)
**Example:** `"2024-01-15"`

#### DateTime (`datetime`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Published At',
  field_type: 'datetime',
  api_key: 'published_at',
  appearance: {
    editor: 'date_time_picker',
    parameters: {
      time_format: '24' // or '12'
    }
  }
});
```

**Value Type:** `string` (ISO 8601 datetime)
**Example:** `"2024-01-15T14:30:00Z"`

### Media Fields

#### Single File (`file`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Featured Image',
  field_type: 'file',
  api_key: 'featured_image',
  validators: {
    required: {},
    extension: { 
      predefined_list: 'image' // or custom: ['jpg', 'png']
    },
    file_size: {
      max: 5242880 // 5MB in bytes
    },
    dimension: {
      width: { min: 800 },
      height: { min: 600 }
    }
  }
});
```

**Value Type:** `object`
**Example:**
```javascript
{
  upload_id: "upload-123",
  alt: "Alt text",
  title: "Image title"
}
```

#### Gallery (`gallery`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Product Images',
  field_type: 'gallery',
  api_key: 'images',
  validators: {
    size: { min: 1, max: 10 }
  }
});
```

**Value Type:** `array`
**Example:**
```javascript
[
  { upload_id: "upload-1", alt: "Image 1" },
  { upload_id: "upload-2", alt: "Image 2" }
]
```

### Relationship Fields

#### Single Link (`link`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Author',
  field_type: 'link',
  api_key: 'author',
  validators: {
    item_item_type: {
      on_publish_with_unpublished_references_strategy: 'fail',
      on_reference_unpublish_strategy: 'delete_references',
      on_reference_delete_strategy: 'delete_references',
      item_types: ['author-model-id']
    }
  },
  deep_filtering_enabled: true
});
```

**Value Type:** `string` (item ID)
**Example:** `"author-123"`

#### Multiple Links (`links`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Related Articles',
  field_type: 'links',
  api_key: 'related_articles',
  validators: {
    items_item_type: {
      item_types: ['article-model-id'],
      on_publish_with_unpublished_references_strategy: 'fail'
    },
    size: { max: 5 }
  }
});
```

**Value Type:** `array` (of item IDs)
**Example:** `["article-1", "article-2", "article-3"]`

#### Modular Content (`modular_content`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Page Sections',
  field_type: 'modular_content',
  api_key: 'sections',
  validators: {
    rich_text_blocks: {
      item_types: ['hero-block-id', 'text-block-id', 'cta-block-id']
    }
  }
});
```

**Value Type:** `array` (of block objects)
**Example:**
```javascript
[
  {
    item_type: "hero_section",
    headline: "Welcome"
  },
  {
    item_type: "text_section",
    content: "Content..."
  }
]
```

**For a complete guide on working with Modular Content, see the [Modular Content Guide](./modular-content.md).**

### Complex Fields

#### Structured Text (`structured_text`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Rich Content',
  field_type: 'structured_text',
  api_key: 'content',
  validators: {
    structured_text_blocks: {
      item_types: ['quote-block-id', 'gallery-block-id']
    },
    structured_text_links: {
      item_types: ['article-model-id', 'page-model-id']
    }
  },
  appearance: {
    editor: 'structured_text',
    parameters: {
      marks: ['strong', 'emphasis', 'code', 'highlight'],
      nodes: ['heading', 'quote', 'list', 'code'],
      heading_levels: [2, 3, 4]
    }
  }
});
```

**Value Type:** `object` (Structured Text AST)

**For a complete guide on working with Structured Text, see the [Structured Text Guide](./structured-text.md).**

#### JSON (`json`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Configuration',
  field_type: 'json',
  api_key: 'config',
  appearance: {
    editor: 'json',
    parameters: {
      schema: {
        type: 'object',
        properties: {
          key: { type: 'string' },
          value: { type: 'number' }
        }
      }
    }
  }
});
```

**Value Type:** `object` (arbitrary JSON)
**Example:**
```javascript
{
  custom: "data",
  nested: { values: [1, 2, 3] }
}
```

#### Color (`color`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Brand Color',
  field_type: 'color',
  api_key: 'brand_color',
  default_value: { en: { hex: '#FF6B6B', alpha: 100 } }
});
```

**Value Type:** `object`
**Example:**
```javascript
{
  red: 255,
  green: 87,
  blue: 51,
  alpha: 1,
  hex: "#FF5733"
}
```

#### Geographic Location (`lat_lon`)
```typescript
await client.fields.create('item-type-id', {
  label: 'Store Location',
  field_type: 'lat_lon',
  api_key: 'location',
  default_value: { 
    en: { 
      latitude: 40.7128,
      longitude: -74.0060 
    } 
  }
});
```

**Value Type:** `object`
**Example:**
```javascript
{
  latitude: 40.7128,
  longitude: -74.0060
}
```

#### SEO (`seo`)
```typescript
await client.fields.create('item-type-id', {
  label: 'SEO',
  field_type: 'seo',
  api_key: 'seo',
  appearance: {
    editor: 'seo',
    parameters: {
      fields: ['title', 'description', 'image', 'no_index']
    }
  }
});
```

**Value Type:** `object`
**Example:**
```javascript
{
  title: "Page Title",
  description: "Meta description",
  image: { upload_id: "123" }
}
```

## Validators

### Common Validators

```typescript
// Required field
validators: {
  required: {}
}

// Unique value
validators: {
  unique: {}
}
```

### Text Validators

```typescript
// Length constraints
validators: {
  length: {
    min: 10,
    max: 100,
    eq: 50 // Exact length
  }
}

// Format/Pattern matching
validators: {
  format: {
    // Predefined formats
    predefined_pattern: 'email' // or 'url'
  }
}

// Custom regex
validators: {
  format: {
    custom_pattern: '^[A-Z]{2}-\\d{4}$'
  }
}
```

### Number Validators

```typescript
// Range
validators: {
  number_range: {
    min: 0,
    max: 100
  }
}

// Multiple of
validators: {
  number_multiple_of: 5 // Must be 0, 5, 10, 15...
}
```

### Date Validators

```typescript
validators: {
  date_range: {
    min: '2024-01-01',
    max: '2024-12-31'
  }
}
```

### File Validators

```typescript
validators: {
  // File size in bytes
  file_size: {
    min: 1024,      // 1KB minimum
    max: 10485760   // 10MB maximum
  },
  
  // Extensions
  extension: {
    predefined_list: 'image', // 'image', 'video', 'document'
    // OR custom list:
    extensions: ['jpg', 'png', 'webp']
  },
  
  // MIME types
  mime_type: {
    mime_types: ['image/jpeg', 'image/png']
  },
  
  // Image dimensions
  dimension: {
    width: { min: 800, max: 2000, eq: 1920 },
    height: { min: 600, max: 1500 },
    ratio: { min: 1.5, max: 2 } // Aspect ratio
  }
}
```

### Collection Validators

```typescript
// For gallery, links, modular_content
validators: {
  size: {
    min: 1,
    max: 10,
    eq: 5 // Exactly 5 items
  }
}
```

### Relationship Validators

```typescript
// Single link
validators: {
  item_item_type: {
    item_types: ['author-id', 'editor-id'],
    on_publish_with_unpublished_references_strategy: 'fail', // or 'publish_references'
    on_reference_unpublish_strategy: 'delete_references', // or 'fail'
    on_reference_delete_strategy: 'delete_references' // or 'fail'
  }
}

// Multiple links
validators: {
  items_item_type: {
    item_types: ['article-id'],
    on_publish_with_unpublished_references_strategy: 'fail'
  }
}
```

### Validator Compatibility Matrix

| Field Type | Required | Unique | Length | Format | Range | Size | Custom |
|------------|----------|--------|--------|--------|-------|------|--------|
| string | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ✅ |
| text | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ |
| boolean | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| integer | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ |
| float | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ |
| date | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| datetime | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| color | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |
| json | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| lat_lon | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| seo | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| slug | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| file | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| gallery | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| link | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| links | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| modular_content | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ |
| structured_text | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

## Appearance Configuration

### String Field Appearances

#### Single Line
```typescript
appearance: {
  editor: 'single_line',
  parameters: {
    heading: false,
    placeholder: 'Enter title here...',
    prefix: '@',
    suffix: '.com'
  }
}
```

#### Dropdown Select
```typescript
appearance: {
  editor: 'string_select',
  parameters: {
    options: [
      { label: 'Small', value: 'sm' },
      { label: 'Medium', value: 'md' },
      { label: 'Large', value: 'lg' }
    ]
  }
}
```

#### Radio Buttons
```typescript
appearance: {
  editor: 'string_radio_group',
  parameters: {
    options: [
      { label: 'Yes', value: 'yes' },
      { label: 'No', value: 'no' }
    ]
  }
}
```

### Text Field Appearances

#### Textarea
```typescript
appearance: {
  editor: 'textarea',
  parameters: {
    placeholder: 'Enter description...',
    rows: 5
  }
}
```

#### Markdown Editor
```typescript
appearance: {
  editor: 'markdown',
  parameters: {
    preview: true,
    toolbar: ['bold', 'italic', 'link', 'code', 'quote']
  }
}
```

#### WYSIWYG Editor
```typescript
appearance: {
  editor: 'wysiwyg',
  parameters: {
    toolbar: ['bold', 'italic', 'link', 'bulletList', 'numberedList']
  }
}
```

### Number Field Appearances

#### Integer
```typescript
appearance: {
  editor: 'integer',
  parameters: {
    prefix: '$',
    suffix: 'USD',
    placeholder: '0'
  }
}
```

#### Rating Stars
```typescript
appearance: {
  editor: 'integer_rating',
  parameters: {
    stars: 5
  }
}
```

#### Float
```typescript
appearance: {
  editor: 'float',
  parameters: {
    prefix: '€',
    precision: 2,
    placeholder: '0.00'
  }
}
```

### Boolean Field Appearances

#### Checkbox
```typescript
appearance: {
  editor: 'boolean',
  parameters: {
    label: 'I agree to the terms'
  }
}
```

#### Switch
```typescript
appearance: {
  editor: 'boolean_switch',
  parameters: {
    positive_label: 'Active',
    negative_label: 'Inactive'
  }
}
```

### Date/Time Appearances

#### Date Picker
```typescript
appearance: {
  editor: 'date_picker',
  parameters: {
    placeholder: 'Select date',
    format: 'YYYY-MM-DD'
  }
}
```

#### DateTime Picker
```typescript
appearance: {
  editor: 'date_time_picker',
  parameters: {
    time_format: '24', // or '12'
    display_utc: true
  }
}
```

### Relationship Appearances

#### Link Select
```typescript
appearance: {
  editor: 'link_select',
  parameters: {
    placeholder: 'Select an item'
  }
}
```

#### Link Embed
```typescript
appearance: {
  editor: 'link_embed',
  parameters: {
    start_collapsed: false
  }
}
```

#### Links Embed
```typescript
appearance: {
  editor: 'links_embed',
  parameters: {
    start_collapsed: true,
    sortable: true
  }
}
```

### Color Field Appearance
```typescript
appearance: {
  editor: 'color_picker',
  parameters: {
    preset_colors: [
      { label: 'Primary', value: { r: 0, g: 123, b: 255, a: 1 } },
      { label: 'Secondary', value: { r: 108, g: 117, b: 125, a: 1 } }
    ],
    enable_alpha: true
  }
}
```

### Map Picker Appearance
```typescript
appearance: {
  editor: 'map',
  parameters: {
    initial_latitude: 40.7128,
    initial_longitude: -74.0060,
    initial_zoom: 10,
    map_style: 'streets'
  }
}
```

### JSON Editor Appearance
```typescript
appearance: {
  editor: 'json',
  parameters: {
    schema: {
      type: 'object',
      properties: {
        name: { type: 'string' },
        age: { type: 'number' }
      },
      required: ['name']
    }
  }
}
```

## Advanced Usage

### Field Localization

Enable localization to store different values per locale:

```typescript
await client.fields.create('item-type-id', {
  label: 'Title',
  field_type: 'string',
  api_key: 'title',
  localized: true,
  default_value: {
    en: 'English Title',
    it: 'Titolo Italiano',
    es: 'Título Español'
  }
});
```

### Fieldsets (Grouping Fields)

Group related fields together:

```typescript
// First create a fieldset
const fieldset = await client.fieldsets.create('item-type-id', {
  title: 'Pricing Information',
  hint: 'Product pricing details',
  position: 1,
  collapsible: true,
  start_collapsed: false
});

// Then assign fields to it
await client.fields.create('item-type-id', {
  label: 'Price',
  field_type: 'float',
  api_key: 'price',
  fieldset: {
    type: 'fieldset',
    id: fieldset.id
  }
});
```

**Related:** See [Fieldsets](./fieldset.md) for managing field groups

### Custom Field Extensions

Use plugins to create custom field editors:

```typescript
await client.fields.create('item-type-id', {
  label: 'Custom Field',
  field_type: 'string',
  api_key: 'custom',
  appearance: {
    editor: 'string',
    parameters: {},
    addons: [
      {
        id: 'plugin-id',
        parameters: {
          customParam: 'value'
        }
      }
    ]
  }
});
```

**Related:** See [Create Field Editor](../../03-plugin-sdk/02-hooks/renderFieldExtension.md) for building custom editors

### Deep Filtering

Enable deep filtering for relationship fields to allow filtering items by related content:

```typescript
await client.fields.create('item-type-id', {
  label: 'Category',
  field_type: 'link',
  api_key: 'category',
  deep_filtering_enabled: true
});

// Now you can filter items by category properties:
const items = await client.items.list({
  filter: {
    fields: {
      category: {
        fields: {
          name: { eq: 'Technology' }
        }
      }
    }
  }
});
```

### Field Position Management

Control field order in the editor:

```typescript
// Get all fields
const fields = await client.fields.list('item-type-id');

// Reorder fields
for (let i = 0; i < fields.length; i++) {
  await client.fields.update(fields[i].id, {
    position: i + 1
  });
}
```

### Bulk Field Creation

Create multiple fields at once:

```typescript
const fields = [
  {
    label: 'Title',
    field_type: 'string',
    api_key: 'title',
    validators: { required: {} }
  },
  {
    label: 'Slug',
    field_type: 'slug',
    api_key: 'slug',
    validators: { 
      required: {},
      slug_title_field: { title_field_id: 'title' }
    }
  },
  {
    label: 'Content',
    field_type: 'structured_text',
    api_key: 'content'
  },
  {
    label: 'Author',
    field_type: 'link',
    api_key: 'author',
    validators: {
      item_item_type: { item_types: ['author'] }
    }
  }
];

// Create all fields
for (const fieldConfig of fields) {
  const job = await client.fields.create(modelId, fieldConfig);
  await client.jobResults.wait(job.id);
  console.log(`Created field: ${fieldConfig.api_key}`);
}
```

### Dynamic Appearance Updates

Update appearance based on context:

```typescript
// Different appearance based on field type
async function setOptimalAppearance(fieldId) {
  const field = await client.fields.find(fieldId);
  
  let appearance = {};
  
  switch (field.field_type) {
    case 'string':
      if (field.validators.length?.max <= 50) {
        appearance = {
          editor: 'single_line',
          parameters: { placeholder: `Max ${field.validators.length.max} chars` }
        };
      } else {
        appearance = {
          editor: 'textarea',
          parameters: { rows: 3 }
        };
      }
      break;
      
    case 'text':
      appearance = {
        editor: field.api_key.includes('markdown') ? 'markdown' : 'textarea',
        parameters: { preview: true }
      };
      break;
      
    case 'integer':
      if (field.validators.number_range?.max <= 5) {
        appearance = {
          editor: 'integer_rating',
          parameters: { stars: field.validators.number_range.max }
        };
      }
      break;
  }
  
  if (appearance.editor) {
    const job = await client.fields.update(fieldId, { appearance });
    await client.jobResults.wait(job.id);
  }
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.fields.create('item-type-id', {
    label: 'Title',
    field_type: 'string',
    api_key: 'title' // Might already exist
  });
} catch (error) {
  if (error instanceof ApiError) {
    const apiKeyError = error.findError('api_key');
    if (apiKeyError?.code === 'VALIDATION_UNIQUE') {
      console.log('Field with this API key already exists');
    }
  }
}
```

## Field Object Structure

```typescript
interface Field {
  id: string;
  label: string;
  field_type: string;
  api_key: string;
  hint?: string;
  localized: boolean;
  validators: Record<string, any>;
  position: number;
  appearance: {
    editor: string;
    parameters: Record<string, any>;
    addons: Array<{
      id: string;
      parameters: Record<string, any>;
    }>;
  };
  default_value?: any;
  deep_filtering_enabled?: boolean;
  item_type: {
    type: 'item_type';
    id: string;
  };
  fieldset?: {
    type: 'fieldset';
    id: string;
  };
}
```

## Field Selection Guide

| Need | Use Field Type | Example |
|------|----------------|---------|
| Short text (< 255 chars) | `string` | Title, Name |
| Rich formatted text and Long text | `structured_text` | Article content |
| Yes/No choice | `boolean` | Published, Featured |
| Whole numbers | `integer` | Quantity, Views |
| Decimals | `float` | Price, Rating |
| Dates | `date` | Birth date |
| Date + time | `datetime` | Event time |
| Colors | `color` | Brand color |
| Coordinates | `lat_lon` | Store location |
| URL-friendly text | `slug` | URL path |
| Single image/file | `file` | Avatar, PDF |
| Multiple images | `gallery` | Product photos |
| Reference one item | `link` | Author |
| Reference many items | `links` | Categories |
| Inline blocks | `modular_content` | Page sections |
| Arbitrary data | `json` | Settings |
| SEO metadata | `seo` | Page SEO |
| Simple (poor) Long text (prefer structure text unless specified otherwise) | `text` | Description, Bio |

## Best Practices

1. **Plan field structure** before creation
2. **Use semantic API keys** (lowercase, underscores)
3. **Add helpful hints** for editors
4. **Set appropriate validators**
5. **Consider localization needs**
6. **Group related fields** with fieldsets
7. **Order fields logically**
8. **Match editor to use case** - Choose appropriate UI for data entry
9. **Use placeholders** - Guide users with helpful hints
10. **Configure constraints** - Set appropriate options/limits
11. **Consider mobile** - Some editors work better on mobile
12. **Test with content** - Verify appearance works with real data
13. **Document custom parameters** - For plugin appearances
14. **Provide defaults** - Set sensible default values
15. **Start with required fields** - Add required validation first
16. **Consider data migration** - Existing data must pass new validations
17. **Balance strictness** - Don't over-validate, consider UX

## Integration Patterns

### Form Validation in Plugins

```typescript
// Use validators in plugin field extensions
const field = await ctx.field;
const validators = field.attributes.validators;

// Check if field is required
if (validators.required) {
  // Show required indicator
}

// Get length constraints
if (validators.length) {
  const { min, max } = validators.length;
  // Show character counter
}
```

### Plugin Field Editors

```typescript
// Register custom field editor in plugin
export default function CustomFieldEditor({ ctx }) {
  const { field, parameters } = ctx;
  
  // Use appearance parameters
  const theme = parameters.theme || 'light';
  const showAdvanced = parameters.show_advanced || false;
  
  // Render custom UI based on parameters
  return (
    <div className={`editor-${theme}`}>
      {/* Custom editor implementation */}
    </div>
  );
}
```

### Validation Before Save

```typescript
// Validate all fields before creating item
const model = await client.itemTypes.find(modelId);
const fields = await client.fields.list(modelId);

// Check each field's validators
for (const field of fields) {
  if (field.validators.required && !itemData[field.api_key]) {
    throw new Error(`${field.label} is required`);
  }
}
```

## Bulk Operations

### Create Multiple Fields

```typescript
// Create multiple fields at once
async function createFieldSet(modelId, fields) {
  const createdFields = [];
  
  for (const [index, fieldData] of fields.entries()) {
    try {
      const job = await client.fields.create(modelId, {
        ...fieldData,
        position: index + 1
      });
      const field = await client.jobResults.wait(job.id);
      createdFields.push(field);
    } catch (error) {
      console.error(`Failed to create field ${fieldData.api_key}:`, error);
    }
  }
  
  return createdFields;
}

// Usage
const blogFields = [
  {
    label: 'Title',
    api_key: 'title',
    field_type: 'string',
    validators: { required: {} }
  },
  {
    label: 'Content',
    api_key: 'content',
    field_type: 'structured_text',
    validators: { required: {} }
  },
  {
    label: 'Author',
    api_key: 'author',
    field_type: 'link',
    validators: {
      item_item_type: { item_types: ['author'] }
    }
  }
];

const fields = await createFieldSet('blog_model_id', blogFields);
```

### Field Migration

```typescript
async function migrateFieldData(oldFieldId, newFieldId) {
  const oldField = await client.fields.find(oldFieldId);
  const newField = await client.fields.find(newFieldId);
  
  // Verify same model
  if (oldField.relationships.item_type.data.id !== 
      newField.relationships.item_type.data.id) {
    throw new Error('Fields must belong to same model');
  }
  
  const modelId = oldField.relationships.item_type.data.id;
  
  // Get all items
  const items = await client.items.list({
    filter: { type: modelId }
  });
  
  // Migrate data
  const results = await Promise.allSettled(
    items.map(async item => {
      if (item[oldField.api_key]) {
        return client.items.update(item.id, {
          [newField.api_key]: item[oldField.api_key],
          [oldField.api_key]: null
        });
      }
    })
  );
  
  const successful = results.filter(r => r.status === 'fulfilled').length;
  const failed = results.filter(r => r.status === 'rejected').length;
  
  return { successful, failed, total: items.length };
}
```

### Field Usage Analysis

```typescript
async function analyzeFieldUsage(fieldId) {
  const field = await client.fields.find(fieldId);
  const modelId = field.relationships.item_type.data.id;
  
  const items = await client.items.list({
    filter: { type: modelId }
  });
  
  const analysis = {
    fieldId,
    totalItems: items.length,
    usage: {
      filled: 0,
      empty: 0,
      locales: {}
    },
    values: {
      unique: new Set(),
      mostCommon: new Map()
    }
  };
  
  for (const item of items) {
    const value = item[field.api_key];
    
    if (value !== null && value !== undefined && value !== '') {
      analysis.usage.filled++;
      
      // Track unique values (for non-complex types)
      if (typeof value === 'string' || typeof value === 'number') {
        analysis.values.unique.add(value);
        
        // Count occurrences
        const count = analysis.values.mostCommon.get(value) || 0;
        analysis.values.mostCommon.set(value, count + 1);
      }
      
      // Track locale usage
      if (field.localized && typeof value === 'object') {
        Object.keys(value).forEach(locale => {
          analysis.usage.locales[locale] = 
            (analysis.usage.locales[locale] || 0) + 1;
        });
      }
    } else {
      analysis.usage.empty++;
    }
  }
  
  // Convert to sorted array
  analysis.values.mostCommon = Array.from(analysis.values.mostCommon)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 10);
  
  analysis.values.unique = analysis.values.unique.size;
  
  return analysis;
}
```

### Field Validation Testing

```typescript
async function testFieldValidation(fieldId, testValue) {
  const field = await client.fields.find(fieldId);
  const modelId = field.relationships.item_type.data.id;
  
  try {
    // Create test item
    const testItem = await client.items.create({
      item_type: { type: 'item_type', id: modelId },
      [field.api_key]: testValue
    });
    
    // Validate
    const validation = await client.items.validate(testItem.id);
    
    // Clean up
    await client.items.destroy(testItem.id);
    
    return {
      valid: validation.is_valid,
      errors: validation.errors
    };
  } catch (error) {
    return {
      valid: false,
      errors: error.response?.data?.errors || []
    };
  }
}
```

## Related Resources

- [Item Types](./item-type.md) - Define content models
- [Fieldsets](./fieldset.md) - Group related fields
- [Items](./item.md) - Work with content
- [Field Type Matrix](../../05-reference/field-type-matrix.md) - Field type reference