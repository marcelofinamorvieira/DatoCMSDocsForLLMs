# SDK Manifest

The SDK manifest is a comprehensive metadata file that describes all hooks, context types, and their properties available in the DatoCMS Plugin SDK. It powers development tools like the ContextInspector and provides type information for plugin developers.

## Purpose

The `manifest.json` file serves several critical purposes:

1. **Type Documentation**: Provides complete type information for all hooks and context objects
2. **Development Tools**: Powers the ContextInspector component that helps developers explore available properties
3. **Code Generation**: Used to generate TypeScript type definitions
4. **API Reference**: Acts as a machine-readable API reference

## Manifest Structure

The manifest has the following top-level structure:

```typescript
// From packages/sdk/src/manifestTypes.ts
export type Manifest = {
  /** Hooks and their respective information. */
  hooks: Record<string, HookInfo>;
  /** Base properties and methods available on every context argument. */
  baseCtx: {
    /** Properties in the base context. */
    properties: PropertiesOrMethodsGroup[];
    /** Methods in the base context. */
    methods: PropertiesOrMethodsGroup[];
  };
  /** Extra properties and methods available on every SelfResizingPluginFrameCtx context argument */
  selfResizingPluginFrameCtxSizingUtilities: PropertiesOrMethodsGroup;
};
```

## Core Type Definitions

### HookInfo

Describes a single hook available in the SDK:

```typescript
export type HookInfo = {
  /** Name of the hook. */
  name: string;
  /** JSDoc comment and tag for the hook. */
  comment?: Comment;
  /** Non-context arguments for the hook. */
  nonCtxArguments: NonCtxArgument[];
  /** Context argument for the hook, if any. */
  ctxArgument?: CtxArgument;
  /** Return type of the hook function. */
  returnType: string;
  /** Code location where the hook is defined. */
  location: CodeLocation;
};
```

### PropertiesOrMethodsGroup

Groups related properties or methods:

```typescript
export type PropertiesOrMethodsGroup = {
  /** Name of the group */
  name?: string;
  /** Description of the group */
  comment?: Comment;
  /** Type of the parameter or method. */
  items: Record<string, PropertyOrMethod>;
};
```

### CtxArgument

Defines the context argument passed to hooks:

```typescript
export type CtxArgument = {
  /** Type of the context argument. */
  type: string;
  /** Additional properties in the context argument, if any. */
  additionalProperties?: PropertiesOrMethodsGroup[];
  /** Additional methods in the context argument, if any. */
  additionalMethods?: PropertiesOrMethodsGroup[];
};
```

### PropertyOrMethod

Describes a single property or method:

```typescript
export type PropertyOrMethod = {
  /** Description of the parameter or method. */
  comment?: Comment;
  /** Code location where the parameter or method is defined. */
  location: CodeLocation;
  /** Type of the parameter or method. */
  type: string;
};
```

### Comment

Contains documentation information:

```typescript
export type Comment = {
  /** The comment itself. */
  markdownText: string;
  /** JSDoc tag, if any. */
  tag?: string;
  /** Example or example code, if any. */
  example?: string;
  /** Info about deprecation, if any. */
  deprecatedMarkdownText?: string;
};
```

### CodeLocation

Points to where code is defined:

```typescript
export type CodeLocation = {
  /** File path where the code is defined. */
  filePath: string;
  /** Line number in the file where the hook is defined. */
  lineNumber: number;
};
```

### NonCtxArgument

Non-context arguments passed to hooks:

```typescript
export type NonCtxArgument = {
  /** Name of the non-context argument. */
  name: string;
  /** Type name of the non-context argument. */
  typeName: string;
};
```

## Generation Process

The manifest is automatically generated from TypeScript source files using the `generateManifest.ts` script:

```bash
cd packages/sdk
npm run generate-manifest
```

The generation process:

1. **Scans Hook Files**: Looks for all TypeScript files in `src/hooks/` (excluding `.guard.ts` files)
2. **Extracts Type Information**: Uses TypeScript's Compiler API to parse type definitions
3. **Resolves References**: Follows type references to build complete type information
4. **Extracts Documentation**: Pulls JSDoc comments and tags from the source
5. **Outputs Files**: 
   - `manifest.json` - JSON format for tools
   - `src/manifest.ts` - TypeScript format for type-safe imports

## Example Manifest Entry

Here's an example of how a hook appears in the manifest:

```json
{
  "hooks": {
    "fieldDropdownActions": {
      "name": "fieldDropdownActions",
      "comment": {
        "markdownText": "Use this function to define custom actions for field dropdowns.",
        "tag": "fieldActions"
      },
      "nonCtxArguments": [
        {
          "name": "field",
          "typeName": "Field"
        }
      ],
      "ctxArgument": {
        "type": "Ctx",
        "additionalProperties": [
          {
            "name": "ItemFormAdditionalProperties",
            "comment": {
              "markdownText": "Additional properties available in item form contexts."
            },
            "items": {
              "formValues": {
                "comment": {
                  "markdownText": "Current values of the record's fields."
                },
                "location": {
                  "filePath": "src/ctx/commonExtras/itemForm.ts",
                  "lineNumber": 42
                },
                "type": "Record<string, unknown>"
              }
            }
          }
        ],
        "additionalMethods": []
      },
      "returnType": "FieldDropdownActionGroup[]",
      "location": {
        "filePath": "src/hooks/fieldDropdownActions.ts",
        "lineNumber": 18
      }
    }
  }
}
```

## Using the Manifest

### In Development Tools

The ContextInspector component uses the manifest to show available properties:

```typescript
import { manifest } from 'datocms-plugin-sdk/lib/manifest';
import { ContextInspector } from 'datocms-react-ui';

// The ContextInspector reads the manifest to display context info
<ContextInspector ctx={ctx} />
```

### For Type Generation

The manifest can be used to generate TypeScript definitions:

```typescript
import { manifest } from './manifest';

// Generate type definitions from manifest
Object.entries(manifest.hooks).forEach(([hookName, hookInfo]) => {
  console.log(`Hook: ${hookName}`);
  console.log(`Returns: ${hookInfo.returnType}`);
  console.log(`Context type: ${hookInfo.ctxArgument?.type}`);
});
```

### For Documentation

Tools can read the manifest to generate documentation:

```typescript
// Extract all hooks with their descriptions
const hookDocs = Object.entries(manifest.hooks).map(([name, info]) => ({
  name,
  description: info.comment?.markdownText,
  tag: info.comment?.tag,
  example: info.comment?.example
}));
```

## Context Type Resolution

The manifest generator intelligently resolves context types:

1. **Base Context**: Properties and methods available to all hooks
2. **Specific Context Types**:
   - `Ctx` - Basic context
   - `SelfResizingPluginFrameCtx` - Context with sizing utilities
   - `ImposedSizePluginFrameCtx` - Context for full-screen modes
3. **Additional Properties**: Type-specific properties like `ItemFormAdditionalProperties`
4. **Additional Methods**: Type-specific methods like `ItemFormAdditionalMethods`

## Best Practices

When contributing to the SDK:

1. **Always add JSDoc comments** to hooks and types
2. **Use @tag annotations** to categorize hooks
3. **Include @example blocks** for complex functionality
4. **Mark deprecated features** with @deprecated
5. **Run manifest generation** after making changes to hooks

Example hook with complete documentation:

```typescript
/**
 * Use this function to define custom actions for field dropdowns.
 * @tag fieldActions
 * @example
 * ```typescript
 * fieldDropdownActions(field, ctx) {
 *   return [
 *     {
 *       id: 'custom-action',
 *       label: 'My Custom Action',
 *       icon: 'pencil'
 *     }
 *   ];
 * }
 * ```
 * @deprecated Use fieldActions instead (will be removed in v2.0.0)
 */
export type FieldDropdownActionsHook = {
  fieldDropdownActions(
    field: Field,
    ctx: Ctx<ItemFormAdditionalProperties, ItemFormAdditionalMethods>
  ): FieldDropdownActionGroup[];
};
```