# Cloudflare

Cloudflare dipakai untuk edge, DNS, dan tunnel (mengekspos server lokal
ke publik tanpa buka port firewall).

## Komponen yang dipakai

- **Cloudflare Tunnel (cloudflared)** — menghubungkan lokal ke domain aman.
- **Cloudflare Pages** — deploy dokumentasi statis (repo ini).
- **DNS** — manajemen domain & proxy.

## Kenapa pakai Tunnel?

- Tidak perlu buka port inbound di firewall.
- Ada layer auth (Cloudflare Access) kalau mau batasi akses.
- Gratis untuk kebutuhan tim kecil.

Lihat panduan setup di [Panduan Setup → Cloudflare Tunnel](../setup/cloudflare-tunnel.md).
