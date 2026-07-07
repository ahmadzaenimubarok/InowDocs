# Expose Webhook Lokal dengan Domain Statis

> **Use case:** testing payment gateway, webhook, OAuth callback, atau integrasi API tanpa perlu deploy ke server publik. Menggunakan URL **statis** dan **HTTPS**, cocok untuk Stripe, Midtrans, Xendit, Twilio, GitHub, GitLab, dan layanan serupa.

Ngrok versi gratis kurang ideal untuk pengujian webhook jangka panjang karena:

- URL berubah setiap kali tunnel dibuat
- Memiliki batasan request dan koneksi
- Perlu memperbarui URL webhook di dashboard provider

Alternatif yang lebih stabil adalah **Cloudflare Tunnel** dengan domain sendiri.

---

## Arsitektur

```text
Payment Gateway
        │
        ▼
https://webhook.example.com
        │
        ▼
Cloudflare
        │
        ▼
Cloudflare Tunnel (cloudflared)
        │
        ▼
http://localhost:8000
```

Webhook hanya mengetahui domain publik, sementara aplikasi tetap berjalan secara lokal.

---

## Prasyarat

Pastikan telah memenuhi hal berikut:

- Memiliki domain yang DNS-nya dikelola oleh Cloudflare
- Memiliki akun Cloudflare
- Aplikasi lokal sudah berjalan (misalnya `http://localhost:8000`)
- `cloudflared` sudah terpasang

---

## 1. Install cloudflared

Ikuti panduan [instalasi resmi](https://developers.cloudflare.com/tunnel/downloads/) sesuai sistem operasi yang digunakan:

- Windows
- macOS
- Linux (Ubuntu, Debian, Fedora, Arch, dan distribusi lainnya)

Setelah selesai, pastikan instalasi berhasil.

```bash
cloudflared --version
```

---

## 2. Login ke Cloudflare

Jalankan:

```bash
cloudflared login
```

Browser akan terbuka.

Selanjutnya:

1. Login ke akun Cloudflare.
2. Pilih domain yang akan digunakan.
3. Berikan izin akses.

Jika berhasil, folder konfigurasi akan dibuat secara otomatis.

**Linux/macOS**

```text
~/.cloudflared/
```

**Windows**

```text
%USERPROFILE%\.cloudflared\
```

---

## 3. Buat Tunnel

```bash
cloudflared tunnel create webhook-local
```

Perintah ini akan menghasilkan:

- Tunnel UUID
- File kredensial (`<TUNNEL_UUID>.json`)

Contoh:

```text
Tunnel UUID:
12345678-abcd-1234-abcd-1234567890ab
```

---

## 4. Hubungkan Domain ke Tunnel

Misalnya menggunakan subdomain:

```text
webhook.example.com
```

Jalankan:

```bash
cloudflared tunnel route dns webhook-local webhook.example.com
```

Cloudflare akan otomatis membuat DNS Record berupa **CNAME**.

Pastikan status DNS adalah **Proxied (Orange Cloud)**.

---

## 5. Buat File Konfigurasi

Buat file:

```text
config.yml
```

Lokasi default:

**Linux/macOS**

```text
~/.cloudflared/config.yml
```

**Windows**

```text
%USERPROFILE%\.cloudflared\config.yml
```

Contoh konfigurasi:

```yaml
tunnel: TUNNEL_UUID
credentials-file: /path/to/TUNNEL_UUID.json

ingress:
  - hostname: webhook.example.com
    service: http://localhost:8000

  - service: http_status:404
```

Sesuaikan:

- `TUNNEL_UUID`
- `credentials-file`
- `hostname`
- `localhost:8000`

Contoh path file credentials:

### Linux

```text
/home/username/.cloudflared/12345678-abcd.json
```

### macOS

```text
/Users/username/.cloudflared/12345678-abcd.json
```

### Windows

```text
C:\Users\Username\.cloudflared\12345678-abcd.json
```

!!! tip "Perhatikan Indentasi YAML"

    File `config.yml` menggunakan format YAML yang sensitif terhadap spasi. Pastikan indentasi sesuai contoh.

---

## 6. Jalankan Tunnel

```bash
cloudflared tunnel run webhook-local
```

Jika berhasil, maka request ke:

```text
https://webhook.example.com
```

akan diteruskan ke:

```text
http://localhost:8000
```

---

## Menjalankan Tanpa `config.yml` (Opsional)

Untuk kebutuhan sederhana, tunnel juga dapat dijalankan langsung:

```bash
cloudflared tunnel --url http://localhost:8000
```

Namun cara ini **tidak menggunakan domain statis**, sehingga kurang cocok untuk webhook dari layanan pihak ketiga.

---

## Menjalankan Sebagai Service (Opsional)

Agar tunnel otomatis berjalan saat komputer menyala, `cloudflared` dapat dijalankan sebagai service.

Implementasinya berbeda untuk setiap sistem operasi:

| Sistem Operasi | Service Manager |
|----------------|-----------------|
| Windows | Windows Service |
| Linux | systemd |
| macOS | launchd |

Silakan mengikuti dokumentasi resmi Cloudflare sesuai sistem operasi yang digunakan.

---

## Contoh Endpoint Webhook

Pisahkan endpoint berdasarkan provider agar lebih mudah melakukan debugging.

```text
POST /webhooks/payment/stripe
POST /webhooks/payment/midtrans
POST /webhooks/payment/xendit
POST /webhooks/payment/paypal

POST /webhooks/message/twilio
POST /webhooks/github
POST /webhooks/gitlab
```

---

## Troubleshooting

Jika tunnel tidak berfungsi, periksa beberapa hal berikut.

- Pastikan aplikasi lokal masih berjalan.
- Pastikan DNS sudah mengarah ke tunnel.
- Pastikan DNS menggunakan status **Proxied**.
- Pastikan isi `config.yml` benar.
- Pastikan path `credentials-file` benar.
- Jalankan kembali tunnel dan periksa log error.

Untuk melihat daftar tunnel:

```bash
cloudflared tunnel list
```

Melihat informasi tunnel:

```bash
cloudflared tunnel info webhook-local
```

Menguji konfigurasi:

```bash
cloudflared tunnel ingress validate
```

Melihat log lebih detail:

```bash
cloudflared tunnel run webhook-local --loglevel debug
```

---

## Kelebihan Cloudflare Tunnel

- URL publik tetap (statis)
- HTTPS otomatis
- Tidak perlu membuka port pada router
- Tidak memerlukan IP publik
- Mendukung banyak subdomain dalam satu tunnel
- Gratis untuk penggunaan dasar
- Cocok untuk webhook, callback OAuth, dan pengembangan aplikasi

---

## Kesimpulan

| Cloudflare Tunnel | Ngrok (Free) |
|-------------------|--------------|
| URL statis | ❌ |
| HTTPS otomatis | ✅ |
| Domain sendiri | ✅ |
| Cocok untuk webhook | ✅ |
| Perlu update URL setiap restart | ❌ |

Cloudflare Tunnel merupakan pilihan yang lebih tepat untuk pengembangan yang melibatkan webhook dan integrasi dengan layanan pihak ketiga karena menyediakan URL yang tetap, HTTPS otomatis, dan tidak memerlukan deploy ke server publik.