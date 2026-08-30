# Pertemuan 15: Testing, CI/CD, Reverse Proxy dan Deployment

## Materi Testing

- Unit testing
- API testing
- Integration testing

## Materi Git

- Git workflow
- Branch
- Pull request
- Merge

## Materi CI/CD

- Continuous Integration
- Continuous Delivery
- Continuous Deployment
- GitHub Actions
- Self-hosted Runner
- Automated testing
- Automated build
- Automated deployment

## Materi Reverse Proxy

- Konsep reverse proxy
- Caddy / Nginx
- Domain
- Port forwarding
- HTTPS
- Routing domain ke service

## Materi Tunnel

- Cloudflare Tunnel
- Public access tanpa IP public

## Arsitektur CI/CD

```
Git Push
    |
    v
GitHub
    |
    v
GitHub Actions
    |
    v
Test
    |
    v
Build
    |
    v
Self-hosted Runner
    |
    v
Local Computer
    |
    v
Caddy / Nginx
    |
    +--------> Svelte Frontend
    |
    +--------> Elysia Backend
```

## Struktur Project

```
pertemuan-15/
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── backend/
│   ├── src/
│   ├── tests/
│   └── package.json
├── frontend/
│   ├── src/
│   ├── tests/
│   └── package.json
├── Caddyfile
└── cloudflared.yml
```