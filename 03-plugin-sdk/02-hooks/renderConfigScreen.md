# renderConfigScreen Hook

## Purpose

The `renderConfigScreen` hook renders the plugin's global configuration interface, accessible from the plugin settings page in DatoCMS. This is where users configure API keys, set default behaviors, manage integrations, and customize how the plugin operates across the entire project.

## Signature

```typescript
renderConfigScreen(ctx: RenderConfigScreenCtx): void
```

## Parameters

- `ctx`: RenderConfigScreenCtx - The context object containing all necessary data and methods for rendering the configuration screen

## Type Definitions

```typescript
// Hook signature
export type RenderConfigScreenHook = {
  renderConfigScreen: (ctx: RenderConfigScreenCtx) => void;
};

// Context type
export type RenderConfigScreenCtx = SelfResizingPluginFrameCtx<'renderConfigScreen'>;

// Expanded full context type
type RenderConfigScreenCtx = {
  // Mode and frame properties
  mode: 'renderConfigScreen';
  bodyPadding: [number, number, number, number];
  
  // Base properties
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  currentUserAccessToken: string | undefined;
  site: Site;
  environment: string;
  environmentId: string;
  isEnvironmentPrimary: boolean;
  mainLocale: string;
  currentUserMenuLocale: string;
  owner: Account | Organization;
  account: Account | undefined; // deprecated
  ui: { locale: string };
  theme: Theme;
  itemTypes: Partial<Record<string, ItemType>>;
  fields: Partial<Record<string, Field>>;
  fieldsets: Partial<Record<string, Fieldset>>;
  users: Partial<Record<string, User>>;
  ssoUsers: Partial<Record<string, SsoUser>>;
  
  // Base methods
  loadItemTypeFields: (itemTypeId: string) => Promise<Field[]>;
  loadItemTypeFieldsets: (itemTypeId: string) => Promise<Fieldset[]>;
  loadFieldsUsingPlugin: () => Promise<Field[]>;
  loadUsers: () => Promise<User[]>;
  loadSsoUsers: () => Promise<SsoUser[]>;
  updatePluginParameters: (params: Record<string, unknown>) => Promise<void>;
  updateFieldAppearance: (fieldId: string, changes: FieldAppearanceChange[]) => Promise<void>;
  getFieldIntlTitle: (field: Field, locale?: string, returnUntranslatedTitle?: boolean) => string | null;
  createClient: (options?: { apiToken?: string; environment?: string }) => SiteApiClient;
  
  // Item dialogs
  createNewItem: (itemTypeId: string) => Promise<Item | null>;
  selectItem: {
    (itemTypeId: string, options: { multiple: true; initialLocationQuery?: ItemListLocationQuery }): Promise<Item[] | null>;
    (itemTypeId: string, options?: { multiple: false; initialLocationQuery?: ItemListLocationQuery }): Promise<Item | null>;
  };
  editItem: (itemId: string) => Promise<Item | null>;
  
  // Upload dialogs
  selectUpload: {
    (options: { multiple: true }): Promise<Upload[] | null>;
    (options?: { multiple: false }): Promise<Upload | null>;
  };
  editUpload: (uploadId: string) => Promise<(Upload & { deleted?: true }) | null>;
  editUploadMetadata: (fileFieldValue: FileFieldValue, locale?: string) => Promise<FileFieldValue | null>;
  
  // Notifications
  alert: (message: string) => Promise<void>;
  notice: (message: string) => Promise<void>;
  customToast: <CtaValue = unknown>(toast: Toast<CtaValue>) => Promise<CtaValue | null>;
  
  // Modals and navigation
  openModal: (modal: Modal) => Promise<unknown>;
  openConfirm: (options: ConfirmOptions) => Promise<unknown>;
  navigateTo: (path: string) => Promise<void>;
  
  // Configuration specific
  parameters: Record<string, unknown>;
  setParameters: (params: Record<string, unknown>) => Promise<void>;
  getParameters: () => Record<string, unknown>;
  
  // Sizing utilities
  startAutoResizer: () => Promise<void>;
  stopAutoResizer: () => Promise<void>;
  updateHeight: (height: number) => Promise<void>;
  isAutoResizerActive: () => boolean;
  setHeight: (height: number) => Promise<void>;
  
  // Frame communication
  getSettings: () => Promise<RenderConfigScreenCtx>;
};

// Supporting types
type Theme = {
  primaryColor: string;
  accentColor: string;
  semiTransparentAccentColor: string;
  lightColor: string;
  darkColor: string;
};

type Toast<CtaValue = unknown> = {
  message: string;
  type: 'notice' | 'alert' | 'warning';
  cta?: {
    label: string;
    value: CtaValue;
  };
  dismissOnPageChange?: boolean;
  dismissAfterTimeout?: boolean | number;
};

type Modal = {
  id: string;
  title?: string;
  closeDisabled?: boolean;
  width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth' | number;
  parameters?: Record<string, unknown>;
  initialHeight?: number;
};

type ConfirmOptions = {
  title: string;
  content: string;
  choices: ConfirmChoice[];
  cancel: ConfirmChoice;
};

type ConfirmChoice = {
  label: string;
  value: unknown;
  intent?: 'positive' | 'negative';
};

type FileFieldValue = {
  upload_id: string;
  alt: string | null;
  title: string | null;
  focal_point: FocalPoint | null;
  custom_data: Record<string, string>;
};

type FocalPoint = {
  x: number;
  y: number;
};

type ItemListLocationQuery = {
  locale?: string;
  filter?: {
    query?: string;
    fields?: Record<string, unknown>;
  };
};

type FieldAppearanceChange =
  | { operation: 'removeEditor' }
  | { operation: 'updateEditor'; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'setEditor'; fieldExtensionId: string; parameters: Record<string, unknown> }
  | { operation: 'removeAddon'; index: number }
  | { operation: 'updateAddon'; index: number; newFieldExtensionId?: string; newParameters?: Record<string, unknown> }
  | { operation: 'insertAddon'; index: number; fieldExtensionId: string; parameters: Record<string, unknown> };

type Plugin = {
  id: string;
  attributes: {
    name: string;
    description?: string;
    homepage_url?: string;
    preview_image?: string;
    parameters: Record<string, unknown>;
    field_types?: string[];
    icon?: string;
  };
};

type User = {
  id: string;
  type: 'user';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    full_name: string;
    is_2fa_active: boolean;
    created_at: string;
  };
};

type SsoUser = {
  id: string;
  type: 'sso_user';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    full_name: string;
    created_at: string;
  };
};

type Account = {
  id: string;
  type: 'account';
  attributes: {
    email: string;
    first_name: string | null;
    last_name: string | null;
    full_name: string;
    created_at: string;
    company: string | null;
  };
};

type Organization = {
  id: string;
  type: 'organization';
  attributes: {
    name: string;
    created_at: string;
  };
};

type Role = {
  id: string;
  type: 'role';
  attributes: {
    name: string;
    can_edit_schema: boolean;
    can_edit_site: boolean;
    can_manage_users: boolean;
    can_publish_to_production: boolean;
    can_manage_webhooks: boolean;
    can_manage_access_tokens: boolean;
    environments_access: 'all' | 'primary_only' | 'sandbox_only';
    positive_item_type_permissions: Array<{
      item_type: string;
      environment: string;
      action: 'all' | 'read' | 'edit' | 'create' | 'delete' | 'publish';
    }>;
    negative_item_type_permissions: Array<{
      item_type: string;
      environment: string;
      action: 'all' | 'read' | 'edit' | 'create' | 'delete' | 'publish';
    }>;
  };
};

type Site = {
  id: string;
  type: 'site';
  attributes: {
    name: string;
    internal_subdomain: string;
    domain: string | null;
    frontend_url: string | null;
    global_seo: object | null;
    favicon: object | null;
    locales: string[];
    timezone: string;
  };
};

type ItemType = {
  id: string;
  type: 'item_type';
  attributes: {
    name: string;
    api_key: string;
    collection_appearance: 'table' | 'gallery';
    all_locales_required: boolean;
    has_singleton_item: boolean;
    sortable: boolean;
    modular_block: boolean;
    tree: boolean;
    draft_mode_active: boolean;
  };
};

type Field = {
  id: string;
  type: 'field';
  attributes: {
    label: string;
    api_key: string;
    field_type: string;
    appearance: Record<string, unknown>;
    validators: Record<string, unknown>;
    position: number;
    hint: string | null;
    default_value: unknown;
    localized: boolean;
  };
};

type Fieldset = {
  id: string;
  type: 'fieldset';
  attributes: {
    title: string;
    hint: string | null;
    position: number;
    collapsible: boolean;
    start_collapsed: boolean;
  };
};

type Item = {
  id: string;
  type: 'item';
  attributes: Record<string, unknown>;
  meta: {
    created_at: string;
    updated_at: string;
    published_at: string | null;
    first_published_at: string | null;
    publication_scheduled_at: string | null;
    unpublishing_scheduled_at: string | null;
    status: 'draft' | 'published' | 'updated';
    is_current_version_valid: boolean;
    is_published_version_valid: boolean;
    current_version: string;
    stage: string | null;
  };
};

type Upload = {
  id: string;
  type: 'upload';
  attributes: {
    path: string;
    basename: string;
    filename: string;
    url: string;
    size: number;
    width: number | null;
    height: number | null;
    format: string;
    alt: string | null;
    title: string | null;
    notes: string | null;
    md5: string;
    created_at: string;
    tags: string[];
    smart_tags: string[];
    colors: Array<{ hex: string; alpha: number }>;
    blurhash: string | null;
    thumbhash: string | null;
    is_image: boolean;
  };
};

type SiteApiClient = {
  items: {
    list(params?: object): Promise<Item[]>;
    find(id: string): Promise<Item>;
    create(params: object): Promise<Item>;
    update(id: string, params: object): Promise<Item>;
    destroy(id: string): Promise<void>;
    publish(id: string): Promise<Item>;
    unpublish(id: string): Promise<Item>;
  };
  uploads: {
    list(params?: object): Promise<Upload[]>;
    find(id: string): Promise<Upload>;
    create(params: object): Promise<Upload>;
    update(id: string, params: object): Promise<Upload>;
    destroy(id: string): Promise<void>;
  };
};
```

## Context Properties

The hook receives a context object that includes:

### Base Properties
All standard context properties like `plugin`, `currentUser`, `site`, etc.

### Configuration-Specific Properties
- `parameters`: Record<string, unknown> - Current plugin configuration parameters

### Configuration Methods
- `setParameters(params: Record<string, unknown>)`: Promise<void> - Update plugin configuration
- `getParameters()`: Record<string, unknown> - Get current parameters

### Frame Properties
- Full-screen rendering context
- Automatic height adjustment capabilities

## Use Cases

### 1. API Integration Configuration

Configure external service connections:

```typescript
renderConfigScreen(ctx: RenderConfigScreenCtx) {
  return {
    render: () => import('./components/ConfigScreen'),
  };
}

// ConfigScreen.tsx
import { RenderConfigScreenCtx } from 'datocms-plugin-sdk';
import { Canvas, Form, TextField, SwitchField, Button, Section } from 'datocms-react-ui';
import { useState, useEffect } from 'react';

export default function ConfigScreen({ ctx }: { ctx: RenderConfigScreenCtx }) {
  const [config, setConfig] = useState(ctx.parameters);
  const [testingConnection, setTestingConnection] = useState(false);
  const [hasChanges, setHasChanges] = useState(false);

  useEffect(() => {
    // Track changes
    const configChanged = JSON.stringify(config) !== JSON.stringify(ctx.parameters);
    setHasChanges(configChanged);
  }, [config, ctx.parameters]);

  const handleTestConnection = async () => {
    if (!config.apiEndpoint || !config.apiKey) {
      ctx.alert('Please provide both API endpoint and key');
      return;
    }

    setTestingConnection(true);
    
    try {
      const response = await fetch(`${config.apiEndpoint}/health`, {
        headers: {
          'Authorization': `Bearer ${config.apiKey}`,
          'Content-Type': 'application/json'
        }
      });

      if (response.ok) {
        const data = await response.json();
        ctx.notice(`✅ Connection successful! API Version: ${data.version}`);
      } else {
        throw new Error(`API returned ${response.status}`);
      }
    } catch (error) {
      ctx.alert(`❌ Connection failed: ${error.message}`);
    } finally {
      setTestingConnection(false);
    }
  };

  const handleSave = async () => {
    // Validate configuration
    const errors = [];
    
    if (!config.apiEndpoint) {
      errors.push('API Endpoint is required');
    } else {
      try {
        new URL(config.apiEndpoint);
      } catch {
        errors.push('API Endpoint must be a valid URL');
      }
    }

    if (!config.apiKey) {
      errors.push('API Key is required');
    }

    if (errors.length > 0) {
      ctx.alert(errors.join('\n'));
      return;
    }

    try {
      await ctx.setParameters(config);
      ctx.notice('Configuration saved successfully');
      setHasChanges(false);
    } catch (error) {
      ctx.alert('Failed to save configuration');
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ maxWidth: '800px', margin: '0 auto', padding: '40px 20px' }}>
        <h1>Plugin Configuration</h1>
        
        <Form>
          <Section title="API Settings">
            <TextField
              id="apiEndpoint"
              label="API Endpoint"
              value={config.apiEndpoint || ''}
              onChange={(value) => setConfig({ ...config, apiEndpoint: value })}
              placeholder="https://api.example.com/v1"
              hint="The base URL for your API"
              required
            />

            <TextField
              id="apiKey"
              label="API Key"
              value={config.apiKey || ''}
              onChange={(value) => setConfig({ ...config, apiKey: value })}
              placeholder="sk_live_..."
              hint="Your secret API key"
              required
              textInputProps={{
                type: 'password'
              }}
            />

            <Button
              onClick={handleTestConnection}
              disabled={testingConnection}
              buttonType="muted"
              buttonSize="s"
            >
              {testingConnection ? 'Testing...' : 'Test Connection'}
            </Button>
          </Section>

          <Section title="Feature Toggles">
            <SwitchField
              id="enableAnalytics"
              label="Enable Analytics"
              value={config.enableAnalytics || false}
              onChange={(value) => setConfig({ ...config, enableAnalytics: value })}
              hint="Track usage statistics and performance metrics"
            />

            <SwitchField
              id="enableAI"
              label="Enable AI Features"
              value={config.enableAI || false}
              onChange={(value) => setConfig({ ...config, enableAI: value })}
              hint="Use AI for content suggestions and improvements"
            />

            <SwitchField
              id="debugMode"
              label="Debug Mode"
              value={config.debugMode || false}
              onChange={(value) => setConfig({ ...config, debugMode: value })}
              hint="Enable verbose logging for troubleshooting"
            />
          </Section>

          <Section title="Default Behaviors">
            <SelectField
              id="defaultLanguage"
              label="Default Language"
              value={config.defaultLanguage || 'en'}
              onChange={(value) => setConfig({ ...config, defaultLanguage: value })}
              selectInputProps={{
                options: [
                  { label: 'English', value: 'en' },
                  { label: 'Spanish', value: 'es' },
                  { label: 'French', value: 'fr' },
                  { label: 'German', value: 'de' },
                  { label: 'Italian', value: 'it' }
                ]
              }}
            />

            <TextField
              id="defaultImageQuality"
              label="Default Image Quality (%)"
              value={config.defaultImageQuality || '85'}
              onChange={(value) => setConfig({ ...config, defaultImageQuality: value })}
              hint="Quality for image optimization (1-100)"
              textInputProps={{
                type: 'number',
                min: 1,
                max: 100
              }}
            />

            <TextareaField
              id="customCSS"
              label="Custom CSS"
              value={config.customCSS || ''}
              onChange={(value) => setConfig({ ...config, customCSS: value })}
              hint="Add custom styles to field extensions"
              textareaInputProps={{
                rows: 5,
                style: { fontFamily: 'monospace' }
              }}
            />
          </Section>

          <div style={{ 
            marginTop: '40px', 
            display: 'flex', 
            gap: '10px',
            justifyContent: 'flex-end'
          }}>
            <Button
              onClick={() => setConfig(ctx.parameters)}
              buttonType="muted"
              disabled={!hasChanges}
            >
              Reset
            </Button>
            <Button
              onClick={handleSave}
              buttonType="primary"
              disabled={!hasChanges}
            >
              Save Configuration
            </Button>
          </div>
        </Form>
      </div>
    </Canvas>
  );
}
```

### 2. Multi-Service Configuration

Manage multiple service integrations:

```typescript
// ConfigScreen.tsx
export default function ConfigScreen({ ctx }: { ctx: RenderConfigScreenCtx }) {
  const [activeService, setActiveService] = useState('stripe');
  const [services, setServices] = useState(ctx.parameters.services || {
    stripe: { enabled: false },
    sendgrid: { enabled: false },
    slack: { enabled: false },
    github: { enabled: false }
  });

  const ServiceConfig = ({ service, config, onChange }) => {
    switch (service) {
      case 'stripe':
        return (
          <StripeConfig
            config={config}
            onChange={onChange}
            onTest={() => testStripeConnection(config)}
          />
        );
      case 'sendgrid':
        return (
          <SendGridConfig
            config={config}
            onChange={onChange}
            onTest={() => testSendGridConnection(config)}
          />
        );
      case 'slack':
        return (
          <SlackConfig
            config={config}
            onChange={onChange}
            onTest={() => testSlackConnection(config)}
          />
        );
      case 'github':
        return (
          <GitHubConfig
            config={config}
            onChange={onChange}
            onTest={() => testGitHubConnection(config)}
          />
        );
    }
  };

  const handleServiceUpdate = (service: string, config: any) => {
    setServices({
      ...services,
      [service]: config
    });
  };

  const handleSave = async () => {
    await ctx.setParameters({
      ...ctx.parameters,
      services
    });
    ctx.notice('Services configuration saved');
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ display: 'flex', height: '100vh' }}>
        {/* Service Sidebar */}
        <div style={{ 
          width: '250px', 
          borderRight: '1px solid #e0e0e0',
          padding: '20px'
        }}>
          <h2>Services</h2>
          
          {Object.entries(services).map(([key, config]) => (
            <ServiceItem
              key={key}
              name={key}
              enabled={config.enabled}
              active={activeService === key}
              onClick={() => setActiveService(key)}
              onToggle={(enabled) => handleServiceUpdate(key, { ...config, enabled })}
            />
          ))}
        </div>

        {/* Service Configuration */}
        <div style={{ flex: 1, padding: '40px' }}>
          <ServiceConfig
            service={activeService}
            config={services[activeService]}
            onChange={(config) => handleServiceUpdate(activeService, config)}
          />
          
          <div style={{ marginTop: '40px' }}>
            <Button onClick={handleSave} buttonType="primary">
              Save All Services
            </Button>
          </div>
        </div>
      </div>
    </Canvas>
  );
}
```

### 3. Advanced Settings with Tabs

Organize complex configurations with tabs:

```typescript
export default function ConfigScreen({ ctx }: { ctx: RenderConfigScreenCtx }) {
  const [activeTab, setActiveTab] = useState('general');
  const [config, setConfig] = useState(ctx.parameters);

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px' }}>
        <h1>Plugin Settings</h1>
        
        <Tabs value={activeTab} onChange={setActiveTab}>
          <Tab value="general" label="General" />
          <Tab value="api" label="API Configuration" />
          <Tab value="webhooks" label="Webhooks" />
          <Tab value="advanced" label="Advanced" />
          <Tab value="about" label="About" />
        </Tabs>

        <div style={{ marginTop: '30px' }}>
          {activeTab === 'general' && (
            <GeneralSettings 
              config={config} 
              onChange={setConfig}
              ctx={ctx}
            />
          )}

          {activeTab === 'api' && (
            <ApiSettings 
              config={config} 
              onChange={setConfig}
              ctx={ctx}
            />
          )}

          {activeTab === 'webhooks' && (
            <WebhookSettings 
              config={config} 
              onChange={setConfig}
              ctx={ctx}
            />
          )}

          {activeTab === 'advanced' && (
            <AdvancedSettings 
              config={config} 
              onChange={setConfig}
              ctx={ctx}
            />
          )}

          {activeTab === 'about' && (
            <AboutSection ctx={ctx} />
          )}
        </div>

        <SaveBar
          hasChanges={JSON.stringify(config) !== JSON.stringify(ctx.parameters)}
          onSave={() => ctx.setParameters(config)}
          onReset={() => setConfig(ctx.parameters)}
        />
      </div>
    </Canvas>
  );
}
```

### 4. Configuration Import/Export

Allow users to backup and restore settings:

```typescript
export default function ConfigScreen({ ctx }: { ctx: RenderConfigScreenCtx }) {
  const [config, setConfig] = useState(ctx.parameters);

  const handleExport = () => {
    const exportData = {
      version: '1.0',
      timestamp: new Date().toISOString(),
      environment: ctx.environment,
      config: config
    };

    const blob = new Blob([JSON.stringify(exportData, null, 2)], {
      type: 'application/json'
    });
    
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `plugin-config-${Date.now()}.json`;
    a.click();
    
    ctx.notice('Configuration exported successfully');
  };

  const handleImport = async (file: File) => {
    try {
      const text = await file.text();
      const importData = JSON.parse(text);
      
      // Validate import
      if (!importData.version || !importData.config) {
        throw new Error('Invalid configuration file');
      }

      const confirmed = await ctx.openConfirm({
        title: 'Import Configuration',
        content: `This will overwrite your current configuration. The file was exported on ${new Date(importData.timestamp).toLocaleDateString()}. Continue?`,
        choices: [
          { label: 'Cancel', value: false },
          { label: 'Import', value: true, intent: 'negative' }
        ]
      });

      if (confirmed) {
        setConfig(importData.config);
        ctx.notice('Configuration imported successfully');
      }
    } catch (error) {
      ctx.alert(`Import failed: ${error.message}`);
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '40px', maxWidth: '800px', margin: '0 auto' }}>
        <Section title="Configuration Management">
          <div style={{ display: 'flex', gap: '20px', marginBottom: '30px' }}>
            <div style={{ flex: 1, padding: '20px', border: '1px solid #e0e0e0', borderRadius: '4px' }}>
              <h3>Export Configuration</h3>
              <p>Download current configuration as a JSON file</p>
              <Button onClick={handleExport} buttonType="primary">
                Export Settings
              </Button>
            </div>

            <div style={{ flex: 1, padding: '20px', border: '1px solid #e0e0e0', borderRadius: '4px' }}>
              <h3>Import Configuration</h3>
              <p>Restore configuration from a JSON file</p>
              <FileInput
                accept=".json"
                onChange={handleImport}
                buttonText="Import Settings"
              />
            </div>
          </div>
        </Section>

        {/* Rest of configuration UI */}
      </div>
    </Canvas>
  );
}
```

### 5. Guided Setup Wizard

Create an onboarding experience for new users:

```typescript
export default function ConfigScreen({ ctx }: { ctx: RenderConfigScreenCtx }) {
  const [setupComplete, setSetupComplete] = useState(ctx.parameters.setupComplete || false);
  const [currentStep, setCurrentStep] = useState(1);
  const [wizardData, setWizardData] = useState({});

  if (!setupComplete) {
    return (
      <Canvas ctx={ctx}>
        <SetupWizard
          currentStep={currentStep}
          onStepChange={setCurrentStep}
          data={wizardData}
          onDataChange={setWizardData}
          onComplete={async (data) => {
            await ctx.setParameters({
              ...data,
              setupComplete: true
            });
            setSetupComplete(true);
            ctx.notice('Setup completed successfully!');
          }}
        />
      </Canvas>
    );
  }

  // Regular configuration screen
  return (
    <Canvas ctx={ctx}>
      <ConfigurationInterface ctx={ctx} />
    </Canvas>
  );
}

const SetupWizard = ({ currentStep, onStepChange, data, onDataChange, onComplete }) => {
  const steps = [
    { id: 1, title: 'Welcome', component: WelcomeStep },
    { id: 2, title: 'API Setup', component: ApiSetupStep },
    { id: 3, title: 'Features', component: FeaturesStep },
    { id: 4, title: 'Review', component: ReviewStep }
  ];

  const CurrentStepComponent = steps[currentStep - 1].component;

  return (
    <div style={{ maxWidth: '600px', margin: '0 auto', padding: '40px' }}>
      <StepIndicator steps={steps} currentStep={currentStep} />
      
      <CurrentStepComponent
        data={data}
        onUpdate={onDataChange}
        onNext={() => {
          if (currentStep < steps.length) {
            onStepChange(currentStep + 1);
          } else {
            onComplete(data);
          }
        }}
        onPrev={() => currentStep > 1 && onStepChange(currentStep - 1)}
        isLastStep={currentStep === steps.length}
      />
    </div>
  );
};
```

### 6. Environment-Specific Configuration

Manage different settings per environment:

```typescript
export default function ConfigScreen({ ctx }: { ctx: RenderConfigScreenCtx }) {
  const [environments, setEnvironments] = useState(
    ctx.parameters.environments || {
      [ctx.environment]: {}
    }
  );
  const [selectedEnv, setSelectedEnv] = useState(ctx.environment);

  const handleSave = async () => {
    await ctx.setParameters({
      ...ctx.parameters,
      environments,
      currentEnvironment: ctx.environment
    });
    ctx.notice('Configuration saved');
  };

  const copyFromEnvironment = async (source: string) => {
    const confirmed = await ctx.openConfirm({
      title: 'Copy Configuration',
      content: `Copy all settings from ${source} to ${selectedEnv}?`,
      choices: [
        { label: 'Cancel', value: false },
        { label: 'Copy', value: true }
      ]
    });

    if (confirmed) {
      setEnvironments({
        ...environments,
        [selectedEnv]: { ...environments[source] }
      });
      ctx.notice('Configuration copied');
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ padding: '20px' }}>
        <div style={{ marginBottom: '20px' }}>
          <label>Configure for environment:</label>
          <SelectInput
            value={selectedEnv}
            onChange={setSelectedEnv}
            options={Object.keys(environments).map(env => ({
              label: env === 'main' ? 'Production' : env,
              value: env
            }))}
          />
          
          {selectedEnv !== ctx.environment && (
            <WarningBanner>
              You're editing configuration for {selectedEnv} environment
            </WarningBanner>
          )}
        </div>

        <EnvironmentConfig
          config={environments[selectedEnv] || {}}
          onChange={(config) => setEnvironments({
            ...environments,
            [selectedEnv]: config
          })}
          environment={selectedEnv}
          onCopyFrom={copyFromEnvironment}
          availableEnvironments={Object.keys(environments).filter(e => e !== selectedEnv)}
        />

        <Button onClick={handleSave} buttonType="primary">
          Save Configuration
        </Button>
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Validate Input**: Always validate configuration before saving
2. **Test Connections**: Provide test buttons for external services
3. **Show Changes**: Indicate when configuration has unsaved changes
4. **Secure Secrets**: Use password fields for sensitive data
5. **Provide Defaults**: Set sensible defaults for optional settings
6. **Clear Documentation**: Add hints and descriptions for each setting
7. **Environment Awareness**: Consider per-environment configurations

## Related Hooks

- **onBoot**: Uses configuration to initialize services
- **renderManualFieldExtensionConfigScreen**: Similar pattern for field-specific config
- **renderPage**: Can create additional configuration pages

## Important Notes

- Configuration is stored globally for the plugin
- Changes affect all users of the plugin
- Sensitive data should be properly secured
- The configuration screen is only accessible to users with plugin management permissions
- Always provide a way to test/validate configuration
- Consider backwards compatibility when changing configuration structure