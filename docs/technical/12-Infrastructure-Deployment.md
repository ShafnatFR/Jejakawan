# 12 — Infrastructure & Deployment

## Environment Tiers

```
Development (local)
├── Supabase local (supabase start — Docker)
├── Frontend: Vite dev server (localhost:5173)
├── Env: .env.local
└── Database: seeded dengan dummy data

Staging
├── Supabase project terpisah (gratis)
├── Frontend: Vercel preview deployment (auto dari PR)
├── Env: .env.staging
├── Database: subset data production (sanitized)
└── URL: staging.jejakawan.id

Production
├── Supabase project production
├── Frontend: Vercel production deployment
├── Env: .env.production (di Vercel environment variables)
├── Database: full data
└── URL: app.jejakawan.id
```

---

## Repository Structure

```
jejakawan/
├── apps/
│   ├── web/                    # PWA (React + Vite)
│   │   ├── src/
│   │   │   ├── components/     # Reusable UI components
│   │   │   ├── pages/          # Route pages
│   │   │   ├── hooks/          # Custom React hooks
│   │   │   ├── stores/         # Zustand state stores
│   │   │   ├── services/       # API calls (Supabase client)
│   │   │   ├── utils/          # Helper functions
│   │   │   └── assets/         # Images, icons, fonts
│   │   ├── public/             # Static files, PWA manifest
│   │   ├── vite.config.ts
│   │   ├── tailwind.config.ts
│   │   └── package.json
│   │
│   └── admin/                  # Admin dashboard (opsional, bisa pakai Supabase Studio)
│       └── ...
│
├── packages/
│   ├── shared/                 # Shared types, utils, constants
│   │   ├── types/              # TypeScript interfaces
│   │   ├── constants/          # Gamification rules, badge config
│   │   └── utils/              # Scoring formula, validation
│   └── ui/                     # Shared UI component library (opsional)
│
├── supabase/
│   ├── migrations/             # SQL migrations (auto-generated)
│   ├── functions/              # Edge Functions (Deno)
│   │   ├── matching/           # Trip matching engine
│   │   ├── recommendations/    # Recommendation engine
│   │   ├── xp-calculation/     # XP & level calculation
│   │   └── webhooks/           # Midtrans webhook handler
│   ├── seed.sql                # Seed data (dummy destinasi, misi, badge)
│   └── config.toml             # Supabase local config
│
├── docs/                       # Dokumen teknis (link ke Obsidian vault)
├── .github/
│   └── workflows/
│       ├── ci.yml              # Lint + test on PR
│       └── deploy.yml          # Deploy to Vercel on merge to main
├── .env.example
├── turbo.json                  # Turborepo config (monorepo)
└── package.json
```

---

## CI/CD Pipeline

### Continuous Integration (PR)
```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
    branches: [main, develop]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run lint          # ESLint
      - run: npm run type-check    # TypeScript strict check
      - run: npm run test          # Vitest
      - run: npm run build         # Vite build check
```

### Continuous Deployment (merge to main)
```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### Database Migration (manual trigger)
```yaml
# .github/workflows/migrate.yml
name: Database Migration
on:
  workflow_dispatch:  # manual trigger only

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
      - run: supabase db push --db-url ${{ secrets.SUPABASE_DB_URL }}
```

---

## Supabase Setup

### Project Configuration
```
Region: Singapore (terdekat ke Indonesia)
Plan: Free → Pro ($25/bulan) saat scale

Free tier limits:
├── Database: 500MB
├── Storage: 1GB
├── Edge Functions: 500K invocation/bulan
├── Auth: unlimited
├── Realtime: 200 concurrent connections
├── Bandwidth: 5GB/bulan
└── API requests: unlimited

Pro tier ($25/bulan):
├── Database: 8GB
├── Storage: 100GB
├── Edge Functions: 2M invocation/bulan
├── Bandwidth: 250GB/bulan
└── Bisa add-on: Point-in-time recovery ($100/bulan)
```

### Database Configuration
```sql
-- Enable PostGIS untuk geolocation queries
CREATE EXTENSION IF NOT EXISTS postgis;

-- Enable pg_trgm untuk fuzzy text search
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Enable pg_cron untuk scheduled jobs
CREATE EXTENSION IF NOT EXISTS pg_cron;

-- Scheduled jobs:
-- 1. Update leaderboard setiap jam
SELECT cron.schedule('update-leaderboard', '0 * * * *', $$
  REFRESH MATERIALIZED VIEW CONCURRENTLY leaderboard_cache;
$$);

-- 2. Cek expired subscription setiap hari jam 00:00
SELECT cron.schedule('check-expired-subs', '0 0 * * *', $$
  UPDATE user_profiles
  SET subscription = 'free'
  WHERE subscription != 'free'
    AND subscription_expires_at < NOW()
    AND subscription_expires_at > NOW() - INTERVAL '3 days';
$$);

-- 3. Expire trip matches yang tidak direspon 48 jam
SELECT cron.schedule('expire-matches', '0 */6 * * *', $$
  UPDATE trip_matches
  SET status = 'expired'
  WHERE status = 'pending'
    AND expires_at < NOW();
$$);

-- 4. Expire missions yang melewati deadline
SELECT cron.schedule('expire-missions', '0 0 * * *', $$
  UPDATE user_missions
  SET status = 'expired'
  WHERE status = 'active'
    AND deadline_at < NOW();
$$);
```

---

## Monitoring & Alerting

```
Error Tracking: Sentry
├── Frontend: capture JS errors, unhandled promises
├── Edge Functions: capture server-side errors
├── Alert: email + Slack kalau error rate > 1%
└── Free tier: 5.000 events/bulan

Uptime Monitoring: UptimeRobot
├── Check setiap 5 menit:
│   ├── https://app.jejakawan.id (PWA)
│   ├── https://*.supabase.co/rest/v1/ (API)
│   └── Custom health endpoint: /api/health
├── Alert: email + SMS kalau down > 5 menit
└── Free tier: 50 monitors

Performance: Supabase Dashboard + PostHog
├── Database query performance (Supabase)
├── Page load time (PostHog web vitals)
├── API response time (Edge Function logs)
└── User behavior analytics (PostHog events)

Database Monitoring:
├── Supabase dashboard: connections, query load, disk usage
├── Alert: kalau disk usage > 80%
├── Alert: kalau connection pool > 80%
└── Backup: daily automatic (Supabase Pro), point-in-time recovery
```

---

## Domain & DNS

```
Records:
├── app.jejakawan.id      → Vercel (PWA production)
├── staging.jejakawan.id  → Vercel (PWA staging)
├── api.jejakawan.id      → Supabase (Phase 2, custom domain)
├── live.jejakawan.id     → Vercel (live location public page)
└── *.jejakawan.id        → Vercel (wildcard untuk future)

Provider: Cloudflare (DNS + CDN + DDoS protection)
├── Free tier sudah cukup untuk MVP
├── CDN caching untuk static assets
└── SSL otomatis via Vercel + Cloudflare
```

---

## Backup & Disaster Recovery

```
Database Backup:
├── Supabase automatic daily backup (Pro plan)
├── Point-in-time recovery (Pro plan add-on)
├── Manual export: pg_dump setiap minggu → simpan di cloud storage
└── Retensi: 30 hari

Storage Backup:
├── Supabase Storage: bucket-level backup (manual)
├── Konten UGC: foto disimpan di Supabase Storage + Cloudinary mirror
└── Backup ke Google Cloud Storage / S3 setiap minggu

Disaster Recovery:
├── RTO (Recovery Time Objective): 4 jam
├── RPO (Recovery Point Objective): 24 jam
├── Procedure:
│   ├── 1. Identifikasi masalah (monitoring alert)
│   ├── 2. Isolate affected service
│   ├── 3. Restore dari backup terakhir
│   ├── 4. Test di staging dulu
│   ├── 5. Deploy ke production
│   └── 6. Post-mortem & update SOP
└── Database failover: Supabase automatic (Pro plan)
```

---

## Scaling Checklist

```
KAPAN HARUS UPGRADE:

Supabase Free → Pro ($25/bulan):
├── Database > 400MB (80% dari limit)
├── Storage > 800MB
├── Bandwidth > 4GB/bulan
├── Atau realtime connections > 150
└── Trigger: biasanya di ~3.000-5.000 active user

Tambah Custom Backend (Node.js/FastAPI):
├── Edge Functions sering timeout (> 5 detik)
├── Butuh background job processing (queue)
├── Butuh WebSocket custom (beyond Supabase Realtime)
├── Butuh caching layer (Redis)
└── Trigger: biasanya di ~20.000-50.000 active user

Pindah ke Dedicated Database:
├── Query performance degrade
├── Butuh read replicas
├── Butuh custom indexing strategy
├── Butuh sharding (per region)
└── Trigger: biasanya di ~100.000+ active user

Multi-Region:
├── Latency dari Indonesia ke Singapore acceptable (< 50ms)
├── Tapi kalau user base besar di luar SEA → tambah region
├── Supabase belum support multi-region → butuh self-hosted
└── Trigger: biasanya di ~500.000+ user global
```

---

## Development Workflow

```
Branching Strategy:
├── main         → production (auto-deploy ke Vercel)
├── develop      → staging (auto-deploy ke staging)
└── feature/*    → PR ke develop

Commit Convention:
├── feat: fitur baru
├── fix: bug fix
├── refactor: refactor code
├── docs: update dokumentasi
├── style: formatting, CSS
├── test: tambah test
├── chore: dependency update, config
└── perf: performance improvement

Code Review:
├── Semua PR harus 1 approval sebelum merge
├── CI harus pass (lint, type-check, test, build)
├── PR description wajib: apa yang diubah, kenapa, screenshot (kalau UI)
└── Squash merge ke develop, merge commit ke main
```

---

## Tech Stack Summary (Final)

```
LAYER           | TOOL                  | COST
────────────────┼───────────────────────┼──────────────
Frontend        | React + Vite + TS     | Rp0
Styling         | TailwindCSS           | Rp0
State           | Zustand               | Rp0
Backend/BaaS    | Supabase (Free→Pro)   | Rp0 → $25/bln
Database        | PostgreSQL (Supabase)  | included
Auth            | Supabase Auth         | included
Storage         | Supabase Storage      | included
Realtime        | Supabase Realtime     | included
Edge Functions  | Supabase Edge (Deno)  | included
Maps            | Google Maps JS API    | $200 free/bln
Payment         | Midtrans              | 2.5% per tx
Push Notif      | FCM (Firebase)        | Rp0
Analytics       | PostHog Cloud         | Rp0 (free tier)
Error Tracking  | Sentry                | Rp0 (free tier)
Monitoring      | UptimeRobot           | Rp0 (free tier)
DNS/CDN         | Cloudflare            | Rp0 (free tier)
Hosting (PWA)   | Vercel                | Rp0 (free tier)
CI/CD           | GitHub Actions        | Rp0 (free tier)
Domain          | .id domain            | ~Rp150rb/tahun
────────────────┼───────────────────────┼──────────────
TOTAL MVP       |                       | ~Rp15rb/bulan
TOTAL SCALE     |                       | ~Rp500rb-2jt/bulan
```

---

## Security Audit Checklist (Pre-Launch)

```
OWASP Top 10 Review:
├── A01 Broken Access Control:
│   ├── RLS policies tested (user A cannot read user B data)
│   ├── Admin endpoints require admin role
│   └── API endpoints check ownership before update/delete
├── A02 Cryptographic Failures:
│   ├── HTTPS everywhere (Vercel + Supabase default)
│   ├── KTP NIK stored as SHA-256 hash + salt (not plaintext)
│   ├── JWT tokens signed with Supabase secret
│   └── Password: Supabase Auth handles (bcrypt)
├── A03 Injection:
│   ├── Parameterized queries (Supabase client auto-handles)
│   ├── No raw SQL concatenation in Edge Functions
│   └── Input validation on all endpoints (schema enforcement)
├── A04 Insecure Design:
│   ├── Rate limiting on all endpoints
│   ├── Feature flags for kill switches
│   └── Graceful degradation for third-party failures
├── A05 Security Misconfiguration:
│   ├── Supabase: RLS enabled on ALL tables
│   ├── Storage: bucket policies restrict access
│   ├── CORS: only allow app.jejakawan.id
│   └── Debug mode: OFF in production
├── A06 Vulnerable Components:
│   ├── npm audit: run weekly (CI step)
│   ├── Snyk / Dependabot: auto-PR for security updates
│   └── Lock file: package-lock.json committed
├── A07 Auth Failures:
│   ├── Brute force: rate limit 5 attempt/min on login
│   ├── Token expiry: 1 hour access, 7 day refresh
│   └── Password policy: min 8 chars, 1 uppercase, 1 number
├── A08 Data Integrity:
│   ├── Midtrans webhook: signature verification (SHA512)
│   ├── File upload: type + size validation
│   └── EXIF data: stripped before storage (privacy)
├── A09 Logging & Monitoring:
│   ├── Audit logs for all admin actions
│   ├── Error logs to Sentry
│   └── Anomaly detection for suspicious patterns
└── A10 SSRF:
    ├── No user-controlled URLs in server-side requests
    ├── Image URLs validated before fetch
    └── API calls only to whitelisted domains

Dependency Scanning:
├── Tool: npm audit (built-in) + Snyk (free tier)
├── Frequency: every PR (CI) + weekly full scan
├── Action: auto-PR for patch updates, manual review for major
└── Threshold: 0 high/critical vulnerabilities in production
```

---

## Environment Variable Management

```
.env.example (template, committed to git):
├── VITE_SUPABASE_URL=https://xxx.supabase.co
├── VITE_SUPABASE_ANON_KEY=eyJhbG...
├── VITE_GOOGLE_MAPS_API_KEY=AIza...
├── VITE_POSTHOG_KEY=phc_...
├── VITE_SENTRY_DSN=https://...@sentry.io/...
├── VITE_MIDTRANS_CLIENT_KEY=SB-Mid-...
├── VITE_APP_URL=https://app.jejakawan.id
└── VITE_APP_ENV=production|staging|development

Secret management:
├── Local: .env.local (git-ignored)
├── Staging: Vercel environment variables (staging scope)
├── Production: Vercel environment variables (production scope)
├── Supabase: Supabase Dashboard → Settings → API
├── NEVER commit: .env files, API keys, secrets
└── Rotation: every 90 days for API keys (calendar reminder)

Vercel Environment Variables:
├── Set per environment: development / preview / production
├── Sensitive values: marked as "Sensitive" (hidden in UI)
├── Access: only project owner + team admins can view
└── Pull locally: vercel env pull .env.local
```

---

## Capacity Planning

```
Supabase Free Tier (50.000 user target):
├── Database: 500MB
│   ├── Estimasi: 25 tabel x rata-rata 1KB/row
│   ├── 10.000 users = ~10MB (profil + metadata)
│   ├── 50.000 trips = ~50MB
│   ├── 100.000 messages = ~100MB
│   ├── 500.000 XP logs = ~50MB
│   └── Total estimasi: ~300MB → MASIH muat di free tier
├── Storage: 1GB
│   ├── 10.000 foto UGC (500KB avg) = 5GB → TIDAK muat
│   ├── Solusi: Cloudinary free tier (25GB bandwidth) untuk images
│   └── Supabase Storage: hanya untuk KTP & critical docs
├── Bandwidth: 5GB/bulan
│   ├── 10.000 daily API calls x 2KB avg = 20MB/hari = 600MB/bulan
│   ├── Images: di-serve dari Cloudinary → hemat bandwidth Supabase
│   └── Total: ~1GB/bulan → MASIH muat di free tier
├── Edge Functions: 500K invocation/bulan
│   ├── 10.000 users x 5 calls/day x 30 days = 1.5M → TIDAK muat
│   ├── Solusi: cache aggressively, kurangi unnecessary calls
│   └── Atau: Supabase Pro ($25/bulan) = 2M invocation
└── Kapan upgrade ke Pro:
    ├── Database > 400MB (80%)
    ├── Storage > 800MB
    ├── Bandwidth > 4GB/bulan
    ├── Edge Function > 400K/bulan
    └── Atau: realtime connections > 150 concurrent

Vercel Free Tier:
├── Bandwidth: 100GB/bulan
├── Builds: 100/hour
├── Serverless Function: 100GB-Hrs
├── Custom domains: unlimited
├── PWA static files: ~5MB → 100GB bandwidth = 20M page views/bulan
└── Tidak perlu upgrade untuk MVP

Cost projection per scale:
├── 0-5.000 user: Rp15rb/bulan (domain only)
├── 5.000-50.000 user: Rp500rb/bulan (Supabase Pro)
├── 50.000-200.000 user: Rp2jt/bulan (+ Cloudinary Pro, monitoring)
└── 200.000+: Rp5jt+/bulan (+ dedicated backend, Redis, CDN)
```

---

## Disaster Recovery Plan (Detail)

```
Scenario 1: Database Corruption
├── Detection: Supabase health check + Sentry alerts
├── RPO: 24 jam (daily backup) / Point-in-time (Pro add-on)
├── RTO: 4 jam
├── Steps:
│   ├── 1. Matikan app (maintenance mode)
│   ├── 2. Restore dari backup (Supabase Dashboard → Backups)
│   ├── 3. Verify data integrity (spot check critical tables)
│   ├── 4. Test app functionality di staging
│   ├── 5. Deploy ke production
│   ├── 6. Matikan maintenance mode
│   └── 7. Post-mortem & notification ke user
└── Prevention: daily backup + monthly restore test

Scenario 2: Supabase Outage
├── Detection: UptimeRobot + health endpoint
├── Impact: app completely down (depends on Supabase)
├── RTO: depends on Supabase (usually < 1 hour)
├── Steps:
│   ├── 1. Tampilkan maintenance banner
│   ├── 2. Monitor Supabase status page
│   ├── 3. Communicate ke user via social media
│   └── 4. Resume setelah Supabase recovered
└── Mitigation: cache critical data di client (offline mode)

Scenario 3: Third-Party API Failure (Midtrans/Maps/Verihubs)
├── Detection: circuit breaker + error rate spike
├── Steps:
│   ├── Maps API down → switch ke list view (tanpa peta)
│   ├── Midtrans down → tampilkan "pembayaran sementara tidak tersedia"
│   ├── Verihubs down → manual verification queue
│   └── FCM down → in-app notification fallback
└── Prevention: implement circuit breaker (5 failures → open 30 detik)

Scenario 4: Security Breach
├── Detection: anomaly detection + user report
├── RTO: 1 jam untuk isolate
├── Steps:
│   ├── 1. Matikan affected endpoint/feature (feature flag)
│   ├── 2. Identify scope: data apa yang bocor?
│   ├── 3. Force logout semua user (rotate JWT secret)
│   ├── 4. Notify affected users (UU PDP: max 3x24 jam)
│   ├── 5. Report ke Kominfo (jika > 1000 user terdampak)
│   ├── 6. Fix vulnerability
│   ├── 7. Security audit penuh
│   └── 8. Post-mortem + public disclosure
└── Prevention: security audit checklist (di atas) + dependency scanning
```
