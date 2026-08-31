# Pertemuan 16: Final Project dan Presentasi

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 15](../pertemuan-15/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini (Final Project), mahasiswa mampu:

- Membangun aplikasi full-stack lengkap: Svelte+TS (frontend) + Bun+Elysia+TS (backend) + MySQL + REST API + JWT/OAuth 2.0
- Menerapkan fitur minimal: authentication, authorization (role), CRUD, validation, error handling, security, responsive frontend
- Menerapkan version control (Git+GitHub), CI/CD (GitHub Actions), deployment di Local Computer, reverse proxy (Caddy/Nginx), dan public access via Cloudflare Tunnel + domain
- Mendokumentasikan arsitektur, alur auth, dan endpoint serta mempresentasikan demo hidup
- Mempertahankan keputusan teknis dan menjawab pertanyaan penguji secara argumentatif

> **Indikator ketercapaian:** Final project memenuhi semua minimal fitur & deployment checklist, demo berjalan di domain, dan presentasi memuaskan.

---
## Deskripsi

Mahasiswa mengembangkan dan mempresentasikan aplikasi full-stack sebagai puncak dari 15 pertemuan sebelumnya. Project harus dideploy di Local Computer dan dapat diakses via domain melalui Cloudflare Tunnel.

## Minimal Requirements

### Frontend
- **Svelte + TypeScript** — component, props, state, store
- **Responsive design** — mobile & desktop friendly
- **User-friendly UI** — loading, error, dan feedback jelas

### Backend
- **Bun + Elysia + TypeScript** — routing, middleware, validation
- **REST API** — resource-based, status code tepat
- **JWT / OAuth 2.0 Authentication** — login, refresh, protected route

### Database
- **MySQL** — minimal 2 tabel berelasi, foreign key, dan JOIN

### Fitur yang Harus Ada

- Authentication & Authorization (role-based: admin/user)
- CRUD lengkap
- REST API + Database integration
- Validation (client & server) + Error handling
- Security (CORS, SQL Injection prevention, XSS sanitasi, .env)
- Testing (unit/API)
- CI/CD (GitHub Actions + self-hosted runner)
- Deployment (Local Computer) + Reverse Proxy (Caddy/Nginx) + **Cloudflare Tunnel**

## Struktur Project

```
final-project/
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── backend/
│   ├── src/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── repositories/
│   │   ├── models/
│   │   ├── middleware/
│   │   ├── routes/
│   │   ├── dtos/
│   │   ├── utils/
│   │   └── server.ts
│   ├── tests/
│   ├── Dockerfile
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── lib/
│   │   ├── routes/
│   │   └── app.d.ts
│   ├── static/
│   ├── tests/
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml
├── Caddyfile
├── cloudflared.yml
└── README.md
```

## Presentasi

- Demo aplikasi
- Penjelasan arsitektur
- Demo authentication flow
- Demo CRUD
- Demo deployment