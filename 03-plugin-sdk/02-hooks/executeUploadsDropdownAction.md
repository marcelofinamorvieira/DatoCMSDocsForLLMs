# executeUploadsDropdownAction Hook

## Purpose

The `executeUploadsDropdownAction` hook executes bulk actions on multiple media assets when users select them from the dropdown menu in the media area. This hook handles operations like bulk optimization, metadata updates, batch transformations, or any custom processing you need to perform on selected uploads.

## Function Signature

```typescript
import { connect, IntentCtx } from 'datocms-plugin-sdk';
import { RenderConfigScreenCtx } from 'datocms-plugin-sdk';

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    // Implementation here
  }
});
```

## Parameters

- `actionId`: string - The unique identifier of the action being executed (matches ID from uploadsDropdownActions)
- `uploads`: Upload[] - Array of selected media assets on which the action should be executed
- `ctx`: ExecuteUploadsDropdownActionCtx - Context object with utilities and site data

## Context Type Definition

```typescript
type ExecuteUploadsDropdownActionCtx = {
  // Plugin configuration
  plugin: {
    id: string;
    attributes: {
      parameters: Record<string, unknown>;
      settings: Record<string, unknown>;
    };
  };

  // Current user information
  currentUser: {
    id: string;
    attributes: {
      email: string;
      first_name: string;
      last_name: string;
      avatar_url?: string;
      can_edit_uploads: boolean;
      can_delete_uploads: boolean;
    };
  };

  // Site information
  site: {
    id: string;
    attributes: {
      name: string;
      locale: string;
      locales: string[];
      domain?: string;
      internal_domain: string;
      no_index: boolean;
    };
  };

  // Upload operations
  updateUpload: (uploadId: string, updates: UploadUpdateParams) => Promise<Upload>;
  updateUploads: (uploadIds: string[], updates: UploadUpdateParams) => Promise<Upload[]>;
  deleteUploads: (uploadIds: string[]) => Promise<void>;
  uploadFromUrl: (url: string, options: UploadFromUrlOptions) => Promise<Upload>;
  createUploadUrl: (uploadId: string, transformations?: ImageTransformations) => string;

  // Progress tracking
  reportProgress: (progress: number, message?: string) => void;

  // UI methods
  notice: (message: string) => void;
  alert: (message: string) => void;
  openModal: <P = Record<string, unknown>, R = unknown>(options: {
    id: string;
    title: string;
    width?: 's' | 'm' | 'l' | 'xl' | 'fullWidth';
    parameters?: P;
  }) => Promise<R | null>;
  openConfirm: (options: {
    title: string;
    content: string;
    choices: Array<{
      label: string;
      value: boolean;
      intent?: 'positive' | 'negative';
    }>;
  }) => Promise<boolean>;

  // Environment info
  environment: 'development' | 'production';
  locale: string;
};

// Upload type with complete attributes
type Upload = {
  id: string;
  type: 'upload';
  attributes: {
    filename: string;
    basename: string;
    format: string;
    mime_type: string;
    size: number;
    url: string;
    path: string;
    
    is_image: boolean;
    width: number | null;
    height: number | null;
    alt: string | null;
    title: string | null;
    custom_data: Record<string, string>;
    focal_point: { x: number; y: number } | null;
    blurhash?: string;
    colors?: Array<{ hex: string; alpha: number }>;
    thumbhash?: string | null;
    default_field_metadata: {
      [locale: string]: {
        alt: string | null;
        title: string | null;
        custom_data: Record<string, string>;
        focal_point: { x: number; y: number } | null;
      };
    };
  };
  relationships: {
    upload_collection?: {
      data?: {
        id: string;
        type: 'upload_collection';
      };
    };
  };
  meta: {
    created_at: string;
    updated_at: string;
    is_valid: boolean;
    upload_collection?: {
      id: string;
      name: string;
    };
  };
};

// Upload update parameters
type UploadUpdateParams = {
  filename?: string;
  alt?: string;
  title?: string;
  custom_data?: Record<string, unknown>;
  metadata?: Record<string, unknown>;
  tags?: string[];
  focal_point?: {
    x: number;
    y: number;
  };
  upload_collection_id?: string | null;
};

// Upload from URL options
type UploadFromUrlOptions = {
  filename?: string;
  alt?: string;
  title?: string;
  custom_data?: Record<string, unknown>;
  tags?: string[];
  upload_collection_id?: string;
  default_field_metadata?: Record<string, {
    alt?: string;
    title?: string;
    custom_data?: Record<string, unknown>;
    focal_point?: {
      x: number;
      y: number;
    };
  }>;
};

// Image transformation options
type ImageTransformations = {
  w?: number;                    // Width
  h?: number;                    // Height
  fit?: 'crop' | 'clip' | 'scale' | 'fillmax' | 'max'; // Fit mode
  crop?: 'top' | 'bottom' | 'left' | 'right' | 'center' | 'face' | 'faces' | 'entropy' | 'edges' | 'focalpoint'; // Crop mode
  q?: number;                    // Quality (1-100)
  auto?: 'compress' | 'format' | 'enhance'; // Auto optimizations
  fm?: 'jpg' | 'png' | 'webp' | 'gif' | 'avif'; // Format
  bg?: string;                   // Background color
  blur?: number;                 // Blur amount
  sharp?: number;                // Sharpening amount
  sat?: number;                  // Saturation adjustment
  hue?: number;                  // Hue adjustment
  con?: number;                  // Contrast adjustment
  bri?: number;                  // Brightness adjustment
  monochrome?: boolean;          // Convert to monochrome
  invert?: boolean;              // Invert colors
  watermark?: string;            // Watermark asset ID
  'watermark-position'?: 'top-left' | 'top-center' | 'top-right' | 'middle-left' | 'middle-center' | 'middle-right' | 'bottom-left' | 'bottom-center' | 'bottom-right';
  'watermark-opacity'?: number;  // Watermark opacity (0-100)
  'watermark-scale'?: number;    // Watermark scale (0-300)
};
```

## Use Cases

### 1. Bulk Image Optimization

Optimize multiple images at once:

```typescript
import { connect } from 'datocms-plugin-sdk';

type OptimizationOptions = {
  quality: number;
  maxWidth: number;
  format?: string;
  replaceOriginal?: boolean;
};

type OptimizationResult = {
  optimized: number;
  savedBytes: number;
  errors: Array<{
    upload: Upload;
    error: string;
  }>;
};

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    if (actionId === 'optimize-images') {

      // Filter for images only
      const images = uploads.filter((u: Upload) => 
        u.attributes.mime_type.startsWith('image/')
      );

      if (images.length === 0) {
        ctx.alert('No images selected');
        return;
      }

      // Show optimization options
      const options = await ctx.openModal<{
        images: Upload[];
        presets: Array<{
          name: string;
          quality: number;
          maxWidth: number;
        }>;
      }, OptimizationOptions>({
        id: 'optimization-options',
        title: `Optimize ${images.length} Images`,
        width: 'm',
        parameters: {
          images,
          presets: [
            { name: 'Web optimized', quality: 85, maxWidth: 1920 },
            { name: 'Thumbnail', quality: 80, maxWidth: 400 },
            { name: 'High compression', quality: 60, maxWidth: 1200 }
          ]
        }
      });

      if (!options) return;

      try {
        ctx.notice(`Optimizing ${images.length} images...`);

        const results: OptimizationResult = {
          optimized: 0,
          savedBytes: 0,
          errors: []
        };

      for (let i = 0; i < images.length; i++) {
        const image = images[i];
        ctx.reportProgress(
          Math.round((i / images.length) * 100),
          `Optimizing ${image.attributes.filename}`
        );

        try {
          // Skip if already optimized
          if (image.attributes.metadata?.optimized_at) {
            const skipNew = Date.now() - new Date(image.attributes.metadata.optimized_at).getTime();
            if (skipNew < 7 * 24 * 60 * 60 * 1000) { // 7 days
              continue;
            }
          }

          // Optimize image
          const optimized = await optimizeImage({
            url: image.attributes.url,
            quality: options.quality,
            maxWidth: options.maxWidth,
            format: options.format || image.attributes.format
          });

          // Upload optimized version
          const newUpload = await ctx.uploadFromUrl(optimized.url, {
            filename: image.attributes.filename,
            default_field_metadata: {
              en: {
                alt: image.attributes.alt,
                title: image.attributes.title,
                custom_data: {
                  ...image.attributes.custom_data,
                  original_id: image.id,
                  optimized_at: new Date().toISOString(),
                  optimization_settings: options
                }
              }
            }
          });

          // Calculate savings
          const savedBytes = image.attributes.size - newUpload.attributes.size;
          results.savedBytes += savedBytes;
          results.optimized++;

          // Optional: Replace original with optimized
          if (options.replaceOriginal) {
            await replaceUploadReferences(image.id, newUpload.id);
            await ctx.deleteUploads([image.id]);
          }
        } catch (error) {
          results.errors.push({
            upload: image,
            error: error.message
          });
        }
      }

      // Show results
      const savedMB = (results.savedBytes / 1024 / 1024).toFixed(2);
      ctx.notice(`✅ Optimized ${results.optimized} images, saved ${savedMB}MB`);

      if (results.errors.length > 0) {
        await ctx.openModal({
          id: 'optimization-errors',
          title: 'Optimization Errors',
          width: 'm',
          parameters: { errors: results.errors }
        });
      }
      } catch (error) {
        ctx.alert(`Optimization failed: ${error.message}`);
      }
    }

    if (actionId === 'generate-responsive-variants') {
    const { uploads } = ctx;

    const images = uploads.filter(u => 
      u.attributes.mime_type.startsWith('image/') &&
      !u.attributes.is_image_derivative
    );

    if (images.length === 0) {
      ctx.alert('No suitable images selected');
      return;
    }

    const config = await ctx.openModal({
      id: 'responsive-config',
      title: 'Generate Responsive Variants',
      width: 'm',
      parameters: {
        breakpoints: [
          { name: 'mobile', width: 640 },
          { name: 'tablet', width: 1024 },
          { name: 'desktop', width: 1920 },
          { name: 'retina', width: 3840 }
        ]
      }
    });

    if (!config) return;

    try {
      for (let i = 0; i < images.length; i++) {
        const image = images[i];
        ctx.reportProgress(
          Math.round((i / images.length) * 100),
          `Processing ${image.attributes.filename}`
        );

        for (const breakpoint of config.breakpoints) {
          // Skip if image is smaller than breakpoint
          if (image.attributes.width <= breakpoint.width) continue;

          // Generate variant
          const variantUrl = ctx.createUploadUrl(image.id, {
            w: breakpoint.width,
            q: 85,
            auto: 'format'
          });

          // Store variant metadata
          await ctx.updateUpload(image.id, {
            metadata: {
              ...image.attributes.metadata,
              responsive_variants: {
                ...image.attributes.metadata?.responsive_variants,
                [breakpoint.name]: variantUrl
              }
            }
          });
        }
      }

        ctx.notice(`Generated responsive variants for ${images.length} images`);
      } catch (error) {
        ctx.alert(`Failed to generate variants: ${error.message}`);
      }
    }
  }
});
```

### 2. Metadata Management

Update metadata for multiple assets:

```typescript
import { connect } from 'datocms-plugin-sdk';

type MetadataField = {
  name: string;
  type: 'text' | 'tags';
  label: string;
};

type MetadataFormValues = {
  alt?: string;
  title?: string;
  copyright?: string;
  author?: string;
  tags?: string[];
  custom_data?: Record<string, unknown>;
};

type ExifData = {
  Make?: string;
  Model?: string;
  LensModel?: string;
  FocalLength?: string;
  FNumber?: string;
  ExposureTime?: string;
  ISO?: string;
  DateTimeOriginal?: string;
  GPSLatitude?: number;
  GPSLongitude?: number;
};

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    if (actionId === 'bulk-update-metadata') {

      if (uploads.length === 0) {
        ctx.alert('No uploads selected');
        return;
      }

      // Helper function to extract common metadata
      const extractCommonMetadata = (uploads: Upload[]): Partial<MetadataFormValues> => {
        const firstUpload = uploads[0];
        return {
          alt: uploads.every(u => u.attributes.alt === firstUpload.attributes.alt) 
            ? firstUpload.attributes.alt : undefined,
          title: uploads.every(u => u.attributes.title === firstUpload.attributes.title) 
            ? firstUpload.attributes.title : undefined
        };
      };

      // Show metadata form
      const metadata = await ctx.openModal<{
        uploads: Upload[];
        fields: MetadataField[];
        currentValues: Partial<MetadataFormValues>;
      }, MetadataFormValues>({
        id: 'metadata-editor',
        title: `Update Metadata for ${uploads.length} Assets`,
        width: 'l',
        parameters: {
          uploads,
          fields: [
            { name: 'alt', type: 'text', label: 'Alt Text' },
            { name: 'title', type: 'text', label: 'Title' },
            { name: 'copyright', type: 'text', label: 'Copyright' },
            { name: 'author', type: 'text', label: 'Author' },
            { name: 'tags', type: 'tags', label: 'Tags' }
          ],
          currentValues: extractCommonMetadata(uploads)
        }
      });

      if (!metadata) return;

      try {
        const updates: Partial<UploadUpdateParams> = {};

      // Build update object
      if (metadata.alt !== undefined) updates.alt = metadata.alt;
      if (metadata.title !== undefined) updates.title = metadata.title;
      if (metadata.custom_data) {
        updates.custom_data = {
          ...metadata.custom_data
        };
      }

      // Update all selected uploads
      const uploadIds = uploads.map(u => u.id);
      await ctx.updateUploads(uploadIds, updates);

      ctx.notice(`Updated metadata for ${uploads.length} assets`);
    } catch (error) {
      ctx.alert(`Metadata update failed: ${error.message}`);
    }
  }

  if (actionId === 'extract-image-metadata') {
    const { uploads } = ctx;

    const images = uploads.filter(u => 
      u.attributes.mime_type.startsWith('image/') &&
      !u.attributes.metadata?.exif_extracted
    );

    if (images.length === 0) {
      ctx.alert('No images need metadata extraction');
      return;
    }

    try {
      ctx.notice('Extracting EXIF metadata...');

      let extracted = 0;
      for (let i = 0; i < images.length; i++) {
        const image = images[i];
        ctx.reportProgress(
          Math.round((i / images.length) * 100),
          `Processing ${image.attributes.filename}`
        );

        try {
          // Extract EXIF data
          const exifData = await extractEXIF(image.attributes.url);

          if (exifData) {
            const metadata = {
              camera: exifData.Make && exifData.Model ? 
                `${exifData.Make} ${exifData.Model}` : null,
              lens: exifData.LensModel,
              focal_length: exifData.FocalLength,
              aperture: exifData.FNumber,
              shutter_speed: exifData.ExposureTime,
              iso: exifData.ISO,
              taken_at: exifData.DateTimeOriginal,
              gps: exifData.GPSLatitude && exifData.GPSLongitude ? {
                lat: exifData.GPSLatitude,
                lng: exifData.GPSLongitude
              } : null,
              exif_extracted: true
            };

            await ctx.updateUpload(image.id, {
              custom_data: {
                ...image.attributes.custom_data,
                ...metadata
              }
            });

            extracted++;
          }
        } catch (error) {
          console.error(`Failed to extract EXIF from ${image.id}:`, error);
        }
      }

        ctx.notice(`Extracted metadata from ${extracted} images`);
      } catch (error) {
        ctx.alert(`Metadata extraction failed: ${error.message}`);
      }
    }
  }
});

// Helper function to extract EXIF data (would be implemented separately)
declare function extractEXIF(url: string): Promise<ExifData | null>;
```

### 3. Asset Organization

Organize and categorize uploads:

```typescript
import { connect } from 'datocms-plugin-sdk';

type AssetGroups = {
  images: Upload[];
  videos: Upload[];
  documents: Upload[];
  other: Upload[];
};

type OrganizationPlan = {
  [K in keyof AssetGroups]?: {
    folder?: string;
    tags?: string[];
  };
};

type OrganizationSuggestion = {
  folder: string;
  tags: string[];
};

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    if (actionId === 'organize-by-type') {
      // Group uploads by type
      const groups: AssetGroups = {
        images: uploads.filter((u: Upload) => u.attributes.mime_type.startsWith('image/')),
        videos: uploads.filter((u: Upload) => u.attributes.mime_type.startsWith('video/')),
        documents: uploads.filter((u: Upload) => 
          u.attributes.mime_type.includes('pdf') ||
          u.attributes.mime_type.includes('document') ||
          u.attributes.mime_type.includes('sheet') ||
          u.attributes.mime_type.includes('presentation')
        ),
        other: uploads.filter((u: Upload) => 
          !u.attributes.mime_type.startsWith('image/') &&
          !u.attributes.mime_type.startsWith('video/') &&
          !u.attributes.mime_type.includes('pdf') &&
          !u.attributes.mime_type.includes('document')
        )
      };

      // Show organization options
      const organizationPlan = await ctx.openModal<{
        groups: AssetGroups;
        suggestions: Record<keyof AssetGroups, OrganizationSuggestion>;
      }, OrganizationPlan>({
        id: 'organization-plan',
        title: 'Organize Assets by Type',
        width: 'l',
        parameters: {
          groups,
          suggestions: {
            images: { folder: 'images', tags: ['image'] },
            videos: { folder: 'videos', tags: ['video'] },
            documents: { folder: 'documents', tags: ['document'] },
            other: { folder: 'misc', tags: ['other'] }
          }
        }
      });

    if (!organizationPlan) return;

    try {
      let organized = 0;

      for (const [type, assets] of Object.entries(groups)) {
        if (assets.length === 0) continue;
        
        const plan = organizationPlan[type];
        if (!plan) continue;

        const updates = {};
        
        if (plan.folder) {
          updates.custom_data = {
            folder: plan.folder
          };
        }

        if (plan.tags && plan.tags.length > 0) {
          updates.tags = plan.tags;
        }

        await ctx.updateUploads(
          assets.map(a => a.id),
          updates
        );

        organized += assets.length;
      }

      ctx.notice(`Organized ${organized} assets`);
    } catch (error) {
      ctx.alert(`Organization failed: ${error.message}`);
    }
  }

  if (actionId === 'auto-tag') {
    const { uploads } = ctx;

    const untaggedImages = uploads.filter(u => 
      u.attributes.mime_type.startsWith('image/') &&
      (!u.attributes.tags || u.attributes.tags.length === 0)
    );

    if (untaggedImages.length === 0) {
      ctx.alert('No untagged images found');
      return;
    }

    const taggingOptions = await ctx.openModal({
      id: 'auto-tag-options',
      title: 'Auto-tag Images',
      width: 'm',
      parameters: {
        imageCount: untaggedImages.length,
        services: ['ai-vision', 'color-detection', 'face-detection']
      }
    });

    if (!taggingOptions) return;

    try {
      ctx.notice('Analyzing images for auto-tagging...');

      const results = {
        tagged: 0,
        totalTags: 0
      };

      for (let i = 0; i < untaggedImages.length; i++) {
        const image = untaggedImages[i];
        ctx.reportProgress(
          Math.round((i / untaggedImages.length) * 100),
          `Analyzing ${image.attributes.filename}`
        );

        const tags = new Set();

        // AI Vision tagging
        if (taggingOptions.services.includes('ai-vision')) {
          const visionTags = await analyzeImageWithAI(image.attributes.url);
          visionTags.forEach(tag => tags.add(tag));
        }

        // Color detection
        if (taggingOptions.services.includes('color-detection')) {
          const colors = await detectDominantColors(image.attributes.url);
          colors.forEach(color => tags.add(`color:${color}`));
        }

        // Face detection
        if (taggingOptions.services.includes('face-detection')) {
          const faces = await detectFaces(image.attributes.url);
          if (faces > 0) {
            tags.add('people');
            tags.add(`faces:${faces}`);
          }
        }

        // Add dimension-based tags
        if (image.attributes.width > 3000) tags.add('high-res');
        if (image.attributes.width === image.attributes.height) tags.add('square');
        if (image.attributes.width > image.attributes.height * 2) tags.add('panoramic');

        if (tags.size > 0) {
          await ctx.updateUpload(image.id, {
            tags: Array.from(tags)
          });
          results.tagged++;
          results.totalTags += tags.size;
        }
      }

        ctx.notice(`Tagged ${results.tagged} images with ${results.totalTags} tags`);
      } catch (error) {
        ctx.alert(`Auto-tagging failed: ${error.message}`);
      }
    }
  }
});

// Helper functions (would be implemented separately)
declare function analyzeImageWithAI(url: string): Promise<string[]>;
declare function detectDominantColors(url: string): Promise<string[]>;
declare function detectFaces(url: string): Promise<number>;
```

### 4. Batch Transformations

Apply transformations to multiple assets:

```typescript
import { connect } from 'datocms-plugin-sdk';

type ResizePreset = {
  name: string;
  width: number | null;
  height: number | null;
  mode: 'crop' | 'fit';
};

type ResizeOptions = {
  width?: number;
  height?: number;
  mode: 'crop' | 'fit';
  quality?: number;
  prefix?: string;
  name: string;
};

type WatermarkConfig = {
  watermarkId: string;
  position: string;
  opacity: number;
  scale?: number;
};

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    if (actionId === 'batch-resize') {

      const images = uploads.filter((u: Upload) => u.attributes.mime_type.startsWith('image/'));
      
      if (images.length === 0) {
        ctx.alert('No images selected');
        return;
      }

      const resizeOptions = await ctx.openModal<{
        presets: ResizePreset[];
      }, ResizeOptions>({
        id: 'resize-options',
        title: `Resize ${images.length} Images`,
        width: 'm',
        parameters: {
          presets: [
            { name: 'Thumbnail', width: 200, height: 200, mode: 'crop' },
            { name: 'Medium', width: 800, height: null, mode: 'fit' },
            { name: 'Large', width: 1600, height: null, mode: 'fit' },
            { name: 'Square', width: 500, height: 500, mode: 'crop' },
            { name: 'Custom', width: null, height: null, mode: 'fit' }
          ]
        }
      });

    if (!resizeOptions) return;

    try {
      ctx.notice(`Resizing ${images.length} images...`);

      for (let i = 0; i < images.length; i++) {
        const image = images[i];
        ctx.reportProgress(
          Math.round((i / images.length) * 100),
          `Resizing ${image.attributes.filename}`
        );

        // Generate resized version
        const transformations = {
          w: resizeOptions.width,
          h: resizeOptions.height,
          fit: resizeOptions.mode,
          q: resizeOptions.quality || 85
        };

        const resizedUrl = ctx.createUploadUrl(image.id, transformations);

        // Create new upload for resized version
        const filename = `${resizeOptions.prefix || 'resized'}_${image.attributes.filename}`;
        
        await ctx.uploadFromUrl(resizedUrl, {
          filename,
          default_field_metadata: {
            en: {
              alt: image.attributes.alt,
              title: `${image.attributes.title} (${resizeOptions.name})`,
              custom_data: {
                original_id: image.id,
                transformation: transformations,
                created_from: 'batch-resize'
              }
            }
          }
        });
      }

      ctx.notice(`Successfully resized ${images.length} images`);
    } catch (error) {
      ctx.alert(`Resize operation failed: ${error.message}`);
    }
  }

  if (actionId === 'apply-watermark') {
    const { uploads } = ctx;

    const images = uploads.filter(u => 
      u.attributes.mime_type.startsWith('image/') &&
      !u.attributes.custom_data?.watermarked
    );

    if (images.length === 0) {
      ctx.alert('No images to watermark');
      return;
    }

    const watermarkConfig = await ctx.openModal({
      id: 'watermark-config',
      title: 'Apply Watermark',
      width: 'm',
      parameters: {
        watermarks: await loadAvailableWatermarks(),
        positions: ['top-left', 'top-right', 'bottom-left', 'bottom-right', 'center'],
        opacity: [0.3, 0.5, 0.7, 1.0]
      }
    });

    if (!watermarkConfig) return;

    try {
      for (let i = 0; i < images.length; i++) {
        const image = images[i];
        ctx.reportProgress(
          Math.round((i / images.length) * 100),
          `Watermarking ${image.attributes.filename}`
        );

        // Apply watermark
        const watermarkedUrl = await applyWatermark({
          imageUrl: image.attributes.url,
          watermarkId: watermarkConfig.watermarkId,
          position: watermarkConfig.position,
          opacity: watermarkConfig.opacity,
          scale: watermarkConfig.scale
        });

        // Create watermarked version
        await ctx.uploadFromUrl(watermarkedUrl, {
          filename: `watermarked_${image.attributes.filename}`,
          default_field_metadata: {
            en: {
              alt: image.attributes.alt,
              title: `${image.attributes.title} (Watermarked)`,
              custom_data: {
                original_id: image.id,
                watermarked: true,
                watermark_config: watermarkConfig
              }
            }
          }
        });
      }

        ctx.notice(`Applied watermark to ${images.length} images`);
      } catch (error) {
        ctx.alert(`Watermark application failed: ${error.message}`);
      }
    }
  }
});

// Helper functions (would be implemented separately)
declare function applyWatermark(options: {
  imageUrl: string;
  watermarkId: string;
  position: string;
  opacity: number;
  scale?: number;
}): Promise<string>;

declare function loadAvailableWatermarks(): Promise<Array<{
  id: string;
  name: string;
  url: string;
}>>;
```

### 5. Export and Backup

Export selected assets:

```typescript
import { connect } from 'datocms-plugin-sdk';

type ExportOptions = {
  includeMetadata: boolean;
  maxSize: number; // in MB
};

type ExportManifest = {
  exported_at: string;
  total_assets: number;
  total_size: number;
  assets: Array<{
    id: string;
    filename: string;
    mime_type: string;
    size: number;
    dimensions?: {
      width: number;
      height: number;
    } | null;
    metadata?: {
      alt?: string;
      title?: string;
      custom_data?: Record<string, unknown>;
    } | null;
    url: string;
  }>;
};

type DownloadLink = {
  filename: string;
  url: string;
};

type CloudService = {
  id: string;
  name: string;
  type: 's3' | 'gcs' | 'azure';
};

type BackupConfig = {
  service: string;
  path: string;
  preserveStructure: boolean;
};

type BackupResult = {
  success: number;
  failed: number;
  errors: Array<{
    upload: Upload;
    error: string;
  }>;
};

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    if (actionId === 'export-zip') {

      if (uploads.length === 0) {
        ctx.alert('No uploads selected');
        return;
      }

      const exportOptions = await ctx.openModal<{
        includeMetadata: boolean;
        maxSize: number;
      }, ExportOptions>({
        id: 'export-options',
        title: `Export ${uploads.length} Assets`,
        width: 's',
        parameters: {
          includeMetadata: true,
          maxSize: 500 // MB
        }
      });

    if (!exportOptions) return;

    try {
      ctx.notice('Preparing export...');

      // Check total size
      const totalSize = uploads.reduce((sum, u) => sum + u.attributes.size, 0);
      const totalSizeMB = totalSize / 1024 / 1024;

      if (totalSizeMB > exportOptions.maxSize) {
        const proceed = await ctx.openConfirm({
          title: 'Large Export',
          content: `Total size is ${totalSizeMB.toFixed(2)}MB. This may take a while. Continue?`,
          choices: [
            { label: 'Cancel', value: false },
            { label: 'Continue', value: true }
          ]
        });

        if (!proceed) return;
      }

      // Create export manifest
      const manifest = {
        exported_at: new Date().toISOString(),
        total_assets: uploads.length,
        total_size: totalSize,
        assets: uploads.map(u => ({
          id: u.id,
          filename: u.attributes.filename,
          mime_type: u.attributes.mime_type,
          size: u.attributes.size,
          dimensions: u.attributes.is_image ? {
            width: u.attributes.width,
            height: u.attributes.height
          } : null,
          metadata: exportOptions.includeMetadata ? {
            alt: u.attributes.alt,
            title: u.attributes.title,
            custom_data: u.attributes.custom_data
          } : null,
          url: u.attributes.url
        }))
      };

      // Generate download links
      const downloadLinks = [];
      for (const upload of uploads) {
        downloadLinks.push({
          filename: upload.attributes.filename,
          url: upload.attributes.url
        });
      }

      // Create zip file
      const zipUrl = await createZipFromUrls(downloadLinks, manifest);

      // Download zip
      const link = document.createElement('a');
      link.href = zipUrl;
      link.download = `datocms-export-${Date.now()}.zip`;
      link.click();

      ctx.notice('Export completed successfully');
    } catch (error) {
      ctx.alert(`Export failed: ${error.message}`);
    }
  }

  if (actionId === 'backup-to-cloud') {
    const { uploads } = ctx;

    const cloudServices = await getConfiguredCloudServices(ctx.plugin.attributes.parameters);
    
    if (cloudServices.length === 0) {
      ctx.alert('No cloud storage services configured');
      return;
    }

    const backupConfig = await ctx.openModal({
      id: 'cloud-backup',
      title: 'Backup to Cloud Storage',
      width: 'm',
      parameters: {
        uploads,
        services: cloudServices
      }
    });

    if (!backupConfig) return;

    try {
      ctx.notice('Starting cloud backup...');

      const results = {
        success: 0,
        failed: 0,
        errors: []
      };

      for (let i = 0; i < uploads.length; i++) {
        const upload = uploads[i];
        ctx.reportProgress(
          Math.round((i / uploads.length) * 100),
          `Backing up ${upload.attributes.filename}`
        );

        try {
          await backupToCloud({
            upload,
            service: backupConfig.service,
            path: backupConfig.path,
            preserveStructure: backupConfig.preserveStructure
          });

          // Mark as backed up
          await ctx.updateUpload(upload.id, {
            custom_data: {
              ...upload.attributes.custom_data,
              backed_up: {
                service: backupConfig.service,
                path: backupConfig.path,
                date: new Date().toISOString()
              }
            }
          });

          results.success++;
        } catch (error) {
          results.failed++;
          results.errors.push({
            upload,
            error: error.message
          });
        }
      }

      if (results.failed > 0) {
        await ctx.openModal({
          id: 'backup-errors',
          title: 'Backup Completed with Errors',
          width: 'm',
          parameters: results
        });
      } else {
        ctx.notice(`✅ Backed up ${results.success} assets to ${backupConfig.service}`);
      }
    } catch (error) {
      ctx.alert(`Backup failed: ${error.message}`);
    }
  }
});

// Helper functions (would be implemented separately)
declare function createZipFromUrls(
  downloadLinks: DownloadLink[], 
  manifest: ExportManifest
): Promise<string>;

declare function getConfiguredCloudServices(
  parameters: Record<string, unknown>
): Promise<CloudService[]>;

declare function backupToCloud(options: {
  upload: Upload;
  service: string;
  path: string;
  preserveStructure: boolean;
}): Promise<void>;
```

### 6. Compliance and Security

Handle compliance-related operations:

```typescript
import { connect } from 'datocms-plugin-sdk';

type ComplianceIssues = {
  missingAlt: Upload[];
  missingCopyright: Upload[];
  sensitiveContent: Upload[];
  largeFiles: Upload[];
  expiredLicenses: Upload[];
};

type ComplianceActions = {
  fixMissingAlt: boolean;
  addCopyright: boolean;
  reviewSensitive: boolean;
};

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    if (actionId === 'check-compliance') {
      ctx.notice('Running compliance checks...');

      const issues: ComplianceIssues = {
        missingAlt: [],
        missingCopyright: [],
        sensitiveContent: [],
        largeFiles: [],
        expiredLicenses: []
      };

    for (let i = 0; i < uploads.length; i++) {
      const upload = uploads[i];
      ctx.reportProgress(
        Math.round((i / uploads.length) * 100),
        `Checking ${upload.attributes.filename}`
      );

      // Check alt text (accessibility)
      if (upload.attributes.is_image && !upload.attributes.alt) {
        issues.missingAlt.push(upload);
      }

      // Check copyright
      if (!upload.attributes.custom_data?.copyright) {
        issues.missingCopyright.push(upload);
      }

      // Check for sensitive content
      if (upload.attributes.tags?.some(tag => 
        ['personal-data', 'confidential', 'restricted'].includes(tag)
      )) {
        issues.sensitiveContent.push(upload);
      }

      // Check file size
      if (upload.attributes.size > 50 * 1024 * 1024) { // 50MB
        issues.largeFiles.push(upload);
      }

      // Check license expiry
      const licenseExpiry = upload.attributes.custom_data?.license_expires_at;
      if (licenseExpiry && new Date(licenseExpiry) < new Date()) {
        issues.expiredLicenses.push(upload);
      }
    }

      // Show compliance report
      await ctx.openModal<{
        issues: ComplianceIssues;
        total: number;
        actions: ComplianceActions;
      }>({
        id: 'compliance-report',
        title: 'Compliance Check Results',
        width: 'xl',
        parameters: {
          issues,
          total: uploads.length,
          actions: {
            fixMissingAlt: issues.missingAlt.length > 0,
            addCopyright: issues.missingCopyright.length > 0,
            reviewSensitive: issues.sensitiveContent.length > 0
          }
        }
      });
  }

  if (actionId === 'anonymize-metadata') {
    const { uploads } = ctx;

    const confirmed = await ctx.openConfirm({
      title: 'Anonymize Metadata',
      content: `Remove personal information from ${uploads.length} assets? This includes EXIF data, author info, and GPS coordinates.`,
      choices: [
        { label: 'Cancel', value: false },
        { label: 'Anonymize', value: true, intent: 'negative' }
      ]
    });

    if (!confirmed) return;

    try {
      const fieldsToRemove = [
        'author',
        'copyright_holder',
        'photographer',
        'camera',
        'lens',
        'gps',
        'taken_at'
      ];

      for (const upload of uploads) {
        const customData = { ...upload.attributes.custom_data };
        
        // Remove specified fields
        fieldsToRemove.forEach(field => {
          delete customData[field];
        });

        // Add anonymization flag
        customData.anonymized = true;
        customData.anonymized_at = new Date().toISOString();

        await ctx.updateUpload(upload.id, {
          custom_data: customData
        });
      }

        ctx.notice(`Anonymized metadata for ${uploads.length} assets`);
      } catch (error) {
        ctx.alert(`Anonymization failed: ${error.message}`);
      }
    }
  }
});
```

## Best Practices

1. **Filter Asset Types**: Check mime types before processing
2. **Show Progress**: Use `reportProgress()` for long operations
3. **Batch Operations**: Process in batches to avoid timeouts
4. **Preserve Originals**: Consider keeping originals when transforming
5. **Handle Errors**: Continue processing even if some assets fail
6. **Validate Options**: Ensure user inputs are valid before processing
7. **Resource Limits**: Be mindful of API rate limits and quotas

## Error Handling

```typescript
import { connect } from 'datocms-plugin-sdk';

type ProcessingError = {
  upload: Upload;
  error: Error;
};

connect({
  executeUploadsDropdownAction: async (
    actionId: string,
    uploads: Upload[],
    ctx: ExecuteUploadsDropdownActionCtx
  ): Promise<void> => {
    try {
      // Validate selection
      if (uploads.length === 0) {
        ctx.alert('No uploads selected');
        return;
      }

      // Check permissions
      if (!ctx.currentUser.attributes.can_edit_uploads) {
        ctx.alert('You do not have permission to modify uploads');
        return;
      }

      // Execute action with error tracking
      const errors: ProcessingError[] = [];
      
      for (const upload of uploads) {
        try {
          await processUpload(upload, ctx);
        } catch (error) {
          errors.push({ 
            upload, 
            error: error instanceof Error ? error : new Error(String(error))
          });
        }
      }

      // Report results
      if (errors.length > 0) {
        await showErrorReport(errors, ctx);
      }
      
    } catch (error) {
      console.error('Upload action failed:', error);
      const message = error instanceof Error ? error.message : String(error);
      ctx.alert(`Operation failed: ${message}`);
    }
  }
});

// Helper functions (would be implemented separately)
declare function processUpload(
  upload: Upload, 
  ctx: ExecuteUploadsDropdownActionCtx
): Promise<void>;

declare function showErrorReport(
  errors: ProcessingError[], 
  ctx: ExecuteUploadsDropdownActionCtx
): Promise<void>;
```

## Related Hooks

- **uploadsDropdownActions**: Defines available upload actions
- **uploadSidebarPanels**: Add panels to upload sidebar
- **renderUploadSidebar**: Custom upload sidebar content
- **assetSources**: Define external asset sources

## Important Notes

- Always check mime types before processing media
- Large operations should show progress and be cancellable
- Consider creating new uploads rather than replacing originals
- Respect upload quotas and storage limits
- Some transformations may not be available for all file types
- Metadata updates don't count against upload quotas
- Batch operations should handle partial failures gracefully