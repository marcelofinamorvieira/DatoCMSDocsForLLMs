# Shared Types

This document covers the shared types exported from `packages/sdk/src/shared.ts`. These types are fundamental building blocks used across multiple hooks in the DatoCMS Plugin SDK.

## DropdownAction

An object expressing a dropdown action to be shown in the interface. Used in hooks like `fieldDropdownActions`, `itemFormDropdownActions`, `itemsDropdownActions`, etc.

```typescript
// From packages/sdk/src/shared.ts
export type DropdownAction = {
  /**
   * ID of action. Will be the first argument for the
   * execute function
   */
  id: string;

  /**
   * An arbitrary configuration object that will be passed as the `parameters`
   * property of the second argument of the execute function
   */
  parameters?: Record<string, unknown>;

  /** Label to be shown. Must be unique. */
  label: string;

  /**
   * Icon to be shown alongside the label. Can be a FontAwesome icon name (ie.
   * `"address-book"`) or a custom SVG definition. To maintain visual
   * consistency with the rest of the interface, try to use FontAwesome icons
   * whenever possible.
   */
  icon: Icon;

  active?: boolean;
  alert?: boolean;
  disabled?: boolean;
  closeMenuOnClick?: boolean;

  /**
   * Actions will be displayed by ascending `rank`. If you want to specify an
   * explicit value for `rank`, make sure to offer a way for final users to
   * customize it inside the plugin's settings form, otherwise the hardcoded
   * value you choose might clash with the one of another plugin!
   */
  rank?: number;
};
```

### Example Usage

```typescript
import type { DropdownAction } from 'datocms-plugin-sdk';

const action: DropdownAction = {
  id: 'export-csv',
  label: 'Export to CSV',
  icon: 'download',
  parameters: { format: 'csv', includeHeaders: true },
  active: false,
  alert: false, // Note: this is boolean, not a string message
  disabled: false,
  closeMenuOnClick: true,
  rank: 10
};
```

## DropdownActionGroup

An object expressing a dropdown submenu containing actions to be shown in the interface.

```typescript
// From packages/sdk/src/shared.ts
export type DropdownActionGroup = {
  /** Label to be shown. Must be unique. */
  label: string;

  /**
   * Icon to be shown alongside the label. Can be a FontAwesome icon name (ie.
   * `"address-book"`) or a custom SVG definition. To maintain visual
   * consistency with the rest of the interface, try to use FontAwesome icons
   * whenever possible.
   */
  icon: Icon;

  actions: DropdownAction[];

  /**
   * Actions will be displayed by ascending `rank`. If you want to specify an
   * explicit value for `rank`, make sure to offer a way for final users to
   * customize it inside the plugin's settings form, otherwise the hardcoded
   * value you choose might clash with the one of another plugin!
   */
  rank?: number;
};
```

### Example Usage

```typescript
import type { DropdownActionGroup } from 'datocms-plugin-sdk';

const actionGroup: DropdownActionGroup = {
  label: 'Export Options',
  icon: 'file-export',
  actions: [
    { 
      id: 'export-csv', 
      label: 'Export as CSV', 
      icon: 'file-csv',
      parameters: { format: 'csv' }
    },
    { 
      id: 'export-json', 
      label: 'Export as JSON', 
      icon: 'file-code',
      parameters: { format: 'json' }
    }
  ],
  rank: 20
};
```

## ItemFormSidebarPanelPlacement

Defines where to place custom sidebar panels in the item form relative to built-in panels.

```typescript
// From packages/sdk/src/shared.ts
export type ItemFormSidebarPanelPlacement = [
  'before' | 'after',
  'info' | 'publishedVersion' | 'schedule' | 'links' | 'history',
];
```

### Built-in Panel References
- `'info'` - Basic item information panel
- `'publishedVersion'` - Published version panel  
- `'schedule'` - Publishing schedule panel
- `'links'` - Linked items panel
- `'history'` - Revision history panel

### Example Usage

```typescript
import type { ItemFormSidebarPanelPlacement } from 'datocms-plugin-sdk';

// Place custom panel after the info panel
const placement: ItemFormSidebarPanelPlacement = ['after', 'info'];

// Place custom panel before the history panel
const placement2: ItemFormSidebarPanelPlacement = ['before', 'history'];

// Use in itemFormSidebarPanels hook
itemFormSidebarPanels(itemType, ctx) {
  return [
    {
      id: 'my-panel',
      label: 'Custom Panel',
      placement: ['after', 'info']
    }
  ];
}
```

## Type Guard Functions

The SDK provides type guard functions to validate objects at runtime:

### isDropdownAction

Validates if a value is a valid DropdownAction:

```typescript
// From packages/sdk/src/shared.ts
export function isDropdownAction(value: unknown): value is DropdownAction {
  if (!isRecord(value)) return false;

  return (
    isString(value.id) &&
    (isNullish(value.parameters) || isRecord(value.parameters)) &&
    isString(value.label) &&
    isIcon(value.icon) &&
    (isNullish(value.active) || isBoolean(value.active)) &&
    (isNullish(value.alert) || isBoolean(value.alert)) &&
    (isNullish(value.disabled) || isBoolean(value.disabled)) &&
    (isNullish(value.closeMenuOnClick) || isBoolean(value.closeMenuOnClick)) &&
    (isNullish(value.rank) || isNumber(value.rank))
  );
}
```

### isDropdownActionGroup

Validates if a value is a valid DropdownActionGroup:

```typescript
// From packages/sdk/src/shared.ts
export function isDropdownActionGroup(
  value: unknown,
): value is DropdownActionGroup {
  if (!isRecord(value)) return false;

  return (
    isString(value.label) &&
    isIcon(value.icon) &&
    isArray(value.actions, isDropdownAction) &&
    (isNullish(value.rank) || isNumber(value.rank))
  );
}
```

### isDropdownActionOrGroupArray

Validates if a value is a valid array of DropdownAction or DropdownActionGroup:

```typescript
// From packages/sdk/src/shared.ts
export function isDropdownActionOrGroupArray(
  value: unknown,
): value is Array<DropdownAction | DropdownActionGroup> {
  return isArray(
    value,
    (innerValue) =>
      isDropdownAction(innerValue) || isDropdownActionGroup(innerValue),
  );
}
```

### Example Usage

```typescript
import { isDropdownAction, isDropdownActionGroup } from 'datocms-plugin-sdk';

// Runtime validation
function processAction(action: unknown) {
  if (isDropdownAction(action)) {
    console.log('Valid action:', action.id);
  } else if (isDropdownActionGroup(action)) {
    console.log('Valid group:', action.label);
    action.actions.forEach(a => console.log('  - ', a.label));
  } else {
    throw new Error('Invalid action format');
  }
}
```

## Complete Example: Using Shared Types in Hooks

```typescript
import { connect, DropdownAction, DropdownActionGroup } from 'datocms-plugin-sdk';

connect({
  fieldDropdownActions(field, ctx) {
    const actions: Array<DropdownAction | DropdownActionGroup> = [];

    // Single action
    actions.push({
      id: 'validate',
      label: 'Validate Field',
      icon: 'check-circle',
      parameters: { fieldId: field.id }
    });

    // Group of actions
    if (field.attributes.field_type === 'string') {
      actions.push({
        label: 'Text Operations',
        icon: 'font',
        actions: [
          {
            id: 'uppercase',
            label: 'Convert to Uppercase',
            icon: 'arrow-up',
            parameters: { fieldId: field.id, operation: 'upper' }
          },
          {
            id: 'lowercase',
            label: 'Convert to Lowercase', 
            icon: 'arrow-down',
            parameters: { fieldId: field.id, operation: 'lower' }
          }
        ]
      });
    }

    return actions;
  },

  async executeFieldDropdownAction(actionId, ctx) {
    const { parameters } = ctx;
    
    switch (actionId) {
      case 'validate':
        // Validation logic
        await ctx.notice('Field validated successfully');
        break;
        
      case 'uppercase':
      case 'lowercase':
        const fieldPath = ctx.fieldPath;
        const currentValue = ctx.formValues[fieldPath];
        
        if (typeof currentValue === 'string') {
          const newValue = actionId === 'uppercase' 
            ? currentValue.toUpperCase()
            : currentValue.toLowerCase();
            
          await ctx.setFieldValue(fieldPath, newValue);
          await ctx.notice(`Text converted to ${actionId}`);
        }
        break;
    }
  }
});
```

## Best Practices

1. **Always specify icons**: Icons improve UX and help users quickly identify actions
2. **Use unique IDs**: Action IDs must be unique within your plugin
3. **Provide rank configuration**: Let users customize the order of actions in plugin settings
4. **Type parameters**: Define interfaces for your parameters object for type safety
5. **Validate at runtime**: Use the provided type guard functions when processing external data

## Other Shared Types

While the types documented above (DropdownAction, DropdownActionGroup, ItemFormSidebarPanelPlacement) are specific to `shared.ts`, the SDK exports many other commonly used types. For comprehensive documentation of all available types, see:

- [Type Exports](./type-exports.md) - Complete list of all exported types
- [Icon Type Reference](./icon-type.md) - Icon type and usage
- [Context Object](./context-object.md) - Context types and properties