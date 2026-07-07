# Install Laravel

Panduan instalasi project Laravel baru di lingkungan lokal maupun server.

## 1. Prasyarat

Pastikan sudah terpasang:

```bash
php --version        # >= 8.1
composer --version
```

## 2. Buat project

```bash
composer create-project laravel/laravel nama-project
cd nama-project
```

## 3. Konfigurasi environment

```bash
cp .env.example .env
php artisan key:generate
```

Edit `.env` sesuai database lokal:

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_db
DB_USERNAME=root
DB_PASSWORD=
```

## 4. Jalankan migration & seeder

```bash
php artisan migrate
php artisan db:seed   # kalau ada seeder
```

## 5. Jalankan development server

```bash
php artisan serve
# Akses di http://127.0.0.1:8000
```

## 6. (Opsional) Ekspos ke publik

Kalau butuh demo ke klien tanpa deploy, pakai
[Cloudflare Tunnel](cloudflare-tunnel.md).

!!! warning "Jangan lupa"
    File `.env` **tidak boleh di-commit** ke Git. Pastikan sudah masuk `.gitignore`.
