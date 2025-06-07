# EmojiSuggestions (Private API)

> ⚠️ **Warning**: This is a private API that may change without notice. It is not officially supported and should be used with caution in production applications.

The EmojiSuggestions resource provides emoji recommendations based on search terms. It helps users find relevant emojis when creating content, enhancing visual communication and engagement in DatoCMS projects.

## Available Operations

### Find Emoji Suggestions

```javascript
const suggestions = await client.emojiSuggestions.find({ term: "happy" });
```

**Parameters:**
- `term` (string, optional): The search term for emoji suggestions

**Returns:** EmojiSuggestions object containing an array of emojis

## The EmojiSuggestions Object

```typescript
{
  id: string;                    // Suggestions identifier
  type: 'emoji_suggestions';     // Always 'emoji_suggestions'
  emojis: string[];             // Array of emoji characters
}
```

## Common Use Cases

### 1. Content Editor Emoji Picker

```javascript
// Build an emoji picker for content editors
class EmojiPicker {
  constructor(client) {
    this.client = client;
    this.cache = new Map();
  }
  
  async getSuggestions(term) {
    // Cache results to reduce API calls
    if (this.cache.has(term)) {
      return this.cache.get(term);
    }
    
    try {
      const result = await this.client.emojiSuggestions.find({ term });
      this.cache.set(term, result.emojis);
      return result.emojis;
    } catch (error) {
      console.error('Failed to get emoji suggestions:', error);
      return [];
    }
  }
  
  async getCommonEmojis() {
    // Get emojis for common categories
    const categories = [
      { term: 'face', label: 'Emotions' },
      { term: 'food', label: 'Food & Drink' },
      { term: 'nature', label: 'Nature' },
      { term: 'activity', label: 'Activities' },
      { term: 'travel', label: 'Travel' },
      { term: 'object', label: 'Objects' },
      { term: 'symbol', label: 'Symbols' }
    ];
    
    const emojisByCategory = {};
    
    for (const category of categories) {
      const emojis = await this.getSuggestions(category.term);
      emojisByCategory[category.label] = emojis;
    }
    
    return emojisByCategory;
  }
}

// Usage
const picker = new EmojiPicker(client);
const happyEmojis = await picker.getSuggestions('happy');
console.log('Happy emojis:', happyEmojis.join(' '));
```

### 2. Auto-Suggest While Typing

```javascript
// React component for emoji auto-suggestions
function EmojiAutoSuggest({ client }) {
  const [input, setInput] = useState('');
  const [suggestions, setSuggestions] = useState([]);
  const [loading, setLoading] = useState(false);
  
  // Debounced search function
  const searchEmojis = useCallback(
    debounce(async (term) => {
      if (term.length < 2) {
        setSuggestions([]);
        return;
      }
      
      setLoading(true);
      try {
        const result = await client.emojiSuggestions.find({ term });
        setSuggestions(result.emojis.slice(0, 8)); // Limit to 8 suggestions
      } catch (error) {
        console.error('Emoji search failed:', error);
        setSuggestions([]);
      } finally {
        setLoading(false);
      }
    }, 300),
    [client]
  );
  
  useEffect(() => {
    searchEmojis(input);
  }, [input, searchEmojis]);
  
  return (
    <div>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Type to search emojis..."
      />
      
      {loading && <span>Loading...</span>}
      
      {suggestions.length > 0 && (
        <div className="emoji-suggestions">
          {suggestions.map((emoji, index) => (
            <button
              key={index}
              onClick={() => {
                // Insert emoji at cursor position
                document.execCommand('insertText', false, emoji);
                setSuggestions([]);
              }}
            >
              {emoji}
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
```

### 3. Smart Emoji Replacements

```javascript
// Replace text emoticons with emojis
class SmartEmojiReplacer {
  constructor(client) {
    this.client = client;
    this.replacements = new Map([
      [':)', 'smile'],
      [':(', 'sad'],
      [':D', 'laugh'],
      [':P', 'tongue'],
      [';)', 'wink'],
      ['<3', 'heart'],
      [':o', 'surprised']
    ]);
  }
  
  async processText(text) {
    let processedText = text;
    
    for (const [emoticon, term] of this.replacements) {
      if (processedText.includes(emoticon)) {
        try {
          const suggestions = await this.client.emojiSuggestions.find({ term });
          if (suggestions.emojis.length > 0) {
            processedText = processedText.replace(
              new RegExp(escapeRegex(emoticon), 'g'),
              suggestions.emojis[0]
            );
          }
        } catch (error) {
          console.error(`Failed to replace ${emoticon}:`, error);
        }
      }
    }
    
    return processedText;
  }
}

// Helper function to escape regex special characters
function escapeRegex(string) {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// Usage
const replacer = new SmartEmojiReplacer(client);
const processed = await replacer.processText("I'm so happy :) Love this <3");
// Output: "I'm so happy 😊 Love this ❤️"
```

### 4. Emoji Category Browser

```javascript
// Browse emojis by category
async function createEmojiCatalog(client) {
  const categories = {
    emotions: ['happy', 'sad', 'angry', 'love', 'surprised', 'tired'],
    food: ['fruit', 'vegetable', 'meal', 'drink', 'dessert', 'snack'],
    nature: ['animal', 'plant', 'weather', 'landscape', 'flower', 'tree'],
    activities: ['sport', 'game', 'music', 'art', 'travel', 'work'],
    objects: ['tool', 'device', 'clothing', 'household', 'office', 'tech'],
    symbols: ['arrow', 'number', 'punctuation', 'currency', 'math', 'flag']
  };
  
  const catalog = {};
  
  for (const [category, terms] of Object.entries(categories)) {
    catalog[category] = [];
    
    for (const term of terms) {
      try {
        const result = await client.emojiSuggestions.find({ term });
        catalog[category].push({
          term,
          emojis: result.emojis.slice(0, 5) // Top 5 per term
        });
      } catch (error) {
        console.error(`Failed to get emojis for ${term}:`, error);
      }
    }
  }
  
  return catalog;
}

// Display catalog
const catalog = await createEmojiCatalog(client);

Object.entries(catalog).forEach(([category, groups]) => {
  console.log(`\n${category.toUpperCase()}`);
  groups.forEach(({ term, emojis }) => {
    console.log(`  ${term}: ${emojis.join(' ')}`);
  });
});
```

### 5. Content Enhancement Suggestions

```javascript
// Suggest emojis for content enhancement
async function suggestContentEmojis(client, content) {
  // Extract potential keywords from content
  const keywords = extractKeywords(content);
  const suggestions = new Map();
  
  for (const keyword of keywords) {
    try {
      const result = await client.emojiSuggestions.find({ term: keyword });
      if (result.emojis.length > 0) {
        suggestions.set(keyword, result.emojis);
      }
    } catch (error) {
      console.error(`Failed to get suggestions for ${keyword}:`, error);
    }
  }
  
  return suggestions;
}

function extractKeywords(content) {
  // Simple keyword extraction (in production, use NLP library)
  const commonWords = new Set(['the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at']);
  
  const words = content.toLowerCase()
    .split(/\W+/)
    .filter(word => word.length > 3 && !commonWords.has(word));
  
  // Return unique words
  return [...new Set(words)];
}

// Usage
const content = "I love traveling to tropical beaches and eating fresh fruit";
const suggestions = await suggestContentEmojis(client, content);

console.log('Emoji suggestions for your content:');
suggestions.forEach((emojis, keyword) => {
  console.log(`${keyword}: ${emojis.slice(0, 3).join(' ')}`);
});
```

### 6. Emoji Search UI Component

```javascript
// Comprehensive emoji search interface
class EmojiSearchUI {
  constructor(client) {
    this.client = client;
    this.recentSearches = [];
    this.favorites = new Set();
  }
  
  async search(term, options = {}) {
    const { limit = 20, includeRelated = true } = options;
    
    try {
      // Get direct suggestions
      const result = await this.client.emojiSuggestions.find({ term });
      let emojis = result.emojis;
      
      // Get related suggestions if requested
      if (includeRelated && emojis.length < limit) {
        const relatedTerms = this.getRelatedTerms(term);
        
        for (const relatedTerm of relatedTerms) {
          if (emojis.length >= limit) break;
          
          const relatedResult = await this.client.emojiSuggestions.find({ 
            term: relatedTerm 
          });
          
          // Add unique emojis only
          const newEmojis = relatedResult.emojis.filter(e => !emojis.includes(e));
          emojis = emojis.concat(newEmojis);
        }
      }
      
      // Track search
      this.addRecentSearch(term);
      
      return emojis.slice(0, limit);
    } catch (error) {
      console.error('Emoji search failed:', error);
      return [];
    }
  }
  
  getRelatedTerms(term) {
    // Simple related terms mapping
    const relations = {
      happy: ['joy', 'smile', 'laugh'],
      sad: ['cry', 'unhappy', 'tear'],
      food: ['eat', 'meal', 'snack'],
      love: ['heart', 'romance', 'kiss'],
      work: ['office', 'computer', 'business']
    };
    
    return relations[term.toLowerCase()] || [];
  }
  
  addRecentSearch(term) {
    this.recentSearches = [term, ...this.recentSearches.filter(t => t !== term)]
      .slice(0, 10);
  }
  
  toggleFavorite(emoji) {
    if (this.favorites.has(emoji)) {
      this.favorites.delete(emoji);
    } else {
      this.favorites.add(emoji);
    }
  }
  
  getFavorites() {
    return Array.from(this.favorites);
  }
  
  getRecentSearches() {
    return this.recentSearches;
  }
}

// Usage
const emojiUI = new EmojiSearchUI(client);

// Search with options
const results = await emojiUI.search('happy', { 
  limit: 30, 
  includeRelated: true 
});

// Manage favorites
results.forEach(emoji => {
  if (Math.random() > 0.7) { // Randomly favorite some
    emojiUI.toggleFavorite(emoji);
  }
});

console.log('Favorites:', emojiUI.getFavorites());
console.log('Recent searches:', emojiUI.getRecentSearches());
```

## Caching Strategy

Since this is a private API, implement caching to reduce API calls:

```javascript
class CachedEmojiService {
  constructor(client) {
    this.client = client;
    this.cache = new Map();
    this.cacheExpiry = 3600000; // 1 hour
  }
  
  async getSuggestions(term) {
    const cached = this.cache.get(term);
    
    if (cached && Date.now() - cached.timestamp < this.cacheExpiry) {
      return cached.data;
    }
    
    const result = await this.client.emojiSuggestions.find({ term });
    
    this.cache.set(term, {
      data: result.emojis,
      timestamp: Date.now()
    });
    
    return result.emojis;
  }
  
  clearCache() {
    this.cache.clear();
  }
  
  preloadCommonTerms() {
    const commonTerms = [
      'happy', 'sad', 'love', 'food', 'work',
      'sport', 'music', 'travel', 'party', 'sleep'
    ];
    
    return Promise.all(
      commonTerms.map(term => this.getSuggestions(term))
    );
  }
}
```

## Important Considerations

### API Status
- **Private/Deprecated**: This API is not officially supported
- **Subject to Change**: May be modified or removed without notice
- **No SLA**: No service level agreement applies

### Best Practices
1. **Implement Caching**: Reduce API calls with client-side caching
2. **Handle Errors**: Always implement error handling for failed requests
3. **Limit Requests**: Avoid excessive API calls
4. **Fallback Options**: Have backup emoji data for critical features

### Alternatives
Consider these alternatives for production use:
1. Use a client-side emoji library
2. Implement your own emoji search database
3. Use official emoji APIs with proper support

## Error Handling

```javascript
try {
  const suggestions = await client.emojiSuggestions.find({ term: 'test' });
} catch (error) {
  if (error.message.includes('DEPRECATED')) {
    console.warn('Emoji suggestions API is deprecated');
  } else if (error.message.includes('RATE_LIMIT')) {
    console.error('Rate limit exceeded for emoji suggestions');
  } else {
    console.error('Unexpected error:', error.message);
  }
  
  // Fallback to default emojis
  return ['😊', '👍', '❤️', '🎉', '⭐'];
}
```

## Raw Response Access

```javascript
const response = await client.emojiSuggestions.rawFind({ term: 'happy' });
console.log(response.data); // Raw API response with full structure
```

## Related Resources

- [Items](/01-cma-client/01-content-management/item.md) - Content management
- [Fields](/01-cma-client/01-content-management/field.md) - Field configuration
- [Plugins](/01-cma-client/04-site-configuration/plugin.md) - Building custom UI