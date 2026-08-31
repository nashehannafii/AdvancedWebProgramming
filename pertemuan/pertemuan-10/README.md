# Pertemuan 10: Integrasi Svelte dengan REST API

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 09](../pertemuan-09/README.md) | [Selanjutnya: Pertemuan 11 ->](../pertemuan-11/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Menggunakan Fetch API untuk GET/POST/PUT/DELETE dari frontend Svelte dengan `async`/`await`
- Mendesain API client terpusat (`ApiClient`) dan modul endpoint (`mahasiswaAPI`) yang reusable
- Mengelola loading state, error state, dan menampilkan response API di UI dengan benar
- Menerapkan store Svelte (`writable`, `derived`) untuk state global (list, pagination, total)
- Mengintegrasikan Svelte <-> Elysia <-> MySQL secara end-to-end

> **Indikator ketercapaian:** UI dapat list/create/update/delete mahasiswa yang tersinkron dengan backend; loading & error ditangani tanpa crash.

---
## Materi

### 1. Fetch API

Fetch API adalah browser built-in untuk membuat HTTP requests.

```typescript
const response = await fetch(url, {
  method: "GET",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer token"
  },
  body: JSON.stringify(data)
});

const data = await response.json();
```

### 2. HTTP Methods dari Frontend

#### GET - Mengambil Data

```typescript
async function getUsers() {
  const response = await fetch("/api/users");
  return response.json();
}
```

#### POST - Membuat Data Baru

```typescript
async function createUser(userData: any) {
  const response = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(userData)
  });
  return response.json();
}
```

#### PUT - Replace Data

```typescript
async function updateUser(id: number, data: any) {
  const response = await fetch(`/api/users/${id}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data)
  });
  return response.json();
}
```

#### DELETE - Hapus Data

```typescript
async function deleteUser(id: number) {
  const response = await fetch(`/api/users/${id}`, {
    method: "DELETE"
  });
  return response.json();
}
```

### 3. Async/Await

```typescript
async function fetchData() {
  try {
    const response = await fetch("/api/data");
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error("Error fetching data:", error);
    throw error;
  }
}
```

### 4. Loading State

```svelte
<script>
  let data = null;
  let loading = false;
  let error = null;
  
  async function loadData() {
    loading = true;
    error = null;
    
    try {
      const response = await fetch("/api/data");
      data = await response.json();
    } catch (e) {
      error = e.message;
    } finally {
      loading = false;
    }
  }
</script>

{#if loading}
  <p>Loading...</p>
{:else if error}
  <p>Error: {error}</p>
{:else if data}
  <p>Data: {data}</p>
{/if}

<button on:click={loadData} disabled={loading}>
  {loading ? "Memuat..." : "Load Data"}
</button>
```

### 5. Error State

```typescript
interface ApiResponse<T> {
  data: T;
  success: boolean;
  error?: string;
}

async function safeFetch<T>(url: string): Promise<ApiResponse<T>> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      return {
        data: null as any,
        success: false,
        error: `HTTP ${response.status}: ${response.statusText}`
      };
    }
    return {
      data: await response.json(),
      success: true
    };
  } catch (error) {
    return {
      data: null as any,
      success: false,
      error: (error as Error).message
    };
  }
}
```

### 6. API Response Handling

```typescript
// Response wrapper standard
interface APIResponse<T> {
  success: boolean;
  data?: T;
  meta?: any;
  message?: string;
  errors?: string[];
}

// Usage
async function fetchUsers() {
  const response = await fetch("/api/users");
  const result: APIResponse<User[]> = await response.json();
  
  if (!result.success) {
    throw new Error(result.message || "Fetch failed");
  }
  
  return result.data || [];
}
```

## Praktikum

**Tujuan**: Menghubungkan Svelte dengan REST API Elysia.

### Arsitektur

```
Svelte Frontend
    |
    | HTTP
    v
Elysia Backend
    |
    v
MySQL
```

### Struktur Project

```
src/
├── lib/
│   ├── api/
│   │   ├── client.ts
│   │   └── mahasiswa.ts
│   ├── stores/
│   │   └── mahasiswa-store.ts
│   └── components/
│       ├── MahasiswaForm.svelte
│       └── MahasiswaList.svelte
└── routes/
    └── +page.svelte
```

### Implementasi

**1. API Client (`src/lib/api/client.ts`)**

```typescript
const API_BASE_URL = import.meta.env.VITE_API_URL || "http://localhost:3000";

class ApiClient {
  private baseUrl: string = API_BASE_URL;

  async request<T>(endpoint: string, options: RequestInit = {}): Promise<T> {
    const url = `${this.baseUrl}${endpoint}`;
    
    const response = await fetch(url, {
      headers: {
        "Content-Type": "application/json",
        ...options.headers
      },
      ...options
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return response.json();
  }

  get<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: "GET" });
  }

  post<T>(endpoint: string, body: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: "POST",
      body: JSON.stringify(body)
    });
  }

  put<T>(endpoint: string, body: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: "PUT",
      body: JSON.stringify(body)
    });
  }

  delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: "DELETE" });
  }
}

export default new ApiClient();
```

**2. API Module (`src/lib/api/mahasiswa.ts`)**

```typescript
import client from "./client";

export interface Mahasiswa {
  id: number;
  nama: string;
  nim: string;
  email: string;
  nilai?: number;
}

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

export class MahasiswaAPI {
  getAll(params?: { page?: number; limit?: number; search?: string }) {
    const queryString = new URLSearchParams(
      Object.entries(params || {}).map(([k, v]) => [k, String(v)])
    ).toString();
    
    return client.get<PaginatedResponse<Mahasiswa>>(
      `/api/mahasiswa${queryString ? `?${queryString}` : ""}`
    );
  }

  get(id: number) {
    return client.get<Mahasiswa>(`/api/mahasiswa/${id}`);
  }

  create(data: Omit<Mahasiswa, "id">) {
    return client.post<Mahasiswa>("/api/mahasiswa", data);
  }

  update(id: number, data: Partial<Mahasiswa>) {
    return client.put<Mahasiswa>(`/api/mahasiswa/${id}`, data);
  }

  delete(id: number) {
    return client.delete<{ message: string }>(`/api/mahasiswa/${id}`);
  }
}

export const mahasiswaAPI = new MahasiswaAPI();
```

**3. Store (`src/lib/stores/mahasiswa-store.ts`)**

```typescript
import { writable, derived } from "svelte/store";
import { mahasiswaAPI, type Mahasiswa } from "../api/mahasiswa";

export interface MahasiswaState {
  mahasiswas: Mahasiswa[];
  loading: boolean;
  error: string | null;
  page: number;
  limit: number;
  total: number;
}

const initialState: MahasiswaState = {
  mahasiswas: [],
  loading: false,
  error: null,
  page: 1,
  limit: 10,
  total: 0
};

export const mahasiswaStore = writable<MahasiswaState>(initialState);

export function fetchMahasiswa(params?: { page?: number; limit?: number; search?: string }) {
  mahasiswaStore.update(state => ({
    ...state,
    loading: true,
    error: null
  }));

  mahasiswaAPI.getAll(params)
    .then(response => {
      mahasiswaStore.set({
        ...initialState,
        mahasiswas: response.data,
        page: response.meta.page,
        limit: response.meta.limit,
        total: response.meta.total,
        loading: false
      });
    })
    .catch(error => {
      mahasiswaStore.update(state => ({
        ...state,
        error: error.message,
        loading: false
      }));
    });
}

export const totalPages = derived(
  mahasiswaStore,
  $state => Math.ceil($state.total / $state.limit)
);
```

## Run Demo

```bash
# Terminal 1 - Backend
bun run ../pertemuan-05/src/server.ts

# Terminal 2 - Frontend
npm run dev
```

## Referensi

- [Fetch API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)