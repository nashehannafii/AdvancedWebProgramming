# Pertemuan 09: Svelte dan TypeScript

## Navigasi

[🏠 Beranda](../../README.md) | [🔧 Prerequest](../../prerequest/README.md) | ... | [Pertemuan 08](../pertemuan-08/README.md) | **Pertemuan 09** | [Pertemuan 10](../pertemuan-10/README.md) | ... | [Pertemuan 16](../pertemuan-16/README.md)

---

## Materi

### 1. Pengenalan Svelte

Svelte adalah framework frontend yang tidak menggunakan virtual DOM.

#### Fitur Utama

- **Reaktif**: State berubah, UI ikut berubah otomatis
- **Compile-time**: Kompilasi ke vanilla JS
- **Bundle size kecil**: Tanpa runtime overhead
- **Simple syntax**: Markup yang mudah dibaca

### 2. Svelte Project Structure

```bash
npm create svelte@latest my-app
cd my-app
npm install
```

Struktur dasar:

```
src/
├── routes/
│   ├── +layout.svelte    # Layout komponen
│   └── +page.svelte      # Page utama
├── lib/                  # Utility komponen
└── app.d.ts             # Type declarations
```

### 3. Component

Komponen utama di Svelte adalah file `.svelte`.

```svelte
<!-- components/UserCard.svelte -->
<script lang="ts">
  export let nama: string;
  export let email: string;
  export let usia: number = 0;
</script>

<div class="card">
  <h2>{nama}</h2>
  <p>{email}</p>
  <p>Usia: {usia}</p>
</div>

<style>
  .card {
    border: 1px solid #ccc;
    padding: 1rem;
    border-radius: 8px;
  }
</style>
```

### 4. Props

Props dikirim dari komponen induk ke anak:

```svelte
<!-- Parent.svelte -->
<script>
  import UserCard from "./components/UserCard.svelte";
</script>

<UserCard 
  nama="Budi" 
  email="budi@example.com" 
  usia={25} 
/>
```

### 5. State Management

```svelte
<script lang="ts">
  let count = 0;
  let pesan = "Hello World";
  
  function increment() {
    count += 1;
  }
  
  // Reactive statement
  $: totalKata = pesan.split(" ").length;
</script>

<button on:click={increment}>
  Count: {count}
</button>
<p>Kata terakhir: {pesan} - Jumlah kata: {totalKata}</p>
```

### 6. Reactive Programming

```typescript
// $: adalah reactive statement
let a = 1;
let b = 2;

$: c = a + b;  // otomatis update when a atau b berubah

// Reactive declaration fungsional
$: doubled = [1, 2, 3].map(x => x * 2);
```

### 7. Event Handling

```svelte
<script>
  let clicks = 0;
  
  function handleClick(event: MouseEvent) {
    clicks += 1;
    console.log("Clicked:", event);
  }
  
  function handleKeydown(event: KeyboardEvent) {
    if (event.key === "Escape") {
      // do something
    }
  }
</script>

<button on:click={handleClick}>
  Click me ({clicks})
</button>

<input on:keydown={handleKeydown} placeholder="Type ESC to test" />
```

### 8. Conditional Rendering

```svelte
<script>
  let isVisible = true;
  let status = "loading";
</script>

{#if isVisible}
  <p>Element ini terlihat</p>
{/if}

{#if status === "loading"}
  <p>Loading...</p>
{:else if status === "success"}
  <p>Berhasil!</p>
{:else}
  <p>Error</p>
{/if}
```

### 9. List Rendering

```svelte
<script lang="ts">
  interface Item {
    id: number;
    nama: string;
  }
  
  let items: Item[] = [
    { id: 1, nama: "Item 1" },
    { id: 2, nama: "Item 2" },
    { id: 3, nama: "Item 3" }
  ];
</script>

<ul>
  {#each items as item (item.id)}
    <li>{item.nama}</li>
  {/each}
</ul>
```

### 10. TypeScript di Svelte

**Konfigurasi (`tsconfig.json`)**

```json
{
  "extends": "svelte-ts-preprocess",
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true
  }
}
```

**Type Declarations (`app.d.ts`)**

```typescript
/// <reference types="svelte" />

declare namespace App {
  interface Locals {}
  interface Session {}
  interface Context {}
  interface Callbacks {}
}
```

## Praktikum

**Tujuan**: Membuat frontend sederhana menggunakan Svelte + TypeScript.

### Struktur Project

```
src/
├── routes/
│   ├── +layout.svelte
│   └── +page.svelte
├── lib/
│   └── components/
│       └── MahasiswaList.svelte
├── app.d.ts
└── svelte.config.js
```

### Implementasi

**1. Layout (`src/routes/+layout.svelte`)**

```svelte
<script lang="ts">
  import "../app.css";
</script>

<slot />
```

**2. Page Utama (`src/routes/+page.svelte`)**

```svelte
<script lang="ts">
  import MahasiswaList from "../lib/components/MahasiswaList.svelte";
  
  let title = "Daftar Mahasiswa";
</script>

<header>
  <h1>{title}</h1>
</header>

<main>
  <MahasiswaList />
</main>

<style>
  header {
    text-align: center;
    margin-bottom: 2rem;
  }
</style>
```

**3. Komponen (`src/lib/components/MahasiswaList.svelte`)**

```svelte
<script lang="ts">
  interface Mahasiswa {
    id: number;
    nama: string;
    nim: string;
    email: string;
    nilai?: number;
  }
  
  let mahasiswas: Mahasiswa[] = [];
  let loading = false;
  let error: string | null = null;
  
  async function fetchData() {
    loading = true;
    error = null;
    
    try {
      const response = await fetch("/api/mahasiswa");
      if (!response.ok) throw new Error("Gagal fetch data");
      const data = await response.json();
      mahasiswas = data.data;
    } catch (e) {
      error = (e as Error).message;
    } finally {
      loading = false;
    }
  }
  
  // Fetch on mount
  fetchData();
</script>

{#if loading}
  <p>Memuat data...</p>
{:else if error}
  <p>Error: {error}</p>
{:else}
  <table>
    <thead>
      <tr>
        <th>ID</th>
        <th>Nama</th>
        <th>NIM</th>
        <th>Email</th>
        <th>Nilai</th>
      </tr>
    </thead>
    <tbody>
      {#each mahasiswas as mhs}
        <tr>
          <td>{mhs.id}</td>
          <td>{mhs.nama}</td>
          <td>{mhs.nim}</td>
          <td>{mhs.email}</td>
          <td>{mhs.nilai ?? "-"}</td>
        </tr>
      {/each}
    </tbody>
  </table>
{/if}
```

## Run Project

```bash
npm run dev
# Buka http://localhost:5173
```

## Referensi

- [Svelte Tutorial](https://svelte.dev/tutorial)