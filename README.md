# Jarchive Infrastructure

Repository ini berisi konfigurasi infrastruktur (Infrastructure as Code) untuk melakukan deployment aplikasi **Jarchive**.

Aplikasi ini terdiri dari:
- **Frontend:** React + Vite
- **Backend:** Node.js + Express
- **Database:** MongoDB

## Prasyarat

Pastikan server Anda sudah terinstall:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Git

## Cara Install & Menjalankan (Deployment)

1. **Clone Repository (dengan Submodules)**
   Penting untuk menggunakan `--recursive` agar kode backend dan frontend ikut terdownload.
   ```bash
   git clone https://GHP_KEY@github.com/siJarchive/jarchive-infrastructure.git
   cd jarchive-infrastructure

2. **Konfigurasi Environment (.env)**
   Salin file `.env.example` menjadi `.env` di folder `jarchive-infrastructure`. Berkat pembaruan terbaru, seluruh konfigurasi port, kredensial aplikasi, dan keamanan database dipusatkan di file ini. Anda **wajib** menyesuaikan variabel berikut:
   
   * **Jaringan & Port:**
     * `BACKEND_PORT`: Port yang akan terekspos ke luar untuk backend.
     * `FRONTEND_PORT`: Port web untuk mengakses frontend.
     * `VITE_API_URL`: Isi dengan IP server/komputer beserta port backend agar frontend dapat berkomunikasi dengan API.
   * **Keamanan Aplikasi:**
     * `JWT_SECRET`: Ganti dengan string acak untuk keamanan token login.
     * `ADMIN_USER` & `ADMIN_PASS`: Kredensial login untuk Guru.
     * `SISWA_USER` & `SISWA_PASS`: Kredensial login untuk Siswa.
   * **Keamanan Database (MongoDB):**
     * `MONGO_ROOT_USER` & `MONGO_ROOT_PASS`: Kredensial autentikasi untuk mengunci database MongoDB.
     * `MONGO_URI`: Format koneksi backend ke database (wajib memuat username & password root di atas, contoh: `mongodb://root:password123@mongo:27017/jarchive_db?authSource=admin`).

3. **Jalankan Container**
   Setelah `.env` disesuaikan, jalankan perintah berikut di terminal:
   ```bash
   docker compose up -d --build