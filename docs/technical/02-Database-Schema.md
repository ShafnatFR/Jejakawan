# 02 — Database Schema

## Overview

Database pakai PostgreSQL (via Supabase). Semua tabel pakai UUID primary key, RLS enabled, dan audit timestamps.

---

## Core Tables

### 1. users (extends auth.users)
```sql
CREATE TABLE public.user_profiles (
  id              UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  display_name    VARCHAR(100) NOT NULL,
  avatar_url      TEXT,
  bio             TEXT CHECK (char_length(bio) <= 500),
  current_mode    VARCHAR(10) DEFAULT 'tourist' CHECK (current_mode IN ('tourist','explorer')),
  preferred_interests TEXT[] DEFAULT '{}',
  budget_min      INT DEFAULT 0,
  budget_max      INT DEFAULT 10000000,
  transport_modes TEXT[] DEFAULT '{}',
  home_location   GEOGRAPHY(Point, 4326),
  current_location GEOGRAPHY(Point, 4326),
  ktp_verified    BOOLEAN DEFAULT FALSE,
  phone_verified  BOOLEAN DEFAULT FALSE,
  ktp_hash        TEXT,          -- hashed NIK, bukan plaintext
  rating_avg      DECIMAL(3,2) DEFAULT 0.00,
  rating_count    INT DEFAULT 0,
  xp_total        INT DEFAULT 0,
  level           INT DEFAULT 1,
  badge_count     INT DEFAULT 0,
  subscription    VARCHAR(20) DEFAULT 'free' CHECK (subscription IN ('free','weekly','monthly','exclusive')),
  subscription_expires_at TIMESTAMPTZ,
  is_active       BOOLEAN DEFAULT TRUE,
  is_banned       BOOLEAN DEFAULT FALSE,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 2. destinasi
```sql
CREATE TABLE public.destinations (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            VARCHAR(200) NOT NULL,
  slug            VARCHAR(200) UNIQUE NOT NULL,
  description     TEXT,
  category        TEXT[] DEFAULT '{}',  -- alam, kuliner, budaya, kota, adventure
  location        GEOGRAPHY(Point, 4326) NOT NULL,
  address         TEXT,
  province        VARCHAR(100),
  regency         VARCHAR(100),
  cover_image_url TEXT,
  image_urls      TEXT[] DEFAULT '{}',
  is_underrated   BOOLEAN DEFAULT FALSE,  -- flag underrated spot
  entry_fee_min   INT DEFAULT 0,
  entry_fee_max   INT DEFAULT 0,
  signal_strength VARCHAR(10) CHECK (signal_strength IN ('strong','moderate','weak','none')),
  best_season     TEXT[],  -- jan,feb,mar,... (bulan terbaik)
  rating_avg      DECIMAL(3,2) DEFAULT 0.00,
  rating_count    INT DEFAULT 0,
  visit_count     INT DEFAULT 0,  -- total kunjungan via app
  curated_by      UUID REFERENCES public.user_profiles(id),  -- admin yang kurasi
  is_verified     BOOLEAN DEFAULT FALSE,
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_destinations_location ON public.destinations USING GIST(location);
CREATE INDEX idx_destinations_category ON public.destinations USING GIN(category);
CREATE INDEX idx_destinations_underrated ON public.destinations(is_underrated) WHERE is_underrated = TRUE;
```

### 3. trip_requests (Trip Matching)
```sql
CREATE TABLE public.trip_requests (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  creator_id      UUID NOT NULL REFERENCES public.user_profiles(id),
  destination_id  UUID REFERENCES public.destinations(id),
  destination_name VARCHAR(200),  -- kalau destinasi belum ada di DB
  destination_custom GEOGRAPHY(Point, 4326),  -- kalau custom location
  date_from       DATE NOT NULL,
  date_to         DATE NOT NULL,
  duration_days   INT,
  budget_min      INT DEFAULT 0,
  budget_max      INT DEFAULT 10000000,
  transport_modes TEXT[] DEFAULT '{}',
  max_members     INT DEFAULT 4 CHECK (max_members BETWEEN 1 AND 10),
  current_members INT DEFAULT 1,
  notes           TEXT,
  status          VARCHAR(20) DEFAULT 'open' CHECK (status IN ('open','matched','in_progress','completed','cancelled')),
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 4. trip_matches
```sql
CREATE TABLE public.trip_matches (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  trip_request_id UUID NOT NULL REFERENCES public.trip_requests(id),
  inviter_id      UUID NOT NULL REFERENCES public.user_profiles(id),
  invitee_id      UUID NOT NULL REFERENCES public.user_profiles(id),
  compatibility   DECIMAL(5,2),  -- 0.00 - 100.00
  status          VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending','accepted','declined','expired')),
  responded_at    TIMESTAMPTZ,
  expires_at      TIMESTAMPTZ DEFAULT (NOW() + INTERVAL '48 hours'),
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 5. trip_groups (grup setelah match)
```sql
CREATE TABLE public.trip_groups (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  trip_request_id UUID NOT NULL REFERENCES public.trip_requests(id),
  name            VARCHAR(200),
  status          VARCHAR(20) DEFAULT 'planning' CHECK (status IN ('planning','active','completed','cancelled')),
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE public.trip_group_members (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  group_id        UUID NOT NULL REFERENCES public.trip_groups(id) ON DELETE CASCADE,
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  role            VARCHAR(10) DEFAULT 'member' CHECK (role IN ('host','member')),
  joined_at       TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(group_id, user_id)
);
```

---

## Marketplace Tables

### 6. agency_profiles
```sql
CREATE TABLE public.agency_profiles (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL UNIQUE REFERENCES public.user_profiles(id),
  company_name    VARCHAR(200) NOT NULL,
  description     TEXT,
  logo_url        TEXT,
  npwp            TEXT,
  is_verified     BOOLEAN DEFAULT FALSE,
  rating_avg      DECIMAL(3,2) DEFAULT 0.00,
  rating_count    INT DEFAULT 0,
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 7. open_trip_packages
```sql
CREATE TABLE public.open_trip_packages (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agency_id       UUID NOT NULL REFERENCES public.agency_profiles(id),
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  destination_id  UUID REFERENCES public.destinations(id),
  destination_name VARCHAR(200),
  itinerary       JSONB,  -- structured itinerary per day
  price_per_person INT NOT NULL,
  max_participants INT NOT NULL,
  current_participants INT DEFAULT 0,
  date_from       DATE NOT NULL,
  date_to         DATE NOT NULL,
  includes        TEXT[],  -- apa yang termasuk
  excludes        TEXT[],  -- apa yang tidak termasuk
  cover_image_url TEXT,
  image_urls      TEXT[] DEFAULT '{}',
  rating_avg      DECIMAL(3,2) DEFAULT 0.00,
  rating_count    INT DEFAULT 0,
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 8. bookings
```sql
CREATE TABLE public.bookings (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  package_id      UUID NOT NULL REFERENCES public.open_trip_packages(id),
  num_participants INT NOT NULL DEFAULT 1,
  total_price     INT NOT NULL,
  commission      INT NOT NULL,  -- 10% dari total_price
  payment_status  VARCHAR(20) DEFAULT 'pending' CHECK (payment_status IN ('pending','paid','refunded','failed')),
  payment_method  VARCHAR(50),
  payment_id      VARCHAR(200),  -- Midtrans order_id
  booked_at       TIMESTAMPTZ DEFAULT NOW(),
  paid_at         TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Gamifikasi Tables

### 9. missions
```sql
CREATE TABLE public.missions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  target_count    INT NOT NULL,  -- berapa spot yang harus dikunjungi
  xp_reward       INT NOT NULL,
  badge_id        UUID,  -- badge yang didapat kalau selesai
  deadline_days   INT DEFAULT 30,  -- hari setelah klaim
  region          VARCHAR(100),  -- filter region (e.g., "Jawa Barat")
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 10. user_missions
```sql
CREATE TABLE public.user_missions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  mission_id      UUID NOT NULL REFERENCES public.missions(id),
  progress        INT DEFAULT 0,
  status          VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active','completed','expired','abandoned')),
  claimed_at      TIMESTAMPTZ DEFAULT NOW(),
  deadline_at     TIMESTAMPTZ,  -- claimed_at + deadline_days
  completed_at    TIMESTAMPTZ,
  UNIQUE(user_id, mission_id)
);
```

### 11. mission_submissions
```sql
CREATE TABLE public.mission_submissions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_mission_id UUID NOT NULL REFERENCES public.user_missions(id),
  destination_id  UUID NOT NULL REFERENCES public.destinations(id),
  photo_url       TEXT NOT NULL,
  caption         TEXT,
  location        GEOGRAPHY(Point, 4326),  -- dari EXIF
  taken_at        TIMESTAMPTZ,  -- dari EXIF
  status          VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending','approved','rejected')),
  reviewed_by     UUID REFERENCES public.user_profiles(id),
  reviewed_at     TIMESTAMPTZ,
  reject_reason   TEXT,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 12. xp_logs
```sql
CREATE TABLE public.xp_logs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  amount          INT NOT NULL,  -- positif = earn, negatif = redeem
  source          VARCHAR(50) NOT NULL,  -- 'mission_spot','mission_complete','trip_complete','review','redemption'
  reference_id    UUID,  -- misi, trip, dll
  multiplier      DECIMAL(3,2) DEFAULT 1.00,  -- subscription multiplier
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 13. badges
```sql
CREATE TABLE public.badges (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            VARCHAR(100) NOT NULL,
  description     TEXT,
  icon_url        TEXT,
  category        VARCHAR(50),  -- explorer, social, achievement
  requirement     TEXT,  -- deskripsi cara dapat
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE public.user_badges (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  badge_id        UUID NOT NULL REFERENCES public.badges(id),
  earned_at       TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, badge_id)
);
```

---

## Communication Tables

### 14. chat_rooms
```sql
CREATE TABLE public.chat_rooms (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  trip_group_id   UUID REFERENCES public.trip_groups(id),
  type            VARCHAR(20) DEFAULT 'group' CHECK (type IN ('group','direct')),
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 15. chat_messages
```sql
CREATE TABLE public.chat_messages (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  room_id         UUID NOT NULL REFERENCES public.chat_rooms(id) ON DELETE CASCADE,
  sender_id       UUID NOT NULL REFERENCES public.user_profiles(id),
  message_type    VARCHAR(20) DEFAULT 'text' CHECK (message_type IN ('text','image','location','system')),
  content         TEXT NOT NULL,
  metadata        JSONB,  -- untuk image url, location coords, dll
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_chat_messages_room ON public.chat_messages(room_id, created_at DESC);
```

---

## Safety & Trust Tables

### 16. ratings
```sql
CREATE TABLE public.ratings (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  rater_id        UUID NOT NULL REFERENCES public.user_profiles(id),
  ratee_id        UUID REFERENCES public.user_profiles(id),
  destination_id  UUID REFERENCES public.destinations(id),
  package_id      UUID REFERENCES public.open_trip_packages(id),
  trip_group_id   UUID REFERENCES public.trip_groups(id),
  score           INT NOT NULL CHECK (score BETWEEN 1 AND 5),
  review          TEXT,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  -- ratee_id ATAU destination_id ATAU package_id harus terisi
  CHECK (
    (ratee_id IS NOT NULL)::INT + 
    (destination_id IS NOT NULL)::INT + 
    (package_id IS NOT NULL)::INT = 1
  )
);
```

### 17. reports
```sql
CREATE TABLE public.reports (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id     UUID NOT NULL REFERENCES public.user_profiles(id),
  reported_user   UUID REFERENCES public.user_profiles(id),
  reported_content UUID,  -- bisa message, submission, dll
  reason          VARCHAR(50) NOT NULL,  -- fraud, inappropriate, dangerous, fake, other
  details         TEXT,
  evidence_urls   TEXT[] DEFAULT '{}',
  status          VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending','reviewing','resolved','dismissed')),
  reviewed_by     UUID REFERENCES public.user_profiles(id),
  resolution      TEXT,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  resolved_at     TIMESTAMPTZ
);
```

### 18. live_location_shares
```sql
CREATE TABLE public.live_location_shares (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  trip_group_id   UUID REFERENCES public.trip_groups(id),
  emergency_contact_phone VARCHAR(20) NOT NULL,
  share_token     VARCHAR(64) UNIQUE NOT NULL,  -- token unik untuk link publik
  current_location GEOGRAPHY(Point, 4326),
  last_updated    TIMESTAMPTZ,
  expires_at      TIMESTAMPTZ NOT NULL,
  is_active       BOOLEAN DEFAULT TRUE,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Subscription & Payment Tables

### 19. subscriptions
```sql
CREATE TABLE public.subscriptions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  plan            VARCHAR(20) NOT NULL CHECK (plan IN ('weekly','monthly','exclusive')),
  price           INT NOT NULL,
  starts_at       TIMESTAMPTZ NOT NULL,
  expires_at      TIMESTAMPTZ NOT NULL,
  auto_renew      BOOLEAN DEFAULT TRUE,
  payment_id      VARCHAR(200),
  status          VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active','expired','cancelled')),
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 20. transactions
```sql
CREATE TABLE public.transactions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  type            VARCHAR(30) NOT NULL,  -- 'subscription','booking','donation'
  reference_id    UUID,  -- subscription id / booking id
  amount          INT NOT NULL,
  commission      INT DEFAULT 0,
  net_amount      INT NOT NULL,  -- amount - commission
  payment_method  VARCHAR(50),
  payment_id      VARCHAR(200),  -- Midtrans order_id
  status          VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending','completed','refunded','failed')),
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  completed_at    TIMESTAMPTZ
);
```

---

## ERD Summary (Relasi Utama)

```
auth.users 1──1 user_profiles
user_profiles 1──N trip_requests
user_profiles 1──N bookings
user_profiles 1──N user_missions
user_profiles 1──N ratings (as rater)
user_profiles 1──N user_badges
user_profiles 1──N xp_logs
user_profiles 1──N subscriptions
user_profiles 1──N reports (as reporter)

destinations 1──N trip_requests
destinations 1──N mission_submissions
destinations 1──N ratings

trip_requests 1──N trip_matches
trip_requests 1──1 trip_groups
trip_groups 1──N trip_group_members
trip_groups 1──1 chat_rooms

missions 1──N user_missions
user_missions 1──N mission_submissions

agency_profiles 1──N open_trip_packages
open_trip_packages 1──N bookings

chat_rooms 1──N chat_messages

badges 1──N user_badges
```

---

## RLS Policies (Ringkasan)

```sql
-- User hanya bisa baca/edit profil sendiri
CREATE POLICY "Users can read own profile" ON user_profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update own profile" ON user_profiles
  FOR UPDATE USING (auth.uid() = id);

-- User bisa baca profil publik user lain
CREATE POLICY "Public profiles are readable" ON user_profiles
  FOR SELECT USING (TRUE);  -- tapi kolom sensitif di-protect di app layer

-- Destinasi bisa dibaca semua orang
CREATE POLICY "Destinations are public" ON destinations
  FOR SELECT USING (TRUE);

-- Trip request hanya bisa diubah oleh creator
CREATE POLICY "Users can manage own trips" ON trip_requests
  FOR ALL USING (auth.uid() = creator_id);

-- Chat messages hanya bisa dibaca oleh anggota room
CREATE POLICY "Room members can read messages" ON chat_messages
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM trip_group_members tgm
      JOIN chat_rooms cr ON cr.trip_group_id = tgm.group_id
      WHERE cr.id = chat_messages.room_id AND tgm.user_id = auth.uid()
    )
  );

-- Reports hanya bisa dibaca oleh reporter + admin
CREATE POLICY "Reporter can read own reports" ON reports
  FOR SELECT USING (auth.uid() = reporter_id);
```

---

## Missing Tables

### 21. audit_logs
```sql
CREATE TABLE public.audit_logs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  actor_id        UUID REFERENCES public.user_profiles(id),
  action          VARCHAR(50) NOT NULL,  -- 'create','update','delete','ban','suspend','approve','reject'
  table_name      VARCHAR(100) NOT NULL,
  record_id       UUID NOT NULL,
  old_value       JSONB,
  new_value       JSONB,
  ip_address      INET,
  user_agent      TEXT,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_actor ON public.audit_logs(actor_id, created_at DESC);
CREATE INDEX idx_audit_logs_table ON public.audit_logs(table_name, record_id);
```

### 22. notifications
```sql
CREATE TABLE public.notifications (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  type            VARCHAR(50) NOT NULL,  -- 'match_invitation','mission_progress','subscription_reminder','report_update','trip_reminder','xp_earned','badge_earned','payment_success'
  title           VARCHAR(200) NOT NULL,
  body            TEXT,
  data            JSONB,  -- contextual data (trip_id, mission_id, etc.)
  read_at         TIMESTAMPTZ,
  sent_push       BOOLEAN DEFAULT FALSE,
  sent_at         TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_notifications_user ON public.notifications(user_id, created_at DESC);
CREATE INDEX idx_notifications_unread ON public.notifications(user_id) WHERE read_at IS NULL;
```

### 23. user_favorites
```sql
CREATE TABLE public.user_favorites (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  destination_id  UUID NOT NULL REFERENCES public.destinations(id),
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(user_id, destination_id)
);
```

### 24. media_uploads
```sql
CREATE TABLE public.media_uploads (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.user_profiles(id),
  file_url        TEXT NOT NULL,
  file_type       VARCHAR(20) NOT NULL,  -- 'image/jpeg','image/png','video/mp4'
  file_size       INT NOT NULL,  -- bytes
  purpose         VARCHAR(30) NOT NULL,  -- 'profile_avatar','ugc_photo','ugc_video','ktp','mission_proof','chat_image'
  metadata        JSONB,  -- EXIF, dimensions, duration
  status          VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active','deleted','flagged')),
  created_at      TIMESTAMPTZ DEFAULT NOW()
);
```

### 25. user_searches
```sql
CREATE TABLE public.user_searches (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES public.user_profiles(id),  -- nullable untuk anonymous
  query           TEXT,
  filters         JSONB,  -- {category, budget_range, location, radius}
  result_count    INT,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_searches_user ON public.user_searches(user_id, created_at DESC);
```

---

## Soft Delete Pattern

Standardisasi di semua tabel utama:
```sql
-- Tambahkan kolom ini ke tabel yang perlu soft delete:
ALTER TABLE {table_name} ADD COLUMN deleted_at TIMESTAMPTZ DEFAULT NULL;

-- Semua query filter:
WHERE deleted_at IS NULL

-- Restore: UPDATE ... SET deleted_at = NULL
-- Hard delete: cron job hapus data dengan deleted_at > 90 hari
```

Tabel yang butuh soft delete:
- user_profiles, destinations, trip_requests, open_trip_packages, agency_profiles
- Tabel transaksi (bookings, transactions) TIDAK boleh soft delete (audit trail)

---

## Composite Index Strategy

```sql
-- Trip matching query optimization
CREATE INDEX idx_trip_matching ON trip_requests(status, destination_id, date_from, date_to)
  WHERE status = 'open';

-- Recommendation query optimization
CREATE INDEX idx_destinations_recommend ON destinations(is_active, is_underrated, rating_avg DESC)
  WHERE is_active = TRUE;

-- Chat messages (sudah ada, tapi tambahan)
CREATE INDEX idx_chat_room_time ON chat_messages(room_id, created_at DESC);

-- Booking lookups
CREATE INDEX idx_bookings_user_status ON bookings(user_id, payment_status, created_at DESC);

-- Mission active lookups
CREATE INDEX idx_missions_active ON user_missions(user_id, status)
  WHERE status = 'active';

-- Notification unread count (fast)
CREATE INDEX idx_notif_unread ON notifications(user_id)
  WHERE read_at IS NULL AND sent_push = FALSE;
```

---

## Data Seeding Strategy

```
Destinasi awal (target: 500 destinasi di 5 provinsi pilot):
├── Jawa Barat: 150 destinasi (Bandung, Bogor, Sukabumi, Garut)
├── Jawa Tengah: 100 destinasi (Solo, Semarang, Dieng)
├── Jawa Timur: 100 destinasi (Malang, Banyuwangi, Bromo)
├── Yogyakarta: 75 destinasi (kota + kabupaten)
└── Bali: 75 destinasi (selain Kuta/Seminyak — underrated)

Sumber data:
├── Kemenparekraf API (kalau tersedia)
├── OpenStreetMap / Wikidata (destinasi + koordinat)
├── Manual curation oleh tim (foto + deskripsi + kategori)
└── Web scraping dari travel blog (dengan credit)

Seed script:
├── supabase/seed.sql: data destinasi, badge, misi template
├── supabase/seed-dev.sql: dummy users, trips, submissions (untuk testing)
└── Jalankan: supabase db seed
```

---

## Migration Strategy

```
Development workflow:
├── Perubahan schema: edit file di supabase/migrations/
├── Generate migration: supabase db diff --use-migra
├── Test lokal: supabase db reset (re-run semua migration)
├── Push ke staging: supabase db push --db-url STAGING_URL
├── Test di staging → verify
├── Push ke production: supabase db push --db-url PRODUCTION_URL
└── Rollback: manual SQL undo (catatan di migration file)

Naming convention:
├── 20261115120000_create_users_table.sql
├── 20261116090000_add_trust_score_to_users.sql
└── Format: YYYYMMDDHHMMSS_description.sql

Breaking changes policy:
├── Column rename: buat kolom baru → copy data → drop kolom lama (3 step)
├── Column drop: soft delete (rename ke _deprecated) → drop setelah 30 hari
├── Table rename: create new → copy → update app → drop old
└── Never: DROP TABLE tanpa backup confirmation
```
