# Ta'lim Afkar — Software Requirements Specification (SRS)

**Version:** 1.0.0  
**Status:** Living Document  
**Last Updated:** 2026-05  
**Maintainers:** Engineering Team + AI Development Agents  
**Related:** [`design.md`](./design.md) · [`todo.md`](./todo.md) · [`claude.md`](./claude.md) · [`changelog.md`](./changelog.md)

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Product Vision](#2-product-vision)
3. [Business Goals](#3-business-goals)
4. [User Personas](#4-user-personas)
5. [User Roles & Permissions](#5-user-roles--permissions)
6. [Functional Requirements](#6-functional-requirements)
7. [Non-Functional Requirements](#7-non-functional-requirements)
8. [Game Mechanics Specification](#8-game-mechanics-specification)
9. [Multiplayer & Realtime Requirements](#9-multiplayer--realtime-requirements)
10. [AI Feature Requirements](#10-ai-feature-requirements)
11. [Gamification Systems](#11-gamification-systems)
12. [Learning Progression Systems](#12-learning-progression-systems)
13. [Admin Dashboard Requirements](#13-admin-dashboard-requirements)
14. [Muallim/Teacher Systems](#14-muallimteacher-systems)
15. [Moderation Systems](#15-moderation-systems)
16. [Analytics Requirements](#16-analytics-requirements)
17. [Notification Systems](#17-notification-systems)
18. [Security Requirements](#18-security-requirements)
19. [Scalability Requirements](#19-scalability-requirements)
20. [Accessibility Considerations](#20-accessibility-considerations)
21. [SEO Strategy](#21-seo-strategy)
22. [API Overview](#22-api-overview)
23. [Risk Analysis](#23-risk-analysis)
24. [Technical Constraints](#24-technical-constraints)
25. [Monetization Opportunities](#25-monetization-opportunities)
26. [Future Extensibility Roadmap](#26-future-extensibility-roadmap)

---

## 1. Executive Summary

Ta'lim Afkar adalah platform pembelajaran Islam berbasis gamifikasi yang dirancang khusus untuk ekosistem pesantren/mahasantri di Indonesia. Platform ini bukan sekadar LMS atau CBT biasa — ia adalah ekosistem pembelajaran sosial yang menggabungkan kompetisi real-time, penguasaan kitab bertahap, dan kecerdasan buatan adaptif.

Produk ini diposisikan sebagai **"Duolingo untuk kitab kuning"** — familiar secara UX bagi generasi digital native, namun terasa asli dan relevan secara konten Islam dan konteks pesantren.

**Core Value Proposition:**
- Mahasantri belajar fiqih, ushul, dan adab melalui gameplay yang adiktif
- Muallim mengelola kelas dan kompetisi berbasis data real-time
- AI menyesuaikan jalur pembelajaran berdasarkan pola kelemahan individual
- Komunitas pesantren terhubung melalui kompetisi seasonal dan leaderboard

---

## 2. Product Vision

> *"Menjadikan penguasaan ilmu agama semenarik dan semudah bermain — tanpa mengorbankan kedalaman dan keotentikan keilmuan Islam."*

### Vision Pillars

| Pilar | Deskripsi |
|-------|-----------|
| **Kedalaman Konten** | Konten berbasis kitab mu'tabar (At-Tadzhib, Qomiut Tughyan, dll) yang diverifikasi ulama |
| **Keterlibatan (Engagement)** | Mekanisme game yang membuat belajar terasa seperti bermain |
| **Komunitas** | Kompetisi antar mahasantri, antar kelas, antar pesantren |
| **Kecerdasan** | AI yang mengenal pola belajar tiap mahasantri secara individual |
| **Skalabilitas** | Arsitektur yang tumbuh bersama jumlah pengguna dan konten |

---

## 3. Business Goals

### Primary Goals (6–12 bulan pertama)

| # | Goal | Metrik Sukses |
|---|------|---------------|
| BG-01 | Akuisisi pengguna dari lingkungan pesantren | 1.000 MAU aktif dalam 6 bulan |
| BG-02 | Retention mahasantri melalui streak & kompetisi | D7 retention ≥ 40% |
| BG-03 | Adopsi muallim untuk manajemen kelas digital | ≥ 50 kelas aktif dalam 6 bulan |
| BG-04 | Validasi model pembelajaran adaptif | Skor rata-rata post-game naik ≥ 15% setelah 30 hari |

### Secondary Goals (12–24 bulan)

- Ekspansi ke pesantren lintas provinsi
- Kemitraan institusional dengan Kemenag / LP Ma'arif
- Monetisasi melalui fitur premium institusional
- Pembangunan data set konten kitab terstruktur sebagai aset jangka panjang

---

## 4. User Personas

### Persona 1: Arif — Mahasantri Aktif

- **Usia:** 18–23 tahun
- **Konteks:** Mahasiswa di Ma'had atau Pesantren Tinggi
- **Motivasi:** Ingin menguasai fiqih dan ushul fiqh lebih cepat, ingin diakui oleh teman-teman
- **Pain Point:** Metode belajar tekstual membosankan, sulit mengukur kemajuan diri
- **Behavior:** Aktif di sosial media, terbiasa dengan aplikasi mobile, kompetitif
- **Kebutuhan di Platform:** Leaderboard, badge, tantangan harian, duel cepat

### Persona 2: Ustadz Hasan — Muallim / Pengajar

- **Usia:** 30–50 tahun
- **Konteks:** Pengajar kitab di pesantren atau mahad
- **Motivasi:** Ingin memantau pemahaman santri secara individual, tidak hanya ujian akhir
- **Pain Point:** Tidak ada alat untuk melihat perkembangan tiap santri secara real-time
- **Behavior:** Lebih konservatif secara teknologi, butuh antarmuka yang simpel
- **Kebutuhan di Platform:** Dashboard kelas, laporan kemajuan, hosting sesi Musabaqoh Kelas

### Persona 3: Lailatul — Mahasantri Pemula

- **Usia:** 17–19 tahun
- **Konteks:** Baru masuk pesantren, background pendidikan umum
- **Motivasi:** Ingin mengejar ketertinggalan pemahaman kitab
- **Pain Point:** Tidak tahu harus mulai dari mana, merasa tertinggal dari teman
- **Behavior:** Butuh panduan bertahap, lebih suka mode solo
- **Kebutuhan di Platform:** Tangga Ilmu (progressive unlocking), AI tutor, penjelasan kontekstual

### Persona 4: Admin Ma'had — Administrator Institusi

- **Usia:** 25–45 tahun
- **Konteks:** Staff IT atau bidang kurikulum pesantren
- **Motivasi:** Mengelola data mahasantri, memantau aktivitas platform
- **Pain Point:** Tidak ada sistem terpusat untuk memantau pembelajaran digital
- **Kebutuhan di Platform:** Dashboard admin, manajemen pengguna, laporan institusi, impor bulk siswa

---

## 5. User Roles & Permissions

### Role Matrix

| Fitur | Mahasantri | Muallim | Admin Institusi | Super Admin |
|-------|-----------|---------|-----------------|-------------|
| Main game modes | ✅ | ✅ | — | — |
| Lihat leaderboard | ✅ | ✅ | ✅ | ✅ |
| Buat sesi Musabaqoh | ❌ | ✅ | ✅ | ✅ |
| Lihat laporan kelas | ❌ | ✅ (kelas sendiri) | ✅ (institusi) | ✅ (semua) |
| Kelola konten soal | ❌ | Terbatas | ✅ | ✅ |
| Manajemen pengguna | ❌ | ❌ | ✅ (institusi) | ✅ (global) |
| Moderasi laporan | ❌ | ❌ | Terbatas | ✅ |
| Konfigurasi platform | ❌ | ❌ | ❌ | ✅ |
| Akses analytics global | ❌ | ❌ | Terbatas | ✅ |
| Generate AI soal | ❌ | ✅ (review required) | ✅ | ✅ |

### Permission Groups

```
ROLE: mahasantri
  - game:play
  - profile:read_own
  - profile:update_own
  - leaderboard:read
  - achievement:read_own
  - streak:manage_own
  - duel:challenge
  - duel:accept
  - report:create

ROLE: muallim
  - INHERITS: mahasantri
  - classroom:create
  - classroom:manage_own
  - musabaqoh:host
  - student:read (own class)
  - report:classroom
  - question:suggest
  - ai:generate_question (pending review)

ROLE: admin_institusi
  - INHERITS: muallim
  - institution:manage
  - user:manage (institution scope)
  - report:institution
  - question:approve
  - content:manage (institution)

ROLE: super_admin
  - *:* (full access)
```

---

## 6. Functional Requirements

### 6.1 Authentication & Onboarding

| ID | Requirement |
|----|-------------|
| FR-AUTH-01 | Registrasi dengan email/password atau kode undangan institusi |
| FR-AUTH-02 | Login dengan email/password |
| FR-AUTH-03 | OAuth login (Google) |
| FR-AUTH-04 | Verifikasi email wajib sebelum akses penuh |
| FR-AUTH-05 | Reset password via email |
| FR-AUTH-06 | Onboarding wizard: pilih pesantren, level awal, kategori minat |
| FR-AUTH-07 | Placement quiz opsional untuk menentukan level awal |
| FR-AUTH-08 | Muallim dapat mengundang mahasantri dengan kode kelas |

### 6.2 Core Game Modes

#### FR-GAME-01: Kilat Fiqih
- Soal pilihan ganda (4 opsi), timer 15 detik per soal
- Skor berdasarkan kecepatan menjawab: semakin cepat, poin lebih tinggi
- Sesi 10–20 soal per permainan
- Kategori: Fiqih (At-Tadzhib / Matan Abu Syuja')
- Mode: Solo; akan dikembangkan ke mode ranked season

#### FR-GAME-02: Tebak Kitab
- Ditampilkan kutipan matan/teks Arab (atau terjemahan)
- Mahasantri menebak: nama kitab, nama bab, atau nama pengarang
- Tiga level hint yang dapat digunakan (mengurangi poin)
- Penilaian berdasarkan akurasi jawaban

#### FR-GAME-03: Duel Afkar
- Matchmaking real-time antara dua mahasantri
- Soal yang sama dikirim simultan
- Pemenang: siapa yang lebih banyak menjawab benar + lebih cepat
- Rating ELO system: menang naik rating, kalah turun
- Waiting room dengan animasi dan chat terbatas

#### FR-GAME-04: Tangga Ilmu
- Konten dibagi per kitab → per bab → per subtopik
- Setiap node harus di-unlock secara berurutan
- Progress visualisasi: pohon atau tangga interaktif
- Setiap chapter memiliki quiz penutup untuk unlock chapter berikutnya
- Stars system (1–3 bintang) per chapter berdasarkan skor

#### FR-GAME-05: Urutan Dalil
- Ditampilkan sejumlah ayat, hadith, atau dalil yang teracak
- Mahasantri drag-and-drop untuk mengurutkan dengan benar
- Kategori: urutan dalil fiqih, urutan syarat/rukun, urutan sifat wajib Allah
- Skor berdasarkan akurasi urutan dan kecepatan

#### FR-GAME-06: Musabaqoh Kelas
- Muallim membuat sesi dengan room code (6 karakter)
- Mahasantri bergabung melalui kode
- Muallim mengontrol: mulai, jeda, next soal
- Live leaderboard berubah real-time selama sesi
- Soal bisa dipilih muallim atau di-generate AI
- Hasil sesi tersimpan di laporan kelas

### 6.3 Profil Mahasantri

| ID | Requirement |
|----|-------------|
| FR-PROFILE-01 | Avatar/foto profil |
| FR-PROFILE-02 | Level dan XP ditampilkan |
| FR-PROFILE-03 | Riwayat permainan (per mode) |
| FR-PROFILE-04 | Daftar achievement yang diperoleh |
| FR-PROFILE-05 | Streak kalender harian |
| FR-PROFILE-06 | Statistik per kategori (akurasi, kecepatan, dll) |
| FR-PROFILE-07 | ELO rating dan rank Duel Afkar |
| FR-PROFILE-08 | Progres Tangga Ilmu per kitab |

### 6.4 Konten & Soal

| ID | Requirement |
|----|-------------|
| FR-CONTENT-01 | Bank soal terstruktur per kategori, kitab, bab |
| FR-CONTENT-02 | Soal memiliki metadata: tingkat kesulitan, kitab referensi, bab, tag |
| FR-CONTENT-03 | Soal dapat diajukan muallim, di-review admin sebelum publish |
| FR-CONTENT-04 | AI dapat men-generate soal (wajib review muallim sebelum publish) |
| FR-CONTENT-05 | Setiap soal memiliki penjelasan (syarah) yang dapat diakses setelah menjawab |
| FR-CONTENT-06 | Konten teks kitab (matan) untuk Tebak Kitab mode |
| FR-CONTENT-07 | Soal dapat di-flag mahasantri jika ada kesalahan |

---

## 7. Non-Functional Requirements

### 7.1 Performance

| Metrik | Target |
|--------|--------|
| API response time (P95) | < 200ms untuk endpoint non-AI |
| API response time (AI endpoint) | < 3000ms |
| WebSocket latency (game real-time) | < 100ms |
| Page load (LCP) | < 2.5 detik |
| Time to Interactive | < 3.5 detik |
| Musabaqoh: max concurrent sessions | 500 sesi simultan (MVP target) |
| Duel Afkar: matchmaking time | < 10 detik |

### 7.2 Reliability

| Metrik | Target |
|--------|--------|
| Uptime SLA | 99.5% (MVP), 99.9% (Scale phase) |
| Game session recovery | Auto-resume jika koneksi putus < 30 detik |
| Data loss tolerance | 0% untuk data progress pengguna |
| Backup frequency | Harian + point-in-time recovery |

### 7.3 Keamanan

- Semua data sensitif dienkripsi at-rest dan in-transit
- JWT dengan refresh token rotation
- Rate limiting pada semua endpoint publik
- Input sanitization dan SQL injection prevention
- OWASP Top 10 compliance

### 7.4 Maintainability

- Test coverage ≥ 80% untuk core domain logic
- Dokumentasi API auto-generated (OpenAPI 3.0)
- Semua modul dapat di-develop dan di-test secara independen
- Kode harus dapat dipahami AI coding agents (lihat `claude.md`)

---

## 8. Game Mechanics Specification

### 8.1 Sistem Poin

```
Kilat Fiqih:
  - Jawaban benar + 15 detik sisa: 100 poin
  - Jawaban benar + 10-14 detik: 80 poin
  - Jawaban benar + 5-9 detik: 60 poin
  - Jawaban benar + 0-4 detik: 40 poin
  - Jawaban salah: 0 poin (no penalty)
  - Bonus streak 3 benar berturut: +20 poin
  - Bonus streak 5 benar berturut: +50 poin

Tebak Kitab:
  - Jawaban tepat (nama kitab + bab): 150 poin
  - Jawaban tepat (hanya kitab): 80 poin
  - Hint level 1 digunakan: -20 poin dari total
  - Hint level 2 digunakan: -40 poin dari total
  - Hint level 3 digunakan: -60 poin dari total

Duel Afkar:
  - Menang: +25 ELO (base, dimodifikasi selisih rating)
  - Kalah: -15 ELO (base)
  - Draw: 0 ELO
  - XP hadiah: 200 XP (menang), 50 XP (kalah)

Urutan Dalil:
  - Semua urutan benar: 200 poin
  - Per posisi benar: 20 poin
  - Bonus kecepatan: 0–50 poin berdasarkan waktu
```

### 8.2 Sistem Level & XP

```
Level   | XP Required (total) | Title
--------|---------------------|---------------------------
1       | 0                   | Thalib Mubtadi
2       | 500                 | Thalib Mujid
3       | 1,500               | Thalib Mutawassit
4       | 3,500               | Faqih Mubtadi
5       | 7,000               | Faqih Mujid
6       | 13,000              | Faqih Mutaqaddim
7       | 22,000              | Mujtahid Mubtadi
8       | 35,000              | Mujtahid Mutawassit
9       | 55,000              | Mujtahid Mutaqaddim
10      | 80,000              | 'Alim Mutabahir
```

### 8.3 ELO Rating System (Duel Afkar)

```
Rating Bracket | Title           | Badge Color
---------------|-----------------|------------
< 800          | Thalib          | Bronze
800–1199       | Muta'allim      | Silver
1200–1599      | Mutafaqqih      | Gold
1600–1999      | Faqih           | Platinum
2000+          | Mujtahid        | Diamond
```

Formula ELO standard (K-factor = 32 untuk < 1200, K = 16 untuk ≥ 1200):
```
Expected Score = 1 / (1 + 10^((RatingLawan - RatingKita) / 400))
ΔRating = K × (AktualScore - ExpectedScore)
```

---

## 9. Multiplayer & Realtime Requirements

### 9.1 Duel Afkar — Matchmaking

```
Flow:
  1. Mahasantri klik "Cari Lawan"
  2. Sistem mencari lawan dengan delta ELO ≤ 200 (timeout 5 detik)
  3. Jika tidak ditemukan, expand ke delta ELO ≤ 500 (timeout 5 detik lagi)
  4. Jika masih tidak ditemukan, match dengan bot AI (labeled jelas sebagai AI)
  5. Lobby room dibuat, countdown 3 detik, game dimulai
  6. Setiap soal dikirim simultan ke kedua player via WebSocket
  7. Jawaban dikirim → server memproses dan broadcast hasil ke kedua pihak
  8. Setelah semua soal, hasil dihitung → ELO updated → ditampilkan

Fault Tolerance:
  - Jika salah satu player disconnect > 15 detik: forfeit, lawan menang
  - Reconnect window: 15 detik (game dijeda otomatis)
  - State tersimpan di Redis selama session aktif
```

### 9.2 Musabaqoh Kelas — Room System

```
Room States: LOBBY → ACTIVE → PAUSED → ENDED

Muallim Actions:
  - CREATE_ROOM (set soal, durasi per soal)
  - START_SESSION
  - PAUSE_SESSION
  - RESUME_SESSION
  - NEXT_QUESTION (manual atau otomatis)
  - END_SESSION

Mahasantri Actions:
  - JOIN_ROOM (via kode)
  - SUBMIT_ANSWER
  - VIEW_LEADERBOARD (real-time update)

Broadcasting:
  - Server → semua peserta: QUESTION_START, LEADERBOARD_UPDATE, SESSION_END
  - Mahasantri → server: ANSWER_SUBMIT
  - Server → Muallim: ANSWER_RECEIVED, ROOM_STATUS

Max peserta per room: 200 (MVP), 500 (Scale phase)
```

### 9.3 WebSocket Events Schema

```json
// Client → Server
{ "type": "ANSWER_SUBMIT", "question_id": "...", "answer": "A", "timestamp_ms": 1234567890 }
{ "type": "JOIN_ROOM", "room_code": "ABC123", "user_id": "..." }

// Server → Client
{ "type": "QUESTION_START", "question": {...}, "time_limit_ms": 15000, "seq": 3 }
{ "type": "LEADERBOARD_UPDATE", "ranks": [...] }
{ "type": "OPPONENT_ANSWERED", "correct": true } // Duel only, no answer revealed
{ "type": "SESSION_END", "results": {...} }
```

---

## 10. AI Feature Requirements

### 10.1 AI-Generated Questions

| ID | Requirement |
|----|-------------|
| FR-AI-01 | AI dapat menghasilkan soal pilihan ganda dari input teks kitab |
| FR-AI-02 | AI menghasilkan 4 opsi jawaban dengan 1 benar, distractors yang masuk akal |
| FR-AI-03 | AI menyertakan syarah/penjelasan untuk setiap soal yang dihasilkan |
| FR-AI-04 | Soal AI wajib masuk review queue muallim sebelum dipublish |
| FR-AI-05 | Muallim dapat edit soal AI sebelum approve |
| FR-AI-06 | Sistem melacak performa soal AI vs soal manual (akurasi, completion rate) |

### 10.2 Adaptive Learning Engine

| ID | Requirement |
|----|-------------|
| FR-AI-10 | Sistem menganalisis pola jawaban salah per mahasantri per topik |
| FR-AI-11 | Rekomendasi soal latihan berdasarkan kelemahan yang terdeteksi |
| FR-AI-12 | Penyesuaian tingkat kesulitan soal secara dinamis (spaced repetition) |
| FR-AI-13 | Prediksi topik yang kemungkinan besar akan salah dijawab |
| FR-AI-14 | Weekly learning report: strengths, weaknesses, rekomendasi |

### 10.3 AI Tutor Assistant

| ID | Requirement |
|----|-------------|
| FR-AI-20 | Chat assistant yang dapat menjelaskan materi fiqih dalam konteks kitab |
| FR-AI-21 | AI dapat menjawab pertanyaan "Mengapa jawaban ini benar/salah?" |
| FR-AI-22 | AI hanya menjawab dalam domain konten platform (no hallucination di luar scope) |
| FR-AI-23 | Riwayat percakapan disimpan per mahasantri |
| FR-AI-24 | Rate limiting: 20 pertanyaan per hari (MVP), unlimited (premium) |

### 10.4 Semantic Search Kitab

| ID | Requirement |
|----|-------------|
| FR-AI-30 | Pencarian konten kitab menggunakan semantic similarity (bukan keyword only) |
| FR-AI-31 | Search dapat menemukan soal berdasarkan konsep, bukan hanya kata kunci |
| FR-AI-32 | Hasil search diurutkan berdasarkan relevansi dan level mahasantri |

### 10.5 Knowledge Graph

| ID | Requirement |
|----|-------------|
| FR-AI-40 | Topik-topik dalam satu kitab memiliki relasi konseptual yang terstruktur |
| FR-AI-41 | Visualisasi knowledge graph per kitab (peta konsep interaktif) |
| FR-AI-42 | Sistem menggunakan graph untuk menentukan urutan optimal belajar |

---

## 11. Gamification Systems

### 11.1 Daily Streak

- Streak dihitung jika mahasantri menyelesaikan minimal 1 game session per hari
- Streak freeze: 2 kali per minggu (dapat diklaim jika streak ≥ 7 hari)
- Bonus XP: streak × 10% XP tambahan per sesi (max 100%)
- Visual kalender streak di profil
- Notifikasi pengingat jika belum main di hari itu (jam 20.00 WIB)

### 11.2 Achievement Badge System

| Kategori | Contoh Badge | Trigger |
|----------|-------------|---------|
| Streak | "Seminggu Penuh" | 7 hari streak |
| Streak | "Ramadan Istiqomah" | 30 hari streak di bulan Ramadan |
| Duel | "Juara Duel" | 10 duel menang berturut |
| Mastery | "Hafidz At-Tadzhib" | Selesaikan seluruh chapter At-Tadzhib |
| Speed | "Kilat" | 10 soal Kilat Fiqih benar semua < 5 detik |
| Social | "Guru Sejati" | Muallim dengan 50 mahasantri aktif di kelas |
| Rare | "Diamond Mujtahid" | Raih ELO 2000+ |

### 11.3 Leaderboard System

| Type | Scope | Reset Period |
|------|-------|-------------|
| Global | Semua pengguna | Weekly + All-time |
| Institusi | Per pesantren/ma'had | Weekly + Monthly |
| Kelas | Per kelas muallim | Per semester |
| Game Mode | Per mode (Kilat, Duel, dll) | Weekly |
| Seasonal | Event khusus (Ramadan, dll) | Per event |

### 11.4 Seasonal Events

```
Ramadan Challenge:
  - Mode khusus: Soal tematik Puasa, Zakat, Lailatul Qadar
  - Bonus XP 2x selama bulan Ramadan
  - Badge eksklusif Ramadan
  - Countdown visual Iftar/Sahur

Muharram Challenge:
  - Soal tematik hijriah, sejarah Islam awal
  - Event leaderboard 10 hari

Maulid Celebration:
  - Soal biografi Nabi
  - Reward eksklusif
```

---

## 12. Learning Progression Systems

### 12.1 Tangga Ilmu — Kitab Progression

```
Kitab: At-Tadzhib (Fiqih)
├── Bab Thaharah
│   ├── Subtopik: Definisi & Jenis Najis
│   ├── Subtopik: Cara Bersuci
│   └── Subtopik: Syarat Wudhu [LOCKED — selesaikan Najis dulu]
├── Bab Shalat [LOCKED — selesaikan Thaharah dulu]
│   └── ...
└── Bab Zakat [LOCKED]
    └── ...
```

### 12.2 Star Rating per Chapter

```
3 Bintang: Skor ≥ 90%, selesai dalam 1 attempt
2 Bintang: Skor ≥ 70%, atau selesai dalam 2 attempt
1 Bintang: Skor ≥ 50%
0 Bintang: Tidak lulus (dapat retry)
```

### 12.3 Mastery Score

Formula mastery per topik:
```
Mastery = (Accuracy × 0.5) + (Recency × 0.3) + (Consistency × 0.2)

Accuracy   = % jawaban benar sepanjang waktu
Recency    = bobot jawaban terbaru lebih tinggi (decaying average)
Consistency= variasi rendah antar sesi → skor tinggi
```

Visualisasi mastery: progress ring per bab, peta warna per subtopik.

---

## 13. Admin Dashboard Requirements

### 13.1 Super Admin

- **User Management:** CRUD pengguna, impor bulk CSV, reset password, suspend akun
- **Content Management:** Approve/reject soal dari muallim/AI, edit soal, manage bank soal
- **Institution Management:** Daftar institusi, verifikasi pesantren, konfigurasi per institusi
- **Analytics Global:** DAU/MAU, retention cohort, funnel onboarding, revenue (jika monetisasi aktif)
- **System Health:** Error rate, latency P95, WebSocket active connections, queue depth
- **Feature Flags:** Toggle fitur per institusi atau global
- **AI Management:** Monitor AI usage, review AI-generated soal, konfigurasi model

### 13.2 Admin Institusi

- **User Management (scope institusi):** Daftar mahasantri, muallim per institusi
- **Classroom Overview:** Aktivitas per kelas, muallim aktif, rata-rata skor institusi
- **Report:** Export laporan kemajuan per mahasantri, per kelas, per semester
- **Content (scope institusi):** Buat soal khusus institusi, manage konten lokal

---

## 14. Muallim/Teacher Systems

### 14.1 Classroom Management

- Buat, edit, arsipkan kelas
- Generate kode undangan kelas (expires in 30 hari atau custom)
- Lihat daftar mahasantri per kelas dengan statistik ringkas
- Assign materi/kitab tertentu untuk kelas

### 14.2 Musabaqoh Kelas — Hosting

- Pilih bank soal (existing atau custom) untuk sesi
- Set durasi per soal (5–60 detik)
- Control panel: Start, Pause, Next Soal, End
- Tampilkan leaderboard ke layar besar (presenter mode)
- Download hasil sesi sebagai PDF/Excel

### 14.3 Progress Monitoring

- Lihat kemajuan tiap mahasantri: topik mana yang lemah
- Bandingkan performa antar mahasantri (anonymized jika setting privasi aktif)
- Alert: "Mahasantri X belum aktif 7 hari"
- Rekomendasi AI: "Fokuskan kelas pada topik Wudhu — 60% mahasantri salah di area ini"

### 14.4 Question Contribution

- Form pengajuan soal dengan metadata lengkap
- Preview soal sebelum submit
- Status tracking: Draft → Submitted → Under Review → Approved/Rejected
- Feedback dari admin jika soal ditolak

---

## 15. Moderation Systems

### 15.1 Content Moderation

| Trigger | Action |
|---------|--------|
| Mahasantri flag soal | Masuk review queue admin |
| AI-generated soal | Otomatis masuk review queue muallim |
| Edit soal yang sudah approved | Otomatis re-review |
| Komentar/chat Duel Afkar | Filter otomatis kata tidak pantas |

### 15.2 User Moderation

| Violation | Action |
|-----------|--------|
| Nama profil tidak pantas | Auto-flag, admin review |
| Laporan dari pengguna lain | Review queue, 24 jam response time |
| Cheat detection (jawaban terlalu cepat) | Flag untuk review, data analysis |
| Akun duplikat | Suspend pending review |

---

## 16. Analytics Requirements

### 16.1 Learning Analytics

- Per mahasantri: accuracy trend, velocity trend, topik kuat/lemah
- Per kelas: distribusi skor, engagement rate, completion rate
- Per soal: difficulty index, discrimination index, distractor analysis
- Per kitab/bab: mastery rate distribusi

### 16.2 Product Analytics

| Metrik | Dashboard |
|--------|-----------|
| DAU, WAU, MAU | Super Admin |
| Retention D1, D7, D30 | Super Admin |
| Funnel Onboarding | Super Admin |
| Mode paling populer | Super Admin + Muallim |
| Session length per mode | Super Admin |
| Streak distribution | Super Admin |
| Leaderboard participation rate | Super Admin |

### 16.3 AI Analytics

- Token usage per fitur AI
- Acceptance rate soal AI oleh muallim
- AI tutor conversation volume dan satisfaction
- Adaptive recommendation click-through rate

---

## 17. Notification Systems

### 17.1 Tipe Notifikasi

| Trigger | Channel | Waktu |
|---------|---------|-------|
| Pengingat streak harian | Push + Email | 20.00 WIB |
| Duel challenge dari teman | Push | Real-time |
| Musabaqoh akan dimulai | Push | 15 menit sebelum |
| Achievement baru | In-app | Real-time |
| Mahasantri bergabung kelas (ke muallim) | In-app + Email | Harian digest |
| Soal AI siap direview (ke muallim) | In-app + Email | Real-time |
| Weekly learning report | Email | Senin pagi |

### 17.2 Notification Preferences

- Mahasantri dapat disable/enable per kategori notifikasi
- Push notification memerlukan explicit permission
- Email unsubscribe wajib tersedia di setiap email

---

## 18. Security Requirements

### 18.1 Authentication Security

- Password: bcrypt (cost factor 12+)
- JWT access token: 15 menit expiry
- Refresh token: 30 hari, rotation on use, stored in HTTP-only cookie
- Rate limiting login: 5 gagal → 15 menit lockout

### 18.2 API Security

- HTTPS wajib untuk semua endpoint
- CORS dikonfigurasi strict (whitelist domain)
- Input validation di semua endpoint (schema validation)
- SQL injection prevention (ORM parameterized queries)
- XSS prevention (output encoding, CSP headers)
- CSRF protection untuk semua state-changing requests

### 18.3 Data Privacy

- Mahasantri di bawah 18 tahun: minimal data collection, persetujuan wali
- Data tidak dijual ke pihak ketiga
- Data dapat dihapus atas permintaan (right to erasure)
- Audit log untuk akses data sensitif

### 18.4 Game Integrity

- Server-side answer validation (client tidak pernah dipercaya untuk skor)
- Timestamp validation: jawaban dengan timestamp tidak valid = reject
- Anti-cheat: deteksi pola jawaban tidak wajar (< 500ms response time)
- ELO manipulation detection: win rate > 95% dalam 50 game → flag

---

## 19. Scalability Requirements

### 19.1 Target Kapasitas

| Fase | MAU | Concurrent Users | Musabaqoh Sessions |
|------|-----|-----------------|-------------------|
| MVP | 1.000 | 200 | 20 |
| Phase 2 | 10.000 | 2.000 | 200 |
| Phase 3 | 100.000 | 20.000 | 2.000 |
| Scale | 1.000.000+ | 200.000+ | 20.000+ |

### 19.2 Scalability Approach

- Stateless API server → horizontal scaling mudah
- WebSocket server dengan sticky sessions atau Redis pub/sub
- Database: read replicas untuk query analytics
- Cache aggressive untuk konten statis (soal, kitab)
- CDN untuk aset statis dan media
- Queue untuk operasi async (AI generation, email, notifikasi)

---

## 20. Accessibility Considerations

- WCAG 2.1 Level AA compliance sebagai target
- Teks Arab: font yang mendukung harakat dengan benar (Scheherazade New, Amiri)
- Ukuran font minimum 16px untuk teks konten
- Kontras warna minimum 4.5:1
- Keyboard navigation untuk semua fitur inti
- Screen reader labels pada semua elemen interaktif
- Drag-and-drop (Urutan Dalil) memiliki alternatif keyboard

---

## 21. SEO Strategy

- Platform bersifat app-like dengan konten dinamis → SSR/SSG untuk halaman landing, profile publik
- Halaman publik: profil mahasantri, leaderboard, info kitab → SEO-friendly
- Metadata lengkap (OG tags, Twitter cards) untuk sharing
- Konten kitab dapat diindex oleh search engine (jika policy memungkinkan)
- Sitemap auto-generated
- Structured data (JSON-LD) untuk konten edukatif

---

## 22. API Overview

### 22.1 API Style

RESTful API untuk operasi CRUD standar. WebSocket untuk komunikasi real-time game. GraphQL dipertimbangkan untuk fase lanjut (query fleksibel untuk analytics dashboard).

### 22.2 API Domains

```
/api/v1/auth/*          — Authentication
/api/v1/users/*         — User management
/api/v1/profile/*       — Profile operations
/api/v1/games/*         — Game session management
/api/v1/questions/*     — Question bank
/api/v1/content/*       — Kitab content
/api/v1/classroom/*     — Muallim classroom
/api/v1/musabaqoh/*     — Musabaqoh session
/api/v1/leaderboard/*   — Leaderboard data
/api/v1/achievements/*  — Achievement system
/api/v1/ai/*            — AI features (tutor, generation)
/api/v1/admin/*         — Admin operations
/api/v1/analytics/*     — Analytics data
/api/v1/notifications/* — Notification management

ws://host/ws/duel/*         — Duel Afkar WebSocket
ws://host/ws/musabaqoh/*    — Musabaqoh WebSocket
```

---

## 23. Risk Analysis

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Konten tidak akurat secara fiqih | Medium | High | Review wajib oleh ulama/muallim sebelum publish |
| AI hallucination pada tutor | High | High | RAG dari sumber terbatas, disclaimer, review loop |
| Cheating di Duel/Musabaqoh | Medium | Medium | Server-side validation, anti-cheat detection |
| Low retention mahasantri | Medium | High | Focus pada social features, streak, event seasonal |
| Skalabilitas WebSocket | Low (MVP) | High | Arsitektur dengan Redis pub/sub dari awal |
| Adopsi muallim rendah | Medium | High | Onboarding simpel, training, dukungan komunitas |
| Ketergantungan AI cost tinggi | Medium | Medium | Caching response AI, rate limiting, pilih model optimal |

---

## 24. Technical Constraints

- Target pengguna: Indonesia → koneksi internet tidak selalu stabil; offline-ready untuk fitur tertentu
- Dukungan device: smartphone Android/iOS (prioritas), desktop (secondary)
- Teks Arab harus render dengan benar di semua platform
- AI API (OpenAI/Anthropic) memiliki latency dan biaya → bukan untuk fitur real-time kritis
- Budget infrastruktur awal terbatas → prioritas efisiensi resource

---

## 25. Monetization Opportunities

### Freemium Model

| Fitur | Free | Premium (Mahasantri) | Institusi |
|-------|------|---------------------|-----------|
| Semua game mode | ✅ | ✅ | ✅ |
| AI Tutor | 20 Q/hari | Unlimited | Unlimited |
| Leaderboard | ✅ | ✅ | ✅ |
| Analytics detail | — | ✅ | ✅ |
| Offline mode | — | ✅ | ✅ |
| Musabaqoh hosting | — | — | ✅ |
| Custom soal institusi | — | — | ✅ |
| Priority support | — | — | ✅ |
| Branding pesantren | — | — | ✅ |

---

## 26. Future Extensibility Roadmap

| Fase | Fitur Potensial |
|------|----------------|
| 1 tahun | Mobile app native (React Native), notifikasi push native |
| 1.5 tahun | Integrasi dengan sistem akademik pesantren (nilai resmi) |
| 2 tahun | Multi-bahasa (Arab, English), ekspansi ke negara mayoritas Muslim |
| 2 tahun | Marketplace konten (muallim jual kurikulum digital) |
| 3 tahun | Sertifikasi digital yang diakui institusi |
| 3 tahun | Live streaming kajian dengan interactive quiz |
| 5 tahun | API publik untuk integrasi pihak ketiga |
| 5 tahun | VR/AR learning untuk visualisasi konsep fiqih |
