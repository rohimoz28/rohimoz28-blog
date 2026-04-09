# Repository Guidelines

## Project Structure & Module Organization
Repository ini adalah blog statis berbasis Hugo dengan theme `PaperMod`.
- `content/blog/`: sumber artikel utama. Setiap post idealnya berada di folder sendiri (`content/blog/<slug>/index.md`) agar mudah menyimpan gambar.
- `content/about.md`, `content/archives.md`: halaman statis.
- `layouts/`: override template dari theme (gunakan ini untuk kustomisasi sebelum menyentuh theme langsung).
- `themes/PaperMod/`: submodule theme, hindari edit langsung kecuali benar-benar perlu.
- `public/` dan `docs/`: output build statis. `hugo.toml` saat ini publish ke `public`.
- `archetypes/`: template front matter post baru.

## Build, Test, and Development Commands
Gunakan Hugo CLI (versi acuan di Netlify: `0.126.3`).
- `hugo server -D`: jalankan local dev server termasuk draft.
- `hugo --gc --minify`: build production seperti di Netlify (`netlify.toml`).
- `hugo --gc --minify --buildFuture`: validasi deploy preview dengan konten bertanggal masa depan.
- `hugo new blog/<slug>/index.md`: buat draft post baru dari archetype.

## Coding Style & Naming Conventions
- Gunakan Markdown yang konsisten dan ringkas; heading berurutan (`##`, `###`).
- Gunakan indentation 2 spasi pada `hugo.toml` agar konsisten dengan file existing.
- Nama slug gunakan lowercase dan kebab-case, contoh: `setup-formatter-odoo`.
- Simpan aset post di folder post masing-masing: `content/blog/<slug>/images/...`.
- Hindari hardcode URL absolut internal; pakai path relatif Hugo bila memungkinkan.

## Testing Guidelines
Belum ada test framework otomatis. Validasi perubahan dengan:
- Build tanpa error: `hugo --gc --minify`.
- Cek manual halaman terdampak di local server (`hugo server -D`).
- Untuk perubahan layout, verifikasi tampilan mobile dan desktop.

## Commit & Pull Request Guidelines
Pola commit di histori campuran, namun mayoritas mengikuti gaya ringkas + prefix (`feat:`, `fix:`, `docs:`). Disarankan:
- Format: `<type>(<scope>): <ringkasan>` (contoh: `feat(content): add postgresql backup article`).
- Satu commit untuk satu perubahan logis.
- PR harus berisi: tujuan perubahan, area terdampak, langkah verifikasi, dan screenshot jika mengubah UI/layout.
- Jika update konten, sertakan path post yang diubah di deskripsi PR.

## Security & Configuration Tips
- Jangan commit secret/token ke Markdown, gambar, atau config.
- Perubahan `baseURL`, `publishDir`, dan Netlify environment harus direview ekstra karena memengaruhi deployment.
