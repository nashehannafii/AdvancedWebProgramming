# Prerequest - Prerequisites

## Navigasi

[🏠 Beranda](../README.md) | [🔧 Prerequest](./README.md) | [Pertemuan 01](../pertemuan/pertemuan-01/README.md) | [Pertemuan 02](../pertemuan/pertemuan-02/README.md) | ... | [Pertemuan 16](../pertemuan/pertemuan-16/README.md)

---

## Required Tools

### Runtime

| Tool | Deskripsi | Link |
|------|-----------|------|
| **Bun** | JavaScript Runtime cepat | [bun.sh](https://bun.sh/) |

### Frameworks

| Tool | Deskripsi | Link |
|------|-----------|------|
| **Elysia** | Web Framework untuk Backend | [elysiajs.com](https://elysiajs.com/) |
| **Svelte** | Frontend Framework | [svelte.dev](https://svelte.dev/) |

### Database

| Tool | Deskripsi | Link |
|------|-----------|------|
| **MySQL** | Database Server | [mysql.com](https://www.mysql.com/) |

### DevOps

| Tool | Deskripsi | Link |
|------|-----------|------|
| **Git** | Version Control | [git-scm.com](https://git-scm.com/) |
| **GitHub** | Git Platform | [github.com](https://github.com/) |
| **Cloudflare Tunnel** | Remote Access | [cloudflare.com](https://developers.cloudflare.com/cloudflare-one/connections/build-tunnels/) |
| **Caddy/Nginx** | Reverse Proxy | - |

## Installation

### Bun

**Windows (menggunakan winget):**

```bash
winget install Bun
```

**Menggunakan npm:**

```bash
npm install -g bun
```

**Verifikasi instalasi:**

```bash
bun --version
```

### Elysia

```bash
mkdir backend && cd backend
bun init
bun add elysia
```

### Svelte (TypeScript)

```bash
npm create svelte@latest frontend
cd frontend
npm install
npm install -D svelte-jester @sveltejs/vite-plugin-svelte
```

### MySQL

```bash
# Windows: gunakan MySQL Installer atau download dari mysql.com
# Setup root password saat instalasi
```

### Git & GitHub

```bash
# Windows
winget install Git.Git

# Configure git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Caddy

```bash
# Windows
winget install caddy

# atau download langsung dari caddyserver.com
```

### Cloudflare Tunnel

```bash
# Install Cloudflare CLI (Windows)
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-win-x64.zip -o cloudflared.zip
# Extract dan letakkan di PATH
```

## Project Structure

```text
AdvancedWebProgramming/
├── prerequest/           # Prerequisites and installation
├── pertemuan/
│   ├── pertemuan-01/     # TypeScript dasar
│   ├── pertemuan-02/     # TypeScript lanjutan & modular
│   ├── pertemuan-03/     # Sync & Async programming
│   ├── pertemuan-04/     # HTTP & Web Service
│   ├── pertemuan-05/     # Bun & Elysia
│   ├── pertemuan-06/     # REST API dengan Elysia
│   ├── pertemuan-07/     # Database MySQL & Integrasi
│   ├── pertemuan-08/     # Ujian Tengah Semester
│   ├── pertemuan-09/     # Svelte dasar
│   ├── pertemuan-10/     # Integrasi Svelte dengan API
│   ├── pertemuan-11/     # Form & Validasi
│   ├── pertemuan-12/     # JWT Authentication
│   ├── pertemuan-13/     # OAuth 2.0
│   ├── pertemuan-14/     # Web Security & Architecture
│   ├── pertemuan-15/     # Testing, CI/CD, Deployment
│   └── pertemuan-16/     # Final Project
├── ACUAN.md              # Pedoman modul
├── PERTEMUAN.md          # Detail pertemuan
└── README.md             # File ini
```