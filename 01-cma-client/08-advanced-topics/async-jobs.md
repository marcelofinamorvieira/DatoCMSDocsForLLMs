# Async Jobs

## When to Use Async Jobs
- Processing 200+ items
- Long-running operations
- Bulk imports/exports
- Heavy computations

## Create Async Job

### Bulk Item Creation
```javascript
// Prepare large dataset
const items = Array.from({ length: 1000 }, (_, i) => ({
  item_type: 'product',
  name: `Product ${i + 1}`,
  sku: `SKU-${String(i + 1).padStart(4, '0')}`,
  price: Math.random() * 100
}));

// Start async job
const job = await client.items.bulkCreateJob({
  data: items
});

console.log('Job started:', job.id);
console.log('Status:', job.attributes.status);
```

### Bulk Update Job
```javascript
// Update many items
const updates = existingItems.map(item => ({
  id: item.id,
  attributes: {
    status: 'archived',
    archived_at: new Date().toISOString()
  }
}));

const job = await client.items.bulkUpdateJob({
  data: updates
});
```

### Bulk Destroy Job
```javascript
// Delete many items
const itemIds = await client.items.list({
  filter: {
    type: 'old_content',
    fields: {
      created_at: { lt: '2020-01-01' }
    }
  },
  page: { limit: 500 }
}).then(items => items.map(i => i.id));

const job = await client.items.bulkDestroyJob({
  items: itemIds
});
```

## Monitor Job Progress

### Poll for Completion
```javascript
async function waitForJob(jobId) {
  let job;
  
  do {
    job = await client.jobs.find(jobId);
    
    console.log(`Progress: ${job.attributes.progress * 100}%`);
    console.log(`Status: ${job.attributes.status}`);
    
    if (job.attributes.status === 'in_progress') {
      await new Promise(resolve => setTimeout(resolve, 2000)); // Wait 2s
    }
  } while (job.attributes.status === 'in_progress');
  
  return job;
}

// Use it
const completedJob = await waitForJob(job.id);
```

### Progress Callback Pattern
```javascript
async function runJobWithProgress(jobPromise, onProgress) {
  const job = await jobPromise;
  
  const checkProgress = async () => {
    const currentJob = await client.jobs.find(job.id);
    
    onProgress({
      progress: currentJob.attributes.progress,
      status: currentJob.attributes.status,
      details: currentJob.attributes.details
    });
    
    if (currentJob.attributes.status === 'in_progress') {
      setTimeout(checkProgress, 1000);
    } else {
      return currentJob;
    }
  };
  
  return checkProgress();
}

// Usage
await runJobWithProgress(
  client.items.bulkCreateJob({ data: items }),
  ({ progress, status }) => {
    console.log(`${(progress * 100).toFixed(1)}% - ${status}`);
  }
);
```

## Handle Job Results

### Success Handling
```javascript
const completedJob = await waitForJob(job.id);

if (completedJob.attributes.status === 'completed') {
  const result = completedJob.attributes.result;
  
  console.log('Job completed successfully');
  console.log(`Processed: ${result.processed_items} items`);
  console.log(`Success: ${result.successful_items} items`);
  console.log(`Failed: ${result.failed_items} items`);
  
  // Access created/updated items
  if (result.data) {
    result.data.forEach(item => {
      console.log(`Created: ${item.id}`);
    });
  }
}
```

### Error Handling
```javascript
if (completedJob.attributes.status === 'failed') {
  const error = completedJob.attributes.error;
  console.error('Job failed:', error.message);
  
  // Get detailed errors
  if (error.details) {
    error.details.forEach((detail, index) => {
      console.error(`Item ${index}: ${detail.error}`);
    });
  }
}
```

### Partial Success
```javascript
if (completedJob.attributes.status === 'completed_with_errors') {
  const result = completedJob.attributes.result;
  
  console.warn('Job completed with some errors');
  console.log(`Success: ${result.successful_items}`);
  console.log(`Failed: ${result.failed_items}`);
  
  // Handle failed items
  result.errors.forEach((error, index) => {
    console.error(`Item ${index} failed: ${error.message}`);
  });
}
```

## Job Types

### Import Job
```javascript
// Import from external source
async function importFromCSV(csvData) {
  const items = parseCSV(csvData).map(row => ({
    item_type: 'product',
    name: row.name,
    price: parseFloat(row.price),
    sku: row.sku,
    description: row.description
  }));
  
  const job = await client.items.bulkCreateJob({
    data: items,
    skip_duplicates: true
  });
  
  return job;
}
```

### Migration Job
```javascript
// Migrate between models
async function migrateContent(fromModel, toModel, transformer) {
  // Fetch all items
  const items = await client.items.list({
    filter: { type: fromModel },
    page: { limit: 500 }
  });
  
  // Transform data
  const newItems = items.map(item => transformer(item));
  
  // Create in new model
  const createJob = await client.items.bulkCreateJob({
    data: newItems
  });
  
  // Wait for completion
  const result = await waitForJob(createJob.id);
  
  if (result.attributes.status === 'completed') {
    // Delete old items
    const deleteJob = await client.items.bulkDestroyJob({
      items: items.map(i => i.id)
    });
    
    return waitForJob(deleteJob.id);
  }
}
```

### Export Job
```javascript
// Export large dataset
async function exportAllContent() {
  const job = await client.items.exportJob({
    filter: { type: { in: ['article', 'page', 'product'] } },
    format: 'json',
    include_relationships: true
  });
  
  const completed = await waitForJob(job.id);
  
  if (completed.attributes.status === 'completed') {
    const downloadUrl = completed.attributes.result.url;
    console.log('Download export:', downloadUrl);
  }
}
```

## Advanced Patterns

### Chunked Processing
```javascript
async function processInChunks(items, chunkSize = 100) {
  const chunks = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }
  
  const jobs = [];
  
  for (const [index, chunk] of chunks.entries()) {
    console.log(`Starting chunk ${index + 1}/${chunks.length}`);
    
    const job = await client.items.bulkCreateJob({
      data: chunk
    });
    
    jobs.push(job);
    
    // Optional: wait between chunks
    if (index < chunks.length - 1) {
      await new Promise(r => setTimeout(r, 1000));
    }
  }
  
  // Wait for all jobs
  const results = await Promise.all(
    jobs.map(job => waitForJob(job.id))
  );
  
  return results;
}
```

### Retry Failed Items
```javascript
async function retryFailedItems(failedJob) {
  const failedItems = failedJob.attributes.result.failed_items_data;
  
  if (!failedItems || failedItems.length === 0) {
    console.log('No failed items to retry');
    return;
  }
  
  console.log(`Retrying ${failedItems.length} failed items`);
  
  // Fix common issues
  const fixedItems = failedItems.map(item => {
    // Apply fixes based on error type
    if (item.error.includes('required')) {
      item.data.status = 'draft'; // Add missing required field
    }
    return item.data;
  });
  
  // Retry
  const retryJob = await client.items.bulkCreateJob({
    data: fixedItems
  });
  
  return waitForJob(retryJob.id);
}
```

### Progress UI
```javascript
function createProgressBar(total) {
  return {
    total,
    current: 0,
    update(progress) {
      this.current = Math.floor(progress * this.total);
      const percentage = Math.floor(progress * 100);
      const filled = Math.floor(progress * 20);
      const empty = 20 - filled;
      
      process.stdout.clearLine();
      process.stdout.cursorTo(0);
      process.stdout.write(
        `[${'█'.repeat(filled)}${' '.repeat(empty)}] ${percentage}% (${this.current}/${this.total})`
      );
    },
    complete() {
      process.stdout.write('\n');
    }
  };
}

// Usage
const progressBar = createProgressBar(items.length);

await runJobWithProgress(
  client.items.bulkCreateJob({ data: items }),
  ({ progress }) => progressBar.update(progress)
);

progressBar.complete();
```

## Best Practices

1. **Use for large operations** (200+ items)
2. **Monitor progress** for user feedback
3. **Handle partial failures** gracefully
4. **Log job IDs** for debugging
5. **Implement retry logic** for failures
6. **Clean up completed jobs** periodically
7. **Set reasonable timeouts**

## Common Patterns

### Import with Validation
```javascript
async function validateAndImport(items) {
  // Pre-validate
  const valid = [];
  const invalid = [];
  
  items.forEach((item, index) => {
    const errors = validateItem(item);
    if (errors.length === 0) {
      valid.push(item);
    } else {
      invalid.push({ index, item, errors });
    }
  });
  
  console.log(`Valid: ${valid.length}, Invalid: ${invalid.length}`);
  
  if (valid.length > 0) {
    const job = await client.items.bulkCreateJob({
      data: valid
    });
    
    return {
      job: await waitForJob(job.id),
      invalid
    };
  }
}
```

## Next Steps
- [Handle large datasets](./handle-large-datasets.md)
- [Progress tracking](./progress-tracking.md)
- [Webhook integration](../integrations/webhook-integration.md)
- [Bulk updates](../../02-content-operations/update-items/bulk-update.md)