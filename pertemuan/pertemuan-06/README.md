# Pertemuan 06: REST API dengan Elysia

## Navigasi

[🏠 Beranda](../../README.md) | [🔧 Prerequest](../../prerequest/README.md) | [Pertemuan 01](../pertemuan-01/README.md) | ... | [Pertemuan 05](../pertemuan-05/README.md) | **Pertemuan 06** | [Pertemuan 07](../pertemuan-07/README.md) | ... | [Pertemuan 16](../pertemuan-16/README.md)

---

## Materi

### 1. REST API Principles

REST (Representational State Transfer) adalah arsitektur untuk membuat API yang scalable.

#### Prinsip Utama

| Prinsip | Penjelasan |
|---------|------------|
| **Resource-based** | Setiap resource = satu endpoint |
| **HTTP Verbs** | GET, POST, PUT, PATCH, DELETE |
| **Stateless** | Setiap request standalone |
| **Cacheable** | Response dapat di-cache |
| **Uniform Interface** | Standard communication |

### 2. Resource dan Endpoint

```typescript
// Resource = Represenstasi data
// Collection = Grup resource
// Item = Resource tunggal

// Collection endpoints
GET    /api/users           // List semua users
POST   /api/users           // Buat user baru

// Item endpoints
GET    /api/users/1         // Detail user 1
PUT    /api/users/1         // Replace user 1
PATCH  /api/users/1         // Update sebagian user 1
DELETE /api/users/1         // Delete user 1
```

### 3. Route Parameter

```typescript
// Route parameter mendukung berbagai tipe
.get("/users/:id", ({ params: { id } }) => ({ id }))

// Multiple parameters
.get("/posts/:userId/comments/:commentId", ({ params }) => {
  const { userId, commentId } = params;
  // Fetch comment for user
})

// Route parameter dengan tipe
.get("/users/:id", ({ params: { id } }) => {}, {
  params: t.Object({ id: t.Number() })
})
```

### 4. Query Parameter

```typescript
// GET /api/users?page=1&limit=10&search=john

.get("/api/users", ({ query }) => {
  const { page = 1, limit = 10, search } = query;
  
  // Filter, pagination logic
  return { page, limit, search };
}, {
  query: t.Object({
    page: t.Optional(t.Number()),
    limit: t.Optional(t.Number()),
    search: t.Optional(t.String())
  })
})
```

### 5. Request Body

```typescript
// Validation body dengan Elysia
.post("/api/users", ({ body }) => ({
  id: crypto.randomUUID(),
  ...body,
  createdAt: new Date().toISOString()
}), {
  body: t.Object({
    nama: t.String({ min: 1, max: 100 }),
    email: t.Email(),
    password: t.String({ min: 8 }),
    role: t.Optional(t.Enum("admin", "user", "moderator")),
    isActive: t.Optional(t.Boolean())
  })
})
```

### 6. HTTP Status Code

```typescript
// Success responses
200 OK                    // GET, PUT, PATCH
201 Created               // POST (resource baru)
204 No Content            // DELETE, PUT (tanpa response body)
202 Accepted              // Request diterima, proses async

// Error responses
400 Bad Request           // Invalid input
401 Unauthorized          // Auth diperlukan
403 Forbidden             // Tidak diizinkan
404 Not Found             // Resource tidak ada
409 Conflict              // Duplicate resource
422 Unprocessable Entity  // Validation error
500 Internal Server Error // Error server
```

### 7. Error Response

```typescript
// Standardized error response
interface ErrorResponse {
  success: false;
  message: string;
  errors?: string[];
  code?: string;
}

// Error handling di Elysia
.catch(({ code, error }) => {
  if (code === "VALIDATION") {
    return {
      success: false,
      message: "Validation error",
      errors: error.all.map(e => `${e.path}: ${e.message}`)
    };
  }
  
  return {
    success: false,
    message: "Internal server error",
    code: "INTERNAL_ERROR"
  };
})
```

### 8. API Documentation

```typescript
// Swagger/OpenAPI alternative - manual docs
.get("/docs", () => ({
  title: "User API",
  version: "1.0.0",
  endpoints: [
    {
      method: "GET",
      path: "/api/users",
      description: "List semua users",
      params: "page, limit, search"
    },
    {
      method: "POST",
      path: "/api/users",
      description: "Buat user baru",
      body: "nama, email, password"
    }
  ]
}))
```

## Praktikum

**Tujuan**: Membuat CRUD REST API menggunakan Elysia.

### Endpoint contoh

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| GET | /api/mahasiswa | Daftar mahasiswa |
| GET | /api/mahasiswa/:id | Detail mahasiswa |
| POST | /api/mahasiswa | Tambah mahasiswa |
| PUT | /api/mahasiswa/:id | Update mahasiswa |
| PATCH | /api/mahasiswa/:id | Update parsial |
| DELETE | /api/mahasiswa/:id | Hapus mahasiswa |

### Implementasi (`src/server.ts`)

```typescript
import { Elysia, t } from "elysia";

interface Mahasiswa {
  id: number;
  nama: string;
  nim: string;
  email: string;
  nilai?: number;
  createdAt: string;
}

const mahasiswas: Mahasiswa[] = [];
let nextId = 1;

// Validation schemas
const MahasiswaSchema = t.Object({
  nama: t.String({ min: 2, max: 100 }),
  nim: t.String({ min: 8, max: 8 }),
  email: t.Email(),
  nilai: t.Optional(t.Number({ min: 0, max: 100 }))
});

const UpdateMahasiswaSchema = t.Object({
  nama: t.Optional(t.String({ min: 2, max: 100 })),
  email: t.Optional(t.Email()),
  nilai: t.Optional(t.Number({ min: 0, max: 100 }))
});

const app = new Elysia()
  // Health check
  .get("/health", () => ({ status: "ok", timestamp: new Date().toISOString() }))
  
  // List mahasiswa with pagination
  .get("/api/mahasiswa", ({ query }) => {
    const { page = 1, limit = 10, search = "" } = query;
    const startIndex = (page - 1) * limit;
    
    let filtered = mahasiswas;
    if (search) {
      filtered = filtered.filter(m => 
        m.nama.toLowerCase().includes(search.toLowerCase()) ||
        m.nim.includes(search)
      );
    }
    
    const paginated = filtered.slice(startIndex, startIndex + limit);
    
    return {
      data: paginated,
      meta: {
        total: filtered.length,
        page: Number(page),
        limit: Number(limit),
        totalPages: Math.ceil(filtered.length / limit)
      }
    };
  })
  
  // Get by ID
  .get("/api/mahasiswa/:id", ({ params: { id } }) => {
    const m = mahasiswas.find(m => m.id === Number(id));
    if (!m) {
      throw new Error("Mahasiswa tidak ditemukan");
    }
    return m;
  }, {
    params: t.Object({ id: t.Number() })
  })
  
  // Create
  .post("/api/mahasiswa", ({ body }) => {
    const m: Mahasiswa = {
      id: nextId++,
      ...body,
      createdAt: new Date().toISOString()
    };
    mahasiswas.push(m);
    return new Response(JSON.stringify(m), { status: 201 });
  }, { body: MahasiswaSchema })
  
  // Update (PUT - full replacement)
  .put("/api/mahasiswa/:id", ({ params: { id }, body }) => {
    const index = mahasiswas.findIndex(m => m.id === Number(id));
    if (index === -1) {
      throw new Error("Mahasiswa tidak ditemukan");
    }
    mahasiswas[index] = { ...mahasiswas[index], ...body, id: Number(id) };
    return mahasiswas[index];
  }, {
    params: t.Object({ id: t.Number() }),
    body: MahasiswaSchema
  })
  
  // Partial update (PATCH)
  .patch("/api/mahasiswa/:id", ({ params: { id }, body }) => {
    const index = mahasiswas.findIndex(m => m.id === Number(id));
    if (index === -1) {
      throw new Error("Mahasiswa tidak ditemukan");
    }
    mahasiswas[index] = { ...mahasiswas[index], ...body };
    return mahasiswas[index];
  }, {
    params: t.Object({ id: t.Number() }),
    body: UpdateMahasiswaSchema
  })
  
  // Delete
  .delete("/api/mahasiswa/:id", ({ params: { id } }) => {
    const index = mahasiswas.findIndex(m => m.id === Number(id));
    if (index === -1) {
      throw new Error("Mahasiswa tidak ditemukan");
    }
    const deleted = mahasiswas.splice(index, 1)[0];
    return { message: "Dihapus", mahasiswa: deleted };
  }, {
    params: t.Object({ id: t.Number() })
  })
  
  // Error handling
  .catch(({ error }) => {
    return { success: false, error: error.message };
  })
  
  .listen(3000);

console.log(`Server running on http://localhost:${app.port}`);
console.log(`API: http://localhost:3000/api/mahasiswa`);
```

### Test dengan HTTP Client

```typescript
// test-client.ts
const BASE_URL = "http://localhost:3000";

async function testAPI() {
  // Create
  const created = await fetch(`${BASE_URL}/api/mahasiswa`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      nama: "Budi Prima",
      nim: "20240001",
      email: "budi@example.com",
      nilai: 85.5
    })
  });
  console.log("Created:", await created.json());
  
  // List
  const list = await fetch(`${BASE_URL}/api/mahasiswa`);
  console.log("List:", await list.json());
  
  // Update
  const updated = await fetch(`${BASE_URL}/api/mahasiswa/1`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      nama: "Budi Prima Seth",
      nim: "20240001",
      email: "budi.prima@example.com",
      nilai: 90
    })
  });
  console.log("Updated:", await updated.json());
  
  // Delete
  const deleted = await fetch(`${BASE_URL}/api/mahasiswa/1`, {
    method: "DELETE"
  });
  console.log("Deleted:", await deleted.json());
}

testAPI();
```

## Tugas Tambahan

1. Tambahkan soft delete
2. Implementasikan filtering by nilai
3. Tambahkan rate limiting middleware

## Referensi

- [Elysia REST API Guide](https://elysiajs.com/routing/rest-api.html)