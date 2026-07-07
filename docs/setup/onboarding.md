# Onboarding Developer Baru

Panduan dari nol sampai siap develop untuk developer yang baru bergabung di tim Inowtech.

## Overview Stack

Sebelum mulai, kenali dulu teknologi yang dipakai tim:

- **Backend:** Laravel (PHP >= 8.1) — [detail stack](../stack/laravel.md)
- **Database:** MySQL / PostgreSQL — [konvensi](../stack/database.md)
- **Edge / Tunnel:** Cloudflare — [detail](../stack/cloudflare.md)

## Langkah 1: Setup Lingkungan Lokal

### Install prasyarat

```bash
# Cek versi minimum
php --version      # harus >= 8.1
composer --version
git --version
```

Jika belum ada, install sesuai OS masing-masing.

### Clone repo project

Jika belum punya akses ke repo, minta SSH key kamu ditambahkan oleh lead terlebih dahulu atau minta ssh yang sudah konek ke repo.

```bash
git clone <url-repo-project>
cd nama-project
```

### Install Laravel & setup environment

Ikuti panduan lengkap di [Install Laravel](install-laravel.md).

Ringkasan:

```bash
composer install
cp .env.example .env
php artisan key:generate
# Edit .env sesuai DB lokal
php artisan migrate
php artisan db:seed   # jika ada seeder
php artisan serve
```

Buka [http://localhost:8000](http://localhost:8000) — jika muncul halaman Laravel, setup berhasil.

## Langkah 2: Pahami Workflow Tim

Baca dokumen berikut sebelum mulai coding:

1. [Git Flow](../workflow/git-flow.md) — konvensi branch dan commit
2. [Environments](../workflow/environments.md) — perbedaan lokal, staging, production
3. [Testing](../workflow/testing.md) — cara test yang benar
4. [Code Review SOP](../workflow/code-review.md) — cara review dan membuat PR

## Langkah 3: Setup Git

Pastikan identitas Git sudah dikonfigurasi:

```bash
git config --global user.name "<Tanyakan ke Lead>"
git config --global user.email "<Tanyakan ke Lead>"
```

Minta akses ke repo dari lead jika belum punya.

## Langkah 4: Mulai Task Pertama

1. Ambil task dari board Notion
2. Buat branch dari `develop`:
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b feature/nama-fitur
   ```
3. Kerjakan task
4. Test sebelum push: `php artisan test`
5. Geser card ke Need Review
6. Minta review ke lead

## Checklist Onboarding

- [ ] PHP, Composer, Git sudah terinstall
- [ ] Project bisa jalan di lokal (`php artisan serve`)
- [ ] Migration berhasil dijalankan
- [ ] Sudah baca Git Flow, Testing, dan Code Review SOP
- [ ] Sudah kenal channel komunikasi tim (Notion / Discord / dll)
- [ ] Sudah punya akses ke repo dan board task
- [ ] Sudah buat branch dan push pertama (meski hanya test)

## Butuh Bantuan?

- Cek [Troubleshooting](../troubleshooting/common-errors.md) untuk error umum
- Tanya di channel tim — tidak ada pertanyaan yang bodoh
- Tag lead jika blocking lebih dari 1 jam
