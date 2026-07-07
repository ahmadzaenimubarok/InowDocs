# Git Flow

Konvensi branch dan commit yang dipakai tim produksi.

## Branch

| Branch | Fungsi |
|--------|--------|
| `main` | Produksi, selalu stabil |
| `develop` | Integrasi fitur |
| `feature/*` | Fitur baru |
| `hotfix/*` | Perbaikan mendesak di produksi |

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

Gunakan prefix: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`.
