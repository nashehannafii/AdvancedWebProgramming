# Pertemuan 14: Web Security dan Application Architecture

## Materi Keamanan

- CORS
- XSS
- SQL Injection
- CSRF
- Password security
- Token security
- Input validation
- Environment variables

## Materi Architecture

- Separation of concerns
- Controller
- Service
- Repository
- Middleware
- Model
- DTO
- Modular architecture

## Praktikum

Refactoring aplikasi menjadi struktur backend yang lebih terorganisasi dan menerapkan security dasar.

## Struktur Backend

```
src/
├── controllers/
│   └── mahasiswa-controller.ts
├── services/
│   └── mahasiswa-service.ts
├── repositories/
│   └── mahasiswa-repository.ts
├── models/
│   └── mahasiswa.ts
├── middleware/
│   ├── auth-middleware.ts
│   └── error-handler.ts
├── routes/
│   └── api.ts
├── dtos/
│   └── mahasiswa-dto.ts
├── utils/
│   └── validation.ts
└── server.ts
```

## Struktur Frontend

```
frontend/
├── src/
│   ├── lib/
│   │   ├── components/
│   │   ├── stores/
│   │   └── utils/
│   ├── routes/
│   │   ├── +layout.svelte
│   │   └── +page.svelte
│   └── app.d.ts
└── svelte.config.js
```