# Kritik & Saran Teknis — DIGIVERSE FESTIVAL Business Plan
## Aplikasi Jejakawan (Kawanusa/Penjejak)

---

## 1. Stack Tim Terlalu Gemuk

### Peran yang Bisa DIHAPUS:

**AI/Algoritma Specialist (Rp7jt/bulan):**
- Recommendation engine bisa pakai rule-based filtering atau collaborative filtering basic di backend
- "AI" di konteks ini cuma filter preferensi — logika IF-ELSE + weighted scoring, bukan machine learning
- Backend dev bisa handle

**Cloud Infrastructure Engineer (Rp3jt/bulan):**
- Untuk skala awal, managed service (Firebase, Supabase, Railway, Vercel) sudah cukup
- Auto-scaling disediakan provider, tidak perlu orang dedicated

**Penghematan: Rp10jt/bulan = Rp120jt/tahun**

---

## 2. Tim Realistis untuk MVP/Pilot (5 orang)

| Role | Tugas | Estimasi |
|---|---|---|
| UI/UX Designer | Desain Figma, user flow, prototyping | Bisa part-time / freelance |
| Fullstack/Mobile Dev (1-2 orang) | Frontend + Backend + Database + deploy | Core team |
| Trust & Safety / Community Ops | Moderasi konten UGC, verifikasi spot, handle laporan | Bisa part-time awal |
| Customer Support | CS + onboarding user | Bisa part-time / dipegang founder |
| Partnership / BD | Jalin mitra agen travel, pemda, local guide | Founder sendiri |

Total gaji bisa turun ke ~Rp20-25jt/bulan (dari Rp42,5jt).

---

## 3. Fitur "AI Recommendation Engine" — Overkill

- **Tahap 1:** Rule-based filtering — user isi preferensi (budget, minat, lokasi) -> query DB destinasi yang match -> sort by rating/jarak. Ini SQL biasa, bukan AI.
- **Tahap 2:** Collaborative filtering sederhana (kalau data user sudah cukup).
- Tidak perlu ML pipeline, model training, GPU compute di tahun pertama.

---

## 4. Teknologi Stack Realistis untuk Pilot

```
Mobile    : Flutter / React Native (cross-platform, satu codebase)
Backend   : Node.js (Express) / Python (FastAPI) / Supabase (BaaS)
Database  : PostgreSQL (Supabase managed)
Maps      : Google Maps Platform API (free tier $200/bulan)
Payment   : Midtrans / Xendit
Auth      : Firebase Auth / Supabase Auth (verifikasi KTP manual dulu)
Hosting   : Vercel (frontend) + Railway/Fly.io (backend) atau full Supabase
```

Biaya infrastruktur bulanan skala pilot (<5.000 user): **Rp500rb-2jt/bulan** (bukan Rp5jt).

---

## 5. Fitur Terlalu Banyak untuk MVP

### MVP yang Realistis (launch pertama):
1. Preferensi + rekomendasi destinasi (rule-based)
2. Trip matching sederhana (filter by tujuan + tanggal)
3. Profil user + verifikasi dasar
4. Gamifikasi basic (XP + badge)

### Fitur DITUNDA ke Fase 2+:
- Open Trip Marketplace (butuh banyak mitra dulu)
- Itinerary Builder Kolaboratif (kompleks, UX sulit)
- Offline Mode (effort engineering besar)
- Donasi/Support Point (belum krusial tanpa user base)
- Local Guide/Host (butuh onboarding manual per daerah)

---

## 6. "Mobile App" vs "Progressive Web App"

- **PWA:** Satu codebase, jalan di browser HP, bisa install ke home screen, push notification, akses kamera/GPS
- Tidak perlu submit ke Play Store/App Store (hindari review process & biaya developer account)
- Lebih cepat ke market, lebih murah develop & maintain
- Kalau sudah validated, baru port ke native app

---

## 7. Safety Layer — Tekankan Lebih

- **Verifikasi identitas:** integrasi API verifikasi KTP (Tada, Verihubs, atau manual upload + review). Ada biaya per verifikasi.
- **Live location sharing:** Google Maps realtime API — bisa diakali dengan interval update (tiap 5 menit, bukan realtime) untuk hemat biaya.
- **Moderasi konten UGC:** pipeline sederhana (auto-detect NSFW via cloud vision API + manual review).

---

## Ringkasan Penghematan

```
PENGHEMATAN BULANAN:
  Hapus AI Specialist        : -Rp7.000.000
  Hapus Cloud Engineer       : -Rp3.000.000
  Turunkan infrastruktur     : -Rp3.000.000 (dari Rp5jt ke Rp2jt)
  UI/UX jadi part-time       : -Rp2.000.000
  ─────────────────────────────────────
  Total hemat/bulan          : ~Rp15.000.000
  Total hemat/tahun          : ~Rp180.000.000
```

Proyeksi laba tahun 1 naik dari Rp185jt (~11%) menjadi ~Rp365jt (~22%).

---

# Tinjauan Perhitungan Keuangan

## Masalah 1: Weekly Pass — Inkonsistensi Perhitungan

Skenario "Aqil FIX" (115.000 download):
```
Weekly Pass = 17.250 user x Rp9.900 = Rp170.775.000 (sekali purchase)
x4  = Rp2.277.000.000  (kalau purchase 4x setahun)
x12 = Rp27.324.000.000 (kalau purchase 12x setahun)
```

**Masalah:** Weekly pass = bayar per pekan (52x/tahun), bukan per bulan (12x).
```
Seharusnya x52 = Rp170.775.000 x 52 = Rp8.880.300.000
```
Bukan x4 atau x12. Angka yang dipakai di total final tidak jelas pakai yang mana.

---

## Masalah 2: Monthly Pass — Asumsi Retention 100%

```
Monthly Pass: 34.500 user x Rp29.000 x 12 = Rp12.006.000.000 (Rp12 Miliar)
```

Mengasumsikan SEMUA 34.500 subscriber RETAIN 100% selama 12 bulan. Churn rate app subscription Indonesia ~30-50%/bulan. Dengan retention 70%/bulan:
```
Effective subscriber-tahun = 34.500 x (1-0.7^12)/(1-0.7) = ~112.264 user-months
Pendapatan = 112.264 x Rp29.000 = Rp3.255.656.000 (~Rp3,2 Miliar)
```
Bukan Rp12 Miliar. Lebih realistis di Rp3,2-3,5 Miliar.

---

## Masalah 3: Conversion Rate 30% Terlalu Agresif

```
115.000 download -> 34.500 Monthly Pass (30%)
```

Conversion rate app travel ke paid subscription di Indonesia rata-rata **3-8%**, bukan 30%. Dengan 5%:
```
115.000 x 5% = 5.750 subscriber (bukan 34.500)
```

---

## Masalah 4: Double-Counting Weekly + Monthly

34.500 Monthly + 17.250 Weekly (50% dari Monthly). Apakah 17.250 SUBSET dari 34.500 atau TERPISAH?
- Kalau subset: jangan dijumlahkan dua-duanya.
- Kalau terpisah: total paying user = 51.750 dari 115.000 = 45% conversion, makin tidak realistis.

---

## Masalah 5: Total "Sekali Purchase" Selisih Rp600jt

Ditulis: Rp1.632.525.000
Cek manual:
```
Rp1.000.500.000 + Rp1.031.550.000 + Rp170.775.000 + Rp30.000.000
= Rp2.232.825.000
```
Selisih **Rp600.300.000** — ada komponen yang salah hitung atau dikurangi tanpa penjelasan.

---

## Masalah 6: Total "Purchase Kembali" = Rp43 Miliar Tidak Verifiable

Pakai Monthly x12 (Rp12M) + Weekly x12 (Rp27,3M) yang keduanya bermasalah di atas. Seharusnya pakai x52 untuk weekly, bukan x12.

---

## Masalah 7: Cost Tidak Diskalakan

Cost tetap Rp1,4Miliar padahal revenue bisa turun drastis. Dengan asumsi realistis (5% conversion, 70% retention):
```
Estimasi revenue realistis: ~Rp2,0-2,3 Miliar/tahun
Total cost                : ~Rp1,4 Miliar/tahun
Laba                      : ~Rp600-900jt (margin ~30-40%)
```
Margin lebih sehat, tapi top-line jauh dari klaim Rp43 Miliar.

---

## Masalah 8: IRR Tidak Dihitung

Dokumen menyebut IRR di panduan BMC tapi tidak ada perhitungannya. Perlu ditambahkan.

---

## Masalah 9: Perbandingan Skenario Pilot vs General Tidak Konsisten

| Aspek | Pilot UI (5.000 dl) | General (115.000 dl) |
|---|---|---|
| Monthly subscriber | 1.500 (30%) | 34.500 (30%) |
| Scaling factor | 1x | 23x |
| Tapi download ratio | 1x | 23x |

Conversion rate dipertahankan 30% di kedua skenario — seharusnya skala kecil (pilot di kampus UI) justru punya conversion LEBIH TINGGI karena komunitas tertutup, sedangkan skala nasional conversion PASTI lebih rendah.

---

## Masalah 10: Kompetitor & Monetisasi = "XXX"

Dua pertanyaan krusial di BMC diisi "XXX":
- Research Competitors and Alternatives
- Think About How the Business Will Make Money (detail)

Ini bagian yang dinilai juri — harus diisi.

---

## Ringkasan Masalah Perhitungan

| # | Masalah | Dampak |
|---|---------|--------|
| 1 | Weekly Pass x12 bukan x52 | Revenue over/under-count |
| 2 | 100% retention 12 bulan | Monthly revenue over-count ~3-4x |
| 3 | 30% conversion (realistis 5%) | Revenue over-count ~6x |
| 4 | Double-count Weekly + Monthly | Potensi revenue ganda |
| 5 | Total "sekali purchase" selisih Rp600jt | Hitungan salah |
| 6 | Total "kembali" Rp43M tidak verifiable | Tidak bisa diverifikasi |
| 7 | Cost tidak diskalakan | IRR/margin misleading |
| 8 | IRR tidak dihitung | Tidak lengkap |
| 9 | Skenario pilot vs general konsisten di 30% | Tidak realistis |
| 10 | Kompetitor & monetisasi = "XXX" | Bagian krusial kosong |

---

# Tinjauan Tambahan

## A. Struktur BMC — Bagian yang Lemah

### Competitors (diisi "XXX"):
Padahal ada kompetitor langsung:
- **Traveloka/Tiket.com** (open trip + booking)
- **Couchsurfing** (trip matching antar stranger)
- **TripAdvisor/Wikiloc** (rekomendasi destinasi)
- **Backpacker Indonesia** (komunitas)
- **Jejak.in** (nama mirip, bisa masalah trademark!)

Perlu diisi: apa differensiasi Jejakawan vs mereka?

### Monetisasi Detail (diisi "XXX"):
Revenue streams sudah ada di bagian Aqil — tinggal copy paste + refine ke format BMC.

---

## B. Value Proposition — Perlu Diperkuat

Dokumen tidak menjawab pertanyaan kunci juri: *"Kenapa user tidak pakai Traveloka yang sudah punya semua fitur + brand kuat?"*

Jawaban yang harus ada:
- Traveloka = booking transactional, tidak ada trip matching
- Trip matching = fitur unik (belum ada pemain besar doing this for travel)
- Gamifikasi ke underrated spot = social impact angle yang Traveloka tidak punya
- Tapi harus di-back data: berapa % user travel yang actually solo travel & butuh teman?

---

## C. Target Market — Gap di Segmen Tersier

Segmen Tersier = warga lokal UMKM. Masalah:
- Bagaimana warga di NTT/Sulawesi yang sinyalnya lemah bisa onboarding?
- Siapa yang kurasi & verifikasi local guide di daerah terpencil?
- Ada cost onboarding yang tidak dihitung di Variable Cost

---

## D. Asumsi Download 115.000 di Tahun Pertama — Terlalu Agresif

Untuk app baru tanpa brand awareness:
- Tahun 1 app travel baru Indonesia: 5.000-20.000 download realistis
- 115.000 butuh budget marketing besar
- CAC Rp10.000/user sangat optimistis (realistis Rp15.000-50.000/user untuk app travel)
- Di dokumen, CAC 40% dari 115.000 x Rp10.000 = Rp460jt — ini sudah underestimate

---

## E. Tidak Ada Break-Even Analysis

Dokumen tidak hitung:
- Berapa user minimum untuk break-even?
- Berapa bulan sampai break-even?
- Burn rate bulanan sebelum revenue masuk?

Quick estimate:
```
Fixed Cost/bulan         = Rp53.000.000
Revenue/user/bulan       = ~Rp25.000 (blended dari semua pass)
Break-even paying user   = Rp53.000.000 / Rp25.000 = 2.120 user
Dengan 5% conversion     = butuh 42.400 download untuk break-even
```

Ini target yang realistis — tapi perlu ditulis eksplisit di BMC.

---

## F. Tidak Ada Analisis Risiko

Bagian yang sering diminta juri tapi tidak ada:
- **Risiko regulasi:** verifikasi KTP = data pribadi, butuh compliance PDP Law (UU Perlindungan Data Pribadi 2022)
- **Risiko keamanan insiden:** kalau terjadi kejahatan antar user yang matched, reputasi app hancur
- **Risiko teknis:** dependency pada API pihak ketiga (Google Maps, payment gateway) — kalau pricing berubah atau service down
- **Risiko pasar:** Traveloka/Tiket.com tiba-tiba launch fitur serupa
- **Risiko operasional:** kurasi konten UGC di 17.000+ pulau Indonesia butuh manusia, tidak scalable otomatis

---

## G. Legal & Compliance — Tidak Dibahas

Dokumen tidak menyentuh:
- **PT/Entity:** butuh badan hukum untuk terima payment & kerja sama B2B
- **UU PDP:** proses KTP + live location = data sensitif, wajib compliance
- **Perizinan:** platform digital yang handle transaksi butuh izin khusus (PSE / Kominfo)
- **Asuransi/mitigasi:** tanggung jawab kalau terjadi insiden selama trip matching
- Di Fixed Cost hanya tertulis "Legal & perizinan (Rp15jt)" tanpa detail — ini sangat underestimate

---

## H. Aspek UX yang Tidak Dibahas

Dari 13 fitur yang disebut, flow antar fitur tidak jelas:
- **Onboarding friction:** kalau user harus isi preferensi panjang sebelum lihat apa-apa, drop-off tinggi. Best practice: progressive profiling (isi sedikit dulu, pelajari behavior)
- **Trip matching trust:** bagaimana user percaya orang asing? Perlu "warm introduction" mechanism, bukan langsung match
- **Gamifikasi balance:** kalau XP terlalu mudah didapat, meaningless. Kalau terlalu sulit, frustrasi. Perlu tuning yang butuh data
- **Offline vs online conflict:** fitur offline mode tapi banyak fitur real-time (trip matching, live location) — perlu jelas mana yang fungsi offline vs online

---

## I. Data & Analytics — Tidak Ada

Dokumen tidak menyebut:
- Tracking metrics apa yang dipakai? (DAU, MAU, retention rate, churn)
- A/B testing plan untuk fitur
- Feedback loop dari user ke recommendation engine
- Dashboard internal untuk monitor performa

Ini krusial untuk iterasi produk dan presentasi ke investor/juri.

---

## J. Competitive Moat — Tidak Jelas

Pertanyaan juri: *"Apa yang membuat kalian tidak bisa disalin Traveloka dalam 6 bulan?"*

Jawaban yang harus ada:
- **Network effect:** makin banyak user → makin akurat trip matching → makin menarik untuk user baru
- **Community & content:** konten UGC tentang underrated spot = aset yang tidak mudah disalin
- **Local partnerships:** hubungan eksklusif dengan local guide/host di daerah terpencil
- **Data moat:** data preferensi user yang terakumulasi = recommendation makin bagus seiring waktu

Tanpa ini, juri akan skeptis terhadap sustainability bisnis.

---

## Ringkasan Tinjauan Tambahan

| # | Aspek | Status |
|---|-------|--------|
| A | Kompetitor & monetisasi di BMC | KOSONG (isi "XXX") |
| B | Value proposition vs Traveloka | LEMAH |
| C | Segmen Tersier onboarding | TIDAK DIHITUNG |
| D | Asumsi download 115K | TERLALU AGRESIF |
| E | Break-even analysis | TIDAK ADA |
| F | Analisis risiko | TIDAK ADA |
| G | Legal & compliance | MINIMAL |
| H | UX flow antar fitur | TIDAK DIBAHAS |
| I | Data & analytics plan | TIDAK ADA |
| J | Competitive moat | TIDAK JELAS |

---

## K. Inkonsistensi Nama Produk

Dokumen pakai beberapa nama bergantian:
- **Kawanusa** (Drafting Idea)
- **Penjejak** (Drafting Idea)
- **Jejakawan** (BMC & Value Proposition)

Pilih SATU nama dan pakai konsisten di seluruh dokumen. Cek juga:
- "Jejak.in" sudah ada sebagai platform lain — perlu cek trademark
- Nama harus mudah diucapkan, diingat, dan tersedia sebagai domain/app name

---

## L. Analisis SDG — Perlu Indikator Terukur

SDG 8, 10, 11 disebut tapi hanya narasi. Juri minta angka target:
- **SDG 8 (target 8.9):** berapa lapangan kerja dari local host? Berapa % peningkatan pendapatan UMKM lokal?
- **SDG 10 (target 10.2):** berapa destinasi underrated yang terangkat? Metric: kunjungan daerah non-populer sebelum vs sesudah app
- **SDG 11 (target 11.4):** berapa warisan budaya yang dipromosikan?

Tanpa angka, SDG jadi sekadar label.

---

## M. Struktur Tim — Pembagian Tidak Merata

| Anggota | Beban | Masalah |
|---|---|---|
| Emma | 2 elemen (Segments + Relationships) | Paling ringan |
| Chika | 3 elemen (Value Prop + Channels + Resources/modal) | Terlalu berat, inti BMC |
| Dimas | 3 elemen (Activities + Partnerships + Resources/ops) | Oke |
| Aqil | 2 elemen (Revenue + Cost) | Paling berat teknis, banyak error |

- Tidak ada yang handle: Competitors, Break-Even, Risk Analysis, IRR (makanya kosong)

Saran redistribusi:
- **Emma:** Customer Segments + Customer Relationships + **Competitors**
- **Chika:** Value Propositions + Channels
- **Dimas:** Key Activities + Key Partnerships + Key Resources (semua)
- **Aqil:** Revenue Streams + Cost Structure + **IRR + Break-Even + Risk Analysis**

---

## N. Dokumen SAFE (Ide 2) — Mentah

Dibandingkan Jejakawan, SAFE cuma list fitur tanpa:
- BMC, target market, revenue model, competitor analysis, cost structure
- Data statistik bagus (I-NAMHS, WHO) tapi tidak jadi business case

Kalau untuk kompetisi yang sama, perlu BMC lengkap. Kalau ide cadangan, cukup jadikan referensi.

---

## O. Tebengs (Ide 3) — Cukup Matang tapi Perlu Angka

Sudah punya: problem statement, value proposition, target market (UI pilot), revenue model, expansion roadmap.

Yang kurang:
- Angka (cost, revenue projection, break-even)
- Competitor analysis (Grab Hitch, GoRide share, BlaBlaCar)
- Risk analysis keamanan penumpang

---

## P. Alokasi Waktu Presentasi

Kalau kompetisi 10-15 menit:
```
2 menit : Problem & opportunity
3 menit : Solution & value proposition
3 menit : Business model & revenue
2 menit : Market size & traction plan
2 menit : Team & competitive advantage
1 menit : Q&A buffer
```
Dokumen terlalu banyak narasi — perlu distilasi ke poin-poin.

---

## Q. Referensi & Sumber Data

Saat ini cuma 1 referensi (APJII/BI). Perlu tambah:
- Data pasar pariwisata Indonesia (Kemenparekraf)
- Data penggunaan app travel di Indonesia
- Benchmark subscription conversion rate
- Data kompetitor (user base, revenue)
- Riset tentang solo travel trend

---

## R. Kekuatan yang Perlu Dipertahankan

| # | Kekuatan | Kenapa Penting |
|---|----------|----------------|
| 1 | Trip Matching — belum ada pemain besar di niche ini | Differensiasi paling kuat vs Traveloka/Tiket.com |
| 2 | SDG angle — social impact ke daerah underrated | Bikin juri terkesan, beda dari startup travel biasa |
| 3 | Segmentasi user (wisatawan biasa vs explorer) | Pemikiran bisnis yang matang |
| 4 | Safety & Trust Layer sebagai fitur utama | Awareness tinggi, juri akan tanya ini duluan |
| 5 | TAM/SAM/SOM sudah dihitung dengan sumber data | Banyak proposal BMC tidak sampai tahap ini |
| 6 | Gamifikasi double purpose (retensi + social impact) | Bukan gimmick, ada tujuan bisnis |
| 7 | Customer Relationships sangat detail (7 tipe) | Konsep sudah dikuasai dengan baik |

---

## S. Pertanyaan Juri yang Harus Disiapkan

1. "Kenapa user tidak pakai Traveloka?" → Trip matching + gamifikasi underrated spot
2. "Traveloka bisa bikin fitur serupa dalam 6 bulan?" → Network effect + community UGC + local partnerships
3. "Conversion rate 30% dari mana?" → HARUS dikoreksi ke 5-8%
4. "Keamanan user yang matched dengan orang asing?" → Verifikasi KTP + rating + live location + lapor/blokir
5. "Break-even point?" → ~2.120 paying user = ~42.400 download
6. "Asumsi 115.000 download dari mana?" → HARUS dikoreksi atau justifikasi detail
7. "Siapa kompetitor langsung?" → Jawaban belum ada ("XXX") — ISI DULU
8. "Handle konten UGC palsu?" → Voting komunitas + verifikasi manual Trust & Safety
9. "IRR berapa?" → Belum dihitung — HARUS ISI
10. "Rencana kalau Traveloka launch fitur serupa?" → Pivot community-first

---

## T. Prioritas Perbaikan

### PRIORITAS 1 — KOREKSI ANGKA (sebelum presentasi):
- [ ] Koreksi conversion rate 30% → 5-8%
- [ ] Koreksi retention 100% → 70%/bulan
- [ ] Koreksi Weekly Pass x12 → x52
- [ ] Hitung ulang semua revenue dengan asumsi baru
- [ ] Hitung IRR
- [ ] Hitung break-even point
- [ ] Cek selisih Rp600jt di total "sekali purchase"

### PRIORITAS 2 — ISI BAGIAN KOSONG (sebelum submit):
- [ ] Competitors & Alternatives (bukan "XXX")
- [ ] Monetisasi detail (bukan "XXX")
- [ ] Analisis risiko
- [ ] Referensi tambahan (Kemenparekraf, data travel, dll)

### PRIORITAS 3 — PERKUAT NARASI (sebelum rehearsal):
- [ ] Pilih SATU nama (Kawanusa/Penjejak/Jejakawan)
- [ ] Kuatkan value proposition vs Traveloka
- [ ] SDG dengan indikator terukur
- [ ] Redistribusi beban tim
- [ ] Siapkan jawaban 10 pertanyaan juri

### PRIORITAS 4 — NICE TO HAVE:
- [ ] UX flow visual (bukan cuma tulisan)
- [ ] Dashboard metrics plan
- [ ] Competitive moat section
- [ ] Legal & compliance detail
