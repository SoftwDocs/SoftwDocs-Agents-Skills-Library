---
name: auto-test-architect
description: Comprehensive testing strategy for Next.js and FastAPI. Unit tests, integration tests, E2E with Playwright, visual regression, and automated CI/CD testing pipelines.
tags: [testing, playwright, jest, vitest, cypress, ci-cd, coverage]
version: 2.0.0
author: SoftwDocs
---

# Auto Test Architect

## Overview

A comprehensive testing skill that establishes a multi-layered testing strategy for production-grade applications. Covers unit testing, integration testing, end-to-end testing, visual regression, and CI/CD automation.

## Testing Pyramid

```
                    /\
                   /  \
                  / E2E \        <- Few tests, high confidence
                 /________\         (Playwright/Cypress)
                /          \
               / Integration \   <- Medium tests, API/DB testing
              /________________\      (Vitest + MSW)
             /                  \
            /      Unit Tests     \ <- Many tests, fast feedback
           /________________________\   (Vitest/Jest)
```

## Testing Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    TESTING LAYERS                               │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 1: E2E TESTS                                             │
│  ├─ Playwright (Primary)                                        │
│  ├─ User flow testing                                           │
│  ├─ Cross-browser testing                                       │
│  ├─ Visual regression                                           │
│  └─ Performance testing                                         │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 2: INTEGRATION TESTS                                     │
│  ├─ API route testing                                           │
│  ├─ Database integration                                        │
│  ├─ Component integration                                       │
│  └─ Mock Service Worker (MSW)                                   │
├─────────────────────────────────────────────────────────────────┤
│  LAYER 3: UNIT TESTS                                            │
│  ├─ Component unit tests (React Testing Library)                │
│  ├─ Hook tests                                                  │
│  ├─ Utility function tests                                      │
│  ├─ Server action tests                                         │
│  └─ Snapshot tests                                              │
└─────────────────────────────────────────────────────────────────┘
```

## 1. Unit Testing (Vitest + React Testing Library)

### Setup Configuration
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.*',
      ],
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80,
        },
      },
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './'),
    },
  },
});
```

### Test Setup File
```typescript
// tests/setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { vi } from 'vitest';

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock next/navigation
vi.mock('next/navigation', () => ({
  useRouter: vi.fn(() => ({
    push: vi.fn(),
    replace: vi.fn(),
    refresh: vi.fn(),
    back: vi.fn(),
    forward: vi.fn(),
    prefetch: vi.fn(),
  })),
  useSearchParams: vi.fn(() => new URLSearchParams()),
  usePathname: vi.fn(() => '/'),
}));

// Mock next/headers
vi.mock('next/headers', () => ({
  cookies: vi.fn(() => ({
    get: vi.fn(),
    set: vi.fn(),
    delete: vi.fn(),
  })),
  headers: vi.fn(() => new Headers()),
}));
```

### Component Unit Tests
```typescript
// components/__tests__/button.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/ui/button';

describe('Button', () => {
  it('renders with default variant', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('handles click events', () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when loading', () => {
    render(<Button isLoading>Loading</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });

  it('applies correct variant classes', () => {
    const { rerender } = render(<Button variant="primary">Primary</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-blue-600');

    rerender(<Button variant="secondary">Secondary</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-gray-200');

    rerender(<Button variant="danger">Danger</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-red-600');
  });

  it('renders as different element when asChild is true', () => {
    render(
      <Button asChild>
        <a href="/link">Link Button</a>
      </Button>
    );
    expect(screen.getByRole('link')).toHaveTextContent('Link Button');
  });
});
```

### Hook Tests
```typescript
// hooks/__tests__/use-local-storage.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { useLocalStorage } from '@/hooks/use-local-storage';

describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
    vi.clearAllMocks();
  });

  it('returns initial value when localStorage is empty', () => {
    const { result } = renderHook(() => useLocalStorage('key', 'default'));
    expect(result.current[0]).toBe('default');
  });

  it('reads existing value from localStorage', () => {
    localStorage.setItem('key', JSON.stringify('stored'));
    const { result } = renderHook(() => useLocalStorage('key', 'default'));
    expect(result.current[0]).toBe('stored');
  });

  it('updates localStorage when value changes', () => {
    const { result } = renderHook(() => useLocalStorage('key', 'initial'));
    
    act(() => {
      result.current[1]('updated');
    });

    expect(result.current[0]).toBe('updated');
    expect(localStorage.getItem('key')).toBe(JSON.stringify('updated'));
  });

  it('handles JSON parse errors gracefully', () => {
    localStorage.setItem('key', 'invalid json');
    const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});
    
    const { result } = renderHook(() => useLocalStorage('key', 'fallback'));
    expect(result.current[0]).toBe('fallback');
    
    consoleSpy.mockRestore();
  });
});
```

### Server Action Tests
```typescript
// lib/actions/__tests__/auth.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { login, register } from '@/lib/actions/auth';

// Mock prisma
vi.mock('@/lib/prisma', () => ({
  prisma: {
    user: {
      findUnique: vi.fn(),
      create: vi.fn(),
    },
  },
}));

describe('Auth Actions', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('login', () => {
    it('returns error for invalid credentials', async () => {
      const { prisma } = await import('@/lib/prisma');
      vi.mocked(prisma.user.findUnique).mockResolvedValue(null);

      const result = await login({ email: 'test@test.com', password: 'wrong' });
      
      expect(result.success).toBe(false);
      expect(result.error).toBe('Invalid credentials');
    });

    it('returns success with tokens for valid credentials', async () => {
      const { prisma } = await import('@/lib/prisma');
      vi.mocked(prisma.user.findUnique).mockResolvedValue({
        id: '1',
        email: 'test@test.com',
        password: 'hashed_password',
      });

      // Mock bcrypt compare
      vi.mock('bcryptjs', () => ({
        compare: vi.fn().mockResolvedValue(true),
      }));

      const result = await login({ email: 'test@test.com', password: 'correct' });
      
      expect(result.success).toBe(true);
      expect(result.data).toHaveProperty('accessToken');
      expect(result.data).toHaveProperty('refreshToken');
    });
  });

  describe('register', () => {
    it('returns error if email already exists', async () => {
      const { prisma } = await import('@/lib/prisma');
      vi.mocked(prisma.user.findUnique).mockResolvedValue({ id: '1' } as any);

      const result = await register({
        email: 'exists@test.com',
        password: 'password123',
        name: 'Test',
      });

      expect(result.success).toBe(false);
      expect(result.error).toBe('Email already registered');
    });

    it('creates user with hashed password', async () => {
      const { prisma } = await import('@/lib/prisma');
      vi.mocked(prisma.user.findUnique).mockResolvedValue(null);
      vi.mocked(prisma.user.create).mockResolvedValue({
        id: '1',
        email: 'new@test.com',
        name: 'Test',
      } as any);

      const result = await register({
        email: 'new@test.com',
        password: 'password123',
        name: 'Test',
      });

      expect(result.success).toBe(true);
      expect(prisma.user.create).toHaveBeenCalledWith(
        expect.objectContaining({
          data: expect.objectContaining({
            email: 'new@test.com',
            name: 'Test',
            // Password should be hashed
            password: expect.not.stringMatching('password123'),
          }),
        })
      );
    });
  });
});
```

## 2. Integration Testing

### API Route Testing
```typescript
// app/api/users/__tests__/route.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { GET, POST } from '../route';
import { NextRequest } from 'next/server';

// Mock database
vi.mock('@/lib/prisma', () => ({
  prisma: {
    user: {
      findMany: vi.fn(),
      create: vi.fn(),
    },
  },
}));

describe('Users API', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('GET /api/users', () => {
    it('returns list of users', async () => {
      const { prisma } = await import('@/lib/prisma');
      const mockUsers = [
        { id: '1', email: 'user1@test.com', name: 'User 1' },
        { id: '2', email: 'user2@test.com', name: 'User 2' },
      ];
      vi.mocked(prisma.user.findMany).mockResolvedValue(mockUsers);

      const response = await GET();
      const data = await response.json();

      expect(response.status).toBe(200);
      expect(data).toEqual(mockUsers);
    });

    it('handles database errors', async () => {
      const { prisma } = await import('@/lib/prisma');
      vi.mocked(prisma.user.findMany).mockRejectedValue(new Error('DB Error'));

      const response = await GET();
      
      expect(response.status).toBe(500);
    });
  });

  describe('POST /api/users', () => {
    it('creates a new user', async () => {
      const { prisma } = await import('@/lib/prisma');
      const newUser = { id: '3', email: 'new@test.com', name: 'New User' };
      vi.mocked(prisma.user.create).mockResolvedValue(newUser);

      const request = new NextRequest('http://localhost/api/users', {
        method: 'POST',
        body: JSON.stringify({ email: 'new@test.com', name: 'New User' }),
      });

      const response = await POST(request);
      const data = await response.json();

      expect(response.status).toBe(201);
      expect(data).toEqual(newUser);
    });

    it('validates required fields', async () => {
      const request = new NextRequest('http://localhost/api/users', {
        method: 'POST',
        body: JSON.stringify({ name: 'No Email' }),
      });

      const response = await POST(request);
      const data = await response.json();

      expect(response.status).toBe(400);
      expect(data.error).toContain('email');
    });
  });
});
```

### MSW (Mock Service Worker) Setup
```typescript
// tests/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // Mock external API
  http.get('https://api.external.com/users', () => {
    return HttpResponse.json([
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' },
    ]);
  }),

  // Mock with query params
  http.get('https://api.external.com/search', ({ request }) => {
    const url = new URL(request.url);
    const query = url.searchParams.get('q');
    
    return HttpResponse.json({
      results: [{ id: 1, title: `Result for ${query}` }],
    });
  }),

  // Mock POST request
  http.post('https://api.external.com/webhook', async ({ request }) => {
    const body = await request.json();
    
    return HttpResponse.json({
      id: 'webhook-123',
      received: body,
      status: 'processed',
    });
  }),

  // Mock error response
  http.get('https://api.external.com/error', () => {
    return new HttpResponse(null, { status: 500 });
  }),
];
```

```typescript
// tests/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// tests/setup.ts (updated)
import { server } from './mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## 3. E2E Testing (Playwright)

### Playwright Configuration
```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['list'],
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### E2E Test Examples
```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('user can login with valid credentials', async ({ page }) => {
    await page.fill('[data-testid="email-input"]', 'user@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    await page.click('[data-testid="login-button"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test('shows error for invalid credentials', async ({ page }) => {
    await page.fill('[data-testid="email-input"]', 'wrong@example.com');
    await page.fill('[data-testid="password-input"]', 'wrongpassword');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('[data-testid="error-message"]')).toContainText('Invalid credentials');
    await expect(page).toHaveURL('/login');
  });

  test('validates email format', async ({ page }) => {
    await page.fill('[data-testid="email-input"]', 'invalid-email');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('[data-testid="email-error"]')).toContainText('Invalid email');
  });

  test('redirects to login when accessing protected route', async ({ page }) => {
    await page.goto('/dashboard');
    await expect(page).toHaveURL('/login?redirect=/dashboard');
  });
});
```

```typescript
// e2e/checkout.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Checkout Flow', () => {
  test('complete purchase flow', async ({ page }) => {
    // Add items to cart
    await page.goto('/products');
    await page.click('[data-testid="add-to-cart-1"]');
    await page.click('[data-testid="add-to-cart-2"]');

    // Go to cart
    await page.click('[data-testid="cart-icon"]');
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('2');

    // Proceed to checkout
    await page.click('[data-testid="checkout-button"]');
    await expect(page).toHaveURL('/checkout');

    // Fill shipping info
    await page.fill('[name="name"]', 'John Doe');
    await page.fill('[name="address"]', '123 Main St');
    await page.fill('[name="city"]', 'New York');
    await page.selectOption('[name="country"]', 'US');

    // Fill payment (mock)
    await page.fill('[name="cardNumber"]', '4242424242424242');
    await page.fill('[name="expiry"]', '12/25');
    await page.fill('[name="cvc"]', '123');

    // Submit order
    await page.click('[data-testid="place-order-button"]');

    // Verify success
    await expect(page).toHaveURL('/order-confirmation');
    await expect(page.locator('[data-testid="order-success"]')).toBeVisible();
    await expect(page.locator('[data-testid="order-number"]')).toContainText(/ORD-\d+/);
  });

  test('calculates totals correctly', async ({ page }) => {
    await page.goto('/products');
    
    // Add $50 item
    await page.click('[data-testid="add-to-cart-1"]');
    
    // Check cart subtotal
    await page.click('[data-testid="cart-icon"]');
    await expect(page.locator('[data-testid="subtotal"]')).toHaveText('$50.00');
    
    // Add another $30 item
    await page.goto('/products');
    await page.click('[data-testid="add-to-cart-2"]');
    
    // Verify updated total
    await page.click('[data-testid="cart-icon"]');
    await expect(page.locator('[data-testid="subtotal"]')).toHaveText('$80.00');
    await expect(page.locator('[data-testid="tax"]')).toHaveText('$8.00'); // 10% tax
    await expect(page.locator('[data-testid="total"]')).toHaveText('$88.00');
  });
});
```

### Visual Regression Testing
```typescript
// e2e/visual.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Visual Regression', () => {
  test('homepage matches snapshot', async ({ page }) => {
    await page.goto('/');
    await page.waitForLoadState('networkidle');
    
    await expect(page).toHaveScreenshot('homepage.png', {
      fullPage: true,
      threshold: 0.2,
    });
  });

  test('dark mode toggle works', async ({ page }) => {
    await page.goto('/');
    
    // Light mode screenshot
    await expect(page).toHaveScreenshot('homepage-light.png');
    
    // Toggle dark mode
    await page.click('[data-testid="theme-toggle"]');
    
    // Dark mode screenshot
    await expect(page).toHaveScreenshot('homepage-dark.png');
  });

  test('responsive layouts', async ({ page }) => {
    const viewports = [
      { width: 375, height: 667, name: 'mobile' },
      { width: 768, height: 1024, name: 'tablet' },
      { width: 1920, height: 1080, name: 'desktop' },
    ];

    for (const viewport of viewports) {
      await page.setViewportSize({ width: viewport.width, height: viewport.height });
      await page.goto('/');
      
      await expect(page).toHaveScreenshot(`homepage-${viewport.name}.png`);
    }
  });
});
```

## 4. Performance Testing

### Lighthouse CI
```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI
on: [push]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Use Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build
        run: npm run build
        
      - name: Run Lighthouse CI
        run: |
          npm install -g @lhci/cli
          lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

```json
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      startServerCommand: 'npm run start',
      url: ['http://localhost:3000/'],
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        'categories:performance': ['warn', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 1 }],
        'categories:best-practices': ['warn', { minScore: 0.9 }],
        'categories:seo': ['warn', { minScore: 0.9 }],
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

## 5. CI/CD Integration

### GitHub Actions Workflow
```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run unit tests
        run: npm run test:unit
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
          
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Setup database
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
          
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Install Playwright
        run: npx playwright install --with-deps
        
      - name: Build application
        run: npm run build
        
      - name: Run E2E tests
        run: npm run test:e2e
        
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

## Testing Checklist

### Unit Tests
- [ ] All components have basic render tests
- [ ] Event handlers are tested
- [ ] Props and variants are validated
- [ ] Hooks have comprehensive tests
- [ ] Utilities have edge case coverage
- [ ] Coverage threshold >= 80%

### Integration Tests
- [ ] API routes tested with mocked DB
- [ ] External APIs mocked with MSW
- [ ] Database operations tested
- [ ] Error handling verified

### E2E Tests
- [ ] Critical user flows covered
- [ ] Authentication flows tested
- [ ] Payment/checkout flows tested
- [ ] Cross-browser testing configured
- [ ] Mobile responsiveness tested
- [ ] Visual regression baseline set

### CI/CD
- [ ] Tests run on every PR
- [ ] Coverage reports generated
- [ ] E2E tests run in CI
- [ ] Lighthouse scores monitored
- [ ] Failed tests block deployment

## Test Commands

```bash
# Run all tests
npm test

# Unit tests only
npm run test:unit

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e

# E2E with UI mode
npm run test:e2e:ui

# Coverage report
npm run test:coverage

# Watch mode
npm run test:watch
```
