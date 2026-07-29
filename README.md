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

Buka file .env dan sesuaikan konfigurasi URL dan Database dan Komponen lainnya:

APP_NAME=SmartTableOrdering
APP_ENV=local
APP_KEY=base64:
APP_DEBUG=true
APP_URL=

APP_LOCALE=en
APP_FALLBACK_LOCALE=en
APP_FAKER_LOCALE=en_US

APP_MAINTENANCE_DRIVER=file
# APP_MAINTENANCE_STORE=database

# PHP_CLI_SERVER_WORKERS=4

BCRYPT_ROUNDS=12

LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=

SESSION_DRIVER=database
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null

BROADCAST_CONNECTION=reverb
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync

CACHE_STORE=database
# CACHE_PREFIX=

MEMCACHED_HOST=127.0.0.1

REDIS_CLIENT=phpredis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=
MAIL_FROM_ADDRESS=
MAIL_FROM_NAME=

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

VITE_APP_NAME="${APP_NAME}"

# midtrans
MIDTRANS_SERVER_KEY=
MIDTRANS_CLIENT_KEY=
MIDTRANS_IS_PRODUCTION=false

#reverb
BROADCAST_CONNECTION=reverb
REVERB_APP_ID=
REVERB_APP_KEY=
REVERB_APP_SECRET=
REVERB_HOST="127.0.0.1"
REVERB_PORT=8080
REVERB_SCHEME=http

VITE_REVERB_APP_KEY="${REVERB_APP_KEY}"
VITE_REVERB_HOST="${REVERB_HOST}"
VITE_REVERB_PORT="${REVERB_PORT}"
VITE_REVERB_SCHEME="${REVERB_SCHEME}"

2. Konfigurasi SSL (Cloudflare Origin Certificate)
Agar aplikasi dapat diakses menggunakan HTTPS (Port 443), Anda wajib menambahkan sertifikat dari Cloudflare Origin Server.

Buat direktori untuk menyimpan sertifikat:
mkdir -p docker/nginx/ssl
Buat file cert.pem dan key.pem, lalu paste kode sertifikat dari dashboard Cloudflare:
nano docker/nginx/ssl/cert.pem  # Masukkan Origin Certificate
nano docker/nginx/ssl/key.pem   # Masukkan Private Key

Penting: Pastikan pengaturan SSL/TLS di dashboard Cloudflare Anda diatur ke mode Full atau Full (strict).

3. Jalankan Docker Containers
Setelah konfigurasi siap, jalankan semua container di latar belakang:

docker compose up -d --build
4. Inisialisasi Aplikasi (Composer & Permission)
Lakukan instalasi dependensi PHP dan atur hak akses folder agar Nginx dan PHP-FPM dapat membaca/menulis data sistem:

# Install dependensi PHP
docker exec -it sto-app composer install --no-dev --no-interaction --optimize-autoloader

# Atur hak akses folder storage dan bootstrap
docker exec -it sto-app chown -R www-data:www-data storage bootstrap/cache
docker exec -it sto-app chmod -R 775 storage bootstrap/cache

5. Konfigurasi Kunci, Database, dan Storage
Hasilkan kunci aplikasi, jalankan migrasi tabel beserta seeder (untuk akun admin default), dan buat symlink untuk akses gambar publik:

docker exec -it sto-app php artisan key:generate
docker exec -it sto-app php artisan migrate --seed
docker exec -it sto-app php artisan storage:link

6. Build Frontend Assets (Vite)
Karena aplikasi menggunakan Vite, Anda wajib melakukan build assets (CSS & JS) agar tidak terjadi error ViteManifestNotFoundException. Proses ini menggunakan container Node.js sementara agar VPS Anda tetap bersih:

# Install NPM packages & Build
docker run --rm -v $(pwd):/var/www/html -w /var/www/html node:20 npm install
docker run --rm -v $(pwd):/var/www/html -w /var/www/html node:20 npm run build

# Berikan hak akses untuk folder hasil build
docker exec -it sto-app chown -R www-data:www-data public/build

7. Clear Cache & Finalisasi
Bersihkan semua cache Laravel agar konfigurasi terbaru segera diterapkan:

docker exec -it sto-app php artisan optimize:clear

Aplikasi Smart Table Ordering sekarang berhasil di-deploy dan dapat diakses dengan aman melalui domain HTTPS Anda!
```
