# MODUL PEMBELAJARAN PEMROGRAMAN WEB LANJUT

## A. Identitas Mata Kuliah

- Nama Mata Kuliah: Pemrograman Web Lanjut
- Level: Perguruan Tinggi / Semester Lanjut
- Jumlah Pertemuan: 16 Pertemuan
- Pendekatan: Theory + Hands-on Practice + Project-Based Learning
- Bahasa Pemrograman: TypeScript
- Backend Runtime: Bun
- Backend Framework: Elysia
- Frontend Framework: Svelte
- Database: MySQL
- API Architecture: REST API
- Version Control: Git + GitHub
- CI/CD: GitHub Actions
- Reverse Proxy: Caddy atau Nginx
- Public Access: Cloudflare Tunnel
- Deployment Environment: Local Computer sebagai Server
- Authentication: JWT dan OAuth 2.0

---

# B. Tujuan Pembelajaran Umum

Setelah menyelesaikan mata kuliah ini, mahasiswa mampu:

1. Mengembangkan aplikasi web menggunakan TypeScript.
2. Memahami konsep synchronous dan asynchronous programming.
3. Memahami HTTP dan komunikasi client-server.
4. Membangun Web Service dan REST API menggunakan Bun dan Elysia.
5. Menghubungkan backend dengan database MySQL.
6. Mengembangkan frontend menggunakan Svelte dan TypeScript.
7. Mengintegrasikan frontend dengan REST API.
8. Menerapkan form validation dan error handling.
9. Menerapkan authentication menggunakan JWT.
10. Memahami dan menerapkan OAuth 2.0.
11. Menerapkan authorization berbasis role dan permission.
12. Memahami keamanan aplikasi web.
13. Menerapkan arsitektur aplikasi full-stack yang terstruktur.
14. Melakukan testing aplikasi.
15. Menerapkan Git dan GitHub dalam pengembangan aplikasi.
16. Membangun CI/CD menggunakan GitHub Actions.
17. Memahami dan menerapkan reverse proxy.
18. Melakukan deployment aplikasi pada komputer lokal.
19. Menggunakan Cloudflare Tunnel agar aplikasi lokal dapat diakses melalui domain.
20. Mengembangkan aplikasi full-stack sebagai final project.

---

# C. Teknologi yang Digunakan

## Frontend

- Svelte
- TypeScript
- HTML
- CSS
- Fetch API

## Backend

- Bun
- TypeScript
- Elysia
- REST API

## Database

- MySQL
- SQL

## Authentication

- JWT
- OAuth 2.0

## Development Tools

- VS Code
- Git
- GitHub
- Postman atau Bruno

## DevOps

- GitHub Actions
- Self-hosted Runner
- Caddy atau Nginx
- Cloudflare Tunnel

---

# D. Arsitektur Aplikasi

Gunakan arsitektur berikut sebagai target pembelajaran:

```text
                         INTERNET
                             |
                             v
                         Domain
                             |
                             v
                    Cloudflare Tunnel
                             |
                             v
                     Local Computer
                             |
                         Reverse Proxy
                        Caddy / Nginx
                             |
                +------------+------------+
                |                         |
                v                         v
        Svelte Frontend            Elysia Backend
           TypeScript                 TypeScript
                                        |
                                        v
                                       Bun
                                        |
                                        v
                                      MySQL