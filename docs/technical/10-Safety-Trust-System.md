# 10 — Safety & Trust System

## Prinsip

Trip matching = pertemukan orang asing. Keamanan adalah fitur #1 yang menentukan user mau pakai app atau tidak. Setiap insiden = potensi viral negatif.

---

## Layer 1: Preventive (Cegah Masalah Sebelum Terjadi)

### Identity Verification

```
Tier 1 — Phone Verification (wajib untuk semua user):
├── User input nomor HP Indonesia (+62)
├── Sistem kirim OTP via SMS (Supabase Auth phone)
├── User input OTP → phone_verified = true
├── Rate limit: 3 OTP per jam, 10 per hari
├── Cost: Rp0 (termasuk Supabase Auth free tier)
└── Block: nomor HP yang sudah dipakai 3 akun berbeda

Tier 2 — KTP Verification (wajib untuk trip matching):
├── User upload foto KTP (depan)
├── User input nama sesuai KTP + 16 digit NIK
├── Sistem kirim ke Verihubs API:
│   ├── Cek NIK valid (sesuai format Kemendagri)
│   ├── Cek foto KTP (blurry/invalid)
│   └── Cek kesesuaian NIK + nama
├── Match → ktp_verified = true, NIK disimpan hashed
├── Not match → retry (max 3x), setelah itu manual review
├── Cost: ~Rp1.500 per verifikasi
└── Privacy: NIK di-hash (SHA-256 + salt), tidak disimpan plaintext

Tier 3 — Selfie Verification (opsional, Phase 2):
├── User foto selfie + foto KTP berdampingan
├── Sistem cek face match (Verihubs face match API)
├── Memastikan KTP bukan milik orang lain
├── Cost: ~Rp2.000 per verifikasi
└── Trigger: otomatis kalau user akan join trip dengan 3+ orang asing
```

### Profile Trust Score

```
Setiap user punya trust_score internal (0-100) yang menentukan
seberapa "trusted" mereka. TIDAK terlihat user lain.

Faktor yang menambah trust:
├── phone_verified: +10
├── ktp_verified: +20
├── rating_avg > 4.0: +15
├── rating_count > 5: +10
├── trip_count > 3: +10
├── account_age > 30 hari: +5
├── konten_approved > 5: +5
└── referral_count > 0: +5

Faktor yang mengurangi trust:
├── Pernah di-warning: -10 per warning
├── Pernah di-suspend: -30
├── Ada report pending: -15
├── Account < 7 hari: -10
└── Tidak ada foto profil: -5

Trust score menentukan:
├── Visibilitas di trip matching (trust < 30 = tidak muncul)
├── Prioritas di search results
├── Akses ke fitur tertentu (guide booking butuh trust > 50)
└── Threshold moderasi konten (trust > 80 = auto-approve)
```

### Matching Safety Filters

```
Sebelum user muncul di hasil matching, dicek:
├── ktp_verified = TRUE (hard requirement)
├── trust_score >= 30
├── rating_avg >= 3.0 (atau belum ada rating)
├── is_banned = FALSE
├── Tidak ada report dengan status 'pending' atau 'reviewing'
├── Account age >= 24 jam
└── phone_verified = TRUE

Kalau gagal salah satu → tidak muncul di hasil match
```

---

## Layer 2: Monitoring (Deteksi Masalah Selama Trip)

### Live Location Sharing

```
Fitur: User bisa share lokasi real-time ke kontak darurat

Teknis:
├── User aktifkan sharing → generate share_token (random 64 char)
├── App kirim GPS position ke server tiap 5 MENIT (bukan realtime)
│   → hemat battery, cukup untuk safety monitoring
├── Kontak darurat akses link: https://app.jejakawan.id/live/{token}
│   → halaman publik, tidak perlu install app
│   → tampilkan posisi di map + waktu terakhir update
├── Tombol "Hubungi User" (telepon langsung)
├── Auto-expire:
│   ├── Saat user matikan sharing
│   ├── Saat trip selesai
│   ├── Durasi yang dipilih habis
│   └── 24 jam setelah terakhir update (fallback)
└── Data retention: lokasi dihapus 24 jam setelah sharing berakhir

Privacy:
├── Hanya user sendiri yang bisa mengaktifkan
├── Lokasi hanya dibagi ke kontak yang dipilih (bukan publik)
├── Tidak ada tracking tersembunyi
└── User bisa lihat riwayan sharing sendiri
```

### Anomaly Detection (Phase 2)

```
Sistem monitoring otomatis untuk deteksi pola mencurigakan:

Deteksi:
├── User A dan User B matched, tapi tidak ada chat activity selama 24 jam
│   → trigger: friendly reminder "Sudah ngobrol dengan trip partner?"
├── User upload konten dari lokasi yang sangat berbeda dari trip plan
│   → flag untuk review manual
├── User mendadak change profile drastically sebelum trip
│   → log perubahan, tapi tidak auto-action
├── Rating bintang 1 berturut-turut dari 3+ user berbeda
│   → trigger: admin review
└── Report masuk dari 2+ user berbeda dalam 24 jam untuk user yang sama
│   → trigger: auto-suspend sementara + admin priority review
```

---

## Layer 3: Response (Tangani Masalah yang Sudah Terjadi)

### Report System

```
Jenis report:
├── FRAUD: Penipuan, meminta uang di luar app
├── INAPPROPRIATE: Konten tidak pantas (SARA, pornografi, dll)
├── DANGEROUS: Perilaku berbahaya, ancaman, pelecehan
├── FAKE: Profil palsu, foto orang lain, informasi bohong
├── SPAM: Promosi, iklan, link mencurigakan
└── OTHER: Lainnya (isi manual)

Evidence yang bisa dilampirkan:
├── Screenshot chat
├── Foto konten yang dilaporkan
├── Deskripsi kronologi
└── Bukti transaksi (kalau fraud)
```

### Moderation Pipeline

```
Queue priority:
├── P0 (DANGEROUS): review dalam 2 JAM
│   → auto-suspend reported_user sampai review selesai
├── P1 (FRAUD, INAPPROPRIATE): review dalam 12 JAM
├── P2 (FAKE, SPAM): review dalam 24 JAM
└── P3 (OTHER): review dalam 48 JAM

Review process:
├── Admin buka dashboard → lihat report + evidence
├── Cek history user (semua report, rating, konten)
├── Keputusan:
│   ├── DISMISS: tidak terbukti → hapus flag, notifikasi pelapor
│   ├── WARNING: pelanggaran ringan → notifikasi ke user
│   │   → trust_score -= 10
│   │   → record di user_warnings
│   ├── SUSPEND: pelanggaran sedang → akun di-freeze 3-30 hari
│   │   → trust_score -= 30
│   │   → semua trip_request status = 'cancelled'
│   │   → notifikasi ke semua trip partner
│   └── BAN: pelanggaran berat → akun permanen
│       → user.is_banned = TRUE
│       → semua data di-anonymize (nama, foto dihapus)
│       → data trip/payment tetap ada untuk audit
│       → banding via email: support@jejakawan.id
└── Logging: semua aksi admin di-log di admin_actions table
```

### Incident Response Plan

```
SKENARIO 1: Kecurigaan/Pelecehan selama trip
├── User laporkan via app (panic button) atau WA support
├── Admin hubungi pelapor dalam 1 JAM
├── Admin hubungi reported_user untuk klarifikasi
├── Keputusan:
│   ├── Kalau ada bukti kuat → immediate ban
│   ├── Kalau unclear → suspend investigasi 7 hari
│   └── Kalau tidak terbukti → dismiss + edukasi
├── Rekomendasi ke pelapor: hubungi pihak berwajib kalau kriminal
└── Post-incident: review proses, update SOP

SKENARIO 2: Data breach / kebocoran data
├── Deteksi: monitoring anomalous access pattern
├── Response time: 1 JAM untuk isolate
├── Tim: admin_super handle
├── Langkah:
│   ├── Matikan akses yang terdampak
│   ├── Identifikasi data yang bocor
│   ├── Notifikasi ke user yang terdampak (UU PDP: max 3x24 jam)
│   ├── Laporan ke Kominfo (jika wajib)
│   └── Post-mortem: perbaiki vulnerability
└── Komunikasi: transparan ke user, jangan sembunyikan

SKENARIO 3: User menghilang selama trip
├── Kontak darurat tidak bisa hubungi user selama 6 JAM
├── Kontak darurat laporkan ke support
├── Admin cek last known location dari live_location_shares
├── Admin hubungi trip partner untuk info
├── Rekomendasi: hubungi pihak berwajib
└── App provide: data lokasi terakhir, info trip, kontak partner
```

---

## Layer 4: Education (Cegah Masalah dengan Edukasi)

```
In-app education:
├── Safety tips saat onboarding:
│   ├── "Selalu verifikasi identitas partner sebelum trip"
│   ├── "Share live location ke keluarga/teman"
│   ├── "Trip pertama: pilih destinasi yang crowded dulu"
│   └── "Jangan transfer uang di luar app"
├── Safety checklist sebelum trip:
│   ├── ✅ Partner sudah KTP verified
│   ├── ✅ Rating partner > 4.0
│   ├── ✅ Live location sudah aktif
│   ├── ✅ Kontak darurat sudah diisi
│   └── ✅ Rencana trip sudah dibagikan ke keluarga
└── Community guidelines (wajib baca saat daftar)
```

---

## Data Privacy (UU PDP Compliance)

```
Data yang dikumpulkan dan retensi:
├── Nama, email, HP → disimpan selama akun aktif
├── Foto KTP → disimpan selama akun aktif, dihapus saat delete account
├── NIK → DISimpan HASHED (SHA-256 + salt), bukan plaintext
├── GPS location → hanya disimpan saat trip aktif / live sharing aktif
│   → dihapus 24 jam setelah trip selesai
├── Chat messages → disimpan selama trip aktif + 30 hari
│   → dihapus otomatis setelah 30 hari
├── Foto UGC → disimpan selama konten aktif
│   → user bisa hapus kapan saja
└── Payment data → disimpan 5 tahun (kewajiban pajak)

User rights (UU PDP):
├── Hak akses: user bisa download semua data mereka
├── Hak koreksi: user bisa edit profil kapan saja
├── Hak hapus: user bisa delete account → semua data dihapus dalam 30 hari
├── Hak portabilitas: user bisa export data ke format JSON
└── Hak keberatan: user bisa opt-out dari analytics

Implementation:
├── Endpoint: GET /api/profile/data-export (download semua data)
├── Endpoint: DELETE /api/profile (delete account)
├── Privacy policy: wajib ditampilkan saat onboarding
└── Consent: checkbox "Saya setuju dengan kebijakan privasi" saat daftar
```

---

## Emergency Contact Flow

```
Fitur: Panic Button (tombol darurat)

Trigger: user tekan tombol "SOS" di halaman trip aktif

Flow:
├── 1. App tampilkan konfirmasi: "Hubungi bantuan darurat?"
├── 2. User konfirmasi → app kirim:
│   ├── Lokasi terakhir ke kontak darurat (push notif + SMS)
│   ├── Lokasi terakhir ke admin (audit log)
│   └── Link live location ke kontak darurat
├── 3. App tampilkan:
│   ├── "Kontak darurat sudah diberitahu"
│   ├── "Hubungi 112 (nomor darurat Indonesia)" → direct call
│   ├── "Hubungi polisi terdekat" → nomor polisi daerah
│   └── "Hubungi support Jejakawan" → WhatsApp/telepon
├── 4. Data yang dikirim ke kontak darurat:
│   ├── Nama user + foto profil
│   ├── Lokasi GPS terakhir
│   ├── Info trip partner (nama, foto)
│   ├── Tujuan trip + tanggal
│   └── Link live location (kalau aktif)
└── 5. Log: semua emergency trigger di-log di audit_logs

Implementation:
├── Tombol SOS: floating button di halaman trip aktif
├── Tidak perlu konfirmasi kalau ditekan 3x berturut-turut (quick trigger)
├── Kontak darurat: wajib diisi sebelum trip matching dimulai
└── Tabel: emergency_contacts (user_id, name, phone, relationship)
```

---

## Appeal Process (User yang di-Ban)

```
Flow banding:
├── 1. User dapat notifikasi: "Akun kamu telah di-suspend/ban"
│   ├── Alasan: {reason dari admin}
│   ├── Durasi: {3 hari / 30 hari / permanen}
│   └── Info: "Kamu bisa mengajukan banding"
├── 2. User kirim banding via:
│   ├── Email: appeal@jejakawan.id
│   ├── Form: POST /api/appeals (kalau belum ban permanen)
│   └── Deadline: 30 hari setelah ban
├── 3. Banding harus berisi:
│   ├── Penjelasan dari sudut pandang user
│   ├── Bukti pendukung (screenshot, chat, dll)
│   └── Permintaan: review ulang / pengurangan hukuman
├── 4. Admin review banding (berbeda admin dari yang ban):
│   ├── Cek ulang evidence + context
│   ├── Bisa: uphold ban, reduce (suspend → warning), atau lift ban
│   └── SLA: keputusan dalam 7 hari kerja
├── 5. Notifikasi hasil:
│   ├── "Banding diterima: akun kamu diaktifkan kembali"
│   ├── "Banding ditolak: ban tetap berlaku"
│   └── "Banding dikabulkan sebagian: hukuman dikurangi"
└── 6. Maksimal 2 banding per ban (tidak unlimited appeal)

Tabel: appeals
├── id, user_id, ban_reference (audit_log id)
├── reason (text dari user)
├── evidence_urls (text[])
├── status (pending, reviewed, accepted, rejected)
├── reviewed_by (admin id)
├── reviewed_at, decision_notes
└── created_at
```

---

## Admin Training Procedure

```
Trust & Safety Admin Onboarding:

Week 1: Understanding the System
├── Pelajari semua fitur app dari user perspective
├── Pelajari Safety & Trust System (dokumen ini)
├── Pelajari Community Guidelines
├── Praktik: review 10 sample reports (dummy data)
└── Assessment: quiz tentang policy

Week 2: Hands-on Practice
├── Review real reports dengan supervision senior admin
├── Praktik: approve/reject mission submissions
├── Praktik: verify identity (manual review)
├── Praktik: handle chat moderation
└── Assessment: review 20 reports sendiri, compare dengan senior

Week 3: Independent + Shadowing
├── Handle reports sendiri (senior review keputusan)
├── Shadow senior handle incident (P0/P1)
├── Pelajari escalation path
├── Pelajari data retention & privacy rules
└── Assessment: independent handling dengan < 10% error rate

Ongoing:
├── Weekly calibration: review keputusan minggu lalu, diskusi edge cases
├── Monthly: update SOP kalau ada pattern baru
├── Quarterly: re-training untuk policy update
└── Burnout prevention: rotate task (jangan hanya review negative content)
```
