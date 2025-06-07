# Publishing Workflow Resources

## Overview

The Publishing Workflow resources provide powerful tools for managing content lifecycles, including scheduling publications, automatic unpublishing, and multi-stage approval workflows. These features enable teams to coordinate content releases, manage time-sensitive materials, and implement review processes.

### Available Resources

- **Scheduled Publication** - Automatically publish content at future dates
- **Scheduled Unpublishing** - Automatically remove content after specific dates
- **Workflows** - Multi-stage content approval processes

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Schedule a publication
await client.scheduledPublication.create('item-id', {
  publication_scheduled_at: '2024-12-25T00:00:00Z'
});

// Schedule unpublishing
await client.scheduledUnpublishing.create('item-id', {
  unpublishing_scheduled_at: '2024-12-31T23:59:59Z',
  content_in_locales: []
});

// Create workflow
await client.workflows.create({
  name: 'Editorial Review',
  api_key: 'editorial_review',
  stages: [
    { name: 'Draft', id: 'draft', color: { hex: '#9CA3AF' } },
    { name: 'Review', id: 'review', color: { hex: '#F59E0B' } },
    { name: 'Approved', id: 'approved', color: { hex: '#10B981' } }
  ]
});
```

---

# Scheduled Publication Resource

## Overview

Scheduled Publications allow you to automatically publish content at a specific future date and time. This is useful for embargo periods, coordinated launches, and time-sensitive content releases. The scheduled publication system ensures content goes live exactly when needed, supporting both full publication and selective locale publication.

## API Client Section

`client.scheduledPublication`

## API Reference

### create() - Schedule Item Publication

**Signature**: `create(itemId, body): Promise<ScheduledPublication>`

Schedules an item to be published at a specific future date and time.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID to schedule |
| `body` | object | Scheduling configuration |
| `body.publication_scheduled_at` | string | ISO 8601 date when to publish |
| `body.selective_publication` | object | Optional selective publication settings |
| `body.selective_publication.content_in_locales` | array | Locale codes to publish |
| `body.selective_publication.non_localized_content` | boolean | Include non-localized fields |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Schedule simple publication
const scheduled = await client.scheduledPublication.create('item_id', {
  publication_scheduled_at: '2024-12-25T00:00:00Z'
});

// Schedule with selective locale publication
const selectiveScheduled = await client.scheduledPublication.create('item_id', {
  publication_scheduled_at: '2024-06-01T14:00:00Z',
  selective_publication: {
    content_in_locales: ['en', 'es'], // Only publish English and Spanish
    non_localized_content: true // Also publish non-localized fields
  }
});
```

**Advanced Example (Time Zone Handling)**:
```typescript
// Helper to schedule in specific timezone
async function scheduleInTimezone(itemId, localTime, timezone) {
  // Convert local time to UTC
  const date = new Date(localTime);
  const utcDate = new Date(date.toLocaleString('en-US', { timeZone: 'UTC' }));
  
  return client.scheduledPublication.create(itemId, {
    publication_scheduled_at: utcDate.toISOString()
  });
}

// Schedule for 9 AM EST
await scheduleInTimezone(
  'item_id',
  '2024-06-01 09:00:00',
  'America/New_York'
);
```

**Response Structure**:
```typescript
{
  id: 'scheduled_publication_id',
  type: 'scheduled_publication',
  attributes: {
    publication_scheduled_at: '2024-12-25T00:00:00.000Z',
    selective_publication: {
      content_in_locales: ['en', 'es'],
      non_localized_content: true
    }
  },
  relationships: {
    item: {
      data: { type: 'item', id: 'item_id' }
    }
  }
}
```

**Error Handling**:
```typescript
try {
  const scheduled = await client.scheduledPublication.create('item_id', {
    publication_scheduled_at: '2020-01-01T00:00:00Z' // Past date
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Publication date must be in the future');
  }
}
```

### destroy() - Cancel Scheduled Publication

**Signature**: `destroy(itemId): Promise<Item>`

Cancels a scheduled publication, preventing the item from being automatically published.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID with scheduled publication |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Cancel scheduled publication
await client.scheduledPublication.destroy('item_id');
```

**Advanced Example (Cancel and Reschedule)**:
```typescript
// Cancel and reschedule pattern
async function reschedulePublication(itemId, newDate) {
  try {
    // Cancel existing schedule if any
    await client.scheduledPublication.destroy(itemId);
  } catch (error) {
    // Ignore if no existing schedule
  }
  
  // Create new schedule
  return client.scheduledPublication.create(itemId, {
    publication_scheduled_at: newDate.toISOString()
  });
}
```

**Response**: Returns the updated item object with cleared scheduling metadata.

**Error Handling**:
```typescript
try {
  await client.scheduledPublication.destroy('item_id');
  console.log('Scheduled publication cancelled');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('No scheduled publication found for this item');
  }
}
```

## Working with Scheduled Publications

### Scheduling Patterns

#### Content Calendar

Schedule content for specific dates and times:

```typescript
async function scheduleContentCalendar(calendar: Array<{
  itemId: string;
  publishDate: Date;
  title: string;
}>) {
  const scheduled = [];
  
  for (const entry of calendar) {
    try {
      const result = await client.scheduledPublication.create(entry.itemId, {
        publication_scheduled_at: entry.publishDate.toISOString()
      });
      
      scheduled.push({
        ...entry,
        scheduled: true,
        scheduledAt: result.publication_scheduled_at
      });
      
      console.log(`✓ Scheduled "${entry.title}" for ${entry.publishDate}`);
    } catch (error) {
      console.error(`✗ Failed to schedule "${entry.title}":`, error.message);
      scheduled.push({
        ...entry,
        scheduled: false,
        error: error.message
      });
    }
  }
  
  return scheduled;
}

// Schedule blog posts for the week
const weeklyPosts = [
  { itemId: 'post-1', publishDate: new Date('2024-06-03T09:00:00'), title: 'Monday Post' },
  { itemId: 'post-2', publishDate: new Date('2024-06-05T09:00:00'), title: 'Wednesday Post' },
  { itemId: 'post-3', publishDate: new Date('2024-06-07T09:00:00'), title: 'Friday Post' }
];

await scheduleContentCalendar(weeklyPosts);
```

#### Embargo Management

Hold content until a specific release time:

```typescript
async function embargoContent(itemId: string, embargoUntil: Date, reason?: string) {
  // Add metadata to item
  await client.items.update(itemId, {
    embargo_reason: reason || 'Embargoed content',
    embargo_until: embargoUntil.toISOString()
  });
  
  // Schedule publication for embargo end
  const scheduled = await client.scheduledPublication.create(itemId, {
    publication_scheduled_at: embargoUntil.toISOString()
  });
  
  // Create audit entry
  console.log(`Content embargoed until ${embargoUntil}`);
  
  return scheduled;
}

// Embargo press release until announcement
const announcementTime = new Date('2024-06-15T14:00:00Z');
await embargoContent('press-release-id', announcementTime, 'Product launch announcement');
```

#### Time Zone Coordination

Schedule across different time zones:

```typescript
class TimeZoneScheduler {
  /**
   * Schedule publication at a specific time in a given timezone
   */
  static async scheduleInTimezone(
    client: any,
    itemId: string,
    localTime: string, // "2024-06-01 09:00"
    timezone: string   // "America/New_York"
  ) {
    // Parse local time
    const [date, time] = localTime.split(' ');
    const [year, month, day] = date.split('-').map(Number);
    const [hour, minute] = time.split(':').map(Number);
    
    // Create date in specified timezone
    const localDate = new Date(year, month - 1, day, hour, minute);
    
    // Convert to UTC for API
    const utcString = localDate.toLocaleString('en-US', {
      timeZone: timezone,
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
      hour12: false
    });
    
    const utcDate = new Date(utcString + ' UTC');
    
    return client.scheduledPublication.create(itemId, {
      publication_scheduled_at: utcDate.toISOString()
    });
  }
  
  /**
   * Schedule same content for multiple timezones at local time
   */
  static async scheduleMultiTimezone(
    client: any,
    items: Array<{ itemId: string; timezone: string }>,
    localTime: string // Same local time for all
  ) {
    const results = [];
    
    for (const { itemId, timezone } of items) {
      try {
        const scheduled = await this.scheduleInTimezone(
          client,
          itemId,
          localTime,
          timezone
        );
        results.push({ itemId, timezone, success: true, scheduled });
      } catch (error) {
        results.push({ itemId, timezone, success: false, error });
      }
    }
    
    return results;
  }
}

// Schedule for 9 AM local time in different regions
await TimeZoneScheduler.scheduleMultiTimezone(client, [
  { itemId: 'us-announcement', timezone: 'America/New_York' },
  { itemId: 'uk-announcement', timezone: 'Europe/London' },
  { itemId: 'jp-announcement', timezone: 'Asia/Tokyo' }
], '2024-06-01 09:00');
```

### Selective Locale Publishing

Schedule publication of specific language versions:

```typescript
async function scheduleLocalizedContent(releases: Array<{
  itemId: string;
  locales: string[];
  releaseDate: Date;
  includeNonLocalized?: boolean;
}>) {
  const scheduled = [];
  
  for (const release of releases) {
    const result = await client.scheduledPublication.create(release.itemId, {
      publication_scheduled_at: release.releaseDate.toISOString(),
      selective_publication: {
        content_in_locales: release.locales,
        non_localized_content: release.includeNonLocalized ?? true
      }
    });
    
    scheduled.push({
      itemId: release.itemId,
      scheduledLocales: release.locales,
      scheduledAt: result.publication_scheduled_at
    });
  }
  
  return scheduled;
}

// Schedule phased international release
await scheduleLocalizedContent([
  {
    itemId: 'product-page',
    locales: ['en', 'en-US', 'en-GB'],
    releaseDate: new Date('2024-06-01T00:00:00Z')
  },
  {
    itemId: 'product-page',
    locales: ['es', 'es-MX', 'pt', 'pt-BR'],
    releaseDate: new Date('2024-06-08T00:00:00Z')
  },
  {
    itemId: 'product-page',
    locales: ['ja', 'ko', 'zh'],
    releaseDate: new Date('2024-06-15T00:00:00Z')
  }
]);
```

### Bulk Scheduling

Schedule multiple items efficiently:

```typescript
async function bulkScheduleItems(
  schedules: Array<{
    itemId: string;
    publishAt: Date;
    locales?: string[];
  }>
) {
  const results = {
    scheduled: [],
    failed: []
  };
  
  // Process in parallel with concurrency limit
  const batchSize = 5;
  for (let i = 0; i < schedules.length; i += batchSize) {
    const batch = schedules.slice(i, i + batchSize);
    
    const batchResults = await Promise.allSettled(
      batch.map(schedule => 
        client.scheduledPublication.create(schedule.itemId, {
          publication_scheduled_at: schedule.publishAt.toISOString(),
          selective_publication: schedule.locales ? {
            content_in_locales: schedule.locales,
            non_localized_content: true
          } : undefined
        })
      )
    );
    
    batchResults.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        results.scheduled.push({
          ...batch[index],
          scheduledPublication: result.value
        });
      } else {
        results.failed.push({
          ...batch[index],
          error: result.reason
        });
      }
    });
  }
  
  console.log(`Scheduled: ${results.scheduled.length}`);
  console.log(`Failed: ${results.failed.length}`);
  
  return results;
}
```

## Monitoring Scheduled Publications

### List Scheduled Items

Find all items with scheduled publications:

```typescript
async function getScheduledItems() {
  // Get all draft items with scheduled publication
  const items = await client.items.list({
    filter: {
      publication_scheduled_at: { exists: true }
    },
    order_by: ['publication_scheduled_at_ASC']
  });
  
  return items.map(item => ({
    id: item.id,
    title: item.title || item.name || 'Untitled',
    scheduledAt: item.meta.publication_scheduled_at,
    timeUntil: getTimeUntil(new Date(item.meta.publication_scheduled_at))
  }));
}

function getTimeUntil(date: Date): string {
  const now = new Date();
  const diff = date.getTime() - now.getTime();
  
  if (diff < 0) return 'Overdue';
  
  const hours = Math.floor(diff / (1000 * 60 * 60));
  const days = Math.floor(hours / 24);
  
  if (days > 0) return `${days} days`;
  if (hours > 0) return `${hours} hours`;
  return `${Math.floor(diff / (1000 * 60))} minutes`;
}
```

### Publication Calendar View

Create a calendar view of scheduled content:

```typescript
async function getPublicationCalendar(startDate: Date, endDate: Date) {
  const items = await client.items.list({
    filter: {
      publication_scheduled_at: {
        gte: startDate.toISOString(),
        lte: endDate.toISOString()
      }
    },
    order_by: ['publication_scheduled_at_ASC']
  });
  
  // Group by date
  const calendar = {};
  
  items.forEach(item => {
    const date = new Date(item.meta.publication_scheduled_at)
      .toISOString()
      .split('T')[0];
    
    if (!calendar[date]) {
      calendar[date] = [];
    }
    
    calendar[date].push({
      time: new Date(item.meta.publication_scheduled_at).toLocaleTimeString(),
      item: item.title || item.name,
      type: item.item_type,
      id: item.id
    });
  });
  
  return calendar;
}

// Get next week's schedule
const nextWeek = new Date();
nextWeek.setDate(nextWeek.getDate() + 7);

const calendar = await getPublicationCalendar(new Date(), nextWeek);
console.log('Publication Schedule:', calendar);
```

## Advanced Patterns

### Scheduled Publication Workflow

Implement approval before scheduling:

```typescript
class ScheduledPublicationWorkflow {
  static async requestScheduledPublication(
    client: any,
    itemId: string,
    requestedDate: Date,
    requestedBy: string
  ) {
    // Add to approval queue (custom implementation)
    const request = {
      itemId,
      requestedDate,
      requestedBy,
      status: 'pending',
      createdAt: new Date()
    };
    
    // Store request (e.g., in a separate model)
    // ... implementation specific
    
    return request;
  }
  
  static async approveAndSchedule(
    client: any,
    requestId: string,
    approvedBy: string
  ) {
    // Get request details
    const request = await this.getRequest(requestId);
    
    if (request.status !== 'pending') {
      throw new Error('Request already processed');
    }
    
    // Schedule the publication
    const scheduled = await client.scheduledPublication.create(
      request.itemId,
      {
        publication_scheduled_at: request.requestedDate.toISOString()
      }
    );
    
    // Update request status
    request.status = 'approved';
    request.approvedBy = approvedBy;
    request.approvedAt = new Date();
    
    return { request, scheduled };
  }
  
  static async getRequest(requestId: string) {
    // Implementation specific
    return {} as any;
  }
}
```

### Recurring Publications

Schedule content to publish on a recurring basis:

```typescript
async function scheduleRecurring(
  itemId: string,
  pattern: {
    startDate: Date;
    interval: 'daily' | 'weekly' | 'monthly';
    count: number;
    time: string; // "HH:mm"
  }
) {
  const schedules = [];
  const [hours, minutes] = pattern.time.split(':').map(Number);
  
  for (let i = 0; i < pattern.count; i++) {
    const publishDate = new Date(pattern.startDate);
    
    // Calculate next date based on interval
    switch (pattern.interval) {
      case 'daily':
        publishDate.setDate(publishDate.getDate() + i);
        break;
      case 'weekly':
        publishDate.setDate(publishDate.getDate() + (i * 7));
        break;
      case 'monthly':
        publishDate.setMonth(publishDate.getMonth() + i);
        break;
    }
    
    publishDate.setHours(hours, minutes, 0, 0);
    
    // Clone the item for each scheduled publication
    const clonedItem = await client.items.duplicate(itemId);
    
    // Schedule the clone
    const scheduled = await client.scheduledPublication.create(
      clonedItem.id,
      {
        publication_scheduled_at: publishDate.toISOString()
      }
    );
    
    schedules.push({
      itemId: clonedItem.id,
      publishDate,
      scheduled
    });
  }
  
  return schedules;
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.scheduledPublication.create('item-id', {
    publication_scheduled_at: '2020-01-01T00:00:00Z' // Past date
  });
} catch (error) {
  if (error instanceof ApiError) {
    if (error.response.status === 422) {
      const dateError = error.findError('publication_scheduled_at');
      if (dateError?.code === 'VALIDATION_DATETIME_IN_FUTURE') {
        console.log('Publication date must be in the future');
      }
    }
  }
}

// Handle cancellation of non-existent schedule
try {
  await client.scheduledPublication.destroy('item-without-schedule');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 404) {
    console.log('No scheduled publication found for this item');
  }
}

// Handle scheduling conflicts
try {
  // Try to schedule an already scheduled item
  await client.scheduledPublication.create('already-scheduled-item', {
    publication_scheduled_at: new Date().toISOString()
  });
} catch (error) {
  if (error instanceof ApiError && error.response.status === 422) {
    console.log('Item already has a scheduled publication');
    // Cancel existing and reschedule
    await client.scheduledPublication.destroy('already-scheduled-item');
    await client.scheduledPublication.create('already-scheduled-item', {
      publication_scheduled_at: new Date().toISOString()
    });
  }
}
```

## Scheduled Publication Object Structure

```typescript
interface ScheduledPublication {
  id: string;
  type: 'scheduled_publication';
  publication_scheduled_at: string; // ISO 8601 date
  selective_publication?: {
    content_in_locales: string[];
    non_localized_content: boolean;
  };
  item: {
    type: 'item';
    id: string;
  };
}

// Item metadata when scheduled
interface ItemMeta {
  publication_scheduled_at?: string;
  unpublishing_scheduled_at?: string;
  // ... other metadata
}
```

## Common Patterns

### Bulk Scheduling

```typescript
async function bulkSchedulePublications(schedules) {
  const results = {
    successful: [],
    failed: []
  };
  
  // Process in batches to avoid rate limits
  const batchSize = 10;
  
  for (let i = 0; i < schedules.length; i += batchSize) {
    const batch = schedules.slice(i, i + batchSize);
    
    const batchResults = await Promise.allSettled(
      batch.map(schedule => 
        client.scheduledPublication.create(schedule.itemId, {
          publication_scheduled_at: schedule.publishAt.toISOString(),
          selective_publication: schedule.locales ? {
            content_in_locales: schedule.locales,
            non_localized_content: true
          } : undefined
        })
      )
    );
    
    batchResults.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        results.successful.push({
          ...batch[index],
          scheduledPublication: result.value
        });
      } else {
        results.failed.push({
          ...batch[index],
          error: result.reason.message
        });
      }
    });
    
    // Add delay between batches
    if (i + batchSize < schedules.length) {
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
  
  return results;
}

// Usage
const schedules = [
  { itemId: 'post-1', publishAt: new Date('2024-06-01T09:00:00Z') },
  { itemId: 'post-2', publishAt: new Date('2024-06-02T09:00:00Z') },
  { itemId: 'post-3', publishAt: new Date('2024-06-03T09:00:00Z'), locales: ['en'] }
];

const bulkResults = await bulkSchedulePublications(schedules);
console.log(`Scheduled: ${bulkResults.successful.length}, Failed: ${bulkResults.failed.length}`);
```

### Content Calendar Management

```typescript
async function createContentCalendar(calendar) {
  const scheduled = [];
  
  for (const entry of calendar) {
    try {
      // Check if item is already scheduled
      const item = await client.items.find(entry.itemId);
      
      if (item.meta.publication_scheduled_at) {
        console.warn(`Item ${entry.itemId} already scheduled for ${item.meta.publication_scheduled_at}`);
        
        // Optionally reschedule
        if (entry.force) {
          await client.scheduledPublication.destroy(entry.itemId);
        } else {
          continue;
        }
      }
      
      // Schedule publication
      const result = await client.scheduledPublication.create(entry.itemId, {
        publication_scheduled_at: entry.publishDate.toISOString(),
        selective_publication: entry.locales ? {
          content_in_locales: entry.locales,
          non_localized_content: entry.includeNonLocalized ?? true
        } : undefined
      });
      
      scheduled.push({
        ...entry,
        scheduled: true,
        scheduledPublication: result
      });
      
      console.log(`✓ Scheduled "${entry.title}" for ${entry.publishDate}`);
    } catch (error) {
      console.error(`✗ Failed to schedule "${entry.title}":`, error.message);
      scheduled.push({
        ...entry,
        scheduled: false,
        error: error.message
      });
    }
  }
  
  return scheduled;
}

// Create weekly blog schedule
const weeklyBlogPosts = [
  {
    itemId: 'monday-post',
    title: 'Monday Motivation',
    publishDate: new Date('2024-06-03T09:00:00Z')
  },
  {
    itemId: 'wednesday-tips',
    title: 'Wednesday Tips',
    publishDate: new Date('2024-06-05T09:00:00Z')
  },
  {
    itemId: 'friday-roundup',
    title: 'Friday Roundup',
    publishDate: new Date('2024-06-07T09:00:00Z')
  }
];

const calendarResults = await createContentCalendar(weeklyBlogPosts);
```

### Monitoring Scheduled Publications

```typescript
async function getScheduledPublications(options = {}) {
  // Get all items with scheduled publications
  const items = await client.items.list({
    filter: {
      ...options.filter,
      'meta.publication_scheduled_at': { exists: true }
    },
    order_by: ['meta.publication_scheduled_at_ASC'],
    page: options.page || { limit: 100 }
  });
  
  const now = new Date();
  
  return items.map(item => ({
    id: item.id,
    title: item.title || item.name || `Item ${item.id}`,
    type: item.item_type,
    scheduledAt: item.meta.publication_scheduled_at,
    scheduledIn: getTimeUntil(new Date(item.meta.publication_scheduled_at)),
    isOverdue: new Date(item.meta.publication_scheduled_at) < now,
    locales: item.meta.publication_scheduled_at_selective_publication?.content_in_locales || 'all'
  }));
}

function getTimeUntil(date) {
  const now = new Date();
  const diff = date.getTime() - now.getTime();
  
  if (diff < 0) return 'Overdue';
  
  const days = Math.floor(diff / (1000 * 60 * 60 * 24));
  const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  
  if (days > 0) return `${days}d ${hours}h`;
  if (hours > 0) return `${hours}h ${minutes}m`;
  return `${minutes}m`;
}

// Get upcoming publications
const upcoming = await getScheduledPublications({
  filter: {
    'meta.publication_scheduled_at': {
      gte: new Date().toISOString(),
      lte: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString()
    }
  }
});

console.table(upcoming);
```

---

# Scheduled Unpublishing Resource

## Overview

Scheduled Unpublishing allows you to automatically unpublish content at a specific future date and time. This is useful for time-limited content like promotions, seasonal campaigns, event announcements, and temporary notices. The system supports both full unpublishing and selective locale unpublishing, giving you granular control over content availability.

## API Client Section

`client.scheduledUnpublishing`

## API Reference

### create() - Schedule Item Unpublishing

**Signature**: `create(itemId, body): Promise<ScheduledUnpublishing>`

Schedules a published item to be unpublished at a specific future date and time.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID to schedule for unpublishing |
| `body` | object | Unpublishing configuration |
| `body.unpublishing_scheduled_at` | string | ISO 8601 date when to unpublish |
| `body.content_in_locales` | array | Locale codes to unpublish (empty = all) |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Schedule unpublishing for all locales
const scheduled = await client.scheduledUnpublishing.create('item_id', {
  unpublishing_scheduled_at: '2024-12-31T23:59:59Z',
  content_in_locales: [] // Empty array means all locales
});

// Schedule unpublishing for specific locales only
const selectiveScheduled = await client.scheduledUnpublishing.create('item_id', {
  unpublishing_scheduled_at: '2024-07-01T00:00:00Z',
  content_in_locales: ['en', 'es'] // Only unpublish English and Spanish versions
});
```

**Advanced Example (Time-Limited Content)**:
```typescript
async function createTimeLimitedContent(options) {
  const {
    itemId,
    publishAt,
    unpublishAt,
    locales = [],
    title
  } = options;
  
  // Validate dates
  if (unpublishAt <= publishAt) {
    throw new Error('Unpublish date must be after publish date');
  }
  
  try {
    // Schedule publication first
    await client.scheduledPublication.create(itemId, {
      publication_scheduled_at: publishAt.toISOString()
    });
    
    // Then schedule unpublishing
    await client.scheduledUnpublishing.create(itemId, {
      unpublishing_scheduled_at: unpublishAt.toISOString(),
      content_in_locales: locales
    });
    
    const duration = unpublishAt.getTime() - publishAt.getTime();
    const durationHours = Math.floor(duration / (1000 * 60 * 60));
    
    console.log(`✓ "${title}" scheduled:`);
    console.log(`  - Publish: ${publishAt.toISOString()}`);
    console.log(`  - Unpublish: ${unpublishAt.toISOString()}`);
    console.log(`  - Duration: ${durationHours} hours`);
    
    return {
      itemId,
      publishAt,
      unpublishAt,
      duration: durationHours,
      locales: locales.length > 0 ? locales : 'all'
    };
  } catch (error) {
    console.error(`Failed to schedule "${title}":`, error.message);
    throw error;
  }
}

// Create a 48-hour flash sale
await createTimeLimitedContent({
  itemId: 'flash-sale-banner',
  title: 'Weekend Flash Sale',
  publishAt: new Date('2024-06-14T00:00:00Z'),
  unpublishAt: new Date('2024-06-16T00:00:00Z')
});
```

**Response Structure**:
```typescript
{
  id: 'scheduled_unpublishing_id',
  type: 'scheduled_unpublishing',
  attributes: {
    unpublishing_scheduled_at: '2024-12-31T23:59:59.000Z',
    content_in_locales: [] // or ['en', 'es']
  },
  relationships: {
    item: {
      data: { type: 'item', id: 'item_id' }
    }
  }
}
```

**Error Handling**:
```typescript
try {
  const scheduled = await client.scheduledUnpublishing.create('item_id', {
    unpublishing_scheduled_at: '2020-01-01T00:00:00Z', // Past date
    content_in_locales: []
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Unpublishing date must be in the future');
  }
}
```

### destroy() - Cancel Scheduled Unpublishing

**Signature**: `destroy(itemId): Promise<Item>`

Cancels a scheduled unpublishing, ensuring the item remains published past the originally scheduled time.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `itemId` | string | The item ID with scheduled unpublishing |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Cancel scheduled unpublishing
await client.scheduledUnpublishing.destroy('item_id');
```

**Advanced Example (Cancel and Reschedule)**:
```typescript
// Cancel and reschedule pattern
async function rescheduleUnpublishing(itemId, newDate, locales = []) {
  try {
    // Cancel existing schedule if any
    await client.scheduledUnpublishing.destroy(itemId);
  } catch (error) {
    // Ignore if no existing schedule
  }
  
  // Create new schedule
  return client.scheduledUnpublishing.create(itemId, {
    unpublishing_scheduled_at: newDate.toISOString(),
    content_in_locales: locales
  });
}
```

**Response**: Returns the updated item object with cleared unpublishing metadata.

**Error Handling**:
```typescript
try {
  await client.scheduledUnpublishing.destroy('item_id');
  console.log('Scheduled unpublishing cancelled');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('No scheduled unpublishing found for this item');
  }
}
```

## Common Use Cases

### Time-Limited Promotions

Schedule promotional content to automatically expire:

```typescript
async function createTimeLimitedPromotion(
  promoItemId: string,
  startDate: Date,
  endDate: Date
) {
  // First, schedule publication
  await client.scheduledPublication.create(promoItemId, {
    publication_scheduled_at: startDate.toISOString()
  });
  
  // Then, schedule unpublishing
  await client.scheduledUnpublishing.create(promoItemId, {
    unpublishing_scheduled_at: endDate.toISOString()
  });
  
  console.log(`Promotion scheduled from ${startDate} to ${endDate}`);
  
  return {
    itemId: promoItemId,
    activeFrom: startDate,
    activeUntil: endDate
  };
}

// Black Friday Sale - Nov 24-27, 2024
const blackFriday = await createTimeLimitedPromotion(
  'black-friday-banner-id',
  new Date('2024-11-24T00:00:00Z'),
  new Date('2024-11-27T23:59:59Z')
);
```

### Event Management

Automatically remove event content after the event ends:

```typescript
async function scheduleEventContent(event: {
  itemId: string;
  eventDate: Date;
  removeAfterDays?: number;
}) {
  // Calculate removal date
  const removalDate = new Date(event.eventDate);
  removalDate.setDate(removalDate.getDate() + (event.removeAfterDays || 1));
  
  // Schedule unpublishing
  await client.scheduledUnpublishing.create(event.itemId, {
    unpublishing_scheduled_at: removalDate.toISOString()
  });
  
  return {
    eventDate: event.eventDate,
    contentExpiresAt: removalDate
  };
}

// Conference announcement - remove 1 day after event
await scheduleEventContent({
  itemId: 'conference-2024-id',
  eventDate: new Date('2024-09-15T18:00:00Z'),
  removeAfterDays: 1
});
```

### Seasonal Content

Manage seasonal campaigns and decorations:

```typescript
class SeasonalContentManager {
  static async scheduleSeasonalContent(seasons: Array<{
    itemId: string;
    seasonName: string;
    publishDate: Date;
    unpublishDate: Date;
    locales?: string[];
  }>) {
    const scheduled = [];
    
    for (const season of seasons) {
      try {
        // Schedule publication
        await client.scheduledPublication.create(season.itemId, {
          publication_scheduled_at: season.publishDate.toISOString()
        });
        
        // Schedule unpublishing
        await client.scheduledUnpublishing.create(season.itemId, {
          unpublishing_scheduled_at: season.unpublishDate.toISOString(),
          content_in_locales: season.locales || []
        });
        
        scheduled.push({
          ...season,
          status: 'scheduled'
        });
        
        console.log(`✓ ${season.seasonName} content scheduled`);
      } catch (error) {
        scheduled.push({
          ...season,
          status: 'failed',
          error: error.message
        });
      }
    }
    
    return scheduled;
  }
  
  static getSeasonDates(year: number) {
    return {
      valentines: {
        publish: new Date(`${year}-02-01T00:00:00Z`),
        unpublish: new Date(`${year}-02-15T00:00:00Z`)
      },
      easter: {
        publish: new Date(`${year}-03-15T00:00:00Z`),
        unpublish: new Date(`${year}-04-30T00:00:00Z`)
      },
      summer: {
        publish: new Date(`${year}-06-01T00:00:00Z`),
        unpublish: new Date(`${year}-09-01T00:00:00Z`)
      },
      halloween: {
        publish: new Date(`${year}-10-01T00:00:00Z`),
        unpublish: new Date(`${year}-11-01T00:00:00Z`)
      },
      christmas: {
        publish: new Date(`${year}-11-15T00:00:00Z`),
        unpublish: new Date(`${year}-12-27T00:00:00Z`)
      }
    };
  }
}

// Schedule all seasonal content for 2024
const seasons2024 = SeasonalContentManager.getSeasonDates(2024);
await SeasonalContentManager.scheduleSeasonalContent([
  {
    itemId: 'valentines-banner-id',
    seasonName: 'Valentine\'s Day',
    publishDate: seasons2024.valentines.publish,
    unpublishDate: seasons2024.valentines.unpublish
  },
  {
    itemId: 'christmas-theme-id',
    seasonName: 'Christmas',
    publishDate: seasons2024.christmas.publish,
    unpublishDate: seasons2024.christmas.unpublish
  }
]);
```

### Temporary Notices

Display important notices for a limited time:

```typescript
async function createTemporaryNotice(notice: {
  title: string;
  content: string;
  severity: 'info' | 'warning' | 'critical';
  displayHours: number;
  locales?: string[];
}) {
  // Create the notice item
  const item = await client.items.create({
    item_type: { type: 'item_type', id: 'site_notice' },
    title: notice.title,
    content: notice.content,
    severity: notice.severity
  });
  
  // Publish immediately
  await client.items.publish(item.id);
  
  // Schedule unpublishing
  const unpublishDate = new Date();
  unpublishDate.setHours(unpublishDate.getHours() + notice.displayHours);
  
  await client.scheduledUnpublishing.create(item.id, {
    unpublishing_scheduled_at: unpublishDate.toISOString(),
    content_in_locales: notice.locales || []
  });
  
  return {
    noticeId: item.id,
    displayUntil: unpublishDate,
    autoRemoval: true
  };
}

// Maintenance notice - display for 4 hours
await createTemporaryNotice({
  title: 'Scheduled Maintenance',
  content: 'The site will be under maintenance from 2 AM to 4 AM EST.',
  severity: 'warning',
  displayHours: 4
});
```

## Locale-Specific Unpublishing

### Phased Content Retirement

Remove content from different markets at different times:

```typescript
async function phaseOutContent(itemId: string, schedule: Array<{
  locales: string[];
  unpublishDate: Date;
  reason?: string;
}>) {
  // Sort by date to handle earliest first
  const sorted = schedule.sort((a, b) => 
    a.unpublishDate.getTime() - b.unpublishDate.getTime()
  );
  
  // Schedule first phase
  const firstPhase = sorted[0];
  await client.scheduledUnpublishing.create(itemId, {
    unpublishing_scheduled_at: firstPhase.unpublishDate.toISOString(),
    content_in_locales: firstPhase.locales
  });
  
  console.log(`Phase 1: Unpublishing ${firstPhase.locales.join(', ')} on ${firstPhase.unpublishDate}`);
  
  // Note: Currently, only one scheduled unpublishing can exist per item
  // For multiple phases, you'd need to chain them or use a job system
  
  return {
    itemId,
    phases: sorted,
    warning: 'Only first phase scheduled - implement chaining for multiple phases'
  };
}

// Example: Phase out product from different markets
await phaseOutContent('product-id', [
  {
    locales: ['en-GB', 'en-IE'],
    unpublishDate: new Date('2024-06-30T23:59:59Z'),
    reason: 'UK/Ireland discontinuation'
  },
  {
    locales: ['en-US', 'en-CA'],
    unpublishDate: new Date('2024-07-31T23:59:59Z'),
    reason: 'North America discontinuation'
  },
  {
    locales: ['en'],
    unpublishDate: new Date('2024-08-31T23:59:59Z'),
    reason: 'Global discontinuation'
  }
]);
```

## Monitoring Scheduled Unpublishing

### List Items with Scheduled Unpublishing

Find all items scheduled to be unpublished:

```typescript
async function getScheduledUnpublishings() {
  // Get all published items with scheduled unpublishing
  const items = await client.items.list({
    filter: {
      unpublishing_scheduled_at: { exists: true },
      publication_status: { eq: 'published' }
    },
    order_by: ['unpublishing_scheduled_at_ASC']
  });
  
  const now = new Date();
  
  return items.map(item => ({
    id: item.id,
    title: item.title || item.name || 'Untitled',
    scheduledAt: item.meta.unpublishing_scheduled_at,
    timeRemaining: getTimeRemaining(new Date(item.meta.unpublishing_scheduled_at)),
    isOverdue: new Date(item.meta.unpublishing_scheduled_at) < now
  }));
}

function getTimeRemaining(unpublishDate: Date): string {
  const now = new Date();
  const diff = unpublishDate.getTime() - now.getTime();
  
  if (diff < 0) return 'Overdue';
  
  const days = Math.floor(diff / (1000 * 60 * 60 * 24));
  const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  
  if (days > 0) return `${days}d ${hours}h remaining`;
  if (hours > 0) return `${hours}h remaining`;
  
  const minutes = Math.floor(diff / (1000 * 60));
  return `${minutes}m remaining`;
}
```

### Expiration Dashboard

Create a dashboard for content expiration:

```typescript
async function getExpirationDashboard() {
  const items = await client.items.list({
    filter: {
      unpublishing_scheduled_at: { exists: true }
    }
  });
  
  const now = new Date();
  const in24Hours = new Date(now.getTime() + 24 * 60 * 60 * 1000);
  const in7Days = new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000);
  
  const categorized = {
    expired: [],
    within24Hours: [],
    within7Days: [],
    later: []
  };
  
  items.forEach(item => {
    const unpublishDate = new Date(item.meta.unpublishing_scheduled_at);
    
    if (unpublishDate < now) {
      categorized.expired.push(item);
    } else if (unpublishDate < in24Hours) {
      categorized.within24Hours.push(item);
    } else if (unpublishDate < in7Days) {
      categorized.within7Days.push(item);
    } else {
      categorized.later.push(item);
    }
  });
  
  return {
    summary: {
      total: items.length,
      expired: categorized.expired.length,
      expiringSoon: categorized.within24Hours.length,
      expiringThisWeek: categorized.within7Days.length
    },
    items: categorized
  };
}
```

## Combined Publishing Lifecycle

### Full Content Lifecycle Management

Manage the complete lifecycle of time-bound content:

```typescript
class ContentLifecycleManager {
  static async scheduleFullLifecycle(content: {
    itemId: string;
    publishAt: Date;
    unpublishAt: Date;
    locales?: string[];
    notifications?: {
      beforeUnpublish?: number; // hours
      webhook?: string;
    };
  }) {
    // Validate dates
    if (content.unpublishAt <= content.publishAt) {
      throw new Error('Unpublish date must be after publish date');
    }
    
    // Schedule publication
    const publication = await client.scheduledPublication.create(
      content.itemId,
      {
        publication_scheduled_at: content.publishAt.toISOString()
      }
    );
    
    // Schedule unpublishing
    const unpublishing = await client.scheduledUnpublishing.create(
      content.itemId,
      {
        unpublishing_scheduled_at: content.unpublishAt.toISOString(),
        content_in_locales: content.locales || []
      }
    );
    
    // Set up notification if requested
    if (content.notifications?.beforeUnpublish) {
      const notifyAt = new Date(content.unpublishAt);
      notifyAt.setHours(
        notifyAt.getHours() - content.notifications.beforeUnpublish
      );
      
      // Schedule notification (implementation specific)
      console.log(`Notification scheduled for ${notifyAt}`);
    }
    
    return {
      itemId: content.itemId,
      lifecycle: {
        publishAt: content.publishAt,
        unpublishAt: content.unpublishAt,
        duration: content.unpublishAt.getTime() - content.publishAt.getTime()
      },
      scheduled: {
        publication,
        unpublishing
      }
    };
  }
}

// Schedule a 48-hour flash sale
await ContentLifecycleManager.scheduleFullLifecycle({
  itemId: 'flash-sale-banner',
  publishAt: new Date('2024-06-15T00:00:00Z'),
  unpublishAt: new Date('2024-06-17T00:00:00Z'),
  notifications: {
    beforeUnpublish: 2, // Notify 2 hours before
    webhook: 'https://api.example.com/content-expiring'
  }
});
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.scheduledUnpublishing.create('item-id', {
    unpublishing_scheduled_at: '2020-01-01T00:00:00Z', // Past date
    content_in_locales: []
  });
} catch (error) {
  if (error instanceof ApiError) {
    const dateError = error.findError('unpublishing_scheduled_at');
    if (dateError?.code === 'VALIDATION_DATETIME_IN_FUTURE') {
      console.log('Unpublishing date must be in the future');
    }
  }
}

// Handle item not published
try {
  await client.scheduledUnpublishing.create('draft-item-id', {
    unpublishing_scheduled_at: new Date().toISOString(),
    content_in_locales: []
  });
} catch (error) {
  if (error instanceof ApiError && error.response.status === 422) {
    console.log('Cannot schedule unpublishing for unpublished item');
  }
}

// Handle existing schedule conflict
try {
  await client.scheduledUnpublishing.create('already-scheduled', {
    unpublishing_scheduled_at: new Date().toISOString(),
    content_in_locales: []
  });
} catch (error) {
  if (error instanceof ApiError) {
    // Cancel existing and create new
    await client.scheduledUnpublishing.destroy('already-scheduled');
    await client.scheduledUnpublishing.create('already-scheduled', {
      unpublishing_scheduled_at: new Date().toISOString(),
      content_in_locales: []
    });
  }
}
```

## Scheduled Unpublishing Object Structure

```typescript
interface ScheduledUnpublishing {
  id: string;
  type: 'scheduled_unpublishing';
  unpublishing_scheduled_at: string; // ISO 8601 date
  content_in_locales: string[]; // Empty array = all locales
  item: {
    type: 'item';
    id: string;
  };
}

// Item metadata when scheduled
interface ItemMeta {
  publication_scheduled_at?: string;
  unpublishing_scheduled_at?: string;
  published_at?: string;
  // ... other metadata
}
```

## Common Patterns

### Promotional Campaign Management

```typescript
async function scheduleCampaign(campaign) {
  const results = {
    scheduled: [],
    failed: []
  };
  
  for (const item of campaign.items) {
    try {
      // Check if item is published or will be published
      const itemData = await client.items.find(item.id);
      
      if (!itemData.meta.published_at && !itemData.meta.publication_scheduled_at) {
        // Schedule publication if needed
        await client.scheduledPublication.create(item.id, {
          publication_scheduled_at: campaign.startDate.toISOString()
        });
      }
      
      // Schedule unpublishing
      const scheduled = await client.scheduledUnpublishing.create(item.id, {
        unpublishing_scheduled_at: campaign.endDate.toISOString(),
        content_in_locales: item.locales || []
      });
      
      results.scheduled.push({
        itemId: item.id,
        name: item.name,
        scheduled
      });
    } catch (error) {
      results.failed.push({
        itemId: item.id,
        name: item.name,
        error: error.message
      });
    }
  }
  
  return results;
}

// Schedule Black Friday campaign
const blackFridayCampaign = {
  name: 'Black Friday 2024',
  startDate: new Date('2024-11-24T00:00:00Z'),
  endDate: new Date('2024-11-27T23:59:59Z'),
  items: [
    { id: 'bf-hero-banner', name: 'Hero Banner' },
    { id: 'bf-popup', name: 'Promotional Popup' },
    { id: 'bf-countdown', name: 'Countdown Timer' },
    { id: 'bf-products', name: 'Special Offers Page', locales: ['en', 'es'] }
  ]
};

const campaignResults = await scheduleCampaign(blackFridayCampaign);
console.log(`Campaign scheduled: ${campaignResults.scheduled.length} items`);
```

### Seasonal Content Management

```typescript
class SeasonalContentScheduler {
  static async scheduleSeasonalContent(season) {
    const scheduled = [];
    
    for (const content of season.contents) {
      try {
        // Schedule unpublishing
        const result = await client.scheduledUnpublishing.create(
          content.itemId,
          {
            unpublishing_scheduled_at: season.endDate.toISOString(),
            content_in_locales: content.locales || []
          }
        );
        
        scheduled.push({
          ...content,
          scheduledUnpublishing: result,
          status: 'scheduled'
        });
        
        console.log(`✓ Scheduled removal of "${content.name}"`);
      } catch (error) {
        scheduled.push({
          ...content,
          status: 'failed',
          error: error.message
        });
      }
    }
    
    return {
      season: season.name,
      endDate: season.endDate,
      scheduled
    };
  }
  
  static getSeasonEndDates(year) {
    return {
      valentines: new Date(`${year}-02-15T00:00:00Z`),
      easter: new Date(`${year}-04-30T00:00:00Z`),
      summer: new Date(`${year}-09-01T00:00:00Z`),
      halloween: new Date(`${year}-11-01T00:00:00Z`),
      christmas: new Date(`${year}-12-27T00:00:00Z`),
      newYear: new Date(`${year}-01-08T00:00:00Z`)
    };
  }
}

// Schedule Christmas content removal
const endDates = SeasonalContentScheduler.getSeasonEndDates(2024);

await SeasonalContentScheduler.scheduleSeasonalContent({
  name: 'Christmas 2024',
  endDate: endDates.christmas,
  contents: [
    { itemId: 'xmas-banner', name: 'Christmas Banner' },
    { itemId: 'xmas-theme', name: 'Holiday Theme' },
    { itemId: 'xmas-products', name: 'Gift Guide' },
    { itemId: 'xmas-shipping', name: 'Holiday Shipping Info' }
  ]
});
```

### Monitoring Scheduled Unpublishing

```typescript
async function getScheduledUnpublishings(options = {}) {
  // Get all items with scheduled unpublishing
  const items = await client.items.list({
    filter: {
      ...options.filter,
      'meta.unpublishing_scheduled_at': { exists: true },
      'meta.published_at': { exists: true } // Only published items
    },
    order_by: ['meta.unpublishing_scheduled_at_ASC'],
    page: options.page || { limit: 100 }
  });
  
  const now = new Date();
  
  return items.map(item => ({
    id: item.id,
    title: item.title || item.name || `Item ${item.id}`,
    type: item.item_type,
    scheduledAt: item.meta.unpublishing_scheduled_at,
    expiresIn: getTimeRemaining(new Date(item.meta.unpublishing_scheduled_at)),
    isOverdue: new Date(item.meta.unpublishing_scheduled_at) < now,
    locales: 'all' // Note: locale info not available in meta
  }));
}

function getTimeRemaining(unpublishDate) {
  const now = new Date();
  const diff = unpublishDate.getTime() - now.getTime();
  
  if (diff < 0) return 'Overdue';
  
  const days = Math.floor(diff / (1000 * 60 * 60 * 24));
  const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
  
  if (days > 0) return `${days}d ${hours}h`;
  if (hours > 0) return `${hours}h ${minutes}m`;
  return `${minutes}m`;
}

// Get items expiring soon
const expiringSoon = await getScheduledUnpublishings({
  filter: {
    'meta.unpublishing_scheduled_at': {
      gte: new Date().toISOString(),
      lte: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString() // Next 24 hours
    }
  }
});

console.log('Content expiring in next 24 hours:');
console.table(expiringSoon);
```

---

# Workflow Resource

## Overview

Workflows enable content approval processes by defining stages that content must pass through before publication. Teams can use workflows to implement review processes, quality checks, and multi-step approval chains. Each workflow consists of multiple stages that items move through, providing visibility and control over the content lifecycle.

## API Client Section

`client.workflows`

## API Reference

### list() - Retrieve All Workflows

**Signature**: `list(): Promise<Workflow[]>`

Retrieves all workflows in the project.

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get all workflows
const workflows = await client.workflows.list();

// Find workflow with most stages
const complexWorkflow = workflows.reduce((prev, current) => 
  current.stages.length > prev.stages.length ? current : prev
);
```

**Advanced Example (Raw Response)**:
```typescript
// Get raw JSON:API response
const rawResponse = await client.workflows.rawList();
// Returns JSON:API formatted response
```

### find() - Get Single Workflow

**Signature**: `find(workflowId): Promise<Workflow>`

Retrieves a specific workflow by ID.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `workflowId` | string | The workflow ID or api_key |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get workflow by ID
const workflow = await client.workflows.find('workflow_id');

// Get workflow by API key
const workflowByKey = await client.workflows.find('editorial_review');
```

**Response Structure**:
```typescript
{
  id: 'workflow_id',
  type: 'workflow',
  attributes: {
    name: 'Editorial Review',
    api_key: 'editorial_review',
    stages: [
      {
        id: 'draft',
        name: 'Draft',
        color: { hex: '#9CA3AF' },
        description: 'Initial content creation'
      },
      {
        id: 'review',
        name: 'Review',
        color: { hex: '#F59E0B' },
        description: 'Under editorial review'
      },
      {
        id: 'approved',
        name: 'Approved',
        color: { hex: '#10B981' },
        description: 'Ready to publish'
      }
    ]
  },
  meta: {
    created_at: '2024-01-01T00:00:00Z',
    updated_at: '2024-01-01T00:00:00Z'
  }
}
```

**Error Handling**:
```typescript
try {
  const workflow = await client.workflows.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Workflow not found');
  }
}
```

### create() - Create New Workflow

**Signature**: `create(body): Promise<Workflow>`

Creates a new workflow with multiple stages.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `body` | object | Workflow configuration |
| `body.name` | string | Human-readable workflow name |
| `body.api_key` | string | API identifier (letters, numbers, underscores) |
| `body.stages` | array | Workflow stages in order |
| `body.stages[].name` | string | Stage display name |
| `body.stages[].id` | string | Stage identifier |
| `body.stages[].color` | object | Stage color with `hex` property |
| `body.stages[].description` | string | Optional stage description |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create simple workflow
const simpleWorkflow = await client.workflows.create({
  name: 'Simple Approval',
  api_key: 'simple_approval',
  stages: [
    {
      id: 'pending',
      name: 'Pending Approval',
      color: { hex: '#F59E0B' }
    },
    {
      id: 'approved',
      name: 'Approved',
      color: { hex: '#10B981' }
    }
  ]
});
```

**Advanced Example (Complex Workflow)**:
```typescript
// Create complex workflow with descriptions
const complexWorkflow = await client.workflows.create({
  name: 'Content Production Pipeline',
  api_key: 'content_pipeline',
  stages: [
    {
      id: 'writing',
      name: 'Writing',
      color: { hex: '#6B7280' },
      description: 'Content is being written'
    },
    {
      id: 'editorial_review',
      name: 'Editorial Review',
      color: { hex: '#F59E0B' },
      description: 'Editor reviewing for style and accuracy'
    },
    {
      id: 'legal_review',
      name: 'Legal Review',
      color: { hex: '#DC2626' },
      description: 'Legal team approval required'
    },
    {
      id: 'ready',
      name: 'Ready to Publish',
      color: { hex: '#10B981' },
      description: 'Approved for publication'
    }
  ]
});
```

**Error Handling**:
```typescript
try {
  const workflow = await client.workflows.create({
    name: 'Test Workflow',
    api_key: 'test_workflow', // May already exist
    stages: [] // Must have at least one stage
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation errors:', error.response.data.errors);
    // Errors might include:
    // - api_key must be unique
    // - stages must have at least one element
    // - stage IDs must be unique within workflow
  }
}
```

### update() - Update Workflow

**Signature**: `update(workflowId, body): Promise<Workflow>`

Modifies an existing workflow's configuration or stages.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `workflowId` | string | The workflow ID |
| `body` | object | Update parameters (same as create) |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Update workflow name
const updated = await client.workflows.update('workflow_id', {
  name: 'Updated Editorial Process'
});
```

**Advanced Example (Add New Stage)**:
```typescript
// Add new stage to workflow
const workflow = await client.workflows.find('workflow_id');
const updatedWithNewStage = await client.workflows.update('workflow_id', {
  stages: [
    ...workflow.stages,
    {
      id: 'fact_check',
      name: 'Fact Check',
      color: { hex: '#8B5CF6' },
      description: 'Verify all facts and sources'
    }
  ]
});

// Reorder stages
const reorderedWorkflow = await client.workflows.update('workflow_id', {
  stages: [
    { id: 'draft', name: 'Draft', color: { hex: '#9CA3AF' } },
    { id: 'fact_check', name: 'Fact Check', color: { hex: '#8B5CF6' } },
    { id: 'review', name: 'Review', color: { hex: '#F59E0B' } },
    { id: 'approved', name: 'Approved', color: { hex: '#10B981' } }
  ]
});
```

**Note**: Changing stages may affect items currently in the workflow.

### destroy() - Delete Workflow

**Signature**: `destroy(workflowId): Promise<void>`

Removes a workflow. Models using this workflow will have it unassigned.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `workflowId` | string | The workflow ID |

**Basic Example**:
```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Delete workflow
await client.workflows.destroy('workflow_id');
```

**Advanced Example (Safe Delete)**:
```typescript
// Safe delete with usage check
async function safeDeleteWorkflow(workflowId) {
  try {
    // Check if workflow is assigned to any models
    const itemTypes = await client.itemTypes.list();
    const modelsUsingWorkflow = itemTypes.filter(
      model => model.workflow?.id === workflowId
    );
    
    if (modelsUsingWorkflow.length > 0) {
      console.warn(`Workflow is used by ${modelsUsingWorkflow.length} models:`);
      modelsUsingWorkflow.forEach(model => 
        console.warn(`- ${model.name} (${model.api_key})`)
      );
      
      // Optionally unassign workflow from models
      for (const model of modelsUsingWorkflow) {
        await client.itemTypes.update(model.id, {
          workflow: null
        });
      }
    }
    
    // Now safe to delete
    await client.workflows.destroy(workflowId);
    return true;
  } catch (error) {
    console.error('Failed to delete workflow:', error);
    return false;
  }
}
```

**Warning**: Items in workflow stages will lose their stage assignment.

## Working with Workflow Stages

### Assign Workflow to Models

Workflows must be assigned to item types (models) to be used:

```typescript
// Create workflow
const workflow = await client.workflows.create({
  name: 'Blog Editorial',
  api_key: 'blog_editorial',
  stages: [
    { name: 'Draft', id: 'draft', color: { hex: '#9CA3AF' } },
    { name: 'Review', id: 'review', color: { hex: '#F59E0B' } },
    { name: 'Published', id: 'published', color: { hex: '#10B981' } }
  ]
});

// Assign to blog post model
await client.itemTypes.update('blog_post', {
  workflow: {
    type: 'workflow',
    id: workflow.id
  }
});
```

### Move Items Through Stages

Items in a workflow can be moved between stages:

```typescript
// Create item in first stage
const item = await client.items.create({
  item_type: { type: 'item_type', id: 'blog_post' },
  title: 'New Article',
  meta: {
    stage: 'draft' // Initial stage
  }
});

// Move to review stage
await client.items.update(item.id, {
  meta: {
    stage: 'review'
  }
});

// Move to final stage
await client.items.update(item.id, {
  meta: {
    stage: 'published'
  }
});
```

### Bulk Stage Transitions

Move multiple items to a stage at once:

```typescript
const job = await client.items.bulkMoveToStage({
  items: [
    { type: 'item', id: 'item-1' },
    { type: 'item', id: 'item-2' },
    { type: 'item', id: 'item-3' }
  ],
  stage: {
    type: 'workflow_stage',
    id: 'review' // Target stage ID
  }
});

// Wait for completion
await client.jobResults.wait(job.id);
```

### Query Items by Stage

Filter items based on their workflow stage:

```typescript
// Get all items in review stage
const itemsInReview = await client.items.list({
  filter: {
    type: 'blog_post',
    workflow_stage: { eq: 'review' }
  }
});

// Count items per stage
const stages = ['draft', 'review', 'published'];
const stageCounts = await Promise.all(
  stages.map(async (stage) => {
    const items = await client.items.list({
      filter: {
        type: 'blog_post',
        workflow_stage: { eq: stage }
      },
      page: { limit: 0 } // Just get count
    });
    return {
      stage,
      count: items.meta.total_count
    };
  })
);
```

## Workflow Patterns

### Simple Approval Workflow

Basic two-stage approval process:

```typescript
const simpleWorkflow = await client.workflows.create({
  name: 'Simple Approval',
  api_key: 'simple_approval',
  stages: [
    {
      name: 'Pending Approval',
      id: 'pending',
      color: { hex: '#F59E0B' }
    },
    {
      name: 'Approved',
      id: 'approved',
      color: { hex: '#10B981' }
    }
  ]
});
```

### Multi-Department Workflow

Complex workflow with multiple review stages:

```typescript
const complexWorkflow = await client.workflows.create({
  name: 'Enterprise Content Pipeline',
  api_key: 'enterprise_pipeline',
  stages: [
    {
      name: 'Draft',
      id: 'draft',
      color: { hex: '#6B7280' },
      description: 'Initial content creation'
    },
    {
      name: 'Content Review',
      id: 'content_review',
      color: { hex: '#3B82F6' },
      description: 'Editorial team review'
    },
    {
      name: 'SEO Review',
      id: 'seo_review',
      color: { hex: '#8B5CF6' },
      description: 'SEO optimization check'
    },
    {
      name: 'Legal Review',
      id: 'legal_review',
      color: { hex: '#DC2626' },
      description: 'Legal compliance check'
    },
    {
      name: 'Final Approval',
      id: 'final_approval',
      color: { hex: '#F59E0B' },
      description: 'Management sign-off'
    },
    {
      name: 'Ready',
      id: 'ready',
      color: { hex: '#10B981' },
      description: 'Ready for publication'
    }
  ]
});
```

### Localization Workflow

Workflow for managing translated content:

```typescript
const localizationWorkflow = await client.workflows.create({
  name: 'Localization Pipeline',
  api_key: 'localization',
  stages: [
    {
      name: 'Source Content',
      id: 'source',
      color: { hex: '#6B7280' },
      description: 'Original language content'
    },
    {
      name: 'In Translation',
      id: 'translating',
      color: { hex: '#F59E0B' },
      description: 'Being translated'
    },
    {
      name: 'Translation Review',
      id: 'translation_review',
      color: { hex: '#3B82F6' },
      description: 'Native speaker review'
    },
    {
      name: 'Localized',
      id: 'localized',
      color: { hex: '#10B981' },
      description: 'Translation complete'
    }
  ]
});
```

## Automation with Workflows

### Webhook Integration

Trigger actions when items change stages:

```typescript
// Create webhook for stage transitions
const stageWebhook = await client.webhooks.create({
  name: 'Stage Change Notifications',
  url: 'https://api.example.com/workflow-handler',
  events: [
    {
      entity_type: 'item',
      event_types: ['update'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['blog_post']
        }
      ]
    }
  ],
  custom_payload: `{
    "item_id": "{{ entity.id }}",
    "title": "{{ entity.attributes.title }}",
    "old_stage": "{{ previous_entity.meta.stage }}",
    "new_stage": "{{ entity.meta.stage }}",
    "changed_by": "{{ entity.meta.updated_by }}"
  }`
});
```

### Stage Transition Validation

Implement business rules for stage transitions:

```typescript
// Example: Ensure required fields before moving to review
async function canMoveToReview(itemId: string): Promise<boolean> {
  const item = await client.items.find(itemId);
  
  // Check required fields
  const requiredFields = ['title', 'content', 'seo', 'featured_image'];
  for (const field of requiredFields) {
    if (!item[field]) {
      console.log(`Missing required field: ${field}`);
      return false;
    }
  }
  
  // Check content length
  if (item.content && item.content.length < 500) {
    console.log('Content too short for review');
    return false;
  }
  
  return true;
}

// Validate before transition
const itemId = 'item-123';
if (await canMoveToReview(itemId)) {
  await client.items.update(itemId, {
    meta: { stage: 'review' }
  });
}
```

### Workflow Reports

Generate reports on workflow efficiency:

```typescript
async function getWorkflowMetrics(workflowId: string, itemTypeId: string) {
  const workflow = await client.workflows.find(workflowId);
  const stages = workflow.stages;
  
  // Get items in each stage
  const metrics = await Promise.all(
    stages.map(async (stage) => {
      const items = await client.items.list({
        filter: {
          type: itemTypeId,
          workflow_stage: { eq: stage.id }
        },
        page: { limit: 500 }
      });
      
      // Calculate average time in stage
      const times = items.map(item => {
        const updatedAt = new Date(item.meta.updated_at);
        const createdAt = new Date(item.meta.created_at);
        return (updatedAt - createdAt) / (1000 * 60 * 60); // Hours
      });
      
      const avgTime = times.length > 0 
        ? times.reduce((a, b) => a + b, 0) / times.length 
        : 0;
      
      return {
        stage: stage.name,
        count: items.length,
        avgHoursInStage: avgTime.toFixed(1)
      };
    })
  );
  
  return metrics;
}

// Generate report
const metrics = await getWorkflowMetrics('workflow-id', 'blog_post');
console.table(metrics);
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.workflows.create({
    name: 'Test Workflow',
    api_key: 'test_workflow', // May already exist
    stages: []
  });
} catch (error) {
  if (error instanceof ApiError) {
    const apiKeyError = error.findError('api_key');
    if (apiKeyError?.code === 'VALIDATION_UNIQUE') {
      console.log('Workflow API key must be unique');
    }
    
    const stagesError = error.findError('stages');
    if (stagesError?.code === 'VALIDATION_REQUIRED') {
      console.log('At least one stage is required');
    }
  }
}
```

## Workflow Object Structure

```typescript
interface Workflow {
  id: string;
  name: string;
  api_key: string;
  stages: Array<{
    id: string;
    name: string;
    color: {
      hex: string;
      alpha?: number;
    };
    description?: string;
  }>;
  created_at: string;
  updated_at: string;
}

// Item metadata when in a workflow
interface ItemMeta {
  stage?: string; // Current workflow stage ID
  created_at: string;
  updated_at: string;
  published_at?: string;
  publication_scheduled_at?: string;
  unpublishing_scheduled_at?: string;
  creator?: {
    type: 'account' | 'user' | 'sso_user' | 'access_token';
    id: string;
  };
}
```

## Common Patterns

### Assigning Workflows to Models

```typescript
// Create workflow
const workflow = await client.workflows.create({
  name: 'Blog Editorial',
  api_key: 'blog_editorial',
  stages: [
    { id: 'draft', name: 'Draft', color: { hex: '#9CA3AF' } },
    { id: 'review', name: 'Review', color: { hex: '#F59E0B' } },
    { id: 'published', name: 'Published', color: { hex: '#10B981' } }
  ]
});

// Assign to item type
await client.itemTypes.update('blog_post', {
  workflow: {
    type: 'workflow',
    id: workflow.id
  }
});

// Remove workflow from model
await client.itemTypes.update('blog_post', {
  workflow: null
});
```

### Moving Items Through Workflow Stages

```typescript
// Create item in first stage
const item = await client.items.create({
  item_type: { type: 'item_type', id: 'blog_post' },
  title: 'New Article',
  meta: {
    stage: 'draft' // Initial stage
  }
});

// Move to next stage
await client.items.update(item.id, {
  meta: {
    stage: 'review'
  }
});

// Bulk move items to stage
const job = await client.items.bulkMoveToStage({
  items: [
    { type: 'item', id: 'item-1' },
    { type: 'item', id: 'item-2' },
    { type: 'item', id: 'item-3' }
  ],
  stage: {
    type: 'workflow_stage',
    id: 'review'
  }
});

// Wait for bulk operation to complete
const result = await client.jobResults.wait(job.id);
```

### Workflow Analytics

```typescript
async function getWorkflowAnalytics(workflowId, itemTypeId, dateRange) {
  const workflow = await client.workflows.find(workflowId);
  
  // Get items in workflow
  const items = await client.items.list({
    filter: {
      type: itemTypeId,
      updated_at: {
        gte: dateRange.start.toISOString(),
        lte: dateRange.end.toISOString()
      }
    },
    page: { limit: 500 }
  });
  
  // Analyze stage transitions
  const stageMetrics = {};
  workflow.stages.forEach(stage => {
    stageMetrics[stage.id] = {
      name: stage.name,
      count: 0,
      totalTimeInStage: 0,
      transitions: {
        to: {},
        from: {}
      }
    };
  });
  
  // Calculate metrics (simplified example)
  items.forEach(item => {
    const currentStage = item.meta.stage;
    if (currentStage && stageMetrics[currentStage]) {
      stageMetrics[currentStage].count++;
    }
  });
  
  return {
    workflow: {
      id: workflow.id,
      name: workflow.name
    },
    period: dateRange,
    totalItems: items.length,
    stageMetrics,
    bottlenecks: identifyBottlenecks(stageMetrics)
  };
}

function identifyBottlenecks(stageMetrics) {
  // Identify stages with high item counts
  const stages = Object.entries(stageMetrics);
  const avgCount = stages.reduce((sum, [_, metrics]) => 
    sum + metrics.count, 0
  ) / stages.length;
  
  return stages
    .filter(([_, metrics]) => metrics.count > avgCount * 1.5)
    .map(([stageId, metrics]) => ({
      stageId,
      stageName: metrics.name,
      itemCount: metrics.count,
      severity: metrics.count > avgCount * 2 ? 'high' : 'medium'
    }));
}
```

### Workflow Templates

```typescript
const workflowTemplates = {
  simple: {
    name: 'Simple Approval',
    api_key: 'simple_approval',
    stages: [
      { id: 'pending', name: 'Pending', color: { hex: '#F59E0B' } },
      { id: 'approved', name: 'Approved', color: { hex: '#10B981' } }
    ]
  },
  
  editorial: {
    name: 'Editorial Workflow',
    api_key: 'editorial',
    stages: [
      { id: 'draft', name: 'Draft', color: { hex: '#9CA3AF' } },
      { id: 'review', name: 'Review', color: { hex: '#F59E0B' } },
      { id: 'copy_edit', name: 'Copy Edit', color: { hex: '#3B82F6' } },
      { id: 'approved', name: 'Approved', color: { hex: '#10B981' } }
    ]
  },
  
  translation: {
    name: 'Translation Workflow',
    api_key: 'translation',
    stages: [
      { id: 'source', name: 'Source Content', color: { hex: '#6B7280' } },
      { id: 'translating', name: 'Translating', color: { hex: '#F59E0B' } },
      { id: 'review', name: 'Review', color: { hex: '#3B82F6' } },
      { id: 'complete', name: 'Complete', color: { hex: '#10B981' } }
    ]
  }
};

async function createFromTemplate(templateName) {
  const template = workflowTemplates[templateName];
  if (!template) {
    throw new Error(`Template ${templateName} not found`);
  }
  
  return client.workflows.create(template);
}
```

## Related Resources

- [Items](../01-content-management/item.md) - Move items through workflow stages
- [Item Types](../01-content-management/item-type.md) - Assign workflows to models
- [Scheduled Publication](./scheduled-publication.md) - Schedule content publishing
- [Scheduled Unpublishing](./scheduled-unpublishing.md) - Schedule content removal
- [Webhooks](../04-site-configuration/webhook.md) - Automate workflow transitions
- [Roles](../05-access-control/role.md) - Control who can manage workflows