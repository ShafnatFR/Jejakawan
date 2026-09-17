# 09 — Trip Matching Algorithm

## Overview

Algoritma matching menentukan seberapa cocok dua user untuk trip bareng. Output berupa compatibility score (0-100) yang dipakai untuk sorting calon match.

---

## Input Factors

| Factor | Bobot | Tipe Data | Sumber |
|---|---|---|---|
| Destinasi overlap | 25% | boolean (sama/tidak) | trip_requests |
| Tanggal overlap | 20% | int (hari overlap) | trip_requests |
| Budget overlap | 20% | range overlap % | trip_requests |
| Interest match | 15% | % interest yang sama | user_profiles |
| Transport match | 10% | boolean (cocok/tidak) | trip_requests |
| Reputation | 10% | rating_avg normalized | user_profiles |

---

## Scoring Formula

```
compatibility_score = (
    destination_score * 0.25 +
    date_score * 0.20 +
    budget_score * 0.20 +
    interest_score * 0.15 +
    transport_score * 0.10 +
    reputation_score * 0.10
) * 100

Range: 0.00 - 100.00
```

### 1. Destination Score (25%)

```
Logic:
├── Exact match (destinasi_id sama): score = 1.0
├── Nearby match (within 25km): score = 1.0 - (distance_km / 25)
│   Contoh: 10km → score = 1.0 - 0.4 = 0.6
├── Different destination: score = 0.0

Query:
SELECT tr2.id, tr2.creator_id,
  CASE
    WHEN tr2.destination_id = tr1.destination_id THEN 1.0
    WHEN ST_DWithin(
      COALESCE(tr2.destination_custom, d2.location),
      COALESCE(tr1.destination_custom, d1.location),
      25000  -- 25km dalam meter
    ) THEN 1.0 - (ST_Distance(
      COALESCE(tr2.destination_custom, d2.location),
      COALESCE(tr1.destination_custom, d1.location)
    ) / 25000.0)
    ELSE 0.0
  END AS destination_score
FROM trip_requests tr1
JOIN trip_requests tr2 ON tr2.id != tr1.id
LEFT JOIN destinations d1 ON tr1.destination_id = d1.id
LEFT JOIN destinations d2 ON tr2.destination_id = d2.id
WHERE tr1.id = :user_trip_request_id;
```

### 2. Date Score (20%)

```
Logic:
├── Hitung jumlah hari overlap antara dua date range
├── score = overlap_days / min(duration_A, duration_B)
│   Contoh: A = 15-17 Nov (3 hari), B = 16-18 Nov (3 hari)
│   Overlap = 2 hari (16-17)
│   score = 2/3 = 0.67
├── Tidak ada overlap: score = 0.0

Query (simplified):
overlap_start = MAX(date_from_A, date_from_B)
overlap_end = MIN(date_to_A, date_to_B)
overlap_days = MAX(0, overlap_end - overlap_start + 1)
duration = MIN(date_to_A - date_from_A + 1, date_to_B - date_from_B + 1)
date_score = overlap_days / duration
```

### 3. Budget Score (20%)

```
Logic:
├── Hitung overlap antara dua budget range
├── overlap_low = MAX(budget_min_A, budget_min_B)
├── overlap_high = MIN(budget_max_A, budget_max_B)
├── overlap_range = MAX(0, overlap_high - overlap_low)
├── total_range = MAX(budget_max_A, budget_max_B) - MIN(budget_min_A, budget_min_B)
├── score = overlap_range / total_range
│   Contoh: A = 200rb-500rb, B = 300rb-700rb
│   Overlap = 300rb-500rb = 200rb range
│   Total = 700rb-200rb = 500rb range
│   score = 200/500 = 0.4
├── Total range = 0 (keduanya budget sama persis): score = 1.0
```

### 4. Interest Score (15%)

```
Logic:
├── Hitung % interest yang sama antara dua user
├── common = INTERSECT(preferred_interests_A, preferred_interests_B)
├── total = UNION(preferred_interests_A, preferred_interests_B)
├── score = COUNT(common) / COUNT(total)
│   Contoh: A = [alam, kuliner, budaya], B = [alam, kota]
│   Common = [alam] (1), Total = [alam, kuliner, budaya, kota] (4)
│   score = 1/4 = 0.25

Note: Ini Jaccard similarity index.
```

### 5. Transport Score (10%)

```
Logic:
├── Ada transport mode yang sama? score = 1.0
├── Kompatibel (e.g., user A punya mobil, user B naik umum)?
│   → score = 0.5 (mereka bisa bareng tapi B perlu transport sendiri ke meeting point)
├── Tidak ada yang cocok: score = 0.0

Compatible pairs:
├── mobil ↔ motor: 0.5
├── mobil ↔ umum: 0.5
├── motor ↔ umum: 0.3
├── pesawat ↔ pesawat: 1.0
└── pesawat ↔ lainnya: 0.0 (beda moda, tidak bisa bareng)
```

### 6. Reputation Score (10%)

```
Logic:
├── Normalize rating_avg ke 0-1 range
├── score = rating_avg / 5.0
│   Contoh: rating = 4.5 → score = 0.9
├── Belum ada rating (rating_count = 0): score = 0.5 (neutral)
├── Bonus: rating_count > 10 → score += 0.1 (max 1.0)

Pinalti:
├── Ada report aktif (status = 'pending' atau 'reviewing'): score *= 0.5
├── Pernah di-suspend: score *= 0.8
└── Baru daftar (< 7 hari) dan belum verifikasi: score *= 0.7
```

---

## Filtering (Pre-Scoring)

Sebelum scoring, candidate difilter dulu:

```
Hard filters (wajib lolos):
├── trip_requests.status = 'open'
├── creator.ktp_verified = TRUE
├── creator.is_banned = FALSE
├── creator.is_active = TRUE
├── creator.rating_avg >= 3.0 (atau belum ada rating)
├── Bukan user yang sama (creator_id != current_user_id)
└── Ada date overlap >= 1 hari

Soft filters (bisa dilonggarkan):
├── Budget overlap > 0% (idealnya > 20%)
├── Destinasi within 100km (bukan 25km)
└── Interest ada minimal 1 yang sama
```

---

## Subscription Priority Boost

User dengan subscription dapat prioritas di matching queue:

```
Priority order:
1. Exclusive Pass    → +15% effective score (visual boost)
2. Monthly Pass      → +10% effective score
3. Weekly Pass       → +5% effective score
4. Free              → no boost

Implementation:
effective_score = base_score * (1 + subscription_boost)

Contoh:
  User A (Monthly): base_score = 70 → effective = 70 * 1.10 = 77
  User B (Free): base_score = 75 → effective = 75 * 1.00 = 75
  → User A muncul lebih tinggi di daftar match User C
```

---

## Matching Flow (Step by Step)

```
Step 1: User A buat trip_request
Step 2: Sistem trigger matching query (async, via Edge Function)
Step 3: Filter semua trip_request yang eligible
Step 4: Hitung compatibility_score per candidate
Step 5: Apply subscription priority boost
Step 6: Sort by effective_score DESC
Step 7: Return top 20 candidates ke User A
Step 8: User A pilih "Ajak Match" → undangan dikirim
Step 9: Sistem juga menjalankan reverse matching:
        → Cek apakah ada trip_request lain yang match dengan User A
        → Kalau ada, kirim notifikasi ke mereka: "Ada yang cocok dengan trip kamu!"
```

---

## Edge Cases

```
1. Tidak ada match sama sekali:
   → Tampilkan: "Belum ada yang cocok. Kami akan notifikasi kalau ada match baru."
   → Simpan search → cron job cek tiap 6 jam
   → Notifikasi kalau match baru muncul

2. Terlalu banyak match (> 50):
   → Ambil top 20 berdasarkan score
   → Tampilkan "Ada 50+ traveler cocok" → load more

3. User cancel trip request setelah mengirim undangan:
   → Undangan otomatis expired
   → Notifikasi ke invitee: "Trip request telah dibatalkan"

4. Grup sudah penuh tapi ada undangan pending:
   → Undangan otomatis expired
   → Notifikasi: "Grup sudah penuh"

5. User yang sama punya 2 trip request berbeda:
   → Dibolehkan (misal: Bandung weekend + Yogyakarta bulan depan)
   → Matching dijalankan per trip request
```

---

## Performance Optimization

```
Phase 1 (MVP, < 5.000 user):
├── Matching query langsung di PostgreSQL
├── PostGIS spatial index untuk geolocation queries
├── Composite index pada trip_requests (status, destination_id, date_from, date_to)
├── Query timeout: 5 detik
└── Cache hasil matching per user: 1 jam TTL

Phase 2 (5.000 - 50.000 user):
├── Pre-compute compatibility matrix (cron harian)
├── Materialized view untuk frequent queries
├── Edge Function dengan caching layer
└── Query timeout: 2 detik

Phase 3 (50.000+ user):
├── Dedicated matching service (microservice)
├── ML model untuk compatibility (kalau data cukup)
├── Real-time matching via WebSocket
├── Geographic sharding
```

---

## Cold Start Problem

```
Masalah: user baru belum punya interest data → recommendation & matching kosong

Solutions:
├── Onboarding preference collection (sudah ada di flow)
├── Popular destinations fallback:
│   ├── Kalau rekomendasi kosong → tampilkan top 10 destinasi populer di region user
│   ├── Filter: within 100km dari lokasi user
│   └── Sort: rating_avg DESC, visit_count DESC
├── "Baru di sini?" guided experience:
│   ├── Tampilkan 5 rekomendasi populer
│   ├── User pilih 1 yang menarik → sistem belajar preferensi
│   ├── "Kami akan sesuaikan rekomendasi berdasarkan pilihanmu"
│   └── Progressive profiling: tambah preferensi dari behavior (click, save, search)
├── For matching:
│   ├── User baru bisa browse tanpa match → lihat siapa yang mau ke destinasi yang sama
│   ├── "Popular trips near you" → trip request orang lain yang trending
│   └── Suggest: "Buat trip request pertamamu" → guided flow 3 langkah
└── Data collection timeline:
    ├── Hari 1: preferensi dari onboarding
    ├── Hari 3: behavior data (clicks, searches)
    ├── Hari 7: cukup data untuk personalized recommendation
    └── Hari 14: cukup data untuk matching yang akurat
```

---

## Matching untuk Daerah Non-Populer

```
Masalah: user di NTT/Sulawesi → 0 match dalam 100km

Solutions:
├── Expand radius otomatis:
│   ├── Default: 50km → kalau 0 match → expand ke 200km → 500km → nasional
│   ├── Banner: "Belum ada match di sekitarmu. Menampilkan dari radius lebih luas."
│   └── User bisa set manual: "Cari dari seluruh Indonesia"
├── Destination grouping:
│   ├── Kalau destinasi A (di NTT) tidak ada match → cari destinasi dalam 200km
│   ├── "Traveler yang mau ke Flores juga tertarik ke: Komodo, Ende, Maumere"
│   └── Group nearby destinations → increase chance of match
├── Suggest open trip:
│   ├── Kalau matching 0 → "Coba lihat open trip ke daerah ini"
│   ├── Agen travel yang punya paket ke NTT → tampilkan
│   └── Funnel: matching fail → marketplace → value tetap tersampaikan
├── Notify on match:
│   ├── Simpan search → cron job cek tiap 6 jam
│   ├── Kalau ada match baru → push notification
│   └── "Ada traveler baru yang mau ke destinasi yang sama!"
└── Community seeding:
    ├── Partner dengan komunitas traveler lokal (Backpacker NTT, dll)
    ├── Mereka daftar → seed demand di daerah tersebut
    └── Incentive: badge "Pioneer" untuk user pertama di setiap provinsi
```

---

## Post-Trip Feedback Loop

```
Setelah trip selesai (3 hari grace period):

Step 1: Rating prompt
├── Push notif: "Gimana trip ke {destinasi}? Rate partner kamu!"
├── Form: score (1-5 bintang) + review text (opsional)
├── Anti-bias: tidak bisa lihat rating dari partner sebelum submit sendiri
└── Deadline: 7 hari setelah trip selesai

Step 2: Rating processing
├── Update ratee.rating_avg = weighted average
│   weighted_avg = (old_avg * old_count + new_score) / (old_count + 1)
├── Update ratee.rating_count += 1
├── Recalculate trust_score
└── Tabel: ratings

Step 3: Feedback loop ke matching
├── User yang diberi rating tinggi (4-5) → prioritaskan di matching berikutnya
│   ├── "Preferred partner" boost: +5% compatibility score
│   └── Simpan di tabel: user_preferred_partners (user_id, partner_id, score)
├── User yang diberi rating rendai (1-2) → deprioritize
│   ├── "Deprioritized partner": -10% compatibility score
│   ├── Setelah 3 rating rendah dari user berbeda → admin review
│   └── Simpan di tabel: user_deprioritized (user_id, partner_id, score, reason)
├── Destination feedback:
│   ├── Rating destinasi dari banyak user → update destination.rating_avg
│   ├── Review keywords extraction (Phase 2): "jalan rusak", "pemandangan bagus"
│   └── Low rating streak → flag destinasi untuk curator review
└── Matching algorithm learning:
    ├── Track: "User A rate User B tinggi, apa kesamaan mereka?"
    ├── Analisis: interest overlap, age proximity, budget similarity
    └── Adjust bobot compatibility factor berdasarkan data (Phase 2 ML)
```

---

## Performance Benchmark

```
Target latency per query type:

| Query | Users | Target | Max Acceptable |
|---|---|---|---|
| Recommendation (basic filter) | 1K | < 200ms | 500ms |
| Recommendation (basic filter) | 50K | < 500ms | 2s |
| Trip matching (6-factor scoring) | 1K | < 1s | 3s |
| Trip matching (6-factor scoring) | 50K | < 3s | 10s |
| Destination search (text + geo) | any | < 300ms | 1s |
| Chat history load (50 messages) | any | < 200ms | 500ms |
| Leaderboard load | any | < 100ms | 300ms |
| XP calculation (trigger) | any | < 50ms | 200ms |

Optimization techniques:
├── PostGIS spatial index: GIST index pada location column
├── Materialized view: leaderboard_cache (refresh setiap jam)
├── Composite index: pada trip_requests (status, destination_id, date_range)
├── Connection pooling: Supabase built-in (PgBouncer)
├── Query caching: Edge Function cache (TTL 5 menit untuk destinasi populer)
├── Lazy loading: gambar dan map tiles dimuat saat scroll
└── Prefetch: preload destinasi detail saat user hover card
```
