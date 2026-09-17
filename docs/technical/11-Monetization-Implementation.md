# 11 — Monetization Implementation

## Revenue Streams (Teknis)

### 1. Subscription Pass

```
Plan & Pricing:
├── Free:       Rp0         — fitur dasar, iklan, rekomendasi terbatas (3/hari)
├── Weekly:     Rp9.900     — XP 1.5x, no iklan, 10 rekomendasi/hari
├── Monthly:    Rp29.000    — semua weekly + priority matching + filter detail
└── Exclusive:  Rp299.000/thn — semua monthly + early access + diskon partner + XP 2x

Perbedaan fitur per plan:
┌────────────────────────────┬──────┬────────┬─────────┬───────────┐
│ Fitur                      │ Free │ Weekly │ Monthly │ Exclusive │
├────────────────────────────┼──────┼────────┼─────────┼───────────┤
│ Rekomendasi destinasi/hari │ 3    │ 10     │ ∞       │ ∞         │
│ Trip matching              │ ✅    │ ✅      │ ✅       │ ✅         │
│ Priority matching          │ ❌    │ ❌      │ ✅       │ ✅         │
│ Filter partner detail      │ ❌    │ ❌      │ ✅       │ ✅         │
│ XP multiplier              │ 1x   │ 1.5x   │ 1.5x    │ 2x        │
│ Iklan                      │ ✅    │ ❌      │ ❌       │ ❌         │
│ Early access open trip     │ ❌    │ ❌      │ ❌       │ ✅         │
│ Diskon partner travel      │ ❌    │ ❌      │ ❌       │ ✅ (5%)    │
│ Badge premium              │ ❌    │ ❌      │ ❌       │ ✅         │
└────────────────────────────┴──────┴────────┴─────────┴───────────┘
```

### Payment Flow (Midtrans)

```
User pilih plan → checkout:

1. Frontend request POST /api/subscriptions
   { plan: "monthly", payment_method: "gopay" }

2. Backend buat transaction record
   └── transactions: type='subscription', amount=29000, status='pending'

3. Backend create Midtrans Snap transaction
   └── parameter = {
     transaction_details: {
       order_id: "SUB-{uuid}",
       gross_amount: 29000
     },
     customer_details: {
       first_name: user.display_name,
       email: user.email
     },
     callbacks: {
       finish: "https://app.jejakawan.id/payment/finish"
     }
   }

4. Midtrans return snap_token + redirect_url
5. Frontend redirect user ke Midtrans payment page
6. User pilih metode bayar (VA, e-wallet, QRIS, CC)
7. User bayar
8. Midtrans kirim webhook ke backend:
   POST /api/webhooks/midtrans
   {
     order_id: "SUB-{uuid}",
     transaction_status: "capture",
     fraud_status: "accept",
     signature_key: "..."
   }

9. Backend verifikasi signature (SHA512):
   hash = SHA512(order_id + status_code + gross_amount + MIDTRANS_SERVER_KEY)
   if hash != signature_key → reject

10. Backend update:
    ├── transactions.status = 'completed'
    ├── user_profiles.subscription = 'monthly'
    ├── user_profiles.subscription_expires_at = NOW() + 30 days
    └── subscriptions: create new record

11. Frontend: user redirect ke halaman sukses
```

### Auto-Renewal (Recurring)

```
Midtrans support recurring payment untuk kartu kredit.

Implementation:
├── Saat user bayar pertama kali dengan CC:
│   ├── Simpan token_recurring dari Midtrans
│   └── Set auto_renew = true di subscriptions table
├── Cron job harian jam 00:00 WIB:
│   ├── Cek semua subscriptions yang expires_at = HARI INI + 1 HARI
│   ├── Kalau auto_renew = true:
│   ├── Charge via Midtrans recurring API pakai token_recurring
│   ├── Success → extend expires_at + 1 period
│   └── Fail → kirim notifikasi reminder ke user
│       → Grace period 3 hari → downgrade ke free
├── Non-CC payment (VA, e-wallet, QRIS):
│   ├── Tidak bisa auto-renew
│   ├── Notifikasi 3 hari sebelum expired
│   ├── User harus bayar manual
│   └── Grace period 3 hari → downgrade ke free
```

---

### 2. Open Trip Marketplace Commission

```
Model: Komisi 10% dari setiap booking

Flow:
├── User booking paket: Rp500.000
├── Midtrans charge: Rp500.000
├── Backend catat:
│   ├── booking.total_price = 500000
│   ├── booking.commission = 50000 (10%)
│   └── booking.net_to_agency = 450000
├── Settlement ke agen travel:
│   ├── Mingguan (setiap Senin)
│   ├── Total net_amount dari semua booking minggu lalu
│   ├── Transfer ke rekening agen via Midtrans Disbursement
│   └── Invoice dikirim ke email agen
└── Fee breakdown:
    ├── Midtrans fee: ~2.5% (ditanggung Jejakawan)
    ├── Komisi Jejakawan: 10%
    └── Agen terima: ~87.5% dari harga paket

Contoh perhitungan:
  Booking: Rp500.000
  Midtrans fee (2.5%): Rp12.500
  Komisi (10%): Rp50.000
  Net ke agen: Rp450.000
  Net ke Jejakawan: Rp50.000 - Rp12.500 = Rp37.500
```

---

### 3. Advertising (Phase 2)

```
Tipe iklan:
├── Banner promoted destinasi dari pemda
│   ├── Harga: Rp2.000.000/bulan per banner
│   ├── Tampil di: Home screen, Explore tab
│   ├── Target: user di region terkait
│   └── Label: "Promoted" (transparan ke user)
├── Sponsored open trip listing
│   ├── Harga: Rp500.000/minggu per listing
│   ├── Listing muncul di posisi atas search results
│   └── Label: "Sponsored"
└── Google AdMob (generic ads untuk free user)
    ├── Tampil di: antar halaman, bottom banner
    ├── Hanya untuk: user tanpa subscription
    └── Revenue: eCPM Rp15.000-30.000 per 1000 impression

Implementation:
├── Iklan hanya tampil untuk user subscription = 'free'
├── User premium → zero ads (kecuali promoted destinasi dari pemda,
│   itu dianggap konten bukan iklan)
└── Ad placement: bottom banner + interstitial setiap 5 page view
```

---

### 4. Partnership (B2G & B2B)

```
Model: Langganan bulanan dari mitra

Pemda/Dinas Pariwisata:
├── Paket: Rp1.000.000/bulan
├── Dapat:
│   ├── Banner promoted destinasi di app
│   ├── Analytics dashboard kunjungan ke daerah mereka
│   ├── Laporan bulanan: berapa user yang visit, dari mana asalnya
│   └── Prioritas kurasi destinasi oleh Trust & Safety
├── Minimum commitment: 3 bulan
└── Target: 10-30 pemda di tahun pertama

Restoran Lokal:
├── Paket: Rp500.000/bulan
├── Dapat:
│   ├── Listing di halaman destinasi terkait
│   ├── Rekomendasi "Kuliner terdekat" saat user explore
│   └── Analytics: berapa user yang klik listing
└── Target: 20-50 restoran di tahun pertama

Implementation:
├── Tabel: partnerships
│   ├── partner_type (gov, restaurant, guide_agency)
│   ├── plan, price, starts_at, expires_at
│   └── features (JSON: banner, analytics, priority)
├── Billing: invoice manual di awal bulan
└── Payment: transfer bank (belum otomatis di Phase 1)
```

---

### 5. Donasi User

```
Model: User bisa donasi ke developer

Tier & Badge:
├── Rp10.000 - Rp49.999   → Badge "Bronze Supporter"
├── Rp50.000 - Rp199.999  → Badge "Silver Supporter"
├── Rp200.000 - Rp499.999 → Badge "Gold Supporter"
└── Rp500.000+            → Badge "Platinum Supporter"

Payment: Midtrans (sama seperti subscription)
Implementation:
├── Endpoint: POST /api/donations
├── Tabel: donations (user_id, amount, payment_id, status)
├── Setelah payment sukses:
│   ├── Badge ditambah ke user_badges
│   ├── XP +100
│   └── Tampil di profil publik: "❤️ Gold Supporter"
└── Tidak ada recurring (manual setiap kali mau donasi)
```

---

## Revenue Projection (Realistis)

### Asumsi Tahun 1 (MVP, 20.000 download, 5% conversion)

```
SUBSCRIPTION:
├── Download: 20.000
├── Paying users (5%): 1.000
├── Weekly (60%): 600 user x Rp9.900 x 26 cycle/tahun = Rp154.440.000
├── Monthly (30%): 300 user x Rp29.000 x 8 cycle (retention) = Rp69.600.000
├── Exclusive (10%): 100 user x Rp299.000 = Rp29.900.000
└── Subtotal: Rp253.940.000

MARKETPLACE COMMISSION:
├── 50 booking/bulan x Rp500.000 x 10% x 12 bulan = Rp30.000.000
└── Subtotal: Rp30.000.000

PARTNERSHIP:
├── 5 pemda x Rp1.000.000 x 12 = Rp60.000.000
├── 10 restoran x Rp500.000 x 12 = Rp60.000.000
└── Subtotal: Rp120.000.000

DONASI:
├── 200 donatur x rata-rata Rp50.000 = Rp10.000.000
└── Subtotal: Rp10.000.000

IKLAN (AdMob):
├── 19.000 free user x 30 page views/hari x 365 hari x eCPM Rp20/1000
├── = 19.000 x 30 x 365 x 0.020 = Rp4.161.000.000
│   → ini sangat optimistic, realistis 10-20% fill rate
├── Realistis: Rp400.000.000 - Rp800.000.000
└── Subtotal: Rp500.000.000 (estimasi tengah)

TOTAL REVENUE TAHUN 1: ~Rp913.940.000
TOTAL COST TAHUN 1: ~Rp1.447.063.125 (dari dokumen asli)
DEFISIT: ~Rp533.000.000

→ Tahun 1 kemungkinan BESAR merugi (normal untuk startup)
→ Break-even di bulan ke-15-18 kalau growth 10%/bulan
→ Butuh seed funding atau bootstrapping dari tim
```

---

## Cost Optimization (Teknis)

```
Yang bisa ditekan:
├── Gaji: Rp42.5jt/bulan → Rp20jt/bulan (tim ramping 3-4 orang)
├── Infrastruktur: Rp5jt/bulan → Rp500rb/bulan (Supabase free tier)
├── CAC: Rp10.000/user → Rp5.000/user (organic + community growth)
├── Verifikasi KTP: Rp2.000/user → Rp1.500/user (negotiate volume)
└── Total hemat: ~Rp25jt/bulan = Rp300jt/tahun

Revised cost: Rp1.447jt - Rp300jt = Rp1.147jt/tahun
Revised laba: Rp913jt - Rp1.147jt = -Rp234jt (defisit lebih kecil)

Break-even point: ~3.300 paying user x Rp29.000/bln rata-rata
  = Rp435jt/bulan revenue
  vs Rp95jt/bulan cost (revised)
  = butuh ~3.300 paying user untuk break-even
  = butuh ~66.000 download (5% conversion)
```

---

## Tax Handling (PPh & PPN)

```
Status: Jejakawan sebagai platform digital di Indonesia

PPN 11% (dikenakan ke user):
├── Berlaku untuk: subscription, marketplace booking, paid features
├── TIDAK berlaku untuk: donasi, free tier, voucher redemption
├── Implementation:
│   ├── Harga yang terlihat user: SUDAH termasuk PPN (tax-inclusive)
│   ├── Contoh: Rp29.000/bulan → PPN = Rp29.000 x 11/111 = Rp2.874
│   ├── Net revenue = Rp29.000 - Rp2.874 = Rp26.126
│   └── PPN dikumpulkan → disetor ke negara setiap bulan
├── Faktur pajak: wajib untuk B2B partnership (pemda, agen travel)
├── Threshold: wajib daftar PPN kalau revenue > Rp4.8 Miliar/tahun
│   └── Kalau belum threshold → belum wajib PPN, tapi tetap baik untuk siap
└── Registration: daftar sebagai Pengusaha Kena Pajak (PKP) di KPP

PPh 0.5% (UMKM):
├── Berlaku kalau: revenue bruto < Rp4.8 Miliar/tahun
├── Tarif: 0.5% dari revenue bruto
├── Implementation: setor sendiri setiap bulan (SPT Masa)
├── Contoh: revenue Rp1 Miliar/tahun → PPh = Rp5 juta/tahun
└── Ini sangat kecil, tidak signifikan impact ke margin

PPh 23 (untuk pembayaran ke mitra):
├── Berlaku: kalau bayar jasa ke badan usaha (agen travel, restoran)
├── Tarif: 2% dari bruto
├── Implementation: potong dari pembayaran ke mitra → setor ke negara
├── Mitra bisa kreditkan di SPT tahunan mereka
└── Contoh: bayar Rp450.000 ke agen → potong PPh 23 = Rp9.000

Cost impact:
├── Tahun 1 revenue ~Rp900jt → PPN collected ~Rp89jt (disetor, bukan cost)
├── PPh 0.5% = ~Rp4.5jt/tahun (cost kecil)
├── PPh 23 = ~Rp600rb/tahun (dari 50 booking x Rp600rb mitra payment)
└── Total tax cost: ~Rp5jt/tahun (sangat kecil relatif terhadap revenue)
```

---

## Refund Automation

```
Refund policy:
├── Subscription: no refund (cancel = tetap aktif sampai expired)
├── Open Trip booking:
│   ├── > 7 hari sebelum trip: refund 100%
│   ├── 3-7 hari: refund 50%
│   ├── < 3 hari: no refund
│   └── Force majeure (cuaca, bencana): refund 100%
├── Donasi: no refund

Refund flow (Midtrans):
├── 1. User request cancel → sistem cek refund eligibility
├── 2. Hitung refund amount berdasarkan policy
├── 3. Admin approve (manual di Phase 1, auto di Phase 2)
├── 4. Panggil Midtrans refund API:
│   └── POST /v2/{order_id}/refund
│       {refund_amount: amount, reason: "user cancellation"}
├── 5. Midtrans process: 3-7 hari kerja (tergantung metode bayar)
│   ├── VA Bank: refund ke VA → user perlu klaim manual
│   ├── E-wallet: refund ke saldo e-wallet
│   ├── QRIS: refund ke source
│   └── CC: refund ke kartu kredit
├── 6. Update booking.payment_status = 'refunded'
├── 7. Update transactions table
├── 8. Notifikasi ke user: "Refund Rp{amount} sedang diproses"
└── 9. Tabel: refunds (booking_id, amount, reason, status, midtrans_refund_id)

Auto-refund rules (Phase 2):
├── Trip cancelled oleh agen travel → auto-refund 100%
├── Trip cancelled karena force majeure → admin trigger → auto-refund 100%
└── User cancel dalam grace period (30 menit setelah booking) → auto-refund 100%
```

---

## Financial Reporting Dashboard (Admin)

```
Dashboard metrics:

Revenue Overview:
├── Revenue hari ini / minggu ini / bulan ini / tahun ini
├── Revenue per stream (subscription, marketplace, partnership, donation, ads)
├── Revenue trend chart (line chart, per hari)
├── Revenue by region (provinsi mana yang paling banyak transaksi)
└── Comparison: bulan ini vs bulan lalu (% change)

Subscription Metrics:
├── Active subscribers: {count} per plan
├── New subscribers this month: {count}
├── Churn rate: {percentage}
├── MRR (Monthly Recurring Revenue): {amount}
├── ARPU (Average Revenue Per User): {amount}
├── LTV (Lifetime Value): ARPU / churn rate
└── Subscription funnel: view → select plan → payment → active

Marketplace Metrics:
├── Bookings this month: {count}
├── Total GMV (Gross Merchandise Value): {amount}
├── Commission earned: {amount}
├── Top 10 packages by bookings
├── Cancellation rate: {percentage}
└── Average booking value: {amount}

Operational:
├── Payment success rate: {percentage}
├── Refund rate: {percentage}
├── Payment method distribution (VA vs e-wallet vs QRIS vs CC)
├── Failed payment reasons (pie chart)
└── Outstanding settlements ke mitra: {amount}

Export:
├── CSV export untuk semua metrics
├── Monthly report auto-generate (PDF)
└── Integration ke Google Sheets (untuk tim non-teknis)
```
