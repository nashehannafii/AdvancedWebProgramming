Pertemuan 1
Modern JavaScript dan TypeScript

Materi:

Review JavaScript modern
ES6+
TypeScript
Static typing
Type inference
Primitive types
Array
Object
Function
Interface

Praktikum:
Membuat program pengelolaan data mahasiswa menggunakan TypeScript.

Pertemuan 2
TypeScript Lanjutan dan Modular Programming

Materi:

Type
Interface
Union type
Optional property
Generic
Function typing
Module
Import
Export
Struktur project

Praktikum:
Membuat aplikasi TypeScript modular dengan pemisahan model, service, dan utility.

Pertemuan 3
Synchronous dan Asynchronous Programming

Materi:

Synchronous programming
Asynchronous programming
Callback
Promise
async
await
Event loop
Error handling asynchronous

Praktikum:
Membuat program asynchronous dan simulasi pengambilan data dari server.

Pertemuan 4
HTTP dan Web Service

Materi:

Client-server architecture
HTTP
Request
Response
Header
Body
Status code
HTTP Method
JSON
REST
Endpoint

HTTP Method:

GET
POST
PUT
PATCH
DELETE

Praktikum:
Membuat HTTP server sederhana menggunakan Bun tanpa framework.

Pertemuan 5
Bun dan Elysia

Materi:

Pengenalan Bun
Bun sebagai JavaScript/TypeScript runtime
Instalasi dan konfigurasi Bun
Pengenalan Elysia
Server
Routing
Request
Response

Praktikum:
Membuat web service sederhana menggunakan Bun + Elysia.

Pertemuan 6
REST API dengan Elysia

Materi:

REST API
Resource
Endpoint
Routing
Route parameter
Query parameter
Request body
Response
HTTP status code
Error response
API documentation

Praktikum:
Membuat CRUD REST API menggunakan Elysia.

Endpoint contoh:

GET    /api/mahasiswa
GET    /api/mahasiswa/:id
POST   /api/mahasiswa
PUT    /api/mahasiswa/:id
PATCH  /api/mahasiswa/:id
DELETE /api/mahasiswa/:id
Pertemuan 7
Database dan Integrasi Backend

Materi:

Database relational
MySQL
Database design
Table
Primary key
Foreign key
Relationship
SQL
SELECT
INSERT
UPDATE
DELETE
JOIN
Database connection

Praktikum:
Menghubungkan Elysia dengan MySQL dan membuat CRUD menggunakan database.

Pertemuan 8
Ujian Tengah Semester

Bentuk:

Praktik
Project-based assessment

Tugas:
Membangun REST API menggunakan:

TypeScript
Bun
Elysia
MySQL

Minimal:

CRUD
REST API
Database
Validation
Error handling

Mahasiswa wajib mampu menjelaskan kode yang dibuat.

Pertemuan 9
Svelte dan TypeScript

Materi:

Pengenalan Svelte
Svelte project
Component
Props
State
Reactive programming
Event handling
Conditional rendering
List rendering
TypeScript pada Svelte

Praktikum:
Membuat frontend sederhana menggunakan Svelte + TypeScript.

Pertemuan 10
Integrasi Svelte dengan REST API

Materi:

Fetch API
HTTP request dari frontend
GET
POST
PUT
DELETE
Async/await
Loading state
Error state
API response

Praktikum:
Menghubungkan Svelte dengan REST API Elysia.

Target:

Svelte
   |
   | HTTP
   v
Elysia
   |
   v
MySQL
Pertemuan 11
Form, Validation dan Error Handling

Materi:

Form handling
Input binding
Form submission
Client-side validation
Server-side validation
Validation error
API error
Error handling
User feedback

Praktikum:
Membuat form CRUD mahasiswa dengan validasi frontend dan backend.

Pertemuan 12
Authentication dan JWT

Materi:

Authentication
Authorization
Login
Password hashing
Access token
Refresh token
JWT
Authorization header
Protected endpoint

Praktikum:
Membuat sistem login dengan JWT.

Contoh:

POST /api/login
POST /api/register
GET  /api/profile

Gunakan:

Authorization: Bearer <token>
Pertemuan 13
OAuth 2.0 dan Authorization

Materi:

OAuth 2.0
Authentication vs Authorization
OAuth roles
Client
Resource Owner
Authorization Server
Resource Server
Access Token
Scope
Authorization Code Flow
Redirect URI

Praktikum:
Mengimplementasikan login menggunakan OAuth 2.0 melalui identity provider.

Contoh:

Svelte
   |
   v
OAuth Provider
   |
   v
Authorization
   |
   v
Elysia Backend
Pertemuan 14
Web Security dan Application Architecture

Materi keamanan:

CORS
XSS
SQL Injection
CSRF
Password security
Token security
Input validation
Environment variables

Materi architecture:

Separation of concerns
Controller
Service
Repository
Middleware
Model
DTO
Modular architecture

Praktikum:
Refactoring aplikasi menjadi struktur backend yang lebih terorganisasi dan menerapkan security dasar.

Contoh:

src/
├── controllers/
├── services/
├── repositories/
├── models/
├── middleware/
├── routes/
├── utils/
└── server.ts
Pertemuan 15
Testing, CI/CD, Reverse Proxy dan Deployment

Materi Testing:

Unit testing
API testing
Integration testing

Materi Git:

Git workflow
Branch
Pull request
Merge

Materi CI/CD:

Continuous Integration
Continuous Delivery
Continuous Deployment
GitHub Actions
Self-hosted Runner
Automated testing
Automated build
Automated deployment

Materi Reverse Proxy:

Konsep reverse proxy
Caddy / Nginx
Domain
Port forwarding konsep
HTTPS
Routing domain ke service

Materi Tunnel:

Cloudflare Tunnel
Public access tanpa IP public

Target:

Git Push
   |
   v
GitHub
   |
   v
GitHub Actions
   |
   v
Test
   |
   v
Build
   |
   v
Self-hosted Runner
   |
   v
Local Computer
   |
   v
Caddy / Nginx
   |
   +--------> Svelte
   |
   +--------> Elysia

Kemudian:

Internet
   |
   v
Domain
   |
   v
Cloudflare Tunnel
   |
   v
Local Computer
Pertemuan 16
Final Project dan Presentasi

Mahasiswa mengembangkan dan mempresentasikan aplikasi full-stack.

Minimal menggunakan:

Frontend
Svelte + TypeScript

Backend
Bun + Elysia + TypeScript

Database
MySQL

API
REST API

Authentication
JWT atau OAuth 2.0

Version Control
Git + GitHub

CI/CD
GitHub Actions

Deployment
Local Computer

Reverse Proxy
Caddy / Nginx

Public Access
Cloudflare Tunnel

Minimal fitur:

Authentication
Authorization
CRUD
REST API
Database
Validation
Error handling
Security
Responsive frontend
CI/CD
Deployment
Domain