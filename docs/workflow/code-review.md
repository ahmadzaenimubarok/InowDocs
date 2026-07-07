# Code Review SOP

Panduan untuk reviewer dan author agar review konsisten dan efektif.

## Tujuan Review

- Cegah bug masuk ke `develop` / `main`
- Pastikan kode mudah dipahami tim lain
- Berbagi pengetahuan antar anggota tim

## Alur Review

1. Author push branch `feature/nama-fitur`.
2. Geser tiket ke kolom **Need Review** di board.
3. Reviewer ambil tiket, checkout branch, dan lakukan review.
4. Jika ada catatan → tinggalkan komentar di tiket / chat, geser kembali ke **Plan / Todo**.
5. Jika oke → geser tiket ke **Done**, author merge ke `develop`.

## Status Review

| Status | Artinya |
|--------|---------|
| **Need Review** | Menunggu reviewer |
| **Plan / Todo** | Ada catatan, author perlu revisi |
| **Done** | Reviewer oke, sudah di-merge |

## Tanggung Jawab Author

Sebelum geser tiket ke Need Review, pastikan:

- [ ] Deskripsi tiket sudah menjelaskan apa yang berubah dan kenapa
- [ ] Semua test lokal hijau
- [ ] Tidak ada file tidak relevan ter-commit (`.env`, `node_modules`, log)
- [ ] Branch sudah di-rebase / sync dengan `develop` terbaru
- [ ] Self-review sudah dilakukan (baca diff sendiri sebelum minta review)

## Tanggung Jawab Reviewer

### Yang Harus Dicek

**Correctness**

- [ ] Logic sudah benar dan tidak ada edge case yang terlewat
- [ ] Tidak ada N+1 query (gunakan `with()` / eager loading)
- [ ] Validasi input ada di controller / form request

**Security**

- [ ] Tidak ada data sensitif di log atau response
- [ ] Input user selalu divalidasi sebelum diproses
- [ ] Akses resource dicek via policy / middleware

**Code Quality**

- [ ] Nama variabel / method deskriptif dan konsisten
- [ ] Tidak ada dead code atau commented-out code
- [ ] Tidak ada logic duplikat yang bisa di-extract

**Database**

- [ ] Migration punya `down()` yang benar
- [ ] Index sudah dipertimbangkan untuk kolom yang sering di-query
- [ ] Tidak rename kolom tanpa cek dependency

**Testing**

- [ ] Ada test untuk fungsionalitas baru
- [ ] Test tidak bergantung pada data hardcoded dari DB production

### Yang Tidak Perlu Diperdebatkan

- Preferensi style yang tidak mempengaruhi fungsionalitas → skip atau beri komentar opsional
- Sesuaikan standar dengan konvensi yang sudah ada di repo, bukan preferensi personal

## SLA Review

| Ukuran Perubahan | Target Review |
|------------------|--------------|
| Kecil (< 100 baris) | 4 jam |
| Sedang (100–400 baris) | 1 hari kerja |
| Besar (> 400 baris) | Diskusi dulu, pertimbangkan dipecah |

## Status Review

| Status | Artinya |
|--------|---------|
| **Need Review** | Menunggu reviewer |
| **In Progress** | Ada catatan, author perlu revisi |
| **Ready to Merge** | Reviewer oke, siap di-merge |

Butuh minimal **1 reviewer** sebelum merge ke `develop`, dan **1 review dari lead** sebelum merge ke `main`.

!!! tip "Reviewers are not gatekeepers"
    Tujuan review bukan mencari kesalahan, tapi memastikan kode bisa di-maintain bersama.
    Beri feedback yang konstruktif dan spesifik.