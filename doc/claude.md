# Ta'lim Afkar — Claude AI Agent Operational Guide

**Version:** 1.1.0
**Target Agents:** Claude (primary via Antigravity), Gemini, Cursor, GPT-based agents
**ADE:** Google Antigravity
**Last Updated:** 2026-05
**Related:** [`srs.md`](./srs.md) · [`design.md`](./design.md) · [`todo.md`](./todo.md) · [`changelog.md`](./changelog.md)

---

## ⚠️ CRITICAL: Read This First

Kamu adalah AI coding agent yang bekerja pada proyek **Ta'lim Afkar** — platform pembelajaran Islam berbasis gamifikasi. Dokumen ini adalah **panduan operasional wajib** yang harus dibaca dan dipatuhi **sebelum melakukan perubahan apapun**.

Proyek ini dikerjakan di dalam **Google Antigravity ADE**. Antigravity adalah Agentic Development Environment yang memberikan kamu kemampuan untuk merencanakan, mengeksekusi, memvalidasi, dan melakukan iterasi secara otonom. Namun, otonomi ini hanya efektif jika kamu mematuhi panduan di dokumen ini.

**Arsitektur proyek ini adalah: Fullstack Tradisional Terpisah (Client-Server)**

```
talim-afkar-api/   → Hono.js REST API + WebSocket Server
talim-afkar-web/   → Next.js 14 Frontend
```

Keduanya adalah project/repository terpisah yang di-deploy secara independen.

---

## Table of Contents

1. [Orientasi Antigravity](#1-orientasi-antigravity)
2. [Development Philosophy](#2-development-philosophy)
3. [Coding Standards](#3-coding-standards)
4. [Architectural Constraints](#4-architectural-constraints)
5. [Domain Boundaries (API)](#5-domain-boundaries-api)
6. [Frontend Rules (Web)](#6-frontend-rules-web)
7. [Safe Modification Rules](#7-safe-modification-rules)
8. [Dependency Management](#8-dependency-management)
9. [Task Decomposition Strategy (Antigravity)](#9-task-decomposition-strategy-antigravity)
10. [Testing Workflow](#10-testing-workflow)
11. [Validation Workflow](#11-validation-workflow)
12. [Documentation Update Workflow](#12-documentation-update-workflow)
13. [Git Workflow](#13-git-workflow)
14. [Antigravity-Specific Behaviors](#14-antigravity-specific-behaviors)
15. [AI Collaboration Strategy](#15-ai-collaboration-strategy)
16. [Anti-Chaos Engineering Rules](#16-anti-chaos-engineering-rules)
17. [Technical Debt Prevention](#17-technical-debt-prevention)

---

## 1. Orientasi Antigravity

### Cara Kerja di Antigravity

Antigravity memisahkan dua concern:

- **Agent Manager** — tempat kamu menerima instruksi, merencanakan, dan mengeksekusi alur kerja
- **Editor** — tempat kamu membaca dan memodifikasi kode secara langsung

### Mode yang Harus Dipilih

```
MODE PLANNING:
  Gunakan untuk:
  - Implementasi fitur baru (Game Engine, Auth System, dll)
  - Refactoring besar (multiple files)
  - Setup infrastruktur (DB schema, queue system)
  - Task yang membutuhkan riset dokumentasi dulu

  Cara kerja di Planning mode:
  1. Kamu akan menyajikan Implementation Plan (Artefak)
  2. Developer mereview dan memberi komentar
  3. Setelah disetujui, kamu lanjut ke implementasi
  4. Sajikan Code Diff Artefak sebelum apply

MODE FAST:
  Gunakan untuk:
  - Fix typo atau rename variabel
  - Update dokumentasi kecil
  - Menjalankan perintah terminal sederhana (npm test, dll)
  - Task yang sangat terisolasi dan jelas
```

### Artefak yang WAJIB Disajikan Sebelum Kode

Setiap task besar WAJIB menyajikan Artefak untuk direview:

```
1. IMPLEMENTATION PLAN
   - Daftar file yang akan dibuat/dimodifikasi
   - Perubahan arsitektur jika ada
   - Dependencies baru jika ada
   - Apakah ada database migration?
   - Estimasi kompleksitas

2. TASK LIST
   Rencana step-by-step yang akan dieksekusi

3. CODE DIFFS (setelah developer approve plan)
   Tampilkan diff sebelum apply untuk perubahan signifikan
```

### Sebelum Memulai SETIAP Task

```
Checklist wajib sebelum menulis satu baris kode:
□ Sudah baca design.md section yang relevan?
□ Sudah baca srs.md requirement yang relevan?
□ Sudah cek todo.md untuk dependencies task?
□ Sudah baca existing code di area yang akan dimodifikasi?
□ Sudah identifikasi semua file yang akan terpengaruh?
□ Task ini di repo API atau Web (atau keduanya)?
□ Apakah perlu database migration?
□ Apakah perlu update @talim-afkar/types?
```

---

## 2. Development Philosophy

### Core Principles

```
1. SIMPLICITY FIRST
   Solusi paling sederhana yang benar adalah solusi terbaik.
   Jangan tambahkan abstraksi yang tidak diperlukan saat ini.

2. DOMAIN INTEGRITY
   Setiap perubahan harus menghormati domain boundaries (design.md section 5).
   Tidak ada shortcut lintas domain yang bypass interface resmi.

3. TESTABILITY AS PREREQUISITE
   Jika kode tidak dapat ditest secara isolated, desainnya salah.
   Perbaiki desain sebelum menulis test.

4. EXPLICIT OVER IMPLICIT
   Kode yang eksplisit lebih baik daripada yang "pintar".
   AI agents (dan manusia) harus bisa membaca kode tanpa konteks tersembunyi.

5. API-FIRST CONTRACT
   API adalah kontrak antara talim-afkar-api dan talim-afkar-web.
   Perubahan API contract = breaking change = versi baru = komunikasi eksplisit.

6. DOCUMENTATION SYNCHRONY
   Setiap perubahan arsitektur harus diikuti update pada docs/*.md.
   Kode dan dokumentasi harus selalu sinkron.
```

### What This Project Is NOT

```
❌ Bukan eksperimen teknikal — ini adalah platform produksi nyata
❌ Bukan showcase teknologi terbaru — simplicity > novelty
❌ Bukan proyek monorepo — API dan Web adalah dua unit terpisah

✅ Platform produksi untuk mahasantri pesantren Indonesia
✅ Setiap keputusan mempertimbangkan maintainability jangka panjang
✅ Simplicity > Cleverness
```

---

## 3. Coding Standards

### 3.1 TypeScript Rules (Berlaku di API dan Web)

```typescript
// ✅ WAJIB: Explicit return types pada semua function publik
async function createGameSession(userId: string, mode: GameMode): Promise<GameSession> { ... }

// ✅ WAJIB: Zod schema untuk semua external input (API) dan form input (Web)
const CreateSessionSchema = z.object({
  mode: z.enum(['kilat_fiqih', 'tebak_kitab', 'duel_afkar', 'tangga_ilmu', 'urutan_dalil']),
  kitabId: z.string().uuid().optional(),
})

// ✅ WAJIB: Error handling eksplisit
const result = await questionService.findById(id)
if (!result) throw new NotFoundError(`Question ${id} tidak ditemukan`)

// ❌ DILARANG: any type tanpa justifikasi
const data: any = response.json()  // TIDAK BOLEH

// ❌ DILARANG: non-null assertion tanpa validasi
const name = user!.profile!.name  // BERBAHAYA

// ✅ Optional chaining yang aman
const name = user?.profile?.name ?? 'Anonymous'
```

### 3.2 Naming Conventions

```
API Files:
  kebab-case.ts                    → kilat-fiqih.engine.ts
  kebab-case.service.ts            → game.service.ts
  kebab-case.repository.ts         → question.repository.ts
  kebab-case.routes.ts             → game.routes.ts
  kebab-case.schema.ts             → auth.schema.ts
  kebab-case.test.ts               → game.service.test.ts

Web Files:
  PascalCase.tsx                   → QuestionCard.tsx (components)
  camelCase.ts                     → useGameSession.ts (hooks)
  kebab-case/page.tsx              → kilat-fiqih/page.tsx (App Router)

TypeScript (keduanya):
  PascalCase                       → GameSession, UserProfile
  camelCase                        → createSession, userId
  SCREAMING_SNAKE_CASE             → MAX_ELO_CHANGE, DEFAULT_TIMER_MS

Database (Drizzle):
  snake_case                       → game_sessions, user_id, created_at
  Table names: plural              → questions, users, game_sessions
```

### 3.3 Error Handling Pattern (API)

```typescript
// src/shared/errors/index.ts
export class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number,
    public readonly details?: unknown,
  ) {
    super(message);
    this.name = "AppError";
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} tidak ditemukan`, "NOT_FOUND", 404);
  }
}

export class ForbiddenError extends AppError {
  constructor(action?: string) {
    super(
      action ? `Tidak diizinkan: ${action}` : "Akses ditolak",
      "FORBIDDEN",
      403,
    );
  }
}

export class ValidationError extends AppError {
  constructor(details: ZodError) {
    super("Input tidak valid", "VALIDATION_ERROR", 400, details.flatten());
  }
}
```

### 3.4 Function Size Rules

```
Maksimal 50 baris per function.
Jika lebih: extract helper functions.
Satu function = satu responsibility yang jelas.

✅ CONTOH BAIK:
async function processGameAnswer(dto: SubmitAnswerDto): Promise<AnswerResult> {
  const session = await this.validateActiveSession(dto.sessionId, dto.userId)
  const result = this.calculateAnswerResult(dto, session)
  await this.persistAnswer(result)
  this.eventBus.emit('game.answer_submitted', result)
  return result
}
```

---

## 4. Architectural Constraints

### Constraints API (talim-afkar-api)

```
CONSTRAINT-API-01: Domain Isolation
  Modul TIDAK BOLEH mengimport langsung dari modul lain.
  Gunakan: service interface, event bus, atau shared types.

  SALAH:  import { UserRepository } from '../user/user.repository'
  BENAR:  inject UserService → panggil userService.findById()

CONSTRAINT-API-02: Database Ownership
  Setiap tabel "dimiliki" satu modul.
  Query ke tabel modul lain HARUS melalui service interface-nya.

CONSTRAINT-API-03: Event Bus untuk Cross-Domain Effects
  Side effects lintas domain HARUS menggunakan event bus.
  CONTOH: game completion → XP → streak → achievement = chain events

CONSTRAINT-API-04: Realtime State di Redis
  State game real-time HARUS di Redis.
  PostgreSQL hanya untuk state final setelah session selesai.

CONSTRAINT-API-05: Server-Side Score Authority
  Semua perhitungan skor, XP, ELO HARUS di server API.
  Client tidak pernah mengirim "skor saya X".
  Client mengirim "jawaban saya A untuk soal X pada detik ke-7".

CONSTRAINT-API-06: AI Calls Selalu Async (kecuali Tutor)
  AI API call untuk generasi soal → wajib via BullMQ queue.
  Pengecualian: AI Tutor chat (max 10 detik timeout, streaming).
```

### Constraints Web (talim-afkar-web)

```
CONSTRAINT-WEB-01: Zero Direct Database Access
  Web TIDAK BOLEH terhubung langsung ke PostgreSQL atau Redis.
  Semua data HARUS via API atau WebSocket.

CONSTRAINT-WEB-02: Centralized API Client
  Semua HTTP calls WAJIB menggunakan src/lib/api-client.ts.
  Tidak ada fetch() atau axios langsung di components/pages.

CONSTRAINT-WEB-03: Token Management
  Access token: simpan di Zustand state (memory only, bukan localStorage)
  Refresh token: HTTP-only cookie (dimanage otomatis oleh browser)

CONSTRAINT-WEB-04: No Business Logic in Components
  React components hanya untuk UI rendering dan event handling.
  Business logic → custom hooks → api-client → API

CONSTRAINT-WEB-05: Game State Authority
  Frontend tidak menghitung skor, XP, atau ELO.
  Frontend hanya menampilkan state yang diterima dari API/WebSocket.
```

---

## 5. Domain Boundaries (API)

### Panduan Per Domain

```
AUTH Module (src/modules/auth/)
  Owns: users, refresh_tokens tables
  Responsible for: JWT generation, token validation, password hashing
  Exposes: authenticate middleware, verifyToken utility
  Never: akses tabel profile, game, dll langsung

USER Module (src/modules/user/)
  Owns: profiles, user_stats, xp_logs, institutions tables
  Exposes: userService.findById(), userService.updateStats()
  Listens to: game:session_completed → update stats
  Never: logic autentikasi, game logic

GAME Module (src/modules/game/)
  Owns: game_sessions, game_answers, learning_progressions
  Contains: semua game engines (kilat-fiqih, dll)
  Emits: game:session_completed, game:duel_result
  Never: akses profiles langsung, buat achievement langsung

CONTENT Module (src/modules/content/)
  Owns: kitabs, babs, questions, matan_excerpts
  Exposes: questionService.getForGame(), questionService.approve()
  Emits: question:approved
  Never: akses user data, game session data

CLASSROOM Module (src/modules/classroom/)
  Owns: classrooms, classroom_members
  Exposes: classroomService.findById(), classroomService.getMembers()
  Never: logic game, logic auth

GAMIFICATION Module (src/modules/gamification/)
  Owns: achievements, user_achievements, streak logic
  Listens to: game:session_completed, game:duel_result
  Emits: achievement:unlocked, user:streak_broken
  Never: akses game engines langsung

MUSABAQOH Module (src/modules/musabaqoh/)
  Owns: musabaqoh_sessions, musabaqoh_results
  Manages: WebSocket rooms state (via Redis)
  Emits: musabaqoh:ended
  Never: akses classroom member data langsung (hanya via service)

AI Module (src/modules/ai/)
  Owns: BullMQ jobs untuk AI tasks
  Interfaces: OpenAI/Anthropic via Vercel AI SDK
  Emits: ai:question_ready
  Never: bypass review queue untuk auto-publish soal

ANALYTICS Module (src/modules/analytics/)
  Owns: daily_activity
  Listens to: semua events untuk agregasi
  Never: write ke tabel domain lain

NOTIFICATION Module (src/modules/notification/)
  Owns: notifications table
  Listens to: achievement:unlocked, user:streak_broken, dll
  Sends: email via Resend, in-app via WebSocket push
  Never: business logic game/gamification
```

---

## 6. Frontend Rules (Web)

### Component Hierarchy

```
app/[route]/page.tsx         ← Server Component (data fetching)
  └── ClientWrapper.tsx      ← Client Component boundary
        └── FeatureComponent.tsx  ← Interaktif, punya state
              └── UIComponents    ← Pure display (shadcn/ui)
```

### State Management Decisions

```
Gunakan TanStack Query (useQuery/useMutation) untuk:
  ✅ Data dari API (user profile, leaderboard, questions)
  ✅ Data yang perlu di-cache atau di-refetch otomatis
  ✅ Mutations dengan optimistic updates

Gunakan Zustand untuk:
  ✅ Auth state (user, accessToken)
  ✅ Active game state (current session, real-time score)
  ✅ UI state yang perlu diakses dari banyak komponen

Gunakan useState/useReducer untuk:
  ✅ UI state lokal (modal open/close, form state)
  ✅ State yang tidak perlu dishare antar komponen
```

### API Call Pattern

```typescript
// ✅ BENAR — Custom hook yang menggunakan api-client
// src/hooks/useQuestion.ts
export function useQuestion(questionId: string) {
  return useQuery({
    queryKey: ["question", questionId],
    queryFn: () => apiClient.get<Question>(`/api/v1/questions/${questionId}`),
  });
}

// ✅ BENAR — Gunakan hook di component
function QuestionDisplay({ questionId }: Props) {
  const { data, isLoading } = useQuestion(questionId);
  // ...
}

// ❌ SALAH — Direct fetch di component
function QuestionDisplay() {
  const [data, setData] = useState();
  useEffect(() => {
    fetch("http://localhost:3001/api/v1/questions/123"); // DILARANG
  }, []);
}
```

---

## 7. Safe Modification Rules

### Sebelum Modifikasi File Apapun

```
1. BACA file yang akan dimodifikasi (gunakan Antigravity view)
2. IDENTIFIKASI semua dependensi file tersebut
3. VERIFIKASI apakah perubahan akan break sesuatu
4. SAJIKAN rencana ke developer sebelum execute (Mode Planning)
```

### File Kritis — WAJIB Human Review

```
❗ CRITICAL FILES (jangan modifikasi tanpa explicit approval):
   API:
   - src/shared/middleware/auth.middleware.ts
   - src/modules/auth/auth.service.ts
   - src/modules/game/engines/*.ts (scoring logic)
   - db/migrations/*.sql
   - src/config/cors.config.ts

   WEB:
   - src/lib/api-client.ts
   - src/stores/auth.store.ts
   - src/middleware.ts (Next.js middleware)
```

### Database Migration Rules

```
WAJIB untuk setiap perubahan schema:
  1. Buat migration file baru (jangan edit yang sudah ada)
  2. Test migration di development dulu
  3. Verifikasi rollback instruction berfungsi
  4. Update changelog.md dengan migration notes

DILARANG KERAS di production:
  ❌ DROP COLUMN langsung
  ❌ RENAME COLUMN (break existing queries)
  ❌ Constraint yang bisa conflict dengan data existing

Safe rename sequence (3 deployment):
  Deploy 1: ADD COLUMN new_name + copy data
  Deploy 2: Switch code ke new_name, keep old
  Deploy 3: DROP COLUMN old_name
```

---

## 8. Dependency Management

### Menambah Dependency Baru

```
Sebelum install dependency baru, tanyakan:
  1. Apakah sudah ada library existing yang bisa digunakan?
  2. Bundle size impact? (khusus Web dependencies)
  3. Maintenance status library? (last commit, open issues)
  4. License compatibility (MIT/Apache preferred)?
  5. Sudah ada di @talim-afkar/types jika shared?

Setelah install:
  → Update changelog.md section Dependency Changelog
  → Tambahkan komentar justifikasi di package.json (jika non-obvious)
```

### Daftar Library yang Sudah Disetujui

```
API — Approved:
  hono, drizzle-orm, drizzle-kit, zod, jose (JWT), bcryptjs,
  socket.io, bullmq, ioredis, pino, @vercel/ai, openai,
  @anthropic-ai/sdk, resend, vitest, supertest

Web — Approved:
  next, react, tailwindcss, shadcn-ui, framer-motion,
  @tanstack/react-query, zustand, socket.io-client,
  react-hook-form, dnd-kit, zod, vitest, @testing-library/react

BELUM disetujui (evaluasi dulu):
  → GraphQL client, Prisma, tRPC, React Native Web,
    Elasticsearch client (untuk MVP, gunakan PG FTS dulu)
```

---

## 9. Task Decomposition Strategy (Antigravity)

### Cara Memecah Task di Antigravity Agent Manager

```
Task besar → Pecah menjadi subtask yang:
  - Setiap subtask selesai dalam 1 sesi Antigravity
  - Setiap subtask menghasilkan kode yang bisa di-commit
  - Setiap subtask punya test yang bisa dijalankan

CONTOH BAIK (Kilat Fiqih Engine):
  Subtask 1: Buat game session schema + migration
  Subtask 2: Implementasi scoring algorithm + unit tests
  Subtask 3: Buat API endpoints (start/answer/end)
  Subtask 4: Integrasi dengan event bus (XP, streak)
  Subtask 5: Build game UI components
  Subtask 6: Integrasi frontend dengan API endpoints

CONTOH BURUK:
  "Implementasi sistem game" — terlalu besar, tak terkontrol
```

### Paralel vs Sequential Tasks

```
Di Antigravity, agen dapat berjalan paralel. Gunakan ini untuk:
  PARALLEL OK:
  - Unit tests sambil menulis implementation
  - Documentation update sambil mengerjakan code
  - Dua fitur yang tidak saling bergantung

  SEQUENTIAL REQUIRED:
  - Database migration → sebelum service code yang bergantung
  - API endpoint → sebelum frontend hook yang konsumsi
  - Auth middleware → sebelum protected routes
```

### Context Preservation Antar Sesi

Karena Antigravity dapat berjalan multi-sesi:

```
Setiap sesi WAJIB dimulai dengan:
  1. Baca todo.md → lihat status task yang sedang dikerjakan
  2. Baca changelog.md [Unreleased] → lihat apa yang sudah selesai
  3. Cek git log terbaru → verifikasi state terkini

Setiap sesi WAJIB diakhiri dengan:
  1. Update todo.md → ubah [ ] ke [x] untuk task selesai
  2. Update changelog.md → tambah entry di [Unreleased]
  3. Pastikan semua tests pass
  4. Commit dengan message yang deskriptif
```

---

## 10. Testing Workflow

### Test Strategy per Layer

```
API Testing:
  Unit Tests (Vitest):
    - Game engines scoring logic (WAJIB 100% coverage)
    - ELO calculation (WAJIB 100% coverage)
    - Zod schema validation
    - Service methods dengan mocked repositories

  Integration Tests (Vitest + Supertest):
    - Auth endpoints (register, login, refresh, logout)
    - Game session lifecycle (start → answer → end)
    - Classroom CRUD
    - WebSocket events (dengan mock Socket.io)

  Test Database:
    - Dedicated test DB (talimafkar_test)
    - Seed di beforeEach, cleanup di afterEach
    - Drizzle transaction rollback untuk test isolation

Web Testing:
  Unit Tests (Vitest):
    - Custom hooks (useGameSession, useLeaderboard)
    - Utility functions
    - Zustand stores

  Component Tests (@testing-library/react):
    - QuestionCard renders correctly
    - CountdownTimer counts down
    - LiveLeaderboard updates on data change
```

### Coverage Requirements

```
WAJIB 100% coverage:
  - Game scoring algorithms (semua engines)
  - ELO calculation
  - XP calculation
  - Streak logic

Target 80%+:
  - Service layer
  - Custom hooks
  - Utility functions

Nice to have:
  - UI components
  - Route handlers (covered oleh integration tests)
```

---

## 11. Validation Workflow

### Sebelum Commit

```
API:
  npm run lint          ← ESLint check
  npm run type-check    ← TypeScript strict check
  npm run test:unit     ← Unit tests

Web:
  npm run lint
  npm run type-check
  npm run build         ← Next.js build (catches errors)
  npm run test:unit
```

### API Contract Validation

```
Jika mengubah API response format:
  1. Update @talim-afkar/types package
  2. Bump versi package (breaking = major)
  3. Update web untuk menggunakan types baru
  4. Update srs.md section 22
  5. Update changelog.md

Jika menambah endpoint baru:
  1. Implementasi di API
  2. Tambahkan types di @talim-afkar/types
  3. Buat hook di web/src/hooks/
  4. Update srs.md section 22
```

---

## 12. Documentation Update Workflow

### Setelah Setiap Task Selesai

```
Wajib update (sebelum commit final):
  1. changelog.md → tambah entry di [Unreleased]
  2. todo.md → update status task [x]

Update jika ada perubahan signifikan:
  3. srs.md → jika requirement berubah atau endpoint baru
  4. design.md → jika arsitektur berubah
  5. README.md → jika ada perubahan cara setup/run

Format entry changelog:
  - [Domain] Deskripsi singkat dalam Bahasa Indonesia
  Domain: Auth, Game, Content, Learning, Gamification, Classroom,
          AI, Analytics, Notification, Admin, Infra, Docs, Security, WS
```

---

## 13. Git Workflow

### Commit Message Convention

```
Format: <type>(<scope>): <description>

Types:
  feat      → Fitur baru
  fix       → Bug fix
  docs      → Perubahan dokumentasi
  refactor  → Refactoring tanpa perubahan behavior
  test      → Menambah atau memperbaiki tests
  chore     → Tooling, dependency, config
  perf      → Performance improvement
  security  → Security fix
  migration → Database migration

Scopes:
  auth, user, game, content, learning, gamification,
  classroom, ai, analytics, notification, admin,
  ws, queue, infra, docs, types

Contoh:
  feat(game): add Kilat Fiqih streak bonus calculation
  fix(auth): fix refresh token not invalidated on logout
  feat(ai): add question generation queue worker
  migration(db): add learning_progressions table
  chore(types): update @talim-afkar/types v1.2.0
```

### Branch Rules

```
main         → Production only
develop      → Staging
feature/*    → Feature development → PR ke develop
fix/*        → Bug fix → PR ke develop
hotfix/*     → Emergency → PR ke main + cherry-pick ke develop

Naming:
  feature/TA-42-kilat-fiqih-engine
  fix/TA-55-elo-calculation-bug
  hotfix/auth-token-leak
```

---

## 14. Antigravity-Specific Behaviors

### Cara Menggunakan Antigravity Browser (Sub-Agen)

```
Gunakan Antigravity Browser untuk:
  ✅ Membaca dokumentasi library (docs.hono.dev, orm.drizzle.team)
  ✅ Verifikasi API response format dari documentation
  ✅ Membaca error message di browser console (jika ada preview UI)
  ✅ Verifikasi UI/UX setelah implementasi komponen

TIDAK BOLEH menggunakan untuk:
  ❌ Membaca internal company data
  ❌ Akses ke repository private tanpa explicit permission
  ❌ Domain di luar browser-allowlist
```

### Cara Menggunakan Antigravity Terminal

```
Perintah yang BOLEH dijalankan otomatis:
  npm run lint
  npm run type-check
  npm run test:unit
  npm run test:integration
  npm run build
  ls, cat, grep (read-only)

Perintah yang WAJIB minta review sebelum dijalankan:
  npm run db:migrate    ← Modifikasi database
  npm run db:seed       ← Insert data
  git push              ← Deploy changes
  docker-compose up/down
  rm, mv (destructive operations)
  curl ke external services
```

### Cara Menyajikan Code Diffs

```
Sebelum apply perubahan signifikan, sajikan Artefak dengan format:

  📁 FILE: src/modules/game/game.service.ts

  + async startKilatFiqih(userId: string): Promise<GameSession> {
  +   const questions = await this.questionService.getForGame('kilat_fiqih')
  +   const session = await this.createSession({ userId, mode: 'kilat_fiqih' })
  +   await this.redisClient.set(`game:session:${session.id}`, JSON.stringify({...}))
  +   return session
  + }

  [APPROVE atau REQUEST REVISION sebelum apply]
```

---

## 15. AI Collaboration Strategy

### Cara Efektif Memberikan Instruksi ke Claude di Antigravity

```
CONTEXT INJECTION yang efektif:
  "Kamu bekerja pada Ta'lim Afkar — platform pembelajaran Islam.
   Arsitektur: Client-Server terpisah (API: Hono.js, Web: Next.js).
   Baca design.md section [X] sebelum menjawab.
   Kamu sedang bekerja di repo: [talim-afkar-api / talim-afkar-web]
   Constraint yang harus dipatuhi: [list dari section 4]."

SCOPE BOUNDARIES yang jelas:
  ✅ "Implementasi hanya file: src/modules/game/game.service.ts"
  ✅ "Buat endpoint POST /api/v1/games/kilat/start di API repo"
  ❌ "Implementasi game system" (terlalu luas)

VALIDATION CHECKPOINT (tanya sebelum code):
  "Sebelum menulis kode, jelaskan:
   1. File mana yang akan dimodifikasi?
   2. Apakah ada domain boundary yang dilanggar?
   3. Apakah perlu database migration?
   4. Apakah ada perubahan API contract yang mempengaruhi Web?
   5. Bagaimana cara test implementasi ini?"
```

### AI Agent Outputs yang Wajib Human Review

```
WAJIB review sebelum commit/deploy:
  ☑ Semua auth & security code
  ☑ Semua database migration files
  ☑ Semua game scoring / ELO logic
  ☑ Semua perubahan API contract
  ☑ Semua WebSocket event handlers
  ☑ Perubahan CORS configuration
  ☑ Semua AI prompt templates

Boleh commit dengan AI review saja:
  ☐ Test files
  ☐ Documentation updates
  ☐ UI components (non-critical)
  ☐ Utility functions dengan coverage > 90%
  ☐ Seed data scripts
```

---

## 16. Anti-Chaos Engineering Rules

### Chaos Prevention Checklist

```
Setiap task, tanyakan:
□ Saya tau EXACTLY file mana yang akan berubah?
□ Perubahan ini minimal untuk tujuan yang ingin dicapai?
□ Ada cara yang lebih sederhana?
□ Sudah membaca existing code sebelum menulis?
□ Tidak akan merusak sesuatu yang saat ini berfungsi?
□ Tidak menambah complexity lebih dari yang diminta?
□ Task ini di repo yang benar (API atau Web)?
□ Jika mengubah API — sudah update Web juga?
□ Jika mengubah DB schema — sudah ada migration?
```

### Stop Conditions

Hentikan dan minta klarifikasi jika:

```
🛑 Requirement tidak jelas atau ambigu
🛑 Implementasi akan melanggar CONSTRAINT dari section 4
🛑 Kompleksitas jauh lebih besar dari yang diperkirakan
🛑 Ada konflik antara srs.md dan design.md
🛑 Perubahan akan mempengaruhi > 5 domain sekaligus
🛑 Perubahan API contract yang tidak direncanakan
🛑 Tidak ada test coverage untuk kode kritis yang akan diubah
🛑 Task memerlukan credentials/secrets yang tidak tersedia
```

---

## 17. Technical Debt Prevention

### Definisi

```
ACCEPTABLE debt (short-term, tracked):
  - TODO comment dengan ticket number: // TODO(TA-123): ...
  - Simplified implementation dengan upgrade note
  - Temporary workaround dengan expiry date

UNACCEPTABLE debt (selesaikan sekarang):
  - Security vulnerabilities
  - Missing tests untuk critical logic
  - Broken error handling
  - Crossing domain boundaries
  - Undocumented breaking API changes
  - Direct DB access dari Web

Format TODO:
  // TODO(TA-123): Ganti dengan semantic search saat Elasticsearch aktif
  // FIXME(TA-456): Race condition possible jika concurrent > 100
  // HACK: Workaround untuk Socket.io race, fix by v1.2
```

---

## Appendix: File Modification Map

| Perubahan            | Files API                                 | Files Web                 | Docs                       |
| -------------------- | ----------------------------------------- | ------------------------- | -------------------------- |
| DB schema baru       | db/schema/\*.ts, migration                | —                         | design.md sec 6, changelog |
| API endpoint baru    | modules/[domain]/\*.ts                    | hooks/use[Feature].ts     | srs.md sec 22, changelog   |
| API response berubah | modules/[domain]/\*.ts                    | @talim-afkar/types, hooks | srs.md sec 22, changelog   |
| Event bus baru       | shared/events/event-bus.ts, handler       | —                         | design.md sec 7, changelog |
| Game mechanic        | modules/game/engines/\*.ts, test          | game/page.tsx, hooks      | srs.md sec 8, changelog    |
| ELO formula          | modules/game/engines/duel-afkar.engine.ts | —                         | srs.md sec 8.3, changelog  |
| AI prompt            | modules/ai/prompts/\*.ts                  | —                         | changelog (AI Changes)     |
| Achievement baru     | modules/gamification/achievements/        | —                         | srs.md sec 11, changelog   |
| New dependency       | package.json (API)                        | package.json (Web)        | changelog (Dependency)     |
| CORS config          | config/cors.config.ts                     | —                         | changelog (Security)       |

## Appendix: Emergency Protocols

```
Production Bug Terkonfirmasi:
  1. Buat hotfix branch dari main (API atau Web — pisah)
  2. Minimal fix — jangan refactor sekalian
  3. Test fix secara isolated
  4. Deploy ke staging, verify 15 menit
  5. Deploy ke production
  6. Post-mortem dalam 24 jam
  7. Add regression test

API Down (WebSocket/REST):
  1. Check Railway/VPS health dashboard
  2. Check /health endpoint
  3. Check PostgreSQL dan Redis connectivity
  4. Jika perlu: rollback ke Docker image sebelumnya

Web Down (Vercel):
  1. Check Vercel deployment status
  2. Rollback ke previous deployment di Vercel dashboard
  3. Verify NEXT_PUBLIC_API_URL masih valid

Data Corruption:
  1. Segera freeze affected feature (feature flag atau deploy revert)
  2. Assess scope corruption
  3. Restore dari backup PostgreSQL
  4. Run data integrity checks
  5. Document incident di changelog (Security section)
```
