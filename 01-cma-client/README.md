# Content Management API Client

The Content Management API (CMA) client provides programmatic access to manage all aspects of your DatoCMS project.

## Installation

```bash
# For Node.js (includes file upload support)
npm install @datocms/cma-client-node

# For browser environments
npm install @datocms/cma-client-browser

# Core client (platform-agnostic)
npm install @datocms/cma-client
```

## Client Initialization

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN',
  environment: 'main' // optional, defaults to primary environment
});
```

## Resource Categories

### [01. Content Management](./01-content-management/)
- **[item.md](./01-content-management/item.md)**: Content records - create, read, update, delete, publish
- **[item-type.md](./01-content-management/item-type.md)**: Content models/schemas
- **[field.md](./01-content-management/field.md)**: Model fields configuration
- **[fieldset.md](./01-content-management/fieldset.md)**: Field grouping for better UI organization
- **[item-version.md](./01-content-management/item-version.md)**: Content versioning and history
- **[modular-content.md](./01-content-management/modular-content.md)**: Block-based content composition
- **[structured-text.md](./01-content-management/structured-text.md)**: Rich text with embedded blocks

### [02. Media Management](./02-media-management/)
- **[upload.md](./02-media-management/upload.md)**: File uploads and media assets
- **[upload-collection.md](./02-media-management/upload-collection.md)**: Organize uploads
- **[upload-tag.md](./02-media-management/upload-tag.md)**: Tag uploads for categorization
- **[upload-smart-tag.md](./02-media-management/upload-smart-tag.md)**: AI-powered auto-tagging
- **[upload-track.md](./02-media-management/upload-track.md)**: Video/audio tracks

### [03. Publishing Workflow](./03-publishing-workflow/)
- **[scheduled-publication.md](./03-publishing-workflow/scheduled-publication.md)**: Schedule content publishing
- **[scheduled-unpublishing.md](./03-publishing-workflow/scheduled-unpublishing.md)**: Schedule content unpublishing
- **[workflow.md](./03-publishing-workflow/workflow.md)**: Content approval workflows

### [04. Site Configuration](./04-site-configuration/)
- **[site.md](./04-site-configuration/site.md)**: Site settings and configuration
- **[environment.md](./04-site-configuration/environment.md)**: Environment management
- **[plugin.md](./04-site-configuration/plugin.md)**: Plugin installation and configuration
- **[webhook.md](./04-site-configuration/webhook.md)**: HTTP webhooks for events
- **[schema-menu-item.md](./04-site-configuration/schema-menu-item.md)**: Customize navigation
- **[build-trigger.md](./04-site-configuration/build-trigger.md)**: CI/CD build triggers

### [05. Access Control](./05-access-control/)
- **[user.md](./05-access-control/user.md)**: User management
- **[role.md](./05-access-control/role.md)**: Roles and permissions
- **[access-token.md](./05-access-control/access-token.md)**: API token management

### [06. Monitoring](./06-monitoring/)
- **[audit-log-event.md](./06-monitoring/audit-log-event.md)**: Audit trail
- **[build-event.md](./06-monitoring/build-event.md)**: Build status and logs
- **[webhook-call.md](./06-monitoring/webhook-call.md)**: Webhook execution logs
- **[search-result.md](./06-monitoring/search-result.md)**: Full-text search

## Common Patterns

### Error Handling

```typescript
import { ApiError } from '@datocms/cma-client';

try {
  await client.items.create({ /* ... */ });
} catch (error) {
  if (error instanceof ApiError) {
    console.error('API Error:', error.message);
    console.error('Error details:', error.errors);
  }
}
```

### Pagination

```typescript
// Using async iterators (recommended)
for await (const item of client.items.listPagedIterator()) {
  console.log(item);
}

// Manual pagination
const page1 = await client.items.list({ 
  page: { limit: 100, offset: 0 } 
});
```

### Raw Methods

Every method has a `raw` variant that returns the full HTTP response:

```typescript
// Regular method - returns just the data
const item = await client.items.find('item-id');

// Raw method - returns full response with headers
const response = await client.items.rawFind('item-id');
console.log(response.data); // The item
console.log(response.headers); // HTTP headers
```

## TypeScript Support

All methods are fully typed:

```typescript
import { SimpleSchemaTypes } from '@datocms/cma-client';

// Type-safe item creation
const newArticle: SimpleSchemaTypes.ItemCreateSchema = {
  item_type: { type: 'item_type', id: 'article' },
  title: 'My Article',
  // TypeScript will enforce correct field types
};
```

## Rate Limiting

The client automatically handles rate limiting with exponential backoff. You can also access rate limit information:

```typescript
try {
  await client.items.list();
} catch (error) {
  if (error instanceof ApiError && error.response?.status === 429) {
    const retryAfter = error.response.headers.get('X-RateLimit-Reset');
    console.log(`Rate limited. Retry after: ${retryAfter}`);
  }
}
```

## Related Documentation

- [Authentication Setup](../00-quick-start/authentication.md)
- [Dashboard Client](../02-dashboard-client/README.md)
- [Plugin SDK](../03-plugin-sdk/README.md)
- [Advanced Topics](../04-advanced-topics/)