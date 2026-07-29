# Smart Table Ordering - Production Deployment Guide

Panduan lengkap untuk melakukan instalasi dan _deployment_ aplikasi **Smart Table Ordering** di _environment production_ menggunakan Docker (Nginx, PHP-FPM, MySQL) dan diintegrasikan dengan Cloudflare SSL (HTTPS).

## 📋 Prasyarat Sistem

Sebelum memulai instalasi, pastikan server/VPS Anda (Ubuntu/Debian) sudah terinstal:

- **Git**
- **Docker** & **Docker Compose**
- Domain aktif yang sudah diarahkan ke IP VPS via **Cloudflare** (Proxy status: Proxied/Orange Cloud).

---

## 🚀 Langkah Instalasi & Deployment

### 1. Clone Repository & Setup Environment

Pertama, unduh kode sumber dari repository branch `development` (atau `main` sesuai kebutuhan).

```bash
git clone [https://github.com/rzkyftrhmn/smart-table-ordering.git](https://github.com/rzkyftrhmn/smart-table-ordering.git)
cd smart-table-ordering

# Salin file konfigurasi environment
cp .env.example .env
```
### 2. Konfigurasi SSL (Cloudflare Origin Certificate)

```bash
Agar aplikasi dapat diakses menggunakan HTTPS (Port 443), Anda wajib menambahkan sertifikat dari Cloudflare Origin Server.

Buat direktori untuk menyimpan sertifikat:
mkdir -p docker/nginx/ssl
Buat file cert.pem dan key.pem, lalu paste kode sertifikat dari dashboard Cloudflare:
nano docker/nginx/ssl/cert.pem  # Masukkan Origin Certificate
nano docker/nginx/ssl/key.pem   # Masukkan Private Key

Penting: Pastikan pengaturan SSL/TLS di dashboard Cloudflare Anda diatur ke mode Full atau Full (strict).
```
3. Jalankan Docker Containers

```bash

Setelah konfigurasi siap, jalankan semua container di latar belakang:

docker compose up -d --build

```

4. Inisialisasi Aplikasi (Composer & Permission)
```bash
Lakukan instalasi dependensi PHP dan atur hak akses folder agar Nginx dan PHP-FPM dapat membaca/menulis data sistem:

# Install dependensi PHP
docker exec -it sto-app composer install --no-dev --no-interaction --optimize-autoloader

# Atur hak akses folder storage dan bootstrap
docker exec -it sto-app chown -R www-data:www-data storage bootstrap/cache
docker exec -it sto-app chmod -R 775 storage bootstrap/cache
```
5. Konfigurasi Kunci, Database, dan Storage
```bash
Hasilkan kunci aplikasi, jalankan migrasi tabel beserta seeder (untuk akun admin default), dan buat symlink untuk akses gambar publik:

docker exec -it sto-app php artisan key:generate
docker exec -it sto-app php artisan migrate --seed
docker exec -it sto-app php artisan storage:link
```
6. Build Frontend Assets (Vite)
```bash
Karena aplikasi menggunakan Vite, Anda wajib melakukan build assets (CSS & JS) agar tidak terjadi error ViteManifestNotFoundException. Proses ini menggunakan container Node.js sementara agar VPS Anda tetap bersih:

# Install NPM packages & Build
docker run --rm -v $(pwd):/var/www/html -w /var/www/html node:20 npm install
docker run --rm -v $(pwd):/var/www/html -w /var/www/html node:20 npm run build

# Berikan hak akses untuk folder hasil build
docker exec -it sto-app chown -R www-data:www-data public/build
```
7. Clear Cache & Finalisasi
```bash
Bersihkan semua cache Laravel agar konfigurasi terbaru segera diterapkan:

docker exec -it sto-app php artisan optimize:clear

Aplikasi Smart Table Ordering sekarang berhasil di-deploy dan dapat diakses dengan aman melalui domain HTTPS Anda!
```
