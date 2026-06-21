<div align="center">
  <h1>Jarchive Infrastructure</h1>
  <p>Konfigurasi Infrastruktur sebagai Kode (IaC) berbasis Docker untuk deployment ekosistem platform Jarchive.</p>
  <p>
    <a href="https://github.com/siJarchive/jarchive-frontend">Frontend</a> | 
    <a href="https://github.com/siJarchive/jarchive-backend">Backend</a>
  </p>
</div>

![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?logo=docker&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Linux-lightgrey?logo=linux)

## Fitur

| Fitur | Deskripsi |
| --- | --- |
| Multi-container Deployment | Mengorkestrasi Frontend, Backend, dan Database MongoDB secara terpusat. |
| Environment Isolation | Memisahkan konfigurasi jaringan, kredensial, dan port eksternal melalui variabel lingkungan. |
| Data Persistence | Mengamankan data base dan file unggahan pengguna (assets) melalui *Docker Volumes*. |

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
|   |  [FRONTEND_PORT]      |           |  [BACKEND_PORT]       |   |
|   +-----------------------+           +-----------------------+   |
|                                                   |               |
|                                                  TCP              |
|                                                   |               |
|                                       +-----------------------+   |
|                                       |                       |   |
|                                       |  jarchive-mongodb     |   |
|                                       |  (MongoDB)            |   |
|                                       |                       |   |
|                                       |  Port Eksternal:      |   |
|                                       |  [MONGO_PORT]         |   |
|                                       +-----------------------+   |
+-------------------------------------------------------------------+

```

## Prasyarat

| Komponen | Spesifikasi / Versi | Keterangan |
| --- | --- | --- |
| OS | Linux / macOS / Windows (WSL2) | Diuji secara optimal pada lingkungan Unix |
| Docker | Engine 20.10+ | Diperlukan untuk runtime container |
| Docker Compose | V2+ | Diperlukan untuk orkestrasi arsitektur |
| Git | Versi CLI Terbaru | Diperlukan untuk kloning *source code* submodul |

## Instalasi

1. Kloning repositori infrastruktur beserta repositori dependennya:

```bash
git clone [https://github.com/siJarchive/jarchive-infrastructure.git](https://github.com/siJarchive/jarchive-infrastructure.git)
cd jarchive-infrastructure

git clone [https://github.com/siJarchive/jarchive-frontend.git](https://github.com/siJarchive/jarchive-frontend.git)
git clone [https://github.com/siJarchive/jarchive-backend.git](https://github.com/siJarchive/jarchive-backend.git)

```

*Jika repositori bersifat private, otentikasi menggunakan token GHP:*

```bash
git clone https://[GHP_TOKEN]@[github.com/siJarchive/jarchive-frontend.git](https://github.com/siJarchive/jarchive-frontend.git)

```

## Konfigurasi

Persiapkan parameter environment sebelum mengeksekusi container. Salin *template* yang telah disediakan:

```bash
cp .env.example .env

```

Sesuaikan nilai di dalam `.env` sesuai kebutuhan.

| Variabel Lingkungan | Status | Default | Deskripsi |
| --- | --- | --- | --- |
| `BACKEND_PORT` | Wajib | 5000 | Port host yang dipetakan ke layanan API backend. |
| `FRONTEND_PORT` | Wajib | 11223 | Port host yang dipetakan ke layanan web antarmuka. |
| `VITE_API_URL` | Wajib | - | Endpoint absolut API backend (contoh: `http://192.168.1.10:5000`). |
| `JWT_SECRET` | Wajib | - | Kunci enkripsi statis untuk validasi autentikasi JWT. |
| `ADMIN_USER` | Wajib | - | Identitas login untuk peran otorisasi `Guru`. |
| `ADMIN_PASS` | Wajib | - | Kredensial login untuk peran otorisasi `Guru`. |
| `SISWA_USER` | Wajib | - | Identitas login untuk peran otorisasi `Siswa`. |
| `SISWA_PASS` | Wajib | - | Kredensial login untuk peran otorisasi `Siswa`. |
| `MONGO_PORT` | Wajib | 27017 | Port host yang dipetakan ke layanan database MongoDB. |
| `MONGO_ROOT_USER` | Wajib | - | Username untuk otentikasi administrator di MongoDB. |
| `MONGO_ROOT_PASS` | Wajib | - | Password untuk otentikasi administrator di MongoDB. |
| `MONGO_URI` | Wajib | - | String koneksi yang mengarah ke internal container, sertakan autentikasi root. |

## Manajemen dan Operasional

Menjalankan infrastruktur dalam mode *detached*:

```bash
docker compose up -d

```

Menghentikan seluruh container terkait tanpa merusak persistensi data (*volume*):

```bash
docker compose stop

```

Melakukan pemantauan log secara *realtime* untuk seluruh container:

```bash
docker compose logs -f

```

Mengecek status dan pemetaan port layanan yang aktif:

```bash
docker compose ps

```

## Uninstal

Menghapus seluruh container, *network*, dan me-reset sistem ke awal.
*Peringatan: Perintah ini menghapus volume secara permanen (menghilangkan file upload dan database).*

```bash
docker compose down -v

```
