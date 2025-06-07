# validateManualFieldExtensionParameters Hook

## Purpose

The `validateManualFieldExtensionParameters` hook validates the configuration parameters for manual field extensions. This ensures that field extensions receive valid configuration before being used, preventing runtime errors and providing immediate feedback during configuration.

## Signature

```typescript
validateManualFieldExtensionParameters(
  fieldExtensionId: string,
  parameters: Record<string, unknown>
): Record<string, string> | Promise<Record<string, string>>
```

## Parameters

- `fieldExtensionId`: string - The ID of the field extension being validated
- `parameters`: Record<string, unknown> - The parameters to validate

## Type Definitions

```typescript
// Function signature
validateManualFieldExtensionParameters(
  fieldExtensionId: string,
  parameters: Record<string, unknown>
): Record<string, string> | Promise<Record<string, string>>;

// Parameter types
type FieldExtensionId = string;
type Parameters = Record<string, unknown>;
type ValidationErrors = Record<string, string>;

// Common parameter validation patterns
interface ApiConfigParameters {
  apiEndpoint?: unknown;
  apiKey?: unknown;
  valueField?: unknown;
  labelField?: unknown;
  cacheMinutes?: unknown;
  dataPath?: unknown;
}

interface ValidationRuleParameters {
  rules?: unknown;
  validationTrigger?: unknown;
}

interface RichTextConfigParameters {
  previewMode?: unknown;
  minHeight?: unknown;
  maxHeight?: unknown;
  enabledTools?: unknown;
  fontFamily?: unknown;
  fontSize?: unknown;
  customCSS?: unknown;
}

interface GeoLocationParameters {
  googleMapsApiKey?: unknown;
  defaultLatitude?: unknown;
  defaultLongitude?: unknown;
  defaultZoom?: unknown;
  restrictToCountries?: unknown;
  boundingBox?: unknown;
}

interface ConditionalFieldParameters {
  conditionType?: unknown;
  sourceField?: unknown;
  compareValue?: unknown;
  searchString?: unknown;
  pattern?: unknown;
  customFunction?: unknown;
  showWhen?: unknown;
  hideWhen?: unknown;
  affectedFields?: unknown;
  cascadeValue?: unknown;
  clearOnHide?: unknown;
}

// Validation rule structure
interface ValidationRule {
  type: 'regex' | 'length' | 'custom' | 'async';
  errorMessage: string;
  config?: {
    pattern?: string;
    min?: number;
    max?: number;
    function?: string;
    endpoint?: string;
    method?: string;
  };
}

// Bounding box structure
interface BoundingBox {
  north: number;
  south: number;
  east: number;
  west: number;
}
```

## Return Value

Returns an object containing validation errors, where:
- Keys are parameter names that have errors
- Values are error messages to display to the user
- An empty object means all parameters are valid

```typescript
// Example return values
{} // All parameters valid

{
  "apiEndpoint": "API endpoint is required",
  "cacheMinutes": "Cache duration must be a positive number"
}

{
  "rules[0].type": "Rule type is required",
  "rules[1].pattern": "Invalid regular expression"
}
```

## Use Cases

### 1. API Configuration Validation

Validate external API settings:

```typescript
validateManualFieldExtensionParameters(
  fieldExtensionId: string,
  parameters: Record<string, unknown>
) {
  if (fieldExtensionId === 'externalDataSelector') {
    const errors: Record<string, string> = {};
    
    // Validate API endpoint
    if (!parameters.apiEndpoint) {
      errors.apiEndpoint = 'API endpoint is required';
    } else if (typeof parameters.apiEndpoint !== 'string') {
      errors.apiEndpoint = 'API endpoint must be a string';
    } else {
      try {
        const url = new URL(parameters.apiEndpoint);
        if (!['http:', 'https:'].includes(url.protocol)) {
          errors.apiEndpoint = 'API endpoint must use HTTP or HTTPS';
        }
      } catch (e) {
        errors.apiEndpoint = 'Invalid URL format';
      }
    }
    
    // Validate API key if provided
    if (parameters.apiKey && typeof parameters.apiKey !== 'string') {
      errors.apiKey = 'API key must be a string';
    }
    
    // Validate field mappings
    if (!parameters.valueField) {
      errors.valueField = 'Value field is required';
    } else if (typeof parameters.valueField !== 'string') {
      errors.valueField = 'Value field must be a string';
    }
    
    if (!parameters.labelField) {
      errors.labelField = 'Label field is required';
    } else if (typeof parameters.labelField !== 'string') {
      errors.labelField = 'Label field must be a string';
    }
    
    // Validate cache duration
    if (parameters.cacheMinutes !== undefined) {
      const cache = Number(parameters.cacheMinutes);
      if (isNaN(cache)) {
        errors.cacheMinutes = 'Cache duration must be a number';
      } else if (cache < 0) {
        errors.cacheMinutes = 'Cache duration cannot be negative';
      } else if (cache > 1440) {
        errors.cacheMinutes = 'Cache duration cannot exceed 24 hours (1440 minutes)';
      }
    }
    
    // Validate data path syntax
    if (parameters.dataPath && typeof parameters.dataPath === 'string') {
      const pathRegex = /^[a-zA-Z_$][a-zA-Z0-9_$]*(\.[a-zA-Z_$][a-zA-Z0-9_$]*|\[\d+\])*$/;
      if (!pathRegex.test(parameters.dataPath)) {
        errors.dataPath = 'Invalid data path syntax (e.g., "data.items" or "results[0].data")';
      }
    }
    
    return errors;
  }
}
```

### 2. Complex Validation Rules

Validate advanced validation configurations:

```typescript
async validateManualFieldExtensionParameters(
  fieldExtensionId: string,
  parameters: Record<string, unknown>
) {
  if (fieldExtensionId === 'advancedValidator') {
    const errors: Record<string, string> = {};
    
    // Validate rules array
    if (!parameters.rules) {
      errors.rules = 'At least one validation rule is required';
    } else if (!Array.isArray(parameters.rules)) {
      errors.rules = 'Rules must be an array';
    } else {
      const rules = parameters.rules as any[];
      
      if (rules.length === 0) {
        errors.rules = 'At least one validation rule is required';
      } else {
        // Validate each rule
        rules.forEach((rule, index) => {
          if (!rule.type) {
            errors[`rules[${index}].type`] = 'Rule type is required';
          } else if (!['regex', 'length', 'custom', 'async'].includes(rule.type)) {
            errors[`rules[${index}].type`] = 'Invalid rule type';
          }
          
          if (!rule.errorMessage) {
            errors[`rules[${index}].errorMessage`] = 'Error message is required';
          }
          
          // Type-specific validation
          switch (rule.type) {
            case 'regex':
              if (!rule.config?.pattern) {
                errors[`rules[${index}].pattern`] = 'Regular expression pattern is required';
              } else {
                try {
                  new RegExp(rule.config.pattern);
                } catch (e) {
                  errors[`rules[${index}].pattern`] = 'Invalid regular expression';
                }
              }
              break;
              
            case 'length':
              const min = rule.config?.min;
              const max = rule.config?.max;
              
              if (min !== undefined && (typeof min !== 'number' || min < 0)) {
                errors[`rules[${index}].min`] = 'Minimum length must be a positive number';
              }
              
              if (max !== undefined && (typeof max !== 'number' || max < 0)) {
                errors[`rules[${index}].max`] = 'Maximum length must be a positive number';
              }
              
              if (min !== undefined && max !== undefined && min > max) {
                errors[`rules[${index}].length`] = 'Minimum length cannot exceed maximum length';
              }
              break;
              
            case 'custom':
              if (!rule.config?.function) {
                errors[`rules[${index}].function`] = 'Validation function is required';
              } else {
                try {
                  // Test if it's valid JavaScript
                  new Function('value', 'field', 'item', rule.config.function);
                } catch (e) {
                  errors[`rules[${index}].function`] = 'Invalid JavaScript function';
                }
              }
              break;
              
            case 'async':
              if (!rule.config?.endpoint) {
                errors[`rules[${index}].endpoint`] = 'Validation endpoint is required';
              } else {
                try {
                  new URL(rule.config.endpoint);
                } catch (e) {
                  errors[`rules[${index}].endpoint`] = 'Invalid endpoint URL';
                }
              }
              
              const method = rule.config?.method || 'POST';
              if (!['GET', 'POST', 'PUT'].includes(method)) {
                errors[`rules[${index}].method`] = 'Invalid HTTP method';
              }
              break;
          }
        });
      }
    }
    
    // Validate trigger
    if (parameters.validationTrigger && 
        !['onChange', 'onBlur', 'onSubmit'].includes(parameters.validationTrigger as string)) {
      errors.validationTrigger = 'Invalid validation trigger';
    }
    
    return errors;
  }
}
```

### 3. UI Configuration Validation

Validate rich text editor settings:

```typescript
validateManualFieldExtensionParameters(
  fieldExtensionId: string,
  parameters: Record<string, unknown>
) {
  if (fieldExtensionId === 'customRichText') {
    const errors: Record<string, string> = {};
    
    // Validate preview mode
    if (parameters.previewMode && 
        !['live', 'side', 'tab'].includes(parameters.previewMode as string)) {
      errors.previewMode = 'Invalid preview mode';
    }
    
    // Validate dimensions
    const minHeight = Number(parameters.minHeight);
    const maxHeight = Number(parameters.maxHeight);
    
    if (parameters.minHeight !== undefined) {
      if (isNaN(minHeight)) {
        errors.minHeight = 'Minimum height must be a number';
      } else if (minHeight < 100 || minHeight > 800) {
        errors.minHeight = 'Minimum height must be between 100 and 800 pixels';
      }
    }
    
    if (parameters.maxHeight !== undefined) {
      if (isNaN(maxHeight)) {
        errors.maxHeight = 'Maximum height must be a number';
      } else if (maxHeight < 200 || maxHeight > 1200) {
        errors.maxHeight = 'Maximum height must be between 200 and 1200 pixels';
      }
    }
    
    if (!isNaN(minHeight) && !isNaN(maxHeight) && minHeight > maxHeight) {
      errors.maxHeight = 'Maximum height must be greater than minimum height';
    }
    
    // Validate enabled tools
    if (parameters.enabledTools) {
      if (!Array.isArray(parameters.enabledTools)) {
        errors.enabledTools = 'Enabled tools must be an array';
      } else {
        const validTools = [
          'bold', 'italic', 'underline', 'strike', 'heading',
          'quote', 'code', 'list', 'link', 'image', 'video',
          'table', 'emoji', 'math'
        ];
        
        const invalidTools = (parameters.enabledTools as string[])
          .filter(tool => !validTools.includes(tool));
        
        if (invalidTools.length > 0) {
          errors.enabledTools = `Invalid tools: ${invalidTools.join(', ')}`;
        }
        
        if ((parameters.enabledTools as string[]).length === 0) {
          errors.enabledTools = 'At least one tool must be enabled';
        }
      }
    }
    
    // Validate font settings
    if (parameters.fontFamily && typeof parameters.fontFamily !== 'string') {
      errors.fontFamily = 'Font family must be a string';
    }
    
    if (parameters.fontSize) {
      const fontSizeRegex = /^\d+(px|em|rem|%)$/;
      if (!fontSizeRegex.test(parameters.fontSize as string)) {
        errors.fontSize = 'Invalid font size format (e.g., "16px", "1.2em")';
      }
    }
    
    // Validate custom CSS
    if (parameters.customCSS && typeof parameters.customCSS === 'string') {
      try {
        // Basic CSS validation - check for syntax errors
        const css = parameters.customCSS as string;
        if (css.includes('<script') || css.includes('javascript:')) {
          errors.customCSS = 'CSS cannot contain scripts';
        }
      } catch (e) {
        errors.customCSS = 'Invalid CSS syntax';
      }
    }
    
    return errors;
  }
}
```

### 4. Async Validation with External Services

Validate parameters by checking external services:

```typescript
async validateManualFieldExtensionParameters(
  fieldExtensionId: string,
  parameters: Record<string, unknown>
) {
  if (fieldExtensionId === 'geoLocationPicker') {
    const errors: Record<string, string> = {};
    
    // Validate Google Maps API key
    if (!parameters.googleMapsApiKey) {
      errors.googleMapsApiKey = 'Google Maps API key is required';
    } else if (typeof parameters.googleMapsApiKey !== 'string') {
      errors.googleMapsApiKey = 'API key must be a string';
    } else {
      // Test the API key
      try {
        const response = await fetch(
          `https://maps.googleapis.com/maps/api/geocode/json?address=test&key=${parameters.googleMapsApiKey}`
        );
        const data = await response.json();
        
        if (data.status === 'REQUEST_DENIED') {
          errors.googleMapsApiKey = 'Invalid API key';
        } else if (data.error_message) {
          errors.googleMapsApiKey = data.error_message;
        }
      } catch (e) {
        errors.googleMapsApiKey = 'Failed to validate API key';
      }
    }
    
    // Validate default location
    if (parameters.defaultLatitude !== undefined) {
      const lat = Number(parameters.defaultLatitude);
      if (isNaN(lat) || lat < -90 || lat > 90) {
        errors.defaultLatitude = 'Latitude must be between -90 and 90';
      }
    }
    
    if (parameters.defaultLongitude !== undefined) {
      const lng = Number(parameters.defaultLongitude);
      if (isNaN(lng) || lng < -180 || lng > 180) {
        errors.defaultLongitude = 'Longitude must be between -180 and 180';
      }
    }
    
    // Validate zoom level
    if (parameters.defaultZoom !== undefined) {
      const zoom = Number(parameters.defaultZoom);
      if (isNaN(zoom) || zoom < 1 || zoom > 20) {
        errors.defaultZoom = 'Zoom level must be between 1 and 20';
      }
    }
    
    // Validate map restrictions
    if (parameters.restrictToCountries) {
      if (!Array.isArray(parameters.restrictToCountries)) {
        errors.restrictToCountries = 'Country restrictions must be an array';
      } else {
        const countries = parameters.restrictToCountries as string[];
        const invalidCountries = countries.filter(
          country => typeof country !== 'string' || country.length !== 2
        );
        
        if (invalidCountries.length > 0) {
          errors.restrictToCountries = 'Countries must be 2-letter ISO codes';
        }
      }
    }
    
    if (parameters.boundingBox) {
      const box = parameters.boundingBox as any;
      if (!box.north || !box.south || !box.east || !box.west) {
        errors.boundingBox = 'Bounding box must include north, south, east, and west';
      } else {
        if (box.north <= box.south) {
          errors.boundingBox = 'North must be greater than south';
        }
        if (box.east <= box.west && !(box.west > 0 && box.east < 0)) {
          errors.boundingBox = 'East must be greater than west (unless crossing date line)';
        }
      }
    }
    
    return errors;
  }
}
```

### 5. Cross-Parameter Validation

Validate parameters that depend on each other:

```typescript
validateManualFieldExtensionParameters(
  fieldExtensionId: string,
  parameters: Record<string, unknown>
) {
  if (fieldExtensionId === 'conditionalField') {
    const errors: Record<string, string> = {};
    
    // Validate condition type
    if (!parameters.conditionType) {
      errors.conditionType = 'Condition type is required';
    } else if (!['equals', 'contains', 'matches', 'custom'].includes(parameters.conditionType as string)) {
      errors.conditionType = 'Invalid condition type';
    }
    
    // Validate source field
    if (!parameters.sourceField) {
      errors.sourceField = 'Source field is required';
    }
    
    // Type-specific validation
    switch (parameters.conditionType) {
      case 'equals':
        if (parameters.compareValue === undefined || parameters.compareValue === '') {
          errors.compareValue = 'Comparison value is required for equals condition';
        }
        break;
        
      case 'contains':
        if (!parameters.searchString) {
          errors.searchString = 'Search string is required for contains condition';
        } else if (typeof parameters.searchString !== 'string') {
          errors.searchString = 'Search string must be text';
        }
        break;
        
      case 'matches':
        if (!parameters.pattern) {
          errors.pattern = 'Pattern is required for matches condition';
        } else {
          try {
            new RegExp(parameters.pattern as string);
          } catch (e) {
            errors.pattern = 'Invalid regular expression pattern';
          }
        }
        break;
        
      case 'custom':
        if (!parameters.customFunction) {
          errors.customFunction = 'Custom function is required';
        } else {
          try {
            new Function('value', 'formValues', parameters.customFunction as string);
          } catch (e) {
            errors.customFunction = 'Invalid JavaScript function';
          }
        }
        break;
    }
    
    // Validate actions
    if (!parameters.showWhen && !parameters.hideWhen) {
      errors.action = 'Either showWhen or hideWhen must be specified';
    }
    
    if (parameters.showWhen && parameters.hideWhen) {
      errors.action = 'Cannot specify both showWhen and hideWhen';
    }
    
    // Validate affected fields
    if (parameters.affectedFields) {
      if (!Array.isArray(parameters.affectedFields)) {
        errors.affectedFields = 'Affected fields must be an array';
      } else if ((parameters.affectedFields as string[]).length === 0) {
        errors.affectedFields = 'At least one affected field must be specified';
      }
    }
    
    // Validate cascade behavior
    if (parameters.cascadeValue && parameters.clearOnHide === false) {
      errors.cascadeValue = 'Cannot cascade value when clearOnHide is disabled';
    }
    
    return errors;
  }
}
```

## Best Practices

1. **Clear Error Messages**: Provide specific, actionable error messages
2. **Type Checking**: Always validate parameter types before using them
3. **Range Validation**: Check numeric values are within acceptable ranges
4. **Format Validation**: Use regex or parsing to validate formats
5. **Dependency Checking**: Validate parameters that depend on each other
6. **Async Validation**: Use async validation sparingly for external checks
7. **Early Return**: Return errors as soon as found for better performance
8. **Comprehensive Coverage**: Validate all parameters that could cause issues
9. **User-Friendly**: Write errors that non-technical users can understand
10. **Consistent Naming**: Use consistent error key naming patterns

## Related Hooks

- `manualFieldExtensions` - Define which field extensions your plugin provides
- `renderManualFieldExtensionConfigScreen` - Render configuration UI
- `renderFieldExtension` - Render the field extension itself
- `overrideFieldExtensions` - Override default field editors

## Important Notes

- Validation runs whenever parameters change in the configuration screen
- Return an empty object `{}` if all parameters are valid
- Async validation should be used sparingly to avoid UI delays
- Error messages appear next to the relevant configuration fields
- Validation prevents saving invalid configurations
- Consider providing helpful suggestions in error messages