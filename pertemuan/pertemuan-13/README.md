# Pertemuan 13: OAuth 2.0 dan Authorization

## Materi

- OAuth 2.0
- Authentication vs Authorization
- OAuth roles (Client, Resource Owner, Authorization Server, Resource Server)
- Access Token
- Scope
- Authorization Code Flow
- Redirect URI

## Praktikum

Mengimplementasikan login menggunakan OAuth 2.0 melalui identity provider.

## Arsitektur

```
Svelte Frontend
    |
    v
OAuth Provider (Google, GitHub, dll)
    |
    v
Elysia Backend
    |
    v
MySQL
```

## Struktur Project

```
pertemuan-13/
├── backend/
│   ├── src/
│   │   ├── services/
│   │   │   └── oauth-service.ts
│   │   ├── controllers/
│   │   │   └── auth-controller.ts
│   │   ├── middleware/
│   │   │   └── auth-middleware.ts
│   │   └── routes/
│   │       └── auth.ts
├── frontend/
│   └── src/
│       └── routes/
│           └── login.svelte
├── .env
└── package.json
```