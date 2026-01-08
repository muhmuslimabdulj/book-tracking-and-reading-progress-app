# 📚 Book Tracking & Reading Progress App

## 1. Executive Summary

### 1.1 Product Vision

**Book Tracking Platform** adalah solusi ekosistem digital terintegrasi yang dirancang untuk memfasilitasi manajemen aktivitas literasi pengguna secara *seamless*. Platform ini menghubungkan pengalaman pengguna di berbagai touchpoint (Mobile & Web) dengan infrastruktur Backend yang scalable, bertujuan untuk meningkatkan *user engagement* melalui pencatatan progres membaca yang terstruktur dan analitik personal.

### 1.2 Development Objectives

- **Scalable Architecture**: Membangun arsitektur API-First yang robust untuk mendukung multi-platform client.
- **Cross-Platform Consistency**: Menjamin konsistensi data dan pengalaman pengguna (UX) antara aplikasi Mobile dan Web.
- **Maintainable Codebase**: Menerapkan standar industri dalam pemisahan tanggung jawab (Separation of Concerns) dan Clean Code.

### 1.3 Scope of Work

Dokumen ini mencakup spesifikasi teknis untuk modul-modul berikut:

- **Identity Access Management (IAM)**: Autentikasi dan otorisasi pengguna berbasis token.
- **Book Inventory System**: Manajemen siklus hidup data buku (CRUD) dan kategori.
- **Progress Tracking Engine**: Pencatatan aktivitas membaca yang granular dan historikal.
- **Analytical Dashboard**: Agregasi data statistik untuk insight pengguna dan administrator.
- **Media Management**: Infrastruktur upload dan hosting aset visual (Cover Buku).

---

## 2. Arsitektur Sistem & Pemisahan Tanggung Jawab

```text
+-------------------------------------------------------+
|                Admin / Internal Ecosystem             |
+-------------------------------------------------------+
|                                                       |
|   [ Browser ]                                         |
|        │                                              |
|        ▼                                              |
|   [ Laravel + Filament ]  ───────┐                    |
|        │                         │ (Direct Access)    |
|        ▼                         ▼                    |
|   [ Eloquent Model ]        [ Database ]              |
|                                  ▲                    |
+-------------------------------------------------------+
                                   │
                                   │ (Data Storage)
                                   │
+-------------------------------------------------------+
|                 User-Facing Ecosystem                 |
+-------------------------------------------------------+
|                                                       |
|   [ Mobile App ]             [ Web App ]              |
|     (Flutter)                 (React)                 |
|        │                         │                    |
|        └───────────┬─────────────┘                    |
|                    │ (REST API)                       |
|                    ▼                                  |
|           [ Laravel API Core ]                        |
|                    │                                  |
|                    ▼                                  |
|           [ Eloquent Model ]                          |
|                                                       |
+-------------------------------------------------------+
```

### 2.1 Prinsip Pemisahan

- **Admin / Internal App**: Dibangun langsung di Laravel menggunakan Filament, tanpa REST API
- **User-facing App**: Flutter & React **WAJIB** menggunakan REST API
- Backend bertindak sebagai:
  - API Provider (untuk user)
  - Admin System (internal)

---

## 3. Role & Platform

### 3.1 Role Pengguna

- **User**: Mengelola buku dan progres membaca milik sendiri

### 3.2 Platform

- Backend: Laravel 12 + Sanctum
- Mobile App: Flutter
- Web App: React (Vite)

---

## 4. Backend (Laravel + Filament)

Bagian ini menjelaskan **secara tegas** mana yang menjadi tanggung jawab **Backend**. Tim Mobile dan Web hanya perlu memahami kontrak API, namun tim Backend harus memahami internal logic berikut.

Backend memiliki **3 komponen TERPISAH namun saling terhubung**:

---

### 4.1 Backend Core (Laravel API)

> Digunakan oleh **Flutter & React** melalui REST API

**Tanggung Jawab Utama:**

- Menyediakan REST API
- Menyimpan & mengelola data
- Menjalankan business logic
- Menjaga keamanan & validasi

**Yang Dikerjakan Backend Core:**

- Database schema & migration
- Model Eloquent
- API Controller (`/api/*`)
- Request validation
- Service layer (logic)
- Authentication (Sanctum)
- Authorization (Policy)

**Yang TIDAK BOLEH dilakukan Backend Core:**

- Mengatur UI
- Mengatur layout frontend
- Menyimpan state UI

---

### 4.2 Backend API Responsibility (Kontrak untuk Frontend)

Backend API **HANYA BERTUGAS**:

- Menerima request
- Memvalidasi data
- Mengolah data
- Mengirim response JSON

Backend **TIDAK PEDULI**:

- UI Flutter atau React
- Tampilan mobile / web

---

### 4.3 Backend Admin / Internal App (Overview)

> Digunakan oleh **ADMIN / INTERNAL SAJA**

**Tujuan:**

- Monitoring
- Manajemen data
- Statistik global

**Karakteristik:**

- URL: `/admin`
- Auth berbasis role `admin`
- Langsung ke Eloquent
- **TIDAK menggunakan REST API**

*(Detail implementasi Filament ada di Bab 9)*

---

## 5. API CONTRACT (SINGLE SOURCE OF TRUTH – LOCKED)

> ⚠️ **SECTION INI DIKUNCI**\
> Digunakan oleh **Backend, Flutter, dan React** sebagai kontrak resmi.\
> **Tidak boleh diduplikasi di bagian lain dokumen.**

---

### 5.1 General Conventions

- Base URL: `/api`
- Auth: **Bearer Token (Laravel Sanctum)**
- Semua response JSON
- Timezone: ISO-8601 (`2026-01-01T10:00:00Z`) - Wajib UTC.

### Standard Pagination Response

Secara default, list data mengembalikan JSON Structure standar Laravel Pagination:

```json
{
  "data": [ ... ],
  "links": {
    "first": "...",
    "last": "...",
    "prev": null,
    "next": "..."
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 1,
    "path": "...",
    "per_page": 15,
    "to": 10,
    "total": 10
  }
}
```

### Standard Query Parameters (Search & Filter)

- `search`: Mencari secara *case-insensitive* pada `title` DAN `author`.
- `page`: Nomor halaman (default: 1).
- `per_page`: Jumlah item per halaman (default: 10, max: 100).

### Common Validation Rules

- **String**: Max 255 chars kecuali disebut lain.
- **Nullable**: Field opsional boleh dikirim `null` atau tidak dikirim sama sekali.
- **Password**: Min 8 chars.
- **Email**: Format valid email, unique di tabel users.

#### Status Buku (RESMI & TERKUNCI)

Nilai `status` yang **VALID dan digunakan oleh API**:

- `reading`
- `finished`
- `reading`
- `finished`
- `wishlist`

#### Standard Media/File URL

- Field yang berisi gambar (seperti `cover_url`) akan selalu mengembalikan **Full URL** (contoh: `https://api.domain.com/storage/covers/abc.jpg`).
- Jika tidak ada gambar, field bernilai `null`.
- Frontend **WAJIB** menampilkan *default placeholder image* (ikon buku) jika nilai field ini `null`.

> ⚠️ **Catatan penting:**
>
> - API **HANYA** mengenal nilai di atas
> - UI (Flutter / React) **BOLEH** menampilkan label berbeda
> - Backend & Frontend **WAJIB** menyimpan dan mengirim nilai API

#### Standard Error Response

```json
{
  "message": "Validation error",
  "errors": {
    "field": ["error message"]
  }
}
```

#### Standard HTTP Error Codes

| Code | Arti             | Keterangan                    |
| ---- | ---------------- | ----------------------------- |
| 401  | Unauthorized     | Token tidak ada / tidak valid |
| 403  | Forbidden        | Akses bukan milik user        |
| 404  | Not Found        | Resource tidak ditemukan      |
| 422  | Validation Error | Input tidak valid             |

---

### 5.2 Authentication Endpoints

#### POST /api/auth/login

**Request**

```json
{
  "email": "user@mail.com",
  "password": "password"
}
```

**Response 200**

```json
{
  "token": "access_token",
  "user": {
    "id": 1,
    "name": "User",
    "email": "user@mail.com"
  }
}
```

---

#### POST /api/auth/register

**Request**

```json
{
  "name": "User",
  "email": "user@mail.com",
  "password": "password",
  "password_confirmation": "password"
}
```

**Response 201**

```json
{
  "message": "User registered successfully"
}
```

---

#### POST /api/auth/logout

**Response 200**

```json
{
  "message": "Logged out"
}
```

---

### 5.3 User Profile Endpoints

#### GET /api/profile

**Response 200**

```json
{
  "id": 1,
  "name": "User",
  "email": "user@mail.com",
  "total_books": 10,
  "created_at": "2026-01-01T10:00:00Z"
}
```

---

### 5.4 Books Endpoints

#### GET /api/books

**Query Params**

- `status` (reading | finished | wishlist)
- `search`

**Response 200 (Paginated)**
```json
{
  "data": [
    {
      "id": 1,
      "title": "Atomic Habits",
      "author": "James Clear",
      "cover_url": "https://api.domain.com/storage/covers/abc.jpg",
      "status": "reading",
      "total_pages": 320,
      "current_page": 120,
      "progress_percent": 37,
      "category": {
        "id": 1,
        "name": "Self Improvement"
      }
    }
  ],
  "links": { ... },
  "meta": { ... }
}
```

---

#### GET /api/books/{id}

**Response 200**

```json
{
  "id": 1,
  "title": "Atomic Habits",
  "author": "James Clear",
  "cover_url": "https://api.domain.com/storage/covers/abc.jpg",
  "status": "reading",
  "total_pages": 320,
  "current_page": 120,
  "progress_percent": 37,
  "category": {
    "id": 1,
    "name": "Self Improvement"
  },
  "created_at": "2026-01-01T10:00:00Z"
}
```

---

#### POST /api/books

**Request**

```json
{
  "title": "Atomic Habits",
  "author": "James Clear",
  "cover_url": "https://api.domain.com/storage/covers/abc.jpg",
  "total_pages": 320,
  "category_id": 1
}
```

**Response 201**

```json
{
  "id": 1,
  "message": "Book created"
}
```

---

#### PUT /api/books/{id}

**Request**

```json
{
  "title": "Atomic Habits",
  "author": "James Clear",
  "cover_url": "https://api.domain.com/storage/covers/new.jpg",
  "status": "reading",
  "category_id": 1
}
```

---

#### DELETE /api/books/{id}

**Response 200**

```json
{
  "message": "Book deleted"
}
```

---

### 5.5 Reading Progress Endpoints

#### POST /api/books/{id}/progress

**Request**

```json
{
  "current_page": 120,
  "note": "Reached chapter 3"
}
```

**Response 200**

```json
{
  "current_page": 120,
  "progress_percent": 37
}
```

---

#### GET /api/books/{id}/progress-history

**Response 200**

```json
[
  {
    "page": 50,
    "note": "First session",
    "created_at": "2026-01-01T08:00:00Z"
  }
]
```

---

### 5.6 Categories Endpoints

#### GET /api/categories

```json
[
  { "id": 1, "name": "Self Improvement" }
]
```

---

#### POST /api/categories

```json
{ "name": "Technology" }
```

---

#### PUT /api/categories/{id}

```json
{ "name": "Business" }
```

---

#### DELETE /api/categories/{id}

```json
{ "message": "Category deleted" }
```

---

### 5.7 Dashboard Summary Endpoints

#### GET /api/dashboard/summary

```json
{
  "total_books": 10,
  "reading": 3,
  "finished": 5,
  "wishlist": 2,
  "pages_read_today": 45
}
```

> ℹ️ **Catatan Konsistensi:**
>
> - Field `finished` adalah istilah **resmi API**
> - Istilah naratif seperti `completed` **tidak digunakan di kontrak**
> - Semua frontend **WAJIB** mengikuti nama field API

---

### 5.8 Upload Media Endpoints

#### POST /api/upload-image

Endpoint serbaguna untuk upload gambar (cover buku, profil, dll) ke server.

**Request (`multipart/form-data`)**

- `image`: File (jpg, png, jpeg), max 2MB.

**Response 200**

```json
{
  "url": "https://api.domain.com/storage/uploads/unique-filename.jpg"
}
```

---

## 6. Frontend – Mobile App (Flutter)

Bagian ini menjelaskan **secara eksplisit setiap screen Flutter**, isi datanya, serta **endpoint API yang digunakan**. Tim Flutter **HANYA mengikuti kontrak ini**.

---

### 6.1 Flutter Responsibility

**Tanggung Jawab:**

- UI & UX mobile
- Navigasi screen
- State management
- Konsumsi REST API

**Flutter BOLEH:**

- Call API (`/api/*`)
- Menyimpan token
- Menampilkan data user

**Flutter TIDAK BOLEH:**

- Mengakses database
- Mengubah business logic
- Mengakses `/admin`

---

### 6.2 Flutter Screen Detail (Mobile App – WAJIB SINKRON DENGAN API)

### 1. Login Screen

**Tujuan:** Autentikasi user

**UI Components:**

- Input Email
- Input Password
- Button Login
- Link ke Register

**API:**

- `POST /api/auth/login`

**State:**

- authToken
- userProfile

**Post-Action Flow:**

- **Success (200):** Simpan token -> Redirect ke **User Dashboard**.
- **Error (401):** Tampilkan Snackbar/Alert "Invalid Credentials".

---

### 2. Register Screen

**UI Components:**

- Input Name
- Input Email
- Input Password
- Input Confirm Password
- Button Register

**API:**

- `POST /api/auth/register`

**Post-Action Flow:**

- **Success (201):** Tampilkan Success Dialog -> Redirect ke **Login Screen**.
- **Error (422):** Tampilkan pesan error di bawah input field masing-masing.

---

### 3. User Dashboard Screen

**Tujuan:** Ringkasan progres membaca user

**UI Components:**

- Total Books
- Reading / Finished / Not Started
- Pages Read Today

**API:**

- `GET /api/dashboard/summary`

---

### 4. Book List Screen

**Tujuan:** Menampilkan semua buku milik user

**UI Components:**

- List card buku
- Progress bar
- Filter status
- Search
- Floating button "Tambah Buku"

**API:**

- `GET /api/books`

---

### 5. Add / Edit Book Screen

**UI Components:**

- Input Title
- Input Author
- Upload/Select Cover Image
- Input Total Pages
- Select Category
- Select Status

**API:**

- `POST /api/books`
- `PUT /api/books/{id}`

**Post-Action Flow:**

- **Success:** Tampilkan Toast "Book Saved" -> Kembali ke **Book List** (refresh list).

---

### 6. Book Detail Screen

**UI Components:**

- Detail buku
- Progress bar
- Button "Update Progress" (Memicu Modal/Bottom Sheet)
- **Action Icon**: Edit Book (ke form Edit)
- Riwayat membaca

**API:**

- `GET /api/books/{id}`
- `GET /api/books/{id}/progress-history`

---

### 7. Update Progress (Modal / Bottom Sheet)

**UI Components:**

- Input Page (Number)
- Input Note (Text - optional)
- Button Submit

**API:**

- `POST /api/books/{id}/progress`

**Post-Action Flow:**

- **Success:** Tutup Modal -> Refresh data di **Book Detail Screen** (progress bar update).

---

### 8. Category Management Screen

**UI Components:**

- List kategori
- Tambah / Edit / Delete

**API:**

- `GET /api/categories`
- `POST /api/categories`
- `PUT /api/categories/{id}`
- `DELETE /api/categories/{id}`

---

### 9. Profile Screen

**UI Components:**

- Nama
- Email
- Total Buku
- Button Logout

**API:**

- `GET /api/profile`
- `POST /api/auth/logout`

---

## 7. Frontend – Web App (React)

Bagian ini menjelaskan **halaman React secara eksplisit**, termasuk data & API yang digunakan.

---

### 7.1 React Responsibility

**Tanggung Jawab:**

- UI web (desktop-first)
- State management
- Konsumsi REST API
- Visualisasi data

**React BOLEH:**

- Call API (`/api/*`)
- Menampilkan tabel & chart

**React TIDAK BOLEH:**

- Mengakses database
- Mengakses `/admin`
- Mengubah business logic

---

### 7.2 React Page Detail

### 1. Login & Register Page

**Komponen:**

- Form login / register

**API:**

- `POST /api/auth/login`
- `POST /api/auth/register`

**Post-Action Flow:**

- **Login Success:** Simpan Token -> Redirect **Dashboard**.
- **Register Success:** Redirect **Login** dengan flash message success.

---

### 2. User Dashboard Page

**Komponen:**

- Summary cards
- Chart progres membaca

**API:**

- `GET /api/dashboard/summary`

---

### 3. Book Management Page

**Komponen:**

- Tabel buku
- Filter status
- Search
- Button tambah/edit/hapus (buka Modal Form)

**API:**

- `GET /api/books`
- `POST /api/books`
- `PUT /api/books/{id}`
- `DELETE /api/books/{id}`

**Post-Action Flow:**

- **Add/Edit Success:** Tutup Modal -> Refresh Tabel.
- **Delete Success:** Show Confirm Dialog -> API Call -> Refresh Tabel.

---

### 4. Book Detail Page

**Komponen:**

- Detail buku
- Progress bar
- Timeline reading history
- **Action Button**: Edit Book (Open Edit Modal)

**API:**

- `GET /api/books/{id}`
- `GET /api/books/{id}/progress-history`

---

### 5. Update Progress Modal

**Komponen:**

- Form update halaman (muncul via tombol di Book Detail)

**API:**

- `POST /api/books/{id}/progress`

**Post-Action Flow:**

- **Success:** Tutup Modal -> Refresh halaman/komponen detail.

---

### 6. Category Management Page

**Komponen:**

- Tabel kategori

**API:**

- `GET /api/categories`
- `POST /api/categories`
- `PUT /api/categories/{id}`
- `DELETE /api/categories/{id}`

---

### 7. Profile Page

**Komponen:**

- Informasi user
- Logout

**API:**

- `GET /api/profile`
- `POST /api/auth/logout`

---

## 8. RESPONSIBILITY SUMMARY (RANGKUMAN)

| Bagian       | Teknologi | Akses DB | Akses API | Target User |
| ------------ | --------- | -------- | --------- | ----------- |
| Backend Core | Laravel   | ✅        | ❌         | Sistem      |
| Admin App    | Filament  | ✅        | ❌         | Admin       |
| Mobile App   | Flutter   | ❌        | ✅         | User        |
| Web App      | React     | ❌        | ✅         | User        |

---

**Rules Utama:**

- Frontend **TIDAK BOLEH** mengakses database langsung.
- Frontend **HANYA BOLEH** mengakses REST API.
- Admin Panel (Filament) **HANYA UNTUK** Internal/Admin, bukan untuk User biasa.

---

## 9. Filament Admin Dashboard (Internal Only)

> **Catatan:** Bagian ini **KHUSUS ADMIN / INTERNAL** dan **TIDAK TERKAIT REST API**. Filament **langsung menggunakan Eloquent Model & Database**.

---

### 9.1 Tujuan & Ruang Lingkup

Filament Admin Dashboard digunakan untuk:

- Monitoring kondisi sistem secara global
- Manajemen data lintas user
- Audit & kontrol data (tanpa mengganggu data user via API)

Filament **BUKAN**:

- Pengganti API
- Digunakan oleh user umum
- Digunakan oleh Flutter / React

---

### 9.2 Akses & Autentikasi

- URL: `/admin`
- Guard: `web`
- Model Auth: `User`
- Syarat akses:
  - Field `role = 'admin'`

> 🔒 **Aturan:**
>
> - User biasa **TIDAK BOLEH** mengakses `/admin`
> - Admin **TIDAK MENGGUNAKAN** token API (Sanctum)

---

### 9.3 Filament Resources (Detail Teknis)

#### 9.3.1 UserResource

**Tujuan:** Monitoring & manajemen akun user

**Fields (Table & Form):**

- `id` (read-only)
- `name`
- `email`
- `role` (user | admin)
- `created_at` (read-only)

**Table Capabilities:**

- **Searchable:** `name`, `email`
- **Filters:** `role`

**Aksi yang DIIZINKAN:**

- Lihat daftar user
- Edit `name`, `email`, `role`

**Aksi yang DILARANG:**

- Mengubah password user
- Menghapus user (opsional, jika diizinkan harus soft delete)

---

#### 9.3.2 BookResource

**Tujuan:** Monitoring data buku lintas user

**Fields:**

- `id` (read-only)
- `title`
- `author`
- `status`
- `total_pages`
- `current_page`
- `user.name` (read-only)
- `category.name`
- `created_at`

**Table Capabilities:**

- **Searchable:** `title`, `author`, `user.name`
- **Filters:** `status`, `category`

**Aksi yang DIIZINKAN:**

- Lihat buku
- Edit metadata buku (title, author, category)

**Aksi yang DILARANG:**

- Mengubah `current_page`
- Mengubah progress user

---

#### 9.3.3 CategoryResource

**Tujuan:** Manajemen kategori global

**Fields:**

- `id` (read-only)
- `name`

**Aksi:**

- Create
- Update
- Delete

> ⚠️ **Catatan:** Penghapusan kategori **TIDAK BOLEH** otomatis menghapus buku

---

### 9.4 Dashboard Widgets (Detail)

Widget utama yang **WAJIB ADA**:

1. **Total Users**

   - Query: `User::count()`

2. **Total Books**

   - Query: `Book::count()`

3. **Books by Status**

   - Reading / Finished / Wishlist
   - Query: `Book::groupBy('status')`

4. **Books Created This Month** (opsional)

   - Filter `created_at`

---

### 9.5 Batasan & Prinsip Keamanan

- Filament **TIDAK MEMANGGIL API**
- Filament **TIDAK MENGUBAH DATA PROGRESS USER**
- Semua action admin harus:
  - Bisa diaudit
  - Tidak memanipulasi data user secara diam-diam

---

## 10. Technical Implementation Guide & Best Practices

> 💡 **Bagian ini berisi panduan teknis implementasi** untuk menghindari masalah umum (Common Pitfalls) saat integrasi.

### 10.1 Authentication & Token Lifecycle

- **Token Type:** Long-lived Access Token (1 tahun).
- **Refresh Token:** Tidak digunakan (untuk kesederhanaan).
- **Token Expired:**
  - Jika API merespon `401 Unauthorized`, Frontend **WAJIB** langsung logout user di sisi client dan redirect ke Login Screen.
  - Jangan mencoba me-request ulang tanpa login kembali.

### 10.2 Frontend Performance (Debounce)

Fitur **Search** (Flutter & React) berpotensi membebani server jika request dikirim per ketukan huruf.

- **Aturan:** Frontend **WAJIB** menerapkan `debounce` minimal **500ms** pada input Search.
- Artinya: Request API hanya dikirim jika user berhenti mengetik selama 0.5 detik.

### 10.3 Handling Empty States (Search/List)

- Jika endpoint List (misal `GET /api/books?search=xyz`) tidak menemukan data:
  - Backend **TETAP** mengembalikan `200 OK`.
  - Body: `data: []` (array kosong).
  - **JANGAN** mengembalikan `404 Not Found`.
- `404 Not Found` hanya digunakan jika endpoint salah atau ID spesifik (misal `GET /api/books/999`) tidak ditemukan.

### 10.4 Format Validasi Error (Detail)

Backend akan mengembalikan error validasi dengan key field yang presisi. Frontend harus mapping key tersebut ke input form terkait.

**Contoh Response 422 (Register):**

```json
{
  "message": "The email has already been taken.",
  "errors": {
    "email": ["The email has already been taken."],
    "password": ["The password confirmation does not match."]
  }
}
```

- **Flutter/React:** Cek key `email` untuk menampilkan pesan merah di bawah input Email.
- **Flutter/React:** Cek key `password` untuk menampilkan pesan merah di bawah input Password.

### 10.5 Manajemen Gambar (Cover Buku)

- Fitur upload menggunakan strategi **Separate Endpoint**.
- Frontend upload gambar dulu ke `/api/upload-image`, dapat URL, baru kirim URL-nya saat Create/Update Book.
- Gunakan library seperti `image_picker` (Flutter) atau Input File standard (React).
