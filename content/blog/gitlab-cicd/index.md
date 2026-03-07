+++
title = 'Implementasi CI/CD menggunakan GitLab & GitLab Runner Untuk Project Odoo'
date = '2026-03-07T23:07:42+07:00'
draft = false
description = ''
categories= ['DevOps']
tags = ['gitlab', 'odoo']
+++
# Membangun CI/CD Pipeline untuk Odoo dengan GitLab dan Shell Executor

> Panduan konseptual membangun pipeline deployment otomatis yang aman, ringan, dan efisien untuk aplikasi Odoo berbasis Docker.

---

## Pendahuluan: Masalah yang Diselesaikan

Dalam lingkungan pengembangan yang aktif, proses deployment manual adalah sumber masalah yang konsisten. Developer harus masuk ke server, menarik kode terbaru, lalu merestart container secara manual — sebuah rutinitas yang memakan waktu, rawan human error, dan tidak terdokumentasi dengan baik.

Pipeline CI/CD yang dirancang dalam artikel ini menjawab permasalahan tersebut dengan mengotomasi seluruh alur: mulai dari push kode ke repository, hingga kode tersebut berjalan di server staging. Tanpa intervensi manual, tanpa risiko lupa langkah, dan dengan jejak audit yang jelas di setiap eksekusi.

---

## Arsitektur: Dua Server, Satu Alur

Skema ini memisahkan tanggung jawab ke dalam dua server yang memiliki peran berbeda.

**Server A** adalah server aplikasi. Di sinilah Odoo dan PostgreSQL berjalan di dalam container Docker. Server ini adalah target deployment — tempat kode akhirnya dieksekusi.

**Server B** adalah server GitLab sekaligus tempat GitLab Runner berjalan. Server ini berperan sebagai orkestrator: menerima trigger dari GitLab, lalu memerintahkan Server A untuk memperbarui dan merestart aplikasi.

Komunikasi antara keduanya dilakukan melalui SSH dengan autentikasi berbasis key pair, bukan password. Ini adalah fondasi keamanan dari seluruh skema.

```
Developer → push ke GitLab (Server B)
                    ↓
           GitLab Runner terpicu
                    ↓
        Runner SSH ke Server A
                    ↓
     git pull + docker compose restart
                    ↓
         Odoo versi terbaru berjalan ✅
```

---

## Mengapa Shell Executor?

GitLab Runner mendukung beberapa jenis executor: Docker, Kubernetes, SSH, dan Shell. Pemilihan **Shell Executor** dalam skema ini bukan tanpa alasan.

### Karakteristik Shell Executor

Shell Executor menjalankan job pipeline langsung di shell sistem operasi tempat Runner berjalan, tanpa lapisan isolasi tambahan seperti container. Ini membuatnya berbeda secara fundamental dari Docker Executor yang membuat container baru untuk setiap job.

### Alasan Memilih Shell Executor

**Kesederhanaan infrastruktur** adalah alasan utama. Shell Executor tidak membutuhkan Docker-in-Docker, tidak perlu konfigurasi network tambahan antara container Runner dan target server, dan tidak memerlukan image Docker khusus sebagai environment job.

**Akses langsung ke sistem host** membuatnya ideal untuk skenario di mana job perlu berinteraksi dengan resource di luar container, seperti melakukan SSH ke server lain. Dengan Docker Executor, hal ini membutuhkan konfigurasi tambahan yang kompleks.

**Overhead minimal** menjadikannya pilihan tepat untuk server dengan resource terbatas. Tidak ada waktu yang terbuang untuk pulling image atau spinning up container sebelum job dieksekusi.

**Stabilitas koneksi SSH** lebih terjamin karena job berjalan langsung di host network, tanpa melewati layer network Docker yang bisa menambah kompleksitas routing ke Server A.

### Trade-off yang Perlu Dipahami

Shell Executor tidak memiliki isolasi environment antar job. Artinya, dependency yang diinstal oleh satu job bisa memengaruhi job lainnya. Untuk use case deployment sederhana seperti ini — yang hanya membutuhkan SSH client — trade-off tersebut tidak relevan dan dapat diabaikan.

---

## Benefit dan Nilai Lebih Skema Ini

### 1. Keamanan Berlapis

Skema ini tidak menggunakan password di mana pun dalam alur otomatisasi. Seluruh autentikasi berbasis SSH key pair yang dikelola secara terpisah: key untuk akses GitLab (Deploy Key) berbeda dengan key untuk akses Server A (Runner Key). Kompromi pada satu layer tidak otomatis membahayakan layer lainnya.

### 2. Prinsip Least Privilege

User `deployer` di Server A dibuat khusus dengan akses yang sangat terbatas: hanya bisa melakukan `git pull` di folder project tertentu dan menjalankan `docker compose`. Tidak ada akses root, tidak ada akses ke folder lain. Ini meminimalkan blast radius jika terjadi insiden keamanan.

### 3. Secret Management yang Benar

Private key SSH Runner tidak pernah ditulis di file konfigurasi pipeline (`.gitlab-ci.yml`) yang tersimpan di repository. Ia disimpan sebagai masked variable di GitLab CI/CD Variables, sehingga tidak pernah muncul di log pipeline maupun riwayat commit.

### 4. Reproducible Deployment

Setiap deployment mengikuti langkah yang identik dan terdokumentasi di file `.gitlab-ci.yml`. Tidak ada lagi perbedaan antara "cara deploy si A" dan "cara deploy si B" — semua mengikuti satu script yang sama.

### 5. Audit Trail

Setiap eksekusi pipeline tercatat di GitLab: siapa yang trigger, kapan dijalankan, berapa lama, dan apakah berhasil. Ini sangat berguna untuk troubleshooting dan compliance.

### 6. Zero Downtime pada Proses Deployment

Proses `docker compose restart` hanya merestart container yang bersangkutan, tidak menyentuh infrastructure lainnya. Downtime terbatas hanya pada waktu restart container Odoo itu sendiri.

---

## Skenario di Mana Skema Ini Bernilai Tinggi

### Tim Kecil dengan Resource Server Terbatas

Skema ini tidak membutuhkan Kubernetes, tidak perlu container registry, dan tidak memerlukan server CI/CD terpisah yang bertenaga. Sangat cocok untuk tim dengan 2–10 developer yang mengelola staging environment dengan budget infrastruktur terbatas.

### Aplikasi dengan Siklus Iterasi Cepat

Odoo sebagai ERP sering mengalami customisasi yang intensif, terutama di fase awal implementasi. Dengan pipeline ini, developer bisa push ke branch `testing` dan melihat hasil perubahan di staging dalam hitungan menit, bukan jam.

### Lingkungan Staging yang Diakses Banyak Stakeholder

Ketika staging environment digunakan untuk demo ke klien atau user acceptance testing (UAT), konsistensi versi yang berjalan menjadi krusial. Pipeline ini memastikan staging selalu mencerminkan kode di branch `testing` yang terbaru.

### Migrasi dari Deployment Manual

Bagi tim yang sebelumnya deploy secara manual via SSH, skema ini adalah jembatan yang natural. Konsepnya familiar (SSH + git pull + restart), hanya diotomasi dan diamankan dengan lapisan tambahan.

### Setup On-Premise atau Private Network

Karena komunikasi hanya bergantung pada SSH antar server di jaringan yang sama, skema ini bekerja sempurna di lingkungan on-premise atau private network tanpa membutuhkan koneksi internet dari server ke layanan cloud eksternal.

---

## Konsep Step-by-Step

### Step 1 — Setup User `deployer` di Server A

Prinsip dasar step ini adalah **isolation of concern**. Dibuat user sistem khusus bernama `deployer` yang tidak memiliki password login, artinya tidak bisa diakses secara langsung kecuali melalui SSH key yang terdaftar.

User ini kemudian diberikan keanggotaan di dua grup yang relevan: grup project (untuk akses baca-tulis ke folder Odoo) dan grup `docker` (untuk menjalankan perintah Docker tanpa sudo). Folder project Odoo juga dikonfigurasi agar mewarisi permission grup secara otomatis untuk setiap file baru yang dibuat oleh `git pull`.

Terakhir, folder `.ssh` disiapkan dengan permission yang ketat sesuai standar SSH — ini adalah syarat mutlak agar autentikasi berbasis key bisa berfungsi.

### Step 2 — Generate dan Register Deploy Key di GitLab

User `deployer` di Server A perlu identitas SSH yang dikenali oleh GitLab agar bisa melakukan `git pull` dari repository private. Identitas ini berupa SSH key pair yang di-generate khusus untuk user `deployer`.

**Public key** dari key pair ini kemudian didaftarkan di GitLab Project sebagai **Deploy Key** melalui menu Settings → Repository → Deploy Keys. Deploy Key adalah mekanisme GitLab yang memberikan akses read (atau read-write) ke satu repository spesifik, tanpa memberikan akses ke seluruh akun GitLab. Ini jauh lebih aman dibanding menggunakan Personal Access Token atau akun pengguna aktif.

Jika pipeline juga perlu melakukan push (misalnya untuk update versi), opsi "Write access allowed" pada Deploy Key perlu diaktifkan. Untuk use case deployment sederhana yang hanya membutuhkan `git pull`, akses read sudah cukup.

### Step 3 — Setup SSH Key Pair dari Server B ke Server A

Ini adalah mekanisme yang memungkinkan GitLab Runner di Server B untuk "mengetuk pintu" Server A tanpa password. Di Server B, dibuat SSH key pair baru atas nama user `gitlab-runner`.

**Public key** dari key pair ini disalin ke file `authorized_keys` milik user `deployer` di Server A. Dengan demikian, setiap koneksi SSH yang menggunakan private key yang cocok akan langsung diterima oleh Server A tanpa diminta password.

Koneksi ini kemudian diverifikasi secara manual sebelum melanjutkan ke step berikutnya — memastikan seluruh konfigurasi permission dan network sudah benar sebelum diintegrasikan ke pipeline otomatis.

### Step 4 — Simpan Private Key Runner ke GitLab CI/CD Variables

**Private key** dari Server B tidak boleh ditulis hardcoded di file pipeline. Solusinya adalah menyimpannya sebagai **masked variable** di GitLab CI/CD Variables.

Masked variable berarti nilai tersebut tidak akan pernah muncul di log pipeline, bahkan jika ada perintah yang tidak sengaja mencetaknya. Variable ini kemudian bisa dipanggil di dalam script pipeline menggunakan nama variabelnya, bukan nilai aslinya.

### Step 5 — Setup GitLab CI/CD Pipeline

File `.gitlab-ci.yml` adalah deklarasi pipeline: mendefinisikan kapan pipeline berjalan (trigger pada push ke branch `testing`), apa yang dilakukan (setup SSH, koneksi ke Server A, jalankan git pull dan restart container), dan variable apa yang digunakan.

Parameter non-sensitif seperti IP server, nama user, dan nama container ditulis langsung di file ini karena tidak mengandung rahasia. Hanya private key yang diambil dari CI/CD Variables. Pipeline kemudian diverifikasi melalui menu CI/CD → Pipelines di GitLab untuk memastikan seluruh job berjalan dengan status `passed`.

---

## Penutup

Skema CI/CD ini adalah bukti bahwa otomasi deployment yang aman tidak selalu membutuhkan infrastruktur yang kompleks. Dengan memahami konsep SSH key pair, prinsip least privilege, dan secret management yang benar, sebuah pipeline yang handal bisa dibangun di atas dua server biasa.

Nilai sesungguhnya dari skema ini bukan hanya pada kecepatan deployment, melainkan pada **konsistensi, keamanan, dan kepercayaan** bahwa setiap perubahan kode akan sampai ke staging dengan cara yang terprediksi dan teraudit — setiap saat, tanpa terkecuali.