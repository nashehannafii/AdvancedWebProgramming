# Pertemuan 16: Final Project dan Presentasi

Mahasiswa mengembangkan dan mempresentasikan aplikasi full-stack.

## Minimal Requirements

### Frontend
- Svelte + TypeScript
- Responsive design
- User-friendly UI

### Backend
- Bun + Elysia + TypeScript
- REST API
- JWT / OAuth 2.0 Authentication

### Database
- MySQL

### Fitur yang Harus Ada

- Authentication
- Authorization (role-based)
- CRUD
- REST API
- Database integration
- Validation
- Error handling
- Security
- Testing
- CI/CD
- Deployment
- Reverse Proxy
- Cloudflare Tunnel

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