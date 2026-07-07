# Environments

Panduan pembedaan environment agar tidak ada kejutan di production.

## Tiga Environment Standar

| Environment | Branch | Tujuan |
|-------------|--------|--------|
| **Local** | `feature/*` / `develop` | Development & unit test |
| **Staging** | `develop` | QA & integration test sebelum ke production |
| **Production** | `main` | Live, dipakai user nyata |

!!! warning "Jangan test di production"
    Semua pengujian harus selesai di lokal atau staging sebelum merge ke `main`.

## Konfigurasi `.env` per Environment

Setiap environment punya `.env` sendiri. Jangan copy-paste mentah dari satu environment ke yang lain.

### Local (`.env`)

```dotenv
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_DATABASE=nama_db_lokal
DB_USERNAME=root
DB_PASSWORD=

MAIL_MAILER=log        # email tidak benar-benar terkirim
```

### Staging

```dotenv
APP_ENV=staging
APP_DEBUG=true          # boleh true untuk debugging
APP_URL=https://staging.example.com

DB_DATABASE=nama_db_staging
```

### Production

```dotenv
APP_ENV=production
APP_DEBUG=false         # wajib false, jangan expose error ke user
APP_URL=https://example.com

DB_DATABASE=nama_db_production
```

!!! danger "`APP_DEBUG=true` di production"
    Ini membuka stack trace ke publik. Wajib `false` di production.

## Database per Environment

Gunakan database yang terpisah untuk setiap environment. Jangan pernah pakai DB production untuk development.

| Environment | Database |
|-------------|----------|
| Local | `nama_db` |
| Testing | `nama_db_test` |
| Staging | `nama_db_staging` |
| Production | `nama_db_production` |

### Setup DB Testing

Laravel membaca `.env.testing` saat menjalankan `php artisan test` atau `phpunit`.
File ini menimpa `.env` secara penuh untuk sesi testing — pastikan file ini ada dan tidak di-commit ke repo.

**1. Buat file `.env.testing`**

Salin dari `.env`, lalu ubah nilai berikut:

```dotenv
APP_ENV=testing
APP_DEBUG=true
APP_KEY=                # boleh kosong, akan di-generate di langkah berikutnya

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_db_test
DB_USERNAME=root
DB_PASSWORD=

CACHE_DRIVER=array      # tidak pakai Redis/Memcached sungguhan
SESSION_DRIVER=array
QUEUE_CONNECTION=sync   # job dijalankan langsung, tidak perlu worker
MAIL_MAILER=array       # email ditangkap di array, tidak terkirim
```

**2. Generate app key untuk environment testing**

```bash
php artisan key:generate --env=testing
```

**3. Buat database testing**

```sql
CREATE DATABASE nama_db_test CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**4. Jalankan migrasi ke DB testing**

```bash
php artisan migrate --env=testing
```

Atau gunakan `RefreshDatabase` trait di test class agar migrasi dijalankan otomatis setiap test:

```php
use Illuminate\Foundation\Testing\RefreshDatabase;

class ContohTest extends TestCase
{
    use RefreshDatabase;

    // ...
}
```

**5. Verifikasi**

```bash
php artisan test
```

Pastikan output tidak menyebut database lokal (`nama_db`) — hanya `nama_db_test`.

> **Catatan:** Tambahkan `.env.testing` ke `.gitignore` seperti `.env` biasa.
> Simpan contoh nilainya di `.env.testing.example` yang boleh di-commit ke repo.

## Mengekspos Local ke Publik (Staging Sementara)

Untuk demo ke klien atau test webhook tanpa deploy ke server, pakai Cloudflare Tunnel.
Lihat [Setup Cloudflare Tunnel](../setup/cloudflare-tunnel.md).
