# 17 — Legal & Compliance

## 1. Privacy Policy (Kebijakan Privasi)

### 1.1 Landasan Hukum
- **UU No. 27/2022** — Pelindungan Data Pribadi (UU PDP)
- **PP 71/2019** — Penyelenggaraan Sistem dan Transaksi Elektronik
- **Permenkominfo 5/2020** — Penyelenggara Sistem Elektronik (PSE)

### 1.2 Data yang Dikumpulkan

```
┌─────────────────────────────────────────────────────────────┐
│ Kategori Data          │ Contoh                │ Basis Hukum │
├─────────────────────────────────────────────────────────────┤
│ Identitas              │ Nama, email, no. HP   │ Persetujuan │
│ Identitas Verifikasi   │ Foto KTP, NIK (hashed)│ Kontraktual │
│ Lokasi                 │ GPS real-time, riwayat │ Persetujuan │
│ Perangkat              │ OS, browser, IP        │ Sah Kepentingan│
│ Transaksi              │ Pembayaran, booking    │ Kontraktual │
│ Preferensi             │ Minat, filter, rating  │ Persetujuan │
│ Konten User            │ Review, foto, komentar │ Persetujuan │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 Tujuan Pemrosesan
1. Menyediakan layanan trip matching & rekomendasi destinasi
2. Memproses pembayaran via Midtrans
3. Verifikasi identitas untuk keamanan komunitas
4. Personalisasi konten & notifikasi
5. Analitik penggunaan untuk peningkatan produk
6. Komunikasi marketing (opt-in)

### 1.4 Dasar Pemrosesan (Pasal 20 UU PDP)
- **Persetujuan eksplisit** — marketing, analytics, lokasi presisi
- **Pelaksanaan kontrak** — booking, pembayaran, akun
- **Kepentingan sah** — keamanan, fraud prevention
- **Kewajiban hukum** — pelaporan pajak, permintaan penegak hukum

---

## 2. KTP Data Handling

### 2.1 Arsitektur KTP Processing

```
User Upload KTP
       │
       ▼
┌──────────────┐     ┌──────────────────┐     ┌───────────────┐
│ Client-side  │────▶│ Supabase Storage │────▶│ Edge Function │
│ compression  │     │ (encrypted)      │     │ (server-side) │
└──────────────┘     └──────────────────┘     └───────┬───────┘
                                                      │
                                              ┌───────▼───────┐
                                              │  Extract NIK  │
                                              │  Hash: SHA-256│
                                              │  + salt pepper│
                                              └───────┬───────┘
                                                      │
                                              ┌───────▼───────┐
                                              │ Store: hash   │
                                              │ Delete: image │
                                              │ after 24 jam  │
                                              └───────────────┘
```

### 2.2 Hashing NIK
```python
import hashlib, os

def hash_nik(nik: str) -> str:
    salt = os.environ.get('NIK_SALT')  # disimpan di Supabase Vault
    pepper = os.environ.get('NIK_PEPPER')  # server-side only
    return hashlib.sha256(f"{salt}:{nik}:{pepper}".encode()).hexdigest()
```

### 2.3 Retention & Deletion Policy

| Data | Retention | Deletion |
|------|-----------|----------|
| Foto KTP mentah | Max 24 jam setelah verifikasi | Auto-delete via cron job |
| Hash NIK | Selama akun aktif + 5 tahun (UU Pajak) | On account deletion |
| Nama & NIK (plain) | Tidak disimpan permanen | Hapus setelah hash |
| Log verifikasi | 1 tahun | Auto-purge |

### 2.4 Supabase RLS untuk KTP
```sql
-- Hanya service_role yang bisa akses KTP data
CREATE POLICY "ktp_service_only" ON ktp_verifications
  FOR ALL USING (auth.role() = 'service_role');

-- User hanya bisa lihat status verifikasi
CREATE POLICY "ktp_status_read" ON ktp_verifications
  FOR SELECT USING (user_id = auth.uid());
```

---

## 3. Terms of Service (Ringkasan)

### 3.1 Klausul Utama
1. **Penggunaan Layanan** — Platform perantara, bukan penyedia jasa travel
2. **Tanggung Jawab** — User bertanggung jawab atas konten yang di-upload
3. **Pembatalan** — Sesuai refund policy mitra + kebijakan Jejakawan
4. **Batasan Tanggung Jawab** — Force majeure, kerugian tidak langsung
5. **Hak Kekayaan Intelektual** — Konten user tetap milik user, beri lisensi non-eksklusif ke Jejakawan
6. **Penyelesaian Sengketa** — Mediasi → Arbitrase BANI → PN Jakarta Selatan

### 3.2 Age Requirement
- Minimum 17 tahun (butuh KTP untuk verifikasi)
- 13-17 tahun: akun terbatas (tanpa verifikasi KTP, fitur read-only)

---

## 4. Cookie & Tracking Policy

### 4.1 Jenis Cookie

| Cookie | Tujuan | Durasi | Wajib? |
|--------|--------|--------|--------|
| `session_id` | Autentikasi | Session | Ya |
| `sb-*` | Supabase auth token | 7 hari | Ya |
| `jejak_pref` | Preferensi user | 1 tahun | Tidak |
| `jejak_analytics` | Usage tracking | 30 hari | Tidak (opt-in) |
| `midtrans_*` | Payment session | 1 jam | Ya |

### 4.2 Implementasi Consent
```typescript
// Cookie consent banner — wajib tampil di first visit
interface CookieConsent {
  necessary: boolean;    // always true, tidak bisa di-disable
  analytics: boolean;    // opt-in
  marketing: boolean;    // opt-in
}
```

---

## 5. User Rights (Pasal 5-12 UU PDP)

### 5.1 Hak-hak Pengguna

| Hak | Deskripsi | SLA | Mekanisme |
|-----|-----------|-----|-----------|
| Akses | Salinan data pribadi | 3 hari kerja | In-app request |
| Koreksi | Perbaiki data salah | 3 hari kerja | In-app edit + request |
| Hapus | Right to be forgotten | 7 hari kerja | In-app + email konfirmasi |
| Portabilitas | Export data (JSON/CSV) | 7 hari kerja | In-app download |
| Batasi | Limitasi pemrosesan | 3 hari kerja | In-app toggle |
| Keberatan | Tolak pemrosesan tertentu | 3 hari kerja | Email ke dpo@jejakawan.id |
| Cabut Persetujuan | Revoke consent | Segera | In-app toggle |

### 5.2 Implementasi Data Export
```typescript
// Supabase Edge Function: data-export
async function exportUserData(userId: string) {
  const data = {
    profile: await supabase.from('profiles').select().eq('id', userId),
    trips: await supabase.from('trips').select().eq('user_id', userId),
    reviews: await supabase.from('reviews').select().eq('user_id', userId),
    achievements: await supabase.from('achievements').select().eq('user_id', userId),
    ktp_status: await supabase.from('ktp_verifications')
      .select('status, verified_at').eq('user_id', userId),
  };
  return JSON.stringify(data, null, 2);
}
```

---

## 6. Kominfo PSE Registration

### 6.1 Persyaratan
- [ ] Daftar di https://pse.kominfo.go.id
- [ ] Kategori: PSE Lingkup Privat — Platform Digital
- [ ] Upload: Akta pendirian, NPWP, KTP penanggung jawab
- [ ] Sertakan: URL, jumlah user, jenis layanan
- [ ] Deadline: 30 hari setelah operasional

### 6.2 PSE Number
- Format: `PSE-XXXXX-XX-XXXXX`
- Update setiap perubahan data perusahaan
- Cantumkan di footer website & halaman "Tentang Kami"

---

## 7. Data Breach Notification

### 7.1 Prosedur (Pasal 26 UU PDP — 3×24 jam)

```
Incident Detected
       │
       ▼
┌─────────────────────────┐
│ T+0: Incident confirmed │
│ → Activate IRT          │
│ → Preserve evidence     │
└───────────┬─────────────┘
            │
       ▼
┌─────────────────────────────────────┐
│ T+6 jam: Initial assessment         │
│ → Scope: data apa, berapa user      │
│ → Severity: P0/P1/P2                │
│ → Containment: revoke tokens, patch │
└───────────┬─────────────────────────┘
            │
       ▼
┌─────────────────────────────────────────────┐
│ T+24 jam: Notify Kominfo & BSSN             │
│ → Form: jenis insiden, data terdampak,       │
│   langkah mitigasi                           │
└───────────┬─────────────────────────────────┘
            │
       ▼
┌─────────────────────────────────────────────┐
│ T+48 jam: Notify affected users             │
│ → In-app banner + email                      │
│ → Apa yang terjadi, apa dampaknya,           │
│   apa yang user harus lakukan                │
└───────────┬─────────────────────────────────┘
            │
       ▼
┌─────────────────────────────────────────────┐
│ T+72 jam: Submit final report ke Kominfo    │
│ → Root cause, remediation, prevention plan   │
└─────────────────────────────────────────────┘
```

### 7.2 Template Notifikasi ke User
```
Subject: [Jejakawan] Pemberitahuan Keamanan Data

Yth. [Nama],

Pada [tanggal], kami mendeteksi akses tidak sah ke [jenis data].
Data yang terdampak: [detail].

Langkah yang kami ambil:
1. [containment action]
2. [remediation action]

Yang dapat Anda lakukan:
- Ganti password Anda
- Aktifkan 2FA
- Monitor aktivitas mencurigakan

Kami mohon maaf atas ketidaknyamanan ini.
Hubungi: security@jejakawan.id

— Tim Keamanan Jejakawan
```

---

## 8. Partnership Contracts

### 8.1 Agen Travel / Local Guide
- **Tipe**: Perjanjian Kerja Sama (PKS) — B2B
- **Klausul**: Revenue share (80:20), cancellation policy, liability
- **SLA**: Response time 24 jam, listing accuracy, pricing update
- **Term**: 1 tahun, auto-renew, terminasi 30 hari notice

### 8.2 Pemda / Dinas Pariwisata
- **Tipe**: MoU → PKS
- **Klausul**: Data sharing (aggregated only), branding, promotion
- **Benefit**: Dashboard analytics gratis, co-marketing
- **Approval**: Bupati/Walikota untuk MoU, Kadis untuk PKS

---

## 9. Refund Policy Legal Basis

| Skenario | Refund | Basis |
|----------|--------|-------|
| Pembatalan user H-7 | 100% | Kebijakan platform |
| Pembatalan user H-3 to H-6 | 50% | Kebijakan platform |
| Pembatalan user < H-3 | 0% | Force majeure exception |
| Mitra cancel | 100% + kompensasi | Pasal 1239 KUHPerdata |
| Force majeure (bencana) | 100% | Pasal 1244-1245 KUHPerdata |
| Service error (bug) | 100% + credit | Goodwill + consumer protection |

### Refund Process
1. User ajukan via in-app (max 7 hari setelah trip date)
2. Auto-approve jika < 24 jam dari pembayaran & belum dikonfirmasi mitra
3. Manual review untuk kasus lain (SLA: 3 hari kerca)
4. Refund via Midtrans reversal → kembali ke metode pembayaran asal

---

## 10. Data Protection Officer (DPO)

- **Penunjukan**: Wajib jika memproses > 10.000 data pribadi (target tercapai bulan ke-6)
- **Kontak**: dpo@jejakawan.id
- **Tugas**: Audit compliance, handle user requests, breach response
- **Pelaporan**: Langsung ke CEO, akses ke board

### Compliance Checklist
- [ ] Privacy policy published & accessible
- [ ] Cookie consent implemented
- [ ] KTP hashing pipeline tested
- [ ] Data export feature working
- [ ] Account deletion pipeline working
- [ ] PSE Kominfo registered
- [ ] Breach notification playbook tested (tabletop exercise)
- [ ] DPO appointed (or external DPO service contracted)
- [ ] Partnership contract templates reviewed by legal
- [ ] Refund policy integrated with Midtrans
