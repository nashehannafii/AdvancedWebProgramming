# Pertemuan 13: OAuth 2.0 & Authorization

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 12](../pertemuan-12/README.md) | [Selanjutnya: Pertemuan 14 ->](../pertemuan-14/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Menjelaskan peran OAuth 2.0 (Client, Resource Owner, Authorization Server, Resource Server), scope, dan Authorization Code Flow
- Membedakan OAuth 2.0 untuk authorization vs authentication biasa
- Mengimplementasikan login via identity provider (Google/GitHub): redirect -> code -> exchange token -> userinfo
- Memetakan user OAuth ke user lokal (MySQL) dan menerbitkan JWT internal setelah verifikasi
- Menangani redirect URI, error OAuth, dan mengamankan `client_secret` di environment variable

> **Indikator ketercapaian:** Tombol "Login dengan Google" berhasil, callback menukar code menjadi token, dan user baru/lama dapat login tanpa password.

---
## Materi

### 1. OAuth 2.0 vs Authentication

| Aspek | OAuth 2.0 | Authentication |
|-------|-----------|--------------|
| Tujuan | Memberi akses ke resource orang lain | Memverifikasi identitas |
| Scope | Memberi akses API | Login verification |

### 2. OAuth Roles

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Client    │────▶│ Auth Server │────▶│  Resource   │
│ (Frontend)  │     │  (Google,   │     │   Server    │
│             │◀────│   Github)   │◀────│ (Elysia)    │
└─────────────┘     └─────────────┘     └─────────────┘
```

- **Client**: Aplikasi yang meminta akses
- **Resource Owner**: Pengguna yang memberi izin
- **Authorization Server**: Server yang memberi token
- **Resource Server**: Server yang menyimpan data

### 3. Authorization Code Flow

```
1. Client -> Redirect ke Authorization Server
2. User -> Login & Authorize
3. Auth Server -> Redirect ke Redirect URI dengan Authorization Code
4. Client -> Request Access Token (dengan Authorization Code)
5. Auth Server -> Response Access Token
6. Client -> Request Resource dengan Access Token
```

### 4. Scope

Scope menentukan hak akses yang diberikan:

- `profile` - Informasi profil dasar
- `email` - Alamat email
- `openid` - OpenID Connect
- `offline_access` - Refresh token

---

## Contoh Flow

```
Svelte Frontend
    |
    v
OAuth Provider (Google)
    |
    v
Authorization Code
    |
    v
Elysia Backend
    |
    v
MySQL (User mapping)
```

---

## Implementasi

### 1. Setup Backend (Elysia)

```typescript
import { Elysia } from "elysia";
import axios from "axios";

const app = new Elysia()
  // Google OAuth callback
  .get("/auth/google/callback", async ({ query }) => {
    const { code } = query;
    
    // Exchange code untuk access token
    const tokenResponse = await axios.post(
      "https://oauth2.googleapis.com/token",
      new URLSearchParams({
        client_id: process.env.GOOGLE_CLIENT_ID,
        client_secret: process.env.GOOGLE_CLIENT_SECRET,
        code: code as string,
        grant_type: "authorization_code",
        redirect_uri: `${process.env.BASE_URL}/auth/google/callback`
      })
    );
    
    const { access_token, id_token } = tokenResponse.data;
    
    // Get user info
    const userInfoResponse = await axios.get(
      "https://www.googleapis.com/oauth2/v3/userinfo",
      { headers: { Authorization: `Bearer ${access_token}` } }
    );
    
    const { email, name, picture } = userInfoResponse.data;
    
    // Create atau get user di database
    // ... simpan/update user
    
    // Generate JWT token
    const jwt = require("jsonwebtoken");
    const accessToken = jwt.sign(
      { email, name, provider: "google" },
      process.env.JWT_SECRET!,
      { expiresIn: "1h" }
    );
    
    // Redirect ke frontend dengan token
    return new Response(null, {
      status: 302,
      headers: {
        Location: `${process.env.FRONTEND_URL}/auth/callback?token=${accessToken}`
      }
    });
  })
  
  .listen(3000);
```

### 2. Frontend Integration (Svelte)

```svelte
<script>
  const GOOGLE_CLIENT_ID = import.meta.env.VITE_GOOGLE_CLIENT_ID;
  const REDIRECT_URI = `${window.location.origin}/auth/callback`;
  
  function initiateGoogleLogin() {
    const authUrl = `https://accounts.google.com/o/oauth2/v2/auth?${new URLSearchParams({
      client_id: GOOGLE_CLIENT_ID,
      redirect_uri: REDIRECT_URI,
      response_type: "code",
      scope: "openid email profile",
      access_type: "offline",
      prompt: "consent"
    })}`;
    
    window.location.href = authUrl;
  }
  
  // Handle callback
  async function handleCallback() {
    const urlParams = new URLSearchParams(window.location.search);
    const code = urlParams.get("code");
    
    if (code) {
      const response = await fetch("/auth/google/callback", {
        method: "GET"
      });
      
      // Handle redirect or error
    }
  }
  
  initialize();
</script>

<button on:click={initiateGoogleLogin}>
  Login dengan Google
</button>
```

### 3. Provider Konfigurasi

```typescript
const providers = {
  google: {
    clientId: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    authUrl: "https://accounts.google.com/o/oauth2/v2/auth",
    tokenUrl: "https://oauth2.googleapis.com/token"
  },
  github: {
    clientId: process.env.GITHUB_CLIENT_ID,
    clientSecret: process.env.GITHUB_CLIENT_SECRET,
    authUrl: "https://github.com/login/oauth/authorize",
    tokenUrl: "https://github.com/login/oauth/access_token"
  }
};
```

### 4. Error Handling

```typescript
try {
  // OAuth flow
} catch (error) {
  if (error.response?.status === 401) {
    // Invalid credentials
  } else if (error.response?.status === 403) {
    // Quota exceeded
  } else {
    // Network error
  }
}
```

---

## Tugas Tambahan

1. Implementasikan login dengan GitHub
2. Tambahkan logout functionality
3. Handle token refresh otomatis

## Referensi

- [Google OAuth 2.0](https://developers.google.com/identity/protocols/oauth2)
- [RFC 6749 - OAuth 2.0](https://datatracker.ietf.org/doc/html/rfc6749)