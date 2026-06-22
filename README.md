<div align="center">
  <h1>Jarchive Infrastructure</h1>
  <p>Konfigurasi Infrastruktur sebagai Kode (IaC) berbasis Docker untuk deployment ekosistem platform Jarchive.</p>
  <p>
    <a href="https://github.com/siJarchive/jarchive-frontend">Frontend</a> |
    <a href="https://github.com/siJarchive/jarchive-backend">Backend</a>
  </p>
</div>

<p align="center">
  <img src="https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white" alt="Docker">
  <img src="https://img.shields.io/badge/Docker_Compose-2496ED?logo=docker&logoColor=white" alt="Docker Compose">
  <img src="https://img.shields.io/badge/Platform-Linux-lightgrey?logo=linux" alt="Platform">
  <img src="https://img.shields.io/badge/License-MIT-green" alt="License">
</p>

---

Repository ini berisi definisi orkestrasi Docker Compose untuk menjalankan seluruh ekosistem Jarchive dalam satu host. Repository tidak memuat source code aplikasi; frontend dan backend tetap berada pada repository masing-masing dan di-clone secara manual ke dalam direktori kerja repository ini sebelum build image dijalankan.

Stack/komponen yang diorkestrasi:
- **Frontend:** React + Vite, disajikan oleh Nginx
- **Backend:** Node.js + Express
- **Database:** MongoDB (image `mongo:latest`)

## Daftar Isi

- [Fitur](#fitur)
- [Konsep dan Arsitektur](#konsep-dan-arsitektur)
- [Struktur Repository](#struktur-repository)
- [Prasyarat](#prasyarat)
- [Instalasi](#instalasi)
- [Konfigurasi Environment](#konfigurasi-environment)
- [Deployment](#deployment)
- [Manajemen dan Operasional](#manajemen-dan-operasional)
- [Troubleshooting](#troubleshooting)
- [Keamanan](#keamanan)
- [Uninstal](#uninstal)
- [Lisensi](#lisensi)

---

## Fitur

| Fitur | Deskripsi |
| --- | --- |
| Multi-container Deployment | Mengorkestrasi Frontend, Backend, dan Database MongoDB secara terpusat. |
| Environment Isolation | Memisahkan konfigurasi jaringan, kredensial, dan port eksternal melalui variabel lingkungan. |
| Data Persistence | Mengamankan data MongoDB melalui Docker named volume; file unggahan (assets) disimpan melalui bind mount ke direktori host (lihat catatan pada bagian Uninstal). |

---

## Konsep dan Arsitektur

```text
+-------------------------------------------------------------------+
|                         Host Environment                          |
|                                                                   |
|   +-----------------------+           +-----------------------+   |
|   |                       |           |                       |   |
|   |  jarchive-frontend    | <--HTTP-- |  jarchive-backend     |   |
|   |  (React/Nginx)        |           |  (Node.js/Express)    |   |
|   |                       |           |                       |   |
|   |  Port Eksternal:      |           |  Port Eksternal:      |   |
|   |  [FRONTEND_PORT]:80   |           |  [BACKEND_PORT]:5000  |   |
|   +-----------------------+           +-----------------------+   |
|                                                   |               |
|                                                  TCP              |
|                                                   |               |
|                                       +-----------------------+   |
|                                       |                       |   |
|                                       |  jarchive-mongo       |   |
|                                       |  (MongoDB)            |   |
|                                       |                       |   |
|                                       |  Port Eksternal:      |   |
|                                       |  [MONGO_PORT]:27017   |   |
|                                       +-----------------------+   |
+-------------------------------------------------------------------+
```

Catatan arsitektur (terverifikasi dari `docker-compose.yml`):
- Nama container service database adalah `jarchive-mongo` (bukan `jarchive-mongodb`).
- Port internal masing-masing service bersifat fixed (backend 5000, frontend 80, mongo 27017); hanya port host yang dapat diatur lewat `.env`.
- Hanya `mongo-data` yang merupakan Docker named volume. Folder upload backend dipetakan melalui bind mount `./jarchive-backend/uploads:/app/uploads`, sehingga secara teknis tidak terhapus oleh `docker compose down -v`.
- Nilai `VITE_API_URL` disuntikkan sebagai build argument saat image frontend dibangun, bukan dibaca saat runtime container.

---

## Struktur Repository

```
jarchive-infrastructure/
├── docker-compose.yml      Definisi 3 service: backend, frontend, mongo.
├── .env.example            Template variabel lingkungan.
├── .gitignore
├── LICENSE
├── package-lock.json       Artefak tersisa di repository, tidak memiliki package.json terkait.
├── jarchive-frontend/      Hasil clone manual repositori frontend (lihat Instalasi, bukan Git submodule).
├── jarchive-backend/       Hasil clone manual repositori backend (lihat Instalasi, bukan Git submodule).
└── README.md
```

---

## Prasyarat

| Komponen | Spesifikasi / Versi | Keterangan |
| --- | --- | --- |
| OS | Linux / macOS / Windows (WSL2) | Diuji secara optimal pada lingkungan Unix |
| Docker | Engine 20.10+ | Diperlukan untuk runtime container |
| Docker Compose | V2+ | Diperlukan untuk orkestrasi arsitektur |
| Git | Versi CLI Terbaru | Diperlukan untuk kloning source code frontend dan backend |

---

## Instalasi

1. Kloning repositori infrastruktur beserta repositori frontend dan backend ke dalam direktori yang sama:

```bash
git clone https://github.com/siJarchive/jarchive-infrastructure.git
cd jarchive-infrastructure

git clone https://github.com/siJarchive/jarchive-frontend.git
git clone https://github.com/siJarchive/jarchive-backend.git
```

Jika repositori bersifat private, otentikasi menggunakan token GHP:

```bash
git clone https://[GHP_TOKEN]@github.com/siJarchive/jarchive-frontend.git
```

---

## Konfigurasi Environment

Salin template yang telah disediakan:

```bash
cp .env.example .env
nano .env
```

| Variabel Lingkungan | Wajib | Default (`.env.example`) | Deskripsi |
| --- | --- | --- | --- |
| `BACKEND_PORT` | Ya | tidak ada | Port host yang dipetakan ke layanan API backend. |
| `FRONTEND_PORT` | Ya | tidak ada | Port host yang dipetakan ke layanan web antarmuka. |
| `VITE_API_URL` | Ya | tidak ada | Endpoint absolut API backend, contoh `http://192.168.1.10:5000`. Dibakukan saat build image frontend. |
| `JWT_SECRET` | Ya | tidak ada | Kunci tanda tangan JWT untuk autentikasi backend. |
| `ADMIN_USER` | Ya | `guru` | Identitas login peran Guru. Sudah terisi nilai contoh pada `.env.example`, disarankan diganti untuk produksi. |
| `ADMIN_PASS` | Ya | `admin123` | Kredensial login peran Guru. Disarankan diganti untuk produksi. |
| `SISWA_USER` | Ya | `siswa` | Identitas login peran Siswa. Disarankan diganti untuk produksi. |
| `SISWA_PASS` | Ya | `jarchive` | Kredensial login peran Siswa. Disarankan diganti untuk produksi. |
| `MONGO_PORT` | Ya | tidak ada | Port host yang dipetakan ke layanan database MongoDB. |
| `MONGO_ROOT_USER` | Ya | tidak ada | Username administrator MongoDB. |
| `MONGO_ROOT_PASS` | Ya | tidak ada | Password administrator MongoDB. |
| `MONGO_URI` | Ya | tidak ada | String koneksi backend ke MongoDB. Format pada `.env.example`: `mongodb://[MONGO_ROOT_USER]:[MONGO_ROOT_PASS]@mongo:27017/jarchive_db?authSource=admin`. |

---

## Deployment

```bash
docker compose up -d --build
```

Flag `--build` diperlukan pada eksekusi pertama (dan setiap kali source code frontend/backend berubah) karena image dibangun langsung dari Dockerfile pada masing-masing folder `jarchive-frontend/` dan `jarchive-backend/`, bukan dari image yang sudah dipublikasikan ke registry.

---

## Manajemen dan Operasional

Menghentikan seluruh container terkait tanpa merusak persistensi data:

```bash
docker compose stop
```

Memantau log secara realtime untuk seluruh container:

```bash
docker compose logs -f
```

Mengecek status dan pemetaan port layanan yang aktif:

```bash
docker compose ps
```

---

## Troubleshooting

**Frontend tetap mengarah ke `VITE_API_URL` lama setelah `.env` diubah**

Nilai `VITE_API_URL` dibakukan ke dalam bundle statis saat image frontend dibangun (build argument pada Vite). Mengubah `.env` saja tidak cukup; image harus dibangun ulang:

```bash
docker compose up -d --build frontend
```

---

## Keamanan

- `MONGO_PORT` mengekspos MongoDB langsung ke jaringan host. Batasi ke jaringan tepercaya atau hapus pemetaan port tersebut jika akses eksternal ke database tidak diperlukan.
- Nilai default `ADMIN_USER`, `ADMIN_PASS`, `SISWA_USER`, `SISWA_PASS` pada `.env.example` bersifat publik (tercantum di repository). Wajib diganti sebelum deployment produksi.
- Berkas `.env` menyimpan kredensial root MongoDB dan JWT secret dalam bentuk plain text. Batasi permission berkas (`chmod 600 .env`) dan jangan commit ke version control.

---

## Uninstal

Menghentikan dan menghapus seluruh container beserta network:

```bash
docker compose down
```

Menghapus juga named volume (`mongo-data`):

```bash
docker compose down -v
```

Peringatan: perintah di atas menghapus data MongoDB secara permanen. Folder upload backend (`./jarchive-backend/uploads`) merupakan bind mount ke host, bukan Docker volume, sehingga file di dalamnya tetap ada di disk host setelah `down -v` dan harus dihapus manual jika memang diinginkan.

---

## Lisensi

Repository ini memiliki berkas `LICENSE` dengan lisensi MIT.

---

<div align="center">
  <sub>Jarchive Infrastructure &nbsp;|&nbsp; <a href="https://github.com/siJarchive/jarchive-frontend">Frontend</a> &nbsp;|&nbsp; <a href="https://github.com/siJarchive/jarchive-backend">Backend</a></sub>
</div>
