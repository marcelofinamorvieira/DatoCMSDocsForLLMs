# Item Version

Item Versions provide a complete history of changes made to content items. Every time an item is saved, a new version is created, allowing you to track changes, compare versions, and restore previous states.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// List all versions of an item
const versions = await client.itemVersions.list('item-id');

// Get a specific version
const version = await client.itemVersions.find('item-id', 'version-id');

// Restore a previous version
const restored = await client.itemVersions.restore('item-id', 'version-id');
```

## API Reference

### List Item Versions

Retrieve all versions of a specific item.

```typescript
// Get all versions
const versions = await client.itemVersions.list('item-id');

// Versions are returned in reverse chronological order
console.log(`Total versions: ${versions.length}`);
console.log(`Current version: ${versions[0].id}`);
console.log(`First version: ${versions[versions.length - 1].id}`);

// Filter to find specific changes
const majorChanges = versions.filter(v => {
  const meta = v.meta;
  return meta.created_by?.type === 'user' && 
         meta.changed_attributes?.includes('title');
});
```

**Parameters:**
- `itemId` (string, required): The item ID

**Returns:** Array of item version objects ordered by creation date (newest first)

### Find Item Version

Retrieve a specific version of an item.

```typescript
const version = await client.itemVersions.find('item-id', 'version-id');

console.log(`Version: ${version.id}`);
console.log(`Created: ${version.meta.created_at}`);
console.log(`Created by: ${version.meta.created_by?.email || 'System'}`);
console.log(`Changes: ${version.meta.changed_attributes?.join(', ')}`);

// Access the item data at this version
console.log('Item data:', version.attributes);
```

**Parameters:**
- `itemId` (string, required): The item ID
- `versionId` (string, required): The version ID

**Returns:** Item version object with full item data

### Restore Item Version

Restore an item to a previous version.

```typescript
// Restore to a specific version
const restoredItem = await client.itemVersions.restore('item-id', 'version-id');

console.log('Item restored to previous version');
console.log(`New current version: ${restoredItem.meta.current_version}`);

// Note: This creates a new version with the old content
// The version history is preserved
```

**Parameters:**
- `itemId` (string, required): The item ID
- `versionId` (string, required): The version ID to restore

**Returns:** The restored item object

**Note:** Restoring creates a new version with the content from the selected version

## Version Management Patterns

### Version Comparison

Compare different versions to see what changed:

```typescript
async function compareVersions(itemId: string, versionA: string, versionB: string) {
  const [verA, verB] = await Promise.all([
    client.itemVersions.find(itemId, versionA),
    client.itemVersions.find(itemId, versionB)
  ]);
  
  const changes = {};
  const allFields = new Set([
    ...Object.keys(verA.attributes),
    ...Object.keys(verB.attributes)
  ]);
  
  allFields.forEach(field => {
    const valueA = verA.attributes[field];
    const valueB = verB.attributes[field];
    
    if (JSON.stringify(valueA) !== JSON.stringify(valueB)) {
      changes[field] = {
        before: valueA,
        after: valueB
      };
    }
  });
  
  return {
    versionA: {
      id: versionA,
      date: verA.meta.created_at,
      author: verA.meta.created_by?.email
    },
    versionB: {
      id: versionB,
      date: verB.meta.created_at,
      author: verB.meta.created_by?.email
    },
    changes
  };
}

// Compare two versions
const diff = await compareVersions('item-id', 'version-1', 'version-2');
console.log('Changes:', diff.changes);
```

### Version History Analysis

Analyze editing patterns and contribution history:

```typescript
async function analyzeVersionHistory(itemId: string) {
  const versions = await client.itemVersions.list(itemId);
  
  const analysis = {
    totalVersions: versions.length,
    timespan: {
      first: versions[versions.length - 1].meta.created_at,
      last: versions[0].meta.created_at
    },
    contributors: {},
    changeFrequency: {},
    fieldChangeCount: {}
  };
  
  // Analyze each version
  versions.forEach((version, index) => {
    // Track contributors
    const author = version.meta.created_by?.email || 'System';
    analysis.contributors[author] = (analysis.contributors[author] || 0) + 1;
    
    // Track change frequency by day
    const date = new Date(version.meta.created_at).toISOString().split('T')[0];
    analysis.changeFrequency[date] = (analysis.changeFrequency[date] || 0) + 1;
    
    // Track which fields change most often
    version.meta.changed_attributes?.forEach(field => {
      analysis.fieldChangeCount[field] = (analysis.fieldChangeCount[field] || 0) + 1;
    });
  });
  
  // Calculate average time between versions
  if (versions.length > 1) {
    const firstDate = new Date(versions[versions.length - 1].meta.created_at);
    const lastDate = new Date(versions[0].meta.created_at);
    const daysBetween = (lastDate - firstDate) / (1000 * 60 * 60 * 24);
    analysis.avgDaysBetweenVersions = daysBetween / (versions.length - 1);
  }
  
  return analysis;
}
```

### Automated Backup

Create periodic backups using versions:

```typescript
class VersionBackupManager {
  static async createBackup(itemId: string, reason?: string) {
    // Get current item state
    const item = await client.items.find(itemId);
    
    // Add backup metadata
    const backupData = {
      ...item,
      _backup_metadata: {
        created_at: new Date().toISOString(),
        reason: reason || 'Manual backup',
        original_version: item.meta.current_version
      }
    };
    
    // Update item to create a new version with backup tag
    await client.items.update(itemId, {
      _backup_tag: `backup_${Date.now()}`
    });
    
    return {
      itemId,
      backupVersion: item.meta.current_version,
      timestamp: new Date().toISOString()
    };
  }
  
  static async findBackups(itemId: string) {
    const versions = await client.itemVersions.list(itemId);
    
    // Find versions that are backups
    return versions.filter(v => 
      v.attributes._backup_tag || 
      v.meta.changed_attributes?.includes('_backup_tag')
    );
  }
}
```

### Version Rollback Workflow

Implement a safe rollback process:

```typescript
async function rollbackWorkflow(itemId: string, targetVersionId: string) {
  try {
    // Step 1: Get current state
    const currentItem = await client.items.find(itemId);
    const currentVersion = currentItem.meta.current_version;
    
    // Step 2: Verify target version exists
    const targetVersion = await client.itemVersions.find(itemId, targetVersionId);
    
    // Step 3: Create a safety backup of current state
    console.log('Creating safety backup...');
    await client.items.update(itemId, {
      _rollback_from: currentVersion
    });
    
    // Step 4: Restore to target version
    console.log(`Rolling back to version ${targetVersionId}...`);
    const restored = await client.itemVersions.restore(itemId, targetVersionId);
    
    // Step 5: Verify restoration
    const newVersion = restored.meta.current_version;
    
    return {
      success: true,
      rollback: {
        from: currentVersion,
        to: targetVersionId,
        newVersion: newVersion
      },
      item: restored
    };
  } catch (error) {
    console.error('Rollback failed:', error);
    throw error;
  }
}
```

## Advanced Patterns

### Version Diff Visualization

Create a detailed diff for review:

```typescript
function generateVersionDiff(versionA: any, versionB: any) {
  const diff = {
    meta: {
      versionA: {
        id: versionA.id,
        created: versionA.meta.created_at,
        author: versionA.meta.created_by?.email || 'System'
      },
      versionB: {
        id: versionB.id,
        created: versionB.meta.created_at,
        author: versionB.meta.created_by?.email || 'System'
      }
    },
    changes: []
  };
  
  // Compare all fields
  const allFields = new Set([
    ...Object.keys(versionA.attributes),
    ...Object.keys(versionB.attributes)
  ]);
  
  allFields.forEach(field => {
    const valueA = versionA.attributes[field];
    const valueB = versionB.attributes[field];
    
    if (JSON.stringify(valueA) !== JSON.stringify(valueB)) {
      const change = {
        field,
        type: !valueA ? 'added' : !valueB ? 'removed' : 'modified',
        before: valueA,
        after: valueB
      };
      
      // Add field type info for better formatting
      if (typeof valueA === 'object' && valueA?.upload_id) {
        change.fieldType = 'file';
      } else if (Array.isArray(valueA)) {
        change.fieldType = 'array';
      } else if (typeof valueA === 'object') {
        change.fieldType = 'object';
      } else {
        change.fieldType = 'scalar';
      }
      
      diff.changes.push(change);
    }
  });
  
  return diff;
}

// Format diff for display
function formatDiff(diff: any) {
  console.log('\n=== Version Comparison ===');
  console.log(`From: v${diff.meta.versionA.id} (${diff.meta.versionA.created})`);
  console.log(`To: v${diff.meta.versionB.id} (${diff.meta.versionB.created})`);
  console.log(`\nChanges (${diff.changes.length}):`);
  
  diff.changes.forEach(change => {
    console.log(`\n- ${change.field} [${change.type}]`);
    if (change.type !== 'added') {
      console.log(`  Before: ${JSON.stringify(change.before, null, 2)}`);
    }
    if (change.type !== 'removed') {
      console.log(`  After: ${JSON.stringify(change.after, null, 2)}`);
    }
  });
}
```

### Version Pruning

Manage version history size:

```typescript
class VersionPruningManager {
  static async pruneOldVersions(itemId: string, options: {
    keepLast?: number;
    keepDays?: number;
    keepMilestones?: boolean;
  } = {}) {
    const keepLast = options.keepLast || 10;
    const keepDays = options.keepDays || 30;
    
    const versions = await client.itemVersions.list(itemId);
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - keepDays);
    
    // Identify versions to keep
    const versionsToKeep = new Set<string>();
    
    // Keep recent versions
    versions.slice(0, keepLast).forEach(v => versionsToKeep.add(v.id));
    
    // Keep versions within date range
    versions.forEach(v => {
      if (new Date(v.meta.created_at) > cutoffDate) {
        versionsToKeep.add(v.id);
      }
    });
    
    // Keep milestone versions (e.g., published versions)
    if (options.keepMilestones) {
      versions.forEach(v => {
        if (v.meta.is_published_version) {
          versionsToKeep.add(v.id);
        }
      });
    }
    
    const versionsToPrune = versions.filter(v => !versionsToKeep.has(v.id));
    
    return {
      totalVersions: versions.length,
      keeping: versionsToKeep.size,
      pruning: versionsToPrune.length,
      oldestKept: Array.from(versionsToKeep).pop(),
      prunedVersions: versionsToPrune.map(v => ({
        id: v.id,
        created: v.meta.created_at
      }))
    };
  }
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.itemVersions.find('item-id', 'invalid-version-id');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 404) {
    console.log('Version not found');
  }
}

// Handle restore errors
try {
  await client.itemVersions.restore('item-id', 'old-version-id');
} catch (error) {
  if (error instanceof ApiError) {
    if (error.response.status === 422) {
      console.log('Cannot restore version - item may be locked or deleted');
    }
  }
}

// Handle permission errors
try {
  await client.itemVersions.list('restricted-item-id');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 403) {
    console.log('No permission to view item versions');
  }
}
```

## Item Version Object Structure

```typescript
interface ItemVersion {
  id: string;
  type: 'item_version';
  attributes: Record<string, any>; // The item's field values at this version
  relationships: {
    item_type: {
      data: {
        type: 'item_type';
        id: string;
      };
    };
  };
  meta: {
    created_at: string;
    created_by?: {
      type: 'user' | 'api_token';
      id: string;
      email?: string;
    };
    changed_attributes?: string[];
    is_published_version?: boolean;
    is_current_version?: boolean;
  };
}

// Example version object
{
  id: 'version-123',
  type: 'item_version',
  attributes: {
    title: 'Blog Post Title',
    content: 'This is the content at this version...',
    author: 'John Doe',
    published_date: '2024-01-15'
  },
  relationships: {
    item_type: {
      data: {
        type: 'item_type',
        id: 'blog_post'
      }
    }
  },
  meta: {
    created_at: '2024-01-15T10:30:00Z',
    created_by: {
      type: 'user',
      id: 'user-456',
      email: 'editor@example.com'
    },
    changed_attributes: ['title', 'content'],
    is_published_version: false,
    is_current_version: false
  }
}
```

## Related Resources

- [Items](./item.md) - Current item state and operations
- [Audit Log Events](../06-monitoring/audit-log-event.md) - Track all changes
- [Users](../05-access-control/user.md) - See who made changes
- [Workflows](../03-publishing-workflow/workflow.md) - Version approval processes