# Jejakawan

Platform travel PWA yang menghubungkan wisatawan dengan destinasi underrated, teman perjalanan, dan warga lokal.

## Fitur Utama

- **Personalized Recommendation** — Rekomendasi destinasi berbasis preferensi (rule-based)
- **Trip Matching** — Cari teman jalan dengan compatibility scoring
- **Gamifikasi** — XP, badge, level, misi eksplorasi ke destinasi underrated
- **Open Trip Marketplace** — Booking paket travel dari agen terdaftar
- **Safety & Trust** — Verifikasi KTP, rating, live location, moderasi

## Tech Stack

| Layer | Tool |
|---|---|
| Frontend | Next.js 14 (App Router) + TypeScript + TailwindCSS |
| Backend | Supabase (PostgreSQL + Auth + Storage + Realtime) |
| Maps | Google Maps Platform |
| Payment | Mayar (produksi); Midtrans direncanakan |
| Hosting | Vercel (frontend + API Routes) + Supabase (backend) |
| Analytics | PostHog |
| Monitoring | Sentry + UptimeRobot |

## Dokumen Teknis

Semua dokumentasi teknis ada di `docs/technical/`:

| # | Dokumen | Isi |
|---|---------|-----|
| 00 | [Index](docs/technical/00-Index.md) | Master index & prinsip desain |
| 01 | [System Architecture](docs/technical/01-System-Architecture.md) | Tech stack, arsitektur, scaling, security |
| 02 | [Database Schema](docs/technical/02-Database-Schema.md) | 27 tabel SQL, ERD, RLS, trigger |
| 03 | [User Segmentation](docs/technical/03-User-Segmentation.md) | 3 segmen, persona, onboarding |
| 04 | [Screen Map](docs/technical/04-Screen-Map.md) | ~25 screen, wireframe, states |
| 05 | [Feature Spec](docs/technical/05-Feature-Spec.md) | 13 fitur, sprint plan, analytics |
| 06 | [Business Process](docs/technical/06-Business-Process.md) | 6 proses bisnis, SLA |
| 07 | [API Design](docs/technical/07-API-Design.md) | ~109 endpoint, error codes |
| 08 | [Gamification System](docs/technical/08-Gamification-System.md) | XP, level, badge, misi |
| 09 | [Trip Matching Algorithm](docs/technical/09-Trip-Matching-Algorithm.md) | 6-factor scoring |
| 10 | [Safety & Trust](docs/technical/10-Safety-Trust-System.md) | 4-layer safety, UU PDP |
| 11 | [Monetization](docs/technical/11-Monetization-Implementation.md) | 5 revenue streams |
| 12 | [Infrastructure](docs/technical/12-Infrastructure-Deployment.md) | CI/CD, monitoring, DR |
| 13 | [Testing Strategy](docs/technical/13-Testing-Strategy.md) | Unit, integration, E2E, load |
| 14 | [Analytics](docs/technical/14-Analytics-Event-Tracking.md) | 50+ events, PostHog |
| 15 | [Notifications](docs/technical/15-Notification-System.md) | Push types, templates, FCM |
| 16 | [Content Strategy](docs/technical/16-Content-Strategy.md) | Seeding plan, UGC guidelines |
| 17 | [Legal & Compliance](docs/technical/17-Legal-Compliance.md) | UU PDP, privacy, contracts |
| 18 | [Launch Plan](docs/technical/18-Launch-Plan.md) | 8-week timeline, marketing |
| 19 | [Competitive Analysis](docs/technical/19-Competitive-Analysis.md) | 6 kompetitor, SWOT, moat |
| 20 | [Operations Runbook](docs/technical/20-Operations-Runbook.md) | On-call, incident playbook |

Juga tersedia: [Kritik & Saran Teknis](docs/Kritik_Teknis_DIGIVERSE.md) — review perhitungan BMC & rekomendasi.

## Project Structure

```
Jejakawan/
├── docs/
│   ├── technical/        # 21 dokumen teknis
│   ├── kritik/           # Review & kritik teknis
│   └── Kritik_Teknis_DIGIVERSE.md
├── .env.example          # Template environment variables
├── .gitignore
└── README.md
```

> **Implementasi kode** ada di repo terpisah: **[ShafnatFR/JejakawanApp](https://github.com/ShafnatFR/JejakawanApp)** (Next.js 14, App Router, API Routes, RLS-enforced).
> **Live:** https://jejakawanapp.vercel.app

## Quick Start

```bash
# 1. Clone repo implementasi
git clone https://github.com/ShafnatFR/JejakawanApp.git
cd JejakawanApp

# 2. Install dependencies
npm install

# 3. Setup environment
cp .env.example .env.local
# Isi Supabase URL/key, Mayar API key, dll

# 4. Run development
npm run dev

# 5. Build production
npm run build
```

## Kompetisi

- **Event:** DIGIVERSE FESTIVAL — Business Plan Competition
- **Tema:** Empowering Digital Innovation Through Business and Creative Design in Era 4.0
- **Subtema:** Tourism and Culture
- **Tim:** Emma, Chika, Dimas, Aqil

## License

Private — untuk keperluan kompetisi DIGIVERSE Festival.
