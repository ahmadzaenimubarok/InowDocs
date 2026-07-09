# InowDocs — AGENTS.md

Repo: dokumentasi produksi tim Inowtech (MkDocs + Material theme, bahasa Indonesia).

## Quick start

```bash
source .venv/bin/activate
mkdocs serve          # live preview at http://127.0.0.1:8000
mkdocs build          # static site ke site/ (gitignored)
```

## Conventions

- Semua konten di `docs/`, format Markdown.
- Deploy otomatis via Cloudflare Pages: push ke `main`.
- Remote GitHub via custom SSH host `github-ahmad.com`.
- Tidak ada test / lint / typecheck — hanya dokumentasi.
