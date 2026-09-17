# 07 — API Design

## Konvensi

```
Base URL: https://api.jejakawan.id (Phase 2)
          Supabase REST API + Edge Functions (Phase 1)

Auth: Bearer token (JWT dari Supabase Auth)
      Header: Authorization: Bearer eyJhbG...

Response format: JSON
Error format: { "error": { "code": "NOT_FOUND", "message": "..." } }

Pagination: ?offset=0&limit=20
Sorting: ?sort=created_at&order=desc
Filtering: ?category=alam&min_rating=3.5
```

---

## Auth Endpoints

```
POST   /auth/register          — Daftar (email/phone + password)
POST   /auth/login             — Login (email/phone + password)
POST   /auth/login/google      — Login via Google OAuth
POST   /auth/login/phone       — Login via OTP (kirim kode)
POST   /auth/verify-otp        — Verifikasi OTP
POST   /auth/refresh           — Refresh JWT token
POST   /auth/logout            — Logout
POST   /auth/reset-password    — Reset password via email
```

---

## User Profile Endpoints

```
GET    /api/profile                    — Profil sendiri
PATCH  /api/profile                    — Update profil sendiri
GET    /api/profile/:id                — Profil publik user lain
PATCH  /api/profile/mode               — Switch tourist/explorer
GET    /api/profile/:id/badges         — Badge collection user
GET    /api/profile/:id/trips          — Trip history user
GET    /api/profile/:id/reviews        — Reviews yang diterima
GET    /api/profile/:id/konten         — Konten UGC user

Request PATCH /api/profile:
{
  "display_name": "Shafnat",
  "bio": "Suka traveling ke tempat sepi",
  "preferred_interests": ["alam", "kuliner"],
  "budget_min": 100000,
  "budget_max": 2000000,
  "transport_modes": ["motor", "mobil"]
}

Response GET /api/profile/:id:
{
  "id": "uuid",
  "display_name": "Shafnat",
  "avatar_url": "https://...",
  "bio": "...",
  "current_mode": "explorer",
  "rating_avg": 4.75,
  "rating_count": 23,
  "xp_total": 2400,
  "level": 5,
  "badge_count": 8,
  "trip_count": 12,
  "subscription": "monthly",
  "ktp_verified": true,
  "created_at": "2026-01-15T..."
}
```

---

## Destination Endpoints

```
GET    /api/destinations               — List & search destinasi
GET    /api/destinations/:id           — Detail destinasi
GET    /api/destinations/:id/reviews   — Reviews destinasi
GET    /api/destinations/:id/guides    — Local guides di destinasi
GET    /api/destinations/:id/weather   — Info cuaca
GET    /api/destinations/nearby        — Destinasi terdekat dari GPS

Query params GET /api/destinations:
  ?q=bandung               — search text
  ?category=alam,kuliner    — filter kategori
  ?min_rating=3.5           — filter rating
  ?max_fee=50000            — filter biaya masuk
  ?underrated=true          — hanya underrated spot
  ?lat=-6.9&lng=107.6      — center point
  ?radius=50               — radius dalam km
  ?sort=rating              — sort: rating, distance, newest, popular
  ?offset=0&limit=20

Response GET /api/destinations:
{
  "destinations": [
    {
      "id": "uuid",
      "name": "Ciletuh Geopark",
      "category": ["alam", "pantai"],
      "location": { "lat": -7.123, "lng": 106.456 },
      "cover_image_url": "https://...",
      "rating_avg": 4.6,
      "rating_count": 128,
      "entry_fee_min": 10000,
      "entry_fee_max": 15000,
      "distance_km": 23.5,
      "is_underrated": true,
      "signal_strength": "weak",
      "upcoming_visitors": 5
    }
  ],
  "total": 145,
  "has_more": true
}
```

---

## Recommendation Endpoints

```
GET    /api/recommendations            — Rekomendasi personal
GET    /api/recommendations/trending   — Destinasi trending

Query params:
  ?mode=tourist|explorer
  ?offset=0&limit=20

Response:
{
  "recommendations": [...],
  "daily_remaining": 7,     // -1 kalau unlimited
  "daily_limit": 10,
  "subscription": "weekly"
}
```

---

## Trip Matching Endpoints

```
POST   /api/trip-requests              — Buat trip request baru
GET    /api/trip-requests              — List trip request sendiri
GET    /api/trip-requests/:id          — Detail trip request
PATCH  /api/trip-requests/:id          — Update trip request
DELETE /api/trip-requests/:id          — Cancel trip request
GET    /api/trip-requests/:id/candidates — Lihat calon match

POST   /api/trip-matches               — Kirim undangan match
GET    /api/trip-matches               — List undangan (masuk + keluar)
POST   /api/trip-matches/:id/accept    — Terima undangan
POST   /api/trip-matches/:id/decline   — Tolak undangan

POST   /api/trip-requests:
{
  "destination_id": "uuid",
  "date_from": "2026-11-15",
  "date_to": "2026-11-17",
  "budget_min": 200000,
  "budget_max": 500000,
  "transport_modes": ["mobil"],
  "max_members": 4,
  "notes": "Suka foto-foto, santai aja"
}

Response GET /api/trip-requests/:id/candidates:
{
  "candidates": [
    {
      "user": {
        "id": "uuid",
        "display_name": "Andi",
        "avatar_url": "...",
        "rating_avg": 4.8,
        "ktp_verified": true
      },
      "compatibility": 87.5,
      "matching_interests": ["alam", "kuliner"],
      "date_overlap_days": 2,
      "budget_overlap": true
    }
  ],
  "total": 12
}
```

---

## Trip Group Endpoints

```
GET    /api/trip-groups                — List grup aktif
GET    /api/trip-groups/:id            — Detail grup
GET    /api/trip-groups/:id/members    — Anggota grup
POST   /api/trip-groups/:id/leave      — Keluar dari grup

PATCH  /api/trip-groups/:id/itinerary  — Update itinerary
POST   /api/trip-groups/:id/vote       — Vote destinasi
POST   /api/trip-groups/:id/expenses   — Input pengeluaran
GET    /api/trip-groups/:id/expenses   — Detail pengeluaran
```

---

## Chat Endpoints

```
GET    /api/chat/rooms                 — List chat rooms
GET    /api/chat/rooms/:id/messages    — Ambil pesan (paginated)
POST   /api/chat/rooms/:id/messages    — Kirim pesan

POST /api/chat/rooms/:id/messages:
{
  "message_type": "text",
  "content": "Halo, kita jadi ke Bandung?"
}

Realtime: subscribe ke channel chat_rooms/{room_id}
  → Setiap pesan baru masuk via WebSocket, tidak perlu polling
```

---

## Open Trip / Booking Endpoints

```
GET    /api/open-trips                 — Browse paket open trip
GET    /api/open-trips/:id             — Detail paket
POST   /api/open-trips/:id/book        — Booking
POST   /api/bookings/:id/cancel        — Cancel booking
GET    /api/bookings                   — List booking sendiri
GET    /api/bookings/:id               — Detail booking

POST /api/open-trips/:id/book:
{
  "num_participants": 2,
  "payment_method": "bca_va"
}

Response:
{
  "booking_id": "uuid",
  "total_price": 1000000,
  "payment_url": "https://app.midtrans.com/snap/v2/...",
  "expires_at": "2026-11-14T23:59:00Z"
}
```

---

## Gamifikasi Endpoints

```
GET    /api/missions                   — List misi tersedia
GET    /api/missions/:id               — Detail misi
POST   /api/missions/:id/claim         — Klaim misi
GET    /api/missions/active            — Misi yang sedang dikerjakan
POST   /api/missions/:id/submit        — Submit bukti (foto + lokasi)

GET    /api/xp                         — Info XP & level saat ini
GET    /api/xp/history                 — Riwayat XP (paginated)
GET    /api/badges                     — Semua badge + yang sudah didapat
GET    /api/badges/earned              — Badge yang sudah didapat
GET    /api/leaderboard                — Leaderboard regional
GET    /api/leaderboard/me             — Posisi sendiri
POST   /api/xp/redeem                  — Tukar XP ke voucher

POST /api/missions/:id/submit:
{
  "destination_id": "uuid",
  "photo_url": "uploaded_via_storage",
  "caption": "Akhirnya sampai di Curug Cikaso!",
  "location": { "lat": -7.123, "lng": 106.456 }
}

Response POST /api/xp/redeem:
{
  "xp_spent": 500,
  "voucher_code": "ALFAMART-XXXX-YYYY",
  "voucher_type": "Alfamart Rp25.000",
  "remaining_xp": 1900
}
```

---

## Rating & Review Endpoints

```
POST   /api/ratings                    — Submit rating
GET    /api/ratings/received           — Rating yang diterima
GET    /api/ratings/given              — Rating yang diberikan

POST /api/ratings:
{
  "ratee_id": "uuid",           // user lain (kalau rate user)
  "destination_id": null,
  "package_id": null,
  "trip_group_id": "uuid",
  "score": 5,
  "review": "Seru banget trip bareng Andi!"
}
```

---

## Safety & Trust Endpoints

```
POST   /api/verify/identity            — Upload KTP untuk verifikasi
GET    /api/verify/status              — Status verifikasi sendiri

POST   /api/location-share             — Mulai live location sharing
PATCH  /api/location-share/:id         — Update posisi
DELETE /api/location-share/:id         — Stop sharing
GET    /api/location-share/:token      — View lokasi (public, tanpa auth)

POST   /api/reports                    — Laporkan user/konten

POST /api/verify/identity:
{
  "full_name": "Shafnat Fuaini Ramadhan",
  "nik": "320xxx...",         // dikirim via HTTPS, disimpan hashed
  "ktp_photo_url": "uploaded_via_storage"
}

Response:
{
  "status": "verified",        // verified | pending_review | failed
  "attempts_remaining": 2
}
```

---

## Subscription & Payment Endpoints

```
GET    /api/subscriptions              — Status subscription saat ini
POST   /api/subscriptions              — Beli/upgrade subscription
DELETE /api/subscriptions              — Cancel auto-renew

GET    /api/transactions               — Riwayat transaksi

POST /api/subscriptions:
{
  "plan": "monthly",
  "payment_method": "gopay"
}

Response:
{
  "subscription_id": "uuid",
  "plan": "monthly",
  "price": 29000,
  "payment_url": "https://app.midtrans.com/snap/v2/...",
  "starts_at": "2026-11-17T...",
  "expires_at": "2026-12-17T..."
}
```

---

## Admin Endpoints (Internal)

```
GET    /api/admin/reports              — List laporan pending
PATCH  /api/admin/reports/:id          — Resolve laporan
GET    /api/admin/submissions          — List konten pending review
PATCH  /api/admin/submissions/:id      — Approve/reject konten
GET    /api/admin/users                — Search/manage users
PATCH  /api/admin/users/:id            — Suspend/ban user
GET    /api/admin/dashboard            — Stats overview

Auth: admin role required (RLS + JWT claim check)
```

---

## Webhook Endpoints

```
POST   /api/webhooks/midtrans          — Midtrans payment callback
POST   /api/webhooks/midtrans-recurring — Midtrans recurring payment

Midtrans callback payload handling:
├── Verify signature (SHA512)
├── Map order_id → transaction
├── Update payment_status
├── Trigger side effects (activate subscription, confirm booking, etc.)
└── Return 200 OK
```

---

## Total Endpoint Count

| Kategori | Jumlah |
|---|---|
| Auth | 8 |
| Profile | 8 |
| Destinations | 6 |
| Recommendations | 2 |
| Trip Matching | 9 |
| Trip Groups | 8 |
| Chat | 3 |
| Open Trip / Booking | 6 |
| Gamifikasi | 10 |
| Ratings | 3 |
| Safety & Trust | 6 |
| Subscription | 4 |
| Admin | 7 |
| Webhooks | 2 |
| **Total** | **~82 endpoint** |

---

## Rate Limiting per Endpoint

```
Endpoint Category        | Limit              | Window
─────────────────────────┼────────────────────┼────────
POST /auth/*             | 5 attempts         | /menit per IP
GET /api/destinations    | 60 requests        | /menit per user
GET /api/recommendations | 30 requests        | /menit per user
GET /api/trip-requests/*/candidates | 10 req | /menit per user
POST /api/trip-matches   | 10 requests        | /menit per user
POST /api/bookings       | 5 requests         | /menit per user
POST /api/subscriptions  | 3 requests         | /menit per user
POST /api/verify/identity | 3 attempts        | /jam per user
POST /api/media/upload   | 10 uploads         | /jam per user
POST /api/reports        | 5 reports          | /jam per user
GET /api/chat/*/messages | 120 requests       | /menit per user
POST /api/chat/*/messages| 60 messages        | /menit per user
GET /api/admin/*         | 200 requests       | /menit per admin
```

---

## Error Code Catalog

```
AUTH errors (1xx):
├── AUTH_100: Invalid credentials
├── AUTH_101: Account not found
├── AUTH_102: Account banned
├── AUTH_103: Invalid OTP
├── AUTH_104: OTP expired
├── AUTH_105: Too many attempts
└── AUTH_106: Token expired

MATCH errors (2xx):
├── MATCH_200: Not KTP verified (requirement)
├── MATCH_201: No candidates found
├── MATCH_202: Trip request not found
├── MATCH_203: Trip request not open
├── MATCH_204: Trip group full
├── MATCH_205: Match invitation expired
├── MATCH_206: Already matched with this user
└── MATCH_207: Trust score too low (< 30)

PAYMENT errors (3xx):
├── PAY_300: Payment gateway error
├── PAY_301: Payment timeout
├── PAY_302: Insufficient funds
├── PAY_303: Payment method not available
├── PAY_304: Invalid promo code
├── PAY_305: Booking slot full
├── PAY_306: Refund period expired
└── PAY_307: Duplicate transaction

GAMIFICATION errors (4xx):
├── GAME_400: Mission not available
├── GAME_401: Mission already claimed
├── GAME_402: Mission expired
├── GAME_403: Location mismatch (GPS tidak cocok)
├── GAME_404: Duplicate submission
├── GAME_405: Insufficient XP for redemption
├── GAME_406: Redemption limit reached (3/bulan)
└── GAME_407: Voucher stock empty

SAFETY errors (5xx):
├── SAFE_500: KTP verification failed
├── SAFE_501: KTP verification attempts exhausted
├── SAFE_502: Report already submitted
├── SAFE_503: User already blocked
├── SAFE_504: Location sharing limit reached
└── SAFE_505: File too large (max 10MB foto, 50MB video)

GENERAL errors (9xx):
├── GEN_900: Bad request (validation error)
├── GEN_901: Unauthorized (no valid token)
├── GEN_902: Forbidden (insufficient permission)
├── GEN_903: Not found
├── GEN_904: Rate limited
├── GEN_905: Internal server error
├── GEN_906: Service unavailable (maintenance)
└── GEN_907: Request timeout
```

---

## Cursor-Based Pagination

```
Offset pagination (current): lambat untuk large dataset
  GET /api/destinations?offset=1000&limit=20
  → DB harus scan 1020 rows, skip 1000

Cursor pagination (recommended): konsisten & cepat
  GET /api/destinations?cursor=abc123&limit=20
  → DB langsung jump ke posisi cursor, scan 20 rows

Implementation:
├── Response selalu include: next_cursor (null kalau sudah habis)
├── Cursor = base64 encoded: {created_at: "...", id: "..."}
├── Sort field + id sebagai composite cursor (jaminan unik)
└── Client: GET /api/destinations?cursor={next_cursor}&limit=20

Response format:
{
  "data": [...],
  "pagination": {
    "next_cursor": "MjAyNi0xMS0xNVQxMDozMDowMF4xMjM0NTY=",
    "has_more": true,
    "count": 20
  }
}

Kapan pakai cursor vs offset:
├── Cursor: destinasi list, chat messages, feed, notifications
└── Offset: admin dashboard (butuh "page 5 of 20"), search results
```

---

## Request Validation Schema

```
Setiap endpoint punya validasi yang di-enforce di Edge Function:

POST /api/trip-requests:
├── destination_id: UUID format (required OR destination_custom)
├── destination_custom: {lat: number, lng: number} (required OR destination_id)
├── date_from: ISO 8601 date, MUST be >= today
├── date_to: ISO 8601 date, MUST be >= date_from
├── budget_min: integer, >= 0, <= budget_max
├── budget_max: integer, >= budget_min, <= 100000000
├── transport_modes: array of enum ["motor","mobil","umum","pesawat"]
├── max_members: integer, 1-10
├── notes: string, max 500 chars, optional
└── Validation error response:
    {
      "error": {
        "code": "GEN_900",
        "message": "Validation failed",
        "details": [
          {"field": "date_from", "message": "Must be a future date"},
          {"field": "budget_max", "message": "Must be >= budget_min"}
        ]
      }
    }

POST /api/open-trips/:id/book:
├── num_participants: integer, 1-10
├── payment_method: enum ["bca_va","mandiri_va","bni_va","bri_va","gopay","ovo","dana","qris","cc"]
└── Validation:
    ├── package.is_active = true
    ├── package.current_participants + num_participants <= package.max_participants
    ├── package.date_from > today (belum lewat)
    └── user.bookings WHERE package_id AND status='pending' < 3 (no spam booking)

POST /api/ratings:
├── score: integer, 1-5
├── review: string, max 1000 chars, optional
├── Exactly one of: ratee_id / destination_id / package_id (required)
├── trip_group_id: UUID, optional
└── Validation:
    ├── User tidak bisa rate diri sendiri
    ├── 1 rating per user per target (no duplicate)
    └── Ratee harus ada di trip yang sama (kalau rate user)
```
