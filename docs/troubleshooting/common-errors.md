# Common Errors

Error yang sering muncul saat kerja dengan stack Inowtech.

## 1. `SQLSTATE[HY000] [2002] Connection refused`

**Penyebab:** Database belum jalan atau `.env` salah.

**Solusi:**
```bash
# Cek service DB
sudo systemctl status mysql
# Atau periksa DB_HOST / DB_PORT di .env
```

## 2. `APP_KEY not set`

**Penyebab:** `php artisan key:generate` belum dijalankan.

**Solusi:**
```bash
php artisan key:generate
```

## 3. Cloudflare Tunnel `connection error`

**Penyebab:** `cloudflared` tidak jalan atau credentials hilang.

**Solusi:**
```bash
cloudflared tunnel list
cloudflared tunnel run inowdocs-tunnel
# Cek juga file credentials di ~/.cloudflared/
```

## 4. Port sudah dipakai (`Address already in use`)

**Solusi:** ubah port `php artisan serve --port=8080` atau matikan proses lama.

!!! tip "Kontribusi"
    Ketemu error baru? Tambahkan di halaman ini supaya tim tidak mengulang
    kesalahan yang sama.
