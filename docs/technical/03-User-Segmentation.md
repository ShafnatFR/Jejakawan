# 03 — User Segmentation & Roles

## Tipe User

Sistem Jejakawan punya 3 segmen user yang menentukan experience, fitur yang terlihat, dan flow utama.

---

### SEGMENT 1: Wisatawan Biasa (Tourist)

```
Profil:
├── Usia: 18-40 tahun
├── Motivasi: liburan, healing, konten sosmed
├── Behavior: booking paket, cari teman jalan, ikut open trip
├── Tech savviness: medium (biasa pakai Traveloka/Gojek)
└── Spending: medium-high (Rp500rb-5jt per trip)

Fitur yang terlihat:
├── Personalized Recommendation (destinasi populer + hidden gems)
├── Trip Matching (cari teman jalan)
├── Open Trip Marketplace (booking paket travel)
├── Itinerary Builder (susun rencana bareng grup)
├── Weather & Seasonal Alert
├── Profil + Rating & Review
└── Subscription Pass (Weekly/Monthly/Exclusive)

Fitur yang TIDAK terlihat / tersier:
├── Claim Misi (bukan fokus utama)
├── Leaderboard Explorer (bukan motivasi utama)
└── Local Host booking (opsional, bukan core flow)

User Flow utama:
  Onboarding → Isi Preferensi → Lihat Rekomendasi → Pilih Destinasi
  → Cari Teman (Trip Matching) / Booking Open Trip → Trip → Rating
```

---

### SEGMENT 2: Explorer / Adventurer

```
Profil:
├── Usia: 18-35 tahun
├── Motivasi: eksplorasi, pengalaman unik, validasi sosial
├── Behavior: solo travel, cari underrated spot, kumpulkan XP/badge
├── Tech savviness: high (gamified apps, early adopter)
└── Spending: variable (budget traveler sampai premium)

Fitur yang terlihat:
├── Claim Misi ke Underrated Spot
├── Gamifikasi (XP, badge, level, leaderboard)
├── Local Guide/Host booking
├── Offline Mode (peta + itinerary)
├── UGC Upload (foto/vlog untuk klaim XP)
├── Preferensi-based Recommendation (fokus hidden gems)
└── Community Feed (berbagi pengalaman)

Fitur yang TIDAK terlihat / tersier:
├── Open Trip Marketplace (mereka lebih mandiri)
├── Trip Matching (opsional, bukan utama)
└── Itinerary Builder (mereka punya style sendiri)

User Flow utama:
  Onboarding → Pilih "Explorer Mode" → Lihat Misi Aktif
  → Pilih Misi → Navigasi ke Spot → Upload Bukti (foto/vlog)
  → Dapat XP + Badge → Share ke Community Feed
```

---

### SEGMENT 3: Mitra / Partner

```
Tipe A: Local Guide / Host
├── Profil: warga lokal di daerah wisata, usia 20-50
├── Motivasi: penghasilan tambahan, promosi daerah
├── Flow: Daftar → Verifikasi → Buat profil guide → Terima booking
└── Dashboard: jadwal booking, pendapatan, rating

Tipe B: Agen Travel
├── Profil: perusahaan travel, organizer open trip
├── Motivasi: jual paket, dapat customer baru
├── Flow: Daftar → Verifikasi bisnis → Upload paket → Terima booking
└── Dashboard: listing paket, booking masuk, pendapatan

Tipe C: Pemda / Dinas Pariwisata
├── Profil: pemerintah daerah yang mau promosi destinasi
├── Motivasi: meningkatkan kunjungan wisata daerah
├── Flow: Partnership → Banner/promoted listing → Analytics kunjungan
└── Dashboard: data kunjungan, efektivitas promosi
```

---

## Sistem Role & Permission

```
Role Hierarchy:
├── user_tourist      → akses fitur wisatawan biasa
├── user_explorer     → akses fitur explorer + gamifikasi
├── partner_guide     → dashboard guide, terima booking
├── partner_agency    → dashboard agen, manage paket
├── partner_gov       → dashboard analytics, promoted listing
├── admin_moderator   → review konten, handle laporan
├── admin_trust       → verifikasi KTP, review insiden
└── admin_super       → full access, manage semua role

Tabel: user_roles
┌──────────────┬────────────────────────────────────┐
│ user_id      │ UUID (FK ke users)                 │
│ role         │ ENUM (tourist, explorer, guide,    │
│              │  agency, gov, mod, trust, super)   │
│ status       │ ENUM (active, suspended, pending)  │
│ granted_at   │ TIMESTAMP                          │
│ granted_by   │ UUID (FK ke users, admin yang kasih)│
└──────────────┴────────────────────────────────────┘

Akses Control:
├── Row Level Security (RLS) di setiap tabel
├── User hanya bisa READ data sendiri
├── User bisa READ data publik (destinasi, paket, profil publik)
├── Guide/Agency bisa WRITE data sendiri (profil, listing)
├── Admin bisa READ/WRITE semua data
└── KTP data: hanya bisa diakses admin_trust + user pemilik
```

---

## Onboarding Flow per Segment

### Tourist Onboarding
```
Screen 1: Welcome
├── "Temukan perjalanan yang sesuai denganmu"
├── Button: "Mulai" / "Masuk"
└── Social login: Google, Email, Phone OTP

Screen 2: Pilih Segmentasi
├── "Aku mau..." 
├── Option A: "Jalan-jalan & cari teman" → Tourist path
├── Option B: "Eksplorasi tempat baru" → Explorer path
└── Bisa diubah nanti di settings

Screen 3: Preferensi (Tourist)
├── Minat: alam, kuliner, budaya, kota, adventure (multi-select)
├── Budget range: slider (Rp100rb - Rp10jt)
├── Moda transportasi: motor, mobil, umum, pesawat (multi-select)
├── Lokasi saat ini: auto-detect GPS / input manual
└── Frekuensi travel: jarang, kadang, sering

Screen 4: Verifikasi (Opsional tapi Dorong)
├── "Verifikasi untuk keamanan trip matching"
├── Input: nomor HP (OTP)
├── Input: upload KTP (opsional di awal, wajib sebelum match)
└── Skip untuk sekarang → bisa nanti

Screen 5: Home
└── Masuk ke personalized recommendation
```

### Explorer Onboarding
```
Screen 1-2: Sama seperti Tourist

Screen 3: Explorer Profile
├── Level explorer: "Pemula" (default)
├── Mis pertama: "Kunjungi 3 underrated spot dalam 30 hari"
├── Penjelasan XP & badge system
└── Banner: "Jadilah penjelajah pertama di daerahmu"

Screen 4: Sama seperti Tourist

Screen 5: Home Explorer
└── Masuk ke halaman Misi + Rekomendasi underrated spot
```

---

## Segment Switching

User bisa switch antara Tourist ↔ Explorer kapan saja:
```
Settings → "Ganti Mode Perjalanan"
├── Tourist → Explorer: langsung switch, XP & badge tetap tersimpan
├── Explorer → Tourist: switch, leaderboard hidden, XP tetap jalan
└── Tidak bisa hilangkan XP/badge yang sudah didapat
```

Alasan: user mungkin kadang mau trip santai (tourist), kadang mau adventure (explorer). Tidak perlu buat akun baru.

---

## Profil User (Unified)

```
Tabel: user_profiles
┌──────────────────┬─────────────────────────────┐
│ user_id          │ UUID (FK ke auth.users)      │
│ display_name     │ VARCHAR(100)                  │
│ avatar_url       │ TEXT (Supabase Storage)       │
│ bio              │ TEXT (max 500 char)           │
│ current_mode     │ ENUM (tourist, explorer)      │
│ preferred_interests │ TEXT[] (array)             │
│ budget_range     │ INT4RANGE                     │
│ transport_mode   │ TEXT[]                        │
│ home_location    │ GEOGRAPHY(Point, 4326)        │
│ current_location │ GEOGRAPHY(Point, 4326)        │
│ ktp_verified     │ BOOLEAN (default false)       │
│ phone_verified   │ BOOLEAN (default false)       │
│ rating_avg       │ DECIMAL(3,2) (default 0)      │
│ rating_count     │ INT (default 0)               │
│ xp_total         │ INT (default 0)               │
│ level            │ INT (default 1)               │
│ badge_count      │ INT (default 0)               │
│ subscription     │ ENUM (free, weekly, monthly,  │
│                  │  exclusive)                    │
│ created_at       │ TIMESTAMP                     │
│ updated_at       │ TIMESTAMP                     │
└──────────────────┴─────────────────────────────┘

Profil Publik (bisa dilihat user lain):
├── display_name, avatar, bio
├── rating_avg, rating_count
├── xp_total, level, badge_count
├── Jumlah trip selesai
└── TIDAK termasuk: KTP, HP, budget, lokasi real-time

Profil saat Trip Matching (bisa dilihat calon match):
├── Semua profil publik +
├── preferred_interests
├── transport_mode
├── Kompatibility score (dihitung sistem)
- TIDAK termasuk: KTP, HP, budget, lokasi real-time
```

---

## User Personas

### Persona 1: Rina (Tourist)
```
Usia: 23 tahun, mahasiswa semester 7, Jakarta
Motivasi: pengen liburan ke Bandung tapi teman-teman sibuk semua
Behavior: scrolling TikTok tiap hari, suka lihat konten travel, FOMO-an
Pain point: males solo travel (takut, boring), gak tau tempat bagus selain yang viral
Budget: Rp500rb-1jt per trip (uang tabungan)
Tech: heavy smartphone user, biasa pakai Traveloka & Gojek
Trigger download: lihat teman post foto di underrated spot, penasaran
Harapan: "Aku pengen jalan tapi gak sendirian, dan gak ke tempat yang itu-itu aja"
```

### Persona 2: Dimas (Explorer)
```
Usia: 27 tahun, freelance designer, Bandung
Motivasi: suka explore tempat yang belum pernah dikunjungi orang
Behavior: solo traveler 2-3x sebulan, suka fotografi landscape, aktif di komunitas hiking
Pain point: capek ke tempat yang udah rame, susah cari info tempat baru yang valid
Budget: fleksibel (Rp200rb-3jt per trip)
Tech: early adopter, suka app gamifikasi (Strava, Duolingo)
Trigger download: gamifikasi + badge system
Harapan: "Aku pengen challenge diri sendiri dan dapat pengakuan atas eksplorasiku"
```

### Persona 3: Bu Sari (Local Guide)
```
Usia: 42 tahun, ibu rumah tangga merangkap penjual kerajinan, Sukabumi
Motivasi: tambahan penghasilan dari jadi guide di Curug Cikaso
Behavior: aktif di Facebook, baru belajar pakai GoFood untuk jualan
Pain point: wisatawan jarang yang tau desanya, promosi mulut-ke-mulut aja
Budget: n/a (dia yang cari uang)
Tech: basic smartphone user, perlu bantuan onboarding
Trigger download: dikenalkan oleh dinas pariwisata daerah
Harapan: "Kalau ada yang datang ke sini, saya bisa dapat uang jadi guide"
```

---

## Accessibility Considerations

```
WCAG 2.1 Level A (minimum):
├── Color contrast ratio >= 4.5:1 untuk text
├── Semua image punya alt text
├── Keyboard navigation untuk semua interaksi
├── Focus visible indicator
└── Error messages yang deskriptif (bukan hanya "Error")

Indonesia-specific:
├── Font size minimum 14px (banyak user HP murah dengan layar kecil)
├── Touch target minimum 44x44px (Apple HIG standard)
├── Support mode hemat data (kompres gambar, kurangi animasi)
├── Bahasa Indonesia default, opsi English di settings
└── Format: Rp 1.000.000 (titik sebagai pemisah ribuan)

Destinasi accessibility info:
├── Wheelchair friendly: ya/tidak/sebagian
├── Akses transportasi umum: tersedia/tidak
├── Jarak jalan kaki dari parkir: X meter
└── Info ini input oleh admin/curator, tampil di detail destinasi
```

---

## User Retention Strategy per Segment

```
TOURIST:
├── Hook: "Temukan destinasi baru yang belum pernah kamu kunjungi"
├── First value: rekomendasi personal dalam 30 detik setelah onboarding
├── Retention loop:
│   ├── Setelah trip → push notif: "Rate trip kamu & dapat XP!"
│   ├── Minggu berikutnya: "Destinasi baru yang cocok untukmu"
│   ├── Bulan berikutnya: "Long weekend sebentar lagi, sudah punya rencana?"
│   └── Seasonal: "Liburan sekolah! Ini rekomendasi untuk keluarga"
├── Churn prevention:
│   ├── 7 hari tidak buka → notifikasi "Kangen jalan-jalan?"
│   ├── 30 hari tidak buka → email "Banyak destinasi baru menunggumu"
│   └── 90 hari tidak buka → survey "Kenapa kamu berhenti?"
└── Target: D7 retention > 40%, D30 retention > 20%

EXPLORER:
├── Hook: "Jadi penjelajah pertama di daerahmu"
├── First value: misi pertama dalam 1 menit setelah onboarding
├── Retention loop:
│   ├── Daily: "Misi baru tersedia!" (kalau ada misi baru)
│   ├── Weekly: "Leaderboard update! Kamu di posisi #X"
│   ├── Setelah level up: "Selamat! Level X. Badge baru menunggu"
│   └── Monthly: "Rekap bulan ini: X spot, Y XP, Z badge"
├── Churn prevention:
│   ├── Streak break (3 hari tidak aktif): "Jangan putus streak kamu!"
│   ├── XP stagnan: "Ada misi mudah untuk tambah XP"
│   └── Level hampir naik: "Butuh X XP lagi untuk level berikutnya!"
└── Target: D7 retention > 50%, D30 retention > 30%

MITRA:
├── Hook: "Dapatkan penghasilan dari keindahan daerahmu"
├── First value: profil guide/listing langsung aktif setelah verifikasi
├── Retention loop:
│   ├── Ada booking baru → notifikasi langsung
│   ├── Mingguan: "Rekap penghasilan minggu ini"
│   ├── Bulanan: "Tips meningkatkan rating kamu"
│   └── Quarterly: "Update foto & deskripsi untuk lebih menarik"
├── Churn prevention:
│   ├── 30 hari tanpa booking → "Coba update harga/promo"
│   ├── Rating turun → "Saran dari traveler: ..."
│   └── Tidak login 14 hari → email reminder
└── Target: 80% mitra aktif setelah 3 bulan
```
