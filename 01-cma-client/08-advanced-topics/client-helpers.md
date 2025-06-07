# Client Helpers

This guide covers utility functions exported by the `@datocms/cma-client` package that are essential for certain workflows. These utilities help with building structured content, generating IDs, and other common operations.

## buildBlockRecord

The `buildBlockRecord` function is essential for creating block data for Modular Content, Structured Text, and Single Block fields. It takes a simple JavaScript object and transforms it into the JSON:API format that the DatoCMS API requires for a block.

### Function Signature

```typescript
import { buildBlockRecord } from '@datocms/cma-client';

function buildBlockRecord(body: SimpleSchemaTypes.ItemUpdateSchema): object
```

### Purpose

This function simplifies the creation of block records by:
- Converting simple JavaScript objects into proper JSON:API format
- Automatically handling the `item_type` relationship
- Supporting both creation (without ID) and update (with ID) scenarios
- Ensuring the correct structure for block records

### How It Works

The function transforms a simple object like:

```javascript
{
  text: 'Hello World',
  item_type: 'block_type_id'
}
```

Into the JSON:API format:

```javascript
{
  type: 'item',
  attributes: {
    text: 'Hello World'
  },
  relationships: {
    item_type: {
      data: {
        id: 'block_type_id',
        type: 'item_type'
      }
    }
  }
}
```

### Complete Example: Creating an Article with Modular Content

```typescript
import { buildClient, buildBlockRecord } from '@datocms/cma-client';

const client = buildClient({
  apiToken: 'YOUR_API_TOKEN'
});

async function createArticleWithBlocks() {
  try {
    // First, find the item types we'll use
    const itemTypes = await client.itemTypes.list();
    const articleType = itemTypes.find(it => it.attributes.api_key === 'article');
    const quoteBlockType = itemTypes.find(it => it.attributes.api_key === 'quote_block');
    const imageBlockType = itemTypes.find(it => it.attributes.api_key === 'image_block');

    // Create an article with modular content
    const article = await client.items.create({
      item_type: articleType.id,
      title: 'Understanding DatoCMS Blocks',
      slug: 'understanding-datocms-blocks',
      // Modular content field with multiple blocks
      content: [
        // Quote Block
        buildBlockRecord({
          quote_text: 'Content is king, but distribution is queen.',
          author: 'Jonathan Perelman',
          item_type: quoteBlockType.id
        }),
        // Image Block
        buildBlockRecord({
          image: {
            upload_id: 'your-upload-id',
            alt: 'Content management illustration',
            title: 'How content flows through DatoCMS'
          },
          caption: 'A visual representation of content management',
          item_type: imageBlockType.id
        })
      ]
    });

    console.log('Created article with blocks:', article);
    return article;

  } catch (error) {
    console.error('Error creating article:', error);
    throw error;
  }
}

// Example: Using buildBlockRecord with Structured Text
async function createStructuredTextWithBlocks() {
  try {
    const itemTypes = await client.itemTypes.list();
    const pageType = itemTypes.find(it => it.attributes.api_key === 'page');
    const ctaBlockType = itemTypes.find(it => it.attributes.api_key === 'cta_block');

    // Import structured text utilities
    const { StructuredText } = await import('datocms-structured-text-utils');
    const u = StructuredText.unist;

    // Create structured text with embedded block
    const structuredTextValue = u('root', [
      u('heading', { level: 1 }, [u('text', 'Welcome to our site')]),
      u('paragraph', [u('text', 'This is some introductory text.')]),
      // Embed a CTA block within the structured text
      u('block', {
        item: buildBlockRecord({
          title: 'Get Started Today',
          button_text: 'Sign Up',
          button_url: 'https://example.com/signup',
          item_type: ctaBlockType.id
        })
      }),
      u('paragraph', [u('text', 'Continue reading below...')])
    ]);

    const page = await client.items.create({
      item_type: pageType.id,
      title: 'Home Page',
      body: structuredTextValue
    });

    return page;

  } catch (error) {
    console.error('Error creating page:', error);
    throw error;
  }
}

// Example: Updating existing blocks
async function updateBlockContent(blockId: string) {
  try {
    // Fetch the existing block
    const block = await client.items.find(blockId);

    // Update using buildBlockRecord
    const updatedBlock = await client.items.update(blockId, {
      item_type: block.relationships.item_type.data.id,
      // buildBlockRecord handles the transformation
      ...buildBlockRecord({
        id: blockId,
        quote_text: 'Updated quote text',
        author: block.attributes.author + ' (Updated)',
        item_type: block.relationships.item_type.data.id
      }).attributes
    });

    return updatedBlock;

  } catch (error) {
    console.error('Error updating block:', error);
    throw error;
  }
}

// Example: Single Block field
async function createItemWithSingleBlock() {
  try {
    const itemTypes = await client.itemTypes.list();
    const productType = itemTypes.find(it => it.attributes.api_key === 'product');
    const heroBlockType = itemTypes.find(it => it.attributes.api_key === 'hero_block');

    const product = await client.items.create({
      item_type: productType.id,
      name: 'Premium Package',
      // Single block field
      hero: buildBlockRecord({
        headline: 'Transform Your Business',
        subheadline: 'With our premium features',
        background_image: {
          upload_id: 'hero-upload-id'
        },
        item_type: heroBlockType.id
      })
    });

    return product;

  } catch (error) {
    console.error('Error creating product:', error);
    throw error;
  }
}
```

## ID Utilities

The `@datocms/cma-client` package exports utility functions for working with DatoCMS IDs, which are URL-safe base64-encoded UUIDs.

### generateId()

Generates a new, client-side, URL-safe, base64-encoded UUID. This ID can be passed to the create() method of any resource (items, itemTypes, fields, etc.) to set an explicit ID before creation, which is useful for data migrations or integrations.

```typescript
import { generateId } from '@datocms/cma-client';

function generateId(): string
```

**Returns**: A 22-character URL-safe base64 string representing a v4 UUID

**Key Features**:
- Generates RFC 4122 v4 (random) UUIDs
- URL-safe format (uses `-` and `_` instead of `+` and `/`)
- Always 22 characters long
- First character is guaranteed to be alphanumeric

### isValidId()

Validates if a given string is a valid DatoCMS ID. This includes both modern UUID-based IDs and legacy integer IDs.

```typescript
import { isValidId } from '@datocms/cma-client';

function isValidId(id: string): boolean
```

**Parameters**:
- `id`: The string to validate

**Returns**: `true` if the ID is valid, `false` otherwise

**Validation Rules**:
- Modern IDs: Must be valid v4 UUIDs in URL-safe base64 format
- Legacy IDs: Must be numeric strings ≤ 281474976710655
- Empty strings return `false`

### Complete Examples

#### Creating an ItemType with Pre-generated ID

```typescript
import { buildClient, generateId, isValidId } from '@datocms/cma-client';

const client = buildClient({
  apiToken: 'YOUR_API_TOKEN'
});

async function createItemTypeWithExplicitId() {
  try {
    // Generate ID before creation
    const itemTypeId = generateId();
    console.log('Generated ID:', itemTypeId); // e.g., "WTyssHtyTzu9_EbszSVhPw"

    // Create item type with explicit ID
    const itemType = await client.itemTypes.create({
      id: itemTypeId,
      name: 'Blog Post',
      api_key: 'blog_post',
      modular_block: false
    });

    console.log('Created item type with ID:', itemType.id);

    // Later, you can reference this ID directly
    const field = await client.fields.create(itemTypeId, {
      label: 'Title',
      field_type: 'string',
      api_key: 'title'
    });

    return itemType;

  } catch (error) {
    console.error('Error creating item type:', error);
    throw error;
  }
}
```

#### Validating IDs Before Use

```typescript
async function safelyFindItem(providedId: string) {
  try {
    // Validate ID before making API call
    if (!isValidId(providedId)) {
      throw new Error(`Invalid DatoCMS ID: ${providedId}`);
    }

    const item = await client.items.find(providedId);
    return item;

  } catch (error) {
    console.error('Error finding item:', error);
    throw error;
  }
}

// Examples of ID validation
console.log(isValidId('WTyssHtyTzu9_EbszSVhPw')); // true - valid modern ID
console.log(isValidId('123456'));                  // true - valid legacy ID
console.log(isValidId('foobar'));                  // false - invalid format
console.log(isValidId(''));                        // false - empty string
console.log(isValidId('123/456'));                 // false - contains invalid characters
```

#### Bulk Import with Pre-generated IDs

```typescript
async function bulkImportWithIds(data: any[]) {
  try {
    // Pre-generate all IDs for referential integrity
    const idMap = new Map();
    
    // First pass: generate IDs
    data.forEach(item => {
      const newId = generateId();
      idMap.set(item.externalId, newId);
    });

    // Second pass: create items with references
    for (const item of data) {
      const itemId = idMap.get(item.externalId);
      
      await client.items.create({
        id: itemId,
        item_type: item.itemTypeId,
        title: item.title,
        // Reference another item by its pre-generated ID
        related_item: item.relatedExternalId 
          ? { type: 'item', id: idMap.get(item.relatedExternalId) }
          : null
      });
    }

    console.log('Bulk import completed');
    return idMap;

  } catch (error) {
    console.error('Error in bulk import:', error);
    throw error;
  }
}
```

#### Migration Script Example

```typescript
async function migrateContentWithStableIds() {
  try {
    const migrationMap = new Map();

    // Create item types with stable IDs
    const articleTypeId = generateId();
    const authorTypeId = generateId();

    await client.itemTypes.create({
      id: articleTypeId,
      name: 'Article',
      api_key: 'article'
    });

    await client.itemTypes.create({
      id: authorTypeId,
      name: 'Author',
      api_key: 'author'
    });

    // Create relationship field using known IDs
    await client.fields.create(articleTypeId, {
      label: 'Author',
      field_type: 'link',
      api_key: 'author',
      validators: {
        item_item_type: {
          item_types: [authorTypeId]
        }
      }
    });

    // Store migration data
    migrationMap.set('article_type', articleTypeId);
    migrationMap.set('author_type', authorTypeId);

    // Save for future reference or rollback
    console.log('Migration IDs:', Object.fromEntries(migrationMap));

    return migrationMap;

  } catch (error) {
    console.error('Migration failed:', error);
    throw error;
  }
}
```

## Best Practices

### Using buildBlockRecord

1. **Always specify item_type**: The item_type field is required for all blocks
2. **Use with modular blocks only**: Ensure the referenced item_type has `modular_block: true`
3. **Validate block types**: Check that fields match the block's field schema
4. **Handle nested structures**: For complex blocks with JSON fields, structure data appropriately

### Using ID Utilities

1. **Pre-generate for references**: When creating related items, generate IDs first
2. **Validate external IDs**: Always validate IDs from external sources
3. **Store generated IDs**: Keep track of generated IDs for rollback or debugging
4. **Use for idempotency**: Pre-generated IDs can make operations idempotent

## Common Pitfalls

1. **Forgetting item_type**: buildBlockRecord requires an item_type - omitting it causes errors
2. **Using wrong block type**: Ensure the item_type is actually a modular block
3. **Invalid ID format**: Don't modify generated IDs - use them as-is
4. **ID collisions**: While extremely unlikely, generated IDs could theoretically collide

## Related Documentation

- [Item Methods](../01-cma-client/01-content-management/item-methods.md) - Creating items with blocks
- [Field Methods](../01-cma-client/01-content-management/field-methods.md) - Understanding block field types
- [Filtering Guide](./filtering-guide.md) - Filtering items by block content
- [Batch Operations](./batch-operations.md) - Bulk creating items with IDs