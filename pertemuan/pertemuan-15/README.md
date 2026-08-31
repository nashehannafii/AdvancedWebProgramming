# Pertemuan 15: Testing, CI/CD, Reverse Proxy dan Deployment

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 14](../pertemuan-14/README.md) | [Selanjutnya: Pertemuan 16 ->](../pertemuan-16/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Menulis unit test, API test, dan integration test untuk backend & frontend
- Menggunakan Git workflow (branch, pull request, merge) dengan benar
- Menjelaskan CI/CD (Continuous Integration/Delivery/Deployment) dan membuat pipeline GitHub Actions (test -> build -> deploy)
- Mengonfigurasi self-hosted runner dan mengotomasi deployment ke Local Computer
- Mengonfigurasi reverse proxy (Caddy/Nginx) untuk routing domain -> service, HTTPS, dan Cloudflare Tunnel agar aplikasi lokal dapat diakses via domain tanpa IP publik

> **Indikator ketercapaian:** Push ke `main` memicu GitHub Actions, test lolos, build ter-deploy ke runner lokal, dan aplikasi dapat diakses via domain melalui Tunnel + reverse proxy.

---
## Materi

### 1. Testing

#### Unit Testing (Bun)

```typescript
// tests/mahasiswa.test.ts
import { describe, it, expect } from "bun:test";
import { Validation } from "../src/utils/Validation";

describe("Validation", () => {
  it("harus menolak email tidak valid", () => {
    expect(Validation.isValidEmail("bukan-email")).toBe(false);
  });
  it("harus menerima NIM 8 digit", () => {
    expect(Validation.isValidNIM("20240001")).toBe(true);
  });
});
```

```bash
bun test
```

#### API Testing

```typescript
// tests/api.test.ts
import { describe, it, expect } from "bun:test";

it("GET /api/mahasiswa harus 200", async () => {
  const res = await fetch("http://localhost:3000/api/mahasiswa");
  expect(res.status).toBe(200);
  const body = await res.json();
  expect(Array.isArray(body.data)).toBe(true);
});
```

#### Integration Testing

- Menguji alur `Svelte -> Elysia -> MySQL` secara end-to-end dengan database test terpisah.

### 2. Git Workflow

```
main (protected)
  ^
  |  PR
feature/mahasiswa-crud --> develop --> main
```

```bash
git checkout -b feature/auth-jwt
git add .
git commit -m "feat: tambah JWT auth"
git push origin feature/auth-jwt
# buat Pull Request di GitHub -> review -> merge
```

- **Branch**: `main`, `develop`, `feature/*`, `hotfix/*`
- **Pull Request**: Wajib review sebelum merge ke `main`

### 3. CI/CD dengan GitHub Actions

#### Jenis CI/CD

| Tipe | Deskripsi |
|------|-----------|
| **Continuous Integration** | Setiap push menjalankan test & lint otomatis |
| **Continuous Delivery** | Build artifact siap deploy otomatis |
| **Continuous Deployment** | Deploy ke server otomatis jika test lolos |

#### Contoh `.github/workflows/ci-cd.yml`

```yaml
name: CI/CD
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install
      - run: bun test
      - run: bun run build

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: self-hosted
    steps:
      - run: pm2 restart elysia || bun run src/server.ts &
```

#### Self-hosted Runner

- Runner diinstall di Local Computer agar dapat deploy langsung ke `localhost`.
- Daftar di `Settings -> Actions -> Runners -> New self-hosted runner`.

### 4. Reverse Proxy (Caddy / Nginx)

Reverse proxy meneruskan request domain ke service internal berbeda port.

#### Caddyfile (contoh)

```caddy
myapp.local {
  handle /api/* {
    reverse_proxy localhost:3000
  }
  handle {
    reverse_proxy localhost:5173
  }
  tls internal
}
```

```bash
caddy run --config Caddyfile --adapter caddyfile
```

#### Nginx (alternatif)

```nginx
server {
  listen 80;
  server_name myapp.local;
  location /api/ { proxy_pass http://localhost:3000; }
  location / { proxy_pass http://localhost:5173; }
}
```

- **Port forwarding**: 80 (HTTP) & 443 (HTTPS) -> Caddy/Nginx -> 3000 (Elysia) & 5173 (Svelte)
- **HTTPS**: Caddy otomatis via Let's Encrypt; Nginx manual via certbot.

### 5. Cloudflare Tunnel

Memungkinkan aplikasi lokal diakses via domain tanpa IP publik.

```bash
# Install cloudflared, lalu
cloudflared tunnel login
cloudflared tunnel create myapp-tunnel
cloudflared tunnel route dns myapp-tunnel myapp.example.com
```

#### `cloudflared.yml` (config.yml)

```yaml
tunnel: myapp-tunnel
credentials-file: ~/.cloudflared/myapp-tunnel.json
ingress:
  - hostname: myapp.example.com
    service: http://localhost:80
  - service: http_status:404
```

```bash
cloudflared tunnel run myapp-tunnel
```

- Alur: `Internet -> Domain (myapp.example.com) -> Cloudflare Tunnel -> Local Computer (Caddy) -> Svelte/Elysia`

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