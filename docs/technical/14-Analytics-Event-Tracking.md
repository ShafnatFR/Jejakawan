# 14. Analytics & Event Tracking — Jejakawan

## Overview

Dokumen ini mendefinisikan strategi analytics untuk Jejakawan, mencakup event taxonomy, PostHog setup, funnel analysis, cohort analysis, dashboard definition, dan KPI targets per user segment.

---

## 1. Analytics Stack

| Tool | Purpose | Integration |
|------|---------|-------------|
| PostHog (self-hosted) | Product analytics, session replay, feature flags | JS SDK + API |
| Supabase Analytics | DB query performance, edge function logs | Built-in |
| Midtrans Dashboard | Payment conversion, failure analysis | Webhook + dashboard |
| Google Search Console | SEO performance (web version) | API integration |
| Custom Dashboard | Business metrics aggregation | Next.js admin panel |

---

## 2. PostHog Setup

### 2.1 Installation & Initialization

```typescript
// src/lib/analytics/posthog.ts
import posthog from 'posthog-js';

export function initPostHog() {
  posthog.init(
    import.meta.env.VITE_POSTHOG_KEY ?? 'phc_xxx',
    {
      api_host: import.meta.env.VITE_POSTHOG_HOST ?? 'https://analytics.jejakawan.id',
      capture_pageview: true,
      capture_pageleave: true,
      persistence: 'localStorage+cookie',
      autocapture: {
        dom_event_allowlist: ['click', 'submit', 'change'],
        url_allowlist: [/^https:\/\/app\.jejakawan\.id/],
        element_allowlist: ['button', 'a', 'input', 'select'],
        css_selector_allowlist: ['[data-ph-capture]'],
      },
      session_recording: {
        recordCrossOriginIframes: false,
        maskAllInputs: true, // privacy: mask all input fields
        maskTextSelector: '[data-private]',
      },
      // Respect user consent
      opt_out_capturing_by_default: true,
      loaded: (ph) => {
        if (localStorage.getItem('cookie_consent') === 'accepted') {
          ph.opt_in_capturing();
        }
      },
    }
  );
}

// User identification after login
export function identifyUser(user: { id: string; email: string; plan: string; segment: string }) {
  posthog.identify(user.id, {
    email: user.email,
    plan: user.plan,
    segment: user.segment, // tourist | explorer | mitra
    created_at: new Date().toISOString(),
  });
}

// Group analytics (for mitra/business accounts)
export function identifyGroup(groupId: string, properties: Record<string, any>) {
  posthog.group('company', groupId, properties);
}
```

### 2.2 Consent Flow

```typescript
// src/components/CookieConsent.tsx (hook)
import posthog from 'posthog-js';

export function useAnalyticsConsent() {
  const accept = () => {
    localStorage.setItem('cookie_consent', 'accepted');
    posthog.opt_in_capturing();
  };

  const decline = () => {
    localStorage.setItem('cookie_consent', 'declined');
    posthog.opt_out_capturing();
  };

  return { accept, decline };
}
```

---

## 3. Event Taxonomy

### 3.1 Naming Convention

```
Format: <object>_<action>
Contoh: destination_viewed, trip_match_requested, booking_completed

Properties naming: snake_case, all lowercase
Timestamp: ISO 8601, auto-attached by PostHog
```

### 3.2 Auth & Onboarding Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `user_registered` | Registrasi berhasil | `method` (email/google/apple), `referral_code` |
| `user_logged_in` | Login berhasil | `method`, `device_type` |
| `onboarding_started` | Mulai onboarding | `step` (1) |
| `onboarding_step_completed` | Setiap step selesai | `step_number`, `step_name`, `time_spent_sec` |
| `onboarding_completed` | Onboarding selesai | `total_time_sec`, `skipped_steps[]` |
| `onboarding_abandoned` | Keluar sebelum selesai | `last_step`, `time_spent_sec` |
| `profile_updated` | Edit profil | `fields_changed[]` |
| `preference_set` | Set preferensi travel | `travel_style`, `budget_range`, `interests[]` |

### 3.3 Discovery & Search Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `destination_viewed` | Buka detail destinasi | `destination_id`, `name`, `province`, `category`, `source` (search/recommendation/map/browse) |
| `destination_searched` | Pencarian | `query`, `filters{}`, `results_count`, `position_clicked` |
| `destination_filtered` | Apply filter | `filter_type`, `filter_value`, `results_count` |
| `destination_favorited` | Tambah ke favorit | `destination_id`, `name` |
| `destination_unfavorited` | Hapus dari favorit | `destination_id` |
| `destination_shared` | Share destinasi | `destination_id`, `platform` (wa/ig/link/copy) |
| `photo_gallery_opened` | Buka galeri foto | `destination_id`, `photo_count` |
| `photo_gallery_swiped` | Swipe foto | `destination_id`, `photo_index`, `direction` |
| `map_pin_clicked` | Klik pin di map | `destination_id`, `zoom_level` |
| `map_explored` | Interaksi map | `bounds`, `zoom_level`, `duration_sec` |
| `review_read` | Baca review | `destination_id`, `review_id`, `scroll_depth` |
| `review_submitted` | Tulis review | `destination_id`, `rating`, `has_photo`, `text_length` |

### 3.4 Trip Matching Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `match_started` | Mulai cari teman | `date`, `duration_days`, `budget_max`, `interests[]` |
| `match_results_viewed` | Hasil match muncul | `results_count`, `top_score`, `filter_applied` |
| `match_profile_viewed` | Lihat profil match | `matched_user_id`, `compatibility_score` |
| `match_invitation_sent` | Kirim undangan | `matched_user_id`, `destination_id` |
| `match_invitation_received` | Terima undangan | `from_user_id`, `destination_id` |
| `match_accepted` | Terima match | `matched_user_id`, `destination_id`, `time_to_accept_sec` |
| `match_declined` | Tolak match | `matched_user_id`, `reason` |
| `match_group_created` | Buat grup trip | `group_size`, `destination_id` |
| `match_chat_opened` | Buka chat match | `match_id`, `message_count` |
| `match_chat_sent` | Kirim pesan | `match_id`, `message_type` (text/photo/location) |

### 3.5 Booking & Payment Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `booking_initiated` | Mulai proses booking | `destination_id`, `date`, `participants` |
| `booking_form_completed` | Form terisi lengkap | `destination_id`, `total_price`, `addons[]` |
| `payment_initiated` | Masuk halaman payment | `booking_id`, `amount`, `payment_method` |
| `payment_method_selected` | Pilih metode bayar | `method` (gopay/ovo/bca/bni/credit_card/alfamart) |
| `payment_snap_opened` | Midtrans Snap terbuka | `booking_id`, `amount` |
| `payment_completed` | Pembayaran sukses | `booking_id`, `amount`, `method`, `time_to_pay_sec` |
| `payment_failed` | Pembayaran gagal | `booking_id`, `amount`, `method`, `error_code`, `error_message` |
| `payment_expired` | Pembayaran expired | `booking_id`, `amount`, `time_held_sec` |
| `booking_confirmed` | Booking terkonfirmasi | `booking_id`, `destination_id`, `amount` |
| `booking_cancelled` | Booking dibatalkan | `booking_id`, `reason`, `refund_amount` |
| `booking_refunded` | Refund diproses | `booking_id`, `refund_amount`, `refund_method` |

### 3.6 Subscription Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `subscription_page_viewed` | Lihat halaman plan | `current_plan` |
| `subscription_plan_selected` | Pilih plan | `plan_id`, `plan_name`, `price`, `interval` (monthly/yearly) |
| `subscription_started` | Mulai langganan | `plan_id`, `amount`, `payment_method`, `trial` |
| `subscription_renewed` | Perpanjangan otomatis | `plan_id`, `amount`, `renewal_number` |
| `subscription_upgraded` | Upgrade plan | `from_plan`, `to_plan`, `prorated_amount` |
| `subscription_downgraded` | Downgrade plan | `from_plan`, `to_plan`, `effective_date` |
| `subscription_cancelled` | Batalkan langganan | `plan_id`, `reason`, `usage_days`, `features_used[]` |
| `subscription_expired` | Langganan habis | `plan_id`, `total_months` |

### 3.7 Gamification Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `checkin_performed` | Check-in di destinasi | `destination_id`, `method` (gps/nfc/qr), `xp_earned` |
| `badge_earned` | Dapat badge | `badge_id`, `badge_name`, `rarity` (common/rare/epic/legendary) |
| `mission_started` | Mulai mission | `mission_id`, `mission_name`, `mission_type` |
| `mission_completed` | Selesai mission | `mission_id`, `time_to_complete_sec`, `xp_earned`, `bonus_xp` |
| `level_up` | Naik level | `new_level`, `total_xp`, `xp_to_next` |
| `leaderboard_viewed` | Lihat leaderboard | `leaderboard_type` (global/local/friends), `user_rank` |
| `achievement_shared` | Share achievement | `achievement_type`, `platform` |
| `streak_maintained` | Streak harian | `streak_days`, `xp_bonus` |
| `streak_broken` | Streak putus | `previous_streak_days` |

### 3.8 Social Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `post_created` | Buat post/story | `type` (photo/story/review), `has_location`, `destination_id` |
| `post_liked` | Like post | `post_id`, `author_id` |
| `post_commented` | Komentar | `post_id`, `comment_length` |
| `post_shared` | Share post | `post_id`, `platform` |
| `user_followed` | Follow user | `followed_user_id`, `source` (search/match/review) |
| `user_unfollowed` | Unfollow | `unfollowed_user_id` |
| `referral_sent` | Kirim referral | `platform`, `referral_code` |
| `referral_completed` | Referral berhasil | `referral_code`, `referred_user_id`, `reward_type` |

### 3.9 Engagement & Retention Events

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `app_opened` | App dibuka | `source` (notification/deeplink/direct/background), `cold_start` |
| `app_backgrounded` | App ke background | `session_duration_sec`, `pages_viewed` |
| `notification_received` | Notif diterima | `notification_type`, `notification_id` |
| `notification_clicked` | Klik notif | `notification_type`, `notification_id`, `deep_link` |
| `offline_mode_entered` | Masuk mode offline | `cached_pages`, `pending_actions` |
| `offline_sync_completed` | Sync selesai | `sync_duration_sec`, `items_synced`, `conflicts` |
| `session_started` | Sesi baru dimulai | `device_type`, `os`, `app_version`, `connection_type` |
| `feature_flag_seen` | Feature flag aktif | `flag_name`, `variant`, `flag_value` |

---

## 4. Funnel Analysis

### 4.1 Core Funnels

#### Funnel 1: Onboarding → First Trip

```
Step 1: user_registered          → 100% (baseline)
Step 2: onboarding_started       → Target: ≥ 90%
Step 3: preference_set           → Target: ≥ 75%
Step 4: onboarding_completed     → Target: ≥ 65%
Step 5: destination_viewed       → Target: ≥ 50%
Step 6: match_started            → Target: ≥ 30%
Step 7: booking_initiated        → Target: ≥ 15%
Step 8: payment_completed        → Target: ≥ 8%

Drop-off yang harus di-monitor:
├── Step 2→3: Jika > 20% drop → simplify preference form
├── Step 5→6: Jika > 40% drop → improve CTA di destination page
├── Step 7→8: Jika > 50% drop → add payment reminders, simplify checkout
```

#### Funnel 2: Re-engagement (D7 Retention)

```
Step 1: user_registered          → 100%
Step 2: app_opened (day 1)       → Target: ≥ 70%
Step 3: app_opened (day 3)       → Target: ≥ 40%
Step 4: app_opened (day 7)       → Target: ≥ 25%
Step 5: meaningful_action (day 7) → Target: ≥ 15%

meaningful_action = destination_viewed | match_started | booking_initiated
```

#### Funnel 3: Subscription Conversion

```
Step 1: subscription_page_viewed → 100%
Step 2: subscription_plan_selected → Target: ≥ 40%
Step 3: payment_initiated         → Target: ≥ 25%
Step 4: subscription_started      → Target: ≥ 15%
Step 5: subscription_renewed      → Target: ≥ 70% (monthly renewal)

Key insight: Track which features users hit limits on BEFORE
seeing the subscription page (conversion trigger analysis).
```

#### Funnel 4: Trip Match Success

```
Step 1: match_started             → 100%
Step 2: match_results_viewed      → Target: ≥ 95%
Step 3: match_profile_viewed      → Target: ≥ 60%
Step 4: match_invitation_sent     → Target: ≥ 30%
Step 5: match_accepted            → Target: ≥ 15%
Step 6: booking_initiated         → Target: ≥ 10%
Step 7: booking_confirmed         → Target: ≥ 7%
```

### 4.2 PostHog Funnel Queries

```javascript
// PostHog Insight: Onboarding Funnel
posthog.create_insight({
  name: 'Onboarding → First Trip Funnel',
  type: 'FunnelViz',
  filters: {
    events: [
      { id: 'user_registered', type: 'events', order: 0 },
      { id: 'onboarding_started', type: 'events', order: 1 },
      { id: 'preference_set', type: 'events', order: 2 },
      { id: 'onboarding_completed', type: 'events', order: 3 },
      { id: 'destination_viewed', type: 'events', order: 4 },
      { id: 'booking_initiated', type: 'events', order: 5 },
      { id: 'payment_completed', type: 'events', order: 6 },
    ],
    funnel_window_interval: 7,
    funnel_window_interval_unit: 'day',
    breakdown: 'signup_source',
  },
});
```

---

## 5. Cohort Analysis

### 5.1 Retention Cohorts

```
Cohort Definition:
├── Cohort = users grouped by registration week
├── Retention = user performs ANY event in subsequent weeks
├── Key actions = {app_opened, destination_viewed, booking_initiated}

Target Retention:
├── Day 1:  ≥ 70%
├── Day 7:  ≥ 25%
├── Day 14: ≥ 18%
├── Day 30: ≥ 12%
├── Day 90: ≥ 6%

Segmented retention:
├── Tourist (casual):    D7 ≥ 15%, D30 ≥ 5%
├── Explorer (regular):  D7 ≥ 40%, D30 ≥ 25%
├── Mitra (business):    D7 ≥ 60%, D30 ≥ 50%
```

### 5.2 PostHog Cohort Queries

```javascript
// Retention by segment
posthog.create_insight({
  name: 'D7 Retention by Segment',
  type: 'RetentionTable',
  filters: {
    returning_entity: { id: 'app_opened', type: 'events' },
    target_entity: { id: 'user_registered', type: 'events' },
    period: 'Week',
    retention_type: 'retention_first_time',
    properties: [
      {
        key: 'segment',
        value: ['tourist', 'explorer', 'mitra'],
        operator: 'exact',
        type: 'event',
      },
    ],
  },
});

// Stickiness: how many days per week does user return?
posthog.create_insight({
  name: 'Weekly Stickiness',
  type: 'Stickiness',
  filters: {
    events: [{ id: 'app_opened', type: 'events' }],
    period: 'Week',
    shown_as: 'Stickiness',
  },
});
```

### 5.3 Churn Indicators

```
Churn signal events yang harus di-track:
├── No app_opened dalam 7 hari (setelah D7 active)
├── subscription_cancelled
├── Notification opt-out semua category
├── Profile incomplete > 30 days
├── Zero bookings dalam 30 days setelah registrasi

Automated alerts:
├── Churn risk user > 40% → trigger re-engagement push
├── D7 inactive → send "Kami kangen" email + recommendation
├── D30 inactive → send exclusive discount code
├── subscription_cancelled → exit survey + win-back offer (day 3)
```

---

## 6. Dashboard Definition

### 6.1 Admin Dashboard — Business Metrics

```
Dashboard: "Jejakawan Business Overview"
Refresh: Real-time (PostHog) + Daily (custom aggregation)

Section 1: Growth Metrics
├── New users today / this week / this month (with WoW trend)
├── MAU (Monthly Active Users)
├── DAU / MAU ratio (stickiness)
├── User growth rate (% MoM)
└── Cumulative users chart

Section 2: Engagement Metrics
├── Avg session duration
├── Avg screens per session
├── D1 / D7 / D30 retention (line chart)
├── Feature adoption rates (% users using each feature)
└── Most visited destinations (top 10)

Section 3: Revenue Metrics
├── GMV (Gross Merchandise Value) today/week/month
├── Revenue (platform commission)
├── ARPU (Average Revenue Per User)
├── ARPPU (Average Revenue Per Paying User)
├── Subscription MRR / ARR
├── Payment success rate
└── Revenue by segment (pie chart)

Section 4: Matching Metrics
├── Match requests per day
├── Match success rate (invitation → accepted)
├── Average time to match
├── Top matched destinations
└── Match → booking conversion rate

Section 5: Gamification Metrics
├── Average XP per user
├── Check-in rate (vs active users)
├── Mission completion rate
├── Badge distribution (rarity chart)
└── Leaderboard engagement (views / participation)
```

### 6.2 Admin Dashboard — Operational Metrics

```
Dashboard: "Jejakawan Operations"
Refresh: Every 5 minutes

Section 1: System Health
├── API response time (p50/p95/p99)
├── Error rate by endpoint
├── Supabase connection pool utilization
├── Edge function execution time
└── Storage usage (images)

Section 2: Content Moderation
├── Pending reviews (count)
├── Flagged content (count + type breakdown)
├── Average moderation response time
├── Auto-flagged vs manual-flagged ratio
└── Reports queue

Section 3: Support Metrics
├── Open tickets
├── Average resolution time
├── CSAT score
├── Top issue categories
└── Escalation rate
```

### 6.3 Mitra Dashboard

```
Dashboard: "Mitra Portal — My Destinations"
Refresh: Daily

Section 1: Performance
├── Views on my destinations (today/week/month)
├── Booking count & trend
├── Revenue (my share)
├── Average rating
└── Review sentiment (positive/neutral/negative)

Section 2: Content
├── Photo gallery completeness per destination
├── Description quality score
├── Response rate to reviews
└── Content freshness (last update date)

Section 3: Comparison
├── My destination vs category average (views, bookings, rating)
├── Regional ranking
└── Seasonal trends
```

---

## 7. KPI Targets Per Segment

### 7.1 Tourist (Pengguna Casual)

| KPI | Target (Launch +3mo) | Target (Launch +12mo) |
|-----|----------------------|-----------------------|
| Monthly Active Users | 5,000 | 50,000 |
| D7 Retention | 15% | 20% |
| D30 Retention | 5% | 8% |
| Onboarding Completion | 60% | 70% |
| First Booking Rate | 8% | 12% |
| Avg Session Duration | 4 min | 6 min |
| Destination Views/Session | 3 | 5 |
| Review Submission Rate | 2% | 5% |
| Referral Rate | 3% | 5% |
| NPS Score | 30 | 45 |

### 7.2 Explorer (Pengguna Aktif)

| KPI | Target (Launch +3mo) | Target (Launch +12mo) |
|-----|----------------------|-----------------------|
| Monthly Active Users | 1,500 | 15,000 |
| D7 Retention | 40% | 50% |
| D30 Retention | 25% | 35% |
| Match Requests/User/Month | 2 | 4 |
| Match Acceptance Rate | 20% | 30% |
| Bookings/User/Quarter | 1.5 | 3 |
| Check-in Rate | 40% | 60% |
| Mission Completion Rate | 30% | 50% |
| Subscription Conversion | 5% | 10% |
| Average Booking Value | Rp 400K | Rp 600K |
| Social Posts/User/Month | 1 | 3 |

### 7.3 Mitra (Business Partner)

| KPI | Target (Launch +3mo) | Target (Launch +12mo) |
|-----|----------------------|-----------------------|
| Registered Mitra | 50 | 500 |
| Active Mitra (monthly login) | 60% | 75% |
| Destinations Listed | 200 | 2,000 |
| Avg Destinations/Mitra | 4 | 8 |
| Photo Upload Rate | 5 photos/dest | 10 photos/dest |
| Description Completion | 70% | 90% |
| Review Response Rate | 40% | 70% |
| Revenue/Mitra/Month | Rp 2M | Rp 8M |
| Content Freshness (updated < 30d) | 50% | 80% |
| CSAT from Mitra | 3.5/5 | 4.2/5 |

---

## 8. Custom Event Properties Schema

### 8.1 Standard Properties (Auto-attached)

```typescript
// Every event automatically gets these
interface StandardProperties {
  // PostHog built-in
  $current_url: string;
  $browser: string;
  $os: string;
  $device_type: 'mobile' | 'tablet' | 'desktop';
  $screen_width: number;
  $screen_height: number;
  $referrer: string;
  $session_id: string;
  $ip: string;

  // Jejakawan custom (set via super_properties)
  app_version: string;
  user_segment: 'tourist' | 'explorer' | 'mitra' | null;
  user_plan: 'free' | 'premium' | 'business';
  is_pwa: boolean;
  install_status: 'installed' | 'browser';
  connection_type: '4g' | '3g' | '2g' | 'wifi' | 'offline';
  province: string | null; // user's current province
}
```

### 8.2 Super Properties Setup

```typescript
// src/lib/analytics/super-properties.ts
import posthog from 'posthog-js';

export function registerSuperProperties() {
  posthog.register({
    app_version: import.meta.env.VITE_APP_VERSION ?? '1.0.0',
    is_pwa: window.matchMedia('(display-mode: standalone)').matches,
    install_status: window.matchMedia('(display-mode: standalone)').matches
      ? 'installed'
      : 'browser',
    connection_type: (navigator as any).connection?.effectiveType ?? 'unknown',
  });

  // Update on user change
  posthog.register_for_session({
    user_segment: getUserSegment(),
    user_plan: getUserPlan(),
  });
}
```

---

## 9. Privacy & Compliance

### 9.1 Data Handling Rules

```
Data yang TIDAK boleh di-track:
├── Password (never)
├── Payment card details (never)
├── Exact GPS coordinates (aggregate to province/city only)
├── Personal chat messages (only metadata: count, length)
├── NIK / KTP numbers (never)
├── Health data (never)

Data yang boleh di-track (dengan consent):
├── Email (hashed for deduplication)
├── Device info (browser, OS, screen size)
├── App behavior (clicks, views, sessions)
├── Location (province level only in analytics)
├── Preferences (travel style, interests)
└── Purchase history (booking IDs, amounts, categories)
```

### 9.2 PostHog Privacy Config

```typescript
// Mask sensitive elements
posthog.init(key, {
  session_recording: {
    maskAllInputs: true,
    maskTextSelector: '[data-private], .email, .phone',
    blockSelector: '[data-no-capture], .payment-form',
  },
  // Don't capture URL fragments (could contain tokens)
  capture_pageview: true,
  sanitize_properties: (properties) => {
    // Remove any accidental PII
    delete properties.$ip; // Don't store IP
    if (properties.email) {
      properties.email_hash = hash(properties.email);
      delete properties.email;
    }
    return properties;
  },
});
```

---

## 10. Automated Alerts & Actions

```typescript
// src/lib/analytics/alerts.ts
// PostHog webhook → Supabase Edge Function → automated actions

interface AlertConfig {
  metric: string;
  threshold: number;
  direction: 'above' | 'below';
  action: 'slack' | 'email' | 'auto_fix';
}

const ALERTS: AlertConfig[] = [
  {
    metric: 'payment_success_rate',
    threshold: 0.90, // below 90%
    direction: 'below',
    action: 'slack',
  },
  {
    metric: 'd1_retention',
    threshold: 0.50, // below 50%
    direction: 'below',
    action: 'email',
  },
  {
    metric: 'error_rate',
    threshold: 0.05, // above 5%
    direction: 'above',
    action: 'slack',
  },
  {
    metric: 'match_success_rate',
    threshold: 0.10, // below 10%
    direction: 'below',
    action: 'slack',
  },
  {
    metric: 'api_p95_latency',
    threshold: 3000, // above 3 seconds
    direction: 'above',
    action: 'slack',
  },
];
```
