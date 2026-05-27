# Ta'lim Afkar — Technical Architecture Design

**Version:** 1.1.0
**Status:** Living Document
**Last Updated:** 2026-05
**Arsitektur:** Fullstack Tradisional Terpisah (Client-Server)
**ADE:** Google Antigravity
**Related:** [`srs.md`](./srs.md) · [`todo.md`](./todo.md) · [`claude.md`](./claude.md) · [`changelog.md`](./changelog.md)

---

## Table of Contents

1. [High-Level Architecture](#1-high-level-architecture)
2. [Keputusan Arsitektur: Client-Server Terpisah](#2-keputusan-arsitektur-client-server-terpisah)
3. [Technology Stack](#3-technology-stack)
4. [Repository & Folder Structure](#4-repository--folder-structure)
5. [Domain Boundaries (API)](#5-domain-boundaries-api)
6. [Database Architecture](#6-database-architecture)
7. [Event-Driven Architecture (Internal)](#7-event-driven-architecture-internal)
8. [Cache Architecture](#8-cache-architecture)
9. [Realtime Multiplayer Architecture](#9-realtime-multiplayer-architecture)
10. [AI Service Architecture](#10-ai-service-architecture)
11. [Authentication & Authorization](#11-authentication--authorization)
12. [API Conventions](#12-api-conventions)
13. [Frontend Architecture](#13-frontend-architecture)
14. [API–Web Communication Contract](#14-apiweb-communication-contract)
15. [Deployment Architecture](#15-deployment-architecture)
16. [DevOps & CI/CD (Per Repository)](#16-devops--cicd-per-repository)
17. [Monitoring & Observability](#17-monitoring--observability)
18. [Scalability Roadmap](#18-scalability-roadmap)
19. [Antigravity ADE Configuration](#19-antigravity-ade-configuration)

---

## 1. High-Level Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                              │
│   ┌──────────────────────┐        ┌───────────────────────────┐   │
│   │  talim-afkar-web     │        │  Admin Panel (Next.js)    │   │
│   │  Next.js 14 (Vercel) │        │  talim-afkar-admin        │   │
│   │  PWA                 │        │  (future phase)           │   │
│   └──────────┬───────────┘        └────────────┬──────────────┘   │
└──────────────┼─────────────────────────────────┼──────────────────┘
               │ HTTPS REST + WebSocket           │ HTTPS REST
               │ (NEXT_PUBLIC_API_URL)            │
               ▼                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                       CDN (Cloudflare)                            │
│   Static assets · DDoS protection · WAF · Edge caching           │
└─────────────────────────────┬────────────────────────────────────┘
                               │
┌──────────────────────────────▼────────────────────────────────────┐
│                 talim-afkar-api (Node.js + Hono.js)               │
│                 Railway / VPS — Docker Container                   │
│                                                                    │
│  ┌────────────────┐   ┌──────────────────────────────────────┐    │
│  │  REST API      │   │    WebSocket Server (Socket.io)      │    │
│  │  Hono.js       │   │    Port 3002                         │    │
│  │  Port 3001     │   │    Duel Afkar + Musabaqoh rooms      │    │
│  └───────┬────────┘   └──────────────┬───────────────────────┘    │
│          │                           │                             │
│  ┌───────▼───────────────────────────▼───────────────────────┐    │
│  │               DOMAIN MODULES (Internal)                    │    │
│  │  Auth │ User │ Game │ Content │ Classroom │ Gamification   │    │
│  │  AI   │ Analytics │ Notification │ Musabaqoh │ Leaderboard │    │
│  └───────────────────────────┬───────────────────────────────┘    │
│                               │                                    │
│  ┌────────────────┐  ┌────────▼─────────┐  ┌──────────────────┐  │
│  │  PostgreSQL 16 │  │  Redis 7         │  │  BullMQ          │  │
│  │  Primary DB    │  │  Cache + Session │  │  Job Queue       │  │
│  └────────────────┘  │  + WS State      │  └──────────────────┘  │
│                       └──────────────────┘                        │
└────────────────────────────────────────────────────────────────────┘
                               │
               ┌───────────────┼───────────────┐
               ▼               ▼               ▼
       ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
       │  OpenAI /    │ │  Cloudflare  │ │  Resend      │
       │  Anthropic   │ │  R2 Storage  │ │  Email       │
       └──────────────┘ └──────────────┘ └──────────────┘
```

---

## 2. Keputusan Arsitektur: Client-Server Terpisah

### ADL-000: Separated Client-Server Architecture

```
Tanggal:    2026-05
Keputusan:  Fullstack Tradisional Terpisah — API dan Web adalah dua
            deployment unit independen.
Alasan:
  1. Deployment independent — API bisa di-scale tanpa menyentuh Web
  2. Team separation — tim backend dan frontend bekerja paralel tanpa konflik
  3. Clarity untuk AI Agents — Antigravity bekerja pada satu konteks repo
     sekaligus; pemisahan repo mencegah agen "merembet" lintas domain
  4. API-first — memudahkan mobile app (React Native) di masa depan
     untuk menggunakan API yang sama
  5. Deployment fleksibel — API ke Railway/VPS (stateful, WebSocket),
     Web ke Vercel (edge, CDN-optimized)
Tradeoff:
  - Tidak ada shared TypeScript types secara langsung (mitigasi: shared
    types package via npm private registry atau git submodule)
  - CORS harus dikonfigurasi eksplisit
  - Developer perlu menjalankan dua service saat development lokal
Mitigasi:
  - Package @talim-afkar/types di-publish ke npm private / GitHub Packages
  - docker-compose.dev.yml menyatukan semua infrastructure
  - Makefile / script untuk `make dev` yang menjalankan keduanya
Review:    Tidak direncanakan untuk diubah. Jika ada bottleneck spesifik,
           pertimbangkan API Gateway (tidak monorepo).
```

### Batasan Keras Arsitektur Ini

```
BATAS-01: Zero Direct DB Access dari Frontend
  talim-afkar-web TIDAK BOLEH mengakses PostgreSQL atau Redis secara langsung.
  Semua data HARUS melalui talim-afkar-api REST endpoints atau WebSocket.

BATAS-02: CORS Whitelist Ketat
  API hanya menerima request dari domain web yang terdaftar.
  Konfigurasi di: api/src/config/cors.config.ts

BATAS-03: JWT Stateless di API
  Token di-generate oleh API, divalidasi di API.
  Web hanya menyimpan token di memory (access) dan HTTP-only cookie (refresh).

BATAS-04: Shared Types via Package
  Duplikasi type definition DILARANG.
  Gunakan: @talim-afkar/types (GitHub Packages atau npm private)
  Berisi: API request/response types, WebSocket event types, enums

BATAS-05: Environment Variables Terpisah
  API:  api/.env  (DATABASE_URL, REDIS_URL, JWT_SECRET, dll)
  Web:  web/.env  (NEXT_PUBLIC_API_URL, NEXT_PUBLIC_WS_URL, dll)
  Tidak ada env var yang di-share secara langsung.
```

---

## 3. Technology Stack

### 3.1 Backend (talim-afkar-api)

| Layer           | Technology         | Versi  | Alasan                                    |
| --------------- | ------------------ | ------ | ----------------------------------------- |
| Runtime         | Node.js            | 20 LTS | Stable, excellent ecosystem               |
| Framework       | Hono.js            | latest | Ultra-fast, TypeScript-native, edge-ready |
| ORM             | Drizzle ORM        | latest | Type-safe, AI-friendly, performant        |
| Database        | PostgreSQL         | 16     | ACID, JSON, full-text search              |
| Cache / Session | Redis              | 7      | Sub-5ms latency untuk game state          |
| WebSocket       | Socket.io          | 4.x    | DX baik, Redis adapter, fallback          |
| Queue           | BullMQ             | latest | Redis-based, excellent DX                 |
| Validation      | Zod                | latest | TypeScript-native, composable             |
| Auth            | JWT + bcrypt       | —      | Stateless, standard                       |
| AI              | Vercel AI SDK      | latest | Multi-provider, streaming                 |
| Email           | Resend             | —      | Reliable, DX terbaik                      |
| Storage         | Cloudflare R2      | —      | No egress fee                             |
| Logging         | Pino               | latest | Structured JSON, performant               |
| Testing         | Vitest + Supertest | —      | Fast, ESM-native                          |

### 3.2 Frontend (talim-afkar-web)

| Layer          | Technology               | Versi  | Alasan                             |
| -------------- | ------------------------ | ------ | ---------------------------------- |
| Framework      | Next.js                  | 14+    | App Router, SSR/SSG, PWA ready     |
| Language       | TypeScript               | 5.x    | Type safety end-to-end             |
| Styling        | Tailwind CSS             | 3.x    | Utility-first, AI-agent friendly   |
| UI Components  | shadcn/ui                | latest | Composable, fully owned code       |
| Animation      | Framer Motion            | latest | Game-feel animations               |
| State (server) | TanStack Query           | latest | Cache, refetch, optimistic updates |
| State (client) | Zustand                  | latest | Minimal, performant, TypeScript    |
| WebSocket      | Socket.io Client         | 4.x    | Sinkron dengan server              |
| Form           | React Hook Form          | latest | Performant forms                   |
| Drag & Drop    | dnd-kit                  | latest | Urutan Dalil game mode             |
| Testing        | Vitest + Testing Library | —      | Fast, idiomatic React testing      |

### 3.3 Shared Types Package

```
@talim-afkar/types
  Dipublish ke: GitHub Packages (private)

  Berisi:
    - API request/response interfaces
    - WebSocket event types
    - Enum definitions (GameMode, UserRole, dll)
    - Zod schemas yang di-share (opsional)

  Versioning: Semantic versioning ketat
  Update policy: Breaking changes = major version bump
```

---

## 4. Repository & Folder Structure

### 4.1 talim-afkar-api

```
talim-afkar-api/
├── .agents/                          ← Antigravity configuration
│   ├── rules/
│   │   ├── code-style-guide.md
│   │   ├── api-conventions.md
│   │   ├── domain-boundaries.md
│   │   └── security-rules.md
│   ├── workflows/
│   │   ├── new-endpoint.md
│   │   ├── new-schema.md
│   │   ├── generate-tests.md
│   │   └── update-changelog.md
│   └── skills/
│       ├── drizzle-schema/SKILL.md
│       ├── hono-endpoint/SKILL.md
│       └── bullmq-worker/SKILL.md
│
├── src/
│   ├── app.ts                        ← Hono app entry point
│   ├── server.ts                     ← HTTP server
│   ├── ws-server.ts                  ← Socket.io server (port 3002)
│   │
│   ├── config/
│   │   ├── env.config.ts             ← Zod-validated env vars
│   │   ├── cors.config.ts            ← CORS whitelist
│   │   └── redis.config.ts
│   │
│   ├── shared/
│   │   ├── errors/                   ← AppError, NotFoundError, dll
│   │   ├── middleware/               ← auth, rateLimit, requestId
│   │   ├── events/
│   │   │   └── event-bus.ts          ← Internal EventEmitter bus
│   │   ├── jobs/
│   │   │   └── queue.ts              ← BullMQ queue definitions
│   │   └── utils/
│   │
│   └── modules/
│       ├── auth/
│       │   ├── auth.routes.ts
│       │   ├── auth.service.ts
│       │   ├── auth.schema.ts        ← Zod schemas
│       │   └── auth.service.test.ts
│       ├── user/
│       ├── game/
│       │   ├── engines/
│       │   │   ├── kilat-fiqih.engine.ts
│       │   │   ├── tebak-kitab.engine.ts
│       │   │   ├── duel-afkar.engine.ts
│       │   │   ├── tangga-ilmu.engine.ts
│       │   │   ├── urutan-dalil.engine.ts
│       │   │   └── musabaqoh.engine.ts
│       │   ├── game.routes.ts
│       │   ├── game.service.ts
│       │   └── game.repository.ts
│       ├── content/                  ← Kitab, Bab, Questions
│       ├── classroom/
│       ├── gamification/
│       │   ├── xp.service.ts
│       │   ├── streak.service.ts
│       │   ├── leaderboard.service.ts
│       │   └── achievement.service.ts
│       ├── ai/
│       │   ├── prompts/
│       │   ├── ai-tutor.service.ts
│       │   └── question-gen.service.ts
│       ├── analytics/
│       ├── notification/
│       └── admin/
│
├── db/
│   ├── schema/
│   │   ├── users.schema.ts
│   │   ├── game.schema.ts
│   │   ├── content.schema.ts
│   │   └── index.ts
│   ├── migrations/
│   └── seed/
│
├── Dockerfile
├── docker-compose.dev.yml            ← PostgreSQL + Redis + MailHog
├── .env.example
├── drizzle.config.ts
├── package.json
└── tsconfig.json
```

### 4.2 talim-afkar-web

```
talim-afkar-web/
├── .agents/                          ← Antigravity configuration
│   ├── rules/
│   │   ├── component-conventions.md
│   │   ├── state-management.md
│   │   └── accessibility-rules.md
│   ├── workflows/
│   │   ├── new-page.md
│   │   ├── new-component.md
│   │   └── update-changelog.md
│   └── skills/
│       ├── next-page/SKILL.md
│       ├── game-component/SKILL.md
│       └── socket-client/SKILL.md
│
├── src/
│   ├── app/                          ← Next.js App Router
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   ├── register/page.tsx
│   │   │   └── verify-email/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── profile/page.tsx
│   │   │   └── leaderboard/page.tsx
│   │   ├── (game)/
│   │   │   ├── kilat-fiqih/page.tsx
│   │   │   ├── tebak-kitab/page.tsx
│   │   │   ├── duel-afkar/page.tsx
│   │   │   ├── tangga-ilmu/page.tsx
│   │   │   ├── urutan-dalil/page.tsx
│   │   │   └── musabaqoh/[code]/page.tsx
│   │   ├── (muallim)/
│   │   │   ├── classroom/page.tsx
│   │   │   └── ai-questions/page.tsx
│   │   └── layout.tsx
│   │
│   ├── components/
│   │   ├── ui/                       ← shadcn/ui components
│   │   ├── game/                     ← Game-specific components
│   │   │   ├── QuestionCard.tsx
│   │   │   ├── CountdownTimer.tsx
│   │   │   ├── ScoreDisplay.tsx
│   │   │   └── LiveLeaderboard.tsx
│   │   ├── layout/                   ← Navbar, Sidebar, Footer
│   │   └── shared/
│   │
│   ├── lib/
│   │   ├── api-client.ts             ← Centralized API client (fetch wrapper)
│   │   ├── socket-client.ts          ← Socket.io client singleton
│   │   └── query-client.ts           ← TanStack Query config
│   │
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useGameSession.ts
│   │   ├── useSocket.ts
│   │   └── useLeaderboard.ts
│   │
│   ├── stores/
│   │   ├── auth.store.ts             ← Zustand auth state
│   │   └── game.store.ts             ← Zustand active game state
│   │
│   └── types/                        ← Re-export dari @talim-afkar/types
│
├── public/
│   ├── manifest.json                 ← PWA manifest
│   └── icons/
│
├── .env.example
├── Dockerfile
├── next.config.ts
├── package.json
└── tsconfig.json
```

---

## 5. Domain Boundaries (API)

### Domain Map

```
┌─────────────────────────────────────────────────────────────────┐
│                    talim-afkar-api Domains                       │
│                                                                   │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌─────────────┐  │
│  │   AUTH    │  │   USER    │  │  CONTENT  │  │  CLASSROOM  │  │
│  │ auth.*    │  │ user.*    │  │ content.* │  │ classroom.* │  │
│  │ tokens    │  │ profiles  │  │ kitab     │  │ enrollment  │  │
│  │ sessions  │  │ stats     │  │ questions │  │ codes       │  │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └──────┬──────┘  │
│        │               │               │                │         │
│        └───────────────┴───────────────┴────────────────┘         │
│                               │ Events via EventBus               │
│        ┌──────────────────────┴───────────────────────┐           │
│        ▼                      ▼                       ▼           │
│  ┌───────────┐  ┌──────────────────┐  ┌───────────────────────┐  │
│  │   GAME    │  │  GAMIFICATION    │  │    MUSABAQOH          │  │
│  │ engines   │  │ xp, streak       │  │    WS rooms           │  │
│  │ sessions  │  │ leaderboard      │  │    live sessions      │  │
│  │ scoring   │  │ achievements     │  │                       │  │
│  └─────┬─────┘  └──────────────────┘  └───────────────────────┘  │
│        │                                                           │
│        ▼                                                           │
│  ┌───────────┐  ┌───────────┐  ┌─────────────┐  ┌────────────┐  │
│  │    AI     │  │ ANALYTICS │  │NOTIFICATION │  │   ADMIN    │  │
│  │ tutor     │  │ learning  │  │ in-app      │  │ super admin│  │
│  │ gen soal  │  │ product   │  │ email       │  │ moderation │  │
│  └───────────┘  └───────────┘  └─────────────┘  └────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Aturan Komunikasi Antar Domain

```typescript
// ✅ BENAR — Domain A memanggil public interface Domain B
// game.service.ts memanggil user via injected service
const user = await this.userService.findById(userId)

// ✅ BENAR — Cross-domain side effects via Event Bus
// Di dalam game.service.ts
this.eventBus.emit('game:session_completed', {
  userId, mode, score, xpEarned, sessionId
})

// Gamification module mendengarkan
eventBus.on('game:session_completed', async (payload) => {
  await xpService.award(payload.userId, payload.xpEarned)
  await streakService.record(payload.userId)
  await achievementService.checkTriggers(payload.userId, payload)
})

// ❌ SALAH — Direct import lintas domain
import { UserRepository } from '../user/user.repository'  // DILARANG

// ❌ SALAH — Cross-domain DB query langsung
db.select().from(users).where(...)  // DILARANG di luar modul user
```

### Event Bus Contract

| Event                    | Producer      | Consumers                               |
| ------------------------ | ------------- | --------------------------------------- |
| `game:session_completed` | Game          | Gamification, Analytics, Notification   |
| `game:duel_result`       | Game (Duel)   | Gamification (ELO), Notification        |
| `user:registered`        | Auth          | Notification (welcome email), Analytics |
| `user:streak_broken`     | Gamification  | Notification                            |
| `achievement:unlocked`   | Gamification  | Notification                            |
| `musabaqoh:ended`        | Musabaqoh     | Analytics, Notification                 |
| `question:approved`      | Content/Admin | Notification (ke muallim)               |
| `ai:question_ready`      | AI            | Notification (ke muallim)               |

---

## 6. Database Architecture

### 6.1 Schema Overview (Drizzle ORM)

```typescript
// Ownership per domain (setiap tabel "dimiliki" satu modul)

// AUTH domain
users              → id, email, password_hash, role, email_verified, created_at
refresh_tokens     → id, user_id, token_hash, expires_at, revoked_at

// USER domain
profiles           → user_id, display_name, avatar_url, institution_id, level, bio
user_stats         → user_id, total_xp, current_streak, max_streak, games_played
xp_logs            → id, user_id, amount, source, game_session_id, created_at
institutions       → id, name, type, location, invite_code

// CONTENT domain
kitabs             → id, name, author, category, description, cover_url, is_active
babs               → id, kitab_id, name, order_index, description
questions          → id, bab_id, type, text, options_json, correct_answer,
                     explanation, difficulty, status, created_by, reviewed_by
matan_excerpts     → id, bab_id, arabic_text, translation, source_page

// GAME domain
game_sessions      → id, user_id, mode, status, score, xp_earned,
                     started_at, completed_at, metadata_json
game_answers       → id, session_id, question_id, answer, is_correct,
                     time_taken_ms, score_awarded, answered_at
learning_progressions → user_id, bab_id, stars, completed_at, attempts

// CLASSROOM domain
classrooms         → id, muallim_id, name, institution_id, invite_code, is_active
classroom_members  → classroom_id, user_id, joined_at, role

// GAMIFICATION domain
achievements       → id, key, name, description, icon_url, xp_reward, conditions_json
user_achievements  → user_id, achievement_id, unlocked_at, progress_json

// MUSABAQOH domain
musabaqoh_sessions → id, classroom_id, host_id, status, config_json,
                     started_at, ended_at
musabaqoh_results  → id, session_id, user_id, score, rank, answers_json

// NOTIFICATION domain
notifications      → id, user_id, type, title, body, read_at, metadata_json, created_at

// ANALYTICS domain
daily_activity     → user_id, date, games_played, xp_earned, time_spent_sec
```

### 6.2 Redis Key Schema

```
# Game Real-time State
game:session:{sessionId}:state        → JSON (current question, timer)
game:duel:{duelId}:state              → JSON (both players, answers)
musabaqoh:room:{roomCode}:state       → JSON (participants, scores, current Q)
musabaqoh:room:{roomCode}:participants → Hash

# Leaderboard (sorted sets)
lb:global:weekly:{weekKey}            → ZADD userId score
lb:classroom:{classroomId}:{weekKey}  → ZADD userId score
lb:institution:{institutionId}:{weekKey} → ZADD userId score

# Auth
auth:refresh:{userId}                 → token_hash (TTL: 30 hari)
auth:blacklist:{jti}                  → "1" (TTL: 15 menit)
auth:ratelimit:login:{ip}             → counter (TTL: 15 menit)

# Cache
cache:questions:{babId}:{difficulty}  → JSON array (TTL: 1 jam)
cache:leaderboard:global              → JSON array (TTL: 5 menit)
cache:kitabs:all                      → JSON array (TTL: 24 jam)
cache:ai:question:{hash}              → JSON (TTL: 24 jam)

# Matchmaking
matchmaking:queue:{mode}:{eloRange}   → Sorted Set (userId, timestamp)
```

---

## 7. Event-Driven Architecture (Internal)

### Implementation

```typescript
// src/shared/events/event-bus.ts
import { EventEmitter } from "events";

class TypedEventBus extends EventEmitter {
  emit<K extends keyof AppEvents>(event: K, payload: AppEvents[K]): boolean {
    return super.emit(event, payload);
  }
  on<K extends keyof AppEvents>(
    event: K,
    listener: (payload: AppEvents[K]) => void,
  ) {
    return super.on(event, listener);
  }
}

// AppEvents type definition (dari @talim-afkar/types)
export interface AppEvents {
  "game:session_completed": GameSessionCompletedEvent;
  "game:duel_result": DuelResultEvent;
  "user:registered": UserRegisteredEvent;
  "user:streak_broken": StreakBrokenEvent;
  "achievement:unlocked": AchievementUnlockedEvent;
  "musabaqoh:ended": MusabaqohEndedEvent;
  "question:approved": QuestionApprovedEvent;
  "ai:question_ready": AIQuestionReadyEvent;
}

export const eventBus = new TypedEventBus();
eventBus.setMaxListeners(50);
```

---

## 8. Cache Architecture

### Cache Strategy per Data Type

| Data                | TTL                    | Invalidation              | Strategi      |
| ------------------- | ---------------------- | ------------------------- | ------------- |
| Questions per bab   | 1 jam                  | On question CRUD          | Cache-aside   |
| Leaderboard global  | 5 menit                | Scheduled rebuild         | Read-through  |
| User profile        | 15 menit               | On profile update         | Cache-aside   |
| Kitab list          | 24 jam                 | On kitab update           | Cache-aside   |
| AI-generated answer | 24 jam                 | Never (content-addressed) | Write-through |
| Game session state  | TTL = session duration | On session end            | Write-through |

---

## 9. Realtime Multiplayer Architecture

### Socket.io Server (Port 3002)

```typescript
// src/ws-server.ts — standalone dari REST API server

// Rooms
`musabaqoh:${roomCode}`      → semua peserta musabaqoh
`duel:${duelId}`             → dua player duel
`user:${userId}`             → notifikasi personal (inbox)

// Auth: JWT via handshake query
io.use(async (socket, next) => {
  const token = socket.handshake.auth.token
  const payload = await verifyAccessToken(token)
  socket.data.userId = payload.userId
  socket.data.role = payload.role
  next()
})
```

### WebSocket Events Contract

```typescript
// Musabaqoh Events (Server → Client)
'musabaqoh:joined'           → { participants: Participant[] }
'musabaqoh:started'          → { firstQuestion: Question, timer: number }
'musabaqoh:question'         → { question: Question, questionIndex: number, timer: number }
'musabaqoh:answer_result'    → { isCorrect: boolean, correctAnswer: string, scoreEarned: number }
'musabaqoh:leaderboard'      → { rankings: Ranking[] }
'musabaqoh:ended'            → { finalRankings: Ranking[], stats: SessionStats }
'musabaqoh:error'            → { code: string, message: string }

// Musabaqoh Events (Client → Server)
'musabaqoh:join'             → { roomCode: string }
'musabaqoh:submit_answer'    → { questionId: string, answer: string, timestamp: number }
'musabaqoh:host:start'       → { config: MusabaqohConfig }
'musabaqoh:host:next'        → {}
'musabaqoh:host:end'         → {}

// Duel Events (Server → Client)
'duel:matched'               → { opponent: UserProfile, duelId: string }
'duel:question'              → { question: Question, timer: number }
'duel:opponent_answered'     → { isCorrect: boolean } // tanpa reveal jawaban lawan
'duel:round_result'          → { yourScore: number, opponentScore: number }
'duel:ended'                 → { winner: string, eloChange: number, finalScore: Score }

// Duel Events (Client → Server)
'duel:queue'                 → { mode: 'ranked' | 'casual' }
'duel:cancel_queue'          → {}
'duel:submit_answer'         → { questionId: string, answer: string, timestamp: number }
```

### Anti-Cheat WebSocket

```typescript
// Server-side: validasi timestamp jawaban
const ANSWER_MIN_MS = 500; // < 500ms = bot/cheat
const ANSWER_MAX_MS = (timerDuration + 2) * 1000; // +2 detik toleransi network

function validateAnswerTimestamp(
  questionStartedAt: number,
  answerTimestamp: number,
): boolean {
  const elapsed = answerTimestamp - questionStartedAt;
  return elapsed >= ANSWER_MIN_MS && elapsed <= ANSWER_MAX_MS;
}
```

---

## 10. AI Service Architecture

### AI Pipeline

```
Muallim Request                        BullMQ Queue
"Generate soal dari matan ini"  →→→→  ai:generate_question job
                                            ↓
                               AI Worker (pulls job)
                                            ↓
                            Prompt Template + Matan Input
                                            ↓
                               OpenAI GPT-4o (primary)
                               Anthropic Claude (fallback)
                                            ↓
                              Zod Validate AI Response
                                            ↓
                         Save to questions (status: ai_draft)
                                            ↓
                         Emit: ai:question_ready event
                                            ↓
                         Notify muallim → Review Queue UI
```

### AI Tutor (Synchronous, Max 10 detik)

```
User: "Mengapa jawaban ini benar?"
  ↓
Retrieve context: question + kitab excerpt (RAG dari pgvector)
  ↓
Prompt: system context + question + user's answer + correct answer + retrieved text
  ↓
Streaming response (Vercel AI SDK)
  ↓
Rate limit check (20 Q/hari free tier)
  ↓
Log usage untuk analytics
```

---

## 11. Authentication & Authorization

### Token Flow

```
Login Request
     ↓
Validate credentials (bcrypt compare)
     ↓
Generate access_token (JWT, 15 menit, RS256)
Generate refresh_token (opaque random, 30 hari)
     ↓
Response:
  - Body: { access_token, user }
  - Set-Cookie: refresh_token (HTTP-only, Secure, SameSite=Strict)
     ↓
Client: simpan access_token di memory (Zustand)
        refresh_token auto di cookie
     ↓
Setiap request: Authorization: Bearer {access_token}
     ↓
Jika 401: auto-call POST /api/v1/auth/refresh
  → Kirim refresh cookie → dapat access_token baru
```

### Role-Based Authorization Middleware

```typescript
// src/shared/middleware/auth.middleware.ts

// Usage di routes:
app.get(
  "/api/v1/admin/users",
  authenticate, // verifikasi JWT
  authorize("super_admin"), // cek role
  adminController.getUsers,
);

app.post(
  "/api/v1/classroom",
  authenticate,
  authorize("muallim", "admin_institusi", "super_admin"),
  classroomController.create,
);
```

---

## 12. API Conventions

### Base URL

```
Production:  https://api.talimafkar.id
Staging:     https://api-staging.talimafkar.id
Development: http://localhost:3001
```

### Versioning

```
/api/v1/...  → Production stable
/api/v2/...  → Next version (parallel deployment saat migration)
```

### Request / Response Format

```typescript
// Standard Success Response
{
  "success": true,
  "data": { ... },
  "meta": {           // optional, untuk pagination
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}

// Standard Error Response
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",         // machine-readable
    "message": "Soal tidak ditemukan",  // human-readable
    "details": { ... }           // optional, Zod validation errors
  }
}

// Pagination Query Params (standard)
GET /api/v1/questions?page=1&limit=20&sort=created_at&order=desc
```

### Endpoint Naming

```
GET    /api/v1/questions          → list (paginated)
POST   /api/v1/questions          → create
GET    /api/v1/questions/:id      → single item
PUT    /api/v1/questions/:id      → full update
PATCH  /api/v1/questions/:id      → partial update
DELETE /api/v1/questions/:id      → delete
POST   /api/v1/questions/:id/approve  → action (verb as last segment)
```

### HTTP Status Codes

| Code | Usage                                           |
| ---- | ----------------------------------------------- |
| 200  | Success GET, PUT, PATCH                         |
| 201  | Success POST (created)                          |
| 204  | Success DELETE (no body)                        |
| 400  | Validation error                                |
| 401  | Not authenticated                               |
| 403  | Forbidden (authenticated tapi tidak authorized) |
| 404  | Resource not found                              |
| 409  | Conflict (duplicate, state conflict)            |
| 429  | Rate limited                                    |
| 500  | Server error                                    |

---

## 13. Frontend Architecture

### State Management Strategy

```
Server State:   TanStack Query
  → API data, caching, background refetch, pagination
  → Semua data yang berasal dari API

Client State:   Zustand
  → Auth state (user, token)
  → Active game state (current session, answers)
  → UI preferences (theme, settings)

WebSocket State: Custom hook (useSocket)
  → Real-time game state (musabaqoh, duel)
  → Connection status
  → Event handlers
```

### API Client

```typescript
// src/lib/api-client.ts
// Centralized client — semua API calls HARUS melalui ini

class APIClient {
  private baseURL = process.env.NEXT_PUBLIC_API_URL;

  async get<T>(path: string): Promise<T>;
  async post<T>(path: string, body: unknown): Promise<T>;
  async put<T>(path: string, body: unknown): Promise<T>;
  async delete<T>(path: string): Promise<T>;

  // Auto-refresh token on 401
  // Auto-attach Authorization header
  // Standard error handling
}
```

### Arabic Text Rendering

```css
/* globals.css */
.arabic-text {
  font-family: "Scheherazade New", "Amiri", serif;
  font-size: 1.4rem;
  line-height: 2.2;
  direction: rtl;
  unicode-bidi: bidi-override;
}

/* Minimum 16px untuk teks konten */
/* Kontras minimum 4.5:1 (WCAG 2.1 AA) */
```

---

## 14. API–Web Communication Contract

### CORS Configuration (API)

```typescript
// src/config/cors.config.ts
const allowedOrigins = [
  process.env.WEB_URL, // https://talimafkar.id
  process.env.ADMIN_URL, // https://admin.talimafkar.id
  ...(isDevelopment ? ["http://localhost:3000"] : []),
];

app.use(
  cors({
    origin: allowedOrigins,
    credentials: true, // Untuk cookies (refresh token)
    allowedHeaders: ["Content-Type", "Authorization"],
    methods: ["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"],
  }),
);
```

### Environment Variables

**API (.env)**

```bash
NODE_ENV=production
PORT=3001
WS_PORT=3002

DATABASE_URL=postgresql://user:pass@host:5432/talimafkar
REDIS_URL=redis://host:6379

JWT_ACCESS_SECRET=...
JWT_REFRESH_SECRET=...

OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...

CLOUDFLARE_R2_ENDPOINT=...
CLOUDFLARE_R2_ACCESS_KEY=...
CLOUDFLARE_R2_SECRET_KEY=...
CLOUDFLARE_R2_BUCKET=talim-afkar-media

RESEND_API_KEY=...
SENTRY_DSN=...

# CORS
WEB_URL=https://talimafkar.id
ADMIN_URL=https://admin.talimafkar.id
```

**Web (.env)**

```bash
NEXT_PUBLIC_API_URL=https://api.talimafkar.id
NEXT_PUBLIC_WS_URL=wss://api.talimafkar.id

# Tidak ada secret di sini — semua ke API
SENTRY_DSN=...  # Optional, untuk frontend error tracking
```

---

## 15. Deployment Architecture

### MVP: Independent Container Deployments

```
┌──────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT TOPOLOGY                        │
│                                                               │
│  talim-afkar-web          talim-afkar-api                    │
│  ┌───────────────┐        ┌──────────────────────────────┐   │
│  │ Vercel        │  HTTPS │ Railway / Hetzner VPS        │   │
│  │ Edge Network  │ ──────▶│ Docker Container             │   │
│  │ Auto CDN      │  WSS   │                              │   │
│  │ HTTPS auto    │ ──────▶│ api (port 3001)              │   │
│  └───────────────┘        │ ws  (port 3002)              │   │
│                            └──────────────────────────────┘   │
│                                        │                       │
│                            ┌───────────▼─────────────────┐    │
│                            │ PostgreSQL 16                │    │
│                            │ Redis 7 (AOF persistence)   │    │
│                            └────────────────────────────-─┘    │
└──────────────────────────────────────────────────────────────┘
```

### Docker Configuration (API)

```dockerfile
# api/Dockerfile
FROM node:20-alpine AS base

FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM base AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/db ./db

EXPOSE 3001 3002
CMD ["node", "dist/server.js"]
```

```yaml
# api/docker-compose.dev.yml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: talimafkar_dev
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: devpassword
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    ports: ["6379:6379"]
    volumes: [redisdata:/data]

  mailhog:
    image: mailhog/mailhog
    ports: ["1025:1025", "8025:8025"]

volumes:
  pgdata:
  redisdata:
```

### Deployment Checklist (per service)

**API Deploy:**

```bash
# 1. Run migrations
npm run db:migrate

# 2. Health check endpoint available
GET /health → { status: "ok", db: "ok", redis: "ok" }

# 3. Environment vars tersedia di Railway/VPS
# 4. CORS whitelist sudah include domain web
# 5. Rate limiting aktif
```

**Web Deploy (Vercel):**

```bash
# 1. NEXT_PUBLIC_API_URL sudah diset ke production API
# 2. NEXT_PUBLIC_WS_URL sudah diset ke production WS
# 3. Build berhasil: npm run build
# 4. No TypeScript errors
```

---

## 16. DevOps & CI/CD (Per Repository)

### API CI/CD (GitHub Actions)

```yaml
# .github/workflows/api-ci.yml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: test, POSTGRES_DB: talimafkar_test }
      redis:
        image: redis:7
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20" }
      - run: npm ci
      - run: npm run lint && npm run type-check
      - run: npm run test:unit
      - run: npm run test:integration
        env:
          { DATABASE_URL: postgresql://postgres:test@localhost/talimafkar_test }

  deploy-staging:
    needs: test
    if: github.ref == 'refs/heads/develop'
    steps:
      - run: # Deploy ke Railway staging environment

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - run: npm run db:migrate # Run migrations sebelum deploy
      - run: # Deploy ke Railway production
```

### Web CI/CD (Vercel — Auto)

```
Vercel auto-deploy via GitHub integration:
  - PR → Preview deployment (unique URL)
  - develop branch → Staging deployment
  - main branch → Production deployment

Manual checks sebelum merge ke main:
  - npm run build (no errors)
  - npm run type-check
  - Lighthouse score: Performance > 85, Accessibility > 90
```

---

## 17. Monitoring & Observability

### Health Check Endpoint (API)

```typescript
// GET /health
{
  "status": "ok",
  "timestamp": "2026-05-01T10:00:00Z",
  "version": "1.2.3",
  "checks": {
    "database": "ok",          // PostgreSQL connection
    "redis": "ok",             // Redis connection
    "queue": "ok",             // BullMQ connectivity
    "websocket": "ok"          // Socket.io active connections count
  },
  "metrics": {
    "activeWsConnections": 42,
    "queueDepth": { "ai": 3, "email": 0 },
    "dbPoolActive": 5,
    "dbPoolIdle": 15
  }
}
```

### Alerting Rules

| Condition                                | Severity | Action                  |
| ---------------------------------------- | -------- | ----------------------- |
| API P95 > 500ms (5 menit)                | Warning  | Slack alert             |
| API error rate > 5%                      | Warning  | Slack alert             |
| API error rate > 20%                     | Critical | PagerDuty               |
| PostgreSQL CPU > 80%                     | Warning  | Slack alert             |
| Redis memory > 80%                       | Warning  | Scale / eviction review |
| BullMQ queue depth > 500                 | Warning  | Scale workers           |
| WebSocket dropped connections > 10/menit | Warning  | Investigate             |
| Web Vercel deployment failed             | Critical | Slack alert             |

---

## 18. Scalability Roadmap

| Phase      | MAU      | Infra Changes                                        | Trigger                   |
| ---------- | -------- | ---------------------------------------------------- | ------------------------- |
| MVP        | < 1K     | Single VPS, Docker Compose, Vercel free              | Sekarang                  |
| Growth     | 1K–10K   | Separate WS container, Redis persistence, DB backup  | MAU > 1K                  |
| Scale      | 10K–100K | k8s, Read replica, PgBouncer, Redis Cluster          | API P95 > 300ms sustained |
| Enterprise | 100K+    | Multi-region, Extract WS service, Extract AI service | MAU > 100K                |

### Scaling Path yang Tidak Mengubah Web

Salah satu keuntungan arsitektur terpisah: Web (Vercel) scale otomatis.
Yang perlu di-scale manual hanya API (stateful, WebSocket, DB).

---

## 19. Antigravity ADE Configuration

### Rules untuk API Agent

```markdown
# api/.agents/rules/domain-boundaries.md

- Setiap modul HANYA boleh query tabel yang dimilikinya
- Cross-domain communication WAJIB via eventBus atau service interface
- Tidak ada import langsung antar modul (hanya via dependency injection)
- Semua external input WAJIB divalidasi dengan Zod sebelum diproses
```

```markdown
# api/.agents/rules/api-conventions.md

- Semua route handler WAJIB menggunakan async/await
- Semua response WAJIB menggunakan format standard (success/error)
- Semua endpoint WAJIB ada validasi Zod untuk request body
- Rate limiting WAJIB untuk semua auth endpoints
- Tidak ada business logic di route handler — semua ke service
```

### Workflows (Antigravity `/` commands)

```markdown
# api/.agents/workflows/new-endpoint.md

Saat membuat endpoint baru:

1. Buat route handler di modules/[domain]/[domain].routes.ts
2. Buat/update service method di modules/[domain]/[domain].service.ts
3. Buat Zod schema di modules/[domain]/[domain].schema.ts
4. Tulis unit test di modules/[domain]/[domain].service.test.ts
5. Update srs.md section 22 dengan endpoint baru
6. Update changelog.md section [Unreleased]
```

```markdown
# web/.agents/workflows/new-page.md

Saat membuat page baru:

1. Buat file di src/app/[route]/page.tsx
2. Tambahkan API hook di src/hooks/ (jika butuh data)
3. Gunakan api-client.ts untuk semua API calls (tidak boleh fetch langsung)
4. Tambahkan loading state dan error state
5. Pastikan teks Arab menggunakan class arabic-text
6. Test responsiveness di mobile viewport (priority)
```

### Browser Antigravity (Sub-Agen)

Izinkan akses ke domain berikut untuk referensi dokumentasi:

```
# api/.agents/browser-allowlist.txt
docs.hono.dev
orm.drizzle.team
socket.io
docs.bullmq.io
zod.dev
github.com (untuk @talim-afkar/types)

# web/.agents/browser-allowlist.txt
nextjs.org
tanstack.com
zustand.pmnd.rs
ui.shadcn.com
framer.com/motion
```
