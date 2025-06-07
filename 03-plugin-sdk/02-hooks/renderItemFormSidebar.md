# renderItemFormSidebar Hook

## Purpose

The `renderItemFormSidebar` hook renders custom sidebar components in item forms. Unlike sidebar panels which are collapsible sections, this hook allows you to create entire custom sidebars that can provide comprehensive tools, workflows, or information displays alongside the content editing experience.

## Signature

```typescript
renderItemFormSidebar(
  itemFormSidebarId: string,
  ctx: RenderItemFormSidebarCtx
): void | { render: () => Promise<{ default: React.ComponentType<any> }> }
```

## Parameters

- `itemFormSidebarId: string` - The ID of the sidebar being rendered
- `ctx: RenderItemFormSidebarCtx` - The context object containing all item form data and methods

## Type Definitions

```typescript
// Main hook type
export type RenderItemFormSidebarHook = {
  renderItemFormSidebar: (
    itemFormSidebarId: string,
    ctx: RenderItemFormSidebarCtx
  ) => void | { render: () => Promise<{ default: React.ComponentType<any> }> };
};

// Context type extends base context with item form properties and methods
type RenderItemFormSidebarCtx = Ctx<ItemFormAdditionalProperties & ItemFormAdditionalMethods & {
  itemFormSidebarId: string;
  parameters?: Record<string, unknown>;
}>;

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

### 1. Workflow Management Sidebar

Create a comprehensive workflow management system:

```typescript
async renderItemFormSidebar(
  itemFormSidebarId: string,
  ctx: RenderItemFormSidebarCtx
) {
  if (itemFormSidebarId === 'workflowSidebar') {
    return {
      render: () => import('./components/WorkflowSidebar'),
    };
  }
}

// WorkflowSidebar.tsx
import { RenderItemFormSidebarCtx } from 'datocms-plugin-sdk';
import { Canvas, Section, Button, SelectField, TextareaField } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { 
  FaCheckCircle, 
  FaHourglassHalf, 
  FaEdit, 
  FaUserCheck,
  FaCalendarAlt,
  FaHistory 
} from 'react-icons/fa';

interface WorkflowStep {
  id: string;
  name: string;
  status: 'completed' | 'current' | 'pending';
  assignee?: string;
  completedAt?: string;
  comments?: string;
}

export default function WorkflowSidebar({ ctx }: { ctx: RenderItemFormSidebarCtx }) {
  const [workflowSteps, setWorkflowSteps] = useState<WorkflowStep[]>([
    { id: 'draft', name: 'Draft Creation', status: 'completed' },
    { id: 'review', name: 'Content Review', status: 'current' },
    { id: 'approval', name: 'Editorial Approval', status: 'pending' },
    { id: 'publish', name: 'Publishing', status: 'pending' },
  ]);
  
  const [assignee, setAssignee] = useState('');
  const [dueDate, setDueDate] = useState('');
  const [priority, setPriority] = useState('medium');
  const [notes, setNotes] = useState('');
  const [history, setHistory] = useState<any[]>([]);

  useEffect(() => {
    loadWorkflowData();
  }, [ctx.itemId]);

  const loadWorkflowData = async () => {
    if (!ctx.itemId) return;

    // Load workflow metadata from item meta or custom field
    const workflowData = ctx.formValues._workflow || {};
    
    if (workflowData.steps) {
      setWorkflowSteps(workflowData.steps);
    }
    if (workflowData.assignee) {
      setAssignee(workflowData.assignee);
    }
    if (workflowData.dueDate) {
      setDueDate(workflowData.dueDate);
    }
    if (workflowData.priority) {
      setPriority(workflowData.priority);
    }
    if (workflowData.notes) {
      setNotes(workflowData.notes);
    }

    // Load history
    try {
      const client = ctx.createClient();
      const versions = await client.itemVersions.list(ctx.itemId);
      setHistory(versions.slice(0, 5)); // Last 5 versions
    } catch (error) {
      console.error('Failed to load history:', error);
    }
  };

  const saveWorkflowData = async () => {
    const workflowData = {
      steps: workflowSteps,
      assignee,
      dueDate,
      priority,
      notes,
      lastUpdated: new Date().toISOString(),
    };

    ctx.setFieldValue('_workflow', workflowData);
    await ctx.saveCurrentItem();
    ctx.notice('Workflow updated');
  };

  const moveToNextStep = async () => {
    const currentIndex = workflowSteps.findIndex(step => step.status === 'current');
    if (currentIndex === -1 || currentIndex === workflowSteps.length - 1) return;

    const updatedSteps = workflowSteps.map((step, index) => {
      if (index === currentIndex) {
        return { ...step, status: 'completed' as const, completedAt: new Date().toISOString() };
      } else if (index === currentIndex + 1) {
        return { ...step, status: 'current' as const };
      }
      return step;
    });

    setWorkflowSteps(updatedSteps);
    
    // Auto-save workflow progress
    const workflowData = {
      steps: updatedSteps,
      assignee,
      dueDate,
      priority,
      notes,
      lastUpdated: new Date().toISOString(),
    };
    
    ctx.setFieldValue('_workflow', workflowData);
    await ctx.saveCurrentItem();
    
    ctx.notice(`Moved to ${updatedSteps[currentIndex + 1].name}`);

    // Send notification if configured
    if (assignee && ctx.plugin.attributes.parameters.notificationWebhook) {
      fetch(ctx.plugin.attributes.parameters.notificationWebhook, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          itemId: ctx.itemId,
          itemTitle: ctx.formValues.title || ctx.formValues.name,
          step: updatedSteps[currentIndex + 1].name,
          assignee,
          dueDate,
        }),
      });
    }
  };

  const getStepIcon = (status: string) => {
    switch (status) {
      case 'completed':
        return <FaCheckCircle color="#4CAF50" />;
      case 'current':
        return <FaHourglassHalf color="#2196F3" />;
      default:
        return <FaEdit color="#9E9E9E" />;
    }
  };

  const getPriorityColor = () => {
    switch (priority) {
      case 'high':
        return '#F44336';
      case 'medium':
        return '#FF9800';
      case 'low':
        return '#4CAF50';
      default:
        return '#9E9E9E';
    }
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ height: '100vh', overflowY: 'auto', padding: '20px' }}>
        {/* Workflow Status */}
        <Section title="Workflow Status">
          <div style={{ padding: '15px' }}>
            {workflowSteps.map((step, index) => (
              <div
                key={step.id}
                style={{
                  display: 'flex',
                  alignItems: 'center',
                  marginBottom: '15px',
                  opacity: step.status === 'pending' ? 0.6 : 1,
                }}
              >
                <div style={{ marginRight: '10px' }}>
                  {getStepIcon(step.status)}
                </div>
                <div style={{ flex: 1 }}>
                  <div style={{ fontWeight: step.status === 'current' ? 'bold' : 'normal' }}>
                    {step.name}
                  </div>
                  {step.completedAt && (
                    <div style={{ fontSize: '12px', color: '#666' }}>
                      Completed: {new Date(step.completedAt).toLocaleDateString()}
                    </div>
                  )}
                </div>
              </div>
            ))}
            
            <Button
              onClick={moveToNextStep}
              buttonType="primary"
              fullWidth
              disabled={workflowSteps.every(s => s.status === 'completed')}
            >
              Move to Next Step
            </Button>
          </div>
        </Section>

        {/* Assignment & Scheduling */}
        <Section title="Assignment & Scheduling" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <SelectField
              id="assignee"
              name="assignee"
              label="Assigned To"
              value={assignee}
              selectInputProps={{
                options: [
                  { label: 'Unassigned', value: '' },
                  { label: 'John Editor', value: 'john@example.com' },
                  { label: 'Jane Reviewer', value: 'jane@example.com' },
                  { label: 'Mike Publisher', value: 'mike@example.com' },
                ],
                onChange: setAssignee,
              }}
            />
            
            <div style={{ marginTop: '15px' }}>
              <label style={{ display: 'block', marginBottom: '5px', fontSize: '14px' }}>
                Due Date
              </label>
              <input
                type="date"
                value={dueDate}
                onChange={(e) => setDueDate(e.target.value)}
                style={{
                  width: '100%',
                  padding: '8px',
                  border: '1px solid #ddd',
                  borderRadius: '4px',
                }}
              />
            </div>
            
            <SelectField
              id="priority"
              name="priority"
              label="Priority"
              value={priority}
              selectInputProps={{
                options: [
                  { label: 'High', value: 'high' },
                  { label: 'Medium', value: 'medium' },
                  { label: 'Low', value: 'low' },
                ],
                onChange: setPriority,
              }}
            />
            
            <div style={{
              marginTop: '10px',
              padding: '10px',
              backgroundColor: getPriorityColor(),
              color: 'white',
              borderRadius: '4px',
              textAlign: 'center',
              fontSize: '14px',
            }}>
              {priority.toUpperCase()} PRIORITY
            </div>
          </div>
        </Section>

        {/* Workflow Notes */}
        <Section title="Workflow Notes" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <TextareaField
              id="notes"
              name="notes"
              label=""
              value={notes}
              onChange={setNotes}
              hint="Add notes about this workflow"
              textareaInputProps={{ rows: 4 }}
            />
            
            <Button
              onClick={saveWorkflowData}
              buttonType="primary"
              fullWidth
              style={{ marginTop: '10px' }}
            >
              Save Workflow Data
            </Button>
          </div>
        </Section>

        {/* Version History */}
        <Section title="Version History" startOpen={false}>
          <div style={{ padding: '15px' }}>
            {history.length > 0 ? (
              <div>
                {history.map((version, index) => (
                  <div
                    key={version.id}
                    style={{
                      padding: '10px',
                      marginBottom: '10px',
                      backgroundColor: '#f5f5f5',
                      borderRadius: '4px',
                      fontSize: '13px',
                    }}
                  >
                    <div style={{ fontWeight: 'bold' }}>
                      Version {history.length - index}
                    </div>
                    <div style={{ color: '#666' }}>
                      {new Date(version.createdAt).toLocaleString()}
                    </div>
                    <div>
                      By: {version.createdBy}
                    </div>
                  </div>
                ))}
              </div>
            ) : (
              <p style={{ color: '#666', textAlign: 'center' }}>
                No version history available
              </p>
            )}
          </div>
        </Section>

        {/* Quick Actions */}
        <Section title="Quick Actions" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <Button
              onClick={() => {
                ctx.openModal({
                  id: 'preview',
                  title: 'Preview',
                  width: 'fullWidth',
                  parameters: { itemId: ctx.itemId },
                });
              }}
              fullWidth
              style={{ marginBottom: '10px' }}
            >
              Preview Content
            </Button>
            
            <Button
              onClick={async () => {
                const confirmed = await ctx.openConfirm({
                  title: 'Request Review',
                  content: 'Send this content for review?',
                  cancel: { label: 'Cancel', value: false },
                  choices: [{ label: 'Send', value: true, intent: 'primary' }],
                });
                
                if (confirmed) {
                  // Send review request
                  ctx.notice('Review request sent');
                }
              }}
              fullWidth
              style={{ marginBottom: '10px' }}
            >
              Request Review
            </Button>
            
            <Button
              onClick={() => {
                const scheduledDate = window.prompt('Schedule publishing for (YYYY-MM-DD HH:MM):');
                if (scheduledDate) {
                  ctx.setFieldValue('_scheduled_publishing', scheduledDate);
                  ctx.notice(`Scheduled for ${scheduledDate}`);
                }
              }}
              fullWidth
            >
              Schedule Publishing
            </Button>
          </div>
        </Section>
      </div>
    </Canvas>
  );
}
```

### 2. Translation Management Sidebar

Manage translations across multiple locales:

```typescript
async renderItemFormSidebar(
  itemFormSidebarId: string,
  ctx: RenderItemFormSidebarCtx
) {
  if (itemFormSidebarId === 'translationSidebar') {
    return {
      render: () => import('./components/TranslationSidebar'),
    };
  }
}

// TranslationSidebar.tsx
import { RenderItemFormSidebarCtx } from 'datocms-plugin-sdk';
import { Canvas, Section, Button, ProgressBar } from 'datocms-react-ui';
import { useState, useEffect } from 'react';
import { FaGlobe, FaLanguage, FaCheck, FaTimes, FaCopy } from 'react-icons/fa';

interface TranslationStatus {
  locale: string;
  localeName: string;
  progress: number;
  missingFields: string[];
  lastUpdated?: string;
}

export default function TranslationSidebar({ ctx }: { ctx: RenderItemFormSidebarCtx }) {
  const [translationStatuses, setTranslationStatuses] = useState<TranslationStatus[]>([]);
  const [selectedSourceLocale, setSelectedSourceLocale] = useState(ctx.locale);
  const [isTranslating, setIsTranslating] = useState(false);

  useEffect(() => {
    analyzeTranslations();
  }, [ctx.formValues, ctx.locale]);

  const analyzeTranslations = () => {
    const statuses: TranslationStatus[] = [];

    ctx.locales.forEach(locale => {
      const localeInfo = ctx.internalLocales.find(l => l.id === locale);
      const localeName = localeInfo?.name || locale;
      
      let translatedFields = 0;
      let totalFields = 0;
      const missingFields: string[] = [];

      Object.values(ctx.itemType.attributes.fields).forEach(field => {
        if (field.localized) {
          totalFields++;
          
          const value = ctx.formValues[field.apiKey];
          if (value && value[locale]) {
            translatedFields++;
          } else {
            missingFields.push(field.label);
          }
        }
      });

      const progress = totalFields > 0 ? Math.round((translatedFields / totalFields) * 100) : 100;
      
      statuses.push({
        locale,
        localeName,
        progress,
        missingFields,
        lastUpdated: ctx.item?.meta.updatedAt,
      });
    });

    setTranslationStatuses(statuses);
  };

  const machineTranslate = async (targetLocale: string) => {
    if (!ctx.plugin.attributes.parameters.translationApiKey) {
      ctx.alert('Translation API not configured');
      return;
    }

    setIsTranslating(true);
    const updates: Record<string, any> = {};

    try {
      for (const field of Object.values(ctx.itemType.attributes.fields)) {
        if (field.localized && (field.fieldType === 'string' || field.fieldType === 'text')) {
          const sourceValue = ctx.formValues[field.apiKey]?.[selectedSourceLocale];
          
          if (sourceValue && !ctx.formValues[field.apiKey]?.[targetLocale]) {
            // Call translation API
            const translated = await translateText(
              sourceValue,
              selectedSourceLocale,
              targetLocale
            );
            
            if (!updates[field.apiKey]) {
              updates[field.apiKey] = ctx.formValues[field.apiKey] || {};
            }
            updates[field.apiKey][targetLocale] = translated;
          }
        }
      }

      ctx.setMultipleFieldValues(updates);
      ctx.notice(`Machine translation completed for ${targetLocale}`);
      analyzeTranslations();
    } catch (error) {
      ctx.alert('Translation failed: ' + error.message);
    } finally {
      setIsTranslating(false);
    }
  };

  const translateText = async (
    text: string,
    sourceLocale: string,
    targetLocale: string
  ): Promise<string> => {
    // Example using Google Translate API
    const apiKey = ctx.plugin.attributes.parameters.translationApiKey;
    const response = await fetch(
      `https://translation.googleapis.com/language/translate/v2?key=${apiKey}`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          q: text,
          source: sourceLocale.split('-')[0],
          target: targetLocale.split('-')[0],
          format: 'text',
        }),
      }
    );

    const data = await response.json();
    return data.data.translations[0].translatedText;
  };

  const copyFromLocale = (sourceLocale: string, targetLocale: string) => {
    const updates: Record<string, any> = {};

    Object.values(ctx.itemType.attributes.fields).forEach(field => {
      if (field.localized) {
        const sourceValue = ctx.formValues[field.apiKey]?.[sourceLocale];
        
        if (sourceValue) {
          if (!updates[field.apiKey]) {
            updates[field.apiKey] = ctx.formValues[field.apiKey] || {};
          }
          updates[field.apiKey][targetLocale] = sourceValue;
        }
      }
    });

    ctx.setMultipleFieldValues(updates);
    ctx.notice(`Copied content from ${sourceLocale} to ${targetLocale}`);
    analyzeTranslations();
  };

  const switchToLocale = (locale: string) => {
    // This would need to be implemented based on how DatoCMS handles locale switching
    ctx.navigateTo(`/editor/item-types/${ctx.itemType.id}/items/${ctx.itemId}?locale=${locale}`);
  };

  const getProgressColor = (progress: number) => {
    if (progress === 100) return '#4CAF50';
    if (progress >= 75) return '#8BC34A';
    if (progress >= 50) return '#FFC107';
    if (progress >= 25) return '#FF9800';
    return '#F44336';
  };

  return (
    <Canvas ctx={ctx}>
      <div style={{ height: '100vh', overflowY: 'auto', padding: '20px' }}>
        {/* Translation Overview */}
        <Section title="Translation Status">
          <div style={{ padding: '15px' }}>
            {translationStatuses.map(status => (
              <div
                key={status.locale}
                style={{
                  marginBottom: '20px',
                  padding: '15px',
                  backgroundColor: status.locale === ctx.locale ? '#e3f2fd' : '#f5f5f5',
                  borderRadius: '8px',
                  border: status.locale === ctx.locale ? '2px solid #2196F3' : 'none',
                }}
              >
                <div style={{ 
                  display: 'flex', 
                  justifyContent: 'space-between',
                  alignItems: 'center',
                  marginBottom: '10px'
                }}>
                  <div style={{ display: 'flex', alignItems: 'center' }}>
                    <FaGlobe style={{ marginRight: '8px' }} />
                    <strong>{status.localeName}</strong>
                    {status.locale === ctx.locale && (
                      <span style={{ 
                        marginLeft: '8px',
                        fontSize: '12px',
                        color: '#2196F3'
                      }}>
                        (Current)
                      </span>
                    )}
                  </div>
                  <div style={{ fontSize: '14px', fontWeight: 'bold' }}>
                    {status.progress}%
                  </div>
                </div>

                <div style={{
                  height: '8px',
                  backgroundColor: '#e0e0e0',
                  borderRadius: '4px',
                  overflow: 'hidden',
                  marginBottom: '10px',
                }}>
                  <div style={{
                    height: '100%',
                    width: `${status.progress}%`,
                    backgroundColor: getProgressColor(status.progress),
                    transition: 'width 0.3s ease',
                  }} />
                </div>

                {status.missingFields.length > 0 && (
                  <div style={{ fontSize: '12px', color: '#666', marginBottom: '10px' }}>
                    Missing: {status.missingFields.join(', ')}
                  </div>
                )}

                <div style={{ display: 'flex', gap: '5px' }}>
                  {status.locale !== ctx.locale && (
                    <>
                      <Button
                        onClick={() => switchToLocale(status.locale)}
                        buttonSize="s"
                      >
                        Edit
                      </Button>
                      
                      <Button
                        onClick={() => machineTranslate(status.locale)}
                        buttonSize="s"
                        disabled={isTranslating || status.progress === 100}
                      >
                        Auto-translate
                      </Button>
                      
                      <Button
                        onClick={() => copyFromLocale(ctx.locale, status.locale)}
                        buttonSize="s"
                        leftIcon={<FaCopy />}
                      >
                        Copy
                      </Button>
                    </>
                  )}
                </div>
              </div>
            ))}
          </div>
        </Section>

        {/* Translation Tools */}
        <Section title="Translation Tools" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <div style={{ marginBottom: '15px' }}>
              <label style={{ display: 'block', marginBottom: '5px', fontSize: '14px' }}>
                Source Locale for Translation
              </label>
              <select
                value={selectedSourceLocale}
                onChange={(e) => setSelectedSourceLocale(e.target.value)}
                style={{
                  width: '100%',
                  padding: '8px',
                  border: '1px solid #ddd',
                  borderRadius: '4px',
                }}
              >
                {ctx.locales.map(locale => {
                  const localeInfo = ctx.internalLocales.find(l => l.id === locale);
                  return (
                    <option key={locale} value={locale}>
                      {localeInfo?.name || locale}
                    </option>
                  );
                })}
              </select>
            </div>

            <Button
              onClick={async () => {
                const confirmed = await ctx.openConfirm({
                  title: 'Translate All',
                  content: 'Automatically translate all missing content?',
                  cancel: { label: 'Cancel', value: false },
                  choices: [{ label: 'Translate', value: true, intent: 'primary' }],
                });
                
                if (confirmed) {
                  for (const status of translationStatuses) {
                    if (status.locale !== selectedSourceLocale && status.progress < 100) {
                      await machineTranslate(status.locale);
                    }
                  }
                }
              }}
              buttonType="primary"
              fullWidth
              disabled={isTranslating}
            >
              {isTranslating ? 'Translating...' : 'Translate All Missing'}
            </Button>
          </div>
        </Section>

        {/* Translation Memory */}
        <Section title="Translation Memory" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <p style={{ fontSize: '14px', marginBottom: '15px' }}>
              Similar translations from other content:
            </p>
            
            {/* This would show similar translations from other items */}
            <div style={{
              padding: '10px',
              backgroundColor: '#f5f5f5',
              borderRadius: '4px',
              fontSize: '13px',
            }}>
              <div style={{ marginBottom: '10px' }}>
                <strong>Original:</strong> "Welcome to our website"
                <br />
                <strong>Spanish:</strong> "Bienvenido a nuestro sitio web"
                <br />
                <span style={{ color: '#666' }}>Used in 5 other items</span>
              </div>
              
              <Button
                onClick={() => {
                  ctx.setFieldValue('title', {
                    ...ctx.formValues.title,
                    es: 'Bienvenido a nuestro sitio web',
                  });
                }}
                buttonSize="s"
              >
                Apply
              </Button>
            </div>
          </div>
        </Section>

        {/* Export/Import */}
        <Section title="Export/Import" startOpen={false}>
          <div style={{ padding: '15px' }}>
            <Button
              onClick={() => {
                // Export translations for external translation
                const exportData = {
                  itemId: ctx.itemId,
                  itemType: ctx.itemType.attributes.name,
                  translations: ctx.formValues,
                };
                
                const blob = new Blob([JSON.stringify(exportData, null, 2)], {
                  type: 'application/json',
                });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url;
                a.download = `translations-${ctx.itemId}.json`;
                a.click();
                
                ctx.notice('Translations exported');
              }}
              fullWidth
              style={{ marginBottom: '10px' }}
            >
              Export for Translation
            </Button>
            
            <Button
              onClick={() => {
                const input = document.createElement('input');
                input.type = 'file';
                input.accept = '.json';
                input.onchange = async (e: any) => {
                  const file = e.target.files[0];
                  if (file) {
                    const text = await file.text();
                    const data = JSON.parse(text);
                    
                    if (data.itemId === ctx.itemId) {
                      ctx.setMultipleFieldValues(data.translations);
                      ctx.notice('Translations imported');
                    } else {
                      ctx.alert('This file is for a different item');
                    }
                  }
                };
                input.click();
              }}
              fullWidth
            >
              Import Translations
            </Button>
          </div>
        </Section>
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Sidebar Design**: Design for vertical layouts with scrollable content
2. **Section Organization**: Use sections to organize related functionality
3. **Performance**: Lazy load heavy components to improve initial render
4. **State Persistence**: Consider storing sidebar state in localStorage
5. **Responsive Height**: Handle different viewport heights appropriately
6. **Visual Hierarchy**: Use sections and proper spacing to create clear organization
7. **Loading States**: Show loading indicators for async operations
8. **Error Handling**: Handle API failures gracefully
9. **Keyboard Navigation**: Ensure all interactive elements are keyboard accessible
10. **Mobile Consideration**: Test sidebar behavior on smaller screens

## Related Hooks

- `itemFormSidebars` - Define which sidebars your plugin provides
- `itemFormSidebarPanels` - Add collapsible panels to the default sidebar
- `renderItemFormSidebarPanel` - Render individual sidebar panels
- `renderItemFormOutlet` - Add outlets to other areas of the form

## Important Notes

- Sidebars replace the entire sidebar area when active
- Users can switch between different sidebars if multiple are available
- The sidebar should provide comprehensive functionality related to its purpose
- Consider the available width when designing sidebar content
- Use the Canvas component for consistent styling
- Sidebars have access to all form data and can modify fields