# Pertemuan 11: Form & Validation

## Navigasi

[🏠 Beranda](../README.md) | [🔧 Prerequest](../prerequest/README.md) | ... | [Pertemuan 10](pertemuan-10/README.md) | **Pertemuan 11** | [Pertemuan 12](pertemuan-12/README.md) | ... | [Pertemuan 16](pertemuan-16/README.md)

---

## Materi

### 1. Form Handling di Frontend

```svelte
<script>
  let formData = {
    nama: "",
    email: "",
    password: ""
  };
  
  let errors: Record<string, string> = {};
  
  function validateForm() {
    errors = {};
    
    if (!formData.nama.trim()) {
      errors.nama = "Nama wajib diisi";
    }
    
    if (!formData.email.includes("@")) {
      errors.email = "Email tidak valid";
    }
    
    if (formData.password.length < 8) {
      errors.password = "Password minimal 8 karakter";
    }
    
    return Object.keys(errors).length === 0;
  }
  
  async function handleSubmit(event: Event) {
    event.preventDefault();
    
    if (!validateForm()) return;
    
    try {
      const response = await fetch("/api/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(formData)
      });
      // handle success
    } catch (error) {
      // handle error
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div>
    <label for="nama">Nama</label>
    <input
      id="nama"
      bind:value={formData.nama}
      class:bg-red-300={errors.nama}
    />
    {#if errors.nama}
      <span class="error">{errors.nama}</span>
    {/if}
  </div>
  
  <button type="submit">Submit</button>
</form>
```

### 2. Input Binding

```svelte
<script>
  let nama = "";
  let email = "";
  let active = true;
  let count = 0;
</script>

<input bind:value={nama} placeholder="Nama lengkap" />
<input bind:value={email} type="email" />
<label>
  <input type="checkbox" bind:checked={active} />
  Aktif
</label>
<input type="number" bind:value={count} min="0" />
```

### 3. Client-Side Validation

```typescript
const validationRules = {
  nama: {
    required: true,
    minLength: 3,
    maxLength: 100
  },
  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  },
  password: {
    required: true,
    minLength: 8,
    pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$/
  }
};

function validate(value: string, rule: any): string | null {
  if (rule.required && !value.trim()) {
    return "Field ini wajib diisi";
  }
  
  if (rule.minLength && value.length < rule.minLength) {
    return `Minimal ${rule.minLength} karakter`;
  }
  
  if (rule.pattern && !rule.pattern.test(value)) {
    return "Format tidak valid";
  }
  
  return null;
}
```

### 4. Server-Side Validation (Elysia)

```typescript
import { Elysia, t } from "elysia";

const ValidationError = t.Object({
  field: t.String(),
  message: t.String()
});

const createUserValidator = t.Object({
  nama: t.String().min(3).max(100),
  email: t.Email(),
  password: t.String().min(8).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$/, "Password harus mengandung huruf besar, huruf kecil, dan angka"),
  usia: t.Optional(t.Number().min(17).max(100))
});

const app = new Elysia()
  .post("/api/users", ({ body, error }) => {
    try {
      // Additional custom validation
      if (body.nama.toLowerCase().includes("admin")) {
        throw new Error("Nama tidak boleh mengandung 'admin'");
      }
      
      return createUser(body);
    } catch (e) {
      return error(400, { error: e.message });
    }
  }, { body: createUserValidator });
```

### 5. Validation Error Handling

```typescript
// API Response Error
interface ValidationError {
  field: string;
  message: string;
}

interface ErrorResponse {
  success: false;
  message: string;
  errors: ValidationError[];
}

// Frontend error handling
async function handleSubmit() {
  try {
    const response = await fetch("/api/users", {
      method: "POST",
      body: JSON.stringify(formData)
    });
    
    const result = await response.json();
    
    if (!response.ok) {
      if (result.errors) {
        // Field-specific errors
        errors = {};
        result.errors.forEach((e: ValidationError) => {
          errors[e.field] = e.message;
        });
      } else {
        // General error
        formError = result.message;
      }
      return;
    }
    
    // Success
    resetForm();
  } catch (error) {
    formError = "Network error";
  }
}
```

### 6. API Error & Error Handling

```typescript
// Error middleware di Elysia
const app = new Elysia()
  .onError(({ code, error, request }) => {
    console.error("Error:", error);
    
    // Validation errors
    if (code === "VALIDATION") {
      return {
        success: false,
        message: "Validation failed",
        errors: error.all.map((e: any) => ({
          field: e.path.join("."),
          message: e.message
        }))
      };
    }
    
    // Custom errors
    if (error instanceof Error) {
      const status = error.message.includes("not found") ? 404 : 400;
      return { success: false, message: error.message };
    }
    
    // Default error
    return { success: false, message: "Internal server error" };
  });
```

### 7. User Feedback

```svelte
<script>
  let message = "";
  let messageType = "info"; // info, success, error, warning
  
  function showMessage(text: string, type: "info" | "success" | "error" | "warning") {
    message = text;
    messageType = type;
    
    setTimeout(() => {
      message = "";
    }, 5000);
  }
  
  async function handleSubmit() {
    showMessage("Mengirim data...", "info");
    
    try {
      // API call
      showMessage("Data berhasil disimpan", "success");
    } catch (error) {
      showMessage("Gagal menyimpan data", "error");
    }
  }
</script>

{#if message}
  <div class="notification {messageType}">
    {message}
  </div>
{/if}

<form on:submit|preventDefault={handleSubmit}>
  <!-- form fields -->
</form>
```

## Praktikum

**Tujuan**: Membuat form CRUD mahasiswa dengan validasi frontend dan backend.

### Struktur

```
backend/
├── src/
│   ├── validators/
│   │   └── mahasiswa-validator.ts
│   ├── routes/
│   │   └── mahasiswa.ts
│   └── utils/
│       └── response.ts

frontend/
├── src/
│   ├── lib/
│   │   └── validators/
│   │       └── client-validation.ts
│   └── routes/
│       └── mahasiswa/
│           └── +page.svelte
└── svelte.config.js
```

### Contoh Implementasi Backend

```typescript
// backend/src/validators/mahasiswa-validator.ts
export const mahasiswaSchema = t.Object({
  nama: t.String({ 
    min: 2, 
    max: 100,
    minLength: "Nama minimal 2 karakter",
    maxLength: "Nama maksimal 100 karakter"
  }),
  nim: t.String({ 
    pattern: /^\d{8}$/,
    patternMessage: "NIM harus 8 digit angka"
  }),
  email: t.Email("Email tidak valid"),
  nilai: t.Optional(t.Number({
    min: 0,
    max: 100,
    minMessage: "Nilai minimal 0",
    maxMessage: "Nilai maksimal 100"
  }))
});
```

### Contoh Implementasi Frontend

```svelte
<!-- frontend/src/routes/mahasiswa/+page.svelte -->
<script>
  import { onMount } from "svelte";
  
  let form = {
    nama: "",
    nim: "",
    email: "",
    nilai: null
  };
  
  let errors = {};
  let loading = false;
  let submitting = false;
  
  function validate() {
    errors = {};
    
    if (!form.nama.trim()) errors.nama = "Nama wajib diisi";
    if (!form.nim || !/^\\d{8}$/.test(form.nim)) {
      errors.nim = "NIM harus 8 digit angka";
    }
    if (!form.email.includes("@")) {
      errors.email = "Email tidak valid";
    }
    if (form.nilai !== null && (form.nilai < 0 || form.nilai > 100)) {
      errors.nilai = "Nilai harus antara 0-100";
    }
    
    return Object.keys(errors).length === 0;
  }
  
  async function handleSubmit(e) {
    e.preventDefault();
    
    if (!validate()) return;
    
    submitting = true;
    try {
      const response = await fetch("/api/mahasiswa", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(form)
      });
      
      if (!response.ok) {
        const error = await response.json();
        if (error.errors) {
          errors = {};
          error.errors.forEach(e => errors[e.field] = e.message);
        }
        return;
      }
      
      // Reset form
      form = { nama: "", nim: "", email: "", nilai: null };
    } finally {
      submitting = false;
    }
  }
</script>

<form on:submit|preventDefault={handleSubmit} class:error={Object.keys(errors).length > 0}>
  <div class="field">
    <label>Nama</label>
    <input bind:value={form.nama} />
    {#if errors.nama}<span class="error">{errors.nama}</span>{/if}
  </div>
  
  <button type="submit" disabled={submitting}>
    {submitting ? "Mengirim..." : "Simpan"}
  </button>
</form>
```

## Referensi

- [Elysia Validation](https://elysiajs.com/plugin/validation.html)
- [Svelte Forms](https://svelte.dev/tutorial/forms)