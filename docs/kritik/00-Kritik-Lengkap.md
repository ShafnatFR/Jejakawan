# KRITIK LENGKAP — DIGIVERSE FESTIVAL Business Plan Competition
## Aplikasi Jejakawan (Tourism & Culture)

Dokumen ini menggabungkan kritik konsep (ide, fitur, pricing, BMC) dan kritik teknis (arsitektur, tim, perhitungan keuangan) menjadi satu referensi utuh.

UPDATE TERAKHIR: Koreksi 3 kritik salah + 10 kritik terlewat (lihat bagian F di akhir).

---

# BAGIAN A: KRITIK KONSEP

---

## A1. Masalah Terbesar: 2 Ide Berbeda dalam 1 Dokumen

Dokumen berisi 2 ide bisnis + 1 BPC terpisah:

```
IDE 1: Jejakawan (Tourism & Culture) — line 25-166
  → Travel app + trip matching + gamifikasi
  → Paling matang, sudah ada BMC

IDE 2: SAFE (Health & Wellbeing) — line 171-224
  → Mental health app + mood tracker + AI triase
  → Data statistik SANGAT KUAT (I-NAMHS, SKI, WHO)
  → TAPI: TIDAK ADA BMC + ada masalah etika SERIUS (lihat F2)

BPC: Tebengs (Carpooling) — line 255-276
  → BPC terpisah, BUKAN bersaing dengan Jejakawan
  → Sudah ada narasi bisnis, tapi kurang angka

NOTE: IDEA 2 (line 249-254) bukan ide terpisah — ini DRAFT AWAL Jejakawan.
```

**DAMPAK:** 2 ide bikin dokumen tidak fokus. SAFE punya data lebih kuat tapi risiko etika tinggi.

**SOLUSI:** Fokus ke Jejakawan (paling aman + BMC sudah ada). SAFE bisa jadi cadangan kalau mau ambil risiko.

---

## A2. Fitur Terlalu Banyak (13 fitur)

13 fitur bikin juri bingung mana yang prioritas, value proposition kabur.

### Fitur yang HARUS DIPERTAHANKAN (4 core):

```
1. Recommendation Engine     → fondasi (rekomendasi personal)
2. Trip Matching             → DIFERENSIASI UTAMA (cari teman jalan)
3. Gamifikasi                → retensi + social impact
4. Safety & Trust            → syarat wajib untuk trip matching
+ Marketplace                → monetisasi
```

### Fitur yang DIHAPUS dari presentasi:

```
5.  Sistem Donasi       → terlalu niche, revenue kecil
8.  Itinerary Builder   → kompleks, bukan MVP
11. Weather Alert       → nice-to-have, bukan value prop
12. Offline Mode        → effort engineering besar
13. Leaderboard         → subset dari gamifikasi
```

### Fitur yang DISEBUT tapi BUKAN core:

```
3. Open Trip Marketplace → monetisasi, bukan diferensiasi
6. Segmentasi User      → logic internal
7. Local Guide           → Phase 2, sebut di roadmap
9. Verifikasi UGC        → bagian dari gamifikasi
```

---

## A3. Pricing & Plan

### Tidak Ada Analisis Harga

Harga disebut "misal" — tidak ada dasar mengapa angka itu dipilih. Juri pasti tanya: "Kenapa Rp29.000?"

### Struktur Pricing Sudah Logis (Koreksi)

Awalnya saya kritik "Weekly lebih mahal dari Monthly = error." Tapi setelah baca ulang, fitur Weekly dan Monthly BERBEDA:

```
Weekly (Rp9.900/minggu):
├── Bebas iklan
├── XP multiplier 1.5x
└── Rekomendasi unlimited

Monthly (Rp29.000/bulan) / Tahunan (Rp299.000/tahun):
├── Semua fitur Weekly +
├── Prioritas matching (boost)
├── Filter partner lebih spesifik
├── Early access open trip
├── Diskon 5% dari agen travel mitra
└── XP multiplier 2x
```

Ini tiered pricing yang masuk akal. Weekly = trial level. Yang PERLU diperbaiki: penjelasan di dokumen kenapa Weekly ada (untuk user yang mau coba premium tanpa komitmen sebulan).

### SOLUSI PRICING:

```
Tetap pakai 3 tier (Weekly/Monthly/Yearly), tapi:
├── Tambah penjelasan positioning per tier
├── Weekly: "Coba premium seminggu" → untuk user yang belum yakin
├── Monthly: "Pengguna reguler" → fitur lengkap
├── Yearly: "Hemat 14%" → untuk user yang sudah yakin
└── Tambah tabel perbandingan fitur yang jelas
```

---

## A4. Kompetisi & Positioning

### "XXX" di Bagian Kompetitor

Dua pertanyaan krusial diisi "XXX":
- Research Competitors and Alternatives
- Think About How the Business Will Make Money (detail)

Ini bagian yang dinilai juri. Kosong = penalti besar.

### Kompetitor yang Harus Dianalisis:

```
LANGSUNG:
├── Traveloka: open trip, booking → TAPI tidak ada trip matching
├── Tiket.com: booking travel → TAPI tidak ada gamifikasi
└── Trip.com: international → belum fokus Indonesia domestic

TIDAK LANGSUNG:
├── Couchsurfing: trip matching → TAPI bukan travel app
├── Komunitas Backpacker Indonesia (Facebook): matching manual
└── Tinder/Grindr: model matching yang sudah proven

NICHE:
├── Jejak.in: nama mirip → belum ada gamifikasi/trip matching
└── Wanderlog: itinerary builder → tidak ada social features
```

### Diferensiasi yang Harus Ditekankan:

```
"Tidak ada SATU platform di Indonesia yang menggabungkan keempat fitur berikut:
  1. Rekomendasi destinasi berbasis preferensi
  2. Trip matching (cari teman jalan)
  3. Gamifikasi ke destinasi underrated
  4. Safety & Trust layer

Traveloka punya #1 tapi tidak #2/#3/#4.
Couchsurfing punya #2 tapi tidak #1/#3/#4.
Kami SATU-SATUNYA yang menggabungkan keempatnya."
```

---

## A5. Value Proposition — Perlu Diperkuat

Pertanyaan kunci juri: "Kenapa user tidak pakai Traveloka?"

Jawaban:
- Traveloka = booking transactional, tidak ada trip matching
- Trip matching = fitur unik (belum ada pemain besar)
- Gamifikasi ke underrated spot = social impact angle
- Harus di-back data: berapa % user travel yang solo travel & butuh teman?

---

## A6. Tim & Non-IT

4 anggota tim = non-IT. Juri pasti tanya: "Siapa develop app-nya?"

SOLUSI:
- "Kami akan hire 1-2 freelance developer"
- Atau: "Kami pakai no-code/low-code untuk MVP"
- Atau: "Kami bermitra dengan developer dari kampus"

Hapus "AI" dari deskripsi produk — recommendation engine = rule-based filtering, bukan ML.

---

## A7. Competitive Moat

Pertanyaan juri: "Apa yang membuat kalian tidak bisa disalin Traveloka dalam 6 bulan?"

Jawaban:
- Network effect: makin banyak user → makin akurat matching
- Community & content: konten UGC = aset tidak mudah disalin
- Local partnerships: hubungan eksklusif dengan local guide/host
- Data moat: preferensi user yang terakumulasi

---

## A8. Aspek yang Tidak Dibahas

### Analisis Risiko (tidak ada):
- Risiko regulasi: KTP = data pribadi, butuh compliance UU PDP
- Risiko keamanan: insiden antar matched user = reputasi hancur
- Risiko teknis: dependency API pihak ketiga
- Risiko pasar: Traveloka launch fitur serupa
- Risiko operasional: kurasi UGC di 17.000+ pulau

### Legal & Compliance (minimal):
- PT/Entity: butuh badan hukum untuk payment & B2B
- UU PDP: KTP + live location = data sensitif
- PSE/Kominfo: platform digital butuh izin khusus
- Fixed Cost hanya "Rp15jt" untuk legal — sangat underestimate

### Referensi (cuma 1):
- Tambah: data Kemenparekraf, benchmark conversion rate, data kompetitor, riset solo travel trend

### SDG (hanya narasi):
- Perlu indikator terukur: berapa lapangan kerja, berapa destinasi terangkat, berapa warisan budaya dipromosikan

---

## A9. Inkonsistensi Nama Produk

Dokumen pakai 3 nama bergantian:
- Kawanusa (Drafting Idea)
- Penjejak (Drafting Idea)
- Jejakawan (BMC & Value Proposition)
- Plus "Jejakin" di bagian revenue streams

Pilih SATU nama. Cek juga: "Jejak.in" sudah ada — perlu cek trademark.

---

## A10. Alokasi Presentasi

Kalau kompetisi 10-15 menit:
```
2 menit : Problem & opportunity
3 menit : Solution & value proposition
3 menit : Business model & revenue
2 menit : Market size & traction plan
2 menit : Team & competitive advantage
1 menit : Q&A buffer
```

---

## A11. Kontradiksi Misi vs Model Bisnis (TERLEWAT)

Dokumen menyatakan tujuan ganda (line 242): "uang & perataan kunjungan tiap daerah." Tapi model bisnis justru mengandalkan destinasi populer:

```
SUMBER REVENUE             → DESTINASI APA YANG MENGHASILKAN UANG?
Komisi agen travel         → open trip ke destinasi POPULER (Bali, Bromo, dsb)
Iklan Google               → traffic dari user yang browse destinasi POPULER
Iklan listing pemda        → pemda yang PUNYA BUDGET promosi (biasanya daerah maju)
Fee restoran               → restoran di destinasi RAMAI
─────────────────────────────────────────────────────────────
Gamifikasi ke underrated   → TIDAK MENGHASILKAN UANG langsung
```

**MASALAH:** Semua revenue stream mengarah ke destinasi populer. Fitur gamifikasi ke underrated spot = COST center (voucher, XP), bukan revenue center. Makin banyak user ikut misi underrated = makin banyak voucher yang harus dibayar, tapi tidak ada revenue dari sana.

**SOLUSI:** Jujur di dokumen bahwa model bisnis mengandalkan destinasi populer untuk revenue, sementara gamifikasi underrated spot = CSR / social impact yang disubsili oleh revenue destinasi populer. Ini sebenarnya model yang VALID (cross-subsidization), tapi harus dijelaskan secara eksplisit, bukan disembunyikan.

---

## A12. Voucher Gamifikasi = COST yang Tidak Dikontrol (TERLEWAT)

Dokumen bilang XP bisa ditukar voucher Alfamart/hotel. Tapi:
- Berapa biaya per voucher? Tidak dihitung
- Kalau 1.000 user redeem voucher Rp25.000 = Rp25.000.000/bulan cost
- Di Variable Cost hanya Rp30-60jt/tahun untuk reward → bisa habis dalam 1-2 bulan kalau banyak yang redeem
- Tidak ada cap/batas redemption per bulan
- Tidak ada syarat yang ketat untuk redeem (cukup kumpulkan XP?)

SOLUSI: Tambahkan di dokumen:
- Cap redemption: max 500 voucher/bulan (limited stock)
- Minimum XP untuk redeem: tinggi (misal 2.000 XP = butuh 20+ spot)
- Voucher sponsored: kerja sama dengan Alfamart untuk subsidi (mereka dapat exposure)

---

---

## B1. Stack Tim Terlalu Gemuk

### Peran yang Bisa DIHAPUS:

**AI/Algoritma Specialist (Rp7jt/bulan):**
- Recommendation engine = rule-based filtering (SQL biasa), bukan ML
- Backend dev bisa handle

**Cloud Infrastructure Engineer (Rp3jt/bulan):**
- Managed service (Supabase, Railway, Vercel) sudah cukup
- Tidak perlu orang dedicated untuk skala awal

**Penghematan: Rp10jt/bulan = Rp120jt/tahun**

### Tim Realistis untuk MVP (5 orang):

| Role | Tugas | Estimasi |
|---|---|---|
| UI/UX Designer | Desain Figma, user flow | Part-time / freelance |
| Fullstack/Mobile Dev (1-2) | Frontend + Backend + Database | Core team |
| Trust & Safety / Community Ops | Moderasi konten UGC, verifikasi | Part-time awal |
| Customer Support | CS + onboarding user | Part-time / founder |
| Partnership / BD | Mitra agen travel, pemda | Founder sendiri |

Total gaji: ~Rp20-25jt/bulan (dari Rp42,5jt).

---

## B2. Tech Stack Realistis untuk Pilot

```
Mobile    : Next.js (web) — satu codebase, cepat ke market
Backend   : Supabase (PostgreSQL + Auth + Storage + Realtime)
Maps      : Google Maps Platform API (free tier $200/bulan)
Payment   : Midtrans / Xendit
Hosting   : Vercel (frontend) + Supabase (backend)
```

Biaya infrastruktur skala pilot: **Rp500rb-2jt/bulan** (bukan Rp5jt).

---

## B3. Perhitungan Keuangan — 10 Error

### Error 1: Conversion Rate 30% (seharusnya 5%)
```
Asumsi dokumen: 115.000 download → 34.500 paid (30%)
Realistis:      115.000 download → 5.750 paid (5%)
Dampak:         Revenue over-count 6x lipat
```

### Error 2: Retention 100% (seharusnya 70%/bulan)
```
Dokumen: 34.500 × 12 = 414.000 user-months
Realistis: 34.500 × geometric series = 112.264 user-months
Dampak: Monthly revenue over-count 3-4x
```

### Error 3: Weekly Pass Multiplier Salah
```
Dokumen: Weekly × 12 (bayar 12x setahun?)
Seharusnya: Weekly × 52 (bayar per minggu = 52x/tahun)
```

### Error 4: Double-Counting Weekly + Monthly
34.500 Monthly + 17.250 Weekly → apakah subset atau terpisah?
Kalau subset: jangan dijumlahkan. Kalau terpisah: 45% conversion, makin tidak realistis.

### Error 5: Total "Sekali Purchase" Selisih Rp600jt
```
Ditulis: Rp1.632.525.000
Cek: Rp1.000.500.000 + Rp1.031.550.000 + Rp170.775.000 + Rp30.000.000 = Rp2.232.825.000
Selisih: Rp600.300.000 hilang tanpa penjelasan
```

### Error 6: Total "Purchase Kembali" = Rp43 Miliar Tidak Verifiable
Pakai Monthly x12 + Weekly x12 yang keduanya bermasalah.

### Error 7: Cost Tidak Diskalakan
Cost tetap Rp1,4M padahal revenue bisa turun drastis.

### Error 8: IRR Tidak Dihitung
Panduan BMC minta IRR. Tidak ada perhitungannya.

### Error 9: Skenario Pilot vs General Konsisten di 30%
Conversion rate dipertahankan 30% di kedua skenario — seharusnya pilot (komunitas tertutup) lebih tinggi, nasional lebih rendah.

### Error 10: Kompetitor & Monetisasi = "XXX"
Bagian krusial kosong.

---

## B4. Struktur Tim — Pembagian Tidak Merata

| Anggota | Beban | Masalah |
|---|---|---|
| Emma | 2 elemen (Segments + Relationships) | Paling ringan |
| Chika | 3 elemen (Value Prop + Channels + Resources) | Terlalu berat |
| Dimas | 3 elemen (Activities + Partnerships + Resources) | Oke |
| Aqil | 2 elemen (Revenue + Cost) | Paling berat teknis, banyak error |

Tidak ada yang handle: Competitors, Break-Even, Risk Analysis, IRR.

Saran redistribusi:
- **Emma:** Segments + Relationships + **Competitors**
- **Chika:** Value Propositions + Channels
- **Dimas:** Activities + Partnerships + Resources (semua)
- **Aqil:** Revenue + Cost + **IRR + Break-Even + Risk Analysis**

---

## B5. UX & Data yang Tidak Dibahas

- Onboarding friction: kalau preferensi panjang → drop-off tinggi
- Trip matching trust: bagaimana user percaya orang asing?
- Gamifikasi balance: XP terlalu mudah/sulit → perlu tuning
- Tracking metrics: DAU, MAU, retention, churn → tidak disebut
- A/B testing plan → tidak ada
- Dashboard internal → tidak ada

---

## B6. Kekuatan yang Perlu Dipertahankan

| # | Kekuatan |
|---|----------|
| 1 | Trip Matching — belum ada pemain besar di niche ini |
| 2 | SDG angle — social impact ke daerah underrated |
| 3 | Segmentasi user (wisatawan vs explorer) |
| 4 | Safety & Trust Layer sebagai fitur utama |
| 5 | TAM/SAM/SOM sudah dihitung dengan sumber data |
| 6 | Gamifikasi double purpose (retensi + social impact) |
| 7 | Customer Relationships sangat detail (7 tipe) |

---

# BAGIAN C: SOLUSI PALING DIREKOMENDASIKAN

---

## Step 1: FOKUS ke 1 IDE (Jejakawan)
Buang SAFE (risiko etika tinggi) dan pisahkan Tebengs (BPC terpisah). Fokus ke Jejakawan.

## Step 2: POTONG FITUR ke 4 Core
1. Recommendation Engine → "Rekomendasi personal"
2. Trip Matching → "Cari teman jalan" (DIFERENSIASI UTAMA)
3. Gamifikasi → "Jelajahi & dapat reward"
4. Safety → "Aman bersama"
\+ Marketplace → monetisasi

Sisanya masuk roadmap Phase 2.

## Step 3: PERBAIKI PRICING
Tetap pakai 3 tier, tapi tambah penjelasan positioning:
```
Free:           Rp0            — 3 rekomendasi/hari, iklan
Weekly:         Rp9.900/minggu — trial premium (bebas iklan, XP 1.5x, unlimited recs)
Monthly:        Rp29.000/bulan — full features (priority matching, filter, diskon, XP 2x)
Yearly:         Rp299.000/tahun — hemat 14% vs monthly ×12
```
Tambah tabel perbandingan fitur yang jelas di dokumen.

## Step 4: HITUNG ULANG FINANSIAL (Asumsi Realistis)
```
Download:     20.000 (bukan 115.000)
Conversion:   5% (bukan 30%)
Paid users:   1.000
Retention:    70%/bulan (bukan 100%)
Revenue:      ~Rp264jt/tahun
Cost:         ~Rp348jt/tahun (tim ramping)
Laba:         -Rp83jt (RUGI tahun 1, NORMAL untuk startup)
Break-even:   bulan 16-18
```

## Step 5: ISI BAGIAN KOSONG
- Competitors: analisis 4-5 kompetitor + differensiasi
- Monetisasi: copy dari revenue streams yang sudah ada
- Operasi: tech stack + tim + SOP
- IRR: hitung dari proyeksi 3 tahun
- Break-even: tabel per bulan

## Step 6: BERSIHKAN DOKUMEN
- Hapus duplikasi (3 versi cost → 1 versi)
- Hapus "masih mikir", "XXX"
- Konsistensi nama (pilih 1)
- Tambah referensi (Kemenparekraf, data travel, benchmark)
- Tambah analisis risiko (3-5 risiko + mitigasi)

## Step 7: SIAPKAN JAWABAN 10 PERTANYAAN JURI
1. "Kenapa tidak pakai Traveloka?" → trip matching + gamifikasi
2. "Conversion rate 30% dari mana?" → koreksi ke 5%
3. "Siapa develop app-nya?" → freelance developer / no-code
4. "Break-even kapan?" → bulan 16-18
5. "Kompetitor siapa?" → Traveloka, Tiket.com, Couchsurfing
6. "Kenapa harga segitu?" → benchmark Rp29.000/bulan (setara Netflix mobile Indonesia, lebih murah dari Spotify)
7. "Aman gak trip sama orang asing?" → verifikasi KTP + rating + live location
8. "IRR berapa?" → hitung dari proyeksi 3 tahun
9. "Download 20.000 realistis?" → data industri app travel baru
10. "Apa yang bikin beda?" → SATU-SATUNYA platform combine 4 fitur

---

# BAGIAN D: PRIORITAS PERBAIKAN

```
PRIORITAS 1 — KOREKSI ANGKA (sebelum presentasi):
  [ ] Koreksi conversion rate 30% → 5%
  [ ] Koreksi retention 100% → 70%/bulan
  [ ] Tambah penjelasan positioning Weekly Pass (bukan error, perlu konteks)
  [ ] Hitung ulang semua revenue dengan asumsi baru
  [ ] Hitung IRR
  [ ] Hitung break-even point
  [ ] Cek selisih Rp600jt di total "sekali purchase"

PRIORITAS 2 — ISI BAGIAN KOSONG (sebelum submit):
  [ ] Competitors & Alternatives (bukan "XXX")
  [ ] Monetisasi detail (bukan "XXX")
  [ ] Analisis risiko (3-5 risiko + mitigasi)
  [ ] Referensi tambahan (Kemenparekraf, data travel, dll)

PRIORITAS 3 — PERKUAT NARASI (sebelum rehearsal):
  [ ] Pilih SATU nama (Jejakawan)
  [ ] Potong fitur ke 4 core
  [ ] Kuatkan value proposition vs Traveloka
  [ ] SDG dengan indikator terukur
  [ ] Redistribusi beban tim
  [ ] Siapkan jawaban 10 pertanyaan juri
  [ ] Fokus ke Jejakawan, pisahkan SAFE dan Tebengs dari dokumen

PRIORITAS 4 — NICE TO HAVE:
  [ ] UX flow visual (bukan cuma tulisan)
  [ ] Dashboard metrics plan
  [ ] Competitive moat section
  [ ] Legal & compliance detail
  [ ] Hapus role AI Specialist & Cloud Engineer dari tim
```

---

# BAGIAN E: RINGKASAN AKHIR

```
KONSEP:        Bagus, tapi ada 2 ide dalam 1 dokumen + kontradiksi misi vs revenue
FITUR:         Terlalu banyak (13), potong ke 4 core
PRICING:       Tidak ada analisis harga (harga disebut "misal"), perlu justifikasi benchmark
KOMPETITOR:    KOSONG (isi "XXX")
FINANSIAL:     10 error (conversion, retention, multiplier, selisih)
TIM:           Non-IT + tim terlalu gemuk + pembagian tidak merata
DOKUMEN:       Banyak duplikasi, "masih mikir", inkonsistensi nama
MISI vs BISNIS: Revenue dari destinasi populer, tapi misi ke underrated = cost center
VOUCHER:       Cost tidak dikontrol, perlu cap dan syarat redeem ketat

YANG SUDAH BAGUS (pertahankan):
  ✅ Problem statement kuat
  ✅ Trip matching = fitur unik
  ✅ SDG angle bikin beda
  ✅ TAM/SAM/SOM ada sumber data
  ✅ Customer Segments sangat detail
  ✅ Customer Relationships 7 tipe
  ✅ Safety & Trust Layer diprioritaskan

---

# BAGIAN F: KOREKSI & TAMBAHAN (Hasil Validasi Ulang)

---

## F1. Kritik yang SALAH (perlu dikoreksi)

### Pricing Weekly Pass — BUKAN Error

Saya bilang: "Weekly lebih mahal dari Monthly = pricing error."

Tapi baca ulang dokumen asli (line 712-728), fitur Weekly dan Monthly BERBEDA:
- Weekly (Rp9.900/minggu): bebas iklan, XP 1.5x, rekomendasi unlimited
- Monthly (Rp29.000/bulan): semua Weekly + priority matching + filter detail + early access + diskon 5% + XP 2x

Ini tiered pricing yang masuk akal. Weekly = trial/entry level. Monthly = full features. BUKAN pricing error.

### "3 Ide Bersaing" — Salah Hitung

Yang benar: ada **2 ide** (Jejakawan + SAFE), bukan 3.
- IDEA 2 (line 249-254) = draft awal Jejakawan, bukan ide terpisah
- Tebengs = BPC terpisah (Business Plan Canvas), bukan bersaing dengan Jejakawan

### SAFE "Cuma Mentah" — Terlalu Meremehkan

SAFE punya data statistik yang SANGAT KUAT:
- I-NAMHS (survei kesehatan mental remaja pertama di Indonesia)
- SKI & Riskesdas (data depresi usia produktif)
- Kemenkes, BPS, WHO (data krisis bunuh diri)
- 3 sumber data untuk 3 aspek berbeda

Data empiris SAFE LEBIH KUAT dari Jejakawan (yang cuma punya 1 sumber: APJII).

---

## F2. Kritik yang TERLEWAT (perlu ditambahkan)

### Masalah Etika SERIUS pada SAFE

**RuangTenang AI (Triase Mental Health):**
- AI yang melakukan screening kesehatan mental TANPA pengawasan medis = BERBAHAYA
- False negative → user dalam krisis tidak dapat bantuan
- False positive → user over-medicalized
- Kasus nyata: chatbot AI di luar negeri menyebabkan kematian (Character.AI case)
- Indonesia belum punya regulasi AI-based mental health screening

**Gamifikasi Self-Harm (Streak System):**
- Streak yang kalau putus = user merasa gagal → bisa MEMPERBURUK kondisi
- User bisa SEMBUNYIKAN relapse daripada jujur demi menjaga streak
- Professional mental health TIDAK menyarankan gamifikasi untuk self-harm recovery

**TemanDengar (Peer-Support Anonim):**
- Tanpa moderasi profesional = berisiko
- "Suicide contagion" = sharing ide bunuh diri bisa menular ke user lain
- Butuh moderator berlisensi, bukan hanya community moderator

**Voucher Konseling 30 Menit:**
- Siapa yang BAYAR psikolognya? (Rp100-300rb per sesi)
- Konseling 30 menit via chat = tidak standar untuk krisis mental
- Merendahkan profesi psikolog

### SAFE vs Jejakawan — Perbandingan untuk Kompetisi

```
ASPEK               JEJAKAWAN              SAFE
Data empiris        1 sumber (APJII)       3+ sumber (lebih kuat)
Problem clarity     Kuat                   Sangat kuat
SDG alignment       SDG 8,10,11            SDG 3 (lebih spesifik)
Social impact       Pariwisata merata      Menyelamatkan nyawa
BMC                 Sudah ada              Belum ada
Fitur complexity    13 fitur               5 fitur (lebih fokus)
Etika risk          Rendah                 TINGGI (AI triase, gamifikasi)
Monetisasi          Jelas (5 stream)       Belum jelas
Competitor          Traveloka, dll         Sehati.ai, Riliv
---

Jejakawan LEBIH AMAN untuk kompetisi. SAFE punya data lebih kuat tapi risiko etika tinggi yang bisa jadi bumerang saat Q&A.

### Tebengs — Analisis yang Terlewat

Kekuatan Tebengs yang tidak saya sebut:
- **Closed-community marketplace** (UI sebagai testbed) = strategi SANGAT CERDAS
- Mengatasi chicken-and-egg problem (mana duluan: driver atau passenger?)
- Mirip model Facebook yang mulai dari kampus
- 30.000+ mahasiswa UI = captive audience

Yang kurang dari Tebengs:
- **Regulasi:** carpooling di Indonesia belum jelas legalitasnya
- **Asuransi:** kalau kecelakaan, siapa tanggung jawab?
- **Grab Hitch sudah ada** → apa bedanya Tebengs?

### Monetisasi — Urutan Salah

Dokumen asli (line 160-165):
```
- Iklan Google ← disebut pertama, tapi revenue PALING KECIL
- Iklan/listing berbayar dari daerah/pemda
- Komisi dari agen travel ← seharusnya pertama (paling besar)
- Fee dari kerja sama restoran/local host ← model tidak dijelaskan
- Donasi user ← bukan model bisnis yang bisa diproyeksikan
```

Urutan seharusnya: Komisi marketplace → Partnership → Iklan → Donasi.

### Fee Restoran/Local Host — Tidak Dijelaskan

"Fee dari kerja sama restoran/local host" — modelnya apa?
- Flat fee per bulan? Per booking? Per click? Per referral?
- Tidak ada penjelasan di dokumen

### Gap Drafting Idea vs Drafting BMC

Banyak insight bagus di Drafting Idea yang HILANG di BMC:
- Konsep "dua tujuan: uang & pemerataan" (line 242) → value proposition inti, tapi BMC tidak menekankan ini
- Contoh User A, B, C (line 238-240) → sangat bagus untuk segmentasi, tapi tidak masuk Customer Segments

### Typo yang Tidak Saya Flag

```
Line 254: "dimskut" → catatan internal (Dimas?), hapus
Line 253: "taggal" → "tanggal"
Line 253: "pegguna" → "pengguna"
Line 253: "nyesuain" → "menyesuaikan"
Line 236: "suppoter" → "supporter"
Line 240: "advanture" → "adventure"
Line 229/235/242: "underated" → "underrated"
```

---

## F3. Ringkasan Koreksi

```
KRITIK YANG SAYA KOREKSI:
  1. Weekly Pass pricing error → BUKAN error (tiered pricing logis)
  2. "3 ide bersaing" → sebenarnya 2 ide + 1 BPC terpisah
  3. SAFE "cuma mentah" → data-nya LEBIH KUAT dari Jejakawan

KRITIK YANG SAYA TAMBAHKAN:
  4.  Masalah etika SERIUS pada SAFE (AI triase, gamifikasi self-harm)
  5.  Perbandingan SAFE vs Jejakawan untuk kompetisi
  6.  Tebengs: closed-community strategy + regulasi carpooling
  7.  Monetisasi urutan salah (iklan Google disebut pertama)
  8.  Fee restoran/local host model tidak dijelaskan
  9.  Gap Drafting Idea vs Drafting BMC (insight hilang)
  10. 7 typo di dokumen ("dimskut", "taggal", dll)
```
