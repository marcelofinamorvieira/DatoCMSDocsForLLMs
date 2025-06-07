# Upload Resource

## Overview

Uploads represent media files (images, videos, documents, etc.) stored in your DatoCMS project. This is the authoritative guide for all upload operations, covering platform-specific methods, image transformations, error handling, and best practices.

## API Client Section

`client.uploads`

## Table of Contents

- [Installation](#installation)
- [Platform-Specific Upload Methods](#platform-specific-upload-methods)
  - [Node.js Methods](#nodejs-methods)
  - [Browser Methods](#browser-methods)
- [Generic Upload Method](#generic-upload-method)
- [Upload from URL](#upload-from-url)
- [API Reference](#api-reference)
- [Progress Tracking](#progress-tracking)
- [Image Transformations](#image-transformations)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)

## Installation

Choose the appropriate package for your environment:

```bash
# For Node.js environments
npm install @datocms/cma-client-node

# For browser environments
npm install @datocms/cma-client-browser
```

## Platform-Specific Upload Methods

### Node.js Methods

The Node.js client (`@datocms/cma-client-node`) provides advanced file upload capabilities including local file uploads, stream handling, progress tracking, and retry mechanisms.

#### Create from Local File

The most common method for uploading files in Node.js environments:

```javascript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Basic upload
const upload = await client.uploads.createFromLocalFile({
  localPath: './images/photo.jpg'
});

// With all options
const upload = await client.uploads.createFromLocalFile({
  localPath: '/path/to/image.jpg',
  filename: 'product-image.jpg', // Optional, defaults to file name
  skipCreationIfAlreadyExists: true, // Check MD5 hash to avoid duplicates
  author: 'Marketing Team',
  copyright: '© 2024',
  notes: 'Product launch campaign',
  tags: ['product', 'launch', '2024'],
  default_field_metadata: {
    en: {
      alt: 'New product showcase',
      title: 'Product Launch 2024',
      custom_data: {
        photographer: 'John Doe',
        location: 'Studio A'
      }
    },
    it: {
      alt: 'Presentazione nuovo prodotto',
      title: 'Lancio Prodotto 2024'
    }
  },
  upload_collection: {
    type: 'upload_collection',
    id: '12345'
  },
  onProgress: (progress) => {
    console.log(`Uploaded ${progress.uploaded} of ${progress.total} bytes`);
    console.log(`${progress.percentage}% complete`);
  }
});
```

**Parameters:**
- `localPath` (string, required): Path to the local file
- `filename` (string, optional): Override filename
- `skipCreationIfAlreadyExists` (boolean, optional): Skip if file with same MD5 exists
- `author` (string, optional): File author
- `copyright` (string, optional): Copyright information
- `notes` (string, optional): Internal notes
- `tags` (array, optional): Array of tag strings
- `default_field_metadata` (object, optional): Default metadata per locale
- `upload_collection` (object, optional): Collection assignment
- `onProgress` (function, optional): Progress callback

#### Create from URL (Node.js)

Download and upload a file from a remote URL:

```javascript
const upload = await client.uploads.createFromUrl({
  url: 'https://example.com/image.jpg',
  filename: 'downloaded-image.jpg', // Optional
  skipCreationIfAlreadyExists: true,
  author: 'External Source',
  tags: ['imported', 'external'],
  default_field_metadata: {
    en: {
      alt: 'Downloaded image',
      title: 'External asset'
    }
  },
  onProgress: (progress) => {
    if (progress.type === 'DOWNLOADING') {
      console.log(`Download: ${progress.percentage}%`);
    } else if (progress.type === 'UPLOADING') {
      console.log(`Upload: ${progress.percentage}%`);
    }
  }
});
```

**Parameters:**
- `url` (string, required): URL to download from
- All other parameters same as `createFromLocalFile`

#### Update from Local File (Node.js)

Replace the file content of an existing upload:

```javascript
const updated = await client.uploads.updateFromLocalFile('upload-id-123', {
  localPath: '/path/to/new-image.jpg',
  filename: 'replacement-image.jpg',
  onProgress: (progress) => {
    console.log(`${progress.percentage}% uploaded`);
  }
});
```

#### Update from URL (Node.js)

Replace the file content from a URL:

```javascript
const updated = await client.uploads.updateFromUrl('upload-id-123', {
  url: 'https://example.com/new-image.jpg',
  filename: 'replacement.jpg',
  onProgress: (progress) => {
    console.log(`Progress: ${progress.percentage}%`);
  }
});
```

#### Stream-Based Uploads

The Node.js client automatically handles file streams efficiently:

```typescript
import { buildClient } from '@datocms/cma-client-node';
import { uploadLocalFileAndReturnPath } from '@datocms/cma-client-node';
import fs from 'node:fs';
import { pipeline } from 'node:stream/promises';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// The SDK automatically streams files, but you can also use the lower-level utilities
async function streamUpload() {
  // First, get an upload path
  const uploadPath = await uploadLocalFileAndReturnPath(
    client,
    '/path/to/file.jpg',
    {
      filename: 'streamed-file.jpg',
      onProgress: (info) => {
        if (info.type === 'UPLOADING_FILE') {
          console.log(`Progress: ${info.payload.progress}%`);
        }
      }
    }
  );

  // Then create the upload object
  const upload = await client.uploads.create({
    path: uploadPath
  });

  return upload;
}
```

#### Cancelable Uploads

All upload operations return cancelable promises:

```typescript
import { buildClient, type CancelablePromise } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function cancelableUpload() {
  const uploadPromise: CancelablePromise<any> = client.uploads.createFromLocalFile({
    localPath: '/path/to/very-large-file.zip',
    onProgress: (info) => {
      if (info.type === 'UPLOADING_FILE') {
        console.log(`Progress: ${info.payload.progress}%`);
      }
    }
  });

  // Cancel after 5 seconds
  setTimeout(() => {
    console.log('Canceling upload...');
    uploadPromise.cancel();
  }, 5000);

  try {
    const upload = await uploadPromise;
    console.log('Upload completed:', upload.url);
  } catch (error) {
    if (error.name === 'CanceledPromiseError') {
      console.log('Upload was canceled');
    } else {
      throw error;
    }
  }
}
```

#### Large File Handling

Best practices for uploading large files:

```typescript
import { buildClient } from '@datocms/cma-client-node';
import fs from 'node:fs/promises';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function uploadLargeFile(filePath: string) {
  // Check file size
  const stats = await fs.stat(filePath);
  const fileSizeMB = stats.size / (1024 * 1024);
  
  console.log(`File size: ${fileSizeMB.toFixed(2)} MB`);
  
  if (fileSizeMB > 100) {
    console.log('Large file detected. This may take some time...');
  }
  
  const startTime = Date.now();
  let lastProgress = 0;
  
  const upload = await client.uploads.createFromLocalFile({
    localPath: filePath,
    onProgress: (info) => {
      if (info.type === 'UPLOADING_FILE') {
        const progress = info.payload.progress;
        const elapsed = (Date.now() - startTime) / 1000;
        
        if (progress > lastProgress + 10) { // Log every 10%
          lastProgress = progress;
          const rate = (fileSizeMB * (progress / 100)) / elapsed;
          const eta = (fileSizeMB - (fileSizeMB * (progress / 100))) / rate;
          
          console.log(`Progress: ${progress}% | Rate: ${rate.toFixed(2)} MB/s | ETA: ${eta.toFixed(0)}s`);
        }
      }
    }
  });
  
  const totalTime = (Date.now() - startTime) / 1000;
  console.log(`Upload completed in ${totalTime.toFixed(2)}s`);
  console.log(`Average speed: ${(fileSizeMB / totalTime).toFixed(2)} MB/s`);
  
  return upload;
}
```

### Browser Methods

The browser client (`@datocms/cma-client-browser`) provides comprehensive file upload capabilities for web applications, including file input handling, drag-and-drop, blob uploads, progress tracking, and cancelable uploads.

#### Create from File or Blob

Upload files directly from browser File or Blob objects:

```javascript
import { buildClient } from '@datocms/cma-client-browser';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// From file input
const fileInput = document.querySelector('input[type="file"]');
const file = fileInput.files[0];

const upload = await client.uploads.createFromFileOrBlob({
  fileOrBlob: file,
  filename: 'custom-name.jpg', // Optional, defaults to file.name
  author: 'User Upload',
  tags: ['user-content'],
  default_field_metadata: {
    en: {
      alt: 'User uploaded image',
      title: 'Custom upload'
    }
  },
  onProgress: (progress) => {
    updateProgressBar(progress.percentage);
  }
});

// From canvas
const canvas = document.querySelector('canvas');
canvas.toBlob(async (blob) => {
  const upload = await client.uploads.createFromFileOrBlob({
    fileOrBlob: blob,
    filename: 'canvas-export.png',
    default_field_metadata: {
      en: { alt: 'Canvas drawing' }
    }
  });
});

// From fetch response
const response = await fetch('https://example.com/image.jpg');
const blob = await response.blob();
const upload = await client.uploads.createFromFileOrBlob({
  fileOrBlob: blob,
  filename: 'fetched-image.jpg'
});
```

**Parameters:**
- `fileOrBlob` (File | Blob, required): File or Blob object
- All other parameters same as Node.js methods (except `skipCreationIfAlreadyExists`)

#### Update from File or Blob (Browser)

Replace an existing upload's file:

```javascript
const updated = await client.uploads.updateFromFileOrBlob('upload-id-123', {
  fileOrBlob: newFile,
  filename: 'replacement.jpg',
  onProgress: (progress) => {
    console.log(`Upload progress: ${progress.percentage}%`);
  }
});
```

#### Drag and Drop Upload

Implement a complete drag-and-drop upload interface:

```javascript
class DragDropUploader {
  constructor(dropZoneId) {
    this.dropZone = document.getElementById(dropZoneId);
    this.setupEventListeners();
  }

  setupEventListeners() {
    ['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
      this.dropZone.addEventListener(eventName, this.preventDefaults, false);
    });

    ['dragenter', 'dragover'].forEach(eventName => {
      this.dropZone.addEventListener(eventName, () => this.highlight(), false);
    });

    ['dragleave', 'drop'].forEach(eventName => {
      this.dropZone.addEventListener(eventName, () => this.unhighlight(), false);
    });

    this.dropZone.addEventListener('drop', (e) => this.handleDrop(e), false);
  }

  preventDefaults(e) {
    e.preventDefault();
    e.stopPropagation();
  }

  highlight() {
    this.dropZone.classList.add('highlight');
  }

  unhighlight() {
    this.dropZone.classList.remove('highlight');
  }

  async handleDrop(e) {
    const files = Array.from(e.dataTransfer.files);
    
    for (const file of files) {
      try {
        const upload = await client.uploads.createFromFileOrBlob({
          fileOrBlob: file,
          onProgress: (info) => {
            if (info.type === 'UPLOADING_FILE') {
              console.log(`${file.name}: ${info.payload.progress}%`);
            }
          }
        });
        console.log('Upload successful:', upload);
      } catch (error) {
        console.error('Upload failed:', error);
      }
    }
  }
}

// Initialize
new DragDropUploader('drop-zone');
```

#### Canvas and Image Manipulation

Upload images after client-side processing:

```javascript
class ImageProcessor {
  constructor() {
    this.canvas = document.createElement('canvas');
    this.ctx = this.canvas.getContext('2d');
  }

  async processAndUpload(file, options = {}) {
    const { maxWidth = 1920, maxHeight = 1080, quality = 0.85 } = options;

    // Load image
    const img = await this.loadImage(file);
    
    // Calculate new dimensions
    const { width, height } = this.calculateDimensions(
      img.width, img.height, maxWidth, maxHeight
    );

    // Resize on canvas
    this.canvas.width = width;
    this.canvas.height = height;
    this.ctx.drawImage(img, 0, 0, width, height);

    // Convert to blob
    const blob = await this.canvasToBlob('image/jpeg', quality);

    // Upload processed image
    const upload = await client.uploads.createFromFileOrBlob({
      fileOrBlob: blob,
      filename: `processed-${file.name}`,
      default_field_metadata: {
        en: {
          alt: 'Resized and optimized image',
          custom_data: {
            original_size: file.size,
            processed_size: blob.size,
            dimensions: `${width}x${height}`
          }
        }
      }
    });

    return upload;
  }

  loadImage(file) {
    return new Promise((resolve, reject) => {
      const img = new Image();
      const url = URL.createObjectURL(file);
      img.onload = () => {
        URL.revokeObjectURL(url);
        resolve(img);
      };
      img.onerror = reject;
      img.src = url;
    });
  }

  calculateDimensions(origWidth, origHeight, maxWidth, maxHeight) {
    const scale = Math.min(maxWidth / origWidth, maxHeight / origHeight, 1);
    return {
      width: Math.floor(origWidth * scale),
      height: Math.floor(origHeight * scale)
    };
  }

  canvasToBlob(type, quality) {
    return new Promise((resolve) => {
      this.canvas.toBlob(resolve, type, quality);
    });
  }
}
```

## Generic Upload Method

The base `create` method requires manual S3 upload handling:

```javascript
// First, request upload credentials
const uploadRequest = await client.uploadRequest.create({
  filename: 'document.pdf'
});

// Upload file to S3 using the provided URL
// You need to implement the actual S3 upload
await uploadToS3(uploadRequest.url, fileContent);

// Then create the upload record
const upload = await client.uploads.create({
  path: uploadRequest.url,
  author: 'John Doe',
  copyright: '© 2024 My Company',
  notes: 'Company presentation',
  tags: ['presentation', 'corporate'],
  default_field_metadata: {
    en: {
      alt: 'Company presentation cover',
      title: 'Annual Report 2024',
      custom_data: {
        category: 'reports'
      }
    }
  }
});
```

## Upload from URL

Both Node.js and browser clients can handle URL uploads, but Node.js has native support:

### Basic URL Upload

```javascript
const upload = await client.uploads.createFromUrl({
  url: 'https://example.com/image.jpg',
  default_field_metadata: {
    en: {
      alt: 'Product image',
      title: 'Main product photo'
    }
  }
});
```

### With Custom Filename

```javascript
const upload = await client.uploads.createFromUrl({
  url: 'https://example.com/random-name.jpg',
  filename: 'product-hero-image.jpg',
  default_field_metadata: {
    en: {
      alt: 'Hero image for product page'
    }
  }
});
```

### Bulk Upload from URLs

```javascript
async function bulkUploadFromUrls(urls) {
  const uploads = [];
  
  for (const urlData of urls) {
    try {
      const upload = await client.uploads.createFromUrl({
        url: urlData.url,
        filename: urlData.filename,
        default_field_metadata: {
          en: {
            alt: urlData.alt,
            title: urlData.title
          }
        }
      });
      
      uploads.push(upload);
      console.log(`Uploaded: ${urlData.filename}`);
    } catch (error) {
      console.error(`Failed to upload ${urlData.url}:`, error.message);
    }
  }
  
  return uploads;
}
```

## API Reference

### list() - List Uploads

Retrieve a paginated list of uploads with powerful filtering options.

**Signature**: `list(queryParams): Promise<Upload[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `filter` | object | Complex filtering options |
| `filter.type` | string | Filter by type: 'image', 'video', 'file' |
| `filter.in_use` | boolean | Filter by usage status |
| `filter.size` | object | Filter by file size |
| `filter.mime_type` | string | Filter by MIME type |
| `filter.tags` | object | Filter by tags |
| `filter.upload_collection` | object | Filter by collection |
| `order_by` | array | Sorting criteria |
| `page` | object | Pagination options |
| `page.offset` | number | Starting offset (default: 0) |
| `page.limit` | number | Number of items (default: 30, max: 500) |

**Basic Example**:
```javascript
// Get all uploads
const uploads = await client.uploads.list();

// Get images only
const images = await client.uploads.list({
  filter: { type: 'image' }
});

// Get uploads with pagination
const paginatedUploads = await client.uploads.list({
  page: { offset: 0, limit: 100 }
});

// Get recent uploads
const recentUploads = await client.uploads.list({
  order_by: ['_created_at_DESC']
});
```

**Advanced Example**:
```javascript
// Complex filtering
const specificUploads = await client.uploads.list({
  filter: {
    type: 'image',
    in_use: true,
    size: { gte: 1000000 }, // >= 1MB
    mime_type: { matches: '^image/' },
    tags: { any_in: ['product', 'hero'] }
  },
  order_by: ['-created_at'],
  page: { offset: 0, limit: 30 }
});

// Using async iterator for large datasets
for await (const upload of client.uploads.listPagedIterator()) {
  console.log(upload.filename);
}

// Get raw response with metadata
const rawResponse = await client.uploads.rawList({
  page: { limit: 20 }
});
// rawResponse structure:
// {
//   data: Upload[],
//   meta: {
//     total_count: number,
//     page_count: number,
//     per_page: number,
//     page: number
//   }
// }
```

### find() - Get Single Upload

Retrieve a single upload by ID.

**Signature**: `find(uploadId): Promise<Upload>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadId` | string | The upload ID (required) |

**Example**:
```javascript
const upload = await client.uploads.find('upload-id-123');

// Upload structure includes:
// {
//   id: 'upload_id_123456',
//   type: 'upload',
//   size: 1234567,
//   width: 1920,
//   height: 1080,
//   format: 'jpg',
//   author: 'John Doe',
//   copyright: '© 2024',
//   notes: 'Product photo',
//   default_field_metadata: {
//     en: {
//       alt: 'Product image',
//       title: 'Main product view',
//       custom_data: {}
//     }
//   },
//   is_image: true,
//   url: 'https://www.datocms-assets.com/...',
//   mime_type: 'image/jpeg',
//   tags: ['product', 'featured'],
//   created_at: '2024-01-01T00:00:00Z',
//   updated_at: '2024-01-01T00:00:00Z'
// }

// Error handling
try {
  const upload = await client.uploads.find('non_existent_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Upload not found');
  }
}
```

### create() - Create Upload

Creates a new upload (see platform-specific methods above for easier alternatives).

**Signature**: `create(body): Promise<Upload>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | string | S3 path or local file path (required) |
| `author` | string | File author (optional) |
| `copyright` | string | Copyright information (optional) |
| `notes` | string | Internal notes (optional) |
| `tags` | array | Array of tag strings (optional) |
| `default_field_metadata` | object | Default metadata per locale (optional) |
| `upload_collection` | object | Collection assignment (optional) |

**Example**:
```javascript
// Basic upload (requires S3 handling)
const upload = await client.uploads.create({
  path: 's3://path/to/file.jpg',
  author: 'John Doe',
  copyright: '© 2024 My Company',
  default_field_metadata: {
    en: {
      alt: 'Product image',
      title: 'Main product view'
    }
  }
});
```

### update() - Update Upload

Update an existing upload's metadata.

**Signature**: `update(uploadId, body): Promise<Upload>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadId` | string | The upload ID (required) |
| `author` | string | Updated author (optional) |
| `copyright` | string | Updated copyright (optional) |
| `notes` | string | Updated notes (optional) |
| `tags` | array | Updated tags (optional) |
| `basename` | string | Rename file - keeps extension (optional) |
| `default_field_metadata` | object | Updated metadata (optional) |
| `upload_collection` | object | Move to collection (optional) |

**Example**:
```javascript
const updated = await client.uploads.update('upload-id-123', {
  author: 'Updated Author',
  copyright: '© 2024 Updated',
  notes: 'Updated notes',
  tags: ['updated', 'tags'],
  basename: 'new-filename', // Rename file (keeps extension)
  default_field_metadata: {
    en: {
      alt: 'Updated alt text',
      title: 'Updated title',
      custom_data: {
        reviewed: true,
        reviewer: 'Jane Smith'
      }
    }
  }
});

// Move to collection
const movedToCollection = await client.uploads.update('upload-id-123', {
  upload_collection: {
    type: 'upload_collection',
    id: 'new_collection_id'
  }
});

// Remove from collection
const removedFromCollection = await client.uploads.update('upload-id-123', {
  upload_collection: null
});
```

### destroy() - Delete Upload

Delete a single upload.

**Signature**: `destroy(uploadId): Promise<void>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadId` | string | The upload ID (required) |

**Example**:
```javascript
// Delete single upload
await client.uploads.destroy('upload-id-123');

// Delete with error handling
try {
  await client.uploads.destroy('upload-id-123');
  console.log('Upload deleted successfully');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Upload already deleted or not found');
  } else if (error.response?.status === 422) {
    console.error('Upload is in use and cannot be deleted');
  }
}

// Safe delete with usage check
async function safeDeleteUpload(uploadId) {
  const references = await client.uploads.references(uploadId);
  
  if (references.length > 0) {
    console.warn('Upload is in use by:', references);
    return false;
  }
  
  await client.uploads.destroy(uploadId);
  return true;
}
```

### references() - Get Upload References

Find all items using an upload.

**Signature**: `references(uploadId, queryParams): Promise<Item[]>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploadId` | string | The upload ID (required) |
| `page` | object | Pagination options |

**Example**:
```javascript
const references = await client.uploads.references('upload-id-123', {
  page: { limit: 100 }
});

references.forEach(item => {
  console.log(`Used in ${item.item_type} - ${item.id}`);
});
```

### Bulk Operations

#### bulkTag() - Bulk Tag Uploads

Add tags to multiple uploads at once.

**Signature**: `bulkTag(body): Promise<void>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploads` | array | Array of upload references (required) |
| `tags` | array | Tags to add (required) |

**Example**:
```javascript
await client.uploads.bulkTag({
  uploads: [
    { type: 'upload', id: '123' },
    { type: 'upload', id: '456' }
  ],
  tags: ['batch-processed', '2024']
});
```

#### bulkSetUploadCollection() - Bulk Move to Collection

Move multiple uploads to a collection.

**Signature**: `bulkSetUploadCollection(body): Promise<void>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploads` | array | Array of upload references (required) |
| `upload_collection` | object | Target collection (required) |

**Example**:
```javascript
await client.uploads.bulkSetUploadCollection({
  uploads: [
    { type: 'upload', id: '123' },
    { type: 'upload', id: '456' }
  ],
  upload_collection: {
    type: 'upload_collection',
    id: 'collection-123'
  }
});
```

#### bulkDestroy() - Bulk Delete Uploads

Delete multiple uploads asynchronously.

**Signature**: `bulkDestroy(body): Promise<JobResult>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `uploads` | array | Array of upload references (required) |

**Example**:
```javascript
const job = await client.uploads.bulkDestroy({
  uploads: [
    { type: 'upload', id: '123' },
    { type: 'upload', id: '456' }
  ]
});

// Wait for completion
await client.jobResults.wait(job.id);
```

## Progress Tracking

All upload methods support detailed progress tracking:

```javascript
// Node.js progress tracking
const upload = await client.uploads.createFromLocalFile({
  localPath: './large-video.mp4',
  onProgress: (progress) => {
    // Progress info varies by operation type
    if (progress.type === 'UPLOADING') {
      console.log(`Uploaded: ${progress.uploaded} bytes`);
      console.log(`Total: ${progress.total} bytes`);
      console.log(`Percentage: ${progress.percentage}%`);
    } else if (progress.type === 'CREATING_UPLOAD_OBJECT') {
      console.log('Creating upload record...');
    }
  }
});

// Browser progress with UI update
function createUploadWithProgress(file) {
  const progressBar = document.querySelector('#progress-bar');
  const statusText = document.querySelector('#status');
  
  return client.uploads.createFromFileOrBlob({
    fileOrBlob: file,
    onProgress: (progress) => {
      progressBar.value = progress.percentage;
      statusText.textContent = `${Math.round(progress.percentage)}% uploaded`;
      
      if (progress.percentage === 100) {
        statusText.textContent = 'Processing...';
      }
    }
  });
}
```

### Canceling Uploads

Upload operations return cancelable promises:

```javascript
import { makeCancelablePromise } from '@datocms/rest-client-utils';

const { promise, cancel } = makeCancelablePromise(
  client.uploads.createFromLocalFile({
    localPath: './large-file.zip'
  })
);

// Cancel button handler
cancelButton.onclick = () => {
  cancel();
  console.log('Upload canceled');
};

try {
  const upload = await promise;
  console.log('Upload completed:', upload);
} catch (error) {
  if (error.name === 'CanceledPromiseError') {
    console.log('Upload was canceled');
  }
}
```

## Image Transformations

DatoCMS provides on-the-fly image transformations via URL parameters:

### Basic Transformations

```javascript
const upload = await client.uploads.find('image-id');

// Resize
const thumbnail = upload.url + '?w=200&h=200&fit=crop';
const fitWidth = upload.url + '?w=800&fit=max';
const square = upload.url + '?w=500&h=500&fit=fill';

// Quality and format
const optimized = upload.url + '?q=80&fm=webp&auto=format';

// Device pixel ratio
const retina = upload.url + '?w=400&dpr=2';
```

### Filters and Effects

```javascript
// Blur
const blurred = upload.url + '?blur=20';

// Grayscale
const grayscale = upload.url + '?sat=-100';

// Brightness and contrast
const adjusted = upload.url + '?bri=10&con=20';

// Sepia
const sepia = upload.url + '?sepia=80';

// Sharpen
const sharpened = upload.url + '?sharp=25';

// Combined effects
const stylized = upload.url + '?w=800&sat=-50&con=30&blur=2';
```

### Advanced Filters

```javascript
// Duotone effect
const duotone = upload.url + '?duotone=000080,FFA500';

// Color overlay
const overlay = upload.url + '?blend=FF5722&blend-mode=overlay&blend-alpha=20';

// Monochrome with color cast
const blueMonochrome = upload.url + '?monochrome=002FA7';
```

### Common Transformation Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `w` | Width in pixels | `?w=400` |
| `h` | Height in pixels | `?h=300` |
| `fit` | Resize mode: crop, max, fill | `?fit=crop` |
| `dpr` | Device pixel ratio | `?dpr=2` |
| `q` | JPEG quality (0-100) | `?q=80` |
| `fm` | Output format: jpg, png, webp | `?fm=webp` |
| `auto` | Auto optimization | `?auto=format` |
| `blur` | Blur radius | `?blur=10` |
| `sat` | Saturation (-100 to 100) | `?sat=-100` |
| `bri` | Brightness (-100 to 100) | `?bri=20` |
| `con` | Contrast (-100 to 100) | `?con=30` |
| `sharp` | Sharpen amount | `?sharp=20` |
| `sepia` | Sepia effect (0-100) | `?sepia=80` |

## Error Handling

### Common Errors

```javascript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.uploads.createFromLocalFile({
    localPath: './invalid-path.jpg'
  });
} catch (error) {
  if (error instanceof ApiError) {
    switch (error.response.status) {
      case 413:
        console.error('File too large');
        break;
      case 422:
        console.error('Validation error:', error.errors);
        // Common: unsupported format, invalid metadata
        break;
      case 402:
        console.error('Upload limit reached');
        break;
      case 401:
        console.error('No upload permission');
        break;
    }
  } else if (error.code === 'ENOENT') {
    console.error('File not found');
  }
}
```

### Error Reference

| Error Code | Description | Solution |
|------------|-------------|----------|
| 401 | Unauthorized | Check API token permissions |
| 402 | Payment Required | Upgrade plan or delete old files |
| 413 | Payload Too Large | Reduce file size, check plan limits |
| 422 | Unprocessable Entity | Check file format and parameters |
| 408 | Request Timeout | Try smaller file or better connection |

### File Size Limits

| Plan | Max File Size | Total Storage |
|------|---------------|---------------|
| Free | 10 MB | 100 MB |
| Basic | 250 MB | 10 GB |
| Pro | 500 MB | 100 GB |
| Enterprise | 1 GB+ | Custom |

## Upload Deduplication

Prevent duplicate uploads using MD5 hash checking (Node.js only):

```javascript
// First upload
const upload1 = await client.uploads.createFromLocalFile({
  localPath: './image.jpg',
  skipCreationIfAlreadyExists: true
});

// Second upload of same file - returns existing upload
const upload2 = await client.uploads.createFromLocalFile({
  localPath: './image.jpg',
  skipCreationIfAlreadyExists: true
});

console.log(upload1.id === upload2.id); // true
```

## Working with Collections

Organize uploads using collections:

```javascript
// Create a collection
const collection = await client.uploadCollections.create({
  label: 'Product Images'
});

// Upload directly to collection
const upload = await client.uploads.createFromLocalFile({
  localPath: './product.jpg',
  upload_collection: {
    type: 'upload_collection',
    id: collection.id
  }
});

// Move existing upload to collection
await client.uploads.update(upload.id, {
  upload_collection: {
    type: 'upload_collection',
    id: collection.id
  }
});

// Query uploads in collection
const collectionUploads = await client.uploads.list({
  filter: {
    upload_collection: collection.id
  }
});
```

## Advanced Patterns

### Bulk Upload Processing

```javascript
import fs from 'fs';
import path from 'path';

async function bulkUploadFromDirectory(dirPath, options = {}) {
  const files = fs.readdirSync(dirPath);
  const results = {
    successful: [],
    failed: []
  };
  
  for (const file of files) {
    const filePath = path.join(dirPath, file);
    const stats = fs.statSync(filePath);
    
    if (!stats.isFile()) continue;
    
    try {
      const upload = await client.uploads.createFromLocalFile({
        localPath: filePath,
        author: options.author || 'Bulk Upload',
        copyright: options.copyright || '',
        upload_collection: options.collectionId ? {
          type: 'upload_collection',
          id: options.collectionId
        } : null,
        default_field_metadata: {
          en: {
            alt: path.basename(file, path.extname(file)),
            title: file
          }
        }
      });
      
      results.successful.push({
        file,
        uploadId: upload.id,
        url: upload.url
      });
    } catch (error) {
      results.failed.push({
        file,
        error: error.message
      });
    }
  }
  
  return results;
}
```

### Upload Usage Analysis

```javascript
async function analyzeUploadUsage() {
  const allUploads = await client.uploads.list({
    page: { limit: 500 }
  });
  
  const analysis = {
    total: allUploads.length,
    byType: {
      image: 0,
      video: 0,
      file: 0
    },
    byUsage: {
      used: 0,
      unused: 0
    },
    totalSize: 0,
    averageSize: 0,
    largestFile: null,
    oldestUnused: null,
    formatDistribution: new Map(),
    recommendations: []
  };
  
  for (const upload of allUploads) {
    // Type distribution
    analysis.byType[upload.type]++;
    
    // Usage distribution
    if (upload.in_use) {
      analysis.byUsage.used++;
    } else {
      analysis.byUsage.unused++;
    }
    
    // Size analysis
    analysis.totalSize += upload.size;
    
    if (!analysis.largestFile || upload.size > analysis.largestFile.size) {
      analysis.largestFile = upload;
    }
    
    // Format distribution
    const count = analysis.formatDistribution.get(upload.format) || 0;
    analysis.formatDistribution.set(upload.format, count + 1);
    
    // Find oldest unused
    if (!upload.in_use) {
      if (!analysis.oldestUnused || 
          new Date(upload.created_at) < new Date(analysis.oldestUnused.created_at)) {
        analysis.oldestUnused = upload;
      }
    }
  }
  
  analysis.averageSize = Math.round(analysis.totalSize / analysis.total);
  analysis.totalSize = Math.round(analysis.totalSize / 1000000); // Convert to MB
  
  // Generate recommendations
  if (analysis.byUsage.unused > analysis.total * 0.3) {
    analysis.recommendations.push({
      type: 'cleanup',
      message: `${analysis.byUsage.unused} unused uploads could be removed`,
      potentialSavings: Math.round(
        (analysis.byUsage.unused / analysis.total) * analysis.totalSize
      )
    });
  }
  
  return analysis;
}
```

### Smart Tagging

```javascript
async function autoTagUploads(rules) {
  const uploads = await client.uploads.list({
    filter: { tags: { eq: [] } }
  });
  
  const results = {
    processed: 0,
    tagged: 0,
    errors: []
  };
  
  for (const upload of uploads) {
    const tags = [];
    
    // Apply rules
    for (const rule of rules) {
      if (rule.condition(upload)) {
        tags.push(...rule.tags);
      }
    }
    
    if (tags.length > 0) {
      try {
        await client.uploads.update(upload.id, {
          tags: [...new Set(tags)] // Remove duplicates
        });
        results.tagged++;
      } catch (error) {
        results.errors.push({
          uploadId: upload.id,
          error: error.message
        });
      }
    }
    
    results.processed++;
  }
  
  return results;
}

// Usage with rules
const taggingRules = [
  {
    condition: (upload) => upload.type === 'image' && upload.width > 1920,
    tags: ['high-resolution', 'hd']
  },
  {
    condition: (upload) => upload.mime_type.includes('pdf'),
    tags: ['document', 'pdf']
  },
  {
    condition: (upload) => upload.size > 10000000,
    tags: ['large-file']
  },
  {
    condition: (upload) => upload.author?.includes('Marketing'),
    tags: ['marketing']
  }
];

const tagResults = await autoTagUploads(taggingRules);
```

### Upload Cleanup

```javascript
async function cleanupUnusedUploads(options = {}) {
  const cutoffDate = options.olderThan || 
    new Date(Date.now() - 90 * 24 * 60 * 60 * 1000); // 90 days default
  
  const unusedUploads = await client.uploads.list({
    filter: {
      in_use: false,
      created_at: { lt: cutoffDate.toISOString() }
    }
  });
  
  const cleanup = {
    found: unusedUploads.length,
    totalSize: 0,
    deleted: 0,
    failed: 0,
    errors: []
  };
  
  for (const upload of unusedUploads) {
    cleanup.totalSize += upload.size;
    
    if (!options.dryRun) {
      try {
        await client.uploads.destroy(upload.id);
        cleanup.deleted++;
      } catch (error) {
        cleanup.failed++;
        cleanup.errors.push({
          uploadId: upload.id,
          error: error.message
        });
      }
    }
  }
  
  cleanup.totalSize = Math.round(cleanup.totalSize / 1000000); // MB
  
  return cleanup;
}

// Usage
const cleanupResults = await cleanupUnusedUploads({
  olderThan: new Date('2023-01-01'),
  dryRun: true // Set to false to actually delete
});
```

## Upload Object Structure

```typescript
interface Upload {
  id: string;
  url: string;
  filename: string;
  basename: string;
  format: string;
  size: number;
  width?: number;  // For images/videos
  height?: number; // For images/videos
  duration?: number; // For videos/audio (seconds)
  frame_rate?: number; // For videos
  blurhash?: string; // For images
  thumbhash?: string; // For images
  mime_type: string;
  default_field_metadata: {
    [locale: string]: {
      alt?: string;
      title?: string;
      custom_data?: Record<string, any>;
    };
  };
  tags: string[];
  smart_tags?: string[]; // AI-generated tags
  author?: string;
  copyright?: string;
  notes?: string;
  colors?: Array<{ hex: string; alpha: number }>;
  is_image: boolean;
  in_use: boolean;
  created_at: string;
  updated_at: string;
  creator?: {
    type: 'account' | 'user' | 'sso_user' | 'access_token';
    id: string;
  };
  upload_collection?: {
    type: 'upload_collection';
    id: string;
  };
}
```

## Best Practices

### 1. Optimize Before Upload

```javascript
// Check file size before upload
const stats = await fs.promises.stat(filePath);
if (stats.size > 10 * 1024 * 1024) { // 10MB
  console.warn('Consider optimizing this image before upload');
}
```

### 2. Always Set Alt Text

```javascript
// Good - accessible
const upload = await client.uploads.createFromLocalFile({
  localPath: './image.jpg',
  default_field_metadata: {
    en: { 
      alt: 'Red bicycle leaning against brick wall',
      title: 'Urban transportation'
    }
  }
});

// Bad - missing alt text
const upload = await client.uploads.createFromLocalFile({
  localPath: './image.jpg'
});
```

### 3. Use Appropriate Formats

```javascript
// Use WebP for web images
const webOptimized = upload.url + '?fm=webp&auto=format';

// Use JPEG for photos
const photo = upload.url + '?fm=jpg&q=85';

// Use PNG for graphics with transparency
const graphic = upload.url + '?fm=png';
```

### 4. Handle Errors Gracefully

```javascript
async function safeUpload(filePath, metadata) {
  try {
    return await client.uploads.createFromLocalFile({
      localPath: filePath,
      skipCreationIfAlreadyExists: true,
      ...metadata
    });
  } catch (error) {
    if (error.response?.status === 413) {
      // Try to compress and retry
      const compressedPath = await compressImage(filePath);
      return client.uploads.createFromLocalFile({
        localPath: compressedPath,
        ...metadata
      });
    }
    throw error;
  }
}
```

### 5. Batch Operations

```javascript
// Good - batch processing
const uploadIds = ['123', '456', '789'];
await client.uploads.bulkTag({
  uploads: uploadIds.map(id => ({ type: 'upload', id })),
  tags: ['processed', '2024']
});

// Less efficient - individual updates
for (const id of uploadIds) {
  await client.uploads.update(id, {
    tags: ['processed', '2024']
  });
}
```

### 6. Use Progress Tracking for Large Files

```javascript
function uploadWithUI(filePath) {
  const progressBar = createProgressBar();
  
  return client.uploads.createFromLocalFile({
    localPath: filePath,
    onProgress: (progress) => {
      progressBar.update(progress.percentage);
      
      if (progress.percentage === 100) {
        progressBar.setMessage('Processing...');
      }
    }
  });
}
```

### 7. Implement Retry Logic

```javascript
async function uploadWithRetry(filePath, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await client.uploads.createFromLocalFile({
        localPath: filePath
      });
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      // Wait before retry (exponential backoff)
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
}
```

## Migration Helpers

### Migrate from External Service

```javascript
async function migrateFromExternalService(externalAssets) {
  const migrationResults = {
    successful: [],
    failed: []
  };
  
  for (const asset of externalAssets) {
    try {
      const upload = await client.uploads.createFromUrl({
        url: asset.url,
        filename: asset.filename,
        skipCreationIfAlreadyExists: true,
        default_field_metadata: {
          en: {
            alt: asset.alt || '',
            title: asset.title || '',
            custom_data: {
              originalId: asset.id,
              migratedFrom: 'external-service',
              migratedAt: new Date().toISOString()
            }
          }
        },
        tags: ['migrated', asset.type]
      });
      
      migrationResults.successful.push({
        originalId: asset.id,
        datocmsId: upload.id,
        url: upload.url
      });
    } catch (error) {
      migrationResults.failed.push({
        originalId: asset.id,
        error: error.message
      });
    }
  }
  
  return migrationResults;
}
```

## Platform Comparison

### Node.js vs Browser Upload Features

| Feature | Node.js (`@datocms/cma-client-node`) | Browser (`@datocms/cma-client-browser`) |
|---------|--------------------------------------|------------------------------------------|
| Local file upload | ✅ Direct file path support | ❌ Must use File/Blob objects |
| Stream support | ✅ Automatic streaming for large files | ✅ Uses browser streams API |
| Progress tracking | ✅ Detailed progress with multiple stages | ✅ Upload progress percentage |
| URL uploads | ✅ Native `createFromUrl` method | ✅ Requires manual implementation |
| Large files | ✅ Optimized for very large files | ⚠️ Limited by browser memory |
| File system access | ✅ Full access to local filesystem | ❌ Limited to user-selected files |
| Batch processing | ✅ Efficient server-side processing | ⚠️ Subject to browser limitations |
| MD5 deduplication | ✅ `skipCreationIfAlreadyExists` | ✅ Supported |
| Cancelable uploads | ✅ Built-in promise cancellation | ✅ Via AbortController |
| Retry mechanisms | ✅ Easy to implement with full control | ✅ Can implement with limitations |
| Memory efficiency | ✅ Streams prevent memory issues | ⚠️ Large files may cause issues |
| CORS restrictions | ❌ Not applicable | ✅ Handled by DatoCMS automatically |

### When to Use Each Platform

**Use Node.js Client When:**
- Building server-side applications or APIs
- Processing large batches of files
- Implementing automated upload workflows
- Need direct filesystem access
- Working with very large files (>100MB)
- Building CLI tools or scripts
- Implementing server-side image processing

**Use Browser Client When:**
- Building web applications with user uploads
- Implementing drag-and-drop interfaces
- Need real-time file preview
- Working with canvas or generated content
- Building single-page applications
- Implementing direct user file selection
- Creating interactive upload experiences

## Related Resources

- [Upload Collections](./upload-collection.md) - Organize uploads
- [Upload Tags](./upload-tag.md) - Tag management
- [Upload Smart Tags](./upload-smart-tag.md) - AI-powered tagging
- [Upload Tracks](./upload-track.md) - Video/audio tracks
- [Items](../01-content-management/item.md) - Using uploads in content