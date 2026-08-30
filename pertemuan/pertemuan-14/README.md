# Pertemuan 14: Web Security & Application Architecture

## Navigasi

[🏠 Beranda](../README.md) | [🔧 Prerequest](../prerequest/README.md) | ... | [Pertemuan 13](pertemuan-13/README.md) | **Pertemuan 14** | [Pertemuan 15](pertemuan-15/README.md) | ... | [Pertemuan 16](pertemuan-16/README.md)

---

## Materi Keamanan

### 1. CORS (Cross-Origin Resource Sharing)

```typescript
// Elysia CORS configuration
new Elysia()
  .use(({ onBeforeHandle }) => {
    onBeforeHandle(({ request, headers }) => {
      const origin = headers.origin || "http://localhost:5173";
      const allowedOrigins = [
        "http://localhost:5173",
        "https://myapp.com"
      ];
      
      if (allowedOrigins.includes(origin)) {
        response.headers.set("Access-Control-Allow-Origin", origin);
        response.headers.set("Access-Control-Allow-Methods", "GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS");
        response.headers.set("Access-Control-Allow-Headers", "Content-Type,Authorization");
        response.headers.set("Access-Control-Allow-Credentials", "true");
        response.headers.set("Access-Control-Max-Age", "86400");
      }
    });
  })
```

### 2. XSS (Cross-Site Scripting)

```typescript
// Sanitize output
import DOMPurify from "dompurify";

const cleanHTML = DOMPurify.sanitize(userInput);
```

### 3. SQL Injection

```typescript
// JANGAN pakai string concatenation
const query = `SELECT * FROM users WHERE email = '${email}'`; // DANGEROUS

// Pakai parameter queries
const query = "SELECT * FROM users WHERE email = ?";
const [rows] = await pool.execute(query, [email]); // SAFE
```

### 4. CSRF (Cross-Site Request Forgery)

```typescript
// Generate CSRF token
function generateCSRFToken(): string {
  return crypto.randomBytes(32).toString("hex");
}

// Protected POST endpoint
.post("/api/orders", ({ body, headers, verifyCSRFToken }) => {
  const csrfToken = headers["x-csrf-token"];
  
  if (!verifyCSRFToken(csrfToken)) {
    throw new Error("Invalid CSRF token");
  }
  
  return createOrder(body);
})
```

### 5. Password Security

```typescript
import { hash, compare } from "bcrypt";

// Hash password
const password_hash = await hash(password, 12);

// Verify password timing-safe
const isValid = await compare(password, password_hash);
```

### 6. Token Security

```typescript
// JWT token dengan additional claims
const payload = {
  userId: 1,
  email: "user@example.com",
  role: "user",
  iat: Math.floor(Date.now() / 1000),
  exp: Math.floor(Date.now() / 1000) + 3600,
  jti: crypto.randomUUID() // JWT ID untuk unique token
};
```

### 7. Input Validation

```typescript
// Elysia validation patterns
const schemas = {
  email: t.String().email(),
  phone: t.String().pattern(/^\+?[0-9]{10,15}$/),
  url: t.String().url(),
  slug: t.String().pattern(/^[a-z0-9-]+$/)
};

// Validation middleware
const validationMiddleware = (schema: Schema) => ({
  body: schema,
  query: schema,
  params: schema
});
```

### 8. Environment Variables

```bash
# .env file
JWT_SECRET=your-super-secret-key-change-this
DB_PASSWORD=secure-password
API_KEY=external-api-key

# JANGAN commit .env ke git
echo ".env" >> .gitignore
```

```typescript
// Load environment
import { config } from "dotenv";
config();

const jwtSecret = process.env.JWT_SECRET!;
if (!jwtSecret) throw new Error("JWT_SECRET not set");
```

---

## Materi Architecture

### 1. Separation of Concerns

```
src/
├── controllers/    # Handle request/response
├── services/       # Business logic
├── repositories/   # Data access
├── models/         # Data structures
├── middleware/     # Request processing
├── routes/         # Route definitions
├── utils/          # Helper functions
└── server.ts       # Entry point
```

### 2. Layer Architecture

```
┌─────────────────────────────────────┐
│           Presentation              │
│            (Controllers)            │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│            Application              │
│             (Services)              │
└────────────────┬────────────────────┘
                 │
┌────────────────▼────────────────────┐
│            Infrastructure           │
│            (Repositories)           │
└─────────────────────────────────────┘
```

### 3. Controller

```typescript
class UserController {
  constructor(private userService: UserService) {}
  
  async getProfile(req: Request, res: Response) {
    const userId = req.user.id;
    const profile = await this.userService.getProfile(userId);
    return res.json(profile);
  }
}
```

### 4. Service

```typescript
class UserService {
  constructor(private userRepo: UserRepository) {}
  
  async getProfile(userId: number): Promise<UserProfile> {
    // Business logic
    const user = await this.userRepo.findById(userId);
    
    // Additional processing
    const profile = {
      ...user,
      fullName: `${user.firstName} ${user.lastName}`,
      displayPicture: user.avatarUrl || "/default-avatar.png"
    };
    
    return profile;
  }
}
```

### 5. Repository

```typescript
class UserRepository {
  constructor(private db: Database) {}
  
  async findById(id: number): Promise<User> {
    const [rows] = await this.db.execute(
      "SELECT * FROM users WHERE id = ?",
      [id]
    );
    return rows[0] as User;
  }
}
```

### 6. Middleware

```typescript
// Auth middleware
function authMiddleware() {
  return ({ request, response, next }) => {
    const token = request.headers.authorization?.split(" ")[1];
    
    if (!token) {
      return response.status(401).json({ error: "Unauthorized" });
    }
    
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      request.user = decoded;
      next();
    } catch (error) {
      return response.status(401).json({ error: "Invalid token" });
    }
  };
}
```

### 7. Model & DTO

```typescript
// Entity (Database model)
interface UserEntity {
  id: number;
  email: string;
  password_hash: string;
  created_at: Date;
  updated_at: Date;
}

// DTO (Data Transfer Object)
interface UserDTO {
  id: number;
  email: string;
  name: string;
  role: string;
}

// Mapping function
function toUserDTO(entity: UserEntity): UserDTO {
  return {
    id: entity.id,
    email: entity.email,
    name: `${entity.firstName} ${entity.lastName}`,
    role: entity.role
  };
}
```

### 8. Modular Architecture

```
src/
├── controllers/
│   ├── auth.controller.ts
│   └── user.controller.ts
├── services/
│   ├── auth.service.ts
│   └── user.service.ts
├── repositories/
│   ├── auth.repository.ts
│   └── user.repository.ts
├── models/
│   ├── user.model.ts
│   └── index.ts
├── middleware/
│   ├── auth.middleware.ts
│   └── error.middleware.ts
├── routes/
│   ├── auth.routes.ts
│   └── user.routes.ts
├── dtos/
│   ├── auth.dto.ts
│   └── user.dto.ts
├── utils/
│   ├── logger.ts
│   └── validator.ts
└── server.ts
```

---

## Praktikum

**Tujuan**: Refactoring aplikasi menjadi struktur yang lebih terorganisir.

### Struktur Backend yang Diinginkan

```
src/
├── controllers/
│   └── mahasiswa.controller.ts
├── services/
│   └── mahasiswa.service.ts
├── repositories/
│   └── mahasiswa.repository.ts
├── models/
│   └── mahasiswa.model.ts
├── middleware/
│   ├── auth.middleware.ts
│   └── error.middleware.ts
├── routes/
│   └── api.ts
├── dtos/
│   └── mahasiswa.dto.ts
├── utils/
│   └── validation.ts
└── server.ts
```

---

## Referensi

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [SvelteKit Security](https://kit.svelte.dev/docs/security)