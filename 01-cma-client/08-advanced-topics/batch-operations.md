# Batch Operations

This guide covers efficient strategies for performing bulk operations with DatoCMS APIs.

## Overview

When working with large datasets, batch operations are essential for:
- Improved performance through parallel processing
- Respecting API rate limits
- Reducing network overhead
- Maintaining data consistency

## Batch Creation

### Basic Batch Create

```typescript
async function batchCreateItems(items: any[], itemType: string) {
  const batchSize = 20; // Adjust based on rate limits
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    
    // Create items in parallel within batch
    const promises = batch.map(item => 
      client.items.create({
        itemType,
        ...item
      })
    );
    
    const batchResults = await Promise.allSettled(promises);
    results.push(...batchResults);
    
    // Add delay between batches to respect rate limits
    if (i + batchSize < items.length) {
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
    
    console.log(`Processed ${Math.min(i + batchSize, items.length)}/${items.length} items`);
  }
  
  return results;
}
```

### Advanced Batch Creation with Error Handling

```typescript
interface BatchResult<T> {
  successful: T[];
  failed: Array<{
    index: number;
    data: any;
    error: Error;
  }>;
  totalTime: number;
}

class BatchProcessor {
  constructor(
    private client: any,
    private batchSize: number = 20,
    private delayMs: number = 1000
  ) {}
  
  async createItems<T extends { id: string }>(
    items: any[],
    itemType: string
  ): Promise<BatchResult<T>> {
    const startTime = Date.now();
    const result: BatchResult<T> = {
      successful: [],
      failed: [],
      totalTime: 0
    };
    
    for (let i = 0; i < items.length; i += this.batchSize) {
      const batch = items.slice(i, i + this.batchSize);
      
      const promises = batch.map(async (item, batchIndex) => {
        const absoluteIndex = i + batchIndex;
        try {
          const created = await this.client.items.create({
            itemType,
            ...item
          });
          return { success: true, data: created, index: absoluteIndex };
        } catch (error) {
          return { 
            success: false, 
            error: error as Error, 
            data: item,
            index: absoluteIndex 
          };
        }
      });
      
      const results = await Promise.all(promises);
      
      results.forEach(r => {
        if (r.success) {
          result.successful.push(r.data);
        } else {
          result.failed.push({
            index: r.index,
            data: r.data,
            error: r.error
          });
        }
      });
      
      // Progress update
      const processed = Math.min(i + this.batchSize, items.length);
      console.log(`Progress: ${processed}/${items.length} (${Math.round(processed/items.length * 100)}%)`);
      
      // Rate limit delay
      if (i + this.batchSize < items.length) {
        await new Promise(r => setTimeout(r, this.delayMs));
      }
    }
    
    result.totalTime = Date.now() - startTime;
    return result;
  }
}

// Usage
const processor = new BatchProcessor(client, 20, 1000);
const result = await processor.createItems(articlesData, 'article');

console.log(`Created: ${result.successful.length} items`);
console.log(`Failed: ${result.failed.length} items`);
console.log(`Total time: ${result.totalTime}ms`);
```

## Batch Updates

### Efficient Batch Updates

```typescript
async function batchUpdateItems(
  updates: Array<{ id: string; data: any }>,
  options: { 
    batchSize?: number;
    preserveOrder?: boolean;
  } = {}
) {
  const { batchSize = 10, preserveOrder = false } = options;
  const results = [];
  
  if (preserveOrder) {
    // Sequential processing
    for (const update of updates) {
      try {
        const result = await client.items.update(update.id, update.data);
        results.push({ success: true, id: update.id, data: result });
      } catch (error) {
        results.push({ success: false, id: update.id, error });
      }
    }
  } else {
    // Parallel processing in batches
    for (let i = 0; i < updates.length; i += batchSize) {
      const batch = updates.slice(i, i + batchSize);
      
      const promises = batch.map(async update => {
        try {
          const result = await client.items.update(update.id, update.data);
          return { success: true, id: update.id, data: result };
        } catch (error) {
          return { success: false, id: update.id, error };
        }
      });
      
      const batchResults = await Promise.all(promises);
      results.push(...batchResults);
      
      if (i + batchSize < updates.length) {
        await new Promise(r => setTimeout(r, 1000));
      }
    }
  }
  
  return results;
}
```

### Conditional Batch Updates

```typescript
async function conditionalBatchUpdate(
  filter: any,
  updateFn: (item: any) => any,
  options: {
    dryRun?: boolean;
    maxItems?: number;
  } = {}
) {
  const { dryRun = false, maxItems = Infinity } = options;
  const updatedItems = [];
  let page = 1;
  let hasMore = true;
  let totalProcessed = 0;
  
  while (hasMore && totalProcessed < maxItems) {
    // Fetch items
    const response = await client.items.list({
      filter,
      page: { limit: 100, offset: (page - 1) * 100 }
    });
    
    const items = response.data;
    hasMore = items.length === 100;
    
    // Process items
    const updates = items
      .slice(0, maxItems - totalProcessed)
      .map(item => ({
        id: item.id,
        original: item,
        updated: updateFn(item)
      }))
      .filter(u => JSON.stringify(u.original) !== JSON.stringify(u.updated));
    
    if (updates.length > 0) {
      console.log(`Found ${updates.length} items to update on page ${page}`);
      
      if (!dryRun) {
        const results = await batchUpdateItems(
          updates.map(u => ({ id: u.id, data: u.updated }))
        );
        updatedItems.push(...results);
      } else {
        console.log('Dry run - would update:', updates);
        updatedItems.push(...updates);
      }
    }
    
    totalProcessed += items.length;
    page++;
  }
  
  return updatedItems;
}

// Usage: Add a prefix to all article titles
const results = await conditionalBatchUpdate(
  { type: 'article' },
  (item) => ({
    ...item,
    title: `[Updated] ${item.title}`
  }),
  { dryRun: true, maxItems: 1000 }
);
```

## Batch Deletion

### Safe Batch Delete

```typescript
interface DeleteOptions {
  confirmFn?: (items: any[]) => Promise<boolean>;
  backupFn?: (items: any[]) => Promise<void>;
  batchSize?: number;
}

async function batchDeleteItems(
  itemIds: string[],
  options: DeleteOptions = {}
) {
  const { 
    confirmFn, 
    backupFn, 
    batchSize = 10 
  } = options;
  
  // Fetch items to be deleted
  const itemsToDelete = await Promise.all(
    itemIds.map(id => client.items.find(id).catch(() => null))
  ).then(items => items.filter(Boolean));
  
  console.log(`Found ${itemsToDelete.length} items to delete`);
  
  // Confirmation step
  if (confirmFn) {
    const confirmed = await confirmFn(itemsToDelete);
    if (!confirmed) {
      console.log('Deletion cancelled');
      return { deleted: [], cancelled: true };
    }
  }
  
  // Backup step
  if (backupFn) {
    console.log('Creating backup...');
    await backupFn(itemsToDelete);
  }
  
  // Delete in batches
  const deleted = [];
  const failed = [];
  
  for (let i = 0; i < itemIds.length; i += batchSize) {
    const batch = itemIds.slice(i, i + batchSize);
    
    const promises = batch.map(async id => {
      try {
        await client.items.destroy(id);
        return { success: true, id };
      } catch (error) {
        return { success: false, id, error };
      }
    });
    
    const results = await Promise.all(promises);
    
    results.forEach(r => {
      if (r.success) {
        deleted.push(r.id);
      } else {
        failed.push({ id: r.id, error: r.error });
      }
    });
    
    console.log(`Deleted ${deleted.length}/${itemIds.length} items`);
    
    if (i + batchSize < itemIds.length) {
      await new Promise(r => setTimeout(r, 1000));
    }
  }
  
  return { deleted, failed };
}

// Usage with confirmation and backup
const result = await batchDeleteItems(
  itemIdsToDelete,
  {
    confirmFn: async (items) => {
      console.log(`About to delete ${items.length} items`);
      return true; // In real app, prompt user
    },
    backupFn: async (items) => {
      // Save to file or external storage
      await fs.writeFile(
        `backup-${Date.now()}.json`,
        JSON.stringify(items, null, 2)
      );
    }
  }
);
```

## Batch Media Operations

### Batch Upload Processing

```typescript
interface UploadTask {
  path: string;
  defaultFieldMetadata?: any;
  skipCreationIfAlreadyExists?: boolean;
}

class BatchUploadProcessor {
  private uploadQueue: UploadTask[] = [];
  private results = {
    successful: [] as any[],
    failed: [] as any[],
    skipped: [] as any[]
  };
  
  constructor(
    private client: any,
    private concurrency: number = 3
  ) {}
  
  addUpload(task: UploadTask): void {
    this.uploadQueue.push(task);
  }
  
  async processAll(): Promise<typeof this.results> {
    const chunks = [];
    
    // Split into chunks for concurrent processing
    for (let i = 0; i < this.uploadQueue.length; i += this.concurrency) {
      chunks.push(this.uploadQueue.slice(i, i + this.concurrency));
    }
    
    for (const [index, chunk] of chunks.entries()) {
      console.log(`Processing chunk ${index + 1}/${chunks.length}`);
      
      const promises = chunk.map(async task => {
        try {
          // Check if already exists
          if (task.skipCreationIfAlreadyExists) {
            const existing = await this.findExistingUpload(task.path);
            if (existing) {
              this.results.skipped.push({ ...task, existing });
              return;
            }
          }
          
          // Create upload
          const upload = await this.client.uploads.createFromLocalFile({
            localPath: task.path,
            defaultFieldMetadata: task.defaultFieldMetadata,
            filename: path.basename(task.path)
          });
          
          this.results.successful.push(upload);
        } catch (error) {
          this.results.failed.push({
            task,
            error: error.message
          });
        }
      });
      
      await Promise.all(promises);
      
      // Rate limit between chunks
      if (index < chunks.length - 1) {
        await new Promise(r => setTimeout(r, 2000));
      }
    }
    
    return this.results;
  }
  
  private async findExistingUpload(localPath: string): Promise<any> {
    const filename = path.basename(localPath);
    const uploads = await this.client.uploads.list({
      filter: { fields: { filename: { eq: filename } } }
    });
    return uploads.data[0];
  }
}

// Usage
const uploader = new BatchUploadProcessor(client, 3);

// Add files to process
const imageFiles = await glob('assets/images/*.{jpg,png}');
imageFiles.forEach(file => {
  uploader.addUpload({
    path: file,
    defaultFieldMetadata: {
      en: {
        alt: 'Image description',
        title: path.basename(file, path.extname(file))
      }
    },
    skipCreationIfAlreadyExists: true
  });
});

const results = await uploader.processAll();
console.log(`Uploaded: ${results.successful.length}`);
console.log(`Skipped: ${results.skipped.length}`);
console.log(`Failed: ${results.failed.length}`);
```

## Performance Optimization

### Parallel Processing with Concurrency Control

```typescript
class ConcurrencyManager {
  private running = 0;
  private queue: Array<() => Promise<any>> = [];
  
  constructor(private maxConcurrent: number) {}
  
  async add<T>(task: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    this.running++;
    
    try {
      const result = await task();
      return result;
    } finally {
      this.running--;
    }
  }
  
  async drain(): Promise<void> {
    while (this.running > 0) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }
}

// Usage
const manager = new ConcurrencyManager(5);
const results = [];

for (const item of largeDatset) {
  results.push(
    manager.add(async () => {
      return await processItem(item);
    })
  );
}

await Promise.all(results);
await manager.drain();
```

### Memory-Efficient Streaming

```typescript
async function* streamItems(filter: any, pageSize: number = 100) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const response = await client.items.list({
      filter,
      page: { limit: pageSize, offset: (page - 1) * pageSize }
    });
    
    const items = response.data;
    hasMore = items.length === pageSize;
    
    for (const item of items) {
      yield item;
    }
    
    page++;
  }
}

// Process items without loading all into memory
async function processLargeDataset() {
  let processed = 0;
  
  for await (const item of streamItems({ type: 'article' })) {
    await processItem(item);
    processed++;
    
    if (processed % 100 === 0) {
      console.log(`Processed ${processed} items`);
    }
  }
}
```

## Best Practices

1. **Respect Rate Limits**: Always add delays between batches
2. **Handle Errors Gracefully**: Use Promise.allSettled() instead of Promise.all()
3. **Provide Progress Feedback**: Log progress for long-running operations
4. **Implement Retry Logic**: Retry failed operations with exponential backoff
5. **Use Transactions When Possible**: Group related operations
6. **Monitor Memory Usage**: Stream large datasets instead of loading all at once
7. **Create Backups**: Before bulk deletions or updates
8. **Test with Small Batches**: Validate logic before processing entire dataset

## Related Resources

- [Error Handling Patterns](./error-handling-patterns.md)
- [Async Jobs](./async-jobs.md)
- [Handle Large Datasets](./handle-large-datasets.md)
- [Progress Tracking](./progress-tracking.md)
- [Rate Limits and Quotas](../05-reference/limits-and-quotas.md)