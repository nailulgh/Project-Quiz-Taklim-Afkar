# Ta'lim Afkar — Technical Architecture Design

**Version:** 1.0.0  
**Status:** Living Document  
**Last Updated:** 2026-05  
**Related:** [`srs.md`](./srs.md) · [`todo.md`](./todo.md) · [`claude.md`](./claude.md) · [`changelog.md`](./changelog.md)

---

## Table of Contents

1. [High-Level Architecture](#1-high-level-architecture)
2. [Technology Stack Decision](#2-technology-stack-decision)
3. [Modular Monolith Design](#3-modular-monolith-design)
4. [Domain Boundaries](#4-domain-boundaries)
5. [Database Architecture](#5-database-architecture)
6. [Event-Driven Architecture](#6-event-driven-architecture)
7. [Cache Architecture](#7-cache-architecture)
8. [Realtime Multiplayer Architecture](#8-realtime-multiplayer-architecture)
9. [AI Service Architecture](#9-ai-service-architecture)
10. [Authentication & Authorization](#10-authentication--authorization)
11. [Search Architecture](#11-search-architecture)
12. [Folder Structure](#12-folder-structure)
13. [API Conventions](#13-api-conventions)
14. [Frontend Architecture](#14-frontend-architecture)
15. [Deployment Architecture](#15-deployment-architecture)
16. [DevOps & CI/CD](#16-devops--cicd)
17. [Monitoring & Observability](#17-monitoring--observability)
18. [Storage Strategy](#18-storage-strategy)
19. [Scalability Roadmap](#19-scalability-roadmap)
20. [Future Microservices Migration](#20-future-microservices-migration)

---

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌───────────────┐   │
│  │   Web App (PWA) │  │  Mobile (RN)    │  │  Admin Panel  │   │
│  │  Next.js 14+    │  │  Future Phase   │  │  Next.js      │   │
│  └────────┬────────┘  └────────┬────────┘  └───────┬───────┘   │
└───────────┼────────────────────┼───────────────────┼───────────┘
            │ HTTPS              │ HTTPS             │ HTTPS
            ▼                    ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                      CDN / EDGE (Cloudflare)                     │
│  - Static assets, media files, edge caching                      │
│  - DDoS protection, WAF, rate limiting                           │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                   API GATEWAY / LOAD BALANCER                    │
│  - Nginx / Caddy (MVP), AWS ALB (Scale)                          │
│  - SSL termination, routing, WebSocket upgrade                   │
└──────┬───────────────────────────────────────┬──────────────────┘
       │ REST/HTTP                              │ WebSocket
       ▼                                        ▼
┌──────────────────┐                 ┌──────────────────────────┐
│   API Server     │                 │   WS / Realtime Server   │
│  (Hono.js /      │                 │   (Node.js + uWebSockets)│
│   Fastify)       │                 │   Duel, Musabaqoh rooms  │
│  Modular Monolith│                 └──────────┬───────────────┘
└──────┬───────────┘                            │
       │                              ┌─────────▼──────────┐
       │                              │   Redis Pub/Sub    │
       │                              │   Session State    │
       │                              └────────────────────┘
       │
┌──────▼──────────────────────────────────────────────────────┐
│                   DOMAIN MODULES (Internal)                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │   Auth   │ │   Game   │ │ Learning │ │   Classroom  │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐   │
│  │  Content │ │    AI    │ │ Gamifi-  │ │  Analytics   │   │
│  │  (Kitab) │ │ Services │ │ cation   │ │              │   │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘   │
└──────┬───────────────────────────────────────────┬──────────┘
       │                                           │
┌──────▼──────────┐  ┌────────────────┐  ┌────────▼───────┐
│   PostgreSQL    │  │  Redis Cache   │  │  Message Queue │
│   Primary DB    │  │  + Sessions    │  │  BullMQ/Redis  │
│   + Read Replica│  └────────────────┘  └────────────────┘
└─────────────────┘
       │
┌──────▼──────────┐  ┌────────────────┐  ┌────────────────┐
│  Elasticsearch  │  │ Object Storage │  │  AI APIs       │
│  Semantic Search│  │ (S3/R2) Media  │  │ OpenAI/Anthr.  │
└─────────────────┘  └────────────────┘  └────────────────┘
```

---

## 2. Technology Stack Decision

### 2.1 Backend

**Decision: Node.js + Hono.js (atau Fastify)**

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| Node.js + Hono | Ultra-fast, edge-ready, TypeScript-native, excellent ecosystem | Relative newcomer vs Express | ✅ PILIH |
| Node.js + Fastify | Mature, excellent performance, schema validation built-in | Lebih verbose | ✅ Alternatif valid |
| Go (Fiber/Gin) | Blazing fast, low memory | Ekosistem AI/ML lebih terbatas, less AI-agent friendly | ❌ |
| Python (FastAPI) | AI/ML native, excellent library | Performance lebih rendah untuk realtime, runtime overhead | ❌ MVP |
| Elixir (Phoenix) | Realtime superior (LiveView, channels) | Learning curve tinggi, ekosistem terbatas | ❌ |

**Rationale:** Hono.js memberikan performance mendekati Fastify dengan DX yang lebih modern. TypeScript end-to-end dari backend ke frontend memudahkan AI coding agents memahami kode. Single runtime (Node.js) memudahkan DevOps awal.

### 2.2 Frontend

**Decision: Next.js 14+ (App Router)**

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| Next.js 14 | SSR/SSG/ISR, App Router, Server Components, edge-ready | Bundle size jika tidak di-optimize | ✅ PILIH |
| Remix | Excellent DX, form handling | Ekosistem lebih kecil | ❌ |
| SvelteKit | Minimal bundle, excellent performance | TypeScript support kurang mature, AI agents kurang familiar | ❌ |
| Nuxt (Vue) | Mature, baik untuk CMS-heavy | React ecosystem lebih dominan untuk game-like UI | ❌ |

**UI Library Decision: Tailwind CSS + shadcn/ui + Framer Motion**
- Tailwind: utility-first, mudah dikerjakan AI agents
- shadcn/ui: composable, tidak opinionated, fully owned code
- Framer Motion: animasi yang dibutuhkan untuk feel "game"

### 2.3 Database

**Decision: PostgreSQL (Primary) + Redis (Cache/Session/Queue)**

| Option | Pros | Cons |
|--------|------|------|
| PostgreSQL | ACID, JSON support, full-text search, mature, excellent tooling | Horizontal write scaling butuh effort |
| MySQL | Familiar | Fitur lebih terbatas dari Postgres |
| MongoDB | Flexible schema | Eventual consistency masalah untuk game scoring |
| PlanetScale | Horizontal scaling mudah | Vendor lock-in, biaya |

**Rationale:** PostgreSQL dengan Drizzle ORM (type-safe, performant, AI-friendly schema definition). Untuk MVP, single primary cukup. Read replica ditambah saat MAU > 10.000.

### 2.4 Realtime

**Decision: Node.js + uWebSockets.js (atau Socket.io)**

- **uWebSockets.js:** Performance tertinggi, untuk production scale
- **Socket.io:** DX lebih baik, fallback otomatis, lebih mudah untuk MVP

**MVP:** Socket.io (DX > raw performance untuk awal)  
**Scale:** Migrasi ke uWebSockets.js dengan Redis Adapter

### 2.5 Queue System

**Decision: BullMQ (Redis-based)**

Alasan: sudah menggunakan Redis, satu dependency, battle-tested, excellent DX, dashboard tersedia (Bull Board).

Jobs yang di-queue:
- AI question generation
- Email notifications
- Weekly report generation
- Analytics aggregation
- AI training data preprocessing

### 2.6 Search

**Decision: PostgreSQL Full-Text Search (MVP) → Elasticsearch/OpenSearch (Scale)**

MVP: PostgreSQL FTS sudah cukup untuk bank soal < 100.000 soal.  
Scale: Elasticsearch dengan text embedding untuk semantic search.

### 2.7 AI Integration

**Decision: Vercel AI SDK + OpenAI (primary) + Anthropic (fallback)**

- Vercel AI SDK abstraksi multi-provider
- OpenAI GPT-4o untuk generasi soal dan AI tutor
- Embeddings: text-embedding-3-small untuk semantic search
- Caching response AI di Redis (TTL 24 jam untuk soal similar)

### 2.8 Deployment

**MVP:** Railway / Render (simplicity) atau VPS (Hetzner) dengan Docker Compose  
**Scale:** Kubernetes (k8s) di AWS/GCP atau managed k8s (EKS/GKE)

**CDN:** Cloudflare (gratis tier cukup untuk MVP)  
**Object Storage:** Cloudflare R2 (no egress fee) atau AWS S3

### 2.9 Observability

- **Logging:** Pino (structured JSON) → Loki / Logtail
- **Metrics:** Prometheus + Grafana
- **Tracing:** OpenTelemetry → Jaeger / Grafana Tempo
- **Error Tracking:** Sentry (free tier cukup untuk MVP)
- **Uptime:** Better Uptime / UptimeRobot

---

## 3. Modular Monolith Design

### Prinsip

Modular Monolith adalah pendekatan di mana semua modul berjalan dalam satu proses, namun dengan batas domain yang **ketat dan eksplisit**. Ini memberikan:

1. **Simplicity deployment** — satu aplikasi, satu deploy
2. **Performance** — in-process calls, no network overhead antar modul
3. **Developer productivity** — debugging mudah, testing mudah
4. **Migration path** — setiap modul dapat diextract menjadi service saat diperlukan

### Aturan Ketat Modular Monolith

```
❌ DILARANG:
  - Modul A langsung import internal class dari Modul B
  - Direct database query ke tabel yang "dimiliki" modul lain
  - Shared mutable state antar modul

✅ DIIZINKAN:
  - Modul A memanggil public interface/contract Modul B
  - Modul berkomunikasi via Events (internal event bus)
  - Shared read-only reference data (enum, constants)
```

### Internal Event Bus

Implementasi dengan `EventEmitter` atau library seperti `mitt` untuk komunikasi antar modul:

```typescript
// Contoh: GameModule emit event, GamificationModule listen
eventBus.emit('game:session_completed', {
  userId: 'xxx',
  mode: 'kilat_fiqih',
  score: 850,
  xpEarned: 120,
})

// GamificationModule
eventBus.on('game:session_completed', async (payload) => {
  await gamificationService.processXP(payload.userId, payload.xpEarned)
  await streakService.recordActivity(payload.userId)
  await achievementService.checkTriggers(payload.userId, payload)
})
```

---

## 4. Domain Boundaries

### Domain Map

```
┌─────────────────────────────────────────────────────────────┐
│                    CORE DOMAINS                              │
│                                                              │
│  ┌─────────────────┐    ┌─────────────────────────────────┐ │
│  │   Auth Domain   │    │      User Domain                │ │
│  │                 │    │                                 │ │
│  │ - Registration  │    │ - Profile                       │ │
│  │ - Login/Logout  │    │ - Preferences                   │ │
│  │ - JWT/Session   │    │ - Avatars                       │ │
│  │ - OAuth         │    │ - Statistics                    │ │
│  └─────────────────┘    └─────────────────────────────────┘ │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   Game Domain                         │   │
│  │                                                       │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │   │
│  │  │ Kilat Fiqih  │  │ Tebak Kitab  │  │ Duel Afkar │ │   │
│  │  │   Engine     │  │   Engine     │  │  Engine    │ │   │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │   │
│  │  │ Tangga Ilmu  │  │ Urutan Dalil │  │ Musabaqoh  │ │   │
│  │  │   Engine     │  │   Engine     │  │  Engine    │ │   │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │   │
│  │                                                       │   │
│  │  Shared: Session Manager, Score Calculator           │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌─────────────────┐    ┌─────────────────────────────────┐ │
│  │ Content Domain  │    │    Learning Domain              │ │
│  │                 │    │                                 │ │
│  │ - Question Bank │    │ - Progression (Tangga Ilmu)     │ │
│  │ - Kitab Content │    │ - Mastery Scoring               │ │
│  │ - Tags/Metadata │    │ - Spaced Repetition             │ │
│  │ - Review Queue  │    │ - Learning Path                 │ │
│  └─────────────────┘    └─────────────────────────────────┘ │
│                                                              │
│  ┌─────────────────┐    ┌─────────────────────────────────┐ │
│  │Gamification Dom │    │    Classroom Domain             │ │
│  │                 │    │                                 │ │
│  │ - XP/Level      │    │ - Class Management              │ │
│  │ - Streak        │    │ - Student Enrollment            │ │
│  │ - Achievements  │    │ - Muallim Tools                 │ │
│  │ - Leaderboard   │    │ - Musabaqoh Sessions            │ │
│  │ - Seasonal      │    │ - Progress Reports              │ │
│  └─────────────────┘    └─────────────────────────────────┘ │
│                                                              │
│  ┌─────────────────┐    ┌─────────────────────────────────┐ │
│  │    AI Domain    │    │    Analytics Domain             │ │
│  │                 │    │                                 │ │
│  │ - Question Gen  │    │ - Event Ingestion               │ │
│  │ - AI Tutor      │    │ - Aggregation                   │ │
│  │ - Adaptive Rec  │    │ - Reports                       │ │
│  │ - Embeddings    │    │ - Dashboards                    │ │
│  └─────────────────┘    └─────────────────────────────────┘ │
│                                                              │
│  ┌─────────────────┐    ┌─────────────────────────────────┐ │
│  │ Notification    │    │    Admin Domain                 │ │
│  │ Domain          │    │                                 │ │
│  │ - Push          │    │ - User Management               │ │
│  │ - Email         │    │ - Institution Management        │ │
│  │ - In-App        │    │ - Feature Flags                 │ │
│  └─────────────────┘    └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Domain Ownership Rules

| Domain | Owns Tables | Exposes |
|--------|-------------|---------|
| Auth | `sessions`, `refresh_tokens`, `oauth_accounts` | `AuthService` interface |
| User | `users`, `profiles`, `preferences` | `UserService` interface |
| Game | `game_sessions`, `game_answers` | `GameService` interface |
| Content | `questions`, `kitabs`, `babs`, `matan_excerpts` | `ContentService` interface |
| Learning | `progressions`, `mastery_scores`, `spaced_repetition` | `LearningService` interface |
| Gamification | `xp_logs`, `levels`, `streaks`, `achievements`, `user_achievements` | `GamificationService` interface |
| Classroom | `classrooms`, `enrollments`, `musabaqoh_sessions` | `ClassroomService` interface |
| AI | `ai_conversations`, `ai_generated_questions`, `embeddings` | `AIService` interface |
| Analytics | `events`, `aggregated_metrics` | `AnalyticsService` interface |
| Notification | `notifications`, `notification_preferences` | `NotificationService` interface |

---

## 5. Database Architecture

### 5.1 Database Schema (ERD — Key Tables)

```sql
-- AUTH DOMAIN
CREATE TABLE users (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email       VARCHAR(255) UNIQUE NOT NULL,
  password    VARCHAR(255),           -- nullable jika OAuth only
  role        VARCHAR(50) NOT NULL DEFAULT 'mahasantri',
  is_verified BOOLEAN DEFAULT false,
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE profiles (
  user_id        UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  username       VARCHAR(100) UNIQUE NOT NULL,
  display_name   VARCHAR(255),
  avatar_url     VARCHAR(500),
  pesantren_id   UUID REFERENCES institutions(id),
  bio            TEXT,
  created_at     TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE institutions (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name         VARCHAR(255) NOT NULL,
  slug         VARCHAR(100) UNIQUE NOT NULL,
  city         VARCHAR(100),
  province     VARCHAR(100),
  is_verified  BOOLEAN DEFAULT false,
  created_at   TIMESTAMPTZ DEFAULT NOW()
);

-- CONTENT DOMAIN
CREATE TABLE kitabs (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug         VARCHAR(100) UNIQUE NOT NULL,
  name         VARCHAR(255) NOT NULL,
  author       VARCHAR(255),
  category     VARCHAR(100) NOT NULL, -- fiqih, syuabul_iman, adab, ulama
  description  TEXT,
  cover_url    VARCHAR(500),
  created_at   TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE babs (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  kitab_id     UUID REFERENCES kitabs(id) ON DELETE CASCADE,
  name         VARCHAR(255) NOT NULL,
  order_index  INTEGER NOT NULL,
  parent_bab_id UUID REFERENCES babs(id), -- untuk nested chapters
  created_at   TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE questions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  kitab_id        UUID REFERENCES kitabs(id),
  bab_id          UUID REFERENCES babs(id),
  question_text   TEXT NOT NULL,
  question_arabic TEXT,              -- optional Arab text
  options         JSONB NOT NULL,    -- [{key: 'A', text: '...', is_correct: bool}]
  correct_option  VARCHAR(1) NOT NULL,
  explanation     TEXT,              -- syarah/penjelasan
  difficulty      VARCHAR(20) DEFAULT 'medium', -- easy, medium, hard
  game_modes      VARCHAR[] DEFAULT ARRAY['kilat_fiqih'],
  source          VARCHAR(50) DEFAULT 'manual', -- manual, ai_generated
  status          VARCHAR(50) DEFAULT 'draft', -- draft, under_review, approved, rejected
  created_by      UUID REFERENCES users(id),
  approved_by     UUID REFERENCES users(id),
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_questions_kitab_bab ON questions(kitab_id, bab_id);
CREATE INDEX idx_questions_status ON questions(status);
CREATE INDEX idx_questions_game_modes ON questions USING gin(game_modes);

-- GAME DOMAIN
CREATE TABLE game_sessions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES users(id),
  game_mode       VARCHAR(50) NOT NULL,
  status          VARCHAR(50) NOT NULL DEFAULT 'active',
  score           INTEGER DEFAULT 0,
  xp_earned       INTEGER DEFAULT 0,
  questions_total INTEGER DEFAULT 0,
  questions_correct INTEGER DEFAULT 0,
  duration_ms     INTEGER,
  metadata        JSONB,              -- mode-specific data
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  completed_at    TIMESTAMPTZ
);

CREATE TABLE game_answers (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id      UUID REFERENCES game_sessions(id) ON DELETE CASCADE,
  question_id     UUID REFERENCES questions(id),
  user_answer     VARCHAR(1),
  is_correct      BOOLEAN,
  time_taken_ms   INTEGER,
  points_earned   INTEGER DEFAULT 0,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- GAMIFICATION DOMAIN
CREATE TABLE user_stats (
  user_id         UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
  total_xp        BIGINT DEFAULT 0,
  current_level   INTEGER DEFAULT 1,
  elo_rating      INTEGER DEFAULT 1000,
  streak_current  INTEGER DEFAULT 0,
  streak_longest  INTEGER DEFAULT 0,
  streak_last_activity_date DATE,
  total_games     INTEGER DEFAULT 0,
  total_wins      INTEGER DEFAULT 0,
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE achievements (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug            VARCHAR(100) UNIQUE NOT NULL,
  name            VARCHAR(255) NOT NULL,
  description     TEXT,
  badge_url       VARCHAR(500),
  category        VARCHAR(100),
  rarity          VARCHAR(50) DEFAULT 'common',
  criteria        JSONB NOT NULL       -- trigger conditions
);

CREATE TABLE user_achievements (
  user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
  achievement_id  UUID REFERENCES achievements(id),
  earned_at       TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (user_id, achievement_id)
);

-- LEARNING DOMAIN
CREATE TABLE learning_progressions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
  bab_id          UUID REFERENCES babs(id),
  status          VARCHAR(50) DEFAULT 'locked', -- locked, available, in_progress, completed
  stars           INTEGER DEFAULT 0,            -- 0-3
  best_score      INTEGER DEFAULT 0,
  attempts        INTEGER DEFAULT 0,
  completed_at    TIMESTAMPTZ,
  UNIQUE(user_id, bab_id)
);

CREATE TABLE spaced_repetition (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
  question_id     UUID REFERENCES questions(id),
  due_date        DATE NOT NULL,
  interval_days   INTEGER DEFAULT 1,
  ease_factor     DECIMAL(4,2) DEFAULT 2.5,
  repetition_count INTEGER DEFAULT 0,
  UNIQUE(user_id, question_id)
);

-- CLASSROOM DOMAIN
CREATE TABLE classrooms (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name            VARCHAR(255) NOT NULL,
  muallim_id      UUID REFERENCES users(id),
  institution_id  UUID REFERENCES institutions(id),
  invite_code     VARCHAR(10) UNIQUE NOT NULL,
  code_expires_at TIMESTAMPTZ,
  status          VARCHAR(50) DEFAULT 'active',
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE classroom_enrollments (
  classroom_id    UUID REFERENCES classrooms(id) ON DELETE CASCADE,
  student_id      UUID REFERENCES users(id) ON DELETE CASCADE,
  enrolled_at     TIMESTAMPTZ DEFAULT NOW(),
  status          VARCHAR(50) DEFAULT 'active',
  PRIMARY KEY (classroom_id, student_id)
);

CREATE TABLE musabaqoh_sessions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  classroom_id    UUID REFERENCES classrooms(id),
  host_id         UUID REFERENCES users(id),
  room_code       VARCHAR(10) UNIQUE NOT NULL,
  status          VARCHAR(50) DEFAULT 'lobby',
  question_ids    UUID[] NOT NULL,
  time_per_question_s INTEGER DEFAULT 15,
  current_question_idx INTEGER DEFAULT 0,
  started_at      TIMESTAMPTZ,
  ended_at        TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- ANALYTICS DOMAIN
CREATE TABLE analytics_events (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID,               -- nullable untuk anonymous
  event_type      VARCHAR(100) NOT NULL,
  event_data      JSONB,
  session_id      VARCHAR(100),
  created_at      TIMESTAMPTZ DEFAULT NOW()
) PARTITION BY RANGE (created_at); -- Time-based partitioning
```

### 5.2 Database Design Decisions

**Mengapa UUID bukan SERIAL/BIGINT?**
- UUID tidak expose jumlah records (security)
- Aman untuk future distributed system (tidak ada id collision)
- Tradeoff: index sedikit lebih besar → acceptable

**Mengapa JSONB untuk options soal?**
- Struktur opsi dapat bervariasi per game mode
- Query dengan `@>` operator tetap efisien
- Hindari over-normalization untuk MVP

**Partitioning analytics_events:**
- Table ini akan tumbuh sangat cepat
- Range partition per bulan untuk query efisien dan archiving

---

## 6. Event-Driven Architecture

### 6.1 Internal Events (In-Process)

```typescript
// Domain Events Catalog
type DomainEvent =
  // Game Events
  | { type: 'game.session_started'; userId: string; mode: GameMode; sessionId: string }
  | { type: 'game.session_completed'; userId: string; mode: GameMode; score: number; xpEarned: number; accuracy: number }
  | { type: 'game.answer_submitted'; userId: string; questionId: string; correct: boolean; timeTakenMs: number }
  | { type: 'game.duel_result'; winnerId: string; loserId: string; eloChange: number }

  // Learning Events
  | { type: 'learning.chapter_completed'; userId: string; babId: string; stars: number }
  | { type: 'learning.kitab_completed'; userId: string; kitabId: string }

  // Social Events
  | { type: 'classroom.student_joined'; classroomId: string; studentId: string }
  | { type: 'musabaqoh.session_ended'; sessionId: string; results: RankResult[] }

  // User Events
  | { type: 'user.streak_updated'; userId: string; streakCount: number; isNewRecord: boolean }
  | { type: 'user.level_up'; userId: string; newLevel: number; oldLevel: number }
  | { type: 'user.achievement_earned'; userId: string; achievementId: string }
```

### 6.2 Async Jobs (BullMQ)

```typescript
// Queue Definitions
const queues = {
  'ai:generate-question':    { concurrency: 5, timeout: 30000 },
  'ai:analyze-weakness':     { concurrency: 10, timeout: 60000 },
  'email:send':              { concurrency: 50, timeout: 10000 },
  'notification:push':       { concurrency: 100, timeout: 5000 },
  'analytics:aggregate':     { concurrency: 2, timeout: 120000 },
  'report:generate':         { concurrency: 5, timeout: 60000 },
  'streak:check-reminders':  { concurrency: 1, timeout: 300000 }, // cron job
}
```

---

## 7. Cache Architecture

### 7.1 Cache Strategy per Data Type

| Data | TTL | Strategy | Key Pattern |
|------|-----|----------|-------------|
| Leaderboard global | 5 menit | Cache-aside | `lb:global:{period}` |
| Leaderboard institusi | 5 menit | Cache-aside | `lb:inst:{id}:{period}` |
| User stats | 60 detik | Write-through | `user:stats:{userId}` |
| Question detail | 1 jam | Cache-aside | `q:{questionId}` |
| Active game session | Session duration | Direct Redis | `game:session:{sessionId}` |
| Duel room state | Session duration | Direct Redis | `duel:room:{roomId}` |
| Musabaqoh room state | Session duration | Direct Redis | `musabaqoh:{roomCode}` |
| AI response (soal generation) | 24 jam | Cache-aside | `ai:gen:{hash(prompt)}` |
| JWT blacklist | Token TTL | Set | `jwt:blacklist` |
| Rate limit counters | 15 menit | Incr/Expire | `rl:{ip}:{endpoint}` |

### 7.2 Redis Data Structures

```
Leaderboard (Sorted Set):
  ZADD lb:global:weekly {score} {userId}
  ZREVRANGE lb:global:weekly 0 99 WITHSCORES

Streak (String + Hash):
  HSET user:streak:{userId} current 7 longest 15 last_date "2026-05-20"

Duel Room (Hash + Pub/Sub):
  HSET duel:room:{roomId} status "active" player1 "{json}" player2 "{json}"
  PUBLISH duel:{roomId} '{"type":"QUESTION_START", ...}'

Musabaqoh Scores (Sorted Set per session):
  ZADD musabaqoh:scores:{sessionId} {score} {userId}
  TTL musabaqoh:scores:{sessionId} 86400
```

---

## 8. Realtime Multiplayer Architecture

### 8.1 WebSocket Architecture

```
Client ──WebSocket──► WS Handler ──► Room Manager ──► Redis Pub/Sub
                                          │
                                    Game Engine
                                    (business logic)
                                          │
                               PostgreSQL (final persist)
```

### 8.2 Horizontal Scaling dengan Redis Pub/Sub

Saat ada lebih dari 1 instance WS server:

```
WS Server 1         WS Server 2
   │                     │
Player A connects    Player B connects
   │                     │
   └────► Redis Channel "room:ABC123" ◄────┘

When Player A answers:
  WS Server 1 → publish to "room:ABC123"
  WS Server 2 → subscribes "room:ABC123" → forward to Player B
```

### 8.3 Room Lifecycle Management

```typescript
interface RoomState {
  roomCode: string
  type: 'duel' | 'musabaqoh'
  status: 'lobby' | 'active' | 'paused' | 'ended'
  participants: Map<string, ParticipantState>
  currentQuestion: Question | null
  questionIndex: number
  scores: Map<string, number>
  createdAt: Date
  startedAt?: Date
}

// Room TTL: 2 jam dari creation, cleanup otomatis
// State disimpan di Redis, backup ke PostgreSQL setiap question selesai
```

### 8.4 Disconnect & Reconnect Handling

```
Player disconnect detected (ping timeout):
  1. Set participant status = "disconnected"
  2. Start 15-second grace timer
  3. If reconnect within 15s: restore state, continue game
  4. If no reconnect:
     - Duel: forfeit → opponent wins
     - Musabaqoh: mark as disconnected, score preserved
```

---

## 9. AI Service Architecture

### 9.1 AI Module Structure

```
src/modules/ai/
├── services/
│   ├── question-generator.service.ts   # Generate soal dari teks
│   ├── ai-tutor.service.ts             # Chat tutor
│   ├── adaptive-learning.service.ts   # Weakness analysis + recommendation
│   ├── embedding.service.ts           # Vector embeddings
│   └── content-moderator.service.ts   # Moderate user content
├── providers/
│   ├── openai.provider.ts             # OpenAI integration
│   └── anthropic.provider.ts          # Anthropic fallback
├── prompts/
│   ├── question-generation.prompt.ts  # Prompt templates
│   └── tutor.prompt.ts
└── cache/
    └── ai-cache.service.ts            # Cache AI responses
```

### 9.2 Question Generation Pipeline

```
Input: Teks matan/kitab + metadata (kitab, bab, difficulty)
  ↓
Prompt construction (structured template)
  ↓
Cache check (hash prompt → Redis)
  ↓ (cache miss)
OpenAI API call (GPT-4o)
  ↓
Response parsing & validation
  ↓
Store as draft question (status: 'under_review')
  ↓
Notify muallim via notification queue
  ↓
Muallim review → Approve/Edit/Reject
  ↓
If approved: status = 'approved', available in game
```

### 9.3 Adaptive Learning Engine

```
Every 24 hours (cron job):
  For each active user:
    1. Fetch last 30 game answers
    2. Group by topic/bab
    3. Calculate accuracy per topic
    4. Identify weakest topics (accuracy < 60%)
    5. Score topics by: weakness × recency × importance
    6. Generate recommendation list
    7. Cache in Redis (TTL 24 jam)
    8. Push notification "Ada rekomendasi belajar baru"

Spaced Repetition (SM-2 algorithm):
  - For each incorrectly answered question
  - Schedule next review: interval based on ease factor
  - Store in spaced_repetition table
  - Surface in "Revisi" mode
```

### 9.4 AI Tutor — RAG Architecture

```
User question input
  ↓
Intent classification (is it in-scope?)
  ↓ (in-scope)
Semantic search in kitab embeddings (top 5 relevant chunks)
  ↓
Construct context-augmented prompt:
  "Berdasarkan kitab berikut: [context]
   Jawab pertanyaan berikut dalam Bahasa Indonesia: [question]
   Jika tidak ada dalam konteks, katakan tidak tahu."
  ↓
OpenAI API call
  ↓
Response + citations (kitab/bab reference)
  ↓
Store in conversation history
  ↓
Rate limit check (20/day free)
```

---

## 10. Authentication & Authorization

### 10.1 Auth Flow

```
Registration:
  POST /api/v1/auth/register
  → Validate input
  → Check email unique
  → Hash password (bcrypt cost 12)
  → Create user record
  → Send verification email (queue)
  → Return: { message: "Cek email untuk verifikasi" }

Login:
  POST /api/v1/auth/login
  → Validate credentials
  → Check email verified
  → Generate access token (JWT, 15 menit)
  → Generate refresh token (UUID, 30 hari)
  → Store refresh token in DB (hashed) + Redis
  → Set refresh token in HTTP-only cookie
  → Return: { accessToken, user }

Token Refresh:
  POST /api/v1/auth/refresh
  → Read refresh token from cookie
  → Validate against DB
  → Rotate: delete old, issue new refresh token
  → Return: new access token

Logout:
  POST /api/v1/auth/logout
  → Invalidate refresh token
  → Add access token to blacklist (Redis, TTL = remaining TTL)
```

### 10.2 Authorization Middleware

```typescript
// RBAC dengan route-level guard
const authorize = (...roles: Role[]) => async (ctx: Context, next: Next) => {
  const user = ctx.get('user') // dari JWT middleware
  if (!roles.includes(user.role)) {
    throw new ForbiddenError()
  }
  return next()
}

// Resource-level ownership check
const ownsClassroom = async (ctx: Context, next: Next) => {
  const classroom = await classroomService.findById(ctx.param('id'))
  if (classroom.muallimId !== ctx.get('user').id) {
    throw new ForbiddenError()
  }
  ctx.set('classroom', classroom)
  return next()
}

// Usage
app.put('/api/v1/classroom/:id', authorize('muallim', 'admin'), ownsClassroom, handler)
```

---

## 11. Search Architecture

### 11.1 MVP: PostgreSQL Full-Text Search

```sql
-- Add tsvector column to questions
ALTER TABLE questions ADD COLUMN search_vector tsvector;

CREATE INDEX idx_questions_fts ON questions USING gin(search_vector);

-- Update trigger
CREATE TRIGGER questions_search_vector_update
  BEFORE INSERT OR UPDATE ON questions
  FOR EACH ROW EXECUTE FUNCTION
    tsvector_update_trigger(search_vector, 'pg_catalog.indonesian',
      question_text, explanation);

-- Search query
SELECT * FROM questions
WHERE search_vector @@ plainto_tsquery('indonesian', $1)
  AND status = 'approved'
ORDER BY ts_rank(search_vector, plainto_tsquery('indonesian', $1)) DESC;
```

### 11.2 Scale: Semantic Search dengan Embeddings

```typescript
// Generate embedding saat soal di-approve
const embedding = await openai.embeddings.create({
  model: 'text-embedding-3-small',
  input: question.questionText + ' ' + question.explanation,
})

// Store di pgvector extension
await db.execute(sql`
  INSERT INTO question_embeddings (question_id, embedding)
  VALUES (${questionId}, ${JSON.stringify(embedding.data[0].embedding)}::vector)
`)

// Semantic search
const results = await db.execute(sql`
  SELECT q.*, 1 - (qe.embedding <=> ${queryEmbedding}::vector) as similarity
  FROM questions q
  JOIN question_embeddings qe ON q.id = qe.question_id
  WHERE q.status = 'approved'
  ORDER BY similarity DESC
  LIMIT 10
`)
```

---

## 12. Folder Structure

```
talim-afkar/
├── apps/
│   ├── web/                          # Next.js frontend
│   │   ├── app/
│   │   │   ├── (auth)/              # Login, register pages
│   │   │   ├── (game)/              # Game mode pages
│   │   │   │   ├── kilat/
│   │   │   │   ├── tebak-kitab/
│   │   │   │   ├── duel/
│   │   │   │   ├── tangga-ilmu/
│   │   │   │   ├── urutan-dalil/
│   │   │   │   └── musabaqoh/
│   │   │   ├── (dashboard)/         # User dashboard
│   │   │   ├── (muallim)/           # Muallim features
│   │   │   ├── (admin)/             # Admin panel
│   │   │   └── layout.tsx
│   │   ├── components/
│   │   │   ├── ui/                  # shadcn/ui base components
│   │   │   ├── game/                # Game-specific components
│   │   │   ├── learning/            # Learning components
│   │   │   └── shared/              # Shared components
│   │   └── lib/
│   │       ├── api/                 # API client functions
│   │       └── hooks/               # Custom React hooks
│   │
│   └── api/                         # Hono.js backend
│       ├── src/
│       │   ├── modules/             # CORE: domain modules
│       │   │   ├── auth/
│       │   │   │   ├── auth.routes.ts
│       │   │   │   ├── auth.service.ts
│       │   │   │   ├── auth.repository.ts
│       │   │   │   └── auth.schema.ts   # Zod schemas
│       │   │   ├── user/
│       │   │   ├── game/
│       │   │   │   ├── engines/
│       │   │   │   │   ├── kilat-fiqih.engine.ts
│       │   │   │   │   ├── duel-afkar.engine.ts
│       │   │   │   │   └── musabaqoh.engine.ts
│       │   │   │   ├── game.service.ts
│       │   │   │   └── game.routes.ts
│       │   │   ├── content/
│       │   │   ├── learning/
│       │   │   ├── gamification/
│       │   │   ├── classroom/
│       │   │   ├── ai/
│       │   │   ├── analytics/
│       │   │   └── notification/
│       │   ├── shared/
│       │   │   ├── middleware/          # Auth, rate limit, logging
│       │   │   ├── events/              # Internal event bus
│       │   │   ├── queue/               # BullMQ workers
│       │   │   ├── cache/               # Redis utilities
│       │   │   └── errors/              # Error classes
│       │   ├── infrastructure/
│       │   │   ├── database/            # Drizzle setup, migrations
│       │   │   ├── redis/               # Redis client
│       │   │   └── storage/             # R2/S3 client
│       │   └── index.ts                 # App entry point
│       └── tests/
│           ├── unit/
│           ├── integration/
│           └── e2e/
│
├── packages/
│   ├── shared-types/                # TypeScript types shared between apps
│   ├── config/                      # Shared config (eslint, tsconfig)
│   └── utils/                       # Shared utilities
│
├── infrastructure/
│   ├── docker/
│   │   ├── docker-compose.yml       # Local development
│   │   └── docker-compose.prod.yml
│   ├── k8s/                         # Kubernetes manifests (Scale phase)
│   └── terraform/                   # IaC (Scale phase)
│
├── docs/                            # Project documentation
│   ├── srs.md
│   ├── design.md
│   ├── claude.md
│   ├── todo.md
│   └── changelog.md
│
├── .github/
│   └── workflows/                   # CI/CD pipelines
│
├── turbo.json                       # Turborepo config
├── package.json
└── README.md
```

**Tooling decisions:**
- **Monorepo:** Turborepo (simpler than Nx for this size)
- **ORM:** Drizzle (type-safe, performant, migration support)
- **Validation:** Zod (shared frontend-backend via shared-types)
- **Testing:** Vitest (unit) + Supertest (integration) + Playwright (E2E)

---

## 13. API Conventions

### 13.1 REST Conventions

```
GET    /api/v1/questions              # List (dengan pagination)
POST   /api/v1/questions              # Create
GET    /api/v1/questions/:id          # Get single
PUT    /api/v1/questions/:id          # Full update
PATCH  /api/v1/questions/:id          # Partial update
DELETE /api/v1/questions/:id          # Delete

# Nested resources
GET    /api/v1/kitabs/:id/babs        # Babs dalam kitab
POST   /api/v1/classrooms/:id/enroll  # Action endpoint

# Query conventions
?page=1&limit=20                      # Pagination
?sort=created_at&order=desc           # Sorting
?filter[status]=approved              # Filtering
?search=thaharah                      # Search
?include=kitab,bab                    # Eager loading
```

### 13.2 Response Format

```typescript
// Success Response
{
  "success": true,
  "data": { ... } | [...],
  "meta": {                    // untuk list responses
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Input tidak valid",
    "details": [
      { "field": "email", "message": "Email tidak valid" }
    ]
  }
}

// HTTP Status Codes:
// 200 OK, 201 Created, 204 No Content
// 400 Bad Request, 401 Unauthorized, 403 Forbidden
// 404 Not Found, 409 Conflict, 422 Unprocessable Entity
// 429 Too Many Requests, 500 Internal Server Error
```

---

## 14. Frontend Architecture

### 14.1 State Management

```
Server State:    TanStack Query (React Query) — caching, refetching, optimistic updates
Client State:    Zustand — game state, UI state, user preferences
Form State:      React Hook Form + Zod resolver
Realtime State:  Socket.io client + Zustand integration
```

### 14.2 Game UI Architecture

```typescript
// Game State Machine (per mode)
type GamePhase = 'idle' | 'loading' | 'countdown' | 'playing' | 'result' | 'review'

// Setiap game mode memiliki isolated state store
const useKilatFiqihStore = create<KilatFiqihState>((set, get) => ({
  phase: 'idle',
  questions: [],
  currentIndex: 0,
  score: 0,
  answers: [],
  timer: 15,
  // actions...
}))
```

### 14.3 Arabic Text Rendering

```css
/* Global Arabic font setup */
@import url('https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&display=swap');

.arabic-text {
  font-family: 'Amiri', serif;
  direction: rtl;
  font-size: 1.25rem;
  line-height: 2;
  unicode-bidi: bidi-override;
}
```

---

## 15. Deployment Architecture

### 15.1 MVP Deployment (Railway / Docker Compose)

```yaml
# docker-compose.yml (production)
services:
  api:
    image: talim-afkar/api:latest
    environment:
      - DATABASE_URL=...
      - REDIS_URL=...
    ports:
      - "3001:3001"
    depends_on:
      - postgres
      - redis

  ws:
    image: talim-afkar/api:latest
    command: node dist/ws-server.js
    ports:
      - "3002:3002"

  web:
    image: talim-afkar/web:latest
    ports:
      - "3000:3000"

  postgres:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
      - "443:443"
```

### 15.2 Environment Configuration

```
# .env.production (semua secrets via secret manager)
NODE_ENV=production
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
JWT_ACCESS_SECRET=...
JWT_REFRESH_SECRET=...
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
CLOUDFLARE_R2_BUCKET=...
CLOUDFLARE_R2_ENDPOINT=...
RESEND_API_KEY=...          # Email service
SENTRY_DSN=...
```

---

## 16. DevOps & CI/CD

### 16.1 CI Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci.yml
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test:unit
      - run: npm run test:integration
        env:
          DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: docker build -t talim-afkar/api .
      - run: docker build -t talim-afkar/web ./apps/web

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - run: # Deploy ke Railway/VPS
```

### 16.2 Branch Strategy

```
main          → Production deployment
develop       → Staging environment
feature/*     → Feature branches, PR ke develop
hotfix/*      → Emergency fixes, PR ke main
```

---

## 17. Monitoring & Observability

### 17.1 Metrics yang Di-monitor

```
Infrastructure:
  - CPU, memory, disk per service
  - Database connections, query time, slow queries
  - Redis memory, hit rate, operations/sec
  - WebSocket connections count

Application:
  - API response time (P50, P95, P99) per endpoint
  - Error rate per endpoint
  - Active game sessions
  - Queue depth per job type
  - AI API latency dan cost/hour

Business:
  - Active users (real-time)
  - Game sessions started/completed per hour
  - Leaderboard refresh rate
```

### 17.2 Alerting Rules

| Condition | Severity | Action |
|-----------|----------|--------|
| API P95 > 500ms selama 5 menit | Warning | Slack alert |
| API P95 > 2000ms selama 2 menit | Critical | PagerDuty + auto-scale |
| Error rate > 5% | Warning | Slack alert |
| Error rate > 20% | Critical | Immediate response |
| DB connections > 80% max | Warning | Slack alert |
| Redis memory > 80% | Warning | Evaluate cache policy |
| Queue depth > 1000 | Warning | Scale workers |

---

## 18. Storage Strategy

### 18.1 Object Storage (Cloudflare R2)

```
Bucket structure:
  talim-afkar-media/
  ├── avatars/
  │   └── {userId}/{filename}.webp
  ├── kitab-covers/
  │   └── {kitabId}/{filename}.webp
  ├── achievement-badges/
  │   └── {achievementId}.svg
  └── exports/
      └── reports/{date}/{filename}.pdf

Upload flow:
  1. Client request pre-signed URL dari API
  2. API generate pre-signed URL (valid 5 menit)
  3. Client upload langsung ke R2 (bypass API server)
  4. Client notify API: "Upload selesai, URL: ..."
  5. API update database dengan URL final
```

### 18.2 Database Backup

```
Backup Strategy:
  - Continuous WAL archiving ke S3/R2 (point-in-time recovery)
  - Daily full backup
  - Weekly backup disimpan 90 hari
  - Monthly backup disimpan 1 tahun

Restore Testing:
  - Monthly restore drill ke staging environment
  - RTO target: < 1 jam
  - RPO target: < 5 menit (WAL)
```

---

## 19. Scalability Roadmap

| Phase | Infra Changes |
|-------|--------------|
| MVP (< 1K MAU) | Single server, Postgres + Redis, Docker Compose |
| Growth (1K–10K MAU) | Separate API & WS servers, Read replica, Redis cluster |
| Scale (10K–100K MAU) | k8s, auto-scaling, CDN optimization, DB connection pooler (PgBouncer) |
| Enterprise (100K+ MAU) | Multi-region, microservices extraction untuk game engine, dedicated AI service |

---

## 20. Future Microservices Migration

### Extraction Priority (jika diperlukan)

```
Phase 1 extraction (scale triggers):
  1. Realtime/WebSocket Service — karena stateful, beda scaling pattern
  2. AI Service — karena biaya dan scaling terpisah

Phase 2 extraction:
  3. Notification Service — high volume async
  4. Analytics Service — beda read/write pattern

Tetap monolith (tidak perlu diextract):
  - Auth, User, Content, Gamification, Learning
  → Tidak ada alasan teknis kuat untuk dipisah

Migration Strategy:
  - Strangler Fig pattern
  - API gateway routing bertahap
  - Dual-write period untuk sync data
  - Feature flag untuk traffic shifting
```

### Anti-Pattern yang Dihindari

```
❌ JANGAN lakukan premature extraction:
   "Kita harus microservices dari awal karena skalabel"
   → Microservices tanpa skala adalah kompleksitas tanpa manfaat

✅ LAKUKAN:
   Modular monolith dengan domain boundaries ketat
   → Ketika tiba saatnya extract: batas sudah ada, tinggal pisah process
```
