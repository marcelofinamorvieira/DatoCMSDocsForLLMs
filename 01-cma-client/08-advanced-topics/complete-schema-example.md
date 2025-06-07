# Complete Schema Example: Building a Blog Platform

This comprehensive example demonstrates how to build a complete content schema for a modern blog platform using DatoCMS. We'll create all necessary models, fields, and relationships to support articles, authors, categories, tags, and related content.

## Overview

We'll build a blog platform with these features:
- **Articles** with rich content and SEO optimization
- **Authors** with profiles and social links
- **Categories** for content organization
- **Tags** for flexible content labeling
- **Related articles** and content recommendations
- **Comments** system
- **Newsletter** integration
- **Media** library with optimized images

## Schema Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Article   │────│   Author    │    │  Category   │
│             │    │             │    │             │
│ - title     │    │ - name      │    │ - name      │
│ - content   │    │ - bio       │    │ - slug      │
│ - seo       │    │ - avatar    │    │ - color     │
│ - featured  │    │ - social    │    │ - parent    │
└─────────────┘    └─────────────┘    └─────────────┘
       │                  │                   │
       │                  │                   │
       ▼                  ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│    Tag      │    │   Media     │    │  Comment    │
│             │    │             │    │             │
│ - name      │    │ - file      │    │ - content   │
│ - slug      │    │ - caption   │    │ - author    │
│ - color     │    │ - alt_text  │    │ - article   │
└─────────────┘    └─────────────┘    └─────────────┘
```

## Step 1: Create the Author Model

Authors are content creators with profiles and social media links.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function createAuthorModel() {
  // Create Author model
  const authorModel = await client.itemTypes.create({
    name: 'Author',
    api_key: 'author',
    collection_appearance: 'table',
    modular_block: false,
    tree: false,
    sortable: false,
    hint: 'Content authors and contributors'
  });

  // Name field
  await client.fields.create(authorModel.id, {
    label: 'Full Name',
    api_key: 'name',
    field_type: 'string',
    hint: 'Author\'s full name',
    validators: {
      required: {},
      length: { min: 2, max: 100 }
    },
    appearance: {
      editor: 'single_line',
      parameters: {
        placeholder: 'e.g. Jane Smith'
      }
    }
  });

  // Slug field (auto-generated from name)
  await client.fields.create(authorModel.id, {
    label: 'Slug',
    api_key: 'slug',
    field_type: 'slug',
    hint: 'URL-friendly version of the name',
    validators: {
      required: {},
      slug_title_field: {
        title_field_id: 'name' // Will be updated after creation
      }
    }
  });

  // Email field
  await client.fields.create(authorModel.id, {
    label: 'Email',
    api_key: 'email',
    field_type: 'string',
    hint: 'Contact email address',
    validators: {
      required: {},
      format: {
        predefined_pattern: 'email'
      }
    },
    appearance: {
      editor: 'single_line',
      parameters: {
        placeholder: 'author@example.com'
      }
    }
  });

  // Bio field
  await client.fields.create(authorModel.id, {
    label: 'Bio',
    api_key: 'bio',
    field_type: 'text',
    hint: 'Short author biography',
    validators: {
      length: { max: 500 }
    },
    appearance: {
      editor: 'textarea',
      parameters: {
        placeholder: 'Tell us about yourself...'
      }
    }
  });

  // Avatar field
  await client.fields.create(authorModel.id, {
    label: 'Avatar',
    api_key: 'avatar',
    field_type: 'file',
    hint: 'Profile picture',
    validators: {
      required: {},
      file_extension: {
        extensions: ['jpg', 'jpeg', 'png', 'webp']
      },
      file_size: {
        max_value: 2,
        max_value_unit: 'MB'
      }
    },
    appearance: {
      editor: 'file',
      parameters: {
        show_remove_button: true
      }
    }
  });

  // Social media links as JSON field
  await client.fields.create(authorModel.id, {
    label: 'Social Links',
    api_key: 'social_links',
    field_type: 'json',
    hint: 'Social media profiles',
    appearance: {
      editor: 'json',
      parameters: {
        schema: {
          type: 'object',
          properties: {
            twitter: { type: 'string', format: 'uri', title: 'Twitter URL' },
            linkedin: { type: 'string', format: 'uri', title: 'LinkedIn URL' },
            github: { type: 'string', format: 'uri', title: 'GitHub URL' },
            website: { type: 'string', format: 'uri', title: 'Personal Website' }
          },
          additionalProperties: false
        }
      }
    }
  });

  // Publication count (auto-calculated)
  await client.fields.create(authorModel.id, {
    label: 'Articles Published',
    api_key: 'articles_count',
    field_type: 'integer',
    hint: 'Total number of published articles',
    validators: {
      integer_range: { min: 0 }
    },
    appearance: {
      editor: 'integer',
      parameters: {
        readonly: true
      }
    }
  });

  return authorModel;
}
```

## Step 2: Create the Category Model

Categories provide hierarchical content organization.

```typescript
async function createCategoryModel() {
  // Create Category model with tree structure
  const categoryModel = await client.itemTypes.create({
    name: 'Category',
    api_key: 'category',
    collection_appearance: 'tree',
    modular_block: false,
    tree: true, // Enable hierarchical structure
    sortable: true,
    hint: 'Content categories for organization'
  });

  // Name field
  await client.fields.create(categoryModel.id, {
    label: 'Category Name',
    api_key: 'name',
    field_type: 'string',
    hint: 'Display name for the category',
    validators: {
      required: {},
      length: { min: 2, max: 50 }
    }
  });

  // Slug field
  await client.fields.create(categoryModel.id, {
    label: 'Slug',
    api_key: 'slug',
    field_type: 'slug',
    hint: 'URL segment for this category',
    validators: {
      required: {},
      slug_title_field: {
        title_field_id: 'name'
      }
    }
  });

  // Description field
  await client.fields.create(categoryModel.id, {
    label: 'Description',
    api_key: 'description',
    field_type: 'text',
    hint: 'Brief description of the category',
    validators: {
      length: { max: 200 }
    }
  });

  // Color field for visual distinction
  await client.fields.create(categoryModel.id, {
    label: 'Theme Color',
    api_key: 'color',
    field_type: 'color',
    hint: 'Color used in category displays',
    validators: {
      required: {}
    },
    default_value: {
      red: 59,
      green: 130,
      blue: 246,
      alpha: 255
    }
  });

  // Parent category (for hierarchical structure)
  await client.fields.create(categoryModel.id, {
    label: 'Parent Category',
    api_key: 'parent',
    field_type: 'link',
    hint: 'Parent category (leave empty for top-level)',
    validators: {
      item_item_type: {
        item_types: [categoryModel.id]
      }
    }
  });

  // Feature image for category pages
  await client.fields.create(categoryModel.id, {
    label: 'Featured Image',
    api_key: 'featured_image',
    field_type: 'file',
    hint: 'Image displayed on category pages',
    validators: {
      file_extension: {
        extensions: ['jpg', 'jpeg', 'png', 'webp']
      }
    }
  });

  // SEO fields
  await client.fields.create(categoryModel.id, {
    label: 'SEO Title',
    api_key: 'seo_title',
    field_type: 'string',
    hint: 'Title tag for search engines',
    validators: {
      length: { max: 60 }
    }
  });

  await client.fields.create(categoryModel.id, {
    label: 'SEO Description',
    api_key: 'seo_description',
    field_type: 'text',
    hint: 'Meta description for search engines',
    validators: {
      length: { max: 160 }
    }
  });

  return categoryModel;
}
```

## Step 3: Create the Tag Model

Tags provide flexible labeling for cross-cutting topics.

```typescript
async function createTagModel() {
  const tagModel = await client.itemTypes.create({
    name: 'Tag',
    api_key: 'tag',
    collection_appearance: 'table',
    modular_block: false,
    tree: false,
    sortable: false,
    hint: 'Flexible tags for content labeling'
  });

  // Name field
  await client.fields.create(tagModel.id, {
    label: 'Tag Name',
    api_key: 'name',
    field_type: 'string',
    hint: 'Display name for the tag',
    validators: {
      required: {},
      length: { min: 2, max: 30 }
    }
  });

  // Slug field
  await client.fields.create(tagModel.id, {
    label: 'Slug',
    api_key: 'slug',
    field_type: 'slug',
    hint: 'URL-friendly version of the tag name',
    validators: {
      required: {},
      slug_title_field: {
        title_field_id: 'name'
      }
    }
  });

  // Color field
  await client.fields.create(tagModel.id, {
    label: 'Color',
    api_key: 'color',
    field_type: 'color',
    hint: 'Visual color for tag display',
    default_value: {
      red: 156,
      green: 163,
      blue: 175,
      alpha: 255
    }
  });

  // Usage count (calculated field)
  await client.fields.create(tagModel.id, {
    label: 'Usage Count',
    api_key: 'usage_count',
    field_type: 'integer',
    hint: 'Number of articles using this tag',
    validators: {
      integer_range: { min: 0 }
    },
    appearance: {
      editor: 'integer',
      parameters: {
        readonly: true
      }
    }
  });

  return tagModel;
}
```

## Step 4: Create Content Block Models

Block models for modular content within articles.

```typescript
async function createContentBlocks() {
  // Image Block
  const imageBlock = await client.itemTypes.create({
    name: 'Image Block',
    api_key: 'image_block',
    modular_block: true, // This is a block model
    hint: 'Embedded image with caption'
  });

  await client.fields.create(imageBlock.id, {
    label: 'Image',
    api_key: 'image',
    field_type: 'file',
    validators: {
      required: {},
      file_extension: {
        extensions: ['jpg', 'jpeg', 'png', 'webp', 'gif']
      }
    }
  });

  await client.fields.create(imageBlock.id, {
    label: 'Caption',
    api_key: 'caption',
    field_type: 'string',
    hint: 'Image caption or description'
  });

  await client.fields.create(imageBlock.id, {
    label: 'Alt Text',
    api_key: 'alt_text',
    field_type: 'string',
    hint: 'Alternative text for accessibility',
    validators: {
      required: {}
    }
  });

  await client.fields.create(imageBlock.id, {
    label: 'Size',
    api_key: 'size',
    field_type: 'string',
    hint: 'Image display size',
    validators: {
      enum: {
        values: ['small', 'medium', 'large', 'full']
      }
    },
    default_value: 'large'
  });

  // Quote Block
  const quoteBlock = await client.itemTypes.create({
    name: 'Quote Block',
    api_key: 'quote_block',
    modular_block: true,
    hint: 'Highlighted quote or testimonial'
  });

  await client.fields.create(quoteBlock.id, {
    label: 'Quote Text',
    api_key: 'quote',
    field_type: 'text',
    validators: {
      required: {},
      length: { max: 500 }
    }
  });

  await client.fields.create(quoteBlock.id, {
    label: 'Author',
    api_key: 'author',
    field_type: 'string',
    hint: 'Who said this quote?'
  });

  await client.fields.create(quoteBlock.id, {
    label: 'Author Title',
    api_key: 'author_title',
    field_type: 'string',
    hint: 'Author\'s title or position'
  });

  // Code Block
  const codeBlock = await client.itemTypes.create({
    name: 'Code Block',
    api_key: 'code_block',
    modular_block: true,
    hint: 'Syntax-highlighted code snippet'
  });

  await client.fields.create(codeBlock.id, {
    label: 'Code',
    api_key: 'code',
    field_type: 'text',
    validators: {
      required: {}
    },
    appearance: {
      editor: 'textarea',
      parameters: {
        monospace_font: true
      }
    }
  });

  await client.fields.create(codeBlock.id, {
    label: 'Language',
    api_key: 'language',
    field_type: 'string',
    hint: 'Programming language for syntax highlighting',
    validators: {
      enum: {
        values: [
          'javascript', 'typescript', 'python', 'java', 'php',
          'ruby', 'go', 'rust', 'css', 'html', 'json', 'yaml'
        ]
      }
    },
    default_value: 'javascript'
  });

  await client.fields.create(codeBlock.id, {
    label: 'Caption',
    api_key: 'caption',
    field_type: 'string',
    hint: 'Optional code block caption'
  });

  // Newsletter CTA Block
  const newsletterBlock = await client.itemTypes.create({
    name: 'Newsletter CTA',
    api_key: 'newsletter_cta',
    modular_block: true,
    hint: 'Newsletter subscription call-to-action'
  });

  await client.fields.create(newsletterBlock.id, {
    label: 'Headline',
    api_key: 'headline',
    field_type: 'string',
    default_value: 'Subscribe to our newsletter',
    validators: {
      required: {}
    }
  });

  await client.fields.create(newsletterBlock.id, {
    label: 'Description',
    api_key: 'description',
    field_type: 'text',
    default_value: 'Get the latest articles delivered straight to your inbox.'
  });

  await client.fields.create(newsletterBlock.id, {
    label: 'Button Text',
    api_key: 'button_text',
    field_type: 'string',
    default_value: 'Subscribe Now'
  });

  return {
    imageBlock,
    quoteBlock,
    codeBlock,
    newsletterBlock
  };
}
```

## Step 5: Create the Article Model

The main content model with rich fields and relationships.

```typescript
async function createArticleModel(authorModel, categoryModel, tagModel, blocks) {
  const articleModel = await client.itemTypes.create({
    name: 'Article',
    api_key: 'article',
    collection_appearance: 'table',
    modular_block: false,
    tree: false,
    sortable: true,
    hint: 'Blog articles and posts'
  });

  // Title field
  await client.fields.create(articleModel.id, {
    label: 'Title',
    api_key: 'title',
    field_type: 'string',
    hint: 'Article headline',
    validators: {
      required: {},
      length: { min: 5, max: 100 }
    },
    appearance: {
      editor: 'single_line',
      parameters: {
        placeholder: 'Enter a compelling title...'
      }
    }
  });

  // Slug field
  await client.fields.create(articleModel.id, {
    label: 'Slug',
    api_key: 'slug',
    field_type: 'slug',
    hint: 'URL path for this article',
    validators: {
      required: {},
      slug_title_field: {
        title_field_id: 'title'
      },
      unique: {}
    }
  });

  // Excerpt field
  await client.fields.create(articleModel.id, {
    label: 'Excerpt',
    api_key: 'excerpt',
    field_type: 'text',
    hint: 'Brief summary for previews and SEO',
    validators: {
      required: {},
      length: { min: 50, max: 300 }
    }
  });

  // Content field (Structured Text with blocks)
  await client.fields.create(articleModel.id, {
    label: 'Content',
    api_key: 'content',
    field_type: 'structured_text',
    hint: 'Main article content',
    validators: {
      required: {},
      structured_text_blocks: {
        item_types: [
          blocks.imageBlock.id,
          blocks.quoteBlock.id,
          blocks.codeBlock.id,
          blocks.newsletterBlock.id
        ]
      },
      structured_text_links: {
        item_types: [articleModel.id] // Self-referencing for internal links
      }
    }
  });

  // Featured image
  await client.fields.create(articleModel.id, {
    label: 'Featured Image',
    api_key: 'featured_image',
    field_type: 'file',
    hint: 'Main image for the article',
    validators: {
      required: {},
      file_extension: {
        extensions: ['jpg', 'jpeg', 'png', 'webp']
      }
    }
  });

  // Author relationship
  await client.fields.create(articleModel.id, {
    label: 'Author',
    api_key: 'author',
    field_type: 'link',
    hint: 'Article author',
    validators: {
      required: {},
      item_item_type: {
        item_types: [authorModel.id]
      }
    }
  });

  // Category relationship
  await client.fields.create(articleModel.id, {
    label: 'Category',
    api_key: 'category',
    field_type: 'link',
    hint: 'Primary category',
    validators: {
      required: {},
      item_item_type: {
        item_types: [categoryModel.id]
      }
    }
  });

  // Tags relationship (multiple)
  await client.fields.create(articleModel.id, {
    label: 'Tags',
    api_key: 'tags',
    field_type: 'links',
    hint: 'Related tags',
    validators: {
      items_item_type: {
        item_types: [tagModel.id]
      },
      size: { max: 10 }
    }
  });

  // Related articles
  await client.fields.create(articleModel.id, {
    label: 'Related Articles',
    api_key: 'related_articles',
    field_type: 'links',
    hint: 'Manually curated related articles',
    validators: {
      items_item_type: {
        item_types: [articleModel.id]
      },
      size: { max: 5 }
    }
  });

  // Publication status
  await client.fields.create(articleModel.id, {
    label: 'Status',
    api_key: 'status',
    field_type: 'string',
    hint: 'Publication status',
    validators: {
      required: {},
      enum: {
        values: ['draft', 'review', 'published', 'archived']
      }
    },
    default_value: 'draft',
    appearance: {
      editor: 'select',
      parameters: {
        options: [
          { label: 'Draft', value: 'draft' },
          { label: 'In Review', value: 'review' },
          { label: 'Published', value: 'published' },
          { label: 'Archived', value: 'archived' }
        ]
      }
    }
  });

  // Publication date
  await client.fields.create(articleModel.id, {
    label: 'Published At',
    api_key: 'published_at',
    field_type: 'date_time',
    hint: 'When this article was/will be published'
  });

  // Reading time (calculated)
  await client.fields.create(articleModel.id, {
    label: 'Reading Time',
    api_key: 'reading_time',
    field_type: 'integer',
    hint: 'Estimated reading time in minutes',
    validators: {
      integer_range: { min: 1 }
    }
  });

  // View count
  await client.fields.create(articleModel.id, {
    label: 'View Count',
    api_key: 'view_count',
    field_type: 'integer',
    hint: 'Number of page views',
    default_value: 0,
    validators: {
      integer_range: { min: 0 }
    }
  });

  // Featured flag
  await client.fields.create(articleModel.id, {
    label: 'Featured',
    api_key: 'featured',
    field_type: 'boolean',
    hint: 'Mark as featured article',
    default_value: false
  });

  // SEO fields group
  await client.fields.create(articleModel.id, {
    label: 'SEO Title',
    api_key: 'seo_title',
    field_type: 'string',
    hint: 'Title tag for search engines (defaults to article title)',
    validators: {
      length: { max: 60 }
    }
  });

  await client.fields.create(articleModel.id, {
    label: 'SEO Description',
    api_key: 'seo_description',
    field_type: 'text',
    hint: 'Meta description for search engines (defaults to excerpt)',
    validators: {
      length: { max: 160 }
    }
  });

  await client.fields.create(articleModel.id, {
    label: 'SEO Keywords',
    api_key: 'seo_keywords',
    field_type: 'string',
    hint: 'Comma-separated keywords for SEO'
  });

  // Open Graph image
  await client.fields.create(articleModel.id, {
    label: 'Social Share Image',
    api_key: 'og_image',
    field_type: 'file',
    hint: 'Image for social media sharing (defaults to featured image)',
    validators: {
      file_extension: {
        extensions: ['jpg', 'jpeg', 'png']
      }
    }
  });

  return articleModel;
}
```

## Step 6: Create Additional Support Models

Models for comments, analytics, and other supporting content.

```typescript
async function createSupportModels(articleModel, authorModel) {
  // Comment Model
  const commentModel = await client.itemTypes.create({
    name: 'Comment',
    api_key: 'comment',
    collection_appearance: 'table',
    modular_block: false,
    tree: false,
    sortable: true,
    hint: 'Reader comments on articles'
  });

  await client.fields.create(commentModel.id, {
    label: 'Content',
    api_key: 'content',
    field_type: 'text',
    validators: {
      required: {},
      length: { min: 10, max: 1000 }
    }
  });

  await client.fields.create(commentModel.id, {
    label: 'Author Name',
    api_key: 'author_name',
    field_type: 'string',
    validators: {
      required: {}
    }
  });

  await client.fields.create(commentModel.id, {
    label: 'Author Email',
    api_key: 'author_email',
    field_type: 'string',
    validators: {
      required: {},
      format: {
        predefined_pattern: 'email'
      }
    }
  });

  await client.fields.create(commentModel.id, {
    label: 'Article',
    api_key: 'article',
    field_type: 'link',
    validators: {
      required: {},
      item_item_type: {
        item_types: [articleModel.id]
      }
    }
  });

  await client.fields.create(commentModel.id, {
    label: 'Status',
    api_key: 'status',
    field_type: 'string',
    validators: {
      enum: {
        values: ['pending', 'approved', 'spam', 'deleted']
      }
    },
    default_value: 'pending'
  });

  // Newsletter Subscriber Model
  const subscriberModel = await client.itemTypes.create({
    name: 'Newsletter Subscriber',
    api_key: 'newsletter_subscriber',
    collection_appearance: 'table',
    hint: 'Email newsletter subscribers'
  });

  await client.fields.create(subscriberModel.id, {
    label: 'Email',
    api_key: 'email',
    field_type: 'string',
    validators: {
      required: {},
      unique: {},
      format: {
        predefined_pattern: 'email'
      }
    }
  });

  await client.fields.create(subscriberModel.id, {
    label: 'First Name',
    api_key: 'first_name',
    field_type: 'string'
  });

  await client.fields.create(subscriberModel.id, {
    label: 'Subscribed At',
    api_key: 'subscribed_at',
    field_type: 'date_time',
    default_value: new Date().toISOString()
  });

  await client.fields.create(subscriberModel.id, {
    label: 'Active',
    api_key: 'active',
    field_type: 'boolean',
    default_value: true
  });

  return {
    commentModel,
    subscriberModel
  };
}
```

## Step 7: Set Up Publishing Workflow

Configure editorial workflow and permissions.

```typescript
async function setupPublishingWorkflow() {
  // Create roles for different user types
  const editorRole = await client.roles.create({
    name: 'Editor',
    can_edit_schema: false,
    can_edit_favicon: false,
    can_edit_site: false,
    can_manage_users: false,
    can_manage_access_tokens: false,
    can_perform_site_search: true,
    can_duplicate_records: true,
    can_import_and_export: false
  });

  const authorRole = await client.roles.create({
    name: 'Author',
    can_edit_schema: false,
    can_edit_favicon: false,
    can_edit_site: false,
    can_manage_users: false,
    can_manage_access_tokens: false,
    can_perform_site_search: true,
    can_duplicate_records: false,
    can_import_and_export: false
  });

  // Set up item type permissions
  const articleModel = await client.itemTypes.find('article');
  
  // Authors can create and edit their own articles
  await client.itemTypeFilters.create({
    item_type: { type: 'item_type', id: articleModel.id },
    name: 'Own Articles Only',
    filter: {
      fields: {
        author: {
          eq: '$currentUser.id'
        }
      }
    }
  });

  return {
    editorRole,
    authorRole
  };
}
```

## Step 8: Complete Implementation Example

Here's how to create a complete article with all relationships:

```typescript
async function createSampleContent() {
  // Create an author
  const author = await client.items.create({
    item_type: { type: 'item_type', id: 'author' },
    name: 'Jane Developer',
    email: 'jane@example.com',
    bio: 'Full-stack developer with 10 years of experience in web technologies.',
    social_links: {
      twitter: 'https://twitter.com/janedev',
      github: 'https://github.com/janedev',
      linkedin: 'https://linkedin.com/in/janedev'
    }
  });

  // Upload author avatar
  const avatarUpload = await client.uploads.create({
    path: './assets/jane-avatar.jpg', // Local file path
    filename: 'jane-avatar.jpg',
    default_field_metadata: {
      en: {
        alt: 'Jane Developer profile photo',
        title: 'Jane Developer',
        custom_data: {}
      }
    }
  });

  // Update author with avatar
  await client.items.update(author.id, {
    avatar: { upload_id: avatarUpload.id }
  });

  // Create categories
  const techCategory = await client.items.create({
    item_type: { type: 'item_type', id: 'category' },
    name: 'Technology',
    slug: 'technology',
    description: 'Latest in tech trends and tutorials',
    color: { red: 59, green: 130, blue: 246, alpha: 255 }
  });

  const webdevCategory = await client.items.create({
    item_type: { type: 'item_type', id: 'category' },
    name: 'Web Development',
    slug: 'web-development',
    description: 'Frontend and backend development guides',
    parent: { type: 'item', id: techCategory.id },
    color: { red: 16, green: 185, blue: 129, alpha: 255 }
  });

  // Create tags
  const reactTag = await client.items.create({
    item_type: { type: 'item_type', id: 'tag' },
    name: 'React',
    slug: 'react',
    color: { red: 97, green: 218, blue: 251, alpha: 255 }
  });

  const javascriptTag = await client.items.create({
    item_type: { type: 'item_type', id: 'tag' },
    name: 'JavaScript',
    slug: 'javascript',
    color: { red: 255, green: 221, blue: 87, alpha: 255 }
  });

  // Upload featured image
  const featuredImageUpload = await client.uploads.create({
    path: './assets/react-tutorial-cover.jpg',
    filename: 'react-tutorial-cover.jpg',
    default_field_metadata: {
      en: {
        alt: 'React hooks tutorial illustration',
        title: 'React Hooks Tutorial',
        custom_data: {
          photographer: 'UI Designer',
          source: 'Internal'
        }
      }
    }
  });

  // Create article with structured content
  const article = await client.items.create({
    item_type: { type: 'item_type', id: 'article' },
    title: 'Mastering React Hooks: A Complete Guide',
    slug: 'mastering-react-hooks-complete-guide',
    excerpt: 'Learn how to effectively use React Hooks to build modern, functional components with state management, effects, and custom logic.',
    content: {
      schema: 'dast',
      document: {
        type: 'root',
        children: [
          {
            type: 'heading',
            level: 1,
            children: [
              { type: 'span', value: 'Mastering React Hooks: A Complete Guide' }
            ]
          },
          {
            type: 'paragraph',
            children: [
              { type: 'span', value: 'React Hooks revolutionized how we write React components. Introduced in React 16.8, hooks allow us to use state and lifecycle features in functional components.' }
            ]
          },
          {
            type: 'heading',
            level: 2,
            children: [
              { type: 'span', value: 'What are React Hooks?' }
            ]
          },
          {
            type: 'paragraph',
            children: [
              { type: 'span', value: 'Hooks are functions that let you "hook into" React features. They start with ' },
              { type: 'span', marks: ['code'], value: 'use' },
              { type: 'span', value: ' and follow specific rules.' }
            ]
          },
          {
            type: 'block',
            item: {
              type: 'item',
              id: 'temp-code-block-1',
              attributes: {
                code: 'import React, { useState, useEffect } from \'react\';\n\nfunction Counter() {\n  const [count, setCount] = useState(0);\n  \n  useEffect(() => {\n    document.title = `Count: ${count}`;\n  }, [count]);\n  \n  return (\n    <div>\n      <p>You clicked {count} times</p>\n      <button onClick={() => setCount(count + 1)}>\n        Click me\n      </button>\n    </div>\n  );\n}',
                language: 'javascript',
                caption: 'Basic useState and useEffect example'
              },
              relationships: {
                item_type: {
                  data: { type: 'item_type', id: 'code_block' }
                }
              }
            }
          },
          {
            type: 'heading',
            level: 2,
            children: [
              { type: 'span', value: 'Common Hooks' }
            ]
          },
          {
            type: 'list',
            style: 'bulleted',
            children: [
              {
                type: 'listItem',
                children: [
                  {
                    type: 'paragraph',
                    children: [
                      { type: 'span', marks: ['strong'], value: 'useState' },
                      { type: 'span', value: ' - Manage component state' }
                    ]
                  }
                ]
              },
              {
                type: 'listItem',
                children: [
                  {
                    type: 'paragraph',
                    children: [
                      { type: 'span', marks: ['strong'], value: 'useEffect' },
                      { type: 'span', value: ' - Handle side effects' }
                    ]
                  }
                ]
              },
              {
                type: 'listItem',
                children: [
                  {
                    type: 'paragraph',
                    children: [
                      { type: 'span', marks: ['strong'], value: 'useContext' },
                      { type: 'span', value: ' - Access React context' }
                    ]
                  }
                ]
              }
            ]
          },
          {
            type: 'block',
            item: {
              type: 'item',
              id: 'temp-newsletter-block-1',
              attributes: {
                headline: 'Want to learn more React?',
                description: 'Subscribe to our newsletter for weekly React tips and tutorials.',
                button_text: 'Subscribe Now'
              },
              relationships: {
                item_type: {
                  data: { type: 'item_type', id: 'newsletter_cta' }
                }
              }
            }
          }
        ]
      }
    },
    featured_image: { upload_id: featuredImageUpload.id },
    author: { type: 'item', id: author.id },
    category: { type: 'item', id: webdevCategory.id },
    tags: [
      { type: 'item', id: reactTag.id },
      { type: 'item', id: javascriptTag.id }
    ],
    status: 'published',
    published_at: new Date().toISOString(),
    reading_time: 8,
    featured: true,
    seo_title: 'Master React Hooks - Complete Tutorial & Guide 2024',
    seo_description: 'Learn React Hooks with practical examples. Complete guide covering useState, useEffect, custom hooks, and best practices for modern React development.',
    seo_keywords: 'react hooks, useState, useEffect, react tutorial, javascript'
  });

  return {
    author,
    article,
    categories: { techCategory, webdevCategory },
    tags: { reactTag, javascriptTag }
  };
}
```

## Step 9: Frontend Integration

Example of how to query and render this content:

```typescript
// GraphQL query for article list
const ARTICLES_QUERY = `
  query GetArticles($first: IntType!, $skip: IntType!) {
    allArticles(
      first: $first
      skip: $skip
      filter: { status: { eq: "published" } }
      orderBy: publishedAt_DESC
    ) {
      id
      title
      slug
      excerpt
      readingTime
      publishedAt
      featured
      featuredImage {
        url
        alt
        responsiveImage(imgixParams: { fit: crop, w: 400, h: 250 }) {
          srcSet
          webpSrcSet
          sizes
          src
          width
          height
          aspectRatio
          alt
          title
          base64
        }
      }
      author {
        name
        slug
        avatar {
          url
        }
      }
      category {
        name
        slug
        color {
          hex
        }
      }
      tags {
        name
        slug
        color {
          hex
        }
      }
    }
  }
`;

// GraphQL query for single article
const ARTICLE_QUERY = `
  query GetArticle($slug: String!) {
    article(filter: { slug: { eq: $slug } }) {
      id
      title
      slug
      excerpt
      content {
        value
        blocks {
          ... on ImageBlockRecord {
            id
            __typename
            image {
              url
              alt
              responsiveImage(imgixParams: { fit: crop, w: 800 }) {
                srcSet
                webpSrcSet
                sizes
                src
                width
                height
                aspectRatio
                alt
                title
                base64
              }
            }
            caption
            size
          }
          ... on QuoteBlockRecord {
            id
            __typename
            quote
            author
            authorTitle
          }
          ... on CodeBlockRecord {
            id
            __typename
            code
            language
            caption
          }
          ... on NewsletterCtaRecord {
            id
            __typename
            headline
            description
            buttonText
          }
        }
      }
      featuredImage {
        url
        alt
        responsiveImage(imgixParams: { fit: crop, w: 1200, h: 630 }) {
          srcSet
          webpSrcSet
          sizes
          src
          width
          height
          aspectRatio
          alt
          title
          base64
        }
      }
      author {
        name
        slug
        bio
        avatar {
          url
        }
        socialLinks
      }
      category {
        name
        slug
        color {
          hex
        }
      }
      tags {
        name
        slug
        color {
          hex
        }
      }
      relatedArticles {
        id
        title
        slug
        excerpt
        featuredImage {
          url
          alt
        }
      }
      publishedAt
      readingTime
      viewCount
      seoTitle
      seoDescription
      ogImage {
        url
      }
    }
  }
`;

// React component example
import { StructuredText } from 'react-datocms';

function Article({ article }) {
  return (
    <article className="max-w-4xl mx-auto px-4 py-8">
      <header className="mb-8">
        <h1 className="text-4xl font-bold mb-4">{article.title}</h1>
        <div className="flex items-center gap-4 text-gray-600 mb-6">
          <img 
            src={article.author.avatar.url} 
            alt={article.author.name}
            className="w-10 h-10 rounded-full"
          />
          <div>
            <p className="font-medium">{article.author.name}</p>
            <p className="text-sm">
              {new Date(article.publishedAt).toLocaleDateString()} • 
              {article.readingTime} min read
            </p>
          </div>
        </div>
        <img 
          src={article.featuredImage.url}
          alt={article.featuredImage.alt}
          className="w-full h-64 object-cover rounded-lg"
        />
      </header>
      
      <div className="prose prose-lg max-w-none">
        <StructuredText
          data={article.content}
          renderBlock={({ record }) => {
            switch (record.__typename) {
              case 'ImageBlockRecord':
                return (
                  <figure className={`image-block image-${record.size}`}>
                    <img src={record.image.url} alt={record.image.alt} />
                    {record.caption && <figcaption>{record.caption}</figcaption>}
                  </figure>
                );
              case 'QuoteBlockRecord':
                return (
                  <blockquote className="quote-block">
                    <p>"{record.quote}"</p>
                    <cite>
                      {record.author}
                      {record.authorTitle && `, ${record.authorTitle}`}
                    </cite>
                  </blockquote>
                );
              case 'CodeBlockRecord':
                return (
                  <div className="code-block">
                    <pre>
                      <code className={`language-${record.language}`}>
                        {record.code}
                      </code>
                    </pre>
                    {record.caption && <p className="caption">{record.caption}</p>}
                  </div>
                );
              case 'NewsletterCtaRecord':
                return (
                  <div className="newsletter-cta bg-blue-50 p-6 rounded-lg">
                    <h3>{record.headline}</h3>
                    <p>{record.description}</p>
                    <button className="bg-blue-600 text-white px-4 py-2 rounded">
                      {record.buttonText}
                    </button>
                  </div>
                );
              default:
                return null;
            }
          }}
        />
      </div>
      
      <footer className="mt-12 pt-8 border-t">
        <div className="flex flex-wrap gap-2 mb-6">
          {article.tags.map(tag => (
            <span 
              key={tag.slug}
              className="px-3 py-1 rounded-full text-sm"
              style={{ backgroundColor: tag.color.hex + '20', color: tag.color.hex }}
            >
              #{tag.name}
            </span>
          ))}
        </div>
        
        {article.relatedArticles.length > 0 && (
          <div>
            <h3 className="text-xl font-semibold mb-4">Related Articles</h3>
            <div className="grid md:grid-cols-2 gap-4">
              {article.relatedArticles.map(related => (
                <a key={related.id} href={`/articles/${related.slug}`} className="block">
                  <img src={related.featuredImage.url} alt={related.featuredImage.alt} />
                  <h4>{related.title}</h4>
                  <p>{related.excerpt}</p>
                </a>
              ))}
            </div>
          </div>
        )}
      </footer>
    </article>
  );
}
```

## Complete Setup Function

Here's a single function that sets up the entire schema:

```typescript
async function setupCompleteBlogSchema() {
  console.log('🚀 Setting up complete blog schema...');

  try {
    // Step 1: Create basic models
    console.log('📝 Creating Author model...');
    const authorModel = await createAuthorModel();
    
    console.log('📂 Creating Category model...');
    const categoryModel = await createCategoryModel();
    
    console.log('🏷️ Creating Tag model...');
    const tagModel = await createTagModel();
    
    console.log('🧩 Creating content blocks...');
    const blocks = await createContentBlocks();
    
    console.log('📰 Creating Article model...');
    const articleModel = await createArticleModel(authorModel, categoryModel, tagModel, blocks);
    
    console.log('💬 Creating support models...');
    const supportModels = await createSupportModels(articleModel, authorModel);
    
    console.log('👥 Setting up publishing workflow...');
    const workflow = await setupPublishingWorkflow();
    
    console.log('🎨 Creating sample content...');
    const sampleContent = await createSampleContent();
    
    console.log('✅ Blog schema setup complete!');
    
    return {
      models: {
        author: authorModel,
        category: categoryModel,
        tag: tagModel,
        article: articleModel,
        ...blocks,
        ...supportModels
      },
      workflow,
      sampleContent
    };
    
  } catch (error) {
    console.error('❌ Error setting up blog schema:', error);
    throw error;
  }
}

// Run the setup
setupCompleteBlogSchema()
  .then(result => {
    console.log('🎉 Blog platform ready!');
    console.log('Models created:', Object.keys(result.models).length);
    console.log('Sample content:', Object.keys(result.sampleContent).length);
  })
  .catch(error => {
    console.error('Setup failed:', error);
  });
```

## Best Practices Demonstrated

This example demonstrates several DatoCMS best practices:

1. **Modular Design**: Separate models for different content types
2. **Relationships**: Proper use of links and references between models
3. **Validation**: Comprehensive field validation rules
4. **SEO Optimization**: Built-in SEO fields and meta tags
5. **Editor Experience**: User-friendly field hints and appearances
6. **Content Blocks**: Reusable modular content components
7. **Workflow Management**: Editorial roles and permissions
8. **Performance**: Optimized image handling and responsive images
9. **Extensibility**: Easy to add new content types and fields
10. **Real-world Usage**: Practical field types and constraints

This complete example provides a solid foundation for any content-driven website or application using DatoCMS.