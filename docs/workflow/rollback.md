# Rollback & Incident Response

Langkah yang harus diambil jika deployment menyebabkan masalah di production.

!!! danger "Prioritas utama: stabilkan production"
    Jangan habiskan waktu mencari root cause saat production masih rusak.
    Rollback dulu, investigasi belakangan.

## Langkah Pertama (< 5 menit pertama)

1. **Konfirmasi masalah** — cek log, cek apakah memang terjadi setelah deploy
2. **Notif lead / tim** — jangan handle sendiri diam-diam
3. **Putuskan: fix forward atau rollback?**
   - Fix forward: bug kecil, bisa dipatch dalam < 15 menit
   - Rollback: bug kritikal, tidak ada fix cepat

## Rollback Kode

### Cara 1: Revert commit (direkomendasikan)

```bash
# Lihat commit yang bermasalah
git log --oneline -10

# Revert commit spesifik (buat commit baru yang membalik perubahan)
git revert <commit-hash>
git push origin main
```

Kemudian deploy ulang di server.

### Cara 2: Reset ke commit sebelumnya

!!! warning "Destruktif"
    Cara ini menghapus history. Gunakan hanya jika revert tidak memungkinkan
    dan sudah disetujui lead.

```bash
git reset --hard <commit-hash-sebelum-deploy>
git push --force origin main
```

### Deploy rollback di server

```bash
cd /path/to/project
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## Rollback Migration Database

!!! danger "Hati-hati"
    Rollback migration bisa menyebabkan kehilangan data jika ada data baru
    yang masuk setelah migration dijalankan. Selalu backup dulu.

### Backup dulu sebelum rollback

```bash
# MySQL
mysqldump -u root -p nama_db_production > backup_$(date +%Y%m%d_%H%M%S).sql

# PostgreSQL
pg_dump nama_db_production > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Rollback 1 migration terakhir

```bash
php artisan migrate:rollback
```

### Rollback beberapa step

```bash
php artisan migrate:rollback --step=3
```

### Cek status migration

```bash
php artisan migrate:status
```

## Checklist Pasca-Rollback

- [ ] Production sudah stabil kembali
- [ ] Lead dan tim sudah dinotif bahwa rollback berhasil
- [ ] Root cause sudah diidentifikasi (meski belum di-fix)
- [ ] Bug sudah di-create di issue tracker
- [ ] Post-mortem dijadwalkan (untuk incident besar)

## Template Notif Incident

Kirim ke channel tim segera setelah incident terdeteksi:

```
🔴 [INCIDENT] Nama masalah singkat
Waktu: HH:MM
Dampak: [apa yang terdampak, siapa yang kena]
Status: Investigasi / Rollback / Resolved
PIC: @nama
Update berikutnya: HH:MM
```

Setelah resolved:

```
✅ [RESOLVED] Nama masalah singkat
Durasi: X menit
Root cause: [penjelasan singkat]
Fix: [apa yang dilakukan]
Tindak lanjut: [link issue / PR]
```

## Post-Mortem (untuk incident besar)

Lakukan dalam 1–2 hari setelah incident. Fokus pada sistem, bukan menyalahkan orang.

Yang perlu dijawab:
1. Apa yang terjadi?
2. Apa dampaknya?
3. Apa root cause-nya?
4. Mengapa tidak terdeteksi sebelum production?
5. Apa yang bisa diperbaiki di proses (testing, review, monitoring)?
