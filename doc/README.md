# Ta'lim Afkar — Engineering Documentation

> _Platform pembelajaran Islam berbasis gamifikasi untuk ekosistem pesantren dan mahasantri Indonesia._
> **"Duolingo untuk kitab kuning"**

---

## ⚡ Antigravity ADE — Quick Start

Proyek ini dikembangkan menggunakan **Google Antigravity** sebagai Agentic Development Environment.
Antigravity mengubah VS Code menjadi platform orkestrasi AI otonom dengan dua jendela utama:

- **Agent Manager** — orkestrasi tugas, perencanaan, dan visualisasi aliran kerja agen
- **Editor** — editing kode dengan dukungan ekstensi VS Code penuh

### Cara Memulai di Antigravity

```
1. Buka Agent Manager → New Conversation
2. Mode: PLANNING (untuk task kompleks) atau FAST (untuk task isolasi kecil)
3. Jalankan workflow: /onboarding-agent
4. Biarkan agen membaca semua .md sebelum mengerjakan task apapun
```

### Workflows Tersimpan (ketik `/` di Agent Manager)

| Workflow                    | Fungsi                                                     |
| --------------------------- | ---------------------------------------------------------- |
| `/onboarding-agent`         | Onboarding agen baru — baca semua docs + pahami arsitektur |
| `/generate-unit-tests`      | Generate unit tests untuk file yang aktif                  |
| `/review-domain-boundaries` | Cek apakah kode melanggar domain isolation                 |
| `/new-api-endpoint`         | Scaffold endpoint baru sesuai API conventions              |
| `/new-db-schema`            | Scaffold Drizzle schema + migration                        |
| `/deploy-check`             | Pre-deploy checklist untuk API dan Web                     |
| `/update-changelog`         | Tambah entry ke changelog setelah task selesai             |

### Antigravity Rules Location

Semua aturan sistem (code style, API conventions, dll) tersimpan di:

```
api/.agents/rules/
  ├── code-style-guide.md
  ├── api-conventions.md
  ├── domain-boundaries.md
  └── security-rules.md

web/.agents/rules/
  ├── component-conventions.md
  ├── state-management.md
  └── accessibility-rules.md
```

### Antigravity Skills Location

```
api/.agents/skills/
  ├── drizzle-schema/        → Panduan membuat schema Drizzle
  ├── hono-endpoint/         → Panduan scaffold endpoint Hono.js
  └── bullmq-worker/         → Panduan membuat BullMQ worker

web/.agents/skills/
  ├── next-page/             → Panduan membuat page Next.js
  ├── game-component/        → Panduan komponen UI game
  └── socket-client/         → Panduan integrasi Socket.io client
```

---

## Arsitektur: Fullstack Tradisional Terpisah (Client-Server)

```
┌─────────────────────────────────────────────────────────────┐
│                    REPOSITORY STRUCTURE                       │
│                                                               │
│   talim-afkar-api/          talim-afkar-web/                 │
│   (Backend — Hono.js)       (Frontend — Next.js 14)          │
│   Deploy: Railway/VPS       Deploy: Vercel                    │
│   Port: 3001                Port: 3000 (Vercel managed)       │
└─────────────────────────────────────────────────────────────┘
```

> **Keputusan Arsitektur:** Proyek ini menggunakan arsitektur **Client-Server Terpisah**, bukan monorepo. API dan Web adalah dua repository (atau dua root project) yang di-deploy secara independen. Komunikasi via REST API (HTTPS) dan WebSocket.

---

## Dokumen Utama

| Dokumen                          | Deskripsi                                                | Audience             |
| -------------------------------- | -------------------------------------------------------- | -------------------- |
| [`srs.md`](./srs.md)             | Software Requirements Specification — WHAT yang dibangun | Semua                |
| [`design.md`](./design.md)       | Technical Architecture — HOW cara membangunnya           | Engineers, AI Agents |
| [`claude.md`](./claude.md)       | AI Agent Operational Guide — aturan pengembangan         | AI Coding Agents     |
| [`todo.md`](./todo.md)           | Phased Implementation Roadmap — WHEN dan ORDER           | Engineers, PM        |
| [`changelog.md`](./changelog.md) | Changelog & ADL — riwayat semua perubahan                | Semua                |

---

## Platform Identity

```
Nama:     Ta'lim Afkar (تعليم أفكار)
Tagline:  "Duolingo untuk kitab kuning"
Target:   Mahasantri pesantren Indonesia
Produk:   Gamified Islamic Learning Ecosystem

Backend:  Node.js + Hono.js (REST API + WS Server)
Frontend: Next.js 14 App Router (PWA)
Database: PostgreSQL 16 + Redis 7
Realtime: Socket.io dengan Redis Adapter
Queue:    BullMQ (Redis-based)
Deploy:   API → Railway/VPS | Web → Vercel
```

---

## Quick Start untuk AI Agents (via Antigravity)

Jika kamu adalah AI coding agent yang baru bergabung ke proyek ini:

1. **Jalankan `/onboarding-agent`** di Antigravity Agent Manager
2. **Baca `claude.md` dulu** — panduan operasional wajib
3. **Baca `srs.md` section 1–5** — pahami produk dan domain
4. **Baca `design.md` section 1–6** — pahami arsitektur Client-Server terpisah
5. **Cek `todo.md`** — lihat task yang sedang dikerjakan
6. **Update `changelog.md`** setelah setiap task selesai menggunakan `/update-changelog`

### PENTING: Dua Repository, Dua Konteks

```
Saat mengerjakan task API:
  → Bekerja di dalam folder: talim-afkar-api/
  → Base URL komunikasi: NEXT_PUBLIC_API_URL (env var di web)
  → Setiap endpoint baru → update srs.md section 22 + changelog

Saat mengerjakan task Web:
  → Bekerja di dalam folder: talim-afkar-web/
  → Semua API calls via: src/lib/api-client.ts
  → Tidak ada direct DB access dari frontend, PERNAH
```

---

## Repository Links

```
API Repository:  https://github.com/[org]/talim-afkar-api
Web Repository:  https://github.com/[org]/talim-afkar-web
```

---

## Environment Setup (Lokal)

```bash
# 1. Clone kedua repository
git clone https://github.com/[org]/talim-afkar-api
git clone https://github.com/[org]/talim-afkar-web

# 2. Start infrastructure (PostgreSQL + Redis + MailHog)
cd talim-afkar-api
docker-compose -f docker-compose.dev.yml up -d

# 3. Setup API
cd talim-afkar-api
cp .env.example .env
npm install
npm run db:migrate
npm run db:seed
npm run dev          # → http://localhost:3001

# 4. Setup Web (terminal baru)
cd talim-afkar-web
cp .env.example .env
# set NEXT_PUBLIC_API_URL=http://localhost:3001
# set NEXT_PUBLIC_WS_URL=ws://localhost:3002
npm install
npm run dev          # → http://localhost:3000
```
