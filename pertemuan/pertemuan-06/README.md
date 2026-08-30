# Pertemuan 06: REST API dengan Elysia

## Materi

- REST API
- Resource
- Endpoint
- Routing
- Route parameter
- Query parameter
- Request body
- Response
- HTTP status code
- Error response
- API documentation

## Praktikum

Membuat CRUD REST API menggunakan Elysia.

## Tujuan

Mahasiswa mampu:
1. Membuat API REST yang komplett
2. Menggunakan HTTP status code yang tepat
3. Menangani error response

## Endpoint Contoh

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| GET | /api/mahasiswa | Daftar mahasiswa |
| GET | /api/mahasiswa/:id | Detail mahasiswa |
| POST | /api/mahasiswa | Tambah mahasiswa |
| PUT | /api/mahasiswa/:id | Update mahasiswa |
| PATCH | /api/mahasiswa/:id | Update parsial |
| DELETE | /api/mahasiswa/:id | Hapus mahasiswa |

## Struktur Project

```
pertemuan-06/
├── src/
│   ├── server.ts
│   └── routes/
│       └── mahasiswa.ts
└── package.json
```