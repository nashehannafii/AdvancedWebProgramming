# Pertemuan 04: HTTP dan Web Service

## Navigasi

[🏠 Beranda](../../README.md) | [🔧 Prerequest](../../prerequest/README.md) | [Pertemuan 01](../pertemuan-01/README.md) | [Pertemuan 02](../pertemuan-02/README.md) | [Pertemuan 03](../pertemuan-03/README.md) | **Pertemuan 04** | [Pertemuan 05](../pertemuan-05/README.md) | ... | [Pertemuan 16](../pertemuan-16/README.md)

---

## Materi

### 1. Client-Server Architecture

```
Client (Browser/Application)  <--HTTP-->  Server (API/Web Service)
        |                                              |
        v                                              v
   User Interface                                  Business Logic
                                                  Database
```

### 2. HTTP Protocol

HTTP (HyperText Transfer Protocol) adalah protokol untuk komunikasi antara client dan server.

#### HTTP Request Structure

```
POST /api/users HTTP/1.1
Host: example.com
Content-Type: application/json
Authorization: Bearer <token>

{
  "nama": "Budi",
  "email": "budi@example.com"
}
```

#### HTTP Response Structure

```
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/users/1

{
  "id": 1,
  "nama": "Budi",
  "email": "budi@example.com"
}
```

### 3. HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve data | ✓ | ✓ |
| POST | Create data | ✗ | ✗ |
| PUT | Replace data | ✓ | ✗ |
| PATCH | Modify data | ✗ | ✗ |
| DELETE | Remove data | ✓ | ✗ |

### 4. HTTP Headers

```typescript
// Request Headers
const headers = {
  "Content-Type": "application/json",
  "Authorization": "Bearer token123",
  "Accept": "application/json",
  "User-Agent": "MyApp/1.0"
};

// Response Headers
const responseHeaders = {
  "Content-Type": "application/json; charset=utf-8",
  "Cache-Control": "max-age=3600",
  "X-Request-Id": "abc-123-def-456"
};
```

### 5. HTTP Status Codes

#### 2xx Success

| Code | Status | Usage |
|------|--------|-------|
| 200 | OK | Request berhasil |
| 201 | Created | Resource baru dibuat |
| 204 | No Content | Delete berhasil, tidak ada content |

#### 4xx Client Error

| Code | Status | Usage |
|------|--------|-------|
| 400 | Bad Request | Invalid request format |
| 401 | Unauthorized | Authentication diperlukan |
| 403 | Forbidden | Tidak diizinkan |
| 404 | Not Found | Resource tidak ditemukan |
| 405 | Method Not Allowed | Method tidak didukung |

#### 5xx Server Error

| Code | Status | Usage |
|------|--------|-------|
| 500 | Internal Server Error | Error pada server |
| 502 | Bad Gateway | Proxy error |
| 503 | Service Unavailable | Server tidak tersedia |

### 6. Request & Response Body

```typescript
interface User {
  id: number;
  nama: string;
  email: string;
}

// Request body
const requestBody: User = {
  id: 1,
  nama: "Alice",
  email: "alice@example.com"
};

// Response body
interface APIResponse<T> {
  data: T;
  meta?: {
    total: number;
    page: number;
    perPage: number;
  };
  error?: string;
}
```

### 7. JSON

JSON (JavaScript Object Notation) adalah format data yang ringan untuk pertukaran data.

```json
{
  "status": "success",
  "data": {
    "id": 1,
    "nama": "Budi",
    "aktif": true,
    "nilai": [85, 90, 78]
  }
}
```

### 8. REST API Principles

REST (Representational State Transfer) memiliki principia:

1. **Client-Server**: Pemisahan concern
2. **Stateless**: Setiap request tidak bergantung pada state sebelumnya
3. **Cacheable**: Response dapat di-cache
4. **Layered System**: Client tidak perlu tahu sekumpulan server
5. **Uniform Interface**: Standardized communication

### 9. Endpoint Design

```typescript
// Good REST design
GET    /api/users              // List semua users
GET    /api/users/:id          // Detail user spesifik
POST   /api/users              // Buat user baru
GET    /api/users/:id/posts    // Posts milik user
POST   /api/users/:id/posts    // Buat post milik user
PUT    /api/users/:id          // Replace user
PATCH  /api/users/:id          // Update sebagian user
DELETE /api/users/:id          // Delete user
```

## Praktikum

**Tujuan**: Membuat HTTP server sederhana menggunakan Bun tanpa framework.

### Implementasi Server Dasar (`src/server.ts`)

```typescript
// Simple HTTP server using Bun native API
const server = new Bun.Server({
  port: 3000,
  fetch(request) {
    const url = new URL(request.url);
    const method = request.method;
    
    const routes = {
      "/": () => new Response("Hello World", { status: 200 }),
      "/api/status": () => new Response(JSON.stringify({ status: "ok" }), {
        headers: { "Content-Type": "application/json" },
        status: 200
      })
    };
    
    // Route: GET /
    if (method === "GET" && url.pathname === "/") {
      return new Response("Welcome to API", { status: 200 });
    }
    
    // Route: GET /api/status
    if (method === "GET" && url.pathname === "/api/status") {
      const response = { status: "ok", timestamp: new Date().toISOString() };
      return new Response(JSON.stringify(response), {
        headers: { "Content-Type": "application/json" },
        status: 200
      });
    }
    
    // Route: POST /api/users
    if (method === "POST" && url.pathname === "/api/users") {
      const body = await request.json();
      console.log("Created user:", body);
      return new Response(JSON.stringify({ id: 1, ...body }), {
        headers: { "Content-Type": "application/json" },
        status: 201
      });
    }
    
    // 404 Not Found
    return new Response(JSON.stringify({ error: "Not Found" }), {
      headers: { "Content-Type": "application/json" },
      status: 404
    });
  }
});

console.log(`Server running on http://localhost:${server.port}`);
```

### Test dengan Fetch API

```typescript
// Test client
async function testAPI() {
  // GET
  const response1 = await fetch("http://localhost:3000/");
  console.log("GET /:", await response1.text());
  
  // POST
  const response2 = await fetch("http://localhost:3000/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ nama: "Budi", email: "budi@example.com" })
  });
  console.log("POST /api/users:", await response2.json());
  
  // 404
  const response3 = await fetch("http://localhost:3000/notfound");
  console.log("GET /notfound:", await response3.json());
}

testAPI();
```

### Run Server

```bash
bun run src/server.ts
```

## Tugas Tambahan

1. Tambahkan endpoint CRUD untuk resource mahasiswa
2. Implementasikan validasi request body
3. Tambahkan CORS headers

## Referensi

- [MDN: HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [REST API Tutorial](https://restfulapi.net/)