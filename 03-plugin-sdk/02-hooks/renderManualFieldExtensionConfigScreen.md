# renderManualFieldExtensionConfigScreen Hook

## Purpose

The `renderManualFieldExtensionConfigScreen` hook renders the configuration interface for manual field extensions. This allows users to configure field extension parameters when installing them on specific fields, providing a user-friendly way to customize the behavior of your field extensions.

## Signature

```typescript
renderManualFieldExtensionConfigScreen(
  fieldExtensionId: string,
  ctx: RenderManualFieldExtensionConfigScreenCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

### fieldExtensionId: string
The ID of the field extension being configured

### ctx: RenderManualFieldExtensionConfigScreenCtx
Context object containing all properties and methods for the configuration screen

## Type Definitions

```typescript
// Complete type definitions for renderManualFieldExtensionConfigScreen hook
export type RenderManualFieldExtensionConfigScreenHook = {
  renderManualFieldExtensionConfigScreen: (
    fieldExtensionId: string,
    ctx: RenderManualFieldExtensionConfigScreenCtx
  ) => void | {
    render: () => Promise<{ default: React.ComponentType<any> }>;
  };
};

export type RenderManualFieldExtensionConfigScreenCtx = SelfResizingPluginFrameCtx<
  'renderManualFieldExtensionConfigScreen',
  {
    fieldExtensionId: string;
    parameters: Record<string, unknown>;
    field: Field;
    fieldType: FieldType;
    itemType: ItemType;
    errors: Record<string, string>;
  },
  ManualFieldExtensionConfigMethods
>;

// Methods specific to manual field extension config
type ManualFieldExtensionConfigMethods = {
  setParameters: (params: Record<string, unknown>) => void;
  setParameter: (key: string, value: unknown) => void;
  setErrors: (errors: Record<string, string>) => void;
};

// Base context type with self-resizing capabilities
type SelfResizingPluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = ImposedSizePluginFrameCtx<Mode, AdditionalProperties, AdditionalMethods> &
  SizingUtilities &
  IframeMethods;

// Imposed size context
type ImposedSizePluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BasePluginFrameCtx<Mode, AdditionalProperties, AdditionalMethods>;

// Base plugin frame context
type BasePluginFrameCtx<
  Mode extends keyof FullConnectParameters,
  AdditionalProperties extends Record<string, unknown> = Record<string, never>,
  AdditionalMethods extends Record<string, unknown> = Record<string, never>,
> = BaseProperties &
  BaseMethods &
  AdditionalMethods &
  AdditionalProperties;

// Sizing utilities for self-resizing contexts
type SizingUtilities = {
  startAutoResizer: () => void;
  stopAutoResizer: () => void;
  updateHeight: (height: number) => void;
};

// Iframe-specific methods
type IframeMethods = {
  getSettings: () => Promise<{ mode: string }>;
};

// Base properties included in all contexts
type BaseProperties = {
  plugin: Plugin;
  currentUserAccessToken: string | null;
  environment: string;
  site: Site;
  currentUser: User | SsoUser | null;
  currentRole: Role;
  account: Account;
  mainLocale: string;
  theme: 'dark' | 'light';
  currentUserMenuLocale: string;
  owner: Account | Organization;
  itemTypes: ItemType[];
  fields: Field[];
  fieldsets: Fieldset[];
  users: User[];
  ssoUsers: SsoUser[];
};

// Base methods included in all contexts
type BaseMethods = {
  createClient(options?: {
    apiToken?: string;
    environment?: string;
  }): CmaClient;
  loadItemTypeFields(itemTypeId: string): Promise<Field[]>;
  loadItemTypeFieldsets(itemTypeId: string): Promise<Fieldset[]>;
  loadFieldsUsingPlugin(): Promise<Field[]>;
  loadUsers(): Promise<User[]>;
  loadSsoUsers(): Promise<SsoUser[]>;
  navigateTo(path: string): void;
  scrollToField(path: string): void;
  openUrl(url: string): void;
  copyToClipboard(text: string): void;
  getFieldValue(path: string): unknown;
  toggleField(path: string, show: boolean): void;
  disableField(path: string, disable: boolean): void;
  setFieldValue(path: string, value: unknown): void;
  notice(message: string): void;
  alert(message: string): void;
  customToast(options: {
    message: string;
    type?: 'notice' | 'alert' | 'warning';
    dismissAfterTimeout?: number | false;
    cta?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  openConfirm(options: {
    title: string;
    content?: ReactNode;
    choices?: Array<{
      label: string;
      value: unknown;
      intent?: 'positive' | 'negative';
    }>;
    cancel?: {
      label: string;
      value: unknown;
    };
  }): Promise<unknown>;
  updatePluginParameters(params: Record<string, unknown>): Promise<void>;
  selectItem(itemTypeId: string, options?: {
    multiple?: boolean;
    locale?: string;
    filters?: ItemListFilter;
  }): Promise<Item | Item[] | null>;
  editItem(itemId: string, options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
  }): Promise<Item | null>;
  createNewItem(itemTypeId: string, options?: {
    locale?: string;
    initialValues?: Record<string, unknown>;
  }): Promise<Item | null>;
  openModal(modalId: string, options?: {
    title?: string;
    width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth';
    parameters?: Record<string, unknown>;
  }): Promise<unknown>;
  renderModal(modalId: string): void;
};

// Field type definition
type Field = {
  id: string;
  type: string;
  attributes: {
    label: string;
    api_key: string;
    field_type: string;
    hint?: string;
    localized: boolean;
    required: boolean;
    unique: boolean;
    position: number;
    appearance: {
      editor: string;
      parameters: Record<string, any>;
      type: string;
      field_size: 12;
      groups?: Array<{
        id: string;
        title: string;
        description?: string;
      }>;
    };
    default_value?: any;
    validators: Record<string, any>;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
    fieldset?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
  };
};

// Field type enum
type FieldType = 
  | 'boolean'
  | 'color'
  | 'date'
  | 'date_time'
  | 'file'
  | 'float'
  | 'gallery'
  | 'integer'
  | 'json'
  | 'lat_lon'
  | 'link'
  | 'links'
  | 'modular_content'
  | 'rich_text'
  | 'seo'
  | 'slug'
  | 'string'
  | 'structured_text'
  | 'text'
  | 'video';

// ItemType definition
type ItemType = {
  id: string;
  type: string;
  attributes: {
    name: string;
    api_key: string;
    singleton: boolean;
    sortable: boolean;
    tree: boolean;
    modular_block: boolean;
    hint?: string;
    draft_mode_active: boolean;
    all_locales_required: boolean;
    collection_appearance: 'compact' | 'tabular';
    ordering_direction?: 'asc' | 'desc';
    has_singleton_item: boolean;
  };
  relationships: {
    fields: {
      data: Array<{
        id: string;
        type: string;
      }>;
    };
    fieldsets: {
      data: Array<{
        id: string;
        type: string;
      }>;
    };
    ordering_field?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
    title_field?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
    singleton_item?: {
      data: {
        id: string;
        type: string;
      } | null;
    };
  };
};

// Supporting types
type Plugin = {
  id: string;
  type: string;
  attributes: {
    name: string;
    description?: string;
    homepage_url?: string;
    package_name?: string;
    package_version?: string;
    parameters: Record<string, any>;
  };
  relationships?: {
    installed_by?: {
      data: {
        id: string;
        type: string;
      };
    };
  };
};

type Site = {
  id: string;
  type: string;
  attributes: {
    name: string;
    internal_subdomain: string;
    production_frontend_url?: string;
    primary_locale: string;
    locales: string[];
    timezone: string;
    global_seo?: Record<string, any>;
    favicon?: any;
    no_index: boolean;
  };
};

type User = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    avatar_url?: string;
    is_admin: boolean;
  };
};

type SsoUser = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    is_admin: boolean;
  };
};

type Role = {
  id: string;
  type: string;
  attributes: {
    name: string;
    can_edit_content: boolean;
    can_edit_schema: boolean;
    can_manage_users: boolean;
    can_manage_access_tokens: boolean;
    can_perform_site_search: boolean;
    can_duplicate_records: boolean;
    can_access_audit_log: boolean;
    can_manage_webhooks: boolean;
    can_edit_environment: boolean;
    can_access_build_triggers: boolean;
    environments_access: 'all' | 'primary_only' | string[];
    positive_item_type_permissions: ItemTypePermission[];
    negative_item_type_permissions: ItemTypePermission[];
  };
};

type ItemTypePermission = {
  action: 'create' | 'update' | 'delete' | 'take_over' | 'edit_metadata' | 'edit_schema' | 'publish';
  item_type?: string;
  on_environment?: string;
  locales?: string[];
  item_id?: string;
};

type Account = {
  id: string;
  type: string;
  attributes: {
    email: string;
    first_name?: string;
    last_name?: string;
    company?: string;
  };
};

type Organization = {
  id: string;
  type: string;
  attributes: {
    name: string;
  };
};

type Fieldset = {
  id: string;
  type: string;
  attributes: {
    title: string;
    hint?: string;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
  };
  relationships: {
    item_type: {
      data: {
        id: string;
        type: string;
      };
    };
  };
};

type CmaClient = any; // Simplified for this context
type ItemListFilter = any; // Simplified for this context
type Item = any; // Simplified for this context
type ReactNode = any; // Simplified for this context
type FullConnectParameters = any; // Simplified for this context
```

## Context Properties

The `renderManualFieldExtensionConfigScreen` hook receives a context object with:

### Base Properties
- `plugin`: Plugin - The current plugin configuration
- `currentUser`: User | SsoUser | Account | Organization - The current DatoCMS user
- `currentRole`: Role - The role for the current DatoCMS user
- `site`: Site - The current DatoCMS project
- `theme`: Theme - Theme colors for the current project

### Configuration Properties
- `fieldExtensionId`: string - The ID of the field extension being configured
- `parameters`: Record<string, unknown> - Current parameter values
- `field`: Field - The field where the extension is being installed
- `fieldType`: FieldType - The type of field
- `itemType`: ItemType - The model containing the field
- `errors`: Record<string, string> - Validation errors for parameters

### Methods
- `setParameters(params: Record<string, unknown>)`: void - Update configuration parameters
- `setParameter(key: string, value: unknown)`: void - Update a single parameter
- `setErrors(errors: Record<string, string>)`: void - Set validation errors
- Standard context methods (notice, alert, etc.)

## Use Cases

### 1. API Integration Configuration

Configure external API settings for a field extension:

```typescript
async renderManualFieldExtensionConfigScreen(
  fieldExtensionId: string,
  ctx: RenderManualFieldExtensionConfigScreenCtx
) {
  if (fieldExtensionId === 'externalDataSelector') {
    return {
      render: () => import('./components/ExternalDataSelectorConfig'),
    };
  }
}

// ExternalDataSelectorConfig.tsx
import { RenderManualFieldExtensionConfigScreenCtx } from 'datocms-plugin-sdk';
import { Canvas, Form, TextField, SelectField, SwitchField, Button } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

export default function ExternalDataSelectorConfig({ 
  ctx 
}: { 
  ctx: RenderManualFieldExtensionConfigScreenCtx 
}) {
  const [testing, setTesting] = useState(false);
  const [testResult, setTestResult] = useState<string | null>(null);

  const parameters = ctx.parameters as {
    apiEndpoint?: string;
    apiKey?: string;
    dataPath?: string;
    valueField?: string;
    labelField?: string;
    enableSearch?: boolean;
    cacheMinutes?: number;
  };

  const testConnection = async () => {
    const endpoint = parameters.apiEndpoint;
    const apiKey = parameters.apiKey;

    if (!endpoint) {
      ctx.setErrors({ apiEndpoint: 'API endpoint is required for testing' });
      return;
    }

    setTesting(true);
    setTestResult(null);

    try {
      const response = await fetch(endpoint, {
        headers: apiKey ? { 'Authorization': `Bearer ${apiKey}` } : {},
      });

      if (response.ok) {
        const data = await response.json();
        setTestResult('✅ Connection successful! Found ' + 
          (Array.isArray(data) ? data.length : 'data') + ' items');
        
        // Try to auto-detect fields
        if (Array.isArray(data) && data.length > 0) {
          const firstItem = data[0];
          const keys = Object.keys(firstItem);
          
          if (!parameters.valueField && keys.includes('id')) {
            ctx.setParameter('valueField', 'id');
          }
          if (!parameters.labelField && keys.includes('name')) {
            ctx.setParameter('labelField', 'name');
          } else if (!parameters.labelField && keys.includes('title')) {
            ctx.setParameter('labelField', 'title');
          }
        }
      } else {
        setTestResult('❌ Connection failed: ' + response.statusText);
      }
    } catch (error) {
      setTestResult('❌ Connection error: ' + error.message);
    } finally {
      setTesting(false);
    }
  };

  const validateParameters = () => {
    const errors: Record<string, string> = {};

    if (!parameters.apiEndpoint) {
      errors.apiEndpoint = 'API endpoint is required';
    } else if (!parameters.apiEndpoint.startsWith('http')) {
      errors.apiEndpoint = 'API endpoint must be a valid URL';
    }

    if (!parameters.valueField) {
      errors.valueField = 'Value field is required';
    }

    if (!parameters.labelField) {
      errors.labelField = 'Label field is required';
    }

    if (parameters.cacheMinutes && parameters.cacheMinutes < 0) {
      errors.cacheMinutes = 'Cache duration must be positive';
    }

    ctx.setErrors(errors);
    return Object.keys(errors).length === 0;
  };

  useEffect(() => {
    validateParameters();
  }, [parameters]);

  return (
    <Canvas ctx={ctx}>
      <Form>
        <div style={{ padding: '20px' }}>
          <h3 style={{ marginBottom: '20px' }}>External Data Selector Configuration</h3>
          
          {/* API Configuration */}
          <div style={{ marginBottom: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>API Configuration</h4>
            
            <TextField
              id="apiEndpoint"
              name="apiEndpoint"
              label="API Endpoint"
              value={parameters.apiEndpoint || ''}
              onChange={(value) => ctx.setParameter('apiEndpoint', value)}
              required
              hint="Full URL to the API endpoint that returns data"
              error={ctx.errors.apiEndpoint}
            />

            <TextField
              id="apiKey"
              name="apiKey"
              label="API Key (Optional)"
              value={parameters.apiKey || ''}
              onChange={(value) => ctx.setParameter('apiKey', value)}
              hint="Bearer token or API key for authentication"
              textInputProps={{ type: 'password' }}
            />

            <TextField
              id="dataPath"
              name="dataPath"
              label="Data Path (Optional)"
              value={parameters.dataPath || ''}
              onChange={(value) => ctx.setParameter('dataPath', value)}
              hint="JSON path to the array of items (e.g., 'data.items')"
              placeholder="Leave empty if response is already an array"
            />

            <div style={{ marginTop: '15px' }}>
              <Button
                onClick={testConnection}
                disabled={testing}
                buttonType="primary"
              >
                {testing ? 'Testing...' : 'Test Connection'}
              </Button>
              
              {testResult && (
                <div style={{
                  marginTop: '10px',
                  padding: '10px',
                  backgroundColor: testResult.startsWith('✅') ? '#E8F5E9' : '#FFEBEE',
                  borderRadius: '4px',
                  fontSize: '13px',
                }}>
                  {testResult}
                </div>
              )}
            </div>
          </div>

          {/* Field Mapping */}
          <div style={{ marginBottom: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>Field Mapping</h4>
            
            <TextField
              id="valueField"
              name="valueField"
              label="Value Field"
              value={parameters.valueField || ''}
              onChange={(value) => ctx.setParameter('valueField', value)}
              required
              hint="Field name in the API response to use as the value"
              error={ctx.errors.valueField}
              placeholder="e.g., 'id'"
            />

            <TextField
              id="labelField"
              name="labelField"
              label="Label Field"
              value={parameters.labelField || ''}
              onChange={(value) => ctx.setParameter('labelField', value)}
              required
              hint="Field name in the API response to display to users"
              error={ctx.errors.labelField}
              placeholder="e.g., 'name' or 'title'"
            />
          </div>

          {/* Advanced Options */}
          <div style={{ marginBottom: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>Advanced Options</h4>
            
            <SwitchField
              id="enableSearch"
              name="enableSearch"
              label="Enable Search"
              value={parameters.enableSearch || false}
              onChange={(value) => ctx.setParameter('enableSearch', value)}
              hint="Allow users to search/filter the options"
            />

            <TextField
              id="cacheMinutes"
              name="cacheMinutes"
              label="Cache Duration (minutes)"
              value={String(parameters.cacheMinutes || 5)}
              onChange={(value) => ctx.setParameter('cacheMinutes', parseInt(value) || 5)}
              hint="How long to cache API responses"
              error={ctx.errors.cacheMinutes}
              textInputProps={{ type: 'number', min: 0 }}
            />
          </div>

          {/* Preview */}
          <div style={{
            padding: '15px',
            backgroundColor: '#f5f5f5',
            borderRadius: '8px',
          }}>
            <h4 style={{ marginBottom: '10px' }}>Configuration Summary</h4>
            <pre style={{
              fontSize: '12px',
              margin: 0,
              whiteSpace: 'pre-wrap',
            }}>
              {JSON.stringify(parameters, null, 2)}
            </pre>
          </div>
        </div>
      </Form>
    </Canvas>
  );
}
```

### 2. Custom Validation Rules Configuration

Configure validation rules for a custom field:

```typescript
async renderManualFieldExtensionConfigScreen(
  fieldExtensionId: string,
  ctx: RenderManualFieldExtensionConfigScreenCtx
) {
  if (fieldExtensionId === 'advancedValidator') {
    return {
      render: () => import('./components/AdvancedValidatorConfig'),
    };
  }
}

// AdvancedValidatorConfig.tsx
import { RenderManualFieldExtensionConfigScreenCtx } from 'datocms-plugin-sdk';
import { Canvas, Form, TextField, SelectField, TextareaField, Button } from 'datocms-react-ui';
import { useState } from 'react';
import { FaPlus, FaTrash } from 'react-icons/fa';

interface ValidationRule {
  id: string;
  type: 'regex' | 'length' | 'custom' | 'async';
  config: Record<string, any>;
  errorMessage: string;
}

export default function AdvancedValidatorConfig({ 
  ctx 
}: { 
  ctx: RenderManualFieldExtensionConfigScreenCtx 
}) {
  const [rules, setRules] = useState<ValidationRule[]>(
    (ctx.parameters.rules as ValidationRule[]) || []
  );

  const addRule = () => {
    const newRule: ValidationRule = {
      id: Date.now().toString(),
      type: 'regex',
      config: {},
      errorMessage: '',
    };
    
    const updatedRules = [...rules, newRule];
    setRules(updatedRules);
    ctx.setParameter('rules', updatedRules);
  };

  const updateRule = (id: string, updates: Partial<ValidationRule>) => {
    const updatedRules = rules.map(rule => 
      rule.id === id ? { ...rule, ...updates } : rule
    );
    setRules(updatedRules);
    ctx.setParameter('rules', updatedRules);
  };

  const removeRule = (id: string) => {
    const updatedRules = rules.filter(rule => rule.id !== id);
    setRules(updatedRules);
    ctx.setParameter('rules', updatedRules);
  };

  const testValidation = () => {
    const testValue = window.prompt('Enter a test value:');
    if (!testValue) return;

    const results = rules.map(rule => {
      try {
        let isValid = true;
        
        switch (rule.type) {
          case 'regex':
            const regex = new RegExp(rule.config.pattern);
            isValid = regex.test(testValue);
            break;
          case 'length':
            const length = testValue.length;
            if (rule.config.min && length < rule.config.min) isValid = false;
            if (rule.config.max && length > rule.config.max) isValid = false;
            break;
          case 'custom':
            // In real implementation, this would execute the custom function
            isValid = true;
            break;
        }

        return {
          rule: rule.errorMessage || `Rule ${rule.type}`,
          valid: isValid,
        };
      } catch (error) {
        return {
          rule: rule.errorMessage || `Rule ${rule.type}`,
          valid: false,
          error: error.message,
        };
      }
    });

    const message = results
      .map(r => `${r.valid ? '✅' : '❌'} ${r.rule}${r.error ? ` (${r.error})` : ''}`)
      .join('\n');
    
    alert('Validation Results:\n\n' + message);
  };

  return (
    <Canvas ctx={ctx}>
      <Form>
        <div style={{ padding: '20px' }}>
          <h3 style={{ marginBottom: '20px' }}>Advanced Validation Configuration</h3>
          
          <div style={{ marginBottom: '20px' }}>
            <p style={{ fontSize: '14px', color: '#666', marginBottom: '15px' }}>
              Configure validation rules that will be applied to this field. Rules are executed in order.
            </p>
          </div>

          {/* Validation Rules */}
          {rules.map((rule, index) => (
            <div
              key={rule.id}
              style={{
                padding: '15px',
                marginBottom: '15px',
                backgroundColor: '#f5f5f5',
                borderRadius: '8px',
                border: '1px solid #e0e0e0',
              }}
            >
              <div style={{ 
                display: 'flex', 
                justifyContent: 'space-between',
                alignItems: 'center',
                marginBottom: '15px'
              }}>
                <h4 style={{ margin: 0 }}>Rule {index + 1}</h4>
                <Button
                  onClick={() => removeRule(rule.id)}
                  buttonType="negative"
                  buttonSize="s"
                  leftIcon={<FaTrash />}
                >
                  Remove
                </Button>
              </div>

              <SelectField
                id={`type-${rule.id}`}
                name={`type-${rule.id}`}
                label="Validation Type"
                value={rule.type}
                selectInputProps={{
                  options: [
                    { label: 'Regular Expression', value: 'regex' },
                    { label: 'Length Validation', value: 'length' },
                    { label: 'Custom Function', value: 'custom' },
                    { label: 'Async Validation', value: 'async' },
                  ],
                  onChange: (value) => updateRule(rule.id, { type: value, config: {} }),
                }}
              />

              {/* Type-specific configuration */}
              {rule.type === 'regex' && (
                <TextField
                  id={`pattern-${rule.id}`}
                  name={`pattern-${rule.id}`}
                  label="Regular Expression Pattern"
                  value={rule.config.pattern || ''}
                  onChange={(value) => updateRule(rule.id, {
                    config: { ...rule.config, pattern: value }
                  })}
                  hint="e.g., ^[A-Z][a-z]+$ for capitalized words"
                  placeholder="Enter regex pattern"
                />
              )}

              {rule.type === 'length' && (
                <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: '10px' }}>
                  <TextField
                    id={`min-${rule.id}`}
                    name={`min-${rule.id}`}
                    label="Minimum Length"
                    value={String(rule.config.min || '')}
                    onChange={(value) => updateRule(rule.id, {
                      config: { ...rule.config, min: parseInt(value) || undefined }
                    })}
                    textInputProps={{ type: 'number', min: 0 }}
                  />
                  <TextField
                    id={`max-${rule.id}`}
                    name={`max-${rule.id}`}
                    label="Maximum Length"
                    value={String(rule.config.max || '')}
                    onChange={(value) => updateRule(rule.id, {
                      config: { ...rule.config, max: parseInt(value) || undefined }
                    })}
                    textInputProps={{ type: 'number', min: 0 }}
                  />
                </div>
              )}

              {rule.type === 'custom' && (
                <TextareaField
                  id={`function-${rule.id}`}
                  name={`function-${rule.id}`}
                  label="Validation Function"
                  value={rule.config.function || ''}
                  onChange={(value) => updateRule(rule.id, {
                    config: { ...rule.config, function: value }
                  })}
                  hint="JavaScript function that returns true if valid"
                  placeholder="(value, field, item) => { return value.length > 0; }"
                  textareaInputProps={{ rows: 4, style: { fontFamily: 'monospace' } }}
                />
              )}

              {rule.type === 'async' && (
                <>
                  <TextField
                    id={`endpoint-${rule.id}`}
                    name={`endpoint-${rule.id}`}
                    label="Validation Endpoint"
                    value={rule.config.endpoint || ''}
                    onChange={(value) => updateRule(rule.id, {
                      config: { ...rule.config, endpoint: value }
                    })}
                    hint="API endpoint that validates the value"
                    placeholder="https://api.example.com/validate"
                  />
                  <TextField
                    id={`method-${rule.id}`}
                    name={`method-${rule.id}`}
                    label="HTTP Method"
                    value={rule.config.method || 'POST'}
                    onChange={(value) => updateRule(rule.id, {
                      config: { ...rule.config, method: value }
                    })}
                    placeholder="POST"
                  />
                </>
              )}

              <TextField
                id={`error-${rule.id}`}
                name={`error-${rule.id}`}
                label="Error Message"
                value={rule.errorMessage}
                onChange={(value) => updateRule(rule.id, { errorMessage: value })}
                required
                hint="Message shown when validation fails"
                placeholder="Please enter a valid value"
              />
            </div>
          ))}

          {/* Add Rule Button */}
          <div style={{ marginBottom: '20px' }}>
            <Button
              onClick={addRule}
              buttonType="primary"
              leftIcon={<FaPlus />}
            >
              Add Validation Rule
            </Button>
          </div>

          {/* Test Validation */}
          {rules.length > 0 && (
            <div style={{
              padding: '15px',
              backgroundColor: '#E3F2FD',
              borderRadius: '8px',
              marginTop: '20px',
            }}>
              <h4 style={{ marginBottom: '10px' }}>Test Your Validation</h4>
              <p style={{ fontSize: '13px', marginBottom: '10px' }}>
                Test how your validation rules work with sample values.
              </p>
              <Button onClick={testValidation}>
                Test Validation Rules
              </Button>
            </div>
          )}

          {/* Additional Options */}
          <div style={{ marginTop: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>Additional Options</h4>
            
            <SelectField
              id="validationTrigger"
              name="validationTrigger"
              label="Validation Trigger"
              value={ctx.parameters.validationTrigger || 'onChange'}
              selectInputProps={{
                options: [
                  { label: 'On Change', value: 'onChange' },
                  { label: 'On Blur', value: 'onBlur' },
                  { label: 'On Submit', value: 'onSubmit' },
                ],
                onChange: (value) => ctx.setParameter('validationTrigger', value),
              }}
              hint="When to run validation"
            />

            <SwitchField
              id="showInlineErrors"
              name="showInlineErrors"
              label="Show Inline Errors"
              value={ctx.parameters.showInlineErrors !== false}
              onChange={(value) => ctx.setParameter('showInlineErrors', value)}
              hint="Display validation errors next to the field"
            />
          </div>
        </div>
      </Form>
    </Canvas>
  );
}
```

### 3. UI Customization Configuration

Configure the appearance and behavior of a field extension:

```typescript
async renderManualFieldExtensionConfigScreen(
  fieldExtensionId: string,
  ctx: RenderManualFieldExtensionConfigScreenCtx
) {
  if (fieldExtensionId === 'customRichText') {
    return {
      render: () => import('./components/CustomRichTextConfig'),
    };
  }
}

// CustomRichTextConfig.tsx
import { RenderManualFieldExtensionConfigScreenCtx } from 'datocms-plugin-sdk';
import { Canvas, Form, SwitchField, SelectField, TextField } from 'datocms-react-ui';
import { useState } from 'react';

export default function CustomRichTextConfig({ 
  ctx 
}: { 
  ctx: RenderManualFieldExtensionConfigScreenCtx 
}) {
  const [previewMode, setPreviewMode] = useState<'live' | 'side' | 'tab'>(
    ctx.parameters.previewMode as any || 'live'
  );

  const toolbarOptions = [
    { id: 'bold', label: 'Bold', default: true },
    { id: 'italic', label: 'Italic', default: true },
    { id: 'underline', label: 'Underline', default: false },
    { id: 'strike', label: 'Strikethrough', default: false },
    { id: 'heading', label: 'Headings', default: true },
    { id: 'quote', label: 'Blockquote', default: true },
    { id: 'code', label: 'Code', default: true },
    { id: 'list', label: 'Lists', default: true },
    { id: 'link', label: 'Links', default: true },
    { id: 'image', label: 'Images', default: true },
    { id: 'video', label: 'Videos', default: false },
    { id: 'table', label: 'Tables', default: false },
    { id: 'emoji', label: 'Emoji Picker', default: false },
    { id: 'math', label: 'Math Formulas', default: false },
  ];

  const enabledTools = ctx.parameters.enabledTools as string[] || 
    toolbarOptions.filter(t => t.default).map(t => t.id);

  return (
    <Canvas ctx={ctx}>
      <Form>
        <div style={{ padding: '20px' }}>
          <h3 style={{ marginBottom: '20px' }}>Rich Text Editor Configuration</h3>

          {/* Editor Layout */}
          <div style={{ marginBottom: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>Editor Layout</h4>
            
            <SelectField
              id="previewMode"
              name="previewMode"
              label="Preview Mode"
              value={previewMode}
              selectInputProps={{
                options: [
                  { label: 'Live Preview (WYSIWYG)', value: 'live' },
                  { label: 'Side-by-side Preview', value: 'side' },
                  { label: 'Tab Preview', value: 'tab' },
                ],
                onChange: (value) => {
                  setPreviewMode(value);
                  ctx.setParameter('previewMode', value);
                },
              }}
              hint="How to display the preview of formatted content"
            />

            <TextField
              id="minHeight"
              name="minHeight"
              label="Minimum Height (px)"
              value={String(ctx.parameters.minHeight || 200)}
              onChange={(value) => ctx.setParameter('minHeight', parseInt(value) || 200)}
              textInputProps={{ type: 'number', min: 100, max: 800 }}
              hint="Minimum height of the editor"
            />

            <TextField
              id="maxHeight"
              name="maxHeight"
              label="Maximum Height (px)"
              value={String(ctx.parameters.maxHeight || 600)}
              onChange={(value) => ctx.setParameter('maxHeight', parseInt(value) || 600)}
              textInputProps={{ type: 'number', min: 200, max: 1200 }}
              hint="Maximum height before scrolling"
            />
          </div>

          {/* Toolbar Configuration */}
          <div style={{ marginBottom: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>Toolbar Options</h4>
            <p style={{ fontSize: '13px', color: '#666', marginBottom: '15px' }}>
              Select which formatting options to include in the toolbar.
            </p>
            
            <div style={{
              display: 'grid',
              gridTemplateColumns: 'repeat(2, 1fr)',
              gap: '10px',
            }}>
              {toolbarOptions.map(option => (
                <label
                  key={option.id}
                  style={{
                    display: 'flex',
                    alignItems: 'center',
                    padding: '8px',
                    backgroundColor: '#f5f5f5',
                    borderRadius: '4px',
                    cursor: 'pointer',
                  }}
                >
                  <input
                    type="checkbox"
                    checked={enabledTools.includes(option.id)}
                    onChange={(e) => {
                      const newTools = e.target.checked
                        ? [...enabledTools, option.id]
                        : enabledTools.filter(t => t !== option.id);
                      ctx.setParameter('enabledTools', newTools);
                    }}
                    style={{ marginRight: '8px' }}
                  />
                  {option.label}
                </label>
              ))}
            </div>
          </div>

          {/* Advanced Features */}
          <div style={{ marginBottom: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>Advanced Features</h4>
            
            <SwitchField
              id="enableMarkdown"
              name="enableMarkdown"
              label="Enable Markdown Shortcuts"
              value={ctx.parameters.enableMarkdown !== false}
              onChange={(value) => ctx.setParameter('enableMarkdown', value)}
              hint="Allow typing markdown syntax for quick formatting"
            />

            <SwitchField
              id="enableAutoSave"
              name="enableAutoSave"
              label="Auto-save Content"
              value={ctx.parameters.enableAutoSave === true}
              onChange={(value) => ctx.setParameter('enableAutoSave', value)}
              hint="Automatically save content as user types"
            />

            <SwitchField
              id="enableWordCount"
              name="enableWordCount"
              label="Show Word Count"
              value={ctx.parameters.enableWordCount !== false}
              onChange={(value) => ctx.setParameter('enableWordCount', value)}
              hint="Display word and character count"
            />

            <SwitchField
              id="enableFullscreen"
              name="enableFullscreen"
              label="Allow Fullscreen Mode"
              value={ctx.parameters.enableFullscreen !== false}
              onChange={(value) => ctx.setParameter('enableFullscreen', value)}
              hint="Add fullscreen editing option"
            />
          </div>

          {/* Custom Styles */}
          <div style={{ marginBottom: '30px' }}>
            <h4 style={{ marginBottom: '15px' }}>Custom Styles</h4>
            
            <TextField
              id="fontFamily"
              name="fontFamily"
              label="Font Family"
              value={ctx.parameters.fontFamily || ''}
              onChange={(value) => ctx.setParameter('fontFamily', value)}
              placeholder="Arial, sans-serif"
              hint="CSS font-family for the editor content"
            />

            <TextField
              id="fontSize"
              name="fontSize"
              label="Font Size"
              value={ctx.parameters.fontSize || ''}
              onChange={(value) => ctx.setParameter('fontSize', value)}
              placeholder="16px"
              hint="Default font size for content"
            />

            <TextareaField
              id="customCSS"
              name="customCSS"
              label="Custom CSS"
              value={ctx.parameters.customCSS || ''}
              onChange={(value) => ctx.setParameter('customCSS', value)}
              hint="Additional CSS to apply to the editor"
              textareaInputProps={{ 
                rows: 4,
                style: { fontFamily: 'monospace' },
                placeholder: '.editor-content { line-height: 1.6; }'
              }}
            />
          </div>

          {/* Preview */}
          <div style={{
            padding: '20px',
            backgroundColor: '#f5f5f5',
            borderRadius: '8px',
          }}>
            <h4 style={{ marginBottom: '15px' }}>Configuration Preview</h4>
            
            <div style={{
              padding: '15px',
              backgroundColor: 'white',
              borderRadius: '4px',
              border: '1px solid #ddd',
              minHeight: ctx.parameters.minHeight || 200,
              maxHeight: ctx.parameters.maxHeight || 600,
              fontFamily: ctx.parameters.fontFamily || 'inherit',
              fontSize: ctx.parameters.fontSize || '14px',
            }}>
              <div style={{ marginBottom: '10px' }}>
                <strong>Enabled Tools:</strong> {enabledTools.join(', ')}
              </div>
              <div style={{ marginBottom: '10px' }}>
                <strong>Preview Mode:</strong> {previewMode}
              </div>
              <div>
                <strong>Features:</strong> 
                {ctx.parameters.enableMarkdown !== false && ' Markdown'}
                {ctx.parameters.enableAutoSave && ', Auto-save'}
                {ctx.parameters.enableWordCount !== false && ', Word Count'}
                {ctx.parameters.enableFullscreen !== false && ', Fullscreen'}
              </div>
            </div>
          </div>
        </div>
      </Form>
    </Canvas>
  );
}
```

## Best Practices

1. **Clear Labels**: Use descriptive labels and hints for all configuration options
2. **Validation**: Validate configuration in real-time and show clear error messages
3. **Defaults**: Provide sensible default values for all parameters
4. **Testing**: Include ways to test the configuration (test buttons, previews)
5. **Organization**: Group related settings into logical sections
6. **Help Text**: Provide helpful hints and examples for complex settings
7. **Visual Feedback**: Show previews or examples of how settings affect behavior
8. **Error Prevention**: Validate inputs to prevent invalid configurations
9. **Progressive Disclosure**: Hide advanced options behind expandable sections
10. **Save State**: Configuration is automatically saved as users make changes

## Related Hooks

- `manualFieldExtensions` - Define which field extensions your plugin provides
- `renderFieldExtension` - Render the actual field extension
- `validateManualFieldExtensionParameters` - Validate configuration parameters
- `fieldDropdownActions` - Add actions to field configuration

## Important Notes

- Configuration screens appear in the field settings modal
- Parameters are automatically saved as the user configures them
- Use the Canvas component for consistent styling
- The configuration should be intuitive for non-technical users
- Consider providing presets for common configurations
- Configuration changes take effect immediately after saving
- Validate parameters to ensure the field extension works correctly