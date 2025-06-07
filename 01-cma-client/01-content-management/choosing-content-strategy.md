# Choosing a Content Strategy: Modular Content vs. Structured Text

This guide helps you understand when to use Modular Content fields versus Structured Text fields with embedded blocks. This is one of the most important architectural decisions when designing your content model in DatoCMS.

## Overview

Both Modular Content and Structured Text can contain blocks, but they serve fundamentally different purposes:

- **Modular Content**: A list of reorderable content blocks for building layouts
- **Structured Text**: Rich text content that can embed blocks within the flow of text

## Comparison Table

| Feature | Modular Content | Structured Text with Blocks |
|---------|-----------------|---------------------------|
| **Best For** | Page layouts, content sections, component libraries | Articles, blog posts, documentation with inline media |
| **Structure** | Array of block records | DAST tree with text nodes and embedded blocks |
| **Content Flow** | Discrete, reorderable sections | Continuous narrative with embedded elements |
| **Editor Experience** | Drag-and-drop block management | Rich text editor with block insertion |
| **API Payload** | Array of objects | DAST JSON structure |
| **Block Placement** | Top-level array items | Nested within document tree |
| **Text Content** | Inside individual blocks | Native to the field itself |
| **Reordering** | Simple array manipulation | Complex tree traversal |
| **Use Cases** | Landing pages, homepages, product pages | Blog posts, news articles, documentation |

## API Payload Comparison

Understanding the different data structures is crucial for making the right choice:

### Modular Content Payload

```javascript
{
  "id": "12345",
  "type": "item",
  "attributes": {
    "title": "Our Services",
    "content_sections": [
      {
        "id": "block-1",
        "type": "block",
        "attributes": {
          "title": "Web Development",
          "description": "We build modern web applications",
          "icon": "code"
        },
        "relationships": {
          "item_type": {
            "data": { "type": "item_type", "id": "service_block" }
          }
        }
      },
      {
        "id": "block-2",
        "type": "block",
        "attributes": {
          "quote": "Best service ever!",
          "author": "Jane Client"
        },
        "relationships": {
          "item_type": {
            "data": { "type": "item_type", "id": "testimonial_block" }
          }
        }
      },
      {
        "id": "block-3",
        "type": "block",
        "attributes": {
          "heading": "Get Started Today",
          "button_text": "Contact Us",
          "button_url": "/contact"
        },
        "relationships": {
          "item_type": {
            "data": { "type": "item_type", "id": "cta_block" }
          }
        }
      }
    ]
  }
}
```

### Structured Text Payload

```javascript
{
  "id": "67890",
  "type": "item",
  "attributes": {
    "title": "Understanding Climate Change",
    "content": {
      "schema": "dast",
      "document": {
        "type": "root",
        "children": [
          {
            "type": "heading",
            "level": 1,
            "children": [
              { "type": "span", "value": "The Science Behind Climate Change" }
            ]
          },
          {
            "type": "paragraph",
            "children": [
              { "type": "span", "value": "Climate change is driven by increasing " },
              { "type": "span", "marks": ["strong"], "value": "greenhouse gases" },
              { "type": "span", "value": " in our atmosphere." }
            ]
          },
          {
            "type": "block",
            "item": {
              "id": "block-123",
              "type": "block",
              "attributes": {
                "chart_type": "temperature_rise",
                "data_source": "NASA",
                "caption": "Global temperature anomaly 1880-2023"
              },
              "relationships": {
                "item_type": {
                  "data": { "type": "item_type", "id": "data_viz_block" }
                }
              }
            }
          },
          {
            "type": "paragraph",
            "children": [
              { "type": "span", "value": "As we can see from the data above, the trend is clear." }
            ]
          },
          {
            "type": "list",
            "style": "bulleted",
            "children": [
              {
                "type": "listItem",
                "children": [
                  {
                    "type": "paragraph",
                    "children": [
                      { "type": "span", "value": "Average temperatures have risen by 1.1°C" }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    }
  }
}
```

## Use Case Scenarios

### When to Use Modular Content

**Landing Page Builder**

You're building a marketing landing page where editors need to:
- Add, remove, and reorder distinct sections
- Choose from a library of pre-designed components
- Build different page layouts for A/B testing

```javascript
import { buildClient, buildBlockRecord } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function createLandingPage() {
  const pageModel = await client.itemTypes.find('landing_page');
  const heroBlockType = await client.itemTypes.find('hero_block');
  const featuresBlockType = await client.itemTypes.find('features_block');
  const testimonialBlockType = await client.itemTypes.find('testimonial_block');
  const ctaBlockType = await client.itemTypes.find('cta_block');

  const landingPage = await client.items.create({
    item_type: { type: 'item_type', id: pageModel.id },
    title: 'Product Launch Page',
    slug: 'new-product-2024',
    sections: [
      // Hero section
      buildBlockRecord({
        item_type: { type: 'item_type', id: heroBlockType.id },
        headline: 'Introducing Product X',
        subheadline: 'The future of productivity',
        background_image: { upload_id: 'hero-bg-123' },
        cta_text: 'Get Early Access',
        cta_link: '#signup'
      }),
      
      // Features grid
      buildBlockRecord({
        item_type: { type: 'item_type', id: featuresBlockType.id },
        title: 'Why Choose Product X',
        features: [
          { icon: 'rocket', title: 'Lightning Fast', description: '10x faster than competitors' },
          { icon: 'shield', title: 'Secure', description: 'Bank-level encryption' },
          { icon: 'users', title: 'Collaborative', description: 'Real-time team features' }
        ]
      }),
      
      // Testimonials carousel
      buildBlockRecord({
        item_type: { type: 'item_type', id: testimonialBlockType.id },
        title: 'What Our Customers Say',
        testimonials: [
          { quote: 'Game changer!', author: 'Sarah J.', company: 'TechCorp' },
          { quote: 'Can\'t imagine working without it', author: 'Mike D.', company: 'StartupXYZ' }
        ]
      }),
      
      // Final CTA
      buildBlockRecord({
        item_type: { type: 'item_type', id: ctaBlockType.id },
        heading: 'Ready to Transform Your Workflow?',
        button_text: 'Start Free Trial',
        button_url: '/signup',
        style: 'primary'
      })
    ]
  });

  return landingPage;
}
```

**Why Modular Content is correct here:**
- Each section is independent and reorderable
- Editors can easily add/remove sections
- The page is composed of distinct components, not flowing text
- Frontend can render each block type with specific components

### When to Use Structured Text

**Blog Post with Inline CTAs**

You're creating a blog post that needs:
- Rich text formatting throughout
- Inline images, quotes, and data visualizations
- CTAs placed strategically within the article flow
- Natural reading experience

```javascript
import { buildClient, buildBlockRecord } from '@datocms/cma-client';
import { 
  root, 
  heading as h, 
  paragraph as p, 
  span as s,
  list,
  listItem,
  block,
  link
} from 'datocms-structured-text-utils';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function createBlogPost() {
  const blogPostModel = await client.itemTypes.find('blog_post');
  const imageBlockType = await client.itemTypes.find('image_block');
  const ctaBlockType = await client.itemTypes.find('inline_cta_block');
  const quoteBlockType = await client.itemTypes.find('pullquote_block');

  const blogPost = await client.items.create({
    item_type: { type: 'item_type', id: blogPostModel.id },
    title: '10 Productivity Tips for Remote Workers',
    author: 'productivity-expert',
    content: {
      schema: 'dast',
      document: root(
        h(1, '10 Productivity Tips for Remote Workers'),
        
        p(
          'Working from home has become the new normal for millions. ',
          'Here are our top tips to stay productive while working remotely.'
        ),
        
        h(2, '1. Create a Dedicated Workspace'),
        
        p(
          'Having a specific area for work helps your brain switch into "work mode". ',
          'It doesn\'t have to be a separate room—even a corner of your living room works.'
        ),
        
        // Inline image showing workspace setup
        block(
          buildBlockRecord({
            item_type: { type: 'item_type', id: imageBlockType.id },
            image: { upload_id: 'workspace-photo-123' },
            caption: 'Example of a well-organized home office setup',
            alt: 'Clean desk with laptop, plant, and notebook'
          })
        ),
        
        h(2, '2. Maintain Regular Hours'),
        
        p(
          'Stick to a schedule. Start and end your workday at consistent times. ',
          'This helps maintain work-life balance and sets expectations with your team.'
        ),
        
        // Inline pullquote for emphasis
        block(
          buildBlockRecord({
            item_type: { type: 'item_type', id: quoteBlockType.id },
            quote: 'Consistency is the key to productivity in any environment.',
            attribution: 'Time Management Expert'
          })
        ),
        
        h(2, '3. Take Regular Breaks'),
        
        p('Use techniques like the Pomodoro method:'),
        
        list('bulleted',
          listItem(p('Work for 25 minutes')),
          listItem(p('Take a 5-minute break')),
          listItem(p('After 4 cycles, take a longer 15-30 minute break'))
        ),
        
        // Strategic CTA placement after giving value
        block(
          buildBlockRecord({
            item_type: { type: 'item_type', id: ctaBlockType.id },
            text: 'Want to boost your productivity even more?',
            button_text: 'Download Our Free Guide',
            button_url: '/productivity-guide-download'
          })
        ),
        
        h(2, '4. Minimize Distractions'),
        
        p(
          'Turn off notifications during focused work time. ',
          'Use apps like ', 
          link('https://freedom.to', 'Freedom'),
          ' or ',
          link('https://coldturkey.com', 'Cold Turkey'),
          ' to block distracting websites.'
        ),
        
        // ... more tips ...
        
        h(2, 'Conclusion'),
        
        p(
          'Remote work can be incredibly productive with the right strategies. ',
          'Start implementing these tips one at a time, and you\'ll see improvements in your focus and output.'
        )
      )
    }
  });

  return blogPost;
}
```

**Why Structured Text is correct here:**
- Content flows naturally as an article
- Blocks are embedded within the narrative, not separate sections
- Text formatting is essential throughout
- Readers experience continuous content with enhanced elements

## Decision Framework

Making the right choice between Modular Content and Structured Text is crucial for your content architecture. Use this comprehensive framework to guide your decision.

### Decision Tree

Follow this step-by-step decision process:

```
START: What type of content are you creating?

┌─ Page Layout/Template ─┐
│                       │
│  Are sections discrete │──YES──▶ MODULAR CONTENT
│  and reorderable?      │
│                       │
└─────────NO─────────────┘
          │
          ▼
┌─ Article/Document ─────┐
│                       │
│  Is it primarily text │──YES──▶ STRUCTURED TEXT
│  with some media?     │
│                       │
└─────────NO─────────────┘
          │
          ▼
┌─ Mixed Content ────────┐
│                       │
│  Do you need both     │──YES──▶ HYBRID APPROACH
│  layouts AND rich     │
│  text sections?       │
│                       │
└─────────NO─────────────┘
          │
          ▼
    Consider your specific use case
    (see detailed criteria below)
```

### Detailed Decision Criteria

#### Choose Modular Content if:

**Content Structure:**
1. ✅ You're building page layouts or templates
2. ✅ Content sections are independent and reorderable
3. ✅ Each block is a self-contained component
4. ✅ Sections can be mixed and matched across pages
5. ✅ Content is organized in discrete chunks

**Editor Experience:**
6. ✅ Editors need drag-and-drop functionality
7. ✅ You want a "page builder" experience
8. ✅ Non-technical users need to create complex layouts
9. ✅ Content can be previewed section by section
10. ✅ Different editors work on different sections

**Technical Requirements:**
11. ✅ You need to query specific block types
12. ✅ Blocks are rendered by different components
13. ✅ A/B testing different section combinations
14. ✅ Conditional section display based on user segments
15. ✅ Section-level analytics or personalization

**Use Cases:**
- Landing pages and homepages
- Product catalogs
- Service pages
- Marketing campaign pages
- Component libraries
- Multi-step forms

#### Choose Structured Text if:

**Content Structure:**
1. ✅ You're creating narrative content (articles, docs, guides)
2. ✅ Blocks enhance the text rather than structure it
3. ✅ Content needs rich formatting throughout
4. ✅ Reading flow and continuity is important
5. ✅ Text is the primary content with embedded media

**Editor Experience:**
6. ✅ Editors are comfortable with rich text editors
7. ✅ Content flows naturally like traditional documents
8. ✅ Inline editing and formatting is required
9. ✅ Writers need to place media within text flow
10. ✅ Content feels like writing in Word/Google Docs

**Technical Requirements:**
11. ✅ You need inline references and links
12. ✅ Search functionality needs to index flowing text
13. ✅ Comments or annotations on specific text
14. ✅ Export to other formats (PDF, EPUB, etc.)
15. ✅ Text-based SEO optimization

**Use Cases:**
- Blog posts and articles
- Documentation and help content
- News stories
- Academic papers
- Case studies
- Long-form guides

### Quick Assessment Checklist

Answer YES/NO to these questions for rapid decision-making:

| Question | Modular Content | Structured Text |
|----------|----------------|-----------------|
| Is this primarily a layout/design task? | ✅ YES | ❌ NO |
| Do sections need to be independently reorderable? | ✅ YES | ❌ NO |
| Is text the primary content medium? | ❌ NO | ✅ YES |
| Do you need rich text formatting throughout? | ❌ NO | ✅ YES |
| Are sections self-contained components? | ✅ YES | ❌ NO |
| Do blocks interrupt or enhance text flow? | ❌ Interrupt | ✅ Enhance |
| Will different team members work on different sections? | ✅ YES | ❌ NO |
| Is reading experience and flow important? | ❌ NO | ✅ YES |
| Do you need a page builder interface? | ✅ YES | ❌ NO |
| Will content be primarily consumed sequentially? | ❌ NO | ✅ YES |

**Scoring:**
- **Mostly YES in Modular Content column**: Use Modular Content
- **Mostly YES in Structured Text column**: Use Structured Text  
- **Mixed results**: Consider a hybrid approach

### Content Type Guidelines

#### Marketing & Sales Content

```javascript
// Homepage sections - Modular Content
const homepage = {
  hero_section: { /* Hero block */ },
  features_grid: { /* Features block */ },
  testimonials: { /* Testimonials block */ },
  pricing_table: { /* Pricing block */ },
  cta_section: { /* CTA block */ }
};

// Case study - Structured Text
const caseStudy = {
  content: {
    schema: 'dast',
    document: root(
      h(1, 'How Company X Increased Revenue by 300%'),
      p('When Company X came to us, they were struggling...'),
      block(statisticsChart),
      p('After implementing our solution...'),
      // ... flowing narrative with embedded blocks
    )
  }
};
```

#### Editorial Content

```javascript
// News article - Structured Text
const newsArticle = {
  content: {
    schema: 'dast',
    document: root(
      h(1, 'Breaking: Major Policy Change Announced'),
      p('In a surprise announcement today...'),
      block(relatedVideoBlock),
      p('The implications of this change include...'),
      // ... continuous narrative
    )
  }
};

// Magazine layout - Modular Content
const magazineLayout = {
  sections: [
    { type: 'featured_article', /* ... */ },
    { type: 'photo_gallery', /* ... */ },
    { type: 'sidebar_ads', /* ... */ },
    { type: 'related_stories', /* ... */ }
  ]
};
```

#### E-commerce Content

```javascript
// Product page layout - Modular Content
const productPage = {
  sections: [
    { type: 'product_hero', /* ... */ },
    { type: 'specifications_table', /* ... */ },
    { type: 'reviews_section', /* ... */ },
    { type: 'recommended_products', /* ... */ }
  ]
};

// Product description - Structured Text (if detailed)
const productDescription = {
  content: {
    schema: 'dast',
    document: root(
      h(2, 'Product Overview'),
      p('This innovative product features...'),
      block(featureComparisonTable),
      h(2, 'Technical Specifications'),
      // ... detailed narrative description
    )
  }
};
```

### Team Workflow Considerations

#### Modular Content Workflow
1. **Designer** creates block types and layouts
2. **Developer** implements block components
3. **Content editor** assembles pages from blocks
4. **Marketing** A/B tests different section combinations

#### Structured Text Workflow
1. **Writer** creates flowing content in rich text editor
2. **Editor** reviews and refines narrative flow
3. **Media specialist** adds inline images and embeds
4. **SEO specialist** optimizes text-heavy content

### Technical Implementation Guide

#### Modular Content Implementation
```javascript
// Frontend rendering (React example)
function ModularContentRenderer({ sections }) {
  return (
    <div className="page-content">
      {sections.map((section, index) => {
        const Component = getComponentForBlockType(section.type);
        return (
          <Component 
            key={section.id} 
            data={section.attributes}
            index={index}
          />
        );
      })}
    </div>
  );
}

// Easy reordering
function reorderSections(sections, fromIndex, toIndex) {
  const result = Array.from(sections);
  const [removed] = result.splice(fromIndex, 1);
  result.splice(toIndex, 0, removed);
  return result;
}
```

#### Structured Text Implementation
```javascript
// Frontend rendering (React example)
import { StructuredText } from 'react-datocms';

function StructuredTextRenderer({ content }) {
  return (
    <StructuredText
      data={content}
      renderBlock={({ record }) => {
        const Component = getComponentForBlockType(record.__typename);
        return <Component data={record} />;
      }}
      renderInlineRecord={({ record }) => {
        // Handle inline records within text
        return <InlineComponent data={record} />;
      }}
    />
  );
}

// Text search and manipulation
function searchInContent(dastDocument, searchTerm) {
  function traverseNodes(nodes) {
    return nodes.flatMap(node => {
      if (node.type === 'span' && node.value.includes(searchTerm)) {
        return [node];
      }
      if (node.children) {
        return traverseNodes(node.children);
      }
      return [];
    });
  }
  
  return traverseNodes(dastDocument.children);
}
```

## Mixed Approach

Sometimes you need both! Here's an example of a model using both field types:

```javascript
async function createHybridPageModel() {
  const pageModel = await client.itemTypes.create({
    name: 'Hybrid Page',
    api_key: 'hybrid_page',
  });

  // Structured text for the main article content
  await client.fields.create(pageModel.id, {
    label: 'Article Body',
    api_key: 'article_body',
    field_type: 'structured_text',
    hint: 'The main article content with inline media'
  });

  // Modular content for sidebar widgets
  await client.fields.create(pageModel.id, {
    label: 'Sidebar Modules',
    api_key: 'sidebar_modules',
    field_type: 'rich_text',
    hint: 'Reorderable sidebar components',
    validators: {
      rich_text_blocks: {
        item_types: ['newsletter_block', 'related_posts_block', 'ad_block']
      }
    }
  });

  return pageModel;
}
```

## Common Mistakes to Avoid

### 1. Using Modular Content for Articles
❌ **Wrong**: Using modular content for a blog post where each paragraph is a block
✅ **Right**: Using structured text with embedded media blocks

### 2. Using Structured Text for Page Layouts
❌ **Wrong**: Trying to build a homepage layout inside structured text
✅ **Right**: Using modular content for distinct page sections

### 3. Over-engineering Simple Content
❌ **Wrong**: Creating complex block structures for simple text
✅ **Right**: Using basic text/string fields when rich content isn't needed

## Migration Strategies

If you need to switch from one approach to another:

### Modular Content → Structured Text
```javascript
async function migrateModularToStructured(itemId) {
  const item = await client.items.find(itemId);
  const blocks = item.modular_content_field;
  
  // Convert blocks array to DAST
  const dastNodes = blocks.flatMap(block => {
    switch (block.relationships.item_type.data.id) {
      case 'text_block':
        return [
          h(2, block.attributes.title),
          p(block.attributes.content)
        ];
      case 'image_block':
        return [
          block, // Can embed the block directly
          p(block.attributes.caption)
        ];
      default:
        return [block];
    }
  });
  
  // Update item with structured text
  await client.items.update(itemId, {
    structured_text_field: {
      schema: 'dast',
      document: root(...dastNodes)
    }
  });
}
```

## Performance Considerations

### Modular Content
- ✅ Simple array operations
- ✅ Easy to query specific blocks
- ✅ Efficient reordering
- ⚠️ Can become unwieldy with many blocks

### Structured Text
- ✅ Single field for entire document
- ✅ Efficient for long-form content
- ⚠️ Complex tree traversal for updates
- ⚠️ Harder to query for specific embedded blocks

## Related Resources

- [Modular Content Guide](../01-cma-client/01-content-management/modular-content.md) - Deep dive into modular content
- [Structured Text Guide](../01-cma-client/01-content-management/structured-text.md) - Complete structured text reference
- [Field Types](../01-cma-client/01-content-management/field.md) - All available field types
- [Performance Guide](./performance-optimization.md) - Optimization strategies