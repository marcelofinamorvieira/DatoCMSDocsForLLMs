# UploadRequest

The UploadRequest resource is the foundation of DatoCMS's secure file upload system. It provides pre-signed Amazon S3 URLs that enable direct, secure file uploads without exposing AWS credentials. This resource is typically used internally by higher-level upload methods but can be accessed directly for custom upload implementations.

## Available Operations

### Create Upload Request

```javascript
const uploadRequest = await client.uploadRequest.create({
  filename: "document.pdf"
});
```

**Parameters:**
- `filename` (string, optional): Suggested filename for the upload

**Returns:** UploadRequest object with S3 upload details

## The UploadRequest Object

```typescript
{
  id: string;                    // Unique identifier (becomes upload path)
  url: string;                   // Pre-signed S3 URL for upload
  request_headers: {             // Required headers for S3 upload
    [key: string]: string;
  }
}
```

## Upload Workflow

DatoCMS uses a three-phase secure upload process:

### Phase 1: Request Upload URL
```javascript
const uploadRequest = await client.uploadRequest.create({
  filename: "image.jpg"
});
```

### Phase 2: Upload to S3
```javascript
const response = await fetch(uploadRequest.url, {
  method: 'PUT',
  body: fileContent,
  headers: uploadRequest.request_headers
});
```

### Phase 3: Create Upload Record
```javascript
const upload = await client.uploads.create({
  path: uploadRequest.id,  // Use the ID from upload request
  // ... other metadata
});
```

## Common Use Cases

### 1. Basic File Upload Implementation

```javascript
async function uploadFile(client, filePath) {
  const fs = require('fs').promises;
  const path = require('path');
  
  try {
    // Step 1: Create upload request
    const uploadRequest = await client.uploadRequest.create({
      filename: path.basename(filePath)
    });
    
    // Step 2: Read file
    const fileContent = await fs.readFile(filePath);
    
    // Step 3: Upload to S3
    const uploadResponse = await fetch(uploadRequest.url, {
      method: 'PUT',
      body: fileContent,
      headers: {
        ...uploadRequest.request_headers,
        'Content-Type': 'application/octet-stream'
      }
    });
    
    if (!uploadResponse.ok) {
      throw new Error(`S3 upload failed: ${uploadResponse.statusText}`);
    }
    
    // Step 4: Create upload record in DatoCMS
    const upload = await client.uploads.create({
      path: uploadRequest.id,
      default_field_metadata: {
        en: {
          alt: null,
          title: path.basename(filePath)
        }
      }
    });
    
    return upload;
  } catch (error) {
    console.error('Upload failed:', error);
    throw error;
  }
}

// Usage
const upload = await uploadFile(client, './assets/logo.png');
console.log('Upload complete:', upload.url);
```

### 2. Browser Upload with File Input

```javascript
// React component for file uploads
function FileUploader({ client }) {
  const [uploading, setUploading] = useState(false);
  const [progress, setProgress] = useState(0);
  
  const handleFileSelect = async (event) => {
    const file = event.target.files[0];
    if (!file) return;
    
    setUploading(true);
    
    try {
      // Step 1: Request upload URL
      const uploadRequest = await client.uploadRequest.create({
        filename: file.name
      });
      
      // Step 2: Upload to S3 with progress
      const xhr = new XMLHttpRequest();
      
      xhr.upload.onprogress = (e) => {
        if (e.lengthComputable) {
          setProgress((e.loaded / e.total) * 100);
        }
      };
      
      const uploadPromise = new Promise((resolve, reject) => {
        xhr.onload = () => {
          if (xhr.status === 200) {
            resolve();
          } else {
            reject(new Error(`Upload failed: ${xhr.statusText}`));
          }
        };
        xhr.onerror = reject;
      });
      
      xhr.open('PUT', uploadRequest.url);
      
      // Add all required headers
      Object.entries(uploadRequest.request_headers).forEach(([key, value]) => {
        xhr.setRequestHeader(key, value);
      });
      
      xhr.send(file);
      await uploadPromise;
      
      // Step 3: Create upload record
      const upload = await client.uploads.create({
        path: uploadRequest.id,
        default_field_metadata: {
          en: {
            alt: file.name,
            title: file.name
          }
        }
      });
      
      console.log('Upload complete:', upload);
      
    } catch (error) {
      console.error('Upload failed:', error);
    } finally {
      setUploading(false);
      setProgress(0);
    }
  };
  
  return (
    <div>
      <input 
        type="file" 
        onChange={handleFileSelect}
        disabled={uploading}
      />
      {uploading && (
        <div>
          <progress value={progress} max="100" />
          <span>{Math.round(progress)}%</span>
        </div>
      )}
    </div>
  );
}
```

### 3. Batch Upload System

```javascript
class BatchUploader {
  constructor(client, concurrency = 3) {
    this.client = client;
    this.concurrency = concurrency;
    this.queue = [];
    this.active = 0;
    this.results = [];
  }
  
  async uploadFiles(files) {
    // Add all files to queue
    this.queue = files.map(file => ({
      file,
      status: 'pending'
    }));
    
    // Start processing
    const promises = [];
    for (let i = 0; i < this.concurrency; i++) {
      promises.push(this.processQueue());
    }
    
    await Promise.all(promises);
    return this.results;
  }
  
  async processQueue() {
    while (this.queue.length > 0) {
      const item = this.queue.find(i => i.status === 'pending');
      if (!item) break;
      
      item.status = 'processing';
      this.active++;
      
      try {
        const upload = await this.uploadSingleFile(item.file);
        item.status = 'completed';
        this.results.push({ file: item.file, upload, error: null });
      } catch (error) {
        item.status = 'failed';
        this.results.push({ file: item.file, upload: null, error });
      } finally {
        this.active--;
      }
    }
  }
  
  async uploadSingleFile(file) {
    // Step 1: Create upload request
    const uploadRequest = await this.client.uploadRequest.create({
      filename: file.name
    });
    
    // Step 2: Upload to S3
    const formData = new FormData();
    formData.append('file', file);
    
    const response = await fetch(uploadRequest.url, {
      method: 'PUT',
      body: file,
      headers: uploadRequest.request_headers
    });
    
    if (!response.ok) {
      throw new Error('S3 upload failed');
    }
    
    // Step 3: Create upload record
    return this.client.uploads.create({
      path: uploadRequest.id,
      default_field_metadata: {
        en: {
          alt: file.name,
          title: file.name
        }
      }
    });
  }
  
  getProgress() {
    const total = this.queue.length;
    const completed = this.queue.filter(i => i.status === 'completed').length;
    const failed = this.queue.filter(i => i.status === 'failed').length;
    
    return {
      total,
      completed,
      failed,
      processing: this.active,
      percentage: (completed / total) * 100
    };
  }
}

// Usage
const uploader = new BatchUploader(client, 5); // 5 concurrent uploads

const files = [
  new File(['content1'], 'file1.txt'),
  new File(['content2'], 'file2.txt'),
  // ... more files
];

const results = await uploader.uploadFiles(files);
console.log('Batch upload complete:', results);
```

### 4. Upload from URL with Progress

```javascript
async function uploadFromUrl(client, sourceUrl, filename) {
  const fetch = require('node-fetch');
  
  console.log(`Downloading ${sourceUrl}...`);
  
  // Download file
  const response = await fetch(sourceUrl);
  if (!response.ok) {
    throw new Error(`Failed to download: ${response.statusText}`);
  }
  
  const contentLength = response.headers.get('content-length');
  const buffer = await response.buffer();
  
  console.log(`Downloaded ${buffer.length} bytes`);
  
  // Step 1: Create upload request
  const uploadRequest = await client.uploadRequest.create({
    filename: filename || sourceUrl.split('/').pop()
  });
  
  // Step 2: Upload to S3
  console.log('Uploading to S3...');
  
  const uploadResponse = await fetch(uploadRequest.url, {
    method: 'PUT',
    body: buffer,
    headers: {
      ...uploadRequest.request_headers,
      'Content-Type': response.headers.get('content-type') || 'application/octet-stream',
      'Content-Length': buffer.length.toString()
    }
  });
  
  if (!uploadResponse.ok) {
    throw new Error(`S3 upload failed: ${uploadResponse.statusText}`);
  }
  
  // Step 3: Create upload record
  console.log('Creating upload record...');
  
  const upload = await client.uploads.create({
    path: uploadRequest.id,
    default_field_metadata: {
      en: {
        alt: null,
        title: filename || 'Downloaded file'
      }
    }
  });
  
  console.log('Upload complete:', upload.url);
  return upload;
}

// Usage
const upload = await uploadFromUrl(
  client,
  'https://example.com/image.jpg',
  'downloaded-image.jpg'
);
```

### 5. Streaming Upload

```javascript
// For large files, stream directly to S3
async function streamingUpload(client, readStream, filename, fileSize) {
  const { pipeline } = require('stream/promises');
  const fetch = require('node-fetch');
  
  // Step 1: Create upload request
  const uploadRequest = await client.uploadRequest.create({ filename });
  
  // Step 2: Stream to S3
  const response = await fetch(uploadRequest.url, {
    method: 'PUT',
    body: readStream,
    headers: {
      ...uploadRequest.request_headers,
      'Content-Length': fileSize.toString()
    }
  });
  
  if (!response.ok) {
    throw new Error(`S3 upload failed: ${response.statusText}`);
  }
  
  // Step 3: Create upload record
  return client.uploads.create({
    path: uploadRequest.id
  });
}

// Usage with file stream
const fs = require('fs');
const stats = await fs.promises.stat('large-file.zip');
const stream = fs.createReadStream('large-file.zip');

const upload = await streamingUpload(
  client,
  stream,
  'large-file.zip',
  stats.size
);
```

### 6. Custom Upload with Retry Logic

```javascript
class ResilientUploader {
  constructor(client, maxRetries = 3) {
    this.client = client;
    this.maxRetries = maxRetries;
  }
  
  async upload(file, metadata = {}) {
    let lastError;
    
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        console.log(`Upload attempt ${attempt}/${this.maxRetries}`);
        
        // Step 1: Create upload request
        const uploadRequest = await this.createUploadRequest(file.name);
        
        // Step 2: Upload to S3
        await this.uploadToS3(uploadRequest, file);
        
        // Step 3: Create upload record
        const upload = await this.createUploadRecord(
          uploadRequest.id,
          metadata
        );
        
        console.log('Upload successful');
        return upload;
        
      } catch (error) {
        lastError = error;
        console.error(`Attempt ${attempt} failed:`, error.message);
        
        if (attempt < this.maxRetries) {
          // Exponential backoff
          const delay = Math.min(1000 * Math.pow(2, attempt - 1), 10000);
          console.log(`Retrying in ${delay}ms...`);
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }
    }
    
    throw new Error(`Upload failed after ${this.maxRetries} attempts: ${lastError.message}`);
  }
  
  async createUploadRequest(filename) {
    return this.client.uploadRequest.create({ filename });
  }
  
  async uploadToS3(uploadRequest, file) {
    const response = await fetch(uploadRequest.url, {
      method: 'PUT',
      body: file,
      headers: uploadRequest.request_headers
    });
    
    if (!response.ok) {
      throw new Error(`S3 upload failed: ${response.statusText}`);
    }
  }
  
  async createUploadRecord(path, metadata) {
    return this.client.uploads.create({
      path,
      ...metadata
    });
  }
}

// Usage
const uploader = new ResilientUploader(client, 5);

try {
  const upload = await uploader.upload(file, {
    default_field_metadata: {
      en: { alt: 'Important document' }
    },
    tags: ['important', 'document']
  });
  
  console.log('Upload completed:', upload);
} catch (error) {
  console.error('Upload failed permanently:', error);
}
```

## Security Considerations

### Benefits
1. **No Credential Exposure**: AWS credentials never leave the server
2. **Time-Limited URLs**: Pre-signed URLs expire quickly
3. **Direct S3 Upload**: Files bypass DatoCMS servers
4. **Request Validation**: Each request is validated server-side

### Best Practices
1. **Use HTTPS**: Always use secure connections
2. **Validate Files**: Check file types and sizes before uploading
3. **Handle Errors**: Implement proper error handling
4. **Clean Up**: Remove failed uploads from your UI

## Important Notes

1. **One-Time Use**: Each upload request ID can only be used once
2. **Expiration**: Pre-signed URLs expire within minutes
3. **Required Headers**: All headers in `request_headers` must be included
4. **Path Parameter**: The `id` becomes the `path` when creating the upload

## When to Use UploadRequest

### Use Directly When:
- Building custom upload interfaces
- Implementing special progress tracking
- Working in non-standard environments
- Creating upload proxy services

### Use Higher-Level Methods When:
- Working with standard Node.js or browser environments
- Want automatic retry and error handling
- Need simplified API

## Error Handling

```javascript
try {
  const uploadRequest = await client.uploadRequest.create({
    filename: 'test.jpg'
  });
} catch (error) {
  if (error.response?.status === 401) {
    console.error('Authentication failed');
  } else if (error.response?.status === 402) {
    console.error('Upload quota exceeded');
  } else if (error.response?.status === 429) {
    console.error('Rate limit exceeded');
  } else {
    console.error('Unexpected error:', error);
  }
}
```

## Raw Response Access

```javascript
const response = await client.uploadRequest.rawCreate({
  filename: 'document.pdf'
});

console.log(response); // Raw response with S3 details
```

## Related Resources

- [Uploads](/01-cma-client/02-media-management/upload.md) - Upload management
- [Platform-Specific Uploads](/04-advanced-topics/platform-specific-uploads.md) - Node.js and browser methods
- [Upload Collections](/01-cma-client/02-media-management/upload-collection.md) - Organize uploads