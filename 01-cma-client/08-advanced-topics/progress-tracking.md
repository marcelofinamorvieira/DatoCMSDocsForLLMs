# Track Progress During Batch Operations

Monitor the progress of batch operations in real-time to provide user feedback and handle long-running tasks effectively.

## Basic Progress Tracking

```typescript
import { buildClient } from '@datocms/cma-client-node';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create items with progress tracking
async function createItemsWithProgress(
  items: Array<{ title: string; content: string }>,
  onProgress?: (completed: number, total: number) => void
) {
  const results = [];
  const total = items.length;
  
  for (let i = 0; i < items.length; i++) {
    const item = items[i];
    
    // Create individual item
    const created = await client.items.create({
      item_type: { type: 'item_type', id: '123' },
      title: item.title,
      content: item.content
    });
    
    results.push(created);
    
    // Report progress
    if (onProgress) {
      onProgress(i + 1, total);
    }
  }
  
  return results;
}

// Usage with console progress
await createItemsWithProgress(
  [
    { title: 'Item 1', content: 'Content 1' },
    { title: 'Item 2', content: 'Content 2' },
    { title: 'Item 3', content: 'Content 3' }
  ],
  (completed, total) => {
    console.log(`Progress: ${completed}/${total} (${Math.round(completed/total * 100)}%)`);
  }
);
```

## Variations

### Batch API with Job Progress

```typescript
// Monitor batch operation job progress
async function batchUpdateWithProgress(updates: any[]) {
  // Start batch operation
  const job = await client.items.batchUpdate({
    operations: updates
  });
  
  // Poll for progress
  let currentJob = job;
  while (currentJob.attributes.status === 'processing') {
    await new Promise(resolve => setTimeout(resolve, 1000)); // Wait 1 second
    
    currentJob = await client.job.find(job.id);
    
    const { progress_percentage, processed_items, total_items } = currentJob.attributes;
    
    console.log(`Job ${job.id}: ${progress_percentage}% (${processed_items}/${total_items})`);
  }
  
  if (currentJob.attributes.status === 'completed') {
    console.log('Batch operation completed successfully');
    return currentJob.attributes.result;
  } else {
    throw new Error(`Job failed: ${currentJob.attributes.error}`);
  }
}
```

### Progress with Time Estimation

```typescript
interface ProgressTracker {
  startTime: number;
  processedCount: number;
  totalCount: number;
  
  getElapsedTime(): number;
  getEstimatedTimeRemaining(): number;
  getProgressPercentage(): number;
  getProcessingRate(): number;
}

class BatchProgressTracker implements ProgressTracker {
  startTime: number;
  processedCount: number = 0;
  totalCount: number;
  
  constructor(totalCount: number) {
    this.startTime = Date.now();
    this.totalCount = totalCount;
  }
  
  increment() {
    this.processedCount++;
  }
  
  getElapsedTime(): number {
    return Date.now() - this.startTime;
  }
  
  getEstimatedTimeRemaining(): number {
    if (this.processedCount === 0) return 0;
    
    const timePerItem = this.getElapsedTime() / this.processedCount;
    const remainingItems = this.totalCount - this.processedCount;
    return timePerItem * remainingItems;
  }
  
  getProgressPercentage(): number {
    return Math.round((this.processedCount / this.totalCount) * 100);
  }
  
  getProcessingRate(): number {
    const elapsedSeconds = this.getElapsedTime() / 1000;
    return elapsedSeconds > 0 ? this.processedCount / elapsedSeconds : 0;
  }
  
  getStatus(): string {
    const percentage = this.getProgressPercentage();
    const rate = this.getProcessingRate().toFixed(2);
    const remaining = Math.ceil(this.getEstimatedTimeRemaining() / 1000);
    
    return `${percentage}% (${this.processedCount}/${this.totalCount}) - ${rate} items/s - ETA: ${remaining}s`;
  }
}

// Usage
async function processItemsWithDetailedProgress(items: any[]) {
  const tracker = new BatchProgressTracker(items.length);
  
  for (const item of items) {
    await processItem(item);
    tracker.increment();
    console.log(tracker.getStatus());
  }
}
```

### Chunked Processing with Progress

```typescript
async function processInChunksWithProgress<T, R>(
  items: T[],
  chunkSize: number,
  processor: (chunk: T[]) => Promise<R[]>,
  onProgress?: (processed: number, total: number, results: R[]) => void
): Promise<R[]> {
  const allResults: R[] = [];
  let processed = 0;
  
  // Process in chunks
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    
    try {
      const chunkResults = await processor(chunk);
      allResults.push(...chunkResults);
      processed += chunk.length;
      
      if (onProgress) {
        onProgress(processed, items.length, chunkResults);
      }
    } catch (error) {
      console.error(`Error processing chunk at index ${i}:`, error);
      // Continue with next chunk or throw based on requirements
    }
  }
  
  return allResults;
}

// Example usage
const results = await processInChunksWithProgress(
  largeDataset,
  50, // Process 50 items at a time
  async (chunk) => {
    // Process chunk
    return await client.items.batchCreate({ items: chunk });
  },
  (processed, total, chunkResults) => {
    console.log(`Processed ${processed}/${total} items`);
    console.log(`Last chunk created ${chunkResults.length} items`);
  }
);
```

## Error Handling

### Progress with Error Recovery

```typescript
interface BatchResult<T> {
  successful: T[];
  failed: Array<{ item: any; error: Error }>;
  progress: {
    completed: number;
    total: number;
    successCount: number;
    failureCount: number;
  };
}

async function batchProcessWithErrorTracking<T>(
  items: any[],
  processor: (item: any) => Promise<T>,
  options: {
    continueOnError?: boolean;
    maxRetries?: number;
    onProgress?: (result: BatchResult<T>) => void;
  } = {}
): Promise<BatchResult<T>> {
  const { continueOnError = true, maxRetries = 3, onProgress } = options;
  
  const result: BatchResult<T> = {
    successful: [],
    failed: [],
    progress: {
      completed: 0,
      total: items.length,
      successCount: 0,
      failureCount: 0
    }
  };
  
  for (const item of items) {
    let retries = 0;
    let lastError: Error | null = null;
    
    while (retries <= maxRetries) {
      try {
        const processed = await processor(item);
        result.successful.push(processed);
        result.progress.successCount++;
        break;
      } catch (error) {
        lastError = error as Error;
        retries++;
        
        if (retries > maxRetries) {
          result.failed.push({ item, error: lastError });
          result.progress.failureCount++;
          
          if (!continueOnError) {
            throw error;
          }
        } else {
          // Exponential backoff
          await new Promise(resolve => setTimeout(resolve, Math.pow(2, retries) * 1000));
        }
      }
    }
    
    result.progress.completed++;
    
    if (onProgress) {
      onProgress(result);
    }
  }
  
  return result;
}
```

### Progress UI Integration

```typescript
// For browser/plugin contexts
class ProgressUI {
  private progressBar: HTMLElement;
  private statusText: HTMLElement;
  
  constructor(container: HTMLElement) {
    this.progressBar = container.querySelector('.progress-bar')!;
    this.statusText = container.querySelector('.status-text')!;
  }
  
  update(completed: number, total: number, message?: string) {
    const percentage = Math.round((completed / total) * 100);
    
    this.progressBar.style.width = `${percentage}%`;
    this.statusText.textContent = message || `${completed} of ${total} items processed`;
    
    // Update ARIA attributes for accessibility
    this.progressBar.setAttribute('aria-valuenow', percentage.toString());
    this.progressBar.setAttribute('aria-valuemax', '100');
  }
  
  setError(message: string) {
    this.progressBar.classList.add('error');
    this.statusText.textContent = message;
  }
  
  setComplete() {
    this.progressBar.classList.add('complete');
    this.statusText.textContent = 'Operation completed successfully';
  }
}

// Usage in async operation
async function performBatchOperationWithUI(items: any[], ui: ProgressUI) {
  try {
    await batchProcessWithErrorTracking(
      items,
      async (item) => await processItem(item),
      {
        onProgress: (result) => {
          const { completed, total, failureCount } = result.progress;
          const message = failureCount > 0 
            ? `${completed}/${total} (${failureCount} errors)`
            : `${completed}/${total} processed`;
          
          ui.update(completed, total, message);
        }
      }
    );
    
    ui.setComplete();
  } catch (error) {
    ui.setError(`Operation failed: ${error.message}`);
  }
}
```

## Integration Patterns

### With Plugin Context

```typescript
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

// In a DatoCMS plugin
export default function BatchImporter({ ctx }: { ctx: RenderFieldExtensionCtx }) {
  const [progress, setProgress] = useState({ current: 0, total: 0, status: 'idle' });
  
  const handleBatchImport = async (data: any[]) => {
    setProgress({ current: 0, total: data.length, status: 'processing' });
    
    try {
      const results = await createItemsWithProgress(
        data,
        (completed, total) => {
          setProgress({ current: completed, total, status: 'processing' });
          
          // Update plugin size if needed
          if (completed % 10 === 0) {
            ctx.updateHeight();
          }
        }
      );
      
      setProgress(prev => ({ ...prev, status: 'completed' }));
      ctx.notice('Batch import completed successfully');
      
    } catch (error) {
      setProgress(prev => ({ ...prev, status: 'error' }));
      ctx.alert(`Import failed: ${error.message}`);
    }
  };
  
  return (
    <div>
      {progress.status === 'processing' && (
        <div className="progress-container">
          <div className="progress-bar" style={{ width: `${(progress.current / progress.total) * 100}%` }} />
          <span>{progress.current} / {progress.total}</span>
        </div>
      )}
      {/* Rest of UI */}
    </div>
  );
}
```

### With WebSocket Updates

```typescript
import { EventSource } from '@datocms/rest-api-events';

// Real-time progress updates via SSE
async function trackJobProgressRealtime(jobId: string, apiToken: string) {
  const eventSource = new EventSource({
    url: `https://site-api.datocms.com/job-results/${jobId}/events`,
    headers: {
      'Authorization': `Bearer ${apiToken}`,
    },
  });
  
  return new Promise((resolve, reject) => {
    eventSource.on('progress', (event) => {
      const data = JSON.parse(event.data);
      console.log(`Progress: ${data.progress_percentage}% (${data.processed_items}/${data.total_items})`);
    });
    
    eventSource.on('complete', (event) => {
      eventSource.close();
      resolve(JSON.parse(event.data));
    });
    
    eventSource.on('error', (error) => {
      eventSource.close();
      reject(error);
    });
    
    eventSource.connect();
  });
}
```

## Next Steps

- Learn about [Async Jobs](/06-advanced-operations/batch-processing/async-jobs.md) for server-side batch processing
- Implement [Error Handling](/backend-api/core/api-error-handling-complete.md) strategies
- Explore [Real-time Features](/06-advanced-operations/real-time-features/webhook-setup.md) for live updates