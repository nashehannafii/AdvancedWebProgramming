# Pertemuan 05: Bun dan Elysia

## Materi

- Pengenalan Bun sebagai JavaScript/TypeScript runtime
- Instalasi dan konfigurasi Bun
- Pengenalan Elysia
- Server
- Routing
- Request
- Response

## Praktikum

Membuat web service sederhana menggunakan Bun + Elysia.

## Tujuan

Mahasiswa mampu:
1. Menggunakan Bun sebagai runtime
2. Membuat server dengan Elysia
3. Mengimplementasikan routing dasar

## Struktur Project

```
pertemuan-05/
├── src/
│   └── server.ts
├── package.json
└── tsconfig.json
```

## Contoh Kode

```typescript
import { Elysia } from "elysia";

const app = new Elysia()
  .get("/", () => "Hello World")
  .listen(3000);

console.log(`Server running on http://localhost:${app.port}`);
```