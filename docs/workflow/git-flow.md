# Git Flow

Konvensi branch dan commit yang dipakai tim produksi.

## Branch

| Branch | Fungsi |
|--------|--------|
| `main` | Produksi, selalu stabil |
| `develop` | Integrasi fitur |
| `feature/*` | Fitur baru |
| `bugfix/*` | Perbaikan di produksi |

## Alur kerja

1. Ambil task dari board.
2. Buat branch `feature/nama-fitur` dari `develop`.
3. Commit dengan pesan jelas:

```bash
   git commit -m "feat: tambah endpoint login"
```

4. Push & buat Pull Request ke `develop`.
5. Setelah review & CI hijau → merge.

## Konvensi commit

Gunakan prefix berikut agar riwayat commit mudah dibaca dan ditelusuri:

| Prefix | Kapan dipakai | Contoh |
|--------|---------------|--------|
| `feat:` | Menambahkan fitur baru yang terlihat oleh pengguna atau sistem lain | `feat: tambah endpoint login` |
| `bugfix:` | Memperbaiki bug atau perilaku yang salah | `bugfix: perbaiki kalkulasi diskon negatif` |
| `docs:` | Perubahan pada dokumentasi saja — tidak ada kode produksi yang berubah | `docs: perbarui README cara setup lokal` |
| `chore:` | Pekerjaan pemeliharaan rutin yang tidak memengaruhi logika aplikasi, seperti update dependency atau konfigurasi tooling | `chore: upgrade eslint ke v9` |
| `refactor:` | Mengubah struktur atau kualitas kode tanpa menambah fitur maupun memperbaiki bug | `refactor: ekstrak logika validasi ke helper` |

### Aturan penulisan pesan commit

- Gunakan **bahasa Indonesia** atau **Inggris** secara konsisten dalam satu proyek.
- Tulis dalam bentuk **kalimat aktif, imperatif** — bayangkan kalimatnya diawali "Commit ini akan…"
  - ✅ `feat: tambah filter pencarian produk`
  - ❌ `feat: menambahkan filter pencarian produk`
- Batasi baris pertama maksimal **72 karakter**.
- Jika perlu penjelasan tambahan, tambahkan baris kosong lalu body commit:

```bash
  git commit -m "refactor: pisahkan logika auth ke service tersendiri

  Sebelumnya logika auth tercampur di controller sehingga sulit diuji.
  Pemisahan ini memudahkan unit testing dan reuse di endpoint lain."
```

### Prefix lain yang umum dipakai (opsional)

| Prefix | Kapan dipakai |
|--------|---------------|
| `test:` | Menambah atau memperbaiki unit/integration test |
| `style:` | Perubahan format kode (spasi, semicolon, dll) tanpa mengubah logika |
| `ci:` | Perubahan konfigurasi CI/CD (GitHub Actions, Dockerfile, dll) |
| `revert:` | Membatalkan commit sebelumnya |