# 05 — Feature Specification (Detail per Fitur)

## Fitur yang Di-MVP-kan (P0)

### F1: Personalized Recommendation Engine

```
Input:
├── user.preferred_interests (array)
├── user.budget_min, budget_max
├── user.transport_modes (array)
├── user.current_location (GPS)
├── user.subscription (untuk limit harian)
└── user.current_mode (tourist / explorer)

Logika (Rule-Based Filtering):
├── Step 1: Query destinations WHERE is_active = TRUE
├── Step 2: Filter by category overlap dengan preferred_interests
├── Step 3: Filter by entry_fee_min <= budget_max
├── Step 4: Filter by ST_DWithin(location, user_location, radius_km)
│   ├── radius default: 50 km (tourist), 100 km (explorer)
│   └── radius bisa diubah oleh user
├── Step 5: Hitung relevance score per destinasi:
│   score = (category_match_pct * 0.3)
│         + (distance_proximity * 0.25)
│         + (rating_avg / 5 * 0.2)
│         + (visit_count_normalized * 0.1)
│         + (underrated_bonus * 0.15)  // bonus kalau explorer mode
├── Step 6: Sort by relevance score DESC
├── Step 7: Limit results:
│   ├── Free user: 3 rekomendasi/hari
│   ├── Weekly: 10/hari
│   ├── Monthly/Exclusive: unlimited
│   └── Counter disimpan di tabel user_recommendation_limits
└── Step 8: Return destinations dengan metadata lengkap

Output:
├── List destinasi (max 20 per request)
├── Setiap item: id, name, category, location, cover_image, rating, entry_fee, distance_km, relevance_score
└── "X wisatawan akan ke sini minggu ini" (count dari trip_requests yang overlap)

API:
├── GET /api/recommendations?mode=tourist&offset=0&limit=20
└── Response: { destinations: [...], total, has_more, daily_remaining }
```

### F2: Trip Matching (Cari Teman Jalan)

```
Input:
├── destination_id (atau custom destination)
├── date_from, date_to
├── budget_min, budget_max
├── transport_modes
├── max_members
└── notes

Logika Matching:
Lihat [[09-Trip-Matching-Algorithm]] untuk detail scoring

Simplified:
├── Query trip_requests WHERE status = 'open'
│   AND destination_id = target (atau within 25km)
│   AND date range overlap >= 1 hari
│   AND budget range overlap
│   AND creator.ktp_verified = TRUE
│   AND creator.rating_avg >= 3.0
│   AND creator.is_banned = FALSE
├── Hitung compatibility_score per candidate
├── Sort by compatibility_score DESC
├── Return top 20 candidates

User Flow:
├── User A buat trip request → sistem simpan
├── User A browse calon match → lihat kartu profil
├── User A klik "Ajak Match" → undangan dikirim
├── User B terima → chat room + trip group dibuat
├── User B tolak → notifikasi ke User A
├── 48 jam tidak respon → auto-expire

Output:
├── trip_requests (data request user A)
├── trip_matches (undangan yang dikirim)
├── trip_groups (kalau match diterima)
├── chat_rooms (room otomatis dibuat)

API:
├── POST /api/trip-requests (buat request)
├── GET /api/trip-requests/:id/matches (lihat calon match)
├── POST /api/trip-matches/:id/accept (terima undangan)
├── POST /api/trip-matches/:id/decline (tolak undangan)
└── GET /api/trip-groups (list grup aktif)
```

### F3: Open Trip Marketplace

```
Input (untuk browsing):
├── destination / location (opsional)
├── date range (opsional)
├── budget range
├── category
└── sort_by (price, rating, date)

Input (untuk booking):
├── package_id
├── num_participants
└── payment_method

Logika Booking:
├── Cek slot tersisa: package.max - package.current >= num_participants
├── Hitung total: price_per_person * num_participants
├── Hitung komisi: total * 10%
├── Buat booking record (status: pending)
├── Redirect ke Midtrans payment page
├── Midtrans callback → update status ke paid
├── Kurangi slot: package.current_participants += num_participants
├── Notifikasi ke user + agen travel

Cancellation Policy:
├── > 7 hari sebelum trip: refund 100%
├── 3-7 hari: refund 50%
├── < 3 hari: no refund
└── Refund via Midtrans refund API

API:
├── GET /api/open-trips (browse packages)
├── GET /api/open-trips/:id (detail package)
├── POST /api/open-trips/:id/book (booking)
├── POST /api/bookings/:id/cancel (cancel booking)
└── GET /api/bookings (list booking user)
```

### F7: Safety & Trust Layer

```
Sub-Feature 7a: Verifikasi Identitas
├── Input: foto KTP, nama, NIK
├── Proses: kirim ke Verihubs API → return match/not
├── Output: user.ktp_verified = true/false
├── Rate limit: max 3 attempt per 24 jam
└── Fallback: manual review oleh admin kalau API gagal

Sub-Feature 7b: Rating & Review
├── Input: score (1-5), review text
├── Proses: simpan ke ratings table → update user.rating_avg
├── Trigger: otomatis muncul setelah trip selesai (3 hari grace period)
├── Anti-abuse: user tidak bisa rate diri sendiri, 1 rating per trip
└── Update: rating_avg dihitung ulang pakai weighted average

Sub-Feature 7c: Live Location Sharing
├── Input: emergency_contact_phone, duration
├── Proses: generate share_token → simpan ke live_location_shares
├── Update: GPS position tiap 5 menit (bukan realtime)
├── Access: link publik dengan token (https://app.jejakawan.id/live/{token})
├── Expiry: auto-expire sesuai durasi / saat trip selesai
└── Privacy: data lokasi dihapus 24 jam setelah sharing berakhir

Sub-Feature 7d: Report User/Konten
├── Input: reason, details, evidence (screenshot)
├── Proses: simpan ke reports → queue untuk admin review
├── SLA: review dalam 24 jam
├── Action: warning / suspend / ban / dismiss
└── Notification: pelapor dapat update status

API:
├── POST /api/verify-identity (upload KTP)
├── POST /api/ratings (submit rating)
├── POST /api/location-share/start (mulai share)
├── POST /api/location-share/update (update posisi)
├── POST /api/location-share/stop (stop share)
├── GET /api/location-share/:token (public view)
└── POST /api/reports (submit report)
```

---

## Fitur P1 (Fase 2)

### F4: Gamifikasi Eksplorasi
Lihat [[08-Gamification-System]] untuk detail lengkap.

```
Ringkasan:
├── XP system dengan multiplier per subscription
├── Badge collection (achievement-based)
├── Level progression (XP threshold per level)
├── Misi dengan deadline + reward
├── Leaderboard regional (per provinsi)
└── Voucher redemption dari XP

API:
├── GET /api/missions (list misi aktif)
├── POST /api/missions/:id/claim (klaim misi)
├── POST /api/missions/:id/submit (upload bukti)
├── GET /api/xp/history (riwayat XP)
├── GET /api/badges (badge collection)
├── GET /api/leaderboard?region=jabar (leaderboard)
└── POST /api/xp/redeem (tukar XP ke voucher)
```

### F8: Itinerary Builder Kolaboratif
```
Input:
├── trip_group_id
├── destinations (drag & drop urutan)
├── time_per_destination (estimasi)
└── notes per destination

Logika:
├── Realtime sync via Supabase Realtime
├── Voting: anggota grup bisa vote destinasi
├── Sistem hitung waktu + jarak antar destinasi via Google Maps Directions API
├── Auto-generate timeline kalau user mau
└── Split bill: input pengeluaran → bagi rata ke anggota

API:
├── GET /api/trip-groups/:id/itinerary
├── POST /api/trip-groups/:id/itinerary (update)
├── POST /api/trip-groups/:id/vote (vote destinasi)
└── POST /api/trip-groups/:id/expenses (input pengeluaran)
```

### F9: Local Guide / Komunitas Lokal
```
Input (untuk guide):
├── nama, foto, bio
├── lokasi operasi (destinasi yang dilayani)
├── harga per hari/per trip
├── bahasa yang dikuasai
└── nomor rekening

Logika:
├── Guide daftar → admin verifikasi (manual)
├── Guide muncul di halaman destinasi terkait
├── User booking guide → payment via Midtrans
├── Komisi 10% ke Jejakawan
├── Rating setelah trip selesai

API:
├── POST /api/guides/register (daftar jadi guide)
├── GET /api/destinations/:id/guides (list guide per destinasi)
├── POST /api/guides/:id/book (booking guide)
└── GET /api/guides/bookings (list booking guide)
```

### F10: Verifikasi Konten UGC
```
Sudah tercakup di BP-6 Business Process.
Fitur ini adalah pipeline yang sama dengan mission_submissions
tapi juga apply ke konten feed biasa.

Logika:
├── Upload → auto-check (size, format, NSFW basic)
├── Auto-tag lokasi dari EXIF
├── Manual review kalau dari user baru atau klaim misi
├── Approved → publik + XP (kalau misi)
└── Rejected → notifikasi + bisa upload ulang
```

### F11: Weather & Seasonal Alert
```
Input:
├── user trip_requests (destinasi + tanggal)
└── destinations.best_season

Logika:
├── Trigger: saat user buat trip request atau buka destinasi
├── Cek OpenWeatherMap API (free tier: 1000 calls/hari)
├── Cek apakah tanggal trip = musim hujan di destinasi
├── Cek apakah tanggal = long weekend / peak season
├── Banner peringatan kalau relevan

API:
├── GET /api/destinations/:id/weather?date=2026-11-15
└── Internal: cron job harian untuk update weather data

Cost: Rp0 (free tier OpenWeatherMap)
```

### F12: Offline Mode
```
Fitur:
├── Download itinerary + peta area destinasi
├── Download disimpan di IndexedDB (browser storage)
├── Peta: Google Maps offline tiles (atau Mapbox offline)
├── Data: destinasi info, itinerary, kontak guide
├── Tidak bisa: trip matching, chat, upload (butuh online)
├── Size limit: max 50MB per download

API:
├── POST /api/offline/download (trigger download)
├── GET /api/offline/status (cek apa yang sudah di-download)
└── DELETE /api/offline/clear (hapus data offline)
```

### F13: Gamifikasi Sosial (Leaderboard)
```
Fitur:
├── Leaderboard per region (provinsi)
├── Ranking berdasarkan XP total
├── Update: setiap jam (cron job)
├── Visible: top 50 per region + posisi user sendiri
├── Badge kolektif: kalau grup trip selesaikan misi bareng

API:
├── GET /api/leaderboard?region={provinsi}&limit=50
└── GET /api/leaderboard/me (posisi user sendiri)
```

---

## Fitur Non-App (Supporting)

### F5: Sistem Donasi & Support Point
```
Fitur:
├── User bisa donasi ke developer (Rp10.000 - Rp1.000.000)
├── Donasi via Midtrans
├── Dapat badge donatur sesuai tier:
│   ├── Bronze: Rp10.000 - Rp49.999
│   ├── Silver: Rp50.000 - Rp199.999
│   ├── Gold: Rp200.000 - Rp499.999
│   └── Platinum: Rp500.000+
├── Badge muncul di profil publik
├── Tabel: donations

API:
├── POST /api/donations (submit donasi)
└── GET /api/donations/me (riwayat donasi)
```

### F6: Segmentasi User
```
Ini bukan fitur terpisah, tapi logic yang menentukan:
├── Tab mana yang terlihat di Home
├── Rekomendasi mana yang diprioritaskan
├── Fitur mana yang di-highlight
├── UI theme yang sedikit berbeda (warna aksen)

Implementasi:
├── user.current_mode = 'tourist' atau 'explorer'
├── Frontend baca mode → render komponen yang sesuai
├── Switch mode = toggle di Settings, langsung apply
└── Data XP/badge tetap persist di kedua mode
```

---

## Feature Dependency Graph

```
Urutan pembangunan (sprint planning):

SPRINT 1 (Minggu 1-2): Foundation
├── Auth & Onboarding (register, login, preferensi)
├── User Profile (CRUD, mode switch)
└── Destination CRUD (admin input data)

SPRINT 2 (Minggu 3-4): Core Discovery
├── F1: Recommendation Engine (rule-based)
├── Explore/Map (search, filter, detail destinasi)
└── Favorite/Simpan destinasi

SPRINT 3 (Minggu 5-6): Social Core
├── F7: Safety - verifikasi KTP & phone
├── F2: Trip Matching (buat request, compatibility, accept/decline)
└── Chat (realtime messaging)

SPRINT 4 (Minggu 7-8): Gamifikasi
├── F4: XP system, level, badge
├── F4: Misi system (claim, submit bukti, validasi)
└── Community Feed (UGC upload)

SPRINT 5 (Minggu 9-10): Monetisasi
├── F3: Open Trip Marketplace (browse, booking, payment)
├── Subscription (plan selection, Midtrans, auto-renew)
└── Rating & Review system

SPRINT 6 (Minggu 11-12): Polish & Launch Prep
├── Notification system (push, in-app)
├── Admin dashboard (moderation, reports)
├── Analytics integration (PostHog events)
├── Load testing & bug fixes
└── Soft launch (beta testing)

DEPENDENCIES:
├── F1 (Recommendation) ← standalone, butuh destination data
├── F2 (Matching) ← F7 (Safety verification) HARUS selesai duluan
├── F4 (Gamifikasi) ← F10 (UGC verification) untuk klaim misi
├── F3 (Marketplace) ← Midtrans integration, butuh agency mitra
├── F5 (Donasi) ← standalone, bisa kapan saja
├── F8 (Itinerary) ← F2 (Matching), F9 (Chat)
├── F9 (Local Guide) ← butuh onboarding manual per daerah
├── F11 (Weather) ← standalone, tapi butuh destinasi data
├── F12 (Offline) ← F1 (Recommendation) data harus ada dulu
└── F13 (Leaderboard) ← F4 (Gamifikasi) XP data harus ada dulu
```

---

## Analytics Events per Feature

```
AUTH:
├── user_registered          {method: google/email/phone}
├── user_logged_in           {method}
├── onboarding_completed     {segment: tourist/explorer, interests: [...]}
└── onboarding_skipped       {step: 1/2/3/4}

RECOMMENDATION:
├── recommendation_viewed    {count, mode, subscription}
├── recommendation_clicked   {destination_id, position, relevance_score}
├── recommendation_scrolled  {depth: 1-10}
└── destination_detail_viewed {destination_id, source: recommendation/explore/search}

TRIP MATCHING:
├── trip_request_created     {destination_id, budget_range, date_range}
├── match_candidates_viewed  {count}
├── match_invitation_sent    {target_user_id}
├── match_invitation_accepted {match_id}
├── match_invitation_declined {match_id}
├── match_invitation_expired {match_id}
└── chat_message_sent        {room_id, type: text/image/location}

GAMIFIKASI:
├── mission_claimed          {mission_id}
├── mission_submitted        {mission_id, destination_id}
├── mission_approved         {mission_id, xp_earned}
├── mission_rejected         {mission_id, reason}
├── mission_completed        {mission_id, badge_earned}
├── xp_earned                {amount, source, multiplier}
├── level_up                 {old_level, new_level}
├── badge_earned             {badge_id, badge_name}
├── xp_redeemed              {amount, voucher_type}
└── leaderboard_viewed       {region}

MARKETPLACE:
├── open_trip_viewed         {package_id, agency_id}
├── booking_initiated        {package_id, participants}
├── booking_completed        {package_id, amount}
├── booking_cancelled        {package_id, reason}
├── payment_method_selected  {method: va/ewallet/qris/cc}
└── payment_failed           {method, error_code}

SUBSCRIPTION:
├── subscription_plan_viewed {plan}
├── subscription_purchased   {plan, amount, method}
├── subscription_renewed     {plan}
├── subscription_expired     {plan}
├── subscription_cancelled   {plan, reason}
└── subscription_upgraded    {from_plan, to_plan}

SAFETY:
├── identity_verification_started {}
├── identity_verification_success {}
├── identity_verification_failed  {reason}
├── location_share_started   {}
├── location_share_stopped   {}
├── report_submitted         {reason, type}
└── user_blocked             {target_user_id}

ENGAGEMENT:
├── session_started          {source: direct/push/deeplink}
├── session_ended            {duration_seconds}
├── screen_viewed            {screen_name}
├── search_performed         {query, filters, result_count}
├── favorite_added           {destination_id}
├── favorite_removed         {destination_id}
└── profile_updated          {fields_changed: [...]}
```

---

## Feature Flags

```
Implementation: environment variable + database toggle

Tools:
├── Phase 1: simple boolean di database (tabel: feature_flags)
├── Phase 2: LaunchDarkly / Unleash (open source) untuk percentage rollout

Tabel: feature_flags
├── name VARCHAR(100) UNIQUE
├── enabled BOOLEAN DEFAULT FALSE
├── rollout_percentage INT DEFAULT 100  (0-100, untuk gradual rollout)
├── target_segments TEXT[]  (kosong = semua, ['explorer'] = hanya explorer)
├── updated_at TIMESTAMPTZ

Flags yang dibutuhkan:
├── gamification_enabled: bisa dimatikan kalau ada bug di XP system
├── trip_matching_enabled: bisa dimatikan kalau ada security issue
├── marketplace_enabled: bisa dimatikan kalau payment bermasalah
├── ktp_verification_enabled: bisa dimatikan kalau Verihubs down
├── live_location_enabled: bisa dimatikan kalau ada privacy concern
├── offline_mode_enabled: bisa dimatikan kalau belum stabil
├── ad_enabled: bisa dimatikan untuk premium user atau saat testing
└── maintenance_mode: matikan semua fitur tulis, hanya read-only

Usage di frontend:
const isGamificationEnabled = useFeatureFlag('gamification_enabled')
if (!isGamificationEnabled) return <MaintenanceNotice />
```
