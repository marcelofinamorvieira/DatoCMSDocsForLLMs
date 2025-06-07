# Implementation Plan: Filling Documentation Gaps for Blocks and Structured Text

## Overview

This document provides specific instructions for an LLM to update the documentation to address all identified gaps related to Structured Text and Modular Content (Blocks). Each instruction is self-contained and designed for optimal RAG system usage.

## Instruction 1: Update and Rename Modular Content Documentation

### File to Modify
- **Current**: `/LLM_DOCS/01-cma-client/01-content-management/modular-content.md`
- **New Name**: `/LLM_DOCS/01-cma-client/01-content-management/blocks-and-modular-content.md`

### Specific Changes

#### 1. Add Field Type Mapping Section (After Line 13)

```markdown
## Field Type Terminology

**IMPORTANT**: DatoCMS uses different terms in the UI versus the API. This mapping is crucial for correctly using the API:

| UI Feature Name | API `field_type` | Description | Use Case |
|-----------------|------------------|-------------|----------|
| **Modular Content** | `rich_text` | An array of reorderable content blocks | Page builders, flexible layouts |
| **Single Block** | `single_block` | Exactly one block from allowed types | Featured components, one-off sections |
| **Structured Text** | `structured_text` | Rich text with inline blocks | Articles with embedded media |

**Note**: Despite the name, "Modular Content" in the UI corresponds to `field_type: 'rich_text'` in the API. This guide covers both Modular Content (`rich_text`) and Single Block (`single_block`) fields.
```

#### 2. Add buildBlockRecord Explanation Section (After Line 157)

```markdown
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
```

#### 3. Add Complete Single Block Section (After Modular Content sections)

```markdown
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
```

## Instruction 2: Enhance Structured Text Documentation

### File to Modify
`/LLM_DOCS/01-cma-client/01-content-management/structured-text.md`

### Specific Changes

#### 1. Add Inline Block Creation Section (After Line 350)

```markdown
## Creating and Embedding Blocks Inline

One of the most powerful features of Structured Text is the ability to create new block records directly within the content, rather than referencing existing blocks. This is achieved by using the `buildBlockRecord` helper inside the `block()` function.

### Basic Inline Block Creation

```javascript
import { buildClient, buildBlockRecord } from '@datocms/cma-client';
import { 
  root, 
  paragraph as p, 
  heading as h, 
  block 
} from 'datocms-structured-text-utils';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function createArticleWithInlineBlocks() {
  const articleModel = await client.itemTypes.find('article');
  const imageBlockType = await client.itemTypes.find('image_block');
  const quoteBlockType = await client.itemTypes.find('quote_block');
  
  // Upload an image first
  const imageUpload = await client.uploads.create({
    path: 'https://www.datocms-assets.com/123/chart.png',
    default_field_metadata: {
      en: {
        alt: 'Sales growth chart',
        title: 'Q4 2023 Sales Performance'
      }
    }
  });

  const content = root(
    h(1, 'Company Achieves Record Growth'),
    
    p('Our latest quarterly results show exceptional performance across all metrics.'),
    
    // Create an image block inline
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: imageBlockType.id },
        image: { upload_id: imageUpload.id },
        caption: 'Q4 2023 sales increased by 47% year-over-year',
        full_width: true
      })
    ),
    
    p('CEO Jane Smith commented on the results:'),
    
    // Create a quote block inline
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: quoteBlockType.id },
        quote_text: 'This growth reflects our team\'s dedication and our customers\' trust in our products.',
        author_name: 'Jane Smith',
        author_title: 'CEO'
      })
    ),
    
    p('The company plans to continue this momentum into 2024.')
  );

  const article = await client.items.create({
    item_type: { type: 'item_type', id: articleModel.id },
    title: 'Q4 2023 Earnings Report',
    content: {
      schema: 'dast',
      document: content
    }
  });

  return article;
}
```

### Inline Block Creation vs Block References

You have two options when adding blocks to Structured Text:

1. **Create Inline** (shown above): The block is created as part of the Structured Text
2. **Reference Existing**: Link to a block that already exists

```javascript
// Method 1: Inline creation (creates a new block)
block(
  buildBlockRecord({
    item_type: { type: 'item_type', id: quoteBlockType.id },
    quote_text: 'New quote created inline',
    author: 'John Doe'
  })
)

// Method 2: Reference existing block (by ID)
block('existing-quote-block-id')
```

### Complex Example: Article with Multiple Inline Blocks

```javascript
async function createComplexArticle() {
  // Get all required block types
  const videoBlockType = await client.itemTypes.find('video_block');
  const galleryBlockType = await client.itemTypes.find('gallery_block');
  const ctaBlockType = await client.itemTypes.find('cta_block');
  const codeBlockType = await client.itemTypes.find('code_block');
  
  // Upload multiple images for gallery
  const galleryImages = await Promise.all([
    client.uploads.create({ path: 'https://example.com/img1.jpg' }),
    client.uploads.create({ path: 'https://example.com/img2.jpg' }),
    client.uploads.create({ path: 'https://example.com/img3.jpg' })
  ]);

  const content = root(
    h(1, 'Building Modern Web Applications'),
    
    p('In this tutorial, we\'ll explore cutting-edge web development techniques.'),
    
    // Inline video block
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: videoBlockType.id },
        video_url: 'https://www.youtube.com/watch?v=dQw4w9WgXcQ',
        title: 'Introduction to Modern Web Development',
        autoplay: false
      })
    ),
    
    h(2, 'Setting Up Your Environment'),
    
    p('First, let\'s install the necessary dependencies:'),
    
    // Inline code block
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: codeBlockType.id },
        language: 'bash',
        code: 'npm install react next.js tailwindcss\nnpm install -D @types/react typescript'
      })
    ),
    
    h(2, 'Project Screenshots'),
    
    // Inline gallery block
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: galleryBlockType.id },
        images: galleryImages.map((img, index) => ({
          upload_id: img.id,
          alt: `Screenshot ${index + 1}`,
          title: `Project view ${index + 1}`
        })),
        columns: 3,
        enable_lightbox: true
      })
    ),
    
    p('Ready to start building? Let\'s dive in!'),
    
    // Inline CTA block
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: ctaBlockType.id },
        headline: 'Get the Complete Course',
        description: 'Learn everything about modern web development',
        button_text: 'Start Learning',
        button_url: 'https://courses.example.com',
        style: 'primary'
      })
    )
  );

  const article = await client.items.create({
    item_type: { type: 'item_type', id: 'tutorial' },
    content: {
      schema: 'dast',
      document: content
    }
  });

  return article;
}
```

### Understanding the Resulting DAST Structure

When you create an inline block, the resulting DAST structure contains the full block data:

```javascript
{
  "type": "block",
  "item": {
    "type": "item",
    "attributes": {
      "quote_text": "This growth reflects our team's dedication...",
      "author_name": "Jane Smith",
      "author_title": "CEO"
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
}
```

This is different from a block reference, which only contains an ID:

```javascript
{
  "type": "block",
  "item": "existing-block-id-12345"
}
```
```

#### 2. Add itemLink vs inlineItem Comparison Section (After Line 244)

```markdown
## Understanding itemLink vs inlineItem

Structured Text provides two distinct ways to reference other DatoCMS records within your content. Understanding the differences is crucial for proper implementation.

### itemLink: Creating Hyperlinks to Records

The `itemLink` node type creates a **clickable hyperlink** to another record, similar to a regular link but pointing to internal content instead of external URLs.

**Structure and Properties:**
- **Has children**: Must contain the link text
- **Supports metadata**: Can include `rel`, `target`, etc.
- **Renders as**: An anchor tag (`<a>`) with custom text

```javascript
import { paragraph as p, itemLink, span as s } from 'datocms-structured-text-utils';

// Create a paragraph with an itemLink
const contentWithItemLink = p(
  'Read more about this topic in ',
  itemLink(
    '38945648', // The ID of the linked record
    [s('our comprehensive guide')], // The link text (children)
    [ // Optional metadata
      { id: 'rel', value: 'related' },
      { id: 'target', value: '_blank' }
    ]
  ),
  '.'
);
```

**Use Cases for itemLink:**
- Author bylines that link to author pages
- References to related articles
- Links to product pages within text
- Any scenario where you want specific link text

### inlineItem: Embedding Record References

The `inlineItem` node type embeds a **reference to a record** without specifying how it should be displayed. The frontend application decides the rendering.

**Structure and Properties:**
- **No children**: Cannot contain text
- **No metadata**: Just the record reference
- **Renders as**: Whatever the frontend decides (widget, card, link, etc.)

```javascript
import { paragraph as p, inlineItem } from 'datocms-structured-text-utils';

// Create a paragraph with an inlineItem
const contentWithInlineItem = p(
  'This article was written by ',
  inlineItem('author-123'), // Just the record ID
  ' and reviewed by ',
  inlineItem('author-456'),
  '.'
);
```

**Use Cases for inlineItem:**
- Author cards or widgets
- Product embeds
- Dynamic components that fetch their own data
- Any scenario where the frontend controls rendering

### Key Differences Comparison

| Feature | `itemLink` | `inlineItem` |
|---------|------------|--------------|
| **Purpose** | Create a hyperlink with custom text | Embed a record reference |
| **Children** | Required (the link text) | Not allowed |
| **Metadata** | Supported (`rel`, `target`, etc.) | Not supported |
| **Display Control** | Content author controls text | Frontend developer controls everything |
| **Typical Rendering** | `<a href="...">custom text</a>` | Custom component/widget |
| **Click Behavior** | Navigate to linked record | Defined by frontend |

### Complete Example Showing Both

```javascript
import { 
  root, 
  heading as h,
  paragraph as p,
  itemLink, 
  inlineItem,
  span as s 
} from 'datocms-structured-text-utils';

async function createArticleWithReferences() {
  const articleModel = await client.itemTypes.find('blog_post');
  const authorId = '123456';
  const relatedArticleId = '789012';
  const productId = '345678';
  
  const content = root(
    h(1, 'Understanding React Hooks'),
    
    // Using itemLink for a traditional hyperlink
    p(
      'This article was written by ',
      itemLink(
        authorId,
        [s('Jane Smith')], // Specific link text
        [{ id: 'rel', value: 'author' }]
      ),
      ', our senior React developer.'
    ),
    
    // Using inlineItem for an author widget
    p(
      'About the author: ',
      inlineItem(authorId) // Frontend will render author card/bio
    ),
    
    p('React Hooks revolutionized how we write React components...'),
    
    // Using inlineItem for a product embed
    p(
      'You can see this pattern in action with ',
      inlineItem(productId), // Frontend will render product widget
      ' which implements hooks throughout.'
    ),
    
    // Using itemLink for related article
    p(
      'For more advanced patterns, check out ',
      itemLink(
        relatedArticleId,
        [s('Advanced Hook Patterns')],
        [{ id: 'target', value: '_blank' }]
      ),
      '.'
    )
  );
  
  const article = await client.items.create({
    item_type: { type: 'item_type', id: articleModel.id },
    content: {
      schema: 'dast',
      document: content
    }
  });
  
  return article;
}
```

### Frontend Rendering Examples

**Rendering itemLink (React example):**
```jsx
function renderItemLink(node, linkedRecords) {
  const linkedRecord = linkedRecords[node.item];
  return (
    <Link 
      to={`/${linkedRecord.slug}`}
      {...metaToProps(node.meta)}
    >
      {node.children.map(child => renderNode(child))}
    </Link>
  );
}
```

**Rendering inlineItem (React example):**
```jsx
function renderInlineItem(node, linkedRecords) {
  const record = linkedRecords[node.item];
  
  switch (record.__typename) {
    case 'AuthorRecord':
      return <AuthorCard author={record} />;
    
    case 'ProductRecord':
      return <ProductWidget product={record} />;
    
    case 'ArticleRecord':
      return <ArticlePreview article={record} />;
    
    default:
      // Fallback: render as simple link
      return <Link to={`/${record.slug}`}>{record.title}</Link>;
  }
}
```

### Best Practices

1. **Use itemLink when:**
   - You need specific link text
   - Creating traditional hyperlinks
   - The link text is important for SEO
   - You want consistent link styling

2. **Use inlineItem when:**
   - Embedding rich components
   - The display varies by context
   - You need dynamic data fetching
   - Creating widgets or cards

3. **Common Mistakes to Avoid:**
   - Don't use `inlineItem` when you just need a simple text link
   - Don't try to add children to `inlineItem` (it will fail)
   - Don't use `itemLink` for complex embeds (use `inlineItem` instead)
   - Remember that `itemLink` requires children nodes
```

## Instruction 3: Update Reference Documentation

### File to Modify
`/LLM_DOCS/05-reference/field-type-matrix.md`

### Specific Changes

Find the field types table and add the `single_block` entry:

```markdown
| single_block | Single Block | One block from allowed types | `single_block_blocks`, `required` | `framed_single_block`, `frameless_single_block` |
```

## Success Metrics

After implementing these changes:

1. ✅ An LLM will understand the mapping between UI terms and API field types
2. ✅ An LLM can generate code for all block-based field types including `single_block`
3. ✅ An LLM understands what `buildBlockRecord` does and can use it correctly
4. ✅ An LLM can create inline blocks within Structured Text
5. ✅ An LLM can choose appropriately between `itemLink` and `inlineItem`
6. ✅ All examples are complete and self-contained for RAG retrieval

## Implementation Notes

- All code examples include necessary imports
- Each section is self-contained for optimal RAG performance
- Examples show both simple and complex use cases
- Error handling is included where appropriate
- Type definitions are explicit throughout