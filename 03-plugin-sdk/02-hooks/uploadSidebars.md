# uploadSidebars Hook

## Purpose

The `uploadSidebars` hook defines complete custom sidebars for the upload/media view. Unlike panels which integrate into the existing sidebar, this hook allows you to create entirely custom sidebar experiences for media management.

## Signature

```typescript
uploadSidebars(
  ctx: UploadSidebarsCtx
): UploadSidebar[]
```

## Parameters

- `ctx`: UploadSidebarsCtx - Context object containing:
  - Base properties and methods from plugin SDK

## Type Definitions

```typescript
// Main context type
interface UploadSidebarsCtx extends BaseCtx {
  environment: string;
}

// Base context properties and methods
interface BaseCtx {
  // Properties
  plugin: Plugin;
  currentUser: User | SsoUser | Account | Organization;
  currentRole: Role;
  site: Site;
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
    customData: Record<string, any> | null;
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
    canEditImages: boolean;
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
    hasApprovalWorkflow: boolean;
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
  uploads: {
    find(uploadId: string): Promise<Upload>;
    list(params?: any): Promise<Upload[]>;
    create(data: any): Promise<Upload>;
    update(uploadId: string, data: any): Promise<Upload>;
    destroy(uploadId: string): Promise<void>;
  };
}

// Upload structure
interface Upload {
  id: string;
  url: string;
  filename: string;
  size: number;
  width?: number;
  height?: number;
  format?: string;
  mimeType?: string;
  tags?: string[];
  smartTags?: string[];
  customData?: Record<string, any>;
  author?: string;
  copyright?: string;
  notes?: string;
  alt?: string | null;
  title?: string | null;
}
```

## Return Value

Returns an array of `UploadSidebar` objects:

```typescript
interface UploadSidebar {
  id: string;
  label: string;
  icon?: ReactElement | ReactNode;
  description?: string;
}
```

## Use Cases

### 1. Professional Photo Editor Sidebar

A comprehensive sidebar for professional photo editing:

```typescript
import { FaCamera, FaMagic, FaImage } from 'react-icons/fa';

uploadSidebars(ctx: UploadSidebarsCtx) {
  // Only show for users with photo editing permissions
  if (!ctx.currentRole.attributes.canEditImages) {
    return [];
  }

  return [
    {
      id: 'professionalPhotoEditor',
      label: 'Professional Photo Editor',
      icon: <FaCamera />,
      description: 'Advanced photo editing tools with non-destructive adjustments, filters, and effects',
    },
  ];
}
```

### 2. Media Analytics Dashboard

A sidebar focused on media performance and analytics:

```typescript
import { FaChartBar, FaTachometerAlt } from 'react-icons/fa';

uploadSidebars(ctx: UploadSidebarsCtx) {
  return [
    {
      id: 'mediaAnalyticsDashboard',
      label: 'Media Analytics',
      icon: <FaChartBar />,
      description: 'Comprehensive analytics for media performance, usage patterns, and optimization insights',
    },
  ];
}
```

### 3. AI-Powered Media Assistant

An intelligent sidebar with AI capabilities:

```typescript
import { FaBrain, FaRobot } from 'react-icons/fa';

uploadSidebars(ctx: UploadSidebarsCtx) {
  // Only show if AI features are enabled
  if (!ctx.plugin.attributes.parameters.aiEnabled) {
    return [];
  }

  return [
    {
      id: 'aiMediaAssistant',
      label: 'AI Media Assistant',
      icon: <FaBrain />,
      description: 'AI-powered tools for automatic tagging, content analysis, and intelligent enhancements',
    },
  ];
}
```

### 4. Brand Asset Manager

A specialized sidebar for brand management:

```typescript
import { FaPalette, FaBuilding } from 'react-icons/fa';

uploadSidebars(ctx: UploadSidebarsCtx) {
  // Only for specific organizations
  if (ctx.currentUser.type !== 'Organization') {
    return [];
  }

  return [
    {
      id: 'brandAssetManager',
      label: 'Brand Asset Manager',
      icon: <FaPalette />,
      description: 'Manage brand guidelines, ensure consistency, and control asset usage across teams',
    },
  ];
}
```

### 5. Multi-Tool Media Workspace

A comprehensive workspace with multiple tools:

```typescript
import { FaTools, FaCogs } from 'react-icons/fa';

uploadSidebars(ctx: UploadSidebarsCtx) {
  const sidebars = [];

  // Development environment tools
  if (ctx.environment === 'development') {
    sidebars.push({
      id: 'mediaDevTools',
      label: 'Media Dev Tools',
      icon: <FaCogs />,
      description: 'Development tools for testing transformations, debugging, and API exploration',
    });
  }

  // Production media workspace
  sidebars.push({
    id: 'mediaWorkspace',
    label: 'Media Workspace',
    icon: <FaTools />,
    description: 'Complete media management workspace with editing, organizing, and publishing tools',
  });

  return sidebars;
}
```

### 6. Integration Hub Sidebar

A sidebar for managing external integrations:

```typescript
import { FaPlug, FaCloudUploadAlt } from 'react-icons/fa';

uploadSidebars(ctx: UploadSidebarsCtx) {
  const integrations = ctx.plugin.attributes.parameters.integrations || {};
  
  // Only show if integrations are configured
  const hasIntegrations = Object.values(integrations).some(enabled => enabled);
  
  if (!hasIntegrations) {
    return [];
  }

  return [
    {
      id: 'integrationHub',
      label: 'Integration Hub',
      icon: <FaPlug />,
      description: 'Manage connections with external services, CDNs, and third-party tools',
    },
  ];
}
```

## Implementation Example

Here's how these sidebars would be implemented with `renderUploadSidebar`:

```typescript
// Professional Photo Editor Implementation
async renderUploadSidebar(uploadSidebarId: string, ctx: RenderUploadSidebarCtx) {
  if (uploadSidebarId === 'professionalPhotoEditor') {
    return {
      render: () => import('./components/ProfessionalPhotoEditor'),
    };
  }
}

// ProfessionalPhotoEditor.tsx
import { RenderUploadSidebarCtx } from 'datocms-plugin-sdk';
import { Canvas, Section } from 'datocms-react-ui';

export default function ProfessionalPhotoEditor({ ctx }: { ctx: RenderUploadSidebarCtx }) {
  return (
    <Canvas ctx={ctx}>
      <div style={{ height: '100vh', display: 'flex', flexDirection: 'column' }}>
        {/* Toolbar */}
        <div style={{ padding: '10px', borderBottom: '1px solid #e0e0e0' }}>
          {/* Tool buttons */}
        </div>
        
        {/* Main editing area */}
        <div style={{ flex: 1, overflow: 'auto' }}>
          <Section title="Adjustments">
            {/* Adjustment controls */}
          </Section>
          
          <Section title="Filters">
            {/* Filter options */}
          </Section>
          
          <Section title="Effects">
            {/* Effects controls */}
          </Section>
        </div>
        
        {/* Action buttons */}
        <div style={{ padding: '20px', borderTop: '1px solid #e0e0e0' }}>
          {/* Save/Cancel buttons */}
        </div>
      </div>
    </Canvas>
  );
}
```

## Best Practices

1. **Clear Purpose**: Each sidebar should have a distinct, clear purpose
2. **Descriptive Labels**: Use labels that immediately convey the sidebar's function
3. **Helpful Descriptions**: Provide descriptions to help users choose the right sidebar
4. **Conditional Display**: Only show sidebars relevant to the user's role or context
5. **Performance**: Consider the performance impact of complex sidebars
6. **Icon Choice**: Use icons that visually represent the sidebar's purpose
7. **Feature Detection**: Check for required features/permissions before showing
8. **Environment Awareness**: Adjust available sidebars based on environment
9. **User Experience**: Design sidebars that enhance, not complicate, workflows
10. **Unique IDs**: Ensure sidebar IDs are unique across your plugin

## Related Hooks

- `renderUploadSidebar` - Implement the actual sidebar rendering
- `uploadSidebarPanels` - Define panels for the default sidebar
- `renderUploadSidebarPanel` - Render individual sidebar panels

## Important Notes

- Sidebar IDs must be unique within your plugin
- Each sidebar must have a corresponding `renderUploadSidebar` implementation
- Custom sidebars completely replace the default sidebar when active
- Users can switch between available sidebars using a selector
- Consider mobile responsiveness when designing custom sidebars
- Sidebars should provide comprehensive functionality for their stated purpose