# Database

Tim produksi umumnya memakai MySQL atau PostgreSQL.

## Konvensi

- Nama tabel: `snake_case` jamak (`users`, `job_applications`).
- Migration wajib punya `down()` yang bersih.
- Jangan rename kolom tanpa cek dependency di kode.

## Akses

- Kredensial ada di `.env` (tidak di-commit).
- Untuk debug lokal, pakai client seperti TablePlus / DBeaver / `mysql` CLI.
