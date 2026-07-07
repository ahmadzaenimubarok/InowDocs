# Testing

Alur testing sebelum fitur dianggap siap merge.

## Level testing

1. **Unit test** — logika dasar (model, helper).
   ```bash
   php artisan test
   ```
2. **Feature test** — alur HTTP / endpoint.
3. **Manual QA** — cek UI di browser / device.

## Check before merge

- [ ] Semua test hijau
- [ ] Tidak ada `.env` / secret ter-commit
- [ ] Migration sudah diuji di DB bersih
- [ ] Sudah di-review minimal 1 orang

!!! warning "Jangan skip"
    Kalau test merah, jangan di-merge. Perbaiki dulu sebelum lanjut.
