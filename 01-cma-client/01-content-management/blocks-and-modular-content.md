# Blocks and Modular Content

This guide provides comprehensive documentation for working with block-based content in DatoCMS, including Modular Content fields and Single Block fields using the DatoCMS CMA Client.

## Overview

DatoCMS offers powerful block-based field types that enable editors to create dynamic, flexible content structures. Each block is a reusable piece of content with its own fields, providing maximum flexibility for content modeling.

### Key Concepts

- **Block**: A reusable content structure (an Item Type with `modular_block: true`)
- **Modular Content Field**: A field type that contains an array of block records
- **Single Block Field**: A field type that contains exactly one block record
- **Block Record**: An instance of a block with specific content

## Field Type Terminology

**IMPORTANT**: DatoCMS uses different terms in the UI versus the API. This mapping is crucial for correctly using the API:

| UI Feature Name | API `field_type` | Description | Use Case |
|-----------------|------------------|-------------|----------|
| **Modular Content** | `rich_text` | An array of reorderable content blocks | Page builders, flexible layouts |
| **Single Block** | `single_block` | Exactly one block from allowed types | Featured components, one-off sections |
| **Structured Text** | `structured_text` | Rich text with inline blocks | Articles with embedded media |

**Note**: Despite the name, "Modular Content" in the UI corresponds to `field_type: 'rich_text'` in the API. This guide covers both Modular Content (`rich_text`) and Single Block (`single_block`) fields.

## Installation

```bash
npm install @datocms/cma-client
```

## Blocks Are Models

**This is the most important concept to understand**: A Block in DatoCMS is not a separate entity type. A Block is simply a standard Item Type (Model) that has the `modular_block` property set to `true`.

When you create an Item Type with `modular_block: true`, it becomes available as a Block that can be used within Modular Content fields and Structured Text fields.

### Creating Block Types

Here's how to create different types of blocks:

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function createBlockTypes() {
  // Create a Quote Block
  const quoteBlock = await client.itemTypes.create({
    name: 'Quote Block',
    api_key: 'quote_block',
    modular_block: true, // This makes it a block!
    inverse_relationships_enabled: false,
  });

  // Add fields to the Quote Block
  await client.fields.create(quoteBlock.id, {
    label: 'Quote Text',
    api_key: 'quote_text',
    field_type: 'text',
    validators: { required: {} },
  });

  await client.fields.create(quoteBlock.id, {
    label: 'Author',
    api_key: 'author',
    field_type: 'string',
    validators: { required: {} },
  });

  // Create an Image Gallery Block
  const imageGalleryBlock = await client.itemTypes.create({
    name: 'Image Gallery Block',
    api_key: 'image_gallery_block',
    modular_block: true,
    inverse_relationships_enabled: false,
  });

  await client.fields.create(imageGalleryBlock.id, {
    label: 'Images',
    api_key: 'images',
    field_type: 'gallery',
    validators: { required: {} },
  });

  await client.fields.create(imageGalleryBlock.id, {
    label: 'Caption',
    api_key: 'caption',
    field_type: 'string',
  });

  // Create a CTA Block
  const ctaBlock = await client.itemTypes.create({
    name: 'CTA Block',
    api_key: 'cta_block',
    modular_block: true,
    inverse_relationships_enabled: false,
  });

  await client.fields.create(ctaBlock.id, {
    label: 'Title',
    api_key: 'title',
    field_type: 'string',
    validators: { required: {} },
  });

  await client.fields.create(ctaBlock.id, {
    label: 'Button Text',
    api_key: 'button_text',
    field_type: 'string',
    validators: { required: {} },
  });

  await client.fields.create(ctaBlock.id, {
    label: 'Button URL',
    api_key: 'button_url',
    field_type: 'string',
    validators: { required: {}, format: { predefined_pattern: 'url' } },
  });

  return { quoteBlock, imageGalleryBlock, ctaBlock };
}
```

## Creating a Modular Content Field

To use blocks, you need to create a Modular Content field on a model and specify which block types it accepts:

```javascript
async function createArticleModelWithModularContent() {
  // First, get the IDs of the block types we want to allow
  const quoteBlock = await client.itemTypes.find('quote_block');
  const imageGalleryBlock = await client.itemTypes.find('image_gallery_block');
  const ctaBlock = await client.itemTypes.find('cta_block');

  // Create an Article model
  const articleModel = await client.itemTypes.create({
    name: 'Article',
    api_key: 'article',
    modular_block: false, // This is a regular model, not a block
  });

  // Add a Modular Content field
  await client.fields.create(articleModel.id, {
    label: 'Content Sections',
    api_key: 'content_sections',
    field_type: 'rich_text',
    validators: {
      rich_text_blocks: {
        item_types: [
          quoteBlock.id,
          imageGalleryBlock.id,
          ctaBlock.id
        ]
      }
    }
  });

  return articleModel;
}
```

## Creating Items with Modular Content

When creating items that contain Modular Content fields, use the `buildBlockRecord` helper to create block instances:

```javascript
import { buildClient, buildBlockRecord } from '@datocms/cma-client';

async function createArticleWithBlocks() {
  // Get the model and block type IDs
  const articleModel = await client.itemTypes.find('article');
  const quoteBlockType = await client.itemTypes.find('quote_block');
  const imageGalleryBlockType = await client.itemTypes.find('image_gallery_block');
  const ctaBlockType = await client.itemTypes.find('cta_block');

  // Upload some images for the gallery
  const upload1 = await client.uploads.create({
    path: 'https://www.datocms-assets.com/123/image1.jpg'
  });
  const upload2 = await client.uploads.create({
    path: 'https://www.datocms-assets.com/123/image2.jpg'
  });

  // Create the article with multiple blocks
  const article = await client.items.create({
    item_type: { type: 'item_type', id: articleModel.id },
    content_sections: [
      // First block: Quote
      buildBlockRecord({
        item_type: { type: 'item_type', id: quoteBlockType.id },
        quote_text: 'Content is king, but distribution is queen and she wears the pants.',
        author: 'Jonathan Perelman'
      }),
      
      // Second block: Image Gallery
      buildBlockRecord({
        item_type: { type: 'item_type', id: imageGalleryBlockType.id },
        images: [
          { upload_id: upload1.id, alt: 'First image', title: 'Image 1' },
          { upload_id: upload2.id, alt: 'Second image', title: 'Image 2' }
        ],
        caption: 'Modern content management interfaces'
      }),
      
      // Third block: CTA
      buildBlockRecord({
        item_type: { type: 'item_type', id: ctaBlockType.id },
        title: 'Ready to revolutionize your content?',
        button_text: 'Get Started',
        button_url: 'https://www.datocms.com/signup'
      })
    ]
  });

  return article;
}
```

## Understanding the buildBlockRecord Helper

The `buildBlockRecord` helper is a crucial utility that transforms simple JavaScript objects into the JSON:API format required by DatoCMS for block records.

### What buildBlockRecord Does

The helper takes a simple object and converts it to the JSON:API structure that the API expects:

**Input (what you write):**
```javascript
const blockData = {
  item_type: { type: 'item_type', id: 'quote_block' },
  quote_text: 'Simplicity is the ultimate sophistication.',
  author: 'Leonardo da Vinci'
};
```

**Output (what gets sent to the API):**
```javascript
buildBlockRecord(blockData) // Returns:
{
  "type": "item",
  "attributes": {
    "quote_text": "Simplicity is the ultimate sophistication.",
    "author": "Leonardo da Vinci"
  },
  "relationships": {
    "item_type": {
      "data": {
        "type": "item_type",
        "id": "quote_block"
      }
    }
  }
}
```

### Manual Alternative (Without buildBlockRecord)

While `buildBlockRecord` is recommended, you can create the JSON:API structure manually:

```javascript
// Manual block creation (not recommended, but useful for understanding)
const manualBlock = {
  type: 'item',
  attributes: {
    quote_text: 'Simplicity is the ultimate sophistication.',
    author: 'Leonardo da Vinci'
  },
  relationships: {
    item_type: {
      data: {
        type: 'item_type',
        id: 'quote_block'
      }
    }
  }
};

// Use in Modular Content field
await client.items.create({
  item_type: { type: 'item_type', id: 'article' },
  content_sections: [manualBlock] // Array for Modular Content
});

// Use in Single Block field
await client.items.create({
  item_type: { type: 'item_type', id: 'page' },
  featured_block: manualBlock // Single object for Single Block
});
```

### When to Use buildBlockRecord

Always use `buildBlockRecord` when:
- Creating new blocks inline (not referencing existing blocks)
- Working with Modular Content (`rich_text`) fields
- Working with Single Block (`single_block`) fields
- Embedding blocks in Structured Text fields

### Implementation Details

The `buildBlockRecord` function:
1. Sets the type to `'item'` (required by JSON:API)
2. Places all fields except `item_type` into `attributes`
3. Transforms `item_type` into a proper relationship structure
4. Returns only the `data` portion (what the API expects)

## Advanced Patterns: Manipulating Modular Content

### Updating a Specific Block

To update a specific block within a Modular Content field without affecting other blocks:

```javascript
async function updateSpecificBlock(itemId, blockIdToUpdate, newData) {
  // Fetch the current item
  const item = await client.items.find(itemId);
  
  // Find and update the specific block
  const updatedBlocks = item.content_sections.map(block => {
    if (block.id === blockIdToUpdate) {
      // Merge the new data with the existing block
      return {
        ...block,
        ...newData
      };
    }
    return block;
  });

  // Update the item with the modified blocks
  const updatedItem = await client.items.update(itemId, {
    content_sections: updatedBlocks
  });

  return updatedItem;
}

// Example usage
await updateSpecificBlock('article-123', 'block-456', {
  quote_text: 'Updated quote text here',
  author: 'New Author'
});
```

### Adding a New Block

To add a new block to an existing Modular Content field:

```javascript
async function addBlockToItem(itemId, newBlock) {
  // Fetch the current item
  const item = await client.items.find(itemId);
  
  // Add the new block to the end of the array
  const updatedBlocks = [...item.content_sections, newBlock];

  // Update the item
  const updatedItem = await client.items.update(itemId, {
    content_sections: updatedBlocks
  });

  return updatedItem;
}

// Example usage
const ctaBlockType = await client.itemTypes.find('cta_block');
const newCta = buildBlockRecord({
  item_type: { type: 'item_type', id: ctaBlockType.id },
  title: 'Special Offer!',
  button_text: 'Learn More',
  button_url: 'https://example.com/offer'
});

await addBlockToItem('article-123', newCta);
```

### Reordering Blocks

To reorder blocks within a Modular Content field:

```javascript
async function reorderBlocks(itemId, newOrder) {
  // Fetch the current item
  const item = await client.items.find(itemId);
  
  // Create a map of blocks by ID for easy lookup
  const blockMap = {};
  item.content_sections.forEach(block => {
    blockMap[block.id] = block;
  });

  // Reorder blocks according to the new order array
  const reorderedBlocks = newOrder.map(blockId => blockMap[blockId]);

  // Update the item
  const updatedItem = await client.items.update(itemId, {
    content_sections: reorderedBlocks
  });

  return updatedItem;
}

// Example usage - reverse the order of blocks
const item = await client.items.find('article-123');
const currentOrder = item.content_sections.map(block => block.id);
const reversedOrder = currentOrder.reverse();

await reorderBlocks('article-123', reversedOrder);
```

### Deleting a Specific Block

To remove a specific block from a Modular Content field:

```javascript
async function deleteBlock(itemId, blockIdToDelete) {
  // Fetch the current item
  const item = await client.items.find(itemId);
  
  // Filter out the block to delete
  const updatedBlocks = item.content_sections.filter(
    block => block.id !== blockIdToDelete
  );

  // Update the item
  const updatedItem = await client.items.update(itemId, {
    content_sections: updatedBlocks
  });

  return updatedItem;
}

// Example usage
await deleteBlock('article-123', 'block-456');
```

## Querying Items by Block Type

To find all items that contain a specific type of block:

```javascript
async function findItemsWithBlockType(blockApiKey) {
  // First, get the block type
  const blockType = await client.itemTypes.find(blockApiKey);
  
  // Query items that have this block type in their modular content
  const items = await client.items.list({
    filter: {
      type: 'article', // The model to search
      fields: {
        content_sections: {
          any: {
            item_type: {
              eq: blockType.id
            }
          }
        }
      }
    }
  });

  return items;
}

// Example: Find all articles with quote blocks
const articlesWithQuotes = await findItemsWithBlockType('quote_block');
```

## Single Block Fields

The Single Block field (`field_type: 'single_block'`) allows you to add exactly one block to a model, chosen from a set of allowed block types. This is perfect for structured, one-off components like a featured section, hero banner, or call-to-action.

### Key Differences from Modular Content

| Feature | Modular Content (`rich_text`) | Single Block (`single_block`) |
|---------|-------------------------------|------------------------------|
| Number of blocks | Array (0 to many) | Exactly one (or null) |
| Reorderable | Yes | N/A (only one block) |
| Use case | Dynamic layouts | Featured components |
| API value | Array of blocks | Single block object |

### Creating a Single Block Field

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function createPageWithSingleBlock() {
  // Create block types (same as for Modular Content)
  const heroBlock = await client.itemTypes.create({
    name: 'Hero Block',
    api_key: 'hero_block',
    modular_block: true, // Must be true to use as a block
  });

  await client.fields.create(heroBlock.id, {
    label: 'Headline',
    api_key: 'headline',
    field_type: 'string',
    validators: { required: {} },
  });

  await client.fields.create(heroBlock.id, {
    label: 'Background Image',
    api_key: 'background_image',
    field_type: 'file',
    validators: { 
      required: {},
      file: { image_dimensions: {} }
    },
  });

  const ctaBlock = await client.itemTypes.create({
    name: 'CTA Block',
    api_key: 'cta_block',
    modular_block: true,
  });

  await client.fields.create(ctaBlock.id, {
    label: 'Button Text',
    api_key: 'button_text',
    field_type: 'string',
    validators: { required: {} },
  });

  await client.fields.create(ctaBlock.id, {
    label: 'Button URL',
    api_key: 'button_url',
    field_type: 'string',
    validators: { required: {}, format: { predefined_pattern: 'url' } },
  });

  // Create a Page model with a Single Block field
  const pageModel = await client.itemTypes.create({
    name: 'Landing Page',
    api_key: 'landing_page',
  });

  // Add a Single Block field
  await client.fields.create(pageModel.id, {
    label: 'Hero Section',
    api_key: 'hero_section',
    field_type: 'single_block',
    validators: {
      single_block_blocks: {
        item_types: [heroBlock.id, ctaBlock.id] // Allowed block types
      },
      required: {} // Make the field required
    },
    appearance: {
      editor: 'framed_single_block', // With collapsible frame
      parameters: {
        start_collapsed: false // Start expanded
      }
    }
  });

  return { pageModel, heroBlock, ctaBlock };
}
```

### Creating Items with Single Block Fields

```javascript
import { buildClient, buildBlockRecord } from '@datocms/cma-client';

async function createLandingPage() {
  const pageModel = await client.itemTypes.find('landing_page');
  const heroBlock = await client.itemTypes.find('hero_block');
  
  // Upload a background image
  const bgImage = await client.uploads.create({
    path: 'https://www.datocms-assets.com/123/hero-bg.jpg'
  });

  // Create a page with a hero block
  const page = await client.items.create({
    item_type: { type: 'item_type', id: pageModel.id },
    title: 'Summer Sale Landing Page',
    // Single Block field takes ONE block object, not an array
    hero_section: buildBlockRecord({
      item_type: { type: 'item_type', id: heroBlock.id },
      headline: 'Summer Sale - Up to 50% Off!',
      background_image: { upload_id: bgImage.id }
    })
  });

  return page;
}
```

### Updating a Single Block Field

```javascript
async function updateHeroSection(pageId, newHeadline) {
  // Fetch the current page
  const page = await client.items.find(pageId);
  
  // Update just the headline in the existing block
  const updatedBlock = {
    ...page.hero_section,
    headline: newHeadline
  };

  // Update the page
  const updatedPage = await client.items.update(pageId, {
    hero_section: updatedBlock
  });

  return updatedPage;
}

// Replace the entire block with a different type
async function switchToCTABlock(pageId) {
  const ctaBlock = await client.itemTypes.find('cta_block');
  
  const updatedPage = await client.items.update(pageId, {
    hero_section: buildBlockRecord({
      item_type: { type: 'item_type', id: ctaBlock.id },
      button_text: 'Shop Now',
      button_url: 'https://shop.example.com'
    })
  });

  return updatedPage;
}
```

### Single Block Field Validators

The `single_block_blocks` validator is required and controls which block types can be used:

```javascript
await client.fields.create(modelId, {
  label: 'Featured Component',
  api_key: 'featured_component',
  field_type: 'single_block',
  validators: {
    single_block_blocks: {
      item_types: ['hero_block_id', 'cta_block_id', 'video_block_id']
    },
    required: {} // Optional: make the field required
  }
});
```

### Editor Options for Single Block Fields

Two editor options are available:

1. **Framed Single Block Editor** (default):
```javascript
appearance: {
  editor: 'framed_single_block',
  parameters: {
    start_collapsed: true // Start with block collapsed
  }
}
```

2. **Frameless Single Block Editor**:
```javascript
appearance: {
  editor: 'frameless_single_block'
  // No parameters available
}
```

### Common Use Cases for Single Block Fields

1. **Hero Sections**: One prominent block at the top of a page
2. **Featured Content**: A single highlighted component
3. **Call-to-Action**: One focused conversion element
4. **Sidebar Widget**: A single configurable sidebar component
5. **Page Headers**: One customizable header per page

### Best Practices

1. **Field Naming**: Use singular names (e.g., `hero_section`, not `hero_sections`)
2. **Validation**: Consider if the field should be required
3. **Block Selection**: Limit allowed blocks to those that make sense as standalone components
4. **Editor Choice**: Use framed editor for complex blocks, frameless for simple ones

## Best Practices

### 1. Block Design
- Keep blocks focused on a single purpose
- Make blocks as reusable as possible
- Use clear, descriptive names for block types
- Consider the editor experience when designing block fields

### 2. Performance
- Limit the number of blocks in a single Modular Content field
- Use pagination when querying items with many blocks
- Consider lazy loading blocks in your frontend

### 3. Validation
- Set appropriate validators on block fields
- Use the `rich_text_blocks` validator to control which blocks are allowed
- Consider setting minimum/maximum number of blocks

### 4. Error Handling

Always handle errors when working with Modular Content:

```javascript
try {
  const article = await client.items.create({
    item_type: { type: 'item_type', id: articleModel.id },
    title: 'My Article',
    content_sections: blocks
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation error:', error.response.data);
    // Handle validation errors
  } else {
    console.error('Unexpected error:', error);
    throw error;
  }
}
```

## Related Resources

- [Structured Text Guide](./structured-text.md) - Learn about embedding blocks within rich text
- [Field Methods](./field-methods.md) - Complete reference for field operations
- [Item Methods](./item-methods.md) - Complete reference for item operations
- [Choosing Content Strategy](../../04-advanced-topics/choosing-content-strategy.md) - When to use Modular Content vs Structured Text