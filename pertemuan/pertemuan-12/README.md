# Pertemuan 12: Authentication & JWT

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 11](../pertemuan-11/README.md) | [Selanjutnya: Pertemuan 13 ->](../pertemuan-13/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Menjelaskan perbedaan authentication vs authorization
- Mengimplementasikan register/login dengan password hashing (`bcrypt` dengan salt) dan verifikasi yang aman
- Membuat dan memverifikasi JWT (header.payload.signature), access token & refresh token dengan `exp`/`iat`
- Mengirim token via `Authorization: Bearer <token>` dan melindungi endpoint (`GET /api/profile`) dengan middleware
- Membangun alur login lengkap (POST /api/login, /api/register, /api/refresh, /api/profile) yang aman

> **Indikator ketercapaian:** Login mengembalikan JWT, endpoint protected menolak tanpa token (401) dan mengizinkan dengan token valid; password tersimpan sebagai hash.

---
## Materi

### 1. Authentication vs Authorization

| Konsep | Penjelasan |
|--------|------------|
| **Authentication** | Memverifikasi siapa pengguna (login) |
| **Authorization** | Memeriksa hak akses (role, permission) |

### 2. Login & Password Hashing

```typescript
import { hash, compare } from "bcrypt";

async function hashPassword(password: string): Promise<string> {
  return await hash(password, 12); // salt rounds
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return await compare(password, hash);
}
```

### 3. JWT (JSON Web Token)

JWT terdiri dari 3 bagian: Header, Payload, Signature

```typescript
import jwt from "jsonwebtoken";

interface Payload {
  userId: number;
  email: string;
  role: string;
  iat: number;
  exp: number;
}

// Generate token
function generateToken(payload: { userId: number; email: string; role: string }): string {
  return jwt.sign(payload, process.env.JWT_SECRET!, {
    expiresIn: "1h"
  });
}

// Verify token
function verifyToken(token: string): Payload | null {
  try {
    return jwt.verify(token, process.env.JWT_SECRET!) as Payload;
  } catch {
    return null;
  }
}
```

### 4. Access Token & Refresh Token

```typescript
interface Tokens {
  accessToken: string;
  refreshToken: string;
}

function generateTokens(userId: number, email: string, role: string): Tokens {
  const accessToken = jwt.sign(
    { userId, email, role },
    process.env.JWT_SECRET!,
    { expiresIn: "15m" }
  );
  
  const refreshToken = jwt.sign(
    { userId },
    process.env.JWT_REFRESH_SECRET!,
    { expiresIn: "7d" }
  );
  
  return { accessToken, refreshToken };
}
```

### 5. Authorization Header

```
Authorization: Bearer <token>
```

---

## Contoh Endpoint

| Endpoint | Method | Fungsi |
|----------|--------|--------|
| POST /api/login | Login | Verifikasi credentials |
| POST /api/register | Register | Buat akun baru |
| POST /api/refresh | Refresh | Update access token |
| GET /api/profile | Profile | Data profile (protected) |
| GET /api/users | Users | List users (admin only) |

---

## Implementasi Backend

```typescript
import { Elysia, t } from "elysia";
import jwt from "jsonwebtoken";
import Database from "../config/database";
import { hashPassword, verifyPassword } from "../utils/auth";

const app = new Elysia()
  // Register
  .post("/api/register", async ({ body }) => {
    const { nama, email, password } = body;
    
    // Check existing user
    const [existing] = await Database.execute(
      "SELECT id FROM users WHERE email = ?",
      [email]
    );
    
    if ((existing as any[]).length > 0) {
      throw new Error("Email sudah terdaftar");
    }
    
    // Hash password
    const password_hash = await hashPassword(password);
    
    // Create user
    const [result] = await Database.execute(
      "INSERT INTO users (nama, email, password_hash) VALUES (?, ?, ?)",
      [nama, email, password_hash]
    );
    
    return { success: true, userId: (result as any).insertId };
  }, {
    body: t.Object({
      nama: t.String(),
      email: t.Email(),
      password: t.String().min(8)
    })
  })
  
  // Login
  .post("/api/login", async ({ body }) => {
    const { email, password } = body;
    
    // Get user
    const [rows] = await Database.execute(
      "SELECT id, email, password_hash, role FROM users WHERE email = ?",
      [email]
    );
    
    const users = rows as any[];
    const user = users[0];
    
    if (!user) {
      throw new Error("Email tidak ditemukan");
    }
    
    // Verify password
    const isValid = await verifyPassword(password, user.password_hash);
    
    if (!isValid) {
      throw new Error("Password salah");
    }
    
    // Generate token
    const accessToken = jwt.sign(
      { userId: user.id, email: user.email, role: user.role },
      process.env.JWT_SECRET!,
      { expiresIn: "1h" }
    );
    
    return {
      success: true,
      accessToken,
      user: { id: user.id, nama: user.nama, email: user.email }
    };
  }, {
    body: t.Object({
      email: t.Email(),
      password: t.String()
    })
  })
  
  // Protected route
  .get("/api/profile", async ({ headers }) => {
    const authHeader = headers.authorization;
    
    if (!authHeader?.startsWith("Bearer ")) {
      throw new Error("Token tidak ditemukan");
    }
    
    const token = authHeader.split(" ")[1];
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
    
    const [rows] = await Database.execute(
      "SELECT id, nama, email, role FROM users WHERE id = ?",
      [decoded.userId]
    );
    
    return (rows as any)[0];
  })
  
  // Middleware untuk auth
  .use(({ onBeforeHandle }) => {
    onBeforeHandle(({ request, response }) => {
      if (request.url.includes("/api/") && 
          !request.url.includes("/api/login") && 
          !request.url.includes("/api/register")) {
        // Check token for protected routes
      }
    });
  })
  
  .listen(3000);
```

---

## Implementasi Frontend

```svelte
<script>
  let loginForm = { email: "", password: "" };
  let errors = {};
  let loading = false;
  
  async function login() {
    loading = true;
    errors = {};
    
    try {
      const response = await fetch("/api/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(loginForm)
      });
      
      const result = await response.json();
      
      if (response.ok && result.accessToken) {
        // Store token
        localStorage.setItem("token", result.accessToken);
        
        // Set auth header for future requests
        api.defaults.headers.Authorization = `Bearer ${result.accessToken}`;
        
        // Redirect
        window.location.href = "/dashboard";
      } else {
        errors.email = result.message || "Login gagal";
      }
    } catch (error) {
      errors.email = "Network error";
    } finally {
      loading = false;
    }
  }
</script>

<form on:submit|preventDefault={login}>
  <input type="email" bind:value={loginForm.email} placeholder="Email" />
  <input type="password" bind:value={loginForm.password} placeholder="Password" />
  {#if errors.email}<p class="error">{errors.email}</p>{/if}
  <button type="submit" disabled={loading}>Login</button>
</form>
```

---

## Tugas Tambahan

1. Implementasikan refresh token
2. Tambahkan role-based access control
3. Buat middleware auth yang reusable

## Referensi

- [JWT.io](https://jwt.io/)
- [Elysia Auth](https://elysiajs.com/plugin/auth.html)