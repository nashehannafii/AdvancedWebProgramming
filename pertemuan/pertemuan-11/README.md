# Pertemuan 11: Form, Validation dan Error Handling

## Materi

- Form handling
- Input binding
- Form submission
- Client-side validation
- Server-side validation
- Validation error
- API error
- Error handling
- User feedback

## Praktikum

Membuat form CRUD mahasiswa dengan validasi frontend dan backend.

## Tujuan

Mahasiswa mampu:
1. Membuat form yang aman dengan validasi
2. Menangani error dari API
3. Memberikan feedback pengguna yang baik

## Struktur Project

```
pertemuan-11/
├── backend/
│   ├── src/
│   │   ├── validators/
│   │   │   └── mahasiswa-validator.ts
│   │   ├── routes/
│   │   │   └── mahasiswa.ts
│   │   └── utils/
│   │       └── response.ts
├── frontend/
│   ├── src/
│   │   ├── lib/
│   │   │   └── validators/
│   │   │       └── client-validation.ts
│   │   └── routes/
│   │       └── mahasiswa/
│   │           └── +page.svelte
│   └── svelte.config.js
└── package.json
```