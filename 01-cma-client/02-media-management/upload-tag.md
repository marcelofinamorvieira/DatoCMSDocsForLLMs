# Upload Tag Resource

## Overview

Upload Tags provide a flexible way to categorize and organize your media assets using keywords. Unlike collections which are hierarchical, tags offer a flat, multi-dimensional way to classify uploads.

## API Client Section

`client.uploadTags`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// List all existing tags
const tags = await client.uploadTags.list();
// Returns: ['product', 'marketing', 'featured', 'banner', '2024']

// Tags are created by adding them to uploads
const upload = await client.uploads.create({
  path: '/path/to/file.jpg',
  tags: ['product', 'featured', '2024']
});
```

## API Reference

### list() - List All Upload Tags

Retrieve all unique tags that have been applied to uploads in your project.

**Signature**: `list(): Promise<string[]>`

**Returns**: Array of tag names (strings)

**Example**:
```typescript
// Get all upload tags
const tags = await client.uploadTags.list();
// Returns array like: ['product', 'marketing', 'featured', 'banner', '2024']

// Get raw response
const response = await client.uploadTags.rawList();
// response.data contains the array of tags
```

## Working with Tags

Upload tags in DatoCMS are managed through the Upload resource. The uploadTags endpoint provides a read-only list of all existing tags.

### Creating Tags

Tags are created automatically when you add them to uploads:

```typescript
// Add tags when creating an upload
const uploadWithTags = await client.uploads.create({
  path: '/path/to/file.jpg',
  tags: ['product', 'featured', '2024']
});

// Add tags to existing upload
const updated = await client.uploads.update('upload-id', {
  tags: ['marketing', 'banner', 'homepage']
});
```

**Note:** Tag names are automatically normalized (lowercase, spaces to hyphens)

### Managing Tags on Uploads

```typescript
// Get current tags on an upload
const upload = await client.uploads.find('upload-id');
const currentTags = upload.tags || [];

// Add new tags (preserving existing)
const addTags = await client.uploads.update('upload-id', {
  tags: [...new Set([...currentTags, 'new-tag', 'another-tag'])]
});

// Remove specific tags
const removeTags = await client.uploads.update('upload-id', {
  tags: currentTags.filter(tag => tag !== 'unwanted-tag')
});

// Replace all tags
const replaceTags = await client.uploads.update('upload-id', {
  tags: ['completely', 'new', 'tags']
});

// Clear all tags
const clearTags = await client.uploads.update('upload-id', {
  tags: []
});
```

### Bulk Tag Operations

Apply tags to multiple uploads at once:

```typescript
// Tag multiple uploads
await client.uploads.bulkTag({
  uploads: [
    { type: 'upload', id: 'upload-1' },
    { type: 'upload', id: 'upload-2' },
    { type: 'upload', id: 'upload-3' }
  ],
  tags: ['batch-processed', 'reviewed']
});

// Tag all uploads matching criteria
const untaggedUploads = await client.uploads.list({
  filter: {
    tags: { eq: [] } // No tags
  }
});

if (untaggedUploads.length > 0) {
  await client.uploads.bulkTag({
    uploads: untaggedUploads.map(u => ({ type: 'upload', id: u.id })),
    tags: ['untagged', 'needs-review']
  });
}
```

### Filtering by Tags

Find uploads with specific tags:

```typescript
// Find uploads with ANY of these tags
const heroImages = await client.uploads.list({
  filter: {
    tags: { any_in: ['hero', 'banner', 'header'] }
  }
});

// Find uploads with ALL of these tags
const featuredProducts = await client.uploads.list({
  filter: {
    tags: { all_in: ['product', 'featured'] }
  }
});

// Find uploads without specific tags
const nonArchived = await client.uploads.list({
  filter: {
    tags: { not_in: ['archived', 'deleted'] }
  }
});
```

## Tag Management Patterns

### Tag Naming Conventions

Establish consistent naming patterns:

```typescript
// Category-based tags
const categoryTags = [
  'category:products',
  'category:blog',
  'category:team',
  'category:events'
];

// Status tags
const statusTags = [
  'status:draft',
  'status:approved',
  'status:published',
  'status:archived'
];

// Date-based tags
const dateTags = [
  'year:2024',
  'month:2024-06',
  'quarter:q2-2024',
  'season:summer-2024'
];

// Create structured tags
async function createStructuredTags() {
  const allTags = [...categoryTags, ...statusTags, ...dateTags];
  
  for (const tagName of allTags) {
    // Apply tag to a temporary upload to create it
    const upload = await client.uploads.list({ page: { limit: 1 } });
    if (upload.length > 0) {
      const currentTags = upload[0].tags || [];
      await client.uploads.update(upload[0].id, {
        tags: [...currentTags, tagName]
      });
    }
  }
}
```

### Tag Hierarchies with Prefixes

Simulate hierarchical organization using prefixes:

```typescript
// Create pseudo-hierarchical tags
const hierarchicalTags = [
  // Location tags
  'location:usa',
  'location:usa:ny',
  'location:usa:ny:nyc',
  'location:europe',
  'location:europe:uk',
  'location:europe:uk:london',
  
  // Product tags
  'product:electronics',
  'product:electronics:phones',
  'product:electronics:laptops',
  'product:clothing',
  'product:clothing:mens',
  'product:clothing:womens'
];

// Filter by hierarchy level
async function getUploadsByLocation(country: string, region?: string, city?: string) {
  let tagFilter = `location:${country}`;
  if (region) tagFilter += `:${region}`;
  if (city) tagFilter += `:${city}`;
  
  return client.uploads.list({
    filter: {
      tags: { any_in: [tagFilter] }
    }
  });
}
```

### Tag Analytics

Track tag usage across your media library:

```typescript
async function analyzeTagUsage() {
  // Get all tags
  const allTags = await client.uploadTags.list();
  
  // Get all uploads to count tag usage
  const uploads = await client.uploads.list({
    page: { limit: 500 }
  });
  
  const tagAnalytics = {
    totalTags: allTags.length,
    tagUsage: new Map(),
    unusedTags: [],
    popularTags: [],
    tagCombinations: new Map()
  };
  
  // Initialize tag counters
  allTags.forEach(tag => tagAnalytics.tagUsage.set(tag, 0));
  
  // Count tag usage
  for (const upload of uploads) {
    if (upload.tags && upload.tags.length > 0) {
      // Count individual tags
      upload.tags.forEach(tag => {
        const count = tagAnalytics.tagUsage.get(tag) || 0;
        tagAnalytics.tagUsage.set(tag, count + 1);
      });
      
      // Track tag combinations
      const combination = upload.tags.sort().join(',');
      const comboCount = tagAnalytics.tagCombinations.get(combination) || 0;
      tagAnalytics.tagCombinations.set(combination, comboCount + 1);
    }
  }
  
  // Find unused tags
  tagAnalytics.unusedTags = allTags.filter(
    tag => tagAnalytics.tagUsage.get(tag) === 0
  );
  
  // Find most popular tags
  const sortedTags = Array.from(tagAnalytics.tagUsage.entries())
    .sort((a, b) => b[1] - a[1])
    .slice(0, 10);
  
  tagAnalytics.popularTags = sortedTags.map(([tag, count]) => ({
    tag,
    count,
    percentage: Math.round((count / uploads.length) * 100)
  }));
  
  return tagAnalytics;
}
```

### Tag Normalization

Ensure consistent tag formatting:

```typescript
async function normalizeUploadTags() {
  const uploads = await client.uploads.list({
    page: { limit: 500 }
  });
  
  const normalizationRules = {
    // Convert to lowercase
    lowercase: (tag) => tag.toLowerCase(),
    
    // Replace spaces with hyphens
    hyphenate: (tag) => tag.replace(/\s+/g, '-'),
    
    // Remove special characters
    alphanumeric: (tag) => tag.replace(/[^a-zA-Z0-9-]/g, ''),
    
    // Trim whitespace
    trim: (tag) => tag.trim(),
    
    // Map synonyms
    synonyms: (tag) => {
      const synonymMap = {
        'photo': 'image',
        'picture': 'image',
        'movie': 'video',
        'film': 'video',
        'doc': 'document',
        'docs': 'document'
      };
      return synonymMap[tag] || tag;
    }
  };
  
  const results = {
    processed: 0,
    updated: 0,
    changes: []
  };
  
  for (const upload of uploads) {
    if (!upload.tags || upload.tags.length === 0) continue;
    
    const originalTags = [...upload.tags];
    const normalizedTags = upload.tags.map(tag => {
      let normalized = tag;
      normalized = normalizationRules.lowercase(normalized);
      normalized = normalizationRules.trim(normalized);
      normalized = normalizationRules.hyphenate(normalized);
      normalized = normalizationRules.alphanumeric(normalized);
      normalized = normalizationRules.synonyms(normalized);
      return normalized;
    });
    
    // Remove duplicates
    const uniqueTags = [...new Set(normalizedTags)].filter(tag => tag.length > 0);
    
    // Check if tags changed
    if (JSON.stringify(originalTags.sort()) !== JSON.stringify(uniqueTags.sort())) {
      try {
        await client.uploads.update(upload.id, {
          tags: uniqueTags
        });
        
        results.updated++;
        results.changes.push({
          uploadId: upload.id,
          before: originalTags,
          after: uniqueTags
        });
      } catch (error) {
        console.error(`Failed to update tags for upload ${upload.id}:`, error);
      }
    }
    
    results.processed++;
  }
  
  return results;
}
```

### Tag Migration

Rename or merge existing tags:

```typescript
async function migrateTag(oldTag: string, newTag: string) {
  // Find all uploads with the old tag
  const uploads = await client.uploads.list({
    filter: {
      tags: { any_in: [oldTag] }
    },
    page: { limit: 500 }
  });
  
  const results = {
    found: uploads.length,
    updated: 0,
    failed: 0
  };
  
  for (const upload of uploads) {
    try {
      const currentTags = upload.tags || [];
      const newTags = currentTags.map(tag => tag === oldTag ? newTag : tag);
      
      // Remove duplicates if newTag already exists
      const uniqueTags = [...new Set(newTags)];
      
      await client.uploads.update(upload.id, {
        tags: uniqueTags
      });
      
      results.updated++;
    } catch (error) {
      results.failed++;
      console.error(`Failed to update upload ${upload.id}:`, error);
    }
  }
  
  return results;
}

// Merge multiple tags into one
async function mergeTags(tagsToMerge: string[], targetTag: string) {
  const uploads = await client.uploads.list({
    filter: {
      tags: { any_in: tagsToMerge }
    },
    page: { limit: 500 }
  });
  
  const results = {
    affected: uploads.length,
    updated: 0
  };
  
  for (const upload of uploads) {
    const currentTags = upload.tags || [];
    
    // Replace any of the merge tags with target tag
    const newTags = currentTags
      .map(tag => tagsToMerge.includes(tag) ? targetTag : tag)
      .filter((tag, index, self) => self.indexOf(tag) === index); // Remove duplicates
    
    if (JSON.stringify(currentTags.sort()) !== JSON.stringify(newTags.sort())) {
      try {
        await client.uploads.update(upload.id, {
          tags: newTags
        });
        results.updated++;
      } catch (error) {
        console.error(`Failed to update upload ${upload.id}:`, error);
      }
    }
  }
  
  return results;
}
```

## Tag Combination Strategies

### Multi-dimensional Tagging

Use multiple tag dimensions for flexible filtering:

```typescript
// Tag uploads with multiple dimensions
async function tagUploadMultiDimensional(uploadId: string, dimensions: {
  type: string;
  status: string;
  visibility: string;
  quality: string;
  usage: string[];
}) {
  const tags = [
    `type:${dimensions.type}`,
    `status:${dimensions.status}`,
    `visibility:${dimensions.visibility}`,
    `quality:${dimensions.quality}`,
    ...dimensions.usage.map(u => `usage:${u}`)
  ];
  
  await client.uploads.update(uploadId, { tags });
}

// Example usage
await tagUploadMultiDimensional('upload-id', {
  type: 'photo',
  status: 'approved',
  visibility: 'public',
  quality: 'high-res',
  usage: ['web', 'print', 'social']
});

// Find uploads matching multiple dimensions
const webReadyPhotos = await client.uploads.list({
  filter: {
    tags: {
      all_in: ['type:photo', 'status:approved', 'usage:web']
    }
  }
});
```

### Smart Tag Suggestions

Build tag suggestions based on upload properties:

```typescript
async function suggestTags(upload: any): string[] {
  const suggestions = [];
  
  // Based on file type
  if (upload.is_image) {
    suggestions.push('type:image');
    
    // Based on dimensions
    if (upload.width > 1920) suggestions.push('resolution:high');
    if (upload.width === upload.height) suggestions.push('aspect:square');
    if (upload.width > upload.height) suggestions.push('aspect:landscape');
    else suggestions.push('aspect:portrait');
  }
  
  // Based on file size
  if (upload.size > 5 * 1024 * 1024) suggestions.push('size:large');
  else if (upload.size < 100 * 1024) suggestions.push('size:small');
  else suggestions.push('size:medium');
  
  // Based on colors (if available)
  if (upload.colors && upload.colors.length > 0) {
    const dominantColor = upload.colors[0];
    // Simple color categorization
    suggestions.push(`color:${categorizeColor(dominantColor.hex)}`);
  }
  
  // Based on date
  const uploadDate = new Date(upload.created_at);
  suggestions.push(`year:${uploadDate.getFullYear()}`);
  suggestions.push(`month:${uploadDate.getFullYear()}-${String(uploadDate.getMonth() + 1).padStart(2, '0')}`);
  
  return suggestions;
}

function categorizeColor(hex: string): string {
  // Simple color categorization logic
  const r = parseInt(hex.slice(1, 3), 16);
  const g = parseInt(hex.slice(3, 5), 16);
  const b = parseInt(hex.slice(5, 7), 16);
  
  if (r > 200 && g < 100 && b < 100) return 'red';
  if (r < 100 && g > 200 && b < 100) return 'green';
  if (r < 100 && g < 100 && b > 200) return 'blue';
  if (r > 200 && g > 200 && b < 100) return 'yellow';
  if (r > 200 && g > 200 && b > 200) return 'white';
  if (r < 50 && g < 50 && b < 50) return 'black';
  return 'mixed';
}
```

### Campaign Tagging

Organize assets by marketing campaigns:

```typescript
async function tagCampaignAssets(campaignId: string, uploadIds: string[]) {
  const campaignTags = [
    `campaign:${campaignId}`,
    `campaign-year:${new Date().getFullYear()}`,
    'campaign-active'
  ];
  
  // Apply to uploads
  await client.uploads.bulkTag({
    uploads: uploadIds.map(id => ({ type: 'upload', id })),
    tags: campaignTags
  });
}

// End campaign
async function archiveCampaign(campaignId: string) {
  const uploads = await client.uploads.list({
    filter: {
      tags: { any_in: [`campaign:${campaignId}`] }
    }
  });
  
  // Remove active tag, add archived
  for (const upload of uploads) {
    const newTags = upload.tags
      .filter(t => t !== 'campaign-active')
      .concat('campaign-archived');
    
    await client.uploads.update(upload.id, { tags: newTags });
  }
}
```

## Integration with Smart Tags

DatoCMS also provides AI-powered automatic tagging:

```typescript
// Get all available smart tags in the project
const smartTags = await client.uploadSmartTags.list();

// Get smart tags for a specific upload
const upload = await client.uploads.find('upload-id');
const uploadSmartTags = upload.smart_tags || [];

// Smart tags include:
// - Object detection (person, car, building, etc.)
// - Scene detection (outdoor, indoor, nature, etc.)
// - Concept detection (happiness, business, technology, etc.)

// Combine manual and smart tags
const smartTagNames = uploadSmartTags.map(st => `ai:${st}`);

// Apply both manual and AI tags
await client.uploads.update('upload-id', {
  tags: [...upload.tags, ...smartTagNames]
});
```

**Related:** See [Upload Smart Tags](./upload-smart-tag.md) for AI-powered tagging

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

// Tags are normalized automatically
// Invalid characters are converted or removed
const upload = await client.uploads.update('upload-id', {
  tags: ['Invalid Tag Name!', 'another@tag'] 
  // Will be normalized to: ['invalid-tag-name', 'anothertag']
});

// Handle empty tags
try {
  await client.uploads.update('upload-id', {
    tags: ['', 'valid-tag'] // Empty string will be filtered out
  });
} catch (error) {
  console.error('Error updating tags:', error);
}

// Note: Tags cannot be directly created or deleted
// They are automatically managed based on upload references
```

## Upload Tag Object Structure

```typescript
// Upload tags are returned as an array of strings
type UploadTags = string[];

// When used in uploads
interface Upload {
  // ... other properties
  tags: string[]; // Array of tag names
  smart_tags: string[]; // Array of AI-generated tags (read-only)
}
```

## Best Practices

1. **Naming Convention**: Use lowercase, hyphenated tags (e.g., 'product-photo', not 'Product Photo')
2. **Consistency**: Establish and document tagging conventions for your team
3. **Avoid Duplication**: Use tag normalization to prevent variations of the same tag
4. **Meaningful Tags**: Use descriptive tags that add searchable value
5. **Limit Tag Count**: Avoid over-tagging; 3-7 tags per upload is usually sufficient
6. **Regular Cleanup**: Periodically review and consolidate similar tags
7. **Hierarchical Structure**: Use naming patterns to simulate tag hierarchies
8. **Bulk Operations**: Use bulk tagging for better performance
9. **Character Restrictions**: Use lowercase letters, numbers, and hyphens

## Limitations

1. **No Direct Management**: Tags cannot be created, renamed, or deleted directly via API
2. **Automatic Cleanup**: Tags are automatically removed when no uploads reference them
3. **Case Insensitive**: Tags are case-insensitive ('Product' and 'product' are the same)
4. **No Hierarchy**: Tags are flat; use prefixes for pseudo-hierarchy
5. **Normalization**: Tag names are automatically normalized

## Related Resources

- [Uploads](./upload.md) - Apply tags to uploads
- [Upload Collections](./upload-collection.md) - Hierarchical organization
- [Upload Smart Tags](./upload-smart-tag.md) - AI-powered tagging
- [Upload Filters](./upload-filter.md) - Create saved filters using tags