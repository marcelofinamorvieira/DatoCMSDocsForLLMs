# JobResult

The JobResult resource allows you to retrieve the results of asynchronous operations in DatoCMS. When you perform bulk operations or long-running tasks, the API returns a job ID immediately, allowing you to check the status and retrieve results asynchronously.

## Available Operations

### Find a Job Result

```javascript
const jobResult = await client.jobResults.find(jobId);
```

**Parameters:**
- `jobId` (string, required): The ID of the job result to retrieve

**Returns:** A JobResult object containing the operation status and payload

## The JobResult Object

```typescript
{
  id: string;                    // The job result ID
  type: 'job_result';           // Always 'job_result'
  status: number;               // HTTP status code (200 = success)
  payload: null | {             // Operation results (varies by job type)
    [k: string]: unknown;
  };
}
```

## Common Use Cases

### 1. Bulk Operations

When performing bulk creates, updates, or deletes, the API returns a job:

```javascript
// Initiate bulk operation
const job = await client.items.bulkCreate({
  items: [
    { item_type: { type: 'item_type', id: '123' }, title: 'Item 1' },
    { item_type: { type: 'item_type', id: '123' }, title: 'Item 2' }
  ]
});

// The client automatically polls for results, but you can check manually:
const result = await client.jobResults.find(job.id);

if (result.status === 200) {
  console.log('Created items:', result.payload);
} else {
  console.error('Operation failed:', result.payload);
}
```

### 2. Import/Export Operations

```javascript
// Start an export
const exportJob = await client.items.export({
  filter: { type: 'article' }
});

// Check export status
const exportResult = await client.jobResults.find(exportJob.id);

if (exportResult.status === 200) {
  const downloadUrl = exportResult.payload.download_url;
  // Download the exported data
}
```

### 3. Schema Migrations

```javascript
// Apply schema changes
const migrationJob = await client.itemTypes.bulkUpdate({
  // Migration configuration
});

// Monitor migration progress
const migrationResult = await client.jobResults.find(migrationJob.id);
```

## Understanding Status Codes

The `status` field contains standard HTTP status codes:

- **200**: Operation completed successfully
- **400**: Bad request - validation errors in payload
- **404**: Resource not found
- **422**: Unprocessable entity - business logic errors
- **500**: Internal server error

## Payload Structure

The `payload` structure varies by operation type:

### Bulk Create Success
```javascript
{
  status: 200,
  payload: {
    data: [
      { id: '123', type: 'item', attributes: { /* ... */ } },
      { id: '124', type: 'item', attributes: { /* ... */ } }
    ]
  }
}
```

### Validation Errors
```javascript
{
  status: 422,
  payload: {
    data: [
      {
        id: 'temp-id-1',
        type: 'api_error',
        attributes: {
          code: 'VALIDATION_FAILED',
          details: { field: ['is required'] }
        }
      }
    ]
  }
}
```

## Automatic Job Handling

The DatoCMS client automatically handles job polling for async operations:

```javascript
// This automatically waits for job completion:
const items = await client.items.bulkCreate({ /* ... */ });

// Behind the scenes, the client:
// 1. Initiates the bulk operation
// 2. Receives a job ID
// 3. Polls the job result
// 4. Returns the final result
```

## Advanced: Real-time Job Monitoring

For long-running operations, use event-based monitoring instead of polling:

```javascript
import { jobResultsFetcher } from '@datocms/cma-client';

// Set up real-time monitoring
const { data, error } = await jobResultsFetcher({
  apiToken: client.config.apiToken,
  jobId: job.id
});

if (data) {
  console.log('Job completed:', data);
} else {
  console.error('Job failed:', error);
}
```

## Important Notes

1. **Transient Data**: Job results are temporary and deleted after completion
2. **No List Method**: You cannot list all job results, only retrieve specific ones
3. **Large Payloads**: Very large results may require special handling
4. **Timeout**: Jobs have a maximum execution time, after which they fail

## Related Resources

- [Async Jobs Guide](/04-advanced-topics/async-jobs.md) - Deep dive into async operations
- [Batch Processing](/04-advanced-topics/batch-processing.md) - Bulk operation best practices
- [Error Handling](/error-handling-patterns.md) - Handling job failures

---

# Job Result Methods

This file documents all methods available for Job Result resources in the DatoCMS CMA Client. Job Results track the status and progress of asynchronous operations like imports, exports, and validations.

## Available Methods

### `client.jobResults.list()`

Lists all async job results for the current site. Returns paginated results with job status, progress, and metadata.

```typescript
import { buildClient, JobResult } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Basic usage - list all job results
async function listJobResults() {
  try {
    const jobResults = await client.jobResults.list();
    
    jobResults.forEach((job: JobResult) => {
      console.log(`Job ${job.id}:`);
      console.log(`  Type: ${job.type}`);
      console.log(`  Status: ${job.attributes.status}`);
      console.log(`  Progress: ${job.attributes.progress_percentage}%`);
      console.log(`  Created: ${job.attributes.created_at}`);
    });
  } catch (error) {
    console.error('Error listing job results:', error);
  }
}

// With pagination parameters
async function listJobResultsWithPagination() {
  try {
    const jobResults = await client.jobResults.list({
      page: {
        offset: 0,
        limit: 20
      }
    });
    
    console.log(`Total jobs: ${jobResults.meta.total_count}`);
    
    return jobResults;
  } catch (error) {
    console.error('Error listing job results:', error);
  }
}

// Filter by status
async function listPendingJobs() {
  try {
    const allJobs = await client.jobResults.list();
    
    const pendingJobs = allJobs.filter((job: JobResult) => 
      job.attributes.status === 'pending' || job.attributes.status === 'in_progress'
    );
    
    console.log(`Found ${pendingJobs.length} pending/in-progress jobs`);
    
    return pendingJobs;
  } catch (error) {
    console.error('Error filtering job results:', error);
  }
}
```

### `client.jobResults.find(jobResultId)`

Retrieves a specific job result by ID. Returns detailed information about the job including its current status, progress, and any result data.

```typescript
import { buildClient, JobResult } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Basic usage - find specific job result
async function findJobResult(jobId: string) {
  try {
    const job = await client.jobResults.find(jobId);
    
    console.log(`Job ${job.id} Details:`);
    console.log(`  Type: ${job.type}`);
    console.log(`  Status: ${job.attributes.status}`);
    console.log(`  Progress: ${job.attributes.progress_percentage}%`);
    console.log(`  Created at: ${job.attributes.created_at}`);
    
    if (job.attributes.status === 'success' && job.attributes.result) {
      console.log('  Result:', JSON.stringify(job.attributes.result, null, 2));
    }
    
    if (job.attributes.status === 'error' && job.attributes.error) {
      console.log('  Error:', job.attributes.error);
    }
    
    return job;
  } catch (error) {
    console.error('Error finding job result:', error);
  }
}

// Monitor job until completion
async function waitForJobCompletion(jobId: string, pollInterval = 2000) {
  try {
    let job = await client.jobResults.find(jobId);
    
    while (job.attributes.status === 'pending' || job.attributes.status === 'in_progress') {
      console.log(`Job ${jobId} progress: ${job.attributes.progress_percentage}%`);
      
      // Wait before polling again
      await new Promise(resolve => setTimeout(resolve, pollInterval));
      
      // Fetch updated job status
      job = await client.jobResults.find(jobId);
    }
    
    console.log(`Job ${jobId} completed with status: ${job.attributes.status}`);
    
    return job;
  } catch (error) {
    console.error('Error monitoring job:', error);
    throw error;
  }
}
```

## Response Structure

```typescript
// Job Result structure
interface JobResult {
  id: string;
  type: 'job_result';
  attributes: {
    status: 'pending' | 'in_progress' | 'success' | 'error';
    progress_percentage: number; // 0-100
    created_at: string; // ISO 8601 timestamp
    updated_at: string; // ISO 8601 timestamp
    result?: any; // Present when status is 'success'
    error?: {
      message: string;
      details?: any;
    }; // Present when status is 'error'
  };
}

// List response structure
interface JobResultsListResponse {
  data: JobResult[];
  meta: {
    total_count: number;
  };
}
```

## Job Types and Their Payloads

Different operations create different types of job results:

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Example: Content Import Job
async function monitorImportJob(jobId: string) {
  try {
    const job = await client.jobResults.find(jobId);
    
    if (job.attributes.status === 'success') {
      // Import job result contains imported records info
      const result = job.attributes.result;
      console.log(`Imported ${result.imported_records} records`);
      console.log(`Skipped ${result.skipped_records} records`);
      
      if (result.errors && result.errors.length > 0) {
        console.log('Import errors:', result.errors);
      }
    }
    
    return job;
  } catch (error) {
    console.error('Error monitoring import job:', error);
  }
}

// Example: Content Export Job
async function monitorExportJob(jobId: string) {
  try {
    const job = await client.jobResults.find(jobId);
    
    if (job.attributes.status === 'success') {
      // Export job result contains download URL
      const result = job.attributes.result;
      console.log(`Export completed. Download URL: ${result.download_url}`);
      console.log(`File expires at: ${result.expires_at}`);
    }
    
    return job;
  } catch (error) {
    console.error('Error monitoring export job:', error);
  }
}

// Example: Content Validation Job
async function monitorValidationJob(jobId: string) {
  try {
    const job = await client.jobResults.find(jobId);
    
    if (job.attributes.status === 'success') {
      // Validation job result contains validation errors
      const result = job.attributes.result;
      console.log(`Validation completed. Found ${result.errors.length} errors`);
      
      result.errors.forEach((error: any) => {
        console.log(`- Item ${error.item_id}: ${error.message}`);
      });
    }
    
    return job;
  } catch (error) {
    console.error('Error monitoring validation job:', error);
  }
}
```

## Progress Tracking Workflows

### Real-time Progress Updates

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Progress tracking with callbacks
async function trackJobProgress(
  jobId: string,
  onProgress?: (percentage: number) => void,
  onComplete?: (result: any) => void,
  onError?: (error: any) => void
) {
  const pollInterval = 1000; // 1 second
  let lastProgress = 0;
  
  try {
    let job = await client.jobResults.find(jobId);
    
    while (job.attributes.status === 'pending' || job.attributes.status === 'in_progress') {
      // Call progress callback if progress changed
      if (job.attributes.progress_percentage !== lastProgress) {
        lastProgress = job.attributes.progress_percentage;
        onProgress?.(lastProgress);
      }
      
      await new Promise(resolve => setTimeout(resolve, pollInterval));
      job = await client.jobResults.find(jobId);
    }
    
    // Handle completion
    if (job.attributes.status === 'success') {
      onComplete?.(job.attributes.result);
    } else if (job.attributes.status === 'error') {
      onError?.(job.attributes.error);
    }
    
    return job;
  } catch (error) {
    onError?.(error);
    throw error;
  }
}

// Usage example
async function importContentWithProgress() {
  try {
    // Start an import job (example)
    const importJob = await client.items.bulkImport({
      data: [/* your import data */]
    });
    
    // Track progress
    await trackJobProgress(
      importJob.id,
      (progress) => {
        console.log(`Import progress: ${progress}%`);
      },
      (result) => {
        console.log('Import completed successfully:', result);
      },
      (error) => {
        console.error('Import failed:', error);
      }
    );
  } catch (error) {
    console.error('Error during import:', error);
  }
}
```

### Batch Job Monitoring

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Monitor multiple jobs simultaneously
async function monitorMultipleJobs(jobIds: string[]) {
  const jobStatuses = new Map<string, string>();
  
  // Initialize status map
  jobIds.forEach(id => jobStatuses.set(id, 'pending'));
  
  try {
    while (Array.from(jobStatuses.values()).some(status => 
      status === 'pending' || status === 'in_progress'
    )) {
      // Check all jobs
      await Promise.all(jobIds.map(async (jobId) => {
        const job = await client.jobResults.find(jobId);
        const previousStatus = jobStatuses.get(jobId);
        
        if (job.attributes.status !== previousStatus) {
          console.log(`Job ${jobId} status changed: ${previousStatus} -> ${job.attributes.status}`);
          jobStatuses.set(jobId, job.attributes.status);
        }
      }));
      
      // Wait before next poll
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
    
    // Fetch final results
    const results = await Promise.all(
      jobIds.map(id => client.jobResults.find(id))
    );
    
    return results;
  } catch (error) {
    console.error('Error monitoring jobs:', error);
    throw error;
  }
}
```

## Error Handling Examples

```typescript
import { buildClient, ApiError } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Comprehensive error handling
async function handleJobErrors(jobId: string) {
  try {
    const job = await client.jobResults.find(jobId);
    
    if (job.attributes.status === 'error') {
      const error = job.attributes.error;
      
      // Handle different error types
      switch (error?.message) {
        case 'Import validation failed':
          console.error('Validation errors:', error.details);
          // Handle validation errors
          break;
          
        case 'Export timeout':
          console.error('Export took too long, please try with fewer records');
          // Handle timeout
          break;
          
        case 'Insufficient permissions':
          console.error('User lacks required permissions for this operation');
          // Handle permission errors
          break;
          
        default:
          console.error('Job failed with error:', error);
      }
      
      return { success: false, error };
    }
    
    return { success: true, result: job.attributes.result };
  } catch (error) {
    if (error instanceof ApiError) {
      console.error('API Error:', error.message);
      console.error('Status:', error.status);
      console.error('Request ID:', error.requestId);
    } else {
      console.error('Unexpected error:', error);
    }
    throw error;
  }
}

// Retry logic for failed jobs
async function retryFailedJob(originalJobId: string, retryOperation: () => Promise<any>) {
  try {
    const originalJob = await client.jobResults.find(originalJobId);
    
    if (originalJob.attributes.status !== 'error') {
      console.log('Job did not fail, no retry needed');
      return originalJob;
    }
    
    console.log('Retrying failed operation...');
    
    // Execute the retry operation
    const newJob = await retryOperation();
    
    // Monitor the new job
    return await waitForJobCompletion(newJob.id);
  } catch (error) {
    console.error('Retry failed:', error);
    throw error;
  }
}
```

## Best Practices for Async Operations

### 1. Implement Exponential Backoff

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

async function waitForJobWithBackoff(jobId: string, maxRetries = 10) {
  let retries = 0;
  let delay = 1000; // Start with 1 second
  
  try {
    while (retries < maxRetries) {
      const job = await client.jobResults.find(jobId);
      
      if (job.attributes.status === 'success' || job.attributes.status === 'error') {
        return job;
      }
      
      console.log(`Attempt ${retries + 1}: Job ${jobId} is ${job.attributes.status}`);
      
      // Exponential backoff with jitter
      const jitter = Math.random() * 1000;
      await new Promise(resolve => setTimeout(resolve, delay + jitter));
      
      delay = Math.min(delay * 2, 30000); // Max 30 seconds
      retries++;
    }
    
    throw new Error(`Job ${jobId} did not complete after ${maxRetries} attempts`);
  } catch (error) {
    console.error('Error waiting for job:', error);
    throw error;
  }
}
```

### 2. Resource Cleanup

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

class JobMonitor {
  private intervalId?: NodeJS.Timeout;
  
  async monitorJob(jobId: string, onUpdate: (job: any) => void) {
    try {
      this.intervalId = setInterval(async () => {
        try {
          const job = await client.jobResults.find(jobId);
          onUpdate(job);
          
          if (job.attributes.status === 'success' || job.attributes.status === 'error') {
            this.cleanup();
          }
        } catch (error) {
          console.error('Error in job monitor:', error);
          this.cleanup();
          throw error;
        }
      }, 2000);
    } catch (error) {
      this.cleanup();
      throw error;
    }
  }
  
  cleanup() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = undefined;
    }
  }
}
```

### 3. Job Queue Management

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

class JobQueue {
  private queue: string[] = [];
  private processing = false;
  private concurrency = 3;
  
  async addJob(jobId: string) {
    this.queue.push(jobId);
    if (!this.processing) {
      this.processQueue();
    }
  }
  
  private async processQueue() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      // Process jobs in batches
      const batch = this.queue.splice(0, this.concurrency);
      
      await Promise.all(batch.map(async (jobId) => {
        try {
          const result = await waitForJobCompletion(jobId);
          console.log(`Job ${jobId} completed with status: ${result.attributes.status}`);
        } catch (error) {
          console.error(`Job ${jobId} failed:`, error);
        }
      }));
    }
    
    this.processing = false;
  }
}
```

## Job Completion Callbacks

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Event-driven job monitoring
class JobEventEmitter extends EventTarget {
  async monitorJob(jobId: string) {
    let previousStatus = 'pending';
    
    const checkJob = async () => {
      try {
        const job = await client.jobResults.find(jobId);
        
        // Emit progress events
        if (job.attributes.status === 'in_progress') {
          this.dispatchEvent(new CustomEvent('progress', {
            detail: {
              jobId,
              progress: job.attributes.progress_percentage
            }
          }));
        }
        
        // Emit status change events
        if (job.attributes.status !== previousStatus) {
          previousStatus = job.attributes.status;
          this.dispatchEvent(new CustomEvent('statusChange', {
            detail: {
              jobId,
              status: job.attributes.status,
              job
            }
          }));
        }
        
        // Emit completion events
        if (job.attributes.status === 'success') {
          this.dispatchEvent(new CustomEvent('success', {
            detail: {
              jobId,
              result: job.attributes.result
            }
          }));
          return;
        } else if (job.attributes.status === 'error') {
          this.dispatchEvent(new CustomEvent('error', {
            detail: {
              jobId,
              error: job.attributes.error
            }
          }));
          return;
        }
        
        // Continue monitoring
        setTimeout(checkJob, 1000);
      } catch (error) {
        this.dispatchEvent(new CustomEvent('error', {
          detail: {
            jobId,
            error
          }
        }));
      }
    };
    
    checkJob();
  }
}

// Usage
const monitor = new JobEventEmitter();

monitor.addEventListener('progress', (event: any) => {
  console.log(`Progress: ${event.detail.progress}%`);
});

monitor.addEventListener('success', (event: any) => {
  console.log('Job completed successfully:', event.detail.result);
});

monitor.addEventListener('error', (event: any) => {
  console.error('Job failed:', event.detail.error);
});

monitor.monitorJob('job-id-123');
```

## Related Resources

- [Async Jobs Documentation](../04-advanced-topics/async-jobs.md) - Comprehensive guide to async job patterns
- [Batch Operations](../04-advanced-topics/batch-operations.md) - Bulk operations that create jobs
- [Progress Tracking](../04-advanced-topics/progress-tracking.md) - Advanced progress monitoring techniques
- [Error Handling Patterns](../04-advanced-topics/error-handling-patterns.md) - Error recovery strategies
- [REST API Events](../04-advanced-topics/rest-api-events.md) - Real-time job status via WebSocket

## Common Use Cases

1. **Import Operations** - Monitor CSV/JSON imports
2. **Export Operations** - Track data export progress
3. **Validation Jobs** - Check content validation results
4. **Migration Jobs** - Monitor schema migrations
5. **Bulk Updates** - Track mass content updates
6. **Media Processing** - Monitor asset processing jobs