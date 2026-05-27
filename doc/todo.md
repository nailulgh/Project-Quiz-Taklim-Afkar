# Ta'lim Afkar — Implementation Roadmap

**Version:** 1.1.0
**Last Updated:** 2026-05
**Arsitektur:** Fullstack Tradisional Terpisah (Client-Server)
**ADE:** Google Antigravity
**Related:** [`srs.md`](./srs.md) · [`design.md`](./design.md) · [`claude.md`](./claude.md) · [`changelog.md`](./changelog.md)

---

## Status Legend

```
[ ]  Not started
[~]  In progress (tambahkan: siapa/agen yang mengerjakan)
[x]  Completed
[!]  Blocked / needs attention (tambahkan: alasannya)
[-]  Deferred / cancelled (tambahkan: alasannya)
```

## Priority Legend

```
P0  Critical path — blocker untuk semua hal lain
P1  High priority — harus di fase ini
P2  Medium priority — dijadwalkan tapi bisa geser
P3  Low priority — nice to have
```

## Risk Level

```
🔴 High   — teknologi baru, kompleks, atau dependencies kritis
🟡 Medium — familiar tapi perlu perhatian
🟢 Low    — straightforward, well-understood
```

## Repo Label

```
[API]  → Dikerjakan di talim-afkar-api
[WEB]  → Dikerjakan di talim-afkar-web
[SHARED] → Dikerjakan di @talim-afkar/types (shared package)
[INFRA]  → Infrastructure / DevOps
```

---

## MVP (Minggu 1–8)

### Target: Platform bisa dipakai untuk satu kelas pesantren

**MVP Success Criteria:**

- ≥ 1 muallim dapat membuat kelas dan hosting Musabaqoh
- ≥ 20 mahasantri dapat bermain Kilat Fiqih dan melihat leaderboard
- Data tersimpan dengan benar, tidak ada data loss
- Platform berjalan stabil 8 jam tanpa crash
- API dan Web bisa di-deploy secara terpisah dan independen

---

### MINGGU 1–2: Fondasi Infrastructure & Auth

#### INFRA-01: Project Setup (DUAL REPO)

- [ ] Inisialisasi repository `talim-afkar-api` `P0` `🟢` `[API]`
  - Setup Node.js + Hono.js + TypeScript strict mode
  - Setup ESLint + Prettier + Husky pre-commit hooks
  - Setup Vitest untuk testing
  - Struktur folder sesuai design.md section 4.1
  - **Estimasi:** S (3 jam)

- [ ] Inisialisasi repository `talim-afkar-web` `P0` `🟢` `[WEB]`
  - Setup Next.js 14 App Router + TypeScript strict mode
  - Setup ESLint + Prettier + Husky
  - Setup TanStack Query + Zustand
  - Setup Tailwind CSS + shadcn/ui
  - Struktur folder sesuai design.md section 4.2
  - **Estimasi:** S (3 jam)

- [ ] Setup `@talim-afkar/types` package `P0` `🟢` `[SHARED]`
  - GitHub Package atau npm workspace lokal
  - Types awal: UserRole, GameMode, APIResponse wrapper
  - Setup auto-publish via GitHub Actions
  - **Estimasi:** S (2 jam)

- [ ] Development infrastructure `P0` `🟢` `[API]`
  - `docker-compose.dev.yml`: PostgreSQL 16, Redis 7, MailHog
  - `.env.example` lengkap dengan komentar
  - `Makefile` atau `make dev` script untuk jalankan keduanya sekaligus
  - **Estimasi:** S (2 jam)

- [ ] Database setup `P0` `🟢` `[API]`
  - Install & configure Drizzle ORM
  - Schema awal: `users`, `profiles`, `institutions`
  - Setup migration system (drizzle-kit)
  - Seed script untuk data awal
  - **Estimasi:** M (5 jam)
  - **Antigravity:** Gunakan Skill: `drizzle-schema`

#### AUTH-01: Authentication System

- [ ] Registration endpoint `P0` `🟡` `[API]`
  - `POST /api/v1/auth/register`
  - Zod validation, bcrypt (cost 12), email verification token
  - Email via BullMQ queue → MailHog (dev), Resend (prod)
  - **Estimasi:** M (6 jam)
  - **Dependencies:** INFRA-01
  - **Human Review:** WAJIB sebelum merge

- [ ] Login & JWT system `P0` `🟡` `[API]`
  - `POST /api/v1/auth/login`
  - Access token RS256 (15 menit) + Refresh token (30 hari, HTTP-only cookie)
  - Refresh rotation: `POST /api/v1/auth/refresh`
  - Logout + blacklist: `POST /api/v1/auth/logout`
  - Rate limiting: 5 attempts / 15 menit per IP
  - **Estimasi:** M (6 jam)
  - **Human Review:** WAJIB sebelum merge

- [ ] Auth middleware `P0` `🟢` `[API]`
  - JWT validation middleware
  - Role-based authorization middleware
  - **Estimasi:** S (3 jam)

- [ ] Update `@talim-afkar/types`: Auth types `P0` `🟢` `[SHARED]`
  - LoginRequest, LoginResponse, RegisterRequest, UserProfile, UserRole
  - **Estimasi:** XS (1 jam)

- [ ] Auth pages & state (Web) `P0` `🟢` `[WEB]`
  - Login, Register, Verify Email pages
  - `src/lib/api-client.ts` — centralized HTTP client
  - Auth Zustand store (user, accessToken, isLoading)
  - Protected route middleware (Next.js middleware.ts)
  - Auto-refresh token on 401
  - **Estimasi:** M (8 jam)
  - **Dependencies:** AUTH-01 endpoints selesai

---

### MINGGU 3–4: Core Content & User System

#### USER-01: User Profile System

- [ ] Profile CRUD `P0` `🟢` `[API]`
  - `GET/PUT /api/v1/profile`
  - User stats initialization (user_stats table)
  - Avatar: pre-signed URL ke R2 (atau local storage untuk dev)
  - **Estimasi:** M (5 jam)

- [ ] Institution management `P1` `🟢` `[API]`
  - `GET /api/v1/institutions` (public list)
  - **Estimasi:** S (2 jam)

- [ ] Update `@talim-afkar/types`: Profile & Stats types `P1` `🟢` `[SHARED]`
  - **Estimasi:** XS (1 jam)

- [ ] Profile & Dashboard pages `P1` `🟢` `[WEB]`
  - Profile page (GET + edit)
  - User stats display (XP, level, streak)
  - Institution selection
  - **Estimasi:** M (6 jam)

#### CONTENT-01: Question Bank

- [ ] Kitab & Bab schema + CRUD `P0` `🟢` `[API]`
  - Schema: `kitabs`, `babs`
  - Admin endpoints untuk manage kitab/bab
  - **Estimasi:** M (5 jam)

- [ ] Question CRUD + Review System `P0` `🟡` `[API]`
  - Schema: `questions` dengan semua metadata
  - Status workflow: draft → under_review → approved / rejected
  - Muallim: submit soal (draft), Admin: approve/reject
  - `GET /api/v1/questions?babId=&difficulty=&status=`
  - **Estimasi:** M (8 jam)
  - **Dependencies:** AUTH-01 (role middleware)

- [ ] Seed initial question bank `P1` `🟢` `[API]`
  - Minimal 100 soal Fiqih (At-Tadzhib / Matan Abu Syuja')
  - Format CSV/JSON → seed script
  - **Estimasi:** L (content work, non-coding)

- [ ] Update `@talim-afkar/types`: Content types `P0` `🟢` `[SHARED]`
  - Kitab, Bab, Question, QuestionStatus, QuestionDifficulty

---

### MINGGU 5–6: Game Mode MVP (Kilat Fiqih)

#### GAME-01: Game Engine Foundation (API)

- [ ] Game session schema `P0` `🟢` `[API]`
  - Schema: `game_sessions`, `game_answers`
  - Drizzle migration
  - **Estimasi:** S (3 jam)
  - **Antigravity:** Gunakan Skill: `drizzle-schema`

- [ ] Kilat Fiqih engine `P0` `🟡` `[API]`
  - `src/modules/game/engines/kilat-fiqih.engine.ts`
  - Score formula: base_score × time_multiplier × streak_multiplier
  - Anti-cheat: timestamp validation (500ms – 17000ms window)
  - **Unit Test: 100% coverage WAJIB**
  - **Estimasi:** M (8 jam)
  - **Human Review:** WAJIB (scoring logic)

- [ ] Game API endpoints `P0` `🟡` `[API]`
  - `POST /api/v1/games/kilat/start`
  - `POST /api/v1/games/kilat/answer`
  - `POST /api/v1/games/kilat/end`
  - Server-side question selection (random per bab, avoid recent)
  - **Estimasi:** M (6 jam)
  - **Human Review:** WAJIB

- [ ] Gamification foundation `P1` `🟡` `[API]`
  - Schema: `user_stats`, `xp_logs`
  - XP accumulation + level calculation
  - Streak tracking (daily activity check)
  - Event: `game:session_completed` → update stats + streak
  - **Estimasi:** M (8 jam)

- [ ] Update `@talim-afkar/types`: Game types `P0` `🟢` `[SHARED]`
  - GameSession, GameAnswer, StartGameResponse, SubmitAnswerRequest/Response

#### FRONTEND-01: Kilat Fiqih UI

- [ ] Kilat Fiqih game pages `P0` `🟡` `[WEB]`
  - Game lobby: pilih kategori, mulai sesi
  - `src/hooks/useKilatFiqih.ts` — game state management
  - Question display dengan teks Arab support (class arabic-text)
  - CountdownTimer component (15 detik, animated)
  - Answer selection dengan animasi correct/incorrect
  - Score display real-time + streak indicator
  - Result screen: total skor, XP earned, accuracy
  - **Estimasi:** L (14 jam)
  - **Risk:** `🔴` timer sync + animasi perlu perhatian

---

### MINGGU 7–8: Classroom & Musabaqoh (MVP Core)

#### CLASSROOM-01: Classroom Management

- [ ] Classroom CRUD `P0` `🟡` `[API]`
  - Muallim: create/manage kelas
  - Invite code generation (6 karakter, expires 7 hari)
  - Student enrollment via kode: `POST /api/v1/classroom/join`
  - `GET /api/v1/classroom/:id/members`
  - **Estimasi:** M (8 jam)

- [ ] Classroom pages `P1` `🟢` `[WEB]`
  - Muallim: buat kelas, lihat members, generate invite code
  - Mahasantri: join kelas via kode
  - **Estimasi:** M (6 jam)

#### REALTIME-01: Musabaqoh Kelas (WebSocket)

- [ ] Socket.io server setup `P0` `🔴` `[API]`
  - Install Socket.io + Redis adapter
  - JWT authentication via handshake
  - Dedicated port 3002 (terpisah dari REST API 3001)
  - Error handling, reconnection strategy
  - **Estimasi:** M (8 jam)
  - **Human Review:** WAJIB

- [ ] Musabaqoh engine + room management `P0` `🔴` `[API]`
  - Schema: `musabaqoh_sessions`, `musabaqoh_results`
  - Room state di Redis
  - WebSocket events sesuai design.md section 9
  - Muallim control: start, next question, end
  - Server-side scoring (tidak trusted dari client)
  - **Estimasi:** L (16 jam)
  - **Tests:** Integration test dengan 2+ simulated clients WAJIB
  - **Human Review:** WAJIB

- [ ] Update `@talim-afkar/types`: WebSocket event types `P0` `🔴` `[SHARED]`
  - Semua MusabaqohEvent types (server→client dan client→server)
  - **Estimasi:** S (2 jam)

- [ ] Socket.io client singleton `P0` `🔴` `[WEB]`
  - `src/lib/socket-client.ts`
  - `src/hooks/useSocket.ts`
  - Connection management, reconnection handling
  - **Estimasi:** M (5 jam)

- [ ] Musabaqoh frontend `P0` `🔴` `[WEB]`
  - Join room page (input kode 6 karakter)
  - Waiting lobby + participant list (WebSocket updates)
  - Game screen: soal + countdown timer + answer buttons
  - Live leaderboard (update setiap soal)
  - Result screen + final ranking
  - Muallim control panel (start, next, end)
  - **Estimasi:** L (16 jam)
  - **Risk:** `🔴` WebSocket state management kompleks

#### MVP-COMPLETE: Leaderboard & Dashboard

- [ ] Leaderboard endpoints `P1` `🟢` `[API]`
  - Global weekly leaderboard (Redis sorted set, rebuild tiap jam)
  - Kelas leaderboard
  - `GET /api/v1/leaderboard/global?period=weekly`
  - `GET /api/v1/leaderboard/classroom/:id`
  - **Estimasi:** S (4 jam)

- [ ] Dashboard & Leaderboard pages `P1` `🟢` `[WEB]`
  - User dashboard: profil, XP, level, streak, recent games, quick play buttons
  - Leaderboard page dengan filter global/kelas
  - **Estimasi:** M (8 jam)

- [ ] MVP Deployment Setup `P0` `🟡` `[INFRA]`
  - `api/Dockerfile` production-ready
  - Railway deployment config untuk API
  - Vercel deployment config untuk Web
  - Environment variables di Railway + Vercel
  - CI/CD GitHub Actions untuk kedua repo
  - **Estimasi:** M (6 jam)
  - **Human Review:** WAJIB (deployment config)

---

## Phase 1 (Bulan 3–4)

### Target: Platform lengkap untuk 500–1000 MAU

**Phase 1 Success Criteria:**

- Semua 6 game modes berfungsi
- Achievement system aktif
- Basic analytics tersedia untuk muallim
- Mahasantri retention D7 ≥ 30%

---

#### GAME-02: Game Modes Tambahan

- [ ] Tebak Kitab mode `P1` `🟡` `[API]` + `[WEB]`
  - API: engine + endpoints, schema `matan_excerpts`
  - 3-level hint system (setiap hint kurangi skor potensial)
  - Web: UI dengan teks Arab + hint reveal animation
  - **Estimasi API:** M (8 jam) | **Estimasi Web:** M (10 jam)

- [ ] Urutan Dalil mode `P1` `🔴` `[API]` + `[WEB]`
  - API: engine dengan ordered items validation
  - Web: drag-and-drop UI (dnd-kit) + keyboard accessibility fallback
  - **Estimasi API:** M (6 jam) | **Estimasi Web:** L (14 jam)
  - **Risk:** `🔴` drag-and-drop mobile touch kompleks

- [ ] Tangga Ilmu progression `P1` `🔴` `[API]` + `[WEB]`
  - API: progression tracking, node unlocking logic, star rating
  - Web: visual progression map (SVG atau CSS)
  - **Estimasi API:** M (10 jam) | **Estimasi Web:** L (16 jam)
  - **Risk:** `🔴` progression logic perlu careful design

- [ ] Duel Afkar mode `P2` `🔴` `[API]` + `[WEB]`
  - API: matchmaking queue (Redis sorted set), ELO system, WebSocket rooms
  - Web: matchmaking UI, duel game screen, ELO display
  - Bot fallback jika tidak ada lawan dalam 30 detik
  - **Estimasi API:** XL (25 jam) | **Estimasi Web:** L (16 jam)
  - **Risk:** `🔴` paling kompleks di proyek ini

#### GAMIFICATION-02: Achievement System

- [ ] Achievement engine `P1` `🟡` `[API]`
  - Schema: `achievements`, `user_achievements`
  - Trigger system via event bus
  - 20+ achievement definitions
  - **Estimasi:** M (10 jam)

- [ ] In-app achievement notifications `P1` `🟢` `[WEB]`
  - Toast notification saat achievement unlock (WebSocket push)
  - Achievement showcase di profile
  - **Estimasi:** M (6 jam)

#### ANALYTICS-01: Muallim Analytics

- [ ] Learning analytics API `P1` `🟡` `[API]`
  - Per mahasantri: accuracy trend, topik kuat/lemah
  - Per kelas: distribusi skor, engagement rate
  - `GET /api/v1/analytics/classroom/:id`
  - **Estimasi:** M (10 jam)

- [ ] Muallim dashboard analytics `P1` `🟡` `[WEB]`
  - Chart: skor distribusi, progress per mahasantri
  - Export report (CSV)
  - **Estimasi:** L (12 jam)

---

## Phase 2 (Bulan 5–7)

### Target: AI features + Admin dashboard + 1000–5000 MAU

---

#### AI-01: Question Generation Pipeline

- [ ] OpenAI / Vercel AI SDK setup `P0` `🔴` `[API]`
  - Vercel AI SDK + OpenAI provider (primary) + Anthropic (fallback)
  - Response caching di Redis (TTL 24 jam)
  - **Estimasi:** M (5 jam)

- [ ] Question generation pipeline `P1` `🔴` `[API]`
  - BullMQ job: `ai:generate_question`
  - Input: teks matan → Output: formatted question JSON
  - Prompt template system di `src/modules/ai/prompts/`
  - Auto-save ke questions (status: ai_draft)
  - Emit: `ai:question_ready`
  - **Estimasi:** L (12 jam)
  - **Human Review:** WAJIB (AI prompts)

- [ ] AI question review UI `P1` `🟡` `[WEB]`
  - Muallim: list AI-generated soal
  - Edit, approve, atau reject dengan alasan
  - **Estimasi:** M (8 jam)

#### AI-02: AI Tutor Chat

- [ ] RAG system: embed kitab content `P1` `🔴` `[API]`
  - PostgreSQL pgvector extension
  - Embed konten dengan text-embedding-3-small
  - Retrieval pipeline: semantic similarity search
  - **Estimasi:** L (14 jam)
  - **Risk:** `🔴` RAG quality krusial

- [ ] AI Tutor streaming API `P1` `🔴` `[API]`
  - `POST /api/v1/ai/tutor/chat`
  - Streaming response (Vercel AI SDK)
  - Rate limiting: 20 Q/hari (free), unlimited (premium)
  - Context: question + user's answer + retrieved kitab text
  - **Estimasi:** L (12 jam)

- [ ] AI Tutor chat UI `P1` `🟡` `[WEB]`
  - Chat interface embedded dalam result screen
  - Streaming text display
  - Rate limit indicator
  - **Estimasi:** M (8 jam)

#### ADMIN-01: Admin Dashboard

- [ ] Super Admin panel `P1` `🟡` `[WEB]` + `[API]`
  - User management (CRUD, suspend, role change)
  - Question review queue global
  - Institution management
  - System metrics overview
  - **Estimasi API:** L (16 jam) | **Estimasi Web:** L (16 jam)

#### NOTIFY-01: Notification System

- [ ] In-app notifications `P1` `🟢` `[API]` + `[WEB]`
  - Schema: `notifications`
  - API: CRUD endpoints
  - Web: notification bell, unread badge, notification center
  - **Estimasi API:** M (6 jam) | **Estimasi Web:** M (6 jam)

- [ ] Email notifications `P1` `🟡` `[API]`
  - Resend integration via BullMQ jobs
  - Templates: streak reminder, achievement, weekly report
  - Unsubscribe mechanism
  - **Estimasi:** M (8 jam)

---

## Phase 3 (Bulan 8–12)

### Target: Social features + PWA + 10K–50K MAU

---

#### SOCIAL-01: Social Features

- [ ] Friend system `P2` `🟡` `[API]` + `[WEB]`
  - Follow/friend requests
  - Compare progress dengan teman
  - **Estimasi:** M (10 jam each)

- [ ] Seasonal events `P1` `🟡` `[API]` + `[WEB]`
  - Event configuration (admin)
  - Ramadan challenge implementation
  - Event leaderboard
  - **Estimasi API:** L (12 jam) | **Estimasi Web:** M (10 jam)

#### PWA-01: Progressive Web App

- [ ] PWA setup `P2` `🟡` `[WEB]`
  - Service Worker (next-pwa atau custom)
  - App manifest
  - Offline mode untuk Tangga Ilmu (soal di-cache)
  - Install prompt
  - **Estimasi:** M (10 jam)

#### PERF-01: Performance Optimization

- [ ] API performance `P2` `🟡` `[API]`
  - Query optimization (EXPLAIN ANALYZE)
  - PgBouncer connection pooling
  - Read replica untuk analytics
  - **Estimasi:** M (8 jam)

- [ ] Frontend optimization `P2` `🟡` `[WEB]`
  - Code splitting per route
  - Arabic font loading optimization (subset)
  - Image optimization (next/image)
  - **Estimasi:** M (6 jam)

---

## Scaling Phase (Bulan 12+)

### Trigger untuk mulai phase ini:

- API P95 > 300ms sustained
- PostgreSQL CPU > 70% sustained
- WebSocket > 5000 concurrent connections
- MAU > 50.000

---

#### SCALE-01: Infrastructure

- [ ] Kubernetes migration `P0` `🔴` `[INFRA]`
  - Containerize semua services
  - K8s manifests (Deployment, Service, HPA)
  - Helm charts
  - **Estimasi:** XL (dedicated DevOps effort)

- [ ] Database scaling `P0` `🔴` `[INFRA]`
  - PostgreSQL read replicas
  - PgBouncer connection pooling
  - **Estimasi:** L (with DBA)

- [ ] Redis Cluster `P1` `🔴` `[INFRA]`
  - Redis Cluster untuk horizontal scaling
  - Socket.io Redis Cluster adapter
  - **Estimasi:** L (12 jam + infra)

#### SCALE-02: Service Extraction

- [ ] Extract WebSocket/Realtime Service `P1` `🔴` `[API]`
  - Socket.io sebagai separate service (scaled independently)
  - Communication via Redis pub/sub
  - **Estimasi:** XL (20–30 jam + careful migration)

- [ ] Extract AI Service `P2` `🔴` `[API]`
  - Semua AI operations di separate service
  - GPU instances untuk embedding
  - **Estimasi:** XL

#### MOBILE-01: React Native

- [ ] React Native app `P2` `🔴`
  - Mengkonsumsi API yang sama (talim-afkar-api)
  - Shared business logic minimal (bukan shared code)
  - Native push notifications
  - **Estimasi:** XL (dedicated mobile team)

---

## Dependency Graph

```
INFRA-01 (Project Setup — API + Web + Shared Types)
  └── AUTH-01 (Authentication — API)
        ├── AUTH-WEB (Auth Pages — Web)
        ├── USER-01 (User Profile — API)
        │     ├── USER-WEB (Profile Pages — Web)
        │     └── CLASSROOM-01 (Classroom — API + Web)
        │           └── REALTIME-01 (Musabaqoh WebSocket — API + Web)
        └── CONTENT-01 (Question Bank — API)
              └── GAME-01 (Kilat Fiqih — API + Web)
                    └── GAMIFICATION-02 (Achievements — API + Web)
                          └── AI-01 (Question Generation — API)
                                └── AI-02 (AI Tutor — API + Web)
                                      └── AI-03 (Adaptive — API)
```

---

## AI Agent Task Assignments (Antigravity)

### Safe untuk Full AI Execution (Mode Fast)

```
✅ Database schema definition (Drizzle) — review setelah selesai
✅ Zod schema definitions
✅ Repository pattern boilerplate
✅ Unit test writing (dengan human review untuk scoring tests)
✅ API endpoint boilerplate (routing, tidak termasuk logic)
✅ UI components (non-critical, non-auth)
✅ Seed data scripts
✅ Documentation updates
✅ @talim-afkar/types type definitions
✅ TanStack Query hooks
```

### Requires Human Review Before Merge

```
⚠️  Auth & security code (semua auth-related)
⚠️  Game scoring algorithms
⚠️  ELO calculation logic
⚠️  Database migrations
⚠️  WebSocket event handlers
⚠️  AI prompt templates
⚠️  CORS configuration
⚠️  Rate limiting configuration
⚠️  API contract changes (@talim-afkar/types breaking changes)
⚠️  Deployment configuration (Dockerfile, Railway config, Vercel config)
```

### Human-Led (AI assists only)

```
👨‍💻  Architecture decisions
👨‍💻  Production deployment execution
👨‍💻  Database schema major changes
👨‍💻  Security audits
👨‍💻  AI model selection dan cost evaluation
👨‍💻  Content moderation policies
```

---

## Progress Tracking

Update section ini setiap sprint:

```
MVP Sprint 1 (Minggu 1-2):     [ ] Not started
  → INFRA-01, AUTH-01 (API + Web)

MVP Sprint 2 (Minggu 3-4):     [ ] Not started
  → USER-01, CONTENT-01 (API + Web)

MVP Sprint 3 (Minggu 5-6):     [ ] Not started
  → GAME-01, FRONTEND-01 (Kilat Fiqih)

MVP Sprint 4 (Minggu 7-8):     [ ] Not started
  → CLASSROOM-01, REALTIME-01, MVP-COMPLETE

Phase 1 Sprint 1 (Bulan 3):    [ ] Not started
  → GAME-02 (Tebak Kitab, Urutan Dalil, Tangga Ilmu)

Phase 1 Sprint 2 (Bulan 4):    [ ] Not started
  → GAME-02 (Duel Afkar), GAMIFICATION-02, ANALYTICS-01

Phase 2 Sprint 1 (Bulan 5):    [ ] Not started
  → AI-01, AI-02

Phase 2 Sprint 2 (Bulan 6):    [ ] Not started
  → AI-03, ADMIN-01, NOTIFY-01

Phase 2 Sprint 3 (Bulan 7):    [ ] Not started
  → Buffer + hardening + performance

Phase 3 (Bulan 8-12):          [ ] Not started
Scaling Phase (Bulan 12+):     [ ] Not started
```

---

## Antigravity Agent Manager — Recommended Task Format

Gunakan format ini saat membuka task baru di Agent Manager:

```
TASK: [TASK-ID] [Short Name]
REPO: talim-afkar-api | talim-afkar-web | both
MODE: Planning | Fast
PRIORITY: P0 | P1 | P2

CONTEXT:
  - Baca design.md section: [X]
  - Baca srs.md section: [Y]
  - Dependencies task sebelumnya: [Z]

SCOPE:
  Files yang akan dibuat/dimodifikasi:
  - api/src/modules/[domain]/[file].ts
  - web/src/[path]/[file].tsx

CONSTRAINTS:
  - [Constraint spesifik untuk task ini dari claude.md]

DONE WHEN:
  - [ ] Endpoint dapat dipanggil dan return expected response
  - [ ] Unit tests pass (coverage target: X%)
  - [ ] @talim-afkar/types diupdate jika ada perubahan
  - [ ] changelog.md diupdate
  - [ ] todo.md diupdate
```
