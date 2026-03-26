+++
title = 'Setup Formatter Odoo'
date = '2026-03-26T13:51:58+07:00'
draft = false
description = 'Tutorial Setup Formatter untuk Odoo di VSCode dan PyCharm'
categories= ['Odoo']
tags = ['python', 'odoo']
+++

# Standarisasi Code Formatting Python dengan Black, isort, dan Flake8

> Setup lengkap untuk VSCode dan PyCharm pada project Odoo v17

---

## Overview

Ketika tim developer bekerja pada codebase yang sama, perbedaan gaya penulisan kode antar individu dapat menjadi sumber noise yang tidak perlu — mulai dari perbedaan indentasi, urutan import, hingga panjang baris kode. Hal ini membuat code review menjadi lebih sulit dan git diff menjadi lebih berisik.

Solusinya adalah menggunakan **code formatter** dan **linter** yang dijalankan secara konsisten di semua environment developer. Artikel ini menjelaskan setup **Black**, **isort**, dan **Flake8** di VSCode dan PyCharm, khususnya untuk project Odoo v17 yang berjalan di atas virtual environment (`.venv`).

---

## Mengapa Black, isort, dan Flake8?

### Black

Black adalah _opinionated code formatter_ untuk Python. "Opinionated" berarti Black tidak memberi banyak pilihan konfigurasi — ia memformat kode dengan satu cara yang konsisten tanpa perlu debat style di dalam tim. Black menangani indentasi, spasi, tanda kurung, dan struktur kode secara otomatis.

**Alasan memilih Black:**

- Zero-config, langsung jalan tanpa banyak konfigurasi
- Didukung secara native oleh VSCode dan PyCharm
- Deterministik — output selalu sama untuk input yang sama
- Aktif digunakan di ekosistem Python modern

### isort

isort secara otomatis mengurutkan dan mengelompokkan import statement sesuai standar PEP8 — memisahkan antara standard library, third-party packages, dan local imports.

**Alasan memilih isort:**

- Mengeliminasi debat urutan import di code review
- Kompatibel penuh dengan Black menggunakan profil `black`
- Ringan dan cepat

### Flake8

Flake8 adalah linter yang mendeteksi error sintaks, style violation, dan potensi bug di kode Python tanpa mengubah file.

**Alasan memilih Flake8:**

- Deteksi error dasar tanpa overhead tinggi
- Mudah dikonfigurasi
- Cocok sebagai safety net setelah Black dan isort jalan

---

## Prasyarat

Sebelum memulai setup, pastikan kondisi berikut terpenuhi:

- Python 3.11 terinstall di sistem
- VSCode dengan koneksi ke environment (local atau Remote SSH)
- PyCharm versi terbaru

---

## Langkah 1: Menyiapkan Virtual Environment

Virtual environment memastikan setiap project memiliki dependency yang terisolasi satu sama lain. Formatter diinstall di sini, bukan di Python system maupun di dalam Docker container.

### 1.1 Cek Versi Python

```bash
python3 --version
```

Pastikan output menunjukkan Python 3.11.x. Jika belum terinstall:

```bash
# Debian/Ubuntu
sudo apt install python3.11 python3.11-venv
```

### 1.2 Buat Virtual Environment di Root Project

```bash
# Masuk ke root project
cd /path/to/project-root

# Buat virtual environment dengan nama .venv
python3.11 -m venv .venv
```

Setelah perintah ini selesai, akan muncul folder `.venv` di dalam root project. Jangan lupa masukkan ke `.gitignore`:

```bash
echo ".venv/" >> .gitignore
```

### 1.3 Aktifkan Virtual Environment

```bash
# Linux/Mac
source .venv/bin/activate

# Windows
.venv\Scripts\activate
```

---

## Langkah 2: Install Package di Virtual Environment

Pastikan `.venv` sudah aktif (ada tanda `(.venv)` di terminal), lalu install ketiga package utama:

```bash
# Install formatter dan linter
pip install black isort flake8
```

Untuk kemudahan tim, pisahkan dependency formatter dari dependency aplikasi menggunakan file terpisah `requirements-dev.txt`:

```text
black
isort
flake8
```

---

## Langkah 3: (Opsional) Install Odoo Core untuk Autocomplete

Jika anda menggunakan Odoo di dalam Docker, linter IDE (seperti Pylance) tidak akan menemukan library `odoo`. Instal core Odoo ke dalam `.venv` lokal agar fitur *autocomplete* dan *Go to Definition* berjalan lancar.

**Penting:** Ini hanya untuk kebutuhan IDE di lokal, eksekusi aplikasi tetap berjalan di dalam Docker.

```bash
# 1. Instal library pendukung Odoo (versi binary)
pip install psycopg2-binary gevent lxml_html_clean

# 2. Instal core Odoo v17.0 (Tanpa menarik dependensi sistem yang berat)
pip install --no-deps https://github.com/odoo/odoo/archive/17.0.zip

# 3. Instal library Python minimal yang dibutuhkan untuk indexing
pip install urllib3 Jinja2 MarkupSafe PyPDF2 chardet cryptography decorator \
docutils geoip2 num2words ofxparse passlib polib psutil reportlab requests \
vobject Werkzeug XlsxWriter xlwt zeep
```

---

## Langkah 4: Buat File Konfigurasi Project

Agar semua developer memiliki standar yang sama, kita perlu membuat file konfigurasi di root project.

### 4.1 `pyproject.toml` (Black & isort)

Buat file `pyproject.toml` di root project:

```toml
[tool.black]
line-length = 120
target-version = ["py311"]
exclude = '''
/(
    migrations
  | __pycache__
  | .git
)/
'''

[tool.isort]
profile = "black"
line_length = 120
skip = ["migrations"]
```

### 4.2 `.flake8` (Flake8)

Buat file `.flake8` di root project (Flake8 tidak menggunakan format TOML):

```ini
[flake8]
max-line-length = 120
exclude = migrations,__pycache__,.git
extend-ignore = E203,W503
```

---

## Struktur Project Akhir

Setelah mengikuti Langkah 1-4, struktur project anda akan terlihat seperti ini:

```text
project-root/
├── .venv/                  ← virtual environment
├── .vscode/
│   └── settings.json       ← konfigurasi VSCode (Langkah 5)
├── .flake8                 ← konfigurasi Flake8
├── pyproject.toml          ← konfigurasi Black dan isort
├── requirements-dev.txt    ← daftar dev dependencies
└── docker-compose.yml
```

---

## Langkah 5: Setup di VSCode

### 5.1 Install Extension

Buka VSCode → `Ctrl+Shift+X` → install extension resmi dari **Microsoft**:
- **Black Formatter**
- **isort**
- **Flake8**

### 5.2 Konfigurasi `.vscode/settings.json`

Buat file `.vscode/settings.json` agar formatter berjalan otomatis saat save:

```json
{
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },
  "flake8.args": ["--config", "${workspaceFolder}/.flake8"],
  "isort.args": ["--settings-path", "${workspaceFolder}"],
  "black-formatter.args": ["--config", "${workspaceFolder}/pyproject.toml"]
}
```

### 5.3 Set Python Interpreter

- Tekan `Ctrl+Shift+P` → ketik **"Python: Select Interpreter"**
- Pilih interpreter dari folder `.venv` project: `./.venv/bin/python`

---

## Langkah 6: Setup di PyCharm

### 6.1 Set Python Interpreter

- `Settings` → `Project` → `Python Interpreter` → **Add Local Interpreter**
- Arahkan ke `.venv/bin/python` (Linux) atau `.venv/Scripts/python.exe` (Windows).

### 6.2 Integrasi Black (via Plugin BlackConnect)

- Install plugin **BlackConnect** dari Marketplace.
- Di `Settings` → `Tools` → `BlackConnect`, centang **Trigger on save** dan arahkan **Path to pyproject.toml** ke file di root project.

### 6.3 Integrasi isort & Flake8 (via External Tools)

Gunakan **External Tools** di PyCharm untuk menjalankan `isort` dan `flake8` yang ada di dalam `.venv`. anda bisa membuat **Macro** agar `isort` berjalan otomatis saat `Ctrl+S`.

---

## Ringkasan Konfigurasi per Editor

|Fitur|VSCode|PyCharm|
|---|---|---|
|Black auto on save|✅ via `formatOnSave`|✅ via BlackConnect|
|isort auto on save|✅ via `organizeImports`|✅ via Macro|
|Flake8|✅ highlight realtime|⚠️ manual via External Tools|
|Sumber konfigurasi|`pyproject.toml` + `.flake8`|`pyproject.toml` + `.flake8`|

---

## File yang Perlu Di-commit ke Repository

```
project-root/
├── .vscode/
│   └── settings.json       ← commit: membantu pengguna VSCode
├── .flake8                 ← commit: konfigurasi flake8
├── pyproject.toml          ← commit: konfigurasi black & isort
└── requirements-dev.txt    ← commit: daftar dev dependencies
```

Dengan meng-commit file-file ini, setiap anggota tim baru hanya perlu:

```bash
# Clone repo, buat venv, lalu:
pip install -r requirements-dev.txt
```

Dan semua formatter langsung siap digunakan di editor masing-masing.

---

## Apa yang Perlu Dilakukan (Quick Start)

Setelah konfigurasi di atas selesai disiapkan, setiap developer hanya perlu melakukan 3 hal ini di VSCode mereka:

1. **Install Extensions:** Pastikan extension berikut dari **Microsoft** sudah terinstall: `Black Formatter`, `isort`, dan `Flake8`.
2. **Pilih Python Interpreter:** Tekan `Ctrl+Shift+P` → ketik **"Python: Select Interpreter"** → pilih interpreter yang mengarah ke `./.venv/bin/python`.
3. **Uji Coba:** Buka file Python, buat sedikit berantakan (misal spasi berlebih), lalu simpan (`Ctrl+S`). Kode akan otomatis rapi, import terurut, dan linter akan memberikan highlight jika ada error.

---

## Kesimpulan

Kombinasi Black + isort + Flake8 memberikan keseimbangan antara kemudahan setup dan code quality enforcement yang cukup untuk tim kecil. Black menangani formatting otomatis tanpa debat, isort menjaga kerapian import, dan Flake8 menjadi safety net untuk mendeteksi error dasar.

Dengan satu `pyproject.toml` dan satu `.flake8` di root project, seluruh tim dapat menggunakan konfigurasi yang identik regardless editor yang digunakan.