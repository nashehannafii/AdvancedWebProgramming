# Pertemuan 10: Integrasi Svelte dengan REST API

## Materi

- Fetch API
- HTTP request dari frontend
- GET, POST, PUT, DELETE
- Async/await
- Loading state
- Error state
- API response

## Praktikum

Menghubungkan Svelte dengan REST API Elysia.

## Tujuan

Mahasiswa mampu:
1. Mengakses API dari frontend Svelte
2. Mengelola state loading dan error
3. Mensub-krip UI dengan data API

## Arsitektur

```
Svelte Frontend
    |
    | HTTP
    v
Elysia Backend
    |
    v
MySQL
```

## Struktur Project

```
pertemuan-10/
├── src/
│   ├── lib/
│   │   ├── api/
│   │   │   ├── mahasiswa.ts
│   │   │   └── client.ts
│   │   ├── stores/
│   │   │   └── mahasiswa-store.ts
│   │   └── components/
│   │       ├── MahasiswaForm.svelte
│   │       └── MahasiswaList.svelte
│   └── routes/
│       └── +page.svelte
└── svelte.config.js
```