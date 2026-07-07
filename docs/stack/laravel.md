# Laravel

Laravel adalah PHP framework utama yang dipakai tim produksi.

## Versi & Prasyarat

- PHP >= 8.1
- Composer
- Ekstensi PHP: `mbstring`, `xml`, `ctype`, `json`, `tokenizer`, `bcmath`

## Struktur direktori penting

```
app/         # Logic aplikasi (Models, Controllers, dll)
routes/      # Definisi route (web.php, api.php)
config/      # Konfigurasi
database/    # Migration & seeder
public/      # Document root (index.php)
```

## Catatan tim

- Selalu jalankan `php artisan migrate` setelah pull migration baru.
- Gunakan `.env.example` sebagai template; jangan commit `.env` asli.
