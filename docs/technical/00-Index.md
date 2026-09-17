# Jejakawan — Technical Design Documents

## Dokumen Teknis (21 file)

### Core Architecture (01-04)
| # | Dokumen | Deskripsi | Status |
|---|---------|-----------|--------|
| 00 | [[00-Index]] | Master index (file ini) | Done |
| 01 | [[01-System-Architecture]] | Tech stack, arsitektur, scaling, security, testing, logging, rate limiting, error handling | Done |
| 02 | [[02-Database-Schema]] | 27 tabel SQL, ERD, RLS, index, trigger, audit logs, soft delete, seeding, migration | Done |
| 03 | [[03-User-Segmentation]] | 3 segmen, persona, role/permission, onboarding, accessibility, retention strategy | Done |
| 04 | [[04-Screen-Map]] | ~25 screen MVP, navigasi, wireframe, empty/error/loading states, deep linking, notification center | Done |

### Features & Process (05-06)
| # | Dokumen | Deskripsi | Status |
|---|---------|-----------|--------|
| 05 | [[05-Feature-Spec]] | 13 fitur spesifikasi, dependency graph, sprint plan, analytics events, feature flags | Done |
| 06 | [[06-Business-Process]] | 6 proses bisnis, SLA, data retention, i18n strategy | Done |

### API & Algorithms (07-09)
| # | Dokumen | Deskripsi | Status |
|---|---------|-----------|--------|
| 07 | [[07-API-Design]] | ~109 endpoint, error code catalog, cursor pagination, request validation, rate limiting per endpoint | Done |
| 08 | [[08-Gamification-System]] | XP, 10 level, 20+ badge, misi, anti-abuse, admin dashboard, engagement analytics, A/B testing, seasonal events | Done |
| 09 | [[09-Trip-Matching-Algorithm]] | 6-factor scoring, filter, priority boost, cold start, non-populer matching, feedback loop, performance benchmark | Done |

### Safety & Business (10-12)
| # | Dokumen | Deskripsi | Status |
|---|---------|-----------|--------|
| 10 | [[10-Safety-Trust-System]] | 4-layer safety, trust score, moderation, incident response, UU PDP, emergency contact, appeal process, admin training | Done |
| 11 | [[11-Monetization-Implementation]] | 5 revenue streams, Midtrans, recurring, projection, tax (PPN/PPh), refund automation, financial dashboard | Done |
| 12 | [[12-Infrastructure-Deployment]] | CI/CD, Supabase setup, monitoring, backup, OWASP checklist, env management, capacity planning, disaster recovery | Done |

### Operations & Launch (13-20)
| # | Dokumen | Deskripsi | Status |
|---|---------|-----------|--------|
| 13 | [[13-Testing-Strategy]] | Unit (Vitest), integration (Supabase), E2E (Playwright), load test (k6), coverage targets, mocking | Done |
| 14 | [[14-Analytics-Event-Tracking]] | Event taxonomy, PostHog setup, funnel, cohort, dashboard, KPI targets | Done |
| 15 | [[15-Notification-System]] | Push types, templates, scheduling, preference center, FCM implementation, deep linking | Done |
| 16 | [[16-Content-Strategy]] | 500 destinasi seeding plan, UGC guidelines, tone of voice, moderation, SEO, social media | Done |
| 17 | [[17-Legal-Compliance]] | UU PDP compliance, privacy policy, KTP handling, PSE registration, breach notification, contracts | Done |
| 18 | [[18-Launch-Plan]] | Pre/post-launch timeline, beta testing, community seeding, marketing, influencer, success metrics | Done |
| 19 | [[19-Competitive-Analysis]] | Feature matrix vs 6 kompetitor, SWOT, positioning, moat analysis, competitive response playbook | Done |
| 20 | [[20-Operations-Runbook]] | On-call rotation, incident playbook, escalation, admin SOP, support templates, partner onboarding | Done |

---

## Ringkasan Teknis

```
Dokumen: 21 file
Tech Stack MVP:
  Frontend: React + Vite + TypeScript + TailwindCSS (PWA)
  Backend:  Supabase (PostgreSQL + Auth + Storage + Realtime + Edge Functions)
  Maps:     Google Maps JS API (free tier $200/bln)
  Payment:  Midtrans (2.5% per tx)
  Hosting:  Vercel (PWA) + Supabase (backend)
  Cost:     ~Rp15rb/bulan (free tiers)

Database: 27 tabel, PostGIS geolocation, RLS, audit logs, soft delete
API: ~109 endpoint + ~50 error codes + cursor pagination
Screen: ~25 screen MVP + empty/error/loading states
Gamifikasi: XP (6 source) + 10 level + 20+ badge + misi + seasonal events
Matching: 6-factor compatibility scoring + cold start + non-populer fallback
Safety: 4-layer + trust score + emergency button + appeal process
Monetisasi: 5 streams + PPN/PPh + refund automation
Testing: Vitest + Playwright + k6 + 80% coverage target
Analytics: 50+ PostHog events + funnel + cohort + KPI targets
Notifications: 8 types + preference center + quiet hours
Content: 500 destinasi seeding + UGC guidelines + tone of voice
Legal: UU PDP + privacy policy + PSE registration + contracts
Launch: 8-week plan + beta testing + community seeding
Competitive: 6 kompetitor analysis + SWOT + moat
Ops: On-call rotation + incident playbook + SOP templates
```

## Prinsip Desain
- **MVP First:** fitur inti dulu (rekomendasi, matching, gamifikasi, safety), sisanya fase 2+
- **Cost Efficient:** Supabase free tier + Vercel + managed service > custom infrastruktur
- **Mobile-First:** PWA dulu (React + Vite), native app kalau sudah validated
- **Rule-Based > AI:** recommendation engine pakai filtering biasa, bukan ML
- **Security by Design:** verifikasi & moderasi dari awal, OWASP Top 10 review
- **4-Layer Safety:** preventive → monitoring → response → education
- **Data-Driven:** PostHog analytics, A/B testing, funnel analysis dari hari 1

## Referensi
- [[../projects/DIGIVERSE-Festival]] — Ringkasan proyek & BMC
- Kritik_Teknis_DIGIVERSE.md (Downloads) — Review kritik & saran teknis
