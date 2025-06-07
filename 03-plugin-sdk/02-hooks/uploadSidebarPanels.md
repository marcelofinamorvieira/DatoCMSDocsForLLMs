# uploadSidebarPanels Hook

## Purpose

The `uploadSidebarPanels` hook defines which sidebar panels your plugin provides for the upload/media view. This allows you to register multiple panels that users can enable in their upload sidebar, providing modular functionality for media management.

## Signature

```typescript
uploadSidebarPanels(
  ctx: UploadSidebarPanelsCtx
): UploadSidebarPanel[]
```

## Parameters

- `ctx`: UploadSidebarPanelsCtx - Context object containing:
  - Base properties and methods from plugin SDK

## Type Definitions

```typescript
// Main context type
interface UploadSidebarPanelsCtx extends BaseCtx {}

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

Returns an array of `UploadSidebarPanel` objects:

```typescript
interface UploadSidebarPanel {
  id: string;
  label: string;
  startOpen?: boolean;
  icon?: ReactElement | ReactNode;
  description?: string;
}
```

## Use Cases

### 1. Image Optimization Panels

Register panels for image optimization features:

```typescript
import { FaImage, FaPalette, FaCompress, FaMagic } from 'react-icons/fa';

uploadSidebarPanels(ctx: UploadSidebarPanelsCtx) {
  return [
    {
      id: 'imageOptimization',
      label: 'Image Optimization',
      icon: <FaCompress />,
      startOpen: true,
      description: 'Optimize images for web delivery with format conversion and compression',
    },
    {
      id: 'colorPalette',
      label: 'Color Palette',
      icon: <FaPalette />,
      startOpen: false,
      description: 'Extract dominant colors from images for design consistency',
    },
    {
      id: 'smartCrop',
      label: 'Smart Crop',
      icon: <FaMagic />,
      startOpen: false,
      description: 'AI-powered automatic cropping for different aspect ratios',
    },
  ];
}
```

### 2. Metadata Management Panels

Provide comprehensive metadata tools:

```typescript
import { FaTags, FaMapMarkerAlt, FaCamera, FaShieldAlt } from 'react-icons/fa';

uploadSidebarPanels(ctx: UploadSidebarPanelsCtx) {
  const panels = [
    {
      id: 'basicMetadata',
      label: 'Basic Information',
      icon: <FaTags />,
      startOpen: true,
      description: 'Title, description, and tags',
    },
    {
      id: 'locationData',
      label: 'Location Data',
      icon: <FaMapMarkerAlt />,
      startOpen: false,
      description: 'GPS coordinates and location information',
    },
    {
      id: 'cameraExif',
      label: 'Camera Information',
      icon: <FaCamera />,
      startOpen: false,
      description: 'EXIF data and camera settings',
    },
    {
      id: 'usageRights',
      label: 'Usage Rights',
      icon: <FaShieldAlt />,
      startOpen: false,
      description: 'Copyright, licensing, and usage restrictions',
    },
  ];

  // Only show EXIF panel for photographers
  if (ctx.currentUser.customData?.role === 'photographer') {
    panels.find(p => p.id === 'cameraExif').startOpen = true;
  }

  return panels;
}
```

### 3. AI-Powered Panels

Register AI analysis and enhancement panels:

```typescript
import { FaBrain, FaEye, FaFont, FaRobot } from 'react-icons/fa';

uploadSidebarPanels(ctx: UploadSidebarPanelsCtx) {
  // Only show if AI features are enabled
  if (!ctx.plugin.attributes.parameters.aiEnabled) {
    return [];
  }

  return [
    {
      id: 'imageAnalysis',
      label: 'AI Image Analysis',
      icon: <FaBrain />,
      startOpen: false,
      description: 'Detect objects, faces, and text in images',
    },
    {
      id: 'autoAltText',
      label: 'Auto Alt Text',
      icon: <FaFont />,
      startOpen: false,
      description: 'Generate accessible alt text using AI',
    },
    {
      id: 'similarImages',
      label: 'Similar Images',
      icon: <FaEye />,
      startOpen: false,
      description: 'Find visually similar images in your media library',
    },
    {
      id: 'aiEnhancement',
      label: 'AI Enhancement',
      icon: <FaRobot />,
      startOpen: false,
      description: 'Upscale, denoise, and enhance image quality',
    },
  ];
}
```

### 4. Workflow Integration Panels

Add workflow and collaboration features:

```typescript
import { FaUsers, FaComments, FaHistory, FaTasks } from 'react-icons/fa';

uploadSidebarPanels(ctx: UploadSidebarPanelsCtx) {
  const panels = [];

  // Approval workflow panel
  if (ctx.site.attributes.hasApprovalWorkflow) {
    panels.push({
      id: 'approvalStatus',
      label: 'Approval Status',
      icon: <FaTasks />,
      startOpen: true,
      description: 'Track approval status and workflow state',
    });
  }

  // Collaboration panels
  panels.push(
    {
      id: 'assetComments',
      label: 'Comments',
      icon: <FaComments />,
      startOpen: false,
      description: 'Discussion and feedback on this asset',
    },
    {
      id: 'assetHistory',
      label: 'Version History',
      icon: <FaHistory />,
      startOpen: false,
      description: 'Track changes and previous versions',
    },
    {
      id: 'assetUsage',
      label: 'Usage Tracking',
      icon: <FaUsers />,
      startOpen: false,
      description: 'See where this asset is being used',
    }
  );

  return panels;
}
```

### 5. Integration-Specific Panels

Provide panels for external service integrations:

```typescript
import { FaCloudUploadAlt, FaShareAlt, FaPrint, FaFileExport } from 'react-icons/fa';

uploadSidebarPanels(ctx: UploadSidebarPanelsCtx) {
  const panels = [];
  const integrations = ctx.plugin.attributes.parameters.integrations || {};

  // CDN management
  if (integrations.cloudinary) {
    panels.push({
      id: 'cloudinarySync',
      label: 'Cloudinary Sync',
      icon: <FaCloudUploadAlt />,
      startOpen: false,
      description: 'Sync and transform with Cloudinary',
    });
  }

  // Social media
  if (integrations.socialMedia) {
    panels.push({
      id: 'socialMediaOptimizer',
      label: 'Social Media Optimizer',
      icon: <FaShareAlt />,
      startOpen: false,
      description: 'Optimize images for social platforms',
    });
  }

  // Print services
  if (integrations.printful) {
    panels.push({
      id: 'printPrep',
      label: 'Print Preparation',
      icon: <FaPrint />,
      startOpen: false,
      description: 'Prepare images for print production',
    });
  }

  // DAM integration
  if (integrations.dam) {
    panels.push({
      id: 'damSync',
      label: 'DAM Synchronization',
      icon: <FaFileExport />,
      startOpen: false,
      description: 'Sync with external Digital Asset Management',
    });
  }

  return panels;
}
```

### 6. Performance and Analytics Panels

Add performance monitoring and analytics:

```typescript
import { FaTachometerAlt, FaChartLine, FaGlobe, FaFileAlt } from 'react-icons/fa';

uploadSidebarPanels(ctx: UploadSidebarPanelsCtx) {
  return [
    {
      id: 'performanceMetrics',
      label: 'Performance Metrics',
      icon: <FaTachometerAlt />,
      startOpen: false,
      description: 'Loading times and optimization suggestions',
    },
    {
      id: 'usageAnalytics',
      label: 'Usage Analytics',
      icon: <FaChartLine />,
      startOpen: false,
      description: 'Views, downloads, and engagement metrics',
    },
    {
      id: 'cdnStatus',
      label: 'CDN Status',
      icon: <FaGlobe />,
      startOpen: false,
      description: 'CDN distribution and cache status',
    },
    {
      id: 'seoOptimization',
      label: 'SEO Optimization',
      icon: <FaFileAlt />,
      startOpen: false,
      description: 'Image SEO analysis and recommendations',
    },
  ];
}
```

## Best Practices

1. **Clear Naming**: Use descriptive labels that explain the panel's purpose
2. **Helpful Descriptions**: Provide descriptions to help users understand each panel
3. **Meaningful Icons**: Choose icons that visually represent the panel's function
4. **Smart Defaults**: Set `startOpen` based on common usage patterns
5. **Conditional Panels**: Only register panels that are relevant to the user/project
6. **Unique IDs**: Ensure each panel ID is unique within your plugin
7. **Logical Order**: Return panels in a logical order for the sidebar
8. **Performance**: Don't register too many panels that might slow down the UI
9. **User Preferences**: Consider user roles and preferences when setting defaults
10. **Feature Flags**: Use configuration to enable/disable panels

## Related Hooks

- `renderUploadSidebarPanel` - Render the actual panel content
- `uploadSidebars` - Define complete custom sidebars
- `renderUploadSidebar` - Render custom sidebar implementations

## Important Notes

- Panel IDs must be unique within your plugin
- Each panel ID must have a corresponding `renderUploadSidebarPanel` implementation
- Users can enable/disable panels in their sidebar settings
- The `startOpen` property is only a default; users can change it
- Panels appear in the order they are returned
- Consider performance impact of having many panels open by default