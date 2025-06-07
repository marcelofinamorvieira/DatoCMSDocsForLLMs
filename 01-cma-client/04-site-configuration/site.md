# Site Resource

## Overview

The Site resource (`client.site`) provides access to project-level settings and configuration. This is a singleton resource - each project has exactly one site object that contains global settings, localization configuration, timezone, and other project-wide properties.

**Resource Type**: `site`  
**JSON:API Type**: `site`

## Available Methods

### Find Site

Retrieves the site configuration (singleton resource).

```typescript
const site = await client.site.find();

// Returns:
{
  id: '12345',
  type: 'site',
  name: 'My DatoCMS Project',
  domain: 'my-project-12345.admin.datocms.com',
  internal_subdomain: 'my-project-12345',
  google_tag_manager_id: null,
  locales: ['en', 'it', 'es'],
  timezone: 'Europe/Rome',
  favicon: { type: 'upload', id: '67890' },
  require_2fa: false,
  ip_whitelist: null,
  frontend_url: 'https://www.example.com',
  sso_settings: null,
  seo_settings: {
    title_suffix: ' - My Company',
    fallback_seo: {
      title: 'My Company Website',
      description: 'Welcome to our website',
      image: null,
      twitter_account: '@mycompany'
    }
  }
}
```

### Update Site

Updates the site configuration.

```typescript
const updatedSite = await client.site.update({
  name: 'Updated Project Name',
  locales: ['en', 'it', 'es', 'fr'],
  timezone: 'America/New_York',
  frontend_url: 'https://www.new-example.com'
});
```

**Parameters:**

```typescript
interface SimpleSiteUpdateSchema {
  name?: string;                              // Project name
  locales?: string[];                         // ISO locale codes
  timezone?: string;                          // IANA timezone
  google_tag_manager_id?: string | null;      // GTM container ID
  global_meta_tags?: Array<{                  // Global meta tags
    tag_name: string;
    attributes: Record<string, string>;
    content?: string;
  }>;
  favicon?: { type: 'upload'; id: string } | null;  // Favicon upload
  no_index?: boolean;                         // Disable search indexing
  frontend_url?: string | null;               // Production URL
  require_2fa?: boolean;                      // Require 2FA for all users
  ip_whitelist?: string[] | null;             // IP address restrictions
  sso_settings?: SsoSettings | null;          // SSO configuration
  seo_settings?: {                           // SEO defaults
    title_suffix?: string | null;
    fallback_seo?: {
      title?: string | null;
      description?: string | null;
      image?: { type: 'upload'; id: string } | null;
      twitter_account?: string | null;
    } | null;
  };
}
```

## Site Properties

### Basic Information

| Property | Type | Description |
|----------|------|-------------|
| `name` | string | Project display name |
| `domain` | string | Admin panel domain |
| `internal_subdomain` | string | Unique project identifier |
| `frontend_url` | string \| null | Production website URL |

### Localization

```typescript
interface LocalizationSettings {
  locales: string[];          // Array of ISO locale codes
  timezone: string;           // IANA timezone identifier
}

// Example: Multi-language setup
await client.site.update({
  locales: ['en-US', 'en-GB', 'es-ES', 'fr-FR', 'de-DE'],
  timezone: 'UTC'
});
```

### Available Locales

DatoCMS supports all standard ISO 639-1 language codes with optional region codes:
- Simple: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `zh`
- With region: `en-US`, `en-GB`, `es-ES`, `es-MX`, `pt-BR`, `zh-CN`

### Security Settings

```typescript
// Enable 2FA requirement
await client.site.update({
  require_2fa: true
});

// Set IP whitelist
await client.site.update({
  ip_whitelist: [
    '192.168.1.0/24',
    '10.0.0.0/8',
    '172.16.0.0/12'
  ]
});
```

### SEO Configuration

```typescript
await client.site.update({
  seo_settings: {
    title_suffix: ' | My Brand',
    fallback_seo: {
      title: 'Welcome to My Brand',
      description: 'Default meta description for all pages',
      image: { type: 'upload', id: 'default-og-image-id' },
      twitter_account: '@mybrand'
    }
  }
});
```

### Global Meta Tags

```typescript
await client.site.update({
  global_meta_tags: [
    {
      tag_name: 'meta',
      attributes: {
        name: 'viewport',
        content: 'width=device-width, initial-scale=1.0'
      }
    },
    {
      tag_name: 'meta',
      attributes: {
        property: 'og:site_name',
        content: 'My Brand'
      }
    },
    {
      tag_name: 'link',
      attributes: {
        rel: 'preconnect',
        href: 'https://fonts.googleapis.com'
      }
    }
  ]
});
```

## Common Patterns

### Initial Project Setup

```typescript
async function setupProject(config: {
  name: string;
  locales: string[];
  timezone: string;
  production_url?: string;
}) {
  // 1. Update basic settings
  const site = await client.site.update({
    name: config.name,
    locales: config.locales,
    timezone: config.timezone,
    frontend_url: config.production_url
  });
  
  // 2. Upload and set favicon
  const faviconUpload = await client.uploads.createFromUrl({
    url: 'https://example.com/favicon.ico',
    default_field_metadata: {
      en: {
        alt: 'Site favicon',
        title: config.name
      }
    }
  });
  
  await client.site.update({
    favicon: { type: 'upload', id: faviconUpload.id }
  });
  
  // 3. Configure SEO defaults
  await client.site.update({
    seo_settings: {
      title_suffix: ` - ${config.name}`,
      fallback_seo: {
        title: config.name,
        description: `Welcome to ${config.name}`
      }
    }
  });
  
  return site;
}
```

### Multi-Environment Configuration

```typescript
async function configureEnvironment(environment: string) {
  const baseConfig = await client.site.find();
  
  switch(environment) {
    case 'main':
      await client.site.update({
        frontend_url: 'https://www.production.com',
        no_index: false,
        require_2fa: true
      });
      break;
      
    case 'staging':
      await client.site.update({
        frontend_url: 'https://staging.production.com',
        no_index: true,  // Don't index staging
        require_2fa: false
      });
      break;
      
    case 'development':
      await client.site.update({
        frontend_url: 'http://localhost:3000',
        no_index: true,
        require_2fa: false,
        ip_whitelist: null  // No restrictions in dev
      });
      break;
  }
}
```

### Locale Management

```typescript
async function addLocale(newLocale: string) {
  const site = await client.site.find();
  
  if (site.locales.includes(newLocale)) {
    console.log('Locale already exists');
    return;
  }
  
  // Add new locale
  await client.site.update({
    locales: [...site.locales, newLocale]
  });
  
  // Update all models to support new locale
  const models = await client.itemTypes.list();
  
  for (const model of models) {
    const fields = await client.fields.list(model.id);
    
    for (const field of fields) {
      if (field.localized) {
        // Field already supports localization
        console.log(`Field ${field.api_key} in ${model.api_key} ready for ${newLocale}`);
      }
    }
  }
}
```

### SSO Configuration

```typescript
// SAML SSO setup
await client.site.update({
  sso_settings: {
    type: 'saml',
    enabled: true,
    issuer: 'https://www.datocms.com',
    entry_point: 'https://idp.example.com/sso/saml',
    certificate: '-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----',
    name_id_format: 'urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress',
    first_name_attribute: 'firstName',
    last_name_attribute: 'lastName',
    email_attribute: 'email',
    groups_attribute: 'groups'
  }
});

// OIDC SSO setup
await client.site.update({
  sso_settings: {
    type: 'oidc',
    enabled: true,
    client_id: 'datocms-client',
    client_secret: 'secret',
    issuer_url: 'https://idp.example.com',
    first_name_attribute: 'given_name',
    last_name_attribute: 'family_name',
    email_attribute: 'email',
    groups_attribute: 'groups'
  }
});
```

## Site Statistics

### Usage Information

The site object includes usage statistics when requested:

```typescript
const site = await client.site.find({
  include: 'usage'
});

// Additional usage data available:
{
  // ... standard site fields
  usage: {
    uploads_count: 1234,
    uploads_size: 5368709120,  // bytes
    items_count: 5678,
    users_count: 10,
    roles_count: 5,
    locales_count: 3,
    build_triggers_count: 2
  }
}
```

## Response Format

### Simplified Response

```typescript
interface Site {
  id: string;
  type: 'site';
  name: string;
  domain: string;
  internal_subdomain: string;
  google_tag_manager_id: string | null;
  locales: string[];
  timezone: string;
  global_meta_tags: Array<{
    tag_name: string;
    attributes: Record<string, string>;
    content?: string;
  }>;
  favicon: { type: 'upload'; id: string } | null;
  no_index: boolean;
  frontend_url: string | null;
  require_2fa: boolean;
  ip_whitelist: string[] | null;
  sso_settings: SsoSettings | null;
  seo_settings: {
    title_suffix: string | null;
    fallback_seo: {
      title: string | null;
      description: string | null;
      image: { type: 'upload'; id: string } | null;
      twitter_account: string | null;
    } | null;
  };
  created_at: string;  // ISO 8601
  updated_at: string;  // ISO 8601
}
```

### Raw JSON:API Response

```typescript
interface SiteResponse {
  data: {
    type: 'site';
    id: string;
    attributes: {
      name: string;
      domain: string;
      internal_subdomain: string;
      google_tag_manager_id: string | null;
      locales: string[];
      timezone: string;
      global_meta_tags: Array<GlobalMetaTag>;
      no_index: boolean;
      frontend_url: string | null;
      require_2fa: boolean;
      ip_whitelist: string[] | null;
      sso_settings: SsoSettings | null;
      seo_settings: SeoSettings;
      created_at: string;
      updated_at: string;
    };
    relationships: {
      favicon: {
        data: { type: 'upload'; id: string } | null;
      };
    };
  };
}
```

## Error Handling

Common errors when working with site settings:

```typescript
try {
  await client.site.update({
    locales: []  // Empty locales array
  });
} catch (error) {
  // ApiError with:
  // - 422: At least one locale is required
}

try {
  await client.site.update({
    timezone: 'Invalid/Timezone'
  });
} catch (error) {
  // ApiError with:
  // - 422: Invalid timezone identifier
}

try {
  await client.site.update({
    ip_whitelist: ['invalid-ip']
  });
} catch (error) {
  // ApiError with:
  // - 422: Invalid IP address or CIDR notation
}
```

## Best Practices

1. **Locale Planning**: Plan your locale structure before adding content
2. **Environment Separation**: Use different settings for different environments
3. **SEO Defaults**: Set fallback SEO to ensure all pages have meta tags
4. **Security**: Enable 2FA for production environments
5. **IP Whitelisting**: Use CIDR notation for IP ranges
6. **Frontend URL**: Keep this updated for proper preview functionality

## Important Notes

- The site resource is a singleton - there's only one per project
- Changes to locales affect all localized fields in the project
- Removing a locale doesn't delete content, but hides it from the API
- SSO settings require proper configuration on the identity provider side
- IP whitelist applies to all users, including administrators
- The `domain` field is read-only and set by DatoCMS

---

# Site Methods - Complete Reference

This document contains all methods available for the Site resource in the DatoCMS CMA Client.

## Available Methods for client.site

### find() - Retrieve Site Configuration

Retrieves the site's configuration and settings.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get site configuration
const site = await client.site.find();

// Site structure includes:
// {
//   id: 'site_id',
//   type: 'site',
//   attributes: {
//     name: 'My DatoCMS Project',
//     internal_subdomain: 'my-project-123',
//     domain: 'www.example.com',
//     locales: ['en', 'es', 'it'],
//     global_seo: {
//       site_name: 'Example Site',
//       title_suffix: ' - Example Site',
//       twitter_account: '@example',
//       facebook_page_url: 'https://facebook.com/example',
//       fallback_seo: {
//         title: 'Default Title',
//         description: 'Default description',
//         image: {
//           type: 'upload',
//           id: 'upload_id'
//         }
//       }
//     },
//     theme: {
//       primary_color: { hex: '#FF6B6B' },
//       light_color: { hex: '#FFECEC' },
//       dark_color: { hex: '#CC0000' },
//       accent_color: { hex: '#4ECDC4' }
//     },
//     timezone: 'America/New_York',
//     favicon: {
//       type: 'upload',
//       id: 'favicon_upload_id'
//     },
//     no_index: false,
//     frontend_url: 'https://www.example.com',
//     sso_enabled: false,
//     require_2fa: false,
//     ip_whitelist: null
//   },
//   relationships: {
//     account: {
//       data: { type: 'account', id: 'account_id' }
//     },
//     internal_domain: {
//       data: { type: 'internal_domain', id: 'domain_id' }
//     },
//     menu_items: {
//       data: [
//         { type: 'menu_item', id: 'menu_item_1' },
//         { type: 'menu_item', id: 'menu_item_2' }
//       ]
//     },
//     schema_menu_items: {
//       data: [
//         { type: 'schema_menu_item', id: 'schema_item_1' }
//       ]
//     },
//     environments: {
//       data: [
//         { type: 'environment', id: 'main' },
//         { type: 'environment', id: 'staging' }
//       ]
//     },
//     build_triggers: {
//       data: [
//         { type: 'build_trigger', id: 'trigger_1' }
//       ]
//     }
//   }
// }

// Get with query parameters (for specific fields)
const siteWithParams = await client.site.find({
  // Add any specific query parameters if needed
});
```

### update() - Update Site Configuration

Updates the site's settings and configuration.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Update basic site info
const updated = await client.site.update({
  name: 'Updated Project Name',
  domain: 'www.newdomain.com'
});

// Update locales
const withNewLocales = await client.site.update({
  locales: ['en', 'es', 'it', 'fr'] // Add French
});

// Update global SEO settings
const withSeo = await client.site.update({
  global_seo: {
    site_name: 'My Amazing Site',
    title_suffix: ' | Amazing Site',
    twitter_account: '@amazingsite',
    facebook_page_url: 'https://facebook.com/amazingsite',
    fallback_seo: {
      title: 'Welcome to Amazing Site',
      description: 'The most amazing site on the web',
      image: {
        type: 'upload',
        id: 'seo_image_upload_id'
      }
    }
  }
});

// Update theme colors
const withTheme = await client.site.update({
  theme: {
    primary_color: { hex: '#0066CC' },
    light_color: { hex: '#E6F2FF' },
    dark_color: { hex: '#003D7A' },
    accent_color: { hex: '#FF6B35' }
  }
});

// Update timezone
const withTimezone = await client.site.update({
  timezone: 'Europe/London'
});

// Update security settings
const withSecurity = await client.site.update({
  require_2fa: true,
  ip_whitelist: [
    '192.168.1.0/24',
    '10.0.0.0/8'
  ]
});

// Update favicon
const withFavicon = await client.site.update({
  favicon: {
    type: 'upload',
    id: 'new_favicon_upload_id'
  }
});

// Update frontend URL for preview links
const withFrontendUrl = await client.site.update({
  frontend_url: 'https://preview.example.com'
});

// Disable indexing
const noIndex = await client.site.update({
  no_index: true // Prevents search engines from indexing CMS content
});

// Complex update with multiple settings
const complexUpdate = await client.site.update({
  name: 'Production Site',
  domain: 'www.production.com',
  locales: ['en', 'es', 'de', 'fr', 'it'],
  timezone: 'UTC',
  theme: {
    primary_color: { hex: '#1E40AF' },
    light_color: { hex: '#DBEAFE' },
    dark_color: { hex: '#1E3A8A' },
    accent_color: { hex: '#F59E0B' }
  },
  global_seo: {
    site_name: 'Production Site',
    title_suffix: ' - Production',
    fallback_seo: {
      title: 'Welcome',
      description: 'Production site for our application'
    }
  },
  require_2fa: true,
  no_index: false
});

// Error handling
try {
  const updated = await client.site.update({
    locales: [] // Cannot have empty locales
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation error:', error.response.data.errors);
  }
}
```

### Advanced Site Configuration Methods

These methods are deprecated but still available. They activate specific features on the site:

```typescript
// Activate improved timezone management
await client.site.activateImprovedTimezoneManagement();

// Activate improved hex color management
await client.site.activateImprovedHexManagement();

// Activate improved GraphQL multi-locale fields
await client.site.activateImprovedGqlMultilocaleFields();

// Activate improved GraphQL visibility control
await client.site.activateImprovedGqlVisibilityControl();

// Activate improved boolean fields
await client.site.activateImprovedBooleanFields();

// Set draft mode as default for all models
await client.site.activateDraftModeAsDefault();

// Activate improved validation at publishing
await client.site.activateImprovedValidationAtPublishing();

// Activate improved exposure of inline blocks in CDA
await client.site.activateImprovedExposureOfInlineBlocksInCda();

// Update CDN default settings for assets
await client.site.updateAssetsCdnDefaultSettings({
  imgix_params: {
    auto: 'format',
    fit: 'max',
    q: 80
  }
});
```

## Working with Site Configuration

### Locale Management

```typescript
async function addLocale(newLocale) {
  const site = await client.site.find();
  const currentLocales = site.locales;
  
  if (currentLocales.includes(newLocale)) {
    console.log(`Locale ${newLocale} already exists`);
    return site;
  }
  
  const updated = await client.site.update({
    locales: [...currentLocales, newLocale]
  });
  
  console.log(`Added locale: ${newLocale}`);
  return updated;
}

// Add multiple locales
async function addMultipleLocales(localesToAdd) {
  const site = await client.site.find();
  const currentLocales = new Set(site.locales);
  
  localesToAdd.forEach(locale => currentLocales.add(locale));
  
  return client.site.update({
    locales: Array.from(currentLocales)
  });
}

// Remove locale (requires data migration)
async function removeLocale(localeToRemove) {
  const site = await client.site.find();
  
  // WARNING: This will delete all content in this locale
  const newLocales = site.locales.filter(l => l !== localeToRemove);
  
  if (newLocales.length === 0) {
    throw new Error('Cannot remove all locales');
  }
  
  return client.site.update({
    locales: newLocales
  });
}
```

### Theme Management

```typescript
async function applyThemePreset(preset) {
  const themes = {
    corporate: {
      primary_color: { hex: '#1E40AF' },
      light_color: { hex: '#DBEAFE' },
      dark_color: { hex: '#1E3A8A' },
      accent_color: { hex: '#F59E0B' }
    },
    startup: {
      primary_color: { hex: '#7C3AED' },
      light_color: { hex: '#EDE9FE' },
      dark_color: { hex: '#5B21B6' },
      accent_color: { hex: '#10B981' }
    },
    minimal: {
      primary_color: { hex: '#000000' },
      light_color: { hex: '#F9FAFB' },
      dark_color: { hex: '#111827' },
      accent_color: { hex: '#6B7280' }
    }
  };
  
  const theme = themes[preset];
  if (!theme) {
    throw new Error(`Unknown theme preset: ${preset}`);
  }
  
  return client.site.update({ theme });
}

// Generate theme from brand color
function generateThemeFromColor(brandHex) {
  // Simple theme generation logic
  const lighten = (hex, percent) => {
    // Lighten color implementation
    return hex; // Simplified
  };
  
  const darken = (hex, percent) => {
    // Darken color implementation
    return hex; // Simplified
  };
  
  return {
    primary_color: { hex: brandHex },
    light_color: { hex: lighten(brandHex, 80) },
    dark_color: { hex: darken(brandHex, 20) },
    accent_color: { hex: brandHex } // Or complementary color
  };
}
```

### SEO Configuration

```typescript
async function configureSEO(seoSettings) {
  const currentSite = await client.site.find();
  
  return client.site.update({
    global_seo: {
      ...currentSite.global_seo,
      ...seoSettings,
      fallback_seo: {
        ...currentSite.global_seo?.fallback_seo,
        ...seoSettings.fallback_seo
      }
    }
  });
}

// Set up complete SEO
async function setupCompleteSEO({
  siteName,
  titleSuffix,
  description,
  imageUploadId,
  twitterAccount,
  facebookPageUrl
}) {
  return client.site.update({
    global_seo: {
      site_name: siteName,
      title_suffix: titleSuffix || ` - ${siteName}`,
      twitter_account: twitterAccount,
      facebook_page_url: facebookPageUrl,
      fallback_seo: {
        title: `Welcome to ${siteName}`,
        description: description,
        image: imageUploadId ? {
          type: 'upload',
          id: imageUploadId
        } : null
      }
    }
  });
}
```

### Security Configuration

```typescript
async function enhanceSecurity(options = {}) {
  const updates = {};
  
  // Two-factor authentication
  if (options.require2FA !== undefined) {
    updates.require_2fa = options.require2FA;
  }
  
  // IP whitelist
  if (options.ipWhitelist) {
    // Validate IP addresses/ranges
    const validIPs = options.ipWhitelist.filter(ip => {
      // Simple validation
      return /^[\d.\/]+$/.test(ip);
    });
    
    updates.ip_whitelist = validIPs.length > 0 ? validIPs : null;
  }
  
  // SSO (if available on plan)
  if (options.enableSSO !== undefined) {
    updates.sso_enabled = options.enableSSO;
  }
  
  return client.site.update(updates);
}

// Lock down to specific IPs
async function restrictToOfficeIPs() {
  return client.site.update({
    ip_whitelist: [
      '203.0.113.0/24',  // Office network
      '198.51.100.14',   // VPN server
      '192.0.2.0/24'     // Development network
    ],
    require_2fa: true
  });
}
```

### Site Information Export

```typescript
async function exportSiteConfiguration() {
  const site = await client.site.find();
  
  return {
    basic: {
      name: site.name,
      subdomain: site.internal_subdomain,
      domain: site.domain,
      timezone: site.timezone
    },
    locales: site.locales,
    theme: site.theme,
    seo: site.global_seo,
    security: {
      require_2fa: site.require_2fa,
      sso_enabled: site.sso_enabled,
      ip_whitelist: site.ip_whitelist
    },
    metadata: {
      account_id: site.relationships.account.data.id,
      environment_count: site.relationships.environments.data.length,
      created_at: site.created_at,
      updated_at: site.updated_at
    }
  };
}

// Clone site configuration to another project
async function cloneSiteConfiguration(sourceClient, targetClient) {
  const sourceConfig = await sourceClient.site.find();
  
  // Apply transferable settings to target
  return targetClient.site.update({
    locales: sourceConfig.locales,
    theme: sourceConfig.theme,
    global_seo: sourceConfig.global_seo,
    timezone: sourceConfig.timezone,
    no_index: sourceConfig.no_index
  });
}
```

## Error Handling

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Comprehensive error handling
async function safeSiteUpdate(updates) {
  try {
    const updated = await client.site.update(updates);
    return updated;
  } catch (error) {
    if (error.response) {
      switch (error.response.status) {
        case 422:
          const validationErrors = error.response.data.errors;
          validationErrors.forEach(err => {
            if (err.source?.pointer?.includes('locales')) {
              console.error('Locale error:', err.detail);
              // Common: empty locales, invalid locale codes
            } else if (err.source?.pointer?.includes('theme')) {
              console.error('Theme error:', err.detail);
              // Common: invalid hex colors
            } else if (err.source?.pointer?.includes('ip_whitelist')) {
              console.error('IP whitelist error:', err.detail);
              // Common: invalid IP formats
            }
          });
          break;
        case 403:
          console.error('Insufficient permissions to update site settings');
          break;
        default:
          console.error('Unexpected error:', error.response.status);
      }
    }
    throw error;
  }
}

// Validate before update
function validateSiteUpdate(updates) {
  const errors = [];
  
  // Validate locales
  if (updates.locales) {
    if (!Array.isArray(updates.locales) || updates.locales.length === 0) {
      errors.push('At least one locale is required');
    }
    
    // Validate locale codes
    const validLocalePattern = /^[a-z]{2}(-[A-Z]{2})?$/;
    updates.locales.forEach(locale => {
      if (!validLocalePattern.test(locale)) {
        errors.push(`Invalid locale code: ${locale}`);
      }
    });
  }
  
  // Validate colors
  if (updates.theme) {
    const colorFields = ['primary_color', 'light_color', 'dark_color', 'accent_color'];
    colorFields.forEach(field => {
      if (updates.theme[field]) {
        const hex = updates.theme[field].hex;
        if (!/^#[0-9A-F]{6}$/i.test(hex)) {
          errors.push(`Invalid hex color for ${field}: ${hex}`);
        }
      }
    });
  }
  
  // Validate timezone
  if (updates.timezone) {
    // Simple check - you might want to use a timezone library
    if (typeof updates.timezone !== 'string' || updates.timezone.length === 0) {
      errors.push('Invalid timezone');
    }
  }
  
  return errors;
}
```

## Best Practices

1. **Locale Management**: Always backup content before removing locales
2. **Theme Consistency**: Use a consistent color palette across all theme colors
3. **SEO Defaults**: Set comprehensive fallback SEO for better search visibility
4. **Security**: Enable 2FA for production environments
5. **IP Whitelisting**: Use CIDR notation for IP ranges
6. **Timezone**: Set appropriate timezone for your primary audience
7. **Frontend URL**: Keep this updated for accurate preview links

---

# Build Trigger Resource

## Overview

Build Triggers allow you to automatically trigger builds on external services (like Netlify, Vercel, or custom webhooks) when content changes in DatoCMS. They're essential for static site generation workflows, enabling automatic deployments when content is updated, published, or scheduled.

**API Client Section**: `client.buildTriggers`

## Build Trigger Methods

### list() - List All Build Triggers

Retrieve all build triggers configured for the site.

```typescript
const triggers = await client.buildTriggers.list();

// Filter by adapter type
const netlifyTriggers = triggers.filter(t => t.adapter === 'netlify');
const customTriggers = triggers.filter(t => t.adapter === 'custom');
```

### find() - Get Single Build Trigger

Retrieve a specific build trigger by ID.

```typescript
const trigger = await client.buildTriggers.find('trigger-id');

console.log(`Name: ${trigger.name}`);
console.log(`Adapter: ${trigger.adapter}`);
console.log(`Frontend URL: ${trigger.frontend_url}`);
```

### create() - Create New Build Trigger

Create a new build trigger with specified configuration.

```typescript
// Netlify trigger
const netlifyTrigger = await client.buildTriggers.create({
  name: 'Netlify Production',
  adapter: 'netlify',
  adapter_settings: {
    trigger_url: 'https://api.netlify.com/build_hooks/YOUR_HOOK_ID',
    site_id: 'your-netlify-site-id'
  },
  frontend_url: 'https://www.example.com',
  auto_trigger_on_scheduled_publications: true
});

// Vercel trigger
const vercelTrigger = await client.buildTriggers.create({
  name: 'Vercel Production',
  adapter: 'vercel',
  adapter_settings: {
    trigger_url: 'https://api.vercel.com/v1/integrations/deploy/YOUR_DEPLOY_HOOK',
    project_id: 'your-vercel-project-id'
  },
  frontend_url: 'https://www.example.com'
});

// Custom webhook with authentication
const customTrigger = await client.buildTriggers.create({
  name: 'Custom CI/CD Pipeline',
  adapter: 'custom',
  adapter_settings: {
    trigger_url: 'https://ci.example.com/api/builds/trigger',
    headers: {
      'Authorization': 'Bearer YOUR_CI_TOKEN',
      'X-Project-ID': 'datocms-site',
      'Content-Type': 'application/json'
    },
    request_body: JSON.stringify({
      branch: 'main',
      source: 'datocms',
      trigger_type: 'content_update',
      environment: 'production'
    })
  },
  webhook_token: 'your_secret_token',
  frontend_url: 'https://www.example.com',
  indexing_enabled: true
});
```

### update() - Update Build Trigger

Update an existing build trigger configuration.

```typescript
// Update name and URL
const updated = await client.buildTriggers.update('trigger-id', {
  name: 'Production Build v2',
  frontend_url: 'https://new.example.com'
});

// Enable features
const withFeatures = await client.buildTriggers.update('trigger-id', {
  indexing_enabled: true,
  auto_trigger_on_scheduled_publications: true
});
```

### destroy() - Delete Build Trigger

Permanently delete a build trigger.

```typescript
await client.buildTriggers.destroy('trigger-id');
```

### trigger() - Manually Trigger a Build

Manually trigger a deployment using the build trigger.

```typescript
await client.buildTriggers.trigger('trigger-id');
console.log('Build triggered successfully');
```

### abort() - Abort a Deploy

Abort an ongoing deployment and mark it as failed.

```typescript
await client.buildTriggers.abort('trigger-id');
```

### reindex() - Trigger Site Search Indexing

Trigger a new site search indexing/spidering of the website.

```typescript
await client.buildTriggers.reindex('trigger-id');
```

### abortIndexing() - Abort Site Indexing

Abort an ongoing site search indexing and mark it as failed.

```typescript
await client.buildTriggers.abortIndexing('trigger-id');
```

## Build Trigger Configuration Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Display name for the trigger (required) |
| `adapter` | string | Adapter type: 'netlify', 'vercel', 'custom' (required) |
| `adapter_settings` | object | Adapter-specific configuration (required) |
| `adapter_settings.trigger_url` | string | Webhook URL to trigger builds |
| `adapter_settings.headers` | object | Custom HTTP headers (for custom adapter) |
| `adapter_settings.request_body` | string | Custom request body (for custom adapter) |
| `frontend_url` | string | Frontend URL for the deployed site |
| `webhook_token` | string | Secret token for webhook validation |
| `indexing_enabled` | boolean | Enable site search indexing |
| `auto_trigger_on_scheduled_publications` | boolean | Auto-trigger on scheduled publish/unpublish |

## Build Trigger Patterns

### Multi-Environment Deployment

```typescript
async function setupMultiEnvironmentDeployment() {
  const environments = [
    {
      name: 'Production',
      url: 'https://www.example.com',
      hookUrl: 'https://api.netlify.com/build_hooks/prod_hook',
      autoTrigger: true
    },
    {
      name: 'Staging',
      url: 'https://staging.example.com',
      hookUrl: 'https://api.netlify.com/build_hooks/staging_hook',
      autoTrigger: false
    }
  ];
  
  const triggers = [];
  
  for (const env of environments) {
    const trigger = await client.buildTriggers.create({
      name: `${env.name} Deploy`,
      adapter: 'netlify',
      adapter_settings: {
        trigger_url: env.hookUrl
      },
      frontend_url: env.url,
      auto_trigger_on_scheduled_publications: env.autoTrigger
    });
    
    triggers.push({
      environment: env.name,
      triggerId: trigger.id,
      url: env.url
    });
  }
  
  return triggers;
}
```

### GitHub Actions Integration

```typescript
async function setupGitHubActions({ owner, repo, token }) {
  return client.buildTriggers.create({
    name: 'GitHub Actions Deploy',
    adapter: 'custom',
    adapter_settings: {
      trigger_url: `https://api.github.com/repos/${owner}/${repo}/dispatches`,
      headers: {
        'Authorization': `Bearer ${token}`,
        'Accept': 'application/vnd.github.v3+json'
      },
      request_body: JSON.stringify({
        event_type: 'datocms-content-update',
        client_payload: {
          environment: 'production',
          triggered_by: 'datocms'
        }
      })
    },
    frontend_url: 'https://www.example.com'
  });
}
```

### Build Trigger Health Check

```typescript
async function healthCheckBuildTriggers() {
  const triggers = await client.buildTriggers.list();
  const results = [];
  
  for (const trigger of triggers) {
    const health = {
      id: trigger.id,
      name: trigger.name,
      adapter: trigger.adapter,
      checks: {
        configuration: !!trigger.adapter_settings?.trigger_url,
        lastBuild: trigger.meta.last_build_at ? {
          timestamp: trigger.meta.last_build_at,
          hoursAgo: Math.floor(
            (Date.now() - new Date(trigger.meta.last_build_at).getTime()) / 
            (1000 * 60 * 60)
          )
        } : null,
        indexing: trigger.indexing_enabled ? {
          enabled: true,
          status: trigger.indexing_status,
          isHealthy: trigger.indexing_status === 'success'
        } : null
      }
    };
    
    health.overallHealth = 
      health.checks.configuration &&
      (!health.checks.indexing || health.checks.indexing.isHealthy);
    
    results.push(health);
  }
  
  return {
    total: results.length,
    healthy: results.filter(r => r.overallHealth).length,
    unhealthy: results.filter(r => !r.overallHealth).length,
    details: results
  };
}
```

## Build Trigger Object Structure

```typescript
interface BuildTrigger {
  id: string;
  type: 'build_trigger';
  attributes: {
    name: string;
    adapter: 'netlify' | 'vercel' | 'custom';
    adapter_settings: {
      trigger_url: string;
      site_id?: string;
      project_id?: string;
      headers?: Record<string, string>;
      request_body?: string;
    };
    webhook_token?: string | null;
    indexing_enabled: boolean;
    indexing_status?: 'success' | 'error' | 'in_progress' | null;
    frontend_url?: string | null;
    auto_trigger_on_scheduled_publications: boolean;
  };
  meta: {
    last_build_at?: string | null;
    last_indexing_at?: string | null;
  };
}
```

---

# Environment Resource

## Overview

Environments allow you to create isolated copies of your DatoCMS project where you can safely experiment with schema changes, content modifications, and configuration updates without affecting your production environment. Each environment is a complete fork including all content, schema, and most configuration settings.

**API Client Section**: `client.environments`

## Environment Methods

### list() - List All Environments

Retrieve all environments in the project, including primary and sandbox environments.

```typescript
const environments = await client.environments.list();

// Find primary environment
const primary = environments.find(env => env.is_primary);

// List sandbox environments
const sandboxes = environments.filter(env => !env.is_primary);
```

### find() - Get Single Environment

Retrieve details about a specific environment by ID.

```typescript
const environment = await client.environments.find('staging');

console.log(`Name: ${environment.label}`);
console.log(`Primary: ${environment.is_primary}`);
console.log(`Status: ${environment.meta.status}`);
console.log(`Items: ${environment.meta.item_count}`);
```

### create() - Fork a New Environment

Create a new environment by forking an existing one. This is an asynchronous operation that copies all content, schema, and configuration.

```typescript
import { waitForJobResult } from '@datocms/rest-client-utils';

// Fork primary environment
const job = await client.environments.create('main', {
  id: 'staging',
  label: 'Staging Environment'
});

// Wait for completion
const result = await waitForJobResult(job.id, client);
console.log(`Environment ${result.payload.data.id} created`);

// Fast fork without indexing
const quickForkJob = await client.environments.create('main', {
  id: 'quick-test',
  label: 'Quick Test',
  indexing_enabled: false // Faster creation, no search
});
```

**Fork Time Estimates**:
- Small projects (<1,000 items): 1-2 minutes
- Medium projects (1,000-10,000 items): 2-5 minutes  
- Large projects (>10,000 items): 5-15+ minutes
- With indexing disabled: 30-50% faster

### promote() - Promote Environment to Primary

Replace the current primary environment with a sandbox environment. This is how you deploy changes from development to production.

⚠️ **WARNING**: Promotion is irreversible. The current primary environment will be destroyed and replaced.

```typescript
// CAUTION: This replaces your production environment!
const job = await client.environments.promote('staging');

// Wait for completion
const result = await waitForJobResult(job.id, client);
console.log('Environment promoted to primary');
```

### update() - Update Environment

Update environment properties (currently only the label can be updated).

```typescript
// Simple rename
const updated = await client.environments.update('staging', {
  label: 'Staging (Ready for QA)'
});

// Update with workflow status
await client.environments.update('feature-auth', {
  label: 'Feature: Authentication (In Review)'
});
```

### destroy() - Delete Environment

Delete a sandbox environment. Only non-primary environments can be deleted.

```typescript
// Delete single environment
await client.environments.destroy('old-feature');
```

## Environment Configuration Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | Unique identifier for new environment (required) |
| `label` | string | Human-readable name (optional) |
| `indexing_enabled` | boolean | Enable content search indexing (default: true) |

## Environment Patterns

### Development/Staging/Production Workflow

```typescript
async function setupEnvironmentWorkflow() {
  const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });
  
  // 1. Create staging from production
  const stagingJob = await client.environments.create('main', {
    id: 'staging',
    label: 'Staging Environment'
  });
  await waitForJobResult(stagingJob.id, client);
  
  // 2. Create development from staging
  const devJob = await client.environments.create('staging', {
    id: 'development',
    label: 'Development Environment',
    indexing_enabled: false // Faster, no search needed
  });
  await waitForJobResult(devJob.id, client);
  
  return {
    production: buildClient({ apiToken: 'YOUR_API_TOKEN', environment: 'main' }),
    staging: buildClient({ apiToken: 'YOUR_API_TOKEN', environment: 'staging' }),
    development: buildClient({ apiToken: 'YOUR_API_TOKEN', environment: 'development' })
  };
}
```

### Feature Branch Workflow

```typescript
async function createFeatureBranch(featureName) {
  const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });
  
  const id = `feature-${featureName.toLowerCase().replace(/\s+/g, '-')}`;
  
  // Validate ID format
  if (!/^[a-z0-9]([a-z0-9-]{0,28}[a-z0-9])?$/.test(id)) {
    throw new Error('Invalid environment ID format');
  }
  
  const job = await client.environments.create('main', {
    id,
    label: `Feature: ${featureName}`,
    indexing_enabled: false
  });
  
  const result = await waitForJobResult(job.id, client);
  
  return {
    environmentId: id,
    client: buildClient({ 
      apiToken: process.env.DATOCMS_API_TOKEN,
      environment: id 
    })
  };
}
```

### Safe Promotion with Backup

```typescript
async function safePromote(environmentId) {
  // 1. Create backup of current production
  console.log('Creating backup of production...');
  const backupJob = await client.environments.create('main', {
    id: `backup-${new Date().toISOString().split('T')[0]}`,
    label: `Production Backup ${new Date().toLocaleDateString()}`,
    indexing_enabled: false
  });
  await waitForJobResult(backupJob.id, client);
  
  // 2. Validate target environment
  const testClient = buildClient({ 
    apiToken: process.env.DATOCMS_API_TOKEN,
    environment: environmentId 
  });
  
  // Check for invalid items
  const invalidItems = await testClient.items.list({
    filter: { valid: { eq: false } },
    page: { limit: 1 }
  });
  
  if (invalidItems.length > 0) {
    throw new Error(`Cannot promote: ${invalidItems.length} invalid items`);
  }
  
  // 3. Proceed with promotion
  console.log('Promoting environment...');
  const promoteJob = await client.environments.promote(environmentId);
  await waitForJobResult(promoteJob.id, client);
  
  console.log('✅ Promotion completed successfully');
  return true;
}
```

### Environment Cleanup

```typescript
async function cleanupOldEnvironments(daysOld = 30) {
  const environments = await client.environments.list();
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() - daysOld);
  
  const toDelete = environments.filter(env => {
    if (env.is_primary) return false;
    const created = new Date(env.created_at);
    return created < cutoffDate;
  });
  
  console.log(`Found ${toDelete.length} environments older than ${daysOld} days`);
  
  for (const env of toDelete) {
    console.log(`Deleting: ${env.id} (${env.label})`);
    await client.environments.destroy(env.id);
  }
  
  return toDelete.length;
}
```

### Environment Comparison

```typescript
async function compareEnvironments(env1, env2) {
  const client1 = buildClient({ apiToken: 'YOUR_API_TOKEN', environment: env1 });
  const client2 = buildClient({ apiToken: 'YOUR_API_TOKEN', environment: env2 });
  
  const [count1, count2] = await Promise.all([
    client1.items.rawList({ page: { limit: 1 } }).then(r => r.meta.total_count),
    client2.items.rawList({ page: { limit: 1 } }).then(r => r.meta.total_count)
  ]);
  
  return {
    [env1]: count1,
    [env2]: count2,
    difference: count1 - count2
  };
}
```

## Environment Object Structure

```typescript
interface Environment {
  id: string;                           // Unique identifier (e.g., 'main', 'staging')
  type: 'environment';                  // Resource type
  label: string;                        // Human-readable name
  created_at: string;                   // ISO 8601 timestamp
  is_primary: boolean;                  // True for production environment
  forked_from: string | null;           // Source environment ID
  indexing_enabled: boolean;            // Search indexing status
  meta: {
    status: 'ready' | 'creating' | 'destroying' | 'forking' | 'promoting';
    last_data_change_at?: string;       // ISO 8601 timestamp
    last_schema_change_at?: string;     // ISO 8601 timestamp
    item_count?: number;                // Total content items
    upload_count?: number;              // Total uploads
  };
}
```

## Environment Best Practices

1. **Limit Active Environments**: Delete unused environments to stay within quota
2. **Use Indexing Wisely**: Disable for temporary/test environments
3. **Regular Backups**: Create backups before major changes
4. **Clear Naming**: Use descriptive names indicating purpose
5. **Validate Before Promote**: Always check for issues before promoting
6. **Automate Cleanup**: Implement automatic deletion for temporary environments

---

# Maintenance Mode Resource

## Overview

The Maintenance Mode resource allows you to control public access to your DatoCMS website while performing updates or maintenance. When activated, your public website returns a 503 Service Unavailable status to visitors, while administrators can continue to access and modify content through the DatoCMS interface and APIs.

**API Client Section**: `client.maintenanceMode`

## Maintenance Mode Methods

### find() - Get Maintenance Mode Status

Retrieve the current maintenance mode status for the site.

```typescript
const maintenanceMode = await client.maintenanceMode.find();

console.log(`Active: ${maintenanceMode.active}`);
console.log(`Scheduled deactivation: ${maintenanceMode.scheduled_deactivation_at}`);

// Check if currently in maintenance
if (maintenanceMode.active) {
  console.log('Site is in maintenance mode');
  
  if (maintenanceMode.scheduled_deactivation_at) {
    const endTime = new Date(maintenanceMode.scheduled_deactivation_at);
    const remaining = Math.ceil((endTime - new Date()) / 60000);
    console.log(`Will deactivate in ${remaining} minutes`);
  }
}
```

### activate() - Enable Maintenance Mode

Activate maintenance mode for the site, optionally with scheduled deactivation.

```typescript
// Activate immediately
const maintenanceMode = await client.maintenanceMode.activate();

// Activate with scheduled deactivation (2 hours)
const endTime = new Date();
endTime.setHours(endTime.getHours() + 2);

const scheduled = await client.maintenanceMode.activate({
  scheduled_deactivation_at: endTime.toISOString()
});

// Force activation (bypass active editor check)
const forced = await client.maintenanceMode.activate({ 
  force: true 
});
```

### deactivate() - Disable Maintenance Mode

Deactivate maintenance mode, restoring normal public access to the site.

```typescript
// Deactivate maintenance mode
const maintenanceMode = await client.maintenanceMode.deactivate();

console.log('Maintenance mode deactivated');
console.log(`Active: ${maintenanceMode.active}`); // false

// Safe deactivation with check
async function safeDeactivate() {
  const current = await client.maintenanceMode.find();
  
  if (current.active) {
    await client.maintenanceMode.deactivate();
    console.log('Maintenance mode deactivated');
    return true;
  } else {
    console.log('Maintenance mode was not active');
    return false;
  }
}
```

## Maintenance Mode Configuration Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `scheduled_deactivation_at` | string | ISO 8601 datetime for automatic deactivation (optional) |
| `force` | boolean | Force activation even if users are editing (optional) |

## Maintenance Mode Patterns

### Scheduled Maintenance Window

```typescript
async function performMaintenance(client, maintenanceTask) {
  let wasActivated = false;
  
  try {
    // Check current status
    const currentStatus = await client.maintenanceMode.find();
    
    if (currentStatus.active) {
      console.log('Maintenance mode already active');
    } else {
      // Activate maintenance mode
      console.log('Activating maintenance mode...');
      await client.maintenanceMode.activate({ force: true });
      wasActivated = true;
      console.log('✅ Maintenance mode activated');
    }
    
    // Perform maintenance tasks
    console.log('Performing maintenance...');
    await maintenanceTask();
    console.log('✅ Maintenance completed');
    
  } catch (error) {
    console.error('❌ Maintenance failed:', error.message);
    throw error;
  } finally {
    // Only deactivate if we activated it
    if (wasActivated) {
      console.log('Deactivating maintenance mode...');
      await client.maintenanceMode.deactivate();
      console.log('✅ Maintenance mode deactivated');
    }
  }
}

// Usage
await performMaintenance(client, async () => {
  // Your maintenance tasks here
  await client.itemTypes.update('model_id', { /* updates */ });
});
```

### Emergency Maintenance

```typescript
async function handleEmergency(errorType) {
  try {
    // Immediately activate maintenance mode
    await client.maintenanceMode.activate({ force: true });
    
    // Log the emergency
    console.error(`Emergency maintenance activated due to: ${errorType}`);
    
    // Perform emergency fixes
    switch (errorType) {
      case 'database_corruption':
        await runDatabaseRepair();
        break;
      case 'security_breach':
        await lockdownAccess();
        await runSecurityAudit();
        break;
      case 'performance_degradation':
        await clearCaches();
        await restartServices();
        break;
    }
    
    // Run health checks before deactivating
    const healthCheckPassed = await runHealthChecks();
    if (healthCheckPassed) {
      await client.maintenanceMode.deactivate();
      console.log('Emergency resolved, site restored');
    } else {
      console.error('Health checks failed, keeping maintenance mode active');
    }
    
  } catch (error) {
    console.error('Emergency handler failed:', error);
    throw error;
  }
}
```

### Safe Data Migration

```typescript
async function safeMigration(client, migrationSteps) {
  const rollbackActions = [];
  
  try {
    // Step 1: Activate maintenance mode
    console.log('🔒 Activating maintenance mode for migration...');
    await client.maintenanceMode.activate({ force: true });
    
    // Step 2: Execute migration steps
    for (const [index, step] of migrationSteps.entries()) {
      console.log(`Executing step ${index + 1}/${migrationSteps.length}: ${step.name}`);
      
      try {
        const rollback = await step.execute(client);
        if (rollback) {
          rollbackActions.unshift(rollback); // Add to beginning for reverse order
        }
        console.log(`✅ Step ${index + 1} completed`);
      } catch (error) {
        console.error(`❌ Step ${index + 1} failed:`, error.message);
        throw error;
      }
    }
    
    // Step 3: Verify migration success
    console.log('🔍 Verifying migration...');
    await verifyMigration(client);
    
    console.log('✅ Migration completed successfully');
    
  } catch (error) {
    console.error('❌ Migration failed, starting rollback...');
    
    // Execute rollback actions
    for (const rollback of rollbackActions) {
      try {
        await rollback(client);
      } catch (rollbackError) {
        console.error('Rollback action failed:', rollbackError);
      }
    }
    
    throw error;
  } finally {
    // Always deactivate maintenance mode
    console.log('🔓 Deactivating maintenance mode...');
    await client.maintenanceMode.deactivate();
  }
}
```

### Deployment Coordination

```typescript
async function safeDeployment(deploymentConfig) {
  let wasActivated = false;
  
  try {
    // Check current status
    const currentStatus = await client.maintenanceMode.find();
    
    // Only activate if not already active
    if (!currentStatus.active) {
      await client.maintenanceMode.activate();
      wasActivated = true;
      console.log('Maintenance mode activated for deployment');
    }
    
    // Perform deployment steps
    await buildApplication(deploymentConfig);
    await runMigrations();
    await updateAssets();
    await warmupCaches();
    
    // Run health checks
    const healthCheckPassed = await runHealthChecks();
    if (!healthCheckPassed) {
      throw new Error('Health checks failed after deployment');
    }
    
    // Only deactivate if we activated it
    if (wasActivated) {
      await client.maintenanceMode.deactivate();
      console.log('Deployment complete, maintenance mode deactivated');
    }
    
  } catch (error) {
    console.error('Deployment failed:', error);
    // Keep maintenance mode active on failure for investigation
    throw error;
  }
}
```

## Maintenance Mode Object Structure

```typescript
interface MaintenanceMode {
  id: string;                              // Resource ID (e.g., "3210")
  type: 'maintenance_mode';                // Resource type
  active: boolean;                         // Whether maintenance mode is active
  scheduled_activation_at: string | null;   // ISO 8601 datetime for scheduled activation
  scheduled_deactivation_at: string | null; // ISO 8601 datetime for scheduled deactivation
}

// Example responses

// Inactive state
{
  id: "3210",
  type: "maintenance_mode",
  active: false,
  scheduled_activation_at: null,
  scheduled_deactivation_at: null
}

// Active with scheduled deactivation
{
  id: "3210",
  type: "maintenance_mode",
  active: true,
  scheduled_activation_at: null,
  scheduled_deactivation_at: "2024-01-15T14:00:00.000Z"
}
```

## What Happens During Maintenance Mode

### Public Website
- Returns 503 Service Unavailable status
- Shows maintenance page (customizable)
- No access to regular content

### Admin Access
- DatoCMS interface remains accessible
- Content can be edited and published
- All admin features work normally

### API Behavior
- **Content Management API**: Fully operational
- **Content Delivery API**: Continues serving cached content
- **GraphQL API**: Fully operational
- **Real-time Updates API**: Continues to work

## Maintenance Mode Best Practices

1. **Always Check Current Status**: Verify maintenance mode status before activating
2. **Use Scheduled Deactivation**: Set automatic deactivation to prevent forgetting
3. **Implement Health Checks**: Verify system health before deactivating
4. **Coordinate with Team**: Notify team members before maintenance
5. **Use Try/Finally for Safety**: Ensure maintenance mode is deactivated even on errors

---

# Menu Item Resource

## Overview

The MenuItem resource allows you to create and manage navigation menus for your frontend website. These items form hierarchical navigation structures with support for external links, dynamic content links, and nested submenus. MenuItem is specifically for frontend navigation, not to be confused with SchemaMenuItem which organizes the DatoCMS admin interface.

**API Client Section**: `client.menuItems`

## Menu Item Methods

### list() - List All Menu Items

Retrieve all menu items in the project navigation.

```typescript
const menuItems = await client.menuItems.list();

// Filter top-level items
const topLevelItems = menuItems.filter(item => !item.parent);

// Filter by item type
const contentItems = menuItems.filter(item => 
  item.item_type_id !== null
);
```

### find() - Get Single Menu Item

Retrieve a specific menu item by ID.

```typescript
const menuItem = await client.menuItems.find('menu_item_id');

console.log(`Name: ${menuItem.label}`);
console.log(`URL: ${menuItem.external_url}`);
console.log(`Position: ${menuItem.position}`);
```

### create() - Create New Menu Item

Create a new menu item in the navigation.

```typescript
// Create menu item for a model
const modelMenuItem = await client.menuItems.create({
  label: 'Products',
  item_type: {
    type: 'item_type',
    id: 'product_model_id'
  },
  position: 1
});

// Create external link menu item
const externalMenuItem = await client.menuItems.create({
  label: 'Documentation',
  external_url: 'https://docs.example.com',
  open_in_new_tab: true,
  position: 2
});

// Create nested menu item
const nestedMenuItem = await client.menuItems.create({
  label: 'Blog Categories',
  item_type: {
    type: 'item_type',
    id: 'blog_category_model_id'
  },
  parent: {
    type: 'menu_item',
    id: 'parent_menu_item_id'
  },
  position: 1
});
```

### update() - Update Menu Item

Update an existing menu item's properties.

```typescript
// Update label
const updated = await client.menuItems.update('menu_item_id', {
  label: 'Updated Label'
});

// Update position
const repositioned = await client.menuItems.update('menu_item_id', {
  position: 5
});

// Change to external link
const externalLink = await client.menuItems.update('menu_item_id', {
  item_type: null,
  external_url: 'https://external.example.com',
  open_in_new_tab: true
});
```

### destroy() - Delete Menu Item

Delete a menu item from the navigation.

```typescript
await client.menuItems.destroy('menu_item_id');
```

## Menu Item Configuration Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | string | Display text for the menu item (required) |
| `external_url` | string | URL for external links (optional) |
| `position` | number | Order among siblings (required) |
| `open_in_new_tab` | boolean | Link behavior (optional) |
| `item_type` | object | Link to a DatoCMS model (optional) |
| `item_type_filter` | object | Filter for model items (optional) |
| `parent` | object | Parent menu item relationship (optional) |

## Menu Item Patterns

### Building Complete Navigation Structure

```typescript
async function buildMainNavigation(client) {
  // Create main navigation items
  const menuStructure = [
    { label: "Home", url: "/", position: 1 },
    { label: "Products", url: "/products", position: 2 },
    { label: "Services", url: "/services", position: 3 },
    { label: "About", url: "/about", position: 4 },
    { label: "Contact", url: "/contact", position: 5 }
  ];
  
  const mainItems = {};
  
  // Create root items
  for (const item of menuStructure) {
    mainItems[item.label] = await client.menuItems.create({
      label: item.label,
      external_url: item.url,
      position: item.position
    });
  }
  
  // Add product categories
  const categories = [
    { label: "Electronics", slug: "electronics" },
    { label: "Clothing", slug: "clothing" },
    { label: "Books", slug: "books" }
  ];
  
  for (let i = 0; i < categories.length; i++) {
    await client.menuItems.create({
      label: categories[i].label,
      external_url: `/products/${categories[i].slug}`,
      parent: { type: "menu_item", id: mainItems["Products"].id },
      position: i + 1
    });
  }
  
  return mainItems;
}
```

### Dynamic Blog Navigation

```typescript
async function createBlogNavigation(client) {
  // Get blog post model
  const blogPostModel = await client.itemTypes.find("blog_post");
  
  // Create main blog menu
  const blogMenu = await client.menuItems.create({
    label: "Blog",
    item_type: { type: "item_type", id: blogPostModel.id },
    position: 4
  });
  
  // Get all blog categories
  const categories = await client.items.list({
    filter: { type: "blog_category" }
  });
  
  // Create submenu for each category
  for (let i = 0; i < categories.length; i++) {
    const category = categories[i];
    
    // Create filter for posts in this category
    const filter = await client.itemTypeFilters.create(blogPostModel.id, {
      name: `${category.name} Posts`,
      filter: {
        fields: {
          category: { 
            any: { 
              eq: { type: "item", id: category.id } 
            }
          }
        }
      }
    });
    
    // Create menu item
    await client.menuItems.create({
      label: category.name,
      item_type: { type: "item_type", id: blogPostModel.id },
      item_type_filter: { type: "item_type_filter", id: filter.id },
      parent: { type: "menu_item", id: blogMenu.id },
      position: i + 1
    });
  }
}
```

### Navigation Tree Fetcher

```typescript
async function getNavigationTree(client) {
  // Fetch all menu items
  const allItems = await client.menuItems.list();
  
  // Build hierarchical structure
  function buildTree(items, parentId = null) {
    return items
      .filter(item => {
        const itemParentId = item.parent?.id || null;
        return itemParentId === parentId;
      })
      .sort((a, b) => a.position - b.position)
      .map(item => ({
        id: item.id,
        label: item.label,
        url: item.external_url,
        openInNewTab: item.open_in_new_tab,
        itemType: item.item_type,
        itemTypeFilter: item.item_type_filter,
        children: buildTree(items, item.id)
      }));
  }
  
  const tree = buildTree(allItems);
  
  // Separate main nav from footer nav (assuming position >= 100 is footer)
  const mainNav = tree.filter(item => item.position < 100);
  const footerNav = tree.filter(item => item.position >= 100);
  
  return {
    main: mainNav,
    footer: footerNav,
    all: tree
  };
}
```

## Menu Item Object Structure

```typescript
interface MenuItem {
  id: string;                       // Unique identifier
  type: 'menu_item';               // Resource type
  label: string;                   // Display text
  external_url: string | null;     // External link URL
  position: number;                // Order position
  open_in_new_tab: boolean;        // Link behavior
  item_type: {                     // Link to model
    id: string;
    type: 'item_type';
  } | null;
  item_type_filter: {              // Model filter
    id: string;
    type: 'item_type_filter';
  } | null;
  parent: {                        // Parent item
    id: string;
    type: 'menu_item';
  } | null;
  children: MenuItem[];            // Child items (read-only)
}
```

---

# Schema Menu Item Resource

## Overview

Schema Menu Items organize how content models appear in the DatoCMS sidebar navigation. You can create folders, group related models, and customize the content editing experience for your team.

**API Client Section**: `client.schemaMenuItems`

## Schema Menu Item Methods

### list() - List All Schema Menu Items

Retrieve all schema menu items in the project, including folders and model references.

```typescript
const schemaMenuItems = await client.schemaMenuItems.list();

// Filter top-level items
const topLevelItems = schemaMenuItems.filter(item => !item.parent);

// Filter folders only
const folders = schemaMenuItems.filter(item => item.kind === 'folder');

// Filter model items only
const modelItems = schemaMenuItems.filter(item => item.kind === 'item_type');
```

### find() - Get Single Schema Menu Item

Retrieve a specific schema menu item by ID.

```typescript
const schemaMenuItem = await client.schemaMenuItems.find('schema_menu_item_id');

// Check if it's a folder
if (schemaMenuItem.kind === 'folder') {
  console.log(`Folder: ${schemaMenuItem.label}`);
  console.log(`Contains ${schemaMenuItem.children.length} items`);
} else {
  console.log(`Model: ${schemaMenuItem.label}`);
  console.log(`Links to: ${schemaMenuItem.item_type.id}`);
}
```

### create() - Create New Schema Menu Item

Create a new schema menu item (folder or model reference) in the schema navigation.

```typescript
// Create a top-level folder
const contentFolder = await client.schemaMenuItems.create({
  label: 'Content',
  kind: 'folder',
  position: 1
});

// Create a nested folder
const blogFolder = await client.schemaMenuItems.create({
  label: 'Blog',
  kind: 'folder',
  position: 1,
  parent: {
    type: 'schema_menu_item',
    id: contentFolder.id
  }
});

// Add a model to the navigation
const articleMenuItem = await client.schemaMenuItems.create({
  label: 'Articles',
  kind: 'item_type',
  position: 1,
  item_type: {
    type: 'item_type',
    id: 'article_model_id'
  },
  parent: {
    type: 'schema_menu_item',
    id: blogFolder.id
  }
});
```

### update() - Update Schema Menu Item

Update an existing schema menu item's properties.

```typescript
// Update label
const updated = await client.schemaMenuItems.update('schema_menu_item_id', {
  label: 'Updated Label'
});

// Move to different parent folder
const moved = await client.schemaMenuItems.update('schema_menu_item_id', {
  parent: {
    type: 'schema_menu_item',
    id: 'new_parent_folder_id'
  },
  position: 1
});

// Move to root level
const movedToRoot = await client.schemaMenuItems.update('schema_menu_item_id', {
  parent: null,
  position: 1
});
```

### destroy() - Delete Schema Menu Item

Delete a schema menu item from the navigation. Note: Deleting a folder moves its contents to the root level.

```typescript
await client.schemaMenuItems.destroy('schema_menu_item_id');
```

## Schema Menu Item Configuration Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | string | Display name in the menu (required) |
| `kind` | string | Either 'folder' or 'item_type' (required) |
| `position` | number | Order position within parent (optional) |
| `item_type` | object | Reference to model (required for kind='item_type') |
| `parent` | object | Parent folder reference (optional) |

## Schema Menu Item Patterns

### Department-Based Organization

```typescript
async function createDepartmentStructure() {
  // Create department folders
  const departments = {
    editorial: await client.schemaMenuItems.create({
      label: '📝 Editorial',
      kind: 'folder',
      position: 1
    }),
    marketing: await client.schemaMenuItems.create({
      label: '📈 Marketing',
      kind: 'folder',
      position: 2
    }),
    ecommerce: await client.schemaMenuItems.create({
      label: '🛍️ E-commerce',
      kind: 'folder',
      position: 3
    }),
    admin: await client.schemaMenuItems.create({
      label: '⚙️ Administration',
      kind: 'folder',
      position: 4
    })
  };
  
  // Assign models to departments
  const assignments = {
    editorial: ['article', 'author', 'category', 'tag'],
    marketing: ['landing_page', 'campaign', 'testimonial', 'case_study'],
    ecommerce: ['product', 'collection', 'discount', 'customer'],
    admin: ['user', 'role', 'site_settings', 'redirect']
  };
  
  for (const [dept, models] of Object.entries(assignments)) {
    const folder = departments[dept];
    for (let i = 0; i < models.length; i++) {
      await client.schemaMenuItems.create({
        label: models[i].charAt(0).toUpperCase() + models[i].slice(1).replace(/_/g, ' '),
        kind: 'item_type',
        position: i + 1,
        item_type: { type: 'item_type', id: models[i] },
        parent: { type: 'schema_menu_item', id: folder.id }
      });
    }
  }
}
```

### Content Structure Organization

```typescript
const sections = [
  { label: 'Website', position: 1 },
  { label: 'Blog', position: 2 },
  { label: 'Shop', position: 3 },
  { label: 'Settings', position: 4 }
];

const folders = {};
for (const section of sections) {
  folders[section.label] = await client.schemaMenuItems.create({
    ...section,
    kind: 'folder'
  });
}

// Organize models into sections
const modelOrganization = {
  'Website': ['home_page', 'landing_page', 'about_page'],
  'Blog': ['blog_post', 'blog_category', 'blog_author'],
  'Shop': ['product', 'product_category', 'order'],
  'Settings': ['site_settings', 'navigation', 'footer']
};

for (const [folderName, modelApiKeys] of Object.entries(modelOrganization)) {
  const folder = folders[folderName];
  
  for (let i = 0; i < modelApiKeys.length; i++) {
    await client.schemaMenuItems.create({
      label: modelApiKeys[i].replace(/_/g, ' ').replace(/\b\w/g, l => l.toUpperCase()),
      kind: 'item_type',
      position: i + 1,
      item_type: { type: 'item_type', id: modelApiKeys[i] },
      parent: { type: 'schema_menu_item', id: folder.id }
    });
  }
}
```

### Menu Cleanup

```typescript
async function cleanupMenu() {
  const menuItems = await client.schemaMenuItems.list();
  
  // Find empty folders
  const folders = menuItems.filter(item => item.kind === 'folder');
  const emptyFolders = folders.filter(folder => 
    !menuItems.some(item => item.parent?.id === folder.id)
  );
  
  // Delete empty folders
  for (const folder of emptyFolders) {
    console.log(`Removing empty folder: ${folder.label}`);
    await client.schemaMenuItems.destroy(folder.id);
  }
  
  // Find models not in menu
  const itemTypes = await client.itemTypes.list();
  const menuModelIds = menuItems
    .filter(item => item.kind === 'item_type')
    .map(item => item.item_type.id);
  
  const orphanedModels = itemTypes.filter(model => 
    !menuModelIds.includes(model.id) && !model.modular_block
  );
  
  // Create "Uncategorized" folder for orphans
  if (orphanedModels.length > 0) {
    const uncategorized = await client.schemaMenuItems.create({
      label: 'Uncategorized',
      kind: 'folder',
      position: 999
    });
    
    for (const model of orphanedModels) {
      await client.schemaMenuItems.create({
        label: model.name,
        kind: 'item_type',
        item_type: { type: 'item_type', id: model.id },
        parent: { type: 'schema_menu_item', id: uncategorized.id }
      });
    }
  }
}
```

## Schema Menu Item Object Structure

```typescript
interface SchemaMenuItem {
  id: string;
  label: string;
  position: number;
  kind: 'folder' | 'item_type';
  item_type?: {
    type: 'item_type';
    id: string;
  };
  parent?: {
    type: 'schema_menu_item';
    id: string;
  };
  children?: Array<{
    type: 'schema_menu_item';
    id: string;
  }>;
}
```

## Menu Item Best Practices

1. **Structure and Organization**: Limit nesting depth to 2-3 levels for better UX
2. **Performance**: Cache navigation in your frontend to minimize API calls
3. **Content Management**: Use dynamic links (item_type) over hardcoded URLs when possible
4. **Clear Labels**: Use descriptive, concise labels for menu items
5. **Logical Grouping**: Group related models in folders by domain or purpose
6. **Consistent Naming**: Use clear, descriptive labels for folders and items

---

# Plugin

Plugins extend DatoCMS functionality by adding custom field editors, sidebar panels, and other UI extensions. You can install plugins from the marketplace or develop your own custom plugins.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Install a plugin from URL
const plugin = await client.plugins.create({
  name: 'My Custom Plugin',
  description: 'Adds custom functionality',
  url: 'https://my-plugin.example.com',
  plugin_type: 'field_editor'
});
```

## API Reference

### Create Plugin

Install a new plugin in your project.

```typescript
// Install a field editor plugin
const fieldPlugin = await client.plugins.create({
  name: 'Color Picker Plus',
  description: 'Advanced color picker with palette support',
  url: 'https://color-picker-plus.example.com',
  plugin_type: 'field_editor',
  field_types: ['color'], // Which field types this plugin can handle
  permissions: ['currentUserAccessToken'], // Required permissions
  parameter_definitions: {
    palette: {
      type: 'string',
      required: false,
      default: 'material',
      hint: 'Choose a color palette'
    }
  }
});

// Install a sidebar plugin
const sidebarPlugin = await client.plugins.create({
  name: 'SEO Assistant',
  description: 'Real-time SEO analysis and suggestions',
  url: 'https://seo-assistant.example.com',
  plugin_type: 'sidebar',
  permissions: ['currentUserAccessToken', 'environment'],
  parameter_definitions: {
    targetKeywords: {
      type: 'string',
      required: false,
      hint: 'Comma-separated keywords to optimize for'
    }
  }
});

// Install from npm package
const npmPlugin = await client.plugins.create({
  package_name: 'datocms-plugin-star-rating',
  name: 'Star Rating',
  description: 'Star rating field editor',
  url: 'https://unpkg.com/datocms-plugin-star-rating/dist/index.html',
  plugin_type: 'field_editor',
  field_types: ['integer', 'float'],
  package_version: '0.1.5'
});
```

**Parameters:**
- `name` (string, required): Plugin display name
- `description` (string, optional): Plugin description
- `url` (string, required): Plugin entry point URL
- `plugin_type` (string, required): Type - 'field_editor' or 'sidebar'
- `field_types` (array, optional): For field editors - supported field types
- `permissions` (array, optional): Required permissions
- `parameter_definitions` (object, optional): Configuration parameters
- `package_name` (string, optional): NPM package name
- `package_version` (string, optional): Package version

**Returns:** The created plugin object

### Update Plugin

Modify plugin settings or update to a new version.

```typescript
// Update plugin configuration
const updated = await client.plugins.update('plugin-id', {
  name: 'Updated Plugin Name',
  description: 'New description',
  parameters: {
    apiKey: 'new-api-key',
    enableFeature: true
  }
});

// Update to new version
const versionUpdate = await client.plugins.update('plugin-id', {
  package_version: '0.2.0',
  url: 'https://unpkg.com/my-plugin@0.2.0/dist/index.html'
});

// Update permissions
const permissionUpdate = await client.plugins.update('plugin-id', {
  permissions: ['currentUserAccessToken', 'environment', 'site']
});
```

**Parameters:**
- `pluginId` (string, required): The plugin ID
- Update parameters (all optional):
  - `name`: New display name
  - `description`: New description
  - `url`: New plugin URL
  - `parameters`: Plugin configuration
  - `package_version`: New version
  - `permissions`: Updated permissions

**Returns:** The updated plugin object

### List Plugins

Get all installed plugins in your project.

```typescript
// Get all plugins
const plugins = await client.plugins.list();

// Filter by type
const fieldEditors = plugins.filter(p => p.plugin_type === 'field_editor');
const sidebarPanels = plugins.filter(p => p.plugin_type === 'sidebar');

// Find plugins supporting specific field type
const colorPlugins = plugins.filter(p => 
  p.field_types?.includes('color')
);
```

**Returns:** Array of plugin objects

### Find Plugin

Retrieve a specific plugin by ID.

```typescript
const plugin = await client.plugins.find('plugin-id');

console.log(`Plugin: ${plugin.name}`);
console.log(`Type: ${plugin.plugin_type}`);
console.log(`Version: ${plugin.package_version || 'custom'}`);
```

**Parameters:**
- `pluginId` (string, required): The plugin ID

**Returns:** The plugin object

### Delete Plugin

Remove a plugin from your project.

```typescript
// Check if plugin is in use first
const fields = await client.plugins.fields('plugin-id');

if (fields.length > 0) {
  console.log(`Plugin is used by ${fields.length} fields`);
  // Handle field reassignment before deletion
}

// Delete the plugin
await client.plugins.destroy('plugin-id');
```

**Parameters:**
- `pluginId` (string, required): The plugin ID

**Note:** Deleting a plugin will reset any fields using it to default editor

### Get Plugin Fields

Find all fields using a specific plugin.

```typescript
const fields = await client.plugins.fields('plugin-id');

// Group by model
const fieldsByModel = {};
for (const field of fields) {
  const modelId = field.item_type.id;
  if (!fieldsByModel[modelId]) {
    fieldsByModel[modelId] = [];
  }
  fieldsByModel[modelId].push(field);
}

console.log('Fields using this plugin:', fieldsByModel);
```

**Parameters:**
- `pluginId` (string, required): The plugin ID

**Returns:** Array of field objects using the plugin

## Plugin Development

### Field Editor Plugin

Create custom field editors:

```typescript
// Example: Install a custom markdown editor
const markdownPlugin = await client.plugins.create({
  name: 'Advanced Markdown Editor',
  description: 'Markdown editor with live preview and toolbar',
  url: 'https://my-markdown-editor.netlify.app',
  plugin_type: 'field_editor',
  field_types: ['text', 'string'],
  permissions: ['currentUserAccessToken'],
  parameter_definitions: {
    toolbar: {
      type: 'boolean',
      required: false,
      default: true,
      hint: 'Show formatting toolbar'
    },
    preview: {
      type: 'boolean',
      required: false,
      default: true,
      hint: 'Show live preview'
    },
    height: {
      type: 'integer',
      required: false,
      default: 300,
      hint: 'Editor height in pixels'
    }
  }
});

// Configure field to use the plugin
await client.fields.update('field-id', {
  appearance: {
    editor: plugin.id,
    parameters: {
      toolbar: true,
      preview: true,
      height: 400
    },
    addons: []
  }
});
```

### Sidebar Panel Plugin

Add custom sidebar panels:

```typescript
// Install a sidebar analytics plugin
const analyticsPlugin = await client.plugins.create({
  name: 'Content Analytics',
  description: 'View content performance metrics',
  url: 'https://analytics-plugin.example.com',
  plugin_type: 'sidebar',
  permissions: [
    'currentUserAccessToken',
    'environment',
    'currentItem',
    'currentModel'
  ],
  parameter_definitions: {
    analyticsProvider: {
      type: 'string',
      required: true,
      default: 'google',
      enum: ['google', 'plausible', 'matomo'],
      hint: 'Choose your analytics provider'
    },
    trackingId: {
      type: 'string',
      required: true,
      hint: 'Analytics tracking ID'
    }
  }
});
```

## Plugin Configuration

### Parameter Definitions

Define plugin configuration options:

```typescript
const parameterDefinitions = {
  // String parameter
  apiKey: {
    type: 'string',
    required: true,
    hint: 'Your API key',
    default: ''
  },
  
  // Boolean parameter
  enableFeature: {
    type: 'boolean',
    required: false,
    default: false,
    hint: 'Enable advanced features'
  },
  
  // Integer parameter
  maxItems: {
    type: 'integer',
    required: false,
    default: 10,
    hint: 'Maximum number of items'
  },
  
  // Enum parameter
  theme: {
    type: 'string',
    required: false,
    default: 'light',
    enum: ['light', 'dark', 'auto'],
    hint: 'Choose UI theme'
  },
  
  // JSON parameter
  customConfig: {
    type: 'json',
    required: false,
    hint: 'Advanced configuration (JSON)'
  }
};
```

### Plugin Permissions

Available permissions for plugins:

```typescript
const permissions = [
  'currentUserAccessToken',     // Access to API token
  'environment',                // Current environment info
  'site',                      // Site configuration
  'currentItem',               // Current item data (sidebar)
  'currentModel',              // Current model info
  'currentUser',               // User information
  'currentRole',               // User role and permissions
  'manageMenuItems',           // Manage navigation menu
  'manageUploadCollections',   // Manage upload collections
  'performSiteSearch',         // Search across content
  'loadItemTypeFields',        // Load model field definitions
  'loadFieldsInLocale',        // Load field values by locale
  'autoResizeHeight'           // Auto-resize plugin height
];
```

## Plugin Management

### Plugin Marketplace

Install plugins from the DatoCMS marketplace:

```typescript
// Popular marketplace plugins
const popularPlugins = [
  {
    name: 'Conditional Fields',
    package: 'datocms-plugin-conditional-fields',
    description: 'Show/hide fields based on conditions'
  },
  {
    name: 'Computed Fields',
    package: 'datocms-plugin-computed-fields',
    description: 'Calculate field values automatically'
  },
  {
    name: 'Web Previews',
    package: 'datocms-plugin-web-previews',
    description: 'Preview content on your website'
  }
];

// Install from marketplace
async function installMarketplacePlugin(packageName: string) {
  // Fetch package info from npm
  const response = await fetch(`https://registry.npmjs.org/${packageName}/latest`);
  const packageInfo = await response.json();
  
  return client.plugins.create({
    package_name: packageName,
    name: packageInfo.datoCmsPlugin?.title || packageInfo.name,
    description: packageInfo.description,
    url: `https://unpkg.com/${packageName}@${packageInfo.version}/dist/index.html`,
    plugin_type: packageInfo.datoCmsPlugin?.pluginType || 'field_editor',
    field_types: packageInfo.datoCmsPlugin?.fieldTypes,
    permissions: packageInfo.datoCmsPlugin?.permissions || [],
    package_version: packageInfo.version
  });
}
```

### Plugin Updates

Check and install plugin updates:

```typescript
async function checkPluginUpdates() {
  const plugins = await client.plugins.list();
  const updates = [];
  
  for (const plugin of plugins) {
    if (!plugin.package_name) continue;
    
    try {
      // Check npm for latest version
      const response = await fetch(
        `https://registry.npmjs.org/${plugin.package_name}/latest`
      );
      const latestInfo = await response.json();
      
      if (latestInfo.version !== plugin.package_version) {
        updates.push({
          plugin,
          currentVersion: plugin.package_version,
          latestVersion: latestInfo.version,
          updateUrl: `https://unpkg.com/${plugin.package_name}@${latestInfo.version}/dist/index.html`
        });
      }
    } catch (error) {
      console.error(`Failed to check updates for ${plugin.name}`);
    }
  }
  
  return updates;
}

// Apply updates
async function updatePlugins() {
  const updates = await checkPluginUpdates();
  
  for (const update of updates) {
    console.log(`Updating ${update.plugin.name} from ${update.currentVersion} to ${update.latestVersion}`);
    
    await client.plugins.update(update.plugin.id, {
      package_version: update.latestVersion,
      url: update.updateUrl
    });
  }
}
```

### Plugin Usage Analytics

Track which plugins are most used:

```typescript
async function analyzePluginUsage() {
  const plugins = await client.plugins.list();
  const usage = [];
  
  for (const plugin of plugins) {
    const fields = await client.plugins.fields(plugin.id);
    
    // Count by model
    const modelUsage = {};
    fields.forEach(field => {
      const modelId = field.item_type.id;
      modelUsage[modelId] = (modelUsage[modelId] || 0) + 1;
    });
    
    usage.push({
      plugin: plugin.name,
      type: plugin.plugin_type,
      totalFields: fields.length,
      modelsUsing: Object.keys(modelUsage).length,
      usage: modelUsage
    });
  }
  
  // Sort by usage
  return usage.sort((a, b) => b.totalFields - a.totalFields);
}
```

## Advanced Patterns

### Plugin Migration

Migrate fields between plugins:

```typescript
async function migrateFieldsToNewPlugin(
  oldPluginId: string,
  newPluginId: string,
  fieldMapping?: (field: any) => any
) {
  // Get all fields using old plugin
  const fields = await client.plugins.fields(oldPluginId);
  
  console.log(`Migrating ${fields.length} fields...`);
  
  for (const field of fields) {
    // Apply custom mapping if provided
    const newParameters = fieldMapping ? fieldMapping(field) : {};
    
    await client.fields.update(field.id, {
      appearance: {
        editor: newPluginId,
        parameters: newParameters,
        addons: field.appearance.addons || []
      }
    });
    
    console.log(`✓ Migrated field: ${field.label}`);
  }
  
  return fields.length;
}
```

### Plugin Development Setup

Configure local plugin development:

```typescript
// Development plugin configuration
const devPlugin = await client.plugins.create({
  name: 'My Plugin (Dev)',
  description: 'Development version',
  url: 'http://localhost:3000', // Local development server
  plugin_type: 'field_editor',
  field_types: ['string', 'text'],
  permissions: ['currentUserAccessToken', 'environment']
});

// Production plugin
const prodPlugin = await client.plugins.create({
  name: 'My Plugin',
  description: 'Production version',
  url: 'https://my-plugin.netlify.app',
  plugin_type: 'field_editor',
  field_types: ['string', 'text'],
  permissions: ['currentUserAccessToken', 'environment']
});

// Switch between dev and prod
async function useDevPlugin(fieldId: string, isDev: boolean) {
  const pluginId = isDev ? devPlugin.id : prodPlugin.id;
  
  await client.fields.update(fieldId, {
    appearance: {
      editor: pluginId,
      parameters: {},
      addons: []
    }
  });
}
```

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.plugins.create({
    name: 'Test Plugin',
    url: 'invalid-url',
    plugin_type: 'invalid_type'
  });
} catch (error) {
  if (error instanceof ApiError) {
    const urlError = error.findError('url');
    if (urlError?.code === 'VALIDATION_FORMAT') {
      console.log('Invalid plugin URL');
    }
    
    const typeError = error.findError('plugin_type');
    if (typeError?.code === 'VALIDATION_INCLUSION') {
      console.log('Invalid plugin type - use field_editor or sidebar');
    }
  }
}

// Handle plugin in use
try {
  await client.plugins.destroy('plugin-in-use');
} catch (error) {
  if (error instanceof ApiError && error.response.status === 422) {
    console.log('Cannot delete plugin: still in use by fields');
    
    // List fields using it
    const fields = await client.plugins.fields('plugin-in-use');
    console.log(`Used by ${fields.length} fields`);
  }
}
```

## Plugin Object Structure

```typescript
interface Plugin {
  id: string;
  name: string;
  description?: string;
  url: string;
  plugin_type: 'field_editor' | 'sidebar';
  field_types?: string[]; // For field editors
  permissions: string[];
  parameters?: Record<string, any>;
  parameter_definitions?: Record<string, {
    type: 'string' | 'boolean' | 'integer' | 'float' | 'json';
    required?: boolean;
    default?: any;
    hint?: string;
    enum?: string[];
  }>;
  package_name?: string;
  package_version?: string;
  created_at: string;
  updated_at: string;
}
```

## Related Resources

- [Fields](./field.md) - Configure fields to use plugins
- [Create Field Editor](../../05-build-plugins/create-field-editor/basic-field-plugin.md) - Build custom field editors
- [Add Sidebar Panel](../../05-build-plugins/add-sidebar-panel/define-sidebar-panel.md) - Build sidebar panels
- Plugin Configuration - Plugin settings guide

---

# Plugin Methods - Complete Reference

This document contains all methods available for the Plugin resource in the DatoCMS CMA Client.

## Available Methods for client.plugins

### list() - Retrieve All Plugins

Retrieves all plugins installed in the project.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get all plugins
const plugins = await client.plugins.list();

// Filter by plugin type
const fieldEditorPlugins = plugins.filter(p => 
  p.field_types.length > 0 || p.fieldset_types.length > 0
);

const sidebarPlugins = plugins.filter(p => 
  p.plugin_type === 'sidebar'
);

// Get plugins from marketplace
const marketplacePlugins = plugins.filter(p => 
  p.package_name && p.package_name.startsWith('datocms-plugin-')
);

// Get private/custom plugins
const customPlugins = plugins.filter(p => 
  !p.package_name || p.url
);

// Get raw response
const rawResponse = await client.plugins.rawList();
```

### find() - Get Single Plugin

Retrieves a specific plugin by ID.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get plugin by ID
const plugin = await client.plugins.find('plugin_id');

// Plugin structure includes:
// {
//   id: 'plugin_id',
//   type: 'plugin',
//   attributes: {
//     name: 'SEO Analysis',
//     description: 'Analyze and improve SEO for your content',
//     url: 'https://plugins.datocms.com/seo-analysis',
//     package_name: 'datocms-plugin-seo-analysis',
//     package_version: '1.2.3',
//     permissions: ['currentUserAccessToken'],
//     overrides_item_form_sidebar: false,
//     field_types: ['seo_analysis'],
//     fieldset_types: [],
//     plugin_type: 'field_editor',
//     parameters: {
//       apiToken: 'default_token',
//       minTitleLength: 30,
//       maxTitleLength: 60
//     },
//     parameter_definitions: {
//       apiToken: {
//         type: 'string',
//         required: true,
//         label: 'API Token',
//         hint: 'Token for SEO analysis service'
//       },
//       minTitleLength: {
//         type: 'integer',
//         required: false,
//         default: 30,
//         label: 'Minimum Title Length'
//       },
//       maxTitleLength: {
//         type: 'integer',
//         required: false,
//         default: 60,
//         label: 'Maximum Title Length'
//       }
//     }
//   }
// }

// Error handling
try {
  const plugin = await client.plugins.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Plugin not found');
  }
}
```

### create() - Install New Plugin

Installs a new plugin in the project.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Install plugin from npm package
const npmPlugin = await client.plugins.create({
  package_name: 'datocms-plugin-seo-analysis',
  parameters: {
    apiToken: 'your_seo_service_token',
    minTitleLength: 30,
    maxTitleLength: 60
  }
});

// Install plugin from URL
const urlPlugin = await client.plugins.create({
  name: 'Custom Field Editor',
  url: 'https://my-plugin.example.com',
  parameters: {
    endpoint: 'https://api.example.com',
    apiKey: 'secret_key'
  }
});

// Install marketplace plugin with specific version
const versionedPlugin = await client.plugins.create({
  package_name: 'datocms-plugin-star-rating',
  package_version: '0.2.5',
  parameters: {
    maxStars: 5,
    allowHalf: true
  }
});

// Install field editor plugin
const fieldEditorPlugin = await client.plugins.create({
  name: 'Color Picker',
  url: 'https://color-picker-plugin.example.com',
  field_types: ['color_picker'],
  parameters: {
    format: 'hex',
    showAlpha: false
  }
});

// Install sidebar plugin
const sidebarPlugin = await client.plugins.create({
  name: 'Preview Links',
  url: 'https://preview-plugin.example.com',
  plugin_type: 'sidebar',
  overrides_item_form_sidebar: false,
  parameters: {
    previewUrl: 'https://preview.example.com'
  }
});

// Install plugin with permissions
const permissionedPlugin = await client.plugins.create({
  package_name: 'datocms-plugin-computed-fields',
  permissions: [
    'currentUserAccessToken',
    'environment',
    'site'
  ],
  parameters: {
    computeOnSave: true
  }
});

// Install fieldset plugin
const fieldsetPlugin = await client.plugins.create({
  name: 'Address Fieldset',
  url: 'https://address-fieldset.example.com',
  fieldset_types: ['address'],
  parameters: {
    defaultCountry: 'US',
    includeMap: true
  }
});

// Error handling
try {
  const plugin = await client.plugins.create({
    package_name: 'non-existent-plugin'
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Plugin validation errors:', error.response.data.errors);
  }
}
```

### update() - Update Plugin Configuration

Updates an existing plugin's settings and parameters.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Update plugin parameters
const updated = await client.plugins.update('plugin_id', {
  parameters: {
    apiToken: 'new_token',
    maxItems: 10,
    enableCache: true
  }
});

// Update plugin version
const upgradedPlugin = await client.plugins.update('plugin_id', {
  package_version: '1.3.0'
});

// Update plugin name and description
const renamedPlugin = await client.plugins.update('plugin_id', {
  name: 'Updated Plugin Name',
  description: 'New description for the plugin'
});

// Update plugin URL (for custom plugins)
const urlUpdated = await client.plugins.update('plugin_id', {
  url: 'https://new-plugin-url.example.com'
});

// Update field types
const fieldTypesUpdated = await client.plugins.update('plugin_id', {
  field_types: ['custom_text', 'custom_number']
});

// Update permissions
const permissionsUpdated = await client.plugins.update('plugin_id', {
  permissions: [
    'currentUserAccessToken',
    'environment',
    'site',
    'currentItem'
  ]
});

// Complex update
const complexUpdate = await client.plugins.update('plugin_id', {
  name: 'Enhanced SEO Plugin',
  package_version: '2.0.0',
  parameters: {
    apiToken: 'new_api_token',
    enableAutoSuggestions: true,
    maxKeywords: 10,
    targetDensity: 2.5
  },
  permissions: ['currentUserAccessToken', 'environment']
});

// Error handling
try {
  const updated = await client.plugins.update('plugin_id', {
    package_name: 'different-plugin' // Cannot change package
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Cannot change plugin package');
  }
}
```

### destroy() - Uninstall Plugin

Removes a plugin from the project.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Uninstall plugin
await client.plugins.destroy('plugin_id');

// Uninstall with error handling
try {
  await client.plugins.destroy('plugin_id');
  console.log('Plugin uninstalled successfully');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Plugin not found');
  } else if (error.response?.status === 422) {
    console.error('Plugin is in use and cannot be removed');
  }
}

// Uninstall unused plugins
async function uninstallUnusedPlugins() {
  const plugins = await client.plugins.list();
  const unused = [];
  
  for (const plugin of plugins) {
    const fields = await client.plugins.fields(plugin.id);
    if (fields.length === 0) {
      unused.push(plugin);
    }
  }
  
  const results = await Promise.allSettled(
    unused.map(plugin => client.plugins.destroy(plugin.id))
  );
  
  return {
    total: unused.length,
    uninstalled: results.filter(r => r.status === 'fulfilled').length,
    failed: results.filter(r => r.status === 'rejected').length
  };
}
```

### fields() - Get Fields Using Plugin

Retrieves all fields that are using a specific plugin.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get fields using the plugin
const fields = await client.plugins.fields('plugin_id');

// Field usage structure:
// [
//   {
//     id: 'field_id',
//     type: 'field',
//     attributes: {
//       label: 'SEO Settings',
//       api_key: 'seo_settings',
//       field_type: 'seo_analysis',
//       appearance: {
//         editor: 'seo_analysis',
//         parameters: {
//           showPreview: true
//         },
//         addons: []
//       }
//     },
//     relationships: {
//       item_type: {
//         data: { type: 'item_type', id: 'blog_post' }
//       }
//     }
//   }
// ]

// Analyze plugin usage
async function analyzePluginUsage(pluginId) {
  const plugin = await client.plugins.find(pluginId);
  const fields = await client.plugins.fields(pluginId);
  
  // Group fields by model
  const usage = {};
  for (const field of fields) {
    const modelId = field.relationships.item_type.data.id;
    if (!usage[modelId]) {
      usage[modelId] = [];
    }
    usage[modelId].push({
      fieldLabel: field.label,
      fieldApiKey: field.api_key
    });
  }
  
  // Get model details
  const models = await client.itemTypes.list();
  const modelMap = models.reduce((acc, model) => {
    acc[model.id] = model.name;
    return acc;
  }, {});
  
  return {
    plugin: plugin.name,
    totalFields: fields.length,
    models: Object.entries(usage).map(([modelId, fields]) => ({
      modelName: modelMap[modelId],
      modelId,
      fields
    }))
  };
}

// Check before uninstalling
async function canUninstallPlugin(pluginId) {
  const fields = await client.plugins.fields(pluginId);
  
  if (fields.length > 0) {
    console.log(`Plugin is used by ${fields.length} fields:`);
    fields.forEach(field => {
      console.log(`- ${field.label} (${field.api_key})`);
    });
    return false;
  }
  
  return true;
}
```

## Plugin Configuration Patterns

### Plugin Parameter Management

```typescript
async function updatePluginConfig(pluginId, newConfig) {
  // Get current plugin configuration
  const plugin = await client.plugins.find(pluginId);
  const currentParams = plugin.parameters || {};
  
  // Merge with new configuration
  const updatedParams = {
    ...currentParams,
    ...newConfig
  };
  
  // Validate against parameter definitions
  const paramDefs = plugin.parameter_definitions || {};
  const errors = [];
  
  for (const [key, def] of Object.entries(paramDefs)) {
    if (def.required && !updatedParams[key]) {
      errors.push(`Missing required parameter: ${key}`);
    }
    
    if (updatedParams[key] !== undefined) {
      // Type validation
      if (def.type === 'integer' && !Number.isInteger(updatedParams[key])) {
        errors.push(`${key} must be an integer`);
      } else if (def.type === 'boolean' && typeof updatedParams[key] !== 'boolean') {
        errors.push(`${key} must be a boolean`);
      }
    }
  }
  
  if (errors.length > 0) {
    throw new Error(`Validation errors: ${errors.join(', ')}`);
  }
  
  // Update plugin
  return client.plugins.update(pluginId, {
    parameters: updatedParams
  });
}
```

### Plugin Migration

```typescript
async function migratePlugin(oldPluginId, newPluginPackage) {
  // Get fields using old plugin
  const fields = await client.plugins.fields(oldPluginId);
  
  if (fields.length === 0) {
    // Simple case - no fields to migrate
    await client.plugins.destroy(oldPluginId);
    return client.plugins.create({
      package_name: newPluginPackage
    });
  }
  
  // Install new plugin
  const newPlugin = await client.plugins.create({
    package_name: newPluginPackage
  });
  
  // Migrate fields to use new plugin
  const migrations = [];
  for (const field of fields) {
    try {
      await client.fields.update(field.id, {
        appearance: {
          editor: newPlugin.field_types[0], // Assuming compatible type
          parameters: field.appearance.parameters,
          addons: field.appearance.addons
        }
      });
      migrations.push({
        field: field.api_key,
        status: 'success'
      });
    } catch (error) {
      migrations.push({
        field: field.api_key,
        status: 'failed',
        error: error.message
      });
    }
  }
  
  // Remove old plugin if all migrations successful
  const allSuccess = migrations.every(m => m.status === 'success');
  if (allSuccess) {
    await client.plugins.destroy(oldPluginId);
  }
  
  return {
    newPluginId: newPlugin.id,
    migrations
  };
}
```

### Plugin Development Setup

```typescript
async function setupDevelopmentPlugin(config) {
  // Create development plugin pointing to local server
  const devPlugin = await client.plugins.create({
    name: `${config.name} (Dev)`,
    url: config.developmentUrl || 'http://localhost:3000',
    field_types: config.fieldTypes || [],
    fieldset_types: config.fieldsetTypes || [],
    plugin_type: config.pluginType || 'field_editor',
    permissions: config.permissions || ['currentUserAccessToken'],
    parameters: config.defaultParameters || {},
    parameter_definitions: config.parameterDefinitions || {}
  });
  
  // Create test field if requested
  if (config.createTestField) {
    const testModel = await client.itemTypes.create({
      name: 'Plugin Test Model',
      api_key: 'plugin_test_model'
    });
    
    const testField = await client.fields.create(testModel.id, {
      label: 'Test Field',
      api_key: 'test_field',
      field_type: devPlugin.field_types[0],
      appearance: {
        editor: devPlugin.field_types[0],
        parameters: config.testFieldParameters || {}
      }
    });
    
    return {
      plugin: devPlugin,
      testModel,
      testField
    };
  }
  
  return { plugin: devPlugin };
}
```

### Plugin Marketplace Integration

```typescript
async function searchAndInstallPlugin(searchTerm, autoInstall = false) {
  // Note: This is a conceptual example - actual marketplace API may differ
  const marketplacePlugins = [
    {
      package: 'datocms-plugin-seo-analysis',
      name: 'SEO Analysis',
      description: 'Analyze SEO for your content'
    },
    {
      package: 'datocms-plugin-star-rating',
      name: 'Star Rating',
      description: 'Add star ratings to your content'
    },
    {
      package: 'datocms-plugin-color-picker',
      name: 'Color Picker',
      description: 'Advanced color picker field'
    }
  ];
  
  // Search plugins
  const matches = marketplacePlugins.filter(p => 
    p.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
    p.description.toLowerCase().includes(searchTerm.toLowerCase())
  );
  
  if (matches.length === 0) {
    console.log('No plugins found matching:', searchTerm);
    return null;
  }
  
  console.log(`Found ${matches.length} plugins:`);
  matches.forEach((p, i) => {
    console.log(`${i + 1}. ${p.name} - ${p.description}`);
  });
  
  if (autoInstall && matches.length === 1) {
    // Auto-install if only one match
    return client.plugins.create({
      package_name: matches[0].package
    });
  }
  
  return matches;
}
```

### Plugin Configuration Templates

```typescript
const pluginTemplates = {
  seo: {
    package_name: 'datocms-plugin-seo-analysis',
    parameters: {
      minTitleLength: 30,
      maxTitleLength: 60,
      minDescriptionLength: 120,
      maxDescriptionLength: 160,
      targetKeywordDensity: 2
    }
  },
  
  computed: {
    package_name: 'datocms-plugin-computed-fields',
    permissions: ['currentUserAccessToken', 'environment'],
    parameters: {
      computeOnSave: true,
      showInForm: false
    }
  },
  
  preview: {
    name: 'Content Preview',
    url: 'https://preview-plugin.example.com',
    plugin_type: 'sidebar',
    parameters: {
      previewUrl: 'https://preview.example.com',
      openInNewTab: true
    }
  }
};

async function installFromTemplate(templateName) {
  const template = pluginTemplates[templateName];
  if (!template) {
    throw new Error(`Unknown template: ${templateName}`);
  }
  
  return client.plugins.create(template);
}
```

### Plugin Health Check

```typescript
async function checkPluginHealth() {
  const plugins = await client.plugins.list();
  const health = {
    total: plugins.length,
    healthy: 0,
    warnings: [],
    errors: []
  };
  
  for (const plugin of plugins) {
    const issues = [];
    
    // Check if plugin is outdated
    if (plugin.package_name && plugin.package_version) {
      // Would need to check against latest version
      // This is a simplified example
      const majorVersion = parseInt(plugin.package_version.split('.')[0]);
      if (majorVersion < 1) {
        issues.push({
          type: 'warning',
          message: 'Plugin is using beta version'
        });
      }
    }
    
    // Check if custom plugin URL is accessible
    if (plugin.url && !plugin.package_name) {
      if (plugin.url.startsWith('http://') && 
          !plugin.url.includes('localhost')) {
        issues.push({
          type: 'error',
          message: 'Custom plugin using insecure HTTP'
        });
      }
    }
    
    // Check field usage
    try {
      const fields = await client.plugins.fields(plugin.id);
      if (fields.length === 0) {
        issues.push({
          type: 'warning',
          message: 'Plugin installed but not used by any fields'
        });
      }
    } catch (error) {
      issues.push({
        type: 'error',
        message: 'Failed to check field usage'
      });
    }
    
    if (issues.length === 0) {
      health.healthy++;
    } else {
      issues.forEach(issue => {
        if (issue.type === 'warning') {
          health.warnings.push({
            plugin: plugin.name,
            issue: issue.message
          });
        } else {
          health.errors.push({
            plugin: plugin.name,
            issue: issue.message
          });
        }
      });
    }
  }
  
  return health;
}
```

## Error Handling

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Comprehensive error handling
async function safePluginOperation() {
  try {
    const plugin = await client.plugins.create({
      package_name: 'datocms-plugin-example',
      parameters: {
        apiKey: 'test_key'
      }
    });
    return plugin;
  } catch (error) {
    if (error.response) {
      switch (error.response.status) {
        case 422:
          const validationErrors = error.response.data.errors;
          validationErrors.forEach(err => {
            if (err.source?.pointer?.includes('package_name')) {
              console.error('Invalid package name:', err.detail);
            } else if (err.source?.pointer?.includes('parameters')) {
              console.error('Invalid parameters:', err.detail);
            } else if (err.source?.pointer?.includes('url')) {
              console.error('Invalid plugin URL:', err.detail);
            }
          });
          break;
        case 403:
          console.error('Insufficient permissions to manage plugins');
          break;
        case 409:
          console.error('Plugin already installed');
          break;
        default:
          console.error('Unexpected error:', error.response.status);
      }
    }
    throw error;
  }
}

// Validate plugin configuration
function validatePluginConfig(pluginData) {
  const errors = [];
  
  // Must have either package_name or url
  if (!pluginData.package_name && !pluginData.url) {
    errors.push('Either package_name or url is required');
  }
  
  // Cannot have both
  if (pluginData.package_name && pluginData.url) {
    errors.push('Cannot specify both package_name and url');
  }
  
  // Validate URL if present
  if (pluginData.url) {
    try {
      const url = new URL(pluginData.url);
      if (url.protocol !== 'https:' && !url.hostname.includes('localhost')) {
        errors.push('Plugin URL should use HTTPS in production');
      }
    } catch {
      errors.push('Invalid plugin URL');
    }
  }
  
  // Validate field types
  if (pluginData.field_types && !Array.isArray(pluginData.field_types)) {
    errors.push('field_types must be an array');
  }
  
  return errors;
}
```

## Best Practices

1. **Version Management**: Keep plugins updated to latest stable versions
2. **Parameter Validation**: Validate parameters against definitions before updating
3. **Permission Scope**: Only request necessary permissions
4. **Development/Production**: Use separate plugins for development and production
5. **Documentation**: Document custom plugin parameters and usage
6. **Error Handling**: Implement proper error handling for plugin operations
7. **Usage Monitoring**: Regularly check which fields use each plugin
8. **Backup**: Document plugin configurations before major updates

## Related Resources

- **[Field Methods](/LLM_DOCS/01-cma-client/01-content-management/field-methods.md)**: Configure fields to use plugins
- **[Item Type Methods](/LLM_DOCS/01-cma-client/01-content-management/item-type-methods.md)**: Models that contain plugin fields
- **[Plugin SDK Documentation](/LLM_DOCS/03-plugin-sdk/README.md)**: Build custom plugins

---

# PublicInfo

The PublicInfo resource provides unauthenticated access to basic project information, including branding, theme colors, and SSO configuration. This endpoint is intentionally public, allowing external applications to retrieve project metadata without requiring API authentication.

## Available Operations

### Retrieve Public Information

```javascript
// Without authentication
const client = new CmaClient({ apiToken: null });
const info = await client.publicInfo.find();

// With authentication (includes extras)
const authClient = new CmaClient({ apiToken: 'YOUR_TOKEN' });
const fullInfo = await authClient.publicInfo.find();
```

**Returns:** Project's public information

## The PublicInfo Object

```typescript
{
  id: string;                              // Public info identifier
  type: 'public_info';                     // Always 'public_info'
  name: string;                            // Project name
  sso_saml_init_url: string | null;        // SSO initialization URL
  logo_url: string | null;                 // Custom logo URL
  white_label: boolean;                    // White-label status
  custom_i18n_messages_template_url: string | null;  // Custom translations URL
  theme: {                                 // Project theme colors
    primary_color: Color;                  // Primary brand color
    light_color: Color;                    // Light theme color
    accent_color: Color;                   // Accent color
    dark_color: Color;                     // Dark theme color
  };
  extras: null | {                         // Additional info (auth required)
    blocks_depth: number;                  // Max nested blocks depth
    blocks_per_item: number;               // Max blocks per item
    maximum_single_upload_bytes: number;   // Max upload size
    [k: string]: unknown;                  // Other configuration
  };
}

// Color object structure
type Color = {
  red: number;    // 0-255
  green: number;  // 0-255
  blue: number;   // 0-255
  alpha: number;  // 0-1
};
```

## Common Use Cases

### 1. Custom Login Page

```javascript
// Build a branded login page using public info
async function createCustomLoginPage() {
  const client = new CmaClient({ apiToken: null });
  const info = await client.publicInfo.find();
  
  // Apply theme colors
  const theme = {
    primary: rgbaToHex(info.theme.primary_color),
    light: rgbaToHex(info.theme.light_color),
    accent: rgbaToHex(info.theme.accent_color),
    dark: rgbaToHex(info.theme.dark_color)
  };
  
  // Generate login page HTML
  const loginPageHtml = `
    <!DOCTYPE html>
    <html>
    <head>
      <title>Login - ${info.name}</title>
      <style>
        :root {
          --primary: ${theme.primary};
          --light: ${theme.light};
          --accent: ${theme.accent};
          --dark: ${theme.dark};
        }
        body {
          background-color: var(--light);
          color: var(--dark);
        }
        .login-button {
          background-color: var(--primary);
          color: white;
        }
        .login-button:hover {
          background-color: var(--accent);
        }
      </style>
    </head>
    <body>
      ${info.logo_url ? `<img src="${info.logo_url}" alt="${info.name} Logo">` : ''}
      <h1>Welcome to ${info.name}</h1>
      ${info.sso_saml_init_url ? 
        `<a href="${info.sso_saml_init_url}" class="login-button">Login with SSO</a>` :
        '<form><!-- Standard login form --></form>'
      }
    </body>
    </html>
  `;
  
  return loginPageHtml;
}

// Helper function to convert RGBA to hex
function rgbaToHex(color) {
  const toHex = (n) => Math.round(n).toString(16).padStart(2, '0');
  return `#${toHex(color.red)}${toHex(color.green)}${toHex(color.blue)}`;
}
```

### 2. SSO Integration Helper

```javascript
// Detect and handle SSO configuration
class SSOIntegration {
  constructor(projectUrl) {
    this.projectUrl = projectUrl;
    this.client = new CmaClient({ 
      apiToken: null,
      baseUrl: projectUrl 
    });
  }
  
  async checkSSOAvailability() {
    try {
      const info = await this.client.publicInfo.find();
      
      return {
        enabled: !!info.sso_saml_init_url,
        loginUrl: info.sso_saml_init_url,
        projectName: info.name,
        isWhiteLabeled: info.white_label
      };
    } catch (error) {
      console.error('Failed to check SSO:', error);
      return { enabled: false };
    }
  }
  
  async redirectToSSO() {
    const ssoInfo = await this.checkSSOAvailability();
    
    if (ssoInfo.enabled) {
      // Store return URL if needed
      sessionStorage.setItem('datocms_return_url', window.location.href);
      
      // Redirect to SSO
      window.location.href = ssoInfo.loginUrl;
    } else {
      throw new Error('SSO is not configured for this project');
    }
  }
  
  async getSSOLoginButton() {
    const ssoInfo = await this.checkSSOAvailability();
    
    if (!ssoInfo.enabled) {
      return null;
    }
    
    return {
      text: ssoInfo.isWhiteLabeled ? 
        `Login to ${ssoInfo.projectName}` : 
        'Login with DatoCMS SSO',
      url: ssoInfo.loginUrl,
      available: true
    };
  }
}

// Usage
const sso = new SSOIntegration('https://my-project.admin.datocms.com');
const ssoButton = await sso.getSSOLoginButton();

if (ssoButton) {
  console.log(`<a href="${ssoButton.url}">${ssoButton.text}</a>`);
}
```

### 3. White-Label Detection

```javascript
// Check white-label status and apply custom branding
async function applyWhiteLabelBranding(element) {
  const client = new CmaClient({ apiToken: null });
  const info = await client.publicInfo.find();
  
  if (!info.white_label) {
    console.log('Standard DatoCMS branding applies');
    return false;
  }
  
  // Apply white-label customizations
  const branding = {
    projectName: info.name,
    logo: info.logo_url,
    customTranslations: info.custom_i18n_messages_template_url,
    theme: info.theme
  };
  
  // Apply logo
  if (branding.logo && element.querySelector('.logo')) {
    element.querySelector('.logo').src = branding.logo;
  }
  
  // Apply theme colors
  const root = element.querySelector(':root') || element;
  Object.entries(branding.theme).forEach(([name, color]) => {
    const cssColor = `rgba(${color.red}, ${color.green}, ${color.blue}, ${color.alpha})`;
    root.style.setProperty(`--${name.replace('_', '-')}`, cssColor);
  });
  
  // Load custom translations if available
  if (branding.customTranslations) {
    const locale = navigator.language.split('-')[0];
    const translationUrl = branding.customTranslations.replace(':locale', locale);
    
    try {
      const response = await fetch(translationUrl);
      const translations = await response.json();
      console.log('Custom translations loaded:', translations);
    } catch (error) {
      console.error('Failed to load custom translations:', error);
    }
  }
  
  return true;
}
```

### 4. Theme Color Extraction

```javascript
// Extract and use DatoCMS theme colors in external apps
class DatoCMSTheme {
  constructor(publicInfo) {
    this.theme = publicInfo.theme;
  }
  
  toCSSVariables() {
    const vars = {};
    
    Object.entries(this.theme).forEach(([key, color]) => {
      const name = `--datocms-${key.replace(/_/g, '-')}`;
      vars[name] = this.colorToCSS(color);
      vars[`${name}-rgb`] = `${color.red}, ${color.green}, ${color.blue}`;
      vars[`${name}-hex`] = this.colorToHex(color);
    });
    
    return vars;
  }
  
  colorToCSS(color) {
    return `rgba(${color.red}, ${color.green}, ${color.blue}, ${color.alpha})`;
  }
  
  colorToHex(color) {
    const toHex = (n) => Math.round(n).toString(16).padStart(2, '0');
    const alphaHex = Math.round(color.alpha * 255).toString(16).padStart(2, '0');
    return `#${toHex(color.red)}${toHex(color.green)}${toHex(color.blue)}${alphaHex}`;
  }
  
  applyToDocument() {
    const cssVars = this.toCSSVariables();
    
    Object.entries(cssVars).forEach(([property, value]) => {
      document.documentElement.style.setProperty(property, value);
    });
  }
  
  generatePalette() {
    return {
      primary: this.colorToHex(this.theme.primary_color),
      light: this.colorToHex(this.theme.light_color),
      accent: this.colorToHex(this.theme.accent_color),
      dark: this.colorToHex(this.theme.dark_color),
      // Generate complementary colors
      primaryLight: this.lighten(this.theme.primary_color, 0.2),
      primaryDark: this.darken(this.theme.primary_color, 0.2),
      accentLight: this.lighten(this.theme.accent_color, 0.2),
      accentDark: this.darken(this.theme.accent_color, 0.2)
    };
  }
  
  lighten(color, amount) {
    return this.colorToHex({
      red: Math.min(255, color.red + (255 - color.red) * amount),
      green: Math.min(255, color.green + (255 - color.green) * amount),
      blue: Math.min(255, color.blue + (255 - color.blue) * amount),
      alpha: color.alpha
    });
  }
  
  darken(color, amount) {
    return this.colorToHex({
      red: Math.max(0, color.red * (1 - amount)),
      green: Math.max(0, color.green * (1 - amount)),
      blue: Math.max(0, color.blue * (1 - amount)),
      alpha: color.alpha
    });
  }
}

// Usage
const client = new CmaClient({ apiToken: null });
const info = await client.publicInfo.find();
const theme = new DatoCMSTheme(info);

// Apply theme to document
theme.applyToDocument();

// Get color palette
const palette = theme.generatePalette();
console.log('Theme palette:', palette);
```

### 5. Configuration Limits (Authenticated)

```javascript
// Access configuration limits with authentication
async function getProjectLimits(apiToken) {
  const client = new CmaClient({ apiToken });
  const info = await client.publicInfo.find();
  
  if (!info.extras) {
    throw new Error('Authentication required to access project limits');
  }
  
  return {
    maxBlockDepth: info.extras.blocks_depth,
    maxBlocksPerItem: info.extras.blocks_per_item,
    maxUploadSize: info.extras.maximum_single_upload_bytes,
    maxUploadSizeMB: Math.round(info.extras.maximum_single_upload_bytes / 1024 / 1024),
    // Access other limits from extras
    ...info.extras
  };
}

// Check if file can be uploaded
async function canUploadFile(client, fileSize) {
  const info = await client.publicInfo.find();
  
  if (!info.extras) {
    console.warn('Cannot verify upload size without authentication');
    return true; // Optimistic
  }
  
  return fileSize <= info.extras.maximum_single_upload_bytes;
}
```

### 6. Multi-Project Branding Manager

```javascript
// Manage branding across multiple DatoCMS projects
class MultiProjectBranding {
  constructor(projects) {
    this.projects = projects; // Array of project URLs
  }
  
  async getAllBranding() {
    const branding = await Promise.all(
      this.projects.map(async (projectUrl) => {
        try {
          const client = new CmaClient({ 
            apiToken: null,
            baseUrl: projectUrl 
          });
          
          const info = await client.publicInfo.find();
          
          return {
            url: projectUrl,
            name: info.name,
            logo: info.logo_url,
            theme: info.theme,
            whiteLabel: info.white_label,
            ssoEnabled: !!info.sso_saml_init_url
          };
        } catch (error) {
          return {
            url: projectUrl,
            error: error.message
          };
        }
      })
    );
    
    return branding;
  }
  
  async generateBrandingReport() {
    const branding = await this.getAllBranding();
    
    const report = {
      total: branding.length,
      successful: branding.filter(b => !b.error).length,
      whiteLabeled: branding.filter(b => b.whiteLabel).length,
      ssoEnabled: branding.filter(b => b.ssoEnabled).length,
      projects: branding
    };
    
    return report;
  }
}

// Usage
const manager = new MultiProjectBranding([
  'https://project1.admin.datocms.com',
  'https://project2.admin.datocms.com',
  'https://project3.admin.datocms.com'
]);

const report = await manager.generateBrandingReport();
console.log(`${report.whiteLabeled}/${report.total} projects are white-labeled`);
```

## Security Considerations

### Public Access
- No authentication required for basic info
- Only branding and configuration exposed
- No sensitive data or content accessible

### Authenticated Access
- API token unlocks `extras` field
- Contains technical limits and configuration
- Same security as other authenticated endpoints

### Best Practices
1. **Cache responses** to reduce API calls
2. **Handle missing data** gracefully (nulls)
3. **Validate SSO URLs** before redirecting
4. **Test theme colors** for accessibility

## Important Notes

1. **No Write Operations**: This is a read-only endpoint
2. **Site-Specific**: Uses project-specific API subdomain
3. **CORS Friendly**: Accessible from browser JavaScript
4. **Extras Field**: Only populated with valid authentication
5. **Theme Colors**: RGBA format with 0-255 for RGB, 0-1 for alpha

## Error Handling

```javascript
try {
  const info = await client.publicInfo.find();
} catch (error) {
  if (error.message.includes('INVALID_SITE')) {
    console.error('Invalid project URL');
  } else if (error.message.includes('NETWORK_ERROR')) {
    console.error('Network error - check connection');
  }
}
```

## Raw Response Access

```javascript
const response = await client.publicInfo.rawFind();
console.log(response.data); // Raw public info with JSON:API structure
```

## Related Resources

- [Sites](/01-cma-client/04-site-configuration/site.md) - Full site configuration (auth required)
- [White Label Settings](/01-cma-client/04-site-configuration/white-label-settings.md) - Customize branding
- [SSO Settings](/01-cma-client/05-access-control/sso-settings.md) - Configure single sign-on

---

# Public Info Methods

The Public Info resource provides access to basic site information without requiring authentication. This makes it ideal for client-side applications, public status pages, and other scenarios where API credentials cannot be safely stored.

## Overview

The Public Info endpoint is the only DatoCMS API endpoint that doesn't require authentication. It exposes a limited set of read-only information about a DatoCMS site that is considered safe for public consumption.

## Available Methods

### find()

Retrieves public information about a DatoCMS site without authentication.

```typescript
import { CmaClient } from '@datocms/cma-client';

// Initialize client WITHOUT API token - this is unique to public info
const client = new CmaClient({ 
  environment: 'main' // Optional: specify environment, defaults to primary
});

// Method signature
async find(): Promise<PublicInfo>
```

## Complete Examples

### Basic Usage - No Authentication Required

```typescript
import { CmaClient } from '@datocms/cma-client';

async function getPublicSiteInfo() {
  try {
    // Create client without API token
    const client = new CmaClient();
    
    // Fetch public info
    const publicInfo = await client.publicInfo.find();
    
    console.log('Site Name:', publicInfo.data.attributes.name);
    console.log('Site Locales:', publicInfo.data.attributes.locales);
    console.log('Primary Locale:', publicInfo.data.attributes.primaryLocale);
    
    return publicInfo;
  } catch (error) {
    console.error('Error fetching public info:', error);
    throw error;
  }
}
```

### Using with Specific Environment

```typescript
import { CmaClient } from '@datocms/cma-client';

async function getEnvironmentPublicInfo(environment: string) {
  try {
    // Specify environment when creating client
    const client = new CmaClient({ 
      environment: environment // e.g., 'main', 'staging', 'development'
    });
    
    const publicInfo = await client.publicInfo.find();
    
    console.log(`Public info for ${environment} environment:`, publicInfo);
    
    return publicInfo;
  } catch (error) {
    if (error.statusCode === 404) {
      console.error(`Environment '${environment}' not found or not public`);
    }
    throw error;
  }
}
```

### Client-Side React Example

```typescript
import React, { useEffect, useState } from 'react';
import { CmaClient } from '@datocms/cma-client-browser'; // Use browser client

interface SiteInfo {
  name: string;
  locales: string[];
  primaryLocale: string;
  timezone: string;
}

function PublicSiteStatus() {
  const [siteInfo, setSiteInfo] = useState<SiteInfo | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchSiteInfo() {
      try {
        // Safe to use in browser - no API key needed
        const client = new CmaClient();
        const response = await client.publicInfo.find();
        
        setSiteInfo({
          name: response.data.attributes.name,
          locales: response.data.attributes.locales,
          primaryLocale: response.data.attributes.primaryLocale,
          timezone: response.data.attributes.timezone
        });
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Failed to load site info');
      } finally {
        setLoading(false);
      }
    }

    fetchSiteInfo();
  }, []);

  if (loading) return <div>Loading site information...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!siteInfo) return null;

  return (
    <div>
      <h1>{siteInfo.name}</h1>
      <p>Available languages: {siteInfo.locales.join(', ')}</p>
      <p>Default language: {siteInfo.primaryLocale}</p>
      <p>Timezone: {siteInfo.timezone}</p>
    </div>
  );
}
```

## Response Structure

```typescript
interface PublicInfo {
  data: {
    type: 'public_info';
    id: string; // The site ID
    attributes: {
      name: string;              // Site name
      locales: string[];         // Array of locale codes (e.g., ['en', 'it', 'es'])
      primaryLocale: string;     // Primary locale code (e.g., 'en')
      timezone: string;          // Site timezone (e.g., 'Europe/Rome')
    };
  };
}

// Example response
{
  "data": {
    "type": "public_info",
    "id": "12345",
    "attributes": {
      "name": "My Company Website",
      "locales": ["en", "it", "es"],
      "primaryLocale": "en",
      "timezone": "Europe/Rome"
    }
  }
}
```

## What Information is Exposed

The public info endpoint exposes only the following limited information:

| Field | Description | Privacy Consideration |
|-------|-------------|----------------------|
| `name` | The public name of the site | Generally safe - usually company/project name |
| `locales` | Available content locales | Safe - indicates supported languages |
| `primaryLocale` | Default locale | Safe - indicates primary language |
| `timezone` | Site timezone setting | Safe - used for scheduling/display |

### Information NOT Exposed

- API tokens or credentials
- User information
- Content model structure
- Actual content records
- Plugin configurations
- Webhook URLs
- Build triggers
- Any write capabilities

## Use Cases

### 1. Public Status Pages

```typescript
import { CmaClient } from '@datocms/cma-client';

async function createStatusWidget() {
  const client = new CmaClient();
  
  try {
    const info = await client.publicInfo.find();
    
    return {
      status: 'operational',
      siteName: info.data.attributes.name,
      supportedLanguages: info.data.attributes.locales.length,
      lastChecked: new Date().toISOString()
    };
  } catch (error) {
    return {
      status: 'error',
      message: 'Unable to reach CMS',
      lastChecked: new Date().toISOString()
    };
  }
}
```

### 2. Language Selector Initialization

```typescript
import { CmaClient } from '@datocms/cma-client-browser';

async function initializeLanguageSelector() {
  const client = new CmaClient();
  
  try {
    const info = await client.publicInfo.find();
    const { locales, primaryLocale } = info.data.attributes;
    
    // Build language selector
    const selector = document.getElementById('language-selector');
    
    locales.forEach(locale => {
      const option = document.createElement('option');
      option.value = locale;
      option.textContent = locale.toUpperCase();
      option.selected = locale === primaryLocale;
      selector?.appendChild(option);
    });
    
    return locales;
  } catch (error) {
    console.error('Failed to load available languages');
    return ['en']; // Fallback
  }
}
```

### 3. Client-Side Timezone Display

```typescript
import { CmaClient } from '@datocms/cma-client-browser';

async function getFormattedTimeInSiteTimezone() {
  const client = new CmaClient();
  
  try {
    const info = await client.publicInfo.find();
    const siteTimezone = info.data.attributes.timezone;
    
    // Format current time in site's timezone
    const formatter = new Intl.DateTimeFormat('en-US', {
      timeZone: siteTimezone,
      dateStyle: 'full',
      timeStyle: 'long'
    });
    
    return {
      timezone: siteTimezone,
      currentTime: formatter.format(new Date())
    };
  } catch (error) {
    // Fallback to user's timezone
    return {
      timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
      currentTime: new Date().toLocaleString()
    };
  }
}
```

## Caching Strategies

### Browser Caching

```typescript
import { CmaClient } from '@datocms/cma-client-browser';

class PublicInfoCache {
  private cache: Map<string, { data: any; timestamp: number }> = new Map();
  private readonly TTL = 5 * 60 * 1000; // 5 minutes

  async getPublicInfo(environment?: string): Promise<any> {
    const cacheKey = environment || 'default';
    const cached = this.cache.get(cacheKey);
    
    // Return cached data if still valid
    if (cached && Date.now() - cached.timestamp < this.TTL) {
      return cached.data;
    }
    
    // Fetch fresh data
    const client = new CmaClient({ environment });
    const data = await client.publicInfo.find();
    
    // Cache the result
    this.cache.set(cacheKey, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }
  
  clearCache(): void {
    this.cache.clear();
  }
}

// Usage
const publicInfoCache = new PublicInfoCache();
const info = await publicInfoCache.getPublicInfo();
```

### Service Worker Caching

```typescript
// In service worker
self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);
  
  // Cache public info requests
  if (url.pathname.includes('/public-info')) {
    event.respondWith(
      caches.open('datocms-public-v1').then(cache => {
        return cache.match(event.request).then(response => {
          // Return cached response if less than 10 minutes old
          if (response) {
            const age = Date.now() - new Date(response.headers.get('date')).getTime();
            if (age < 10 * 60 * 1000) {
              return response;
            }
          }
          
          // Fetch fresh and cache
          return fetch(event.request).then(networkResponse => {
            cache.put(event.request, networkResponse.clone());
            return networkResponse;
          });
        });
      })
    );
  }
});
```

## Error Handling

### Comprehensive Error Handling

```typescript
import { CmaClient } from '@datocms/cma-client';

async function safeGetPublicInfo(options?: { environment?: string }) {
  const client = new CmaClient(options);
  
  try {
    const info = await client.publicInfo.find();
    return { success: true, data: info };
  } catch (error: any) {
    // Network errors
    if (error.code === 'ENOTFOUND' || error.code === 'ECONNREFUSED') {
      return {
        success: false,
        error: 'Network error - Unable to reach DatoCMS',
        code: 'NETWORK_ERROR'
      };
    }
    
    // Environment not found
    if (error.statusCode === 404) {
      return {
        success: false,
        error: 'Environment not found or not accessible',
        code: 'ENVIRONMENT_NOT_FOUND'
      };
    }
    
    // Rate limiting
    if (error.statusCode === 429) {
      const retryAfter = error.headers?.['retry-after'];
      return {
        success: false,
        error: 'Rate limit exceeded',
        code: 'RATE_LIMITED',
        retryAfter: retryAfter ? parseInt(retryAfter) : 60
      };
    }
    
    // Generic error
    return {
      success: false,
      error: error.message || 'Unknown error occurred',
      code: 'UNKNOWN_ERROR'
    };
  }
}
```

### Retry Logic with Exponential Backoff

```typescript
import { CmaClient } from '@datocms/cma-client';

async function getPublicInfoWithRetry(
  maxRetries: number = 3,
  initialDelay: number = 1000
): Promise<any> {
  const client = new CmaClient();
  
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await client.publicInfo.find();
    } catch (error: any) {
      // Don't retry on 404s
      if (error.statusCode === 404) {
        throw error;
      }
      
      // Last attempt, throw the error
      if (attempt === maxRetries - 1) {
        throw error;
      }
      
      // Calculate delay with exponential backoff
      const delay = initialDelay * Math.pow(2, attempt);
      console.log(`Retry attempt ${attempt + 1} after ${delay}ms`);
      
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

## Privacy and Security Considerations

### What's Safe

- ✅ Using public info in client-side applications
- ✅ Caching public info in browsers
- ✅ Displaying site name and languages publicly
- ✅ Using timezone for display formatting
- ✅ Building language selectors

### What to Avoid

- ❌ Don't assume site ID is secret (it's in the response)
- ❌ Don't use public info to infer private information
- ❌ Don't expose internal environment names unnecessarily
- ❌ Don't rely on public info for security decisions

### Security Best Practices

```typescript
// Good: Use public info for display only
async function displaySiteInfo() {
  const client = new CmaClient();
  const info = await client.publicInfo.find();
  
  // Safe to display
  document.getElementById('site-name').textContent = info.data.attributes.name;
  document.getElementById('languages').textContent = 
    `Available in ${info.data.attributes.locales.length} languages`;
}

// Bad: Don't use for access control
async function checkAccess() {
  const client = new CmaClient();
  const info = await client.publicInfo.find();
  
  // DON'T DO THIS - public info is not for authentication
  if (info.data.id === 'expected-site-id') {
    grantAccess(); // WRONG!
  }
}
```

## Best Practices

### 1. Always Handle Errors Gracefully

```typescript
async function initializeApp() {
  try {
    const client = new CmaClient();
    const info = await client.publicInfo.find();
    
    // Initialize with actual data
    initializeWithSiteInfo(info);
  } catch (error) {
    // Fallback to defaults
    initializeWithDefaults({
      name: 'Site',
      locales: ['en'],
      primaryLocale: 'en',
      timezone: 'UTC'
    });
  }
}
```

### 2. Cache Appropriately

```typescript
// Good: Cache with reasonable TTL
const CACHE_KEY = 'datocms_public_info';
const CACHE_TTL = 5 * 60 * 1000; // 5 minutes

async function getCachedPublicInfo() {
  // Check localStorage
  const cached = localStorage.getItem(CACHE_KEY);
  if (cached) {
    const { data, timestamp } = JSON.parse(cached);
    if (Date.now() - timestamp < CACHE_TTL) {
      return data;
    }
  }
  
  // Fetch fresh
  const client = new CmaClient();
  const fresh = await client.publicInfo.find();
  
  // Cache it
  localStorage.setItem(CACHE_KEY, JSON.stringify({
    data: fresh,
    timestamp: Date.now()
  }));
  
  return fresh;
}
```

### 3. Use for Progressive Enhancement

```typescript
// Enhance UI based on available locales
async function enhanceLanguageUI() {
  const fallbackLocales = ['en']; // Default
  
  try {
    const client = new CmaClient();
    const info = await client.publicInfo.find();
    const { locales, primaryLocale } = info.data.attributes;
    
    // Enhance UI with actual locales
    if (locales.length > 1) {
      showLanguageSelector(locales, primaryLocale);
    }
    
    // Set document language
    document.documentElement.lang = primaryLocale;
    
  } catch (error) {
    // Still functional with defaults
    console.warn('Using fallback locales');
    document.documentElement.lang = fallbackLocales[0];
  }
}
```

## Related Resources

- [Site Methods](./site-methods.md) - Full site configuration (requires authentication)
- [Environment Methods](./environment-methods.md) - Environment management
- [Authentication Guide](../../00-quick-start/authentication.md) - When you need authenticated access
- [CMA Client Setup](../../00-quick-start/README.md) - General client configuration

## Common Patterns

### Multi-Environment Public Info

```typescript
import { CmaClient } from '@datocms/cma-client';

class MultiEnvironmentPublicInfo {
  async getAllEnvironmentsInfo(environments: string[]) {
    const results = await Promise.allSettled(
      environments.map(async (env) => {
        const client = new CmaClient({ environment: env });
        try {
          const info = await client.publicInfo.find();
          return { environment: env, info };
        } catch (error) {
          return { environment: env, error };
        }
      })
    );
    
    return results.map((result, index) => {
      if (result.status === 'fulfilled') {
        return result.value;
      } else {
        return { 
          environment: environments[index], 
          error: result.reason 
        };
      }
    });
  }
}

// Usage
const multiEnv = new MultiEnvironmentPublicInfo();
const allInfo = await multiEnv.getAllEnvironmentsInfo(['main', 'staging', 'development']);
```

### Health Check Endpoint

```typescript
import { CmaClient } from '@datocms/cma-client';
import express from 'express';

const app = express();

app.get('/health/datocms', async (req, res) => {
  const client = new CmaClient();
  
  try {
    const start = Date.now();
    const info = await client.publicInfo.find();
    const responseTime = Date.now() - start;
    
    res.json({
      status: 'healthy',
      service: 'datocms',
      siteName: info.data.attributes.name,
      responseTime: `${responseTime}ms`,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      service: 'datocms',
      error: error.message,
      timestamp: new Date().toISOString()
    });
  }
});
```

---

# Webhook

Webhooks allow you to receive HTTP notifications when specific events occur in your DatoCMS project. You can use them to trigger builds, sync data, send notifications, or integrate with external services.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a webhook for content updates
const webhook = await client.webhooks.create({
  name: 'Content Update Notification',
  url: 'https://api.example.com/webhooks/datocms',
  events: [
    { entity_type: 'item', event_types: ['create', 'update', 'publish'] }
  ]
});
```

## API Reference

### Create Webhook

Create a new webhook endpoint.

```typescript
const webhook = await client.webhooks.create({
  name: 'Deploy Trigger',
  url: 'https://api.netlify.com/build_hooks/YOUR_HOOK_ID',
  enabled: true,
  events: [
    { 
      entity_type: 'item',
      event_types: ['publish', 'unpublish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['blog_post', 'landing_page']
        }
      ]
    }
  ],
  headers: {
    'X-Custom-Header': 'custom-value',
    'Authorization': 'Bearer YOUR_TOKEN'
  },
  http_basic_user: 'username',
  http_basic_password: 'password',
  payload_api_version: '3',
  nested_items_in_payload: true,
  auto_retry: true
});
```

**Parameters:**
- `name` (string, required): Webhook name for identification
- `url` (string, required): HTTP endpoint to call
- `enabled` (boolean, optional): Whether webhook is active (default: true)
- `events` (array, required): Events that trigger the webhook
- `headers` (object, optional): Custom HTTP headers
- `http_basic_user` (string, optional): Basic auth username
- `http_basic_password` (string, optional): Basic auth password
- `custom_payload` (string, optional): Custom payload template
- `payload_api_version` (string, optional): API version for payload format
- `nested_items_in_payload` (boolean, optional): Include full nested items
- `auto_retry` (boolean, optional): Retry failed calls automatically

**Returns:** The created webhook object

### Update Webhook

Modify an existing webhook configuration.

```typescript
const updated = await client.webhooks.update('webhook-id', {
  name: 'Updated Webhook',
  enabled: false, // Temporarily disable
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'delete', 'publish', 'unpublish']
    },
    {
      entity_type: 'upload',
      event_types: ['create', 'update', 'delete']
    }
  ]
});
```

**Parameters:**
- `webhookId` (string, required): The webhook ID
- Update parameters (same as create, all optional)

**Returns:** The updated webhook object

### List Webhooks

Get all webhooks in the project.

```typescript
const webhooks = await client.webhooks.list();

// Find enabled webhooks
const activeWebhooks = webhooks.filter(w => w.enabled);

// Find webhooks for specific events
const publishWebhooks = webhooks.filter(w => 
  w.events.some(e => 
    e.entity_type === 'item' && e.event_types.includes('publish')
  )
);
```

**Returns:** Array of webhook objects

### Find Webhook

Retrieve a specific webhook by ID.

```typescript
const webhook = await client.webhooks.find('webhook-id');
```

**Parameters:**
- `webhookId` (string, required): The webhook ID

**Returns:** The webhook object

### Delete Webhook

Remove a webhook.

```typescript
await client.webhooks.destroy('webhook-id');
```

**Parameters:**
- `webhookId` (string, required): The webhook ID

## Event Types

### Item Events

Monitor content changes:

```typescript
{
  entity_type: 'item',
  event_types: [
    'create',     // New item created
    'update',     // Item modified
    'delete',     // Item deleted
    'publish',    // Item published
    'unpublish'   // Item unpublished
  ],
  filters: [
    {
      entity_type: 'item_type',
      entity_ids: ['article', 'page'] // Only these models
    }
  ]
}
```

### Upload Events

Track media changes:

```typescript
{
  entity_type: 'upload',
  event_types: [
    'create',  // New upload
    'update',  // Upload metadata changed
    'delete'   // Upload deleted
  ]
}
```

### Model Events

Monitor schema changes:

```typescript
{
  entity_type: 'item_type',
  event_types: [
    'create',  // New model created
    'update',  // Model updated
    'delete'   // Model deleted
  ]
}
```

### Other Entity Types

```typescript
// Field changes
{ entity_type: 'field', event_types: ['create', 'update', 'delete'] }

// User management
{ entity_type: 'user', event_types: ['create', 'update', 'delete'] }

// Role changes
{ entity_type: 'role', event_types: ['create', 'update', 'delete'] }

// Environment changes
{ entity_type: 'environment', event_types: ['create', 'fork', 'promote', 'delete'] }
```

## Webhook Payloads

### Standard Payload

The default webhook payload structure:

```typescript
{
  entity_type: 'item',
  event_type: 'publish',
  entity: {
    id: '12345',
    type: 'item',
    attributes: {
      title: 'Article Title',
      content: 'Article content...'
    },
    relationships: {
      item_type: {
        data: { type: 'item_type', id: 'article' }
      }
    }
  },
  related_entities: [
    // Nested items if nested_items_in_payload is true
  ]
}
```

### Custom Payload

Use Liquid templating for custom payloads:

```typescript
const webhook = await client.webhooks.create({
  name: 'Slack Notification',
  url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL',
  custom_payload: `{
    "text": "Content updated!",
    "blocks": [
      {
        "type": "section",
        "text": {
          "type": "mrkdwn",
          "text": "*{{ entity.attributes.title }}* was {{ event_type }}d"
        }
      },
      {
        "type": "context",
        "elements": [
          {
            "type": "mrkdwn",
            "text": "Model: {{ entity.relationships.item_type.data.id }} | Environment: {{ environment }}"
          }
        ]
      }
    ]
  }`,
  headers: {
    'Content-Type': 'application/json'
  },
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});
```

### Available Variables in Custom Payloads

- `{{ entity }}` - The affected entity
- `{{ event_type }}` - The event that occurred
- `{{ entity_type }}` - Type of entity
- `{{ environment }}` - Environment ID
- `{{ related_entities }}` - Array of related entities

## Advanced Usage

### Filtering by Environment

Webhooks can be environment-specific:

```typescript
// Create webhook only for production
const productionWebhook = await client.webhooks.create({
  name: 'Production Deploy',
  url: 'https://api.vercel.com/v1/deployments',
  events: [
    {
      entity_type: 'item',
      event_types: ['publish'],
      environments: ['main'] // Only trigger in main environment
    }
  ]
});
```

### Conditional Webhooks

Filter events based on specific criteria:

```typescript
const webhook = await client.webhooks.create({
  name: 'Homepage Update Alert',
  url: 'https://api.example.com/notify',
  events: [
    {
      entity_type: 'item',
      event_types: ['update', 'publish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['homepage'] // Only homepage model
        },
        {
          entity_type: 'item',
          entity_ids: ['homepage-singleton-id'] // Specific item
        }
      ]
    }
  ]
});
```

### Build Triggers

Common pattern for triggering builds:

```typescript
// Netlify
const netlifyWebhook = await client.webhooks.create({
  name: 'Netlify Build',
  url: 'https://api.netlify.com/build_hooks/YOUR_BUILD_HOOK_ID',
  events: [
    { entity_type: 'item', event_types: ['publish', 'unpublish'] }
  ]
});

// Vercel
const vercelWebhook = await client.webhooks.create({
  name: 'Vercel Deploy',
  url: 'https://api.vercel.com/v1/deployments',
  headers: {
    'Authorization': 'Bearer YOUR_VERCEL_TOKEN'
  },
  custom_payload: '{"gitSource": {"ref": "main", "repoId": "YOUR_REPO_ID"}}',
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});

// GitHub Actions
const githubWebhook = await client.webhooks.create({
  name: 'GitHub Action Trigger',
  url: 'https://api.github.com/repos/OWNER/REPO/dispatches',
  headers: {
    'Authorization': 'token YOUR_GITHUB_TOKEN',
    'Accept': 'application/vnd.github.v3+json'
  },
  custom_payload: '{"event_type": "datocms-update"}',
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});
```

### Webhook Security

Implement webhook verification:

```typescript
// DatoCMS signs webhook payloads
const webhook = await client.webhooks.create({
  name: 'Secure Webhook',
  url: 'https://api.example.com/webhook',
  events: [{ entity_type: 'item', event_types: ['create'] }]
});

// In your webhook endpoint (Express example):
import crypto from 'crypto';

app.post('/webhook', (req, res) => {
  const signature = req.headers['x-datocms-signature'];
  const body = JSON.stringify(req.body);
  
  // Verify signature
  const expectedSignature = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(body)
    .digest('hex');
    
  if (signature !== expectedSignature) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process webhook
  console.log('Verified webhook:', req.body);
  res.status(200).send('OK');
});
```

### Retry Configuration

Handle failures gracefully:

```typescript
const reliableWebhook = await client.webhooks.create({
  name: 'Reliable Webhook',
  url: 'https://api.example.com/webhook',
  auto_retry: true, // Enable automatic retries
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});

// DatoCMS will retry failed webhooks:
// - After 10 seconds
// - After 1 minute
// - After 10 minutes
// - After 1 hour
// - After 3 hours
```

### Monitoring Webhook Calls

Check webhook execution history:

```typescript
// Get webhook calls for a specific webhook
const calls = await client.webhookCalls.list('webhook-id');

// Check failed calls
const failedCalls = calls.filter(call => 
  call.response_status >= 400 || call.response_status === null
);

// Resend a failed webhook
await client.webhookCalls.resend('webhook-call-id');
```

**Related:** See [Webhook Calls](../06-monitoring/webhook-call.md) for detailed call history

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.webhooks.create({
    name: 'Test Webhook',
    url: 'invalid-url', // Invalid URL
    events: []
  });
} catch (error) {
  if (error instanceof ApiError) {
    const urlError = error.findError('url');
    if (urlError?.code === 'VALIDATION_FORMAT') {
      console.log('Invalid webhook URL format');
    }
  }
}
```

## Webhook Object Structure

```typescript
interface Webhook {
  id: string;
  name: string;
  url: string;
  enabled: boolean;
  headers: Record<string, string>;
  events: Array<{
    entity_type: string;
    event_types: string[];
    filters?: Array<{
      entity_type: string;
      entity_ids: string[];
    }>;
    environments?: string[];
  }>;
  http_basic_user?: string;
  http_basic_password?: string;
  custom_payload?: string;
  payload_api_version: string;
  nested_items_in_payload: boolean;
  auto_retry: boolean;
  created_at: string;
  updated_at: string;
}
```

## Related Resources

- [Webhook Calls](../06-monitoring/webhook-call.md) - Monitor webhook executions
- [Build Triggers](./build-triggers.md) - Alternative to webhooks for builds
- Real-time Events - Task-oriented guide
- [Job Results](./job-results.md) - Track async operations

---

# Webhook Methods - Complete Reference

This document contains all methods available for the Webhook resource in the DatoCMS CMA Client.

## Available Methods for client.webhooks

### list() - Retrieve All Webhooks

Retrieves all webhooks configured for the project.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get all webhooks
const webhooks = await client.webhooks.list();

// Filter active webhooks
const activeWebhooks = webhooks.filter(webhook => webhook.enabled);

// Group by event type
const webhooksByEvent = webhooks.reduce((acc, webhook) => {
  webhook.events.forEach(event => {
    const key = `${event.entity_type}:${event.event_types.join(',')}`;
    if (!acc[key]) acc[key] = [];
    acc[key].push(webhook);
  });
  return acc;
}, {});

// Get raw response
const rawResponse = await client.webhooks.rawList();
```

### find() - Get Single Webhook

Retrieves a specific webhook by ID.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get webhook by ID
const webhook = await client.webhooks.find('webhook_id');

// Webhook structure includes:
// {
//   id: 'webhook_id',
//   type: 'webhook',
//   attributes: {
//     name: 'Content Published Notification',
//     url: 'https://api.example.com/webhooks/datocms',
//     custom_payload: '{"event": "{{event_type}}", "entity": "{{entity.id}}"}',
//     headers: {
//       'X-Api-Key': 'secret-key',
//       'Content-Type': 'application/json'
//     },
//     http_basic_user: null,
//     http_basic_password: null,
//     enabled: true,
//     payload_api_version: '3',
//     nested_items_in_payload: true,
//     events: [
//       {
//         entity_type: 'item',
//         event_types: ['publish', 'unpublish'],
//         filters: [
//           {
//             entity_type: 'item_type',
//             entity_ids: ['blog_post', 'article']
//           }
//         ]
//       }
//     ]
//   }
// }

// Error handling
try {
  const webhook = await client.webhooks.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Webhook not found');
  }
}
```

### create() - Create New Webhook

Creates a new webhook with specified configuration.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create simple webhook
const simpleWebhook = await client.webhooks.create({
  name: 'New Content Alert',
  url: 'https://api.example.com/webhooks/new-content',
  events: [
    {
      entity_type: 'item',
      event_types: ['create']
    }
  ]
});

// Create webhook with filters
const filteredWebhook = await client.webhooks.create({
  name: 'Blog Post Published',
  url: 'https://api.example.com/webhooks/blog-published',
  events: [
    {
      entity_type: 'item',
      event_types: ['publish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['blog_post'] // Only blog posts
        }
      ]
    }
  ],
  enabled: true
});

// Create webhook with custom payload
const customPayloadWebhook = await client.webhooks.create({
  name: 'Custom Notification',
  url: 'https://api.example.com/webhooks/custom',
  custom_payload: JSON.stringify({
    event: '{{event_type}}',
    entity_id: '{{entity.id}}',
    entity_type: '{{entity_type}}',
    timestamp: '{{timestamp}}',
    item_data: {
      title: '{{entity.title}}',
      slug: '{{entity.slug}}',
      published_at: '{{entity.meta.published_at}}'
    }
  }),
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'publish', 'unpublish']
    }
  ]
});

// Create webhook with authentication
const authenticatedWebhook = await client.webhooks.create({
  name: 'Secure Webhook',
  url: 'https://api.example.com/secure-webhook',
  headers: {
    'Authorization': 'Bearer YOUR_SECRET_TOKEN',
    'X-Custom-Header': 'custom-value'
  },
  events: [
    {
      entity_type: 'item',
      event_types: ['publish']
    }
  ]
});

// Create webhook with HTTP Basic Auth
const basicAuthWebhook = await client.webhooks.create({
  name: 'Basic Auth Webhook',
  url: 'https://api.example.com/webhook',
  http_basic_user: 'username',
  http_basic_password: 'password',
  events: [
    {
      entity_type: 'item',
      event_types: ['create']
    }
  ]
});

// Create comprehensive webhook for multiple events
const comprehensiveWebhook = await client.webhooks.create({
  name: 'All Content Changes',
  url: 'https://api.example.com/webhooks/all-changes',
  enabled: true,
  nested_items_in_payload: true,
  payload_api_version: '3',
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'delete', 'publish', 'unpublish'],
      filters: []
    },
    {
      entity_type: 'item_type',
      event_types: ['create', 'update', 'delete'],
      filters: []
    },
    {
      entity_type: 'upload',
      event_types: ['create', 'update', 'delete'],
      filters: []
    }
  ]
});

// Error handling
try {
  const webhook = await client.webhooks.create({
    name: 'Invalid Webhook',
    url: 'not-a-valid-url',
    events: []
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation errors:', error.response.data.errors);
  }
}
```

### update() - Update Webhook

Updates an existing webhook configuration.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Update webhook URL
const updated = await client.webhooks.update('webhook_id', {
  url: 'https://new-api.example.com/webhooks'
});

// Enable/disable webhook
const disabled = await client.webhooks.update('webhook_id', {
  enabled: false
});

// Update events
const updatedEvents = await client.webhooks.update('webhook_id', {
  events: [
    {
      entity_type: 'item',
      event_types: ['publish', 'unpublish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['blog_post', 'news_article']
        }
      ]
    }
  ]
});

// Update custom payload
const updatedPayload = await client.webhooks.update('webhook_id', {
  custom_payload: JSON.stringify({
    notification: {
      event: '{{event_type}}',
      item: {
        id: '{{entity.id}}',
        type: '{{entity.item_type.api_key}}',
        title: '{{entity.title}}'
      },
      user: '{{performer.email}}',
      timestamp: '{{timestamp}}'
    }
  })
});

// Update headers
const updatedHeaders = await client.webhooks.update('webhook_id', {
  headers: {
    'Authorization': 'Bearer NEW_TOKEN',
    'X-Webhook-Version': '2.0'
  }
});

// Complex update
const complexUpdate = await client.webhooks.update('webhook_id', {
  name: 'Updated Webhook Name',
  url: 'https://api.example.com/v2/webhooks',
  enabled: true,
  headers: {
    'X-API-Version': '2'
  },
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'publish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['product', 'category']
        }
      ]
    }
  ]
});

// Error handling
try {
  const updated = await client.webhooks.update('webhook_id', {
    events: [] // Cannot have empty events
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('At least one event is required');
  }
}
```

### destroy() - Delete Webhook

Deletes a webhook permanently.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Delete webhook
await client.webhooks.destroy('webhook_id');

// Delete with error handling
try {
  await client.webhooks.destroy('webhook_id');
  console.log('Webhook deleted successfully');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Webhook not found');
  }
}

// Bulk delete pattern
async function deleteWebhooksByName(pattern) {
  const webhooks = await client.webhooks.list();
  const toDelete = webhooks.filter(w => w.name.includes(pattern));
  
  const results = await Promise.allSettled(
    toDelete.map(webhook => client.webhooks.destroy(webhook.id))
  );
  
  return {
    deleted: results.filter(r => r.status === 'fulfilled').length,
    failed: results.filter(r => r.status === 'rejected').length
  };
}
```

## Webhook Event Types

### Available Entity Types and Events

```typescript
// Item events
const itemWebhook = {
  entity_type: 'item',
  event_types: [
    'create',     // Item created
    'update',     // Item updated
    'delete',     // Item deleted
    'publish',    // Item published
    'unpublish'   // Item unpublished
  ]
};

// Item Type (Model) events
const itemTypeWebhook = {
  entity_type: 'item_type',
  event_types: [
    'create',     // Model created
    'update',     // Model updated
    'delete'      // Model deleted
  ]
};

// Upload events
const uploadWebhook = {
  entity_type: 'upload',
  event_types: [
    'create',     // File uploaded
    'update',     // File metadata updated
    'delete'      // File deleted
  ]
};

// Environment events
const environmentWebhook = {
  entity_type: 'environment',
  event_types: [
    'create',     // Environment forked
    'promote',    // Environment promoted to primary
    'delete'      // Environment deleted
  ]
};

// Build trigger events
const buildTriggerWebhook = {
  entity_type: 'build_trigger',
  event_types: [
    'trigger'     // Build triggered
  ]
};
```

## Webhook Patterns

### Content Publishing Notifications

```typescript
async function setupPublishingNotifications(notificationUrl) {
  // Notify when any content is published
  const publishWebhook = await client.webhooks.create({
    name: 'Content Published',
    url: notificationUrl,
    custom_payload: JSON.stringify({
      text: 'New content published!',
      content: {
        id: '{{entity.id}}',
        title: '{{entity.title}}',
        url: 'https://example.com/{{entity.slug}}'
      }
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['publish']
      }
    ]
  });
  
  // Notify when content is scheduled
  const scheduleWebhook = await client.webhooks.create({
    name: 'Content Scheduled',
    url: notificationUrl,
    custom_payload: JSON.stringify({
      text: 'Content scheduled for publication',
      content: {
        id: '{{entity.id}}',
        title: '{{entity.title}}',
        scheduled_at: '{{entity.meta.publication_scheduled_at}}'
      }
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['update'],
        filters: []
      }
    ]
  });
  
  return { publishWebhook, scheduleWebhook };
}
```

### Slack Integration

```typescript
async function createSlackWebhook(slackWebhookUrl, channel) {
  return client.webhooks.create({
    name: 'Slack Notifications',
    url: slackWebhookUrl,
    custom_payload: JSON.stringify({
      channel: channel,
      username: 'DatoCMS',
      icon_emoji: ':pencil:',
      text: 'Content update in DatoCMS',
      attachments: [{
        color: 'good',
        title: '{{entity.title}}',
        fields: [
          {
            title: 'Event',
            value: '{{event_type}}',
            short: true
          },
          {
            title: 'Type',
            value: '{{entity.item_type.name}}',
            short: true
          },
          {
            title: 'Updated by',
            value: '{{performer.email}}',
            short: true
          },
          {
            title: 'Time',
            value: '{{timestamp}}',
            short: true
          }
        ]
      }]
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['create', 'update', 'publish']
      }
    ]
  });
}
```

### Build Trigger Integration

```typescript
async function setupAutoBuild(buildUrl, buildToken) {
  // Trigger build on content changes
  const buildWebhook = await client.webhooks.create({
    name: 'Auto Build Trigger',
    url: buildUrl,
    headers: {
      'Authorization': `Bearer ${buildToken}`,
      'X-Trigger-Type': 'content-change'
    },
    custom_payload: JSON.stringify({
      trigger: 'content_update',
      event: '{{event_type}}',
      entity: '{{entity_type}}',
      id: '{{entity.id}}'
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['publish', 'unpublish']
      }
    ]
  });
  
  // Trigger build on model changes
  const schemaWebhook = await client.webhooks.create({
    name: 'Schema Change Build',
    url: buildUrl,
    headers: {
      'Authorization': `Bearer ${buildToken}`,
      'X-Trigger-Type': 'schema-change'
    },
    events: [
      {
        entity_type: 'item_type',
        event_types: ['create', 'update', 'delete']
      }
    ]
  });
  
  return { buildWebhook, schemaWebhook };
}
```

### Audit Log Webhook

```typescript
async function createAuditWebhook(auditEndpoint, apiKey) {
  return client.webhooks.create({
    name: 'Audit Log',
    url: auditEndpoint,
    headers: {
      'X-API-Key': apiKey,
      'Content-Type': 'application/json'
    },
    custom_payload: JSON.stringify({
      timestamp: '{{timestamp}}',
      event_type: '{{event_type}}',
      entity_type: '{{entity_type}}',
      entity_id: '{{entity.id}}',
      performer: {
        id: '{{performer.id}}',
        email: '{{performer.email}}',
        type: '{{performer.type}}'
      },
      entity_data: '{{entity}}',
      environment: '{{environment.id}}'
    }),
    nested_items_in_payload: true,
    events: [
      {
        entity_type: 'item',
        event_types: ['create', 'update', 'delete', 'publish', 'unpublish']
      },
      {
        entity_type: 'item_type',
        event_types: ['create', 'update', 'delete']
      },
      {
        entity_type: 'upload',
        event_types: ['create', 'update', 'delete']
      }
    ]
  });
}
```

### Webhook Testing

```typescript
async function testWebhook(webhookId) {
  const webhook = await client.webhooks.find(webhookId);
  
  // Create a test item to trigger the webhook
  const testItem = await client.items.create({
    item_type: { type: 'item_type', id: 'test_model' },
    title: 'Webhook Test Item',
    test_field: 'This should trigger the webhook'
  });
  
  // Perform action based on webhook events
  for (const event of webhook.events) {
    if (event.event_types.includes('publish')) {
      await client.items.publish(testItem.id);
    }
    if (event.event_types.includes('unpublish')) {
      await client.items.unpublish(testItem.id);
    }
  }
  
  // Clean up
  setTimeout(async () => {
    await client.items.destroy(testItem.id);
  }, 5000);
  
  return {
    webhook: webhook.name,
    testItem: testItem.id,
    triggeredEvents: webhook.events
  };
}
```

### Webhook Monitoring

```typescript
async function getWebhookHealth() {
  const webhooks = await client.webhooks.list();
  
  const health = webhooks.map(webhook => ({
    id: webhook.id,
    name: webhook.name,
    url: webhook.url,
    enabled: webhook.enabled,
    eventCount: webhook.events.reduce(
      (sum, event) => sum + event.event_types.length, 
      0
    ),
    hasFilters: webhook.events.some(e => e.filters && e.filters.length > 0),
    hasCustomPayload: !!webhook.custom_payload,
    hasAuth: !!(webhook.headers?.Authorization || webhook.http_basic_user)
  }));
  
  return {
    total: webhooks.length,
    enabled: health.filter(w => w.enabled).length,
    disabled: health.filter(w => !w.enabled).length,
    webhooks: health
  };
}

// Find misconfigured webhooks
async function findMisconfiguredWebhooks() {
  const webhooks = await client.webhooks.list();
  const issues = [];
  
  for (const webhook of webhooks) {
    const webhookIssues = [];
    
    // Check for localhost URLs in production
    if (webhook.url.includes('localhost') || webhook.url.includes('127.0.0.1')) {
      webhookIssues.push('URL points to localhost');
    }
    
    // Check for HTTP in production
    if (webhook.url.startsWith('http://') && !webhook.url.includes('localhost')) {
      webhookIssues.push('Using insecure HTTP');
    }
    
    // Check for missing authentication
    if (!webhook.headers?.Authorization && !webhook.http_basic_user) {
      webhookIssues.push('No authentication configured');
    }
    
    // Check for overly broad events
    const hasUnfilteredEvents = webhook.events.some(
      event => (!event.filters || event.filters.length === 0) && 
               event.event_types.length > 3
    );
    if (hasUnfilteredEvents) {
      webhookIssues.push('Very broad event configuration without filters');
    }
    
    if (webhookIssues.length > 0) {
      issues.push({
        webhook: webhook.name,
        id: webhook.id,
        issues: webhookIssues
      });
    }
  }
  
  return issues;
}
```

## Error Handling

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Comprehensive error handling
async function safeWebhookOperation() {
  try {
    const webhook = await client.webhooks.create({
      name: 'Test Webhook',
      url: 'https://api.example.com/webhook',
      events: [
        {
          entity_type: 'item',
          event_types: ['publish']
        }
      ]
    });
    return webhook;
  } catch (error) {
    if (error.response) {
      switch (error.response.status) {
        case 422:
          const validationErrors = error.response.data.errors;
          validationErrors.forEach(err => {
            if (err.source?.pointer?.includes('url')) {
              console.error('URL validation failed:', err.detail);
            } else if (err.source?.pointer?.includes('events')) {
              console.error('Events configuration error:', err.detail);
            } else if (err.source?.pointer?.includes('custom_payload')) {
              console.error('Invalid custom payload:', err.detail);
            }
          });
          break;
        case 403:
          console.error('Insufficient permissions to manage webhooks');
          break;
        case 429:
          console.error('Rate limit exceeded');
          break;
        default:
          console.error('Unexpected error:', error.response.status);
      }
    }
    throw error;
  }
}

// Validate webhook configuration
function validateWebhook(webhookData) {
  const errors = [];
  
  // Validate URL
  try {
    const url = new URL(webhookData.url);
    if (url.protocol !== 'https:' && !url.hostname.includes('localhost')) {
      errors.push('Webhook URL should use HTTPS in production');
    }
  } catch {
    errors.push('Invalid webhook URL');
  }
  
  // Validate events
  if (!webhookData.events || webhookData.events.length === 0) {
    errors.push('At least one event is required');
  }
  
  // Validate custom payload if present
  if (webhookData.custom_payload) {
    try {
      JSON.parse(webhookData.custom_payload);
    } catch {
      errors.push('Custom payload must be valid JSON');
    }
  }
  
  return errors;
}
```

## Best Practices

1. **Use HTTPS**: Always use HTTPS URLs for webhooks in production
2. **Authentication**: Implement proper authentication (headers or basic auth)
3. **Specific Events**: Subscribe only to events you need to reduce noise
4. **Filters**: Use filters to limit webhooks to specific content types
5. **Custom Payloads**: Structure payloads to include only necessary data
6. **Error Handling**: Implement retry logic on your webhook endpoint
7. **Monitoring**: Regularly check webhook health and delivery status
8. **Versioning**: Use headers to version your webhook payloads

## Related Resources

- **[Build Trigger Methods](/LLM_DOCS/01-cma-client/04-site-configuration/build-trigger-methods.md)**: Trigger builds via webhooks
- **[Item Methods](/LLM_DOCS/01-cma-client/01-content-management/item-methods.md)**: Content events that trigger webhooks
- **[Environment Methods](/LLM_DOCS/01-cma-client/04-site-configuration/environment-methods.md)**: Environment events

---

# Webhook

Webhooks allow you to receive HTTP notifications when specific events occur in your DatoCMS project. You can use them to trigger builds, sync data, send notifications, or integrate with external services.

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a webhook for content updates
const webhook = await client.webhooks.create({
  name: 'Content Update Notification',
  url: 'https://api.example.com/webhooks/datocms',
  events: [
    { entity_type: 'item', event_types: ['create', 'update', 'publish'] }
  ]
});
```

## API Reference

### Create Webhook

Create a new webhook endpoint.

```typescript
const webhook = await client.webhooks.create({
  name: 'Deploy Trigger',
  url: 'https://api.netlify.com/build_hooks/YOUR_HOOK_ID',
  enabled: true,
  events: [
    { 
      entity_type: 'item',
      event_types: ['publish', 'unpublish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['blog_post', 'landing_page']
        }
      ]
    }
  ],
  headers: {
    'X-Custom-Header': 'custom-value',
    'Authorization': 'Bearer YOUR_TOKEN'
  },
  http_basic_user: 'username',
  http_basic_password: 'password',
  payload_api_version: '3',
  nested_items_in_payload: true,
  auto_retry: true
});
```

**Parameters:**
- `name` (string, required): Webhook name for identification
- `url` (string, required): HTTP endpoint to call
- `enabled` (boolean, optional): Whether webhook is active (default: true)
- `events` (array, required): Events that trigger the webhook
- `headers` (object, optional): Custom HTTP headers
- `http_basic_user` (string, optional): Basic auth username
- `http_basic_password` (string, optional): Basic auth password
- `custom_payload` (string, optional): Custom payload template
- `payload_api_version` (string, optional): API version for payload format
- `nested_items_in_payload` (boolean, optional): Include full nested items
- `auto_retry` (boolean, optional): Retry failed calls automatically

**Returns:** The created webhook object

### Update Webhook

Modify an existing webhook configuration.

```typescript
const updated = await client.webhooks.update('webhook-id', {
  name: 'Updated Webhook',
  enabled: false, // Temporarily disable
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'delete', 'publish', 'unpublish']
    },
    {
      entity_type: 'upload',
      event_types: ['create', 'update', 'delete']
    }
  ]
});
```

**Parameters:**
- `webhookId` (string, required): The webhook ID
- Update parameters (same as create, all optional)

**Returns:** The updated webhook object

### List Webhooks

Get all webhooks in the project.

```typescript
const webhooks = await client.webhooks.list();

// Find enabled webhooks
const activeWebhooks = webhooks.filter(w => w.enabled);

// Find webhooks for specific events
const publishWebhooks = webhooks.filter(w => 
  w.events.some(e => 
    e.entity_type === 'item' && e.event_types.includes('publish')
  )
);
```

**Returns:** Array of webhook objects

### Find Webhook

Retrieve a specific webhook by ID.

```typescript
const webhook = await client.webhooks.find('webhook-id');
```

**Parameters:**
- `webhookId` (string, required): The webhook ID

**Returns:** The webhook object

### Delete Webhook

Remove a webhook.

```typescript
await client.webhooks.destroy('webhook-id');
```

**Parameters:**
- `webhookId` (string, required): The webhook ID

## Event Types

### Item Events

Monitor content changes:

```typescript
{
  entity_type: 'item',
  event_types: [
    'create',     // New item created
    'update',     // Item modified
    'delete',     // Item deleted
    'publish',    // Item published
    'unpublish'   // Item unpublished
  ],
  filters: [
    {
      entity_type: 'item_type',
      entity_ids: ['article', 'page'] // Only these models
    }
  ]
}
```

### Upload Events

Track media changes:

```typescript
{
  entity_type: 'upload',
  event_types: [
    'create',  // New upload
    'update',  // Upload metadata changed
    'delete'   // Upload deleted
  ]
}
```

### Model Events

Monitor schema changes:

```typescript
{
  entity_type: 'item_type',
  event_types: [
    'create',  // New model created
    'update',  // Model updated
    'delete'   // Model deleted
  ]
}
```

### Other Entity Types

```typescript
// Field changes
{ entity_type: 'field', event_types: ['create', 'update', 'delete'] }

// User management
{ entity_type: 'user', event_types: ['create', 'update', 'delete'] }

// Role changes
{ entity_type: 'role', event_types: ['create', 'update', 'delete'] }

// Environment changes
{ entity_type: 'environment', event_types: ['create', 'fork', 'promote', 'delete'] }
```

## Webhook Payloads

### Standard Payload

The default webhook payload structure:

```typescript
{
  entity_type: 'item',
  event_type: 'publish',
  entity: {
    id: '12345',
    type: 'item',
    attributes: {
      title: 'Article Title',
      content: 'Article content...'
    },
    relationships: {
      item_type: {
        data: { type: 'item_type', id: 'article' }
      }
    }
  },
  related_entities: [
    // Nested items if nested_items_in_payload is true
  ]
}
```

### Custom Payload

Use Liquid templating for custom payloads:

```typescript
const webhook = await client.webhooks.create({
  name: 'Slack Notification',
  url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL',
  custom_payload: `{
    "text": "Content updated!",
    "blocks": [
      {
        "type": "section",
        "text": {
          "type": "mrkdwn",
          "text": "*{{ entity.attributes.title }}* was {{ event_type }}d"
        }
      },
      {
        "type": "context",
        "elements": [
          {
            "type": "mrkdwn",
            "text": "Model: {{ entity.relationships.item_type.data.id }} | Environment: {{ environment }}"
          }
        ]
      }
    ]
  }`,
  headers: {
    'Content-Type': 'application/json'
  },
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});
```

### Available Variables in Custom Payloads

- `{{ entity }}` - The affected entity
- `{{ event_type }}` - The event that occurred
- `{{ entity_type }}` - Type of entity
- `{{ environment }}` - Environment ID
- `{{ related_entities }}` - Array of related entities

## Advanced Usage

### Filtering by Environment

Webhooks can be environment-specific:

```typescript
// Create webhook only for production
const productionWebhook = await client.webhooks.create({
  name: 'Production Deploy',
  url: 'https://api.vercel.com/v1/deployments',
  events: [
    {
      entity_type: 'item',
      event_types: ['publish'],
      environments: ['main'] // Only trigger in main environment
    }
  ]
});
```

### Conditional Webhooks

Filter events based on specific criteria:

```typescript
const webhook = await client.webhooks.create({
  name: 'Homepage Update Alert',
  url: 'https://api.example.com/notify',
  events: [
    {
      entity_type: 'item',
      event_types: ['update', 'publish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['homepage'] // Only homepage model
        },
        {
          entity_type: 'item',
          entity_ids: ['homepage-singleton-id'] // Specific item
        }
      ]
    }
  ]
});
```

### Build Triggers

Common pattern for triggering builds:

```typescript
// Netlify
const netlifyWebhook = await client.webhooks.create({
  name: 'Netlify Build',
  url: 'https://api.netlify.com/build_hooks/YOUR_BUILD_HOOK_ID',
  events: [
    { entity_type: 'item', event_types: ['publish', 'unpublish'] }
  ]
});

// Vercel
const vercelWebhook = await client.webhooks.create({
  name: 'Vercel Deploy',
  url: 'https://api.vercel.com/v1/deployments',
  headers: {
    'Authorization': 'Bearer YOUR_VERCEL_TOKEN'
  },
  custom_payload: '{"gitSource": {"ref": "main", "repoId": "YOUR_REPO_ID"}}',
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});

// GitHub Actions
const githubWebhook = await client.webhooks.create({
  name: 'GitHub Action Trigger',
  url: 'https://api.github.com/repos/OWNER/REPO/dispatches',
  headers: {
    'Authorization': 'token YOUR_GITHUB_TOKEN',
    'Accept': 'application/vnd.github.v3+json'
  },
  custom_payload: '{"event_type": "datocms-update"}',
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});
```

### Webhook Security

Implement webhook verification:

```typescript
// DatoCMS signs webhook payloads
const webhook = await client.webhooks.create({
  name: 'Secure Webhook',
  url: 'https://api.example.com/webhook',
  events: [{ entity_type: 'item', event_types: ['create'] }]
});

// In your webhook endpoint (Express example):
import crypto from 'crypto';

app.post('/webhook', (req, res) => {
  const signature = req.headers['x-datocms-signature'];
  const body = JSON.stringify(req.body);
  
  // Verify signature
  const expectedSignature = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(body)
    .digest('hex');
    
  if (signature !== expectedSignature) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process webhook
  console.log('Verified webhook:', req.body);
  res.status(200).send('OK');
});
```

### Retry Configuration

Handle failures gracefully:

```typescript
const reliableWebhook = await client.webhooks.create({
  name: 'Reliable Webhook',
  url: 'https://api.example.com/webhook',
  auto_retry: true, // Enable automatic retries
  events: [
    { entity_type: 'item', event_types: ['publish'] }
  ]
});

// DatoCMS will retry failed webhooks:
// - After 10 seconds
// - After 1 minute
// - After 10 minutes
// - After 1 hour
// - After 3 hours
```

### Monitoring Webhook Calls

Check webhook execution history:

```typescript
// Get webhook calls for a specific webhook
const calls = await client.webhookCalls.list('webhook-id');

// Check failed calls
const failedCalls = calls.filter(call => 
  call.response_status >= 400 || call.response_status === null
);

// Resend a failed webhook
await client.webhookCalls.resend('webhook-call-id');
```

**Related:** See [Webhook Calls](../06-monitoring/webhook-call.md) for detailed call history

## Error Handling

```typescript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.webhooks.create({
    name: 'Test Webhook',
    url: 'invalid-url', // Invalid URL
    events: []
  });
} catch (error) {
  if (error instanceof ApiError) {
    const urlError = error.findError('url');
    if (urlError?.code === 'VALIDATION_FORMAT') {
      console.log('Invalid webhook URL format');
    }
  }
}
```

## Webhook Object Structure

```typescript
interface Webhook {
  id: string;
  name: string;
  url: string;
  enabled: boolean;
  headers: Record<string, string>;
  events: Array<{
    entity_type: string;
    event_types: string[];
    filters?: Array<{
      entity_type: string;
      entity_ids: string[];
    }>;
    environments?: string[];
  }>;
  http_basic_user?: string;
  http_basic_password?: string;
  custom_payload?: string;
  payload_api_version: string;
  nested_items_in_payload: boolean;
  auto_retry: boolean;
  created_at: string;
  updated_at: string;
}
```

## Related Resources

- [Webhook Calls](../06-monitoring/webhook-call.md) - Monitor webhook executions
- [Build Triggers](./build-triggers.md) - Alternative to webhooks for builds
- Real-time Events - Task-oriented guide
- [Job Results](./job-results.md) - Track async operations

---

# Webhook Methods - Complete Reference

This document contains all methods available for the Webhook resource in the DatoCMS CMA Client.

## Available Methods for client.webhooks

### list() - Retrieve All Webhooks

Retrieves all webhooks configured for the project.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get all webhooks
const webhooks = await client.webhooks.list();

// Filter active webhooks
const activeWebhooks = webhooks.filter(webhook => webhook.enabled);

// Group by event type
const webhooksByEvent = webhooks.reduce((acc, webhook) => {
  webhook.events.forEach(event => {
    const key = `${event.entity_type}:${event.event_types.join(',')}`;
    if (!acc[key]) acc[key] = [];
    acc[key].push(webhook);
  });
  return acc;
}, {});

// Get raw response
const rawResponse = await client.webhooks.rawList();
```

### find() - Get Single Webhook

Retrieves a specific webhook by ID.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Get webhook by ID
const webhook = await client.webhooks.find('webhook_id');

// Webhook structure includes:
// {
//   id: 'webhook_id',
//   type: 'webhook',
//   attributes: {
//     name: 'Content Published Notification',
//     url: 'https://api.example.com/webhooks/datocms',
//     custom_payload: '{"event": "{{event_type}}", "entity": "{{entity.id}}"}',
//     headers: {
//       'X-Api-Key': 'secret-key',
//       'Content-Type': 'application/json'
//     },
//     http_basic_user: null,
//     http_basic_password: null,
//     enabled: true,
//     payload_api_version: '3',
//     nested_items_in_payload: true,
//     events: [
//       {
//         entity_type: 'item',
//         event_types: ['publish', 'unpublish'],
//         filters: [
//           {
//             entity_type: 'item_type',
//             entity_ids: ['blog_post', 'article']
//           }
//         ]
//       }
//     ]
//   }
// }

// Error handling
try {
  const webhook = await client.webhooks.find('invalid_id');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Webhook not found');
  }
}
```

### create() - Create New Webhook

Creates a new webhook with specified configuration.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create simple webhook
const simpleWebhook = await client.webhooks.create({
  name: 'New Content Alert',
  url: 'https://api.example.com/webhooks/new-content',
  events: [
    {
      entity_type: 'item',
      event_types: ['create']
    }
  ]
});

// Create webhook with filters
const filteredWebhook = await client.webhooks.create({
  name: 'Blog Post Published',
  url: 'https://api.example.com/webhooks/blog-published',
  events: [
    {
      entity_type: 'item',
      event_types: ['publish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['blog_post'] // Only blog posts
        }
      ]
    }
  ],
  enabled: true
});

// Create webhook with custom payload
const customPayloadWebhook = await client.webhooks.create({
  name: 'Custom Notification',
  url: 'https://api.example.com/webhooks/custom',
  custom_payload: JSON.stringify({
    event: '{{event_type}}',
    entity_id: '{{entity.id}}',
    entity_type: '{{entity_type}}',
    timestamp: '{{timestamp}}',
    item_data: {
      title: '{{entity.title}}',
      slug: '{{entity.slug}}',
      published_at: '{{entity.meta.published_at}}'
    }
  }),
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'publish', 'unpublish']
    }
  ]
});

// Create webhook with authentication
const authenticatedWebhook = await client.webhooks.create({
  name: 'Secure Webhook',
  url: 'https://api.example.com/secure-webhook',
  headers: {
    'Authorization': 'Bearer YOUR_SECRET_TOKEN',
    'X-Custom-Header': 'custom-value'
  },
  events: [
    {
      entity_type: 'item',
      event_types: ['publish']
    }
  ]
});

// Create webhook with HTTP Basic Auth
const basicAuthWebhook = await client.webhooks.create({
  name: 'Basic Auth Webhook',
  url: 'https://api.example.com/webhook',
  http_basic_user: 'username',
  http_basic_password: 'password',
  events: [
    {
      entity_type: 'item',
      event_types: ['create']
    }
  ]
});

// Create comprehensive webhook for multiple events
const comprehensiveWebhook = await client.webhooks.create({
  name: 'All Content Changes',
  url: 'https://api.example.com/webhooks/all-changes',
  enabled: true,
  nested_items_in_payload: true,
  payload_api_version: '3',
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'delete', 'publish', 'unpublish'],
      filters: []
    },
    {
      entity_type: 'item_type',
      event_types: ['create', 'update', 'delete'],
      filters: []
    },
    {
      entity_type: 'upload',
      event_types: ['create', 'update', 'delete'],
      filters: []
    }
  ]
});

// Error handling
try {
  const webhook = await client.webhooks.create({
    name: 'Invalid Webhook',
    url: 'not-a-valid-url',
    events: []
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('Validation errors:', error.response.data.errors);
  }
}
```

### update() - Update Webhook

Updates an existing webhook configuration.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Update webhook URL
const updated = await client.webhooks.update('webhook_id', {
  url: 'https://new-api.example.com/webhooks'
});

// Enable/disable webhook
const disabled = await client.webhooks.update('webhook_id', {
  enabled: false
});

// Update events
const updatedEvents = await client.webhooks.update('webhook_id', {
  events: [
    {
      entity_type: 'item',
      event_types: ['publish', 'unpublish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['blog_post', 'news_article']
        }
      ]
    }
  ]
});

// Update custom payload
const updatedPayload = await client.webhooks.update('webhook_id', {
  custom_payload: JSON.stringify({
    notification: {
      event: '{{event_type}}',
      item: {
        id: '{{entity.id}}',
        type: '{{entity.item_type.api_key}}',
        title: '{{entity.title}}'
      },
      user: '{{performer.email}}',
      timestamp: '{{timestamp}}'
    }
  })
});

// Update headers
const updatedHeaders = await client.webhooks.update('webhook_id', {
  headers: {
    'Authorization': 'Bearer NEW_TOKEN',
    'X-Webhook-Version': '2.0'
  }
});

// Complex update
const complexUpdate = await client.webhooks.update('webhook_id', {
  name: 'Updated Webhook Name',
  url: 'https://api.example.com/v2/webhooks',
  enabled: true,
  headers: {
    'X-API-Version': '2'
  },
  events: [
    {
      entity_type: 'item',
      event_types: ['create', 'update', 'publish'],
      filters: [
        {
          entity_type: 'item_type',
          entity_ids: ['product', 'category']
        }
      ]
    }
  ]
});

// Error handling
try {
  const updated = await client.webhooks.update('webhook_id', {
    events: [] // Cannot have empty events
  });
} catch (error) {
  if (error.response?.status === 422) {
    console.error('At least one event is required');
  }
}
```

### destroy() - Delete Webhook

Deletes a webhook permanently.

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Delete webhook
await client.webhooks.destroy('webhook_id');

// Delete with error handling
try {
  await client.webhooks.destroy('webhook_id');
  console.log('Webhook deleted successfully');
} catch (error) {
  if (error.response?.status === 404) {
    console.error('Webhook not found');
  }
}

// Bulk delete pattern
async function deleteWebhooksByName(pattern) {
  const webhooks = await client.webhooks.list();
  const toDelete = webhooks.filter(w => w.name.includes(pattern));
  
  const results = await Promise.allSettled(
    toDelete.map(webhook => client.webhooks.destroy(webhook.id))
  );
  
  return {
    deleted: results.filter(r => r.status === 'fulfilled').length,
    failed: results.filter(r => r.status === 'rejected').length
  };
}
```

## Webhook Event Types

### Available Entity Types and Events

```typescript
// Item events
const itemWebhook = {
  entity_type: 'item',
  event_types: [
    'create',     // Item created
    'update',     // Item updated
    'delete',     // Item deleted
    'publish',    // Item published
    'unpublish'   // Item unpublished
  ]
};

// Item Type (Model) events
const itemTypeWebhook = {
  entity_type: 'item_type',
  event_types: [
    'create',     // Model created
    'update',     // Model updated
    'delete'      // Model deleted
  ]
};

// Upload events
const uploadWebhook = {
  entity_type: 'upload',
  event_types: [
    'create',     // File uploaded
    'update',     // File metadata updated
    'delete'      // File deleted
  ]
};

// Environment events
const environmentWebhook = {
  entity_type: 'environment',
  event_types: [
    'create',     // Environment forked
    'promote',    // Environment promoted to primary
    'delete'      // Environment deleted
  ]
};

// Build trigger events
const buildTriggerWebhook = {
  entity_type: 'build_trigger',
  event_types: [
    'trigger'     // Build triggered
  ]
};
```

## Webhook Patterns

### Content Publishing Notifications

```typescript
async function setupPublishingNotifications(notificationUrl) {
  // Notify when any content is published
  const publishWebhook = await client.webhooks.create({
    name: 'Content Published',
    url: notificationUrl,
    custom_payload: JSON.stringify({
      text: 'New content published!',
      content: {
        id: '{{entity.id}}',
        title: '{{entity.title}}',
        url: 'https://example.com/{{entity.slug}}'
      }
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['publish']
      }
    ]
  });
  
  // Notify when content is scheduled
  const scheduleWebhook = await client.webhooks.create({
    name: 'Content Scheduled',
    url: notificationUrl,
    custom_payload: JSON.stringify({
      text: 'Content scheduled for publication',
      content: {
        id: '{{entity.id}}',
        title: '{{entity.title}}',
        scheduled_at: '{{entity.meta.publication_scheduled_at}}'
      }
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['update'],
        filters: []
      }
    ]
  });
  
  return { publishWebhook, scheduleWebhook };
}
```

### Slack Integration

```typescript
async function createSlackWebhook(slackWebhookUrl, channel) {
  return client.webhooks.create({
    name: 'Slack Notifications',
    url: slackWebhookUrl,
    custom_payload: JSON.stringify({
      channel: channel,
      username: 'DatoCMS',
      icon_emoji: ':pencil:',
      text: 'Content update in DatoCMS',
      attachments: [{
        color: 'good',
        title: '{{entity.title}}',
        fields: [
          {
            title: 'Event',
            value: '{{event_type}}',
            short: true
          },
          {
            title: 'Type',
            value: '{{entity.item_type.name}}',
            short: true
          },
          {
            title: 'Updated by',
            value: '{{performer.email}}',
            short: true
          },
          {
            title: 'Time',
            value: '{{timestamp}}',
            short: true
          }
        ]
      }]
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['create', 'update', 'publish']
      }
    ]
  });
}
```

### Build Trigger Integration

```typescript
async function setupAutoBuild(buildUrl, buildToken) {
  // Trigger build on content changes
  const buildWebhook = await client.webhooks.create({
    name: 'Auto Build Trigger',
    url: buildUrl,
    headers: {
      'Authorization': `Bearer ${buildToken}`,
      'X-Trigger-Type': 'content-change'
    },
    custom_payload: JSON.stringify({
      trigger: 'content_update',
      event: '{{event_type}}',
      entity: '{{entity_type}}',
      id: '{{entity.id}}'
    }),
    events: [
      {
        entity_type: 'item',
        event_types: ['publish', 'unpublish']
      }
    ]
  });
  
  // Trigger build on model changes
  const schemaWebhook = await client.webhooks.create({
    name: 'Schema Change Build',
    url: buildUrl,
    headers: {
      'Authorization': `Bearer ${buildToken}`,
      'X-Trigger-Type': 'schema-change'
    },
    events: [
      {
        entity_type: 'item_type',
        event_types: ['create', 'update', 'delete']
      }
    ]
  });
  
  return { buildWebhook, schemaWebhook };
}
```

### Audit Log Webhook

```typescript
async function createAuditWebhook(auditEndpoint, apiKey) {
  return client.webhooks.create({
    name: 'Audit Log',
    url: auditEndpoint,
    headers: {
      'X-API-Key': apiKey,
      'Content-Type': 'application/json'
    },
    custom_payload: JSON.stringify({
      timestamp: '{{timestamp}}',
      event_type: '{{event_type}}',
      entity_type: '{{entity_type}}',
      entity_id: '{{entity.id}}',
      performer: {
        id: '{{performer.id}}',
        email: '{{performer.email}}',
        type: '{{performer.type}}'
      },
      entity_data: '{{entity}}',
      environment: '{{environment.id}}'
    }),
    nested_items_in_payload: true,
    events: [
      {
        entity_type: 'item',
        event_types: ['create', 'update', 'delete', 'publish', 'unpublish']
      },
      {
        entity_type: 'item_type',
        event_types: ['create', 'update', 'delete']
      },
      {
        entity_type: 'upload',
        event_types: ['create', 'update', 'delete']
      }
    ]
  });
}
```

### Webhook Testing

```typescript
async function testWebhook(webhookId) {
  const webhook = await client.webhooks.find(webhookId);
  
  // Create a test item to trigger the webhook
  const testItem = await client.items.create({
    item_type: { type: 'item_type', id: 'test_model' },
    title: 'Webhook Test Item',
    test_field: 'This should trigger the webhook'
  });
  
  // Perform action based on webhook events
  for (const event of webhook.events) {
    if (event.event_types.includes('publish')) {
      await client.items.publish(testItem.id);
    }
    if (event.event_types.includes('unpublish')) {
      await client.items.unpublish(testItem.id);
    }
  }
  
  // Clean up
  setTimeout(async () => {
    await client.items.destroy(testItem.id);
  }, 5000);
  
  return {
    webhook: webhook.name,
    testItem: testItem.id,
    triggeredEvents: webhook.events
  };
}
```

### Webhook Monitoring

```typescript
async function getWebhookHealth() {
  const webhooks = await client.webhooks.list();
  
  const health = webhooks.map(webhook => ({
    id: webhook.id,
    name: webhook.name,
    url: webhook.url,
    enabled: webhook.enabled,
    eventCount: webhook.events.reduce(
      (sum, event) => sum + event.event_types.length, 
      0
    ),
    hasFilters: webhook.events.some(e => e.filters && e.filters.length > 0),
    hasCustomPayload: !!webhook.custom_payload,
    hasAuth: !!(webhook.headers?.Authorization || webhook.http_basic_user)
  }));
  
  return {
    total: webhooks.length,
    enabled: health.filter(w => w.enabled).length,
    disabled: health.filter(w => !w.enabled).length,
    webhooks: health
  };
}

// Find misconfigured webhooks
async function findMisconfiguredWebhooks() {
  const webhooks = await client.webhooks.list();
  const issues = [];
  
  for (const webhook of webhooks) {
    const webhookIssues = [];
    
    // Check for localhost URLs in production
    if (webhook.url.includes('localhost') || webhook.url.includes('127.0.0.1')) {
      webhookIssues.push('URL points to localhost');
    }
    
    // Check for HTTP in production
    if (webhook.url.startsWith('http://') && !webhook.url.includes('localhost')) {
      webhookIssues.push('Using insecure HTTP');
    }
    
    // Check for missing authentication
    if (!webhook.headers?.Authorization && !webhook.http_basic_user) {
      webhookIssues.push('No authentication configured');
    }
    
    // Check for overly broad events
    const hasUnfilteredEvents = webhook.events.some(
      event => (!event.filters || event.filters.length === 0) && 
               event.event_types.length > 3
    );
    if (hasUnfilteredEvents) {
      webhookIssues.push('Very broad event configuration without filters');
    }
    
    if (webhookIssues.length > 0) {
      issues.push({
        webhook: webhook.name,
        id: webhook.id,
        issues: webhookIssues
      });
    }
  }
  
  return issues;
}
```

---

# WhiteLabelSettings

The WhiteLabelSettings resource allows you to customize the DatoCMS interface by providing custom internationalization (i18n) messages. This enterprise feature enables you to replace default UI text with your own terminology, supporting brand consistency and industry-specific language requirements.

## Available Operations

### Retrieve White Label Settings

```javascript
const settings = await client.whiteLabelSettings.find();
```

**Returns:** Current white label configuration

### Update White Label Settings

```javascript
const updated = await client.whiteLabelSettings.update({
  custom_i18n_messages_template_url: "https://translations.example.com/:locale/messages.json"
});
```

**Parameters:**
- `custom_i18n_messages_template_url` (string | null): URL template for custom translations
  - Must include `:locale` placeholder
  - Set to `null` to remove customization

**Returns:** Updated WhiteLabelSettings object

## The WhiteLabelSettings Object

```typescript
{
  id: string;                                    // Settings identifier
  type: 'white_label_settings';                  // Always 'white_label_settings'
  custom_i18n_messages_template_url: string | null;  // Custom translations URL
}
```

## Common Use Cases

### 1. Basic White Label Setup

```javascript
// Configure custom translations for your organization
async function setupWhiteLabel(client, translationsBaseUrl) {
  try {
    // Check current settings
    const currentSettings = await client.whiteLabelSettings.find();
    console.log('Current URL:', currentSettings.custom_i18n_messages_template_url);
    
    // Update with new translations URL
    const settings = await client.whiteLabelSettings.update({
      custom_i18n_messages_template_url: `${translationsBaseUrl}/:locale/datocms-ui.json`
    });
    
    console.log('✅ White label configured:', settings.custom_i18n_messages_template_url);
    
    return settings;
  } catch (error) {
    console.error('Failed to configure white label:', error.message);
    throw error;
  }
}

// Usage
await setupWhiteLabel(client, 'https://cdn.company.com/translations');
```

### 2. Multi-Language Customization

```javascript
// Manage translations for multiple languages
class TranslationManager {
  constructor(client, baseUrl) {
    this.client = client;
    this.baseUrl = baseUrl;
    this.supportedLocales = ['en', 'es', 'fr', 'de', 'it', 'ja'];
  }
  
  async deployTranslations(translations) {
    // First, deploy translation files to your CDN
    for (const locale of this.supportedLocales) {
      const messages = translations[locale] || translations['en']; // Fallback to English
      await this.uploadTranslationFile(locale, messages);
    }
    
    // Then update DatoCMS to use the translations
    const settings = await this.client.whiteLabelSettings.update({
      custom_i18n_messages_template_url: `${this.baseUrl}/:locale/messages.json`
    });
    
    console.log('Translations deployed for locales:', this.supportedLocales.join(', '));
    
    return settings;
  }
  
  async uploadTranslationFile(locale, messages) {
    // Implementation depends on your CDN/hosting solution
    // Example: Upload to S3, Netlify, Vercel, etc.
    console.log(`Uploading ${locale} translations...`);
    
    // Ensure proper JSON structure
    const validatedMessages = this.validateMessages(messages);
    
    // Upload logic here
    // await uploadToS3(`${locale}/messages.json`, JSON.stringify(validatedMessages));
  }
  
  validateMessages(messages) {
    // Ensure messages follow DatoCMS structure
    const required = {
      'header.projects': messages['header.projects'] || 'Projects',
      'header.account': messages['header.account'] || 'Account',
      // Add more required keys as needed
    };
    
    return { ...required, ...messages };
  }
}

// Usage
const manager = new TranslationManager(client, 'https://translations.mycompany.com');

await manager.deployTranslations({
  en: {
    'header.projects': 'Workspaces',
    'header.account': 'My Profile',
    'content.item': 'Content Entry',
    'content.items': 'Content Entries'
  },
  es: {
    'header.projects': 'Espacios de Trabajo',
    'header.account': 'Mi Perfil',
    'content.item': 'Entrada de Contenido',
    'content.items': 'Entradas de Contenido'
  }
});
```

### 3. Industry-Specific Terminology

```javascript
// Customize DatoCMS for different industries
const industryTerminology = {
  healthcare: {
    'content.item': 'Patient Record',
    'content.items': 'Patient Records',
    'media.upload': 'Medical Image',
    'media.uploads': 'Medical Images',
    'content.publish': 'Approve for Clinical Use',
    'content.unpublish': 'Revoke Clinical Approval'
  },
  
  ecommerce: {
    'content.item': 'Product',
    'content.items': 'Products',
    'media.upload': 'Product Image',
    'media.uploads': 'Product Images',
    'content.publish': 'Make Available for Sale',
    'content.unpublish': 'Remove from Store'
  },
  
  education: {
    'content.item': 'Course Material',
    'content.items': 'Course Materials',
    'media.upload': 'Educational Resource',
    'media.uploads': 'Educational Resources',
    'content.publish': 'Release to Students',
    'content.unpublish': 'Archive Material'
  }
};

async function applyIndustryTerminology(client, industry) {
  const terms = industryTerminology[industry];
  
  if (!terms) {
    throw new Error(`Unknown industry: ${industry}`);
  }
  
  // Deploy industry-specific translations
  // This is a simplified example - in production, you'd merge with full translations
  const translationsUrl = await deployIndustryTranslations(industry, terms);
  
  // Update DatoCMS settings
  const settings = await client.whiteLabelSettings.update({
    custom_i18n_messages_template_url: translationsUrl
  });
  
  console.log(`✅ Applied ${industry} terminology`);
  
  return settings;
}
```

### 4. Dynamic Translation Updates

```javascript
// System for hot-reloading translations without DatoCMS API calls
class DynamicTranslationSystem {
  constructor(client) {
    this.client = client;
    this.cdnUrl = 'https://translations-api.company.com';
  }
  
  async initialize() {
    // Set up the URL template once
    const settings = await this.client.whiteLabelSettings.update({
      custom_i18n_messages_template_url: `${this.cdnUrl}/api/translations/:locale`
    });
    
    console.log('Dynamic translation system initialized');
    
    return settings;
  }
  
  // The actual translation updates happen on your server
  // DatoCMS will fetch fresh translations on each page load
  async updateTranslations(locale, updates) {
    // This updates your translation API, not DatoCMS
    const response = await fetch(`${this.cdnUrl}/api/translations/${locale}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updates)
    });
    
    if (!response.ok) {
      throw new Error('Failed to update translations');
    }
    
    console.log(`Updated ${locale} translations on CDN`);
    
    // Changes will be reflected immediately in DatoCMS UI
  }
}

// Usage
const translationSystem = new DynamicTranslationSystem(client);
await translationSystem.initialize();

// Later, update translations without touching DatoCMS API
await translationSystem.updateTranslations('en', {
  'content.save': 'Save Changes',
  'content.publish': 'Go Live'
});
```

### 5. White Label Verification

```javascript
// Verify white label setup and test translations
async function verifyWhiteLabelSetup(client) {
  const report = {
    configured: false,
    url: null,
    localesChecked: [],
    errors: [],
    warnings: []
  };
  
  try {
    // Check if white label is configured
    const settings = await client.whiteLabelSettings.find();
    report.url = settings.custom_i18n_messages_template_url;
    report.configured = !!report.url;
    
    if (!report.configured) {
      report.warnings.push('White label not configured');
      return report;
    }
    
    // Test translation URLs for common locales
    const testLocales = ['en', 'es', 'fr', 'de', 'it'];
    
    for (const locale of testLocales) {
      const url = report.url.replace(':locale', locale);
      
      try {
        const response = await fetch(url);
        
        if (response.ok) {
          const messages = await response.json();
          report.localesChecked.push({
            locale,
            status: 'ok',
            messageCount: Object.keys(messages).length
          });
        } else {
          report.errors.push(`${locale}: HTTP ${response.status}`);
        }
      } catch (error) {
        report.errors.push(`${locale}: ${error.message}`);
      }
    }
    
    // Check for common issues
    if (report.localesChecked.length === 0) {
      report.errors.push('No translations could be loaded');
    }
    
    const englishMessages = report.localesChecked.find(l => l.locale === 'en');
    if (englishMessages && englishMessages.messageCount < 10) {
      report.warnings.push('English translations seem incomplete');
    }
    
  } catch (error) {
    report.errors.push(`Setup check failed: ${error.message}`);
  }
  
  return report;
}

// Run verification
const report = await verifyWhiteLabelSetup(client);
console.log('White Label Status:', report.configured ? '✅ Configured' : '❌ Not configured');
console.log('URL:', report.url || 'None');
console.log('Locales tested:', report.localesChecked.length);

if (report.errors.length > 0) {
  console.error('Errors:', report.errors);
}
```

### 6. Remove White Label Customization

```javascript
// Reset to default DatoCMS interface
async function removeWhiteLabel(client) {
  try {
    const settings = await client.whiteLabelSettings.update({
      custom_i18n_messages_template_url: null
    });
    
    console.log('✅ White label customization removed');
    console.log('DatoCMS will now use default interface text');
    
    return settings;
  } catch (error) {
    console.error('Failed to remove white label:', error.message);
    throw error;
  }
}
```

## Translation File Format

Your custom translation files should follow this JSON structure:

```json
{
  "header.projects": "Custom Projects Label",
  "header.account": "Custom Account Label",
  "content.item": "Custom Item Label",
  "content.items": "Custom Items Label",
  "content.new": "Create New",
  "content.edit": "Modify",
  "content.delete": "Remove",
  "content.publish": "Make Live",
  "content.unpublish": "Take Offline",
  "media.upload": "Custom Upload Label",
  "media.uploads": "Custom Uploads Label"
}
```

## Important Considerations

### Plan Requirements
- White label features require an **Enterprise plan**
- Check availability with subscription features API
- Contact sales for plan upgrades

### Technical Requirements
1. **HTTPS Required**: Translation URLs must use HTTPS
2. **CORS Headers**: Your translation endpoint must allow DatoCMS origins
3. **Caching**: Consider CDN caching for performance
4. **Availability**: Ensure high availability for translation endpoints

### Best Practices
1. **Version Control**: Keep translations in version control
2. **Fallbacks**: Always provide English fallback translations
3. **Testing**: Test all locales before deploying
4. **Monitoring**: Monitor translation endpoint availability
5. **Documentation**: Document custom terminology for team members

## Error Handling

```javascript
try {
  await client.whiteLabelSettings.update({
    custom_i18n_messages_template_url: 'invalid-url'
  });
} catch (error) {
  if (error.message.includes('VALIDATION_ERROR')) {
    console.error('Invalid URL format - must include :locale placeholder');
  } else if (error.message.includes('FORBIDDEN')) {
    console.error('White label not available on your plan');
  }
}
```

## Raw Response Access

```javascript
const response = await client.whiteLabelSettings.rawFind();
console.log(response.data); // Raw white label settings
```

## Related Resources

- [Subscription Features](/01-cma-client/07-monitoring-usage/subscription-feature.md) - Check feature availability
- [Sites](/01-cma-client/04-site-configuration/site.md) - Project configuration
- [Locales](https://www.datocms.com/docs/localization) - Content localization

---

# White Label Settings Methods

This file documents all available methods for managing White Label Settings in DatoCMS. White Label Settings allow you to customize the DatoCMS interface with your own branding, including logos, colors, custom CSS, and support links.

## Available Methods

### find()

Retrieve the current white label settings for the project.

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function getWhiteLabelSettings() {
  try {
    // Get current white label settings
    const whiteLabelSettings = await client.whiteLabelSettings.find();
    
    console.log('Current white label settings:', whiteLabelSettings);
    
    return whiteLabelSettings;
  } catch (error) {
    console.error('Error fetching white label settings:', error);
    throw error;
  }
}

// Example response structure
const exampleResponse: SimpleSchemaTypes.WhiteLabelSettings = {
  id: '123456',
  type: 'white_label_setting',
  // Logo configuration
  logo: {
    url: 'https://www.datocms-assets.com/12345/1234567890-logo.png',
    alt: null,
    title: null,
    custom_data: {},
    focal_point: null,
    upload_id: '789012'
  },
  // Favicon configuration
  favicon: {
    url: 'https://www.datocms-assets.com/12345/1234567891-favicon.ico',
    alt: null,
    title: null,
    custom_data: {},
    focal_point: null,
    upload_id: '789013'
  },
  // Color customization
  primary_color: {
    red: 18,
    green: 108,
    blue: 237,
    alpha: 255
  },
  light_color: {
    red: 229,
    green: 239,
    blue: 252,
    alpha: 255
  },
  dark_color: {
    red: 14,
    green: 83,
    blue: 184,
    alpha: 255
  },
  accent_color: {
    red: 255,
    green: 107,
    blue: 0,
    alpha: 255
  },
  // Custom CSS
  custom_css: `
    /* Custom branding CSS */
    .DatoCMS-button--primary {
      border-radius: 8px;
    }
    
    .DatoCMS-navbar {
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
  `,
  // Support customization
  custom_support_url: 'https://support.example.com',
  custom_support_email: 'support@example.com',
  // Login page customization
  login_page_copy: 'Welcome to our content management system. Please log in to continue.',
  // Domain configuration
  custom_domain: 'cms.example.com',
  custom_domain_with_ssl: true
};
```

### update()

Update the white label settings with custom branding, colors, and configuration.

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function updateWhiteLabelSettings() {
  try {
    // Update white label settings with custom branding
    const updatedSettings = await client.whiteLabelSettings.update({
      // Logo upload (requires upload_id from a previously uploaded asset)
      logo: {
        upload_id: '789012',
        alt: null,
        title: null,
        custom_data: {},
        focal_point: null
      },
      // Favicon upload
      favicon: {
        upload_id: '789013',
        alt: null,
        title: null,
        custom_data: {},
        focal_point: null
      },
      // Brand colors (RGBA format)
      primary_color: {
        red: 18,
        green: 108,
        blue: 237,
        alpha: 255
      },
      light_color: {
        red: 229,
        green: 239,
        blue: 252,
        alpha: 255
      },
      dark_color: {
        red: 14,
        green: 83,
        blue: 184,
        alpha: 255
      },
      accent_color: {
        red: 255,
        green: 107,
        blue: 0,
        alpha: 255
      },
      // Custom CSS for additional styling
      custom_css: `
        /* Override default styles */
        .DatoCMS-button--primary {
          border-radius: 8px;
          font-weight: 600;
        }
        
        .DatoCMS-navbar {
          background-color: #f8f9fa;
          box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .DatoCMS-logo {
          margin: 20px;
        }
        
        /* Custom form styling */
        .DatoCMS-field__label {
          text-transform: uppercase;
          font-size: 12px;
          letter-spacing: 0.5px;
        }
      `,
      // Support customization
      custom_support_url: 'https://support.example.com',
      custom_support_email: 'support@example.com',
      // Login page customization
      login_page_copy: 'Welcome to Example CMS. Please enter your credentials to access the content management system.',
      // Custom domain configuration
      custom_domain: 'cms.example.com',
      custom_domain_with_ssl: true
    });
    
    console.log('White label settings updated:', updatedSettings);
    
    return updatedSettings;
  } catch (error) {
    console.error('Error updating white label settings:', error);
    throw error;
  }
}
```

## Complete Examples

### Uploading and Setting a Custom Logo

```typescript
import { buildClient } from '@datocms/cma-client';
import { createReadStream } from 'fs';
import { resolve } from 'path';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function uploadAndSetLogo(logoPath: string) {
  try {
    // Step 1: Upload the logo file
    const upload = await client.uploads.create({
      path: createReadStream(resolve(logoPath)),
      filename: 'company-logo.png',
      default_field_metadata: {
        en: {
          alt: 'Company Logo',
          title: 'Our Company Logo',
          custom_data: {}
        }
      }
    });
    
    console.log('Logo uploaded:', upload.id);
    
    // Step 2: Set the uploaded logo in white label settings
    const whiteLabelSettings = await client.whiteLabelSettings.update({
      logo: {
        upload_id: upload.id,
        alt: 'Company Logo',
        title: null,
        custom_data: {},
        focal_point: null
      }
    });
    
    console.log('Logo set in white label settings');
    
    return whiteLabelSettings;
  } catch (error) {
    console.error('Error uploading and setting logo:', error);
    throw error;
  }
}

// Usage
uploadAndSetLogo('./assets/logo.png');
```

### Setting Up Complete Brand Identity

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface BrandIdentity {
  logoUploadId: string;
  faviconUploadId: string;
  primaryColor: string;
  accentColor: string;
  supportEmail: string;
  supportUrl: string;
}

async function setupBrandIdentity(brand: BrandIdentity) {
  try {
    // Convert hex colors to RGBA
    const hexToRgba = (hex: string): SimpleSchemaTypes.ColorField => {
      const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
      if (!result) {
        throw new Error('Invalid hex color');
      }
      return {
        red: parseInt(result[1], 16),
        green: parseInt(result[2], 16),
        blue: parseInt(result[3], 16),
        alpha: 255
      };
    };
    
    // Generate light and dark variants
    const generateColorVariants = (baseColor: SimpleSchemaTypes.ColorField) => {
      return {
        light: {
          red: Math.min(255, baseColor.red + 50),
          green: Math.min(255, baseColor.green + 50),
          blue: Math.min(255, baseColor.blue + 50),
          alpha: 255
        },
        dark: {
          red: Math.max(0, baseColor.red - 50),
          green: Math.max(0, baseColor.green - 50),
          blue: Math.max(0, baseColor.blue - 50),
          alpha: 255
        }
      };
    };
    
    const primaryColor = hexToRgba(brand.primaryColor);
    const accentColor = hexToRgba(brand.accentColor);
    const { light: lightColor, dark: darkColor } = generateColorVariants(primaryColor);
    
    // Update all white label settings
    const settings = await client.whiteLabelSettings.update({
      logo: {
        upload_id: brand.logoUploadId,
        alt: null,
        title: null,
        custom_data: {},
        focal_point: null
      },
      favicon: {
        upload_id: brand.faviconUploadId,
        alt: null,
        title: null,
        custom_data: {},
        focal_point: null
      },
      primary_color: primaryColor,
      light_color: lightColor,
      dark_color: darkColor,
      accent_color: accentColor,
      custom_support_email: brand.supportEmail,
      custom_support_url: brand.supportUrl,
      login_page_copy: 'Welcome to our content platform. Please sign in to manage your content.',
      custom_css: `
        /* Brand-specific customizations */
        :root {
          --brand-primary: rgb(${primaryColor.red}, ${primaryColor.green}, ${primaryColor.blue});
          --brand-accent: rgb(${accentColor.red}, ${accentColor.green}, ${accentColor.blue});
        }
        
        .DatoCMS-button--primary {
          background-color: var(--brand-primary);
          border-color: var(--brand-primary);
        }
        
        .DatoCMS-button--primary:hover {
          background-color: var(--brand-accent);
          border-color: var(--brand-accent);
        }
        
        .DatoCMS-navbar__logo {
          max-height: 40px;
        }
        
        /* Custom scrollbar */
        ::-webkit-scrollbar {
          width: 8px;
        }
        
        ::-webkit-scrollbar-track {
          background: #f1f1f1;
        }
        
        ::-webkit-scrollbar-thumb {
          background: var(--brand-primary);
          border-radius: 4px;
        }
      `
    });
    
    console.log('Brand identity successfully configured');
    
    return settings;
  } catch (error) {
    console.error('Error setting up brand identity:', error);
    throw error;
  }
}

// Usage
setupBrandIdentity({
  logoUploadId: '789012',
  faviconUploadId: '789013',
  primaryColor: '#126CED',
  accentColor: '#FF6B00',
  supportEmail: 'help@example.com',
  supportUrl: 'https://help.example.com'
});
```

### Applying Theme Variations

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

type ThemePreset = 'corporate' | 'startup' | 'agency' | 'minimal';

async function applyThemePreset(theme: ThemePreset) {
  try {
    const themes = {
      corporate: {
        primary_color: { red: 0, green: 48, blue: 135, alpha: 255 },
        light_color: { red: 230, green: 236, blue: 245, alpha: 255 },
        dark_color: { red: 0, green: 32, blue: 90, alpha: 255 },
        accent_color: { red: 255, green: 152, blue: 0, alpha: 255 },
        custom_css: `
          /* Corporate theme */
          .DatoCMS-button {
            border-radius: 2px;
            text-transform: uppercase;
            letter-spacing: 0.5px;
          }
          
          .DatoCMS-navbar {
            background: linear-gradient(to right, #003087, #0048d6);
          }
        `,
        login_page_copy: 'Welcome to the corporate content management system.'
      },
      startup: {
        primary_color: { red: 102, green: 126, blue: 234, alpha: 255 },
        light_color: { red: 237, green: 242, blue: 255, alpha: 255 },
        dark_color: { red: 67, green: 83, blue: 154, alpha: 255 },
        accent_color: { red: 255, green: 71, blue: 87, alpha: 255 },
        custom_css: `
          /* Startup theme */
          .DatoCMS-button {
            border-radius: 20px;
            font-weight: 500;
          }
          
          .DatoCMS-navbar {
            backdrop-filter: blur(10px);
            background: rgba(255, 255, 255, 0.9);
          }
          
          .DatoCMS-card {
            border-radius: 12px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.07);
          }
        `,
        login_page_copy: 'Start creating amazing content today!'
      },
      agency: {
        primary_color: { red: 17, green: 17, blue: 17, alpha: 255 },
        light_color: { red: 245, green: 245, blue: 245, alpha: 255 },
        dark_color: { red: 0, green: 0, blue: 0, alpha: 255 },
        accent_color: { red: 255, green: 0, blue: 127, alpha: 255 },
        custom_css: `
          /* Agency theme */
          .DatoCMS-button {
            border-radius: 0;
            border: 2px solid currentColor;
            background: transparent;
            color: #111;
            transition: all 0.3s ease;
          }
          
          .DatoCMS-button:hover {
            background: #111;
            color: white;
          }
          
          .DatoCMS-navbar {
            border-bottom: 1px solid #111;
            background: white;
          }
          
          * {
            font-family: 'Inter', -apple-system, sans-serif;
          }
        `,
        login_page_copy: 'Creative content management for creative teams.'
      },
      minimal: {
        primary_color: { red: 51, green: 51, blue: 51, alpha: 255 },
        light_color: { red: 250, green: 250, blue: 250, alpha: 255 },
        dark_color: { red: 17, green: 17, blue: 17, alpha: 255 },
        accent_color: { red: 0, green: 122, blue: 255, alpha: 255 },
        custom_css: `
          /* Minimal theme */
          .DatoCMS-button {
            border-radius: 4px;
            border: 1px solid #e5e5e5;
            background: white;
            color: #333;
          }
          
          .DatoCMS-button:hover {
            border-color: #333;
          }
          
          .DatoCMS-navbar {
            background: #fafafa;
            border-bottom: 1px solid #e5e5e5;
          }
          
          .DatoCMS-field {
            border: 1px solid #e5e5e5;
            border-radius: 4px;
          }
          
          /* Remove unnecessary shadows */
          * {
            box-shadow: none !important;
          }
        `,
        login_page_copy: 'Simple content management.'
      }
    };
    
    const selectedTheme = themes[theme];
    
    const settings = await client.whiteLabelSettings.update(selectedTheme);
    
    console.log(`Applied ${theme} theme successfully`);
    
    return settings;
  } catch (error) {
    console.error('Error applying theme:', error);
    throw error;
  }
}

// Usage
applyThemePreset('startup');
```

### Managing Custom Domain with SSL

```typescript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function setupCustomDomain(domain: string, enableSSL: boolean = true) {
  try {
    // Update custom domain settings
    const settings = await client.whiteLabelSettings.update({
      custom_domain: domain,
      custom_domain_with_ssl: enableSSL
    });
    
    console.log(`Custom domain ${domain} configured`);
    console.log(`SSL: ${enableSSL ? 'Enabled' : 'Disabled'}`);
    
    // Important: DNS configuration required
    console.log('\nNext steps:');
    console.log('1. Add a CNAME record pointing to: cname.datocms.com');
    console.log('2. Wait for DNS propagation (up to 48 hours)');
    console.log('3. SSL certificate will be automatically generated if enabled');
    
    return settings;
  } catch (error) {
    console.error('Error setting up custom domain:', error);
    throw error;
  }
}

// Usage
setupCustomDomain('cms.example.com', true);
```

## Error Handling

### Common Error Scenarios

```typescript
import { buildClient, ApiError } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

async function updateWithErrorHandling() {
  try {
    const settings = await client.whiteLabelSettings.update({
      primary_color: { red: 256, green: 0, blue: 0, alpha: 255 } // Invalid: red > 255
    });
    
    return settings;
  } catch (error) {
    if (error instanceof ApiError) {
      // Handle specific API errors
      switch (error.statusCode) {
        case 422:
          console.error('Validation error:', error.message);
          // Check for specific field errors
          if (error.findError('primary_color')) {
            console.error('Invalid color values. RGB values must be 0-255');
          }
          break;
        case 401:
          console.error('Authentication failed. Check your API token');
          break;
        case 403:
          console.error('Permission denied. White label settings require appropriate plan');
          break;
        default:
          console.error('API error:', error.statusCode, error.message);
      }
    } else {
      console.error('Unexpected error:', error);
    }
    throw error;
  }
}
```

### Validation Before Update

```typescript
import { buildClient, SimpleSchemaTypes } from '@datocms/cma-client';

const client = buildClient({ 
  apiToken: 'YOUR_API_TOKEN' 
});

interface ValidationResult {
  valid: boolean;
  errors: string[];
}

function validateColorField(color: SimpleSchemaTypes.ColorField): ValidationResult {
  const errors: string[] = [];
  
  if (color.red < 0 || color.red > 255) {
    errors.push('Red value must be between 0 and 255');
  }
  if (color.green < 0 || color.green > 255) {
    errors.push('Green value must be between 0 and 255');
  }
  if (color.blue < 0 || color.blue > 255) {
    errors.push('Blue value must be between 0 and 255');
  }
  if (color.alpha < 0 || color.alpha > 255) {
    errors.push('Alpha value must be between 0 and 255');
  }
  
  return {
    valid: errors.length === 0,
    errors
  };
}

async function safeUpdateColors(colors: {
  primary: SimpleSchemaTypes.ColorField;
  accent: SimpleSchemaTypes.ColorField;
}) {
  try {
    // Validate colors before update
    const primaryValidation = validateColorField(colors.primary);
    const accentValidation = validateColorField(colors.accent);
    
    if (!primaryValidation.valid || !accentValidation.valid) {
      console.error('Color validation failed:');
      [...primaryValidation.errors, ...accentValidation.errors].forEach(
        error => console.error(`- ${error}`)
      );
      return;
    }
    
    // Safe to update
    const settings = await client.whiteLabelSettings.update({
      primary_color: colors.primary,
      accent_color: colors.accent
    });
    
    console.log('Colors updated successfully');
    
    return settings;
  } catch (error) {
    console.error('Error updating colors:', error);
    throw error;
  }
}
```

## Best Practices

### 1. Plan Access Requirements

```typescript
// Check if white label settings are available for your plan
async function checkWhiteLabelAvailability() {
  try {
    const settings = await client.whiteLabelSettings.find();
    console.log('White label settings are available');
    return true;
  } catch (error) {
    if (error instanceof ApiError && error.statusCode === 403) {
      console.log('White label settings require a higher plan');
      return false;
    }
    throw error;
  }
}
```

### 2. Test CSS in Development

```typescript
// Test CSS before applying to production
async function testCustomCSS(css: string) {
  try {
    // Validate CSS syntax
    const cssValidator = new CSSValidator(); // Hypothetical validator
    const isValid = await cssValidator.validate(css);
    
    if (!isValid) {
      console.error('Invalid CSS syntax');
      return false;
    }
    
    // Apply to staging environment first
    const stagingClient = buildClient({ 
      apiToken: 'STAGING_API_TOKEN' 
    });
    
    await stagingClient.whiteLabelSettings.update({
      custom_css: css
    });
    
    console.log('CSS applied to staging. Test before applying to production.');
    return true;
  } catch (error) {
    console.error('Error testing CSS:', error);
    return false;
  }
}
```

### 3. Maintain Brand Consistency

```typescript
// Create a brand configuration object
const brandConfig = {
  colors: {
    primary: '#1a73e8',
    secondary: '#fbbc04',
    success: '#34a853',
    danger: '#ea4335'
  },
  fonts: {
    heading: "'Roboto', sans-serif",
    body: "'Open Sans', sans-serif"
  },
  borderRadius: '4px',
  shadowLevel: 'medium'
};

// Generate consistent CSS from brand config
function generateBrandCSS(config: typeof brandConfig): string {
  return `
    /* Brand Colors */
    :root {
      --brand-primary: ${config.colors.primary};
      --brand-secondary: ${config.colors.secondary};
      --brand-success: ${config.colors.success};
      --brand-danger: ${config.colors.danger};
    }
    
    /* Typography */
    .DatoCMS-heading {
      font-family: ${config.fonts.heading};
    }
    
    body {
      font-family: ${config.fonts.body};
    }
    
    /* Components */
    .DatoCMS-button,
    .DatoCMS-card,
    .DatoCMS-field {
      border-radius: ${config.borderRadius};
    }
    
    /* Shadows */
    ${config.shadowLevel === 'medium' ? `
      .DatoCMS-card {
        box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      }
    ` : ''}
  `;
}
```

### 4. Image Optimization for Logos

```typescript
// Optimize logo before upload
async function optimizeAndUploadLogo(logoPath: string) {
  try {
    // Recommended logo specifications
    const specs = {
      maxWidth: 400,
      maxHeight: 100,
      format: 'png',
      background: 'transparent'
    };
    
    console.log('Logo recommendations:');
    console.log('- Maximum dimensions: 400x100px');
    console.log('- Format: PNG with transparent background');
    console.log('- File size: Under 500KB');
    
    // Upload the logo
    const upload = await client.uploads.create({
      path: createReadStream(logoPath),
      filename: 'logo.png',
      optimize: true,
      default_field_metadata: {
        en: {
          alt: 'Company Logo',
          title: '',
          custom_data: {}
        }
      }
    });
    
    return upload;
  } catch (error) {
    console.error('Error uploading logo:', error);
    throw error;
  }
}
```

## Related Resources

- **Site Settings**: Configure general site settings and locales
- **Uploads**: Manage logo and favicon uploads
- **Plugins**: Extend functionality with custom plugins
- **Webhooks**: Set up notifications for white label changes
- **Environments**: Apply different white label settings per environment

## Common Use Cases

### Agency Managing Multiple Client Projects

```typescript
// Switch between client brandings
async function applyClientBranding(clientId: string) {
  const clientBrands = {
    'client-a': {
      logo_upload_id: '123456',
      primary_color: { red: 0, green: 100, blue: 200, alpha: 255 },
      support_email: 'support@client-a.com'
    },
    'client-b': {
      logo_upload_id: '789012',
      primary_color: { red: 200, green: 50, blue: 50, alpha: 255 },
      support_email: 'help@client-b.com'
    }
  };
  
  const brand = clientBrands[clientId];
  if (!brand) {
    throw new Error('Unknown client');
  }
  
  await client.whiteLabelSettings.update({
    logo: { upload_id: brand.logo_upload_id },
    primary_color: brand.primary_color,
    custom_support_email: brand.support_email
  });
}
```

### SaaS Platform Integration

```typescript
// Sync white label settings with external platform
async function syncWithPlatform(platformSettings: any) {
  const settings = await client.whiteLabelSettings.update({
    logo: { upload_id: platformSettings.logoId },
    primary_color: hexToRgba(platformSettings.brandColor),
    custom_domain: platformSettings.customDomain,
    custom_support_url: platformSettings.supportUrl,
    login_page_copy: platformSettings.welcomeMessage
  });
  
  return settings;
}
```

# MaintenanceMode Resource

## Overview

The MaintenanceMode resource (`client.maintenanceMode`) manages the maintenance mode status for your DatoCMS project. Maintenance mode temporarily restricts access to the admin interface, allowing you to perform critical operations without user interference. This is a singleton resource - each project has exactly one maintenance mode configuration.

**Resource Type**: `maintenance_mode`  
**JSON:API Type**: `maintenance_mode`

## Installation

```bash
npm install @datocms/cma-client
```

## Basic Usage

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Check maintenance mode status
const status = await client.maintenanceMode.find();
console.log('Maintenance mode active:', status.active);

// Activate maintenance mode
await client.maintenanceMode.activate();

// Deactivate maintenance mode
await client.maintenanceMode.deactivate();
```

## API Reference

### find() - Check Maintenance Mode Status

Retrieves the current maintenance mode status.

**Signature**: `find(): Promise<MaintenanceMode>`

**Parameters**: None

**Returns**: MaintenanceMode object with current status

**Example**:
```javascript
const status = await client.maintenanceMode.find();

console.log('Maintenance mode details:', {
  id: status.id,
  active: status.active,
  type: status.type
});

if (status.active) {
  console.log('⚠️ Site is currently in maintenance mode');
} else {
  console.log('✅ Site is operating normally');
}
```

### activate() - Enable Maintenance Mode

Activates maintenance mode for the project, restricting admin access.

**Signature**: `activate(queryParams?: { force?: boolean }): Promise<MaintenanceMode>`

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `queryParams.force` | boolean | Force activation even if collaborators are editing (optional) |

**Returns**: Updated MaintenanceMode object

**Basic Example**:
```javascript
// Standard activation
const result = await client.maintenanceMode.activate();
console.log('Maintenance mode activated:', result.active);

// Check for active editors first
const sessions = await client.editingSessions.list();
if (sessions.length > 0) {
  console.log(`Warning: ${sessions.length} users are currently editing`);
  console.log('Consider using force: true to override');
}
```

**Force Activation Example**:
```javascript
// Force activation despite active collaborators
const result = await client.maintenanceMode.activate({ force: true });
console.log('Maintenance mode forcefully activated');

// This will disconnect active editing sessions
console.log('All active editing sessions have been terminated');
```

### deactivate() - Disable Maintenance Mode

Deactivates maintenance mode, restoring normal admin access.

**Signature**: `deactivate(): Promise<MaintenanceMode>`

**Parameters**: None

**Returns**: Updated MaintenanceMode object

**Example**:
```javascript
const result = await client.maintenanceMode.deactivate();
console.log('Maintenance mode deactivated:', !result.active);
console.log('Admin interface is now accessible to all users');
```

## Object Structure

MaintenanceMode objects have this simple structure:

```typescript
interface MaintenanceMode {
  id: string;                    // Maintenance mode identifier
  type: 'maintenance_mode';      // Always 'maintenance_mode'
  active: boolean;               // Whether maintenance mode is active
}
```

## Common Patterns

### Safe Maintenance Activation

```javascript
async function safeMaintenanceActivation() {
  try {
    // Check for active editing sessions
    const sessions = await client.editingSessions.list();
    
    if (sessions.length > 0) {
      console.log(`⚠️ ${sessions.length} users currently editing:`);
      sessions.forEach(session => {
        console.log(`- ${session.editor.type} ${session.editor.id} editing ${session.active_item.id}`);
      });
      
      const response = prompt('Force activation? (y/N)');
      if (response?.toLowerCase() !== 'y') {
        console.log('Activation cancelled');
        return null;
      }
    }
    
    // Activate maintenance mode
    const result = await client.maintenanceMode.activate({ 
      force: sessions.length > 0 
    });
    
    console.log('✅ Maintenance mode activated successfully');
    return result;
    
  } catch (error) {
    console.error('Failed to activate maintenance mode:', error);
    throw error;
  }
}
```

### Maintenance Window Workflow

```javascript
async function performMaintenanceWindow(maintenanceTask) {
  console.log('🔧 Starting maintenance window...');
  
  // Step 1: Activate maintenance mode
  await client.maintenanceMode.activate({ force: true });
  console.log('✅ Maintenance mode activated');
  
  try {
    // Step 2: Perform maintenance tasks
    console.log('🔄 Performing maintenance tasks...');
    await maintenanceTask();
    console.log('✅ Maintenance tasks completed');
    
  } catch (error) {
    console.error('❌ Maintenance failed:', error);
    throw error;
    
  } finally {
    // Step 3: Always deactivate maintenance mode
    await client.maintenanceMode.deactivate();
    console.log('✅ Maintenance mode deactivated');
    console.log('🎉 Maintenance window completed');
  }
}

// Usage
await performMaintenanceWindow(async () => {
  // Example maintenance tasks
  await client.itemTypes.create({ /* new schema */ });
  await migrateExistingContent();
  await updatePluginConfiguration();
});
```

### Maintenance Status Monitor

```javascript
class MaintenanceMonitor {
  constructor(client, checkInterval = 30000) {
    this.client = client;
    this.checkInterval = checkInterval;
    this.monitoring = false;
    this.listeners = [];
  }

  async start() {
    if (this.monitoring) return;
    
    this.monitoring = true;
    console.log('🔍 Starting maintenance mode monitoring...');
    
    while (this.monitoring) {
      try {
        const status = await this.client.maintenanceMode.find();
        this.notifyListeners(status);
        
        await new Promise(resolve => 
          setTimeout(resolve, this.checkInterval)
        );
      } catch (error) {
        console.error('Monitoring error:', error);
      }
    }
  }

  stop() {
    this.monitoring = false;
    console.log('🛑 Maintenance mode monitoring stopped');
  }

  onStatusChange(callback) {
    this.listeners.push(callback);
  }

  notifyListeners(status) {
    this.listeners.forEach(listener => {
      try {
        listener(status);
      } catch (error) {
        console.error('Listener error:', error);
      }
    });
  }
}

// Usage
const monitor = new MaintenanceMonitor(client);

monitor.onStatusChange((status) => {
  if (status.active) {
    console.log('🚨 Maintenance mode is ACTIVE');
    // Notify users, pause operations, etc.
  } else {
    console.log('✅ Maintenance mode is INACTIVE');
    // Resume normal operations
  }
});

await monitor.start();
```

### Scheduled Maintenance

```javascript
async function scheduleMaintenanceWindow(scheduledTime, maintenanceTask) {
  const now = new Date();
  const delay = scheduledTime.getTime() - now.getTime();
  
  if (delay <= 0) {
    throw new Error('Scheduled time must be in the future');
  }
  
  console.log(`🕒 Maintenance scheduled for ${scheduledTime.toISOString()}`);
  console.log(`⏱️ Time until maintenance: ${Math.round(delay / 1000 / 60)} minutes`);
  
  // Warning notifications
  const warningTimes = [
    { minutes: 15, message: '⚠️ Maintenance in 15 minutes' },
    { minutes: 5, message: '⚠️ Maintenance in 5 minutes' },
    { minutes: 1, message: '🚨 Maintenance starting in 1 minute!' }
  ];
  
  // Schedule warnings
  warningTimes.forEach(warning => {
    const warningTime = delay - (warning.minutes * 60 * 1000);
    if (warningTime > 0) {
      setTimeout(() => {
        console.log(warning.message);
        // Send notifications to users
      }, warningTime);
    }
  });
  
  // Schedule maintenance
  setTimeout(async () => {
    await performMaintenanceWindow(maintenanceTask);
  }, delay);
  
  return {
    scheduledTime,
    delay,
    cancel: () => clearTimeout() // Implementation would track timeout ID
  };
}

// Usage
const scheduledMaintenance = await scheduleMaintenanceWindow(
  new Date(Date.now() + 30 * 60 * 1000), // 30 minutes from now
  async () => {
    console.log('Performing scheduled database migration...');
    await performDatabaseMigration();
  }
);
```

## Error Handling

```javascript
import { ApiError } from '@datocms/rest-client-utils';

try {
  await client.maintenanceMode.activate();
} catch (error) {
  if (error instanceof ApiError) {
    switch (error.statusCode) {
      case 403:
        console.error('Insufficient permissions to control maintenance mode');
        break;
      case 409:
        console.error('Maintenance mode state conflict - may already be active');
        break;
      case 422:
        console.error('Invalid maintenance mode operation');
        break;
      default:
        console.error('Unexpected API error:', error.message);
    }
  } else {
    console.error('Network or other error:', error);
  }
}
```

### Common Maintenance Mode Errors

| Status | Error | Cause | Solution |
|--------|-------|-------|----------|
| 403 | FORBIDDEN | Insufficient permissions | Ensure API token has admin permissions |
| 409 | CONFLICT | State conflict | Check current status before changing |
| 422 | VALIDATION_ERROR | Invalid operation | Verify request parameters |

### Best Practices for Error Handling

```javascript
async function robustMaintenanceControl(action, options = {}) {
  const maxRetries = 3;
  const retryDelay = 1000;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      switch (action) {
        case 'activate':
          return await client.maintenanceMode.activate(options);
        case 'deactivate':
          return await client.maintenanceMode.deactivate();
        case 'status':
          return await client.maintenanceMode.find();
        default:
          throw new Error(`Unknown action: ${action}`);
      }
    } catch (error) {
      console.error(`Attempt ${attempt} failed:`, error.message);
      
      if (attempt === maxRetries) {
        throw error;
      }
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, retryDelay));
    }
  }
}

// Usage with retry logic
try {
  const result = await robustMaintenanceControl('activate', { force: true });
  console.log('Maintenance mode activated successfully');
} catch (error) {
  console.error('Failed to activate maintenance mode after retries:', error);
}
```

## Integration with Other Resources

### Maintenance Mode + Webhooks

```javascript
// Monitor maintenance mode changes via webhooks
async function setupMaintenanceWebhook() {
  const webhook = await client.webhooks.create({
    name: 'Maintenance Mode Monitor',
    url: 'https://your-app.com/webhooks/maintenance',
    events: [
      'maintenance_mode:activate',
      'maintenance_mode:deactivate'
    ],
    headers: {
      'X-API-Key': 'your-webhook-secret'
    }
  });
  
  console.log('Maintenance mode webhook created:', webhook.id);
  return webhook;
}
```

### Maintenance Mode + Build Triggers

```javascript
// Pause build triggers during maintenance
async function pauseBuildsForMaintenance() {
  // Get current maintenance status
  const maintenance = await client.maintenanceMode.find();
  
  if (maintenance.active) {
    // Disable all build triggers
    const triggers = await client.buildTriggers.list();
    
    for (const trigger of triggers) {
      await client.buildTriggers.update(trigger.id, {
        enabled: false
      });
    }
    
    console.log(`Disabled ${triggers.length} build triggers for maintenance`);
  }
}
```

## Use Cases

### 1. Schema Migration Maintenance

```javascript
async function performSchemaMigration(migrationSteps) {
  await client.maintenanceMode.activate({ force: true });
  
  try {
    for (const [index, step] of migrationSteps.entries()) {
      console.log(`Step ${index + 1}/${migrationSteps.length}: ${step.description}`);
      await step.execute();
    }
  } finally {
    await client.maintenanceMode.deactivate();
  }
}
```

### 2. Bulk Content Operations

```javascript
async function bulkContentUpdate(items, updateFunction) {
  console.log(`Updating ${items.length} items - activating maintenance mode`);
  
  await client.maintenanceMode.activate();
  
  try {
    const results = [];
    for (const item of items) {
      const updated = await updateFunction(item);
      results.push(updated);
    }
    return results;
  } finally {
    await client.maintenanceMode.deactivate();
  }
}
```

### 3. Plugin Installation/Updates

```javascript
async function installPlugin(pluginConfig) {
  await client.maintenanceMode.activate({ force: true });
  
  try {
    const plugin = await client.plugins.create(pluginConfig);
    
    // Wait for plugin initialization
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    // Verify plugin is working
    const installedPlugin = await client.plugins.find(plugin.id);
    if (installedPlugin.status !== 'active') {
      throw new Error('Plugin installation failed');
    }
    
    return plugin;
  } finally {
    await client.maintenanceMode.deactivate();
  }
}
```

# MenuItem Resource

## Overview

The MenuItem resource (`client.menuItems`) manages navigation menu items in your DatoCMS project. Menu items can link to content types, external URLs, or other menu items in a hierarchical structure.

**Resource Type**: `menu_item`  
**JSON:API Type**: `menu_item`

## Basic Usage

```javascript
import { buildClient } from '@datocms/cma-client';

const client = buildClient({ apiToken: 'YOUR_API_TOKEN' });

// Create a menu item
const menuItem = await client.menuItems.create({
  label: 'Articles',
  item_type: { type: 'item_type', id: 'article' },
  position: 0
});

// List all menu items
const items = await client.menuItems.list();

// Update menu item
await client.menuItems.update(menuItem.id, {
  label: 'Blog Posts',
  open_in_new_tab: true
});
```

## API Reference

### create() - Create Menu Item

**Signature**: `create(data: MenuItemCreateSchema): Promise<MenuItem>`

```javascript
// Link to content type
const contentLink = await client.menuItems.create({
  label: 'Blog',
  item_type: { type: 'item_type', id: 'article' },
  position: 0
});

// External URL
const externalLink = await client.menuItems.create({
  label: 'Company Website',
  external_url: 'https://company.com',
  open_in_new_tab: true,
  position: 1
});

// Submenu item
const submenu = await client.menuItems.create({
  label: 'Tech Articles',
  item_type: { type: 'item_type', id: 'article' },
  item_type_filter: { type: 'item_type_filter', id: 'tech-filter' },
  parent: { type: 'menu_item', id: contentLink.id },
  position: 0
});
```

### list() - List Menu Items

**Signature**: `list(): Promise<MenuItem[]>`

```javascript
const menuItems = await client.menuItems.list();

// Build navigation structure
function buildMenuTree(items) {
  const rootItems = items.filter(item => !item.parent);
  
  function attachChildren(item) {
    const children = items.filter(child => child.parent?.id === item.id);
    return {
      ...item,
      children: children.map(attachChildren)
    };
  }
  
  return rootItems.map(attachChildren);
}

const menuTree = buildMenuTree(menuItems);
```

### update() - Update Menu Item

**Signature**: `update(id: string, data: MenuItemUpdateSchema): Promise<MenuItem>`

```javascript
// Update label and URL
await client.menuItems.update('123', {
  label: 'Updated Label',
  external_url: 'https://new-url.com'
});

// Move to different parent
await client.menuItems.update('123', {
  parent: { type: 'menu_item', id: 'parent-id' },
  position: 2
});
```

## Object Structure

```typescript
interface MenuItem {
  id: string;
  type: 'menu_item';
  label: string;                     // Display text
  external_url: string | null;       // External link URL
  position: number;                  // Sort order within parent
  open_in_new_tab: boolean;         // Open in new tab
  item_type: ItemType | null;       // Linked content type
  item_type_filter: ItemTypeFilter | null; // Content filter
  parent: MenuItem | null;          // Parent menu item
  children: MenuItem[];             // Child menu items
}
```

# PublicInfo Resource

## Overview

The PublicInfo resource (`client.publicInfo`) provides read-only access to public site information including branding, theme colors, and SSO settings. This is a singleton resource.

**Resource Type**: `public_info`  
**JSON:API Type**: `public_info`

## Basic Usage

```javascript
// Get public site information
const publicInfo = await client.publicInfo.find();

console.log('Site name:', publicInfo.name);
console.log('Logo URL:', publicInfo.logo_url);
console.log('White-label enabled:', publicInfo.white_label);
console.log('Primary color:', publicInfo.theme.primary_color);
```

## API Reference

### find() - Get Public Information

**Signature**: `find(): Promise<PublicInfo>`

```javascript
const info = await client.publicInfo.find();

// Extract theme colors
const { primary_color, light_color, accent_color, dark_color } = info.theme;

// Check SSO configuration
if (info.sso_saml_init_url) {
  console.log('SSO enabled with URL:', info.sso_saml_init_url);
}

// Use in frontend
const brandingData = {
  siteName: info.name,
  logoUrl: info.logo_url,
  isWhiteLabel: info.white_label,
  colors: {
    primary: `rgba(${primary_color.red}, ${primary_color.green}, ${primary_color.blue}, ${primary_color.alpha / 255})`,
    accent: `rgba(${accent_color.red}, ${accent_color.green}, ${accent_color.blue}, ${accent_color.alpha / 255})`
  }
};
```

## Object Structure

```typescript
interface PublicInfo {
  id: string;
  type: 'public_info';
  name: string;                      // Site name
  sso_saml_init_url: string | null;  // SSO login URL
  logo_url: string | null;           // Site logo URL
  white_label: boolean;              // White-label mode
  custom_i18n_messages_template_url: string | null; // Custom i18n URL
  theme: {
    primary_color: { red: number; green: number; blue: number; alpha: number };
    light_color: { red: number; green: number; blue: number; alpha: number };
    accent_color: { red: number; green: number; blue: number; alpha: number };
    dark_color: { red: number; green: number; blue: number; alpha: number };
  };
}
```

# WhiteLabelSettings Resource

## Overview

The WhiteLabelSettings resource (`client.whiteLabelSettings`) manages white-label customization settings, particularly custom internationalization message templates. This is a singleton resource.

**Resource Type**: `white_label_settings`  
**JSON:API Type**: `white_label_settings`

## Basic Usage

```javascript
// Get current white-label settings
const settings = await client.whiteLabelSettings.find();

// Update custom i18n messages URL
await client.whiteLabelSettings.update({
  custom_i18n_messages_template_url: 'https://your-domain.com/i18n-template.json'
});
```

## API Reference

### find() - Get White-Label Settings

**Signature**: `find(): Promise<WhiteLabelSettings>`

```javascript
const settings = await client.whiteLabelSettings.find();

if (settings.custom_i18n_messages_template_url) {
  console.log('Custom i18n template URL:', settings.custom_i18n_messages_template_url);
}
```

### update() - Update White-Label Settings

**Signature**: `update(data: WhiteLabelSettingsUpdateSchema): Promise<WhiteLabelSettings>`

```javascript
// Set custom i18n messages template
await client.whiteLabelSettings.update({
  custom_i18n_messages_template_url: 'https://cdn.example.com/datocms-i18n.json'
});

// Remove custom template (revert to default)
await client.whiteLabelSettings.update({
  custom_i18n_messages_template_url: null
});
```

## Object Structure

```typescript
interface WhiteLabelSettings {
  id: string;
  type: 'white_label_settings';
  custom_i18n_messages_template_url: string | null; // Custom i18n template URL
}
```

## Integration Examples

### Menu Navigation Builder

```javascript
async function buildSiteNavigation() {
  const [menuItems, publicInfo] = await Promise.all([
    client.menuItems.list(),
    client.publicInfo.find()
  ]);

  return {
    siteName: publicInfo.name,
    logoUrl: publicInfo.logo_url,
    navigation: buildMenuTree(menuItems),
    theme: publicInfo.theme
  };
}
```

### White-Label Configuration

```javascript
async function setupWhiteLabel(customDomain, i18nTemplateUrl) {
  // Update white-label settings
  await client.whiteLabelSettings.update({
    custom_i18n_messages_template_url: i18nTemplateUrl
  });

  // Get updated public info
  const publicInfo = await client.publicInfo.find();
  
  return {
    whiteLabel: publicInfo.white_label,
    customI18n: i18nTemplateUrl,
    branding: publicInfo.theme
  };
}
```

## Related Resources

- **[Environment Methods](/LLM_DOCS/01-cma-client/04-site-configuration/environment-methods.md)**: Manage multiple environments
- **[Menu Item Methods](/LLM_DOCS/01-cma-client/04-site-configuration/menu-item-methods.md)**: Customize navigation
- **[White Label Methods](/LLM_DOCS/01-cma-client/04-site-configuration/white-label-methods.md)**: Brand customization