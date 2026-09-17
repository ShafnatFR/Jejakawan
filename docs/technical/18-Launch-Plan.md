# 18 — Launch Plan

## 1. Timeline Overview

```
Minggu -4  -3  -2  -1  [LAUNCH]  +1  +2  +3  +4
  │─────── Pre-Launch ───────│──── Post-Launch ────│
  │ Beta    Seed   Finalize  │  Launch  Scale     │
  │ Test    Comm.  Marketing │  Week   Growth     │
  └───────────────────────────┴─────────────────────┘
```

---

## 2. Pre-Launch (Minggu -4 sampai -1)

### 2.1 Week -4: Beta Testing

**Target**: 100 beta tester dari kalangan UI (Universitas Indonesia)

| Aktivitas | PIC | Deadline | Status |
|-----------|-----|----------|--------|
| Recruit 100 beta tester (mahasiswa UI) | Emma + Chika | Day 1 | ☐ |
| Setup TestFlight/Google Play Internal Testing | Dimas | Day 1 | ☐ |
| Buat feedback form (Google Forms) | Aqil | Day 2 | ☐ |
| Distribute beta invite via WhatsApp group | Chika | Day 2 | ☐ |
| Monitor crash reports (Sentry/Supabase logs) | Dimas | Ongoing | ☐ |
| Collect feedback & prioritize bugs | All | Day 5 | ☐ |
| Hotfix cycle #1 | Dimas | Day 6-7 | ☐ |

**Beta Tester Recruitment Script**:
```
Halo! 👋

Kami dari Jejakawan, platform travel baru yang bantu kamu
temukan destinasi underrated di Indonesia. 

Kami butuh 100 tester beta dari UI untuk coba aplikasi kami
dan kasih feedback. 

Benefit:
✅ Akses pertama ke aplikasi
✅ Badge "Pioneer Explorer" (permanent, exclusive)
✅ Voucher Rp 50.000 untuk trip pertama
✅ Kontribusi langsung ke pengembangan produk

Minat? Klik link ini: [URL]
Terima kasih! 🙏
```

**Feedback Categories**:
```
1. Onboarding — Apakah registrasi mudah? Verifikasi KTP lancar?
2. Discover — Apakah rekomendasi destinasi relevan?
3. Trip Matching — Apakah match-nya cocok?
4. Booking — Apakah proses booking jelas & aman?
5. Gamifikasi — Apakah achievement & badge menarik?
6. Performance — Loading time, crash, bug
7. UI/UX — Navigasi, desain, keterbacaan
```

### 2.2 Week -3: Community Seeding

**Target**: Bangun presence di komunitas traveler

| Channel | Target | Aktivitas | PIC |
|---------|--------|-----------|-----|
| Facebook: Backpacker Indonesia | 200K member | Share content, engage (bukan spam) | Chika |
| Facebook: Hiking & Camping ID | 150K member | Share trail reviews + link Jejakawan | Chika |
| Reddit: r/indonesia | — | AMA thread tentang underrated destinations | Aqil |
| Kaskus: Travel Forum | — | Thread Jejakawan + beta tester stories | Aqil |
| Twitter/X: Travel community | — | Thread storytelling destinasi underrated | Emma |
| WhatsApp: Komunitas hiking lokal | 10-20 grup | Share via personal network | All |

**Content Strategy**:
```
Week -3 Content Calendar:
├── Senin: Thread "5 Destinasi yang Gak Kamu Tahu Ada di Indonesia"
├── Rabu: Behind-the-scenes building Jejakawan (dev journey)
├── Jumat: User testimonial dari beta tester
└── Minggu: Infographic "Indonesia's Hidden Gems Map"
```

### 2.3 Week -2: Finalize Marketing Assets

| Asset | Format | PIC | Deadline |
|-------|--------|-----|----------|
| App icon & splash screen | PNG/SVG | Emma | Mon |
| Social media kit (templates) | Figma | Emma | Mon |
| Promo video 30 detik | MP4 | Chika | Wed |
| Landing page copy | HTML/MD | Aqil | Tue |
| Press release draft | DOCX | Emma | Wed |
| Influencer brief | PDF | Chika | Thu |
| Launch email template | HTML | Aqil | Thu |
| ASO metadata (Play Store) | Text | Dimas | Fri |

### 2.4 Week -1: Final Prep

| Aktivitas | PIC | Detail |
|-----------|-----|--------|
| Production deploy & smoke test | Dimas | Full E2E test |
| Load test (target: 500 concurrent) | Dimas | k6/Artillery |
| Payment integration final test (Midtrans sandbox→prod) | Dimas | End-to-end |
| Backup & disaster recovery drill | Dimas | Restore test |
| Launch day runbook review | All | Walkthrough |
| Social media scheduled posts | Chika + Emma | Buffer/Later |
| PR distribution list finalized | Emma | 20 media contacts |
| Monitoring dashboard setup | Dimas | Grafana/Supabase Dashboard |
| Support email auto-reply configured | Aqil | support@jejakawan.id |

---

## 3. Launch Week (Minggu 0)

### 3.1 Launch Day (Hari-H)

**Timeline**:
```
06:00 — Final production check (Dimas)
07:00 — Team huddle, confirm go/no-go (All)
08:00 — 🚀 LAUNCH: App set to public on Play Store & web PWA
08:30 — Social media blast: Instagram, TikTok, Twitter, Facebook
09:00 — Press release sent ke 20 media
10:00 — WhatsApp broadcast ke komunitas
12:00 — Monitor: signups, crashes, payment (Dimas)
14:00 — Influencer posts go live (terjadwal)
16:00 — First metrics check (All)
20:00 — Evening social engagement & reply comments
22:00 — Day 1 wrap: metrics review & issue triage
```

### 3.2 Monitoring Dashboard (Launch Day)

```
┌─────────────────────────────────────────────────────┐
│ Jejakawan Launch Dashboard                          │
├─────────────┬───────────┬───────────┬───────────────┤
│ Signups     │ Bookings  │ Crashes   │ DAU           │
│    [LIVE]   │  [LIVE]   │  [LIVE]   │  [LIVE]       │
├─────────────┼───────────┼───────────┼───────────────┤
│ Payment OK  │ Support   │ NPS Score │ Load Time     │
│    [LIVE]   │ Tickets   │           │ p95           │
└─────────────┴───────────┴───────────┴───────────────┘
```

### 3.3 Launch Week Activities

| Hari | Aktivitas | PIC |
|------|-----------|-----|
| Sen (H) | Launch day (see above) | All |
| Sel (H+1) | Reply semua review & feedback | Chika + Aqil |
| Rab (H+2) | Push update: hotfix minor bugs | Dimas |
| Kam (H+3) | TikTok content: "Pertama kali pakai Jejakawan" | Chika |
| Jum (H+4) | Instagram Live: Q&A dengan founding team | Emma |
| Sab (H+5) | Share weekend trip stories dari user | Aqil |
| Min (H+6) | Week 1 metrics report & retro | All |

---

## 4. Marketing Channels Detail

### 4.1 Instagram Strategy
```
Content Mix:
├── 40% — Destinasi underrated (carousel, Reels)
├── 25% — User-generated content (repost stories)
├── 20% — Behind-the-scenes (team, tech, journey)
└── 15% — Promotional (launch offer, features)

Posting Schedule: 1 post/day + 3-5 stories/day
Hashtags: #Jejakawan #DestinasiUnderrated #ExploreIndonesia
          #HiddenGemIndonesia #TripMatching
```

### 4.2 TikTok Strategy
```
Content Types:
├── "POV: Kamu nemuin hidden gem di [kota]" — storytelling
├── "Rate my trip" — user review format
├── "3 destinasi underrated di [provinsi]" — listicle
├── "How we built Jejakawan" — founder story
└── Duet/stitch dengan travel creator

Posting: 3-5 videos/minggu
Target: FYP travel & lifestyle Indonesia
```

### 4.3 WhatsApp Groups
```
Approach: VALUE FIRST, jangan spam
├── Share tips & stories (bukan langsung promo)
├── Invite ke grup "Jejakawan Travelers" (official)
├── Referral program: Ajak teman → dapat voucher
└── Weekly "Destination of the Week" broadcast
```

### 4.4 PR Strategy

**Target Media**:
```
Tier 1 (Nasional):
├── Kompas.com / Kompas Travel
├── Detik.com / Detik Travel
├── CNN Indonesia
├── Liputan6
└── Tirto.id

Tier 2 (Tech):
├── DailySocial.id
├── TechInAsia ID
├── StartupSnesia
├── Selular.id
└── Nextren

Tier 3 (Travel Niche):
├── Traveloka Blog (guest post)
├── TripZilla Indonesia
├── Wonderful Indonesia (pemda angle)
└── Backpacker Jakarta community blog
```

**Press Release Angle**:
> "Jejakawan: Platform Travel Indonesia yang Pakai AI untuk Hubungkan Wisatawan dengan Destinasi Underrated — Subtema Tourism & Culture DIGIVERSE Festival"

### 4.5 Influencer Outreach

**Micro Influencer (10K-100K followers)**:
| Kategori | Target | Budget | Deliverable |
|----------|--------|--------|-------------|
| Travel blogger | 5 orang | Rp 500K/orang | 1 blog post + 2 stories |
| Travel vlogger | 3 orang | Rp 1.5J/orang | 1 video YouTube/TikTok |
| Hiking/outdoor | 3 orang | Rp 500K/orang | 1 Reel + 3 stories |
| Food & culture | 2 orang | Rp 500K/orang | 1 carousel + stories |

**Total Influencer Budget**: Rp 10 Juta

**Outreach Template**:
```
Subject: Kolaborasi dengan Jejakawan — Platform Travel Baru 🌴

Halo [Nama],

Saya [Emma/Chika] dari Jejakawan, platform travel yang bantu
traveler menemukan destinasi underrated di Indonesia.

Kami suka konten kamu tentang [topik spesifik] dan ingin
kolaborasi:

🎯 Brief: Coba fitur trip matching Jejakawan, lalu share
   pengalamanmu di [platform]

💰 Kompensasi: Rp [amount] + akses premium gratis

📅 Timeline: [tanggal]

Tertarik? Balas email ini atau WA ke [nomor].

Salam,
[Nama] — Jejakawan Team
```

---

## 5. Post-Launch (Minggu +1 sampai +4)

### 5.1 Week +1: Stabilize & Listen
- Fix top 5 bugs reported by users
- Respond to all reviews (Play Store, social media)
- Publish "Week 1 Thank You" post
- A/B test onboarding flow

### 5.2 Week +2: Content Push
- Publish 3 blog posts (SEO: "destinasi underrated [kota]")
- Launch referral program (give Rp 25K, get Rp 25K)
- Start Google Ads (budget: Rp 5 Juta/bulan)
- Partner with 3 local guides for content

### 5.3 Week +3: Partnership Push
- Pitch ke Dinas Pariwisata 3 daerah (Jawa Barat, Jawa Tengah, Bali)
- Onboard 10 agen travel lokal
- Launch "Local Expert" badge program

### 5.4 Week +4: Scale & Measure
- Full metrics review (retention, conversion, NPS)
- Plan V1.1 features based on user feedback
- Prepare investor pitch deck update
- Retrospective + OKR setting for bulan depan

---

## 6. Success Metrics per Phase

### 6.1 Pre-Launch Metrics
| Metric | Target | Measurement |
|--------|--------|-------------|
| Beta testers recruited | 100 | Sign-up count |
| Beta feedback collected | 80% response rate | Form submissions |
| Critical bugs fixed | 100% P0/P1 | Issue tracker |
| Social media followers (all) | 1,000 | Platform analytics |
| Email waitlist | 500 | Landing page |
| Media contacts secured | 20 | CRM |

### 6.2 Launch Week Metrics
| Metric | Target | Measurement |
|--------|--------|-------------|
| Downloads/installs | 500 | Play Store + PWA |
| Signups | 300 | Supabase auth |
| First bookings | 50 | Transaction table |
| Crash-free rate | >99% | Sentry |
| App store rating | ≥4.0 | Play Store |
| Social media reach | 50K | Platform analytics |
| Press coverage | 5 articles | Media monitoring |

### 6.3 Post-Launch (Month 1) Metrics
| Metric | Target | Measurement |
|--------|--------|-------------|
| MAU | 1,000 | Supabase analytics |
| D7 retention | >30% | Cohort analysis |
| Booking conversion | >5% | Funnel |
| NPS | >40 | In-app survey |
| Revenue | Rp 5 Juta | Midtrans |
| Partner onboarded | 15 | Admin dashboard |
| Support tickets resolution | <24 jam avg | Helpdesk |
| Referral signups | 100 | Referral tracking |

### 6.4 Metrics Tracking Template
```sql
-- Daily launch metrics query
SELECT
  date_trunc('day', created_at) as date,
  COUNT(*) FILTER (WHERE type = 'signup') as signups,
  COUNT(*) FILTER (WHERE type = 'booking') as bookings,
  SUM(amount) FILTER (WHERE type = 'booking') as revenue,
  COUNT(*) FILTER (WHERE type = 'review') as reviews
FROM events
WHERE created_at >= '2026-10-01'
GROUP BY 1
ORDER BY 1;
```

---

## 7. Launch Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Server overload | Medium | High | Load test + Supabase auto-scaling |
| Payment failure | Low | Critical | Midtrans sandbox test + fallback manual |
| Negative reviews | Medium | Medium | Fast response team + hotfix cycle |
| Low adoption | Medium | High | Referral incentives + influencer push |
| Competitor response | Low | Medium | Focus on unique features (trip matching) |
| KTP verification issues | Medium | High | Manual fallback + support escalation |

---

## 8. Launch Budget Summary

| Item | Budget |
|------|--------|
| Influencer marketing | Rp 10.000.000 |
| Google Ads (bulan pertama) | Rp 5.000.000 |
| Social media tools (Buffer, Canva Pro) | Rp 500.000 |
| PR distribution service | Rp 2.000.000 |
| Merchandise/swag (beta tester) | Rp 1.500.000 |
| Domain & email hosting | Rp 500.000 |
| Contingency (10%) | Rp 1.950.000 |
| **TOTAL** | **Rp 21.450.000** |
