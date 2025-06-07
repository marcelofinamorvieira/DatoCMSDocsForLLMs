# Handle Large Datasets

Efficiently process large amounts of data with pagination, batching, and parallel processing strategies.

## Basic Batch Processing

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Process all items of a model in batches
async function processLargeDataset(modelId, processor, options = {}) {
  const {
    batchSize = 100,
    concurrency = 5,
    onProgress = () => {},
    filter = {}
  } = options;
  
  let processed = 0;
  let total = 0;
  const errors = [];
  
  // First, get total count
  const countResponse = await client.items.list({
    filter: { type: modelId, ...filter },
    page: { limit: 1 }
  });
  total = countResponse.meta.total_count;
  
  // Process using cursor-based pagination
  let cursor = null;
  
  do {
    // Fetch batch
    const response = await client.items.list({
      filter: { type: modelId, ...filter },
      page: { 
        limit: batchSize,
        cursor: cursor
      }
    });
    
    const items = response.data;
    cursor = response.meta.next_cursor;
    
    // Process batch with controlled concurrency
    const chunks = [];
    for (let i = 0; i < items.length; i += concurrency) {
      chunks.push(items.slice(i, i + concurrency));
    }
    
    for (const chunk of chunks) {
      const results = await Promise.allSettled(
        chunk.map(item => processor(item))
      );
      
      results.forEach((result, index) => {
        processed++;
        
        if (result.status === 'rejected') {
          errors.push({
            item: chunk[index],
            error: result.reason
          });
        }
        
        onProgress({
          processed,
          total,
          percentage: Math.round((processed / total) * 100),
          currentItem: chunk[index],
          error: result.status === 'rejected' ? result.reason : null
        });
      });
    }
    
    // Rate limiting pause between batches
    await new Promise(resolve => setTimeout(resolve, 100));
    
  } while (cursor);
  
  return {
    processed,
    total,
    errors,
    success: errors.length === 0
  };
}
```

## Variations

### Memory-Efficient Streaming

```javascript
// Generator-based approach for memory efficiency
async function* streamItems(modelId, options = {}) {
  const { batchSize = 100, filter = {} } = options;
  let cursor = null;
  
  do {
    const response = await client.items.list({
      filter: { type: modelId, ...filter },
      page: { 
        limit: batchSize,
        cursor: cursor
      }
    });
    
    for (const item of response.data) {
      yield item;
    }
    
    cursor = response.meta.next_cursor;
    
    // Small pause to avoid rate limiting
    if (cursor) {
      await new Promise(resolve => setTimeout(resolve, 50));
    }
  } while (cursor);
}

// Usage with async iteration
async function processWithStream(modelId) {
  const processor = async (item) => {
    // Your processing logic here
    console.log(`Processing item ${item.id}`);
    return item;
  };
  
  let count = 0;
  
  for await (const item of streamItems(modelId)) {
    await processor(item);
    count++;
    
    if (count % 100 === 0) {
      console.log(`Processed ${count} items...`);
    }
  }
  
  return count;
}
```

### Parallel Batch Updates

```javascript
async function batchUpdateItems(updates, options = {}) {
  const {
    concurrency = 10,
    retries = 3,
    onProgress = () => {},
    validateBeforeUpdate = null
  } = options;
  
  const results = {
    successful: [],
    failed: [],
    total: updates.length
  };
  
  // Create a queue for controlled concurrency
  const queue = [...updates];
  const inProgress = new Set();
  
  async function processNext() {
    if (queue.length === 0 || inProgress.size >= concurrency) {
      return;
    }
    
    const update = queue.shift();
    const taskId = `${update.id}-${Date.now()}`;
    inProgress.add(taskId);
    
    try {
      // Optional validation
      if (validateBeforeUpdate) {
        const isValid = await validateBeforeUpdate(update);
        if (!isValid) {
          throw new Error('Validation failed');
        }
      }
      
      // Retry logic
      let lastError;
      for (let attempt = 1; attempt <= retries; attempt++) {
        try {
          const result = await client.items.update(update.id, update.data);
          results.successful.push({ id: update.id, result });
          
          onProgress({
            completed: results.successful.length + results.failed.length,
            total: results.total,
            success: true,
            item: result
          });
          
          break;
        } catch (error) {
          lastError = error;
          
          if (attempt < retries) {
            // Exponential backoff
            await new Promise(resolve => 
              setTimeout(resolve, Math.pow(2, attempt) * 1000)
            );
          }
        }
      }
      
      if (lastError) {
        throw lastError;
      }
      
    } catch (error) {
      results.failed.push({ 
        id: update.id, 
        error: error.message,
        data: update.data 
      });
      
      onProgress({
        completed: results.successful.length + results.failed.length,
        total: results.total,
        success: false,
        error: error.message,
        itemId: update.id
      });
    } finally {
      inProgress.delete(taskId);
      // Process next item
      processNext();
    }
  }
  
  // Start initial batch
  const initialBatch = Math.min(concurrency, queue.length);
  for (let i = 0; i < initialBatch; i++) {
    processNext();
  }
  
  // Wait for all to complete
  while (inProgress.size > 0 || queue.length > 0) {
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return results;
}
```

### Chunked File Processing

```javascript
// Process large CSV data in chunks
import { parse } from 'csv-parse';
import { createReadStream } from 'fs';

async function importLargeCSV(filePath, modelId, options = {}) {
  const {
    chunkSize = 100,
    mapping = {},
    skipHeader = true,
    onChunk = () => {},
    validateRow = null
  } = options;
  
  const stats = {
    total: 0,
    imported: 0,
    skipped: 0,
    errors: []
  };
  
  return new Promise((resolve, reject) => {
    const chunks = [];
    let currentChunk = [];
    let isFirstRow = true;
    
    const stream = createReadStream(filePath)
      .pipe(parse({
        columns: skipHeader,
        skip_empty_lines: true
      }));
    
    stream.on('data', async (row) => {
      if (isFirstRow && skipHeader) {
        isFirstRow = false;
        return;
      }
      
      stats.total++;
      
      // Validate row if validator provided
      if (validateRow && !validateRow(row)) {
        stats.skipped++;
        return;
      }
      
      // Map CSV columns to DatoCMS fields
      const itemData = {};
      for (const [csvColumn, datoCmsField] of Object.entries(mapping)) {
        if (row[csvColumn] !== undefined && row[csvColumn] !== '') {
          itemData[datoCmsField] = row[csvColumn];
        }
      }
      
      currentChunk.push(itemData);
      
      // Process chunk when it reaches the size limit
      if (currentChunk.length >= chunkSize) {
        stream.pause();
        
        try {
          const results = await createItemsBatch(modelId, currentChunk);
          stats.imported += results.created;
          
          onChunk({
            processed: stats.imported + stats.skipped + stats.errors.length,
            total: stats.total,
            chunkSize: currentChunk.length,
            results
          });
          
          currentChunk = [];
          stream.resume();
        } catch (error) {
          stats.errors.push({
            chunk: currentChunk,
            error: error.message
          });
          currentChunk = [];
          stream.resume();
        }
      }
    });
    
    stream.on('end', async () => {
      // Process remaining items
      if (currentChunk.length > 0) {
        try {
          const results = await createItemsBatch(modelId, currentChunk);
          stats.imported += results.created;
        } catch (error) {
          stats.errors.push({
            chunk: currentChunk,
            error: error.message
          });
        }
      }
      
      resolve(stats);
    });
    
    stream.on('error', reject);
  });
}

async function createItemsBatch(modelId, items) {
  const results = {
    created: 0,
    failed: 0,
    errors: []
  };
  
  // Create items in parallel with concurrency control
  const concurrency = 5;
  
  for (let i = 0; i < items.length; i += concurrency) {
    const batch = items.slice(i, i + concurrency);
    
    const batchResults = await Promise.allSettled(
      batch.map(itemData => 
        client.items.create({
          item_type: { type: 'item_type', id: modelId },
          ...itemData
        })
      )
    );
    
    batchResults.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        results.created++;
      } else {
        results.failed++;
        results.errors.push({
          data: batch[index],
          error: result.reason.message
        });
      }
    });
  }
  
  return results;
}
```

## Error Handling

```javascript
class BatchProcessingError extends Error {
  constructor(message, stats) {
    super(message);
    this.name = 'BatchProcessingError';
    this.stats = stats;
  }
}

async function robustBatchProcessor(items, processor, options = {}) {
  const {
    maxRetries = 3,
    onError = () => {},
    continueOnError = true,
    errorThreshold = 0.1 // Stop if error rate exceeds 10%
  } = options;
  
  const stats = {
    total: items.length,
    processed: 0,
    successful: 0,
    failed: 0,
    errors: []
  };
  
  for (const [index, item] of items.entries()) {
    let lastError;
    let attempts = 0;
    
    while (attempts < maxRetries) {
      try {
        attempts++;
        const result = await processor(item, index);
        stats.processed++;
        stats.successful++;
        break;
      } catch (error) {
        lastError = error;
        
        // Check if it's a retryable error
        if (isRetryableError(error)) {
          await new Promise(resolve => 
            setTimeout(resolve, Math.pow(2, attempts) * 1000)
          );
          continue;
        }
        
        // Non-retryable error, break retry loop
        break;
      }
    }
    
    if (lastError) {
      stats.failed++;
      stats.errors.push({
        item,
        error: lastError,
        attempts
      });
      
      onError({ item, error: lastError, stats });
      
      // Check error threshold
      const errorRate = stats.failed / stats.processed;
      if (errorRate > errorThreshold && !continueOnError) {
        throw new BatchProcessingError(
          `Error rate (${(errorRate * 100).toFixed(1)}%) exceeded threshold`,
          stats
        );
      }
    }
  }
  
  return stats;
}

function isRetryableError(error) {
  const retryableCodes = [
    'RATE_LIMIT_EXCEEDED',
    'TIMEOUT',
    'ECONNRESET',
    'ETIMEDOUT'
  ];
  
  return retryableCodes.includes(error.code) || 
         error.response?.status >= 500;
}
```

## Integration Patterns

### Job Queue Integration

```javascript
import Queue from 'bull';
import { Worker } from 'worker_threads';

const batchQueue = new Queue('batch-processing');

// Producer - Split large job into smaller jobs
async function enqueueLargeDataset(modelId, operation, options = {}) {
  const { chunkSize = 1000 } = options;
  
  // Get total count
  const response = await client.items.list({
    filter: { type: modelId },
    page: { limit: 1 }
  });
  
  const totalItems = response.meta.total_count;
  const totalChunks = Math.ceil(totalItems / chunkSize);
  
  // Create parent job
  const parentJob = await batchQueue.add('parent-job', {
    modelId,
    operation,
    totalItems,
    totalChunks,
    startedAt: new Date()
  });
  
  // Create child jobs for each chunk
  const childJobs = [];
  for (let i = 0; i < totalChunks; i++) {
    const job = await batchQueue.add('process-chunk', {
      parentJobId: parentJob.id,
      modelId,
      operation,
      offset: i * chunkSize,
      limit: chunkSize,
      chunkIndex: i,
      totalChunks
    }, {
      attempts: 3,
      backoff: {
        type: 'exponential',
        delay: 2000
      }
    });
    
    childJobs.push(job.id);
  }
  
  // Update parent job with child IDs
  await parentJob.update({
    ...parentJob.data,
    childJobs
  });
  
  return {
    parentJobId: parentJob.id,
    childJobs,
    totalChunks
  };
}

// Consumer - Process chunks
batchQueue.process('process-chunk', async (job) => {
  const { modelId, operation, offset, limit } = job.data;
  
  // Update progress
  job.progress(0);
  
  // Fetch items for this chunk
  const items = [];
  const response = await client.items.list({
    filter: { type: modelId },
    page: { offset, limit }
  });
  
  items.push(...response.data);
  
  // Process items based on operation
  const results = [];
  for (const [index, item] of items.entries()) {
    const result = await processItem(item, operation);
    results.push(result);
    
    // Update job progress
    job.progress(Math.round((index + 1) / items.length * 100));
  }
  
  return {
    processed: items.length,
    results: results.length,
    chunkIndex: job.data.chunkIndex
  };
});

// Monitor completion
batchQueue.on('completed', async (job) => {
  if (job.name === 'process-chunk') {
    const parentJob = await batchQueue.getJob(job.data.parentJobId);
    const childJobs = await Promise.all(
      parentJob.data.childJobs.map(id => batchQueue.getJob(id))
    );
    
    const allCompleted = childJobs.every(j => 
      j.finishedOn !== null
    );
    
    if (allCompleted) {
      console.log(`All chunks completed for parent job ${parentJob.id}`);
      // Aggregate results, send notifications, etc.
    }
  }
});
```

### Worker Thread Processing

```javascript
// main.js
import { Worker } from 'worker_threads';
import os from 'os';

async function processWithWorkers(items, options = {}) {
  const {
    workerCount = os.cpus().length,
    workerScript = './item-processor-worker.js'
  } = options;
  
  const chunks = [];
  const chunkSize = Math.ceil(items.length / workerCount);
  
  // Split items into chunks
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }
  
  // Create workers
  const workers = chunks.map((chunk, index) => {
    return new Promise((resolve, reject) => {
      const worker = new Worker(workerScript, {
        workerData: {
          chunk,
          chunkIndex: index,
          apiToken: process.env.DATOCMS_API_TOKEN
        }
      });
      
      worker.on('message', resolve);
      worker.on('error', reject);
      worker.on('exit', (code) => {
        if (code !== 0) {
          reject(new Error(`Worker stopped with exit code ${code}`));
        }
      });
    });
  });
  
  // Wait for all workers to complete
  const results = await Promise.all(workers);
  
  // Aggregate results
  return results.reduce((acc, result) => ({
    processed: acc.processed + result.processed,
    successful: acc.successful + result.successful,
    failed: acc.failed + result.failed
  }), { processed: 0, successful: 0, failed: 0 });
}

// item-processor-worker.js
import { parentPort, workerData } from 'worker_threads';
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: workerData.apiToken });

async function processChunk() {
  const { chunk, chunkIndex } = workerData;
  const results = {
    processed: 0,
    successful: 0,
    failed: 0
  };
  
  for (const item of chunk) {
    try {
      // Your processing logic here
      await processItem(item);
      results.successful++;
    } catch (error) {
      console.error(`Worker ${chunkIndex} - Error processing item:`, error);
      results.failed++;
    }
    results.processed++;
  }
  
  parentPort.postMessage(results);
}

processChunk().catch(error => {
  console.error('Worker error:', error);
  process.exit(1);
});
```

## Next Steps

- [Async Jobs](./async-jobs.md) - Track long-running operations
- [Webhook Integration](../integrations/webhook-integration.md) - Trigger batch processing via webhooks
- [Rate Limiting](../../advanced/rate-limiting.md) - Handle API rate limits in batch operations