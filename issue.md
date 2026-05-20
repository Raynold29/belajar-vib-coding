# Planning: Inisiasi Project Bun dengan ElysiaJS, Drizzle, dan MySQL

Dokumen ini berisi panduan langkah-demi-langkah (planning) untuk membuat proyek backend baru menggunakan runtime **Bun**, framework **ElysiaJS**, ORM **Drizzle**, dan database **MySQL**. Panduan ini dirancang agar mudah dipahami dan diimplementasikan oleh programmer atau model AI lainnya.

---

## 📂 Struktur Proyek yang Direkomendasikan

```text
├── src/
│   ├── db/
│   │   ├── index.ts      # Koneksi & inisialisasi Drizzle
│   │   └── schema.ts     # Definisi schema database (tabel)
│   └── index.ts          # Entrypoint server ElysiaJS
├── .env                  # Environment variables (Database URL, dll.)
├── .env.example          # Template environment variables
├── drizzle.config.ts     # Konfigurasi Drizzle Kit
├── package.json
└── tsconfig.json
```

---

## 🛠️ Langkah-Langkah Implementasi

### Langkah 1: Inisialisasi Proyek Bun
1. Masuk ke direktori proyek dan jalankan perintah inisialisasi:
   ```bash
   bun init
   ```
   *Pilih entry point default yaitu `src/index.ts` jika ditanya.*

### Langkah 2: Instalasi Dependencies
Instal semua framework, ORM, dan driver database yang dibutuhkan:

1. **ElysiaJS**:
   ```bash
   bun add elysia
   ```
2. **Drizzle ORM & MySQL Driver**:
   ```bash
   bun add drizzle-orm mysql2
   ```
3. **Drizzle Kit & Type Definitions** (sebagai devDependencies):
   ```bash
   bun add -d drizzle-kit @types/node
   ```

### Langkah 3: Konfigurasi Environment Variables (`.env`)
1. Buat file `.env` di root directory dengan konfigurasi MySQL:
   ```env
   DATABASE_URL=mysql://root:password@localhost:3306/nama_database
   PORT=3000
   ```
2. Buat juga `.env.example` sebagai dokumentasi template.

### Langkah 4: Konfigurasi Drizzle ORM
1. Buat file `drizzle.config.ts` di root directory untuk konfigurasi migrasi database:
   ```typescript
   import { defineConfig } from "drizzle-kit";

   export default defineConfig({
     dialect: "mysql",
     schema: "./src/db/schema.ts",
     out: "./drizzle",
     dbCredentials: {
       url: process.env.DATABASE_URL || "",
     },
   });
   ```

2. Buat file schema database di `src/db/schema.ts` (contoh schema tabel `users` sederhana):
   ```typescript
   import { mysqlTable, serial, varchar, timestamp } from "drizzle-orm/mysql-core";

   export const users = mysqlTable("users", {
     id: serial("id").primaryKey(),
     name: varchar("name", { length: 255 }).notNull(),
     email: varchar("email", { length: 255 }).notNull().unique(),
     createdAt: timestamp("created_at").defaultNow(),
   });
   ```

3. Buat file koneksi database di `src/db/index.ts`:
   ```typescript
   import { drizzle } from "drizzle-orm/mysql2";
   import mysql from "mysql2/promise";
   import * as schema from "./schema";

   const connection = await mysql.createConnection(process.env.DATABASE_URL || "");
   export const db = drizzle(connection, { schema, mode: "default" });
   ```

### Langkah 5: Membuat Server ElysiaJS (`src/index.ts`)
Buat entrypoint server di `src/index.ts` untuk melayani request HTTP dan integrasikan dengan Drizzle ORM:

1. Setup router dasar dan tambahkan endpoint contoh untuk mengambil data `users`:
   ```typescript
   import { Elysia } from "elysia";
   import { db } from "./db";
   import { users } from "./db/schema";

   const app = new Elysia()
     .get("/", () => ({ message: "Hello Elysia with Bun, Drizzle & MySQL!" }))
     .get("/users", async () => {
       try {
         const allUsers = await db.select().from(users);
         return { success: true, data: allUsers };
       } catch (error: any) {
         return { success: false, error: error.message };
       }
     })
     .listen(process.env.PORT || 3000);

   console.log(`🦊 Elysia is running at ${app.server?.hostname}:${app.server?.port}`);
   ```

### Langkah 6: Tambahkan Scripts di `package.json`
Modifikasi file `package.json` agar mempermudah proses menjalankan server dan migrasi database:
```json
"scripts": {
  "dev": "bun --watch src/index.ts",
  "db:generate": "bunx drizzle-kit generate",
  "db:migrate": "bunx drizzle-kit migrate"
}
```

---

## 🚦 Cara Menjalankan & Verifikasi

1. **Jalankan Migrasi Database**:
   - Pertama, pastikan database MySQL Anda sudah berjalan dan database target sudah dibuat.
   - Buat file migrasi dengan perintah:
     ```bash
     bun run db:generate
     ```
   - Terapkan migrasi tersebut ke database MySQL:
     ```bash
     bun run db:migrate
     ```

2. **Jalankan Aplikasi Backend**:
   - Jalankan server dalam mode development:
     ```bash
     bun run dev
     ```

3. **Verifikasi API**:
   - Buka browser atau HTTP Client (Postman/Thunder Client/Curl) dan akses:
     - `GET http://localhost:3000/` (Harus mengembalikan pesan hello world)
     - `GET http://localhost:3000/users` (Harus mengembalikan list user atau array kosong `[]`)
