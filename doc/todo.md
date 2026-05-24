# Ta'lim Afkar — Implementation Roadmap

**Version:** 1.0.0  
**Last Updated:** 2026-05  
**Related:** [`srs.md`](./srs.md) · [`design.md`](./design.md) · [`claude.md`](./claude.md) · [`changelog.md`](./changelog.md)

---

## Status Legend

```
[ ]  Not started
[~]  In progress  
[x]  Completed
[!]  Blocked / needs attention
[-]  Deferred / cancelled
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

---

## MVP (Minggu 1–8)
### Target: Platform bisa dipakai untuk satu kelas pesantren

**MVP Success Criteria:**
- ≥ 1 muallim dapat membuat kelas dan hosting Musabaqoh
- ≥ 20 mahasantri dapat bermain Kilat Fiqih dan melihat leaderboard
- Data tersimpan dengan benar, tidak ada data loss
- Platform berjalan stabil selama 8 jam session tanpa crash

---

### MINGGU 1–2: Fondasi (Infrastructure & Auth)

#### INFRA-01: Project Setup
- [ ] Inisialisasi Turborepo monorepo `P0` `🟢`
  - Setup `apps/api`, `apps/web`, `packages/shared-types`
  - Configure TypeScript strict mode di semua package
  - Setup ESLint + Prettier + Husky pre-commit hooks
  - **Estimasi:** XS (2–4 jam)

- [ ] Setup development environment `P0` `🟢`
  - `docker-compose.yml` dengan: PostgreSQL 16, Redis 7, MailHog (email dev)
  - `.env.example` dengan semua env yang diperlukan
  - Script: `npm run dev` menjalankan semua services sekaligus
  - **Estimasi:** S (2–4 jam)

- [ ] Database setup `P0` `🟢`
  - Install & configure Drizzle ORM
  - Create initial schema: `users`, `profiles`, `institutions`
  - Setup migration system (`drizzle-kit`)
  - Seed script untuk data awal (kitab list, question samples)
  - **Estimasi:** M (4–6 jam)

#### AUTH-01: Authentication System
- [ ] Registration endpoint `P0` `🟡`
  - `POST /api/v1/auth/register`
  - Validasi Zod, bcrypt hashing, email verification token
  - Email verification via queue (MailHog dev, Resend prod)
  - **Estimasi:** M (6–8 jam)
  - **Dependencies:** INFRA-01

- [ ] Login & JWT system `P0` `🟡`
  - `POST /api/v1/auth/login`
  - Access token (15 menit) + Refresh token (30 hari, HTTP-only cookie)
  - Refresh token rotation (`POST /api/v1/auth/refresh`)
  - Logout dengan token invalidation (`POST /api/v1/auth/logout`)
  - **Estimasi:** M (6–8 jam)
  - **Dependencies:** AUTH-01 registration

- [ ] Auth middleware `P0` `🟢`
  - JWT validation middleware
  - Role-based authorization middleware
  - Rate limiting middleware (login: 5 attempts/15min)
  - **Estimasi:** S (3–4 jam)

---

### MINGGU 3–4: Core Content & User System

#### USER-01: User Profile System
- [ ] Profile CRUD `P0` `🟢`
  - `GET/PUT /api/v1/profile`
  - User stats initialization
  - Avatar upload (pre-signed URL ke R2/local storage dev)
  - **Estimasi:** M (6 jam)

- [ ] Institution management `P1` `🟢`
  - `GET /api/v1/institutions` (public list)
  - User dapat pilih institusi saat onboarding
  - **Estimasi:** S (3 jam)

#### CONTENT-01: Question Bank
- [ ] Kitab & Bab schema + CRUD `P0` `🟢`
  - Schema: `kitabs`, `babs` (nested)
  - Admin endpoint untuk manage kitab/bab
  - **Estimasi:** M (5 jam)

- [ ] Question CRUD + Review System `P0` `🟡`
  - Schema: `questions` dengan semua metadata
  - Muallim: submit soal (status: draft)
  - Admin: approve/reject soal
  - Status workflow: draft → under_review → approved/rejected
  - **Estimasi:** M (8 jam)
  - **Dependencies:** AUTH-01 (role middleware)

- [ ] Seed initial question bank `P1` `🟢`
  - Minimal 100 soal Fiqih (At-Tadzhib / Matan Abu Syuja')
  - Metadata lengkap: kitab, bab, difficulty, explanation
  - **Estimasi:** L (ongoing content work, bukan coding)

---

### MINGGU 5–6: Game Mode MVP (Kilat Fiqih)

#### GAME-01: Game Engine Foundation
- [ ] Game session management `P0` `🟡`
  - Schema: `game_sessions`, `game_answers`
  - Session lifecycle: create → active → completed
  - Question selection algorithm (random per bab, avoid recent repeats)
  - **Estimasi:** M (8 jam)

- [ ] Kilat Fiqih game logic `P0` `🟡`
  - `POST /api/v1/games/kilat/start` — mulai sesi
  - `POST /api/v1/games/kilat/answer` — submit jawaban
  - `POST /api/v1/games/kilat/end` — akhiri sesi
  - Score calculation: waktu-based + streak bonus
  - XP award setelah sesi selesai
  - **Estimasi:** M (10 jam)
  - **Tests:** Unit test untuk scoring wajib 100% coverage

- [ ] Gamification foundation `P1` `🟡`
  - Schema: `user_stats`, `xp_logs`
  - XP accumulation + level calculation
  - Streak tracking: daily activity check
  - Event bus: `game.session_completed` → update stats + streak
  - **Estimasi:** M (8 jam)

#### FRONTEND-01: Basic Web App
- [ ] Next.js 14 App Router setup `P0` `🟢`
  - Layout, routing structure
  - TanStack Query setup
  - Zustand store setup
  - Tailwind + shadcn/ui
  - **Estimasi:** S (4 jam)

- [ ] Auth pages `P0` `🟢`
  - Login, Register, Verify Email pages
  - Protected route middleware
  - Auth state management
  - **Estimasi:** M (8 jam)

- [ ] Kilat Fiqih game UI `P0` `🟡`
  - Game lobby, countdown, question display
  - Timer component (15 detik countdown)
  - Answer selection dengan animasi
  - Result screen dengan skor + XP earned
  - **Estimasi:** L (12–16 jam)
  - **Risk:** `🔴` animasi dan timer sync perlu perhatian

---

### MINGGU 7–8: Classroom & Musabaqoh (MVP core)

#### CLASSROOM-01: Classroom Management
- [ ] Classroom CRUD `P0` `🟡`
  - Muallim create/manage kelas
  - Invite code generation (6 karakter, expires)
  - Student enrollment via kode
  - **Estimasi:** M (8 jam)

#### REALTIME-01: Musabaqoh Kelas (WebSocket)
- [ ] Socket.io server setup `P0` `🔴`
  - Install & configure Socket.io dengan Redis adapter
  - Connection authentication (JWT via handshake)
  - Error handling & disconnect detection
  - **Estimasi:** M (8 jam)

- [ ] Musabaqoh room management `P0` `🔴`
  - Schema: `musabaqoh_sessions`
  - Room state management di Redis
  - Events: JOIN, START, QUESTION, ANSWER, LEADERBOARD, END
  - Muallim control: start, pause, next, end
  - **Estimasi:** L (16–20 jam)
  - **Tests:** Integration test wajib dengan 2 simulated clients

- [ ] Musabaqoh frontend `P0` `🔴`
  - Join room page (input kode)
  - Waiting lobby dengan participant list
  - Game screen: soal + timer + jawab
  - Live leaderboard (update setiap soal selesai)
  - Result screen
  - **Estimasi:** L (16 jam)
  - **Risk:** `🔴` WebSocket state management di frontend kompleks

#### MVP-COMPLETE: Leaderboard & Dashboard Dasar
- [ ] Leaderboard endpoint `P1` `🟢`
  - Global weekly leaderboard (based on XP)
  - Kelas leaderboard
  - **Estimasi:** S (4 jam)

- [ ] User dashboard `P1` `🟢`
  - Profil, level, streak, recent games
  - Navigation ke semua game modes (yang sudah ada)
  - **Estimasi:** M (8 jam)

---

## Phase 1 (Bulan 3–4)
### Target: Platform lengkap untuk 500–1000 MAU

**Phase 1 Success Criteria:**
- Semua 6 game modes berfungsi
- Achievement system aktif
- Basic analytics tersedia untuk muallim
- Mahasantri retention D7 ≥ 30%

---

#### GAME-02: Remaining Game Modes

- [ ] Tebak Kitab mode `P1` `🟡`
  - Matan excerpt display (dengan atau tanpa Arab)
  - 3-level hint system
  - Schema: `matan_excerpts`
  - **Estimasi:** M (10 jam)
  - **Dependencies:** CONTENT-01

- [ ] Urutan Dalil mode `P1` `🔴`
  - Schema untuk ordered items (dalil, syarat/rukun)
  - Drag-and-drop UI (dnd-kit library)
  - Keyboard accessibility fallback
  - **Estimasi:** L (14 jam)
  - **Risk:** `🔴` drag-and-drop kompleks, terutama mobile touch

- [ ] Tangga Ilmu progression `P1` `🔴`
  - Schema: `learning_progressions`
  - Node unlocking logic: selesaikan A → unlock B
  - Star rating (1-3) per chapter
  - Visual progression map (SVG atau CSS-based)
  - **Estimasi:** L (20 jam)
  - **Risk:** `🔴` progression logic perlu careful design

- [ ] Duel Afkar mode `P2` `🔴`
  - Matchmaking queue (Redis sorted set)
  - ELO system implementation
  - Duel room WebSocket (extended dari Musabaqoh infra)
  - Bot fallback jika tidak ada lawan
  - **Estimasi:** XL (25–30 jam)
  - **Risk:** `🔴` paling kompleks, defer ke Phase 1 akhir

#### GAMIFICATION-02: Achievement System
- [ ] Achievement engine `P1` `🟡`
  - Schema: `achievements`, `user_achievements`
  - Trigger system via event bus
  - 20+ achievement definitions (minimal)
  - Achievement notification (in-app)
  - **Estimasi:** M (10 jam)

- [ ] Badge UI `P2` `🟢`
  - Achievement gallery di profil
  - Badge earn animation
  - **Estimasi:** S (5 jam)

#### CLASSROOM-02: Muallim Analytics
- [ ] Student progress dashboard `P1` `🟡`
  - Per-student: accuracy per topik, games played, streak
  - Per-class: distribusi skor, topik terlemah
  - **Estimasi:** M (10 jam)
  - **Dependencies:** Analytics events harus sudah di-track

- [ ] Musabaqoh session results `P1` `🟢`
  - Download hasil sesi (PDF/CSV)
  - Riwayat sesi per kelas
  - **Estimasi:** M (8 jam)

#### ANALYTICS-01: Event Tracking
- [ ] Analytics event system `P1` `🟡`
  - Schema: `analytics_events` (partitioned)
  - Track semua game events, UI events penting
  - BullMQ worker untuk async processing
  - **Estimasi:** M (10 jam)

- [ ] Basic metrics aggregation `P2` `🟡`
  - Daily active users
  - Game mode popularity
  - Average session length
  - **Estimasi:** M (8 jam)

---

## Phase 2 (Bulan 5–7)
### Target: AI features + 5.000–10.000 MAU

**Phase 2 Success Criteria:**
- AI tutor tersedia dan memiliki satisfaction rate > 70%
- AI-generated questions memiliki acceptance rate > 60% dari muallim
- Adaptive recommendations aktif
- Admin dashboard production-ready

---

#### AI-01: Question Generation
- [ ] OpenAI integration setup `P0` `🔴`
  - Vercel AI SDK + OpenAI provider
  - Prompt template system
  - Response caching (Redis)
  - **Estimasi:** M (6 jam)

- [ ] Question generation pipeline `P1` `🔴`
  - Muallim input: teks matan → AI generates soal
  - BullMQ job untuk async generation
  - Review queue untuk generated questions
  - **Estimasi:** L (14 jam)

- [ ] AI-generated question review UI `P1` `🟡`
  - Muallim: list soal AI menunggu review
  - Edit, approve, atau reject
  - **Estimasi:** M (8 jam)

#### AI-02: AI Tutor Chat
- [ ] RAG system foundation `P1` `🔴`
  - Embed kitab content menggunakan text-embedding-3-small
  - Store embeddings (pgvector extension PostgreSQL)
  - Semantic retrieval pipeline
  - **Estimasi:** L (16 jam)
  - **Risk:** `🔴` RAG quality sangat menentukan tutor usefulness

- [ ] AI Tutor API + UI `P1` `🔴`
  - Chat interface terintegrasi dalam game context
  - "Mengapa jawaban ini benar/salah?"
  - Streaming response
  - Rate limiting (20 Q/hari free)
  - **Estimasi:** L (14 jam)

#### AI-03: Adaptive Learning
- [ ] Weakness analysis engine `P2` `🔴`
  - Analisis per-user per-topik accuracy
  - Identify weak areas
  - Generate recommendation list
  - **Estimasi:** M (10 jam)

- [ ] Spaced repetition system `P2` `🔴`
  - SM-2 algorithm implementation
  - Schedule soal untuk review
  - "Revisi Harian" mode
  - **Estimasi:** L (12 jam)

- [ ] Weekly learning report `P2` `🟡`
  - Auto-generate per user setiap Senin
  - Email report: strengths, weaknesses, rekomendasi
  - **Estimasi:** M (8 jam)

#### ADMIN-01: Admin Dashboard
- [ ] Super admin panel `P1` `🟡`
  - User management (CRUD, suspend)
  - Content management (question review queue global)
  - Institution management
  - **Estimasi:** L (20 jam)

- [ ] System health dashboard `P2` `🟡`
  - Error rate, latency metrics
  - Active sessions counter
  - Queue depth monitoring
  - **Estimasi:** M (10 jam)

#### NOTIFY-01: Notification System
- [ ] In-app notifications `P1` `🟢`
  - Schema: `notifications`
  - Unread badge, notification center
  - Mark as read
  - **Estimasi:** M (8 jam)

- [ ] Email notifications `P1` `🟡`
  - Resend integration
  - Templates: streak reminder, achievement earned, weekly report
  - Unsubscribe mechanism
  - **Estimasi:** M (8 jam)

- [ ] Push notifications `P2` `🔴`
  - Web Push API (PWA)
  - Permission request flow
  - Streak reminder (20.00 WIB)
  - **Estimasi:** L (12 jam)
  - **Risk:** `🟡` permission rates biasanya rendah, manage expectations

---

## Phase 3 (Bulan 8–12)
### Target: Social features + 10.000–50.000 MAU

**Phase 3 Success Criteria:**
- Seasonal events berjalan (Ramadan challenge)
- Semantic search aktif
- Knowledge graph ter-visualisasi
- PWA offline capability
- Mobile performance optimized

---

#### SOCIAL-01: Enhanced Social Features
- [ ] Friend system `P2` `🟡`
  - Follow/friend requests
  - Compare progress dengan teman
  - **Estimasi:** M (10 jam)

- [ ] Seasonal events system `P1` `🟡`
  - Event configuration (admin)
  - Ramadan challenge implementation
  - Event leaderboard
  - **Estimasi:** L (16 jam)

#### SEARCH-01: Semantic Search
- [ ] Elasticsearch/pgvector semantic search `P2` `🔴`
  - Reindex soal dengan embeddings
  - Search API dengan semantic similarity
  - **Estimasi:** L (14 jam)

#### KNOWLEDGE-01: Knowledge Graph
- [ ] Topic relationship graph `P3` `🔴`
  - Define konsep relationships antar topik
  - Visual knowledge map per kitab
  - **Estimasi:** XL (tergantung konten)

#### PWA-01: Progressive Web App
- [ ] PWA setup `P2` `🟡`
  - Service Worker
  - App manifest
  - Offline capability untuk Tangga Ilmu (soal di-cache)
  - Install prompt
  - **Estimasi:** M (10 jam)

#### PERF-01: Performance Optimization
- [ ] Frontend bundle optimization `P2` `🟡`
  - Code splitting per route
  - Image optimization (next/image)
  - Font loading optimization (Arabic fonts)
  - **Estimasi:** M (8 jam)

- [ ] API performance `P2` `🟡`
  - Query optimization (explain analyze)
  - Read replica untuk analytics queries
  - PgBouncer connection pooling
  - **Estimasi:** M (8 jam)

---

## Scaling Phase (Bulan 12+)
### Target: 50.000–500.000+ MAU

**Scaling Triggers (kapan phase ini dimulai):**
- API P95 > 300ms secara konsisten
- Database CPU > 70% sustained
- WebSocket connections > 5000 concurrent
- MAU > 50.000

---

#### SCALE-01: Infrastructure Scaling
- [ ] Kubernetes migration `P0` `🔴`
  - Containerize semua services
  - K8s manifests (Deployment, Service, HPA)
  - Helm charts
  - **Estimasi:** XL (dedicated DevOps effort)

- [ ] Database scaling `P0` `🔴`
  - Read replicas (RDS Multi-AZ atau Postgres HA)
  - PgBouncer connection pooling
  - Analytics: separate read replica
  - **Estimasi:** L (dengan DBA support)

- [ ] Redis Cluster `P1` `🔴`
  - Redis Cluster untuk horizontal Redis scaling
  - Socket.io Redis Cluster adapter
  - **Estimasi:** L (12 jam + infrastructure)

#### SCALE-02: Service Extraction
- [ ] Extract Realtime Service `P1` `🔴`
  - WebSocket/Socket.io sebagai separate Node.js service
  - Scaled independently dari API
  - **Estimasi:** XL (20–30 jam + careful migration)

- [ ] Extract AI Service `P2` `🔴`
  - AI operations sebagai separate service
  - Bisa scale ke GPU instances
  - **Estimasi:** XL

#### MOBILE-01: Native Mobile App
- [ ] React Native app `P2` `🔴`
  - Share business logic dengan web
  - Native push notifications
  - Haptic feedback untuk game interactions
  - **Estimasi:** XL (dedicated mobile team)

---

## Dependency Graph (Key)

```
INFRA-01 (Project Setup)
  └── AUTH-01 (Authentication)
        ├── USER-01 (User Profile)
        │     └── CLASSROOM-01 (Classroom)
        │           └── REALTIME-01 (Musabaqoh)
        └── CONTENT-01 (Question Bank)
              └── GAME-01 (Kilat Fiqih)
                    └── GAMIFICATION-02 (Achievements)
                          └── AI-01 (Question Generation)
                                └── AI-02 (AI Tutor)
                                      └── AI-03 (Adaptive Learning)
```

---

## AI Agent Task Assignments

Tasks yang direkomendasikan untuk dikerjakan AI coding agents:

### Safe for Full AI Execution
```
✅ Database schema creation (dengan review)
✅ Zod schema definitions
✅ Repository pattern implementations
✅ Unit test writing
✅ API endpoint boilerplate
✅ UI component creation (non-critical)
✅ Seed data scripts
✅ Documentation updates
```

### Requires Human Review Before Merge
```
⚠️ Auth & security code
⚠️ ELO calculation logic
⚠️ Game scoring formulas
⚠️ AI prompt templates
⚠️ Database migrations
⚠️ WebSocket event handlers
```

### Human-Led (AI assists only)
```
👨‍💻 Architecture decisions
👨‍💻 Content moderation policies
👨‍💻 Security audits
👨‍💻 Production deployment
👨‍💻 Database schema major changes
```

---

## Progress Tracking

Update section ini setiap sprint:

```
MVP Sprint 1 (Minggu 1-2):     [ ] Not started
MVP Sprint 2 (Minggu 3-4):     [ ] Not started
MVP Sprint 3 (Minggu 5-6):     [ ] Not started
MVP Sprint 4 (Minggu 7-8):     [ ] Not started

Phase 1 Sprint 1 (Bulan 3):    [ ] Not started
Phase 1 Sprint 2 (Bulan 4):    [ ] Not started

Phase 2 Sprint 1 (Bulan 5):    [ ] Not started
Phase 2 Sprint 2 (Bulan 6):    [ ] Not started
Phase 2 Sprint 3 (Bulan 7):    [ ] Not started

Phase 3 (Bulan 8-12):          [ ] Not started
Scaling Phase (Bulan 12+):     [ ] Not started
```
