# 06 — Business Process Flow

## Proses Bisnis Inti

Dokumen ini mendefinisikan alur proses bisnis utama yang harus diimplementasikan di sistem. Setiap proses punya trigger, langkah, decision point, dan output.

---

## BP-1: Trip Matching (Cari Teman Jalan)

```
Trigger: User buka "Cari Teman Jalan" → "Buat Trip Baru"

┌─────────────────────────────────────────────┐
│ 1. User isi detail trip                      │
│    ├── Destinasi (search/select dari map)    │
│    ├── Tanggal keberangkatan (range)         │
│    ├── Durasi (hari)                         │
│    ├── Budget range                          │
│    ├── Moda transportasi                     │
│    ├── Jumlah anggota dicari (1-5)           │
│    └── Catatan (opsional)                    │
├─────────────────────────────────────────────┤
│ 2. Sistem cek verifikasi                     │
│    ├── KTP verified? → LANJUT                │
│    ├── HP verified? → LANJUT (warning)       │
│    └── Tidak verified? → BLOKIR, minta verif │
├─────────────────────────────────────────────┤
│ 3. Sistem simpan trip request                │
│    └── Tabel: trip_requests                   │
├─────────────────────────────────────────────┤
│ 4. Matching Engine berjalan (background)     │
│    ├── Cari trip_requests lain dengan:       │
│    │   ├── Destinasi overlap/sama            │
│    │   ├── Tanggal overlap (min 1 hari)      │
│    │   ├── Budget range overlap              │
│    │   └── Transportasi kompatibel           │
│    ├── Hitung compatibility score (0-100)    │
│    ├── Filter: ktp_verified = true           │
│    ├── Filter: rating_avg >= 3.0             │
│    ├── Filter: tidak ada report aktif        │
│    └── Sort: compatibility score DESC        │
├─────────────────────────────────────────────┤
│ 5. Hasil match ditampilkan                   │
│    ├── Kartu profil calon match              │
│    ├── Compatibility score                   │
│    ├── Minat & preference                    │
│    └── Rating & badge                        │
├─────────────────────────────────────────────┤
│ 6. User pilih aksi                           │
│    ├── "Ajak Match" → notifikasi ke calon    │
│    ├── "Skip" → tampilkan berikutnya         │
│    └── "Lihat Profil" → detail profil        │
├─────────────────────────────────────────────┤
│ 7. Calon match terima notifikasi             │
│    ├── "Terima" → chat room dibuka           │
│    │   ├── Kedua user masuk grup trip        │
│    │   ├── Itinerary builder aktif           │
│    │   └── Status: matched                   │
│    ├── "Tolak" → notifikasi ke pengajak      │
│    └── Tidak respon 48 jam → auto-expire     │
├─────────────────────────────────────────────┤
│ 8. Setelah match                             │
│    ├── Trip berjalan normal                  │
│    ├── Live location sharing (opsional)      │
│    ├── Chat grup aktif                       │
│    └── Setelah trip: rating antar anggota    │
└─────────────────────────────────────────────┘

Output:
├── trip_requests (data request)
├── trip_matches (data match)
├── chat_rooms (room per trip)
└── ratings (setelah trip selesai)
```

---

## BP-2: Booking Open Trip Marketplace

```
Trigger: User buka detail paket Open Trip → "Booking Sekarang"

┌─────────────────────────────────────────────┐
│ 1. User lihat detail paket                   │
│    ├── Nama paket, agen travel               │
│    ├── Tanggal tersedia                      │
│    ├── Itinerary lengkap                     │
│    ├── Harga per orang                       │
│    ├── Slot tersisa                          │
│    └── Rating agen                           │
├─────────────────────────────────────────────┤
│ 2. User pilih tanggal & jumlah peserta       │
│    └── Sistem hitung total harga             │
├─────────────────────────────────────────────┤
│ 3. Checkout                                  │
│    ├── Ringkasan booking                     │
│    ├── Metode pembayaran (Midtrans)          │
│    │   ├── VA Bank (BCA, Mandiri, BNI, BRI)  │
│    │   ├── E-wallet (GoPay, OVO, DANA)       │
│    │   ├── QRIS                              │
│    │   └── Credit Card                       │
│    └── Kode promo (opsional)                 │
├─────────────────────────────────────────────┤
│ 4. Pembayaran diproses                       │
│    ├── Midtrans callback → webhook ke sistem │
│    ├── Status: pending → paid                │
│    ├── Booking confirmed                     │
│    └── Notifikasi ke user + agen travel      │
├─────────────────────────────────────────────┤
│ 5. Komisi dipotong otomatis                  │
│    ├── Total bayar user: Rp500.000           │
│    ├── Komisi Jejakawan (10%): Rp50.000      │
│    ├── Agen travel terima: Rp450.000         │
│    └── Tabel: transactions                   │
├─────────────────────────────────────────────┤
│ 6. Setelah trip selesai                      │
│    ├── User kasih rating ke paket + agen     │
│    ├── Agen kasih rating ke user             │
│    └── XP ditambahkan ke user                │
└─────────────────────────────────────────────┘

Output:
├── bookings (data booking)
├── transactions (data pembayaran + komisi)
└── ratings (review paket + agen)
```

---

## BP-3: Gamifikasi — Klaim Misi & Earn XP

```
Trigger: User buka "Misi Aktif" → "Klaim Misi"

┌─────────────────────────────────────────────┐
│ 1. User lihat misi tersedia                  │
│    ├── Nama misi (e.g., "Kunjungi 3 spot    │
│    │   underrated di Jawa Barat")            │
│    ├── Deadline (30 hari)                    │
│    ├── Reward: XP + badge                    │
│    ├── Progress saat ini                     │
│    └── Daftar spot yang bisa dikunjungi      │
├─────────────────────────────────────────────┤
│ 2. User klaim misi                           │
│    ├── Misi masuk ke "Misi Aktif" user       │
│    ├── Timer 30 hari mulai                   │
│    └── Tabel: user_missions                  │
├─────────────────────────────────────────────┤
│ 3. User datangi spot underrated              │
│    ├── GPS mendeteksi user di lokasi spot    │
│    ├── Sistem cek: apakah ini salah satu     │
│    │   spot dalam misi?                      │
│    └── Munculkan popup: "Upload bukti!"      │
├─────────────────────────────────────────────┤
│ 4. User upload bukti                         │
│    ├── Foto / vlog (wajib)                   │
│    ├── Caption (opsional)                    │
│    ├── Auto-tag lokasi + waktu dari EXIF     │
│    └── Tabel: mission_submissions            │
├─────────────────────────────────────────────┤
│ 5. Validasi konten                           │
│    ├── Auto: EXIF lokasi cocok dengan spot?  │
│    ├── Auto: NSFW check (basic)              │
│    ├── Manual: Trust & Safety review (24 jam)│
│    └── Status: pending → approved / rejected │
├─────────────────────────────────────────────┤
│ 6. Setelah approved                          │
│    ├── XP ditambah (e.g., +100 XP per spot)  │
│    ├── Progress misi bertambah               │
│    ├── Konten masuk Community Feed           │
│    └── Notifikasi: "Kamu dapat +100 XP!"    │
├─────────────────────────────────────────────┤
│ 7. Misi selesai (semua spot tercapai)        │
│    ├── Bonus XP (e.g., +500 XP completion)   │
│    ├── Badge baru didapat                    │
│    ├── Level up (kalau XP cukup)             │
│    ├── Leaderboard update                    │
│    └── Notifikasi: "Badge baru: Penjelajah!" │
├─────────────────────────────────────────────┤
│ 8. XP Redemption (opsional)                  │
│    ├── User buka "Tukar Poin"                │
│    ├── Pilih voucher (Alfamart, hotel, dll)  │
│    ├── Konfirmasi penukaran                   │
│    ├── XP dikurangi, voucher dikirim         │
│    └── Tabel: xp_redemptions                 │
└─────────────────────────────────────────────┘

Output:
├── user_missions (klaim misi)
├── mission_submissions (bukti upload)
├── xp_logs (riwayat XP)
├── badges (badge yang didapat)
├── xp_redemptions (penukaran voucher)
└── community_feed (konten terverifikasi)
```

---

## BP-4: Safety & Trust — Verifikasi & Pelaporan

### BP-4a: Verifikasi Identitas
```
Trigger: User buka "Verifikasi Identitas"

1. User upload foto KTP
2. User input nama sesuai KTP + NIK
3. Sistem kirim ke API verifikasi (Verihubs)
4. API return: match/not match + confidence score
5. Kalau match:
   ├── ktp_verified = true
   ├── NIK di-hash (tidak disimpan plaintext)
   └── User dapat badge "Verified"
6. Kalau not match:
   ├── Minta user upload ulang
   └── Setelah 3x gagal → manual review oleh admin
```

### BP-4b: Pelaporan User/Konten
```
Trigger: User klik "Laporkan" pada profil/konten user lain

1. User pilih alasan pelaporan
   ├── Penipuan / Spam
   ├── Konten tidak pantas
   ├── Perilaku berbahaya
   ├── Informasi palsu
   └── Lainnya (isi manual)
2. User tambahkan bukti (screenshot, chat)
3. Laporan masuk ke tabel reports
4. Admin Trust & Safety review (target: 24 jam)
5. Keputusan:
   ├── Warning ke user dilaporkan
   ├── Suspend akun sementara
   ├── Ban permanen
   └── Laporan ditolak (tidak terbukti)
6. Notifikasi ke pelapor: "Laporan kamu sedang ditinjau"
7. Notifikasi hasil: "Laporan telah ditindaklanjuti"
```

### BP-4c: Live Location Sharing (selama trip)
```
Trigger: User aktifkan "Share Live Location" saat trip

1. User pilih kontak darurat (HP yang bisa dihubungi)
2. User pilih durasi sharing (sampai trip selesai / X jam)
3. Aplikasi kirim lokasi ke server tiap 5 menit
4. Kontak darurat dapat link unik (web view, tidak perlu install app)
5. Link menampilkan:
   ├── Posisi user real-time di map
   ├── Terakhir update: X menit lalu
   └── Tombol "Hubungi User" (telepon/WA)
6. Saat sharing dimatikan / trip selesai:
   ├── Link expired
   └── Data lokasi dihapus dari server setelah 24 jam
```

---

## BP-5: Subscription & Payment

```
Trigger: User buka "Upgrade ke Premium Pass"

1. User lihat perbandingan plan
   ├── Free: fitur dasar, iklan, rekomendasi terbatas (3/hari)
   ├── Weekly (Rp9.900): XP 1.5x, no iklan, rekomendasi 10/hari
   ├── Monthly (Rp29.000): semua weekly + priority matching + filter detail
   └── Exclusive (Rp299.000/tahun): semua monthly + early access + diskon partner + XP 2x

2. User pilih plan
3. Checkout via Midtrans
4. Payment confirmed → subscription aktif
5. Tabel: user_subscriptions
   ├── user_id
   ├── plan (weekly/monthly/exclusive)
   ├── starts_at
   ├── expires_at
   ├── auto_renew (boolean)
   └── payment_id

6. Setiap fitur dicek subscription level:
   ├── Rekomendasi harian: free (3/hari), premium (unlimited)
   ├── XP multiplier: free (1x), premium (1.5x-2x)
   ├── Priority matching: free (no), premium (yes)
   ├── Filter detail: free (basic), premium (advanced)
   └── Iklan: free (ada), premium (tidak ada)

7. Auto-renew:
   ├── 3 hari sebelum expired → notifikasi reminder
   ├── Hari H → charge otomatis via Midtrans recurring
   ├── Gagal → grace period 3 hari → downgrade ke free
   └── Tabel: subscription_payments
```

---

## BP-6: Content Moderation Pipeline

```
Trigger: User upload foto/vlog (dari mana saja: misi, review, feed)

1. Upload ke Supabase Storage
2. Auto-check (langsung):
   ├── File size: max 10MB foto, 50MB video
   ├── Format: jpg, png, mp4
   ├── NSFW detection (Cloud Vision API — optional di Phase 1)
   └── EXIF data: lokasi + waktu

3. Metadata extraction:
   ├── Koordinat GPS dari EXIF
   ├── Timestamp dari EXIF
   └── Auto-tag destinasi terdekat

4. Manual review queue (Trust & Safety):
   ├── Konten dari user baru (< 5 trip) → prioritas review
   ├── Konten klaim misi → wajib review sebelum XP dikasih
   └── Konten feed biasa → review jika ada report

5. Status:
   ├── approved → publik, XP dikasih (kalau misi)
   ├── rejected → user diberitahu alasan, bisa upload ulang
   └── flagged → admin review lebih lanjut
```

---

## Proses yang TIDAK di-MVP-kan

| Proses | Alasan | Target Fase |
|---|---|---|
| Local Guide booking | Butuh onboarding manual per daerah | Fase 2 |
| Itinerary builder kolaboratif | Kompleks UX, butuh realtime sync | Fase 2 |
| Offline mode | Engineering effort besar | Fase 2 |
| Donasi/Support point | Belum krusial tanpa user base | Fase 3 |
| Weather alert | Butuh API cuaca, bukan prioritas | Fase 2 |
| Leaderboard regional | Butuh user base cukup dulu | Fase 2 |

---

## SLA per Proses

```
BP-1 Trip Matching:
├── Matching query result: < 5 detik (target < 2 detik)
├── Match invitation delivery: < 30 detik
├── Accept/decline → chat room created: < 5 detik
└── Auto-expire: 48 jam

BP-2 Open Trip Booking:
├── Payment page load: < 3 detik
├── Payment confirmation (Midtrans callback): < 60 detik
├── Booking confirmed notification: < 2 menit
└── Refund processing: 3-7 hari kerja

BP-3 Gamifikasi Klaim Misi:
├── Upload bukti: < 10 detik (tergantung file size)
├── Auto-validasi (GPS + EXIF): < 5 detik
├── Manual review: < 24 jam
├── XP dikasih setelah approval: instant
└── Misi completion → badge unlock: instant

BP-4 Safety & Trust:
├── KTP verification: < 30 detik (API) / < 24 jam (manual)
├── Report review (P0 - DANGEROUS): < 2 jam
├── Report review (P1): < 12 jam
├── Report review (P2-P3): < 24-48 jam
├── Live location update interval: 5 menit
└── Incident response (data breach): < 1 jam isolate

BP-5 Subscription:
├── Payment → subscription active: < 60 detik
├── Auto-renew charge: H-1 sebelum expired
├── Grace period: 3 hari
└── Downgrade ke free: instant setelah grace period

BP-6 Content Moderation:
├── Auto-check (size, format): instant
├── Manual review: < 24 jam
├── Appeal (user submit counter-argument): < 48 jam review
└── NSFW detection: < 3 detik per image
```

---

## Data Retention per Proses

```
Chat messages:
├── Active trip: disimpan selama trip aktif
├── Setelah trip selesai: 30 hari retention
├── Setelah 30 hari: auto-delete (cron job)
└── Exception: messages yang dilaporkan → disimpan 90 hari

Location data:
├── Live sharing: disimpan selama sharing aktif
├── Setelah sharing stop: dihapus dalam 24 jam
├── GPS track selama trip: disimpan 7 hari setelah trip selesai
└── Admin access: hanya untuk investigation (audit logged)

Mission submissions:
├── Approved: disimpan permanent (bisa dipakai ulang di feed)
├── Rejected: disimpan 30 hari (user bisa lihat alasan)
├── Setelah 30 hari rejected: file dihapus, metadata tetap
└── Bukti foto: disimpan di Supabase Storage, dihapus sesuai di atas

User data setelah delete account:
├── Profil: anonymized dalam 7 hari (nama → "Deleted User")
├── Konten UGC: tetap ada tapi author anonymous (community benefit)
├── Trip history: anonymized (untuk analytics)
├── KTP data: dihapus permanen dalam 7 hari
├── Payment data: disimpan 5 tahun (kewajiban pajak Indonesia)
└── Chat messages: anonymized (pesan tetap terbaca tapi tanpa identitas)

General retention rules:
├── Data aktif: disimpan selama user aktif
├── Data inaktif: archive setelah 12 bulan tidak aktif
├── Archive → delete: setelah 24 bulan total tidak aktif
└── Hard delete: semua data non-financial setelah user request (UU PDP)
```

---

## Internationalization (i18n)

```
Decision: Bahasa Indonesia only (Phase 1), bilingual (Phase 2+)

Rationale:
├── Target market = 100% Indonesia
├── Tim development kecil, maintain 2 bahasa = beban
├── Banyak istilah lokal yang tidak bisa diterjemahkan (e.g., "underrated spot")
└── Phase 2: tambah English kalau ada user diaspora/expat

Implementation (kalau bilingual nanti):
├── Library: react-i18next (React ecosystem)
├── Structure:
│   ├── locales/id.json (default)
│   ├── locales/en.json
│   └── User pilih bahasa di Settings
├── Format:
│   ├── Tanggal: "15 November 2026" (id) / "November 15, 2026" (en)
│   ├── Uang: "Rp 1.000.000" (titik) / "$1,000.000" (koma)
│   ├── Angka: "1.234.567" (titik) / "1,234,567" (koma)
│   └── Jam: "15.30 WIB" / "3:30 PM WIB"
├── Cultural UX:
│   ├── Warna: hijau (positif, Islami), merah (peringatan)
│   ├── Ramadhan mode: rekomendasi destinasi yang ramah Muslim
│   └── Libur nasional: otomatis detect & rekomendasi
└── Testing: native speaker review semua teks sebelum launch
```
