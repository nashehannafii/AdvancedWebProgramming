# Pertemuan 07: Database dan Integrasi Backend

## Materi

- Database relational
- MySQL
- Database design
- Table
- Primary key
- Foreign key
- Relationship
- SQL (SELECT, INSERT, UPDATE, DELETE, JOIN)
- Database connection

## Praktikum

Menghubungkan Elysia dengan MySQL dan membuat CRUD menggunakan database.

## Tujuan

Mahasiswa mampu:
1. Mendesain database relational
2. Menghubungkan backend dengan MySQL
3. Membuat CRUD berbasis database

## Struktur Project

```
pertemuan-07/
├── src/
│   ├── server.ts
│   ├── repositories/
│   │   └── mahasiswa-repository.ts
│   └── config/
│       └── database.ts
├── sql/
│   └── schema.sql
└── package.json
```

##chema Database

```sql
CREATE TABLE mahasiswa (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    nim VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    nilai_double DOUBLE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```