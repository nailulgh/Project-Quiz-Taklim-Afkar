# Ta'lim Afkar — Changelog

**Format:** [Semantic Versioning 2.0.0](https://semver.org)
**Status:** Living Document
**Last Updated:** 2026-05
**Arsitektur:** Fullstack Tradisional Terpisah (Client-Server)
**ADE:** Google Antigravity
**Related:** [`srs.md`](./srs.md) · [`design.md`](./design.md) · [`todo.md`](./todo.md) · [`claude.md`](./claude.md)

---

## Changelog Philosophy

Dokumen ini adalah **sumber kebenaran tunggal** untuk semua perubahan platform Ta'lim Afkar. Ia berfungsi sebagai:

1. **Memori arsitektur** — AI coding agents di Antigravity membaca ini untuk memahami evolusi sistem
2. **Audit trail** — setiap keputusan teknikal penting tercatat
3. **Koordinasi multi-repo** — perubahan di `talim-afkar-api` dan `talim-afkar-web` dicatat di satu tempat
4. **API contract history** — perubahan breaking pada `@talim-afkar/types` selalu tercatat
5. **Rollback guide** — instruksi jelas saat reversi diperlukan

### Aturan Penulisan Changelog

```
WAJIB ditambahkan saat:
  ✓ Setiap rilis (major, minor, patch)
  ✓ Penambahan dependency baru (di repo manapun)
  ✓ Perubahan database schema
  ✓ Perubahan API contract (endpoint, request/response format)
  ✓ Perubahan @talim-afkar/types (terutama breaking changes)
  ✓ Perubahan arsitektur signifikan
  ✓ Security fix (tanpa detail teknis yang bisa disalahgunakan)
  ✓ Breaking changes dengan migration notes
  ✓ Perubahan AI prompts yang signifikan
  ✓ Perubahan CORS whitelist

FORMAT per entry:
  "- [Repo][Domain] Deskripsi singkat dan jelas dalam Bahasa Indonesia"

  Repo:   [API], [WEB], [SHARED], [INFRA]
  Domain: Auth, Game, Content, Learning, Gamification, Classroom, AI,
          Analytics, Notification, Admin, Infra, Docs, Security, WS, Types

AI AGENT (Antigravity): Setiap kali kamu menyelesaikan task, tambahkan
entry di section [Unreleased] menggunakan workflow /update-changelog
SEBELUM melakukan commit.
```

---

## Versioning Strategy

```
Karena arsitektur terpisah, versi di-track per component:

API versioning:   talim-afkar-api@MAJOR.MINOR.PATCH
Web versioning:   talim-afkar-web@MAJOR.MINOR.PATCH
Types versioning: @talim-afkar/types@MAJOR.MINOR.PATCH

MAJOR — Breaking changes:
  API:    Endpoint dihapus/berubah tidak backward-compatible
  Web:    Architecture rewrite signifikan
  Types:  Interface/type yang sudah ada dihapus atau berubah signature

MINOR — Fitur baru backward-compatible:
  API:    Endpoint baru, field response baru (non-breaking)
  Web:    Halaman/fitur baru
  Types:  Interface/type baru

PATCH — Bug fixes:
  API:    Bug fix, performance improvement
  Web:    UI fix, typo
  Types:  Correction yang backward-compatible

Pre-release:
  0.x.x → Development/internal
  1.0.0 → Public MVP launch

PENTING: Jika @talim-afkar/types berubah MAJOR, WAJIB update
kedua repo (API dan Web) di Sprint yang sama.
```

---

## Release Categories

```
### Added        → Fitur/endpoint/komponen baru
### Changed      → Perubahan pada yang sudah ada
### Deprecated   → Fitur yang akan dihapus
### Removed      → Fitur/endpoint yang dihapus
### Fixed        → Bug fixes
### Security     → Security-related changes
### Performance  → Performance improvements
### Migration    → Database migration instructions
### API Changes  → Perubahan API contract + types
### AI Changes   → Perubahan AI prompts, models, pipelines
### Infra        → Infrastructure dan DevOps
### Docs         → Dokumentasi updates
### Types        → Perubahan @talim-afkar/types
```

---

## [Unreleased]

> Entry di sini menunggu rilis berikutnya.
> AI Agents: tambahkan entry di sini setelah setiap task selesai.

### Added

- [SHARED][Docs] Inisialisasi dokumentasi proyek: `srs.md`, `design.md`, `claude.md`, `todo.md`, `changelog.md`, `README.md`
- [INFRA][Infra] Keputusan arsitektur: Fullstack Tradisional Terpisah (Client-Server) — API dan Web sebagai dua deployment unit independen
- [INFRA][Infra] Definisi stack teknologi: Hono.js (API), Next.js 14 (Web), PostgreSQL + Redis, Socket.io, BullMQ
- [SHARED][Types] Definisi struktur `@talim-afkar/types` package untuk shared type definitions
- [INFRA][Infra] Konfigurasi Antigravity ADE: rules, workflows, skills, browser-allowlist untuk API dan Web

### Architecture Decisions Log (ADL)

> Keputusan arsitektur yang direkam sebagai referensi.

```
ADL-000: Separated Client-Server Architecture
  Tanggal: 2026-05
  Keputusan: Fullstack Tradisional Terpisah — API dan Web terpisah
  Alasan: Deployment independent, team separation, API-first untuk
          future mobile app, clarity untuk AI Agents di Antigravity
  Implikasi: Perlu CORS config, shared types via npm package,
             dua CI/CD pipeline terpisah
  Review: Tidak direncanakan untuk diubah

ADL-001: Modular Monolith First (untuk API)
  Tanggal: 2026-05
  Keputusan: API dibangun sebagai modular monolith, bukan microservices
  Alasan: Complexity microservices tidak justified di skala awal
  Implikasi: Domain boundaries ketat dari hari pertama
  Review: Saat MAU > 50.000 atau ada bottleneck spesifik

ADL-002: PostgreSQL sebagai primary database
  Tanggal: 2026-05
  Keputusan: PostgreSQL 16 (bukan MongoDB atau MySQL)
  Alasan: ACID compliance untuk game scoring, JSON support,
          full-text search, pgvector untuk AI embeddings
  Review: Tidak direncanakan untuk diubah

ADL-003: Redis untuk realtime game state
  Tanggal: 2026-05
  Keputusan: Semua state game real-time di Redis
  Alasan: Latency < 5ms vs < 50ms untuk operasi game kritis
  Mitigasi: Redis AOF persistence, checkpoint ke Postgres tiap soal

ADL-004: Socket.io untuk MVP WebSocket
  Tanggal: 2026-05
  Keputusan: Socket.io (bukan raw WebSocket)
  Alasan: DX lebih baik, fallback transport, rooms management built-in
  Port terpisah: 3002 (berbeda dari REST API 3001)
  Review: Pertimbangkan uWebSockets.js saat > 10.000 concurrent WS

ADL-005: BullMQ untuk job queue
  Tanggal: 2026-05
  Keputusan: BullMQ (Redis-based)
  Alasan: Sudah menggunakan Redis, satu dependency, excellent DX
  Mitigasi: Redis Sentinel untuk produksi

ADL-006: Vercel AI SDK + OpenAI primary
  Tanggal: 2026-05
  Keputusan: Vercel AI SDK sebagai abstraksi, OpenAI primer, Anthropic fallback
  Review: Evaluasi cost vs quality setelah 3 bulan

ADL-007: Vercel untuk Web deployment
  Tanggal: 2026-05
  Keputusan: Next.js Web di-deploy ke Vercel (bukan Railway)
  Alasan: Edge network optimal untuk Next.js, auto CDN, preview deployments
  Catatan: API tetap di Railway/VPS karena butuh WebSocket (stateful)

ADL-008: @talim-afkar/types sebagai shared package
  Tanggal: 2026-05
  Keputusan: Shared types dipublish ke GitHub Packages (private)
  Alasan: Hindari duplikasi type antara API dan Web, single source of truth
  Versioning: Semantic versioning ketat, breaking = MAJOR bump
  Review: Pertimbangkan tRPC jika type inference ingin lebih otomatis (Phase 3)

ADL-009: Antigravity sebagai ADE
  Tanggal: 2026-05
  Keputusan: Google Antigravity sebagai Agentic Development Environment
  Alasan: AI dapat merencanakan, execute, dan validasi secara otonom
  Config: .agents/ folder di setiap repo (rules, workflows, skills)
  Catatan: Mode Planning wajib untuk task besar, terminal guardrails aktif
```

---

## [0.1.0] — MVP Phase Start

> Rilis pertama akan didokumentasikan di sini setelah minggu 1-8 selesai.

---

## Template Rilis

```markdown
## [API/WEB/SHARED vX.Y.Z] — YYYY-MM-DD

### Added

- [API][Domain] Deskripsi fitur/endpoint baru
- [WEB][Domain] Deskripsi komponen/halaman baru
- [SHARED][Types] Type baru ditambahkan ke @talim-afkar/types

### Changed

- [API][Domain] Deskripsi perubahan
- [WEB][Domain] Deskripsi perubahan

### Fixed

- [API][Domain] Deskripsi bug yang diperbaiki
- [WEB][Domain] Deskripsi bug yang diperbaiki

### Security

- [API][Security] Deskripsi security fix (tanpa detail eksploitasi)

### API Changes

> Wajib diisi jika ada perubahan endpoint atau response format

- `POST /api/v1/[endpoint]` — ditambahkan, request: ..., response: ...
- `GET /api/v1/[endpoint]` — response field `old_field` → renamed `new_field`
- @talim-afkar/types@X.Y.Z — interface `[Name]` diubah

### Migration Notes

> Wajib diisi jika ada perubahan database, API contract, atau environment

**Database (jalankan di API):**
\`\`\`bash
npm run db:migrate
\`\`\`

**Environment Variables Baru (API):**
\`\`\`
NEW_API_VAR=value # Deskripsi
\`\`\`

**Environment Variables Baru (Web):**
\`\`\`
NEXT_PUBLIC_NEW_VAR=value # Deskripsi
\`\`\`

**Breaking API Changes (Web perlu update):**

- `GET /api/v1/old-endpoint` → Dihapus, gunakan `GET /api/v1/new-endpoint`
- Response field `old_field` → Renamed ke `new_field`

**@talim-afkar/types Update:**
\`\`\`bash

# Di API dan Web:

npm install @talim-afkar/types@X.Y.Z
\`\`\`

**Rollback Instructions:**
\`\`\`bash

# API rollback:

npm run db:migrate:down # jika ada migration
git checkout v[PREVIOUS_API_VERSION]
docker build dan redeploy

# Web rollback:

# Gunakan Vercel dashboard → Deployments → Rollback

\`\`\`

### AI Agent Notes (Antigravity)

> Catatan untuk AI coding agents tentang perubahan yang perlu diperhatikan

- File X diubah — baca komentar di baris Y sebelum modifikasi
- Event baru `Z` ditambahkan — pastikan handler terupdate
- @talim-afkar/types diupdate — re-run `npm install` di kedua repo
```

---

## Rollback Procedures

### API Rollback (Railway / VPS)

```bash
# Railway:
railway rollback [deployment-id]

# VPS / Docker:
docker pull talim-afkar/api:v[PREVIOUS_VERSION]
docker-compose up -d

# Database rollback (BACKUP DULU!):
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d_%H%M%S).sql
npm run db:migrate:down
# Atau ke versi spesifik:
npm run db:migrate:down -- --to [timestamp]
```

### Web Rollback (Vercel)

```bash
# Via Vercel Dashboard:
# Settings → Deployments → pilih versi sebelumnya → Promote to Production

# Via Vercel CLI:
vercel rollback [deployment-url]
```

### Rollback Shared Types

```bash
# Di API dan Web — downgrade ke versi sebelumnya:
npm install @talim-afkar/types@[PREVIOUS_VERSION]
# Pastikan tidak ada TypeScript errors setelah downgrade
npm run type-check
```

### Redis State Rollback

```bash
# Jika Redis data corrupt (game sessions, leaderboard):
redis-cli DEL "lb:global:weekly:[weekKey]"    # Leaderboard akan rebuild otomatis
redis-cli DEL "game:session:*"                 # Flush active sessions (graceful if possible)
redis-cli FLUSHDB                              # NUCLEAR: hanya jika benar-benar perlu
```

---

## Breaking Changes History

| Versi | Repo | Tanggal | Breaking Change | Migration Path |
| ----- | ---- | ------- | --------------- | -------------- |
| —     | —    | —       | Belum ada       | —              |

---

## API Endpoint Registry

> Track semua endpoint yang tersedia. Update saat ada endpoint baru atau dihapus.

### Authentication

| Method | Endpoint                     | Status  | Since |
| ------ | ---------------------------- | ------- | ----- |
| POST   | /api/v1/auth/register        | Planned | 0.1.0 |
| POST   | /api/v1/auth/login           | Planned | 0.1.0 |
| POST   | /api/v1/auth/logout          | Planned | 0.1.0 |
| POST   | /api/v1/auth/refresh         | Planned | 0.1.0 |
| POST   | /api/v1/auth/verify-email    | Planned | 0.1.0 |
| POST   | /api/v1/auth/forgot-password | Planned | 0.1.0 |

### Users & Profile

| Method | Endpoint             | Status  | Since |
| ------ | -------------------- | ------- | ----- |
| GET    | /api/v1/profile      | Planned | 0.1.0 |
| PUT    | /api/v1/profile      | Planned | 0.1.0 |
| GET    | /api/v1/institutions | Planned | 0.1.0 |

### Game

| Method | Endpoint                   | Status  | Since |
| ------ | -------------------------- | ------- | ----- |
| POST   | /api/v1/games/kilat/start  | Planned | 0.1.0 |
| POST   | /api/v1/games/kilat/answer | Planned | 0.1.0 |
| POST   | /api/v1/games/kilat/end    | Planned | 0.1.0 |
| POST   | /api/v1/games/tebak/start  | Planned | 0.2.0 |
| POST   | /api/v1/games/duel/queue   | Planned | 0.3.0 |

### Content

| Method | Endpoint                        | Status  | Since |
| ------ | ------------------------------- | ------- | ----- |
| GET    | /api/v1/content/kitabs          | Planned | 0.1.0 |
| GET    | /api/v1/content/kitabs/:id/babs | Planned | 0.1.0 |
| GET    | /api/v1/questions               | Planned | 0.1.0 |

### Classroom

| Method | Endpoint                      | Status  | Since |
| ------ | ----------------------------- | ------- | ----- |
| POST   | /api/v1/classroom             | Planned | 0.1.0 |
| GET    | /api/v1/classroom/:id         | Planned | 0.1.0 |
| POST   | /api/v1/classroom/join        | Planned | 0.1.0 |
| GET    | /api/v1/classroom/:id/members | Planned | 0.1.0 |

### Leaderboard

| Method | Endpoint                          | Status  | Since |
| ------ | --------------------------------- | ------- | ----- |
| GET    | /api/v1/leaderboard/global        | Planned | 0.1.0 |
| GET    | /api/v1/leaderboard/classroom/:id | Planned | 0.1.0 |

### WebSocket

| Event Direction | Event Name              | Room             | Since |
| --------------- | ----------------------- | ---------------- | ----- |
| Server→Client   | musabaqoh:joined        | musabaqoh:{code} | 0.1.0 |
| Server→Client   | musabaqoh:question      | musabaqoh:{code} | 0.1.0 |
| Server→Client   | musabaqoh:ended         | musabaqoh:{code} | 0.1.0 |
| Client→Server   | musabaqoh:join          | —                | 0.1.0 |
| Client→Server   | musabaqoh:submit_answer | —                | 0.1.0 |

---

## Dependency Changelog

> Track semua perubahan dependencies penting di kedua repo.

### talim-afkar-api Dependencies

| Package | From | To  | Tanggal | Alasan       | Breaking? |
| ------- | ---- | --- | ------- | ------------ | --------- |
| —       | —    | —   | 2026-05 | Inisialisasi | —         |

### talim-afkar-web Dependencies

| Package | From | To  | Tanggal | Alasan       | Breaking? |
| ------- | ---- | --- | ------- | ------------ | --------- |
| —       | —    | —   | 2026-05 | Inisialisasi | —         |

### @talim-afkar/types Changelog

| Version | Tanggal | Perubahan                                      | Impact |
| ------- | ------- | ---------------------------------------------- | ------ |
| 0.1.0   | 2026-05 | Inisialisasi — UserRole, GameMode, APIResponse | —      |

---

## AI Model & Prompt Changelog

| Tanggal | Repo | Komponen | Model | Perubahan    | Impact |
| ------- | ---- | -------- | ----- | ------------ | ------ |
| 2026-05 | —    | —        | —     | Inisialisasi | —      |

---

## Security Incident Log

| Tanggal | Severity | Repo | Komponen | Status | Resolved |
| ------- | -------- | ---- | -------- | ------ | -------- |
| —       | —        | —    | —        | —      | —        |

---

## Deprecated Features

| Fitur | Repo | Deprecated Sejak | Rencana Removal | Pengganti |
| ----- | ---- | ---------------- | --------------- | --------- |
| —     | —    | —                | —               | —         |

---

## Kontributor & Acknowledgments

```
Core Team:
  - Engineering Team Ta'lim Afkar

AI Development Agents (via Google Antigravity):
  - Claude (Anthropic) — primary coding agent
  - Gemini — secondary review agent
  - Cursor — IDE-integrated assistance

@talim-afkar/types Package Maintainers:
  - Engineering Team (breaking changes = human approval required)

Content Contributors:
  - [Nama muallim/ulama yang berkontribusi validasi konten]

Special Thanks:
  - Komunitas pesantren dan ma'had yang memberikan feedback awal
  - Google Antigravity ADE team
```
