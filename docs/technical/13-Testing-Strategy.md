# 13. Testing Strategy — Jejakawan

## Overview

Dokumen ini mendefinisikan strategi pengujian menyeluruh untuk platform Jejakawan, mencakup unit test, integration test, end-to-end test, dan load test. Target: **80% code coverage minimum** pada semua module kritis.

---

## 1. Tech Stack Testing

| Layer | Tool | Purpose | Config File |
|-------|------|---------|-------------|
| Unit Test | Vitest | Component & utility testing | `vitest.config.ts` |
| Integration | Vitest + Supabase Local | DB/Auth/Storage flows | `vitest.config.integration.ts` |
| E2E Test | Playwright | Full user journey | `playwright.config.ts` |
| Load Test | k6 | Performance & stress testing | `k6/scripts/` |
| Mocking | MSW (Mock Service Worker) | API mocking | `src/mocks/` |
| Coverage | v8 (built-in Vitest) | Coverage reporting | `.c8rc.json` |
| CI | GitHub Actions | Automated pipeline | `.github/workflows/test.yml` |

---

## 2. Unit Testing (Vitest)

### 2.1 Scope & Coverage Targets

```
Target Coverage:
├── Statements: ≥ 80%
├── Branches:   ≥ 75%
├── Functions:  ≥ 80%
└── Lines:      ≥ 80%

Critical Modules (≥ 90%):
├── src/lib/payment/       → 95% (Midtrans integration)
├── src/lib/auth/          → 95% (Supabase Auth flows)
├── src/lib/matching/      → 90% (Trip matching algorithm)
├── src/lib/gamification/  → 90% (XP, badges, leaderboard)
└── src/lib/subscription/  → 90% (Plan management)
```

### 2.2 Struktur Test File

```
src/
├── components/
│   ├── TripCard.tsx
│   └── TripCard.test.tsx          ← co-located test
├── lib/
│   ├── matching/
│   │   ├── algorithm.ts
│   │   ├── algorithm.test.ts
│   │   └── __fixtures__/          ← test data
│   │       ├── users.json
│   │       └── destinations.json
│   ├── payment/
│   │   ├── midtrans.ts
│   │   └── midtrans.test.ts
│   └── utils/
│       ├── date.ts
│       └── date.test.ts
├── hooks/
│   ├── useTripMatch.ts
│   └── useTripMatch.test.ts
└── pages/
    ├── Home.tsx
    └── Home.test.tsx
```

### 2.3 Vitest Configuration

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
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    exclude: ['src/**/*.integration.test.ts', 'src/**/*.e2e.test.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      reportsDirectory: './coverage',
      include: ['src/**/*.{ts,tsx}'],
      exclude: [
        'src/**/*.test.{ts,tsx}',
        'src/**/*.d.ts',
        'src/main.tsx',
        'src/vite-env.d.ts',
      ],
      thresholds: {
        statements: 80,
        branches: 75,
        functions: 80,
        lines: 80,
      },
    },
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### 2.4 Test Setup

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, beforeAll, afterAll } from 'vitest';
import { server } from '../mocks/server';

// Start MSW before all tests
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));

// Reset handlers after each test (remove one-time overrides)
afterEach(() => {
  server.resetHandlers();
  cleanup();
});

// Close server after all tests
afterAll(() => server.close());
```

### 2.5 Contoh Unit Test — Trip Matching Algorithm

```typescript
// src/lib/matching/algorithm.test.ts
import { describe, it, expect } from 'vitest';
import { matchTrips, calculateCompatibilityScore } from './algorithm';
import type { UserProfile, Destination } from '@/types';
import { mockUsers, mockDestinations } from './__fixtures__';

describe('Trip Matching Algorithm', () => {
  describe('calculateCompatibilityScore', () => {
    it('returns 1.0 for identical preferences', () => {
      const userA = mockUsers.adventureSeeker;
      const userB = mockUsers.adventureSeeker;
      const score = calculateCompatibilityScore(userA, userB);
      expect(score).toBe(1.0);
    });

    it('returns low score for opposite preferences', () => {
      const userA = mockUsers.adventureSeeker;
      const userB = mockUsers.chillRelaxer;
      const score = calculateCompatibilityScore(userA, userB);
      expect(score).toBeLessThan(0.3);
    });

    it('weights budget compatibility at 30%', () => {
      const userA = { ...mockUsers.adventureSeeker, budgetRange: [500000, 1000000] };
      const userB = { ...mockUsers.adventureSeeker, budgetRange: [5000000, 10000000] };
      const score = calculateCompatibilityScore(userA, userB);
      expect(score).toBeLessThan(0.8); // budget mismatch drags down
    });

    it('handles missing optional fields gracefully', () => {
      const userA = { ...mockUsers.minimal, travelStyle: undefined };
      const userB = mockUsers.adventureSeeker;
      expect(() => calculateCompatibilityScore(userA, userB)).not.toThrow();
    });
  });

  describe('matchTrips', () => {
    it('returns top 5 matches sorted by score descending', async () => {
      const matches = await matchTrips(mockUsers.adventureSeeker, mockDestinations);
      expect(matches).toHaveLength(5);
      expect(matches[0].score).toBeGreaterThanOrEqual(matches[1].score);
    });

    it('excludes destinations user has already visited', async () => {
      const user = { ...mockUsers.adventureSeeker, visitedDestinations: ['dest-001'] };
      const matches = await matchTrips(user, mockDestinations);
      const ids = matches.map(m => m.destination.id);
      expect(ids).not.toContain('dest-001');
    });

    it('filters by user budget range', async () => {
      const user = { ...mockUsers.budgetTraveler, budgetRange: [100000, 300000] };
      const matches = await matchTrips(user, mockDestinations);
      matches.forEach(m => {
        expect(m.destination.estimatedCost).toBeLessThanOrEqual(300000);
      });
    });
  });
});
```

### 2.6 Contoh Unit Test — Payment Module

```typescript
// src/lib/payment/midtrans.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { createTransaction, handleNotification, verifySignature } from './midtrans';

// Mock Midtrans SDK
vi.mock('midtrans-client', () => ({
  Snap: vi.fn().mockImplementation(() => ({
    createTransaction: vi.fn().mockResolvedValue({
      token: 'mock-token-123',
      redirect_url: 'https://app.sandbox.midtrans.com/snap/v2/vtweb/mock-token-123',
    }),
  })),
  CoreApi: vi.fn().mockImplementation(() => ({
    transaction: {
      notification: vi.fn().mockResolvedValue({
        transaction_status: 'capture',
        order_id: 'ORDER-001',
        gross_amount: '150000.00',
        signature_key: 'valid-signature',
      }),
    },
  })),
}));

describe('Midtrans Payment Integration', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    process.env.MIDTRANS_SERVER_KEY = 'SB-Mid-server-mock';
    process.env.MIDTRANS_CLIENT_KEY = 'SB-Mid-client-mock';
  });

  describe('createTransaction', () => {
    it('creates Snap token for valid order', async () => {
      const result = await createTransaction({
        orderId: 'ORDER-001',
        grossAmount: 150000,
        customerDetails: { name: 'Budi', email: 'budi@test.com' },
      });
      expect(result.token).toBe('mock-token-123');
      expect(result.redirectUrl).toContain('midtrans.com');
    });

    it('throws on zero amount', async () => {
      await expect(
        createTransaction({ orderId: 'ORDER-002', grossAmount: 0, customerDetails: {} })
      ).rejects.toThrow('Amount must be greater than 0');
    });

    it('formats amount to 2 decimal places', async () => {
      const spy = vi.fn();
      await createTransaction({
        orderId: 'ORDER-003',
        grossAmount: 150000,
        customerDetails: { name: 'Test' },
      });
      // Verify amount is formatted correctly in the call
    });
  });

  describe('handleNotification', () => {
    it('processes capture status as success', async () => {
      const result = await handleNotification({
        transaction_status: 'capture',
        order_id: 'ORDER-001',
        gross_amount: '150000.00',
        signature_key: 'valid-signature',
      });
      expect(result.status).toBe('success');
      expect(result.orderId).toBe('ORDER-001');
    });

    it('handles pending status correctly', async () => {
      const result = await handleNotification({
        transaction_status: 'pending',
        order_id: 'ORDER-002',
        gross_amount: '200000.00',
        signature_key: 'valid-signature',
      });
      expect(result.status).toBe('pending');
    });

    it('handles deny status as failed', async () => {
      const result = await handleNotification({
        transaction_status: 'deny',
        order_id: 'ORDER-003',
        gross_amount: '100000.00',
        signature_key: 'valid-signature',
      });
      expect(result.status).toBe('failed');
    });
  });
});
```

---

## 3. Integration Testing (Supabase)

### 3.1 Supabase Test Environment

```yaml
# docker-compose.test.yml
version: '3.8'
services:
  supabase-db:
    image: supabase/postgres:15.1.0
    ports:
      - "54322:5432"
    environment:
      POSTGRES_PASSWORD: test_password
      POSTGRES_DB: jejakawan_test
    volumes:
      - ./supabase/migrations:/docker-entrypoint-initdb.d
      - ./supabase/seed.sql:/docker-entrypoint-initdb.d/seed.sql

  supabase-auth:
    image: supabase/gotrue:v2.99.0
    ports:
      - "9999:9999"
    environment:
      GOTRUE_DB_DATABASE_URL: postgres://postgres:test_password@supabase-db:5432/jejakawan_test
      GOTRUE_SITE_URL: http://localhost:5173
```

### 3.2 Integration Test Config

```typescript
// vitest.config.integration.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['src/**/*.integration.test.ts'],
    testTimeout: 30000, // Supabase operations may be slower
    hookTimeout: 15000,
    setupFiles: ['./src/test/setup-integration.ts'],
    pool: 'forks', // isolate tests to prevent DB state leakage
    poolOptions: {
      forks: { singleFork: false },
    },
  },
});
```

### 3.3 Integration Test Setup & Helpers

```typescript
// src/test/setup-integration.ts
import { createClient, SupabaseClient } from '@supabase/supabase-js';
import { beforeAll, afterAll, beforeEach } from 'vitest';

let supabase: SupabaseClient;

beforeAll(async () => {
  supabase = createClient(
    process.env.SUPABASE_URL ?? 'http://localhost:54321',
    process.env.SUPABASE_ANON_KEY ?? 'test-anon-key'
  );
  // Run migrations
  // Seed test data
});

beforeEach(async () => {
  // Reset tables between tests
  await supabase.rpc('reset_test_data');
});

afterAll(async () => {
  // Cleanup
});

export { supabase };
```

### 3.4 Contoh Integration Test — Booking Flow

```typescript
// src/lib/booking/booking.integration.test.ts
import { describe, it, expect, beforeAll } from 'vitest';
import { supabase } from '@/test/setup-integration';

describe('Booking Flow Integration', () => {
  let userId: string;
  let destinationId: string;

  beforeAll(async () => {
    // Create test user
    const { data: authData } = await supabase.auth.signUp({
      email: 'integration@test.com',
      password: 'TestPass123!',
    });
    userId = authData.user!.id;

    // Seed a destination
    const { data: dest } = await supabase
      .from('destinations')
      .insert({
        name: 'Pantai Test',
        province: 'Bali',
        category: 'beach',
        estimated_cost: 200000,
        latitude: -8.650,
        longitude: 115.219,
      })
      .select()
      .single();
    destinationId = dest!.id;
  });

  it('creates booking and updates availability', async () => {
    // Create booking
    const { data: booking, error } = await supabase
      .from('bookings')
      .insert({
        user_id: userId,
        destination_id: destinationId,
        date: '2026-10-01',
        participants: 2,
        total_price: 400000,
        status: 'pending',
      })
      .select()
      .single();

    expect(error).toBeNull();
    expect(booking).toBeDefined();
    expect(booking!.status).toBe('pending');
  });

  it('enforces unique booking constraint per user/date', async () => {
    // First booking
    await supabase.from('bookings').insert({
      user_id: userId,
      destination_id: destinationId,
      date: '2026-10-01',
      participants: 1,
      total_price: 200000,
    });

    // Duplicate booking same date
    const { error } = await supabase.from('bookings').insert({
      user_id: userId,
      destination_id: destinationId,
      date: '2026-10-01',
      participants: 1,
      total_price: 200000,
    });

    expect(error).toBeDefined();
    expect(error!.code).toBe('23505'); // unique_violation
  });

  it('RLS prevents viewing other users bookings', async () => {
    // Create second user
    const { data: user2 } = await supabase.auth.signUp({
      email: 'other@test.com',
      password: 'TestPass123!',
    });

    // user2 tries to query user1's bookings
    const { data, error } = await supabase
      .from('bookings')
      .select('*')
      .eq('user_id', userId);

    // Should only see own bookings due to RLS
    expect(data).toHaveLength(0);
  });
});
```

---

## 4. End-to-End Testing (Playwright)

### 4.1 Playwright Configuration

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
    ['html', { outputFolder: 'e2e-results' }],
    ['json', { outputFile: 'e2e-results/results.json' }],
    ['junit', { outputFile: 'e2e-results/junit.xml' }],
  ],
  use: {
    baseURL: process.env.E2E_BASE_URL ?? 'http://localhost:5173',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 13'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:5173',
    reuseExistingServer: !process.env.CI,
  },
});
```

### 4.2 E2E Test Structure

```
e2e/
├── auth/
│   ├── register.spec.ts
│   ├── login.spec.ts
│   └── social-login.spec.ts
├── onboarding/
│   ├── preference-setup.spec.ts
│   └── profile-completion.spec.ts
├── discovery/
│   ├── browse-destinations.spec.ts
│   ├── search-filter.spec.ts
│   └── destination-detail.spec.ts
├── trip-matching/
│   ├── match-flow.spec.ts
│   └── group-trip.spec.ts
├── booking/
│   ├── complete-booking.spec.ts
│   ├── payment-midtrans.spec.ts
│   └── cancel-booking.spec.ts
├── gamification/
│   ├── check-in.spec.ts
│   ├── badge-earning.spec.ts
│   └── leaderboard.spec.ts
├── fixtures/
│   ├── auth.fixture.ts
│   └── test-data.ts
└── utils/
    ├── helpers.ts
    └── page-objects/
```

### 4.3 Contoh E2E Test — Full Booking Journey

```typescript
// e2e/booking/complete-booking.spec.ts
import { test, expect } from '@playwright/test';
import { authFixture } from '../fixtures/auth.fixture';

test.describe('Complete Booking Journey', () => {
  test.use({ storageState: 'e2e/.auth/regular-user.json' });

  test('user can discover, select, and book a destination', async ({ page }) => {
    // 1. Browse destinations
    await page.goto('/discover');
    await expect(page.getByRole('heading', { name: /jelajahi/i })).toBeVisible();

    // 2. Filter by category
    await page.getByRole('button', { name: /pantai/i }).click();
    await page.waitForResponse(resp =>
      resp.url().includes('/rest/v1/destinations') && resp.status() === 200
    );

    // 3. Select a destination
    const firstCard = page.getByTestId('destination-card').first();
    await expect(firstCard).toBeVisible();
    const destName = await firstCard.getByTestId('dest-name').textContent();
    await firstCard.click();

    // 4. Verify detail page
    await expect(page.getByRole('heading', { name: destName! })).toBeVisible();
    await expect(page.getByTestId('dest-map')).toBeVisible();
    await expect(page.getByTestId('dest-photos')).toBeVisible();

    // 5. Initiate booking
    await page.getByRole('button', { name: /pesan sekarang/i }).click();

    // 6. Fill booking form
    await page.getByLabel('Tanggal').fill('2026-11-15');
    await page.getByLabel('Jumlah Peserta').fill('2');
    await page.getByRole('button', { name: /lanjutkan/i }).click();

    // 7. Verify order summary
    await expect(page.getByTestId('order-summary')).toBeVisible();
    await expect(page.getByTestId('total-price')).toContainText('Rp');

    // 8. Proceed to payment
    await page.getByRole('button', { name: /bayar/i }).click();

    // 9. Midtrans Snap popup should appear
    const popup = await page.waitForEvent('popup');
    await expect(popup).toHaveURL(/midtrans.com/);
    await popup.close();

    // 10. Verify booking created (pending status)
    await page.goto('/bookings');
    await expect(page.getByText(/menunggu pembayaran/i)).toBeVisible();
  });

  test('booking fails gracefully on payment timeout', async ({ page }) => {
    // Simulate payment timeout scenario
    await page.route('**/api/payment/timeout', route =>
      route.fulfill({ status: 408, body: 'Timeout' })
    );

    await page.goto('/bookings/pending-booking-id');
    await page.getByRole('button', { name: /coba lagi/i }).click();
    await expect(page.getByText(/waktu pembayaran habis/i)).toBeVisible();
  });
});
```

### 4.4 Contoh E2E Test — Trip Matching

```typescript
// e2e/trip-matching/match-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Trip Matching Flow', () => {
  test.use({ storageState: 'e2e/.auth/regular-user.json' });

  test('complete trip matching with compatible traveler', async ({ page }) => {
    await page.goto('/match');

    // Set preferences
    await page.getByLabel('Tanggal Perjalanan').fill('2026-12-01');
    await page.getByLabel('Durasi').selectOption('3-hari');
    await page.getByLabel('Budget Maks').fill('2000000');
    await page.getByRole('button', { name: /cari teman/i }).click();

    // Wait for match results
    await page.waitForSelector('[data-testid="match-card"]', { timeout: 10000 });
    const matchCards = page.getByTestId('match-card');
    const count = await matchCards.count();
    expect(count).toBeGreaterThan(0);

    // View first match profile
    await matchCards.first().getByRole('button', { name: /lihat profil/i }).click();
    await expect(page.getByTestId('match-profile')).toBeVisible();

    // Send match request
    await page.getByRole('button', { name: /ajak gabung/i }).click();
    await expect(page.getByText(/undangan terkirim/i)).toBeVisible();
  });
});
```

---

## 5. Load Testing (k6)

### 5.1 k6 Configuration

```javascript
// k6/scripts/config.js
export const config = {
  baseUrl: __ENV.BASE_URL || 'https://api-staging.jejakawan.id',
  supabaseUrl: __ENV.SUPABASE_URL || 'https://xxx.supabase.co',
  supabaseKey: __ENV.SUPABASE_ANON_KEY,
};
```

### 5.2 Scenarios & Thresholds

```javascript
// k6/scripts/load-test.js
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { Rate, Trend, Counter } from 'k6/metrics';
import { config } from './config.js';

// Custom metrics
const errorRate = new Rate('errors');
const bookingDuration = new Trend('booking_duration');
const matchDuration = new Trend('match_duration');
const paymentSuccess = new Counter('payment_success');

// Test scenarios
export const options = {
  scenarios: {
    // Scenario 1: Normal browsing load
    browse: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '2m', target: 50 },   // ramp up
        { duration: '5m', target: 50 },   // steady state
        { duration: '2m', target: 100 },  // peak
        { duration: '3m', target: 100 },  // peak hold
        { duration: '2m', target: 0 },    // ramp down
      ],
      exec: 'browseFlow',
    },
    // Scenario 2: Matching burst (weekend evening)
    matching: {
      executor: 'ramping-vus',
      startVUs: 0,
      stages: [
        { duration: '1m', target: 20 },
        { duration: '3m', target: 30 },
        { duration: '1m', target: 0 },
      ],
      exec: 'matchFlow',
      startTime: '2m',
    },
    // Scenario 3: Payment spike (promo launch)
    payment: {
      executor: 'constant-arrival-rate',
      rate: 10,
      timeUnit: '1s',
      duration: '3m',
      preAllocatedVUs: 50,
      exec: 'paymentFlow',
      startTime: '4m',
    },
  },
  thresholds: {
    http_req_duration: ['p(95)<2000', 'p(99)<5000'],
    http_req_failed: ['rate<0.05'],
    errors: ['rate<0.1'],
    booking_duration: ['p(95)<3000'],
    match_duration: ['p(95)<5000'],
  },
};

const headers = {
  'Content-Type': 'application/json',
  'apikey': config.supabaseKey,
};

export function browseFlow() {
  group('Browse Destinations', () => {
    // Home page load
    let res = http.get(`${config.supabaseUrl}/rest/v1/destinations?select=*&limit=20`, {
      headers: { ...headers, 'Authorization': `Bearer ${getToken()}` },
    });
    check(res, { 'home 200': (r) => r.status === 200 });
    errorRate.add(res.status !== 200);

    sleep(2);

    // Search with filters
    res = http.get(
      `${config.supabaseUrl}/rest/v1/destinations?category=eq.beach&province=eq.Bali&limit=10`,
      { headers }
    );
    check(res, { 'search 200': (r) => r.status === 200 });

    sleep(1);

    // View detail
    const destinations = JSON.parse(res.body);
    if (destinations.length > 0) {
      res = http.get(
        `${config.supabaseUrl}/rest/v1/destinations?id=eq.${destinations[0].id}`,
        { headers }
      );
      check(res, { 'detail 200': (r) => r.status === 200 });
    }

    sleep(3);
  });
}

export function matchFlow() {
  group('Trip Matching', () => {
    const start = Date.now();
    let res = http.post(
      `${config.baseUrl}/api/match`,
      JSON.stringify({
        date: '2026-12-01',
        duration: 3,
        budget: 2000000,
        preferences: ['beach', 'culture'],
      }),
      { headers: { ...headers, 'Authorization': `Bearer ${getToken()}` } }
    );
    matchDuration.add(Date.now() - start);
    check(res, {
      'match 200': (r) => r.status === 200,
      'has results': (r) => JSON.parse(r.body).length > 0,
    });
    sleep(5);
  });
}

export function paymentFlow() {
  group('Payment Flow', () => {
    const start = Date.now();
    // Create booking
    let res = http.post(
      `${config.baseUrl}/api/bookings`,
      JSON.stringify({
        destination_id: 'test-dest-001',
        date: '2026-12-15',
        participants: 2,
      }),
      { headers: { ...headers, 'Authorization': `Bearer ${getToken()}` } }
    );
    check(res, { 'booking created': (r) => r.status === 201 });

    if (res.status === 201) {
      const booking = JSON.parse(res.body);

      // Initiate payment
      res = http.post(
        `${config.baseUrl}/api/payments/create`,
        JSON.stringify({ booking_id: booking.id }),
        { headers: { ...headers, 'Authorization': `Bearer ${getToken()}` } }
      );
      check(res, { 'payment token': (r) => r.status === 200 });

      bookingDuration.add(Date.now() - start);
      if (res.status === 200) paymentSuccess.add(1);
    }
    sleep(1);
  });
}
```

### 5.3 Stress Test — Database Connection Pool

```javascript
// k6/scripts/stress-db.js
import { check } from 'k6';
import http from 'k6/http';
import { config } from './config.js';

export const options = {
  scenarios: {
    db_stress: {
      executor: 'ramping-vus',
      stages: [
        { duration: '1m', target: 100 },
        { duration: '2m', target: 200 },
        { duration: '1m', target: 300 },  // push connection pool limit
        { duration: '30s', target: 0 },
      ],
    },
  },
  thresholds: {
    http_req_duration: ['p(95)<5000'],
    http_req_failed: ['rate<0.15'],
  },
};

// Concurrent complex queries to test connection pool
export default function () {
  const queries = [
    'destinations?select=*,reviews(*)&limit=50',
    'destinations?order=rating.desc&limit=100',
    'bookings?select=*,destinations(*)&status=eq.confirmed',
    'users?select=*,achievements(*)&limit=20',
  ];
  const query = queries[Math.floor(Math.random() * queries.length)];
  const res = http.get(`${config.supabaseUrl}/rest/v1/${query}`, { headers });
  check(res, { 'ok': (r) => r.status === 200 });
}
```

---

## 6. Test Data Strategy

### 6.1 Data Tiers

| Tier | Purpose | Location | Lifecycle |
|------|---------|----------|-----------|
| Fixtures | Static test data, co-located | `src/lib/**/__fixtures__/` | Versioned in git |
| Factories | Dynamic data generation | `src/test/factories/` | Generated per test run |
| Seeds | DB baseline for integration | `supabase/seed.test.sql` | Reset per test suite |
| Mocks | API response simulation | `src/mocks/` | MSW handlers |

### 6.2 Factory Pattern

```typescript
// src/test/factories/user.factory.ts
import { faker } from '@faker-js/faker/locale/id_ID';
import type { UserProfile } from '@/types';

export function createUser(overrides?: Partial<UserProfile>): UserProfile {
  return {
    id: faker.string.uuid(),
    email: faker.internet.email(),
    full_name: faker.person.fullName(),
    avatar_url: faker.image.avatar(),
    travel_style: faker.helpers.arrayElement(['adventure', 'cultural', 'relaxation', 'budget']),
    budget_range: [
      faker.number.int({ min: 100000, max: 500000 }),
      faker.number.int({ min: 500000, max: 5000000 }),
    ],
    visited_destinations: [],
    level: faker.number.int({ min: 1, max: 50 }),
    xp: faker.number.int({ min: 0, max: 10000 }),
    created_at: faker.date.past().toISOString(),
    ...overrides,
  };
}

export function createDestination(overrides?: Partial<any>) {
  return {
    id: faker.string.uuid(),
    name: faker.location.city() + ' Beach',
    province: faker.helpers.arrayElement(['Bali', 'NTB', 'Jawa Timur', 'Sulawesi Selatan', 'Sumatera Barat']),
    category: faker.helpers.arrayElement(['beach', 'mountain', 'cultural', 'nature', 'urban']),
    description: faker.lorem.paragraphs(2),
    estimated_cost: faker.number.int({ min: 50000, max: 5000000 }),
    latitude: faker.location.latitude({ min: -11, max: 6 }),
    longitude: faker.location.longitude({ min: 95, max: 141 }),
    rating: faker.number.float({ min: 3.0, max: 5.0, fractionDigits: 1 }),
    photos: Array.from({ length: 5 }, () => faker.image.url()),
    ...overrides,
  };
}
```

### 6.3 Mock Service Worker (MSW) Handlers

```typescript
// src/mocks/handlers.ts
import { http, HttpResponse } from 'msw';
import { createUser, createDestination } from '../test/factories';

const mockUser = createUser();
const mockDestinations = Array.from({ length: 20 }, () => createDestination());

export const handlers = [
  // Auth
  http.post('*/auth/v1/token', () => {
    return HttpResponse.json({
      access_token: 'mock-access-token',
      refresh_token: 'mock-refresh-token',
      user: { id: mockUser.id, email: mockUser.email },
    });
  }),

  // Destinations
  http.get('*/rest/v1/destinations', ({ request }) => {
    const url = new URL(request.url);
    const limit = parseInt(url.searchParams.get('limit') ?? '20');
    return HttpResponse.json(mockDestinations.slice(0, limit));
  }),

  // Trip Matching
  http.post('*/api/match', async () => {
    return HttpResponse.json(
      mockDestinations.slice(0, 5).map(d => ({
        destination: d,
        score: 0.85,
        matchedUsers: [createUser()],
      }))
    );
  }),

  // Payment
  http.post('*/api/payments/create', () => {
    return HttpResponse.json({
      token: 'mock-snap-token',
      redirect_url: 'https://app.sandbox.midtrans.com/snap/v2/vtweb/mock',
    });
  }),

  // Supabase Realtime (mock)
  http.get('*/realtime/v1/websocket', () => {
    return new HttpResponse(null, { status: 101 });
  }),
];
```

---

## 7. Critical Paths — Must-Test Matrix

| Path | Type | Priority | Test Files |
|------|------|----------|------------|
| Registrasi + Onboarding | E2E | 🔴 Critical | `e2e/auth/register.spec.ts`, `e2e/onboarding/*.spec.ts` |
| Login (Email + Google OAuth) | E2E + Unit | 🔴 Critical | `e2e/auth/login.spec.ts` |
| Browse + Search Destinasi | E2E + Integration | 🔴 Critical | `e2e/discovery/*.spec.ts` |
| Trip Matching Algorithm | Unit | 🔴 Critical | `src/lib/matching/algorithm.test.ts` |
| Booking + Payment (Midtrans) | E2E + Integration | 🔴 Critical | `e2e/booking/*.spec.ts` |
| Subscription Plan Management | Unit + Integration | 🟡 High | `src/lib/subscription/*.test.ts` |
| Gamification (Check-in, Badge) | Unit + E2E | 🟡 High | `e2e/gamification/*.spec.ts` |
| Push Notification Delivery | Integration | 🟡 High | `src/lib/notification/*.test.ts` |
| Offline Mode (PWA) | E2E | 🟡 High | `e2e/offline/*.spec.ts` |
| Leaderboard + Social Features | Unit | 🟢 Medium | `src/lib/social/*.test.ts` |
| Admin Dashboard | E2E | 🟢 Medium | `e2e/admin/*.spec.ts` |
| Image Upload + Storage | Integration | 🟢 Medium | `src/lib/storage/*.test.ts` |

---

## 8. CI/CD Integration

### 8.1 GitHub Actions Pipeline

```yaml
# .github/workflows/test.yml
name: Test Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  unit-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:unit -- --coverage
      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  integration-test:
    runs-on: ubuntu-latest
    needs: unit-test
    services:
      postgres:
        image: supabase/postgres:15.1.0
        env:
          POSTGRES_PASSWORD: test_password
          POSTGRES_DB: jejakawan_test
        ports:
          - 54322:5432
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run test:integration
        env:
          SUPABASE_URL: http://localhost:54321
          SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_TEST_ANON_KEY }}

  e2e-test:
    runs-on: ubuntu-latest
    needs: integration-test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: e2e-results/

  coverage-gate:
    runs-on: ubuntu-latest
    needs: unit-test
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: coverage
      - uses: codecov/codecov-action@v4
        with:
          fail_ci_if_error: true
          threshold: 80%

  load-test:
    runs-on: ubuntu-latest
    needs: e2e-test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: grafana/k6-action@v0.3.1
        with:
          filename: k6/scripts/load-test.js
          flags: --out json=k6-results.json
        env:
          BASE_URL: ${{ secrets.STAGING_API_URL }}
          SUPABASE_URL: ${{ secrets.SUPABASE_URL }}
          SUPABASE_ANON_KEY: ${{ secrets.SUPABASE_ANON_KEY }}
```

### 8.2 Pre-commit Hooks

```json
// package.json (relevant sections)
{
  "scripts": {
    "test": "vitest",
    "test:unit": "vitest run --config vitest.config.ts",
    "test:integration": "vitest run --config vitest.config.integration.ts",
    "test:e2e": "playwright test",
    "test:load": "k6 run k6/scripts/load-test.js",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint src/ --ext .ts,.tsx"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "vitest related --run"
    ]
  }
}
```

---

## 9. Test Monitoring & Reporting

### 9.1 Coverage Dashboard

```
Coverage reports di-upload ke:
├── Codecov (badge di README)
├── GitHub PR comments (auto coverage diff)
└── HTML report di artifacts

Metrics yang di-track:
├── Coverage trend per sprint
├── Flaky test rate (target < 2%)
├── Test execution time trend
└── Critical path coverage (%)
```

### 9.2 Load Test Reporting

```
k6 output → InfluxDB → Grafana dashboard

Dashboard panels:
├── Response time percentiles (p50/p95/p99)
├── Request rate (req/s)
├── Error rate by endpoint
├── Virtual users over time
├── Database connection pool utilization
└── Payment success rate
```

---

## 10. Test Execution Quick Reference

```bash
# Unit tests (watch mode during dev)
npm run test:unit

# Unit tests with coverage report
npm run test:coverage

# Integration tests (requires local Supabase)
docker compose -f docker-compose.test.yml up -d
npm run test:integration

# E2E tests
npm run test:e2e

# E2E tests with UI (debug mode)
npx playwright test --ui

# E2E specific test file
npx playwright test e2e/booking/complete-booking.spec.ts

# Load tests (staging only)
k6 run k6/scripts/load-test.js -e BASE_URL=https://api-staging.jejakawan.id

# All tests (CI mode)
npm run test:unit && npm run test:integration && npm run test:e2e
```
