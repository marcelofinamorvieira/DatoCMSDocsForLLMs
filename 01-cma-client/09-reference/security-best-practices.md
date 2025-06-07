# Security Best Practices for DatoCMS Development

This guide covers essential security practices for developing with DatoCMS APIs and plugins. Following these guidelines helps protect your applications, user data, and API credentials.

## API Token Security

### Token Types and Scopes

DatoCMS uses different token types with specific scopes:

```typescript
// Read-only token - safe for client-side
const publicClient = buildClient({
  apiToken: process.env.NEXT_PUBLIC_DATOCMS_READONLY_TOKEN,
  environment: 'main'
});

// Full-access token - server-side only
const privateClient = buildClient({
  apiToken: process.env.DATOCMS_READWRITE_TOKEN,
  environment: 'main'
});

// Account token - administrative operations
const accountClient = buildClient({
  apiToken: process.env.DATOCMS_ACCOUNT_TOKEN
});
```

### Environment Variables

Never hardcode API tokens:

```typescript
// ❌ Bad - Exposed token
const client = buildClient({
  apiToken: 'da123456789abcdef'
});

// ✅ Good - Environment variable
const client = buildClient({
  apiToken: process.env.DATOCMS_API_TOKEN
});
```

### Token Rotation

Implement token rotation for enhanced security:

```typescript
async function rotateApiToken(projectId: string) {
  const accountClient = buildClient({
    apiToken: process.env.DATOCMS_ACCOUNT_TOKEN
  });
  
  // Create new token
  const newToken = await accountClient.accessTokens.create({
    name: `API Token - ${new Date().toISOString()}`,
    can_access_cma: true,
    can_access_cda: true
  });
  
  // Update application configuration
  await updateEnvironmentVariable('DATOCMS_API_TOKEN', newToken.token);
  
  // Delete old token after verification
  await accountClient.accessTokens.destroy(oldTokenId);
}
```

## Client-Side Security

### Use Read-Only Tokens

Always use read-only tokens in client-side code:

```jsx
// pages/index.js (Next.js example)
export async function getStaticProps() {
  const client = buildClient({
    apiToken: process.env.DATOCMS_READONLY_TOKEN // Server-side only
  });
  
  const data = await client.items.list();
  
  return {
    props: { data }
  };
}
```

### GraphQL Security

Protect GraphQL endpoints:

```typescript
// middleware/graphql-security.ts
export function secureGraphQLMiddleware(req: Request, res: Response, next: NextFunction) {
  // Validate origin
  const allowedOrigins = ['https://yourdomain.com'];
  const origin = req.headers.origin;
  
  if (!allowedOrigins.includes(origin)) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  // Rate limiting
  const ip = req.ip;
  if (isRateLimited(ip)) {
    return res.status(429).json({ error: 'Too many requests' });
  }
  
  next();
}
```

### Content Security Policy

Implement CSP headers for plugin development:

```typescript
// plugin-config.ts
export const securityHeaders = {
  'Content-Security-Policy': [
    "default-src 'self'",
    "script-src 'self' 'unsafe-inline' https://unpkg.com",
    "style-src 'self' 'unsafe-inline'",
    "img-src 'self' data: https://www.datocms-assets.com",
    "connect-src 'self' https://site-api.datocms.com"
  ].join('; ')
};
```

## Plugin Security

### Validate User Permissions

Always check user permissions in plugins:

```typescript
connect({
  async onBoot(ctx) {
    // Check if user has required permissions
    const currentUser = await ctx.currentUser;
    const role = await ctx.role;
    
    if (!role.attributes.can_edit_schema) {
      ctx.alert('You need schema editing permissions to use this plugin');
      return;
    }
  },
  
  async executeItemFormDropdownAction(actionId, ctx) {
    // Verify user can perform action
    if (!ctx.currentUserAccessToken) {
      throw new Error('Authentication required');
    }
    
    // Additional permission checks
    const canEdit = ctx.itemFormPermissions.canEdit;
    if (!canEdit) {
      ctx.alert('You do not have permission to edit this item');
      return;
    }
  }
});
```

### Secure Plugin Configuration

Encrypt sensitive plugin parameters:

```typescript
import crypto from 'crypto';

const algorithm = 'aes-256-gcm';

export function encryptPluginSecret(text: string, password: string) {
  const salt = crypto.randomBytes(32);
  const key = crypto.scryptSync(password, salt, 32);
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag();
  
  return {
    encrypted,
    salt: salt.toString('hex'),
    iv: iv.toString('hex'),
    authTag: authTag.toString('hex')
  };
}

export function decryptPluginSecret(encryptedData: any, password: string) {
  const key = crypto.scryptSync(password, Buffer.from(encryptedData.salt, 'hex'), 32);
  const decipher = crypto.createDecipheriv(
    algorithm, 
    key, 
    Buffer.from(encryptedData.iv, 'hex')
  );
  
  decipher.setAuthTag(Buffer.from(encryptedData.authTag, 'hex'));
  
  let decrypted = decipher.update(encryptedData.encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

### Input Validation

Validate all user inputs in plugins:

```typescript
import { z } from 'zod';

const ItemSchema = z.object({
  title: z.string().min(1).max(200),
  slug: z.string().regex(/^[a-z0-9-]+$/),
  content: z.string(),
  published_at: z.string().datetime().optional()
});

connect({
  async onBeforeItemUpsert(createOrUpdateItemPayload, ctx) {
    try {
      // Validate payload
      const validated = ItemSchema.parse(createOrUpdateItemPayload);
      
      // Sanitize HTML content if needed
      if (validated.content) {
        validated.content = sanitizeHtml(validated.content, {
          allowedTags: ['p', 'a', 'strong', 'em'],
          allowedAttributes: {
            'a': ['href']
          }
        });
      }
      
      return validated;
    } catch (error) {
      ctx.alert('Invalid input data');
      throw error;
    }
  }
});
```

## Data Protection

### Sensitive Data Handling

Never log sensitive information:

```typescript
// ❌ Bad - Logs sensitive data
console.log('User data:', { email, password, apiToken });

// ✅ Good - Sanitized logging
console.log('User action:', { 
  userId: user.id, 
  action: 'login',
  timestamp: new Date() 
});
```

### Implement Audit Logging

Track sensitive operations:

```typescript
async function auditLog(action: string, details: any, ctx: any) {
  const auditEntry = {
    action,
    user_id: ctx.currentUser?.id,
    timestamp: new Date().toISOString(),
    ip_address: ctx.request?.ip,
    details: {
      ...details,
      // Remove sensitive fields
      api_token: undefined,
      password: undefined
    }
  };
  
  await ctx.client.items.create({
    item_type: { type: 'item_type', id: 'audit_log' },
    ...auditEntry
  });
}
```

### Data Encryption

Encrypt sensitive field data:

```typescript
connect({
  async renderFieldExtension(fieldExtensionId, ctx) {
    if (fieldExtensionId === 'encrypted-field') {
      return {
        async onChange(newValue) {
          // Encrypt before saving
          const encrypted = await encryptFieldValue(newValue);
          ctx.setFieldValue(ctx.fieldPath, encrypted);
        },
        
        async getValue() {
          const encrypted = ctx.formValues[ctx.fieldPath];
          // Decrypt for display
          return await decryptFieldValue(encrypted);
        }
      };
    }
  }
});
```

## Network Security

### HTTPS Enforcement

Always use HTTPS for API calls:

```typescript
const client = buildClient({
  apiToken: process.env.DATOCMS_API_TOKEN,
  baseUrl: 'https://site-api.datocms.com' // Never use http://
});
```

### Request Validation

Validate webhook signatures:

```typescript
import crypto from 'crypto';

export function validateWebhookSignature(
  payload: string,
  signature: string,
  secret: string
): boolean {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(payload);
  const expectedSignature = hmac.digest('hex');
  
  // Use timing-safe comparison
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// Express middleware
app.post('/webhook', (req, res) => {
  const signature = req.headers['x-datocms-signature'];
  const isValid = validateWebhookSignature(
    JSON.stringify(req.body),
    signature,
    process.env.WEBHOOK_SECRET
  );
  
  if (!isValid) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process webhook
});
```

### Rate Limiting

Implement rate limiting for API operations:

```typescript
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});

// Apply to API routes
app.use('/api/', apiLimiter);

// Stricter limits for sensitive operations
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true
});

app.use('/api/auth/', authLimiter);
```

## Error Handling

### Safe Error Messages

Never expose sensitive information in errors:

```typescript
// ❌ Bad - Exposes internal details
catch (error) {
  res.status(500).json({
    error: error.message,
    stack: error.stack,
    query: originalQuery
  });
}

// ✅ Good - Safe error response
catch (error) {
  console.error('Internal error:', error); // Log full error server-side
  
  res.status(500).json({
    error: 'An error occurred processing your request',
    requestId: generateRequestId()
  });
}
```

### Error Monitoring

Implement secure error tracking:

```typescript
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  beforeSend(event, hint) {
    // Remove sensitive data
    if (event.request) {
      delete event.request.cookies;
      delete event.request.headers?.authorization;
    }
    
    // Sanitize error messages
    if (event.exception) {
      event.exception.values?.forEach(exception => {
        exception.value = sanitizeErrorMessage(exception.value);
      });
    }
    
    return event;
  }
});
```

## Security Checklist

### Development Phase
- [ ] Use environment variables for all credentials
- [ ] Implement input validation for all user inputs
- [ ] Add CSRF protection for forms
- [ ] Enable CORS with specific origins
- [ ] Use HTTPS for all communications
- [ ] Implement rate limiting
- [ ] Add request signing for webhooks

### Code Review
- [ ] No hardcoded credentials
- [ ] No sensitive data in logs
- [ ] Error messages don't expose internals
- [ ] All inputs are validated
- [ ] Permissions are checked before operations
- [ ] Dependencies are up to date

### Deployment
- [ ] Production tokens are different from development
- [ ] Implement token rotation schedule
- [ ] Enable audit logging
- [ ] Configure security headers
- [ ] Set up monitoring and alerting
- [ ] Document security procedures

### Plugin Specific
- [ ] Validate plugin configuration
- [ ] Check user permissions
- [ ] Sanitize user inputs
- [ ] Encrypt sensitive parameters
- [ ] Implement secure communication
- [ ] Handle errors gracefully

## Security Resources

### DatoCMS Security Features
- [API Token Management](https://www.datocms.com/docs/security/api-tokens)
- [Role-Based Access Control](https://www.datocms.com/docs/security/roles)
- [SSO Configuration](https://www.datocms.com/docs/security/sso)
- [Audit Logs](https://www.datocms.com/docs/security/audit-logs)

### General Security Resources
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Node.js Security Checklist](https://blog.risingstack.com/node-js-security-checklist/)
- [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

## Summary

Security is a continuous process. Key principles:
1. **Principle of Least Privilege**: Grant minimum necessary permissions
2. **Defense in Depth**: Layer security measures
3. **Fail Securely**: Handle errors without exposing vulnerabilities
4. **Keep It Simple**: Complex security often creates vulnerabilities
5. **Stay Updated**: Regularly update dependencies and review practices