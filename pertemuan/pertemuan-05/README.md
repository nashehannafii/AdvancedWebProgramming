# Pertemuan 05: Bun dan Elysia

## Navigasi

[🏠 Beranda](../README.md) | [🔧 Prerequest](../prerequest/README.md) | [Pertemuan 01](pertemuan-01/README.md) | ... | [Pertemuan 04](pertemuan-04/README.md) | **Pertemuan 05** | [Pertemuan 06](pertemuan-06/README.md) | ... | [Pertemuan 16](pertemuan-16/README.md)

---

## Materi

### 1. Pengenalan Bun

Bun adalah runtime JavaScript/TypeScript yang cepat dengan:
- **Bun.sh** - Package manager bersama runtime
- **Bundled sqlite3** - Database integrasi
- **Native TypeScript support** - Tanpa transpilasi

#### Perbandingan Kecepatan

| Runtime | Startup Time | Install Time |
|---------|-------------|--------------|
| Bun | ~100ms | ~2s |
| Node.js | ~500ms | ~10s |
| Deno | ~300ms | ~5s |

### 2. Instalasi Bun

```bash
# Windows (using Scoop atau manual)
curl -fsSL https://bun.sh/install | bash

# atau via npm
npm install -g bun

# Verify installation
bun --version
```

### 3. Konfigurasi Projek

**package.json**

```json
{
  "name": "my-api",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "bun run src/server.ts",
    "start": "bun run src/server.ts",
    "test": "bun test"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

**tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bun",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  }
}
```

### 4. Pengenalan Elysia

Elysia adalah framework TypeScript-first untuk Bun dengan:
- Routing deklaratif
- Plugin system
- Type safety maksimal
- Middleware

#### Instalasi

```bash
mkdir backend && cd backend
bun init
bun add elysia
```

#### Struktur Dasar

```typescript
import { Elysia, t } from "elysia";

const app = new Elysia()
  .get("/", () => "Hello World")
  .listen(3000);

console.log(`Server running on http://localhost:${app.port}`);
```

### 5. Server dan Routing

```typescript
import { Elysia } from "elysia";

const server = new Elysia()
  // GET routes
  .get("/", () => ({ message: "Home" }))
  .get("/users", () => [])
  .get("/users/:id", ({ params: { id } }) => ({ id, name: `User ${id}` }))
  
  // POST routes
  .post("/users", ({ body }) => ({
    ...body,
    createdAt: new Date().toISOString()
  }))
  
  // PUT and PATCH
  .put("/users/:id", ({ params, body }) => ({ ...params, ...body }))
  .patch("/users/:id", ({ params, body }) => ({ ...params, ...body }))
  
  // DELETE
  .delete("/users/:id", ({ params }) => ({ deleted: params.id }))
  
  .listen(3000);

console.log(`Running on http://localhost:${server.port}`);
```

### 6. Request Object

```typescript
new Elysia()
  .get("/debug", ({ request, query, params, headers, body, ip }) => ({
    method: request.method,
    url: request.url,
    query,
    params,
    headers,
    body,
    ip
  }))
```

### 7. Response Object

```typescript
new Elysia()
  .get("/response", () => {
    return new Response("Custom Response", {
      status: 201,
      headers: {
        "X-Custom-Header": "value"
      }
    });
  })
  .get("/json", () => ({ data: "JSON Response" }))
  .get("/redirect", () => new Response(null, { status: 302, headers: { Location: "/" } }))
```

### 8. Middleware

```typescript
new Elysia()
  .use(({ onRequest }) => {
    onRequest(({ request }) => {
      console.log(`${request.method} ${request.url}`);
    });
  })
  .get("/", () => "Hello")
```

## Praktikum

**Tujuan**: Membuat web service sederhana menggunakan Bun + Elysia.

### Struktur Project

```
src/
├── server.ts
└── utils/
    └── logger.ts
```

### Implementasi (`src/server.ts`)

```typescript
import { Elysia, t } from "elysia";

interface User {
  id: number;
  nama: string;
  email: string;
  usia?: number;
}

const users: User[] = [];
let nextId = 1;

const app = new Elysia()
  // Logger middleware
  .use(({ onRequest }) => {
    onRequest(({ request }) => {
      console.log(`[${request.method}] ${request.url}`);
    });
  })
  
  // Health check
  .get("/health", () => ({ status: "ok", timestamp: new Date().toISOString() }))
  
  // Home
  .get("/", () => ({
    message: "API Server with Bun & Elysia",
    version: "1.0.0",
    endpoints: [
      { method: "GET", path: "/health" },
      { method: "GET", path: "/api/users" },
      { method: "GET", path: "/api/users/:id" },
      { method: "POST", path: "/api/users" },
      { method: "PUT", path: "/api/users/:id" },
      { method: "DELETE", path: "/api/users/:id" }
    ]
  }))
  
  // Get all users
  .get("/api/users", () => users)
  
  // Get user by ID
  .get("/api/users/:id", ({ params: { id } }) => {
    const user = users.find(u => u.id === parseInt(id));
    if (!user) {
      return new Response(JSON.stringify({ error: "User not found" }), {
        status: 404,
        headers: { "Content-Type": "application/json" }
      });
    }
    return user;
  }, {
    params: t.Object({ id: t.String() })
  })
  
  // Create user
  .post("/api/users", ({ body }) => {
    const user: User = {
      id: nextId++,
      ...body
    };
    users.push(user);
    return new Response(JSON.stringify(user), { status: 201 });
  }, {
    body: t.Object({
      nama: t.String({ min: 1 }),
      email: t.Email(),
      usia: t.Optional(t.Number({ min: 0, max: 120 }))
    })
  })
  
  // Update user
  .put("/api/users/:id", ({ params: { id }, body }) => {
    const index = users.findIndex(u => u.id === parseInt(id));
    if (index === -1) {
      return new Response(JSON.stringify({ error: "User not found" }), {
        status: 404,
        headers: { "Content-Type": "application/json" }
      });
    }
    users[index] = { ...users[index], ...body, id: parseInt(id) };
    return users[index];
  }, {
    body: t.Object({
      nama: t.String(),
      email: t.Email(),
      usia: t.Optional(t.Number())
    })
  })
  
  // Delete user
  .delete("/api/users/:id", ({ params: { id } }) => {
    const index = users.findIndex(u => u.id === parseInt(id));
    if (index === -1) {
      return new Response(JSON.stringify({ error: "User not found" }), {
        status: 404,
        headers: { "Content-Type": "application/json" }
      });
    }
    const deleted = users.splice(index, 1)[0];
    return { message: "User deleted", user: deleted };
  })
  
  // Error handling
  .catch(({ code, error }) => {
    if (code === "VALIDATION") {
      return new Response(JSON.stringify({ error: error.message }), { status: 400 });
    }
    return new Response(JSON.stringify({ error: "Internal Server Error" }), {
      status: 500
    });
  })
  
  .listen(3000);

console.log(`Server running on http://localhost:${app.port}`);
```

### Run Server

```bash
bun run src/server.ts
```

### Test dengan curl

```bash
# Health check
curl http://localhost:3000/health

# Get users
curl http://localhost:3000/api/users

# Create user
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"nama": "Budi", "email": "budi@example.com", "usia": 25}'

# Update user
curl -X PUT http://localhost:3000/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"nama": "Budi Prima", "email": "budi.prima@example.com"}'

# Delete user
curl -X DELETE http://localhost:3000/api/users/1
```

## Tugas Tambahan

1. Tambahkan authentication middleware
2. Implementasikan pagination untuk list users
3. Tambahkan logging request dengan middleware

## Referensi

- [Bun.sh Documentation](https://bun.sh/docs/)
- [Elysia.js Documentation](https://elysiajs.com/)