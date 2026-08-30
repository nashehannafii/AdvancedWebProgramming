# Pertemuan 03: Synchronous dan Asynchronous Programming

## Materi

- Synchronous programming
- Asynchronous programming
- Callback
- Promise
- async
- await
- Event loop
- Error handling asynchronous

## Praktikum

Membuat program asynchronous dan simulasi pengambilan data dari server.

## Tujuan

Mahasiswa mampu:
1. Memahami perbedaan sync dan async
2. Menggunakan Promise, async/await
3. Mengatasi error pada kode async

## Struktur Project

```
pertemuan-03/
├── src/
│   ├── sync-demo.ts
│   ├── async-demo.ts
│   └── server-simulation.ts
└── tsconfig.json
```

## Contoh Kode

```typescript
// Promise
async function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("Data diterima"), 2000);
  });
}

// await
const data = await fetchData();
```