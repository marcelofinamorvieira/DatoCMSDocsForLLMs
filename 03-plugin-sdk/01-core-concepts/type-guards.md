# Type Guards

This guide covers runtime type checking patterns for the DatoCMS Plugin SDK. Type guards help you safely narrow union types and validate data at runtime.

## Table of Contents

- [Overview](#overview)
- [User Type Guards](#user-type-guards)
- [Field Type Guards](#field-type-guards)
- [Item and Upload Guards](#item-and-upload-guards)
- [Dropdown Action Guards](#dropdown-action-guards)
- [Built-in Type Guards](#built-in-type-guards)
- [Creating Custom Type Guards](#creating-custom-type-guards)
- [Best Practices](#best-practices)

## Overview

Type guards are functions that perform runtime checks to help TypeScript narrow types. They're essential when working with union types or validating external data.

### Basic Pattern

```typescript
function isType(value: unknown): value is SpecificType {
  return /* runtime check */;
}
```

## User Type Guards

The `currentUser` property in context objects can be one of several types. Here's how to determine which type you're working with:

### User vs SsoUser vs Account vs Organization

```typescript
import type { User, SsoUser, Account, Organization } from 'datocms-plugin-sdk';

// Type guard for regular User
function isUser(user: User | SsoUser | Account | Organization): user is User {
  return user.type === 'user';
}

// Type guard for SSO User
function isSsoUser(user: User | SsoUser | Account | Organization): user is SsoUser {
  return user.type === 'sso_user';
}

// Type guard for Account
function isAccount(user: User | SsoUser | Account | Organization): user is Account {
  return user.type === 'account';
}

// Type guard for Organization (in organization contexts)
function isOrganization(user: User | SsoUser | Account | Organization): user is Organization {
  return user.type === 'organization';
}

// Usage example
function getUserEmail(ctx: any): string | null {
  const { currentUser } = ctx;
  
  if (isUser(currentUser) || isSsoUser(currentUser)) {
    return currentUser.attributes.email;
  } else if (isAccount(currentUser)) {
    return currentUser.attributes.email;
  } else if (isOrganization(currentUser)) {
    // Organizations don't have emails
    return null;
  }
  
  return null;
}
```

### Checking User Permissions

```typescript
// Check if user has specific role
function hasRole(user: User | SsoUser, roleId: string): boolean {
  if (!isUser(user) && !isSsoUser(user)) {
    return false;
  }
  
  return user.relationships.role.data.id === roleId;
}

// Check if user can edit content
function canEditContent(user: User | SsoUser | Account | Organization): boolean {
  if (isAccount(user)) {
    // Account users have full access
    return true;
  }
  
  if (isUser(user) || isSsoUser(user)) {
    // Check if user's role allows editing
    return user.attributes.can_edit_schema === false ? false : true;
  }
  
  return false;
}
```

## Field Type Guards

Field type validation is crucial for field extensions and content operations:

### Basic Field Type Guards

```typescript
import type { Field } from 'datocms-plugin-sdk';

// Text field types
function isTextField(field: Field): boolean {
  return ['string', 'text'].includes(field.attributes.field_type);
}

function isStringField(field: Field): boolean {
  return field.attributes.field_type === 'string';
}

function isTextareaField(field: Field): boolean {
  return field.attributes.field_type === 'text';
}

// Number field types
function isNumberField(field: Field): boolean {
  return ['integer', 'float'].includes(field.attributes.field_type);
}

function isIntegerField(field: Field): boolean {
  return field.attributes.field_type === 'integer';
}

function isFloatField(field: Field): boolean {
  return field.attributes.field_type === 'float';
}

// Boolean field
function isBooleanField(field: Field): boolean {
  return field.attributes.field_type === 'boolean';
}

// Date/time fields
function isDateField(field: Field): boolean {
  return field.attributes.field_type === 'date';
}

function isDateTimeField(field: Field): boolean {
  return field.attributes.field_type === 'date_time';
}

// Media fields
function isFileField(field: Field): boolean {
  return field.attributes.field_type === 'file';
}

function isGalleryField(field: Field): boolean {
  return field.attributes.field_type === 'gallery';
}

// Relationship fields
function isLinkField(field: Field): boolean {
  return field.attributes.field_type === 'link';
}

function isLinksField(field: Field): boolean {
  return field.attributes.field_type === 'links';
}

// Structured text fields
function isStructuredTextField(field: Field): boolean {
  return field.attributes.field_type === 'structured_text';
}

// Modular content fields
function isModularContentField(field: Field): boolean {
  return field.attributes.field_type === 'rich_text';
}

// JSON field
function isJsonField(field: Field): boolean {
  return field.attributes.field_type === 'json';
}

// SEO field
function isSeoField(field: Field): boolean {
  return field.attributes.field_type === 'seo';
}

// Geolocation field
function isLatLonField(field: Field): boolean {
  return field.attributes.field_type === 'lat_lon';
}

// Color field
function isColorField(field: Field): boolean {
  return field.attributes.field_type === 'color';
}

// Slug field
function isSlugField(field: Field): boolean {
  return field.attributes.field_type === 'slug';
}
```

### Advanced Field Validations

```typescript
// Check if field is localized
function isLocalizedField(field: Field): boolean {
  return field.attributes.localized === true;
}

// Check if field is required
function isRequiredField(field: Field): boolean {
  const validators = field.attributes.validators || {};
  return validators.required?.present === true;
}

// Check if field has specific validator
function hasValidator(field: Field, validatorType: string): boolean {
  const validators = field.attributes.validators || {};
  return validatorType in validators;
}

// Check if field accepts multiple values
function acceptsMultipleValues(field: Field): boolean {
  return ['links', 'gallery'].includes(field.attributes.field_type);
}

// Check if field is a relationship field
function isRelationshipField(field: Field): boolean {
  return ['link', 'links', 'rich_text'].includes(field.attributes.field_type);
}

// Type guard with appearance check
function isColorPickerField(field: Field): boolean {
  return field.attributes.field_type === 'color' && 
         field.attributes.appearance?.parameters?.preset_colors !== undefined;
}
```

## Item and Upload Guards

### Item Type Guards

```typescript
import type { Item } from 'datocms-plugin-sdk';

// Check if item is published
function isPublished(item: Item): boolean {
  return item.meta.status === 'published';
}

// Check if item is draft
function isDraft(item: Item): boolean {
  return item.meta.status === 'draft';
}

// Check if item has been modified
function hasUnpublishedChanges(item: Item): boolean {
  return item.meta.status === 'updated';
}

// Check if item is of specific type
function isItemType(item: Item, itemTypeId: string): boolean {
  return item.relationships.item_type.data.id === itemTypeId;
}

// Check if item has specific field value
function hasFieldValue(item: Item, fieldName: string): boolean {
  return item.attributes[fieldName] !== null && 
         item.attributes[fieldName] !== undefined;
}
```

### Upload Type Guards

```typescript
import type { Upload } from 'datocms-plugin-sdk';

// Check if upload is an image
function isImage(upload: Upload): boolean {
  return upload.attributes.mime_type.startsWith('image/');
}

// Check if upload is a video
function isVideo(upload: Upload): boolean {
  return upload.attributes.mime_type.startsWith('video/');
}

// Check if upload is a document
function isDocument(upload: Upload): boolean {
  const documentTypes = [
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'application/vnd.ms-excel',
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
  ];
  return documentTypes.includes(upload.attributes.mime_type);
}

// Check if upload has alt text
function hasAltText(upload: Upload): boolean {
  return upload.attributes.alt !== null && upload.attributes.alt.trim() !== '';
}

// Check if upload has specific metadata
function hasMetadata(upload: Upload, key: string): boolean {
  return key in (upload.attributes.default_field_metadata || {});
}
```

## Dropdown Action Guards

The SDK provides built-in type guards for dropdown actions:

```typescript
import { 
  isDropdownAction, 
  isDropdownActionGroup,
  isDropdownActionOrGroupArray 
} from 'datocms-plugin-sdk';
import type { DropdownAction, DropdownActionGroup } from 'datocms-plugin-sdk';

// Check if value is a DropdownAction
const action = { id: 'test', label: 'Test', icon: 'plus' };
if (isDropdownAction(action)) {
  console.log('Valid action:', action.label);
}

// Check if value is a DropdownActionGroup
const group = { 
  label: 'Actions', 
  icon: 'folder',
  actions: [action]
};
if (isDropdownActionGroup(group)) {
  console.log('Valid group with', group.actions.length, 'actions');
}

// Check if array contains valid actions/groups
const items = [action, group];
if (isDropdownActionOrGroupArray(items)) {
  console.log('Valid dropdown items');
}

// Custom guards for action states
function isActiveAction(action: DropdownAction): boolean {
  return action.active === true;
}

function isDisabledAction(action: DropdownAction): boolean {
  return action.disabled === true;
}

function hasParameters(action: DropdownAction): boolean {
  return action.parameters !== undefined && 
         Object.keys(action.parameters).length > 0;
}
```

## Built-in Type Guards

The SDK includes several utility type guards:

```typescript
import {
  isNullish,
  isBoolean,
  isString,
  isNumber,
  isRecord,
  isArray,
  isPlacement
} from 'datocms-plugin-sdk';

// Basic type checks
const value: unknown = 'hello';

if (isString(value)) {
  console.log(value.toUpperCase()); // TypeScript knows it's a string
}

if (isNumber(value)) {
  console.log(value.toFixed(2)); // TypeScript knows it's a number
}

if (isArray(value)) {
  console.log(value.length); // TypeScript knows it's an array
}

if (isRecord(value)) {
  console.log(Object.keys(value)); // TypeScript knows it's an object
}

// Nullish check (null or undefined)
if (isNullish(value)) {
  console.log('Value is null or undefined');
}

// Placement check for sidebar panels
const placement: unknown = ['before', 'info'];
if (isPlacement(placement)) {
  console.log('Valid placement:', placement);
}
```

## Creating Custom Type Guards

### Pattern for Complex Types

```typescript
// Type guard for structured text nodes
interface TextNode {
  type: 'text';
  text: string;
  marks?: Array<{ type: string }>;
}

function isTextNode(node: unknown): node is TextNode {
  return isRecord(node) &&
         node.type === 'text' &&
         isString(node.text) &&
         (node.marks === undefined || isArray(node.marks));
}

// Type guard for API responses
interface ApiResponse<T> {
  data: T;
  meta?: Record<string, unknown>;
}

function isApiResponse<T>(
  value: unknown,
  dataGuard: (data: unknown) => data is T
): value is ApiResponse<T> {
  return isRecord(value) &&
         'data' in value &&
         dataGuard(value.data) &&
         (value.meta === undefined || isRecord(value.meta));
}

// Usage
function isUserResponse(value: unknown): value is ApiResponse<User> {
  return isApiResponse(value, (data): data is User => {
    return isRecord(data) && data.type === 'user';
  });
}
```

### Combining Type Guards

```typescript
// Type guard for editable text fields
function isEditableTextField(field: Field): boolean {
  return isTextField(field) && 
         !field.attributes.disabled &&
         field.attributes.field_type !== 'slug'; // Slugs are auto-generated
}

// Type guard for uploadable fields
function acceptsUploads(field: Field): boolean {
  return isFileField(field) || isGalleryField(field);
}

// Type guard for translatable fields
function isTranslatableField(field: Field): boolean {
  return isLocalizedField(field) && 
         (isTextField(field) || isStructuredTextField(field));
}

// Complex validation combining multiple guards
function canOptimizeFieldContent(field: Field): boolean {
  return (isTextField(field) || isStructuredTextField(field)) &&
         !field.attributes.disabled &&
         isLocalizedField(field) &&
         hasValidator(field, 'required');
}
```

## Best Practices

### 1. Use Type Predicates

Always use TypeScript's type predicate syntax (`value is Type`) for proper type narrowing:

```typescript
// Good - TypeScript understands the type narrowing
function isValidEmail(value: unknown): value is string {
  return isString(value) && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
}

// Bad - No type narrowing
function checkValidEmail(value: unknown): boolean {
  return isString(value) && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
}
```

### 2. Guard Against null/undefined

Always check for null/undefined before accessing properties:

```typescript
function getFieldLabel(field: Field | null | undefined): string {
  if (!field) {
    return 'Unknown field';
  }
  
  return field.attributes.label || field.attributes.api_key;
}
```

### 3. Use Built-in Guards When Available

Prefer SDK-provided guards over custom implementations:

```typescript
import { isString, isRecord } from 'datocms-plugin-sdk';

// Good - Use built-in guards
if (isString(value)) {
  // ...
}

// Avoid - Don't reimplement
if (typeof value === 'string') {
  // ...
}
```

### 4. Combine Guards for Complex Validations

```typescript
// Comprehensive field validation
function isValidFieldForExtension(
  field: Field | null | undefined,
  supportedTypes: string[]
): field is Field {
  return field !== null &&
         field !== undefined &&
         supportedTypes.includes(field.attributes.field_type) &&
         !field.attributes.disabled;
}

// Usage
const supportedTypes = ['string', 'text', 'structured_text'];
if (isValidFieldForExtension(ctx.field, supportedTypes)) {
  // TypeScript knows field is defined and valid
  console.log('Field label:', ctx.field.attributes.label);
}
```

### 5. Document Custom Guards

Always add JSDoc comments to custom type guards:

```typescript
/**
 * Checks if the upload is an optimizable image format
 * @param upload - The upload to check
 * @returns True if the upload is a JPEG, PNG, or WebP image
 */
function isOptimizableImage(upload: Upload): boolean {
  const optimizableTypes = ['image/jpeg', 'image/png', 'image/webp'];
  return optimizableTypes.includes(upload.attributes.mime_type);
}
```

### 6. Handle Union Types Systematically

When dealing with union types, check all possibilities:

```typescript
function getUserDisplayName(user: User | SsoUser | Account | Organization): string {
  if (isUser(user) || isSsoUser(user)) {
    return user.attributes.first_name && user.attributes.last_name
      ? `${user.attributes.first_name} ${user.attributes.last_name}`
      : user.attributes.email;
  }
  
  if (isAccount(user)) {
    return user.attributes.company || user.attributes.email;
  }
  
  if (isOrganization(user)) {
    return user.attributes.name;
  }
  
  // TypeScript knows this is unreachable if the union is exhaustive
  throw new Error('Unknown user type');
}
```

## See Also

- [Utility Functions](./utility-functions.md) - Additional utility functions including built-in type guards
- [Shared Types](./shared-types.md) - Type definitions that work with these guards
- [Context Object](./context-object.md) - Understanding the context object and its properties