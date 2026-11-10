# Pertemuan 12: Authentication & JWT

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 11](../pertemuan-11/README.md) | [Selanjutnya: Pertemuan 13 ->](../pertemuan-13/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Menjelaskan perbedaan **authentication** dan **authorization**.
- Merancang alur register, login, logout, dan refresh token.
- Menyimpan password dengan hashing dan salt, bukan sebagai plaintext.
- Menjelaskan struktur JWT: `header`, `payload`, dan `signature`.
- Membuat, memverifikasi, dan membatalkan token dengan masa berlaku yang tepat.
- Mengirim token melalui `Authorization: Bearer <token>` dan melindungi endpoint menggunakan middleware.
- Menerapkan role-based authorization setelah identitas pengguna terverifikasi.
- Mengenali risiko keamanan seperti token yang bocor, algoritma yang salah, dan secret yang lemah.

> **Indikator ketercapaian:** register menyimpan password sebagai hash, login mengembalikan token, endpoint terlindungi menolak request tanpa token dengan status `401`, dan pengguna dengan token valid dapat mengakses resource miliknya.

## Prasyarat dan Instalasi

Materi ini menggunakan TypeScript, Bun, Elysia, MySQL, `bcrypt`, dan `jsonwebtoken`.

```bash
bun add elysia bcrypt jsonwebtoken
bun add -d @types/bcrypt @types/jsonwebtoken
```

Simpan secret di `.env`, lalu tambahkan `.env` ke `.gitignore`:

```env
JWT_ACCESS_SECRET=ganti-dengan-random-secret-minimal-32-karakter
JWT_REFRESH_SECRET=ganti-dengan-secret-berbeda-dari-access-token
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
```

Jangan menaruh secret sungguhan di source code, dokumentasi publik, atau repository.

---

## Materi

### 1. Authentication dan Authorization

**Authentication (AuthN)** menjawab pertanyaan *“Siapa pengguna ini?”*. Prosesnya biasanya berupa verifikasi email dan password, kode OTP, atau identitas dari provider OAuth.

**Authorization (AuthZ)** menjawab pertanyaan *“Apa yang boleh dilakukan pengguna ini?”*. Pemeriksaan ini dilakukan setelah identitas berhasil diverifikasi, misalnya:

- pengguna biasa hanya boleh melihat profilnya sendiri;
- admin boleh melihat daftar seluruh pengguna;
- pemilik resource boleh mengubah resource tersebut;
- pengguna dengan role `editor` boleh membuat artikel.

| Aspek | Authentication | Authorization |
|---|---|---|
| Tujuan | Memverifikasi identitas | Menentukan hak akses |
| Pertanyaan | “Siapa kamu?” | “Apa yang boleh kamu lakukan?” |
| Contoh | Login dengan email dan password | Hanya admin yang dapat menghapus user |
| Hasil | Identitas pengguna | Keputusan allow/deny |

JWT membantu membawa hasil authentication ke request berikutnya. JWT **bukan** pengganti authorization: server tetap harus memeriksa role, permission, dan kepemilikan resource.

### 2. Alur Authentication

Alur umum aplikasi:

```text
Register:
Client -> POST /api/register -> validasi -> hash password -> simpan user

Login:
Client -> POST /api/login -> cari user -> compare password
       <- access token + refresh token

Request protected:
Client -> Authorization: Bearer <access-token>
       -> verifikasi token -> cek permission -> response resource

Access token kedaluwarsa:
Client -> POST /api/refresh dengan refresh token
       <- access token baru
```

Password tidak pernah dikirim kembali ke client dan tidak pernah disimpan dalam bentuk plaintext.

### 3. Password Hashing dengan bcrypt

Hashing berbeda dari enkripsi. Hash bersifat satu arah: server tidak dapat mengubah hash kembali menjadi password. `bcrypt` menambahkan salt secara otomatis sehingga dua pengguna dengan password sama tetap memiliki hash yang berbeda.

```typescript
import { compare, hash } from "bcrypt";

const BCRYPT_ROUNDS = 12;

export async function hashPassword(password: string): Promise<string> {
  return hash(password, BCRYPT_ROUNDS);
}

export async function verifyPassword(
  password: string,
  passwordHash: string
): Promise<boolean> {
  return compare(password, passwordHash);
}
```

Praktik penting:

1. Validasi panjang dan format password sebelum hashing.
2. Gunakan `compare`, jangan membandingkan string hash secara manual.
3. Jangan mencatat password ke log.
4. Gunakan pesan login yang umum, misalnya `Email atau password tidak valid`, agar keberadaan email tidak mudah ditebak.
5. Untuk sistem baru, gunakan algoritma password hashing modern yang direkomendasikan organisasi; `bcrypt` digunakan di materi ini untuk mempelajari konsep.

### 4. Apa Itu JWT?

**JSON Web Token (JWT)** adalah token ringkas yang dapat diverifikasi secara kriptografis. JWT yang ditandatangani tidak otomatis terenkripsi. Karena itu, payload jangan diisi password, nomor kartu, atau data rahasia.

JWT terdiri dari tiga bagian yang dipisahkan oleh titik:

```text
base64url(header).base64url(payload).base64url(signature)
```

Contoh struktur:

```json
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}
```

```json
// Payload
{
  "sub": "42",
  "role": "user",
  "iat": 1710000000,
  "exp": 1710000900,
  "jti": "unique-token-id"
}
```

- **Header** menyatakan tipe token dan algoritma penandatanganan.
- **Payload** berisi claims. `sub` adalah subject atau identitas utama, `iat` adalah waktu token dibuat, dan `exp` adalah waktu kedaluwarsa.
- **Signature** memastikan token tidak diubah setelah diterbitkan.

Siapa pun yang memperoleh JWT dapat membaca header dan payload. Yang tidak dapat dilakukan tanpa secret adalah membuat signature valid untuk payload yang diubah.

### 5. Claims dan Masa Berlaku

Gunakan claims seperlunya:

| Claim | Arti | Catatan |
|---|---|---|
| `sub` | ID pengguna atau subject | Gunakan sebagai sumber identitas utama |
| `iss` | Issuer penerbit token | Berguna saat banyak penerbit |
| `aud` | Audiens token | Batasi token untuk aplikasi tertentu |
| `iat` | Waktu token dibuat | Umumnya dibuat library |
| `exp` | Waktu token kedaluwarsa | Wajib untuk membatasi umur token |
| `jti` | ID unik token | Berguna untuk revoke atau audit |
| `role` | Role pengguna | Jangan dijadikan satu-satunya sumber keputusan untuk data yang sangat sensitif |

Jangan memasukkan data yang mudah berubah ke access token jika server harus selalu membaca nilai terbaru. Untuk perubahan role atau pencabutan akses, gunakan token berumur pendek, cek database, atau mekanisme revocation.

### 6. Access Token dan Refresh Token

**Access token** digunakan pada request API dan berumur pendek, misalnya 15 menit. **Refresh token** digunakan hanya untuk meminta access token baru dan berumur lebih panjang, misalnya 7 hari.

```typescript
import jwt from "jsonwebtoken";

type TokenPair = {
  accessToken: string;
  refreshToken: string;
};

const accessSecret = process.env.JWT_ACCESS_SECRET;
const refreshSecret = process.env.JWT_REFRESH_SECRET;

if (!accessSecret || !refreshSecret) {
  throw new Error("JWT secrets wajib dikonfigurasi");
}

export function createTokenPair(userId: number, role: string): TokenPair {
  const accessToken = jwt.sign(
    { role },
    accessSecret,
    {
      subject: String(userId),
      issuer: "advanced-web-programming",
      audience: "web-client",
      expiresIn: "15m"
    }
  );

  const refreshToken = jwt.sign(
    {},
    refreshSecret,
    {
      subject: String(userId),
      issuer: "advanced-web-programming",
      audience: "web-client",
      expiresIn: "7d"
    }
  );

  return { accessToken, refreshToken };
}
```

Alur refresh token yang aman:

1. Client mengirim refresh token ke `POST /api/refresh`.
2. Server memverifikasi signature, issuer, audience, dan expiry.
3. Server memeriksa bahwa refresh token masih aktif atau belum dicabut.
4. Server menerbitkan access token baru.
5. Jika menggunakan rotation, server juga menerbitkan refresh token baru dan mencabut token lama.

Jangan menerima refresh token di endpoint biasa. Simpan refresh token pada cookie `HttpOnly`, `Secure`, dan `SameSite` yang sesuai, atau simpan hash-nya di database sehingga token asli tidak perlu disimpan server.

### 7. Membaca dan Memverifikasi Token

`decode` hanya membaca isi token dan **bukan** verifikasi. Request terlindungi harus selalu menggunakan `verify`.

```typescript
import jwt, { JwtPayload } from "jsonwebtoken";

type AccessClaims = JwtPayload & {
  role?: string;
};

export function verifyAccessToken(token: string): AccessClaims {
  const payload = jwt.verify(token, accessSecret, {
    issuer: "advanced-web-programming",
    audience: "web-client",
    algorithms: ["HS256"]
  });

  if (typeof payload === "string" || !payload.sub) {
    throw new Error("Payload token tidak valid");
  }

  return payload as AccessClaims;
}
```

Batasi algoritma yang diterima. Jangan mempercayai nilai `alg` dari token tanpa konfigurasi algoritma yang diizinkan.

### 8. Mengambil Bearer Token

Format header yang benar:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

Parser harus menolak header yang hilang atau formatnya salah:

```typescript
export function getBearerToken(authorization?: string): string {
  if (!authorization) {
    throw new Error("Authorization header tidak ditemukan");
  }

  const [scheme, token, ...extra] = authorization.trim().split(/\s+/);

  if (scheme !== "Bearer" || !token || extra.length > 0) {
    throw new Error("Format Authorization harus Bearer <token>");
  }

  return token;
}
```

Status yang umum digunakan:

- `401 Unauthorized`: token tidak ada, malformed, invalid, atau expired.
- `403 Forbidden`: token valid, tetapi role atau permission tidak mencukupi.

### 9. Skema Database

Contoh tabel minimal:

```sql
CREATE TABLE users (
  id INT PRIMARY KEY AUTO_INCREMENT,
  nama VARCHAR(100) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  role ENUM('user', 'admin') NOT NULL DEFAULT 'user',
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE refresh_tokens (
  id INT PRIMARY KEY AUTO_INCREMENT,
  user_id INT NOT NULL,
  token_hash VARCHAR(255) NOT NULL,
  expires_at DATETIME NOT NULL,
  revoked_at DATETIME NULL,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Query harus menggunakan parameter, bukan interpolasi string:

```typescript
const [rows] = await Database.execute(
  "SELECT id, nama, email, password_hash, role FROM users WHERE email = ?",
  [email]
);
```

### 10. Implementasi Backend dengan Elysia

Contoh berikut menunjukkan bagian inti endpoint. Validasi dan error handler produksi tetap perlu disesuaikan dengan arsitektur aplikasi.

```typescript
import { Elysia, t } from "elysia";
import jwt from "jsonwebtoken";
import Database from "../config/database";
import { hashPassword, verifyPassword } from "../utils/auth";

const app = new Elysia()
  .post("/api/register", async ({ body, set }) => {
    const [existingRows] = await Database.execute(
      "SELECT id FROM users WHERE email = ?",
      [body.email]
    );

    if ((existingRows as unknown[]).length > 0) {
      set.status = 409;
      return { message: "Email sudah terdaftar" };
    }

    const passwordHash = await hashPassword(body.password);
    const [result] = await Database.execute(
      "INSERT INTO users (nama, email, password_hash) VALUES (?, ?, ?)",
      [body.nama, body.email, passwordHash]
    );

    set.status = 201;
    return { success: true, userId: (result as { insertId: number }).insertId };
  }, {
    body: t.Object({
      nama: t.String({ minLength: 2, maxLength: 100 }),
      email: t.String({ format: "email" }),
      password: t.String({ minLength: 8, maxLength: 128 })
    })
  })
  .post("/api/login", async ({ body, set }) => {
    const [rows] = await Database.execute(
      "SELECT id, nama, email, password_hash, role FROM users WHERE email = ?",
      [body.email]
    );
    const user = (rows as Array<{
      id: number;
      nama: string;
      email: string;
      password_hash: string;
      role: string;
    }>)[0];

    if (!user || !(await verifyPassword(body.password, user.password_hash))) {
      set.status = 401;
      return { message: "Email atau password tidak valid" };
    }

    const accessToken = jwt.sign(
      { role: user.role },
      process.env.JWT_ACCESS_SECRET!,
      {
        subject: String(user.id),
        issuer: "advanced-web-programming",
        audience: "web-client",
        expiresIn: "15m"
      }
    );

    return {
      accessToken,
      user: { id: user.id, nama: user.nama, email: user.email, role: user.role }
    };
  }, {
    body: t.Object({
      email: t.String({ format: "email" }),
      password: t.String({ minLength: 1 })
    })
  })
  .get("/api/profile", ({ headers, set }) => {
    try {
      const token = getBearerToken(headers.authorization);
      const claims = verifyAccessToken(token);
      return { userId: Number(claims.sub) };
    } catch {
      set.status = 401;
      return { message: "Token tidak valid atau sudah kedaluwarsa" };
    }
  });
```

Pada aplikasi nyata, fungsi `getBearerToken` dan `verifyAccessToken` sebaiknya berada di `middleware/auth.ts`, bukan disalin pada setiap route. Middleware harus menaruh identitas yang sudah diverifikasi ke context request; handler kemudian hanya menggunakan identitas tersebut.

### 11. Authorization Berbasis Role

Authentication middleware memastikan token valid. Middleware authorization memeriksa izin:

```typescript
type AuthUser = { id: number; role: "user" | "admin" };

function requireRole(...allowedRoles: AuthUser["role"][]) {
  return ({ user, set }: { user?: AuthUser; set: { status?: number } }) => {
    if (!user) {
      set.status = 401;
      return { message: "Authentication diperlukan" };
    }

    if (!allowedRoles.includes(user.role)) {
      set.status = 403;
      return { message: "Anda tidak memiliki izin" };
    }
  };
}
```

Jangan hanya menyembunyikan tombol di frontend. Pemeriksaan role wajib dilakukan kembali di backend karena client dapat dimodifikasi oleh pengguna.

### 12. Refresh, Logout, dan Revoke

JWT access token bersifat stateless sehingga logout tidak otomatis menghapus token yang sudah terbit. Strategi yang dapat digunakan:

1. Hapus access dan refresh token dari client.
2. Cabut refresh token di database dengan mengisi `revoked_at`.
3. Gunakan refresh-token rotation untuk mendeteksi reuse.
4. Untuk kebutuhan pencabutan access token segera, simpan `jti` yang dicabut atau gunakan session/token introspection.

Contoh kontrak endpoint:

| Endpoint | Method | Auth | Fungsi |
|---|---:|---|---|
| `/api/register` | POST | Tidak | Membuat akun baru |
| `/api/login` | POST | Tidak | Memverifikasi credential dan menerbitkan token |
| `/api/refresh` | POST | Refresh token | Menerbitkan access token baru |
| `/api/logout` | POST | Refresh token | Mencabut refresh token |
| `/api/profile` | GET | Access token | Mengambil profil pengguna |
| `/api/users` | GET | Access token + admin | Mengambil daftar user |

### 13. Integrasi Frontend

```typescript
type LoginResponse = {
  accessToken: string;
  user: { id: number; nama: string; email: string; role: string };
};

let accessToken = "";

async function login(email: string, password: string) {
  const response = await fetch("/api/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password })
  });

  if (!response.ok) {
    throw new Error("Email atau password tidak valid");
  }

  const result = await response.json() as LoginResponse;
  accessToken = result.accessToken;
  return result.user;
}

async function getProfile() {
  const response = await fetch("/api/profile", {
    headers: { Authorization: `Bearer ${accessToken}` }
  });

  if (response.status === 401) {
    // Panggil refresh token atau arahkan pengguna ke halaman login.
    throw new Error("Sesi telah berakhir");
  }

  return response.json();
}
```

Untuk aplikasi browser, hindari `localStorage` jika dampak XSS belum ditangani dengan baik karena JavaScript dapat membaca token di sana. Pola yang umum adalah access token berumur pendek di memory dan refresh token pada cookie `HttpOnly`, disertai perlindungan CSRF serta konfigurasi `Secure` dan `SameSite`.

### 14. Checklist Keamanan

- [ ] Secret dibuat acak, panjang, berbeda untuk access dan refresh token.
- [ ] Secret hanya disimpan di environment variable atau secret manager.
- [ ] Password di-hash dengan salt dan tidak pernah dicatat ke log.
- [ ] Access token memiliki expiry yang pendek.
- [ ] Refresh token diverifikasi, disimpan dengan aman, dan dapat dicabut.
- [ ] Algoritma, issuer, audience, dan expiry token diverifikasi.
- [ ] Payload tidak berisi data rahasia.
- [ ] Semua input divalidasi; query database menggunakan parameter.
- [ ] Endpoint protected mengembalikan `401`, sedangkan izin kurang mengembalikan `403`.
- [ ] Cookie authentication menggunakan `HttpOnly`, `Secure`, `SameSite`, dan perlindungan CSRF yang tepat.
- [ ] CORS hanya mengizinkan origin yang diperlukan.
- [ ] Rate limiting dan audit log diterapkan pada endpoint login.
- [ ] Pesan error tidak membocorkan apakah email tertentu terdaftar.

---

## Pengujian dengan HTTP Client

Register:

```bash
curl -X POST http://localhost:3000/api/register `
  -H "Content-Type: application/json" `
  -d '{ "nama": "Budi", "email": "budi@example.com", "password": "rahasia123" }'
```

Login:

```bash
curl -X POST http://localhost:3000/api/login `
  -H "Content-Type: application/json" `
  -d '{ "email": "budi@example.com", "password": "rahasia123" }'
```

Akses endpoint protected dengan token dari response login:

```bash
curl http://localhost:3000/api/profile `
  -H "Authorization: Bearer <access-token>"
```

Skenario minimum yang harus diuji:

1. Register dengan email baru berhasil.
2. Register dengan email duplikat ditolak.
3. Password salah menghasilkan `401`.
4. Request tanpa `Authorization` menghasilkan `401`.
5. Token malformed menghasilkan `401`.
6. Token expired menghasilkan `401`.
7. User biasa ke endpoint admin menghasilkan `403`.
8. Token valid hanya dapat mengakses data yang diizinkan.

---

## Tugas

1. Buat tabel `users` dan endpoint register/login menggunakan password hashing.
2. Buat middleware reusable untuk membaca dan memverifikasi access token.
3. Tambahkan endpoint `/api/profile` yang mengambil data berdasarkan `sub`, bukan berdasarkan ID dari query client.
4. Implementasikan refresh-token rotation dan endpoint logout yang mencabut refresh token.
5. Tambahkan role `admin` dan lindungi endpoint `GET /api/users`.
6. Tulis minimal delapan test dari skenario pengujian di atas.
7. Jelaskan dalam laporan mengapa JWT yang ditandatangani tidak sama dengan JWT yang terenkripsi.

## Referensi

- [JWT.io](https://jwt.io/)
- [RFC 7519 - JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP JSON Web Token Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
- [Elysia Documentation](https://elysiajs.com/)
