# GEMINI.md - Panduan Instruksional Blog Rohimoz28

Dokumen ini berisi konteks teknis dan konvensi operasional untuk membantu Gemini CLI dalam mengelola blog Lord Rohimoz28.

## 🚀 Gambaran Project

- **Tujuan:** Blog teknikal dokumentasi (Odoo, DevOps, PostgreSQL, Linux, Git).
- **Tech Stack:**
  - **Generator:** Hugo (v0.128.0+ recommended).
  - **Theme:** PaperMod.
  - **Hosting:** Netlify.
  - **Development:** Docker, Git, Shell Scripting.

## 🏗️ Alur Kerja Teknis

### Perintah Utama (Building & Running)
- **Development Server:** `hugo serve -D`
- **Build Produksi:** `hugo --gc --minify`
- **Buat Artikel Baru:**
  ```bash
  hugo new blog/[nama-slug]/index.md
  ```
  *(Pastikan dijalankan dari root directory blog Lord).*

### Deployment (Netlify)
Konfigurasi deployment berada di `netlify.toml`.
- **Publish Dir:** `public`
- **Hugo Version:** v0.126.3 (terkonfigurasi di Netlify, namun secara lokal Lord menggunakan v0.152.2).

## 📝 Konvensi Pengembangan & Konten

### Struktur Folder Konten
Setiap artikel harus berada dalam directory sendiri di `content/blog/` untuk kemudahan manajemen aset (gambar/file):
```text
content/blog/[nama-slug]/
├── index.md           # File konten utama
├── posting-linkedin.md # Draft promosi LinkedIn
└── images/            # Folder khusus aset visual
```

### Front Matter Standards (Archetypes)
Gunakan `archetypes/blog/index.md` sebagai template:
- **Categories:** Gunakan `Database`, `DevOps`, `Odoo`, `Web Dev`.
- **Tags:** Gunakan keyword spesifik seperti `docker`, `postgresql`, `linux`.
- **Draft:** Set ke `true` saat pengerjaan awal.

### LinkedIn Promotion
Untuk setiap artikel baru, buatkan file `posting-linkedin.md` di folder artikel tersebut dengan kriteria:
- Maksimal 50 kata.
- Tone: Startup/Agresif/Solutif.
- Maksimal 3 icon.
- Fokus pada *Problem -> Solution -> Link*.

## 🛠️ Konteks DevOps & Server (Odoo/PostgreSQL)
Jika Lord meminta bantuan terkait server/DevOps:
- **OS:** Linux (Debian/Fedora).
- **Networking:** Jelaskan secara awam namun teknis tepat.
- **Python Logic:** Gunakan Python untuk menjabarkan logika pemrograman kompleks.

## Additional Instructions

- jika saya perintah untuk buat konten posting linkedin. gunakan kriteria berikut:
  - buatkan sebuah file dengan nama posting-linkedin.md di folder artikel yang baru dibuat sejajar dengan index.md 
  - buatkan dengan gaya bahasa startup dengan audience milienial & gen z
  - konten posting tidak boleh lebih dari 50 kata

---
*Dibuat oleh Gemini CLI sebagai asisten setia Lord Rohimoz28.*
