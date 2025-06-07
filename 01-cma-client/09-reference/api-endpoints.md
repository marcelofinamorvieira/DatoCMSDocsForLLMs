# API Endpoints Reference

## Base URLs

| Environment | URL |
|-------------|-----|
| Content Management API | `https://site-api.datocms.com` |
| Content Delivery API | `https://graphql.datocms.com` |
| Dashboard API | `https://dashboard-api.datocms.com` |

## Authentication Headers

```http
Authorization: Bearer YOUR_API_TOKEN
Content-Type: application/json
Accept: application/json
X-Api-Version: 3
```

## Content Management API Endpoints

### Items (Content)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/items` | List all items |
| POST | `/items` | Create new item |
| GET | `/items/:id` | Get single item |
| PUT | `/items/:id` | Update item |
| DELETE | `/items/:id` | Delete item |
| PUT | `/items/:id/publish` | Publish item |
| PUT | `/items/:id/unpublish` | Unpublish item |
| POST | `/items/:id/duplicate` | Duplicate item |
| POST | `/items/bulk` | Bulk operations |
| GET | `/items/:id/references` | Get referencing items |
| GET | `/items/:id/versions` | Get item versions |

### Item Types (Models)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/item-types` | List all models |
| POST | `/item-types` | Create model |
| GET | `/item-types/:id` | Get model |
| PUT | `/item-types/:id` | Update model |
| DELETE | `/item-types/:id` | Delete model |
| POST | `/item-types/:id/duplicate` | Duplicate model |
| PUT | `/item-types/:id/order-fields` | Reorder fields |

### Fields

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/item-types/:item_type_id/fields` | List fields |
| POST | `/item-types/:item_type_id/fields` | Create field |
| GET | `/fields/:id` | Get field |
| PUT | `/fields/:id` | Update field |
| DELETE | `/fields/:id` | Delete field |
| PUT | `/fields/:id/move` | Move field position |

### Uploads (Media)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/uploads` | List uploads |
| POST | `/uploads` | Create upload |
| POST | `/upload-requests` | Request upload URL |
| POST | `/uploads/create-from-url` | Upload from URL |
| GET | `/uploads/:id` | Get upload |
| PUT | `/uploads/:id` | Update upload metadata |
| DELETE | `/uploads/:id` | Delete upload |
| POST | `/uploads/bulk` | Bulk upload operations |
| GET | `/upload-filters` | Get smart tags |

### Users & Permissions

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users` | List users |
| POST | `/users` | Invite user |
| GET | `/users/:id` | Get user |
| PUT | `/users/:id` | Update user |
| DELETE | `/users/:id` | Remove user |
| POST | `/users/:id/reset_password` | Reset password |

### Roles

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/roles` | List roles |
| POST | `/roles` | Create role |
| GET | `/roles/:id` | Get role |
| PUT | `/roles/:id` | Update role |
| DELETE | `/roles/:id` | Delete role |

### Environments

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/environments` | List environments |
| POST | `/environments` | Create environment |
| GET | `/environments/:id` | Get environment |
| PUT | `/environments/:id` | Update environment |
| DELETE | `/environments/:id` | Delete environment |
| POST | `/environments/:id/fork` | Fork environment |
| POST | `/environments/:id/promote` | Promote to primary |

### Webhooks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/webhooks` | List webhooks |
| POST | `/webhooks` | Create webhook |
| GET | `/webhooks/:id` | Get webhook |
| PUT | `/webhooks/:id` | Update webhook |
| DELETE | `/webhooks/:id` | Delete webhook |
| POST | `/webhooks/:id/trigger` | Test webhook |
| GET | `/webhook-calls` | List webhook calls |

### Plugins

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/plugins` | List installed plugins |
| POST | `/plugins` | Install plugin |
| GET | `/plugins/:id` | Get plugin |
| PUT | `/plugins/:id` | Update plugin config |
| DELETE | `/plugins/:id` | Uninstall plugin |

### Access Tokens

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/access_tokens` | List tokens |
| POST | `/access_tokens` | Create token |
| GET | `/access_tokens/:id` | Get token |
| PUT | `/access_tokens/:id` | Update token |
| DELETE | `/access_tokens/:id` | Delete token |
| POST | `/access_tokens/:id/regenerate` | Regenerate token |

### Site Settings

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/site` | Get site info |
| PUT | `/site` | Update site settings |
| GET | `/site/usage` | Get usage stats |
| GET | `/build-triggers` | List build triggers |
| POST | `/build-triggers` | Create build trigger |

### Scheduled Publishing

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/scheduled-publications` | List scheduled |
| POST | `/scheduled-publications` | Schedule publish |
| DELETE | `/scheduled-publications/:id` | Cancel schedule |

### Jobs (Async Operations)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/jobs` | List jobs |
| GET | `/jobs/:id` | Get job status |
| POST | `/jobs/import` | Start import job |
| POST | `/jobs/export` | Start export job |

## Query Parameters

### Pagination

| Parameter | Description | Example |
|-----------|-------------|---------|
| `page[offset]` | Skip N items | `page[offset]=20` |
| `page[limit]` | Items per page | `page[limit]=50` |
| `page[cursor]` | Cursor pagination | `page[cursor]=eyJpZCI6IjEyMyJ9` |

### Filtering

| Parameter | Description | Example |
|-----------|-------------|---------|
| `filter[type]` | Filter by type | `filter[type]=article` |
| `filter[fields][field_name][operator]` | Field filter | `filter[fields][title][eq]=Hello` |
| `filter[query]` | Full-text search | `filter[query]=search+term` |
| `filter[ids]` | Filter by IDs | `filter[ids]=123,456,789` |

### Field Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `filter[fields][status][eq]=published` |
| `neq` | Not equals | `filter[fields][status][neq]=draft` |
| `lt` | Less than | `filter[fields][price][lt]=100` |
| `lte` | Less than or equal | `filter[fields][price][lte]=100` |
| `gt` | Greater than | `filter[fields][views][gt]=1000` |
| `gte` | Greater than or equal | `filter[fields][stock][gte]=10` |
| `in` | In array | `filter[fields][id][in]=1,2,3` |
| `notIn` | Not in array | `filter[fields][status][notIn]=deleted,archived` |
| `exists` | Has value | `filter[fields][image][exists]=true` |
| `matches` | Pattern match | `filter[fields][title][matches][pattern]=guide` |

### Ordering

| Parameter | Description | Example |
|-----------|-------------|---------|
| `order_by` | Sort field | `order_by=created_at_DESC` |
| Multiple | Comma separated | `order_by=position_ASC,title_DESC` |

### Including Related Data

| Parameter | Description | Example |
|-----------|-------------|---------|
| `include` | Include relations | `include=author,categories` |
| `fields[TYPE]` | Limit fields | `fields[article]=title,slug` |

## Response Headers

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Requests per period |
| `X-RateLimit-Remaining` | Remaining requests |
| `X-RateLimit-Reset` | Reset timestamp |
| `X-Total-Count` | Total items (list endpoints) |
| `X-Request-Id` | Request identifier |

## Status Codes

| Code | Meaning | Common Cause |
|------|---------|--------------|
| 200 | OK | Success |
| 201 | Created | Resource created |
| 204 | No Content | Delete success |
| 400 | Bad Request | Invalid parameters |
| 401 | Unauthorized | Invalid/missing token |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable | Validation error |
| 429 | Too Many Requests | Rate limited |
| 500 | Server Error | Internal error |

## Rate Limits

| Plan | Requests/Second | Burst |
|------|----------------|-------|
| Free | 30 | 60 |
| Pro | 60 | 120 |
| Enterprise | Custom | Custom |

## Example Requests

### Create Item
```http
POST /items
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json

{
  "data": {
    "type": "item",
    "attributes": {
      "title": "New Article",
      "content": "Article content"
    },
    "relationships": {
      "item_type": {
        "data": { "type": "item_type", "id": "article" }
      }
    }
  }
}
```

### Filter Items
```http
GET /items?filter[type]=article&filter[fields][published][eq]=true&page[limit]=20
Authorization: Bearer YOUR_TOKEN
```

### Update Item
```http
PUT /items/123456
Authorization: Bearer YOUR_TOKEN
Content-Type: application/json

{
  "data": {
    "type": "item",
    "id": "123456",
    "attributes": {
      "title": "Updated Title"
    }
  }
}
```

## Dashboard API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/account` | Get account info |
| PUT | `/account` | Update account |
| GET | `/account/sites` | List all projects |
| POST | `/account/sites` | Create project |
| GET | `/billing` | Get billing info |
| PUT | `/billing` | Update billing |

## Webhook Payload

```json
{
  "entity_type": "item",
  "event_type": "publish",
  "entity": {
    "id": "123456",
    "type": "item",
    "attributes": { ... }
  },
  "related_entities": [ ... ],
  "environment": "main"
}
```