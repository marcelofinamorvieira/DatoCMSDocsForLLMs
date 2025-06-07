# uploadsDropdownActions Hook

## Purpose

The `uploadsDropdownActions` hook allows you to add custom dropdown actions for media/upload items. This enables bulk operations, custom workflows, and integrations at the media management level.

## Signature

```typescript
uploadsDropdownActions(
  ctx: UploadsDropdownActionsCtx
): DropdownAction[] | undefined
```

## Parameters

- `ctx`: UploadsDropdownActionsCtx - Context object containing:
  - Base properties and methods from plugin SDK
  - `uploads`: Upload[] - The selected upload items
  - `uploadIds`: string[] - IDs of selected uploads

## Type Definitions

```typescript
// Main context type
interface UploadsDropdownActionsCtx extends BaseCtx {
  uploads: Upload[];
  uploadIds: string[];
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
    value: true | false;
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

// Dropdown action handler extension
interface DropdownActionHandler {
  handler: () => void | Promise<void>;
}
```

## Return Value

Returns an array of `DropdownAction` objects:

```typescript
interface DropdownAction {
  id: string;
  label: string;
  icon?: ReactElement | ReactNode;
  disabled?: boolean;
  tooltip?: string;
  rank?: number;
  group?: string;
  closeMenuOnClick?: boolean;
}

// When used in this hook, actions also include a handler
type UploadsDropdownAction = DropdownAction & DropdownActionHandler;
```

## Use Cases

### 1. Bulk Image Optimization

Optimize multiple images at once:

```typescript
import { FaCompress, FaImage } from 'react-icons/fa';

uploadsDropdownActions(ctx: UploadsDropdownActionsCtx) {
  // Only show for image uploads
  const imageUploads = ctx.uploads.filter(u => u.mimeType?.startsWith('image/'));
  
  if (imageUploads.length === 0) {
    return [];
  }

  return [
    {
      id: 'bulk-optimize',
      label: 'Optimize Images',
      icon: <FaCompress />,
      group: 'optimization',
      rank: 10,
      disabled: ctx.uploadIds.length === 0,
      tooltip: 'Compress and optimize selected images',
      handler: async () => {
        const confirmed = await ctx.openConfirm({
          title: 'Optimize Images',
          content: `Optimize ${imageUploads.length} images? This will create optimized versions without affecting originals.`,
          cancel: { label: 'Cancel', value: false },
          choices: [{ label: 'Optimize', value: true, intent: 'primary' }]
        });

        if (!confirmed) return;

        ctx.notice('Optimizing images...');
        const client = ctx.createClient();
        
        try {
          const results = await Promise.allSettled(
            imageUploads.map(async (upload) => {
              // Create optimized versions
              const optimizations = [
                { width: 1920, quality: 85, format: 'webp' },
                { width: 1200, quality: 85, format: 'webp' },
                { width: 600, quality: 80, format: 'webp' },
                { width: 300, quality: 75, format: 'webp' },
              ];

              const variants = [];
              
              for (const opt of optimizations) {
                if (upload.width > opt.width) {
                  // Generate optimized URL (using your CDN's API)
                  const optimizedUrl = `${upload.url}?w=${opt.width}&q=${opt.quality}&fm=${opt.format}`;
                  
                  variants.push({
                    width: opt.width,
                    format: opt.format,
                    quality: opt.quality,
                    url: optimizedUrl,
                    size: Math.round(upload.size * (opt.width / upload.width) * (opt.quality / 100) * 0.7), // Estimate
                  });
                }
              }

              // Update upload metadata
              await client.uploads.update(upload.id, {
                customData: {
                  ...upload.customData,
                  optimizedVariants: variants,
                  optimizedAt: new Date().toISOString(),
                },
              });

              return { uploadId: upload.id, variants: variants.length };
            })
          );

          const succeeded = results.filter(r => r.status === 'fulfilled').length;
          const failed = results.filter(r => r.status === 'rejected').length;

          if (failed > 0) {
            ctx.alert(`Optimized ${succeeded} images, ${failed} failed`);
          } else {
            ctx.notice(`Successfully optimized ${succeeded} images`);
          }
        } catch (error) {
          ctx.alert('Failed to optimize images: ' + error.message);
        }
      },
    },
  ];
}
```

### 2. Export to External Service

Send media to external services:

```typescript
import { FaCloudUploadAlt, FaDropbox, FaGoogle, FaAws } from 'react-icons/fa';

uploadsDropdownActions(ctx: UploadsDropdownActionsCtx) {
  const actions = [];

  // Dropbox export
  if (ctx.plugin.attributes.parameters.dropboxEnabled) {
    actions.push({
      id: 'export-dropbox',
      label: 'Export to Dropbox',
      icon: <FaDropbox />,
      group: 'export',
      rank: 20,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        const accessToken = ctx.plugin.attributes.parameters.dropboxToken;
        
        if (!accessToken) {
          ctx.alert('Dropbox not configured. Please set up Dropbox integration in settings.');
          return;
        }

        ctx.notice(`Exporting ${ctx.uploads.length} files to Dropbox...`);

        try {
          const results = await Promise.allSettled(
            ctx.uploads.map(async (upload) => {
              const response = await fetch('https://content.dropboxapi.com/2/files/upload', {
                method: 'POST',
                headers: {
                  'Authorization': `Bearer ${accessToken}`,
                  'Dropbox-API-Arg': JSON.stringify({
                    path: `/DatoCMS/${upload.filename}`,
                    mode: 'add',
                    autorename: true,
                    mute: false,
                  }),
                  'Content-Type': 'application/octet-stream',
                },
                body: await fetch(upload.url).then(r => r.blob()),
              });

              if (!response.ok) {
                throw new Error(`Failed to upload ${upload.filename}`);
              }

              return response.json();
            })
          );

          const succeeded = results.filter(r => r.status === 'fulfilled').length;
          ctx.notice(`Successfully exported ${succeeded} files to Dropbox`);
        } catch (error) {
          ctx.alert('Export failed: ' + error.message);
        }
      },
    });
  }

  // Google Drive export
  if (ctx.plugin.attributes.parameters.googleDriveEnabled) {
    actions.push({
      id: 'export-gdrive',
      label: 'Export to Google Drive',
      icon: <FaGoogle />,
      group: 'export',
      rank: 21,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        // Implementation for Google Drive export
        ctx.notice('Exporting to Google Drive...');
      },
    });
  }

  // S3 backup
  if (ctx.plugin.attributes.parameters.s3Enabled) {
    actions.push({
      id: 'backup-s3',
      label: 'Backup to S3',
      icon: <FaAws />,
      group: 'export',
      rank: 22,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        // Implementation for S3 backup
        ctx.notice('Backing up to S3...');
      },
    });
  }

  return actions;
}
```

### 3. AI-Powered Operations

Add AI capabilities to media:

```typescript
import { FaBrain, FaTags, FaFont, FaImage } from 'react-icons/fa';

uploadsDropdownActions(ctx: UploadsDropdownActionsCtx) {
  if (!ctx.plugin.attributes.parameters.aiEnabled) {
    return [];
  }

  return [
    {
      id: 'auto-tag',
      label: 'Auto-generate Tags',
      icon: <FaTags />,
      group: 'ai',
      rank: 30,
      disabled: ctx.uploadIds.length === 0,
      tooltip: 'Use AI to generate relevant tags',
      handler: async () => {
        const imageUploads = ctx.uploads.filter(u => u.mimeType?.startsWith('image/'));
        
        if (imageUploads.length === 0) {
          ctx.alert('Please select image files for auto-tagging');
          return;
        }

        ctx.notice('Analyzing images for tags...');
        const client = ctx.createClient();
        const apiKey = ctx.plugin.attributes.parameters.aiApiKey;

        try {
          const results = await Promise.allSettled(
            imageUploads.map(async (upload) => {
              // Call AI service (e.g., Google Vision, AWS Rekognition)
              const response = await fetch('https://vision.googleapis.com/v1/images:annotate', {
                method: 'POST',
                headers: {
                  'Authorization': `Bearer ${apiKey}`,
                  'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                  requests: [{
                    image: { source: { imageUri: upload.url } },
                    features: [
                      { type: 'LABEL_DETECTION', maxResults: 10 },
                      { type: 'LANDMARK_DETECTION', maxResults: 5 },
                    ],
                  }],
                }),
              });

              const data = await response.json();
              const labels = data.responses[0].labelAnnotations || [];
              const landmarks = data.responses[0].landmarkAnnotations || [];

              const tags = [
                ...labels.filter(l => l.score > 0.7).map(l => l.description.toLowerCase()),
                ...landmarks.map(l => l.description.toLowerCase()),
              ];

              // Update upload with new tags
              await client.uploads.update(upload.id, {
                tags: [...new Set([...(upload.tags || []), ...tags])],
                customData: {
                  ...upload.customData,
                  aiAnalysis: {
                    labels,
                    landmarks,
                    analyzedAt: new Date().toISOString(),
                  },
                },
              });

              return { uploadId: upload.id, tagsAdded: tags.length };
            })
          );

          const totalTags = results
            .filter(r => r.status === 'fulfilled')
            .reduce((sum, r) => sum + r.value.tagsAdded, 0);

          ctx.notice(`Added ${totalTags} tags to ${imageUploads.length} images`);
        } catch (error) {
          ctx.alert('Auto-tagging failed: ' + error.message);
        }
      },
    },
    {
      id: 'generate-alt',
      label: 'Generate Alt Text',
      icon: <FaFont />,
      group: 'ai',
      rank: 31,
      disabled: ctx.uploadIds.length === 0,
      tooltip: 'Generate accessible alt text using AI',
      handler: async () => {
        // Implementation for alt text generation
        ctx.notice('Generating alt text...');
      },
    },
  ];
}
```

### 4. Batch Metadata Operations

Bulk edit metadata across multiple uploads:

```typescript
import { FaEdit, FaCopy, FaEraser } from 'react-icons/fa';

uploadsDropdownActions(ctx: UploadsDropdownActionsCtx) {
  return [
    {
      id: 'bulk-edit-metadata',
      label: 'Edit Metadata',
      icon: <FaEdit />,
      group: 'metadata',
      rank: 40,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        // Open modal for metadata editing
        const metadata = await ctx.openModal({
          id: 'bulkMetadataEdit',
          title: `Edit Metadata for ${ctx.uploads.length} Files`,
          width: 'medium',
          parameters: {
            uploads: ctx.uploads,
            commonFields: ['author', 'copyright', 'notes'],
          },
        });

        if (!metadata) return;

        ctx.notice('Updating metadata...');
        const client = ctx.createClient();

        try {
          await Promise.all(
            ctx.uploads.map(upload =>
              client.uploads.update(upload.id, {
                author: metadata.author || upload.author,
                copyright: metadata.copyright || upload.copyright,
                notes: metadata.notes || upload.notes,
                customData: {
                  ...upload.customData,
                  ...metadata.customData,
                },
              })
            )
          );

          ctx.notice('Metadata updated successfully');
        } catch (error) {
          ctx.alert('Failed to update metadata: ' + error.message);
        }
      },
    },
    {
      id: 'copy-metadata',
      label: 'Copy Metadata from...',
      icon: <FaCopy />,
      group: 'metadata',
      rank: 41,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        // Let user select source upload
        const sourceId = await ctx.openModal({
          id: 'selectUpload',
          title: 'Select Source Upload',
          width: 'large',
        });

        if (!sourceId) return;

        const client = ctx.createClient();
        const source = await client.uploads.find(sourceId);

        const confirmed = await ctx.openConfirm({
          title: 'Copy Metadata',
          content: `Copy metadata from "${source.filename}" to ${ctx.uploads.length} selected files?`,
          choices: [{ label: 'Copy', value: true, intent: 'primary' }],
        });

        if (!confirmed) return;

        // Copy metadata
        await Promise.all(
          ctx.uploads.map(upload =>
            client.uploads.update(upload.id, {
              alt: source.alt,
              title: source.title,
              author: source.author,
              copyright: source.copyright,
              notes: source.notes,
              tags: source.tags,
              customData: source.customData,
            })
          )
        );

        ctx.notice('Metadata copied successfully');
      },
    },
    {
      id: 'clear-metadata',
      label: 'Clear Metadata',
      icon: <FaEraser />,
      group: 'metadata',
      rank: 42,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        const confirmed = await ctx.openConfirm({
          title: 'Clear Metadata',
          content: `Clear all metadata from ${ctx.uploads.length} files?`,
          cancel: { label: 'Cancel', value: false },
          choices: [{ label: 'Clear', value: true, intent: 'negative' }],
        });

        if (!confirmed) return;

        const client = ctx.createClient();
        
        await Promise.all(
          ctx.uploads.map(upload =>
            client.uploads.update(upload.id, {
              alt: null,
              title: null,
              author: null,
              copyright: null,
              notes: null,
              tags: [],
              customData: {},
            })
          )
        );

        ctx.notice('Metadata cleared');
      },
    },
  ];
}
```

### 5. Workflow Actions

Add workflow-related actions:

```typescript
import { FaCheckCircle, FaTimesCircle, FaArchive, FaTrash } from 'react-icons/fa';

uploadsDropdownActions(ctx: UploadsDropdownActionsCtx) {
  return [
    {
      id: 'approve-assets',
      label: 'Approve Assets',
      icon: <FaCheckCircle />,
      group: 'workflow',
      rank: 50,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        const client = ctx.createClient();
        
        await Promise.all(
          ctx.uploads.map(upload =>
            client.uploads.update(upload.id, {
              customData: {
                ...upload.customData,
                approvalStatus: 'approved',
                approvedBy: ctx.currentUser.email,
                approvedAt: new Date().toISOString(),
              },
            })
          )
        );

        ctx.notice(`${ctx.uploads.length} assets approved`);
      },
    },
    {
      id: 'reject-assets',
      label: 'Reject Assets',
      icon: <FaTimesCircle />,
      group: 'workflow',
      rank: 51,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        const reason = await ctx.openModal({
          id: 'rejectionReason',
          title: 'Rejection Reason',
          width: 'small',
        });

        if (!reason) return;

        const client = ctx.createClient();
        
        await Promise.all(
          ctx.uploads.map(upload =>
            client.uploads.update(upload.id, {
              customData: {
                ...upload.customData,
                approvalStatus: 'rejected',
                rejectedBy: ctx.currentUser.email,
                rejectedAt: new Date().toISOString(),
                rejectionReason: reason,
              },
            })
          )
        );

        ctx.notice(`${ctx.uploads.length} assets rejected`);
      },
    },
    {
      id: 'archive-assets',
      label: 'Archive Assets',
      icon: <FaArchive />,
      group: 'workflow',
      rank: 52,
      disabled: ctx.uploadIds.length === 0,
      handler: async () => {
        const confirmed = await ctx.openConfirm({
          title: 'Archive Assets',
          content: `Archive ${ctx.uploads.length} assets? They will be moved to the archive.`,
          choices: [{ label: 'Archive', value: true, intent: 'primary' }],
        });

        if (!confirmed) return;

        // Implementation for archiving
        ctx.notice('Assets archived');
      },
    },
  ];
}
```

## Best Practices

1. **Group Related Actions**: Use the `group` property to organize actions
2. **Clear Labels**: Use action labels that clearly describe what will happen
3. **Meaningful Icons**: Choose icons that visually represent the action
4. **Batch Operations**: Use `Promise.allSettled()` for resilient bulk operations
5. **User Feedback**: Always show progress and results of operations
6. **Error Handling**: Handle partial failures gracefully
7. **Confirmation**: Always confirm destructive or large-scale operations
8. **Performance**: Consider chunking for very large selections
9. **Permissions**: Check user permissions before showing actions
10. **Context Awareness**: Enable/disable actions based on upload types

## Related Hooks

- `executeUploadsDropdownAction` - Handle the execution of these actions
- `uploadSidebarPanels` - Add panels to the upload sidebar
- `assetSources` - Add custom asset sources

## Important Notes

- Actions operate on the selected uploads
- The `handler` function is called when the action is clicked
- Always provide feedback about operation progress and results
- Consider file types when showing actions (e.g., image-only operations)
- Use `Promise.allSettled()` to handle partial failures in bulk operations
- Some operations may require additional API permissions