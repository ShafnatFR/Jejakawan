# 16 — Content Strategy

## Overview

Content di Jejakawan ada 2 jenis: curated content (tim input) dan UGC (user generated). Strategi konten menentukan kualitas rekomendasi, engagement, dan SEO.

---

## Destinasi Seeding Plan

### Target: 500 Destinasi Awal (5 Provinsi Pilot)

```
Jawa Barat (150 destinasi):
├── Bandung Kota: 30 (Braga, Gedung Sate, Dago, Ciumbuleuit)
├── Bandung Kabupaten: 30 (Tebing Keraton, Cikole, Lembang, Ciwidey)
├── Bogor: 25 (Puncak, Curug, Taman Safari, Kebun Raya)
├── Sukabumi: 25 (Ciletuh Geopark, Curug Cikaso, Pelabuhan Ratu)
├── Garut: 20 (Cipanas, Darajat, Santolo, Sayang Heulang)
├── Subang/Sumedang: 10 (Tangkuban Parahu, Situ Cileunca)
└── Cirebon/Kuningan: 10 (Keraton, Banyu Panas Palimanan)

Jawa Tengah (100 destinasi):
├── Solo: 25 (Keraton, Mangkunegaran, Jurug, Balekambang)
├── Semarang: 25 (Kota Lama, Lawang Sewu, Simpang Lima)
├── Dieng: 20 (Telaga Warna, Kawah Sikidang, Candi Arjuna)
├── Karimun Jawa: 15 (pantai, snorkeling, hopping island)
└── Wonogiri/Pacitan: 15 (gua, pantai selatan)

Jawa Timur (100 destinasi):
├── Malang: 30 (Batu, Bromo view, Coban Rondo, Jatim Park area)
├── Banyuwangi: 25 (Ijen, Red Island, Plengkung, Sukamade)
├── Surabaya: 15 (heritage, kuliner, Tugu Pahlawan)
├── Bromo area: 15 (viewpoints, savana, pasir berbisik)
└── Tulungagung/Trenggalek: 15 (pantai underrated)

Yogyakarta (75 destinasi):
├── Kota Jogja: 20 (Malioboro, Tugu, Keraton, Alun-alun)
├── Sleman/Bantul: 20 (Prambanan, Parangtritis, Gumuk Pasir)
├── Gunung Kidul: 20 (pantai-pantai selatan, gua pindul)
├── Kulon Progo: 10 (Waduk Sermo, Kalibiru)
└── Wonosari area: 5 (underrated spots)

Bali - Underrated Only (75 destinasi):
├── Buleleng (Singaraja): 20 (Lovina, Air Terjun Gitgit)
├── Karangasem: 20 (Tirta Gangga, Amed, Tulamben)
├── Tabanan: 15 (Tanah Lot area, Jatiluwih, Soka Beach)
├── Jembrana: 10 (Medewi, Gilimanuk area)
└── Nusa Penida: 10 (Kelingking, Angel's Billabong)
```

### Sumber Data

```
Tier 1 — Primary (paling bisa diandalkan):
├── OpenStreetMap (OSM):
│   ├── API: https://overpass-api.de/api/interpreter
│   ├── Query: node["tourism"="attraction"] di bounding box Indonesia
│   ├── Data: nama, koordinat, kategori, deskripsi
│   ├── Limitasi: tidak semua destinasi underrated ada
│   └── Lisensi: ODbL (bisa dipakai, credit required)
├── Wikidata / Wikipedia:
│   ├── Query: daftar tempat wisata per provinsi
│   ├── Data: deskripsi, koordinat, foto (Wikimedia Commons)
│   └── Lisensi: CC-BY-SA
└── Kemenparekraf:
    ├── Website: pesona.indonesia.travel
    ├── Data: destinasi resmi, deskripsi, foto
    ├── API: tidak publik (scraping dengan hati-hati)
    └── Lisensi: publik (pemerintah)

Tier 2 — Supplementary (tambahan):
├── Google Places API:
│   ├── Nearby search: tourist_attraction dalam radius
│   ├── Data: rating, review, foto, jam buka
│   ├── Free tier: $200/bulan ≈ 5.000 request
│   └── Lisensi: hanya untuk display, tidak boleh cache > 30 hari
├── TripAdvisor scraping (hati-hati, ToS):
│   ├── Data: rating, review summary, foto
│   └── Lisensi: tidak boleh scraping (gunakan API partner)
└── Travel blog / vlog:
    ├── Manual curation dari blog traveler Indonesia
    ├── Sumber: TripAdvisor, Traveloka review, komunitas
    └── Lisensi: minta izin / link back ke source

Tier 3 — Community (dari user):
├── UGC dari beta tester (100 user pertama)
├── Kontribusi komunitas traveler (Reddit, Facebook group)
└── Data ini paling valuable karena "dari lapangan"
```

### Data Per Destinasi yang Perlu Diinput

```
Required fields:
├── name: VARCHAR(200)
├── slug: VARCHAR(200) — URL-friendly, auto-generate
├── category: TEXT[] — alam, kuliner, budaya, kota, adventure, religi, pantai
├── location: lat, lng — dari GPS/OSM
├── province: VARCHAR(100)
├── regency: VARCHAR(100)
├── cover_image_url: TEXT — foto utama
├── description: TEXT — 100-300 kata, engaging
├── is_underrated: BOOLEAN — flag untuk gamefication
└── entry_fee_min: INT

Optional (tambah seiring waktu):
├── image_urls: TEXT[] — foto tambahan
├── signal_strength: strong/moderate/weak/none
├── best_season: TEXT[] — bulan terbaik
├── opening_hours: JSONB
├── facilities: TEXT[] — toilet, parkir, warung, musholla
├── accessibility: wheelchair, transport_umum
└── local_tips: TEXT — tips dari warga lokal

Quality checklist sebelum publish:
├── [ ] Foto ada dan bagus (bukan screenshot)
├── [ ] Koordinat akurat (dicek di map)
├── [ ] Deskripsi engaging dan informatif
├── [ ] Kategori sesuai (bukan asal pilih)
├── [ ] Harga tiket terbaru (bukan 5 tahun lalu)
├── [ ] Tidak duplikat dengan destinasi lain
└── [ ] Underrated flag: kunjungan < 10.000/tahun OR rating > 4.0 tapi kurang dikenal
```

### Seeding Timeline

```
Week 1-2: Data Collection
├── Scrape OSM + Wikidata untuk 5 provinsi
├── Filter: hanya yang punya koordinat + deskripsi
├── Target: 2.000 raw entries → filter ke 500 curated
└── Assign: Dimas (data collection) + Chika (filtering)

Week 3-4: Content Enrichment
├── Tambah foto dari Wikimedia Commons / Google Images (CC licensed)
├── Tulis deskripsi 100-300 kata per destinasi
├── Tentukan underrated flag berdasarkan data kunjungan
├── Assign: Chika (deskripsi) + Emma (foto)
└── Target: 300 destinasi siap publish

Week 5-6: Quality Review & Upload
├── Review semua data: koordinat, foto, deskripsi, kategori
├── Upload ke database via admin script (batch insert)
├── Test: search + filter + map view berfungsi
├── Assign: Aqil (upload) + Dimas (testing)
└── Target: 500 destinasi live di staging

Week 7-8: Beta Feedback
├── Beta tester (100 user) → feedback destinasi
├── Update: destinasi yang sudah tutup/salah info
├── Tambah: destinasi yang diminta beta tester
└── Target: 500 destinasi refined di production
```

---

## UGC Guidelines

### Apa yang Boleh

```
Konten yang diterima:
├── Foto asli dari destinasi (bukan screenshot Google)
├── Vlog pendek (max 60 detik) pengalaman di destinasi
├── Review jujur (positif maupun negatif, tapi konstruktif)
├── Tips praktis (cara ke sana, harga terbaru, jam buka)
├── Foto makanan/minuman di spot kuliner
└── Foto panorama, sunset, sunrise, landscape

Format yang diterima:
├── Foto: JPG, PNG, WebP — max 10MB
├── Video: MP4 — max 50MB, max 60 detik
├── Caption: max 500 karakter
└── EXIF: otomatis di-strip sebelum display (privacy)
```

### Apa yang TIDAK Boleh

```
Konten yang ditolak:
├── Foto/video NSFW (nudity, gore, violence)
├── Screenshot dari Google Maps/Street View
├── Foto milik orang lain tanpa izin (watermark photographer lain)
├── Promo/iklan terselubung (restoran, hotel, travel agent)
├── Spam (copy-paste text, link promosi)
├── SARA (suku, agama, ras, antar-golongan)
├── Hoax / informasi yang jelas salah (lokasi tutup tapi diklaim buka)
├── Konten politik / kampanye
└── Foto selfie tanpa konteks destinasi (profil-worthy ≠ feed-worthy)

Moderation action:
├── Auto-reject: NSFW detection (Cloud Vision API)
├── Manual reject: pelanggaran lain → notifikasi ke user + alasan
├── Repeat offender (3x reject): warning → temporary upload ban
└── Appeal: user bisa submit counter-argument
```

---

## Copywriting Tone of Voice

```
Brand Voice: Teman yang udah pernah ke sana

Karakter:
├── Casual: pakai "kamu", "gue" (bukan "Anda", "saya")
├── Helpful: langsung ke poin, tidak bertele-tele
├── Excited tapi tidak lebay: "Wajib coba!" bukan "LUAR BIASA!!! 😱😱"
├── Informative: kasih fakta, bukan klaim kosong
├── Inclusive: tidak eksklusif (bukan hanya "anak muda")
└── Safe: tidak provokatif, tidak SARA

DO:
├── "Yuk jelajahi Curug Cikaso!" ✅
├── "Tips: datang pagi biar gak rame" ✅
├── "Destinasi ini jarang dikunjungi, tapi pemandangannya amazing" ✅
├── "Budget: Rp50.000 udah termasuk parkir + tiket masuk" ✅
└── "Rating: 4.6 dari 128 traveler" ✅

DON'T:
├── "TEMPAT PALING INDIH YANG PERNAH GUE KUNJUNGIN!!!" ❌
├── "Harga: IDR 50,000.00" (format kaku) ❌
├── "Klik link ini untuk booking" (spam) ❌
├── "Destinasi ini lebih bagus dari X" (komparasi tidak perlu) ❌
└── "Dijamin tidak menyesal!" (klaim berlebihan) ❌

Bahasa per konteks:
├── Deskripsi destinasi: engaging, storytelling, 100-300 kata
├── Tips: short, practical, numbered list
├── Error messages: sopan, jelas, kasih solusi
├── Push notification: pendek (max 100 char), action-oriented
├── Badge name: keren tapi jelas ("Penjelajah Muda" bukan "Badge Level 1")
└── Misi description: motivating, clear goal, ada reward info
```

---

## Content Moderation Guidelines

### Auto-Moderation Rules

```
Level 1 — Auto Reject (langsung tolak tanpa review):
├── File > max size (10MB foto, 50MB video)
├── Format tidak didukung (bukan jpg/png/mp4)
├── NSFW detection confidence > 80%
├── Duplikat EXIF hash dari user yang sama (upload ulang)
└── Speed: instant

Level 2 — Auto Flag (tapi tetap ke manual review):
├── User baru (< 5 upload) → semua konten ke review
├── Klaim misi → wajib review sebelum XP dikasih
├── Caption mengandung link (URL detection)
├── Caption > 1000 karakter (kemungkinan spam)
├── Rating 1 bintang (cek apakah valid)
└── Speed: masuk queue, review < 24 jam

Level 3 — Auto Approve (langsung publik):
├── User trusted (trust_score > 80, > 20 upload approved)
├── Feed post biasa (bukan klaim misi)
├── Tidak ada flag dari Level 2
└── Review nanti kalau ada report dari user lain
```

### Manual Review SOP

```
Reviewer checklist per konten:
├── [ ] Foto relevan dengan destinasi yang diklaim?
├── [ ] Foto bukan dari internet (Google, stock photo)?
├── [ ] Caption tidak melanggar guidelines?
├── [ ] Koordinat GPS match dengan destinasi? (untuk misi)
├── [ ] Timestamp wajar? (bukan foto 5 tahun lalu)
├── [ ] Tidak ada watermark milik orang lain?
└── [ ] Kualitas foto cukup? (tidak blur parah, gelap total)

Decision tree:
├── Semua checklist pass → APPROVE
├── 1-2 minor issue → APPROVE + edit caption (hapus link, dll)
├── Foto tidak relevan → REJECT + "Foto tidak sesuai dengan destinasi"
├── Foto dari internet → REJECT + "Upload foto asli yang kamu ambil sendiri"
├── NSFW → REJECT + "Konten melanggar guidelines"
└── Spam → REJECT + warning ke user

SLA:
├── Queue kosong: approve/reject dalam 4 jam
├── Queue penuh: approve/reject dalam 24 jam
├── P0 (NSFW/spam): dalam 2 jam
└── Escalation: kalau reviewer tidak yakin → ke senior admin
```

---

## SEO Strategy (Web Version)

```
Phase 1 (MVP): PWA = SPA = SEO limited
├── Google bisa index SPA (JavaScript rendering) tapi tidak optimal
├── Focus: app store optimization (PWA install prompt) bukan SEO
└── Sitemap.xml: auto-generate dari destinasi list

Phase 2 (kalau ada web version):
├── SSR (Server-Side Rendering) untuk destinasi pages
├── URL: /dest/{slug} → mudah di-share dan di-index
├── Meta tags per halaman:
│   ├── title: "Ciletuh Geopark — Jelajahi Sukabumi | Jejakawan"
│   ├── description: "Geopark Ciletuh, surga tersembunyi di Sukabumi.
│   │   Temukan tips, foto, dan teman jalan ke sini."
│   ├── og:image: cover_image_url
│   └── og:type: article
├── Structured data (JSON-LD):
│   ├── TouristAttraction schema
│   ├── AggregateRating (dari user ratings)
│   └── GeoCoordinates
├── Content cluster:
│   ├── /dest/{provinsi}/{slug} → per destinasi
│   ├── /guide/{provinsi} → guide per provinsi
│   └── /blog/{slug} → travel tips & stories
└── Link building:
    ├── Travel blog backlinks (partnership)
    ├── Wikipedia references (kalau data akurat)
    └── Social media shares (organic)
```

---

## Social Media Integration

```
Channel yang relevan:
├── Instagram (@jejakawan.id):
│   ├── Konten: foto destinasi underrated, reels perjalanan
│   ├── Frequency: 1 post/hari, 3 stories/hari
│   ├── Strategy: user-generated repost, "Destinasi Hari Ini"
│   └── CTA: "Download Jejakawan untuk temukan tempat seperti ini"
├── TikTok (@jejakawan):
│   ├── Konten: short video destinasi, "Tempat yang belum kamu tahu"
│   ├── Frequency: 1 video/hari
│   ├── Strategy: trend-riding, before-after underrated → populer
│   └── CTA: link di bio → download page
├── WhatsApp Channel:
│   ├── Konten: weekly digest destinasi baru, promo, misi baru
│   ├── Frequency: 2-3x/minggu
│   └── Strategy: direct engagement, reminder buka app
└── YouTube (opsional, Phase 2):
    ├── Konten: mini documentary destinasi underrated
    ├── Frequency: 1 video/minggu
    └── Strategy: long-form SEO + brand awareness

In-app social features:
├── Share button: user bisa share destinasi/trip ke social media
├── Deep link: share URL → langsung buka destinasi di app
├── Share template: auto-generate card dengan foto + nama destinasi
└── Social proof: "12 traveler sudah ke sini bulan ini"
```

---

## Content Calendar Template

```
Bulanan:
├── Minggu 1: "Destinasi Bulan Ini" — 5 destinasi yang cocok untuk bulan ini
├── Minggu 2: "Tips Traveler" — practical tips dari user
├── Minggu 3: "Cerita Perjalanan" — user story dari trip matching
├── Minggu 4: "Misi Baru" — announce misi baru + reward
└── Setiap hari: 1 foto destinasi di Instagram

Event-based:
├── Long weekend: "Rekomendasi Long Weekend" (3 hari sebelum)
├── Liburan sekolah: "Family Trip Guide" (1 minggu sebelum)
├── Ramadhan/Lebaran: "Destinasi Halal Friendly"
├── 17 Agustus: "Jelajahi Sejarah Indonesia"
└── Tahun Baru: "Sunrise Spot Terbaik"

Content pipeline:
├── Ide → draft → review → schedule → publish → monitor
├── Tools: Notion (kalender), Canva (desain), Later/Hootsuite (scheduler)
└── Assign: Chika (copywriting) + Emma (desain visual)
```
