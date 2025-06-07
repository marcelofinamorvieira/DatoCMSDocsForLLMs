# Error Handling Patterns

This guide covers comprehensive error handling patterns for DatoCMS APIs, including error types, recovery strategies, and production best practices.

## Error Types and Status Codes Reference

### ApiError Class Structure

The `ApiError` class is the primary error type thrown by DatoCMS clients:

```typescript
import { ApiError } from '@datocms/cma-client';

// ApiError structure
interface ApiError extends Error {
  name: 'ApiError';
  statusCode: number;
  statusText: string;
  message: string;
  headers: Record<string, string>;
  body: {
    data?: Array<{
      id: string;
      type: 'api_error';
      attributes: {
        code: string;
        details: Record<string, any>;
        field?: string;
        message: string;
      };
    }>;
  };
}

// Type guard for ApiError
function isApiError(error: unknown): error is ApiError {
  return error instanceof Error && error.name === 'ApiError';
}

// Complete error handling example
async function handleApiCall<T>(
  operation: () => Promise<T>
): Promise<{ data?: T; error?: ApiError }> {
  try {
    const data = await operation();
    return { data };
  } catch (error) {
    if (isApiError(error)) {
      return { error };
    }
    // Re-throw non-API errors
    throw error;
  }
}
```

### HTTP Status Codes Reference

```typescript
// Comprehensive status code handling
const StatusCodes = {
  // Success
  OK: 200,
  CREATED: 201,
  NO_CONTENT: 204,
  
  // Client Errors
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  METHOD_NOT_ALLOWED: 405,
  NOT_ACCEPTABLE: 406,
  CONFLICT: 409,
  GONE: 410,
  UNPROCESSABLE_ENTITY: 422,
  TOO_MANY_REQUESTS: 429,
  
  // Server Errors
  INTERNAL_SERVER_ERROR: 500,
  BAD_GATEWAY: 502,
  SERVICE_UNAVAILABLE: 503,
  GATEWAY_TIMEOUT: 504
} as const;

type StatusCode = typeof StatusCodes[keyof typeof StatusCodes];

// Status code categorization
function getErrorCategory(statusCode: number): 'client' | 'server' | 'unknown' {
  if (statusCode >= 400 && statusCode < 500) return 'client';
  if (statusCode >= 500 && statusCode < 600) return 'server';
  return 'unknown';
}

// Detailed error mapping
const ErrorMessages: Record<StatusCode, string> = {
  [StatusCodes.BAD_REQUEST]: 'Invalid request format',
  [StatusCodes.UNAUTHORIZED]: 'Authentication required - check API token',
  [StatusCodes.FORBIDDEN]: 'Access denied - insufficient permissions',
  [StatusCodes.NOT_FOUND]: 'Resource not found',
  [StatusCodes.METHOD_NOT_ALLOWED]: 'HTTP method not allowed',
  [StatusCodes.NOT_ACCEPTABLE]: 'Requested format not available',
  [StatusCodes.CONFLICT]: 'Resource conflict - duplicate or concurrent modification',
  [StatusCodes.GONE]: 'Resource permanently deleted',
  [StatusCodes.UNPROCESSABLE_ENTITY]: 'Validation failed',
  [StatusCodes.TOO_MANY_REQUESTS]: 'Rate limit exceeded',
  [StatusCodes.INTERNAL_SERVER_ERROR]: 'Server error - please retry',
  [StatusCodes.BAD_GATEWAY]: 'Gateway error - upstream service issue',
  [StatusCodes.SERVICE_UNAVAILABLE]: 'Service temporarily unavailable',
  [StatusCodes.GATEWAY_TIMEOUT]: 'Gateway timeout - request took too long'
};
```

### Error Type Hierarchy

```typescript
// Custom error types extending ApiError information
class DatoCMSError extends Error {
  constructor(
    message: string,
    public code: string,
    public context?: Record<string, any>
  ) {
    super(message);
    this.name = 'DatoCMSError';
  }
}

class ValidationError extends DatoCMSError {
  constructor(
    public fields: Array<{
      field: string;
      code: string;
      message: string;
      details?: any;
    }>
  ) {
    super('Validation failed', 'VALIDATION_ERROR', { fields });
    this.name = 'ValidationError';
  }
}

class RateLimitError extends DatoCMSError {
  constructor(
    public retryAfter: number,
    public limit: number,
    public remaining: number,
    public reset: Date
  ) {
    super(
      `Rate limit exceeded. Retry after ${retryAfter} seconds`,
      'RATE_LIMIT_ERROR',
      { retryAfter, limit, remaining, reset }
    );
    this.name = 'RateLimitError';
  }
}

class NetworkError extends DatoCMSError {
  constructor(
    message: string,
    public originalError?: Error
  ) {
    super(message, 'NETWORK_ERROR', { originalError });
    this.name = 'NetworkError';
  }
}

// Error factory
function createTypedError(error: ApiError): DatoCMSError {
  switch (error.statusCode) {
    case StatusCodes.UNPROCESSABLE_ENTITY:
      const fields = error.body.data?.map(err => ({
        field: err.attributes.field || '',
        code: err.attributes.code,
        message: err.attributes.message,
        details: err.attributes.details
      })) || [];
      return new ValidationError(fields);
      
    case StatusCodes.TOO_MANY_REQUESTS:
      const headers = error.headers;
      return new RateLimitError(
        parseInt(headers['retry-after'] || '60'),
        parseInt(headers['x-ratelimit-limit'] || '0'),
        parseInt(headers['x-ratelimit-remaining'] || '0'),
        new Date(parseInt(headers['x-ratelimit-reset'] || '0') * 1000)
      );
      
    default:
      return new DatoCMSError(
        error.message,
        `HTTP_${error.statusCode}`,
        { statusCode: error.statusCode, body: error.body }
      );
  }
}
```

## Retry Strategies

### Exponential Backoff Implementation

```typescript
interface RetryOptions {
  maxRetries: number;
  initialDelayMs: number;
  maxDelayMs: number;
  backoffFactor: number;
  retryableStatuses: number[];
  onRetry?: (attempt: number, delay: number, error: Error) => void;
}

class ExponentialBackoffRetry {
  private defaultOptions: RetryOptions = {
    maxRetries: 3,
    initialDelayMs: 1000,
    maxDelayMs: 30000,
    backoffFactor: 2,
    retryableStatuses: [408, 429, 500, 502, 503, 504],
    onRetry: (attempt, delay) => {
      console.log(`Retry attempt ${attempt} after ${delay}ms`);
    }
  };
  
  async execute<T>(
    operation: () => Promise<T>,
    options?: Partial<RetryOptions>
  ): Promise<T> {
    const config = { ...this.defaultOptions, ...options };
    let lastError: Error | null = null;
    
    for (let attempt = 0; attempt <= config.maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error as Error;
        
        // Check if error is retryable
        if (!this.isRetryable(error, config)) {
          throw error;
        }
        
        // Check if we have retries left
        if (attempt === config.maxRetries) {
          throw new DatoCMSError(
            `Operation failed after ${config.maxRetries} retries`,
            'MAX_RETRIES_EXCEEDED',
            { originalError: error }
          );
        }
        
        // Calculate delay with jitter
        const delay = this.calculateDelay(attempt, config);
        config.onRetry?.(attempt + 1, delay, error as Error);
        
        await this.sleep(delay);
      }
    }
    
    throw lastError;
  }
  
  private isRetryable(error: unknown, config: RetryOptions): boolean {
    if (!isApiError(error)) return false;
    return config.retryableStatuses.includes(error.statusCode);
  }
  
  private calculateDelay(attempt: number, config: RetryOptions): number {
    // Exponential backoff with jitter
    const exponentialDelay = config.initialDelayMs * Math.pow(config.backoffFactor, attempt);
    const delayWithJitter = exponentialDelay * (0.5 + Math.random() * 0.5);
    return Math.min(delayWithJitter, config.maxDelayMs);
  }
  
  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage
const retry = new ExponentialBackoffRetry();

const item = await retry.execute(
  () => client.items.find('123'),
  {
    maxRetries: 5,
    onRetry: (attempt, delay, error) => {
      console.log(`Attempt ${attempt} failed:`, error.message);
      console.log(`Retrying in ${delay}ms...`);
    }
  }
);
```

### Circuit Breaker Pattern

```typescript
enum CircuitState {
  CLOSED = 'CLOSED',
  OPEN = 'OPEN',
  HALF_OPEN = 'HALF_OPEN'
}

interface CircuitBreakerOptions {
  failureThreshold: number;
  resetTimeoutMs: number;
  monitoringPeriodMs: number;
  halfOpenMaxAttempts: number;
}

class CircuitBreaker {
  private state: CircuitState = CircuitState.CLOSED;
  private failures: number = 0;
  private lastFailureTime: number = 0;
  private successfulHalfOpenCalls: number = 0;
  private monitoringStartTime: number = Date.now();
  
  constructor(private options: CircuitBreakerOptions) {}
  
  async execute<T>(operation: () => Promise<T>): Promise<T> {
    // Check if circuit should be reset
    this.checkCircuitReset();
    
    if (this.state === CircuitState.OPEN) {
      throw new DatoCMSError(
        'Circuit breaker is OPEN - service unavailable',
        'CIRCUIT_BREAKER_OPEN'
      );
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private checkCircuitReset(): void {
    if (this.state === CircuitState.OPEN) {
      const timeSinceLastFailure = Date.now() - this.lastFailureTime;
      if (timeSinceLastFailure >= this.options.resetTimeoutMs) {
        this.state = CircuitState.HALF_OPEN;
        this.successfulHalfOpenCalls = 0;
        console.log('Circuit breaker transitioned to HALF_OPEN');
      }
    }
    
    // Reset monitoring period
    const monitoringPeriodElapsed = Date.now() - this.monitoringStartTime;
    if (monitoringPeriodElapsed >= this.options.monitoringPeriodMs) {
      this.failures = 0;
      this.monitoringStartTime = Date.now();
    }
  }
  
  private onSuccess(): void {
    if (this.state === CircuitState.HALF_OPEN) {
      this.successfulHalfOpenCalls++;
      if (this.successfulHalfOpenCalls >= this.options.halfOpenMaxAttempts) {
        this.state = CircuitState.CLOSED;
        this.failures = 0;
        console.log('Circuit breaker transitioned to CLOSED');
      }
    }
  }
  
  private onFailure(): void {
    this.failures++;
    this.lastFailureTime = Date.now();
    
    if (this.failures >= this.options.failureThreshold) {
      this.state = CircuitState.OPEN;
      console.log('Circuit breaker transitioned to OPEN');
    }
    
    if (this.state === CircuitState.HALF_OPEN) {
      this.state = CircuitState.OPEN;
      console.log('Circuit breaker transitioned back to OPEN');
    }
  }
  
  getState(): { state: CircuitState; failures: number } {
    return {
      state: this.state,
      failures: this.failures
    };
  }
}

// Combined retry with circuit breaker
class ResilientClient {
  private circuitBreaker: CircuitBreaker;
  private retry: ExponentialBackoffRetry;
  
  constructor() {
    this.circuitBreaker = new CircuitBreaker({
      failureThreshold: 5,
      resetTimeoutMs: 60000, // 1 minute
      monitoringPeriodMs: 120000, // 2 minutes
      halfOpenMaxAttempts: 3
    });
    
    this.retry = new ExponentialBackoffRetry();
  }
  
  async execute<T>(operation: () => Promise<T>): Promise<T> {
    return this.circuitBreaker.execute(
      () => this.retry.execute(operation)
    );
  }
}
```

## Error Recovery Patterns

### Graceful Degradation with Fallbacks

```typescript
interface FallbackOptions<T> {
  fallbackValue?: T;
  fallbackFunction?: () => T | Promise<T>;
  cache?: Map<string, { value: T; timestamp: number }>;
  cacheKey?: string;
  cacheTTLMs?: number;
}

class GracefulDegradation {
  async withFallback<T>(
    operation: () => Promise<T>,
    options: FallbackOptions<T>
  ): Promise<T> {
    try {
      // Try to get from cache first
      if (options.cache && options.cacheKey) {
        const cached = this.getFromCache(options.cache, options.cacheKey, options.cacheTTLMs);
        if (cached !== null) {
          console.log(`Cache hit for ${options.cacheKey}`);
          return cached;
        }
      }
      
      // Execute operation
      const result = await operation();
      
      // Cache successful result
      if (options.cache && options.cacheKey) {
        this.setCache(options.cache, options.cacheKey, result);
      }
      
      return result;
    } catch (error) {
      console.error('Operation failed, using fallback:', error);
      
      // Try cache as first fallback
      if (options.cache && options.cacheKey) {
        const cached = this.getFromCache(
          options.cache,
          options.cacheKey,
          Infinity // Accept stale cache in error scenarios
        );
        if (cached !== null) {
          console.log(`Using stale cache for ${options.cacheKey}`);
          return cached;
        }
      }
      
      // Use fallback function
      if (options.fallbackFunction) {
        return await options.fallbackFunction();
      }
      
      // Use fallback value
      if (options.fallbackValue !== undefined) {
        return options.fallbackValue;
      }
      
      // No fallback available, re-throw
      throw error;
    }
  }
  
  private getFromCache<T>(
    cache: Map<string, { value: T; timestamp: number }>,
    key: string,
    ttlMs?: number
  ): T | null {
    const cached = cache.get(key);
    if (!cached) return null;
    
    if (ttlMs !== undefined && ttlMs !== Infinity) {
      const age = Date.now() - cached.timestamp;
      if (age > ttlMs) {
        cache.delete(key);
        return null;
      }
    }
    
    return cached.value;
  }
  
  private setCache<T>(
    cache: Map<string, { value: T; timestamp: number }>,
    key: string,
    value: T
  ): void {
    cache.set(key, { value, timestamp: Date.now() });
  }
}

// Usage example
const degradation = new GracefulDegradation();
const cache = new Map();

const items = await degradation.withFallback(
  () => client.items.list({ filter: { type: 'article' } }),
  {
    cache,
    cacheKey: 'articles-list',
    cacheTTLMs: 300000, // 5 minutes
    fallbackFunction: async () => {
      // Try alternative data source
      console.log('Fetching from backup source...');
      return [];
    },
    fallbackValue: [] // Last resort
  }
);
```

### Progressive Error Recovery

```typescript
interface RecoveryStrategy<T> {
  name: string;
  attempt: () => Promise<T>;
  shouldTry?: (previousError: Error) => boolean;
}

class ProgressiveRecovery {
  async recover<T>(
    strategies: RecoveryStrategy<T>[]
  ): Promise<{ result?: T; errors: Array<{ strategy: string; error: Error }> }> {
    const errors: Array<{ strategy: string; error: Error }> = [];
    let lastError: Error | null = null;
    
    for (const strategy of strategies) {
      // Check if we should try this strategy
      if (strategy.shouldTry && lastError && !strategy.shouldTry(lastError)) {
        continue;
      }
      
      try {
        console.log(`Attempting recovery strategy: ${strategy.name}`);
        const result = await strategy.attempt();
        return { result, errors };
      } catch (error) {
        lastError = error as Error;
        errors.push({ strategy: strategy.name, error: error as Error });
        console.error(`Strategy '${strategy.name}' failed:`, error);
      }
    }
    
    return { errors };
  }
}

// Usage example
const recovery = new ProgressiveRecovery();

const { result, errors } = await recovery.recover<any>([
  {
    name: 'Primary API',
    attempt: () => client.items.find('123')
  },
  {
    name: 'Retry with backoff',
    attempt: () => retry.execute(() => client.items.find('123')),
    shouldTry: (error) => isApiError(error) && error.statusCode >= 500
  },
  {
    name: 'Alternative endpoint',
    attempt: () => client.items.list({
      filter: { ids: ['123'] },
      limit: 1
    }).then(items => items[0])
  },
  {
    name: 'Cached version',
    attempt: async () => {
      const cached = localStorage.getItem('item-123');
      if (!cached) throw new Error('No cached version');
      return JSON.parse(cached);
    }
  },
  {
    name: 'Default content',
    attempt: async () => ({
      id: '123',
      title: 'Content unavailable',
      status: 'error'
    })
  }
]);
```

## Validation Error Handling

### Comprehensive Validation Error Processing

```typescript
interface FieldValidation {
  field: string;
  code: string;
  message: string;
  details?: {
    min?: number;
    max?: number;
    pattern?: string;
    required?: boolean;
    unique?: boolean;
    [key: string]: any;
  };
}

class ValidationErrorHandler {
  private fieldRules: Map<string, Array<(value: any) => string | null>> = new Map();
  
  // Extract validation errors from API response
  parseApiValidationErrors(error: ApiError): FieldValidation[] {
    if (error.statusCode !== StatusCodes.UNPROCESSABLE_ENTITY) {
      return [];
    }
    
    return (error.body.data || []).map(err => ({
      field: err.attributes.field || 'unknown',
      code: err.attributes.code,
      message: err.attributes.message,
      details: err.attributes.details
    }));
  }
  
  // Create user-friendly error messages
  formatUserMessage(validation: FieldValidation): string {
    const fieldName = this.humanizeFieldName(validation.field);
    
    switch (validation.code) {
      case 'REQUIRED':
        return `${fieldName} is required`;
      case 'INVALID_FORMAT':
        return `${fieldName} has an invalid format`;
      case 'TOO_SHORT':
        return `${fieldName} must be at least ${validation.details?.min} characters`;
      case 'TOO_LONG':
        return `${fieldName} must be no more than ${validation.details?.max} characters`;
      case 'DUPLICATE':
        return `${fieldName} must be unique`;
      case 'INVALID_TYPE':
        return `${fieldName} has an invalid type`;
      case 'OUT_OF_RANGE':
        return `${fieldName} is out of allowed range`;
      default:
        return validation.message || `${fieldName} is invalid`;
    }
  }
  
  private humanizeFieldName(field: string): string {
    return field
      .split(/[._-]/)
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join(' ');
  }
  
  // Client-side validation
  addFieldRule(field: string, rule: (value: any) => string | null): void {
    if (!this.fieldRules.has(field)) {
      this.fieldRules.set(field, []);
    }
    this.fieldRules.get(field)!.push(rule);
  }
  
  validateField(field: string, value: any): string[] {
    const rules = this.fieldRules.get(field) || [];
    const errors: string[] = [];
    
    for (const rule of rules) {
      const error = rule(value);
      if (error) {
        errors.push(error);
      }
    }
    
    return errors;
  }
  
  validateForm(data: Record<string, any>): Record<string, string[]> {
    const errors: Record<string, string[]> = {};
    
    for (const [field, value] of Object.entries(data)) {
      const fieldErrors = this.validateField(field, value);
      if (fieldErrors.length > 0) {
        errors[field] = fieldErrors;
      }
    }
    
    return errors;
  }
}

// Common validation rules
const ValidationRules = {
  required: (message = 'This field is required') => 
    (value: any) => (!value && value !== 0) ? message : null,
    
  minLength: (min: number, message?: string) =>
    (value: string) => (value && value.length < min) 
      ? (message || `Must be at least ${min} characters`) 
      : null,
      
  maxLength: (max: number, message?: string) =>
    (value: string) => (value && value.length > max)
      ? (message || `Must be no more than ${max} characters`)
      : null,
      
  pattern: (regex: RegExp, message = 'Invalid format') =>
    (value: string) => (value && !regex.test(value)) ? message : null,
    
  email: (message = 'Invalid email address') =>
    (value: string) => {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      return value && !emailRegex.test(value) ? message : null;
    },
    
  url: (message = 'Invalid URL') =>
    (value: string) => {
      try {
        new URL(value);
        return null;
      } catch {
        return message;
      }
    }
};

// Usage example
const validator = new ValidationErrorHandler();

// Setup validation rules
validator.addFieldRule('title', ValidationRules.required());
validator.addFieldRule('title', ValidationRules.minLength(3));
validator.addFieldRule('title', ValidationRules.maxLength(100));
validator.addFieldRule('email', ValidationRules.email());
validator.addFieldRule('website', ValidationRules.url());

// Validate before API call
const formData = {
  title: 'My Article',
  email: 'user@example.com',
  website: 'https://example.com'
};

const clientErrors = validator.validateForm(formData);
if (Object.keys(clientErrors).length > 0) {
  console.error('Validation errors:', clientErrors);
  return;
}

// Handle API validation errors
try {
  await client.items.create({
    itemType: 'article',
    ...formData
  });
} catch (error) {
  if (isApiError(error)) {
    const validationErrors = validator.parseApiValidationErrors(error);
    const userMessages = validationErrors.map(err => ({
      field: err.field,
      message: validator.formatUserMessage(err)
    }));
    
    // Display to user
    userMessages.forEach(({ field, message }) => {
      console.error(`${field}: ${message}`);
    });
  }
}
```

## Network Error Handling

### Comprehensive Network Error Detection and Recovery

```typescript
interface NetworkErrorInfo {
  type: 'timeout' | 'connection' | 'dns' | 'unknown';
  message: string;
  originalError?: Error;
  retryable: boolean;
}

class NetworkErrorHandler {
  private onlineStatus = true;
  private networkListeners: Array<(online: boolean) => void> = [];
  
  constructor() {
    // Monitor network status
    if (typeof window !== 'undefined') {
      window.addEventListener('online', () => this.setOnlineStatus(true));
      window.addEventListener('offline', () => this.setOnlineStatus(false));
    }
  }
  
  classifyNetworkError(error: Error): NetworkErrorInfo {
    const message = error.message.toLowerCase();
    
    // Timeout errors
    if (message.includes('timeout') || message.includes('timed out')) {
      return {
        type: 'timeout',
        message: 'Request timed out',
        originalError: error,
        retryable: true
      };
    }
    
    // Connection errors
    if (
      message.includes('econnrefused') ||
      message.includes('econnreset') ||
      message.includes('enotfound') ||
      message.includes('network')
    ) {
      return {
        type: 'connection',
        message: 'Connection failed',
        originalError: error,
        retryable: true
      };
    }
    
    // DNS errors
    if (message.includes('getaddrinfo') || message.includes('dns')) {
      return {
        type: 'dns',
        message: 'DNS resolution failed',
        originalError: error,
        retryable: true
      };
    }
    
    // Unknown network errors
    return {
      type: 'unknown',
      message: 'Unknown network error',
      originalError: error,
      retryable: false
    };
  }
  
  async withNetworkRetry<T>(
    operation: () => Promise<T>,
    options: {
      maxAttempts?: number;
      timeout?: number;
      onRetry?: (attempt: number, error: NetworkErrorInfo) => void;
    } = {}
  ): Promise<T> {
    const { maxAttempts = 3, timeout = 30000, onRetry } = options;
    
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        // Check network status
        if (!this.onlineStatus) {
          throw new NetworkError('No network connection', new Error('Offline'));
        }
        
        // Execute with timeout
        return await this.withTimeout(operation(), timeout);
      } catch (error) {
        const networkError = this.classifyNetworkError(error as Error);
        
        if (!networkError.retryable || attempt === maxAttempts) {
          throw new NetworkError(
            `Network operation failed: ${networkError.message}`,
            error as Error
          );
        }
        
        onRetry?.(attempt, networkError);
        
        // Wait before retry (with exponential backoff)
        const delay = Math.min(1000 * Math.pow(2, attempt - 1), 10000);
        await this.sleep(delay);
      }
    }
    
    throw new Error('Unreachable');
  }
  
  private withTimeout<T>(promise: Promise<T>, timeoutMs: number): Promise<T> {
    return Promise.race([
      promise,
      new Promise<T>((_, reject) => {
        setTimeout(() => {
          reject(new Error(`Operation timed out after ${timeoutMs}ms`));
        }, timeoutMs);
      })
    ]);
  }
  
  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  private setOnlineStatus(online: boolean): void {
    this.onlineStatus = online;
    this.networkListeners.forEach(listener => listener(online));
  }
  
  onNetworkStatusChange(listener: (online: boolean) => void): () => void {
    this.networkListeners.push(listener);
    return () => {
      this.networkListeners = this.networkListeners.filter(l => l !== listener);
    };
  }
  
  isOnline(): boolean {
    return this.onlineStatus;
  }
}

// Axios interceptor example
import axios, { AxiosError } from 'axios';

const axiosInstance = axios.create({
  baseURL: 'https://site-api.datocms.com',
  timeout: 30000
});

const networkHandler = new NetworkErrorHandler();

// Request interceptor
axiosInstance.interceptors.request.use(
  config => {
    if (!networkHandler.isOnline()) {
      return Promise.reject(new NetworkError('No network connection'));
    }
    return config;
  },
  error => Promise.reject(error)
);

// Response interceptor
axiosInstance.interceptors.response.use(
  response => response,
  async (error: AxiosError) => {
    const originalRequest = error.config;
    
    if (!originalRequest) {
      return Promise.reject(error);
    }
    
    // Handle network errors with retry
    if (!error.response && error.code === 'ECONNABORTED') {
      return networkHandler.withNetworkRetry(
        () => axiosInstance.request(originalRequest),
        {
          onRetry: (attempt, networkError) => {
            console.log(`Network retry attempt ${attempt}: ${networkError.message}`);
          }
        }
      );
    }
    
    return Promise.reject(error);
  }
);
```

## Rate Limit Handling

### Advanced Rate Limit Management

```typescript
interface RateLimitInfo {
  limit: number;
  remaining: number;
  reset: Date;
  retryAfter?: number;
}

class RateLimitManager {
  private queues: Map<string, Array<{
    execute: () => Promise<any>;
    resolve: (value: any) => void;
    reject: (error: any) => void;
  }>> = new Map();
  
  private rateLimits: Map<string, RateLimitInfo> = new Map();
  private processing: Set<string> = new Set();
  
  async execute<T>(
    endpoint: string,
    operation: () => Promise<T>
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      // Get queue for endpoint
      if (!this.queues.has(endpoint)) {
        this.queues.set(endpoint, []);
      }
      
      this.queues.get(endpoint)!.push({
        execute: operation,
        resolve,
        reject
      });
      
      this.processQueue(endpoint);
    });
  }
  
  private async processQueue(endpoint: string): Promise<void> {
    if (this.processing.has(endpoint)) return;
    this.processing.add(endpoint);
    
    const queue = this.queues.get(endpoint);
    if (!queue || queue.length === 0) {
      this.processing.delete(endpoint);
      return;
    }
    
    while (queue.length > 0) {
      const item = queue.shift()!;
      
      try {
        // Check rate limit
        const rateLimit = this.rateLimits.get(endpoint);
        if (rateLimit && rateLimit.remaining === 0) {
          const waitTime = rateLimit.reset.getTime() - Date.now();
          if (waitTime > 0) {
            console.log(`Rate limit reached for ${endpoint}, waiting ${waitTime}ms`);
            await this.sleep(waitTime);
          }
        }
        
        // Execute operation
        const result = await item.execute();
        item.resolve(result);
        
        // Add small delay between requests
        if (queue.length > 0) {
          await this.sleep(100);
        }
      } catch (error) {
        if (isApiError(error) && error.statusCode === StatusCodes.TOO_MANY_REQUESTS) {
          // Update rate limit info
          this.updateRateLimitInfo(endpoint, error);
          
          // Re-queue the request
          queue.unshift(item);
          
          // Wait for rate limit reset
          const retryAfter = parseInt(error.headers['retry-after'] || '60');
          console.log(`Rate limited on ${endpoint}, retrying after ${retryAfter}s`);
          await this.sleep(retryAfter * 1000);
        } else {
          item.reject(error);
        }
      }
    }
    
    this.processing.delete(endpoint);
  }
  
  private updateRateLimitInfo(endpoint: string, error: ApiError): void {
    const headers = error.headers;
    this.rateLimits.set(endpoint, {
      limit: parseInt(headers['x-ratelimit-limit'] || '0'),
      remaining: parseInt(headers['x-ratelimit-remaining'] || '0'),
      reset: new Date(parseInt(headers['x-ratelimit-reset'] || '0') * 1000),
      retryAfter: parseInt(headers['retry-after'] || '60')
    });
  }
  
  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  getRateLimitInfo(endpoint: string): RateLimitInfo | undefined {
    return this.rateLimits.get(endpoint);
  }
}

// Intelligent rate limit predictor
class RateLimitPredictor {
  private requestHistory: Map<string, Array<{
    timestamp: number;
    endpoint: string;
  }>> = new Map();
  
  private limits: Map<string, number> = new Map([
    ['/items', 30],          // 30 requests per second
    ['/uploads', 10],        // 10 requests per second
    ['/item-types', 10],     // 10 requests per second
    ['default', 30]          // Default limit
  ]);
  
  canMakeRequest(endpoint: string): boolean {
    const limit = this.getLimit(endpoint);
    const history = this.getRecentRequests(endpoint);
    
    return history.length < limit;
  }
  
  recordRequest(endpoint: string): void {
    if (!this.requestHistory.has(endpoint)) {
      this.requestHistory.set(endpoint, []);
    }
    
    this.requestHistory.get(endpoint)!.push({
      timestamp: Date.now(),
      endpoint
    });
    
    // Clean old entries
    this.cleanHistory(endpoint);
  }
  
  getWaitTime(endpoint: string): number {
    if (this.canMakeRequest(endpoint)) return 0;
    
    const history = this.getRecentRequests(endpoint);
    const oldestRequest = history[0];
    const waitTime = (oldestRequest.timestamp + 1000) - Date.now();
    
    return Math.max(0, waitTime);
  }
  
  private getLimit(endpoint: string): number {
    for (const [pattern, limit] of this.limits) {
      if (endpoint.includes(pattern)) {
        return limit;
      }
    }
    return this.limits.get('default')!;
  }
  
  private getRecentRequests(endpoint: string): Array<{ timestamp: number; endpoint: string }> {
    const history = this.requestHistory.get(endpoint) || [];
    const oneSecondAgo = Date.now() - 1000;
    
    return history.filter(req => req.timestamp > oneSecondAgo);
  }
  
  private cleanHistory(endpoint: string): void {
    const history = this.requestHistory.get(endpoint);
    if (!history) return;
    
    const oneSecondAgo = Date.now() - 1000;
    const recentRequests = history.filter(req => req.timestamp > oneSecondAgo);
    
    this.requestHistory.set(endpoint, recentRequests);
  }
}

// Combined usage
const rateLimitManager = new RateLimitManager();
const rateLimitPredictor = new RateLimitPredictor();

async function makeApiCall<T>(
  endpoint: string,
  operation: () => Promise<T>
): Promise<T> {
  // Check prediction
  const waitTime = rateLimitPredictor.getWaitTime(endpoint);
  if (waitTime > 0) {
    console.log(`Predicted rate limit, waiting ${waitTime}ms`);
    await new Promise(resolve => setTimeout(resolve, waitTime));
  }
  
  // Record request
  rateLimitPredictor.recordRequest(endpoint);
  
  // Execute with rate limit management
  return rateLimitManager.execute(endpoint, operation);
}
```

## Error Logging and Monitoring

### Comprehensive Error Tracking System

```typescript
interface ErrorContext {
  userId?: string;
  sessionId?: string;
  endpoint?: string;
  method?: string;
  requestData?: any;
  responseData?: any;
  duration?: number;
  retryCount?: number;
  [key: string]: any;
}

interface ErrorReport {
  id: string;
  timestamp: Date;
  error: {
    name: string;
    message: string;
    stack?: string;
    code?: string;
  };
  context: ErrorContext;
  severity: 'low' | 'medium' | 'high' | 'critical';
  fingerprint: string;
}

class ErrorMonitor {
  private reports: ErrorReport[] = [];
  private listeners: Array<(report: ErrorReport) => void> = [];
  private config = {
    maxReports: 1000,
    flushInterval: 60000, // 1 minute
    endpoint: '/api/errors',
    enableConsoleLog: true
  };
  
  constructor(config?: Partial<typeof ErrorMonitor.prototype.config>) {
    this.config = { ...this.config, ...config };
    this.startFlushInterval();
  }
  
  captureError(
    error: Error,
    context: ErrorContext = {},
    severity: ErrorReport['severity'] = 'medium'
  ): void {
    const report: ErrorReport = {
      id: this.generateId(),
      timestamp: new Date(),
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack,
        code: (error as any).code
      },
      context,
      severity,
      fingerprint: this.generateFingerprint(error, context)
    };
    
    // Add API error details
    if (isApiError(error)) {
      report.context.statusCode = error.statusCode;
      report.context.apiError = error.body;
      report.severity = this.getSeverityFromStatusCode(error.statusCode);
    }
    
    // Store report
    this.reports.push(report);
    if (this.reports.length > this.config.maxReports) {
      this.reports.shift();
    }
    
    // Notify listeners
    this.listeners.forEach(listener => listener(report));
    
    // Console logging
    if (this.config.enableConsoleLog) {
      this.logToConsole(report);
    }
    
    // Send critical errors immediately
    if (severity === 'critical') {
      this.flush();
    }
  }
  
  private generateId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  private generateFingerprint(error: Error, context: ErrorContext): string {
    const parts = [
      error.name,
      error.message,
      context.endpoint || '',
      context.method || ''
    ];
    
    return parts.join('|').replace(/[^a-zA-Z0-9|]/g, '');
  }
  
  private getSeverityFromStatusCode(statusCode: number): ErrorReport['severity'] {
    if (statusCode >= 500) return 'high';
    if (statusCode === 429) return 'medium';
    if (statusCode >= 400) return 'low';
    return 'medium';
  }
  
  private logToConsole(report: ErrorReport): void {
    const style = {
      low: 'color: #FFA500',
      medium: 'color: #FF6347',
      high: 'color: #FF0000',
      critical: 'color: #8B0000; font-weight: bold'
    };
    
    console.group(
      `%c[${report.severity.toUpperCase()}] ${report.error.name}`,
      style[report.severity]
    );
    console.error('Message:', report.error.message);
    console.error('Context:', report.context);
    if (report.error.stack) {
      console.error('Stack:', report.error.stack);
    }
    console.groupEnd();
  }
  
  private startFlushInterval(): void {
    setInterval(() => {
      if (this.reports.length > 0) {
        this.flush();
      }
    }, this.config.flushInterval);
  }
  
  private async flush(): Promise<void> {
    if (this.reports.length === 0) return;
    
    const reportsToSend = [...this.reports];
    this.reports = [];
    
    try {
      // Send to monitoring service
      await fetch(this.config.endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ reports: reportsToSend })
      });
    } catch (error) {
      // Re-add reports if sending failed
      this.reports.unshift(...reportsToSend);
      console.error('Failed to send error reports:', error);
    }
  }
  
  onError(listener: (report: ErrorReport) => void): () => void {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }
  
  getErrorSummary(): {
    total: number;
    bySeverity: Record<string, number>;
    byEndpoint: Record<string, number>;
    recentErrors: ErrorReport[];
  } {
    const summary = {
      total: this.reports.length,
      bySeverity: {} as Record<string, number>,
      byEndpoint: {} as Record<string, number>,
      recentErrors: this.reports.slice(-10)
    };
    
    this.reports.forEach(report => {
      // Count by severity
      summary.bySeverity[report.severity] = 
        (summary.bySeverity[report.severity] || 0) + 1;
        
      // Count by endpoint
      if (report.context.endpoint) {
        summary.byEndpoint[report.context.endpoint] = 
          (summary.byEndpoint[report.context.endpoint] || 0) + 1;
      }
    });
    
    return summary;
  }
}

// Integration with external services
class SentryIntegration {
  constructor(private monitor: ErrorMonitor) {
    monitor.onError(report => {
      if (typeof window !== 'undefined' && (window as any).Sentry) {
        (window as any).Sentry.captureException(report.error, {
          level: this.mapSeverity(report.severity),
          tags: {
            endpoint: report.context.endpoint,
            method: report.context.method
          },
          extra: report.context
        });
      }
    });
  }
  
  private mapSeverity(severity: ErrorReport['severity']): string {
    const mapping = {
      low: 'info',
      medium: 'warning',
      high: 'error',
      critical: 'fatal'
    };
    return mapping[severity];
  }
}

// Global error monitor instance
const errorMonitor = new ErrorMonitor({
  enableConsoleLog: process.env.NODE_ENV === 'development'
});

// Global error handler
if (typeof window !== 'undefined') {
  window.addEventListener('unhandledrejection', event => {
    errorMonitor.captureError(
      new Error(event.reason?.message || 'Unhandled Promise Rejection'),
      { reason: event.reason },
      'high'
    );
  });
}
```

## User-Friendly Error Messages

### Error Message Localization and Formatting

```typescript
interface ErrorMessageConfig {
  locale: string;
  fallbackLocale: string;
  messages: Record<string, Record<string, string>>;
}

class UserErrorFormatter {
  private config: ErrorMessageConfig = {
    locale: 'en',
    fallbackLocale: 'en',
    messages: {
      en: {
        'network.offline': 'No internet connection. Please check your network.',
        'network.timeout': 'Request timed out. Please try again.',
        'network.server_error': 'Server error. Our team has been notified.',
        'auth.invalid_token': 'Your session has expired. Please sign in again.',
        'auth.insufficient_permissions': 'You don\'t have permission to perform this action.',
        'validation.required_field': '{field} is required.',
        'validation.invalid_format': '{field} has an invalid format.',
        'validation.too_long': '{field} is too long (maximum {max} characters).',
        'rate_limit.exceeded': 'Too many requests. Please wait {time} seconds.',
        'resource.not_found': 'The requested {resource} was not found.',
        'generic.error': 'Something went wrong. Please try again later.'
      },
      es: {
        'network.offline': 'Sin conexión a internet. Por favor, verifica tu red.',
        'network.timeout': 'La solicitud ha expirado. Por favor, intenta de nuevo.',
        'network.server_error': 'Error del servidor. Nuestro equipo ha sido notificado.',
        'auth.invalid_token': 'Tu sesión ha expirado. Por favor, inicia sesión nuevamente.',
        'auth.insufficient_permissions': 'No tienes permiso para realizar esta acción.',
        'validation.required_field': '{field} es obligatorio.',
        'validation.invalid_format': '{field} tiene un formato inválido.',
        'validation.too_long': '{field} es demasiado largo (máximo {max} caracteres).',
        'rate_limit.exceeded': 'Demasiadas solicitudes. Por favor, espera {time} segundos.',
        'resource.not_found': 'El {resource} solicitado no fue encontrado.',
        'generic.error': 'Algo salió mal. Por favor, intenta más tarde.'
      }
    }
  };
  
  constructor(config?: Partial<ErrorMessageConfig>) {
    if (config) {
      this.config = { ...this.config, ...config };
    }
  }
  
  formatError(error: Error, context?: Record<string, any>): string {
    // Try to get specific message
    const messageKey = this.getMessageKey(error);
    const template = this.getMessage(messageKey);
    
    // Format with context
    return this.interpolate(template, { ...context, error });
  }
  
  private getMessageKey(error: Error): string {
    if (error instanceof NetworkError) {
      if (!navigator.onLine) return 'network.offline';
      if (error.message.includes('timeout')) return 'network.timeout';
      return 'network.server_error';
    }
    
    if (isApiError(error)) {
      switch (error.statusCode) {
        case 401: return 'auth.invalid_token';
        case 403: return 'auth.insufficient_permissions';
        case 404: return 'resource.not_found';
        case 422: return 'validation.invalid_format';
        case 429: return 'rate_limit.exceeded';
        case 500: return 'network.server_error';
        default: return 'generic.error';
      }
    }
    
    return 'generic.error';
  }
  
  private getMessage(key: string): string {
    const messages = this.config.messages[this.config.locale] || 
                    this.config.messages[this.config.fallbackLocale];
    return messages[key] || messages['generic.error'];
  }
  
  private interpolate(template: string, context: Record<string, any>): string {
    return template.replace(/{(\w+)}/g, (match, key) => {
      return context[key]?.toString() || match;
    });
  }
  
  setLocale(locale: string): void {
    this.config.locale = locale;
  }
}

// Error dialog component
class ErrorDialog {
  show(error: Error, options: {
    title?: string;
    retryAction?: () => Promise<void>;
    dismissAction?: () => void;
  } = {}): void {
    const formatter = new UserErrorFormatter();
    const message = formatter.formatError(error);
    
    // Create dialog HTML
    const dialog = document.createElement('div');
    dialog.className = 'error-dialog';
    dialog.innerHTML = `
      <div class="error-dialog-content">
        <h3>${options.title || 'Error'}</h3>
        <p>${message}</p>
        <div class="error-dialog-actions">
          ${options.retryAction ? '<button class="retry-btn">Retry</button>' : ''}
          <button class="dismiss-btn">Dismiss</button>
        </div>
      </div>
    `;
    
    // Add event listeners
    if (options.retryAction) {
      dialog.querySelector('.retry-btn')?.addEventListener('click', async () => {
        try {
          await options.retryAction!();
          this.close(dialog);
        } catch (retryError) {
          // Show retry error
          this.show(retryError as Error, options);
        }
      });
    }
    
    dialog.querySelector('.dismiss-btn')?.addEventListener('click', () => {
      options.dismissAction?.();
      this.close(dialog);
    });
    
    document.body.appendChild(dialog);
  }
  
  private close(dialog: HTMLElement): void {
    dialog.remove();
  }
}
```

## Error Boundary Patterns for React

### React Error Boundaries with Recovery

```typescript
import React, { Component, ErrorInfo, ReactNode } from 'react';

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
  errorInfo: ErrorInfo | null;
  retryCount: number;
}

interface ErrorBoundaryProps {
  fallback?: (error: Error, retry: () => void) => ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
  isolate?: boolean;
  maxRetries?: number;
  children: ReactNode;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState = {
    hasError: false,
    error: null,
    errorInfo: null,
    retryCount: 0
  };
  
  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    // Log to monitoring service
    errorMonitor.captureError(error, {
      component: errorInfo.componentStack,
      props: this.props
    }, 'high');
    
    // Call custom error handler
    this.props.onError?.(error, errorInfo);
    
    this.setState({ errorInfo });
  }
  
  retry = (): void => {
    const { maxRetries = 3 } = this.props;
    
    if (this.state.retryCount < maxRetries) {
      this.setState({
        hasError: false,
        error: null,
        errorInfo: null,
        retryCount: this.state.retryCount + 1
      });
    }
  };
  
  render(): ReactNode {
    if (this.state.hasError && this.state.error) {
      if (this.props.fallback) {
        return this.props.fallback(this.state.error, this.retry);
      }
      
      return (
        <div className="error-boundary-default">
          <h2>Something went wrong</h2>
          <details style={{ whiteSpace: 'pre-wrap' }}>
            <summary>Error details</summary>
            {this.state.error.toString()}
            {this.state.errorInfo?.componentStack}
          </details>
          <button onClick={this.retry}>Try again</button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Async error boundary
interface AsyncErrorBoundaryState {
  asyncError: Error | null;
}

class AsyncErrorBoundary extends Component<
  ErrorBoundaryProps,
  ErrorBoundaryState & AsyncErrorBoundaryState
> {
  state: ErrorBoundaryState & AsyncErrorBoundaryState = {
    hasError: false,
    error: null,
    errorInfo: null,
    retryCount: 0,
    asyncError: null
  };
  
  private resetError = (): void => {
    this.setState({ asyncError: null });
  };
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    super.componentDidCatch?.(error, errorInfo);
  }
  
  render(): ReactNode {
    const error = this.state.error || this.state.asyncError;
    
    if (error) {
      return (
        <ErrorBoundary>
          {this.props.fallback?.(error, this.resetError) || (
            <div>
              <p>An error occurred: {error.message}</p>
              <button onClick={this.resetError}>Dismiss</button>
            </div>
          )}
        </ErrorBoundary>
      );
    }
    
    return (
      <AsyncErrorContext.Provider
        value={{
          setError: (error: Error) => this.setState({ asyncError: error }),
          clearError: this.resetError
        }}
      >
        {this.props.children}
      </AsyncErrorContext.Provider>
    );
  }
}

const AsyncErrorContext = React.createContext<{
  setError: (error: Error) => void;
  clearError: () => void;
}>({
  setError: () => {},
  clearError: () => {}
});

// Hook for async error handling
function useAsyncError(): (error: Error) => void {
  const { setError } = React.useContext(AsyncErrorContext);
  return React.useCallback(
    (error: Error) => {
      setError(error);
    },
    [setError]
  );
}

// Usage example
function MyApp() {
  return (
    <ErrorBoundary
      fallback={(error, retry) => (
        <div className="error-page">
          <h1>Oops! Something went wrong</h1>
          <p>{error.message}</p>
          <button onClick={retry}>Try Again</button>
        </div>
      )}
      onError={(error, errorInfo) => {
        console.error('App error:', error, errorInfo);
      }}
    >
      <AsyncErrorBoundary>
        <AppContent />
      </AsyncErrorBoundary>
    </ErrorBoundary>
  );
}

// Component with error handling
function DataComponent() {
  const throwAsyncError = useAsyncError();
  const [data, setData] = React.useState(null);
  const [loading, setLoading] = React.useState(false);
  
  const fetchData = async () => {
    setLoading(true);
    try {
      const result = await client.items.list();
      setData(result);
    } catch (error) {
      throwAsyncError(error as Error);
    } finally {
      setLoading(false);
    }
  };
  
  React.useEffect(() => {
    fetchData();
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (!data) return null;
  
  return <div>{/* Render data */}</div>;
}
```

## Testing Error Conditions

### Comprehensive Error Testing Patterns

```typescript
import { jest } from '@jest/globals';

// Mock error scenarios
class ErrorTestUtils {
  static createApiError(
    statusCode: number,
    message: string,
    body?: any
  ): ApiError {
    const error = new Error(message) as ApiError;
    error.name = 'ApiError';
    error.statusCode = statusCode;
    error.statusText = this.getStatusText(statusCode);
    error.headers = {
      'x-ratelimit-limit': '30',
      'x-ratelimit-remaining': '0',
      'x-ratelimit-reset': String(Date.now() / 1000 + 60),
      'retry-after': '60'
    };
    error.body = body || {
      data: [{
        id: '1',
        type: 'api_error',
        attributes: {
          code: 'TEST_ERROR',
          message,
          details: {}
        }
      }]
    };
    return error;
  }
  
  static createNetworkError(type: 'timeout' | 'connection' | 'dns'): Error {
    const messages = {
      timeout: 'Request timeout',
      connection: 'ECONNREFUSED',
      dns: 'getaddrinfo ENOTFOUND'
    };
    return new Error(messages[type]);
  }
  
  static createValidationError(fields: Array<{
    field: string;
    code: string;
    message: string;
  }>): ApiError {
    return this.createApiError(422, 'Validation failed', {
      data: fields.map((f, i) => ({
        id: String(i),
        type: 'api_error',
        attributes: {
          field: f.field,
          code: f.code,
          message: f.message,
          details: {}
        }
      }))
    });
  }
  
  private static getStatusText(statusCode: number): string {
    const statusTexts: Record<number, string> = {
      400: 'Bad Request',
      401: 'Unauthorized',
      403: 'Forbidden',
      404: 'Not Found',
      422: 'Unprocessable Entity',
      429: 'Too Many Requests',
      500: 'Internal Server Error',
      503: 'Service Unavailable'
    };
    return statusTexts[statusCode] || 'Unknown';
  }
}

// Test examples
describe('Error Handling', () => {
  let client: any;
  let errorHandler: any;
  
  beforeEach(() => {
    client = {
      items: {
        find: jest.fn(),
        create: jest.fn(),
        list: jest.fn()
      }
    };
    errorHandler = new ErrorHandler();
  });
  
  describe('API Errors', () => {
    it('should handle 404 errors gracefully', async () => {
      const error = ErrorTestUtils.createApiError(404, 'Item not found');
      client.items.find.mockRejectedValue(error);
      
      const result = await errorHandler.handleNotFound(
        () => client.items.find('123'),
        'default-value'
      );
      
      expect(result).toBe('default-value');
      expect(client.items.find).toHaveBeenCalledWith('123');
    });
    
    it('should retry on 503 errors', async () => {
      const error = ErrorTestUtils.createApiError(503, 'Service unavailable');
      client.items.find
        .mockRejectedValueOnce(error)
        .mockRejectedValueOnce(error)
        .mockResolvedValueOnce({ id: '123', title: 'Success' });
      
      const result = await retry.execute(() => client.items.find('123'));
      
      expect(result).toEqual({ id: '123', title: 'Success' });
      expect(client.items.find).toHaveBeenCalledTimes(3);
    });
    
    it('should parse validation errors correctly', async () => {
      const error = ErrorTestUtils.createValidationError([
        { field: 'title', code: 'REQUIRED', message: 'Title is required' },
        { field: 'slug', code: 'DUPLICATE', message: 'Slug must be unique' }
      ]);
      
      client.items.create.mockRejectedValue(error);
      
      try {
        await client.items.create({ itemType: 'article' });
      } catch (e) {
        const validationErrors = validator.parseApiValidationErrors(e as ApiError);
        expect(validationErrors).toHaveLength(2);
        expect(validationErrors[0].field).toBe('title');
        expect(validationErrors[1].field).toBe('slug');
      }
    });
  });
  
  describe('Network Errors', () => {
    it('should classify timeout errors correctly', () => {
      const error = ErrorTestUtils.createNetworkError('timeout');
      const classified = networkHandler.classifyNetworkError(error);
      
      expect(classified.type).toBe('timeout');
      expect(classified.retryable).toBe(true);
    });
    
    it('should handle offline scenarios', async () => {
      // Mock offline
      Object.defineProperty(navigator, 'onLine', {
        writable: true,
        value: false
      });
      
      await expect(
        networkHandler.withNetworkRetry(() => client.items.list())
      ).rejects.toThrow('No network connection');
    });
  });
  
  describe('Rate Limiting', () => {
    it('should queue requests when rate limited', async () => {
      const rateLimitError = ErrorTestUtils.createApiError(429, 'Rate limit exceeded');
      
      client.items.list
        .mockRejectedValueOnce(rateLimitError)
        .mockResolvedValueOnce([{ id: '1' }]);
      
      const start = Date.now();
      const result = await rateLimitManager.execute(
        '/items',
        () => client.items.list()
      );
      const duration = Date.now() - start;
      
      expect(result).toEqual([{ id: '1' }]);
      expect(duration).toBeGreaterThan(50); // Should have waited
    });
  });
  
  describe('Error Recovery', () => {
    it('should use cache when API fails', async () => {
      const cache = new Map();
      cache.set('test-key', { value: { id: '123' }, timestamp: Date.now() });
      
      const error = ErrorTestUtils.createApiError(500, 'Server error');
      client.items.find.mockRejectedValue(error);
      
      const result = await degradation.withFallback(
        () => client.items.find('123'),
        {
          cache,
          cacheKey: 'test-key',
          cacheTTLMs: 300000
        }
      );
      
      expect(result).toEqual({ id: '123' });
    });
    
    it('should execute progressive recovery strategies', async () => {
      const strategies = [
        {
          name: 'Primary',
          attempt: jest.fn().mockRejectedValue(new Error('Primary failed'))
        },
        {
          name: 'Secondary',
          attempt: jest.fn().mockRejectedValue(new Error('Secondary failed'))
        },
        {
          name: 'Fallback',
          attempt: jest.fn().mockResolvedValue({ success: true })
        }
      ];
      
      const { result, errors } = await recovery.recover(strategies);
      
      expect(strategies[0].attempt).toHaveBeenCalled();
      expect(strategies[1].attempt).toHaveBeenCalled();
      expect(strategies[2].attempt).toHaveBeenCalled();
      expect(result).toEqual({ success: true });
      expect(errors).toHaveLength(2);
    });
  });
});

// Integration test example
describe('Error Handling Integration', () => {
  it('should handle complete error flow', async () => {
    const monitor = new ErrorMonitor();
    const errorReports: ErrorReport[] = [];
    
    monitor.onError(report => errorReports.push(report));
    
    // Simulate API error
    try {
      throw ErrorTestUtils.createApiError(401, 'Unauthorized');
    } catch (error) {
      monitor.captureError(error as Error, {
        endpoint: '/items',
        method: 'GET'
      });
    }
    
    expect(errorReports).toHaveLength(1);
    expect(errorReports[0].severity).toBe('low');
    expect(errorReports[0].context.statusCode).toBe(401);
  });
});
```

## Production Best Practices

### Comprehensive Production Error Handling Setup

```typescript
// Production configuration
interface ProductionErrorConfig {
  environment: 'production' | 'staging' | 'development';
  apiEndpoint: string;
  sentryDsn?: string;
  logLevel: 'error' | 'warn' | 'info' | 'debug';
  enableRetries: boolean;
  enableCircuitBreaker: boolean;
  enableRateLimiting: boolean;
  maxRetries: number;
  errorReportingEndpoint?: string;
}

class ProductionErrorHandler {
  private config: ProductionErrorConfig;
  private services: {
    retry?: ExponentialBackoffRetry;
    circuitBreaker?: CircuitBreaker;
    rateLimiter?: RateLimitManager;
    monitor?: ErrorMonitor;
    formatter?: UserErrorFormatter;
  } = {};
  
  constructor(config: ProductionErrorConfig) {
    this.config = config;
    this.initializeServices();
  }
  
  private initializeServices(): void {
    // Initialize retry logic
    if (this.config.enableRetries) {
      this.services.retry = new ExponentialBackoffRetry();
    }
    
    // Initialize circuit breaker
    if (this.config.enableCircuitBreaker) {
      this.services.circuitBreaker = new CircuitBreaker({
        failureThreshold: 5,
        resetTimeoutMs: 60000,
        monitoringPeriodMs: 120000,
        halfOpenMaxAttempts: 3
      });
    }
    
    // Initialize rate limiter
    if (this.config.enableRateLimiting) {
      this.services.rateLimiter = new RateLimitManager();
    }
    
    // Initialize error monitoring
    this.services.monitor = new ErrorMonitor({
      endpoint: this.config.errorReportingEndpoint,
      enableConsoleLog: this.config.environment !== 'production'
    });
    
    // Initialize error formatter
    this.services.formatter = new UserErrorFormatter();
    
    // Initialize Sentry if configured
    if (this.config.sentryDsn && typeof window !== 'undefined') {
      this.initializeSentry();
    }
  }
  
  private initializeSentry(): void {
    if ((window as any).Sentry) {
      (window as any).Sentry.init({
        dsn: this.config.sentryDsn,
        environment: this.config.environment,
        beforeSend: (event: any, hint: any) => {
          // Filter sensitive data
          if (event.request?.cookies) {
            delete event.request.cookies;
          }
          if (event.extra?.requestData) {
            event.extra.requestData = this.sanitizeData(event.extra.requestData);
          }
          return event;
        }
      });
    }
  }
  
  private sanitizeData(data: any): any {
    const sensitiveKeys = ['password', 'token', 'api_key', 'secret'];
    
    if (typeof data !== 'object' || data === null) {
      return data;
    }
    
    const sanitized = Array.isArray(data) ? [...data] : { ...data };
    
    for (const key in sanitized) {
      if (sensitiveKeys.some(sensitive => key.toLowerCase().includes(sensitive))) {
        sanitized[key] = '[REDACTED]';
      } else if (typeof sanitized[key] === 'object') {
        sanitized[key] = this.sanitizeData(sanitized[key]);
      }
    }
    
    return sanitized;
  }
  
  async execute<T>(
    operation: () => Promise<T>,
    options: {
      endpoint?: string;
      retryable?: boolean;
      fallback?: T | (() => T | Promise<T>);
      context?: Record<string, any>;
    } = {}
  ): Promise<T> {
    const startTime = Date.now();
    
    try {
      // Build operation chain
      let finalOperation = operation;
      
      // Add retry logic
      if (this.config.enableRetries && options.retryable !== false) {
        finalOperation = () => this.services.retry!.execute(operation, {
          maxRetries: this.config.maxRetries
        });
      }
      
      // Add circuit breaker
      if (this.config.enableCircuitBreaker) {
        const prevOperation = finalOperation;
        finalOperation = () => this.services.circuitBreaker!.execute(prevOperation);
      }
      
      // Add rate limiting
      if (this.config.enableRateLimiting && options.endpoint) {
        const prevOperation = finalOperation;
        finalOperation = () => this.services.rateLimiter!.execute(
          options.endpoint!,
          prevOperation
        );
      }
      
      // Execute operation
      const result = await finalOperation();
      
      // Log success metrics
      this.logMetrics('success', {
        endpoint: options.endpoint,
        duration: Date.now() - startTime
      });
      
      return result;
    } catch (error) {
      // Log error
      this.services.monitor?.captureError(error as Error, {
        ...options.context,
        endpoint: options.endpoint,
        duration: Date.now() - startTime
      });
      
      // Try fallback
      if (options.fallback !== undefined) {
        if (typeof options.fallback === 'function') {
          return await options.fallback();
        }
        return options.fallback;
      }
      
      // Re-throw with user-friendly message in production
      if (this.config.environment === 'production') {
        const userMessage = this.services.formatter!.formatError(error as Error);
        throw new Error(userMessage);
      }
      
      throw error;
    }
  }
  
  private logMetrics(type: 'success' | 'error', data: any): void {
    if (this.config.logLevel === 'debug' || 
        (this.config.logLevel === 'info' && type === 'error')) {
      console.log(`[Metrics] ${type}:`, data);
    }
  }
}

// Production setup example
const errorHandler = new ProductionErrorHandler({
  environment: process.env.NODE_ENV as any || 'development',
  apiEndpoint: process.env.REACT_APP_API_ENDPOINT!,
  sentryDsn: process.env.REACT_APP_SENTRY_DSN,
  logLevel: process.env.NODE_ENV === 'production' ? 'error' : 'debug',
  enableRetries: true,
  enableCircuitBreaker: true,
  enableRateLimiting: true,
  maxRetries: 3,
  errorReportingEndpoint: '/api/errors'
});

// Global API client wrapper
class DatoCMSClient {
  constructor(
    private client: any,
    private errorHandler: ProductionErrorHandler
  ) {}
  
  items = {
    find: (id: string) => this.errorHandler.execute(
      () => this.client.items.find(id),
      { endpoint: `/items/${id}`, retryable: true }
    ),
    
    create: (data: any) => this.errorHandler.execute(
      () => this.client.items.create(data),
      { endpoint: '/items', retryable: false }
    ),
    
    list: (params?: any) => this.errorHandler.execute(
      () => this.client.items.list(params),
      { 
        endpoint: '/items',
        retryable: true,
        fallback: async () => {
          console.warn('Using cached items list');
          return JSON.parse(localStorage.getItem('cached_items') || '[]');
        }
      }
    ),
    
    update: (id: string, data: any) => this.errorHandler.execute(
      () => this.client.items.update(id, data),
      { endpoint: `/items/${id}`, retryable: false }
    ),
    
    destroy: (id: string) => this.errorHandler.execute(
      () => this.client.items.destroy(id),
      { endpoint: `/items/${id}`, retryable: false }
    )
  };
  
  // Add similar wrappers for other resources
}

// Export configured client
export const datoCMSClient = new DatoCMSClient(
  originalClient,
  errorHandler
);
```

## Summary

This comprehensive guide covers all aspects of error handling for DatoCMS APIs:

1. **Error Types**: Complete understanding of ApiError structure and status codes
2. **Retry Strategies**: Exponential backoff and circuit breaker implementations
3. **Recovery Patterns**: Graceful degradation and progressive recovery
4. **Validation**: Client and server-side validation with user-friendly messages
5. **Network Handling**: Robust network error detection and recovery
6. **Rate Limiting**: Intelligent rate limit management and prediction
7. **Monitoring**: Comprehensive error tracking and reporting
8. **User Experience**: Localized, friendly error messages
9. **React Integration**: Error boundaries and async error handling
10. **Testing**: Complete error scenario testing patterns
11. **Production Setup**: Battle-tested production configuration

Remember to:
- Always handle errors explicitly
- Provide meaningful feedback to users
- Log errors with sufficient context
- Implement appropriate retry strategies
- Test error scenarios thoroughly
- Monitor error patterns in production

## Related Resources

- [API Endpoints Reference](../05-reference/api-endpoints.md)
- [Rate Limits and Quotas](../05-reference/limits-and-quotas.md)
- [Security Best Practices](../05-reference/security-best-practices.md)
- [Performance Optimization](./performance-optimization.md)