# Prerequest - Prerequisites

## Required Tools

### Runtime
- [Bun](https://bun.sh/) - JavaScript Runtime

### Frameworks
- [Elysia](https://elysiajs.com/) - Web Framework untuk Backend
- [Svelte](https://svelte.dev/) - Frontend Framework

### Database
- [MySQL](https://www.mysql.com/) - Database Server

### DevOps
- [Git](https://git-scm.com/) - Version Control
- [GitHub](https://github.com/) - Git Platform
- [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/build-tunnels/) - Remote Access
- Caddy atau Nginx - Reverse Proxy

## Installation

### Bun
```bash
# Windows (using winget)
winget install Bun

# atau menggunakan npm
npm install -g bun

# Verify
bun --version
```

### Elysia
```bash
mkdir backend && cd backend
bun init
bun add elysia
```

### Svelte
```bash
# Create Svelte project
npm create svelte@latest frontend
cd frontend
npm install
npm install -D svelte-jester @sveltejs/vite-plugin-svelte
```

### MySQL
```bash
# Windows (using MySQL Installer atau download dari mysql.com)
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

# atau download dari caddyserver.com
```

### Cloudflare Tunnel
```bash
# Install Cloudflare CLI
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-win-x64.zip -o cloudflared.zip
# Extract dan letakkan di PATH
```

## Project Structure
```text
AdvancedWebProgramming/
├── prerequest/           # Prerequisites and installation
├── pertemuan-01/         # TypeScript dasar
├── pertemuan-02/         # TypeScript lanjutan & modular
├── pertemuan-03/         # Sync & Async programming
├── pertemuan-04/         # HTTP & Web Service
├── pertemuan-05/         # Bun & Elysia
├── pertemuan-06/         # REST API dengan Elysia
├── pertemuan-07/         # Database MySQL & Integrasi
├── pertemuan-08/         # Ujian Tengah Semester
├── pertemuan-09/         # Svelte dasar
├── pertemuan-10/         # Integrasi Svelte dengan API
├── pertemuan-11/         # Form & Validasi
├── pertemuan-12/         # JWT Authentication
├── pertemuan-13/         # OAuth 2.0
├── pertemuan-14/         # Web Security & Architecture
├── pertemuan-15/         # Testing, CI/CD, Deployment
└── pertemuan-16/         # Final Project
```