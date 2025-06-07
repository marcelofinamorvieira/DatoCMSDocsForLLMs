# Code Generation from Hyperschema

DatoCMS API clients are automatically generated from JSON Hyper-Schema definitions, ensuring type safety, consistency, and maintainability across all SDK packages.

## What is JSON Hyper-Schema?

JSON Hyper-Schema extends JSON Schema with hypermedia capabilities:

```json
{
  "$schema": "http://json-schema.org/hyper-schema",
  "type": "object",
  "properties": {
    "item": {
      "type": "object",
      "properties": {
        "id": { "type": "string" },
        "title": { "type": "string" }
      },
      "links": [
        {
          "rel": "self",
          "href": "/items/{id}",
          "method": "GET",
          "title": "Find item by ID"
        }
      ]
    }
  }
}
```

Key features:
- **Schema definitions**: Describe data structures using JSON Schema
- **Links**: Define API endpoints with HTTP methods, URL templates, and relationships
- **Self-documenting**: Includes titles, descriptions, and examples
- **Type constraints**: Validation rules, patterns, and formats
- **Relationships**: Express how resources relate to each other

## DatoCMS API Hyperschema Structure

DatoCMS provides two main hyperschemas:

### 1. Content Management API (CMA)
```
https://site-api.datocms.com/docs/site-api-hyperschema.json
```

Structure:
```
{
  "properties": {
    "item": { /* Item resource schema */ },
    "item_type": { /* Model schema */ },
    "field": { /* Field schema */ },
    "upload": { /* Asset schema */ },
    // ... other resources
  }
}
```

### 2. Account/Dashboard API
```
https://site-api.datocms.com/docs/account-api-hyperschema.json
```

Structure:
```
{
  "properties": {
    "account": { /* Account schema */ },
    "site": { /* Project schema */ },
    "subscription": { /* Billing schema */ },
    // ... other resources
  }
}
```

Each resource contains:
- **Properties**: Field definitions with types and constraints
- **Links**: Available operations (CRUD, queries, actions)
- **Required fields**: Validation requirements
- **Examples**: Sample data for documentation

## Code Generation Process Overview

```
┌─────────────────────┐
│   Hyperschema JSON  │
│  (API Definition)   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Extract & Parse    │
│  - Resources        │
│  - Endpoints        │
│  - Types            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Generate TypeScript│
│  - Interfaces       │
│  - Type definitions │
│  - Enums            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Apply Templates    │
│  - Client class     │
│  - Resource classes │
│  - Methods          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Generated SDK      │
│  - Type-safe        │
│  - Auto-documented  │
│  - Consistent API   │
└─────────────────────┘
```

### Generation Steps:

1. **Download Hyperschema**
   ```typescript
   const response = await fetch(hyperschemaUrl);
   const rawSchema = await response.json();
   ```

2. **Extract Information**
   ```typescript
   // Parse resources, endpoints, and relationships
   const schemaInfo = await extractInfoFromSchema(hyperschemaUrl);
   ```

3. **Generate Types**
   ```typescript
   // Convert JSON Schema to TypeScript interfaces
   const typings = await hyperschemaToTypings(schema);
   ```

4. **Apply Templates**
   ```typescript
   // Use Handlebars templates to generate code
   await writeTemplate('Client.ts', schemaInfo, outputPath);
   ```

## Benefits of Auto-generation

### 1. **Type Safety**
All request/response types are automatically generated:
```typescript
// Auto-generated types ensure compile-time safety
interface ItemCreateSchema {
  data: {
    type: 'item';
    attributes: {
      title: string;
      content?: string;
    };
    relationships?: {
      item_type: {
        data: { type: 'item_type'; id: string };
      };
    };
  };
}
```

### 2. **API Consistency**
All clients follow the same patterns:
```typescript
// Consistent method signatures across resources
client.items.create(body);
client.uploads.create(body);
client.users.create(body);
```

### 3. **Automatic Documentation**
Comments are extracted from hyperschema:
```typescript
/**
 * Create a new record
 *
 * Read more: https://www.datocms.com/docs/content-management-api/resources/item/create
 *
 * @throws {ApiError}
 * @throws {TimeoutError}
 */
create(body: ItemCreateSchema): Promise<ItemCreateTargetSchema>
```

### 4. **Version Management**
API changes are automatically reflected:
- New endpoints appear as methods
- Deprecated endpoints are marked
- Type changes are propagated
- Breaking changes are caught at compile time

### 5. **Reduced Maintenance**
- No manual SDK updates needed
- Eliminates human error
- Consistent with API documentation
- Single source of truth

## How Resources and Methods are Generated

### Resource Class Generation

For each resource in the hyperschema:

```typescript
// Template: ResourceClass.ts.handlebars
export default class Item extends BaseResource {
  static readonly TYPE = 'item' as const;

  // Methods generated from links...
}
```

### Method Generation from Links

Each link in the hyperschema generates methods:

```typescript
// Hyperschema link
{
  "rel": "create",
  "href": "/items",
  "method": "POST",
  "title": "Create a new record",
  "schema": { /* request schema */ },
  "targetSchema": { /* response schema */ }
}

// Generated method
create(body: ItemCreateSchema): Promise<ItemCreateTargetSchema> {
  return this.client.request({
    method: 'POST',
    url: '/items',
    body,
  });
}
```

### Smart Method Generation

The generator creates two versions:

1. **Raw methods** - Direct API mapping:
   ```typescript
   rawCreate(body: SchemaTypes.ItemCreateSchema)
   ```

2. **Simple methods** - User-friendly wrappers:
   ```typescript
   create(body: SimpleSchemaTypes.ItemCreateSchema)
   ```

### Pagination Support

For collection endpoints:
```typescript
// Auto-generated iterator
async *listPagedIterator(
  queryParams?: ItemInstancesHrefSchema,
  iteratorOptions?: IteratorOptions,
) {
  // Automatic pagination handling
}
```

## Type Generation from Schema

### JSON Schema to TypeScript

The `hyperschema-to-ts` compiler converts:

```json
{
  "type": "object",
  "required": ["title"],
  "properties": {
    "title": { "type": "string" },
    "published": { "type": "boolean" }
  }
}
```

Into:
```typescript
export type ItemAttributes = {
  title: string;
  published?: boolean;
};
```

### Complex Type Handling

#### Union Types
```json
{ "type": ["string", "null"] }
```
→
```typescript
string | null
```

#### Enums
```json
{ "enum": ["draft", "published", "updated"] }
```
→
```typescript
'draft' | 'published' | 'updated'
```

#### References
```json
{ "$ref": "#/definitions/item" }
```
→
```typescript
Item
```

## Customization and Extension Points

### 1. **Manual Resource Overrides**

Place custom implementations in:
```
packages/cma-client/src/resources/Item.ts
```

The generator checks for existing files:
```typescript
const path = existsSync(
  `./packages/${prefix}-client/src/resources/${resourceClassName}.ts`
)
  ? `../../resources/${resourceClassName}`
  : `./${resourceClassName}`;
```

### 2. **Custom Methods**

Extend generated classes:
```typescript
// packages/cma-client/src/resources/Item.ts
import GeneratedItem from '../generated/resources/Item';

export default class Item extends GeneratedItem {
  // Add custom methods
  async publishWithDependencies(id: string) {
    // Custom logic
  }
}
```

### 3. **Type Extensions**

Add custom types:
```typescript
// Extend generated types
import { ItemCreateSchema } from '../generated/SchemaTypes';

export interface ExtendedItemCreateSchema extends ItemCreateSchema {
  customField?: string;
}
```

## Manual Overrides and Additions

### Platform-Specific Implementations

For browser/Node.js differences:

```
packages/cma-client-browser/src/resources/Upload.ts
packages/cma-client-node/src/resources/Upload.ts
```

### Adding Non-Generated Functionality

1. **Utility Methods**
   ```typescript
   class ItemUtils {
     static validateSeo(item: Item) { /* ... */ }
     static generateSlug(title: string) { /* ... */ }
   }
   ```

2. **Helper Functions**
   ```typescript
   export function buildTreeFromItems(items: Item[]) {
     // Custom tree building logic
   }
   ```

3. **Event Handlers**
   ```typescript
   client.on('beforeRequest', (config) => {
     // Add custom headers, logging, etc.
   });
   ```

## Version Management

### Regeneration Process

1. **Check for API Updates**
   ```bash
   npm run generate
   ```

2. **Review Changes**
   ```bash
   git diff packages/*/src/generated
   ```

3. **Update Package Versions**
   ```json
   {
     "version": "3.2.0",
     "description": "Updated with new API endpoints"
   }
   ```

### Handling Breaking Changes

The generator helps identify:
- Removed endpoints (compilation errors)
- Changed types (type mismatches)
- New required fields (missing properties)

## Contributing to the Clients

### Understanding the Templates

#### Client Template (`Client.ts.handlebars`)
```handlebars
export class Client {
  {{#each resources}}
    {{{namespace}}}: Resources.{{{resourceClassName}}};
  {{/each}}

  constructor(config: ClientConfigOptions) {
    {{#each resources}}
      this.{{{namespace}}} = new Resources.{{{resourceClassName}}}(this);
    {{/each}}
  }
}
```

#### Resource Template (`ResourceClass.ts.handlebars`)
```handlebars
{{#each endpoints}}
  {{{name}}}(
    {{#each urlPlaceholders}}
      {{{variableName}}}: string,
    {{/each}}
    {{#if requestBodyType}}
      body: SchemaTypes.{{{requestBodyType}}},
    {{/if}}
  ) {
    return this.client.request({
      method: '{{method}}',
      url: `{{urlTemplate}}`,
      {{#if requestBodyType}}body,{{/if}}
    });
  }
{{/each}}
```

### Adding New Features

1. **Update Extraction Logic**
   ```typescript
   // In extractInfoFromSchema.ts
   function extractNewFeature(link: JSONHyperschemaLink) {
     // Extract new hyperschema features
   }
   ```

2. **Modify Templates**
   ```handlebars
   {{#if newFeature}}
     // Generate new feature code
   {{/if}}
   ```

3. **Test Generation**
   ```bash
   GENERATE_FROM_DEV=1 npm run generate
   ```

## Best Practices for Extending

### 1. **Preserve Generated Code**
Never modify files in `/generated` directories - they will be overwritten.

### 2. **Use Composition**
```typescript
// Compose, don't modify
class CustomClient {
  private client: Client;
  
  constructor(config: ClientConfigOptions) {
    this.client = new Client(config);
  }
  
  // Add custom methods
}
```

### 3. **Type-Safe Extensions**
```typescript
// Use generated types as base
import { Item } from '@datocms/cma-client';

interface ExtendedItem extends Item {
  customField: string;
}
```

### 4. **Document Overrides**
```typescript
/**
 * Custom implementation for file uploads
 * Overrides generated method to add progress tracking
 */
class Upload extends GeneratedUpload {
  // ...
}
```

## Comparison with Manual SDK Development

### Auto-generated Approach

**Pros:**
- Always in sync with API
- Consistent patterns
- Type safety guaranteed
- Reduced maintenance
- Automatic documentation

**Cons:**
- Less flexibility
- Template complexity
- Learning curve
- Build step required

### Manual Approach

**Pros:**
- Full control
- Custom abstractions
- No build dependencies
- Simpler to understand

**Cons:**
- Maintenance burden
- Inconsistencies
- Documentation drift
- Human error
- Slower updates

### DatoCMS Choice

DatoCMS chose auto-generation because:
1. **Large API surface** - Hundreds of endpoints
2. **Frequent updates** - Regular API enhancements
3. **Multiple clients** - CMA, Dashboard, browser, Node.js
4. **Type safety** - Critical for developer experience
5. **Consistency** - Same patterns everywhere

## Build Process Explanation

### Complete Build Flow

```bash
# 1. Install dependencies
npm install

# 2. Generate from production API
npm run generate

# 3. Or generate from development
GENERATE_FROM_DEV=1 npm run generate

# 4. Build TypeScript
npm run build

# 5. Run tests
npm test

# 6. Publish packages
npm run publish
```

### Generation Script

```typescript
#!/usr/bin/env node --stack_size=800 -r ts-node/register

// Main generation entry point
async function generate(prefix: string, hyperschemaUrl: string) {
  // 1. Download and parse hyperschema
  const schemaInfo = await extractInfoFromSchema(hyperschemaUrl);
  
  // 2. Clean output directory
  rimraf.sync(`./packages/${prefix}-client/src/generated`);
  
  // 3. Generate Client class
  await writeTemplate('Client.ts', schemaInfo);
  
  // 4. Generate type definitions
  await writeTemplate('SchemaTypes.ts', schemaInfo);
  
  // 5. Generate resource classes
  for (const resource of schemaInfo.resources) {
    await writeTemplate('ResourceClass.ts', resource);
  }
}
```

### CI/CD Integration

```yaml
# .github/workflows/generate.yml
name: Generate SDK
on:
  schedule:
    - cron: '0 0 * * *'  # Daily
jobs:
  generate:
    steps:
      - run: npm run generate
      - run: npm test
      - run: git diff --exit-code  # Fail if changes
```

This ensures the SDK stays synchronized with API changes automatically.