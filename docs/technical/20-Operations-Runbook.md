# 20 — Operations Runbook

## 1. Team Structure & On-Call Rotation

### 1.1 Tim & Role

```
┌─────────────────────────────────────────────────────┐
│                   Jejakawan Team                     │
├──────────┬───────────────────────────────────────────┤
│ Emma     │ Product Lead, UI/UX, PR, Legal            │
│ Chika    │ Marketing, Community, Content, Influencer │
│ Dimas    │ Backend, DevOps, Security, Database       │
│ Aqil     │ Frontend, QA, Support, Documentation      │
└──────────┴───────────────────────────────────────────┘
```

### 1.2 On-Call Rotation

```
Jadwal On-Call (1 minggu shift, 24/7):

Minggu 1: Dimas (primary) + Aqil (backup)
Minggu 2: Aqil (primary) + Dimas (backup)
Minggu 3: Dimas (primary) + Emma (backup)
Minggu 4: Emma (primary) + Aqil (backup)
(ulangi)

Aturan On-Call:
├── Primary: response time < 15 menit untuk P0/P1
├── Backup: response time < 30 menit
├── Shift handoff: Senin 09:00 WIB (briefing 15 menit)
├── Escalation: Jika primary tidak respond 15 menit → auto-call backup
└── Compensatory time-off: +1 hari libur setelah shift on-call
```

### 1.3 Contact Matrix

| Person | Role           | Phone     | WhatsApp | Email              |     |
| ------ | -------------- | --------- | -------- | ------------------ | --- |
| Emma   | Product Lead   | 08xx-xxxx | ✅        | emma@jejakawan.id  |     |
| Chika  | Marketing      | 08xx-xxxx | ✅        | chika@jejakawan.id |     |
| Dimas  | Backend/DevOps | 08xx-xxxx | ✅        | dimas@jejakawan.id |     |
| Aqil   | Frontend/QA    | 08xx-xxxx | ✅        | aqil@jejakawan.id  |     |

**External Contacts**:
| Vendor | Service | Contact | SLA |
|--------|---------|---------|-----|
| Supabase | Database/Auth | support@supabase.com | 24 jam (Pro) |
| Midtrans | Payment | midtrans.com/dashboard | 4 jam (Priority) |
| Vercel/Railway | Hosting | dashboard | 1 jam (Enterprise) |
| Google Maps | Maps API | Cloud Console | N/A |

---

## 2. Incident Response Playbook

### 2.1 Severity Levels

```
┌───────────────────────────────────────────────────────────────┐
│ Level │ Label       │ Examples                    │ Response  │
├───────────────────────────────────────────────────────────────┤
│ P0    │ Critical    │ App down, data breach,      │ < 15 min  │
│       │             │ payment failure, data loss   │ all hands │
├───────────────────────────────────────────────────────────────┤
│ P1    │ Major       │ Feature broken (booking),    │ < 1 jam   │
│       │             │ >10% error rate, slow (>5s)  │ on-call   │
├───────────────────────────────────────────────────────────────┤
│ P2    │ Minor       │ UI bug, non-critical feature │ < 4 jam   │
│       │             │ broken, cosmetic issues      │ on-call   │
├───────────────────────────────────────────────────────────────┤
│ P3    │ Low         │ Enhancement request, typo,   │ < 24 jam  │
│       │             │ nice-to-have                 │ backlog   │
└───────────────────────────────────────────────────────────────┘
```

### 2.2 P0 Incident Response

```bash
# STEP 1: Acknowledge (0-5 min)
# ─────────────────────────────
# - On-call acknowledges alert
# - Open incident channel: #incident-YYYYMMDD
# - Post: "P0 acknowledged by [name], investigating"

# STEP 2: Assess (5-15 min)
# ─────────────────────────
# Check system status:
curl -s https://jejakawan.id/api/health | jq .
curl -s https://status.supabase.com | head -20

# Check error rates (Supabase dashboard):
# → Database → Logs → Filter errors last 1 hour

# Check payment status:
# → Midtrans dashboard → Transactions → Filter failed

# STEP 3: Communicate (15 min)
# ────────────────────────────
# Post in #general:
# "⚠️ [P0] Kami sedang mengalami gangguan pada [fitur]. 
#  Tim sedang melakukan investigasi. Update dalam 30 menit."

# STEP 4: Mitigate (15-60 min)
# ─────────────────────────────
# Common mitigations:
# - Rollback deploy: git revert HEAD && deploy
# - Scale up: increase Supabase compute
# - Feature flag: disable broken feature
# - Maintenance mode: show static page

# STEP 5: Resolve & Verify
# ─────────────────────────
# - Apply fix
# - Verify in production
# - Monitor error rate drops to baseline

# STEP 6: Post-Mortem (within 48 jam)
# ────────────────────────────────────
# Template di section 2.4
```

### 2.3 Escalation Matrix

```
Time Elapsed    │ Action
────────────────┼───────────────────────────────────
0 min           │ On-call primary responds
15 min          │ Notify backup on-call
30 min          │ Notify all team (WhatsApp group)
1 jam           │ Notify founder/CEO
2 jam           │ Consider external help (Supabase/Midtrans support)
4 jam           │ User-facing communication (status page)
24 jam          │ Formal incident report (if not resolved)
```

### 2.4 Post-Mortem Template

```markdown
# Post-Mortem: [Incident Title]

**Date**: YYYY-MM-DD
**Duration**: X jam Y menit
**Severity**: P0/P1/P2
**Author**: [Nama]
**Reviewers**: [Nama tim]

## Summary
[1-2 kalimat: apa yang terjadi, dampaknya]

## Impact
- Users affected: ~X
- Revenue impact: Rp X
- Duration: X jam
- Data exposure: Ya/Tidak

## Timeline (WIB)
| Time | Event |
|------|-------|
| HH:MM | [Alert triggered / user report] |
| HH:MM | On-call acknowledged |
| HH:MM | Root cause identified |
| HH:MM | Fix deployed |
| HH:MM | Verified resolved |

## Root Cause
[Penjelasan teknis mengapa ini terjadi]

## Resolution
[Apa yang dilakukan untuk fix]

## Prevention
| # | Action Item | Owner | Deadline |
|---|-------------|-------|----------|
| 1 | [Preventive action] | [Nama] | [Date] |
| 2 | [Monitoring improvement] | [Nama] | [Date] |
| 3 | [Process change] | [Nama] | [Date] |

## Lessons Learned
- [Lesson 1]
- [Lesson 2]
```

---

## 3. Daily / Weekly / Monthly Operations

### 3.1 Daily Tasks

```
08:00 — Morning Check (Dimas/on-call)
├── [ ] Check Supabase dashboard: DB health, query performance
├── [ ] Check error logs (Sentry/Supabase): new errors?
├── [ ] Check payment status (Midtrans): failed transactions?
├── [ ] Check uptime monitoring: any downtime last 24h?
└── [ ] Check #support channel: unresolved tickets?

09:00 — Standup (All, 15 min, async via Slack)
├── Emma: Product updates, blockers
├── Chika: Marketing metrics, community issues
├── Dimas: Tech status, deployments
└── Aqil: QA status, support summary

10:00 — Support Triage (Aqil)
├── [ ] Review new support tickets
├── [ ] Categorize: bug, question, feedback, complaint
├── [ ] Assign priority: P0-P3
├── [ ] Respond to user inquiries (< 2 jam SLA)
└── [ ] Escalate bugs to Dimas

12:00 — Content Check (Chika)
├── [ ] Review scheduled social media posts
├── [ ] Respond to social media comments/DMs
├── [ ] Monitor brand mentions
└── [ ] Check UGC: new trip stories, reviews

17:00 — End-of-Day (All)
├── [ ] Update task board (done/in-progress/blocked)
├── [ ] Dimas: deploy checklist if ada release
└── [ ] Aqil: support ticket summary for the day
```

### 3.2 Weekly Tasks

```
Senin:
├── 09:00 — Weekly planning (All, 30 min)
│   ├── Review last week metrics
│   ├── Set this week priorities
│   └── Assign tasks
├── Review backlog prioritization (Emma + Dimas)
└── Update investor dashboard (Emma)

Rabu:
├── Content review & approval (Emma reviews Chika's content)
├── Security check: dependency audit (npm audit, pip audit)
└── Database maintenance: VACUUM ANALYZE, index check

Jumat:
├── 15:00 — Weekly retro (All, 30 min)
│   ├── What went well?
│   ├── What didn't go well?
│   └── Action items
├── Metrics report: DAU, bookings, revenue, NPS
├── Partner check-in: any issues from agen travel/local guide?
└── Code freeze for weekend (kecuali hotfix)
```

### 3.3 Monthly Tasks

```
Minggu 1:
├── Monthly metrics review & OKR check (All, 1 jam)
├── Financial reconciliation (Midtrans vs internal records)
├── Supabase usage review (storage, bandwidth, compute)
├── Dependency updates (security patches prioritized)
└── Partner satisfaction survey (agen travel, local guide)

Minggu 2:
├── Performance audit (Lighthouse, Core Web Vitals)
├── Database optimization (slow query log review)
├── Content audit (review flagged content, update listings)
└── Competitive intelligence update

Minggu 3:
├── Feature planning for next month (Emma + All)
├── User research: interview 5 users (Emma + Chika)
├── Compliance check: UU PDP, PSE status
└── Marketing campaign review & next month plan

Minggu 4:
├── Sprint retrospective & planning
├── Infrastructure review (costs, scaling needs)
├── Team 1-on-1s (Emma with each member)
└── Board/investor update (if applicable)
```

---

## 4. Admin Dashboard SOP

### 4.1 Dashboard Access

```
URL: admin.jejakawan.id (atau /admin route di main app)
Auth: Supabase Auth + admin role check
2FA: Wajib untuk semua admin

Supabase RLS:
```sql
CREATE POLICY "admin_full_access" ON ALL TABLES
  USING (
    EXISTS (
      SELECT 1 FROM user_roles 
      WHERE user_id = auth.uid() AND role = 'admin'
    )
  );
```
```

### 4.2 Admin Operations

**User Management**:
```
├── View user list: /admin/users
│   ├── Filter: status (active, banned, pending verification)
│   ├── Filter: verification (KTP verified, unverified)
│   └── Search: email, name, user ID
├── Ban user: /admin/users/[id] → "Ban User"
│   ├── Reason required (stored in audit log)
│   ├── Auto-revoke all sessions
│   └── Notify user via email
└── Reset password: /admin/users/[id] → "Reset Password"
    └── Sends magic link to user email
```

**Content Moderation**:
```
├── Review queue: /admin/moderation
│   ├── Flagged reviews (auto-flagged by keywords or user reports)
│   ├── Reported trip stories
│   └── Reported user profiles
├── Actions:
│   ├── Approve (remove flag)
│   ├── Edit (fix content, keep post)
│   ├── Hide (soft-delete, user can see but others can't)
│   └── Delete (hard-delete, notify user with reason)
└── Audit log: semua modifikasi tercatat
```

**Transaction Management**:
```
├── Transaction list: /admin/transactions
│   ├── Filter: status (pending, success, failed, refunded)
│   ├── Filter: date range, amount range
│   └── Export: CSV
├── Manual refund: /admin/transactions/[id] → "Refund"
│   ├── Requires reason
│   ├── Triggers Midtrans refund API
│   └── Notify user via email + in-app
└── Dispute handling: /admin/disputes
    ├── View dispute details
    ├── Chat with both parties
    └── Resolution: full refund, partial refund, deny
```

---

## 5. Content Moderation SOP

### 5.1 Auto-Moderation Rules

```javascript
// Keyword filter (bahasa Indonesia + English)
const BLOCKED_KEYWORDS = [
  // Hate speech, SARA
  'keyword1', 'keyword2', // (actual list maintained separately)
  // Spam patterns
  /wa\.me\/\d+/gi,  // WhatsApp links
  /bit\.ly\/.+/gi,   // Shortened URLs in reviews
  /\b(beli|jual|promo|diskon)\b.*\b(dm|chat|wa)\b/gi, // Spam selling
];

// Auto-flag rules
const AUTO_FLAG_RULES = {
  review_too_short: (text) => text.length < 20,
  review_all_caps: (text) => text === text.toUpperCase() && text.length > 10,
  review_duplicate: (text, userId) => checkDuplicate(text, userId),
  review_rating_mismatch: (text, rating) => {
    const sentiment = analyzeSentiment(text);
    return (sentiment === 'positive' && rating <= 2) ||
           (sentiment === 'negative' && rating >= 4);
  },
};
```

### 5.2 Manual Review Process

```
Moderation Queue Priority:
1. User reports (highest priority — user took action)
2. Auto-flagged (keyword match)
3. New user first post (trust-building)
4. Random audit (10% sample)

Review SLA:
├── User reports: < 4 jam
├── Auto-flagged: < 12 jam
├── New user: < 24 jam
└── Random audit: < 48 jam

Decision Framework:
├── Apakah melanggar ToS? → Ya/Tidak
├── Apakah berbahaya? → Ya/Tidak  
├── Apakah spam? → Ya/Tidak
└── Action: Approve / Edit / Hide / Delete + Notify
```

### 5.3 Content Guidelines

```
Diperbolehkan:
├── Review jujur (positif maupun negatif)
├── Kritik konstruktif terhadap destinasi/mitra
├── Foto & konten original
├── Pertanyaan & diskusi
└── Tips & rekomendasi

Tidak Diperbolehkan:
├── Konten SARA, pornografi, kekerasan
├── Spam, iklan, promosi tanpa izin
├── Informasi pribadi orang lain (doxxing)
├── Penipuan, misinformasi
├── Plagiarisme konten
└── Ujaran kebencian, bullying
```

---

## 6. Customer Support SOP

### 6.1 Support Channels

```
├── In-app chat (primary) → Supabase Realtime chat
├── Email: support@jejakawan.id → Forwarded ke #support
├── WhatsApp: 08xx-xxxx (business account) → Manual
├── Social media DMs → Chika monitors
└── App store reviews → Aqil responds
```

### 6.2 Ticket Classification

```
Kategori:
├── BUG — Aplikasi error, crash, fitur tidak jalan
├── BILLING — Pembayaran gagal, refund, double charge
├── ACCOUNT — Login, verifikasi KTP, profil
├── BOOKING — Booking issue, cancel, reschedule
├── GENERAL — Pertanyaan umum, feedback, saran
└── ESCALATE — Perlu attention khusus (legal, safety)

Prioritas:
├── P0: Billing error, account hack, safety concern → < 1 jam
├── P1: Bug blocking, booking failure → < 4 jam
├── P2: Non-critical bug, feature question → < 12 jam
└── P3: Feedback, suggestion → < 24 jam
```

### 6.3 Template Responses

**Greeting**:
```
Halo [Nama]! 👋

Terima kasih sudah menghubungi Jejakawan Support.
Saya [Aqil/Emma] dari tim kami.

Ada yang bisa saya bantu?
```

**Payment Failed**:
```
Halo [Nama],

Mohon maaf atas kendala pembayaran yang Anda alami.

Detail transaksi:
- Order ID: [ID]
- Amount: Rp [amount]
- Status: [status]

Langkah yang bisa dilakukan:
1. Coba ulang pembayaran dengan metode yang sama
2. Gunakan metode pembayaran lain
3. Jika dana sudah terpotong, mohon kirimkan bukti 
   transfer ke email ini

Kami akan proses dalam 1×24 jam. 
Terima kasih atas kesabaran Anda. 🙏
```

**Refund Request**:
```
Halo [Nama],

Terima kasih atas pengajuan refund Anda.

Detail:
- Order ID: [ID]
- Trip: [destinasi]
- Tanggal: [date]
- Amount: Rp [amount]

Kebijakan refund kami:
- Pembatalan H-7: 100% refund
- Pembatalan H-3 s/d H-6: 50% refund
- Pembatalan < H-3: Tidak bisa refund*

*Pengecualian: force majeure, kesalahan mitra

Refund akan diproses dalam 3-5 hari kerca ke 
metode pembayaran asal Anda.

Ada pertanyaan lain?
```

**KTP Verification Issue**:
```
Halo [Nama],

Terima kasih atas laporannya terkait verifikasi KTP.

Tips untuk verifikasi yang berhasil:
1. Foto KTP dalam pencahayaan yang baik
2. Pastikan seluruh KTP terlihat (tidak terpotong)
3. Hindari refleksi cahaya pada KTP
4. KTP harus masih berlaku

Jika masih gagal setelah 3 percobaan, 
kami bisa bantu verifikasi manual. 
Silakan kirim foto KTP ke email: verify@jejakawan.id

Kami jamin data Anda aman dan hanya digunakan 
untuk verifikasi. 🛡️
```

**Bug Report Acknowledgment**:
```
Halo [Nama],

Terima kasih sudah melaporkan bug ini! 🐛

Detail laporan:
- Device: [info]
- OS: [info]
- App version: [info]
- Steps to reproduce: [steps]

Tim teknis kami sedang investigasi. 
Kami akan update Anda ketika sudah ada perbaikan.

Terima kasih sudah membantu kami improve! 🙏
```

### 6.4 Escalation Rules

```
Auto-escalate to Dimas:
├── Bug yang mempengaruhi > 10 user
├── Payment system error
├── Database connectivity issue
└── Security concern

Auto-escalate to Emma:
├── Legal/compliance issue
├── Media inquiry
├── Partner dispute
└── User threatening legal action

Auto-escalate to All (P0):
├── Data breach suspected
├── Complete system outage
├── Payment system completely down
└── Negative viral social media incident
```

---

## 7. Partner Onboarding SOP

### 7.1 Agen Travel

```
Step 1: Lead Qualification (Chika)
├── Check: punya izin usaha (NPWP, TDP)?
├── Check: punya track record (Google review, social media)?
├── Check: destinasi yang di-cover underrated?
└── Decision: qualify → lanjut / decline → email penolakan

Step 2: Partnership Agreement (Emma)
├── Kirim draft PKS (template tersedia)
├── Negosiasi: revenue share, SLA, branding
├── Tanda tangan digital (DocuSign/hand-sign scan)
└── Store: document archive (Google Drive ter-encrypt)

Step 3: Technical Onboarding (Dimas)
├── Buat partner account di admin dashboard
├── Setup: partner API key (jika integrasi)
├── Training: cara input listing, manage booking
├── Test: buat test listing + test booking
└── Go-live: listing published

Step 4: Marketing Support (Chika)
├── Co-create listing content (foto, deskripsi)
├── Feature di social media Jejakawan
├── Include di email newsletter
└── Assign "Verified Partner" badge

Step 5: Ongoing Management
├── Monthly check-in call
├── Performance review (quarterly)
├── Revenue payout: tanggal 5 setiap bulan
└── Contract renewal: 60 hari sebelum expiry
```

### 7.2 Local Guide

```
Step 1: Application
├── Apply via form: jejakawan.id/become-guide
├── Requirements:
│   ├── Domisili di destinasi yang di-cover
│   ├── Pengalaman minimal 1 tahun sebagai guide
│   ├── KTP verified (via Jejakawan app)
│   └── Referensi dari 2 traveler (bisa dari platform lain)
└── Review: Aqil verifikasi dalam 3 hari kerja

Step 2: Verification
├── Video call interview (15 menit)
├── Knowledge test: 10 pertanyaan tentang destinasi
├── Background check: social media scan
└── Decision: approve / reject / conditional (perlu training)

Step 3: Onboarding
├── Guide profile setup (foto, bio, expertise)
├── Pricing setup (per trip/per person)
├── Availability calendar
├── Training module: platform usage, safety protocol, SOP
└── First trip: supervised (admin monitor)

Step 4: Live
├── Badge: "Verified Local Expert" ⭐
├── Listing published di app
├── Notification ke matching travelers
└── Review cycle: setiap 10 trip → performance review
```

### 7.3 Pemda / Dinas Pariwisata

```
Step 1: Approach (Emma)
├── Identify target: dinas pariwisata daerah underrated
├── Siapkan proposal deck (sudah ada template)
├── Meeting request via surat resmi + email
└── First meeting: presentasi Jejakawan + value proposition

Step 2: MoU
├── Draft MoU: scope, duration, obligations
├── Review: dinas lawyer + Jejakawan legal
├── Tanda tangan: Bupati/Walikota (MoU) atau Kadis (PKS)
└── Publikasi: press release bersama

Step 3: Implementation
├── Data sharing: destinasi database (public data only)
├── Dashboard access: analytics gratis untuk dinas
├── Co-marketing: joint campaign di media sosial
├── Event: local tourism event sponsorship
└── Reporting: quarterly analytics report ke dinas

Step 4: Renewal
├── Review: impact metrics (kunjungan wisatawan, booking)
├── Proposal: perpanjangan dengan scope lebih luas
└── Testimonial: minta endorsement untuk approach dinas lain
```

---

## 8. Team Communication

### 8.1 Slack/WhatsApp Channels

```
WhatsApp Groups:
├── 🏠 Jejakawan Team — Semua diskusi internal
├── 🚨 Jejakawan On-Call — Alert & incident only
├── 💰 Jejakawan Finance — Transaksi, refund, payout
├── 📢 Jejakawan Marketing — Content, campaign, social
├── 🤝 Jejakawan Partners — Partner communication
└── 🎉 Jejakawan Fun — Non-work, team bonding

Slack (jika migrate):
├── #general — Announcements, non-urgent
├── #dev — Code review, deploy, tech discussion
├── #support — Customer support tickets
├── #marketing — Content, campaigns, analytics
├── #incidents — Auto-alerts + incident coordination
├── #random — Fun stuff
└── #standup — Async daily standup posts
```

### 8.2 Meeting Cadence

```
Daily:
├── 09:00 — Async standup (WhatsApp/Slack)
│   Format: "Kemarin: [X] | Hari ini: [Y] | Blocker: [Z]"
└── Duration: 0 menit (async)

Weekly:
├── Senin 09:00 — Planning (30 min, video call)
├── Rabu 14:00 — Tech sync (Dimas + Aqil, 15 min)
├── Jumat 15:00 — Retro (30 min, video call)
└── Jumat 16:00 — Demo (optional, 15 min, show & tell)

Monthly:
├── Minggu 1: Monthly review & OKR (1 jam)
├── Minggu 3: Feature planning (1 jam)
├── Minggu 4: 1-on-1 (30 min × 4 = 2 jam)
└── Ad-hoc: Investor update (jika perlu)

Quarterly:
├── OKR setting (2 jam)
├── Strategy review (2 jam)
└── Team offsite/activity (half day)
```

### 8.3 Documentation

```
Tools:
├── Notion/Google Docs — Meeting notes, PRD, specs
├── GitHub — Code, issues, wiki
├── Google Drive — Contracts, legal docs, assets
├── Figma — Design files, prototypes
└── Loom — Video walkthroughs, async updates

Rules:
├── Setiap keputusan harus tertulis (di meeting notes)
├── Setiap fitur harus punya PRD sebelum development
├── Setiap deploy harus punya changelog
└── Setiap incident harus punya post-mortem
```

---

## 9. Monitoring & Alerting

### 9.1 Key Metrics to Monitor

```
Infrastructure:
├── Server uptime (target: 99.9%)
├── Response time p50/p95/p99 (target: <200ms/<500ms/<1s)
├── Error rate (target: <0.1%)
├── Database connection pool usage
└── Storage usage (Supabase)

Business:
├── DAU / MAU
├── Sign-up rate
├── Booking conversion rate
├── Revenue (daily, weekly, monthly)
├── Support ticket volume & resolution time
└── NPS score

Security:
├── Failed login attempts (spike = attack?)
├── KTP verification failure rate
├── API rate limit hits
└── Suspicious admin actions
```

### 9.2 Alert Rules

```yaml
# Monitoring config (conceptual)
alerts:
  - name: "High Error Rate"
    condition: error_rate > 1% for 5 min
    severity: P1
    notify: on-call (WhatsApp + phone call)
    
  - name: "Slow Response"
    condition: p95_latency > 2s for 10 min
    severity: P2
    notify: on-call (WhatsApp)
    
  - name: "Payment Failure Spike"
    condition: payment_failure_rate > 5% for 5 min
    severity: P0
    notify: all-team (WhatsApp group + phone)
    
  - name: "Database Connection Exhausted"
    condition: db_connections > 80% pool
    severity: P1
    notify: on-call (WhatsApp + phone call)
    
  - name: "Storage Near Limit"
    condition: storage_usage > 80%
    severity: P2
    notify: Dimas (WhatsApp)
    
  - name: "Supabase Down"
    condition: health_check fails 3 consecutive
    severity: P0
    notify: all-team (WhatsApp group + phone)
```

### 9.3 Status Page

```
Public status page: status.jejakawan.id

Components:
├── 🟢 App (PWA) — Operational
├── 🟢 API — Operational  
├── 🟢 Database — Operational
├── 🟢 Payment (Midtrans) — Operational
├── 🟢 Authentication — Operational
└── 🟢 Maps (Google) — Operational

Incident communication:
├── Investigating → Identified → Monitoring → Resolved
└── Update setiap 30 menit selama incident
```

---

## 10. Disaster Recovery

### 10.1 Backup Schedule

```
Database (Supabase PostgreSQL):
├── Automated daily backup (Supabase Pro plan)
├── Point-in-time recovery: 7 hari retention
├── Manual backup: sebelum setiap major migration
└── Test restore: setiap bulan (Minggu 3)

Storage (Supabase Storage — user uploads):
├── Cross-region replication (jika available)
├── Critical assets (KTP photos): encrypted backup
└── User content (trip photos): Supabase managed backup

Code (GitHub):
├── All code in Git → inherently backed up
├── Protected branches: main, release/*
└── Tag setiap release: v1.0.0, v1.1.0, etc.

Config & Secrets:
├── Supabase Vault untuk secrets
├── .env files: NEVER in Git
├── Recovery: manual re-setup from documentation
└── Document all env vars in README
```

### 10.2 Recovery Procedures

```
Scenario: Complete Database Loss
├── Step 1: Create new Supabase project
├── Step 2: Restore from latest backup
├── Step 3: Update connection strings in app
├── Step 4: Verify data integrity (spot check)
├── Step 5: Update DNS if endpoint changed
└── RTO: 4 jam | RPO: 24 jam (daily backup)

Scenario: Supabase Full Outage
├── Step 1: Check status.supabase.com
├── Step 2: If ETA > 1 jam, activate maintenance mode
├── Step 3: Communicate to users via status page + social
├── Step 4: Monitor Supabase updates
└── Step 5: Restore service when Supabase recovers

Scenario: Code Repository Loss
├── Step 1: All developers have local clones
├── Step 2: Push to new remote (backup: GitLab mirror)
├── Step 3: Re-setup CI/CD
└── RTO: 2 jam | RPO: 0 (distributed)
```
