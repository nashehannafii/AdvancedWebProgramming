# Pertemuan 01: Modern JavaScript dan TypeScript

## Navigasi

[🏠 Beranda](../README.md) | [🔧 Prerequest](../prerequest/README.md) | **Pertemuan 01** | [Pertemuan 02](pertemuan-02/README.md) | [Pertemuan 03](pertemuan-03/README.md) | ... | [Pertemuan 16](pertemuan-16/README.md)

---

## Materi

### 1. Review JavaScript Modern

JavaScript modern mencakup fitur-fitur penting yang memudahkan pengembangan:

| Fitur | Deskripsi |
|-------|-----------|
| `let` / `const` | Deklarasi variabel dengan scope blok |
| Arrow Function | Sintaks fungsi lebih ringkas |
| Template Literals | String interpolation dengan backtick |
| Destructuring | Ekstraksi nilai dari array/objek |
| Spread Operator | Menyebarkan elemen array/objek |
| Default Parameters | Nilai default parameter fungsi |
| Rest Parameter | Mengumpulkan parameter menjadi array |

### 2. ES6+ Features

```javascript
// Arrow Function
const add = (a, b) => a + b;

// Template Literal
const name = "John";
const greeting = `Hello, ${name}!`;

// Destructuring
const { x, y } = { x: 10, y: 20 };

// Spread Operator
const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4]

// Default Parameter
function greet(name = "Guest") {
  return `Hello, ${name}!`;
}
```

### 3. TypeScript Dasar

TypeScript adalah superset JavaScript yang menambahkan static typing.

#### Kenapa Pakai TypeScript?

- **Static Typing**: Deteksi error sebelum runtime
- **IDE Support**: Autocompute, type checking
- **Code Maintainability**: Dokumentasi code lebih baik
- **Refactoring Safety**: Perubahan lebih aman

#### Konfigurasi Dasar (`tsconfig.json`)

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

#### Primitive Types

| Type | Contoh | Keterangan |
|------|--------|------------|
| `number` | `42`, `3.14` | Semua angka |
| `string` | `"hello"` | Teks |
| `boolean` | `true`, `false` | Boolean |
| `null` | `null` | Nilai kosong |
| `undefined` | `undefined` | Nilai tidak diterima |
| `void` | `void` | Tidak ada return value |
| `any` | `any` | Tipe dinamis (hindari) |

### 4. Array Types

```typescript
// Array dengan tipe tertentu
const numbers: number[] = [1, 2, 3, 4, 5];

// Array dengan tipe union
const mixed: (string | number)[] = ["hello", 42];

// Tuple
const person: [string, number] = ["John", 30];
```

### 5. Object Types

```typescript
// Interface
interface Mahasiswa {
  nama: string;
  nim: string;
  nilai?: number;  // optional property
}

// Object literal
const mhs: Mahasiswa = {
  nama: "Budi",
  nim: "12345678",
  nilai: 85.5
};
```

### 6. Function Types

```typescript
// Function declaration
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Function dengan optional parameter
function greet(name: string, greeting: string = "Hello"): string {
  return `${greeting}, ${name}!`;
}
```

### 7. Interface

```typescript
interface User {
  id: number;
  nama: string;
  email: string;
}

// Implementasi interface
const user: User = {
  id: 1,
  nama: "Alice",
  email: "alice@example.com"
};
```

## Praktikum

**Tujuan**: Membuat program pengelolaan data mahasiswa menggunakan TypeScript.

### Langkah-Langkah

1. **Inisialisasi Project**

```bash
bun init
bun add -d typescript @types/node
npx tsc --init
```

2. **Buat Struktur Direktori**

```bash
mkdir src
touch src/mahasiswa.ts
touch tsconfig.json
```

3. **Implementasi Model Mahasiswa** (`src/mahasiswa.ts`)

```typescript
interface Mahasiswa {
  id: number;
  nama: string;
  nim: string;
  email: string;
  nilai: number;
}

class MahasiswaManager {
  private mahasiswas: Mahasiswa[] = [];
  private nextId: number = 1;

  add(nama: string, nim: string, email: string, nilai: number): Mahasiswa {
    const mhs: Mahasiswa = {
      id: this.nextId++,
      nama,
      nim,
      email,
      nilai
    };
    this.mahasiswas.push(mhs);
    return mhs;
  }

  getAll(): Mahasiswa[] {
    return this.mahasiswas;
  }

  getById(id: number): Mahasiswa | undefined {
    return this.mahasiswas.find(mhs => mhs.id === id);
  }

  getByNim(nim: string): Mahasiswa | undefined {
    return this.mahasiswas.find(mhs => mhs.nim === nim);
  }

  update(id: number, data: Partial<Mahasiswa>): Mahasiswa | null {
    const index = this.mahasiswas.findIndex(mhs => mhs.id === id);
    if (index === -1) return null;
    
    this.mahasiswas[index] = { ...this.mahasiswas[index], ...data };
    return this.mahasiswas[index];
  }

  delete(id: number): boolean {
    const index = this.mahasiswas.findIndex(mhs => mhs.id === id);
    if (index === -1) return false;
    
    this.mahasiswas.splice(index, 1);
    return true;
  }

  getKthl(): Mahasiswa[] {
    return this.mahasiswas
      .filter(mhs => mhs.nilai >= 75)
      .sort((a, b) => b.nilai - a.nilai);
  }
}

// Demo penggunaan
const manager = new MahasiswaManager();

manager.add("Budi", "20240001", "budi@example.com", 85.5);
manager.add("Siti", "20240002", "siti@example.com", 90.0);
manager.add("Joko", "20240003", "joko@example.com", 72.3);

console.log("Semua Mahasiswa:", manager.getAll());
console.log("Mahasiswa dengan ID 1:", manager.getById(1));
console.log("Mahasiswa Lulus:", manager.getKthl());

manager.update(1, { nilai: 88 });
console.log("Setelah update:", manager.getById(1));

manager.delete(2);
console.log("Setelah delete:", manager.getAll());
```

4. **Jalankan Program**

```bash
bun run src/mahasiswa.ts
```

### Hasil yang Diharapkan

```
Semua Mahasiswa: [
  { id: 1, nama: 'Budi', nim: '20240001', email: 'budi@example.com', nilai: 85.5 },
  { id: 2, nama: 'Siti', nim: '20240002', email: 'siti@example.com', nilai: 90 },
  { id: 3, nama: 'Joko', nim: '20240003', email: 'joko@example.com', nilai: 72.3 }
]
Mahasiswa dengan ID 1: { id: 1, nama: 'Budi', nim: '20240001', email: 'budi@example.com', nilai: 85.5 }
Mahasiswa Lulus: [
  { id: 2, nama: 'Siti', nim: '20240002', email: 'siti@example.com', nilai: 90 },
  { id: 1, nama: 'Budi', nim: '20240001', email: 'budi@example.com', nilai: 85.5 }
]
Setelah update: { id: 1, nama: 'Budi', nim: '20240001', email: 'budi@example.com', nilai: 88 }
Setelah delete: [
  { id: 1, nama: 'Budi', nim: '20240001', email: 'budi@example.com', nilai: 88 },
  { id: 3, nama: 'Joko', nim: '20240003', email: 'joko@example.com', nilai: 72.3 }
]
```

## Tugas Tambahan

1. Tambahkan method `getRataRataNilai()` untuk menghitung rata-rata nilai semua mahasiswa
2. Implementasikan pencarian mahasiswa berdasarkan nama (case-insensitive)
3. Tambahkan validasi: NIM tidak boleh duplikat

## Referensi

- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Bun Documentation](https://bun.sh/docs/)
- [MDN Web Docs - JavaScript](https://developer.mozilla.org/id/docs/Web/JavaScript)