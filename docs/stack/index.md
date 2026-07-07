# Mapping Stack

Halaman ini memetakan teknologi utama yang dipakai tim produksi Inowtech.
Tujuannya: supaya semua anggota tim punya gambaran utuh sebelum masuk ke
detail setup tiap komponen.

## Overview

| Layer | Teknologi | Catatan |
|-------|-----------|---------|
| Backend | Laravel (PHP) | Framework utama aplikasi |
| Frontend | Blade / asset bundler | Tergantung project |
| Database | MySQL / PostgreSQL | Sesuai kebutuhan |
| Edge & Tunnel | Cloudflare Tunnel | Ekspos lokal ke publik aman |
| Deploy | Cloudflare Pages / server | Static & dinamis |

## Detail per komponen

- [Laravel](laravel.md)
- [Cloudflare](cloudflare.md)
- [Database](database.md)

!!! info "Milestone"
    Halaman ini termasuk **m1 — Mapping Stack** pada KPI dokumentasi.
