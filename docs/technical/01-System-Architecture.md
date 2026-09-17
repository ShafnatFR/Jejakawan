# 01 — System Architecture

## Overview

Jejakawan adalah platform mobile-first yang menghubungkan wisatawan dengan destinasi, teman perjalanan, dan warga lokal. Arsitektur dirancang untuk MVP (5.000 user) dengan jalur scaling ke 100.000+ user.

---

## Tech Stack

### Frontend (Mobile)
```
PWA (Phase 1 — MVP)
├── Framework     : React + Vite (PWA)
├── UI Library    : TailwindCSS + shadcn/ui
├── Maps          : Google Maps JavaScript API / Mapbox GL JS
├── State         : Zustand (lightweight)
├── Auth          : Supabase Auth (email, Google, phone OTP)
├── Offline       : Service Worker + IndexedDB (itinerary cache)
└── Push          : Web Push API (FCM)

Native App (Phase 2 — Validated)
├── Framework     : React Native / Flutter
├── Maps          : react-native-maps / flutter_map
└── Notification  : Firebase Cloud Messaging
```

**Alasan PWA dulu:**
- Satu codebase, jalan di semua device
- Tidak perlu Play Store ($25) / App Store ($99/tahun) developer account
- Update instant, tidak perlu review process
- Push notification, camera, GPS tetap bisa akses
- Kalau sudah validated (10.000+ user), baru port ke native

### Backend
```
Backend-as-a-Service (Phase 1 — MVP)
├── Platform      : Supabase (PostgreSQL + Auth + Storage + Realtime)
├── Edge Function : Supabase Edge Functions (Deno)
├── Cron/Job      : Supabase pg_cron (XP decay, badge check)
└── Search        : Supabase full-text search (destinasi)

Custom Backend (Phase 2 — Scale)
├── Runtime       : Node.js (Express) / Python (FastAPI)
├── ORM           : Prisma / Drizzle
├── Queue         : BullMQ (Redis) — background jobs
└── Cache         : Redis — session, rate limiting
```

**Alasan Supabase dulu:**
- Gratis sampai 50.000 user, 500MB database, 1GB storage
- Auth sudah built-in (email, Google, phone OTP)
- Realtime subscriptions (untuk trip matching live update)
- Row Level Security (RLS) = permission di database level
- Tidak perlu manage server sendiri

### Third-Party Services
```
Maps & Location
├── Google Maps Platform (free tier $200/bulan ≈ 28.500 map load)
├── Geocoding API (koordinat <-> alamat)
└── Places API (autocomplete destinasi)

Payment
├── Midtrans (populer di Indonesia, support VA, e-wallet, CC)
└── Xendit (alternatif, fee sama ~2-2.5%)

Identity Verification (Phase 2)
├── Verihubs (verifikasi KTP via API, ~Rp1.500/verifikasi)
└── Tada (alternatif)

Storage
├── Supabase Storage (foto UGC, profil, konten)
└── Cloudinary (image optimization & CDN — opsional)

Communication
├── Firebase Cloud Messaging (push notification)
└── Supabase Realtime (trip matching live update)

Analytics
├── PostHog (open source, self-hostable, event tracking)
└── Mixpanel (alternatif, free tier 20M events/bulan)

Monitoring
├── Sentry (error tracking, free tier 5K events/bulan)
└── UptimeRobot (uptime monitoring, free)
```

---

## Arsitektur Diagram (Text)

```
┌─────────────────────────────────────────────────────┐
│                    CLIENT LAYER                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │   PWA    │  │  PWA     │  │  Admin Dashboard │  │
│  │ (Mobile) │  │ (Desktop)│  │   (Internal)     │  │
│  └────┬─────┘  └────┬─────┘  └────────┬─────────┘  │
│       │              │                  │            │
└───────┼──────────────┼──────────────────┼────────────┘
        │              │                  │
        ▼              ▼                  ▼
┌─────────────────────────────────────────────────────┐
│                   API GATEWAY                        │
│         Supabase API + Edge Functions                │
│    ┌──────────────────────────────────────┐          │
│    │  Auth │ RLS │ Realtime │ Storage     │          │
│    └──────────────────────────────────────┘          │
└──────────────────────┬──────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
┌──────────┐  ┌──────────────┐  ┌──────────────┐
│PostgreSQL│  │   Storage    │  │  Realtime    │
│ Database │  │  (Files/Img) │  │ (WebSocket)  │
│          │  │              │  │              │
│ - users  │  │ - foto UGC  │  │ - match live │
│ - trips  │  │ - foto profil│ │ - chat       │
│ - dest.  │  │ - konten    │  │ - location   │
│ - xp_log │  │              │  │              │
└──────────┘  └──────────────┘  └──────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────┐
│              THIRD-PARTY INTEGRATIONS                │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌──────────────┐  │
│  │ Google │ │Midtrans│ │Verihubs│ │Firebase FCM  │  │
│  │ Maps   │ │Payment │ │  KTP   │ │Notification  │  │
│  └────────┘ └────────┘ └────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## Data Flow — Contoh Trip Matching

```
1. User A buka "Cari Teman Jalan"
2. Pilih destinasi + tanggal + budget range
3. Frontend kirim request ke Supabase Edge Function
4. Edge Function query database:
   - Cari user lain yang pilih destinasi & tanggal overlap
   - Filter: verifikasi KTP = true, rating >= 3.5
   - Sort: compatibility score (preferensi match %)
5. Hasil dikirim ke frontend → tampilkan kartu profil calon match
6. User A pilih "Ajak Match"
7. Notifikasi push ke User B via FCM
8. User B terima → chat room dibuka (Supabase Realtime)
9. Kedua user susun itinerary bareng (collaborative)
10. Data match tersimpan ke tabel trip_matches
```

---

## Scaling Path

```
Phase 1: MVP (0 - 5.000 user)
├── Supabase free tier
├── PWA saja
├── Manual moderation
├── Cost: Rp500rb-1jt/bulan
└── Timeline: 3-4 bulan development

Phase 2: Growth (5.000 - 50.000 user)
├── Supabase Pro ($25/bulan)
├── Tambah native app (React Native)
├── Automate moderation (basic)
├── Cost: Rp3-5jt/bulan
└── Timeline: 6-12 bulan setelah launch

Phase 3: Scale (50.000 - 500.000 user)
├── Custom backend (Node.js/FastAPI)
├── Dedicated database (Supabase/self-hosted)
├── ML recommendation (kalau data cukup)
├── Cost: Rp15-30jt/bulan
└── Timeline: 12-24 bulan setelah launch

Phase 4: Enterprise (500.000+ user)
├── Microservices architecture
├── Multi-region deployment
├── Dedicated team ops
├── Cost: Rp50jt+/bulan
└── Kalau sampai sini = sudah ada funding/investor
```

---

## Security Architecture

```
Layer 1: Transport
└── HTTPS everywhere (Supabase default)

Layer 2: Authentication
├── Supabase Auth (email + password / Google OAuth / Phone OTP)
├── JWT token untuk semua API call
└── Refresh token rotation

Layer 3: Authorization
├── Row Level Security (RLS) di PostgreSQL
├── User hanya bisa akses data sendiri
├── Admin role untuk moderasi
└── API rate limiting (Supabase Edge Function)

Layer 4: Data Protection
├── KTP data: encrypted at rest, access logged
├── Location data: hanya disimpan saat trip aktif
├── PDP Law compliance: consent sebelum collect data
└── Data retention: hapus data non-aktif setelah 12 bulan

Layer 5: Content Moderation
├── Auto: NSFW detection (Cloud Vision API)
├── Manual: Trust & Safety team review
└── Report: user bisa flag konten/user
```

---

## Cost Estimation per Phase

### Phase 1 (MVP)
| Item | Cost/bulan |
|---|---|
| Supabase Free | Rp0 |
| Google Maps (free tier) | Rp0 |
| Domain (.com) | Rp15.000 |
| Vercel (PWA hosting) | Rp0 |
| Sentry Free | Rp0 |
| PostHog Cloud Free | Rp0 |
| **Total** | **~Rp15.000** |

### Phase 2 (Growth)
| Item | Cost/bulan |
|---|---|
| Supabase Pro | Rp400.000 |
| Google Maps (estimasi) | Rp500.000 |
| Midtrans fee (2.5%) | Variable |
| Cloudinary | Rp0 (free tier) |
| Domain + monitoring | Rp50.000 |
| **Total Fixed** | **~Rp1.000.000** |

---

## Testing Strategy

```
Unit Testing:
├── Framework: Vitest (fast, Vite-native)
├── Target coverage: 80% (critical path 100%)
├── Focus: scoring formula, XP calculation, budget overlap, compatibility score
├── Mock: Supabase client, third-party API
└── Run: npm run test (CI wajib pass sebelum merge)

Integration Testing:
├── Supabase test helpers (supabase-test-helpers)
├── Test RLS policies: user A tidak bisa akses data user B
├── Test database trigger: XP auto-award setelah submission approved
├── Test Edge Function: matching engine return hasil valid
└── Run: terpisah dari unit test, butuh test database

E2E Testing:
├── Framework: Playwright
├── Critical flows yang wajib di-test:
│   ├── Register → Onboarding → Home
│   ├── Search destinasi → Lihat detail → Simpan
│   ├── Buat trip request → Dapat match → Accept → Chat
│   ├── Upload bukti misi → Approve → XP masuk
│   ├── Booking open trip → Bayar → Confirmed
│   └── Upgrade subscription → Benefit aktif
├── Run: nightly di CI (lambat, tidak setiap PR)
└── Environment: staging Supabase + Vercel preview

Load Testing:
├── Tool: k6 (open source, dari Grafana)
├── Scenarios:
│   ├── 100 concurrent users browsing destinasi → target < 500ms
│   ├── 50 concurrent matching queries → target < 2 detik
│   ├── 20 concurrent uploads → target < 5 detik
│   └── 500 concurrent API reads → target < 200ms
├── Run: sebelum launch dan setiap major update
└── Baseline: simpan hasil sebagai benchmark
```

---

## Logging & Observability

```
Structured Logging (JSON):
{
  "timestamp": "2026-11-15T10:30:00Z",
  "level": "info",
  "service": "matching-engine",
  "user_id": "uuid",
  "action": "match_query",
  "duration_ms": 342,
  "result_count": 12,
  "metadata": {}
}

Log Levels:
├── DEBUG: development only, suppressed di production
├── INFO: normal operations (user login, match found, payment success)
├── WARN: degraded but functional (API slow, cache miss, retry)
└── ERROR: failure (payment failed, matching timeout, upload rejected)

Storage:
├── Edge Function logs: Supabase Dashboard (built-in, 7 hari retensi)
├── Error logs: Sentry (5K events/bulan free)
├── Application logs: PostHog custom events (untuk analytics)
└── Production alerting: Sentry → Slack webhook

Distributed Tracing (Phase 2):
├── Trace ID: setiap request dapat unique ID
├── Propagation: trace_id di-pass dari frontend → Edge Function → DB query
├── Tool: OpenTelemetry (standard, free)
└── Purpose: debug slow request yang lintas beberapa service
```

---

## API Versioning & Rate Limiting

```
Versioning:
├── Strategy: URL path prefix (/api/v1/...)
├── Rationale: explicit, mudah di-cache, backward compatible
├── Policy: support versi sebelumnya minimal 6 bulan setelah versi baru
├── Deprecation: header X-Deprecated: true + tanggal sunset
└── Migration: dokumentasi perubahan di changelog

Rate Limiting:
├── Global: 100 requests/menit per user (JWT-based)
├── Per endpoint:
│   ├── Auth (login/register): 5 attempt/menit per IP
│   ├── Matching queries: 10 request/menit per user
│   ├── Recommendation: 30 request/menit per user
│   ├── Upload: 10 file/jam per user
│   ├── Booking: 5 attempt/menit per user
│   └── Admin: 200 request/menit per admin
├── Implementation: Supabase Edge Function + custom header
│   ├── X-RateLimit-Limit: 100
│   ├── X-RateLimit-Remaining: 87
│   └── X-RateLimit-Reset: 1700000000
├── Response: 429 Too Many Requests + retry-after header
└── Abuse detection: IP yang exceed 50x dalam 1 jam → temporary block

Error Handling Philosophy:
├── Graceful degradation:
│   ├── Maps API down → tampilkan list view instead of map
│   ├── Matching timeout → "Coba lagi nanti" + simpan query
│   ├── Payment gateway down → "Sistem pembayaran sedang gangguan"
│   └── Recommendation engine error → tampilkan popular destinations
├── Retry strategy:
│   ├── Third-party API: 3x retry dengan exponential backoff (1s, 2s, 4s)
│   ├── Database query: 2x retry (connection pool exhaustion)
│   └── No retry: user-facing action (booking, payment) — minta user ulangi
├── Circuit breaker:
│   ├── Threshold: 5 consecutive failures → open circuit
│   ├── Duration: 30 detik sebelum half-open
│   └── Applies: Google Maps API, Midtrans API, Verihubs API
└── User feedback:
    ├── Timeout: "Sepertinya koneksi lambat, coba lagi"
    ├── Error: "Terjadi kesalahan, tim kami sedang menangani"
    └── Maintenance: banner "Kami sedang maintenance, kembali jam X"
```
