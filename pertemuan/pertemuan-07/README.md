# Pertemuan 07: Database dan Integrasi Backend

## Navigasi

[Beranda](../../README.md) | [Prerequest](../../prerequest/README.md) | [Daftar Pertemuan](../../README.md#daftar-pertemuan) | [<- Sebelumnya: Pertemuan 06](../pertemuan-06/README.md) | [Selanjutnya: Pertemuan 08 ->](../pertemuan-08/README.md)

---

## Capaian Pembelajaran

Setelah menyelesaikan pertemuan ini, mahasiswa mampu:

- Menjelaskan konsep database relational, tabel, primary key, foreign key, dan relasi
- Merancang skema MySQL untuk mahasiswa & mata kuliah (termasuk junction table) dengan constraint yang tepat
- Menulis SQL dasar: SELECT (JOIN, WHERE, ORDER BY, LIMIT), INSERT, UPDATE, DELETE/soft-delete
- Menghubungkan Elysia dengan MySQL via `mysql2/promise` pool dan mengelola koneksi
- Mengimplementasikan Repository-Service-Controller untuk CRUD berbasis database dengan pagination & pencarian

> **Indikator ketercapaian:** CRUD membaca/menulis ke MySQL (bukan in-memory), JOIN berfungsi, dan soft-delete tidak menghilangkan data permanen.

---
## Materi

### 1. Database Relational

Database relasional menyimpan data dalam tabel dengan hubungan.

#### Karakteristik

- **Structured**: Data dalam rows dan columns
- **Schema**: Struktur tetap
- **ACID**: Atomicity, Consistency, Isolation, Durability
- **SQL**: Query language standar

### 2. MySQL

MySQL adalah database open-source populer untuk web development.

#### Instalasi

```bash
# Windows - MySQL Installer
# atau via package manager
# Mac: brew install mysql
# Linux: sudo apt install mysql-server
```

#### Connect ke MySQL

```typescript
import { createConnection } from "mysql2/promise";

const connection = await createConnection({
  host: "localhost",
  user: "root",
  password: process.env.MYSQL_PASSWORD,
  database: "mahasiswa_db"
});
```

### 3. Database Design

#### Entity-Relationship Diagram (ERD)

```
Mahasiswa ──< Nilai >── MataKuliah
    │
    └── Many-to-One dengan Dosen
```

### 4. Table & Primary Key

```sql
-- Table: mahasiswa
CREATE TABLE mahasiswa (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    nim VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role ENUM('admin', 'user') DEFAULT 'user',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 5. Foreign Key & Relationship

```sql
-- Table: mata_kuliah
CREATE TABLE mata_kuliah (
    id INT AUTO_INCREMENT PRIMARY KEY,
    kode VARCHAR(10) UNIQUE NOT NULL,
    nama VARCHAR(100) NOT NULL,
    sks INT DEFAULT 0
);

-- Table: mahasiswa_mata_kuliah (junction table)
CREATE TABLE mahasiswa_mata_kuliah (
    id INT AUTO_INCREMENT PRIMARY KEY,
    mahasiswa_id INT,
    mata_kuliah_id INT,
    nilai DOUBLE,
    FOREIGN KEY (mahasiswa_id) REFERENCES mahasiswa(id) ON DELETE CASCADE,
    FOREIGN KEY (mata_kuliah_id) REFERENCES mata_kuliah(id) ON DELETE CASCADE
);
```

### 6. SQL Commands

#### SELECT

```sql
-- Basic SELECT
SELECT * FROM mahasiswa;

-- dengan WHERE
SELECT * FROM mahasiswa WHERE role = 'user';

-- dengan ORDER BY
SELECT * FROM mahasiswa ORDER BY created_at DESC;

-- dengan LIMIT
SELECT * FROM mahasiswa LIMIT 10;

-- JOIN
SELECT m.nama, mk.nama as mata_kuliah, mm.nilai
FROM mahasiswa m
JOIN mahasiswa_mata_kuliah mm ON m.id = mm.mahasiswa_id
JOIN mata_kuliah mk ON mm.mata_kuliah_id = mk.id;
```

#### INSERT

```sql
INSERT INTO mahasiswa (nama, nim, email, password_hash)
VALUES ('Budi', '20240001', 'budi@example.com', 'hashed_password');

INSERT INTO mahasiswa (nama, nim, email, password_hash, role)
VALUES ('Siti', '20240002', 'siti@example.com', 'hashed_password', 'admin');
```

#### UPDATE

```sql
UPDATE mahasiswa 
SET email = 'newemail@example.com' 
WHERE id = 1;

UPDATE mahasiswa 
SET last_login = NOW() 
WHERE nim = '20240001';
```

#### DELETE

```sql
DELETE FROM mahasiswa WHERE id = 1;

-- Soft delete
UPDATE mahasiswa SET deleted_at = NOW() WHERE id = 1;
```

### 7. Database Connection

```typescript
import mysql from "mysql2/promise";

class Database {
  private static instance: mysql.Pool;

  static getConnection() {
    if (!this.instance) {
      this.instance = mysql.createPool({
        host: process.env.DB_HOST || "localhost",
        user: process.env.DB_USER || "root",
        password: process.env.DB_PASSWORD || "",
        database: process.env.DB_NAME || "mahasiswa",
        waitForConnections: true,
        connectionLimit: 10,
        queueLimit: 0
      });
    }
    return this.instance;
  }
}

export default Database;
```

## Praktikum

**Tujuan**: Menghubungkan Elysia dengan MySQL dan membuat CRUD menggunakan database.

### Struktur Project

```
src/
├── config/
│   └── database.ts
├── models/
│   └── mahasiswa.ts
├── repositories/
│   └── mahasiswa-repository.ts
├── controllers/
│   └── mahasiswa-controller.ts
├── routes/
│   └── api.ts
└── server.ts
```

### Implementasi

**1. Database Config (`src/config/database.ts`)**

```typescript
import mysql from "mysql2/promise";

const pool = mysql.createPool({
  host: process.env.DB_HOST || "localhost",
  user: process.env.DB_USER || "root",
  password: process.env.DB_PASSWORD || "",
  database: process.env.DB_NAME || "mahasiswa",
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

export default pool;
```

**2. Repository (`src/repositories/mahasiswa-repository.ts`)**

```typescript
import pool from "../config/database";
import { Mahasiswa } from "../models/mahasiswa";

export class MahasiswaRepository {
  async getAll(search?: string): Promise<Mahasiswa[]> {
    let sql = "SELECT * FROM mahasiswa";
    const params: (string | number)[] = [];
    
    if (search) {
      sql += " WHERE nama LIKE ? OR nim LIKE ?";
      params.push(`%${search}%`, `%${search}%`);
    }
    
    const [rows] = await pool.execute(sql, params);
    return rows as Mahasiswa[];
  }

  async getById(id: number): Promise<Mahasiswa | null> {
    const [rows] = await pool.execute(
      "SELECT * FROM mahasiswa WHERE id = ? AND deleted_at IS NULL",
      [id]
    );
    const results = rows as Mahasiswa[];
    return results[0] || null;
  }

  async getByNim(nim: string): Promise<Mahasiswa | null> {
    const [rows] = await pool.execute(
      "SELECT * FROM mahasiswa WHERE nim = ? AND deleted_at IS NULL",
      [nim]
    );
    const results = rows as Mahasiswa[];
    return results[0] || null;
  }

  async create(data: Mahasiswa): Promise<Mahasiswa> {
    const { nama, nim, email, password_hash, role = 'user' } = data;
    const [result] = await pool.execute(
      "INSERT INTO mahasiswa (nama, nim, email, password_hash, role) VALUES (?, ?, ?, ?, ?)",
      [nama, nim, email, password_hash, role]
    );
    
    const insertId = (result as any).insertId;
    return { id: insertId, ...data };
  }

  async update(id: number, data: Partial<Mahasiswa>): Promise<Mahasiswa | null> {
    const fields = Object.keys(data)
      .filter(key => data[key as keyof typeof data] !== undefined)
      .map(key => `${key} = ?`);
    
    const values = Object.values(data);
    values.push(id);
    
    const [result] = await pool.execute(
      `UPDATE mahasiswa SET ${fields.join(", ")} WHERE id = ? AND deleted_at IS NULL`,
      values
    );
    
    if ((result as any).affectedRows === 0) return null;
    return this.getById(id);
  }

  async delete(id: number): Promise<boolean> {
    const [result] = await pool.execute(
      "UPDATE mahasiswa SET deleted_at = NOW() WHERE id = ?",
      [id]
    );
    return (result as any).affectedRows > 0;
  }

  async count(search?: string): Promise<number> {
    let sql = "SELECT COUNT(*) as total FROM mahasiswa";
    const params: (string | number)[] = [];
    
    if (search) {
      sql += " WHERE nama LIKE ? OR nim LIKE ?";
      params.push(`%${search}%`, `%${search}%`);
    }
    
    const [rows] = await pool.execute(sql, params);
    const results = rows as any[];
    return results[0]?.total || 0;
  }
}
```

**3. Model (`src/models/mahasiswa.ts`)**

```typescript
export interface Mahasiswa {
  id: number;
  nama: string;
  nim: string;
  email: string;
  password_hash?: string;
  role?: 'admin' | 'user' | 'moderator';
  created_at?: string;
  updated_at?: string;
  deleted_at?: string | null;
}
```

**4. Controller (`src/controllers/mahasiswa-controller.ts`)**

```typescript
import { MahasiswaRepository } from "../repositories/mahasiswa-repository";

const repo = new MahasiswaRepository();

export class MahasiswaController {
  async getAll(query: any) {
    const { page = 1, limit = 10, search = "" } = query;
    const offset = (page - 1) * limit;
    
    const [mahasiswas, total] = await Promise.all([
      repo.getAll(search || undefined),
      repo.count(search || undefined)
    ]);
    
    return {
      data: mahasisiswas.slice(offset, offset + limit),
      meta: {
        total,
        page: Number(page),
        limit: Number(limit),
        totalPages: Math.ceil(total / limit)
      }
    };
  }

  async getById(id: number) {
    const mhs = await repo.getById(id);
    if (!mhs) throw new Error("Mahasiswa tidak ditemukan");
    return mhs;
  }

  async create(body: any) {
    return repo.create({
      ...body,
      password_hash: "hashed_" + body.password
    });
  }

  async update(id: number, body: any) {
    const mhs = await repo.update(id, body);
    if (!mhs) throw new Error("Mahasiswa tidak ditemukan");
    return mhs;
  }

  async delete(id: number) {
    const success = await repo.delete(id);
    if (!success) throw new Error("Mahasiswa tidak ditemukan");
    return { message: "Mahasiswa berhasil dihapus" };
  }
}
```

**5. Routes (`src/routes/api.ts`)**

```typescript
import { Elysia, t } from "elysia";
import { MahasiswaController } from "../controllers/mahasiswa-controller";

const controller = new MahasiswaController();

export const apiRoutes = new Elysia()
  .get("/api/mahasiswa", async ({ query }) => controller.getAll(query))
  .get("/api/mahasiswa/:id", async ({ params: { id } }) => controller.getById(Number(id)), {
    params: t.Object({ id: t.Number() })
  })
  .post("/api/mahasiswa", async ({ body }) => controller.create(body), {
    body: t.Object({
      nama: t.String(),
      nim: t.String(),
      email: t.Email(),
      password: t.String()
    })
  })
  .put("/api/mahasiswa/:id", async ({ params, body }) => {
    return controller.update(Number(params.id), body);
  }, {
    params: t.Object({ id: t.Number() }),
    body: t.Object({
      nama: t.Optional(t.String()),
      email: t.Optional(t.Email()),
      password: t.Optional(t.String())
    })
  })
  .delete("/api/mahasiswa/:id", async ({ params: { id } }) => {
    return controller.delete(Number(id));
  }, {
    params: t.Object({ id: t.Number() })
  });
```

## Praktikum Lanjutan

Buat database MySQL:

```sql
CREATE DATABASE mahasiswa CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE mahasiswa;

CREATE TABLE mahasiswa (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    nim VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role ENUM('admin', 'user') DEFAULT 'user',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL DEFAULT NULL
);
```

## Referensi

- [MySQL Documentation](https://dev.mysql.com/doc/)
- [mysql2 npm package](https://www.npmjs.com/package/mysql2)