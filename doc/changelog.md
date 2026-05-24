# Ta'lim Afkar — Changelog

**Format:** [Semantic Versioning 2.0.0](https://semver.org)  
**Status:** Living Document  
**Last Updated:** 2026-05  
**Related:** [`srs.md`](./srs.md) · [`design.md`](./design.md) · [`todo.md`](./todo.md) · [`claude.md`](./claude.md)

---

## Changelog Philosophy

Dokumen ini adalah **sumber kebenaran tunggal** untuk semua perubahan platform Ta'lim Afkar. Ia berfungsi sebagai:

1. **Memori arsitektur** — AI coding agents membaca ini untuk memahami evolusi sistem
2. **Audit trail** — setiap keputusan teknikal penting tercatat
3. **Koordinasi** — multiple developers/agents tidak saling tumpang tindih
4. **Rollback guide** — instruksi jelas saat reversi diperlukan

### Aturan Penulisan Changelog

```
WAJIB ditambahkan saat:
  ✓ Setiap rilis (major, minor, patch)
  ✓ Penambahan dependency baru
  ✓ Perubahan database schema
  ✓ Perubahan API contract (endpoint, request/response format)
  ✓ Perubahan arsitektur signifikan
  ✓ Security fix (tanpa detail teknis yang bisa disalahgunakan)
  ✓ Breaking changes dengan migration notes
  ✓ Penambahan/perubahan AI prompts yang signifikan

FORMAT per entry:
  "- [Domain] Deskripsi singkat dan jelas dalam Bahasa Indonesia"
  Domain: Auth, Game, Content, Learning, Gamification, Classroom, AI,
          Analytics, Notification, Admin, Infra, Docs, Security, WS

AI AGENT: Setiap kali kamu menyelesaikan task, tambahkan entry
di section [Unreleased] sebelum melakukan commit.
```

---

## Versioning Strategy

```
MAJOR.MINOR.PATCH

MAJOR (X.0.0) — Breaking changes yang memerlukan:
  - Database migration yang tidak backward compatible
  - API contract changes yang memerlukan client update
  - Architectural shifts besar
  - Contoh: 2.0.0 → microservices extraction

MINOR (0.X.0) — Fitur baru backward compatible:
  - Game mode baru
  - Fitur AI baru
  - Dashboard baru
  - Contoh: 1.3.0 → tambah Duel Afkar

PATCH (0.0.X) — Bug fixes dan improvements kecil:
  - Bug fix
  - Performance improvement
  - Copy/content fix
  - Security patches minor
  - Contoh: 1.2.4 → fix ELO calculation bug

Pre-release:
  0.x.x → Development/internal (sebelum public launch)
  1.0.0 → Public MVP launch
```

---

## Release Categories

Setiap rilis menggunakan section berikut (hanya sertakan yang relevan):

```
### Added        → Fitur baru
### Changed      → Perubahan pada fitur yang sudah ada
### Deprecated   → Fitur yang akan dihapus di versi mendatang
### Removed      → Fitur yang sudah dihapus
### Fixed        → Bug fixes
### Security     → Security-related changes
### Performance  → Performance improvements
### Migration    → Instruksi migration untuk developers/ops
### AI Changes   → Perubahan pada AI prompts, models, atau pipelines
### Infra        → Infrastructure dan DevOps changes
### Docs         → Dokumentasi updates
```

---

## [Unreleased]
> Entry di sini menunggu rilis berikutnya. AI agents: tambahkan entry di sini setelah setiap task selesai.

### Added
- [Docs] Inisialisasi dokumentasi proyek: `srs.md`, `design.md`, `claude.md`, `todo.md`, `changelog.md`
- [Infra] Definisi stack teknologi: Node.js + Hono.js (API), Next.js 14 (Web), PostgreSQL + Redis, Socket.io, BullMQ

### Architecture Decisions Log (ADL)
> Keputusan arsitektur awal yang direkam sebagai referensi.

```
ADL-001: Modular Monolith First
  Tanggal: 2026-05
  Keputusan: Mulai dengan modular monolith, bukan microservices
  Alasan: Complexity microservices tidak justified di skala awal
  Implikasi: Domain boundaries ketat dari hari pertama
  Review: Saat MAU > 50.000 atau ada bottleneck spesifik

ADL-002: PostgreSQL sebagai primary database
  Tanggal: 2026-05
  Keputusan: PostgreSQL (bukan MongoDB atau MySQL)
  Alasan: ACID compliance untuk game scoring, JSON support, full-text search, pgvector
  Implikasi: Schema changes butuh migration, tapi lebih aman
  Review: Tidak direncanakan untuk diubah

ADL-003: Redis untuk realtime game state
  Tanggal: 2026-05
  Keputusan: Semua state game real-time di Redis, bukan PostgreSQL
  Alasan: Latency < 5ms vs < 50ms untuk operasi game kritis
  Implikasi: State game hilang jika Redis restart tanpa persistence
  Mitigasi: Redis AOF persistence aktif, checkpoint ke Postgres tiap soal

ADL-004: Socket.io untuk MVP WebSocket
  Tanggal: 2026-05
  Keputusan: Socket.io (bukan raw WebSocket atau uWebSockets.js)
  Alasan: DX lebih baik, fallback transport, rooms management built-in
  Tradeoff: Sedikit lebih lambat dari raw WS, tapi cukup untuk MVP
  Review: Pertimbangkan migrasi ke uWebSockets.js saat > 10.000 concurrent WS

ADL-005: BullMQ untuk job queue
  Tanggal: 2026-05
  Keputusan: BullMQ (Redis-based) bukan RabbitMQ atau SQS
  Alasan: Sudah menggunakan Redis, satu dependency, excellent DX
  Implikasi: Redis menjadi single point of failure untuk queue juga
  Mitigasi: Redis Sentinel atau Cluster untuk produksi

ADL-006: Vercel AI SDK + OpenAI primary
  Tanggal: 2026-05
  Keputusan: Vercel AI SDK sebagai abstraksi, OpenAI sebagai provider utama
  Alasan: Multi-provider switching mudah, streaming support, TypeScript native
  Fallback: Anthropic Claude sebagai secondary provider
  Review: Evaluasi biaya vs kualitas setelah 3 bulan penggunaan

ADL-007: Turborepo untuk monorepo
  Tanggal: 2026-05
  Keputusan: Turborepo (bukan Nx atau Lerna)
  Alasan: Simpler setup, caching build efisien, sesuai skala proyek
  Implikasi: Terikat pada Turborepo conventions
```

---

## [0.1.0] — MVP Phase Start
> Rilis pertama akan didokumentasikan di sini

---

## Template Rilis

Gunakan template berikut untuk setiap rilis baru:

```markdown
## [X.Y.Z] — YYYY-MM-DD

### Added
- [Domain] Deskripsi fitur baru

### Changed
- [Domain] Deskripsi perubahan

### Fixed
- [Domain] Deskripsi bug yang diperbaiki

### Security
- [Security] Deskripsi security fix (tanpa detail eksploitasi)

### Migration Notes
> Wajib diisi jika ada perubahan database, API, atau environment

**Database:**
```bash
npm run db:migrate
```

**Environment Variables Baru:**
```
NEW_VAR=value  # Deskripsi
```

**Breaking API Changes:**
- `GET /api/v1/old-endpoint` → Dihapus, gunakan `GET /api/v1/new-endpoint`
- Response field `old_field` → Renamed ke `new_field`

**Rollback Instructions:**
```bash
# Jika perlu rollback ke versi sebelumnya:
npm run db:migrate:down -- --to 20260601000000
git checkout v[PREVIOUS_VERSION]
```

### AI Agent Notes
> Catatan untuk AI coding agents tentang perubahan yang perlu diperhatikan

- File X diubah — baca komentar di baris Y sebelum modifikasi
- Event baru `Z` ditambahkan — pastikan handler terupdate
- Skema Zod untuk [entity] berubah — update semua yang menggunakan
```

---

## Rollback Procedures

### Database Rollback

```bash
# Check migration status
npm run db:migrate:status

# Rollback satu migration terakhir
npm run db:migrate:down

# Rollback ke versi spesifik (timestamp)
npm run db:migrate:down -- --to 20260601000000

# PERINGATAN: Selalu backup sebelum rollback di production!
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Application Rollback

```bash
# Docker-based rollback
docker pull talim-afkar/api:v[PREVIOUS_VERSION]
docker pull talim-afkar/web:v[PREVIOUS_VERSION]
docker-compose up -d

# Atau via deployment platform (Railway, Render):
# Gunakan "Rollback" feature di dashboard
```

### Redis State Rollback

```bash
# Jika Redis data corrupt (game sessions, leaderboard):
# Redis tidak memiliki point-in-time recovery seperti Postgres
# Strategi: flush affected keys, data akan rebuild dari Postgres

redis-cli DEL "lb:global:weekly"         # Leaderboard akan rebuild otomatis
redis-cli FLUSHDB                         # NUCLEAR: hanya jika benar-benar perlu
```

---

## Breaking Changes History

> Section ini mencatat semua breaking changes yang pernah terjadi untuk referensi long-term.

| Versi | Tanggal | Breaking Change | Migration Path |
|-------|---------|-----------------|----------------|
| —     | —       | Belum ada       | —              |

---

## Dependency Changelog

> Track semua perubahan dependencies penting.

| Package | From | To | Tanggal | Alasan | Breaking? |
|---------|------|----|---------|--------|-----------|
| —       | —    | — | —      | Inisialisasi | — |

---

## AI Model & Prompt Changelog

> Track perubahan pada AI integration karena perubahan model atau prompt dapat drastis mempengaruhi output.

| Tanggal | Komponen | Model/Prompt | Perubahan | Impact |
|---------|----------|-------------|-----------|--------|
| 2026-05 | — | — | Inisialisasi | — |

---

## Security Incident Log

> Log insiden keamanan (tanpa detail eksploitasi). Bersifat internal.

| Tanggal | Severity | Komponen | Status | Resolved |
|---------|----------|----------|--------|----------|
| —       | —        | —        | —      | —        |

---

## Deprecated Features

> Track fitur yang sedang dalam proses deprecation.

| Fitur | Deprecated Sejak | Rencana Removal | Pengganti |
|-------|-----------------|-----------------|-----------|
| —     | —               | —               | —         |

---

## Kontributor & Acknowledgments

```
Core Team:
  - Engineering Team Ta'lim Afkar

AI Development Agents:
  - Claude (Anthropic) — primary coding agent
  - Cursor — IDE-integrated assistance
  - Gemini — secondary review agent

Content Contributors:
  - [Nama muallim/ulama yang berkontribusi validasi konten]

Special Thanks:
  - Komunitas pesantren dan ma'had yang memberikan feedback awal
```
