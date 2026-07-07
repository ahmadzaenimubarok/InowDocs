# Deployment SOP

Panduan resmi untuk promote kode dari `develop` ke production (`main`).

## Kapan Deploy?

Deploy ke production **hanya** dilakukan jika:

- [ ] Semua fitur sudah di-test (lihat [Testing](testing.md))
- [ ] Semua PR sudah di-review dan merged ke `develop`
- [ ] Tidak ada issue open yang bersifat blocker
- [ ] Sudah disetujui lead / PIC project

## Alur Deploy

```text
feature/* → develop → (staging) → main → production
```

### 1. Final Check di `develop`

```bash
git checkout develop
git pull origin develop
php artisan test
```

Pastikan semua test hijau sebelum lanjut.

### 2. Buat PR `develop` → `main`

- Beri judul: `release: vX.Y.Z` atau `release: nama-fitur`
- Cantumkan changelog singkat di body PR
- Minta review dari minimal 1 orang sebelum merge

### 3. Checklist Sebelum Merge ke `main`

- [ ] Test hijau di CI / lokal
- [ ] Tidak ada file `.env` atau secret ter-commit
- [ ] Migration sudah diuji di DB bersih
- [ ] `composer.lock` sudah di-commit jika ada perubahan dependency
- [ ] Versi semver di-update jika relevan

### 4. Merge & Deploy

Setelah merge ke `main`:

```bash
git checkout main
git pull origin main
```

Di server production:

```bash
cd /path/to/project
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### 5. Verifikasi Post-Deploy

- [ ] Buka aplikasi di browser, cek halaman utama
- [ ] Cek endpoint kritis (login, fitur utama yang baru di-deploy)
- [ ] Pantau log error: `tail -f storage/logs/laravel.log`
- [ ] Pastikan tidak ada job queue yang gagal (jika pakai queue)

!!! warning "Jangan deploy di luar jam kerja"
    Hindari deploy malam hari atau akhir pekan kecuali hotfix mendesak.
    Kalau ada masalah, sulit di-handle.

## Hotfix

Untuk perbaikan mendesak di production:

```bash
git checkout main
git checkout -b hotfix/nama-masalah
# ... fix ...
git commit -m "fix: deskripsi singkat"
```

Buat PR langsung ke `main`, lalu setelah merge, sync balik ke `develop`:

```bash
git checkout develop
git merge main
git push origin develop
```

## Siapa yang Deploy?

| Kondisi | Yang Deploy |
|---------|-------------|
| Fitur baru | Lead / PIC project |
| Hotfix | Developer yang handle + notif lead |
| Rollback | Lead wajib tahu sebelum eksekusi |

Jika ragu, tanya dulu. Lebih baik terlambat deploy daripada production rusak.
