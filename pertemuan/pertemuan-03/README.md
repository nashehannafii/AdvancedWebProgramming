# Pertemuan 03: Synchronous dan Asynchronous Programming

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 02](../pertemuan-02/README.md) | [Selanjutnya: Pertemuan 04 ->](../pertemuan-04/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Menjelaskan perbedaan synchronous vs asynchronous dan peran event loop
- Menggunakan callback, Promise (pending/fulfilled/rejected), `async`/`await` dengan benar
- Melakukan chaining Promise dan error handling (`try/catch`, `.catch()`, `Promise.all`)
- Mensimulasikan pengambilan data dari server dengan delay dan menangani kasus gagal/timeout
- Mencegah callback hell dan memilih pola async yang tepat untuk kasus sekuensial vs paralel

> **Indikator ketercapaian:** Mampu menjalankan simulasi fetch sekuensial dan paralel, menangani error tanpa crash, dan menjelaskan urutan eksekusi event loop.

---
## Materi

### 1. Synchronous Programming

Kode yang dieksekusi secara berurutan, satu per satu. Setiap operasi harus selesai sebelum operasi berikutnya dimulai.

```typescript
function synchronousExample(): void {
  console.log("1. Mulai");
  
  for (let i = 0; i < 3; i++) {
    console.log(`2. Iterasi ${i + 1}`);
  }
  
  console.log("3. Selesai");
}

synchronousExample();
// Output: 1. Mulai -> 2. Iterasi 1 -> 2. Iterasi 2 -> 2. Iterasi 3 -> 3. Selesai
```

### 2. Asynchronous Programming

Kode yang dieksekusi secara tidak berurutan. Hasil akan dikembalikan nanti (_CALLBACK, PROMISE, ASYNC/AWAIT).

```typescript
// Callback Pattern
function fetchData(callback: (data: string) => void): void {
  setTimeout(() => {
    callback("Data diterima");
  }, 1000);
}

fetchData((data) => console.log(data));

// Promise Pattern
function fetchDataPromise(): Promise<string> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve("Data diterima dari promise");
    }, 1000);
  });
}

fetchDataPromise().then(data => console.log(data));

// Async/Await Pattern
async function fetchDataAsync(): Promise<string> {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve("Data diterima dari async");
    }, 1000);
  });
}

async function main() {
  const data = await fetchDataAsync();
  console.log(data);
}

main();
```

### 3. Callback

Fungsi yang dikirim sebagai parameter ke fungsi lain untuk dipanggil kembali.

```typescript
type Callback<T> = (result: T) => void;

function processFile(
  path: string,
  onSuccess: Callback<string>,
  onError: Callback<Error>
): void {
  // Simulate async file operation
  setTimeout(() => {
    if (path.includes("invalid")) {
      onError(new Error("File not found"));
    } else {
      onSuccess("File content");
    }
  }, 500);
}

processFile(
  "data.txt",
  (content) => console.log("Success:", content),
  (error) => console.error("Error:", error.message)
);
```

### 4. Promise

Objek yang merepresentasikan hasil suatu operasi async di masa depan.

```typescript
// Promise states: pending, fulfilled, rejected
interface PromiseState<T> {
  status: "pending" | "fulfilled" | "rejected";
  value?: T;
  reason?: Error;
}

// Creating Promise
const fetchData = new Promise<string>((resolve, reject) => {
  const success = Math.random() > 0.5;
  
  setTimeout(() => {
    if (success) {
      resolve("Data berhasil");
    } else {
      reject(new Error("Gagal mengambil data"));
    }
  }, 1000);
});

// Chaining
fetchData
  .then(data => {
    console.log(data.toUpperCase());
    return data.length;
  })
  .then(length => console.log("Length:", length))
  .catch(error => console.error(error));
```

### 5. Async/Await

Sintaks modern untuk menangani Promise secara synchronous.

```typescript
async function getUser(userId: number): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) {
    throw new Error("User not found");
  }
  return response.json();
}

async function main() {
  try {
    const user = await getUser(1);
    console.log(user);
  } catch (error) {
    console.error("Error:", error.message);
  }
}
```

### 6. Event Loop

Mekanisme di JavaScript yang menangani eksekusi async.

```
Call Stack -> Web APIs -> Callback Queue -> Event Loop -> Call Stack
```

Contoh:

```typescript
console.log("1. Mulai");

setTimeout(() => console.log("3. Timeout"), 0);

Promise.resolve().then(() => console.log("2. Promise"));

console.log("4. Selesai");

// Output: 1, 4, 2, 3
```

### 7. Error Handling Asynchronous

```typescript
async function riskyOperation(): Promise<string> {
  throw new Error("Something went wrong");
}

// Cara 1: Try-catch
async function handleWithError() {
  try {
    const result = await riskyOperation();
    return result;
  } catch (error) {
    return `Error: ${(error as Error).message}`;
  }
}

// Cara 2: Promise.catch
riskyOperation()
  .then(result => console.log(result))
  .catch(error => console.error(error));

// Cara 3: Promise.all (untuk multiple async)
async function handleMultiple() {
  try {
    const [user, posts, comments] = await Promise.all([
      fetchUser(1),
      fetchPosts(1),
      fetchComments(1)
    ]);
    return { user, posts, comments };
  } catch (error) {
    console.error("One of the requests failed:", error);
    throw error;
  }
}
```

## Praktikum

**Tujuan**: Membuat program asynchronous dan simulasi pengambilan data dari server.

### Implementasi

```typescript
interface User {
  id: number;
  nama: string;
  email: string;
}

interface Post {
  id: number;
  userId: number;
  title: string;
  body: string;
}

// Simulate API delay
function apiDelay(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Simulate API responses
async function fetchUser(id: number): Promise<User> {
  await apiDelay(500);
  if (id < 1 || id > 10) throw new Error(`User ${id} not found`);
  return {
    id,
    nama: `User ${id}`,
    email: `user${id}@example.com`
  };
}

async function fetchPosts(userId: number): Promise<Post[]> {
  await apiDelay(300);
  return [
    { id: 1, userId, title: "Post 1", body: "Content 1" },
    { id: 2, userId, title: "Post 2", body: "Content 2" }
  ];
}

async function fetchUserWithPosts(userId: number): Promise<{
  user: User;
  posts: Post[];
}> {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error) {
    throw new Error(`Failed to fetch user ${userId}: ${(error as Error).message}`);
  }
}

// Run simulation
async function main() {
  console.log("=== Async Simulation ===\n");

  // Sequential
  console.log("1. Sequential fetching:");
  const user1 = await fetchUser(1);
  const posts1 = await fetchPosts(user1.id);
  console.log(`User: ${user1.nama}`);
  console.log(`Posts: ${posts1.length}\n`);

  // Parallel
  console.log("2. Parallel fetching:");
  const startTime = Date.now();
  const [user2, user3] = await Promise.all([
    fetchUser(2),
    fetchUser(3)
  ]);
  console.log(`User 2: ${user2.nama}, User 3: ${user3.nama}`);
  console.log(`Time: ${Date.now() - startTime}ms\n`);

  // Error handling
  console.log("3. Error handling:");
  try {
    await fetchUser(999);
  } catch (error) {
    console.log(`Caught error: ${(error as Error).message}`);
  }
}

main();
```

## Tugas Tambahan

1. Implementasikan retry mechanism untuk fetch data
2. Tambahkan timeout untuk setiap request
3. Buat fungsi batch fetch dengan Promise.allSettled

## Referensi

- [MDN: Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [MDN: async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)