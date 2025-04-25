+++
title = 'Cara Backup Odoo Dan Postgresql'
date = '2025-01-25T19:55:30+07:00'
draft = false
description = 'Jika Backup Odoo Melalui Front End Sudah Tidak Bisa Di Lakukan. Mungkin Backup Manual Postgresql dan Odoo Filestore Bisa Menjadi Opsi.'
categories= ['Web Dev']
tags = ['odoo']
+++

### Latar Belakang Masalah
Biasanya, kita dapat melakuakn backup odoo berikut database nya melalui front end aplikasi. Tapi, ketika
ukuran database sudah besar (file compress > 2GB) akan timbul masalah gagal backup. Biasanya karena faktor koneksi
internet yang tidak stabil. 

### Environment
Dalam contoh kasus kali ini, penulis menggunakan Odoo dan Postgresql yang dijalankan menggunakan docker. Pada umumnya, cara yang diguanakan jika
tidak memnggunakan docker adalah sama. Hanya beda syntax.

### Solusi
Untuk latar belakang masalah yang dikemukakan diatas, perlu mekanisme backup lain untuk mengakomodasi database Odoo yang tidak bisa
di backup dari front end.
- manual backup postgresql
- manual backup odoo filestore

### Cara Backup Postgresql
- Jalankan pg_dump di dalam container postgres dan letakkan file backup di folder /tmp \
`docker exec CONTAINER pg_dump -U DB_USER -F c -f /tmp/BACKUP_NAME.dump DB_NAME` \
`docker exec postgres pg_dump -U odoo -F c -f /tmp/db_backup.dump DB_TEST`
- compres file dump menggunakan gzip \
`docker exec postgres gzip /tmp/db_backup.dump`
- copy file dump yang telah di compres ke local \
`docker cp postgres:/tmp/db_backup.dump.gz path/to/local/dir/backup`
- hapus file backup di dalam container \
`docker exec postgres rm /tmp/db_backup.dump.gz`

### Cara Backup Odoo Filestore
- buat file zip untuk odoo filestore \
`docker exec CONTAINER tar -zcvf /var/lib/odoo/filestore/DB_NAME.tar.gz /var/lib/odoo/filestore/DB_NAME` \
`docker exec odoo tar -zcvf /var/lib/odoo/filestore/db_backup.tar.gz /var/lib/odoo/filestore/db_backup`
- pindahkan file zip tersebut ke folder backup lokal \
`docker cp odoo:/var/lib/odoo/filestore/db_backup.tar.gz path/to/local/dir/backup`
- hapus file backup di dalam container \
`docker exec odoo rm /var/lib/odoo/filestore/db_backup.tar.gz`
