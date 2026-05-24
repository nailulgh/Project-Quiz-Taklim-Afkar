# Ta'lim Afkar — Claude AI Agent Operational Guide

**Version:** 1.0.0  
**Target Agents:** Claude (primary), Gemini, Cursor, GPT-based agents  
**Last Updated:** 2026-05  
**Related:** [`srs.md`](./srs.md) · [`design.md`](./design.md) · [`todo.md`](./todo.md) · [`changelog.md`](./changelog.md)

---

## ⚠️ CRITICAL: Read This First

Kamu adalah AI coding agent yang bekerja pada proyek Ta'lim Afkar — sebuah platform pembelajaran Islam berbasis gamifikasi. Dokumen ini adalah **panduan operasional wajib** yang harus dibaca dan dipatuhi **sebelum melakukan perubahan apapun** pada kodebase.

**Pelanggaran terhadap panduan ini akan menghasilkan:**
- Arsitektur yang tidak konsisten
- Regresi bug
- Technical debt yang sulit dibersihkan
- Kerusakan pada sistem yang sedang berjalan

---

## Table of Contents

1. [Development Philosophy](#1-development-philosophy)
2. [Coding Standards](#2-coding-standards)
3. [Architectural Constraints](#3-architectural-constraints)
4. [Modular Development Rules](#4-modular-development-rules)
5. [Safe Modification Rules](#5-safe-modification-rules)
6. [Dependency Management](#6-dependency-management)
7. [Task Decomposition Strategy](#7-task-decomposition-strategy)
8. [Context Preservation Strategy](#8-context-preservation-strategy)
9. [Prompt Chaining Strategy](#9-prompt-chaining-strategy)
10. [Testing Workflow](#10-testing-workflow)
11. [Validation Workflow](#11-validation-workflow)
12. [Documentation Update Workflow](#12-documentation-update-workflow)
13. [Git Workflow](#13-git-workflow)
14. [Refactoring Rules](#14-refactoring-rules)
15. [AI Collaboration Strategy](#15-ai-collaboration-strategy)
16. [Anti-Chaos Engineering Rules](#16-anti-chaos-engineering-rules)
17. [Technical Debt Prevention](#17-technical-debt-prevention)

---

## 1. Development Philosophy

### Core Principles

```
1. SIMPLICITY FIRST
   Solusi paling sederhana yang benar adalah solusi terbaik.
   Jangan tambahkan abstraksi yang tidak diperlukan saat ini.

2. DOMAIN INTEGRITY
   Setiap perubahan harus menghormati domain boundaries yang ada di design.md.
   Tidak ada shortcut lintas domain yang bypass interface resmi.

3. TESTABILITY AS PREREQUISITE
   Jika kode tidak dapat ditest secara isolated, desainnya salah.
   Perbaiki desain sebelum menulis test.

4. EXPLICIT OVER IMPLICIT
   Kode yang eksplisit lebih baik daripada yang "pintar".
   AI agents (dan manusia) harus bisa membaca kode tanpa konteks tersembunyi.

5. DOCUMENTATION SYNCHRONY
   Setiap perubahan arsitektur harus diikuti update pada docs/*.md.
   Kode dan dokumentasi harus selalu sinkron.
```

### What This Project Is NOT

```
❌ Bukan eksperimen teknikal
❌ Bukan showcase teknologi terbaru
❌ Bukan proyek untuk "mencoba hal baru yang menarik"

✅ Ini adalah platform produksi yang akan digunakan mahasantri nyata
✅ Setiap keputusan harus mempertimbangkan maintainability jangka panjang
✅ Simplicity > Cleverness
```

---

## 2. Coding Standards

### 2.1 TypeScript Rules

```typescript
// ✅ WAJIB: Explicit return types pada semua function publik
async function createGameSession(userId: string, mode: GameMode): Promise<GameSession> { ... }

// ✅ WAJIB: Zod schema untuk semua external input
const CreateSessionSchema = z.object({
  mode: z.enum(['kilat_fiqih', 'tebak_kitab', 'duel_afkar', 'tangga_ilmu', 'urutan_dalil']),
  kitabId: z.string().uuid().optional(),
})

// ✅ WAJIB: Error handling eksplisit, tidak ada silent fails
const result = await questionService.findById(id)
if (!result) throw new NotFoundError(`Question ${id} tidak ditemukan`)

// ❌ DILARANG: any type tanpa komentar justifikasi
const data: any = response.json()  // TIDAK BOLEH

// ❌ DILARANG: non-null assertion tanpa validasi sebelumnya
const name = user!.profile!.name  // BERBAHAYA

// ✅ Optional chaining yang aman
const name = user?.profile?.name ?? 'Anonymous'
```

### 2.2 Naming Conventions

```
Files:
  kebab-case.ts                    → kilat-fiqih.engine.ts
  kebab-case.service.ts            → game.service.ts
  kebab-case.repository.ts         → question.repository.ts
  kebab-case.routes.ts             → game.routes.ts
  kebab-case.schema.ts             → auth.schema.ts (Zod schemas)
  kebab-case.test.ts               → game.service.test.ts

TypeScript:
  PascalCase                       → GameSession, UserProfile
  camelCase                        → createSession, userId
  SCREAMING_SNAKE_CASE             → MAX_ELO_CHANGE, DEFAULT_TIMER_MS
  Interfaces prefix 'I'?           → TIDAK, gunakan langsung: GameSession, tidak IGameSession

Database (Drizzle):
  snake_case                       → game_sessions, user_id, created_at
  Table names: plural              → questions, users, game_sessions
```

### 2.3 Function Size Rules

```
Aturan 30-50 baris:
  - Setiap function maksimal 50 baris
  - Jika lebih: extract helper functions
  - Service methods: fokus satu responsibility

Aturan Single Responsibility:
  - satu function = satu hal yang jelas
  - nama function harus menjelaskan SEMUA yang dilakukan

Contoh refactor:
  // ❌ TERLALU BANYAK RESPONSIBILITY
  async function processGameAnswer(sessionId, questionId, answer, userId) {
    // validate session... (20 baris)
    // calculate score... (15 baris)
    // update streak... (10 baris)
    // check achievements... (15 baris)
    // notify... (10 baris)
  }

  // ✅ SETELAH REFACTOR
  async function processGameAnswer(dto: SubmitAnswerDto): Promise<AnswerResult> {
    const session = await this.validateActiveSession(dto.sessionId, dto.userId)
    const result = this.calculateAnswerResult(dto, session)
    await this.persistAnswer(result)
    this.eventBus.emit('game.answer_submitted', result)
    return result
  }
```

### 2.4 Error Handling Pattern

```typescript
// Custom error classes (di shared/errors/)
export class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number,
    public readonly details?: unknown
  ) {
    super(message)
    this.name = 'AppError'
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} tidak ditemukan`, 'NOT_FOUND', 404)
  }
}

export class ForbiddenError extends AppError {
  constructor(action?: string) {
    super(
      action ? `Tidak diizinkan: ${action}` : 'Akses ditolak',
      'FORBIDDEN',
      403
    )
  }
}

export class ValidationError extends AppError {
  constructor(details: ZodError) {
    super('Input tidak valid', 'VALIDATION_ERROR', 400, details.flatten())
  }
}

// Pattern: semua async handler dibungkus error handler di middleware
// JANGAN throw error mentah dari route handler
```

---

## 3. Architectural Constraints

### 3.1 Constraints yang Tidak Dapat Dilanggar

```
CONSTRAINT-01: Domain Isolation
  Modul TIDAK BOLEH mengimport langsung dari modul lain.
  Gunakan: service interface, event bus, atau shared types.
  
  SALAH:  import { UserRepository } from '../user/user.repository'
  BENAR:  inject UserService → panggil userService.findById()

CONSTRAINT-02: Database Ownership
  Setiap tabel "dimiliki" satu modul.
  Query ke tabel modul lain HARUS melalui service interface-nya.
  Tidak ada cross-domain JOIN di query level.

CONSTRAINT-03: Event Bus untuk Cross-Domain Effects
  Side effects lintas domain HARUS menggunakan event bus.
  CONTOH: game completion → XP update → streak check → achievement check
  Ini adalah chain event, bukan direct function call lintas modul.

CONSTRAINT-04: Realtime State di Redis
  State game real-time (session aktif, room, score live) HARUS di Redis.
  PostgreSQL hanya untuk state final setelah session selesai.

CONSTRAINT-05: AI Calls Selalu Async
  Tidak ada AI API call yang blocking request/response cycle.
  Semua AI calls → queue jobs → hasil disimpan → user di-notify.
  Pengecualian: AI Tutor chat (user menunggu jawaban, max 10 detik timeout).

CONSTRAINT-06: Server-Side Score Authority
  Semua perhitungan skor, XP, ELO HARUS di server.
  Client tidak pernah mengirim "skor saya X".
  Client mengirim "jawaban saya A untuk soal X pada detik ke-7".
```

### 3.2 Technology Constraints

```
JANGAN menambahkan dependency baru tanpa:
  1. Cek apakah sudah ada library existing yang bisa digunakan
  2. Evaluasi bundle size impact (frontend)
  3. Evaluasi security & maintenance status library
  4. Update CHANGELOG.md dengan keputusan ini

JANGAN mengubah database schema tanpa:
  1. Menulis migration file dengan Drizzle
  2. Testing migration di environment lokal
  3. Mempertimbangkan backward compatibility
  4. Update ERD di design.md jika ada perubahan signifikan
```

---

## 4. Modular Development Rules

### 4.1 Cara Benar Mengembangkan Modul

```
Urutan pengembangan fitur baru:
  1. Tentukan domain modul mana yang bertanggung jawab
  2. Desain interface publik modul (TypeScript types/interfaces)
  3. Tulis test terlebih dahulu (TDD jika memungkinkan)
  4. Implementasi repository (database layer)
  5. Implementasi service (business logic)
  6. Implementasi routes (HTTP layer)
  7. Registrasi routes di app
  8. Update dokumentasi

Cara SALAH:
  ❌ Langsung coding di route handler tanpa desain
  ❌ Menyebar logic game di berbagai file tanpa struktur
  ❌ Langsung akses database dari route handler
```

### 4.2 Module Interface Contract

```typescript
// Setiap modul HARUS memiliki public interface yang jelas
// Contoh: src/modules/user/user.interface.ts

export interface IUserService {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  updateProfile(userId: string, data: UpdateProfileDto): Promise<Profile>
  getStats(userId: string): Promise<UserStats>
}

// Implementation terpisah dari interface
// src/modules/user/user.service.ts
export class UserService implements IUserService {
  constructor(private readonly repo: UserRepository) {}
  // ...
}

// DI Container (misal, manual atau menggunakan tsyringe)
// src/modules/user/user.module.ts
export function createUserModule(): IUserService {
  const repo = new UserRepository(db)
  return new UserService(repo)
}
```

---

## 5. Safe Modification Rules

### 5.1 Sebelum Membuat Perubahan Apapun

```
CHECKLIST PRE-MODIFICATION:

□ 1. Apakah saya sudah membaca bagian relevan dari design.md?
□ 2. Apakah perubahan ini melanggar salah satu dari 6 CONSTRAINT utama?
□ 3. Apakah ada test yang akan fail akibat perubahan ini?
□ 4. Apakah perubahan ini mempengaruhi API contract yang ada?
□ 5. Apakah ada migrasi database yang diperlukan?
□ 6. Apakah ada dokumen yang perlu diupdate?

Jika jawaban atas pertanyaan 2 adalah YA → STOP, diskusikan dulu.
Jika jawaban atas 3, 4, 5, 6 adalah YA → prepare plan sebelum coding.
```

### 5.2 Modifying Existing Code

```
RULE: Minimal Footprint Changes
  Ubah sesedikit mungkin untuk mencapai tujuan.
  Hindari refactoring "sekalian" yang tidak diminta.

RULE: Backward Compatibility First
  Perubahan API harus backward compatible selama mungkin.
  Jika breaking change tidak dapat dihindari:
    1. Versi baru endpoint (/api/v2/...)
    2. Deprecation notice di response header
    3. Sunset date untuk v1

RULE: Database Migrations are Irreversible
  Setiap migration harus memiliki:
    - UP migration (apply)
    - DOWN migration (rollback)
  Test DOWN migration sebelum deploy.
  Hindari DROP COLUMN di production tanpa soft-delete dulu.
```

### 5.3 High-Risk Areas (Extra Caution)

```
⚠️ AREA BERISIKO TINGGI — double-check sebelum modifikasi:

1. src/modules/auth/*          → Security impact
2. src/shared/middleware/*     → Affects all routes
3. src/infrastructure/database → Schema changes are permanent
4. Game engine scoring logic   → Affects fairness, ELO integrity
5. ELO calculation             → Competitive integrity
6. AI prompt templates         → Quality of AI output
7. WebSocket room management   → Realtime correctness
8. Any queue worker            → Async reliability
```

---

## 6. Dependency Management

### 6.1 Adding New Dependencies

```
Process untuk menambahkan dependency baru:

1. CARI DULU apakah sudah ada solusi:
   - Apakah fungsi ini bisa diimplementasi dengan library yang sudah ada?
   - Apakah ada built-in Node.js API yang bisa digunakan?

2. EVALUASI library:
   - npm weekly downloads > 100K? (indikasi komunitas aktif)
   - Last published < 6 bulan? (maintained)
   - TypeScript support native?
   - Bundle size (untuk frontend)?
   - Security audit: npm audit

3. DOKUMENTASIKAN keputusan:
   Di CHANGELOG.md, section [Added]:
   "Added: [library] v[version] — [alasan singkat kenapa dipilih]"

4. PINNED VERSION:
   Selalu pin exact version: "library": "1.2.3"
   Bukan: "library": "^1.2.3" (untuk production deps)
```

### 6.2 Updating Dependencies

```
Schedule update:
  - Security patches: segera setelah tersedia
  - Minor updates: monthly
  - Major updates: per quarter, dengan testing penuh

Process major update:
  1. Baca changelog library
  2. Test di branch terpisah
  3. Run full test suite
  4. Deploy ke staging, monitor 24 jam
  5. Deploy ke production
```

---

## 7. Task Decomposition Strategy

### 7.1 Cara Memecah Tugas Besar

Ketika mendapat instruksi besar seperti "Implementasi fitur Duel Afkar", pecah menjadi:

```
LEVEL 1 — Epic:
  "Implementasi Duel Afkar Mode"

LEVEL 2 — Stories:
  1. Setup WebSocket infrastructure untuk Duel
  2. Implementasi matchmaking system
  3. Implementasi game session management
  4. Implementasi real-time answer processing
  5. Implementasi ELO rating update
  6. Frontend: Duel lobby & game UI
  7. Frontend: Result screen

LEVEL 3 — Tasks (per story):
  Story 1: Setup WebSocket:
    1.1 Install dan configure Socket.io
    1.2 Create WebSocket handler module
    1.3 Setup Redis adapter untuk horizontal scaling
    1.4 Write connection/disconnect handlers
    1.5 Write integration test untuk WS connection

LEVEL 4 — Implementation:
  Task 1.1: Install Socket.io
    - npm install socket.io
    - npm install @types/socket.io
    - Update changelog.md
```

### 7.2 Estimasi Kompleksitas

Sebelum mulai mengerjakan task, estimasi:

```
XS (< 30 menit):  Bugfix sederhana, update konfigurasi, update docs
S  (30 menit–2 jam): Fitur kecil, endpoint baru, component UI sederhana
M  (2–8 jam):     Fitur medium dengan testing, domain logic baru
L  (1–3 hari):    Fitur kompleks, multi-domain, WebSocket feature
XL (> 3 hari):    Decompose lebih lanjut sebelum mulai
```

---

## 8. Context Preservation Strategy

### 8.1 Context yang Harus Selalu Tersedia

Saat memulai sesi coding baru, pastikan kamu memiliki konteks:

```
MINIMUM CONTEXT:
  ✓ Dokumen srs.md — untuk memahami WHAT kita bangun
  ✓ Dokumen design.md — untuk memahami HOW kita bangun
  ✓ File yang akan dimodifikasi
  ✓ Test files yang relevan

EXTENDED CONTEXT (untuk fitur kompleks):
  ✓ Seluruh module yang terlibat
  ✓ Shared types yang digunakan
  ✓ Database schema yang relevan (Drizzle schema files)
  ✓ Related routes dan middleware
```

### 8.2 State Preservation untuk Long Tasks

```
Untuk tugas yang tidak bisa selesai dalam satu sesi:

1. CHECKPOINT COMMENT di kode:
   // PROGRESS: Sudah selesai matchmaking logic.
   // TODO NEXT: Implementasi answer processing (lihat game.service.ts)
   // STATUS: Test belum ditulis untuk fungsi ini

2. Selalu commit di checkpoint yang logis:
   git commit -m "feat(duel): add matchmaking queue logic [WIP]"

3. Update todo.md dengan status terkini:
   - [x] Task yang selesai
   - [ ] Task yang belum
   - [~] Task yang sedang in-progress
```

---

## 9. Prompt Chaining Strategy

### 9.1 Untuk Fitur Kompleks Multi-Step

Ketika mengerjakan fitur besar, chain prompt secara terstruktur:

```
Step 1: Architecture Review
  Prompt: "Baca design.md section [X] dan srs.md section [Y].
           Jelaskan approach kamu untuk mengimplementasi [fitur].
           Identifikasi dependencies dan risiko."

Step 2: Interface Design
  Prompt: "Berdasarkan approach yang sudah disetujui, desain
           TypeScript interfaces untuk [fitur]. Hanya interfaces,
           belum implementasi."

Step 3: Test-First
  Prompt: "Tulis unit tests untuk service [X] berdasarkan
           interface yang sudah dibuat. Mock semua dependencies."

Step 4: Implementation
  Prompt: "Implementasi [service/repository/route] sehingga
           semua test yang sudah ditulis pass."

Step 5: Integration Test
  Prompt: "Tulis integration test untuk endpoint [X] yang
           melibatkan database real (test database)."

Step 6: Documentation
  Prompt: "Update changelog.md dengan fitur yang baru selesai.
           Update srs.md jika ada hal yang berubah dari requirement."
```

### 9.2 Anti-Pattern dalam Prompt Chaining

```
❌ JANGAN: "Implementasi semua fitur game dalam satu prompt"
❌ JANGAN: Lanjutkan task tanpa commit checkpoint
❌ JANGAN: Skip architecture step langsung ke coding
❌ JANGAN: Ignore test step karena "bisa belakangan"

✅ LAKUKAN: Satu step per session jika fitur kompleks
✅ LAKUKAN: Validate output setiap step sebelum lanjut
✅ LAKUKAN: Commit setelah setiap step yang meaningful
```

---

## 10. Testing Workflow

### 10.1 Testing Pyramid

```
         E2E Tests (Playwright)
        /     ~5% of test suite     \
       /  Critical user journeys     \
      ────────────────────────────────
     Integration Tests (Supertest/Vitest)
    /        ~25% of test suite          \
   /   API endpoints, DB queries, WS     \
  ──────────────────────────────────────────
 Unit Tests (Vitest)
/            ~70% of test suite             \
/  Service logic, utilities, calculations   \
────────────────────────────────────────────
```

### 10.2 Test File Structure

```typescript
// src/modules/game/game.service.test.ts

import { describe, it, expect, beforeEach, vi } from 'vitest'
import { GameService } from './game.service'
import { createMockQuestionService } from '../content/__mocks__/question.service.mock'

describe('GameService', () => {
  let gameService: GameService

  beforeEach(() => {
    gameService = new GameService({
      questionService: createMockQuestionService(),
      // ... other mocks
    })
  })

  describe('calculateKilatScore', () => {
    it('returns 100 points for correct answer with 15 seconds remaining', () => {
      const result = gameService.calculateKilatScore({
        isCorrect: true,
        timeRemainingMs: 15000,
        streakCount: 0,
      })
      expect(result).toBe(100)
    })

    it('returns 0 points for wrong answer', () => {
      const result = gameService.calculateKilatScore({
        isCorrect: false,
        timeRemainingMs: 10000,
        streakCount: 0,
      })
      expect(result).toBe(0)
    })

    it('adds streak bonus of 20 points for 3 consecutive correct', () => {
      const result = gameService.calculateKilatScore({
        isCorrect: true,
        timeRemainingMs: 15000,
        streakCount: 3,
      })
      expect(result).toBe(120) // 100 + 20 bonus
    })
  })
})
```

### 10.3 Coverage Requirements

```
Minimum coverage (tidak boleh dikurangi):
  - Statements:  80%
  - Branches:    75%
  - Functions:   85%
  - Lines:       80%

Prioritas coverage:
  1. Game scoring logic → 100% coverage wajib
  2. ELO calculation → 100% coverage wajib
  3. Auth logic → 100% coverage wajib
  4. Payment/billing (jika ada) → 100% coverage wajib
  5. Semua service → > 80%
  6. Utilities → > 90%
```

---

## 11. Validation Workflow

### 11.1 Sebelum Commit

```bash
# Run sebelum setiap commit (atau setup di pre-commit hook)
npm run lint              # ESLint + Prettier check
npm run type-check        # tsc --noEmit
npm run test:unit         # Vitest unit tests
npm run test:integration  # Integration tests (butuh DB)

# Jika semua pass → commit OK
# Jika ada yang fail → FIX DULU, baru commit
```

### 11.2 Sebelum Merge ke Main

```bash
npm run test:e2e          # Playwright E2E tests
npm run build             # Pastikan build tidak error
npm run db:migrate:check  # Pastikan tidak ada migration yang pending
```

### 11.3 Validasi API Changes

```
Setiap perubahan API endpoint:
  1. Update OpenAPI spec (jika menggunakan auto-generate: pastikan types update)
  2. Verify semua client (web, mobile) masih compatible
  3. Jika breaking change: buat versi baru endpoint

Setiap perubahan WebSocket events:
  1. Update event schema di shared/events/
  2. Update semua handler yang listen event tersebut
  3. Update frontend socket client
  4. Test dengan 2 clients simultaneously
```

---

## 12. Documentation Update Workflow

### 12.1 Kapan Dokumentasi Harus Diupdate

```
WAJIB update saat:
  - Menambah/mengubah/menghapus API endpoint → update srs.md section 22
  - Mengubah arsitektur signifikan → update design.md
  - Menambah dependency baru → update changelog.md
  - Merilis fitur baru → update changelog.md
  - Mengubah database schema → update design.md section 5
  - Mengubah deployment → update design.md section 15

OPSIONAL (tapi dianjurkan):
  - Bugfix kecil → minimal satu baris di changelog
  - Refactor internal yang tidak mengubah behavior → internal comment cukup
```

### 12.2 Changelog Entry Format

```markdown
## [1.2.0] - 2026-06-15

### Added
- [Game] Kilat Fiqih solo mode sepenuhnya implementasi
- [AI] AI question generation pipeline dengan review queue

### Changed
- [Auth] Refresh token rotation ditambahkan untuk keamanan lebih baik

### Fixed
- [Game] Bug: skor Kilat Fiqih tidak menghitung streak bonus dengan benar

### Security
- [Auth] Rate limiting pada endpoint login: 5 attempts per 15 menit

### Migration Notes
- Run: `npm run db:migrate` setelah update (tambahan kolom `streak_bonus`)
```

---

## 13. Git Workflow

### 13.1 Commit Message Convention

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

Scopes (berdasarkan domain):
  auth, user, game, content, learning, gamification,
  classroom, ai, analytics, notification, admin,
  ws (WebSocket), queue, infra, docs

Contoh:
  feat(game): add Kilat Fiqih streak bonus calculation
  fix(auth): fix refresh token not invalidated on logout
  docs(design): update WebSocket architecture diagram
  feat(ai): add question generation queue worker
  test(game): add unit tests for ELO calculation
  chore(deps): update Socket.io to v4.8.0
```

### 13.2 Branch Rules

```
Branch naming:
  feature/[ticket-id]-[short-description]     → feature/TA-42-kilat-fiqih-engine
  fix/[ticket-id]-[short-description]         → fix/TA-55-elo-calculation-bug
  hotfix/[short-description]                  → hotfix/auth-token-leak
  docs/[description]                          → docs/update-design-ws-section
  chore/[description]                         → chore/upgrade-socket-io

Rules:
  - main branch: production-ready only
  - develop branch: staging, pre-production testing
  - feature/* → PR ke develop
  - hotfix/* → PR ke main + cherry-pick ke develop
  - PR wajib: review (minimal 1 manusia atau 1 AI review checkpoint)
  - PR wajib: semua CI checks pass
```

---

## 14. Refactoring Rules

### 14.1 When to Refactor

```
Refactor BOLEH dilakukan ketika:
  - Kode duplikasi yang sama muncul 3 kali atau lebih (Rule of Three)
  - Function melebihi 50 baris
  - Cyclomatic complexity > 10
  - Test coverage di bawah target

Refactor TIDAK BOLEH dilakukan:
  - Sebelum fitur baru diimplementasi ("nanti sekalian refactor")
    → Bahaya: bercampur antara fitur baru dan refactor lama
  - Saat ada bug kritis yang sedang difix
  - Tanpa test suite yang mecover kode yang akan direfactor

Process refactor yang aman:
  1. Pastikan test coverage ada
  2. Refactor dalam commit TERPISAH dari feature/fix
  3. Commit message: "refactor(scope): [description]"
  4. Verify semua tests masih pass setelah refactor
```

### 14.2 Database Schema Refactoring

```
PERATURAN KERAS:
  - Tidak boleh DROP COLUMN pada migration production
  - Tidak boleh RENAME COLUMN (akan break existing queries)
  - Selalu gunakan ADD COLUMN + migrate data + RENAME (3 deployment cycle)

Safe rename sequence:
  Deploy 1: ADD COLUMN new_name (copy data from old_name)
  Deploy 2: Switch code to use new_name, keep old_name
  Deploy 3: DROP COLUMN old_name (setelah dipastikan tidak digunakan)
```

---

## 15. AI Collaboration Strategy

### 15.1 Cara Efektif Menggunakan AI Agents

```
CONTEXT INJECTION yang efektif:
  "Kamu bekerja pada Ta'lim Afkar — platform pembelajaran Islam.
   Baca design.md section [X] sebelum menjawab.
   Constraint yang harus dipatuhi: [list dari section 3]."

SCOPE BOUNDARIES yang jelas:
  ✅ "Implementasi hanya file: src/modules/game/game.service.ts"
  ❌ "Implementasi game system" (terlalu luas, tak terkontrol)

VALIDATION CHECKPOINT:
  "Sebelum menulis kode, jelaskan approach kamu:
   1. File mana yang akan dimodifikasi?
   2. Apakah ada domain boundary yang dilanggar?
   3. Apakah memerlukan migration database?
   4. Bagaimana cara test implementasi ini?"
```

### 15.2 AI Agent Outputs yang Harus Selalu Direview

```
Wajib human review sebelum commit/deploy:
  ☑ Semua AI-generated security code (auth, permission)
  ☑ Semua database migration files
  ☑ Semua perubahan pada game scoring/ELO logic
  ☑ Semua AI-generated API contracts
  ☑ Perubahan pada WebSocket event handling

Boleh commit dengan AI review saja:
  ☐ Test files
  ☐ Documentation updates
  ☐ UI components (non-critical)
  ☐ Utility functions dengan test coverage > 90%
```

---

## 16. Anti-Chaos Engineering Rules

### 16.1 The Chaos Prevention Checklist

```
Setiap sesi development, tanyakan:

□ Apakah saya tau EXACTLY file mana yang akan berubah?
□ Apakah perubahan ini minimal untuk tujuan yang ingin dicapai?
□ Apakah ada cara yang lebih sederhana?
□ Apakah saya sudah membaca existing code sebelum menulis?
□ Apakah saya akan merusak sesuatu yang saat ini berfungsi?
□ Apakah saya menambahkan lebih complexity dari yang diminta?
```

### 16.2 Stop Conditions

Hentikan pengerjaan dan minta klarifikasi jika:

```
🛑 Requirement tidak jelas atau ambigu
🛑 Implementasi akan melanggar CONSTRAINT dari section 3
🛑 Estimasi complexity jauh lebih besar dari yang diperkirakan
🛑 Ada konflik antara srs.md dan design.md
🛑 Perubahan akan mempengaruhi > 5 domain sekaligus
🛑 Tidak ada test coverage untuk kode kritis yang akan diubah
```

---

## 17. Technical Debt Prevention

### 17.1 Definisi Technical Debt dalam Konteks Ini

```
ACCEPTABLE debt (short-term, tracked):
  - TODO comment dengan ticket number
  - Simplified implementation dengan note untuk improvement
  - Temporary workaround dengan expiry date noted

UNACCEPTABLE debt (harus langsung diselesaikan):
  - Security vulnerabilities
  - Missing tests untuk kritical logic
  - Broken error handling
  - Crossing domain boundaries
  - Undocumented breaking changes

Format TODO comment:
  // TODO(TA-123): Replace with proper semantic search when Elasticsearch is setup
  // FIXME(TA-456): Race condition possible jika concurrent requests > 100
  // HACK: Temporary workaround for Socket.io race condition, fix by v1.2
```

### 17.2 Debt Tracking

```
Setiap TODO/FIXME/HACK harus:
  1. Memiliki ticket/issue reference
  2. Di-track dalam todo.md atau issue tracker
  3. Tidak boleh ada > 30 hari tanpa tindakan

Debt yang dibiarkan > 30 hari dianggap:
  - Bug yang tertunda
  - Technical risk yang aktif
  - Penghalang untuk feature baru
```

---

## Appendix: Quick Reference

### File Modification Map

| Perubahan | Files yang Terdampak |
|-----------|---------------------|
| Skema database | `db/schema/*.ts`, `design.md sec 5`, migration file |
| API endpoint baru | `modules/[domain]/[domain].routes.ts`, `srs.md sec 22` |
| Event bus baru | `shared/events/events.ts`, handler files |
| Game mechanic change | `modules/game/engines/*.ts`, `srs.md sec 8`, test files |
| ELO formula change | `modules/game/engines/duel-afkar.engine.ts`, `srs.md sec 8.3` |
| AI prompt change | `modules/ai/prompts/*.ts`, `changelog.md` |
| New achievement | `modules/gamification/achievements/`, `srs.md sec 11.2` |
| Config change | `.env.example`, deployment docs |
| New dependency | `package.json`, `changelog.md` |

### Emergency Protocols

```
Production Bug Terkonfirmasi:
  1. Buat hotfix branch dari main
  2. Minimal fix — jangan refactor sekalian
  3. Test fix secara isolated
  4. Deploy ke staging, verify 15 menit
  5. Deploy ke production
  6. Post-mortem dalam 24 jam
  7. Add regression test

Data Corruption Detected:
  1. IMMEDIATELY flag ke team
  2. Freeze affected feature (feature flag)
  3. Assess scope of corruption
  4. Restore from backup jika perlu
  5. Run data integrity checks
  6. Document incident
```
