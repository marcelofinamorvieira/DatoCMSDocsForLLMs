# Upload Track

Upload Tracks allow you to add subtitles, captions, and alternate audio tracks to video and audio files in DatoCMS. This enables better accessibility and multi-language support for your media content.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Add subtitles to a video
const job = await client.uploadTracks.create('video-upload-id', {
  type: 'subtitles',
  url_or_upload_request_id: 'https://example.com/subtitles-en.vtt',
  language_code: 'en',
  name: 'English Subtitles'
});

// Wait for processing
const result = await client.jobResults.wait(job.id);
```

## API Reference

### Create Upload Track

Add a subtitle or audio track to a video/audio upload. This operation is asynchronous.

```typescript
// Add subtitles from URL
const subtitleJob = await client.uploadTracks.create('video-upload-id', {
  type: 'subtitles',
  url_or_upload_request_id: 'https://example.com/captions-en.vtt',
  name: 'English',
  language_code: 'en',
  closed_captions: true // Mark as SDH/closed captions
});

// Add subtitles from uploaded file
// First, create an upload request
const uploadRequest = await client.uploadRequest.create({
  filename: 'spanish-subtitles.srt'
});

// Upload the file to S3 (implement actual upload)
// Then create the track
const trackJob = await client.uploadTracks.create('video-upload-id', {
  type: 'subtitles',
  url_or_upload_request_id: uploadRequest.id,
  name: 'Español',
  language_code: 'es'
});

// Add alternate audio track
const audioJob = await client.uploadTracks.create('video-upload-id', {
  type: 'audio',
  url_or_upload_request_id: 'https://example.com/french-audio.mp3',
  name: 'French Audio',
  language_code: 'fr'
});
```

**Parameters:**
- `uploadId` (string, required): The video/audio upload ID
- `type` (string, required): Track type - 'subtitles' or 'audio'
- `url_or_upload_request_id` (string, required): URL to download or upload request ID
- `name` (string, optional): Human-readable track name
- `language_code` (string, required): BCP 47 language code (e.g., 'en', 'es', 'fr-CA')
- `closed_captions` (boolean, optional): For subtitles - indicates SDH/closed captions

**Returns:** Job object for tracking the async operation

**Supported Subtitle Formats:**
- WebVTT (.vtt)
- SubRip (.srt)
- TTML (.ttml, .dfxp)

### List Upload Tracks

Get all tracks associated with an upload.

```typescript
const tracks = await client.uploadTracks.list('video-upload-id');

// Group by type
const subtitles = tracks.filter(t => t.type === 'subtitles');
const audioTracks = tracks.filter(t => t.type === 'audio');

// List available languages
const languages = tracks.map(t => ({
  code: t.language_code,
  name: t.name,
  type: t.type
}));

console.log('Available tracks:');
languages.forEach(lang => {
  console.log(`- ${lang.name} (${lang.code}) - ${lang.type}`);
});
```

**Parameters:**
- `uploadId` (string, required): The upload ID

**Returns:** Array of upload track objects

### Delete Upload Track

Remove a track from an upload. This operation is asynchronous.

```typescript
const job = await client.uploadTracks.destroy(
  'video-upload-id',
  'track-id'
);

// Wait for deletion
await client.jobResults.wait(job.id);
console.log('Track removed');
```

**Parameters:**
- `uploadId` (string, required): The upload ID
- `uploadTrackId` (string, required): The track ID to delete

**Returns:** Job object for tracking the async operation

## Working with Subtitles

### Multi-Language Subtitles

Add subtitles in multiple languages:

```typescript
async function addMultiLanguageSubtitles(
  videoId: string,
  subtitleFiles: Array<{
    url: string;
    language: string;
    name: string;
    isSDH?: boolean;
  }>
) {
  const jobs = [];
  
  for (const subtitle of subtitleFiles) {
    const job = await client.uploadTracks.create(videoId, {
      type: 'subtitles',
      url_or_upload_request_id: subtitle.url,
      name: subtitle.name,
      language_code: subtitle.language,
      closed_captions: subtitle.isSDH || false
    });
    
    jobs.push(job);
  }
  
  // Wait for all tracks to be processed
  const results = await Promise.all(
    jobs.map(job => client.jobResults.wait(job.id))
  );
  
  return results;
}

// Usage
await addMultiLanguageSubtitles('video-id', [
  { url: 'https://cdn.example.com/en.vtt', language: 'en', name: 'English' },
  { url: 'https://cdn.example.com/es.vtt', language: 'es', name: 'Español' },
  { url: 'https://cdn.example.com/fr.vtt', language: 'fr', name: 'Français' },
  { url: 'https://cdn.example.com/de.vtt', language: 'de', name: 'Deutsch' },
  { url: 'https://cdn.example.com/en-sdh.vtt', language: 'en', name: 'English (CC)', isSDH: true }
]);
```

### WebVTT Subtitle Format

Example WebVTT file format:

```vtt
WEBVTT

00:00:00.000 --> 00:00:04.000
Welcome to our product demonstration.

00:00:04.500 --> 00:00:08.000
Today we'll show you the key features.

00:00:08.500 --> 00:00:12.000
Let's start with the dashboard.
```

### SRT Subtitle Format

Example SRT file format:

```srt
1
00:00:00,000 --> 00:00:04,000
Welcome to our product demonstration.

2
00:00:04,500 --> 00:00:08,000
Today we'll show you the key features.

3
00:00:08,500 --> 00:00:12,000
Let's start with the dashboard.
```

## Working with Audio Tracks

### Multi-Language Audio

Add alternate audio tracks for different languages:

```typescript
async function addMultiLanguageAudio(
  videoId: string,
  audioTracks: Array<{
    url: string;
    language: string;
    name: string;
  }>
) {
  for (const audio of audioTracks) {
    const job = await client.uploadTracks.create(videoId, {
      type: 'audio',
      url_or_upload_request_id: audio.url,
      name: audio.name,
      language_code: audio.language
    });
    
    await client.jobResults.wait(job.id);
    console.log(`Added ${audio.name} audio track`);
  }
}

// Add dubbed versions
await addMultiLanguageAudio('video-id', [
  { url: 'https://cdn.example.com/audio-es.mp3', language: 'es', name: 'Spanish Dub' },
  { url: 'https://cdn.example.com/audio-fr.mp3', language: 'fr', name: 'French Dub' },
  { url: 'https://cdn.example.com/audio-pt.mp3', language: 'pt', name: 'Portuguese Dub' }
]);
```

### Audio Description Track

Add audio description for accessibility:

```typescript
// Add audio description track
await client.uploadTracks.create('video-id', {
  type: 'audio',
  url_or_upload_request_id: 'https://cdn.example.com/audio-description.mp3',
  name: 'Audio Description',
  language_code: 'en'
});
```

## Track Management Patterns

### Bulk Track Upload

Process multiple videos with tracks:

```typescript
async function bulkAddSubtitles(
  videoSubtitlePairs: Array<{
    videoId: string;
    subtitleUrl: string;
    language: string;
  }>
) {
  const jobs = [];
  
  for (const pair of videoSubtitlePairs) {
    try {
      const job = await client.uploadTracks.create(pair.videoId, {
        type: 'subtitles',
        url_or_upload_request_id: pair.subtitleUrl,
        language_code: pair.language,
        name: getLanguageName(pair.language)
      });
      
      jobs.push({
        videoId: pair.videoId,
        language: pair.language,
        job
      });
    } catch (error) {
      console.error(`Failed to add subtitle for video ${pair.videoId}:`, error);
    }
  }
  
  // Monitor progress
  for (const { videoId, language, job } of jobs) {
    try {
      await client.jobResults.wait(job.id);
      console.log(`✓ Added ${language} subtitles to ${videoId}`);
    } catch (error) {
      console.error(`✗ Failed ${language} subtitles for ${videoId}`);
    }
  }
}

function getLanguageName(code: string): string {
  const languages = {
    'en': 'English',
    'es': 'Español',
    'fr': 'Français',
    'de': 'Deutsch',
    'it': 'Italiano',
    'pt': 'Português',
    'ja': '日本語',
    'ko': '한국어',
    'zh': '中文'
  };
  return languages[code] || code;
}
```

### Track Synchronization

Keep tracks synchronized across video versions:

```typescript
async function syncTracksToNewVideo(
  sourceVideoId: string,
  targetVideoId: string
) {
  // Get tracks from source video
  const sourceTracks = await client.uploadTracks.list(sourceVideoId);
  
  console.log(`Copying ${sourceTracks.length} tracks...`);
  
  // Copy each track to target video
  for (const track of sourceTracks) {
    const job = await client.uploadTracks.create(targetVideoId, {
      type: track.type,
      url_or_upload_request_id: track.url, // Assuming track has URL
      name: track.name,
      language_code: track.language_code,
      closed_captions: track.closed_captions
    });
    
    await client.jobResults.wait(job.id);
    console.log(`Copied ${track.type} track: ${track.name}`);
  }
}
```

### Accessibility Compliance

Ensure videos meet accessibility requirements:

```typescript
async function checkAccessibility(videoId: string): Promise<{
  compliant: boolean;
  missing: string[];
}> {
  const tracks = await client.uploadTracks.list(videoId);
  const missing = [];
  
  // Check for English subtitles
  const hasEnglishSubtitles = tracks.some(t => 
    t.type === 'subtitles' && t.language_code === 'en'
  );
  if (!hasEnglishSubtitles) missing.push('English subtitles');
  
  // Check for closed captions
  const hasClosedCaptions = tracks.some(t => 
    t.type === 'subtitles' && t.closed_captions
  );
  if (!hasClosedCaptions) missing.push('Closed captions (SDH)');
  
  // Check for audio description
  const hasAudioDescription = tracks.some(t => 
    t.type === 'audio' && 
    t.name?.toLowerCase().includes('description')
  );
  if (!hasAudioDescription) missing.push('Audio description');
  
  return {
    compliant: missing.length === 0,
    missing
  };
}

// Check all videos
async function auditVideoAccessibility() {
  const videos = await client.uploads.list({
    filter: { type: 'video' }
  });
  
  const nonCompliant = [];
  
  for (const video of videos) {
    const check = await checkAccessibility(video.id);
    if (!check.compliant) {
      nonCompliant.push({
        video: video.filename,
        missing: check.missing
      });
    }
  }
  
  return nonCompliant;
}
```

## Advanced Usage

### Track Validation

Validate tracks before upload:

```typescript
async function validateSubtitleFile(url: string): Promise<{
  valid: boolean;
  format?: string;
  errors?: string[];
}> {
  try {
    // Fetch subtitle file
    const response = await fetch(url);
    const content = await response.text();
    
    // Check format
    if (content.startsWith('WEBVTT')) {
      return { valid: true, format: 'WebVTT' };
    } else if (content.match(/^\d+\s*\n\d{2}:\d{2}:\d{2},\d{3}/m)) {
      return { valid: true, format: 'SRT' };
    } else if (content.includes('<?xml') && content.includes('<tt')) {
      return { valid: true, format: 'TTML' };
    }
    
    return {
      valid: false,
      errors: ['Unrecognized subtitle format']
    };
  } catch (error) {
    return {
      valid: false,
      errors: ['Failed to fetch subtitle file']
    };
  }
}
```

### Language Detection

Auto-detect language from subtitle content:

```typescript
async function detectSubtitleLanguage(url: string): Promise<string | null> {
  // This is a simplified example
  // In production, use a language detection library
  
  const response = await fetch(url);
  const content = await response.text();
  
  // Extract text from subtitles
  const textOnly = content
    .replace(/\d{2}:\d{2}:\d{2}[,.]\d{3}\s*-->\s*\d{2}:\d{2}:\d{2}[,.]\d{3}/g, '')
    .replace(/^\d+$/gm, '')
    .replace(/WEBVTT.*$/m, '')
    .trim();
  
  // Simple language detection based on common words
  const languagePatterns = {
    'en': /\b(the|and|is|of|to|in|for|with|on|at)\b/gi,
    'es': /\b(el|la|de|que|y|en|un|por|con|para)\b/gi,
    'fr': /\b(le|de|et|la|les|des|un|une|pour|dans)\b/gi,
    'de': /\b(der|die|und|in|das|ist|von|mit|den|für)\b/gi
  };
  
  let bestMatch = { lang: 'en', score: 0 };
  
  for (const [lang, pattern] of Object.entries(languagePatterns)) {
    const matches = textOnly.match(pattern) || [];
    if (matches.length > bestMatch.score) {
      bestMatch = { lang, score: matches.length };
    }
  }
  
  return bestMatch.lang;
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.uploadTracks.create('video-id', {
    type: 'subtitles',
    url_or_upload_request_id: 'invalid-url',
    language_code: 'invalid-code'
  });
} catch (error) {
  if (error instanceof ApiError) {
    // Invalid language code
    const langError = error.findError('language_code');
    if (langError?.code === 'VALIDATION_FORMAT') {
      console.log('Invalid language code - use BCP 47 format');
    }
    
    // Invalid URL
    const urlError = error.findError('url_or_upload_request_id');
    if (urlError?.code === 'VALIDATION_FORMAT') {
      console.log('Invalid URL or upload request ID');
    }
  }
}

// Handle job failures
const job = await client.uploadTracks.create('video-id', {
  type: 'subtitles',
  url_or_upload_request_id: 'https://example.com/broken.vtt',
  language_code: 'en'
});

try {
  await client.jobResults.wait(job.id);
} catch (error) {
  console.error('Track processing failed:', error);
  // Check job error details
}
```

## Upload Track Object Structure

```typescript
interface UploadTrack {
  id: string;
  type: 'subtitles' | 'audio';
  name?: string;
  language_code: string;
  closed_captions?: boolean;
  url: string;
  upload: {
    type: 'upload';
    id: string;
  };
}

// Language codes follow BCP 47 standard:
// 'en' - English
// 'en-US' - English (United States)
// 'en-GB' - English (United Kingdom)
// 'es' - Spanish
// 'es-MX' - Spanish (Mexico)
// 'fr-CA' - French (Canada)
// etc.
```

## Related Resources

- [Uploads](./upload.md) - Upload video and audio files
- Job Results - Track async operations
- Transform Assets - Video transformations