# Plugin Testing Guide

## Overview

Testing DatoCMS plugins presents unique challenges due to their iframe-based architecture and dependency on the CMS context. This guide covers strategies, tools, and best practices for comprehensive plugin testing.

## Testing Strategy

### 1. Testing Pyramid

Structure your tests following the testing pyramid principle:

```
         /\
        /  \  E2E Tests (Few)
       /    \
      /------\ Integration Tests (Some)
     /        \
    /----------\ Unit Tests (Many)
```

### 2. What to Test

Focus on testing:
- Business logic and data transformations
- User interactions and UI behavior
- API integrations and error handling
- Plugin lifecycle and state management
- Accessibility and performance

## Setting Up the Test Environment

### 1. Basic Test Setup

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/user-event": "^14.0.0",
    "@types/jest": "^29.0.0",
    "jest": "^29.0.0",
    "jest-environment-jsdom": "^29.0.0",
    "ts-jest": "^29.0.0"
  }
}
```

### 2. Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/test/setup.ts'],
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/test/**/*'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### 3. Test Setup File

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { TextEncoder, TextDecoder } from 'util';

// Polyfill for Node.js environment
global.TextEncoder = TextEncoder;
global.TextDecoder = TextDecoder as any;

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
} as any;

// Mock ResizeObserver
global.ResizeObserver = class ResizeObserver {
  constructor(callback: any) {}
  disconnect() {}
  observe() {}
  unobserve() {}
} as any;
```

## Mocking the SDK Context

### 1. Context Mock Factory

Create a flexible context mock factory:

```typescript
// src/test/mocks/createMockCtx.ts
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

export function createMockFieldExtensionCtx(
  overrides?: Partial<RenderFieldExtensionCtx>
): RenderFieldExtensionCtx {
  return {
    // Base properties
    mode: 'renderFieldExtension',
    plugin: {
      id: 'test-plugin',
      type: 'plugin',
      attributes: {
        name: 'Test Plugin',
        parameters: {},
        plugin_type: 'field_addon',
        field_types: ['string'],
        addon: false
      }
    },
    currentUser: {
      id: 'user-1',
      type: 'user',
      attributes: {
        email: 'test@example.com',
        first_name: 'Test',
        last_name: 'User'
      }
    },
    environment: 'main',
    site: {
      id: 'site-1',
      type: 'site',
      attributes: {
        name: 'Test Site',
        locales: ['en'],
        timezone: 'UTC'
      }
    },
    
    // Field extension specific
    fieldPath: 'title',
    field: {
      id: 'field-1',
      type: 'field',
      attributes: {
        label: 'Title',
        api_key: 'title',
        field_type: 'string',
        validators: { required: {} }
      }
    },
    
    // Item form properties
    formValues: { title: 'Test Value' },
    itemType: {
      id: 'model-1',
      type: 'item_type',
      attributes: {
        name: 'Article',
        api_key: 'article'
      }
    },
    
    // Methods
    setFieldValue: jest.fn(),
    toggleField: jest.fn(),
    disableField: jest.fn(),
    enableField: jest.fn(),
    addFieldError: jest.fn(),
    clearFieldErrors: jest.fn(),
    
    // Sizing utilities
    startAutoResizer: jest.fn(),
    stopAutoResizer: jest.fn(),
    updateHeight: jest.fn(),
    isAutoResizerActive: jest.fn(() => false),
    setHeight: jest.fn(),
    
    // UI methods
    notice: jest.fn(),
    alert: jest.fn(),
    openModal: jest.fn(),
    openConfirm: jest.fn(),
    navigateTo: jest.fn(),
    
    // Apply overrides
    ...overrides
  } as any as RenderFieldExtensionCtx;
}
```

### 2. Testing Different Context States

```typescript
// src/test/mocks/contextStates.ts
export const contextStates = {
  newItem: (overrides = {}) => createMockFieldExtensionCtx({
    itemId: null,
    item: null,
    itemStatus: 'new',
    formValues: {},
    ...overrides
  }),
  
  publishedItem: (overrides = {}) => createMockFieldExtensionCtx({
    itemId: '123',
    itemStatus: 'published',
    lastPublishedVersion: {
      id: '123',
      type: 'item',
      attributes: { title: 'Published Title' }
    },
    ...overrides
  }),
  
  localized: (locale = 'it', overrides = {}) => createMockFieldExtensionCtx({
    locale,
    locales: ['en', 'it', 'de'],
    formValues: {
      title: { en: 'English', it: 'Italiano', de: 'Deutsch' }
    },
    ...overrides
  }),
  
  disabled: (overrides = {}) => createMockFieldExtensionCtx({
    disabled: true,
    ...overrides
  })
};
```

## Unit Testing

### 1. Testing Pure Functions

```typescript
// src/utils/validation.ts
export function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

export function formatCurrency(amount: number, currency = 'USD'): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(amount);
}

// src/utils/validation.test.ts
import { validateEmail, formatCurrency } from './validation';

describe('validateEmail', () => {
  it('should validate correct email addresses', () => {
    expect(validateEmail('test@example.com')).toBe(true);
    expect(validateEmail('user.name+tag@example.co.uk')).toBe(true);
  });
  
  it('should reject invalid email addresses', () => {
    expect(validateEmail('invalid')).toBe(false);
    expect(validateEmail('@example.com')).toBe(false);
    expect(validateEmail('test@')).toBe(false);
  });
});

describe('formatCurrency', () => {
  it('should format USD amounts correctly', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
    expect(formatCurrency(0)).toBe('$0.00');
  });
  
  it('should support different currencies', () => {
    expect(formatCurrency(1234.56, 'EUR')).toContain('1,234.56');
    expect(formatCurrency(1234.56, 'GBP')).toContain('1,234.56');
  });
});
```

### 2. Testing React Components

```typescript
// src/components/FieldEditor.tsx
import React, { useState } from 'react';
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

interface FieldEditorProps {
  ctx: RenderFieldExtensionCtx;
}

export function FieldEditor({ ctx }: FieldEditorProps) {
  const [value, setValue] = useState(ctx.formValues[ctx.fieldPath] || '');
  const [error, setError] = useState('');
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const newValue = e.target.value;
    setValue(newValue);
    
    if (ctx.field.attributes.validators?.required && !newValue) {
      setError('This field is required');
      ctx.addFieldError(ctx.fieldPath, 'Required field');
    } else {
      setError('');
      ctx.clearFieldErrors(ctx.fieldPath);
      ctx.setFieldValue(ctx.fieldPath, newValue);
    }
  };
  
  return (
    <div>
      <input
        value={value}
        onChange={handleChange}
        disabled={ctx.disabled}
        aria-label={ctx.field.attributes.label}
        aria-invalid={!!error}
        aria-describedby={error ? 'error-message' : undefined}
      />
      {error && (
        <span id="error-message" role="alert">
          {error}
        </span>
      )}
    </div>
  );
}

// src/components/FieldEditor.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { FieldEditor } from './FieldEditor';
import { createMockFieldExtensionCtx } from '../test/mocks/createMockCtx';

describe('FieldEditor', () => {
  it('should render with initial value', () => {
    const ctx = createMockFieldExtensionCtx({
      formValues: { title: 'Initial Value' }
    });
    
    render(<FieldEditor ctx={ctx} />);
    
    const input = screen.getByLabelText('Title');
    expect(input).toHaveValue('Initial Value');
  });
  
  it('should update field value on change', async () => {
    const user = userEvent.setup();
    const ctx = createMockFieldExtensionCtx();
    
    render(<FieldEditor ctx={ctx} />);
    
    const input = screen.getByLabelText('Title');
    await user.clear(input);
    await user.type(input, 'New Value');
    
    expect(ctx.setFieldValue).toHaveBeenCalledWith('title', 'New Value');
  });
  
  it('should show validation error for required field', async () => {
    const user = userEvent.setup();
    const ctx = createMockFieldExtensionCtx({
      field: {
        ...createMockFieldExtensionCtx().field,
        attributes: {
          ...createMockFieldExtensionCtx().field.attributes,
          validators: { required: {} }
        }
      }
    });
    
    render(<FieldEditor ctx={ctx} />);
    
    const input = screen.getByLabelText('Title');
    await user.clear(input);
    
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('This field is required');
      expect(ctx.addFieldError).toHaveBeenCalledWith('title', 'Required field');
    });
  });
  
  it('should be disabled when context is disabled', () => {
    const ctx = createMockFieldExtensionCtx({ disabled: true });
    
    render(<FieldEditor ctx={ctx} />);
    
    expect(screen.getByLabelText('Title')).toBeDisabled();
  });
});
```

## Integration Testing

### 1. Testing Hook Interactions

```typescript
// src/hooks/useFieldValidation.ts
import { useEffect, useState } from 'react';
import { RenderFieldExtensionCtx } from 'datocms-plugin-sdk';

export function useFieldValidation(ctx: RenderFieldExtensionCtx) {
  const [isValid, setIsValid] = useState(true);
  const [errors, setErrors] = useState<string[]>([]);
  
  useEffect(() => {
    const validateField = async () => {
      const value = ctx.formValues[ctx.fieldPath];
      const validators = ctx.field.attributes.validators || {};
      const newErrors: string[] = [];
      
      if (validators.required && !value) {
        newErrors.push('Field is required');
      }
      
      if (validators.length && typeof value === 'string') {
        if (validators.length.min && value.length < validators.length.min) {
          newErrors.push(`Minimum length is ${validators.length.min}`);
        }
        if (validators.length.max && value.length > validators.length.max) {
          newErrors.push(`Maximum length is ${validators.length.max}`);
        }
      }
      
      setErrors(newErrors);
      setIsValid(newErrors.length === 0);
      
      if (newErrors.length > 0) {
        ctx.addFieldError(ctx.fieldPath, newErrors[0]);
      } else {
        ctx.clearFieldErrors(ctx.fieldPath);
      }
    };
    
    validateField();
  }, [ctx.formValues[ctx.fieldPath]]);
  
  return { isValid, errors };
}

// src/hooks/useFieldValidation.test.ts
import { renderHook } from '@testing-library/react';
import { useFieldValidation } from './useFieldValidation';
import { createMockFieldExtensionCtx } from '../test/mocks/createMockCtx';

describe('useFieldValidation', () => {
  it('should validate required fields', () => {
    const ctx = createMockFieldExtensionCtx({
      formValues: { title: '' },
      field: {
        ...createMockFieldExtensionCtx().field,
        attributes: {
          ...createMockFieldExtensionCtx().field.attributes,
          validators: { required: {} }
        }
      }
    });
    
    const { result } = renderHook(() => useFieldValidation(ctx));
    
    expect(result.current.isValid).toBe(false);
    expect(result.current.errors).toContain('Field is required');
    expect(ctx.addFieldError).toHaveBeenCalledWith('title', 'Field is required');
  });
  
  it('should validate length constraints', () => {
    const ctx = createMockFieldExtensionCtx({
      formValues: { title: 'ab' },
      field: {
        ...createMockFieldExtensionCtx().field,
        attributes: {
          ...createMockFieldExtensionCtx().field.attributes,
          validators: {
            length: { min: 3, max: 10 }
          }
        }
      }
    });
    
    const { result } = renderHook(() => useFieldValidation(ctx));
    
    expect(result.current.isValid).toBe(false);
    expect(result.current.errors).toContain('Minimum length is 3');
  });
});
```

### 2. Testing Async Operations

```typescript
// src/services/api.ts
export class ApiService {
  constructor(private ctx: RenderFieldExtensionCtx) {}
  
  async fetchSuggestions(query: string): Promise<string[]> {
    const response = await fetch(`/api/suggestions?q=${query}`, {
      headers: {
        'Authorization': `Bearer ${this.ctx.currentUserAccessToken}`
      }
    });
    
    if (!response.ok) {
      throw new Error('Failed to fetch suggestions');
    }
    
    return response.json();
  }
}

// src/services/api.test.ts
import { ApiService } from './api';
import { createMockFieldExtensionCtx } from '../test/mocks/createMockCtx';

// Mock fetch
global.fetch = jest.fn();

describe('ApiService', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  it('should fetch suggestions successfully', async () => {
    const mockSuggestions = ['suggestion1', 'suggestion2'];
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockSuggestions
    });
    
    const ctx = createMockFieldExtensionCtx({
      currentUserAccessToken: 'test-token'
    });
    const api = new ApiService(ctx);
    
    const suggestions = await api.fetchSuggestions('test');
    
    expect(global.fetch).toHaveBeenCalledWith(
      '/api/suggestions?q=test',
      expect.objectContaining({
        headers: {
          'Authorization': 'Bearer test-token'
        }
      })
    );
    expect(suggestions).toEqual(mockSuggestions);
  });
  
  it('should handle fetch errors', async () => {
    (global.fetch as jest.Mock).mockResolvedValueOnce({
      ok: false,
      status: 500
    });
    
    const ctx = createMockFieldExtensionCtx();
    const api = new ApiService(ctx);
    
    await expect(api.fetchSuggestions('test')).rejects.toThrow(
      'Failed to fetch suggestions'
    );
  });
});
```

## E2E Testing

### 1. Playwright Setup

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  timeout: 30000,
  retries: 2,
  use: {
    baseURL: 'http://localhost:3000',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
});
```

### 2. E2E Test Example

```typescript
// e2e/field-extension.spec.ts
import { test, expect, Page } from '@playwright/test';

class PluginTestHelper {
  constructor(private page: Page) {}
  
  async loadPlugin(config: any) {
    // Navigate to test harness
    await this.page.goto('/test-harness');
    
    // Configure plugin
    await this.page.evaluate((config) => {
      window.testHarness.loadPlugin(config);
    }, config);
    
    // Wait for plugin to load
    await this.page.waitForSelector('iframe#plugin-frame');
  }
  
  async getPluginFrame() {
    const frame = this.page.frame({ name: 'plugin-frame' });
    if (!frame) throw new Error('Plugin frame not found');
    return frame;
  }
}

test.describe('Field Extension', () => {
  test('should validate required fields', async ({ page }) => {
    const helper = new PluginTestHelper(page);
    
    await helper.loadPlugin({
      fieldExtensionId: 'myExtension',
      field: {
        attributes: {
          label: 'Title',
          validators: { required: {} }
        }
      },
      formValues: { title: '' }
    });
    
    const frame = await helper.getPluginFrame();
    
    // Clear the field
    await frame.locator('input').clear();
    
    // Check for error message
    await expect(frame.locator('[role="alert"]')).toContainText('required');
  });
  
  test('should handle value updates', async ({ page }) => {
    const helper = new PluginTestHelper(page);
    
    await helper.loadPlugin({
      fieldExtensionId: 'myExtension',
      formValues: { title: 'Initial' }
    });
    
    const frame = await helper.getPluginFrame();
    
    // Type new value
    await frame.locator('input').fill('Updated Value');
    
    // Verify the value was updated
    const calls = await page.evaluate(() => {
      return window.testHarness.getMethodCalls('setFieldValue');
    });
    
    expect(calls).toContainEqual(['title', 'Updated Value']);
  });
});
```

## Testing Best Practices

### 1. Test Organization

```
src/
├── components/
│   ├── FieldEditor.tsx
│   └── FieldEditor.test.tsx
├── hooks/
│   ├── useValidation.ts
│   └── useValidation.test.ts
├── services/
│   ├── api.ts
│   └── api.test.ts
└── test/
    ├── setup.ts
    ├── mocks/
    │   ├── createMockCtx.ts
    │   └── contextStates.ts
    └── utils/
        └── testHelpers.ts
```

### 2. Test Utilities

```typescript
// src/test/utils/testHelpers.ts
import { RenderOptions, render } from '@testing-library/react';
import { ReactElement } from 'react';

// Custom render with providers
export function renderWithContext(
  ui: ReactElement,
  options?: RenderOptions
) {
  return render(ui, {
    wrapper: ({ children }) => (
      <div data-testid="plugin-container">
        {children}
      </div>
    ),
    ...options
  });
}

// Wait for async operations
export async function waitForAsync() {
  return new Promise(resolve => setTimeout(resolve, 0));
}

// Mock timer helpers
export const mockTimers = {
  setup() {
    jest.useFakeTimers();
    return {
      cleanup: () => jest.useRealTimers()
    };
  },
  
  async runAllTimers() {
    jest.runAllTimers();
    await waitForAsync();
  }
};
```

### 3. Accessibility Testing

```typescript
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

describe('FieldEditor Accessibility', () => {
  it('should have no accessibility violations', async () => {
    const { container } = render(
      <FieldEditor ctx={createMockFieldExtensionCtx()} />
    );
    
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  it('should have proper ARIA attributes', () => {
    const ctx = createMockFieldExtensionCtx({
      field: {
        ...createMockFieldExtensionCtx().field,
        attributes: {
          ...createMockFieldExtensionCtx().field.attributes,
          label: 'Email Address',
          hint: 'Enter your email'
        }
      }
    });
    
    render(<FieldEditor ctx={ctx} />);
    
    const input = screen.getByLabelText('Email Address');
    expect(input).toHaveAttribute('aria-describedby');
    expect(screen.getByText('Enter your email')).toBeInTheDocument();
  });
});
```

## Performance Testing

```typescript
// src/test/performance.test.ts
describe('Performance', () => {
  it('should render quickly', async () => {
    const start = performance.now();
    
    render(<FieldEditor ctx={createMockFieldExtensionCtx()} />);
    
    const renderTime = performance.now() - start;
    expect(renderTime).toBeLessThan(100); // 100ms threshold
  });
  
  it('should handle large datasets efficiently', async () => {
    const largeDataset = Array.from({ length: 10000 }, (_, i) => ({
      id: i,
      name: `Item ${i}`
    }));
    
    const start = performance.now();
    
    render(
      <VirtualizedList 
        items={largeDataset}
        ctx={createMockFieldExtensionCtx()}
      />
    );
    
    const renderTime = performance.now() - start;
    expect(renderTime).toBeLessThan(200);
    
    // Verify only visible items are rendered
    const renderedItems = screen.getAllByTestId('list-item');
    expect(renderedItems.length).toBeLessThan(50);
  });
});
```

## Continuous Integration

```yaml
# .github/workflows/test.yml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run unit tests
        run: npm test -- --coverage
      
      - name: Run E2E tests
        run: |
          npm run build
          npm run test:e2e
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
      
      - name: Upload test artifacts
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: test-artifacts
          path: |
            coverage/
            playwright-report/
            test-results/
```

## Testing Checklist

- [ ] Unit tests for all utility functions
- [ ] Component tests with user interactions
- [ ] Integration tests for API calls
- [ ] E2E tests for critical paths
- [ ] Accessibility tests (jest-axe)
- [ ] Performance tests for large datasets
- [ ] Error boundary testing
- [ ] Loading state testing
- [ ] Localization testing
- [ ] Cross-browser testing
- [ ] Memory leak testing
- [ ] 80%+ code coverage

## Related Documentation

- [Plugin Memory Management](./plugin-memory-management.md)
- [Plugin Performance](./plugin-performance.md)
- [Plugin Lifecycle](../01-core-concepts/plugin-lifecycle.md)