# Pertemuan 02: TypeScript Lanjutan dan Modular Programming

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 01](../pertemuan-01/README.md) | [Selanjutnya: Pertemuan 03 ->](../pertemuan-03/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Membedakan `type` vs `interface` dan memilih yang tepat untuk kasus tertentu
- Menggunakan union type, optional property (`?`), dan generic (`Stack<T>`, `identity<T>`) pada kasus nyata
- Mengetikkan function dengan tepat (parameter, return type, rest & callback)
- Menerapkan modular programming: `import`/`export`, barrel file, dan struktur project `models/ services/ utils/`
- Membangun aplikasi TypeScript modular dengan pemisahan model, service, dan utility yang dapat diuji

> **Indikator ketercapaian:** Project terpisah rapi, tidak ada circular import, service tervalidasi dan dapat di-`import` antar modul.

---

## Materi

### 1. Type dan Type Annotations

```typescript
// Type annotation
let age: number = 25;
let name: string = "Alice";
let isActive: boolean = true;

// Type inference (otomatis)
let count = 0; // TypeScript menebak number

// Explicit any (hindari)
let data: any = "bisa apa-apa";
```

### 2. Interface vs Type

```typescript
// Interface
interface User {
  id: number;
  nama: string;
  email: string;
}

// Type alias
type UserType = {
  id: number;
  nama: string;
  email: string;
};

// Interface dapat extend
interface Admin extends User {
  role: string;
}

// Type dapat extend dengan intersection
type AdminType = UserType & { role: string };
```

### 3. Union Type

```typescript
type Status = "pending" | "active" | "inactive";
let status: Status = "pending";

type ID = number | string;
function getUser(id: ID): User {
  // implementation
}
```

### 4. Optional Property

```typescript
interface Product {
  id: number;
  nama: string;
  deskripsi?: string;  // optional
  harga: number;
}
```

### 5. Generic Types

```typescript
// Generic function
function identity<T>(arg: T): T {
  return arg;
}

const num = identity<number>(42);
const str = identity<string>("hello");

// Generic class
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }
}
```

### 6. Function Typing

```typescript
// Function type dengan parameter dan return type
type CalculateFn = (a: number, b: number) => number;

const add: CalculateFn = (x, y) => x + y;
const multiply: CalculateFn = (x, y) => x * y;

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, curr) => acc + curr, 0);
}

// Callback function
function processData(data: string[], callback: (item: string) => string): string[] {
  return data.map(callback);
}
```

### 7. Module System

```typescript
// utils/math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export const PI = 3.14159;

// Export default
export default class Calculator {
  // ...
}

// utils/index.ts
export { add, PI } from "./math";
export { default as Calculator } from "./math";
```

### 8. Import/Export Patterns

```typescript
// Import default
import Calculator from "./calculator";

// Import named
import { add, PI } from "./math";

// Import all
import * as MathUtils from "./math";

// Import dengan alias
import { add as sum, PI as PI_VALUE } from "./math";
```

### 9. Struktur Project

```
src/
├── models/          # Definisi tipe data
├── services/        # Business logic
├── controllers/     # Request handling
├── utils/           # Helper functions
├── middlewares/     # Middleware
└── index.ts         # Entry point
```

## Praktikum

**Tujuan**: Membuat aplikasi TypeScript modular dengan pemisahan model, service, dan utility.

### Struktur Project

```
src/
├── models/
│   └── Mahasiswa.ts
├── services/
│   └── MahasiswaService.ts
├── utils/
│   └── Validation.ts
└── index.ts
```

### Implementasi

**1. Model (`src/models/Mahasiswa.ts`)**

```typescript
export interface IMahasiswa {
  id: number;
  nama: string;
  nim: string;
  email: string;
  nilai?: number;
}

export class Mahasiswa implements IMahasiswa {
  constructor(
    public id: number,
    public nama: string,
    public nim: string,
    public email: string,
    public nilai?: number
  ) {}
}
```

**2. Validation (`src/utils/Validation.ts`)**

```typescript
export class Validation {
  static isValidEmail(email: string): boolean {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
  }

  static isValidNIM(nim: string): boolean {
    const regex = /^\d{8}$/;
    return regex.test(nim);
  }

  static isValidNilai(nilai: number): boolean {
    return nilai >= 0 && nilai <= 100;
  }
}
```

**3. Service (`src/services/MahasiswaService.ts`)**

```typescript
import { Mahasiswa, IMahasiswa } from "../models/Mahasiswa";
import { Validation } from "../utils/Validation";

export class MahasiswaService {
  private mahasiswas: Mahasiswa[] = [];
  private nextId: number = 1;

  private validate(mhs: Partial<IMahasiswa>): string[] {
    const errors: string[] = [];

    if (mhs.nama && mhs.nama.length < 3) {
      errors.push("Nama minimal 3 karakter");
    }

    if (mhs.nim && !Validation.isValidNIM(mhs.nim)) {
      errors.push("NIM harus 8 digit angka");
    }

    if (mhs.email && !Validation.isValidEmail(mhs.email)) {
      errors.push("Email tidak valid");
    }

    if (mhs.nilai !== undefined && !Validation.isValidNilai(mhs.nilai)) {
      errors.push("Nilai harus antara 0-100");
    }

    return errors;
  }

  create(data: Omit<IMahasiswa, "id">): Mahasiswa | { error: string[] } {
    const errors = this.validate(data);
    if (errors.length > 0) {
      return { error: errors };
    }

    const mhs = new Mahasiswa(
      this.nextId++,
      data.nama,
      data.nim,
      data.email,
      data.nilai
    );
    this.mahasiswas.push(mhs);
    return mhs;
  }

  getAll(): Mahasiswa[] {
    return this.mahasiswas;
  }

  getById(id: number): Mahasiswa | undefined {
    return this.mahasiswas.find(mhs => mhs.id === id);
  }
}
```

**4. Entry Point (`src/index.ts`)**

```typescript
import { MahasiswaService } from "./services/MahasiswaService";

const service = new MahasiswaService();

// Create mahasiswa
const result1 = service.create({
  nama: "Budi Prima",
  nim: "20240001",
  email: "budi@example.com",
  nilai: 85.5
});

const result2 = service.create({
  nama: "Siti",
  nim: "invalid-nim",
  email: "invalid-email",
  nilai: 150
});

console.log("Result 1:", result1);
console.log("Result 2:", result2);
console.log("All mahasiswa:", service.getAll());
```

### Run Program

```bash
bun run src/index.ts
```

## Tugas Tambahan

1. Tambahkan method update dan delete di service
2. Implementasikan pencarian mahasiswa berdasarkan nama
3. Tambahkan unit test untuk validation

## Referensi

- [TypeScript Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [TypeScript Modules](https://www.typescriptlang.org/docs/handbook/modules.html)