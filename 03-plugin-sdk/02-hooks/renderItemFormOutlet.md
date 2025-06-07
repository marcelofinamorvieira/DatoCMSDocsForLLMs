# renderItemFormOutlet Hook

## Purpose

The `renderItemFormOutlet` hook renders custom UI components in designated outlet areas within item forms. These outlets allow you to add supplementary functionality, visualizations, or tools that enhance the content editing experience without interfering with the main form fields.

## Signature

```typescript
renderItemFormOutlet(
  itemFormOutletId: string,
  ctx: RenderItemFormOutletCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

- `itemFormOutletId: string` - The ID of the outlet being rendered
- `ctx: RenderItemFormOutletCtx` - The context object containing all item form data and methods

## Type Definitions

```typescript
// Main hook type
export type RenderItemFormOutletHook = {
  renderItemFormOutlet: (
    itemFormOutletId: string,
    ctx: RenderItemFormOutletCtx
  ) => void | { render: () => Promise<{ default: React.ComponentType<any> }> };
};

// Context type extends base context with item form properties and methods
type RenderItemFormOutletCtx = Ctx<ItemFormAdditionalProperties & ItemFormAdditionalMethods & {
  itemFormOutletId: string;
  placement: ItemFormOutletPlacement;
}>;

// Outlet placement type
type ItemFormOutletPlacement = 
  | 'before_fields'
  | 'after_fields'
  | ['before_field', string]
  | ['after_field', string]
  | ['before_fieldset', string]
  | ['after_fieldset', string]
  | ['before_block', string]
  | ['after_block', string];

// Base context type with all properties and methods
type Ctx<AdditionalProperties = {}> = BaseProperties & BaseMethods & AdditionalProperties;

// Base properties available in all contexts
type BaseProperties = {
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken?: string;
  site: Site;
  environment: string;
  theme: Theme;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  menuItems: MenuItem[];
  schema: Schema;
  users: Partial<Record<string, User>>;
};

// Base UI methods available in all contexts
type BaseMethods = {
  notice: (message: string) => void;
  alert: (message: string, options?: { cancel?: ButtonOptions }) => Promise<void>;
  confirm: (message: string, options?: { 
    cancel?: ButtonOptions; 
    choices?: ButtonChoices 
  }) => Promise<boolean | number | null>;
  prompt: (message: string, options?: { 
    cancel?: ButtonOptions; 
    placeholder?: string; 
    initialValue?: string 
  }) => Promise<string | null>;
  openModal: (options: ModalOptions) => Promise<void>;
  navigateTo: (path: string) => void;
  openUrl: (url: string) => void;
};

// Additional properties specific to item form contexts
type ItemFormAdditionalProperties = {
  item: Item | null;
  itemId: string | null;
  itemType: ItemType;
  itemStatus: 'new' | 'draft' | 'updated' | 'published';
  isSubmitting: boolean;
  isFormDirty: boolean;
  locale: string;
  formValues: Record<string, unknown>;
  fieldHavingError: string[];
};

// Additional methods specific to item form contexts
type ItemFormAdditionalMethods = {
  setFieldValue: (fieldPath: string, value: unknown) => void;
  setMultipleFieldValues: (values: Record<string, unknown>) => void;
  scrollToField: (fieldPath: string) => void;
  toggleField: (fieldPath: string, show: boolean) => void;
  disableField: (fieldPath: string, disable: boolean) => void;
  saveCurrentItem: () => Promise<void>;
  getFieldValue: (fieldPath: string) => unknown;
};

// Plugin configuration
type Plugin = {
  id: string;
  type: 'plugin';
  attributes: {
    name: string;
    description?: string;
    url: string;
    parameters: Record<string, unknown>;
    field_types: string[];
    plugin_type: 'field_editor' | 'field_addon' | 'sidebar' | 'asset_source' | 'page' | 'modal';
  };
  relationships: {
    installed_at_site?: { data: { id: string; type: 'site' } };
  };
};

// User type definitions
type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string;
    last_name: string;
    avatar_url?: string;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_publish_to_production: boolean;
    two_factor_authentication_active?: boolean;
  };
  relationships: {
    role: { data: { id: string; type: 'role' } };
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
  };
  relationships: Record<string, any>;
};

type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
    company?: string;
  };
  relationships: Record<string, any>;
};

type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
    logo_url?: string;
  };
  relationships: Record<string, any>;
};

// Role permissions
type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_publish_to_production: boolean;
    can_manage_webhooks: boolean;
    can_manage_environments: boolean;
    can_access_build_triggers: boolean;
    can_manage_seo_readability_analysis: boolean;
    positive_item_type_permissions: string[];
    negative_item_type_permissions: string[];
    meta_permissions: Record<string, any>;
  };
  relationships: Record<string, any>;
};

// Site configuration
type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    domain?: string;
    internal_domain: string;
    locales: string[];
    timezone: string;
    theme: Record<string, any>;
    no_index: boolean;
    global_seo?: Record<string, any>;
    favicon?: any;
    favicon_meta_tags?: any[];
  };
  relationships: Record<string, any>;
};

// Theme configuration
type Theme = {
  primary_color?: string;
  dark_color?: string;
  light_color?: string;
  accent_color?: string;
};

// ItemType (model) definition
type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    tree: boolean;
    modular_block: boolean;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'table';
    has_singleton_item: boolean;
    hint?: string;
    preview_field?: string;
    title_field?: string;
    image_preview_field?: string;
    excerpt_field?: string;
    workflow?: any;
    ordering_direction?: 'asc' | 'desc';
    ordering_field?: any;
    some_locales_required?: boolean;
    versioning_enabled?: boolean;
  };
  relationships: {
    fields: { data: Array<{ id: string; type: 'field' }> };
    fieldsets: { data: Array<{ id: string; type: 'fieldset' }> };
    singleton_item?: { data: { id: string; type: 'item' } | null };
    ordering_field?: { data: { id: string; type: 'field' } | null };
    title_field?: { data: { id: string; type: 'field' } | null };
    image_preview_field?: { data: { id: string; type: 'field' } | null };
    excerpt_field?: { data: { id: string; type: 'field' } | null };
  };
};

// Field definition
type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    field_type: 'boolean' | 'color' | 'date' | 'date_time' | 'file' | 'float' | 'gallery' | 'integer' | 'json' | 'lat_lon' | 'link' | 'links' | 'rich_text' | 'seo' | 'single_line_string' | 'slug' | 'structured_text' | 'text' | 'video';
    api_key: string;
    localized: boolean;
    validators: Record<string, any>;
    position: number;
    hint?: string;
    default_value?: any;
    appearance: {
      addons: any[];
      editor: string;
      parameters: Record<string, any>;
      type: 'addon' | 'editor' | 'presentation';
    };
    deep_filtering_enabled?: boolean;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fieldset?: { data: { id: string; type: 'fieldset' } | null };
  };
};

// Fieldset (field group)
type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    hint?: string;
    collapsible: boolean;
    start_collapsed: boolean;
    position: number;
  };
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
    fields: { data: Array<{ id: string; type: 'field' }> };
  };
};

// Item definition
type Item = {
  id: string;
  type: 'item';
  attributes: Record<string, any>;
  relationships: {
    item_type: { data: { id: string; type: 'item_type' } };
  };
  meta: {
    created_at: string;
    updated_at: string;
    published_at?: string;
    publication_scheduled_at?: string;
    unpublishing_scheduled_at?: string;
    first_published_at?: string;
    is_valid: boolean;
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    status: 'draft' | 'updated' | 'published';
    current_version: string;
    stage?: string;
  };
};

// Menu structure
type MenuItem = {
  id: string;
  type: 'menu_item';
  attributes: {
    label: string;
    position: number;
    external_url?: string;
    open_in_new_tab?: boolean;
  };
  relationships: {
    item_type?: { data: { id: string; type: 'item_type' } | null };
    parent?: { data: { id: string; type: 'menu_item' } | null };
    children?: { data: Array<{ id: string; type: 'menu_item' }> };
  };
};

// Schema information
type Schema = {
  models: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
};

// UI interaction types
type ButtonOptions = {
  label: string;
  value: boolean;
  intent?: 'primary' | 'secondary' | 'negative';
};

type ButtonChoices = Array<{
  label: string;
  value: boolean | number;
  intent?: 'primary' | 'secondary' | 'negative';
}>;

type ModalOptions = {
  id: string;
  title: string;
  width?: 'xs' | 's' | 'm' | 'l' | 'xl' | 'fullWidth';
  parameters?: Record<string, unknown>;
};
```

## Use Cases

### 1. SEO Preview and Analysis

Show how the content will appear in search results:

```typescript
async renderItemFormOutlet(
  itemFormOutletId: string,
  ctx: RenderItemFormOutletCtx
) {
  if (itemFormOutletId === 'seoPreview') {
    return {
      render: () => import('./components/SeoPreviewOutlet'),
    };
  }
}

// SeoPreviewOutlet.tsx
import { RenderItemFormOutletCtx } from 'datocms-plugin-sdk';
import { Canvas, Section } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

export default function SeoPreviewOutlet({ ctx }: { ctx: RenderItemFormOutletCtx }) {
  const [seoScore, setSeoScore] = useState(0);
  const [suggestions, setSuggestions] = useState<string[]>([]);

  // Get SEO-related fields
  const title = ctx.formValues.title || ctx.formValues.name || '';
  const description = ctx.formValues.meta_description || ctx.formValues.description || '';
  const slug = ctx.formValues.slug || '';
  const content = ctx.formValues.content || ctx.formValues.body || '';

  useEffect(() => {
    analyzeSeo();
  }, [title, description, slug, content]);

  const analyzeSeo = () => {
    let score = 0;
    const newSuggestions = [];

    // Title analysis
    if (title) {
      score += 20;
      if (title.length > 60) {
        newSuggestions.push('Title is too long (60 chars max for Google)');
      } else if (title.length < 30) {
        newSuggestions.push('Title might be too short (30-60 chars recommended)');
      } else {
        score += 10;
      }
    } else {
      newSuggestions.push('Add a title for better SEO');
    }

    // Description analysis
    if (description) {
      score += 20;
      if (description.length > 160) {
        newSuggestions.push('Meta description is too long (160 chars max)');
      } else if (description.length < 120) {
        newSuggestions.push('Meta description might be too short (120-160 chars recommended)');
      } else {
        score += 10;
      }
    } else {
      newSuggestions.push('Add a meta description');
    }

    // Slug analysis
    if (slug) {
      score += 20;
      if (!/^[a-z0-9-]+$/.test(slug)) {
        newSuggestions.push('Slug should only contain lowercase letters, numbers, and hyphens');
      } else {
        score += 10;
      }
    }

    // Content analysis
    if (content) {
      const wordCount = content.split(/\s+/).length;
      if (wordCount > 300) {
        score += 20;
        if (wordCount > 600) {
          score += 10;
        }
      } else {
        newSuggestions.push('Content is too short (300+ words recommended)');
      }

      // Check for headings
      if (content.includes('<h2>') || content.includes('<h3>')) {
        score += 10;
      } else {
        newSuggestions.push('Add subheadings to structure your content');
      }
    }

    setSeoScore(Math.min(score, 100));
    setSuggestions(newSuggestions);
  };

  const getScoreColor = () => {
    if (seoScore >= 80) return '#4CAF50';
    if (seoScore >= 60) return '#FFA726';
    return '#F44336';
  };

  const truncate = (str: string, length: number) => {
    if (str.length <= length) return str;
    return str.substring(0, length) + '...';
  };

  return (
    <Canvas ctx={ctx}>
      <Section title="SEO Preview & Analysis">
        <div style={{ padding: '20px' }}>
          {/* Google Search Preview */}
          <div style={{
            backgroundColor: '#fff',
            padding: '20px',
            borderRadius: '8px',
            boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
            marginBottom: '20px',
          }}>
            <h4 style={{ marginBottom: '15px' }}>Google Search Preview</h4>
            <div style={{ fontFamily: 'Arial, sans-serif' }}>
              <h3 style={{ 
                color: '#1a0dab',
                fontSize: '20px',
                marginBottom: '3px',
                cursor: 'pointer',
              }}>
                {truncate(title || 'Page Title', 60)}
              </h3>
              <div style={{
                color: '#006621',
                fontSize: '14px',
                marginBottom: '5px',
              }}>
                {ctx.site.attributes.domain || 'example.com'}/{slug || 'page-url'}
              </div>
              <div style={{
                color: '#545454',
                fontSize: '14px',
                lineHeight: '1.5',
              }}>
                {truncate(description || 'Page description will appear here...', 160)}
              </div>
            </div>
          </div>

          {/* SEO Score */}
          <div style={{
            backgroundColor: '#fff',
            padding: '20px',
            borderRadius: '8px',
            boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
            marginBottom: '20px',
          }}>
            <h4 style={{ marginBottom: '15px' }}>SEO Score</h4>
            <div style={{ position: 'relative', marginBottom: '10px' }}>
              <div style={{
                height: '30px',
                backgroundColor: '#e0e0e0',
                borderRadius: '15px',
                overflow: 'hidden',
              }}>
                <div style={{
                  height: '100%',
                  width: `${seoScore}%`,
                  backgroundColor: getScoreColor(),
                  transition: 'width 0.3s ease',
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center',
                  color: '#fff',
                  fontWeight: 'bold',
                }}>
                  {seoScore}%
                </div>
              </div>
            </div>
            
            {suggestions.length > 0 && (
              <div>
                <h5 style={{ marginTop: '20px', marginBottom: '10px' }}>
                  Suggestions for improvement:
                </h5>
                <ul style={{ paddingLeft: '20px', margin: 0 }}>
                  {suggestions.map((suggestion, index) => (
                    <li key={index} style={{ marginBottom: '5px', color: '#666' }}>
                      {suggestion}
                    </li>
                  ))}
                </ul>
              </div>
            )}
          </div>

          {/* Quick Actions */}
          <div style={{
            backgroundColor: '#fff',
            padding: '20px',
            borderRadius: '8px',
            boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
          }}>
            <h4 style={{ marginBottom: '15px' }}>Quick Actions</h4>
            <div style={{ display: 'flex', gap: '10px', flexWrap: 'wrap' }}>
              <button
                onClick={() => {
                  const focusKeyword = window.prompt('Enter focus keyword:');
                  if (focusKeyword) {
                    // Check keyword density
                    const contentLower = (content || '').toLowerCase();
                    const keywordCount = (contentLower.match(new RegExp(focusKeyword.toLowerCase(), 'g')) || []).length;
                    const wordCount = contentLower.split(/\s+/).length;
                    const density = ((keywordCount / wordCount) * 100).toFixed(2);
                    
                    ctx.notice(`Keyword "${focusKeyword}" appears ${keywordCount} times (${density}% density)`);
                  }
                }}
                style={{
                  padding: '8px 16px',
                  backgroundColor: '#f0f0f0',
                  border: 'none',
                  borderRadius: '4px',
                  cursor: 'pointer',
                }}
              >
                Check Keyword Density
              </button>
              
              <button
                onClick={() => {
                  if (!title && ctx.formValues.headline) {
                    ctx.setFieldValue('title', ctx.formValues.headline);
                    ctx.notice('Title set from headline');
                  }
                  if (!description && content) {
                    const firstParagraph = content.substring(0, 160).replace(/<[^>]*>/g, '');
                    ctx.setFieldValue('meta_description', firstParagraph);
                    ctx.notice('Meta description generated from content');
                  }
                }}
                style={{
                  padding: '8px 16px',
                  backgroundColor: '#f0f0f0',
                  border: 'none',
                  borderRadius: '4px',
                  cursor: 'pointer',
                }}
              >
                Auto-generate Meta
              </button>
            </div>
          </div>
        </div>
      </Section>
    </Canvas>
  );
}
```

### 2. Content Validation Dashboard

Show validation status and content quality metrics:

```typescript
async renderItemFormOutlet(
  itemFormOutletId: string,
  ctx: RenderItemFormOutletCtx
) {
  if (itemFormOutletId === 'contentValidation') {
    return {
      render: () => import('./components/ContentValidationOutlet'),
    };
  }
}

// ContentValidationOutlet.tsx
import { RenderItemFormOutletCtx } from 'datocms-plugin-sdk';
import { Canvas, Section, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaCheckCircle, FaExclamationTriangle, FaTimesCircle } from 'react-icons/fa';

interface ValidationResult {
  field: string;
  status: 'valid' | 'warning' | 'error';
  message: string;
}

export default function ContentValidationOutlet({ ctx }: { ctx: RenderItemFormOutletCtx }) {
  const [validationResults, setValidationResults] = useState<ValidationResult[]>([]);
  const [isValidating, setIsValidating] = useState(false);

  useEffect(() => {
    validateContent();
  }, [ctx.formValues]);

  const validateContent = async () => {
    setIsValidating(true);
    const results: ValidationResult[] = [];

    // Validate each field
    for (const field of Object.values(ctx.itemType.attributes.fields)) {
      const value = ctx.formValues[field.apiKey];
      
      // Required field validation
      if (field.validators?.required && !value) {
        results.push({
          field: field.label,
          status: 'error',
          message: 'This field is required',
        });
        continue;
      }

      // Field-specific validations
      switch (field.fieldType) {
        case 'string':
        case 'text':
          if (value) {
            // Check length constraints
            if (field.validators?.length?.min && value.length < field.validators.length.min) {
              results.push({
                field: field.label,
                status: 'error',
                message: `Minimum length is ${field.validators.length.min} characters`,
              });
            }
            if (field.validators?.length?.max && value.length > field.validators.length.max) {
              results.push({
                field: field.label,
                status: 'warning',
                message: `Maximum length is ${field.validators.length.max} characters`,
              });
            }
            
            // Check for broken links in text
            const urlRegex = /https?:\/\/[^\s]+/g;
            const urls = value.match(urlRegex) || [];
            for (const url of urls) {
              try {
                const response = await fetch(url, { method: 'HEAD' });
                if (!response.ok) {
                  results.push({
                    field: field.label,
                    status: 'warning',
                    message: `Contains broken link: ${url}`,
                  });
                }
              } catch (e) {
                results.push({
                  field: field.label,
                  status: 'warning',
                  message: `Unable to verify link: ${url}`,
                });
              }
            }
          }
          break;

        case 'date':
        case 'date_time':
          if (value) {
            const date = new Date(value);
            if (field.apiKey === 'publish_date' && date < new Date()) {
              results.push({
                field: field.label,
                status: 'warning',
                message: 'Publish date is in the past',
              });
            }
          }
          break;

        case 'file':
        case 'gallery':
          if (value) {
            // Check file size, alt text, etc.
            const files = Array.isArray(value) ? value : [value];
            for (const file of files) {
              if (file && !file.alt) {
                results.push({
                  field: field.label,
                  status: 'warning',
                  message: 'Missing alt text for accessibility',
                });
              }
            }
          }
          break;
      }

      // Custom business logic validations
      if (field.apiKey === 'slug' && value) {
        // Check for duplicate slugs
        const client = ctx.createClient();
        try {
          const items = await client.items.list({
            filter: {
              type: ctx.itemType.id,
              fields: {
                slug: { eq: value }
              }
            }
          });
          
          if (items.length > 0 && items[0].id !== ctx.itemId) {
            results.push({
              field: field.label,
              status: 'error',
              message: 'This slug is already in use',
            });
          }
        } catch (e) {
          // Ignore API errors
        }
      }
    }

    // If all validations pass, add success message
    if (results.length === 0) {
      results.push({
        field: 'Overall',
        status: 'valid',
        message: 'All validations passed!',
      });
    }

    setValidationResults(results);
    setIsValidating(false);
  };

  const getIcon = (status: string) => {
    switch (status) {
      case 'valid':
        return <FaCheckCircle color="#4CAF50" />;
      case 'warning':
        return <FaExclamationTriangle color="#FFA726" />;
      case 'error':
        return <FaTimesCircle color="#F44336" />;
      default:
        return null;
    }
  };

  const groupedResults = validationResults.reduce((acc, result) => {
    if (!acc[result.status]) {
      acc[result.status] = [];
    }
    acc[result.status].push(result);
    return acc;
  }, {} as Record<string, ValidationResult[]>);

  const scrollToField = (fieldLabel: string) => {
    const field = Object.values(ctx.itemType.attributes.fields).find(
      f => f.label === fieldLabel
    );
    if (field) {
      ctx.scrollToField(field.apiKey);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <Section title="Content Validation">
        <div style={{ padding: '20px' }}>
          {/* Summary */}
          <div style={{
            display: 'grid',
            gridTemplateColumns: 'repeat(3, 1fr)',
            gap: '15px',
            marginBottom: '20px',
          }}>
            <div style={{
              padding: '15px',
              backgroundColor: '#E8F5E9',
              borderRadius: '8px',
              textAlign: 'center',
            }}>
              <FaCheckCircle size={24} color="#4CAF50" />
              <div style={{ marginTop: '5px', fontWeight: 'bold' }}>
                {groupedResults.valid?.length || 0} Valid
              </div>
            </div>
            
            <div style={{
              padding: '15px',
              backgroundColor: '#FFF3E0',
              borderRadius: '8px',
              textAlign: 'center',
            }}>
              <FaExclamationTriangle size={24} color="#FFA726" />
              <div style={{ marginTop: '5px', fontWeight: 'bold' }}>
                {groupedResults.warning?.length || 0} Warnings
              </div>
            </div>
            
            <div style={{
              padding: '15px',
              backgroundColor: '#FFEBEE',
              borderRadius: '8px',
              textAlign: 'center',
            }}>
              <FaTimesCircle size={24} color="#F44336" />
              <div style={{ marginTop: '5px', fontWeight: 'bold' }}>
                {groupedResults.error?.length || 0} Errors
              </div>
            </div>
          </div>

          {/* Validation Results */}
          <div>
            {['error', 'warning', 'valid'].map(status => {
              const results = groupedResults[status];
              if (!results || results.length === 0) return null;

              return (
                <div key={status} style={{ marginBottom: '15px' }}>
                  <h4 style={{ 
                    marginBottom: '10px',
                    color: status === 'error' ? '#F44336' : status === 'warning' ? '#FFA726' : '#4CAF50'
                  }}>
                    {status.charAt(0).toUpperCase() + status.slice(1)}s
                  </h4>
                  {results.map((result, index) => (
                    <div
                      key={index}
                      style={{
                        display: 'flex',
                        alignItems: 'center',
                        padding: '10px',
                        backgroundColor: '#f5f5f5',
                        borderRadius: '4px',
                        marginBottom: '5px',
                        cursor: result.field !== 'Overall' ? 'pointer' : 'default',
                      }}
                      onClick={() => result.field !== 'Overall' && scrollToField(result.field)}
                    >
                      <div style={{ marginRight: '10px' }}>
                        {getIcon(result.status)}
                      </div>
                      <div style={{ flex: 1 }}>
                        <strong>{result.field}:</strong> {result.message}
                      </div>
                    </div>
                  ))}
                </div>
              );
            })}
          </div>

          {/* Re-validate Button */}
          <div style={{ marginTop: '20px' }}>
            <Button
              onClick={validateContent}
              disabled={isValidating}
              fullWidth
            >
              {isValidating ? 'Validating...' : 'Re-validate Content'}
            </Button>
          </div>
        </div>
      </Section>
    </Canvas>
  );
}
```

### 3. AI Content Assistant

Provide AI-powered content suggestions:

```typescript
async renderItemFormOutlet(
  itemFormOutletId: string,
  ctx: RenderItemFormOutletCtx
) {
  if (itemFormOutletId === 'aiAssistant') {
    return {
      render: () => import('./components/AiAssistantOutlet'),
    };
  }
}

// AiAssistantOutlet.tsx
import { RenderItemFormOutletCtx } from 'datocms-plugin-sdk';
import { Canvas, Section, Button, TextareaField } from 'datocms-react-ui';
import { useState } from 'react';
import { FaMagic, FaLightbulb, FaSpellCheck, FaLanguage } from 'react-icons/fa';

export default function AiAssistantOutlet({ ctx }: { ctx: RenderItemFormOutletCtx }) {
  const [suggestions, setSuggestions] = useState<string[]>([]);
  const [isGenerating, setIsGenerating] = useState(false);
  const [selectedText, setSelectedText] = useState('');

  const apiKey = ctx.plugin.attributes.parameters.openaiApiKey;

  const generateSuggestions = async (prompt: string, text: string) => {
    if (!apiKey) {
      ctx.alert('OpenAI API key not configured');
      return;
    }

    setIsGenerating(true);
    try {
      const response = await fetch('https://api.openai.com/v1/completions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${apiKey}`,
        },
        body: JSON.stringify({
          model: 'text-davinci-003',
          prompt: prompt,
          max_tokens: 150,
          n: 3,
          temperature: 0.7,
        }),
      });

      const data = await response.json();
      const newSuggestions = data.choices.map((choice: any) => choice.text.trim());
      setSuggestions(newSuggestions);
    } catch (error) {
      ctx.alert('Failed to generate suggestions');
    } finally {
      setIsGenerating(false);
    }
  };

  const improveWriting = () => {
    const content = ctx.formValues.content || ctx.formValues.description || '';
    const text = selectedText || content;
    
    if (!text) {
      ctx.alert('Please select some text or add content to improve');
      return;
    }

    const prompt = `Improve the following text, making it more engaging and professional:\n\n${text}\n\nImproved version:`;
    generateSuggestions(prompt, text);
  };

  const generateIdeas = () => {
    const title = ctx.formValues.title || ctx.formValues.name || '';
    
    if (!title) {
      ctx.alert('Please add a title first');
      return;
    }

    const prompt = `Generate 3 creative content ideas for an article titled "${title}". For each idea, provide a brief outline:`;
    generateSuggestions(prompt, title);
  };

  const checkGrammar = async () => {
    const content = ctx.formValues.content || ctx.formValues.description || '';
    
    if (!content) {
      ctx.alert('No content to check');
      return;
    }

    setIsGenerating(true);
    try {
      const prompt = `Check the following text for grammar and spelling errors. List any issues found:\n\n${content}`;
      await generateSuggestions(prompt, content);
    } finally {
      setIsGenerating(false);
    }
  };

  const translateContent = async (targetLang: string) => {
    const content = ctx.formValues.content || ctx.formValues.description || '';
    
    if (!content) {
      ctx.alert('No content to translate');
      return;
    }

    const prompt = `Translate the following text to ${targetLang}, maintaining the tone and style:\n\n${content}\n\nTranslation:`;
    await generateSuggestions(prompt, content);
  };

  const applySuggestion = (suggestion: string) => {
    // Try to find the most appropriate field to update
    if (ctx.formValues.content !== undefined) {
      ctx.setFieldValue('content', suggestion);
    } else if (ctx.formValues.description !== undefined) {
      ctx.setFieldValue('description', suggestion);
    } else if (ctx.formValues.body !== undefined) {
      ctx.setFieldValue('body', suggestion);
    }
    
    ctx.notice('Suggestion applied to content');
    setSuggestions([]);
  };

  return (
    <Canvas ctx={ctx}>
      <Section title="AI Content Assistant">
        <div style={{ padding: '20px' }}>
          {/* Action Buttons */}
          <div style={{
            display: 'grid',
            gridTemplateColumns: 'repeat(2, 1fr)',
            gap: '10px',
            marginBottom: '20px',
          }}>
            <Button
              onClick={improveWriting}
              leftIcon={<FaMagic />}
              disabled={isGenerating}
              fullWidth
            >
              Improve Writing
            </Button>
            
            <Button
              onClick={generateIdeas}
              leftIcon={<FaLightbulb />}
              disabled={isGenerating}
              fullWidth
            >
              Generate Ideas
            </Button>
            
            <Button
              onClick={checkGrammar}
              leftIcon={<FaSpellCheck />}
              disabled={isGenerating}
              fullWidth
            >
              Check Grammar
            </Button>
            
            <Button
              onClick={() => {
                const lang = window.prompt('Enter target language (e.g., Spanish, French):');
                if (lang) translateContent(lang);
              }}
              leftIcon={<FaLanguage />}
              disabled={isGenerating}
              fullWidth
            >
              Translate
            </Button>
          </div>

          {/* Text Selection */}
          <div style={{ marginBottom: '20px' }}>
            <TextareaField
              id="selectedText"
              name="selectedText"
              label="Selected Text (optional)"
              value={selectedText}
              onChange={setSelectedText}
              hint="Paste specific text here to improve, or leave empty to use the main content"
              textareaInputProps={{ rows: 3 }}
            />
          </div>

          {/* Loading State */}
          {isGenerating && (
            <div style={{
              textAlign: 'center',
              padding: '20px',
              backgroundColor: '#f5f5f5',
              borderRadius: '8px',
            }}>
              <div className="spinner" />
              <p>Generating suggestions...</p>
            </div>
          )}

          {/* Suggestions */}
          {suggestions.length > 0 && (
            <div>
              <h4 style={{ marginBottom: '15px' }}>AI Suggestions:</h4>
              {suggestions.map((suggestion, index) => (
                <div
                  key={index}
                  style={{
                    padding: '15px',
                    backgroundColor: '#f8f9fa',
                    borderRadius: '8px',
                    marginBottom: '10px',
                    border: '1px solid #e0e0e0',
                  }}
                >
                  <p style={{ marginBottom: '10px', whiteSpace: 'pre-wrap' }}>
                    {suggestion}
                  </p>
                  <div style={{ display: 'flex', gap: '10px' }}>
                    <Button
                      onClick={() => applySuggestion(suggestion)}
                      buttonSize="s"
                      buttonType="primary"
                    >
                      Apply This
                    </Button>
                    <Button
                      onClick={() => {
                        navigator.clipboard.writeText(suggestion);
                        ctx.notice('Copied to clipboard');
                      }}
                      buttonSize="s"
                    >
                      Copy
                    </Button>
                  </div>
                </div>
              ))}
              
              <Button
                onClick={() => setSuggestions([])}
                buttonType="muted"
                fullWidth
              >
                Clear Suggestions
              </Button>
            </div>
          )}

          {/* Tips */}
          <div style={{
            marginTop: '20px',
            padding: '15px',
            backgroundColor: '#e3f2fd',
            borderRadius: '8px',
            fontSize: '14px',
          }}>
            <strong>Tips:</strong>
            <ul style={{ marginTop: '5px', marginBottom: 0, paddingLeft: '20px' }}>
              <li>Select specific text to improve just that portion</li>
              <li>Use "Generate Ideas" when starting a new article</li>
              <li>Check grammar before publishing</li>
              <li>Translate content to quickly create multilingual versions</li>
            </ul>
          </div>
        </div>
      </Section>
    </Canvas>
  );
}
```

## Best Practices

1. **Placement Awareness**: Design outlets based on their placement (top, bottom, sidebar)
2. **Performance**: Avoid heavy computations that slow down form loading
3. **Responsive Design**: Ensure outlets work well in different viewport sizes
4. **Form Integration**: Use form methods to interact with fields appropriately
5. **User Feedback**: Provide clear feedback for all actions
6. **Error Handling**: Handle API failures gracefully
7. **State Management**: Keep outlet state in sync with form values
8. **Accessibility**: Ensure all interactive elements are keyboard accessible
9. **Visual Hierarchy**: Design to complement, not compete with, form fields
10. **Save State**: Consider if outlet state should persist between sessions

## Related Hooks

- `itemFormOutlets` - Define which outlets your plugin provides
- `renderItemCollectionOutlet` - Add outlets to collection views
- `itemFormSidebarPanels` - Add panels to the form sidebar
- `renderFieldExtension` - Create custom field editors

## Important Notes

- Outlets can be placed at the top, bottom, or in the sidebar of item forms
- The context provides full access to form state and methods
- Changes made through outlets are immediately reflected in the form
- Outlets should enhance, not replace, core form functionality
- Consider mobile responsiveness, especially for sidebar outlets
- Use the Canvas component to ensure consistent styling