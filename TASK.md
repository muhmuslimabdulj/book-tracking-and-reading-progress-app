# 📋 Project Task Checklist

Checklist ini disinkronisasi dengan spesifikasi teknis di `README.md`. Gunakan dokumen ini untuk melacak progres pengerjaan paralel tim Backend, Mobile, dan Web.

## 🟢 Phase 1: Foundation Setup (Parallel Start)

Fase ini bisa dikerjakan bersamaan oleh semua tim.

### Backend Team (Laravel)
- [ ] **Init Project**: Setup Laravel 12 + Repo.
- [ ] **Config**: Setup `.env`, Database Connection, Timezone UTC.
- [ ] **Code Style**: Install `Laravel Pint` untuk auto-fix code style.
- [ ] **Auth Setup**: Install & Config `Laravel Sanctum` (API Token).
- [ ] **Filament**: Install FilamentPHP untuk Admin Panel.
- [ ] **Modeling**:
    - [ ] Migration & Model `User` (tambah field `role`).
    - [ ] Migration & Model `Category`.
    - [ ] Migration & Model `Book` (tambah field `cover_url` nullable).
    - [ ] **Storage Link**: Run `php artisan storage:link`.

### Mobile Team (Flutter)
- [ ] **Init Project**: `flutter create`, setup repo.
- [ ] **Structure**: Setup folder structure (Features-first architecture).
- [ ] **Dependencies**: Install `dio` (HTTP), `flutter_riverpod` (State), `go_router` (Routing), `shared_preferences` (Token).
- [ ] **Base Config**: Setup Base URL, Interceptor (untuk Auto-logout 401).

### Web Team (React)
- [ ] **Init Project**: `npm create vite@latest`, setup repo.
- [ ] **Routing**: Setup `react-router-dom` (v6/v7).
- [ ] **Styling**: Setup TailwindCSS / UI Library.
- [ ] **State & Fetch**: Setup `TanStack Query` (Server State) & Axios Instance.

---

## 🟡 Phase 2: Authentication (First Integration)

### Backend Team
- [ ] **API Login**: Implement `POST /api/auth/login` (return token).
- [ ] **API Register**: Implement `POST /api/auth/register` (validate & create user).
- [ ] **API Logout**: Implement `POST /api/auth/logout` (revoke token).
- [ ] **API User Profile**: Implement `GET /api/profile`.

### Mobile Team (Flutter)
- [ ] **UI Login**: Screen Login + Validasi Input.
- [ ] **UI Register**: Screen Register + Validasi Input.
- [ ] **Logic Auth**: Integrasi API Login/Register.
- [ ] **Session**: Simpan token di storage & Handle state logged-in/logged-out.

### Web Team (React)
- [ ] **Page Login**: Form Login + React Query Mutation.
- [ ] **Page Register**: Form Register + React Query Mutation.
- [ ] **Auth Guard**: Protect Route (Redirect ke Login jika belum auth).

---

## 🟠 Phase 3: Core Features (Books & Progress)

### Backend Team
- [ ] **CRUD Category**: Implement API & Filament Resource.
- [ ] **CRUD Books**:
    - [ ] `GET /api/books` (Pagination & Search).
    - [ ] `POST /api/books`.
    - [ ] `GET /api/books/{id}`.
    - [ ] `PUT /api/books/{id}`.
    - [ ] `GET /api/books/{id}`.
    - [ ] `PUT /api/books/{id}`.
    - [ ] `DELETE /api/books/{id}`.
- [ ] **Upload Features**:
    - [ ] `POST /api/upload-image` (Handle file upload to `/storage/public/uploads`).
- [ ] **Reading Progress**:
    - [ ] `POST /api/books/{id}/progress` (Logic update `current_page` & `percent`).
    - [ ] `GET /api/books/{id}/progress-history`.

### Mobile Team (Flutter)
- [ ] **Dashboard**: Tampilkan SummaryCard (Mock dulu sebelum API Dashboard jadi).
- [ ] **Book List**: Tampilkan List dengan Pagination (Infinite Scroll).
- [ ] **Search**: Implement Debounce Search.
- [ ] **Add/Edit Book**: Form Screen + Dropdown Category + **Image Picker** (Cover).
- [ ] **Book Detail**: Tampilkan info buku + History List.
- [ ] **Update Progress**: Implement Bottom Sheet Input Page.

### Web Team (React)
- [ ] **Dashboard Page**: Layout utama.
- [ ] **Book Management**: Table View dengan Pagination & Filter.
- [ ] **Book Form**: Modal/Page untuk Add & Edit Buku + **File Input** (Cover).
- [ ] **Detail Page**: Layout detail + Chart/Timeline History.
- [ ] **Update Action**: Implement Modal Update Progress.

---

## 🔵 Phase 4: Dashboard & Polishing

### Backend Team
- [ ] **API Dashboard**: `GET /api/dashboard/summary` (Hitung statistik).
- [ ] **Admin Panel**: Finalisasi Dashbord Filament (Internal stats).

### Mobile Team (Flutter)
- [ ] **Integrasi Dashboard**: Tarik data real ke Dashboard Screen.
- [ ] **Profile Screen**: Tampilkan data user & Tombol Logout.
- [ ] **Polish**: Loading State, Empty State, Error Snackbar.

### Web Team (React)
- [ ] **Integrasi Dashboard**: Visualisasi angka statistik.
- [ ] **Profile Page**: Info user & Logout.
- [ ] **Polish**: Loading Spinner, Toast Notifications, Responsive Check.

---

## 🏁 Phase 5: Testing & Deployment

- [ ] **Backend**: Test Postman / Automated Tests.
- [ ] **Full Flow Test**: Coba register user baru -> Login Mobile -> Tambah Buku -> Update di Web -> Cek di Admin Panel.
- [ ] **Deployment**: Upload Backend (VPS/Shared), Build APK, Build React App.
