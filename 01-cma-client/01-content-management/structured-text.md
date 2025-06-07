# Structured Text

Structured Text is DatoCMS's powerful rich text field type that supports formatted content, links to other records, and embedded blocks. This guide provides comprehensive documentation for working with Structured Text fields using the DatoCMS CMA Client.

## Overview

A Structured Text field allows editors to create rich text content with:
- Standard formatting (bold, italic, headings, lists, etc.)
- Links to other DatoCMS records
- Embedded blocks for dynamic content
- Custom styling and marks

The content is stored in DAST (DatoCMS Abstract Syntax Tree) format, a JSON structure that represents the document.

## Installation

```bash
npm install @datocms/cma-client
npm install datocms-structured-text-utils # For programmatic DAST creation
```

## DAST (DatoCMS Abstract Syntax Tree)

DAST is a JSON format that represents structured text content. It's based on the Slate.js data model but tailored for DatoCMS's needs.

### Basic DAST Structure

A DAST document has this structure:

```javascript
{
  schema: 'dast',
  document: {
    type: 'root',
    children: [
      // Document nodes go here
    ]
  }
}
```

### Common Node Types

Here are the most common DAST node types:

```javascript
// Paragraph node
{
  type: 'paragraph',
  children: [
    { type: 'span', value: 'This is a paragraph.' }
  ]
}

// Heading node
{
  type: 'heading',
  level: 2,
  children: [
    { type: 'span', value: 'This is a heading' }
  ]
}

// List node
{
  type: 'list',
  style: 'bulleted', // or 'numbered'
  children: [
    {
      type: 'listItem',
      children: [
        {
          type: 'paragraph',
          children: [{ type: 'span', value: 'First item' }]
        }
      ]
    }
  ]
}

// Link to internal record
{
  type: 'inlineItem',
  item: '12345' // ID of the linked record
}

// Embedded block
{
  type: 'block',
  item: {
    type: 'block',
    id: 'block-123',
    attributes: {
      // Block fields
    },
    relationships: {
      item_type: {
        data: {
          type: 'item_type',
          id: 'quote_block'
        }
      }
    }
  }
}
```

### Text Marks

Text can have marks for formatting:

```javascript
{
  type: 'span',
  marks: ['strong', 'emphasis', 'underline', 'strikethrough', 'highlight', 'code'],
  value: 'Formatted text'
}
```

## Creating a Structured Text Field

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function createBlogPostModel() {
  // First, create some block types that can be embedded
  const quoteBlock = await client.itemTypes.create({
    name: 'Quote Block',
    api_key: 'quote_block',
    modular_block: true,
  });

  await client.fields.create(quoteBlock.id, {
    label: 'Quote',
    api_key: 'quote',
    field_type: 'text',
  });

  await client.fields.create(quoteBlock.id, {
    label: 'Author',
    api_key: 'author',
    field_type: 'string',
  });

  // Create a model that can be linked
  const authorModel = await client.itemTypes.create({
    name: 'Author',
    api_key: 'author',
  });

  await client.fields.create(authorModel.id, {
    label: 'Name',
    api_key: 'name',
    field_type: 'string',
  });

  // Create the blog post model
  const blogPost = await client.itemTypes.create({
    name: 'Blog Post',
    api_key: 'blog_post',
  });

  // Add a structured text field
  await client.fields.create(blogPost.id, {
    label: 'Content',
    api_key: 'content',
    field_type: 'structured_text',
    validators: {
      structured_text_blocks: {
        item_types: [quoteBlock.id] // Allowed block types
      },
      structured_text_links: {
        item_types: [authorModel.id] // Allowed link types
      }
    }
  });

  return { blogPost, quoteBlock, authorModel };
}
```

## Programmatic DAST Creation

Creating DAST manually is error-prone. Use the `datocms-structured-text-utils` library:

```javascript
import { buildClient, buildBlockRecord } from '@datocms/cma-client';
import { 
  render,
  renderRule,
  paragraph as p,
  heading as h,
  list,
  listItem,
  span as s,
  link,
  block,
  inlineItem,
  root,
  document,
  blockquote as bq,
  code
} from 'datocms-structured-text-utils';

async function createBlogPostWithStructuredText() {
  const blogPostModel = await client.itemTypes.find('blog_post');
  const quoteBlockType = await client.itemTypes.find('quote_block');
  const authorModel = await client.itemTypes.find('author');

  // Create an author to link to
  const author = await client.items.create({
    item_type: { type: 'item_type', id: authorModel.id },
    name: 'Jane Doe'
  });

  // Create the structured text content programmatically
  const content = root(
    h(1, 'The Future of Content Management'),
    
    p('Welcome to our comprehensive guide on modern content management systems.'),
    
    h(2, 'Why Structured Content Matters'),
    
    p(
      'Structured content allows you to ',
      s('create once', ['strong']),
      ' and ',
      s('publish everywhere', ['emphasis']),
      '. It separates content from presentation.'
    ),
    
    list('bulleted', 
      listItem(p('Reusable across channels')),
      listItem(p('Consistent formatting')),
      listItem(p('Better SEO'))
    ),
    
    p(
      'As ',
      inlineItem(author.id),
      ' explains in their recent article:'
    ),
    
    // Embed a quote block
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: quoteBlockType.id },
        quote: 'Content is the atomic unit of modern digital experiences.',
        author: 'Jane Doe'
      })
    ),
    
    h(2, 'Technical Implementation'),
    
    p('Here\'s how you can implement structured content:'),
    
    code('javascript', `
const content = {
  schema: 'dast',
  document: {
    type: 'root',
    children: [/* your nodes */]
  }
};
    `),
    
    p(
      'For more details, check out the ',
      link('https://www.datocms.com/docs', 'official documentation'),
      '.'
    )
  );

  // Create the blog post with structured text
  const blogPost = await client.items.create({
    item_type: { type: 'item_type', id: blogPostModel.id },
    content: {
      schema: 'dast',
      document: content
    }
  });

  return blogPost;
}
```

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

## Embedding Blocks in Structured Text

Blocks can be embedded within the flow of structured text content. This is different from Modular Content where blocks are separate, reorderable sections.

```javascript
async function createArticleWithInlineBlocks() {
  // Get required models
  const articleModel = await client.itemTypes.find('blog_post');
  const quoteBlockType = await client.itemTypes.find('quote_block');
  const ctaBlockType = await client.itemTypes.find('cta_block');

  // Create structured text with embedded blocks
  const content = root(
    h(1, 'Understanding Climate Change'),
    
    p('Climate change is one of the most pressing issues of our time. Let me explain why.'),
    
    p('Scientists have been studying this phenomenon for decades. The evidence is overwhelming.'),
    
    // Embed a quote block in the middle of the article
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: quoteBlockType.id },
        quote: 'The Earth\'s climate is changing, and human activities are the dominant cause.',
        author: 'NASA Climate Science'
      })
    ),
    
    p('The impacts are already visible around the world:'),
    
    list('bulleted',
      listItem(p('Rising global temperatures')),
      listItem(p('Melting ice caps')),
      listItem(p('More frequent extreme weather events'))
    ),
    
    p('But there\'s still hope if we act now. Every action counts.'),
    
    // Embed a CTA block
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: ctaBlockType.id },
        title: 'Take Action Today',
        button_text: 'Learn How You Can Help',
        button_url: 'https://climate.nasa.gov/solutions'
      })
    ),
    
    p('Together, we can make a difference for future generations.')
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

## Advanced Patterns

### Traversing and Manipulating DAST

To find and modify nodes within a DAST document:

```javascript
function traverseDast(node, callback, parent = null) {
  // Call the callback for the current node
  callback(node, parent);
  
  // Recursively traverse children
  if (node.children && Array.isArray(node.children)) {
    node.children.forEach(child => traverseDast(child, callback, node));
  }
}

// Example: Find all links in a document
function findAllLinks(dastDocument) {
  const links = [];
  
  traverseDast(dastDocument, (node) => {
    if (node.type === 'link') {
      links.push({
        url: node.url,
        children: node.children
      });
    }
  });
  
  return links;
}

// Example: Update all internal links
async function updateInternalLinks(itemId, oldRecordId, newRecordId) {
  const item = await client.items.find(itemId);
  const content = item.content;
  
  traverseDast(content.document, (node) => {
    if (node.type === 'inlineItem' && node.item === oldRecordId) {
      node.item = newRecordId;
    }
  });
  
  const updatedItem = await client.items.update(itemId, {
    content: content
  });
  
  return updatedItem;
}
```

### Finding Items with Specific Embedded Blocks

```javascript
async function findArticlesWithBlockType(blockApiKey) {
  const blockType = await client.itemTypes.find(blockApiKey);
  
  // This requires iterating through items and checking their content
  const allArticles = await client.items.list({
    filter: {
      type: 'blog_post'
    },
    page: {
      limit: 100
    }
  });
  
  const articlesWithBlock = allArticles.filter(article => {
    if (!article.content?.document) return false;
    
    let hasBlock = false;
    traverseDast(article.content.document, (node) => {
      if (node.type === 'block' && 
          node.item?.relationships?.item_type?.data?.id === blockType.id) {
        hasBlock = true;
      }
    });
    
    return hasBlock;
  });
  
  return articlesWithBlock;
}
```

### Creating Custom Marks and Styles

```javascript
// Create structured text with custom marks
const contentWithCustomMarks = root(
  p(
    'This text has ',
    s('multiple', ['strong', 'emphasis']),
    ' marks applied. You can also use ',
    s('code formatting', ['code']),
    ' inline.'
  ),
  
  p(
    s('Highlighted text', ['highlight']),
    ' stands out, while ',
    s('strikethrough text', ['strikethrough']),
    ' indicates removal.'
  )
);
```

### Building Complex Documents

```javascript
async function createComplexDocument() {
  const articleModel = await client.itemTypes.find('blog_post');
  const imageBlockType = await client.itemTypes.find('image_block');
  const tableBlockType = await client.itemTypes.find('table_block');
  
  // Upload an image
  const upload = await client.uploads.create({
    path: 'https://www.datocms-assets.com/123/image.jpg'
  });
  
  const complexContent = root(
    h(1, 'Complete Guide to Structured Text'),
    
    // Table of contents
    h(2, 'Table of Contents'),
    list('numbered',
      listItem(p(link('#introduction', 'Introduction'))),
      listItem(p(link('#basics', 'The Basics'))),
      listItem(p(link('#advanced', 'Advanced Usage')))
    ),
    
    // Introduction section
    h(2, 'Introduction', { id: 'introduction' }),
    p('This guide covers everything you need to know about structured text.'),
    
    // Image block
    block(
      buildBlockRecord({
        item_type: { type: 'item_type', id: imageBlockType.id },
        image: { upload_id: upload.id },
        caption: 'Structured text in action',
        alt_text: 'Screenshot of structured text editor'
      })
    ),
    
    // Basics section
    h(2, 'The Basics', { id: 'basics' }),
    p('Let\'s start with the fundamental concepts:'),
    
    // Nested lists
    list('bulleted',
      listItem(
        p('DAST Format'),
        list('bulleted',
          listItem(p('JSON-based structure')),
          listItem(p('Tree of nodes')),
          listItem(p('Extensible'))
        )
      ),
      listItem(
        p('Node Types'),
        list('bulleted',
          listItem(p('Block nodes (paragraphs, headings)')),
          listItem(p('Inline nodes (spans, links)')),
          listItem(p('Embedded content'))
        )
      )
    ),
    
    // Code example
    h(3, 'Code Example'),
    code('json', JSON.stringify({
      type: 'paragraph',
      children: [
        { type: 'span', value: 'Hello world' }
      ]
    }, null, 2)),
    
    // Advanced section
    h(2, 'Advanced Usage', { id: 'advanced' }),
    
    // Blockquote
    bq(
      p('With great power comes great responsibility.'),
      p('— Spider-Man')
    ),
    
    p('Advanced features include custom renderers, validation, and more.')
  );
  
  const article = await client.items.create({
    item_type: { type: 'item_type', id: articleModel.id },
    content: {
      schema: 'dast',
      document: complexContent
    }
  });
  
  return article;
}
```

## Validation and Error Handling

### Field Validation

```javascript
// Configure structured text field with validation
await client.fields.create(modelId, {
  label: 'Content',
  api_key: 'content',
  field_type: 'structured_text',
  validators: {
    required: {},
    structured_text_blocks: {
      item_types: [quoteBlockId, imageBlockId] // Allowed block types
    },
    structured_text_links: {
      item_types: [authorId, articleId], // Allowed link types
      on_publish_with_unpublished_references_strategy: 'fail'
    },
    size: {
      max: 50000 // Maximum characters
    }
  }
});
```

### Error Handling

```javascript
try {
  const item = await client.items.create({
    item_type: { type: 'item_type', id: modelId },
    content: dastContent
  });
} catch (error) {
  if (error.response?.status === 422) {
    const validationErrors = error.response.data;
    console.error('Validation errors:', validationErrors);
    
    // Handle specific validation errors
    if (validationErrors.data?.attributes?.content) {
      console.error('Content field errors:', 
        validationErrors.data.attributes.content);
    }
  } else {
    throw error;
  }
}
```

## Best Practices

### 1. Content Structure
- Keep DAST documents well-structured and semantic
- Use appropriate heading levels (h1 → h2 → h3)
- Avoid deeply nested structures
- Use blocks for substantial content, inline items for references

### 2. Performance
- For large documents, consider splitting into multiple fields
- Limit the depth of nested structures
- Use pagination when querying items with large structured text fields

### 3. Validation
- Always validate allowed block types and link types
- Set appropriate size limits
- Handle unpublished references appropriately

### 4. Maintainability
- Use the utility library for programmatic creation
- Create helper functions for common patterns
- Document your DAST structures
- Version control your content models

## Common Use Cases

### Blog Posts with Rich Media
```javascript
// Ideal for articles with inline images, quotes, and CTAs
const blogPost = root(
  h(1, 'Article Title'),
  p('Introduction paragraph...'),
  block(imageBlock),
  p('More content...'),
  block(quoteBlock),
  p('Conclusion...')
);
```

### Documentation with Code Examples
```javascript
// Perfect for technical documentation
const docs = root(
  h(1, 'API Reference'),
  p('This endpoint returns...'),
  code('bash', 'curl -X GET https://api.example.com/users'),
  p('The response format is:'),
  code('json', '{ "users": [...] }')
);
```

### Landing Pages with Inline CTAs
```javascript
// For marketing content with strategic CTAs
const landingPage = root(
  h(1, 'Welcome to Our Product'),
  p('Revolutionary solution for...'),
  block(heroImageBlock),
  p('Key benefits include...'),
  list('bulleted', /* benefits */),
  block(ctaBlock),
  p('Join thousands of satisfied customers...')
);
```

## Related Resources

- [Modular Content Guide](./modular-content.md) - Learn about block-based layouts
- [Field Methods](./field-methods.md) - Complete reference for field operations
- [Item Methods](./item-methods.md) - Complete reference for item operations
- [Choosing Content Strategy](../../04-advanced-topics/choosing-content-strategy.md) - When to use Structured Text vs Modular Content
- [DAST Specification](https://www.datocms.com/docs/structured-text/dast) - Official DAST format reference