# customMarksForStructuredTextField Hook

## Purpose

The `customMarksForStructuredTextField` hook allows plugins to define custom inline formatting marks for Structured Text fields. This extends the default formatting options (bold, italic, etc.) with custom inline styles that content editors can apply to selected text, providing enhanced text formatting capabilities tailored to specific content needs.

## Signature

```typescript
customMarksForStructuredTextField(
  field: Field,
  ctx: CustomMarksForStructuredTextFieldCtx
): StructuredTextCustomMark[] | undefined
```

## Context Properties

The hook receives a context object with base properties plus field-specific information:

- All standard context properties (`plugin`, `currentUser`, `site`, etc.)
- `itemType`: ItemType - The model containing the Structured Text field

## Parameters

- `field`: Field - The Structured Text field for which to provide custom marks

## Type Definitions

### Complete Hook Signature

```typescript
export type CustomMarksForStructuredTextFieldHook = {
  customMarksForStructuredTextField: (
    field: Field,
    ctx: CustomMarksForStructuredTextFieldCtx,
  ) => StructuredTextCustomMark[] | undefined;
};
```

### Context Type Definition

```typescript
export type CustomMarksForStructuredTextFieldCtx = {
  // Base Properties
  plugin: Plugin;
  currentUserAccessToken: string | null;
  currentUser: User | SsoUser | null;
  currentRole: Role;
  account: Account;
  site: Site;
  environment: string;
  environmentId: string;
  mainLocale: string;
  currentUserMenuLocale: string;
  theme: 'dark' | 'light';
  ui: Record<string, unknown>;
  itemTypes: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
  users: User[];
  ssoUsers: SsoUser[];
  
  // Hook-specific Properties
  itemType: ItemType;
  
  // Base Methods
  async loadItemTypes(ids: string[]): Promise<ItemType[]>;
  async loadFields(ids: string[]): Promise<Field[]>;
  async loadFieldsets(ids: string[]): Promise<Fieldset[]>;
  async loadUsers(ids: string[]): Promise<User[]>;
  async loadSsoUsers(ids: string[]): Promise<SsoUser[]>;
  async updatePluginParameters(params: Record<string, unknown>): Promise<void>;
  async updateFieldAppearance(
    fieldId: string,
    appearance: Record<string, unknown>
  ): Promise<void>;
  notice(message: string): void;
  alert(message: string): void;
  customToast(toast: Toast): void;
  async selectItem(
    itemTypeId: string,
    options?: {
      multiple?: boolean;
      locale?: string;
      filters?: ItemListFilter;
      initialLocationQuery?: LocationQuery;
    }
  ): Promise<Item | Item[] | null>;
  async createNewItem(
    itemTypeId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<Item | null>;
  async editItem(
    itemId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<Item | null>;
  async selectUpload(options?: {
    multiple?: boolean;
    locale?: string;
    filters?: UploadListFilter;
  }): Promise<Upload | Upload[] | null>;
  async editUpload(
    uploadId: string,
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<Upload | null>;
  async editUploads(
    uploadIds: string[],
    options?: {
      locale?: string;
      initialValues?: Record<string, unknown>;
      finalizeCallback?: FinalizeCallback;
    }
  ): Promise<void>;
  async openModal(
    modalId: string,
    options?: {
      title?: string;
      width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
      height?: 's' | 'm' | 'l' | number;
      initialHeight?: number;
      closeDisabled?: boolean;
      parameters?: Record<string, unknown>;
    }
  ): Promise<unknown>;
  async openConfirm(options: {
    title: string;
    content?: ReactElement | string;
    choices?: Array<{
      label: string;
      value: unknown;
      icon?: Icon;
      intent?: 'positive' | 'negative';
    }>;
    cancel?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  async navigateTo(
    location: 
      | string
      | { 
          pageId: string;
          params?: Record<string, string>;
          query?: Record<string, string>;
          locale?: string;
        }
  ): Promise<void>;
  getFieldIntlTitle(
    field: Field,
    locale?: string,
    returnUntranslatedTitle?: boolean
  ): string | null;
  createClient(options?: { 
    apiToken?: string;
    environment?: string;
  }): SiteApiClient;
};
```

### StructuredTextCustomMark Type

```typescript
export type StructuredTextCustomMark = {
  /** Unique identifier for the mark */
  id: string;
  
  /** Display label shown in toolbar */
  label: string;
  
  /** Icon - FontAwesome name or custom SVG */
  icon: Icon;
  
  /** Where to place the button in toolbar */
  placement?: StructuredTextCustomMarkPlacement;
  
  /** Priority when multiple marks have same placement */
  rank?: number;
  
  /** Keyboard shortcut (e.g., 'mod+shift+x') */
  keyboardShortcut?: string;
  
  /** CSS styles applied to marked text */
  appliedStyle: React.CSSProperties;
};

export type StructuredTextCustomMarkPlacement = [
  'before' | 'after',
  'strong' | 'emphasis' | 'underline' | 'code' | 'highlight' | 'strikethrough',
];

export type Icon = string | SvgIcon;  // FontAwesome icon name or custom SVG

export type SvgIcon = {
  type: 'svg';
  viewBox: string;
  content: string;
};
```

### Field Type

```typescript
interface Field {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    field_type: string; // 'structured_text' for this hook
    api_key: string;
    localized: boolean;
    validators: Record<string, any>;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      addons: Array<{
        id: string;
        parameters: Record<string, any>;
      }>;
    };
    default_value: any;
    hint?: string | null;
    placeholder?: string | null;
    appeareance?: Record<string, any>; // Legacy, use appearance
    position: number;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: 'item_type';
      };
    };
    fieldset?: {
      data: {
        id: string;
        type: 'fieldset';
      } | null;
    };
  };
  meta: {
    valid_validators: Array<{
      type: string;
      config: Record<string, any>;
    }>;
  };
}
```

### ItemType Type

```typescript
interface ItemType {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    modular_block: boolean;
    tree: boolean;
    ordering_direction: 'asc' | 'desc' | null;
    ordering_field: string | null;
    all_locales_required: boolean;
    collection_appearance: 'table' | 'gallery';
    hint?: string | null;
    inverse_relationships_enabled: boolean;
  };
  relationships: {
    fields: {
      data: Array<{
        id: string;
        type: 'field';
      }>;
    };
    fieldsets: {
      data: Array<{
        id: string;
        type: 'fieldset';
      }>;
    };
    singleton_item?: {
      data: {
        id: string;
        type: 'item';
      } | null;
    };
  };
  meta: {
    has_singleton_item: boolean;
  };
}
```

### React.CSSProperties Type

```typescript
interface CSSProperties {
  // Typography
  color?: string;
  fontSize?: string | number;
  fontFamily?: string;
  fontWeight?: string | number;
  fontStyle?: 'normal' | 'italic' | 'oblique';
  lineHeight?: string | number;
  letterSpacing?: string | number;
  textAlign?: 'left' | 'right' | 'center' | 'justify';
  textDecoration?: string;
  textIndent?: string | number;
  textTransform?: 'none' | 'capitalize' | 'uppercase' | 'lowercase';
  textShadow?: string;
  
  // Box Model
  margin?: string | number;
  marginTop?: string | number;
  marginRight?: string | number;
  marginBottom?: string | number;
  marginLeft?: string | number;
  padding?: string | number;
  paddingTop?: string | number;
  paddingRight?: string | number;
  paddingBottom?: string | number;
  paddingLeft?: string | number;
  
  // Borders
  border?: string;
  borderTop?: string;
  borderRight?: string;
  borderBottom?: string;
  borderLeft?: string;
  borderColor?: string;
  borderRadius?: string | number;
  borderStyle?: 'solid' | 'dashed' | 'dotted' | 'double' | 'none';
  borderWidth?: string | number;
  
  // Background
  backgroundColor?: string;
  backgroundImage?: string;
  backgroundPosition?: string;
  backgroundRepeat?: string;
  backgroundSize?: string;
  
  // Layout
  display?: string;
  position?: 'static' | 'relative' | 'absolute' | 'fixed' | 'sticky';
  top?: string | number;
  right?: string | number;
  bottom?: string | number;
  left?: string | number;
  width?: string | number;
  height?: string | number;
  verticalAlign?: string;
  whiteSpace?: string;
  
  // Effects
  opacity?: number;
  boxShadow?: string;
  cursor?: string;
  animation?: string;
  transition?: string;
  fontVariant?: string;
  direction?: 'ltr' | 'rtl';
  
  // Pseudo-selectors (limited support in editor)
  '&:hover'?: CSSProperties;
  '&::before'?: CSSProperties & { content?: string };
  '&::after'?: CSSProperties & { content?: string };
  '&:hover::after'?: CSSProperties;
  
  // Keyframes (limited support)
  '@keyframes pulse'?: Record<string, CSSProperties>;
  
  [key: string]: any;
}
```

### Supporting Types

```typescript
// Plugin Type
interface Plugin {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description: string;
    homepage_url: string;
    icon: string;
    parameters: Record<string, unknown>;
    permissions: string[];
    version: string;
  };
}

// User Types
interface User {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    created_at: string;
    is_2fa_active: boolean;
    last_sign_in_at: string | null;
  };
}

interface SsoUser {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    created_at: string;
  };
}

// Role Type
interface Role {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_site: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_manage_build_triggers: boolean;
    can_manage_menu_items: boolean;
    can_edit_favicon: boolean;
    can_edit_seo_settings: boolean;
    can_manage_workflows: boolean;
    environments_access: 'primary_only' | 'sandbox_only' | 'all';
    positive_item_type_permissions: Array<{
      item_type_id: string;
      action: 'all' | 'create' | 'update' | 'delete' | 'publish' | 'edit_metadata';
    }>;
    negative_item_type_permissions: Array<{
      item_type_id: string;
      action: 'all' | 'create' | 'update' | 'delete' | 'publish' | 'edit_metadata';
    }>;
  };
}

// Account Type
interface Account {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    company: string | null;
  };
}

// Site Type
interface Site {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    domain: string | null;
    global_seo: Record<string, any>;
    favicon: Upload | null;
    locales: string[];
    timezone: string;
  };
}

// Toast Type
interface Toast {
  type: 'notice' | 'alert' | 'warning' | 'primary';
  message: string;
  dismissOnPageChange?: boolean;
  dismissAfterMs?: number;
}

// Type Guards
export function isStructuredTextCustomMark(
  value: unknown,
): value is StructuredTextCustomMark {
  return (
    isRecord(value) &&
    isString(value.id) &&
    isString(value.label) &&
    isIcon(value.icon) &&
    (isNullish(value.placement) || isPlacement(value.placement)) &&
    (isNullish(value.rank) || isNumber(value.rank)) &&
    (isNullish(value.keyboardShortcut) || isString(value.keyboardShortcut)) &&
    isRecord(value.appliedStyle)
  );
}
```

## Return Value

The hook should return an array of `StructuredTextCustomMark` objects or `undefined`:

```typescript
type StructuredTextCustomMark = {
  id: string;                                    // Unique identifier for the mark
  label: string;                                 // Display name in toolbar
  icon: Icon;                                   // FontAwesome icon or custom SVG
  placement?: StructuredTextCustomMarkPlacement; // Position in toolbar
  rank?: number;                                // Priority when multiple marks exist
  keyboardShortcut?: string;                    // Keyboard shortcut (e.g., 'mod+shift+x')
  appliedStyle: React.CSSProperties;            // CSS styles applied to marked text
}
```

## Placement Options

Control where your custom mark button appears in the toolbar:

```typescript
type StructuredTextCustomMarkPlacement = [
  'before' | 'after',
  'strong' | 'emphasis' | 'underline' | 'code' | 'highlight' | 'strikethrough'
];
```

## Use Cases

### 1. Brand-Specific Text Styles

Add custom marks that align with brand guidelines:

```typescript
customMarksForStructuredTextField(
  field: Field,
  ctx: CustomMarksForStructuredTextFieldCtx
): StructuredTextCustomMark[] | undefined {
  const brandColors = ctx.plugin.attributes.parameters.brandColors || {
    primary: '#0066cc',
    secondary: '#00a86b',
    accent: '#ff6b6b'
  };

  return [
    {
      id: 'brand-highlight',
      label: 'Brand Highlight',
      icon: 'highlighter',
      placement: ['after', 'highlight'],
      keyboardShortcut: 'mod+shift+h',
      appliedStyle: {
        backgroundColor: brandColors.primary + '20',
        color: brandColors.primary,
        padding: '2px 4px',
        borderRadius: '2px'
      },
      rank: 1
    },
    {
      id: 'brand-emphasis',
      label: 'Brand Emphasis',
      icon: 'star',
      placement: ['after', 'strong'],
      keyboardShortcut: 'mod+shift+b',
      appliedStyle: {
        fontWeight: 'bold',
        color: brandColors.primary,
        textShadow: `0 0 1px ${brandColors.primary}40`
      },
      rank: 2
    },
    {
      id: 'small-caps',
      label: 'Small Caps',
      icon: 'font',
      placement: ['after', 'emphasis'],
      keyboardShortcut: 'mod+shift+c',
      appliedStyle: {
        fontVariant: 'small-caps',
        letterSpacing: '0.05em'
      },
      rank: 3
    }
  ];
}
```

### 2. Editorial Annotations

Marks for editorial workflows and review processes:

```typescript
customMarksForStructuredTextField(
  field: Field,
  ctx: CustomMarksForStructuredTextFieldCtx
): StructuredTextCustomMark[] | undefined {
  // Only for content that goes through editorial review
  if (!['article', 'blog_post', 'press_release'].includes(ctx.itemType.attributes.api_key)) {
    return undefined;
  }

  return [
    {
      id: 'fact-check',
      label: 'Needs Fact Check',
      icon: 'question-circle',
      placement: ['after', 'highlight'],
      keyboardShortcut: 'mod+shift+f',
      appliedStyle: {
        backgroundColor: '#fff3cd',
        borderBottom: '2px dotted #ffc107',
        cursor: 'help',
        position: 'relative',
        '&::after': {
          content: '"[?]"',
          fontSize: '0.8em',
          verticalAlign: 'super',
          color: '#856404'
        }
      },
      rank: 1
    },
    {
      id: 'citation-needed',
      label: 'Citation Needed',
      icon: 'quote-right',
      placement: ['after', 'highlight'],
      keyboardShortcut: 'mod+shift+q',
      appliedStyle: {
        backgroundColor: '#f8d7da',
        borderBottom: '2px dotted #dc3545',
        position: 'relative',
        '&::after': {
          content: '"[citation needed]"',
          fontSize: '0.7em',
          verticalAlign: 'super',
          color: '#721c24'
        }
      },
      rank: 2
    },
    {
      id: 'editor-note',
      label: 'Editor Note',
      icon: 'sticky-note',
      placement: ['after', 'highlight'],
      keyboardShortcut: 'mod+shift+n',
      appliedStyle: {
        backgroundColor: '#e2e3e5',
        borderBottom: '2px solid #6c757d',
        fontStyle: 'italic',
        color: '#495057'
      },
      rank: 3
    }
  ];
}
```

### 3. Technical Documentation Marks

Specialized marks for technical content:

```typescript
customMarksForStructuredTextField(
  field: Field,
  ctx: CustomMarksForStructuredTextFieldCtx
): StructuredTextCustomMark[] | undefined {
  // Only for technical documentation
  if (ctx.itemType.attributes.api_key !== 'documentation') {
    return undefined;
  }

  return [
    {
      id: 'keyboard-key',
      label: 'Keyboard Key',
      icon: 'keyboard',
      placement: ['after', 'code'],
      keyboardShortcut: 'mod+shift+k',
      appliedStyle: {
        fontFamily: 'monospace',
        backgroundColor: '#f5f5f5',
        border: '1px solid #ccc',
        borderRadius: '3px',
        padding: '2px 6px',
        fontSize: '0.9em',
        boxShadow: '0 1px 0 rgba(0,0,0,0.2)',
        display: 'inline-block'
      },
      rank: 1
    },
    {
      id: 'variable',
      label: 'Variable',
      icon: 'dollar-sign',
      placement: ['after', 'code'],
      keyboardShortcut: 'mod+shift+v',
      appliedStyle: {
        fontFamily: 'monospace',
        fontStyle: 'italic',
        color: '#d73a49',
        backgroundColor: '#f6f8fa',
        padding: '1px 4px',
        borderRadius: '3px'
      },
      rank: 2
    },
    {
      id: 'api-endpoint',
      label: 'API Endpoint',
      icon: 'link',
      placement: ['after', 'code'],
      keyboardShortcut: 'mod+shift+e',
      appliedStyle: {
        fontFamily: 'monospace',
        color: '#005cc5',
        backgroundColor: '#f1f8ff',
        padding: '2px 4px',
        borderRadius: '3px',
        textDecoration: 'none'
      },
      rank: 3
    },
    {
      id: 'deprecated',
      label: 'Deprecated',
      icon: 'exclamation-triangle',
      placement: ['after', 'strikethrough'],
      keyboardShortcut: 'mod+shift+d',
      appliedStyle: {
        textDecoration: 'line-through',
        opacity: '0.6',
        position: 'relative',
        '&::before': {
          content: '"⚠️"',
          marginRight: '4px'
        }
      },
      rank: 1
    }
  ];
}
```

### 4. Language Learning Content

Marks for educational language content:

```typescript
customMarksForStructuredTextField(
  field: Field,
  ctx: CustomMarksForStructuredTextFieldCtx
): StructuredTextCustomMark[] | undefined {
  if (ctx.itemType.attributes.api_key !== 'language_lesson') {
    return undefined;
  }

  return [
    {
      id: 'pronunciation',
      label: 'Pronunciation Guide',
      icon: 'volume-up',
      placement: ['after', 'emphasis'],
      keyboardShortcut: 'mod+shift+p',
      appliedStyle: {
        borderBottom: '1px dotted #28a745',
        position: 'relative',
        cursor: 'help',
        '&::after': {
          content: 'attr(data-pronunciation)',
          position: 'absolute',
          bottom: '100%',
          left: '0',
          backgroundColor: '#28a745',
          color: 'white',
          padding: '2px 6px',
          borderRadius: '3px',
          fontSize: '0.8em',
          whiteSpace: 'nowrap',
          opacity: '0',
          transition: 'opacity 0.3s',
          pointerEvents: 'none'
        },
        '&:hover::after': {
          opacity: '1'
        }
      },
      rank: 1
    },
    {
      id: 'translation',
      label: 'Translation',
      icon: 'language',
      placement: ['after', 'emphasis'],
      keyboardShortcut: 'mod+shift+t',
      appliedStyle: {
        backgroundColor: '#e3f2fd',
        borderBottom: '1px solid #2196f3',
        padding: '0 2px',
        position: 'relative'
      },
      rank: 2
    },
    {
      id: 'grammar-point',
      label: 'Grammar Point',
      icon: 'graduation-cap',
      placement: ['after', 'strong'],
      keyboardShortcut: 'mod+shift+g',
      appliedStyle: {
        fontWeight: 'bold',
        color: '#6f42c1',
        backgroundColor: '#f8f5ff',
        padding: '2px 4px',
        borderRadius: '3px'
      },
      rank: 3
    }
  ];
}
```

### 5. E-commerce Product Descriptions

Marks for highlighting product features:

```typescript
customMarksForStructuredTextField(
  field: Field,
  ctx: CustomMarksForStructuredTextFieldCtx
): StructuredTextCustomMark[] | undefined {
  if (ctx.itemType.attributes.api_key !== 'product') {
    return undefined;
  }

  return [
    {
      id: 'sale-price',
      label: 'Sale Price',
      icon: 'tag',
      placement: ['after', 'strong'],
      keyboardShortcut: 'mod+shift+s',
      appliedStyle: {
        color: '#dc3545',
        fontWeight: 'bold',
        fontSize: '1.1em',
        textDecoration: 'none',
        position: 'relative',
        '&::before': {
          content: '"$"',
          fontSize: '0.9em'
        }
      },
      rank: 1
    },
    {
      id: 'original-price',
      label: 'Original Price',
      icon: 'dollar-sign',
      placement: ['after', 'strikethrough'],
      keyboardShortcut: 'mod+shift+o',
      appliedStyle: {
        textDecoration: 'line-through',
        color: '#6c757d',
        opacity: '0.7'
      },
      rank: 1
    },
    {
      id: 'feature-highlight',
      label: 'Key Feature',
      icon: 'star',
      placement: ['after', 'highlight'],
      keyboardShortcut: 'mod+shift+f',
      appliedStyle: {
        backgroundColor: '#fff3cd',
        fontWeight: '600',
        padding: '2px 6px',
        borderRadius: '3px',
        boxShadow: '0 0 0 2px #ffc10740'
      },
      rank: 1
    },
    {
      id: 'limited-time',
      label: 'Limited Time',
      icon: 'clock',
      placement: ['after', 'emphasis'],
      keyboardShortcut: 'mod+shift+l',
      appliedStyle: {
        color: '#ff6b6b',
        fontWeight: 'bold',
        fontStyle: 'italic',
        animation: 'pulse 2s infinite',
        '@keyframes pulse': {
          '0%': { opacity: 1 },
          '50%': { opacity: 0.7 },
          '100%': { opacity: 1 }
        }
      },
      rank: 1
    }
  ];
}
```

### 6. Multi-Locale Support

Different marks based on content locale:

```typescript
customMarksForStructuredTextField(
  field: Field,
  ctx: CustomMarksForStructuredTextFieldCtx
): StructuredTextCustomMark[] | undefined {
  const locale = ctx.ui.locale;
  
  // Base marks available for all locales
  const baseMarks: StructuredTextCustomMark[] = [
    {
      id: 'emphasis',
      label: 'Strong Emphasis',
      icon: 'exclamation',
      placement: ['after', 'strong'],
      appliedStyle: {
        fontWeight: 'bold',
        textTransform: 'uppercase',
        letterSpacing: '0.05em'
      },
      rank: 1
    }
  ];

  // Locale-specific marks
  if (locale === 'ja') {
    baseMarks.push({
      id: 'furigana',
      label: 'Furigana (ふりがな)',
      icon: 'language',
      placement: ['after', 'emphasis'],
      keyboardShortcut: 'mod+shift+r',
      appliedStyle: {
        position: 'relative',
        display: 'inline-block',
        '&::before': {
          content: 'attr(data-furigana)',
          position: 'absolute',
          top: '-1em',
          left: '0',
          fontSize: '0.5em',
          color: '#666'
        }
      },
      rank: 10
    });
  }

  if (locale === 'ar' || locale === 'he') {
    baseMarks.push({
      id: 'rtl-emphasis',
      label: locale === 'ar' ? 'تأكيد' : 'הדגשה',
      icon: 'align-right',
      placement: ['after', 'emphasis'],
      appliedStyle: {
        direction: 'rtl',
        fontWeight: 'bold',
        backgroundColor: '#f0f0f0',
        padding: '0 4px'
      },
      rank: 10
    });
  }

  return baseMarks;
}
```

## Keyboard Shortcuts

Define keyboard shortcuts using the [is-hotkey](https://github.com/ianstormtaylor/is-hotkey) syntax:

```typescript
keyboardShortcut: 'mod+shift+h'  // Cmd/Ctrl + Shift + H
keyboardShortcut: 'mod+b'        // Cmd/Ctrl + B
keyboardShortcut: 'alt+shift+x'  // Alt + Shift + X
```

## Custom SVG Icons

While FontAwesome icons are recommended, you can use custom SVGs:

```typescript
{
  id: 'custom-mark',
  label: 'Custom Mark',
  icon: {
    type: 'svg',
    viewBox: '0 0 24 24',
    content: '<path d="M12 2L2 7l10 5 10-5-10-5z"/>'
  },
  // ... other properties
}
```

## Best Practices

1. **Unique IDs**: Ensure each mark has a unique ID within your plugin
2. **Clear Labels**: Use descriptive labels that explain the mark's purpose
3. **Intuitive Icons**: Choose icons that visually represent the formatting
4. **Non-Conflicting Shortcuts**: Test keyboard shortcuts don't conflict with system or DatoCMS shortcuts
5. **Visible Styles**: Apply styles that are clearly distinguishable in the editor
6. **Performance**: Keep styles simple to maintain editor performance
7. **Semantic Meaning**: Design marks that add semantic value, not just visual styling

## Related Hooks

- **customBlockStylesForStructuredTextField**: Define custom block-level styles
- **renderFieldExtension**: Create completely custom field editors
- **manualFieldExtensions**: Register custom field types

## Important Notes

- Custom marks only affect the editor display; frontend rendering must be implemented separately
- Not all CSS properties are supported in the editor environment
- Keyboard shortcuts should be documented for users
- The rank property determines the order in the toolbar
- Return `undefined` to skip adding marks for specific fields
- Mark IDs must be unique within your plugin to avoid conflicts
- Complex CSS animations may impact editor performance

