# Modul Advanced Web Programming

> **Deskripsi**: Modul pengembangan web full-stack menggunakan TypeScript, Bun, Elysia, Svelte, MySQL, JWT, OAuth 2.0, serta DevOps lengkap mulai dari setup hingga deployment dengan Cloudflare Tunnel.

## Identitas Mata Kuliah

- **Nama**: Pemrograman Web Lanjut
- **Level**: Perguruan Tinggi / Semester Lanjut
- **Pertemuan**: 16

## Deskripsi Prasyarat

Mahasiswa sebaiknya sudah memiliki pengetahuan dasar sebelum mengikuti modul ini:

- **JavaScript/TypeScript Dasar**: Memahami konsep dasar JavaScript ES6+, tipe data, fungsi, dan async/await
- **Web Development Dasar**: HTML, CSS, dan konsep dasar HTTP/REST API
- **Git & GitHub**: Dasar-dasar version control dan working dengan repository
- **Database Dasar**: Konsep relational database dan SQL dasar
- **Command Line**: Menggunakan terminal/command prompt untuk menjalankan perintah

> **Catatan**: Semua prasyarat akan dibahas kembali di **Pertemuan 01** dan **Pertemuan 02** bila diperlukan.

## Teknologi Utama

| Komponen | Teknologi |
|----------|-----------|
| Runtime | Bun |
| Backend Framework | Elysia |
| Frontend Framework | Svelte |
| Database | MySQL |
| API | REST API |
| Auth | JWT & OAuth 2.0 |

## Daftar Pertemuan

| No | Pertemuan | Materi |
|----|-----------|--------|
| 01 | [TypeScript Dasar](./pertemuan/pertemuan-01/README.md) | Modern JS, TypeScript basics |
| 02 | [TypeScript Lanjutan](./pertemuan/pertemuan-02/README.md) | Type, Interface, Generic, Modules |
| 03 | [Sync & Async](./pertemuan/pertemuan-03/README.md) | Promise, async/await, Event Loop |
| 04 | [HTTP & Web Service](./pertemuan/pertemuan-04/README.md) | HTTP protocol, Request/Response |
| 05 | [Bun & Elysia](./pertemuan/pertemuan-05/README.md) | Setup Bun, Routing dasar |
| 06 | [REST API](./pertemuan/pertemuan-06/README.md) | CRUD API dengan Elysia |
| 07 | [Database Integration](./pertemuan/pertemuan-07/README.md) | MySQL, SQL, CRUD database |
| 08 | [Ujian Tengah Semester](./pertemuan/pertemuan-08/README.md) | Project-based assessment |
| 09 | [Svelte Dasar](./pertemuan/pertemuan-09/README.md) | Komponen, Props, State |
| 10 | [Integrasi API](./pertemuan/pertemuan-10/README.md) | Fetch API, HTTP dari frontend |
| 11 | [Form & Validasi](./pertemuan/pertemuan-11/README.md) | Frontend & backend validation |
| 12 | [JWT Authentication](./pertemuan/pertemuan-12/README.md) | Login, Token, Middleware |
| 13 | [OAuth 2.0](./pertemuan/pertemuan-13/README.md) | OAuth flow, Identity Provider |
| 14 | [Security & Architecture](./pertemuan/pertemuan-14/README.md) | CORS, SQLi, Clean Architecture |
| 15 | [CI/CD & Deployment](./pertemuan/pertemuan-15/README.md) | GitHub Actions, Caddy, Tunnel |
| 16 | [Final Project](./pertemuan/pertemuan-16/README.md) | Full-stack beserta deployment |

## Struktur Proyek

```text
AdvancedWebProgramming/
├── prerequest/           # Prerequisites & instalasi
├── pertemuan/
│   ├── pertemuan-01/     # TypeScript dasar
│   ├── pertemuan-02/     # TypeScript lanjutan & modular
│   ├── pertemuan-03/     # Sync & Async programming
│   ├── pertemuan-04/     # HTTP & Web Service
│   ├── pertemuan-05/     # Bun & Elysia
│   ├── pertemuan-06/     # REST API dengan Elysia
│   ├── pertemuan-07/     # Database MySQL & Integrasi
│   ├── pertemuan-08/     # Ujian Tengah Semester
│   ├── pertemuan-09/     # Svelte dasar
│   ├── pertemuan-10/     # Integrasi Svelte dengan API
│   ├── pertemuan-11/     # Form & Validasi
│   ├── pertemuan-12/     # JWT Authentication
│   ├── pertemuan-13/     # OAuth 2.0
│   ├── pertemuan-14/     # Web Security & Architecture
│   ├── pertemuan-15/     # Testing, CI/CD, Deployment
│   └── pertemuan-16/     # Final Project
├── ACUAN.md              # Pedoman modul
├── PERTEMUAN.md          # Detail pertemuan
└── README.md             # File ini
```

## Quick Start

1. Baca [prerequest](prerequest/README.md) untuk instalasi tools
2. Ikuti tiap modul pertemuan untuk belajar secara bertahap
3. Kerjakan final project di [pertemuan-16](../pertemuan/pertemuan-16/README.md)