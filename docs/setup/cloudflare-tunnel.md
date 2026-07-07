# Setup Cloudflare Tunnel

Cloudflare Tunnel (`cloudflared`) mengekspos server lokal ke internet
melalui domain Cloudflare — tanpa buka port firewall.

## 1. Install cloudflared

**Linux:**

```bash
curl -L --output cloudflared https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/
```

Cek:

```bash
cloudflared --version
```

## 2. Login & autentikasi

```bash
cloudflared tunnel login
```

Browser akan buka untuk pilih domain Cloudflare kamu.

## 3. Buat tunnel

```bash
cloudflared tunnel create inowdocs-tunnel
```

Simpan `Tunnel ID` yang muncul.

## 4. Konfigurasi

Buat file `~/.cloudflared/config.yml`:

```yaml
tunnel: <TUNNEL_ID>
credentials-file: /root/.cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: docs.inowtech.id
    service: http://localhost:8000
  - service: http://localhost:80
```

## 5. Jalankan & route DNS

```bash
cloudflared tunnel route dns inowdocs-tunnel docs.inowtech.id
cloudflared tunnel run inowdocs-tunnel
```

## 6. (Production) Jalankan sebagai service

```bash
cloudflared service install
sudo systemctl enable --now cloudflared
```

!!! tip "Keamanan"
    Kalau mau batasi akses, tambahkan Cloudflare Access di depan hostname
    supaya hanya tim yang punya akses bisa membukanya.
