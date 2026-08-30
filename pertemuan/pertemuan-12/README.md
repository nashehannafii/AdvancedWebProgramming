# Pertemuan 12: Authentication dan JWT

## Materi

- Authentication vs Authorization
- Login
- Password hashing
- Access token
- Refresh token
- JWT (JSON Web Token)
- Authorization header
- Protected endpoint

## Praktikum

Membuat sistem login dengan JWT.

## Contoh Endpoint

| Endpoint | Method | Fungsi |
|----------|--------|--------|
| POST /api/login | Login | Mendapatkan access token |
| POST /api/register | Register | Mendaftarkan pengguna baru |
| GET /api/profile | Profile | Mendapatkan data profil (protected) |

## Penggunaan Authorization Header

```
Authorization: Bearer <token>
```

## Struktur Project

```
pertemuan-12/
├── src/
│   ├── controllers/
│   │   └── auth-controller.ts
│   ├── middleware/
│   │   └── auth-middleware.ts
│   ├── models/
│   │   └── user.ts
│   ├── services/
│   │   └── auth-service.ts
│   ├── routes/
│   │   └── auth.ts
│   └── server.ts
├── .env
└── package.json
```