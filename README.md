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
   docker compose up -d --build