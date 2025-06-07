# schemaItemTypeDropdownActions Hook

## Purpose

The `schemaItemTypeDropdownActions` hook allows you to add custom dropdown actions to model/item type management in the schema editor. This enables you to add functionality like model templates, bulk operations, or integration actions at the schema level.

## Signature

```typescript
schemaItemTypeDropdownActions(
  ctx: SchemaItemTypeDropdownActionsCtx
): DropdownAction[] | undefined
```

## Parameters

- `ctx`: SchemaItemTypeDropdownActionsCtx - Context object containing:
  - Base properties and methods from plugin SDK
  - `itemType`: ItemType - The item type/model where the dropdown is shown

## Type Definitions

```typescript
// Main context type
interface SchemaItemTypeDropdownActionsCtx extends BaseCtx {
  itemType: ItemType;
}

// Base context properties and methods
interface BaseCtx {
  // Properties
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  site: Site;
  environment: string;
  theme: Theme;
  
  // Methods
  notice(message: string): void;
  alert(message: string): void;
  openModal(options: ModalOptions): Promise<any>;
  openConfirm(options: ConfirmOptions): Promise<boolean>;
  navigateTo(path: string): void;
  openNewTab(url: string): void;
  createClient(): CmaClient;
}

// Item type structure
interface ItemType {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    apiKey: string;
    singleton: boolean;
    sortable: boolean;
    modularBlock: boolean;
    tree: boolean;
    orderingDirection: 'asc' | 'desc' | null;
    orderingMeta: string | null;
    draftModeActive: boolean;
    allLocalesRequired: boolean;
    collectionAppearance: 'table' | 'gallery';
    hint: string | null;
    titleField: string | null;
  };
  relationships: {
    fields: {
      data: Array<{ id: string; type: 'field' }>;
    };
    fieldsets: {
      data: Array<{ id: string; type: 'fieldset' }>;
    };
  };
}

// Plugin configuration
interface Plugin {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description: string | null;
    homepage: string | null;
    packageName: string;
    packageVersion: string;
    parameters: Record<string, any>;
  };
}

// User types
type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    firstName: string;
    lastName: string;
    avatarUrl: string | null;
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    firstName: string;
    lastName: string;
    avatarUrl: string | null;
  };
};

type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    name: string;
    avatarUrl: string | null;
  };
};

type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
  };
};

// Role configuration
interface Role {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    canEditSchema: boolean;
    canManageAccessTokens: boolean;
    canManageWebhooks: boolean;
    canManageEnvironments: boolean;
    canAccessDeployLogs: boolean;
  };
}

// Site (project) configuration
interface Site {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    locales: string[];
    domain: string | null;
  };
}

// Theme configuration
interface Theme {
  primaryColor: string;
  accentColor: string;
  semiTransparentAccentColor: string;
  lightColor: string;
  darkColor: string;
}

// Modal options
interface ModalOptions {
  id: string;
  title: string;
  width?: 'small' | 'medium' | 'large' | 'fullWidth';
  parameters?: Record<string, any>;
}

// Confirm dialog options
interface ConfirmOptions {
  title: string;
  content?: string;
  cancel?: {
    label: string;
    value: false;
  };
  choices: Array<{
    label: string;
    value: true;
    intent?: 'positive' | 'negative' | 'primary';
  }>;
}

// CMA Client for API operations
interface CmaClient {
  fields: {
    list(itemTypeId: string): Promise<Field[]>;
    create(itemTypeId: string, data: any): Promise<Field>;
    update(fieldId: string, data: any): Promise<Field>;
    destroy(fieldId: string): Promise<void>;
  };
  itemTypes: {
    create(data: any): Promise<ItemType>;
    update(itemTypeId: string, data: any): Promise<ItemType>;
    list(): Promise<ItemType[]>;
  };
  uploads: {
    create(data: any): Promise<Upload>;
    update(uploadId: string, data: any): Promise<Upload>;
  };
}

// Field structure
interface Field {
  id: string;
  label: string;
  apiKey: string;
  fieldType: string;
  localized: boolean;
  validators: Record<string, any>;
  appearance: {
    editor: string;
    parameters: Record<string, any>;
  };
  position: number;
  hint: string | null;
  defaultValue: any;
}
```

## Return Value

Returns an array of `DropdownAction` objects with an additional `handler` function:

```typescript
interface DropdownAction {
  id: string;
  label: string;
  icon?: ReactElement | ReactNode;
  disabled?: boolean;
  tooltip?: string;
  rank?: number;
  group?: string;
  handler: () => void | Promise<void>;
}
```

## Use Cases

### 1. Model Template Generator

Generate fields from predefined templates:

```typescript
schemaItemTypeDropdownActions(ctx: SchemaItemTypeDropdownActionsCtx) {
  return [
    {
      id: 'apply-template',
      label: 'Apply Field Template',
      icon: <FaClipboardList />,
      group: 'templates',
      rank: 10,
      async handler() {
        const templates = {
          blog: {
            name: 'Blog Post Template',
            fields: [
              { label: 'Title', apiKey: 'title', fieldType: 'string', required: true },
              { label: 'Slug', apiKey: 'slug', fieldType: 'slug', titleField: 'title' },
              { label: 'Author', apiKey: 'author', fieldType: 'link', itemType: 'author' },
              { label: 'Publish Date', apiKey: 'publish_date', fieldType: 'date_time' },
              { label: 'Featured Image', apiKey: 'featured_image', fieldType: 'file' },
              { label: 'Excerpt', apiKey: 'excerpt', fieldType: 'text', maxLength: 200 },
              { label: 'Content', apiKey: 'content', fieldType: 'structured_text' },
              { label: 'Tags', apiKey: 'tags', fieldType: 'links', itemType: 'tag' },
              { label: 'SEO', apiKey: 'seo', fieldType: 'seo' },
            ],
          },
          product: {
            name: 'E-commerce Product Template',
            fields: [
              { label: 'Name', apiKey: 'name', fieldType: 'string', required: true },
              { label: 'SKU', apiKey: 'sku', fieldType: 'string', required: true, unique: true },
              { label: 'Price', apiKey: 'price', fieldType: 'float', required: true, min: 0 },
              { label: 'Description', apiKey: 'description', fieldType: 'text' },
              { label: 'Images', apiKey: 'images', fieldType: 'gallery' },
              { label: 'Category', apiKey: 'category', fieldType: 'link', itemType: 'category' },
              { label: 'In Stock', apiKey: 'in_stock', fieldType: 'boolean', defaultValue: true },
              { label: 'Weight', apiKey: 'weight', fieldType: 'float', min: 0 },
              { label: 'Dimensions', apiKey: 'dimensions', fieldType: 'json' },
            ],
          },
          event: {
            name: 'Event Template',
            fields: [
              { label: 'Event Name', apiKey: 'name', fieldType: 'string', required: true },
              { label: 'Start Date', apiKey: 'start_date', fieldType: 'date_time', required: true },
              { label: 'End Date', apiKey: 'end_date', fieldType: 'date_time' },
              { label: 'Location', apiKey: 'location', fieldType: 'lat_lon' },
              { label: 'Venue', apiKey: 'venue', fieldType: 'string' },
              { label: 'Description', apiKey: 'description', fieldType: 'text' },
              { label: 'Ticket URL', apiKey: 'ticket_url', fieldType: 'string', validators: { format: { predefinedPattern: 'url' } } },
              { label: 'Speakers', apiKey: 'speakers', fieldType: 'links', itemType: 'person' },
            ],
          },
        };

        // Let user select template
        const selectedTemplate = await ctx.openModal({
          id: 'selectTemplate',
          title: 'Select Field Template',
          width: 'medium',
          parameters: { templates },
        });

        if (!selectedTemplate) return;

        const template = templates[selectedTemplate];
        const client = ctx.createClient();

        ctx.notice(`Applying ${template.name}...`);

        try {
          // Get existing fields to avoid duplicates
          const existingFields = await client.fields.list(ctx.itemType.id);
          const existingApiKeys = existingFields.map(f => f.apiKey);

          // Create fields from template
          for (const fieldDef of template.fields) {
            if (existingApiKeys.includes(fieldDef.apiKey)) {
              ctx.notice(`Skipping ${fieldDef.label} - field already exists`);
              continue;
            }

            const fieldData: any = {
              label: fieldDef.label,
              apiKey: fieldDef.apiKey,
              fieldType: fieldDef.fieldType,
              appearance: { editor: fieldDef.fieldType, parameters: {} },
              validators: {},
            };

            // Add validators
            if (fieldDef.required) {
              fieldData.validators.required = {};
            }
            if (fieldDef.unique) {
              fieldData.validators.unique = {};
            }
            if (fieldDef.min !== undefined) {
              fieldData.validators.numberRange = { min: fieldDef.min };
            }
            if (fieldDef.maxLength !== undefined) {
              fieldData.validators.length = { max: fieldDef.maxLength };
            }

            // Handle special field types
            if (fieldDef.fieldType === 'slug' && fieldDef.titleField) {
              const titleField = existingFields.find(f => f.apiKey === fieldDef.titleField);
              if (titleField) {
                fieldData.validators.slugTitleField = { titleFieldId: titleField.id };
              }
            }

            if ((fieldDef.fieldType === 'link' || fieldDef.fieldType === 'links') && fieldDef.itemType) {
              const linkedItemType = Object.values(ctx.site.itemTypes).find(
                it => it.apiKey === fieldDef.itemType
              );
              if (linkedItemType) {
                fieldData.validators.itemItemType = { itemTypes: [linkedItemType.id] };
              }
            }

            await client.fields.create(ctx.itemType.id, fieldData);
          }

          ctx.notice(`Successfully applied ${template.name}`);
          
          // Refresh the page to show new fields
          window.location.reload();
        } catch (error) {
          ctx.alert(`Failed to apply template: ${error.message}`);
        }
      },
    },
  ];
}
```

### 2. Model Documentation Generator

Generate documentation for models:

```typescript
schemaItemTypeDropdownActions(ctx: SchemaItemTypeDropdownActionsCtx) {
  return [
    {
      id: 'generate-docs',
      label: 'Generate Documentation',
      icon: <FaBook />,
      group: 'documentation',
      rank: 20,
      async handler() {
        const client = ctx.createClient();
        
        ctx.notice('Generating documentation...');

        try {
          // Fetch all fields for this model
          const fields = await client.fields.list(ctx.itemType.id);
          
          // Generate markdown documentation
          let markdown = `# ${ctx.itemType.attributes.name} Model Documentation\n\n`;
          markdown += `**API Key:** \`${ctx.itemType.attributes.apiKey}\`\n\n`;
          
          if (ctx.itemType.attributes.hint) {
            markdown += `## Description\n\n${ctx.itemType.attributes.hint}\n\n`;
          }

          markdown += `## Fields\n\n`;
          markdown += `| Field | API Key | Type | Required | Description |\n`;
          markdown += `|-------|---------|------|----------|-------------|\n`;

          fields.forEach(field => {
            const required = field.validators?.required ? '✓' : '';
            const hint = field.hint || '';
            markdown += `| ${field.label} | \`${field.apiKey}\` | ${field.fieldType} | ${required} | ${hint} |\n`;
          });

          // Add validation rules
          markdown += `\n## Validation Rules\n\n`;
          fields.forEach(field => {
            const validators = field.validators || {};
            const rules = [];
            
            if (validators.required) rules.push('Required');
            if (validators.unique) rules.push('Must be unique');
            if (validators.length) {
              if (validators.length.min) rules.push(`Min length: ${validators.length.min}`);
              if (validators.length.max) rules.push(`Max length: ${validators.length.max}`);
            }
            if (validators.numberRange) {
              if (validators.numberRange.min !== undefined) rules.push(`Min value: ${validators.numberRange.min}`);
              if (validators.numberRange.max !== undefined) rules.push(`Max value: ${validators.numberRange.max}`);
            }
            
            if (rules.length > 0) {
              markdown += `- **${field.label}**: ${rules.join(', ')}\n`;
            }
          });

          // Add example JSON
          markdown += `\n## Example JSON\n\n\`\`\`json\n`;
          const example = {};
          fields.forEach(field => {
            switch (field.fieldType) {
              case 'string':
                example[field.apiKey] = 'Example text';
                break;
              case 'text':
                example[field.apiKey] = 'Example longer text content';
                break;
              case 'integer':
                example[field.apiKey] = 123;
                break;
              case 'float':
                example[field.apiKey] = 123.45;
                break;
              case 'boolean':
                example[field.apiKey] = true;
                break;
              case 'date':
                example[field.apiKey] = '2024-01-01';
                break;
              case 'date_time':
                example[field.apiKey] = '2024-01-01T12:00:00Z';
                break;
              case 'json':
                example[field.apiKey] = { key: 'value' };
                break;
              default:
                example[field.apiKey] = null;
            }
          });
          markdown += JSON.stringify(example, null, 2);
          markdown += `\n\`\`\`\n`;

          // Add API examples
          markdown += `\n## API Usage Examples\n\n`;
          markdown += `### Fetch all ${ctx.itemType.attributes.name} items\n\n`;
          markdown += `\`\`\`javascript\n`;
          markdown += `const items = await client.items.list({\n`;
          markdown += `  filter: { type: '${ctx.itemType.id}' }\n`;
          markdown += `});\n\`\`\`\n\n`;
          
          markdown += `### Create a new ${ctx.itemType.attributes.name}\n\n`;
          markdown += `\`\`\`javascript\n`;
          markdown += `const newItem = await client.items.create({\n`;
          markdown += `  itemType: '${ctx.itemType.id}',\n`;
          markdown += `  ...fieldValues\n`;
          markdown += `});\n\`\`\`\n`;

          // Copy to clipboard or download
          const blob = new Blob([markdown], { type: 'text/markdown' });
          const url = URL.createObjectURL(blob);
          const a = document.createElement('a');
          a.href = url;
          a.download = `${ctx.itemType.attributes.apiKey}-documentation.md`;
          a.click();

          ctx.notice('Documentation generated and downloaded!');
        } catch (error) {
          ctx.alert('Failed to generate documentation: ' + error.message);
        }
      },
    },
  ];
}
```

### 3. Model Migration Tools

Tools for migrating or cloning models:

```typescript
schemaItemTypeDropdownActions(ctx: SchemaItemTypeDropdownActionsCtx) {
  return [
    {
      id: 'clone-model',
      label: 'Clone Model',
      icon: <FaCopy />,
      group: 'migration',
      rank: 30,
      async handler() {
        const newName = await ctx.openModal({
          id: 'textInput',
          title: 'Clone Model',
          width: 'small',
          parameters: {
            label: 'New Model Name',
            placeholder: `${ctx.itemType.attributes.name} Copy`,
          },
        });

        if (!newName) return;

        const client = ctx.createClient();
        ctx.notice('Cloning model...');

        try {
          // Create new item type
          const newItemType = await client.itemTypes.create({
            name: newName,
            apiKey: newName.toLowerCase().replace(/\s+/g, '_'),
            collectionAppearance: ctx.itemType.attributes.collectionAppearance,
            singleton: ctx.itemType.attributes.singleton,
            sortable: ctx.itemType.attributes.sortable,
            modularBlock: ctx.itemType.attributes.modularBlock,
            tree: ctx.itemType.attributes.tree,
            orderingDirection: ctx.itemType.attributes.orderingDirection,
            orderingMeta: ctx.itemType.attributes.orderingMeta,
            draftModeActive: ctx.itemType.attributes.draftModeActive,
            allLocalesRequired: ctx.itemType.attributes.allLocalesRequired,
            titleField: null, // Will be set after fields are created
            hint: `Cloned from ${ctx.itemType.attributes.name}`,
          });

          // Clone all fields
          const fields = await client.fields.list(ctx.itemType.id);
          const fieldMapping = {};

          for (const field of fields) {
            const fieldData = {
              label: field.label,
              apiKey: field.apiKey,
              fieldType: field.fieldType,
              localized: field.localized,
              validators: field.validators || {},
              appearance: field.appearance,
              position: field.position,
              hint: field.hint,
              defaultValue: field.defaultValue,
            };

            // Remove references to old model IDs
            if (fieldData.validators.itemItemType) {
              // Will need to update after all models are cloned
              delete fieldData.validators.itemItemType;
            }
            if (fieldData.validators.richTextBlocks) {
              delete fieldData.validators.richTextBlocks;
            }

            const newField = await client.fields.create(newItemType.id, fieldData);
            fieldMapping[field.id] = newField.id;
          }

          // Update title field if it existed
          if (ctx.itemType.attributes.titleField) {
            const newTitleFieldId = fieldMapping[ctx.itemType.attributes.titleField];
            if (newTitleFieldId) {
              await client.itemTypes.update(newItemType.id, {
                titleField: newTitleFieldId,
              });
            }
          }

          ctx.notice(`Successfully cloned model: ${newName}`);
          
          // Navigate to the new model
          ctx.navigateTo(`/configuration/models/${newItemType.id}`);
        } catch (error) {
          ctx.alert('Failed to clone model: ' + error.message);
        }
      },
    },
    {
      id: 'export-schema',
      label: 'Export Model Schema',
      icon: <FaDownload />,
      group: 'migration',
      rank: 31,
      async handler() {
        const client = ctx.createClient();
        
        try {
          const fields = await client.fields.list(ctx.itemType.id);
          
          const schema = {
            itemType: {
              name: ctx.itemType.attributes.name,
              apiKey: ctx.itemType.attributes.apiKey,
              singleton: ctx.itemType.attributes.singleton,
              sortable: ctx.itemType.attributes.sortable,
              modularBlock: ctx.itemType.attributes.modularBlock,
              tree: ctx.itemType.attributes.tree,
              hint: ctx.itemType.attributes.hint,
            },
            fields: fields.map(field => ({
              label: field.label,
              apiKey: field.apiKey,
              fieldType: field.fieldType,
              localized: field.localized,
              validators: field.validators,
              appearance: field.appearance,
              position: field.position,
              hint: field.hint,
              defaultValue: field.defaultValue,
            })),
          };

          const blob = new Blob([JSON.stringify(schema, null, 2)], { 
            type: 'application/json' 
          });
          const url = URL.createObjectURL(blob);
          const a = document.createElement('a');
          a.href = url;
          a.download = `${ctx.itemType.attributes.apiKey}-schema.json`;
          a.click();

          ctx.notice('Schema exported successfully!');
        } catch (error) {
          ctx.alert('Failed to export schema: ' + error.message);
        }
      },
    },
  ];
}
```

### 4. Model Validation Tools

Add validation and consistency checks:

```typescript
schemaItemTypeDropdownActions(ctx: SchemaItemTypeDropdownActionsCtx) {
  return [
    {
      id: 'validate-model',
      label: 'Validate Model Structure',
      icon: <FaCheckCircle />,
      group: 'validation',
      rank: 40,
      async handler() {
        const client = ctx.createClient();
        const issues = [];

        ctx.notice('Validating model structure...');

        try {
          const fields = await client.fields.list(ctx.itemType.id);

          // Check for title field
          if (!ctx.itemType.attributes.singleton && !ctx.itemType.attributes.titleField) {
            issues.push({
              level: 'warning',
              message: 'No title field set - items will display IDs in listings',
            });
          }

          // Check for required fields without defaults
          fields.forEach(field => {
            if (field.validators?.required && !field.defaultValue) {
              issues.push({
                level: 'info',
                message: `Field "${field.label}" is required but has no default value`,
              });
            }
          });

          // Check for SEO field
          const hasSeoField = fields.some(f => f.fieldType === 'seo');
          if (!hasSeoField) {
            issues.push({
              level: 'info',
              message: 'No SEO field found - consider adding one for better search visibility',
            });
          }

          // Check for localized fields in non-localized project
          if (ctx.site.locales.length === 1) {
            const localizedFields = fields.filter(f => f.localized);
            if (localizedFields.length > 0) {
              issues.push({
                level: 'warning',
                message: `${localizedFields.length} localized fields in single-locale project`,
              });
            }
          }

          // Check for unused link fields
          const linkFields = fields.filter(f => f.fieldType === 'link' || f.fieldType === 'links');
          for (const linkField of linkFields) {
            const validators = linkField.validators?.itemItemType;
            if (validators?.itemTypes?.length === 0) {
              issues.push({
                level: 'error',
                message: `Link field "${linkField.label}" has no allowed models`,
              });
            }
          }

          // Show results
          if (issues.length === 0) {
            ctx.notice('✅ Model structure looks good!');
          } else {
            const report = await ctx.openModal({
              id: 'validationReport',
              title: 'Model Validation Report',
              width: 'medium',
              parameters: { issues },
            });
          }
        } catch (error) {
          ctx.alert('Failed to validate model: ' + error.message);
        }
      },
    },
  ];
}
```

## Best Practices

1. **Clear Labels**: Use descriptive action labels
2. **Meaningful Icons**: Choose icons that represent the action
3. **Error Handling**: Handle API errors gracefully
4. **User Feedback**: Show progress for long-running operations
5. **Confirmation**: Confirm destructive or major changes
6. **Grouping**: Group related actions together
7. **Permissions**: Consider user permissions before showing actions
8. **Navigation**: Navigate to relevant screens after actions
9. **Validation**: Validate inputs before making changes
10. **Idempotency**: Make actions safe to retry

## Related Hooks

- `executeSchemaItemTypeDropdownAction` - Handle the execution of these actions
- `itemFormDropdownActions` - Add actions to item forms
- `fieldDropdownActions` - Add actions to fields

## Important Notes

- Actions appear in the dropdown menu for models in the schema editor
- The handler function is called when the action is clicked
- Use async handlers for operations that might take time
- Always provide feedback about operation success or failure
- Consider the impact of actions on existing content
- Some operations may require page refresh to see changes