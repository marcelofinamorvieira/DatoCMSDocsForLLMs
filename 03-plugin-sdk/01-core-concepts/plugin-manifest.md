# Plugin Manifest

The plugin manifest defines your plugin's capabilities, requirements, and configuration options. It's specified in your `package.json` file under the `datoCmsPlugin` key.

## Complete Manifest Structure

```json
{
  "name": "datocms-plugin-example",
  "version": "1.0.0",
  "description": "An example DatoCMS plugin",
  "datoCmsPlugin": {
    "title": "Example Plugin",
    "previewImage": "docs/preview.png",
    "coverImage": "docs/cover.png",
    "entryPoint": "./dist/index.js",
    "pluginType": "field_editor",
    "fieldTypes": ["string", "text", "json"],
    "parameters": {
      "global": [
        {
          "id": "apiKey",
          "label": "API Key",
          "type": "string",
          "required": true,
          "default": "",
          "hint": "Your service API key"
        }
      ],
      "instance": [
        {
          "id": "placeholder",
          "label": "Placeholder text",
          "type": "string",
          "required": false,
          "default": "Enter text..."
        }
      ]
    }
  }
}
```

## Manifest Properties

### Basic Information

#### title (required)
The display name of your plugin in the DatoCMS interface.

```json
"title": "SEO Assistant"
```

#### previewImage (required)
Preview image shown in the plugin marketplace. Should be 688x344px.

```json
"previewImage": "docs/preview.png"
```

#### coverImage (optional)
Cover image for the plugin detail page. Should be 1440x480px.

```json
"coverImage": "docs/cover.png"
```

#### entryPoint (required)
Path to your compiled plugin code.

```json
"entryPoint": "./dist/index.js"
```

### Plugin Type

#### pluginType (required)
Defines what type of plugin this is:

- `field_editor`: Custom field editor
- `field_addon`: Field sidebar addon
- `sidebar`: Item form sidebar

```json
"pluginType": "field_editor"
```

### Field Types

#### fieldTypes (required for field_editor)
Array of field types this plugin can handle:

```json
"fieldTypes": [
  "string",
  "text", 
  "json",
  "float",
  "integer",
  "boolean",
  "date",
  "date_time",
  "color",
  "seo",
  "link",
  "links",
  "file",
  "gallery",
  "lat_lon",
  "structured_text"
]
```

### Parameters

Parameters allow users to configure your plugin. They're divided into:

#### global
Settings that apply to the entire plugin installation:

```json
"global": [
  {
    "id": "apiEndpoint",
    "label": "API Endpoint",
    "type": "string",
    "required": true,
    "default": "https://api.example.com",
    "hint": "The base URL for API requests"
  },
  {
    "id": "debugMode",
    "label": "Enable Debug Mode",
    "type": "boolean",
    "required": false,
    "default": false
  }
]
```

#### instance
Settings specific to each field using the plugin:

```json
"instance": [
  {
    "id": "minLength",
    "label": "Minimum Length",
    "type": "integer",
    "required": false,
    "default": 0,
    "hint": "Minimum character count"
  },
  {
    "id": "maxLength",
    "label": "Maximum Length",
    "type": "integer",
    "required": false,
    "default": 1000
  }
]
```

## Parameter Types

### string
Text input field:

```json
{
  "id": "apiKey",
  "label": "API Key",
  "type": "string",
  "required": true,
  "default": "",
  "hint": "Enter your API key"
}
```

### text
Multi-line text input:

```json
{
  "id": "template",
  "label": "Template",
  "type": "text",
  "required": false,
  "default": "Hello {{name}}!"
}
```

### boolean
Checkbox/toggle:

```json
{
  "id": "showPreview",
  "label": "Show Preview",
  "type": "boolean",
  "required": false,
  "default": true
}
```

### integer
Whole number input:

```json
{
  "id": "maxItems",
  "label": "Maximum Items",
  "type": "integer",
  "required": false,
  "default": 10,
  "hint": "Max number of items to display"
}
```

### float
Decimal number input:

```json
{
  "id": "aspectRatio",
  "label": "Aspect Ratio",
  "type": "float",
  "required": false,
  "default": 1.777,
  "hint": "Width/Height ratio (e.g., 1.777 for 16:9)"
}
```

### color
Color picker:

```json
{
  "id": "highlightColor",
  "label": "Highlight Color",
  "type": "color",
  "required": false,
  "default": {
    "red": 255,
    "green": 127,
    "blue": 0,
    "alpha": 255
  }
}
```

### json
JSON editor:

```json
{
  "id": "configuration",
  "label": "Advanced Configuration",
  "type": "json",
  "required": false,
  "default": {
    "options": {
      "enabled": true,
      "level": 1
    }
  }
}
```

### enum
Dropdown selection:

```json
{
  "id": "theme",
  "label": "Theme",
  "type": "enum",
  "required": false,
  "default": "light",
  "hint": "Choose a theme",
  "enumValues": [
    { "value": "light", "label": "Light" },
    { "value": "dark", "label": "Dark" },
    { "value": "auto", "label": "Auto" }
  ]
}
```

### date
Date picker:

```json
{
  "id": "startDate",
  "label": "Start Date",
  "type": "date",
  "required": false,
  "default": "2024-01-01"
}
```

### date_time
Date and time picker:

```json
{
  "id": "publishAt",
  "label": "Publish At",
  "type": "date_time",
  "required": false,
  "default": "2024-01-01T09:00:00Z"
}
```

## Accessing Parameters in Code

Parameters are available in the plugin context:

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    // Global parameters
    const apiKey = ctx.plugin.attributes.parameters.apiKey;
    const debugMode = ctx.plugin.attributes.parameters.debugMode;
    
    // Instance parameters
    const minLength = ctx.parameters.minLength;
    const maxLength = ctx.parameters.maxLength;
    
    console.log('Plugin config:', {
      apiKey,
      debugMode,
      minLength,
      maxLength
    });
  }
});
```

## Advanced Configuration

### Multiple Field Types

Support multiple field types with type-specific logic:

```json
"fieldTypes": ["string", "text", "structured_text"],
```

```typescript
connect({
  renderFieldExtension(fieldExtensionId, ctx) {
    switch (ctx.field.attributes.field_type) {
      case 'string':
        // Render string field editor
        break;
      case 'text':
        // Render text field editor
        break;
      case 'structured_text':
        // Render structured text editor
        break;
    }
  }
});
```

### Conditional Parameters

You can't conditionally show/hide parameters, but you can validate them:

```typescript
connect({
  validateManualFieldExtensionParameters(ctx) {
    const { minLength, maxLength } = ctx.parameters;
    
    if (minLength && maxLength && minLength > maxLength) {
      return {
        minLength: 'Must be less than maximum length',
        maxLength: 'Must be greater than minimum length'
      };
    }
    
    return {}; // No errors
  }
});
```

## Best Practices

1. **Provide defaults**: Always include sensible default values
2. **Use hints**: Add helpful hints to explain complex parameters
3. **Validate early**: Use parameter validation to catch configuration errors
4. **Keep it simple**: Don't overwhelm users with too many options
5. **Document parameters**: Explain each parameter in your plugin documentation

## Example: Complete SEO Plugin Manifest

```json
{
  "name": "datocms-plugin-seo-helper",
  "version": "2.0.0",
  "description": "SEO optimization assistant for DatoCMS",
  "datoCmsPlugin": {
    "title": "SEO Helper",
    "previewImage": "docs/preview.png",
    "coverImage": "docs/cover.png",
    "entryPoint": "./dist/index.js",
    "pluginType": "field_addon",
    "fieldTypes": ["string", "text", "seo"],
    "parameters": {
      "global": [
        {
          "id": "serpApiKey",
          "label": "SERP API Key",
          "type": "string",
          "required": true,
          "hint": "API key for search preview service"
        },
        {
          "id": "targetLocale",
          "label": "Target Locale",
          "type": "enum",
          "required": false,
          "default": "en-US",
          "enumValues": [
            { "value": "en-US", "label": "English (US)" },
            { "value": "en-GB", "label": "English (UK)" },
            { "value": "de-DE", "label": "German" }
          ]
        }
      ],
      "instance": [
        {
          "id": "minTitleLength",
          "label": "Minimum Title Length",
          "type": "integer",
          "required": false,
          "default": 30,
          "hint": "Recommended minimum characters for SEO"
        },
        {
          "id": "maxTitleLength",
          "label": "Maximum Title Length",
          "type": "integer",
          "required": false,
          "default": 60,
          "hint": "Maximum characters before truncation"
        },
        {
          "id": "keywords",
          "label": "Target Keywords",
          "type": "json",
          "required": false,
          "default": [],
          "hint": "Keywords to check in content"
        }
      ]
    }
  }
}
```

## Related Documentation

- [Context Object](./context-object.md)
- [Plugin Development Guide](../README.md)
- [Parameter Validation](../02-hooks/validateManualFieldExtensionParameters.md)