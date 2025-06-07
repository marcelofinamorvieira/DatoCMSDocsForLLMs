# Search Result

Search Results provide a powerful full-text search capability across all content in your DatoCMS project. You can search through items, uploads, and other resources to quickly find what you need.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Search for content
const results = await client.searchResults.list({
  q: 'product launch',
  locale: 'en'
});
```

## API Reference

### Search Content

Perform full-text search across your project content.

```typescript
// Simple search
const results = await client.searchResults.list({
  q: 'marketing campaign'
});

// Search with locale filter
const englishResults = await client.searchResults.list({
  q: 'product features',
  locale: 'en'
});

// Search with type filter
const articleResults = await client.searchResults.list({
  q: 'tutorial',
  filter: {
    item_types: { any_in: ['article', 'blog_post'] }
  }
});

// Exclude certain types
const nonBlogResults = await client.searchResults.list({
  q: 'announcement',
  filter: {
    excluded_item_types: { any_in: ['blog_post', 'news'] }
  }
});

// Paginated search
const paginatedResults = await client.searchResults.list({
  q: 'documentation',
  page: { offset: 20, limit: 50 }
});
```

**Parameters:**
- `q` (string, required): Search query text
- `locale` (string, optional): Filter by specific locale
- `filter` (object, optional): Additional filters
  - `item_types`: Include only these item types
  - `excluded_item_types`: Exclude these item types
- `page` (object, optional): Pagination options
  - `offset`: Starting position (default: 0)
  - `limit`: Number of results (default: 20, max: 100)

**Returns:** Array of search result objects with pagination metadata

## Search Result Types

### Item Results

Search results for content items:

```typescript
const itemResults = await client.searchResults.list({
  q: 'user guide'
});

itemResults.forEach(result => {
  if (result.type === 'item') {
    console.log(`Title: ${result.title}`);
    console.log(`Model: ${result.item_type.name}`);
    console.log(`Score: ${result.score}`);
    console.log(`Highlights: ${result.highlights.join(', ')}`);
  }
});
```

### Upload Results

Search results for media files:

```typescript
const mediaResults = await client.searchResults.list({
  q: 'presentation'
});

const uploads = mediaResults.filter(r => r.type === 'upload');
uploads.forEach(upload => {
  console.log(`File: ${upload.filename}`);
  console.log(`Type: ${upload.mime_type}`);
  console.log(`Tags: ${upload.tags.join(', ')}`);
});
```

## Advanced Search Patterns

### Multi-Language Search

Search across multiple locales:

```typescript
async function searchAllLocales(query: string) {
  const locales = ['en', 'es', 'fr', 'de', 'it'];
  const allResults = {};
  
  for (const locale of locales) {
    const results = await client.searchResults.list({
      q: query,
      locale: locale
    });
    
    if (results.length > 0) {
      allResults[locale] = results;
    }
  }
  
  return allResults;
}

// Find content in any language
const multilingual = await searchAllLocales('product');
console.log(`Found in ${Object.keys(multilingual).length} languages`);
```

### Relevance Scoring

Work with search relevance scores:

```typescript
async function searchWithRelevance(query: string, threshold = 0.5) {
  const results = await client.searchResults.list({ q: query });
  
  // Sort by relevance score
  const sorted = results.sort((a, b) => b.score - a.score);
  
  // Filter by minimum relevance
  const relevant = sorted.filter(r => r.score >= threshold);
  
  return {
    total: results.length,
    relevant: relevant.length,
    topResults: relevant.slice(0, 5).map(r => ({
      title: r.title,
      score: r.score,
      type: r.type,
      highlights: r.highlights
    }))
  };
}
```

### Search Highlighting

Extract and display search highlights:

```typescript
function formatSearchResults(results: any[]) {
  return results.map(result => {
    // Format highlights with emphasis
    const formattedHighlights = result.highlights.map(h => 
      h.replace(/<mark>/g, '**').replace(/<\/mark>/g, '**')
    );
    
    return {
      title: result.title,
      type: result.type,
      model: result.item_type?.name,
      context: formattedHighlights,
      url: `/admin/items/${result.id}` // Admin panel link
    };
  });
}

// Search and format results
const results = await client.searchResults.list({ q: 'important update' });
const formatted = formatSearchResults(results);

formatted.forEach(item => {
  console.log(`\n${item.title} (${item.model})`);
  item.context.forEach(ctx => console.log(`  - ${ctx}`));
});
```

### Faceted Search

Implement faceted search with type filtering:

```typescript
async function facetedSearch(query: string) {
  // First, get all results to determine facets
  const allResults = await client.searchResults.list({ q: query });
  
  // Count by item type
  const facets = {};
  allResults.forEach(result => {
    if (result.item_type) {
      const typeName = result.item_type.name;
      facets[typeName] = (facets[typeName] || 0) + 1;
    }
  });
  
  // Get item type IDs for filtering
  const itemTypes = await client.itemTypes.list();
  const typeMap = new Map(itemTypes.map(t => [t.name, t.id]));
  
  return {
    query,
    totalResults: allResults.length,
    facets,
    async filterByType(typeName: string) {
      const typeId = typeMap.get(typeName);
      if (!typeId) return [];
      
      return client.searchResults.list({
        q: query,
        filter: {
          item_types: { any_in: [typeId] }
        }
      });
    }
  };
}

// Use faceted search
const search = await facetedSearch('guide');
console.log('Results by type:', search.facets);

// Filter by specific type
const articles = await search.filterByType('Article');
console.log(`Found ${articles.length} articles`);
```

## Search Analytics

### Search Query Logging

Track what users are searching for:

```typescript
class SearchAnalytics {
  private searches: Array<{
    query: string;
    timestamp: Date;
    resultCount: number;
    locale?: string;
  }> = [];
  
  async search(client: any, query: string, locale?: string) {
    const results = await client.searchResults.list({
      q: query,
      locale
    });
    
    // Log the search
    this.searches.push({
      query,
      timestamp: new Date(),
      resultCount: results.length,
      locale
    });
    
    return results;
  }
  
  getTopSearches(limit = 10) {
    const counts = {};
    this.searches.forEach(s => {
      counts[s.query] = (counts[s.query] || 0) + 1;
    });
    
    return Object.entries(counts)
      .sort(([, a], [, b]) => b - a)
      .slice(0, limit)
      .map(([query, count]) => ({ query, count }));
  }
  
  getNoResultsQueries() {
    return this.searches
      .filter(s => s.resultCount === 0)
      .map(s => s.query);
  }
}
```

### Search Performance

Optimize search queries:

```typescript
async function optimizedSearch(queries: string[]) {
  // Batch multiple searches
  const results = await Promise.all(
    queries.map(q => 
      client.searchResults.list({ 
        q, 
        page: { limit: 10 } // Limit results for performance
      })
    )
  );
  
  return queries.map((query, index) => ({
    query,
    results: results[index],
    count: results[index].length
  }));
}

// Cache frequent searches
class SearchCache {
  private cache = new Map<string, any>();
  private maxAge = 5 * 60 * 1000; // 5 minutes
  
  async search(client: any, query: string, options = {}) {
    const cacheKey = JSON.stringify({ query, options });
    const cached = this.cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this.maxAge) {
      return cached.results;
    }
    
    const results = await client.searchResults.list({
      q: query,
      ...options
    });
    
    this.cache.set(cacheKey, {
      results,
      timestamp: Date.now()
    });
    
    return results;
  }
}
```

## Search UI Integration

### Search Suggestions

Build autocomplete functionality:

```typescript
async function getSearchSuggestions(
  query: string,
  maxSuggestions = 5
) {
  if (query.length < 2) return [];
  
  const results = await client.searchResults.list({
    q: query,
    page: { limit: maxSuggestions * 2 }
  });
  
  // Extract unique titles as suggestions
  const suggestions = new Set<string>();
  
  results.forEach(result => {
    if (result.title && suggestions.size < maxSuggestions) {
      suggestions.add(result.title);
    }
    
    // Also add highlighted portions
    result.highlights.forEach(highlight => {
      const clean = highlight.replace(/<\/?mark>/g, '');
      if (clean.length > 10 && suggestions.size < maxSuggestions) {
        suggestions.add(clean.substring(0, 50) + '...');
      }
    });
  });
  
  return Array.from(suggestions);
}
```

### Search Results Grouping

Group results by type or category:

```typescript
async function groupedSearch(query: string) {
  const results = await client.searchResults.list({ q: query });
  
  // Group by item type
  const grouped = results.reduce((acc, result) => {
    const type = result.item_type?.name || 'Other';
    if (!acc[type]) {
      acc[type] = [];
    }
    acc[type].push(result);
    return acc;
  }, {});
  
  // Sort groups by result count
  const sortedGroups = Object.entries(grouped)
    .sort(([, a], [, b]) => b.length - a.length)
    .map(([type, items]) => ({
      type,
      count: items.length,
      items: items.slice(0, 5) // Top 5 per group
    }));
  
  return sortedGroups;
}
```

## Advanced Search Techniques

### Boolean Search

Implement advanced search operators:

```typescript
class AdvancedSearch {
  // Convert boolean query to DatoCMS search
  static parseQuery(query: string): string {
    // Simple boolean operator support
    return query
      .replace(/\sAND\s/gi, ' ')  // AND is implicit
      .replace(/\sOR\s/gi, ' | ')  // OR operator
      .replace(/\sNOT\s/gi, ' -')  // NOT operator
      .replace(/"/g, '');          // Remove quotes (exact match not supported)
  }
  
  static async search(client: any, advancedQuery: string) {
    const parsed = this.parseQuery(advancedQuery);
    
    try {
      return await client.searchResults.list({ q: parsed });
    } catch (error) {
      // Fallback to simple search if parsing fails
      return await client.searchResults.list({ 
        q: advancedQuery.replace(/\s(AND|OR|NOT)\s/gi, ' ')
      });
    }
  }
}

// Examples
await AdvancedSearch.search(client, 'product AND launch');
await AdvancedSearch.search(client, 'update OR announcement');
await AdvancedSearch.search(client, 'security NOT patch');
```

### Content Discovery

Find related content based on search:

```typescript
async function findRelatedContent(itemId: string, maxResults = 5) {
  // Get the original item
  const item = await client.items.find(itemId);
  
  // Extract keywords from title and content
  const keywords = extractKeywords(item);
  
  // Search for related content
  const results = await client.searchResults.list({
    q: keywords.join(' '),
    filter: {
      excluded_item_types: [item.item_type]
    },
    page: { limit: maxResults + 1 }
  });
  
  // Exclude the original item
  return results
    .filter(r => r.id !== itemId)
    .slice(0, maxResults);
}

function extractKeywords(item: any): string[] {
  const text = [
    item.title,
    item.description,
    item.content
  ].filter(Boolean).join(' ');
  
  // Simple keyword extraction
  return text
    .toLowerCase()
    .split(/\s+/)
    .filter(word => word.length > 4)
    .slice(0, 5);
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.searchResults.list({
    q: '', // Empty query
    page: { limit: 1000 } // Exceeds max
  });
} catch (error) {
  if (error instanceof ApiError) {
    if (error.response.status === 422) {
      const queryError = error.findError('q');
      if (queryError?.code === 'VALIDATION_REQUIRED') {
        console.log('Search query is required');
      }
    }
  }
}

// Handle no results
const results = await client.searchResults.list({ q: 'xyz123unlikely' });
if (results.length === 0) {
  console.log('No results found. Try different keywords.');
}
```

## Search Result Object Structure

```typescript
interface SearchResult {
  id: string;
  type: 'item' | 'upload';
  title: string;
  score: number; // Relevance score (0-1)
  highlights: string[]; // Text snippets with <mark> tags
  item_type?: {
    type: 'item_type';
    id: string;
    name: string;
  };
  // Additional fields for uploads
  filename?: string;
  mime_type?: string;
  tags?: string[];
  url?: string;
}

// Example search result
{
  id: '12345',
  type: 'item',
  title: 'Product Launch Announcement',
  score: 0.95,
  highlights: [
    'The new <mark>product launch</mark> is scheduled for next month',
    'This <mark>product</mark> features revolutionary technology'
  ],
  item_type: {
    type: 'item_type',
    id: 'article',
    name: 'Article'
  }
}
```

## Limitations

1. **Query Required**: Search query cannot be empty
2. **Result Limit**: Maximum 100 results per request
3. **No Wildcards**: Wildcard searches not supported
4. **No Exact Match**: Quoted exact phrase matching not available
5. **Relevance Algorithm**: Scoring algorithm is not customizable

## Related Resources

- [Items](../01-content-management/item.md) - Access found items
- [Uploads](../02-media-management/upload.md) - Access found uploads
- [Filter Items](../../02-content-operations/read-items/filter-items.md) - Alternative filtering methods

---

# Search Result Methods

The Search Result resource provides methods to monitor and analyze search queries performed on your DatoCMS site. This helps you understand what content your users are looking for and optimize your content strategy accordingly.

## Available Methods

### list()

Lists all search results from your site's search indexing, providing insights into search queries, result counts, and search performance.

```typescript
import { buildClient, SearchResult } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Basic usage - list all search results
const searchResults = await client.searchResults.list();

// The method returns a paginated collection
searchResults.forEach((result: SearchResult) => {
  console.log(`Query: "${result.query}" - ${result.results_count} results`);
});
```

## Response Structure

```typescript
interface SearchResult {
  id: string;
  type: 'search_result';
  attributes: {
    // The search query entered by the user
    query: string;
    
    // Number of results returned for this query
    results_count: number;
    
    // Locale in which the search was performed
    locale: string;
    
    // IP address of the user who performed the search
    ip_address: string;
    
    // User agent string of the browser/client
    user_agent: string;
    
    // Country code based on IP geolocation
    country_code: string | null;
    
    // Additional location data
    location: {
      city: string | null;
      region: string | null;
      latitude: number | null;
      longitude: number | null;
    } | null;
    
    // Timestamp when the search was performed
    created_at: string; // ISO 8601 format
    
    // Time taken to execute the search (in milliseconds)
    response_time_ms: number;
    
    // Whether the search was performed by a bot/crawler
    is_bot: boolean;
    
    // Referrer URL if available
    referrer: string | null;
    
    // The environment where the search was performed
    environment_id: string;
  };
  relationships: {
    environment: {
      data: {
        id: string;
        type: 'environment';
      };
    };
  };
}
```

## Filtering and Pagination

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Filter search results with parameters
const filteredResults = await client.searchResults.list({
  // Filter by date range
  filter: {
    created_at: {
      gte: '2024-01-01T00:00:00Z',
      lte: '2024-01-31T23:59:59Z'
    },
    // Filter by locale
    locale: {
      eq: 'en'
    },
    // Filter by result count (e.g., only queries with results)
    results_count: {
      gt: 0
    },
    // Filter by environment
    environment: {
      eq: 'main'
    }
  },
  
  // Pagination
  page: {
    offset: 0,
    limit: 100
  },
  
  // Sorting
  order_by: '-created_at' // Most recent first
});

// Process filtered results
filteredResults.forEach(result => {
  console.log(`[${result.attributes.created_at}] "${result.attributes.query}" - ${result.attributes.results_count} results`);
});
```

## Search Query Analysis Patterns

```typescript
import { buildClient, SearchResult } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Analyze most popular search queries
async function analyzePopularSearches(days: number = 30) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const results = await client.searchResults.list({
    filter: {
      created_at: {
        gte: startDate.toISOString()
      }
    },
    page: {
      limit: 500 // Adjust based on your volume
    }
  });
  
  // Group by query and count occurrences
  const queryStats = new Map<string, {
    count: number;
    totalResults: number;
    avgResponseTime: number;
  }>();
  
  results.forEach(result => {
    const query = result.attributes.query.toLowerCase().trim();
    const stats = queryStats.get(query) || {
      count: 0,
      totalResults: 0,
      avgResponseTime: 0
    };
    
    stats.count++;
    stats.totalResults += result.attributes.results_count;
    stats.avgResponseTime = (stats.avgResponseTime * (stats.count - 1) + result.attributes.response_time_ms) / stats.count;
    
    queryStats.set(query, stats);
  });
  
  // Sort by popularity and return top queries
  const topQueries = Array.from(queryStats.entries())
    .sort((a, b) => b[1].count - a[1].count)
    .slice(0, 20)
    .map(([query, stats]) => ({
      query,
      searchCount: stats.count,
      avgResults: Math.round(stats.totalResults / stats.count),
      avgResponseTime: Math.round(stats.avgResponseTime)
    }));
  
  return topQueries;
}

// Usage
const popularSearches = await analyzePopularSearches(30);
popularSearches.forEach((item, index) => {
  console.log(`${index + 1}. "${item.query}" - ${item.searchCount} searches, avg ${item.avgResults} results, ${item.avgResponseTime}ms`);
});
```

## Zero Results Detection

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Find queries that returned no results
async function findZeroResultQueries(days: number = 7) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const zeroResults = await client.searchResults.list({
    filter: {
      created_at: {
        gte: startDate.toISOString()
      },
      results_count: {
        eq: 0
      }
    },
    order_by: '-created_at'
  });
  
  // Group zero-result queries by frequency
  const zeroResultStats = new Map<string, number>();
  
  zeroResults.forEach(result => {
    const query = result.attributes.query.toLowerCase().trim();
    zeroResultStats.set(query, (zeroResultStats.get(query) || 0) + 1);
  });
  
  // Sort by frequency
  const topMissedQueries = Array.from(zeroResultStats.entries())
    .sort((a, b) => b[1] - a[1])
    .slice(0, 10);
  
  console.log('Top queries with no results:');
  topMissedQueries.forEach(([query, count]) => {
    console.log(`- "${query}": ${count} searches`);
  });
  
  return topMissedQueries;
}

// Usage
await findZeroResultQueries(30);
```

## Search Performance Metrics

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Calculate search performance metrics
async function calculateSearchMetrics(days: number = 7) {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - days);
  
  const results = await client.searchResults.list({
    filter: {
      created_at: {
        gte: startDate.toISOString()
      }
    },
    page: {
      limit: 1000
    }
  });
  
  // Calculate metrics
  const metrics = {
    totalSearches: results.length,
    avgResponseTime: 0,
    medianResponseTime: 0,
    p95ResponseTime: 0,
    searchesWithResults: 0,
    searchesWithoutResults: 0,
    avgResultsPerSearch: 0,
    uniqueQueries: new Set<string>(),
    searchesByLocale: new Map<string, number>(),
    searchesByHour: new Array(24).fill(0)
  };
  
  const responseTimes: number[] = [];
  let totalResults = 0;
  
  results.forEach(result => {
    responseTimes.push(result.attributes.response_time_ms);
    totalResults += result.attributes.results_count;
    
    if (result.attributes.results_count > 0) {
      metrics.searchesWithResults++;
    } else {
      metrics.searchesWithoutResults++;
    }
    
    metrics.uniqueQueries.add(result.attributes.query.toLowerCase().trim());
    
    // Count by locale
    const locale = result.attributes.locale;
    metrics.searchesByLocale.set(locale, (metrics.searchesByLocale.get(locale) || 0) + 1);
    
    // Count by hour
    const hour = new Date(result.attributes.created_at).getHours();
    metrics.searchesByHour[hour]++;
  });
  
  // Calculate response time metrics
  responseTimes.sort((a, b) => a - b);
  metrics.avgResponseTime = responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length;
  metrics.medianResponseTime = responseTimes[Math.floor(responseTimes.length / 2)];
  metrics.p95ResponseTime = responseTimes[Math.floor(responseTimes.length * 0.95)];
  metrics.avgResultsPerSearch = totalResults / results.length;
  
  return {
    totalSearches: metrics.totalSearches,
    uniqueQueries: metrics.uniqueQueries.size,
    avgResponseTime: Math.round(metrics.avgResponseTime),
    medianResponseTime: Math.round(metrics.medianResponseTime),
    p95ResponseTime: Math.round(metrics.p95ResponseTime),
    searchSuccessRate: ((metrics.searchesWithResults / metrics.totalSearches) * 100).toFixed(2) + '%',
    avgResultsPerSearch: metrics.avgResultsPerSearch.toFixed(1),
    searchesByLocale: Object.fromEntries(metrics.searchesByLocale),
    peakHour: metrics.searchesByHour.indexOf(Math.max(...metrics.searchesByHour))
  };
}

// Usage
const metrics = await calculateSearchMetrics(7);
console.log('Search Performance Metrics (Last 7 Days):');
console.log(JSON.stringify(metrics, null, 2));
```

## Geographic Analysis

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Analyze searches by geographic location
async function analyzeSearchesByLocation() {
  const results = await client.searchResults.list({
    filter: {
      created_at: {
        gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000).toISOString()
      }
    }
  });
  
  const locationStats = new Map<string, {
    searches: number;
    queries: Set<string>;
    avgResults: number;
  }>();
  
  results.forEach(result => {
    const country = result.attributes.country_code || 'Unknown';
    const stats = locationStats.get(country) || {
      searches: 0,
      queries: new Set(),
      avgResults: 0
    };
    
    stats.searches++;
    stats.queries.add(result.attributes.query.toLowerCase().trim());
    stats.avgResults = (stats.avgResults * (stats.searches - 1) + result.attributes.results_count) / stats.searches;
    
    locationStats.set(country, stats);
  });
  
  // Convert to sorted array
  const topCountries = Array.from(locationStats.entries())
    .sort((a, b) => b[1].searches - a[1].searches)
    .slice(0, 10)
    .map(([country, stats]) => ({
      country,
      searches: stats.searches,
      uniqueQueries: stats.queries.size,
      avgResults: Math.round(stats.avgResults)
    }));
  
  return topCountries;
}
```

## Error Handling

```typescript
import { buildClient, ApiError } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Robust error handling for search result monitoring
async function monitorSearchResults() {
  try {
    const results = await client.searchResults.list({
      page: { limit: 100 }
    });
    
    return results;
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.statusCode) {
        case 401:
          console.error('Invalid API token - check your credentials');
          break;
        case 403:
          console.error('Insufficient permissions to access search results');
          break;
        case 429:
          console.error('Rate limit exceeded - please retry later');
          console.log(`Retry after: ${error.headers['X-RateLimit-Reset']}`);
          break;
        case 500:
          console.error('Server error - please contact support');
          break;
        default:
          console.error(`API error ${error.statusCode}: ${error.message}`);
      }
    } else {
      console.error('Unexpected error:', error);
    }
    throw error;
  }
}
```

## Best Practices for Search Optimization

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Comprehensive search optimization analysis
async function generateSearchOptimizationReport() {
  const thirtyDaysAgo = new Date();
  thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
  
  // Fetch all search data
  const allSearches = await client.searchResults.list({
    filter: {
      created_at: { gte: thirtyDaysAgo.toISOString() }
    },
    page: { limit: 1000 }
  });
  
  // 1. Identify content gaps (queries with no/few results)
  const contentGaps = allSearches
    .filter(s => s.attributes.results_count < 3)
    .reduce((acc, search) => {
      const query = search.attributes.query.toLowerCase().trim();
      acc[query] = (acc[query] || 0) + 1;
      return acc;
    }, {} as Record<string, number>);
  
  // 2. Find slow searches
  const slowSearches = allSearches
    .filter(s => s.attributes.response_time_ms > 1000)
    .map(s => ({
      query: s.attributes.query,
      responseTime: s.attributes.response_time_ms,
      resultCount: s.attributes.results_count
    }));
  
  // 3. Analyze search patterns by time
  const searchesByDayOfWeek = new Array(7).fill(0);
  const searchesByHourOfDay = new Array(24).fill(0);
  
  allSearches.forEach(search => {
    const date = new Date(search.attributes.created_at);
    searchesByDayOfWeek[date.getDay()]++;
    searchesByHourOfDay[date.getHours()]++;
  });
  
  // 4. Common search refinements (queries that might be related)
  const queryPairs = new Map<string, number>();
  const sortedSearches = [...allSearches].sort((a, b) => 
    new Date(a.attributes.created_at).getTime() - new Date(b.attributes.created_at).getTime()
  );
  
  for (let i = 1; i < sortedSearches.length; i++) {
    const prev = sortedSearches[i - 1];
    const curr = sortedSearches[i];
    
    // If same IP and within 2 minutes, might be a refinement
    if (prev.attributes.ip_address === curr.attributes.ip_address) {
      const timeDiff = new Date(curr.attributes.created_at).getTime() - 
                      new Date(prev.attributes.created_at).getTime();
      if (timeDiff < 120000) { // 2 minutes
        const pair = `${prev.attributes.query} → ${curr.attributes.query}`;
        queryPairs.set(pair, (queryPairs.get(pair) || 0) + 1);
      }
    }
  }
  
  return {
    summary: {
      totalSearches: allSearches.length,
      uniqueQueries: new Set(allSearches.map(s => s.attributes.query.toLowerCase().trim())).size,
      avgResponseTime: Math.round(
        allSearches.reduce((sum, s) => sum + s.attributes.response_time_ms, 0) / allSearches.length
      ),
      searchSuccessRate: (
        (allSearches.filter(s => s.attributes.results_count > 0).length / allSearches.length) * 100
      ).toFixed(2) + '%'
    },
    recommendations: {
      contentGaps: Object.entries(contentGaps)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 10)
        .map(([query, count]) => ({
          query,
          searchCount: count,
          recommendation: `Create content for "${query}" - searched ${count} times with poor results`
        })),
      performanceIssues: slowSearches.slice(0, 5).map(s => ({
        ...s,
        recommendation: `Optimize search index for queries like "${s.query}" (${s.responseTime}ms)`
      })),
      peakTimes: {
        busiestDay: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'][
          searchesByDayOfWeek.indexOf(Math.max(...searchesByDayOfWeek))
        ],
        busiestHour: searchesByHourOfDay.indexOf(Math.max(...searchesByHourOfDay)),
        recommendation: 'Schedule search index updates during off-peak hours'
      },
      commonRefinements: Array.from(queryPairs.entries())
        .sort((a, b) => b[1] - a[1])
        .slice(0, 5)
        .map(([pair, count]) => ({
          refinement: pair,
          count,
          recommendation: `Consider improving results for the first query or adding related content links`
        }))
    }
  };
}

// Generate and display optimization report
const report = await generateSearchOptimizationReport();
console.log('Search Optimization Report:');
console.log(JSON.stringify(report, null, 2));
```

## Integration with Build Triggers

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Monitor search index freshness and trigger rebuilds if needed
async function monitorSearchIndexFreshness() {
  // Get recent searches with no results
  const recentZeroResults = await client.searchResults.list({
    filter: {
      created_at: {
        gte: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString() // Last 24 hours
      },
      results_count: { eq: 0 }
    }
  });
  
  // If many zero-result searches, might need to reindex
  if (recentZeroResults.length > 10) {
    console.log(`Found ${recentZeroResults.length} searches with no results in the last 24 hours`);
    
    // Check if content was recently updated
    const recentItems = await client.items.list({
      filter: {
        updated_at: {
          gte: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
        }
      },
      page: { limit: 1 }
    });
    
    if (recentItems.length > 0) {
      console.log('Recent content updates detected - triggering search reindex');
      
      // Find and trigger the search indexing build trigger
      const buildTriggers = await client.buildTriggers.list();
      const searchIndexTrigger = buildTriggers.find(
        trigger => trigger.name.toLowerCase().includes('search') || 
                  trigger.name.toLowerCase().includes('index')
      );
      
      if (searchIndexTrigger) {
        await client.buildTriggers.trigger(searchIndexTrigger.id);
        console.log('Search reindex triggered successfully');
      }
    }
  }
}
```

## Related Resources

- [Build Trigger Methods](../04-site-configuration/build-trigger.md) - Trigger search index rebuilds
- [Item Methods](../01-content-management/item-methods.md) - Manage content that gets indexed
- [Webhook Methods](../04-site-configuration/webhook.md) - Set up webhooks for search events
- [Site Methods](../04-site-configuration/site.md) - Configure site-wide search settings

## Search Result Attributes Reference

| Attribute | Type | Description |
|-----------|------|-------------|
| `query` | string | The search query entered by the user |
| `results_count` | number | Number of results returned for this query |
| `locale` | string | Locale in which the search was performed |
| `ip_address` | string | IP address of the user |
| `user_agent` | string | Browser/client user agent string |
| `country_code` | string \| null | Two-letter country code from IP geolocation |
| `location` | object \| null | Detailed location data (city, region, coordinates) |
| `created_at` | string | ISO 8601 timestamp of when the search occurred |
| `response_time_ms` | number | Time taken to execute the search in milliseconds |
| `is_bot` | boolean | Whether the search was performed by a bot/crawler |
| `referrer` | string \| null | The referring URL if available |
| `environment_id` | string | ID of the environment where search was performed |

## Performance Considerations

- Use pagination when fetching large datasets
- Implement caching for frequently accessed metrics
- Consider aggregating data client-side for complex analytics
- Use date filters to limit the scope of queries
- Monitor API rate limits when processing large volumes