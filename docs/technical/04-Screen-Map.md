# 04 — Screen Map & Navigation

## Arsitektur Navigasi

```
Root
├── Auth Stack (belum login)
│   ├── Splash Screen
│   ├── Login Screen
│   ├── Register Screen
│   └── Onboarding Flow (3-5 screen)
│
├── Main Tab Navigator (sudah login)
│   ├── Tab 1: Home / Discover
│   ├── Tab 2: Explore (jalan + map)
│   ├── Tab 3: Match / Trip (sosial)
│   ├── Tab 4: Profile
│   └── Tab 5: More / Settings (drawer)
│
└── Modal / Overlay Stack
    ├── Detail Destinasi
    ├── Detail Paket Open Trip
    ├── Profil User Lain
    ├── Chat Room
    ├── Itinerary Builder
    ├── Upload Konten
    ├── Klaim Misi
    ├── Verifikasi KTP
    ├── Payment / Subscription
    └── Report User / Konten
```

---

## Tab 1: Home / Discover

### Tourist Mode
```
┌──────────────────────────────┐
│  Header: "Halo, {nama}!"     │
│  Search bar (destinasi)      │
├──────────────────────────────┤
│  📍 Lokasi: Bandung          │
│  [Ganti Mode: Explorer ✏️]   │
├──────────────────────────────┤
│  "Rekomendasi untukmu"       │
│  ┌────┐ ┌────┐ ┌────┐       │
│  │Card│ │Card│ │Card│  → scroll│
│  │ 1  │ │ 2  │ │  3 │       │
│  └────┘ └────┘ └────┘       │
├──────────────────────────────┤
│  "Trip yang bisa kamu ikuti" │
│  ┌──────────────────────┐    │
│  │ Open Trip Card 1     │    │
│  │ Open Trip Card 2     │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  "Wisatawan lain juga pergi" │
│  ┌──────────────────────┐    │
│  │ Trending Destinasi   │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  ⚡ Weather Alert Banner     │
│  (muncul kalau relevan)      │
└──────────────────────────────┘
```

### Explorer Mode
```
┌──────────────────────────────┐
│  Header: "Halo, Penjelajah!" │
│  Level Badge: Lv.3           │
│  XP Bar: ████████░░ 800/1000 │
├──────────────────────────────┤
│  [Ganti Mode: Tourist ✏️]    │
├──────────────────────────────┤
│  "Misi Aktif"                │
│  ┌──────────────────────┐    │
│  │ 🏔️ Kunjungi 3 spot   │    │
│  │ underrated di Jabar   │    │
│  │ Progress: 1/3 █████░░│    │
│  │ [Klaim Misi]         │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  "Spot Underrated di Sekitar"│
│  ┌────┐ ┌────┐ ┌────┐       │
│  │Card│ │Card│ │Card│  → scroll│
│  └────┘ └────┘ └────┘       │
├──────────────────────────────┤
│  "Leaderboard Regional"      │
│  1. @explorer123 - 2,400 XP │
│  2. @petualang  - 2,100 XP │
│  3. Kamu        - 800 XP   │
└──────────────────────────────┘
```

### Komponen Card Destinasi
```
┌──────────────────────────┐
│ [Gambar Destinasi]       │
│                          │
│ Nama Destinasi           │
│ 📍 Lokasi • ⭐ 4.5      │
│ 💰 Est. Rp200rb          │
│ 🏷️ [Alam] [Hidden Gem]  │
│                          │
│ "3 wisatawan akan ke     │
│  sana minggu ini"        │
└──────────────────────────┘
```

---

## Tab 2: Explore (Map + Search)

```
┌──────────────────────────────┐
│  Search: "Cari destinasi..." │
│  Filter: [Jarak] [Budget]    │
│          [Kategori] [Rating] │
├──────────────────────────────┤
│                              │
│    ┌─────────────────────┐   │
│    │                     │   │
│    │   MAP VIEW          │   │
│    │   (Google Maps)     │   │
│    │                     │   │
│    │  📍 📍  📍          │   │
│    │     📍    📍        │   │
│    │  📍      📍  📍     │   │
│    │                     │   │
│    └─────────────────────┘   │
│                              │
│  ┌──────────────────────┐    │
│  │ [List View] [Map View]│   │
│  └──────────────────────┘    │
│                              │
│  Destinasi Terdekat:         │
│  ┌──────────────────────┐    │
│  │ Card 1 (2.3 km)      │    │
│  │ Card 2 (5.1 km)      │    │
│  │ Card 3 (8.7 km)      │    │
│  └──────────────────────┘    │
└──────────────────────────────┘
```

---

## Tab 3: Match / Trip (Sosial)

### Tourist Mode
```
┌──────────────────────────────┐
│  "Cari Teman Jalan"          │
│  [+ Buat Trip Baru]          │
├──────────────────────────────┤
│  Trip Aktif:                 │
│  ┌──────────────────────┐    │
│  │ 🗺️ Bandung, 15-17 Nov│    │
│  │ 👥 3/4 orang         │    │
│  │ Status: Menunggu match│    │
│  │ [Lihat Detail]       │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  Undangan Match Masuk:       │
│  ┌──────────────────────┐    │
│  │ @andi mengajak ke    │    │
│  │ Yogyakarta, 20 Nov   │    │
│  │ Kompatibilitas: 85%  │    │
│  │ [Terima] [Tolak]     │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  Chat Rooms:                 │
│  ┌──────────────────────┐    │
│  │ 💬 Grup Bandung Trip │    │
│  │ 💬 @andi - Yogyakarta│    │
│  └──────────────────────┘    │
└──────────────────────────────┘
```

### Explorer Mode
```
┌──────────────────────────────┐
│  "Misi & Tantangan"          │
├──────────────────────────────┤
│  Misi Aktif:                 │
│  ┌──────────────────────┐    │
│  │ 🏔️ Kunjungi 3 spot   │    │
│  │ underrated di Jabar   │    │
│  │ Progress: 1/3         │    │
│  │ [Klaim Misi Sekarang] │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  Misi Selesai:               │
│  ┌──────────────────────┐    │
│  │ ✅ Jelajahi 5 pantai  │    │
│  │    di Jawa Timur      │    │
│  │    Badge: 🏖️ Beachcomber│  │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  Community Feed:             │
│  ┌──────────────────────┐    │
│  │ @explorer123 upload  │    │
│  │ foto di Curug Cikaso │    │
│  │ ❤️ 45  💬 12         │    │
│  └──────────────────────┘    │
└──────────────────────────────┘
```

---

## Tab 4: Profile

```
┌──────────────────────────────┐
│  ┌────┐                      │
│  │AVA │  Nama User           │
│  │TAR │  @username           │
│  └────┘  Lv.3 Explorer       │
│          ⭐ 4.7 (23 review)  │
├──────────────────────────────┤
│  XP: 800 / 1,000             │
│  ████████████░░░░ 80%        │
│                              │
│  Badge:                      │
│  [🏔️ Penjelajah] [🏖️ Beach] │
│  [🌿 Eco-Warrior]            │
├──────────────────────────────┤
│  Trip Stats:                 │
│  12 trip selesai             │
│  8 destinasi dikunjungi      │
│  5 konten diupload           │
├──────────────────────────────┤
│  Subscription: Free          │
│  [Upgrade ke Premium Pass]   │
├──────────────────────────────┤
│  Menu:                       │
│  ├── 📋 Trip History         │
│  ├── 🏆 Badge Collection     │
│  ├── 📸 Konten Saya          │
│  ├── ⭐ Reviews Saya         │
│  ├── 🔒 Verifikasi Identitas│
│  ├── 💳 Subscription         │
│  ├── ⚙️ Settings             │
│  ├── 🚨 Keamanan & Privasi   │
│  └── 📞 Bantuan / Support    │
└──────────────────────────────┘
```

---

## Detail Screen: Destinasi

```
┌──────────────────────────────┐
│  [← Back]    [♡ Save] [Share]│
├──────────────────────────────┤
│  ┌──────────────────────┐    │
│  │                      │    │
│  │   FOTO DESTINASI     │    │
│  │   (swipe carousel)   │    │
│  │                      │    │
│  └──────────────────────┘    │
│  📍 Ciletuh, Sukabumi        │
│  ⭐ 4.6 (128 review) • 2.3km │
│  🏷️ [Alam] [Pantai] [Hidden] │
├──────────────────────────────┤
│  Deskripsi:                  │
│  Geopark Ciletuh adalah...   │
│  [Baca selengkapnya]         │
├──────────────────────────────┤
│  Info Praktis:               │
│  🕐 Buka: 24 jam             │
│  💰 Tiket: Rp10.000          │
│  📶 Sinyal: Lemah            │
│  🌧️ Musim hujan: Nov-Mar    │
├──────────────────────────────┤
│  "5 wisatawan akan ke sini"  │
│  ┌──────────────────────┐    │
│  │ [Cari Teman ke Sini] │    │
│  │ [Lihat Open Trip]    │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  Local Guide:                │
│  ┌──────────────────────┐    │
│  │ 👤 Bang Ridwan        │    │
│  │ ⭐ 4.9 • 50 trip      │    │
│  │ Rp150.000/hari        │    │
│  │ [Booking Guide]       │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  Konten Wisatawan:           │
│  ┌────┐ ┌────┐ ┌────┐       │
│  │foto│ │foto│ │foto│       │
│  └────┘ └────┘ └────┘       │
│  "Lihat semua 45 foto"       │
├──────────────────────────────┤
│  Review:                     │
│  ┌──────────────────────┐    │
│  │ @andi ⭐⭐⭐⭐⭐       │    │
│  │ "Tempatnya luar biasa"│   │
│  │ 2 hari lalu           │    │
│  └──────────────────────┘    │
└──────────────────────────────┘
```

---

## Detail Screen: Trip Match

```
┌──────────────────────────────┐
│  [← Back]                    │
├──────────────────────────────┤
│  🗺️ Bandung Weekend Trip     │
│  📅 15-17 November 2026      │
│  💰 Budget: Rp500rb-1jt      │
│  🚗 Transport: Mobil         │
├──────────────────────────────┤
│  Anggota Grup (3/4):         │
│  ┌────┐ ┌────┐ ┌────┐       │
│  │AVA │ │AVA │ │AVA │       │
│  │Kamu│ │Andi│ │Sari│       │
│  │Host│ │    │ │    │       │
│  └────┘ └────┘ └────┘       │
├──────────────────────────────┤
│  Calon Match:                │
│  ┌──────────────────────┐    │
│  │ 👤 Budi               │    │
│  │ Kompatibilitas: 87%   │    │
│  │ Minat: Alam, Kuliner  │    │
│  │ ⭐ 4.8 (15 review)    │    │
│  │ ✅ KTP Verified       │    │
│  │ [Undang ke Trip]      │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  Itinerary:                  │
│  Day 1: Braga → Gedung Sate  │
│  Day 2: Tebing Keraton → ... │
│  Day 3: Floating Market → ...│
│  [Edit Itinerary Bareng]     │
├──────────────────────────────┤
│  💬 [Buka Chat Grup]         │
│  📍 [Share Live Location]    │
│  🚨 [Laporkan Masalah]       │
└──────────────────────────────┘
```

---

## Total Screen Count (MVP)

| Kategori | Jumlah Screen | Prioritas |
|---|---|---|
| Auth & Onboarding | 6 | P0 |
| Home/Discover | 2 (Tourist + Explorer) | P0 |
| Explore/Map | 2 (Map + List) | P0 |
| Trip/Match | 3 (Match list, Chat, Itinerary) | P0 |
| Profile | 2 (Own + Other user) | P0 |
| Detail | 3 (Destinasi, Paket, Trip) | P0 |
| Gamifikasi | 2 (Misi detail, Badge collection) | P1 |
| Subscription | 2 (Plan list, Payment) | P1 |
| Settings | 3 (General, Privacy, Verifikasi) | P1 |
| Admin | 3 (Dashboard, Moderation, Reports) | P2 |
| **Total MVP** | **~25 screen** | |
| **Total Full** | **~35 screen** | |

---

## Empty States

```
Kapan muncul: ketika data kosong untuk konteks tertentu.

Home - Rekomendasi Kosong:
├── Illustration: peta Indonesia dengan pin kosong
├── Text: "Belum ada rekomendasi untukmu"
├── CTA: "Lengkapi preferensimu dulu yuk" → Settings
└── Secondary: "Atau coba lihat destinasi populer" → fallback popular

Trip Matching - Tidak Ada Match:
├── Illustration: orang sendirian di gunung
├── Text: "Belum ada traveler yang cocok"
├── CTA: "Kami akan notifikasi kalau ada match baru"
└── Info: "Coba perluas rentang tanggal atau budget"
└── Tabel: saved_searches (user_id, filters, notified_at)

Misi - Belum Ada Misi:
├── Illustration: kompas dengan tanda tanya
├── Text: "Kamu belum punya misi aktif"
├── CTA: "Jelajahi misi yang tersedia" → list misi
└── Auto: kalau user baru < 7 hari, tampilkan "First Timer" misi otomatis

Badge Collection - Kosong:
├── Illustration: trophy case kosong
├── Text: "Koleksi badge-mu masih kosong"
├── CTA: "Selesaikan misi pertamamu" → misi detail
└── Info: "Badge didapat dari menyelesaikan misi dan trip"

Chat - Belum Ada Chat:
├── Illustration: speech bubble kosong
├── Text: "Belum ada percakapan"
├── CTA: "Cari teman jalan dulu" → Trip Matching
└── Info: "Chat akan muncul setelah kamu match dengan traveler lain"

Booking History - Kosong:
├── Illustration: koper kosong
├── Text: "Belum ada perjalanan"
├── CTA: "Mulai petualangan pertamamu" → Home
└── Secondary: "Lihat open trip yang tersedia" → Marketplace
```

---

## Error States

```
Network Error (tidak ada koneksi):
├── Banner: "Tidak ada koneksi internet" (merah, dismissible)
├── Action: "Mode offline: lihat data tersimpan"
├── Retry: auto-retry saat koneksi kembali
└── Detection: navigator.onLine + periodic health check

Server Error (500):
├── Screen: full-page error
├── Text: "Terjadi kesalahan di server kami"
├── CTA: "Coba lagi" button
├── Fallback: "Hubungi support@jejakawan.id"
└── Log: auto-report ke Sentry

Not Found (404):
├── Text: "Halaman tidak ditemukan"
├── CTA: "Kembali ke Home"
└── Possible cause: destinasi dihapus, link expired

Timeout:
├── Banner: "Sepertinya koneksi lambat..."
├── Action: skeleton screen tetap tampilkan
├── Retry: auto-retry 3x dengan backoff
└── After 3x: "Gagal memuat. Coba lagi nanti."

Rate Limited (429):
├── Banner: "Terlalu banyak permintaan. Tunggu sebentar."
├── Action: auto-retry setelah retry-after duration
└── Prevention: client-side rate limiting (disable button sementara)

Payment Error:
├── Screen: "Pembayaran gagal"
├── Info: "Uang kamu TIDAK terpotong"
├── CTA: "Coba bayar lagi" / "Pilih metode lain"
├── Support: "Hubungi kami kalau masalah berlanjut"
└── Note: booking tetap hold selama 30 menit
```

---

## Loading States

```
Pattern: Skeleton Screen (bukan spinner)

Implementation:
├── Komponen yang punya skeleton state:
│   ├── DestinationCard → skeleton card (grey box + shimmer)
│   ├── UserProfile → skeleton avatar + text blocks
│   ├── MatchCandidate → skeleton profile card
│   ├── ChatMessages → skeleton bubble
│   └── Itinerary → skeleton list items
├── Shimmer animation: gradient bergerak kiri-kanan (CSS only)
├── Durasi: tampilkan skeleton minimum 300ms (hindari flash)
└── Progressive loading: text dulu → gambar nanti → map terakhir

Contoh DestinationCard skeleton:
┌──────────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░░░░░ │  ← grey box (image)
│                          │
│ ░░░░░░░░░░░░             │  ← shimmer line (title)
│ ░░░░░░░░░                │  ← shimmer line (subtitle)
│ ░░░░░░░                  │  ← shimmer line (tags)
└──────────────────────────┘
```

---

## Deep Linking Strategy

```
URL Scheme:
├── Base: https://app.jejakawan.id
├── Destinasi: /dest/{slug} → Detail destinasi
├── Open Trip: /trip/{id} → Detail paket
├── Profil: /u/{username} → Profil publik
├── Live Location: /live/{token} → Public (no auth)
├── Match Invitation: /match/{id} → Login → accept/decline
└── Misi: /mission/{id} → Detail misi

Push Notification Deep Links:
├── match_invitation → /match/{id}
├── mission_progress → /mission/{id}
├── xp_earned → /profile (tab badges)
├── subscription_reminder → /subscription
├── trip_reminder → /trip-group/{id}
├── report_update → /notifications
└── payment_success → /bookings/{id}

Universal Links (iOS) / App Links (Android):
├── Set up: assetlinks.json (Android) + apple-app-site-association (iOS)
├── Benefit: link di WhatsApp/Telegram langsung buka app (kalau installed)
└── Fallback: kalau belum install → landing page dengan CTA download
```

---

## Notification Center Screen

```
┌──────────────────────────────┐
│  [← Back]    Notifikasi      │
│  [Mark all read]             │
├──────────────────────────────┤
│  Filter: [Semua] [Match]     │
│  [Misi] [Sistem] [Promo]     │
├──────────────────────────────┤
│  HARI INI                    │
│  ┌──────────────────────┐    │
│  │ 🤝 @andi mengajak     │    │
│  │ ke Bandung, 15 Nov   │    │
│  │ 2 jam lalu            │    │
│  └──────────────────────┘    │
│  ┌──────────────────────┐    │
│  │ ⭐ +100 XP dari misi  │    │
│  │ Curug Cikaso          │    │
│  │ 5 jam lalu            │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│  KEMARIN                     │
│  ┌──────────────────────┐    │
│  │ 💳 Subscription akan   │    │
│  │ expired dalam 3 hari  │    │
│  │ 1 hari lalu            │    │
│  └──────────────────────┘    │
├──────────────────────────────┤
│                              │
│  (kosong → empty state)      │
│                              │
└──────────────────────────────┘

Badge counter: angka merah di tab More/Profile
  → menunjukkan jumlah unread notifications
  → update via realtime subscription ke notifications table
```
