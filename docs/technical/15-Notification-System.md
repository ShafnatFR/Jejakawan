# 15. Notification System — Jejakawan

## Overview

Dokumen ini mendefinisikan sistem notifikasi Jejakawan: tipe notifikasi, template, scheduling rules, preference center, deep linking, FCM implementation, dan in-app notification center.

---

## 1. Notification Architecture

```
┌─────────────────────────────────────────────────┐
│                 Jejakawan App                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐  │
│  │ In-App   │  │ Push     │  │ Preference   │  │
│  │ Center   │  │ Handler  │  │ Center       │  │
│  └────┬─────┘  └────┬─────┘  └──────┬───────┘  │
│       │              │               │           │
└───────┼──────────────┼───────────────┼───────────┘
        │              │               │
   ┌────▼──────────────▼───────────────▼────┐
   │        Notification Service            │
   │  ┌──────────┐  ┌───────────────────┐   │
   │  │ Scheduler│  │ Preference Engine │   │
   │  │ (queue)  │  │ (filter + rate)   │   │
   │  └────┬─────┘  └───────┬───────────┘   │
   │       │                 │               │
   │  ┌────▼─────────────────▼───────────┐   │
   │  │      Dispatch Router             │   │
   │  └──┬────────┬────────┬─────────┬───┘   │
   └─────┼────────┼────────┼─────────┼───────┘
         │        │        │         │
    ┌────▼──┐ ┌───▼──┐ ┌───▼───┐ ┌───▼───┐
    │  FCM  │ │Email │ │In-App │ │WA API │
    │ Push  │ │(SMTP)│ │(Real- │ │(opt.) │
    │       │ │      │ │ time) │ │       │
    └───────┘ └──────┘ └───────┘ └───────┘
         │        │        │         │
    ┌────▼──┐ ┌───▼──┐ ┌───▼───┐ ┌───▼───┐
    │Device │ │Inbox │ │Bell   │ │Chat   │
    └───────┘ └──────┘ └───────┘ └───────┘
```

### 1.1 Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Push Delivery | Firebase Cloud Messaging (FCM) | Cross-platform push notifications |
| In-App | Supabase Realtime | Real-time notification feed |
| Email | Resend / Nodemailer SMTP | Transactional email fallback |
| Queue | Supabase Edge Functions + pg_cron | Scheduled & delayed notifications |
| Storage | Supabase PostgreSQL | Notification log & user preferences |
| WhatsApp | WhatsApp Business API (optional) | High-priority notifications |

---

## 2. Notification Types

### 2.1 Complete Type Registry

| Type ID | Category | Priority | Channels | Description |
|---------|----------|----------|----------|-------------|
| `match_invitation` | Social | High | Push + In-App | Undangan trip dari user lain |
| `match_accepted` | Social | High | Push + In-App | Match diterima |
| `match_declined` | Social | Medium | In-App | Match ditolak |
| `match_new_suggestion` | Social | Medium | Push + In-App | Saran match baru tersedia |
| `booking_confirmed` | Transaction | Critical | Push + Email + In-App | Booking terkonfirmasi |
| `booking_reminder` | Transaction | High | Push + In-App | Pengingat H-1 booking |
| `booking_cancelled` | Transaction | High | Push + Email + In-App | Booking dibatalkan |
| `payment_success` | Transaction | Critical | Push + In-App | Pembayaran berhasil |
| `payment_failed` | Transaction | Critical | Push + Email + In-App | Pembayaran gagal |
| `payment_reminder` | Transaction | High | Push + In-App | Pengingat bayar (pending) |
| `payment_refund` | Transaction | High | Push + Email + In-App | Refund diproses/selesai |
| `subscription_welcome` | Subscription | Medium | Push + Email | Selamat bergabung plan premium |
| `subscription_expiring` | Subscription | High | Push + Email + In-App | Langganan hampir habis |
| `subscription_expired` | Subscription | High | Push + Email + In-App | Langganan sudah habis |
| `subscription_renewed` | Subscription | Medium | Push + In-App | Perpanjangan berhasil |
| `subscription_offer` | Marketing | Low | Push + In-App | Promo/langganan diskon |
| `mission_new` | Gamification | Medium | Push + In-App | Mission baru tersedia |
| `mission_completed` | Gamification | High | Push + In-App | Mission selesai + reward |
| `mission_expiring` | Gamification | Medium | Push + In-App | Mission hampir expired |
| `badge_earned` | Gamification | High | Push + In-App | Badge baru didapat |
| `level_up` | Gamification | High | Push + In-App | Level naik |
| `streak_reminder` | Gamification | Medium | Push + In-App | Jaga streak harian |
| `streak_broken` | Gamification | Low | In-App | Streak putus |
| `leaderboard_change` | Gamification | Low | In-App | Posisi leaderboard berubah |
| `review_received` | Social | Medium | Push + In-App | Review baru di destinasi mitra |
| `review_reply` | Social | Medium | Push + In-App | Balasan review |
| `follower_new` | Social | Low | In-App | Follower baru |
| `post_liked` | Social | Low | In-App | Post dilike |
| `post_commented` | Social | Medium | Push + In-App | Post dikomentari |
| `safety_alert` | Safety | Critical | Push + WA | Peringatan keamanan destinasi |
| `weather_alert` | Safety | High | Push + In-App | Cuaca ekstrem di destinasi |
| `destination_update` | Content | Low | In-App | Update info destinasi favorit |
| `referral_reward` | Marketing | Medium | Push + In-App | Reward referral diterima |
| `welcome` | Onboarding | Medium | Push + Email | Welcome setelah registrasi |
| `onboarding_nudge` | Onboarding | Low | Push + In-App | Ajakan lengkapi profil |

---

## 3. Notification Templates

### 3.1 Template Structure

```typescript
interface NotificationTemplate {
  id: string;
  type: string;
  title: string;         // max 50 chars
  body: string;          // max 150 chars, supports {{variable}}
  icon: string;          // emoji or icon URL
  image?: string;        // large image (for rich notification)
  action_url: string;    // deep link
  action_label: string;  // CTA button text
  sound?: 'default' | 'gentle' | 'urgent';
  category: string;
  ttl_hours: number;     // time-to-live, expired notif auto-hidden
}
```

### 3.2 Core Templates

```typescript
// src/lib/notifications/templates.ts

export const TEMPLATES: Record<string, NotificationTemplate> = {
  // ─── MATCH ───
  match_invitation: {
    id: 'match_invitation',
    type: 'match_invitation',
    title: '🤝 Undangan Trip Baru!',
    body: '{{sender_name}} mengajak kamu trip ke {{destination_name}} tanggal {{trip_date}}',
    icon: '🤝',
    action_url: '/match/{{match_id}}',
    action_label: 'Lihat Undangan',
    sound: 'default',
    category: 'social',
    ttl_hours: 72,
  },
  match_accepted: {
    id: 'match_accepted',
    type: 'match_accepted',
    title: '🎉 Match Diterima!',
    body: '{{matched_name}} menerima undangan trip ke {{destination_name}}. Saatnya rencanakan!',
    icon: '🎉',
    action_url: '/match/{{match_id}}/plan',
    action_label: 'Rencanakan Trip',
    sound: 'default',
    category: 'social',
    ttl_hours: 168,
  },
  match_declined: {
    id: 'match_declined',
    type: 'match_declined',
    title: '😔 Undangan Ditolak',
    body: '{{matched_name}} tidak bisa gabung trip ke {{destination_name}}. Cari teman lain?',
    icon: '😔',
    action_url: '/match?destination={{destination_id}}',
    action_label: 'Cari Match Lain',
    category: 'social',
    ttl_hours: 48,
  },
  match_new_suggestion: {
    id: 'match_new_suggestion',
    type: 'match_new_suggestion',
    title: '✨ Teman Trip Baru!',
    body: 'Ada {{count}} traveler yang cocok dengan kamu di {{destination_name}}',
    icon: '✨',
    action_url: '/match/suggestions',
    action_label: 'Lihat Match',
    category: 'social',
    ttl_hours: 48,
  },

  // ─── BOOKING ───
  booking_confirmed: {
    id: 'booking_confirmed',
    type: 'booking_confirmed',
    title: '✅ Booking Dikonfirmasi!',
    body: 'Trip ke {{destination_name}} tanggal {{trip_date}} sudah terkonfirmasi. Siap-siap ya!',
    icon: '✅',
    action_url: '/bookings/{{booking_id}}',
    action_label: 'Lihat Booking',
    sound: 'default',
    category: 'transaction',
    ttl_hours: 720, // 30 days
  },
  booking_reminder: {
    id: 'booking_reminder',
    type: 'booking_reminder',
    title: '⏰ Besok Trip-nya!',
    body: 'Jangan lupa, trip ke {{destination_name}} besok {{trip_date}}. Sudah siap?',
    icon: '⏰',
    action_url: '/bookings/{{booking_id}}',
    action_label: 'Detail Trip',
    sound: 'default',
    category: 'transaction',
    ttl_hours: 48,
  },
  booking_cancelled: {
    id: 'booking_cancelled',
    type: 'booking_cancelled',
    title: '❌ Booking Dibatalkan',
    body: 'Booking ke {{destination_name}} telah dibatalkan. {{refund_info}}',
    icon: '❌',
    action_url: '/bookings',
    action_label: 'Lihat Booking',
    category: 'transaction',
    ttl_hours: 168,
  },

  // ─── PAYMENT ───
  payment_success: {
    id: 'payment_success',
    type: 'payment_success',
    title: '💰 Pembayaran Berhasil!',
    body: 'Pembayaran Rp {{amount}} untuk trip ke {{destination_name}} berhasil. E-tiket sudah aktif!',
    icon: '💰',
    action_url: '/bookings/{{booking_id}}/e-ticket',
    action_label: 'Lihat E-Tiket',
    sound: 'default',
    category: 'transaction',
    ttl_hours: 720,
  },
  payment_failed: {
    id: 'payment_failed',
    type: 'payment_failed',
    title: '⚠️ Pembayaran Gagal',
    body: 'Pembayaran untuk trip ke {{destination_name}} gagal. Silakan coba lagi sebelum {{expiry_time}}.',
    icon: '⚠️',
    action_url: '/bookings/{{booking_id}}/retry-payment',
    action_label: 'Coba Bayar Lagi',
    sound: 'urgent',
    category: 'transaction',
    ttl_hours: 24,
  },
  payment_reminder: {
    id: 'payment_reminder',
    type: 'payment_reminder',
    title: '⏳ Jangan Lupa Bayar!',
    body: 'Booking ke {{destination_name}} menunggu pembayaran. Bayar sebelum {{expiry_time}}.',
    icon: '⏳',
    action_url: '/bookings/{{booking_id}}/pay',
    action_label: 'Bayar Sekarang',
    category: 'transaction',
    ttl_hours: 12,
  },

  // ─── SUBSCRIPTION ───
  subscription_welcome: {
    id: 'subscription_welcome',
    type: 'subscription_welcome',
    title: '👑 Selamat Datang, {{plan_name}}!',
    body: 'Kamu sudah upgrade ke {{plan_name}}. Nikmati fitur premium mulai sekarang!',
    icon: '👑',
    action_url: '/discover',
    action_label: 'Jelajahi Sekarang',
    category: 'subscription',
    ttl_hours: 168,
  },
  subscription_expiring: {
    id: 'subscription_expiring',
    type: 'subscription_expiring',
    title: '📅 Langganan Hampir Habis',
    body: 'Plan {{plan_name}} kamu habis dalam {{days_left}} hari. Perpanjang agar tetap bisa nikmati fitur premium.',
    icon: '📅',
    action_url: '/subscription/renew',
    action_label: 'Perpanjang',
    category: 'subscription',
    ttl_hours: 72,
  },
  subscription_expired: {
    id: 'subscription_expired',
    type: 'subscription_expired',
    title: '😢 Langganan Berakhir',
    body: 'Plan {{plan_name}} kamu sudah berakhir. Beberapa fitur premium sudah tidak aktif.',
    icon: '😢',
    action_url: '/subscription',
    action_label: 'Berlangganan Lagi',
    category: 'subscription',
    ttl_hours: 336, // 14 days
  },

  // ─── GAMIFICATION ───
  mission_new: {
    id: 'mission_new',
    type: 'mission_new',
    title: '🎯 Mission Baru!',
    body: '{{mission_name}} — Hadiah: {{xp_reward}} XP. Selesaikan sebelum {{deadline}}!',
    icon: '🎯',
    action_url: '/missions/{{mission_id}}',
    action_label: 'Lihat Mission',
    category: 'gamification',
    ttl_hours: 168,
  },
  mission_completed: {
    id: 'mission_completed',
    type: 'mission_completed',
    title: '🏆 Mission Selesai!',
    body: 'Kamu menyelesaikan "{{mission_name}}" dan mendapat {{xp_reward}} XP! Total XP: {{total_xp}}.',
    icon: '🏆',
    action_url: '/profile/badges',
    action_label: 'Lihat Badge',
    sound: 'default',
    category: 'gamification',
    ttl_hours: 168,
  },
  badge_earned: {
    id: 'badge_earned',
    type: 'badge_earned',
    title: '🏅 Badge Baru!',
    body: 'Kamu mendapat badge "{{badge_name}}" ({{rarity}})! Koleksi terus!',
    icon: '🏅',
    action_url: '/profile/badges/{{badge_id}}',
    action_label: 'Pamerin!',
    sound: 'default',
    category: 'gamification',
    ttl_hours: 336,
  },
  level_up: {
    id: 'level_up',
    type: 'level_up',
    title: '⬆️ Level Naik!',
    body: 'Selamat! Kamu naik ke Level {{new_level}}. XP berikutnya: {{xp_to_next}}.',
    icon: '⬆️',
    action_url: '/profile',
    action_label: 'Lihat Profil',
    sound: 'default',
    category: 'gamification',
    ttl_hours: 168,
  },
  streak_reminder: {
    id: 'streak_reminder',
    type: 'streak_reminder',
    title: '🔥 Jaga Streak!',
    body: 'Streak kamu {{streak_days}} hari! Check-in hari ini supaya nggak putus.',
    icon: '🔥',
    action_url: '/checkin',
    action_label: 'Check-in',
    category: 'gamification',
    ttl_hours: 12,
  },

  // ─── SAFETY ───
  safety_alert: {
    id: 'safety_alert',
    type: 'safety_alert',
    title: '🚨 Peringatan Keamanan',
    body: '{{alert_message}} — {{destination_name}}. {{action_advice}}',
    icon: '🚨',
    action_url: '/safety/{{alert_id}}',
    action_label: 'Info Selengkapnya',
    sound: 'urgent',
    category: 'safety',
    ttl_hours: 48,
  },
  weather_alert: {
    id: 'weather_alert',
    type: 'weather_alert',
    title: '🌧️ Peringatan Cuaca',
    body: '{{weather_type}} di {{destination_name}} pada {{alert_date}}. Perhatikan rencana trip kamu.',
    icon: '🌧️',
    action_url: '/bookings/{{booking_id}}',
    action_label: 'Cek Trip',
    sound: 'default',
    category: 'safety',
    ttl_hours: 24,
  },

  // ─── SOCIAL ───
  post_commented: {
    id: 'post_commented',
    type: 'post_commented',
    title: '💬 Komentar Baru',
    body: '{{commenter_name}} mengomentari post kamu: "{{comment_preview}}"',
    icon: '💬',
    action_url: '/post/{{post_id}}',
    action_label: 'Balas',
    category: 'social',
    ttl_hours: 72,
  },
  follower_new: {
    id: 'follower_new',
    type: 'follower_new',
    title: '👤 Follower Baru',
    body: '{{follower_name}} mulai mengikuti kamu!',
    icon: '👤',
    action_url: '/profile/{{follower_id}}',
    action_label: 'Lihat Profil',
    category: 'social',
    ttl_hours: 168,
  },

  // ─── ONBOARDING ───
  welcome: {
    id: 'welcome',
    type: 'welcome',
    title: '👋 Selamat Datang di Jejakawan!',
    body: 'Mulai petualanganmu! Temukan destinasi underrated & trip bareng teman baru.',
    icon: '👋',
    action_url: '/onboarding',
    action_label: 'Mulai',
    category: 'onboarding',
    ttl_hours: 168,
  },
  onboarding_nudge: {
    id: 'onboarding_nudge',
    type: 'onboarding_nudge',
    title: '📝 Lengkapi Profilmu!',
    body: 'Profil lengkap = match yang lebih cocok. Yuk isi preferensi kamu!',
    icon: '📝',
    action_url: '/profile/edit',
    action_label: 'Lengkapi Sekarang',
    category: 'onboarding',
    ttl_hours: 72,
  },
  referral_reward: {
    id: 'referral_reward',
    type: 'referral_reward',
    title: '🎁 Referral Berhasil!',
    body: 'Teman kamu {{referred_name}} sudah daftar! Kamu dapat {{reward_description}}.',
    icon: '🎁',
    action_url: '/rewards',
    action_label: 'Lihat Reward',
    category: 'marketing',
    ttl_hours: 336,
  },
};
```

---

## 4. Scheduling Rules

### 4.1 Quiet Hours

```typescript
// src/lib/notifications/scheduler.ts

interface QuietHoursConfig {
  enabled: boolean;
  start: string;  // HH:mm format in user's timezone
  end: string;
  timezone_field: string; // user table field
}

const QUIET_HOURS: QuietHoursConfig = {
  enabled: true,
  start: '22:00',
  end: '07:00',
  timezone_field: 'timezone',
};

// Exceptions: these types bypass quiet hours
const QUIET_HOURS_BYPASS = [
  'safety_alert',
  'weather_alert',
  'payment_failed',     // user needs to know ASAP
  'booking_confirmed',  // high-value confirmation
];

function shouldSendNow(userId: string, notificationType: string): boolean {
  // Bypass types always send immediately
  if (QUIET_HOURS_BYPASS.includes(notificationType)) return true;

  const userTimezone = getUserTimezone(userId);
  const now = new Date().toLocaleString('en-US', { timeZone: userTimezone });
  const hour = new Date(now).getHours();

  if (hour >= 22 || hour < 7) {
    // Queue for delivery at 07:00 user local time
    scheduleForLater(userId, notificationType, '07:00', userTimezone);
    return false;
  }
  return true;
}
```

### 4.2 Frequency Caps

```typescript
// src/lib/notifications/frequency-cap.ts

interface FrequencyCap {
  type: string;
  max_per_hour: number;
  max_per_day: number;
  cooldown_minutes: number; // minimum gap between same type
}

const FREQUENCY_CAPS: FrequencyCap[] = [
  // Social — jangan spam
  { type: 'post_liked',          max_per_hour: 5,  max_per_day: 20, cooldown_minutes: 2 },
  { type: 'post_commented',      max_per_hour: 3,  max_per_day: 15, cooldown_minutes: 5 },
  { type: 'follower_new',        max_per_hour: 3,  max_per_day: 10, cooldown_minutes: 5 },

  // Gamification — moderate
  { type: 'badge_earned',        max_per_hour: 2,  max_per_day: 5,  cooldown_minutes: 10 },
  { type: 'mission_completed',   max_per_hour: 3,  max_per_day: 10, cooldown_minutes: 5 },
  { type: 'level_up',            max_per_hour: 1,  max_per_day: 3,  cooldown_minutes: 30 },
  { type: 'streak_reminder',     max_per_hour: 1,  max_per_day: 1,  cooldown_minutes: 720 },

  // Transaction — no cap (always important)
  { type: 'booking_confirmed',   max_per_hour: 10, max_per_day: 50, cooldown_minutes: 0 },
  { type: 'payment_success',     max_per_hour: 10, max_per_day: 50, cooldown_minutes: 0 },
  { type: 'payment_failed',      max_per_hour: 5,  max_per_day: 10, cooldown_minutes: 5 },

  // Match — moderate
  { type: 'match_invitation',    max_per_hour: 3,  max_per_day: 10, cooldown_minutes: 10 },
  { type: 'match_new_suggestion',max_per_hour: 2,  max_per_day: 5,  cooldown_minutes: 30 },

  // Marketing — strict
  { type: 'subscription_offer',  max_per_hour: 1,  max_per_day: 1,  cooldown_minutes: 1440 },
  { type: 'referral_reward',     max_per_hour: 1,  max_per_day: 3,  cooldown_minutes: 60 },

  // Safety — no cap
  { type: 'safety_alert',        max_per_hour: 10, max_per_day: 20, cooldown_minutes: 0 },
];

async function checkFrequencyCap(userId: string, type: string): Promise<boolean> {
  const cap = FREQUENCY_CAPS.find(c => c.type === type);
  if (!cap) return true; // no cap defined

  const now = new Date();

  // Check hourly limit
  const hourlyCount = await db.notifications.count({
    where: {
      user_id: userId,
      type: type,
      created_at: { gte: new Date(now.getTime() - 3600000) },
    },
  });
  if (hourlyCount >= cap.max_per_hour) return false;

  // Check daily limit
  const dailyCount = await db.notifications.count({
    where: {
      user_id: userId,
      type: type,
      created_at: { gte: new Date(now.getTime() - 86400000) },
    },
  });
  if (dailyCount >= cap.max_per_day) return false;

  // Check cooldown
  if (cap.cooldown_minutes > 0) {
    const lastNotification = await db.notifications.findFirst({
      where: { user_id: userId, type: type },
      orderBy: { created_at: 'desc' },
    });
    if (lastNotification) {
      const elapsed = now.getTime() - lastNotification.created_at.getTime();
      if (elapsed < cap.cooldown_minutes * 60000) return false;
    }
  }

  return true;
}
```

### 4.3 Batching Strategy

```typescript
// Batch similar notifications to reduce noise

const BATCH_RULES = [
  {
    types: ['post_liked'],
    window_minutes: 30,
    batch_template: 'social_likes_batch',
    min_count: 2, // only batch if ≥ 2 likes
    format: (count: number, names: string[]) =>
      `${names[0]} dan ${count - 1} lainnya menyukai post kamu`,
  },
  {
    types: ['follower_new'],
    window_minutes: 60,
    batch_template: 'followers_batch',
    min_count: 3,
    format: (count: number, names: string[]) =>
      `${names[0]}, ${names[1]}, dan ${count - 2} lainnya mulai mengikuti kamu`,
  },
  {
    types: ['badge_earned', 'level_up'],
    window_minutes: 15,
    batch_template: 'gamification_batch',
    min_count: 2,
    format: (count: number, items: string[]) =>
      `Kamu mendapat ${items.join(' & ')}! Keren!`,
  },
];
```

---

## 5. Preference Center

### 5.1 User Preference Schema

```sql
-- Supabase migration
CREATE TABLE notification_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE UNIQUE,

  -- Master switch
  push_enabled BOOLEAN DEFAULT true,
  email_enabled BOOLEAN DEFAULT true,
  in_app_enabled BOOLEAN DEFAULT true,

  -- Category toggles
  category_social BOOLEAN DEFAULT true,
  category_transaction BOOLEAN DEFAULT true,
  category_subscription BOOLEAN DEFAULT true,
  category_gamification BOOLEAN DEFAULT true,
  category_safety BOOLEAN DEFAULT true,        -- always true, can't disable
  category_marketing BOOLEAN DEFAULT false,     -- opt-in only
  category_onboarding BOOLEAN DEFAULT true,
  category_content BOOLEAN DEFAULT true,

  -- Granular type overrides (JSONB for flexibility)
  type_overrides JSONB DEFAULT '{}'::jsonb,
  -- Example: {"post_liked": false, "streak_reminder": false}

  -- Quiet hours (user-customizable)
  quiet_hours_enabled BOOLEAN DEFAULT true,
  quiet_hours_start TIME DEFAULT '22:00',
  quiet_hours_end TIME DEFAULT '07:00',

  -- Frequency preference
  frequency_preference VARCHAR DEFAULT 'normal', -- 'minimal' | 'normal' | 'all'

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index for fast lookup
CREATE INDEX idx_notif_prefs_user ON notification_preferences(user_id);
```

### 5.2 Preference Check Logic

```typescript
// src/lib/notifications/preference-check.ts

interface NotificationRequest {
  userId: string;
  type: string;
  category: string;
  priority: 'low' | 'medium' | 'high' | 'critical';
  channel: 'push' | 'email' | 'in_app';
}

async function shouldDeliver(req: NotificationRequest): Promise<boolean> {
  // Safety notifications ALWAYS deliver (can't be disabled)
  if (req.category === 'safety') return true;

  // Critical transaction notifications always deliver
  if (req.priority === 'critical' && req.category === 'transaction') return true;

  const prefs = await getNotificationPreferences(req.userId);

  // Master switch check
  if (req.channel === 'push' && !prefs.push_enabled) return false;
  if (req.channel === 'email' && !prefs.email_enabled) return false;
  if (req.channel === 'in_app' && !prefs.in_app_enabled) return false;

  // Category toggle check
  const categoryKey = `category_${req.category}` as keyof typeof prefs;
  if (prefs[categoryKey] === false) return false;

  // Granular type override check
  if (prefs.type_overrides[req.type] === false) return false;

  // Frequency preference check
  if (prefs.frequency_preference === 'minimal') {
    const minimalAllowed = [
      'booking_confirmed', 'payment_success', 'payment_failed',
      'safety_alert', 'match_invitation',
    ];
    if (!minimalAllowed.includes(req.type)) return false;
  }

  // Frequency cap check
  if (!await checkFrequencyCap(req.userId, req.type)) return false;

  // Quiet hours check
  if (req.channel === 'push') {
    if (!shouldSendNow(req.userId, req.type)) return false;
  }

  return true;
}
```

### 5.3 Preference Center UI (Component Tree)

```
PreferenceCenter
├── MasterSwitches
│   ├── PushNotifications (toggle)
│   ├── EmailNotifications (toggle)
│   └── InAppNotifications (toggle)
├── CategorySettings
│   ├── Transaksi (toggle + detail)
│   │   ├── Booking confirmation
│   │   ├── Payment status
│   │   └── Refund updates
│   ├── Sosial (toggle + detail)
│   │   ├── Match invitations
│   │   ├── Comments & likes
│   │   └── New followers
│   ├── Gamifikasi (toggle + detail)
│   │   ├── Missions & badges
│   │   ├── Streak reminders
│   │   └── Leaderboard updates
│   ├── Langganan (toggle + detail)
│   │   ├── Renewal reminders
│   │   └── Offers & promos
│   └── Konten (toggle + detail)
│       ├── Destination updates
│       └── Blog & articles
├── QuietHours
│   ├── Enable quiet hours (toggle)
│   ├── Start time (time picker)
│   └── End time (time picker)
├── Frequency
│   └── Radio: Minimal | Normal | Semua
└── Save button
```

---

## 6. Deep Linking

### 6.1 Deep Link Schema

```typescript
// src/lib/notifications/deep-link.ts

const DEEP_LINKS: Record<string, (params: Record<string, string>) => string> = {
  match: (p) => `/match/${p.match_id}`,
  match_plan: (p) => `/match/${p.match_id}/plan`,
  match_suggestions: () => `/match/suggestions`,
  booking: (p) => `/bookings/${p.booking_id}`,
  booking_pay: (p) => `/bookings/${p.booking_id}/pay`,
  booking_retry: (p) => `/bookings/${p.booking_id}/retry-payment`,
  booking_eticket: (p) => `/bookings/${p.booking_id}/e-ticket`,
  subscription: () => `/subscription`,
  subscription_renew: () => `/subscription/renew`,
  mission: (p) => `/missions/${p.mission_id}`,
  badge: (p) => `/profile/badges/${p.badge_id}`,
  profile: () => `/profile`,
  profile_edit: () => `/profile/edit`,
  post: (p) => `/post/${p.post_id}`,
  user_profile: (p) => `/profile/${p.user_id}`,
  destination: (p) => `/destination/${p.destination_id}`,
  checkin: () => `/checkin`,
  rewards: () => `/rewards`,
  safety: (p) => `/safety/${p.alert_id}`,
  discover: () => `/discover`,
};
```

### 6.2 PWA Deep Link Handling

```typescript
// src/lib/notifications/push-handler.ts

// Service Worker push event handler
self.addEventListener('push', (event) => {
  const data = event.data?.json();
  if (!data) return;

  const template = TEMPLATES[data.type];
  if (!template) return;

  const options: NotificationOptions = {
    body: interpolate(template.body, data.variables),
    icon: template.icon,
    badge: '/icons/badge-96.png',
    image: data.image,
    tag: `${data.type}_${data.target_id}`, // deduplicate
    renotify: true,
    data: {
      url: interpolate(template.action_url, data.variables),
      type: data.type,
      notification_id: data.notification_id,
    },
    actions: [
      {
        action: 'open',
        title: template.action_label,
      },
      {
        action: 'dismiss',
        title: 'Tutup',
      },
    ],
    vibrate: template.sound === 'urgent' ? [200, 100, 200, 100, 200] : [100],
    requireInteraction: data.priority === 'critical',
    silent: false,
  };

  event.waitUntil(
    self.registration.showNotification(template.title, options)
  );
});

// Handle notification click
self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  if (event.action === 'dismiss') return;

  const url = event.notification.data.url;

  event.waitUntil(
    clients.matchAll({ type: 'window', includeUncontrolled: true }).then((windowClients) => {
      // Focus existing window if available
      for (const client of windowClients) {
        if (client.url.includes(self.location.origin) && 'focus' in client) {
          client.focus();
          client.navigate(url);
          return;
        }
      }
      // Open new window
      clients.openWindow(url);
    })
  );

  // Track click
  trackNotificationClick(event.notification.data.notification_id);
});
```

### 6.3 React Router Deep Link Handler

```typescript
// src/App.tsx (route setup)
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

// Handle incoming deep links from push notification
function DeepLinkHandler() {
  const navigate = useNavigate();

  useEffect(() => {
    // Handle URL params from notification click
    const params = new URLSearchParams(window.location.search);
    const deepLink = params.get('deeplink');
    if (deepLink) {
      navigate(deepLink, { replace: true });
    }

    // Handle PWA protocol handler
    if ('protocolHandlers' in navigator) {
      navigator.protocolHandlers?.registerProtocol(
        'web+jejakawan',
        (url) => {
          const path = url.pathname;
          navigate(path);
        }
      );
    }
  }, [navigate]);

  return null;
}
```

---

## 7. FCM Implementation

### 7.1 Firebase Setup

```typescript
// src/lib/notifications/fcm.ts
import { initializeApp } from 'firebase/app';
import { getMessaging, getToken, onMessage, MessagePayload } from 'firebase/messaging';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
const messaging = getMessaging(app);
```

### 7.2 Token Registration

```typescript
// src/lib/notifications/fcm.ts (continued)

const VAPID_KEY = process.env.NEXT_PUBLIC_FCM_VAPID_KEY;

export async function requestNotificationPermission(): Promise<string | null> {
  try {
    const permission = await Notification.requestPermission();
    if (permission !== 'granted') {
      console.log('Notification permission denied');
      return null;
    }

    const token = await getToken(messaging, {
      vapidKey: VAPID_KEY,
      serviceWorkerRegistration: await navigator.serviceWorker.ready,
    });

    if (token) {
      // Save token to Supabase
      await saveFCMToken(token);
      return token;
    }
    return null;
  } catch (err) {
    console.error('FCM token error:', err);
    return null;
  }
}

async function saveFCMToken(token: string) {
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return;

  // Upsert token (one user can have multiple devices)
  await supabase.from('fcm_tokens').upsert(
    {
      user_id: user.id,
      token: token,
      device_info: {
        platform: navigator.platform,
        userAgent: navigator.userAgent,
        isPWA: window.matchMedia('(display-mode: standalone)').matches,
      },
      last_active: new Date().toISOString(),
    },
    { onConflict: 'token' }
  );
}

// Refresh token periodically
export function setupTokenRefresh() {
  // FCM tokens can change; listen for refresh
  navigator.serviceWorker.addEventListener('message', (event) => {
    if (event.data?.type === 'TOKEN_REFRESHED') {
      saveFCMToken(event.data.token);
    }
  });
}
```

### 7.3 Foreground Message Handling

```typescript
// src/lib/notifications/foreground-handler.ts

export function setupForegroundMessages() {
  onMessage(messaging, (payload: MessagePayload) => {
    const { notification, data } = payload;

    // Show in-app notification (not system notification when app is open)
    showInAppNotification({
      id: data?.notification_id,
      type: data?.type,
      title: notification?.title ?? 'Notifikasi Baru',
      body: notification?.body ?? '',
      icon: data?.icon,
      actionUrl: data?.url,
      timestamp: new Date(),
    });

    // Update notification badge count
    updateNotificationBadge();

    // Play sound for high-priority
    if (data?.priority === 'critical' || data?.priority === 'high') {
      playNotificationSound(data?.type);
    }
  });
}
```

### 7.4 Server-Side Push Send

```typescript
// src/lib/notifications/send-push.ts (Supabase Edge Function)

import * as admin from 'firebase-admin';

// Initialize Firebase Admin
if (!admin.apps.length) {
  admin.initializeApp({
    credential: admin.credential.cert({
      projectId: Deno.env.get('FIREBASE_PROJECT_ID'),
      clientEmail: Deno.env.get('FIREBASE_CLIENT_EMAIL'),
      privateKey: Deno.env.get('FIREBASE_PRIVATE_KEY')?.replace(/\\n/g, '\n'),
    }),
  });
}

interface SendPushParams {
  userId: string;
  type: string;
  variables: Record<string, string>;
  priority?: string;
  image?: string;
}

export async function sendPush(params: SendPushParams) {
  // 1. Check user preferences
  const shouldDeliver = await checkDelivery({
    userId: params.userId,
    type: params.type,
    category: getTemplate(params.type).category,
    priority: params.priority ?? 'medium',
    channel: 'push',
  });
  if (!shouldDeliver) return { sent: false, reason: 'filtered' };

  // 2. Get user's FCM tokens
  const { data: tokens } = await supabaseAdmin
    .from('fcm_tokens')
    .select('token')
    .eq('user_id', params.userId)
    .gte('last_active', new Date(Date.now() - 30 * 86400000).toISOString()); // active in 30 days

  if (!tokens?.length) return { sent: false, reason: 'no_tokens' };

  // 3. Build notification
  const template = getTemplate(params.type);
  const message: admin.messaging.MulticastMessage = {
    tokens: tokens.map(t => t.token),
    notification: {
      title: template.title,
      body: interpolate(template.body, params.variables),
      imageUrl: params.image,
    },
    data: {
      type: params.type,
      url: interpolate(template.action_url, params.variables),
      notification_id: crypto.randomUUID(),
      priority: params.priority ?? 'medium',
      icon: template.icon,
    },
    android: {
      priority: params.priority === 'critical' ? 'high' : 'normal',
      notification: {
        channelId: getAndroidChannelId(template.category),
        sound: template.sound === 'urgent' ? 'urgent' : 'default',
        clickAction: 'OPEN_DEEP_LINK',
      },
      ttl: template.ttl_hours * 3600 * 1000,
    },
    apns: {
      payload: {
        aps: {
          alert: {
            title: template.title,
            body: interpolate(template.body, params.variables),
          },
          sound: template.sound === 'urgent' ? 'urgent.aiff' : 'default',
          badge: 1,
          'mutable-content': 1,
          'content-available': 1,
        },
      },
      headers: {
        'apns-priority': params.priority === 'critical' ? '10' : '5',
        'apns-expiration': String(Math.floor(Date.now() / 1000) + template.ttl_hours * 3600),
      },
    },
    webpush: {
      headers: {
        TTL: String(template.ttl_hours * 3600),
      },
      notification: {
        icon: '/icons/icon-192.png',
        badge: '/icons/badge-96.png',
        vibrate: params.priority === 'critical' ? [200, 100, 200] : [100],
        actions: [
          { action: 'open', title: template.action_label },
          { action: 'dismiss', title: 'Tutup' },
        ],
      },
      fcmOptions: {
        link: interpolate(template.action_url, params.variables),
      },
    },
  };

  // 4. Send
  const response = await admin.messaging().sendEachForMulticast(message);

  // 5. Log and handle failures
  const failedTokens: string[] = [];
  response.responses.forEach((resp, idx) => {
    if (!resp.success) {
      const errorCode = resp.error?.code;
      if (
        errorCode === 'messaging/invalid-registration-token' ||
        errorCode === 'messaging/registration-token-not-registered'
      ) {
        failedTokens.push(tokens[idx].token);
      }
    }
  });

  // Clean up invalid tokens
  if (failedTokens.length > 0) {
    await supabaseAdmin
      .from('fcm_tokens')
      .delete()
      .in('token', failedTokens);
  }

  // 6. Store notification in DB
  await supabaseAdmin.from('notifications').insert({
    user_id: params.userId,
    type: params.type,
    title: template.title,
    body: interpolate(template.body, params.variables),
    data: params.variables,
    action_url: interpolate(template.action_url, params.variables),
    read: false,
    created_at: new Date().toISOString(),
  });

  return {
    sent: true,
    success: response.successCount,
    failed: response.failureCount,
    cleanedTokens: failedTokens.length,
  };
}
```

---

## 8. In-App Notification Center

### 8.1 Database Schema

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  type VARCHAR NOT NULL,
  category VARCHAR NOT NULL,
  title VARCHAR NOT NULL,
  body TEXT NOT NULL,
  data JSONB DEFAULT '{}'::jsonb,
  action_url VARCHAR,
  image_url VARCHAR,
  read BOOLEAN DEFAULT false,
  read_at TIMESTAMPTZ,
  dismissed BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ
);

CREATE INDEX idx_notifications_user_created
  ON notifications(user_id, created_at DESC);
CREATE INDEX idx_notifications_user_unread
  ON notifications(user_id, read)
  WHERE read = false;

-- RLS: users can only see their own notifications
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can view own notifications"
  ON notifications FOR SELECT
  USING (auth.uid() = user_id);
CREATE POLICY "Users can update own notifications"
  ON notifications FOR UPDATE
  USING (auth.uid() = user_id);
```

### 8.2 Realtime Subscription

```typescript
// src/hooks/useNotifications.ts
import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabase';

export function useNotifications() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);

  useEffect(() => {
    // Load initial notifications
    loadNotifications();

    // Subscribe to realtime changes
    const channel = supabase
      .channel('notifications')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'notifications',
          filter: `user_id=eq.${getCurrentUserId()}`,
        },
        (payload) => {
          const newNotif = payload.new as Notification;
          setNotifications(prev => [newNotif, ...prev]);
          setUnreadCount(prev => prev + 1);
          // Play sound / show toast
          showToast(newNotif);
        }
      )
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'notifications',
          filter: `user_id=eq.${getCurrentUserId()}`,
        },
        (payload) => {
          const updated = payload.new as Notification;
          setNotifications(prev =>
            prev.map(n => n.id === updated.id ? updated : n)
          );
          if (updated.read) {
            setUnreadCount(prev => Math.max(0, prev - 1));
          }
        }
      )
      .subscribe();

    return () => {
      supabase.removeChannel(channel);
    };
  }, []);

  const markAsRead = async (id: string) => {
    await supabase
      .from('notifications')
      .update({ read: true, read_at: new Date().toISOString() })
      .eq('id', id);
  };

  const markAllAsRead = async () => {
    await supabase
      .from('notifications')
      .update({ read: true, read_at: new Date().toISOString() })
      .eq('user_id', getCurrentUserId())
      .eq('read', false);
  };

  const dismiss = async (id: string) => {
    await supabase
      .from('notifications')
      .update({ dismissed: true })
      .eq('id', id);
  };

  return { notifications, unreadCount, markAsRead, markAllAsRead, dismiss };
}
```

### 8.3 Notification Center Component

```tsx
// src/components/NotificationCenter.tsx

import { Bell, BellDot, Check, CheckCheck, X, ExternalLink } from 'lucide-react';
import { useNotifications } from '@/hooks/useNotifications';
import { formatDistanceToNow } from 'date-fns';
import { id } from 'date-fns/locale';
import { useNavigate } from 'react-router-dom';
import { motion, AnimatePresence } from 'framer-motion';

export function NotificationCenter() {
  const { notifications, unreadCount, markAsRead, markAllAsRead, dismiss } = useNotifications();
  const navigate = useNavigate();
  const [isOpen, setIsOpen] = useState(false);

  const handleClick = (notif: Notification) => {
    markAsRead(notif.id);
    if (notif.action_url) {
      navigate(notif.action_url);
      setIsOpen(false);
    }
  };

  const grouped = groupByDate(notifications);

  return (
    <Popover open={isOpen} onOpenChange={setIsOpen}>
      <PopoverTrigger asChild>
        <Button variant="ghost" size="icon" className="relative">
          {unreadCount > 0 ? <BellDot className="h-5 w-5" /> : <Bell className="h-5 w-5" />}
          {unreadCount > 0 && (
            <Badge className="absolute -top-1 -right-1 h-5 w-5 flex items-center justify-center text-xs">
              {unreadCount > 99 ? '99+' : unreadCount}
            </Badge>
          )}
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-96 max-h-[500px] p-0">
        <div className="flex items-center justify-between p-4 border-b">
          <h3 className="font-semibold">Notifikasi</h3>
          {unreadCount > 0 && (
            <Button variant="ghost" size="sm" onClick={markAllAsRead}>
              <CheckCheck className="h-4 w-4 mr-1" />
              Tandai semua dibaca
            </Button>
          )}
        </div>
        <ScrollArea className="max-h-[400px]">
          {Object.entries(grouped).map(([date, notifs]) => (
            <div key={date}>
              <div className="px-4 py-2 text-xs text-muted-foreground bg-muted/50">
                {date}
              </div>
              <AnimatePresence>
                {notifs.map((notif) => (
                  <motion.div
                    key={notif.id}
                    initial={{ opacity: 0, height: 0 }}
                    animate={{ opacity: 1, height: 'auto' }}
                    exit={{ opacity: 0, height: 0 }}
                    className={cn(
                      'flex items-start gap-3 p-4 border-b cursor-pointer hover:bg-accent/50 transition-colors',
                      !notif.read && 'bg-primary/5'
                    )}
                    onClick={() => handleClick(notif)}
                  >
                    <span className="text-2xl">{getCategoryIcon(notif.category)}</span>
                    <div className="flex-1 min-w-0">
                      <p className={cn('text-sm', !notif.read && 'font-semibold')}>
                        {notif.title}
                      </p>
                      <p className="text-sm text-muted-foreground line-clamp-2">
                        {notif.body}
                      </p>
                      <p className="text-xs text-muted-foreground mt-1">
                        {formatDistanceToNow(new Date(notif.created_at), {
                          addSuffix: true,
                          locale: id,
                        })}
                      </p>
                    </div>
                    <div className="flex gap-1">
                      {!notif.read && (
                        <div className="h-2 w-2 rounded-full bg-primary mt-2" />
                      )}
                      <Button
                        variant="ghost"
                        size="icon"
                        className="h-6 w-6"
                        onClick={(e) => {
                          e.stopPropagation();
                          dismiss(notif.id);
                        }}
                      >
                        <X className="h-3 w-3" />
                      </Button>
                    </div>
                  </motion.div>
                ))}
              </AnimatePresence>
            </div>
          ))}
          {notifications.length === 0 && (
            <div className="p-8 text-center text-muted-foreground">
              <Bell className="h-8 w-8 mx-auto mb-2 opacity-50" />
              <p>Belum ada notifikasi</p>
            </div>
          )}
        </ScrollArea>
      </PopoverContent>
    </Popover>
  );
}
```

---

## 9. Notification Database Tables

```sql
-- FCM tokens per device
CREATE TABLE fcm_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  token VARCHAR NOT NULL UNIQUE,
  device_info JSONB,
  last_active TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_fcm_tokens_user ON fcm_tokens(user_id);

-- Notification delivery log (for analytics & debugging)
CREATE TABLE notification_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID,
  type VARCHAR NOT NULL,
  channel VARCHAR NOT NULL, -- push | email | in_app | whatsapp
  status VARCHAR NOT NULL,  -- sent | failed | filtered | queued
  error_message TEXT,
  metadata JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_notif_log_user_type
  ON notification_log(user_id, type, created_at DESC);

-- Cleanup: auto-delete expired notifications
CREATE OR REPLACE FUNCTION cleanup_expired_notifications()
RETURNS void AS $$
BEGIN
  DELETE FROM notifications
  WHERE expires_at < NOW()
    AND read = true;
END;
$$ LANGUAGE plpgsql;

-- Run cleanup daily via pg_cron
SELECT cron.schedule(
  'cleanup-notifications',
  '0 3 * * *',  -- every day at 3 AM
  'SELECT cleanup_expired_notifications()'
);
```
