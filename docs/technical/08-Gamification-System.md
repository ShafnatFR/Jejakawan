# 08 — Gamification System

## Overview

Sistem gamifikasi Jejakawan punya 4 komponen: XP, Level, Badge, dan Misi. Tujuannya: retensi user + dorong eksplorasi ke destinasi underrated.

---

## XP System

### XP Sources (Sumber XP)

| Aksi | XP Dasar | Multiplier | Catatan |
|---|---|---|---|
| Upload konten ke underrated spot | +100 | x1.5-2x | Via subscription |
| Selesaikan misi (semua spot tercapai) | +500 | x1.5-2x | Completion bonus |
| Trip selesai (dari trip matching) | +50 | x1 | Per trip |
| Beri review valid (setelah trip) | +20 | x1 | Per review |
| Upload konten ke spot populer | +25 | x1.5-2x | Lebih sedikit karena bukan fokus |
| Referral user baru (yang verif KTP) | +200 | x1 | Satu kali per referral |
| Donasi ke developer | +100 | x1 | Per donasi |

### XP Multiplier (per Subscription)

| Plan | Multiplier |
|---|---|
| Free | x1.0 |
| Weekly | x1.5 |
| Monthly | x1.5 |
| Exclusive | x2.0 |

```
Contoh perhitungan:
User Monthly upload foto di underrated spot:
  Base XP: 100
  Multiplier: x1.5
  Final XP: 100 x 1.5 = 150 XP
```

### XP Redemption

| XP Required | Voucher |
|---|---|
| 500 | Alfamart Rp15.000 |
| 1.000 | Alfamart Rp25.000 |
| 2.000 | Hotel discount Rp50.000 |
| 5.000 | Hotel discount Rp150.000 |
| 10.000 | Voucher travel Rp500.000 |

```
Rules:
├── XP yang diredeem tidak bisa dikembalikan
├── Voucher berlaku 30 hari setelah redeem
├── Maksimal 3 redemption per bulan
├── Minimum XP tersisa setelah redeem: 0 (boleh habis)
└── Voucher dikirim ke email + muncul di app
```

---

## Level System

### XP Threshold per Level

```
Level 1:    0 XP        (default saat daftar)
Level 2:  200 XP
Level 3:  500 XP
Level 4: 1.000 XP
Level 5: 2.000 XP
Level 6: 3.500 XP
Level 7: 5.500 XP
Level 8: 8.000 XP
Level 9: 12.000 XP
Level 10: 18.000 XP

Formula: xp_required(level) = 100 * level^2 + 100 * level
  Level 2: 100*4 + 200 = 600... hmm, let me use a simpler curve.
```

### Level Progression Formula

```
xp_for_level(n) = round(100 * n * (n + 1) / 2)

n=1:  100    (butuh 100 XP untuk naik ke level 2)
n=2:  300    (butuh total 300 XP untuk level 3)
n=3:  600
n=4:  1.000
n=5:  1.500
n=6:  2.100
n=7:  2.800
n=8:  3.600
n=9:  4.500
n=10: 5.500

Atau simplified (lebih mudah dijelaskan):
Level  1:      0 XP (default)
Level  2:    200 XP
Level  3:    500 XP
Level  4:  1.000 XP
Level  5:  2.000 XP
Level  6:  3.500 XP
Level  7:  5.500 XP
Level  8:  8.000 XP
Level  9: 12.000 XP
Level 10: 18.000 XP
```

### Level Titles

| Level | Title |
|---|---|
| 1 | Pemula |
| 2 | Penjelajah Muda |
| 3 | Penjelajah |
| 4 | Petualang |
| 5 | Petualang Berpengalaman |
| 6 | Penjelajah Ulung |
| 7 | Explorer |
| 8 | Master Explorer |
| 9 | Petualang Legendaris |
| 10 | Penjelajah Abadi |

### Level Benefits (cosmetic + minor functional)

```
Level 1-3:  Dasar, akses semua fitur free
Level 4-6:  Unlock "Explorer Mode" badge di profil
            Akses misi regional (lebih banyak pilihan)
Level 7-9:  Badge eksklusif "Elite Explorer"
            Prioritas di trip matching queue
            Akses misi nasional
Level 10:   Badge "Penjelajah Abadi" (langka)
            Lifetime priority matching
            Undangan event eksklusif (kalau ada)
```

---

## Badge System

### Badge Categories

```
EXPLORER BADGES (dari misi):
├── 🏔️ Penjelajah Jawa Barat    — Selesaikan 3 misi di Jabar
├── 🏖️ Beachcomber              — Kunjungi 5 pantai underrated
├── 🌿 Eco-Warrior               — Kunjungi 5 destinasi alam
├── 🏛️ Heritage Hunter           — Kunjungi 5 situs budaya
├── 🍜 Foodie Explorer            — Kunjungi 5 spot kuliner lokal
├── 🏔️ Mountain Conqueror         — Kunjungi 3 gunung/bukit
└── 🗺️ Nusantara Explorer         — Kunjungi destinasi di 5 provinsi

SOCIAL BADGES:
├── 🤝 First Match               — Pertama kali trip matching
├── 👥 Team Player               — Selesaikan 5 trip bareng grup
├── ⭐ Trusted Traveler           — Rating 4.5+ dari 10 review
├── 📸 Content Creator             — Upload 20 konten terverifikasi
└── 🎯 Mission Master             — Selesaikan 10 misi

ACHIEVEMENT BADGES:
├── 🌅 Early Bird                 — Daftar dalam 30 hari pertama launch
├── 💎 Premium Member              — Subscribe ke Exclusive Pass
├── ❤️ Supporter                   — Donasi ke developer
├── 🔒 Verified Traveler           — Verifikasi KTP
├── 📝 Reviewer                    — Beri 20 review
└── 🏆 Level 10                    — Capai level 10

SEASONAL / EVENT BADGES:
├── Tahun Baru Explorer            — Trip saat 31 Des - 2 Jan
├── Ramadhan Traveler              — Trip saat Ramadhan
└── HUT RI Explorer                — Trip saat 17 Agustus
```

### Badge Unlock Logic

```sql
-- Contoh logic unlock badge "Beachcomber"
-- Trigger: setiap kali mission_submission di-approve

-- Cek: apakah user sudah punya 5 submission approved di destinasi kategori "pantai"?
SELECT COUNT(*) FROM mission_submissions ms
JOIN destinations d ON ms.destination_id = d.id
WHERE ms.user_id = :user_id
  AND ms.status = 'approved'
  AND 'pantai' = ANY(d.category);

-- Kalau >= 5 dan belum punya badge ini → unlock
```

---

## Misi System

### Misi Template

```
Misi: "Jelajahi Hidden Gems Jawa Barat"
├── Target: Kunjungi 3 destinasi underrated di Jawa Barat
├── Deadline: 30 hari setelah klaim
├── Reward: 500 XP + Badge "Penjelajah Jawa Barat"
├── Kandidat spot: (list destinasi yang is_underrated = true AND province = 'Jawa Barat')
└── Submit: foto di lokasi + EXIF GPS match

Misi: "Beach Hopper"
├── Target: Kunjungi 5 pantai underrated di Indonesia
├── Deadline: 60 hari
├── Reward: 800 XP + Badge "Beachcomber"
└── Kandidat: destinasi kategori "pantai" + "underrated"

Misi: "First Timer"
├── Target: Upload 1 foto ke destinasi manapun
├── Deadline: 7 hari
├── Reward: 50 XP + Badge "First Step"
└── Untuk: user baru (< 7 hari sejak daftar)
```

### Misi Validasi (Upload Bukti)

```
Validasi otomatis:
├── 1. EXIF GPS ada? → Cek koordinat dalam radius 500m dari destinasi
├── 2. EXIF timestamp ada? → Cek masih dalam deadline misi
├── 3. File format valid? → jpg, png, mp4
├── 4. File size valid? → max 10MB foto, 50MB video
└── 5. NSFW basic check (opsional Phase 2)

Validasi manual (Trust & Safety):
├── 1. Foto relevan dengan destinasi? (bukan screenshot Google)
├── 2. Foto bukan duplikat dari user lain
├── 3. Konten tidak melanggar aturan
└── SLA: review dalam 24 jam

Status flow:
pending → (auto-check pass) → pending_review
pending_review → (manual approve) → approved (+ XP dikasih)
pending_review → (manual reject) → rejected (alasan dikirim ke user)
rejected → user bisa upload ulang (count reset)
```

---

## Leaderboard

```
Scope: per provinsi (atau nasional)
Metric: XP total (tidak pernah decay)
Update: setiap jam via cron job
Visible: top 50 + posisi user sendiri

Tabel: leaderboard_cache
├── region (provinsi)
├── user_id
├── xp_total
├── level
├── rank
└── updated_at

Query:
SELECT rank, user_id, display_name, xp_total, level
FROM leaderboard_cache
WHERE region = 'Jawa Barat'
ORDER BY rank ASC
LIMIT 50;
```

---

## XP Decay & Anti-Abuse

```
TIDAK ada XP decay (XP yang didapat tetap selamanya).
Ini sengaja: supaya user merasa progress mereka "aman" dan tidak rugi kalau tidak aktif.

Anti-abuse rules:
├── Upload foto yang sama ke 2 misi berbeda → hanya 1 XP yang dikasih
├── Upload foto yang sudah dipakai user lain → rejected
├── GPS spoofing detection (koordinat terlalu presisi / dari emulator)
├── Rate limit: max 10 upload misi per hari
├── User banned → XP & badge di-freeze (tidak dihapus, tapi tidak bisa earn baru)
└── Referral abuse: 1 referral per unique device + phone number
```

---

## Implementasi Database

```sql
-- Trigger function: setelah mission_submission approved
CREATE OR REPLACE FUNCTION award_xp_on_submission_approved()
RETURNS TRIGGER AS $$
DECLARE
  v_multiplier DECIMAL(3,2);
  v_base_xp INT;
  v_final_xp INT;
BEGIN
  IF NEW.status = 'approved' AND OLD.status != 'approved' THEN
    -- Dapatkan multiplier dari subscription user
    SELECT CASE subscription
      WHEN 'weekly' THEN 1.5
      WHEN 'monthly' THEN 1.5
      WHEN 'exclusive' THEN 2.0
      ELSE 1.0
    END INTO v_multiplier
    FROM user_profiles WHERE id = NEW.user_id;

    v_base_xp := 100; -- per spot
    v_final_xp := ROUND(v_base_xp * v_multiplier);

    -- Insert XP log
    INSERT INTO xp_logs (user_id, amount, source, reference_id, multiplier)
    VALUES (NEW.user_id, v_final_xp, 'mission_spot', NEW.id, v_multiplier);

    -- Update user XP total
    UPDATE user_profiles
    SET xp_total = xp_total + v_final_xp,
        updated_at = NOW()
    WHERE id = NEW.user_id;

    -- Update mission progress
    UPDATE user_missions
    SET progress = progress + 1
    WHERE id = NEW.user_mission_id;

    -- Cek apakah misi selesai
    -- (dicek oleh terpisah oleh cron job atau trigger lanjutan)
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_award_xp
  AFTER UPDATE ON mission_submissions
  FOR EACH ROW
  EXECUTE FUNCTION award_xp_on_submission_approved();
```

---

## Admin Gamification Dashboard

```
Dashboard yang perlu dipantau admin:

Daily Metrics:
├── Total XP earned today (semua user)
├── Top 10 earners hari ini
├── Mission submissions pending review: {count}
├── Mission completion rate hari ini: {percentage}
└── Badge unlocks hari ini: {count}

Weekly Metrics:
├── Active missions: {count} (claimed tapi belum selesai)
├── Expired missions: {count} (deadline lewat)
├── XP redemption: {count} vouchers, total {amount} XP
├── Leaderboard changes: user baru masuk top 10
└── Engagement: user yang earn XP hari ini / total DAU

Admin Actions:
├── Create/edit/hide misi baru
├── Adjust XP values per activity (kalau terlalu mudah/sulit)
├── Approve/reject mission submissions manual
├── Revoke badge (kalau fraud terdeteksi)
├── Freeze XP system (emergency exploit)
└── Export data: CSV semua XP logs untuk audit
```

---

## Engagement Analytics Targets

```
North Star Metrics:
├── DAU (Daily Active Users): target > 10% dari total registered
├── MAU (Monthly Active Users): target > 30% dari total registered
├── DAU/MAU ratio: target > 20% (stickiness)
├── Mission completion rate: target > 30%
└── XP redemption rate: target > 5% dari user yang punya cukup XP

Cohort Analysis:
├── Week 1 retention: target > 50% (user kembali dalam 7 hari)
├── Week 4 retention: target > 25%
├── Week 12 retention: target > 15%
└── Per segment: explorer retention HARUS lebih tinggi dari tourist

Funnel Analysis:
├── Onboarding completion: target > 70% (dari register sampai home)
├── First action rate: target > 40% (lihat rekomendasi / klaim misi dalam 24 jam)
├── First trip rate: target > 15% (match/book dalam 7 hari)
├── Second trip rate: target > 50% (dari yang first trip)
└── Subscription conversion: target > 5% (dari total user)

Gamification-specific:
├── Average XP per user per week: target > 100
├── Badge distribution: tidak boleh > 80% user punya badge yang sama
├── Level distribution: normal curve, bukan skewed
├── Mission claim → completion ratio: target > 25%
└── Streak users (7-day): target > 10% dari DAU
```

---

## A/B Testing Plan

```
Phase 1 (launch): baseline tanpa A/B
Phase 2 (setelah 1000 user): mulai A/B testing

Tests yang perlu dijalankan:

Test 1: XP Amount
├── Hypothesis: XP 100/spot terlalu mudah → badge cepat didapat → meaningless
├── Variant A: 100 XP per spot (current)
├── Variant B: 75 XP per spot (lebih challenging)
├── Metric: mission completion rate, retention D7
├── Sample: 50/50 split dari user baru
└── Duration: 2 minggu

Test 2: Onboarding Length
├── Hypothesis: onboarding 5 screen terlalu panjang → drop-off tinggi
├── Variant A: 5 screen (current)
├── Variant B: 3 screen (skip preferensi, progressive profiling later)
├── Metric: onboarding completion rate, first action rate
├── Sample: 50/50 split
└── Duration: 2 minggu

Test 3: Match Display
├── Hypothesis: menampilkan compatibility score bikin user lebih selective
├── Variant A: tampilkan score (87% match)
├── Variant B: sembunyikan score (hanya tampilkan profil)
├── Metric: match invitation sent rate, acceptance rate
├── Sample: 50/50 split
└── Duration: 3 minggu

Test 4: Subscription Pricing
├── Hypothesis: Rp29.000/bulan terlalu mahal untuk market Indonesia
├── Variant A: Rp29.000/bulan (current)
├── Variant B: Rp19.900/bulan (lower)
├── Metric: conversion rate, total revenue, churn rate
├── Sample: 50/50 split
└── Duration: 4 minggu

Implementation:
├── Tool: PostHog feature flags (built-in A/B testing)
├── Segmentation: berdasarkan user_id hash (konsistent)
├── Statistical significance: p < 0.05
└── Winner: auto-rollout ke 100% setelah significance tercapai
```

---

## Seasonal Event System

```
Event Template:
├── name: "Tahun Baru Explorer 2027"
├── description: "Jelajahi 3 destinasi saat liburan tahun baru"
├── start_date: "2026-12-25"
├── end_date: "2027-01-05"
├── bonus_xp_multiplier: 2.0 (double XP selama event)
├── special_badge: {name: "Tahun Baru Explorer", icon: "🎆"}
├── special_missions: [
│   {title: "Sunrise Hunter", target: 3 spot sunrise, xp: 800},
│   {title: "Kuliner Akhir Tahun", target: 5 spot kuliner, xp: 500}
│ ]
└── auto_activate: true (berdasarkan tanggal)

Daftar event tahunan:
├── 1 Jan: Tahun Baru Explorer
├── 14 Feb: Romantic Getaway (couple trip special)
├── 17 Agustus: HUT RI Explorer (destinasi bersejarah)
├── Ramadhan: Spiritual Journey (destinasi religi)
├── Juni-Juli: School Holiday Explorer
└── Desember: Year-End Adventure

Admin bisa buat event baru via dashboard:
├── Form: name, dates, bonus multiplier, badge, missions
├── Preview: tampilkan seperti apa event di app
├── Schedule: bisa set jauh-jauh hari, auto-activate
└── Analytics: per-event metrics (participation, completion, XP earned)
```
