# Parameter Validation Patterns

This guide covers parameter validation strategies for DatoCMS APIs, including input validation, type checking, error handling, and best practices for ensuring data integrity and security.

## Input Validation Strategies

### Basic Type Validation

```typescript
import { CmaClient } from '@datocms/cma-client';

// Basic type validation with TypeScript
interface CreateItemParams {
  itemType: string;
  attributes: Record<string, any>;
  meta?: {
    status?: 'draft' | 'published';
    firstPublishedAt?: string;
  };
}

function validateCreateItemParams(params: unknown): CreateItemParams {
  if (!params || typeof params !== 'object') {
    throw new Error('Parameters must be an object');
  }
  
  const obj = params as Record<string, any>;
  
  if (!obj.itemType || typeof obj.itemType !== 'string') {
    throw new Error('itemType is required and must be a string');
  }
  
  if (!obj.attributes || typeof obj.attributes !== 'object') {
    throw new Error('attributes is required and must be an object');
  }
  
  if (obj.meta) {
    if (typeof obj.meta !== 'object') {
      throw new Error('meta must be an object');
    }
    
    if (obj.meta.status && !['draft', 'published'].includes(obj.meta.status)) {
      throw new Error('meta.status must be either "draft" or "published"');
    }
    
    if (obj.meta.firstPublishedAt && !isValidISODate(obj.meta.firstPublishedAt)) {
      throw new Error('meta.firstPublishedAt must be a valid ISO date string');
    }
  }
  
  return obj as CreateItemParams;
}

function isValidISODate(dateString: string): boolean {
  const date = new Date(dateString);
  return !isNaN(date.getTime()) && date.toISOString() === dateString;
}
```

### Runtime Type Checking with Type Guards

```typescript
// Type guard functions for runtime validation
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number' && !isNaN(value);
}

function isBoolean(value: unknown): value is boolean {
  return typeof value === 'boolean';
}

function isArray<T>(value: unknown, itemValidator?: (item: unknown) => item is T): value is T[] {
  if (!Array.isArray(value)) return false;
  if (itemValidator) {
    return value.every(itemValidator);
  }
  return true;
}

function isObject(value: unknown): value is Record<string, unknown> {
  return value !== null && typeof value === 'object' && !Array.isArray(value);
}

// Using type guards in validation
interface FieldValidation {
  required?: boolean;
  unique?: boolean;
  pattern?: string;
  min?: number;
  max?: number;
}

function validateFieldValidation(value: unknown): FieldValidation {
  if (!isObject(value)) {
    throw new Error('Field validation must be an object');
  }
  
  const validation: FieldValidation = {};
  
  if ('required' in value) {
    if (!isBoolean(value.required)) {
      throw new Error('required must be a boolean');
    }
    validation.required = value.required;
  }
  
  if ('unique' in value) {
    if (!isBoolean(value.unique)) {
      throw new Error('unique must be a boolean');
    }
    validation.unique = value.unique;
  }
  
  if ('pattern' in value) {
    if (!isString(value.pattern)) {
      throw new Error('pattern must be a string');
    }
    // Validate regex pattern
    try {
      new RegExp(value.pattern);
    } catch {
      throw new Error('pattern must be a valid regular expression');
    }
    validation.pattern = value.pattern;
  }
  
  if ('min' in value) {
    if (!isNumber(value.min)) {
      throw new Error('min must be a number');
    }
    validation.min = value.min;
  }
  
  if ('max' in value) {
    if (!isNumber(value.max)) {
      throw new Error('max must be a number');
    }
    validation.max = value.max;
  }
  
  // Cross-field validation
  if (validation.min !== undefined && validation.max !== undefined) {
    if (validation.min > validation.max) {
      throw new Error('min cannot be greater than max');
    }
  }
  
  return validation;
}
```

## Schema Validation with Zod

```typescript
import { z } from 'zod';
import { CmaClient } from '@datocms/cma-client';

// Define schemas for different resource types
const ItemTypeSchema = z.object({
  name: z.string().min(1).max(255),
  apiKey: z.string().regex(/^[a-z][a-z0-9_]*$/, {
    message: 'API key must start with a letter and contain only lowercase letters, numbers, and underscores'
  }),
  singleton: z.boolean().optional(),
  sortable: z.boolean().optional(),
  modularBlock: z.boolean().optional(),
  draftModeActive: z.boolean().optional(),
  tree: z.boolean().optional(),
  orderingDirection: z.enum(['asc', 'desc']).optional(),
  orderingMeta: z.enum(['created_at', 'updated_at', 'published_at', 'first_published_at', 'position']).optional(),
  allLocalesRequired: z.boolean().optional(),
  titleField: z.string().optional(),
  imagePreviewField: z.string().optional(),
  excerptField: z.string().optional(),
  orderingField: z.string().optional(),
  hint: z.string().max(1000).optional(),
  inverseRelationships: z.object({
    [z.string()]: z.object({
      fieldApiKey: z.string(),
      itemType: z.string()
    })
  }).optional()
});

const FieldSchema = z.object({
  label: z.string().min(1).max(255),
  apiKey: z.string().regex(/^[a-z][a-z0-9_]*$/),
  fieldType: z.enum([
    'boolean', 'color', 'date', 'datetime', 'file', 'float', 'gallery',
    'integer', 'json', 'lat_lon', 'link', 'links', 'rich_text', 'seo',
    'slug', 'string', 'structured_text', 'video'
  ]),
  localized: z.boolean().optional(),
  validators: z.object({
    required: z.object({}).optional(),
    unique: z.object({}).optional(),
    format: z.object({
      predefinedPattern: z.enum(['email', 'url']).optional(),
      customPattern: z.string().optional()
    }).optional(),
    length: z.object({
      min: z.number().int().min(0).optional(),
      max: z.number().int().positive().optional()
    }).optional(),
    size: z.object({
      min: z.number().positive().optional(),
      max: z.number().positive().optional()
    }).optional(),
    numberRange: z.object({
      min: z.number().optional(),
      max: z.number().optional()
    }).optional(),
    itemItemType: z.object({
      itemTypes: z.array(z.string()).min(1)
    }).optional(),
    richTextBlocks: z.object({
      itemTypes: z.array(z.string()).optional()
    }).optional(),
    extension: z.object({
      predefinedList: z.enum(['jpg', 'png', 'gif', 'pdf', 'doc', 'docx']).optional(),
      customRegex: z.string().optional()
    }).optional()
  }).optional(),
  appearance: z.object({
    editor: z.string().optional(),
    parameters: z.record(z.unknown()).optional(),
    addons: z.array(z.object({
      id: z.string(),
      parameters: z.record(z.unknown()).optional()
    })).optional()
  }).optional(),
  position: z.number().int().min(1).optional(),
  hint: z.string().max(1000).optional(),
  defaultValue: z.unknown().optional()
});

// Using Zod schemas for validation
async function createItemTypeWithValidation(client: CmaClient, data: unknown) {
  try {
    // Parse and validate input
    const validated = ItemTypeSchema.parse(data);
    
    // Additional business logic validation
    if (validated.tree && validated.sortable) {
      throw new Error('An item type cannot be both tree and sortable');
    }
    
    if (validated.singleton && validated.sortable) {
      throw new Error('A singleton item type cannot be sortable');
    }
    
    // Create the item type
    const itemType = await client.itemTypes.create(validated);
    
    return itemType;
  } catch (error) {
    if (error instanceof z.ZodError) {
      // Format Zod errors for better readability
      const formattedErrors = error.errors.map(err => ({
        path: err.path.join('.'),
        message: err.message
      }));
      
      throw new Error(`Validation failed: ${JSON.stringify(formattedErrors, null, 2)}`);
    }
    
    throw error;
  }
}

// Nested validation with Zod
const ItemSchema = z.object({
  itemType: z.string(),
  attributes: z.record(z.unknown()),
  meta: z.object({
    status: z.enum(['draft', 'published']).optional(),
    firstPublishedAt: z.string().datetime().optional(),
    createdAt: z.string().datetime().optional(),
    updatedAt: z.string().datetime().optional()
  }).optional(),
  relationships: z.object({
    itemType: z.object({
      data: z.object({
        type: z.literal('item_type'),
        id: z.string()
      })
    }),
    creator: z.object({
      data: z.object({
        type: z.literal('user'),
        id: z.string()
      })
    }).optional()
  }).optional()
});

// Dynamic field validation based on field type
function createFieldValidator(field: { fieldType: string; validators?: any }) {
  switch (field.fieldType) {
    case 'string':
      return z.string().refine(
        val => {
          if (field.validators?.length?.min && val.length < field.validators.length.min) {
            return false;
          }
          if (field.validators?.length?.max && val.length > field.validators.length.max) {
            return false;
          }
          if (field.validators?.format?.customPattern) {
            return new RegExp(field.validators.format.customPattern).test(val);
          }
          return true;
        },
        { message: 'String validation failed' }
      );
      
    case 'integer':
      return z.number().int().refine(
        val => {
          if (field.validators?.numberRange?.min && val < field.validators.numberRange.min) {
            return false;
          }
          if (field.validators?.numberRange?.max && val > field.validators.numberRange.max) {
            return false;
          }
          return true;
        },
        { message: 'Integer validation failed' }
      );
      
    case 'boolean':
      return z.boolean();
      
    case 'json':
      return z.unknown().refine(
        val => {
          try {
            JSON.stringify(val);
            return true;
          } catch {
            return false;
          }
        },
        { message: 'Must be valid JSON' }
      );
      
    default:
      return z.unknown();
  }
}
```

## Schema Validation with Joi

```typescript
import Joi from 'joi';
import { CmaClient } from '@datocms/cma-client';

// Define Joi schemas
const ItemTypeJoiSchema = Joi.object({
  name: Joi.string().min(1).max(255).required(),
  apiKey: Joi.string().pattern(/^[a-z][a-z0-9_]*$/).required(),
  singleton: Joi.boolean(),
  sortable: Joi.boolean(),
  modularBlock: Joi.boolean(),
  draftModeActive: Joi.boolean(),
  tree: Joi.boolean(),
  orderingDirection: Joi.string().valid('asc', 'desc'),
  orderingMeta: Joi.string().valid('created_at', 'updated_at', 'published_at', 'first_published_at', 'position'),
  allLocalesRequired: Joi.boolean(),
  titleField: Joi.string(),
  imagePreviewField: Joi.string(),
  excerptField: Joi.string(),
  orderingField: Joi.string(),
  hint: Joi.string().max(1000),
  inverseRelationships: Joi.object().pattern(
    Joi.string(),
    Joi.object({
      fieldApiKey: Joi.string().required(),
      itemType: Joi.string().required()
    })
  )
}).custom((value, helpers) => {
  // Custom validation rules
  if (value.tree && value.sortable) {
    return helpers.error('custom.treeSortable');
  }
  if (value.singleton && value.sortable) {
    return helpers.error('custom.singletonSortable');
  }
  return value;
}, 'Item Type Validation').messages({
  'custom.treeSortable': 'An item type cannot be both tree and sortable',
  'custom.singletonSortable': 'A singleton item type cannot be sortable'
});

// Upload validation schema
const UploadJoiSchema = Joi.object({
  path: Joi.alternatives().try(
    Joi.string().uri(),
    Joi.binary()
  ).required(),
  defaultFieldMetadata: Joi.object({
    en: Joi.object({
      alt: Joi.string(),
      title: Joi.string(),
      customData: Joi.object(),
      focalPoint: Joi.object({
        x: Joi.number().min(0).max(1).required(),
        y: Joi.number().min(0).max(1).required()
      })
    })
  }),
  skipCreationIfAlreadyExists: Joi.boolean(),
  onProgress: Joi.function(),
  filename: Joi.string(),
  tags: Joi.array().items(Joi.string()),
  notes: Joi.string().max(512),
  author: Joi.string().max(255),
  copyright: Joi.string().max(255)
});

// Using Joi for validation with custom error messages
async function validateAndCreateUpload(client: CmaClient, data: unknown) {
  const { error, value } = UploadJoiSchema.validate(data, {
    abortEarly: false,
    convert: true,
    stripUnknown: true
  });
  
  if (error) {
    const formattedErrors = error.details.map(detail => ({
      path: detail.path.join('.'),
      message: detail.message,
      type: detail.type
    }));
    
    throw new Error(`Upload validation failed:\n${formattedErrors.map(e => 
      `  - ${e.path}: ${e.message}`
    ).join('\n')}`);
  }
  
  // Additional file validation
  if (value.path && typeof value.path === 'string' && !value.path.startsWith('http')) {
    // Local file validation
    const fs = await import('fs');
    const path = await import('path');
    
    if (!fs.existsSync(value.path)) {
      throw new Error(`File not found: ${value.path}`);
    }
    
    const stats = fs.statSync(value.path);
    const maxSize = 100 * 1024 * 1024; // 100MB
    
    if (stats.size > maxSize) {
      throw new Error(`File size exceeds maximum allowed size of ${maxSize} bytes`);
    }
    
    // Validate file extension
    const ext = path.extname(value.path).toLowerCase();
    const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.svg', '.pdf', '.mp4', '.webm'];
    
    if (!allowedExtensions.includes(ext)) {
      throw new Error(`File extension ${ext} is not allowed. Allowed extensions: ${allowedExtensions.join(', ')}`);
    }
  }
  
  return client.uploads.create(value);
}
```

## API-Specific Validation Rules

### Field Type-Specific Validation

```typescript
import { CmaClient } from '@datocms/cma-client';

// Field type validation map
const FIELD_TYPE_VALIDATORS = {
  string: {
    validators: ['required', 'unique', 'format', 'length'],
    defaultValue: (val: unknown) => typeof val === 'string',
    appearance: {
      editors: ['single_line', 'markdown', 'wysiwyg', 'textarea'],
      parameters: {
        single_line: {
          heading: z.boolean().optional(),
          placeholder: z.string().optional()
        },
        markdown: {
          toolbar: z.array(z.string()).optional()
        },
        wysiwyg: {
          toolbar: z.array(z.string()).optional()
        },
        textarea: {
          rows: z.number().int().min(1).optional()
        }
      }
    }
  },
  
  boolean: {
    validators: ['required'],
    defaultValue: (val: unknown) => typeof val === 'boolean',
    appearance: {
      editors: ['boolean'],
      parameters: {
        boolean: {
          label: z.string().optional()
        }
      }
    }
  },
  
  integer: {
    validators: ['required', 'unique', 'numberRange'],
    defaultValue: (val: unknown) => Number.isInteger(val),
    appearance: {
      editors: ['integer'],
      parameters: {
        integer: {
          placeholder: z.string().optional()
        }
      }
    }
  },
  
  float: {
    validators: ['required', 'unique', 'numberRange'],
    defaultValue: (val: unknown) => typeof val === 'number',
    appearance: {
      editors: ['float'],
      parameters: {
        float: {
          placeholder: z.string().optional()
        }
      }
    }
  },
  
  date: {
    validators: ['required', 'dateRange'],
    defaultValue: (val: unknown) => {
      if (typeof val !== 'string') return false;
      const date = new Date(val);
      return !isNaN(date.getTime()) && val.match(/^\d{4}-\d{2}-\d{2}$/);
    },
    appearance: {
      editors: ['date_picker'],
      parameters: {}
    }
  },
  
  datetime: {
    validators: ['required', 'dateTimeRange'],
    defaultValue: (val: unknown) => {
      if (typeof val !== 'string') return false;
      const date = new Date(val);
      return !isNaN(date.getTime());
    },
    appearance: {
      editors: ['date_time_picker'],
      parameters: {}
    }
  },
  
  link: {
    validators: ['required', 'itemItemType'],
    defaultValue: (val: unknown) => {
      return typeof val === 'string' || (typeof val === 'object' && val !== null);
    },
    appearance: {
      editors: ['link_select', 'link_embed'],
      parameters: {}
    }
  },
  
  links: {
    validators: ['required', 'size', 'itemsItemType'],
    defaultValue: (val: unknown) => Array.isArray(val),
    appearance: {
      editors: ['links_select', 'links_embed'],
      parameters: {}
    }
  },
  
  json: {
    validators: ['required'],
    defaultValue: (val: unknown) => {
      try {
        JSON.stringify(val);
        return true;
      } catch {
        return false;
      }
    },
    appearance: {
      editors: ['json'],
      parameters: {}
    }
  },
  
  structured_text: {
    validators: ['required', 'structuredTextBlocks', 'structuredTextLinks'],
    defaultValue: (val: unknown) => {
      return typeof val === 'object' && val !== null && 'schema' in val;
    },
    appearance: {
      editors: ['structured_text'],
      parameters: {
        structured_text: {
          marks: z.array(z.string()).optional(),
          nodes: z.array(z.string()).optional(),
          blocks: z.array(z.string()).optional()
        }
      }
    }
  }
};

// Validate field based on its type
function validateFieldByType(field: { fieldType: string; validators?: any }, value: unknown): void {
  const typeValidator = FIELD_TYPE_VALIDATORS[field.fieldType as keyof typeof FIELD_TYPE_VALIDATORS];
  
  if (!typeValidator) {
    throw new Error(`Unknown field type: ${field.fieldType}`);
  }
  
  // Check if value matches expected type
  if (value !== null && value !== undefined && !typeValidator.defaultValue(value)) {
    throw new Error(`Invalid value type for field type ${field.fieldType}`);
  }
  
  // Validate against field validators
  if (field.validators) {
    // Check for unsupported validators
    const supportedValidators = typeValidator.validators;
    const providedValidators = Object.keys(field.validators);
    
    const unsupportedValidators = providedValidators.filter(
      v => !supportedValidators.includes(v)
    );
    
    if (unsupportedValidators.length > 0) {
      throw new Error(
        `Unsupported validators for field type ${field.fieldType}: ${unsupportedValidators.join(', ')}`
      );
    }
  }
}
```

### Relationship Validation

```typescript
interface RelationshipValidator {
  validateItemReference(itemId: string, allowedTypes?: string[]): Promise<boolean>;
  validateCircularReference(fromId: string, toId: string, fieldPath: string): Promise<boolean>;
  validateRequiredRelationships(item: any, itemType: any): Promise<void>;
}

class DatoCMSRelationshipValidator implements RelationshipValidator {
  constructor(private client: CmaClient) {}
  
  async validateItemReference(itemId: string, allowedTypes?: string[]): Promise<boolean> {
    try {
      const item = await this.client.items.find(itemId);
      
      if (allowedTypes && allowedTypes.length > 0) {
        const itemType = await this.client.itemTypes.find(item.relationships.item_type.data.id);
        
        if (!allowedTypes.includes(itemType.api_key)) {
          throw new Error(
            `Item type "${itemType.api_key}" is not allowed. Allowed types: ${allowedTypes.join(', ')}`
          );
        }
      }
      
      return true;
    } catch (error) {
      if (error instanceof Error && error.message.includes('NOT_FOUND')) {
        throw new Error(`Referenced item ${itemId} does not exist`);
      }
      throw error;
    }
  }
  
  async validateCircularReference(
    fromId: string,
    toId: string,
    fieldPath: string
  ): Promise<boolean> {
    if (fromId === toId) {
      throw new Error(`Circular reference detected: item cannot reference itself in field ${fieldPath}`);
    }
    
    // Check for deeper circular references
    const visited = new Set<string>([fromId]);
    const queue = [toId];
    
    while (queue.length > 0) {
      const currentId = queue.shift()!;
      
      if (visited.has(currentId)) {
        throw new Error(
          `Circular reference detected: ${fromId} -> ... -> ${currentId} in field ${fieldPath}`
        );
      }
      
      visited.add(currentId);
      
      try {
        const item = await this.client.items.find(currentId);
        
        // Extract all referenced items from link/links fields
        const references = this.extractReferences(item);
        queue.push(...references);
      } catch (error) {
        // Item might not exist yet (during creation)
        break;
      }
    }
    
    return true;
  }
  
  private extractReferences(item: any): string[] {
    const references: string[] = [];
    
    for (const [fieldKey, fieldValue] of Object.entries(item.attributes)) {
      if (typeof fieldValue === 'string' && fieldValue.match(/^[0-9]+$/)) {
        // Single link field
        references.push(fieldValue);
      } else if (Array.isArray(fieldValue)) {
        // Links field
        references.push(...fieldValue.filter(v => typeof v === 'string' && v.match(/^[0-9]+$/)));
      }
    }
    
    return references;
  }
  
  async validateRequiredRelationships(item: any, itemType: any): Promise<void> {
    const fields = await this.client.fields.list(itemType.id);
    
    for (const field of fields) {
      if (field.validators?.required && (field.field_type === 'link' || field.field_type === 'links')) {
        const value = item.attributes[field.api_key];
        
        if (field.field_type === 'link' && !value) {
          throw new Error(`Required link field "${field.api_key}" is missing`);
        }
        
        if (field.field_type === 'links' && (!value || !Array.isArray(value) || value.length === 0)) {
          throw new Error(`Required links field "${field.api_key}" must have at least one item`);
        }
      }
    }
  }
}
```

## Sanitization Patterns

```typescript
// Input sanitization utilities
class InputSanitizer {
  // Remove potentially dangerous HTML/scripts
  static sanitizeHtml(input: string): string {
    // Basic HTML entity encoding
    return input
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#x27;')
      .replace(/\//g, '&#x2F;');
  }
  
  // Sanitize API keys
  static sanitizeApiKey(input: string): string {
    // Convert to lowercase and replace invalid characters
    return input
      .toLowerCase()
      .replace(/[^a-z0-9_]/g, '_')
      .replace(/^[0-9_]/, 'field_') // Ensure it starts with a letter
      .replace(/_+/g, '_') // Remove multiple underscores
      .replace(/_$/, ''); // Remove trailing underscore
  }
  
  // Sanitize file names
  static sanitizeFilename(input: string): string {
    // Remove path traversal attempts and invalid characters
    return input
      .replace(/\.\./g, '')
      .replace(/[\/\\]/g, '_')
      .replace(/[^a-zA-Z0-9._-]/g, '_')
      .replace(/_+/g, '_')
      .substring(0, 255); // Limit length
  }
  
  // Sanitize URLs
  static sanitizeUrl(input: string): string {
    try {
      const url = new URL(input);
      
      // Only allow http and https protocols
      if (!['http:', 'https:'].includes(url.protocol)) {
        throw new Error('Invalid protocol');
      }
      
      // Remove credentials from URL
      url.username = '';
      url.password = '';
      
      return url.toString();
    } catch {
      throw new Error('Invalid URL format');
    }
  }
  
  // Sanitize JSON
  static sanitizeJson(input: unknown): unknown {
    // Deep clone to prevent prototype pollution
    const sanitized = JSON.parse(JSON.stringify(input));
    
    // Remove dangerous keys
    const dangerousKeys = ['__proto__', 'constructor', 'prototype'];
    
    function removeDangerousKeys(obj: any): void {
      if (typeof obj !== 'object' || obj === null) return;
      
      for (const key of dangerousKeys) {
        delete obj[key];
      }
      
      for (const value of Object.values(obj)) {
        if (typeof value === 'object') {
          removeDangerousKeys(value);
        }
      }
    }
    
    removeDangerousKeys(sanitized);
    return sanitized;
  }
  
  // Sanitize integers
  static sanitizeInteger(input: unknown, min?: number, max?: number): number {
    const num = Number(input);
    
    if (!Number.isInteger(num)) {
      throw new Error('Value must be an integer');
    }
    
    if (min !== undefined && num < min) {
      throw new Error(`Value must be at least ${min}`);
    }
    
    if (max !== undefined && num > max) {
      throw new Error(`Value must be at most ${max}`);
    }
    
    return num;
  }
}

// Using sanitization in practice
async function createSanitizedItem(client: CmaClient, input: any) {
  const sanitized = {
    itemType: input.itemType,
    attributes: {}
  };
  
  // Get item type to know field types
  const itemType = await client.itemTypes.find(input.itemType);
  const fields = await client.fields.list(itemType.id);
  
  // Sanitize each field based on its type
  for (const field of fields) {
    const value = input.attributes[field.api_key];
    
    if (value === undefined || value === null) continue;
    
    switch (field.field_type) {
      case 'string':
      case 'text':
        sanitized.attributes[field.api_key] = InputSanitizer.sanitizeHtml(String(value));
        break;
        
      case 'slug':
        sanitized.attributes[field.api_key] = InputSanitizer.sanitizeApiKey(String(value));
        break;
        
      case 'integer':
        sanitized.attributes[field.api_key] = InputSanitizer.sanitizeInteger(value);
        break;
        
      case 'json':
        sanitized.attributes[field.api_key] = InputSanitizer.sanitizeJson(value);
        break;
        
      default:
        sanitized.attributes[field.api_key] = value;
    }
  }
  
  return client.items.create(sanitized);
}
```

## Validation Error Handling

```typescript
// Custom validation error class
class ValidationError extends Error {
  constructor(
    message: string,
    public field?: string,
    public code?: string,
    public details?: any
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

// Validation error formatter
class ValidationErrorFormatter {
  static format(error: unknown): { errors: Array<{ field?: string; message: string; code?: string }> } {
    if (error instanceof z.ZodError) {
      return {
        errors: error.errors.map(err => ({
          field: err.path.join('.'),
          message: err.message,
          code: err.code
        }))
      };
    }
    
    if (error instanceof ValidationError) {
      return {
        errors: [{
          field: error.field,
          message: error.message,
          code: error.code
        }]
      };
    }
    
    if (error instanceof Error && error.message.includes('VALIDATION_ERROR')) {
      // Parse DatoCMS validation errors
      try {
        const parsed = JSON.parse(error.message);
        return {
          errors: parsed.data.map((err: any) => ({
            field: err.attributes.details.field,
            message: err.attributes.details.message,
            code: err.attributes.code
          }))
        };
      } catch {
        // Fallback for unparseable errors
      }
    }
    
    return {
      errors: [{
        message: error instanceof Error ? error.message : 'Unknown error'
      }]
    };
  }
}

// Validation middleware
async function withValidation<T>(
  validator: (input: unknown) => T | Promise<T>,
  handler: (validated: T) => Promise<any>
): Promise<any> {
  try {
    const validated = await validator(arguments[1]);
    return await handler(validated);
  } catch (error) {
    const formatted = ValidationErrorFormatter.format(error);
    
    // Log validation errors
    console.error('Validation failed:', formatted);
    
    // Re-throw with formatted error
    throw new Error(JSON.stringify(formatted, null, 2));
  }
}

// Example usage
const createItemWithValidation = withValidation(
  async (input: unknown) => {
    // Validate input
    return ItemSchema.parse(input);
  },
  async (validated) => {
    // Process validated input
    return client.items.create(validated);
  }
);
```

## Client-Side vs Server-Side Validation

```typescript
// Shared validation rules
const SharedValidationRules = {
  email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
  url: /^https?:\/\/.+/,
  slug: /^[a-z0-9]+(?:-[a-z0-9]+)*$/,
  apiKey: /^[a-z][a-z0-9_]*$/
};

// Client-side validation (browser/React)
class ClientSideValidator {
  static validateEmail(value: string): string | null {
    if (!SharedValidationRules.email.test(value)) {
      return 'Please enter a valid email address';
    }
    return null;
  }
  
  static validateRequired(value: any, fieldName: string): string | null {
    if (value === null || value === undefined || value === '') {
      return `${fieldName} is required`;
    }
    return null;
  }
  
  static validateLength(
    value: string,
    min?: number,
    max?: number,
    fieldName?: string
  ): string | null {
    if (min && value.length < min) {
      return `${fieldName} must be at least ${min} characters`;
    }
    if (max && value.length > max) {
      return `${fieldName} must be at most ${max} characters`;
    }
    return null;
  }
  
  static async validateUnique(
    value: string,
    fieldApiKey: string,
    itemTypeId: string,
    client: CmaClient,
    excludeId?: string
  ): Promise<string | null> {
    try {
      const items = await client.items.list({
        filter: {
          type: 'item',
          fields: {
            [fieldApiKey]: {
              eq: value
            }
          }
        },
        page: {
          limit: 1
        }
      });
      
      const conflictingItem = items.find(item => item.id !== excludeId);
      
      if (conflictingItem) {
        return `This value is already taken`;
      }
      
      return null;
    } catch {
      // Unable to validate uniqueness on client
      return null;
    }
  }
}

// Server-side validation
class ServerSideValidator {
  constructor(private client: CmaClient) {}
  
  async validateComplete(itemData: any, itemType: any): Promise<void> {
    const errors: ValidationError[] = [];
    const fields = await this.client.fields.list(itemType.id);
    
    for (const field of fields) {
      const value = itemData.attributes[field.api_key];
      
      // Required validation
      if (field.validators?.required) {
        if (value === null || value === undefined || value === '') {
          errors.push(new ValidationError(
            `${field.label} is required`,
            field.api_key,
            'REQUIRED_FIELD'
          ));
        }
      }
      
      // Type-specific validation
      if (value !== null && value !== undefined) {
        try {
          await this.validateFieldValue(field, value);
        } catch (error) {
          if (error instanceof ValidationError) {
            errors.push(error);
          }
        }
      }
    }
    
    // Check for missing required fields
    const providedFields = new Set(Object.keys(itemData.attributes));
    const requiredFields = fields.filter(f => f.validators?.required);
    
    for (const field of requiredFields) {
      if (!providedFields.has(field.api_key)) {
        errors.push(new ValidationError(
          `Missing required field: ${field.label}`,
          field.api_key,
          'MISSING_FIELD'
        ));
      }
    }
    
    if (errors.length > 0) {
      throw new AggregateValidationError('Validation failed', errors);
    }
  }
  
  private async validateFieldValue(field: any, value: any): Promise<void> {
    // Implementation of field-specific validation
    validateFieldByType(field, value);
  }
}

class AggregateValidationError extends Error {
  constructor(message: string, public errors: ValidationError[]) {
    super(message);
    this.name = 'AggregateValidationError';
  }
}
```

## Performance Considerations

```typescript
// Optimized validation strategies
class OptimizedValidator {
  private cache = new Map<string, any>();
  
  // Cache validation schemas
  private getSchema(key: string, factory: () => any): any {
    if (!this.cache.has(key)) {
      this.cache.set(key, factory());
    }
    return this.cache.get(key);
  }
  
  // Batch validation for multiple items
  async batchValidate<T>(
    items: T[],
    validator: (item: T) => Promise<void>
  ): Promise<{ valid: T[]; invalid: Array<{ item: T; error: Error }> }> {
    const results = await Promise.allSettled(
      items.map(item => validator(item).then(() => ({ item, error: null })))
    );
    
    const valid: T[] = [];
    const invalid: Array<{ item: T; error: Error }> = [];
    
    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        valid.push(items[index]);
      } else {
        invalid.push({
          item: items[index],
          error: result.reason
        });
      }
    });
    
    return { valid, invalid };
  }
  
  // Lazy validation - only validate changed fields
  validateChangedFields(
    original: Record<string, any>,
    updated: Record<string, any>,
    fieldValidators: Record<string, (value: any) => void>
  ): void {
    const changedFields = Object.keys(updated).filter(
      key => JSON.stringify(original[key]) !== JSON.stringify(updated[key])
    );
    
    for (const field of changedFields) {
      if (fieldValidators[field]) {
        fieldValidators[field](updated[field]);
      }
    }
  }
  
  // Streaming validation for large datasets
  async *validateStream<T>(
    items: AsyncIterable<T>,
    validator: (item: T) => Promise<void>
  ): AsyncGenerator<{ item: T; valid: boolean; error?: Error }> {
    for await (const item of items) {
      try {
        await validator(item);
        yield { item, valid: true };
      } catch (error) {
        yield { item, valid: false, error: error as Error };
      }
    }
  }
}

// Memory-efficient validation for large objects
class StreamingValidator {
  async validateLargeJson(stream: NodeJS.ReadableStream): Promise<void> {
    const parser = require('stream-json');
    const { chain } = require('stream-chain');
    const { parser: jsonParser } = require('stream-json');
    const { streamValues } = require('stream-json/streamers/StreamValues');
    
    let itemCount = 0;
    const errors: Error[] = [];
    
    const pipeline = chain([
      stream,
      jsonParser(),
      streamValues(),
      async (data: any) => {
        itemCount++;
        
        try {
          // Validate each item as it streams
          await this.validateItem(data.value);
        } catch (error) {
          errors.push(new Error(
            `Validation error at item ${itemCount}: ${(error as Error).message}`
          ));
          
          // Stop processing after too many errors
          if (errors.length > 100) {
            throw new Error('Too many validation errors');
          }
        }
      }
    ]);
    
    await new Promise((resolve, reject) => {
      pipeline.on('end', resolve);
      pipeline.on('error', reject);
    });
    
    if (errors.length > 0) {
      throw new AggregateValidationError('Validation errors found', errors as ValidationError[]);
    }
  }
  
  private async validateItem(item: any): Promise<void> {
    // Implement item validation
  }
}
```

## Security Implications

```typescript
// Security-focused validation
class SecurityValidator {
  // Prevent NoSQL injection
  static validateMongoQuery(query: any): void {
    const dangerous = ['$where', '$regex', '$options', '$expr'];
    
    function checkObject(obj: any, path: string = ''): void {
      if (typeof obj !== 'object' || obj === null) return;
      
      for (const [key, value] of Object.entries(obj)) {
        const currentPath = path ? `${path}.${key}` : key;
        
        if (dangerous.includes(key)) {
          throw new ValidationError(
            `Dangerous operator "${key}" not allowed`,
            currentPath,
            'DANGEROUS_OPERATOR'
          );
        }
        
        if (typeof value === 'object') {
          checkObject(value, currentPath);
        }
      }
    }
    
    checkObject(query);
  }
  
  // Validate file uploads
  static async validateFileUpload(
    file: { path: string; mimetype: string; size: number }
  ): Promise<void> {
    // Check file size
    const maxSize = 100 * 1024 * 1024; // 100MB
    if (file.size > maxSize) {
      throw new ValidationError(
        'File size exceeds maximum allowed',
        'file',
        'FILE_TOO_LARGE',
        { maxSize, actualSize: file.size }
      );
    }
    
    // Validate MIME type
    const allowedMimeTypes = [
      'image/jpeg',
      'image/png',
      'image/gif',
      'image/svg+xml',
      'application/pdf',
      'video/mp4',
      'video/webm'
    ];
    
    if (!allowedMimeTypes.includes(file.mimetype)) {
      throw new ValidationError(
        'File type not allowed',
        'file',
        'INVALID_MIME_TYPE',
        { allowed: allowedMimeTypes, actual: file.mimetype }
      );
    }
    
    // Check actual file content (magic numbers)
    const fileType = await import('file-type');
    const fs = await import('fs');
    
    const buffer = await fs.promises.readFile(file.path);
    const detectedType = await fileType.fromBuffer(buffer);
    
    if (!detectedType || detectedType.mime !== file.mimetype) {
      throw new ValidationError(
        'File content does not match declared type',
        'file',
        'MIME_TYPE_MISMATCH',
        { declared: file.mimetype, detected: detectedType?.mime }
      );
    }
  }
  
  // Rate limiting validation
  static createRateLimitedValidator(
    validator: Function,
    maxCalls: number,
    windowMs: number
  ): Function {
    const calls = new Map<string, number[]>();
    
    return async function rateLimited(identifier: string, ...args: any[]) {
      const now = Date.now();
      const userCalls = calls.get(identifier) || [];
      
      // Remove old calls outside the window
      const recentCalls = userCalls.filter(time => now - time < windowMs);
      
      if (recentCalls.length >= maxCalls) {
        throw new ValidationError(
          'Rate limit exceeded',
          undefined,
          'RATE_LIMIT_EXCEEDED',
          { limit: maxCalls, window: windowMs }
        );
      }
      
      recentCalls.push(now);
      calls.set(identifier, recentCalls);
      
      return validator(...args);
    };
  }
}
```

## Best Practices

### 1. Fail Fast, Fail Clearly

```typescript
// Good: Clear error messages with context
function validateItemAttributes(attributes: any, fields: any[]): void {
  const errors: string[] = [];
  
  for (const field of fields) {
    if (field.validators?.required && !attributes[field.api_key]) {
      errors.push(`Field "${field.label}" (${field.api_key}) is required`);
    }
  }
  
  if (errors.length > 0) {
    throw new Error(`Validation failed:\n${errors.join('\n')}`);
  }
}

// Bad: Generic error messages
function validateItemAttributesBad(attributes: any, fields: any[]): void {
  for (const field of fields) {
    if (field.validators?.required && !attributes[field.api_key]) {
      throw new Error('Validation failed');
    }
  }
}
```

### 2. Validate at the Right Layer

```typescript
// Client-side: Immediate feedback
const ItemForm: React.FC = () => {
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  const validateField = (name: string, value: any) => {
    // Quick client-side validation
    const error = ClientSideValidator.validateRequired(value, name);
    setErrors(prev => ({ ...prev, [name]: error || '' }));
  };
  
  return (
    <form>
      <input
        onChange={(e) => validateField('title', e.target.value)}
        className={errors.title ? 'error' : ''}
      />
      {errors.title && <span className="error-message">{errors.title}</span>}
    </form>
  );
};

// Server-side: Complete validation
async function createItem(data: any) {
  // Comprehensive server-side validation
  const validator = new ServerSideValidator(client);
  await validator.validateComplete(data, itemType);
  
  // Only create after validation passes
  return client.items.create(data);
}
```

### 3. Use TypeScript for Compile-Time Safety

```typescript
// Define strict types
interface StrictItemCreate {
  itemType: string;
  attributes: Record<string, unknown>;
  meta?: {
    status: 'draft' | 'published';
    firstPublishedAt?: string;
  };
}

// TypeScript will catch errors at compile time
function createTypedItem(data: StrictItemCreate) {
  return client.items.create(data);
}

// This will cause a TypeScript error
createTypedItem({
  itemType: 'article',
  attributes: { title: 'Test' },
  meta: { status: 'invalid' } // TypeScript error!
});
```

### 4. Validate Early in the Request Lifecycle

```typescript
// Express middleware for validation
const validateRequest = (schema: any) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,
      stripUnknown: true
    });
    
    if (error) {
      return res.status(400).json({
        error: 'Validation failed',
        details: error.details
      });
    }
    
    // Replace body with validated/sanitized data
    req.body = value;
    next();
  };
};

// Use in routes
app.post('/api/items', validateRequest(ItemSchema), async (req, res) => {
  // req.body is already validated and sanitized
  const item = await client.items.create(req.body);
  res.json(item);
});
```

### 5. Document Validation Rules

```typescript
/**
 * Creates a new item with validation
 * 
 * @param data - Item data
 * @param data.itemType - The item type ID (required)
 * @param data.attributes - Item attributes (required)
 *   - Must match the field definitions of the item type
 *   - Required fields must be provided
 *   - Field values must match their configured validators
 * @param data.meta - Optional metadata
 * @param data.meta.status - 'draft' or 'published' (default: 'draft')
 * @param data.meta.firstPublishedAt - ISO 8601 datetime string
 * 
 * @throws {ValidationError} If validation fails
 * @throws {Error} If API request fails
 * 
 * @example
 * ```typescript
 * const item = await createValidatedItem({
 *   itemType: '123456',
 *   attributes: {
 *     title: 'My Article',
 *     slug: 'my-article',
 *     content: 'Article content...'
 *   },
 *   meta: {
 *     status: 'published'
 *   }
 * });
 * ```
 */
async function createValidatedItem(data: any) {
  // Implementation
}
```

## Integration with TypeScript Types

```typescript
// Generate TypeScript types from validation schemas
import { z } from 'zod';

// Define schema
const ArticleSchema = z.object({
  title: z.string().min(1).max(255),
  slug: z.string().regex(/^[a-z0-9-]+$/),
  content: z.string(),
  publishedAt: z.string().datetime().optional(),
  author: z.object({
    name: z.string(),
    email: z.string().email()
  }),
  tags: z.array(z.string()).optional(),
  metadata: z.record(z.unknown()).optional()
});

// Infer TypeScript type from schema
type Article = z.infer<typeof ArticleSchema>;

// Use in functions with full type safety
async function createArticle(data: Article): Promise<void> {
  // data is fully typed
  const validated = ArticleSchema.parse(data);
  await client.items.create({
    itemType: 'article',
    attributes: validated
  });
}

// Generate validation from TypeScript types
type ItemTypeFromAPI = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    modular_block: boolean;
  };
};

// Create runtime validator from type
const ItemTypeValidator = z.object({
  id: z.string(),
  type: z.literal('item_type'),
  attributes: z.object({
    name: z.string(),
    api_key: z.string(),
    singleton: z.boolean(),
    sortable: z.boolean(),
    modular_block: z.boolean()
  })
}) satisfies z.ZodType<ItemTypeFromAPI>;
```

## Summary

Parameter validation is crucial for maintaining data integrity, security, and providing good user experience in DatoCMS applications. Key points:

1. **Layer your validation**: Use client-side for immediate feedback, server-side for security
2. **Choose the right tool**: Zod for TypeScript integration, Joi for complex validation
3. **Sanitize all inputs**: Especially for user-generated content
4. **Handle errors gracefully**: Provide clear, actionable error messages
5. **Consider performance**: Use caching, batch validation, and streaming for large datasets
6. **Think security first**: Validate to prevent injection attacks and data corruption
7. **Document thoroughly**: Make validation rules clear for API consumers
8. **Use TypeScript**: Leverage compile-time type checking alongside runtime validation

Remember that validation is not just about preventing errors—it's about guiding users to provide correct data and protecting your system from malicious input.